# Nokia 6610 LCD Hardware interfacing with Arduino to use this library #

Basically - interface the **SPI pins on Arduino** to the LCD Screen's breakout board **through a logic level convertor** circuit .

Most varieties of Arduino operate at 5 V logic levels while the LCD screen and its controller chip operates at 3.3V Logic levels. Therefore, you need to convert the 5V ouptut from Arduino to 3.3V levels by some means. The simplest method is to use a just use a Resistor voltage divider. Something like the following works quite well -

**Arduino OUT -> 3.3K Resistor ->Connect to LCD Screen IN ->5.6K Resistor->GND**

You also need to provide power, ground etc. to the LCD screen. Details will vary depending on the breakout board you use. I will be adding more details on this in the future in this wiki.

The arduino pins to connect to are:
|CLK |SPI Clock Line|connect to Arduino Pin 13|
|:---|:-------------|:------------------------|
|MOSI|Master Out Slave In | connect to Arduino Pin 11- LCD screen is slave|
|CS |Chip Select| connect to Arduino Pin 10|
|RST|LCD Reset line|connect to Arduino Pin 9|

NOTE: **DO NOT** connect anything to **pin 12** - this is the MISO pin which is not being used for SPI here.

<img src='http://ardgrafix6100.googlecode.com/files/xyzzy.jpg'></img>