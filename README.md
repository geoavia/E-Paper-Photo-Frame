# 7 color e-paper photo frame

Video of finished device: https://youtu.be/H_CAdMDsyxA

### Components

* [Arduino Nano compatible board](https://www.amazon.com/dp/B07G99NNXL)
* [7 colour e-paper display panel](https://www.waveshare.com/product/displays/e-paper/epaper-1/5.65inch-e-paper-module-f.htm)
* [Micro SD card board](https://www.amazon.com/gp/product/B07BJ2P6X6)
* Resistors (100k x 2, 100 ohm, 1k)
* Integral circuit SI4542DY (two mosfets package)
* LED
* Push button momentary switch
* 12v Battery (A23)

### e-Paper display connection
According to [waveshare wiki](https://www.waveshare.com/wiki/5.65inch_e-Paper_Module_(F)):<br>
e-Paper display | Arduino
------------ | -------------
Vcc | 5V
GND | GND
DIN | D11
CLK | D13
CS | D10
DC | D9
RST | D8
BUSY | D7

### SD Card board connection
SD card reader | Arduino
------------ | -------------
GND | GND
Vcc | 5V
MISO | D12
MOSI | D11
SCK | D13
CS | D4

### Power off circuit
Power off circuit based on [Indrek Luuk's article](https://circuitjournal.com/arduino-auto-power-off), but using single [SI4542DY IC](https://www.onsemi.com/pdf/datasheet/si4542dy-d.pdf) containing two (P and N channel) mosfets.<br>
![Power off](https://github.com/geoavia/E-Paper-Photo-Frame/blob/main/circuit_pwr.jpg)<br><br>
I have utilized the [TSSOP16](https://www.amazon.com/dp/B00O9W6RLQ) adapter board (although any similar board will do).<br>
![ssop16](https://github.com/geoavia/E-Paper-Photo-Frame/blob/main/ssop16.jpg)<br>

### Inside alignment
![Inside](https://github.com/geoavia/E-Paper-Photo-Frame/blob/main/photo_frame_inside.jpg)<br><br>

Enclosure case 3d model (Fusion 360) and stl files for print [are also provided](https://github.com/geoavia/E-Paper-Photo-Frame/tree/main/enclosure)

### Preparing image files

1. Resize images to resolution of 600x448 pixels
2. Save them according to [this manual](https://www.waveshare.com/wiki/E-Paper_Floyd-Steinberg) (use 7 color version)
3. Convert each file with [converter](https://github.com/geoavia/E-Paper-Photo-Frame/tree/main/converter) utility (compile first)
4. Save resulting BIM files to SD card, obviousely :)
