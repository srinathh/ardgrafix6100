# Overview of Font handling #

Using bitmapped fonts always consumes memory and Arduino models usually  have  very little SRAM available (only 2kb on Uno). This is why most graphics drivers provide merely a basic font and use up much of the SRAM in defining the bitmap array. This approach also ends up leaving very little SRAM for your own program to use.

Arduinos do have more significant amounts of Flash memory (32kb on Uno) but the read/write routines to this are fairly cumbersome to use.

Ardgrafix6100 gets around these limitations by defining bitmapped fonts **programatically** using object oriented programming. That is - when you want to draw a char in a font, the font drawing routine calls the **getcharacter()** method of the font to get the bitmap of the character into a small buffer to draw it on screen. This allows you to easily define multiple fonts in your program - since the **font instructions** are really **code** and sit in the Flash memory.

The font-system also lets you define somewhat **proportional** fonts. Somewhat proportional because _**the height of each font is fixed but width of each character can change**_. This gives you "nicer" text displays & the font drawing routine is smart enough to calculate space by character.

# How to Define your own Font #
This assumes you're familiar with Object Oriented Programming concepts in C++.

To define your own font, simply inherit from the **abstract base** class **genericfont** defined in the file **fonts.h**. Let's look at this class -
```
class genericfont //Abstract Font base class. THis defines minimum features that MUST be common to fonts. 
{
public:
        virtual uint16_t get_max_buf_size();    //this function should return the maximum size of buffer requred to hold a font's mono bitmap
        virtual uint16_t get_character_spacing();       //this function should return the preferred character spacing
        virtual font_return_codes getcharacter(uint8_t asciicode, uint8_t * buf, xy* size);     //this function should return the bitmap of the specified ascii character and its size
};
```

Any class inheriting from this needs to define 3 methods
  * `uint16_t get_max_buf_size()`

since each character can have a variable width, the caller doesn't know the size of uint8\_t (ie. unsigned char) buffer to allocate. The character writing routine gets around this by asking you for the max. buffer size needed for the widest character for this font. You need to calculate number of bits (width x height) of the widest character in your bitmapped font, round it up to the nearest multiple of 8 & return the number of bytes (ie. (width x height-> roundup to multiple of 8)/8. The pre-defined font\_prop\_sans\_height14px\_font specifies that the max buffer size it needs is 20 bytes

```
        uint16_t get_max_buf_size()
        {       return 20;      }
```
  * `uint16_t get_character_spacing()`
used when printing characters to leave a space between letters in pixels. The pre-defined font\_prop\_sans\_height14px\_font specifies spacing of 1 pixel
```
        uint16_t get_character_spacing()
        {       return 1;       };
```
  * `font_return_codes getcharacter(uint8_t asciicode, uint8_t * buf, xy* size)`
This is the core of your font. for every character requested, you return bitmap in the unsigned char (uint8\_t) array  pointed to by the pointer buf. You also return an xy metric in the pointer called size. xy is a utility class defined in the header file graphic\_types.h which simply is used to return size of the character in x & y.
```
// class xy is defined in the file graphic_types.h
class xy
{
protected:
        int16_t x;
        int16_t y;
     .......
}
```

Let's look at how font\_prop\_sans\_height14px\_font defines this function in the code at the bottom. It simply uses a switch statement on the requested ascii code to return a buffer. it first sets the **width** in number of pixels for the character - remember this can vary by character. It then stores a mono-chrome bitmap in the buf. This is simply a set of bits defining each char 1 row at a time.

In this case for char 'A' (ascii 65), we're saying the character is 7 pixels wide. Since the font is 14 pixel high, we need 7\*14 = 98 bits to define this. The next higher multiple of 8 is 104 implying the character needs 13 bytes to define.
![http://ardgrafix6100.googlecode.com/files/customfont.png](http://ardgrafix6100.googlecode.com/files/customfont.png)

At the end, we're simply setting the 'height' (y) parameter of the size to 14 pixels high & returning FONT\_CHARACTER\_EXISTS as the return code. If the character requested was not defined, you can return FONT\_CHARACTER\_DOES\_NOT\_EXIST.

```
font_return_codes font_prop_sans_height14px_font::getcharacter(uint8_t asciicode, uint8_t * buf, xy* size)
{
        switch(asciicode)
        {
        case 32:
                size->setx(3);
                
                buf[0]=0x0;
                buf[1]=0x0;
                buf[2]=0x0;
                buf[3]=0x0;
                buf[4]=0x0;
                buf[5]=0x0;
                break;

              ...............

        case 65:
                size->setx(7);
                
                buf[0]=0x0;
                buf[1]=0x0;
                buf[2]=0x40;
                buf[3]=0x82;
                buf[4]=0x85;
                buf[5]=0x11;
                buf[6]=0x3e;
                buf[7]=0x83;
                buf[8]=0x4;
                buf[9]=0x0;
                buf[10]=0x0;
                buf[11]=0x0;
                buf[12]=0x0;
                break;

              ...............

        case 126:
                size->setx(5);
                
                buf[0]=0x0;
                buf[1]=0x0;
                buf[2]=0xdb;
                buf[3]=0x0;
                buf[4]=0x0;
                buf[5]=0x0;
                buf[6]=0x0;
                buf[7]=0x0;
                buf[8]=0x0;
                break;
        default:
                return FONT_CHARACTER_DOES_NOT_EXIST;
        }
        size->sety(14);
        return FONT_CHARACTER_EXISTS;
}
```