# I2C 40-I/O port expander #

## About ##

* It's a breakout board for the NXP PCA9698
* 40 I/O ports organized in 5 groups of 8
* 3.3v- and 5v-compatible
* Outputs source 10mA, sink 25mA at 5v
* Choose any of 64 I2C addresses, set via AD0-AD2 lines
* 100kHz, 400kHz and 1MHz I2C operation
* Open drain or totem-pole output
* Output enable, reset, and interrupt lines

This board isn't meant to go directly in breadboards. You can solder male or female headers either on the bottom of the board (preferred for installation as a daughter board on main boards) or on the top (preferred for prototyping or wiring to breadboards).

You will need two 2x20 headers and one 2x5 header.

Note that the I2C address is set by the AD0-AD2 lines which get connected to VSS, VDD, SDA, or SCL. See below, or the NXP PCA9698 datasheet section 7.15 for the address map. The table given in the datasheet shows columns A6-A0 and Address for each AD0-AD2 setting. The A6-A0 column gives you the _7-bit address_ that you must use from Arduinos, while the Address column shows the _8-bit address_.

## Files ##

* **i2c-expander-40-schematic.pdf:** The schematic in PDF format.
* **i2c-expander-40-gerbers.zip:** Contains Gerber files for board outline, top/bottom layers, top/bottom soldermask, top/bottom silkscreen, solder paste mask ("cream"), and drills. The files were generated from OSH Park's CAM processor for Eagle.
* **i2c-expander-40.brd, .sch:** Board and schematic files for Eagle (min version 7.5.0)
* **i2c-expander-40.lbr:** Library file containing devices used in the schematic and board, for Eagle (min version 7.5.0)

## AD2-AD0 7-bit address settings ##

Possible addresses are 0x10-0x2F, 0x50-0x67, 0x70-0x77.

Address | AD2 | AD1 | AD0 | Address | AD2 | AD1 | AD0
-------: | --- | --- | --- | -------: | --- | --- | ---
0x10 | Vss | SCL | Vss | 0x50 | Vss | SCL | Vss
0x11 | Vss | SCL | Vdd | 0x51 | Vss | SCL | Vdd
0x12 | Vss | SDA | Vss | 0x52 | Vss | SDA | Vss
0x13 | Vss | SDA | Vdd | 0x53 | Vss | SDA | Vdd
0x14 | Vdd | SCL | Vss | 0x54 | Vdd | SCL | Vss
0x15 | Vdd | SCL | Vdd | 0x55 | Vdd | SCL | Vdd
0x16 | Vdd | SDA | Vss | 0x56 | Vdd | SDA | Vss
0x17 | Vdd | SDA | Vdd | 0x57 | Vdd | SDA | Vdd
0x18 | Vss | SCL | SCL | 0x58 | Vss | SCL | SCL
0x19 | Vss | SCL | SDA | 0x59 | Vss | SCL | SDA
0x1A | Vss | SDA | SCL | 0x5A | Vss | SDA | SCL
0x1B | Vss | SDA | SDA | 0x5B | Vss | SDA | SDA
0x1C | Vdd | SCL | SCL | 0x5C | Vdd | SCL | SCL
0x1D | Vdd | SCL | SDA | 0x5D | Vdd | SCL | SDA
0x1E | Vdd | SDA | SCL | 0x5E | Vdd | SDA | SCL
0x1F | Vdd | SDA | SDA | 0x5F | Vdd | SDA | SDA
0x20 | Vss | Vss | Vss | 0x60 | Vss | Vss | Vss
0x21 | Vss | Vss | Vdd | 0x61 | Vss | Vss | Vdd
0x22 | Vss | Vdd | Vss | 0x62 | Vss | Vdd | Vss
0x23 | Vss | Vdd | Vdd | 0x63 | Vss | Vdd | Vdd
0x24 | Vdd | Vss | Vss | 0x64 | Vdd | Vss | Vss
0x25 | Vdd | Vss | Vdd | 0x65 | Vdd | Vss | Vdd
0x26 | Vdd | Vdd | Vss | 0x66 | Vdd | Vdd | Vss
0x27 | Vdd | Vdd | Vdd | 0x67 | Vdd | Vdd | Vdd
0x28 | Vss | Vss | SCL | 0x70 | Vss | Vss | SCL
0x29 | Vss | Vss | SDA | 0x71 | Vss | Vss | SDA
0x2A | Vss | Vdd | SCL | 0x72 | Vss | Vdd | SCL
0x2B | Vss | Vdd | SDA | 0x73 | Vss | Vdd | SDA
0x2C | Vdd | Vss | SCL | 0x74 | Vdd | Vss | SCL
0x2D | Vdd | Vss | SDA | 0x75 | Vdd | Vss | SDA
0x2E | Vdd | Vdd | SCL | 0x76 | Vdd | Vdd | SCL
0x2F | Vdd | Vdd | SDA | 0x77 | Vdd | Vdd | SDA

## Example commands ##

All values are hex.

### Write commands ###

* `ADDR 98 00 00 00 00 00` Sets the direction of ports 0-4 to output.
* `ADDR 9A 0F` Sets the direction of port 2 bits 3-0 to input.
* `ADDR 09 22` Writes 0x22 to port 1.
* `ADDR 88 11 22 33 44 55` Writes 0x11 to port 0, 0x22 to port 1, 0x33 to port 2, 0x44 to port 3, 0x55 to port 4.
* `ADDR 2A 00` Among other things, sets up the output change mode so that a multiport write causes outputs to change all at once after the command, rather than after each byte.

### Read commands ###

* `ADDR 0x00` Read from port 0. Every read will return the value of port 0.
* `ADDR 0x82` Read starting from port 2. Each read increments the port number, so you could read from port 2, port 3, and port 4 using this command.

## Example Arduino code ##

```C

#include "Wire.h"
#define ADDR 0x10

int setup() {
  Wire.begin(); // Set up Arduino as I2C master

  // Tell board at address 0x10 to set up ports 0-3 for output
  Wire.beginTransmission(ADDR);
  Wire.write({0x98, 0x00, 0x00, 0x00, 0x00}, 5);
  Wire.endTransmission();

  // Tell board at address 0x10 to set up port 4 for input
  Wire.beginTransmission(ADDR);
  Wire.write({0x9C, 0xFF}, 2);
  Wire.endTransmission();
}

int loop() {
  byte data;

  // Write 0x01 to port 0.
  Wire.beginTransmission(ADDR);
  Wire.write({0x08, 0x01}, 2);
  Wire.endTransmission();

  // Read port 4, several times in quick succession
  Wire.beginTransmission(ADDR);
  Wire.write({0x04}, 1);
  Wire.endTransmission();

  Wire.requestFrom(ADDR, 4);
  data = Wire.read();
  data = Wire.read();
  data = Wire.read();
  data = Wire.read();

  delay(1000);

  // Write 0x00 to port 0.
  Wire.beginTransmission(ADDR);
  Wire.write({0x08, 0x00}, 2);
  Wire.endTransmission();

  delay(1000);
}
```

## Further reading ##

* [NXP PCA9698 overview](http://www.nxp.com/products/interface-and-connectivity/interface-and-system-management/i2c/i2c-general-purpose-i-o/40-bit-fm-plus-i2c-bus-advanced-i-o-port-with-reset-oe-and-int:PCA9698)
* [NXP PCA9698 datasheet](http://www.nxp.com/documents/data_sheet/PCA9698.pdf)
* [Arduino Wire Library, Explored](http://playground.arduino.cc/Main/WireLibraryDetailedReference)