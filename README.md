# 7 color e-Paper Photo Frame

![eFrame](https://github.com/geoavia/E-Paper-Photo-Frame/blob/main/eframe2.jpg)

Video of finished device: https://youtu.be/H_CAdMDsyxA

### Components used

* [Arduino Nano compatible board](https://www.amazon.com/dp/B07G99NNXL)
* [7 colour e-paper display panel](https://www.waveshare.com/product/displays/e-paper/epaper-1/5.65inch-e-paper-module-f.htm)
* [Micro SD card board](https://www.amazon.com/gp/product/B07BJ2P6X6)
* Resistors (100k x 2, 100 ohm, 1k)
* Integral circuit SI4542DY or similar (two mosfets in package).
* One LED, color of your preferance :)
* Single push button momentary switch
* 7v-12v battery pack

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
Power off circuit based on [Indrek Luuk's article](https://circuitjournal.com/arduino-auto-power-off), but using single [SI4542DY IC](https://www.onsemi.com/pdf/datasheet/si4542dy-d.pdf) containing two (N and P channel) mosfets.<br>
![Power off](https://github.com/geoavia/E-Paper-Photo-Frame/blob/main/circuit_pwr.jpg)<br><br>
I have utilized the [TSSOP16](https://www.amazon.com/dp/B00O9W6RLQ) adapter board (although any similar board will do).<br>
* [Adapter board and SI4542DY](https://github.com/geoavia/E-Paper-Photo-Frame/blob/main/ssop16_and_chip.jpg)
* [Circuit sheme on adapter board](https://github.com/geoavia/E-Paper-Photo-Frame/blob/main/ssop16.jpg)
* [Assembled power off circuit board](https://github.com/geoavia/E-Paper-Photo-Frame/blob/main/ssop16_assembled.jpg)

### Inside alignment
![Inside](https://github.com/geoavia/E-Paper-Photo-Frame/blob/main/insides.jpg)<br><br>

Enclosure case 3d model (Fusion 360) and stl files for print [are also provided](https://github.com/geoavia/E-Paper-Photo-Frame/tree/main/enclosure)<br>
You may also need some [brass threaded inserts](https://www.amazon.com/dp/B01IYWTCWW) for screen mounting (holes already cut in 3d model)

### Arduino sketch

[Complete Sketch folder](https://github.com/geoavia/E-Paper-Photo-Frame/tree/main/epaper7c)<br>

<details>
<summary>
Main sketch code here
</summary>

```C++
#include <SPI.h>
#include <SD.h>
#include "epd5in65f.h"

#define LOG_NAME "/log.inc"
#define MAX_NAME 32

// Shut down activate pin
const int shutDownPin = 2;

void setup() {

  pinMode(shutDownPin, OUTPUT);
  digitalWrite(shutDownPin, HIGH);   // power on

  // put your setup code here, to run once:
  Serial.begin(9600);

  Serial.print("Initializing SD card...");
  if (!SD.begin(4)) {
    Serial.println("failed!");
    delay(5000);
    exit(1);
  }
  Serial.println("done.");

  char imgFileName[MAX_NAME];
  unsigned long logSize = 0;

  memset(imgFileName, 0, MAX_NAME);

  File logFile = SD.open(LOG_NAME, FILE_WRITE);
  if (logFile) {
    logSize = logFile.size();
    Serial.print("Log size: ");
    Serial.println(logSize);
    logFile.write('.');
    logFile.close();
  }

  NextFileName(logSize, imgFileName);

  File imgFile = SD.open(imgFileName, FILE_READ);
  if (imgFile) {
    unsigned long fs = imgFile.size();
    Serial.print("Show image: ");
    Serial.print(imgFileName);
    Serial.print(" (");
    Serial.print(fs);
    Serial.println(" bytes)...");

    int x = 0;
    int y = 0;
    UBYTE bp;

    /////////////////////////////////////////////
    // start e-Paper code
    
    Epd epd;
    if (epd.Init() == 0) {
      Serial.println("e-Paper: init ok");

      epd.Clear(EPD_5IN65F_WHITE);
      Serial.println("e-Paper: clear");
      
      epd.SendCommand(0x61);//Set Resolution setting
      epd.SendData(0x02);
      epd.SendData(0x58);
      epd.SendData(0x01);
      epd.SendData(0xC0);
      epd.SendCommand(0x10);
  
      while (imgFile.available()) {
        bp = imgFile.read();
        epd.SendData(bp);
        x++;
        if (x == 300) {
          y++;
          x = 0;
        }
      }
  
      epd.SendCommand(0x04);//0x04
      epd.EPD_5IN65F_BusyHigh();
      epd.SendCommand(0x12);//0x12
      epd.EPD_5IN65F_BusyHigh();
      epd.SendCommand(0x02);  //0x02
      epd.EPD_5IN65F_BusyLow();
  
      Serial.println("e-Paper: image sent");

      epd.Sleep();
      Serial.println("e-Paper: sleep");
  
      // end e-Paper code
      /////////////////////////////////////////////
    } else {
      Serial.println("e-Paper: init failed!");
    }
    
    imgFile.close();
  } else {
    Serial.print("error opening image file: ");
    Serial.println(imgFileName);
    exit(1);
  }

  Serial.println("Complete");
  delay(2000);
}

void loop() {
  delay(5000);
  digitalWrite(shutDownPin, LOW);   // power off
}

void NextFileName(unsigned long logSize, char *fname) {
  int count = 0;
  File dir = SD.open("/");
  File file;
  // first loop for counting files...
  while (file = dir.openNextFile()) {
    if (file.isDirectory() || file.size() != 134400) {
      file.close();
      continue;
    }
    Serial.println(file.name());
    file.close();
    count++;
  }
  Serial.print("Total images: ");
  Serial.println(count);
  dir.rewindDirectory();
  logSize = logSize % count;
  count = 0;
  // ... and second loop for finding next file. I know its's wierd but it works :) 
  while (file = dir.openNextFile()) {
    if (file.isDirectory() || file.size() != 134400) {
      file.close();
      continue;
    }
    if (logSize == count) {
      strcpy(fname, file.name());
    }
    file.close();
    count++;
  }
  Serial.print("Current index: ");
  Serial.println(logSize);
  dir.close();  
}
```
</details>


### Preparing image files

1. Resize images to resolution of 600x448 pixels
2. Save them according to [this manual](https://www.waveshare.com/wiki/E-Paper_Floyd-Steinberg) (use 7 color version)
3. Convert each file with [converter](https://github.com/geoavia/E-Paper-Photo-Frame/tree/main/converter) utility (compile first)
4. Save resulting BIM files to SD card, obviousely :)
