ESP32 MP3 decoder
=======================
This is a simple web radio streamer. It connects to a web radio station via wifi, decodes the MP3 stream and outputs the audio data to an I2S codec or
to an attached speaker.

This project is a port of Sprite\_TM's awesome MP3 web radio project for the ESP8266: https://github.com/espressif/ESP8266_MP3_DECODER

## What Changed

The SPI RAM is not needed anymore, the ESP32 has enough memory by itself.
We can also use the built-in 8Bit DAC instead of the external I2S codec.

## Required Software

Get the SDK:

    git clone https://github.com/espressif/esp-idf.git
    cd esp-idf
    git submodule update --init

Set the IDF_PATH environment variable, and point it to this directory.

    export IDF_PATH=/path/to/esp-idf

Download the toolchain from: https://github.com/espressif/esp-idf#setting-up-esp-idf
You will need version 5.2.0.
Add /path/to/xtensa-esp32-elf/bin to your PATH:

    export PATH=/path/to/xtensa-esp32-elf/bin:$PATH

## Configuration

The serial port and wifi credentials are configured using make.
Type 'make menuconfig' and 

* configure your serial port in "Serial flasher config" submenu
* set wifi credentials in the "Wifi Configuration" submenu
* in the "Audio" submenu, choose your output method
* 'save', then exit

To change the radio station URL, modify PLAY_URL in /main/include/playerconfig.h.

## Building
Just type 'make'.

## Flashing
Connect your serial cable and run 'make flash'. To see serial console output run 'make monitor'.

## Controls
You can stop and start playback using the "Boot" button that is present on most development boards.

## UI
You can connect a <a href="https://www.adafruit.com/product/1312">NeoPixel</a> LED to pin 32. Its currently not doing much except blinking while wifi is connecting.

## Connecting the I2S codec

If you don't know about the I2S standard, it is a special protocol for transferring digital audio data between chips, similar to I2C. There are many I2S chips you can choose from, the most important differences are:

1. Amplification: some chips only decode the audio to a low analog level, so you need a separate amp, but some also have a built-in amplifier. Most of these 2-in-1 chips are made for smartphones so their energy output is in the range of 2-4W, but some other ones made for domestic audio appliances can go a lot higher.
2. MCLK: this is a separate clock signal that sometimes needs to be a precise number in the MHz range that depends on the current sample rate, sometimes can be a single constant value ("asynchronous") independent of the current sample rate, and sometimes is not required at all. The ESP32 does not output a MCLK signal, so a chip that does not require MCLK is most convenient. If you already have an asynchronous one lying around (e.g. ES9023), you will need a quartz oscillator, usually in the range of 20-50MHz.

I tested several I2S codecs, and was happiest with the MAX98357A, because it does not require MCLCK and also amplifies the audio to speaker levels. It also seemed to be more immune to signal integrity issues, which do occur on breadboards. There is a convenient breakout board from Adafruit: https://www.adafruit.com/product/3006
However, any I2S codec should work.

Generic wiring:

```
ESP pin   - I2S signal
----------------------
GPIO25/DAC1   - LRCK
GPIO26/DAC2   - BCLK
GPIO22        - DATA
```

If you're using the MAX98357A, connect GND to ground and Vin to +5V (or +3.3V if +5V is unavailable). SD can remain unconnected, and GAIN too unless you want to make it louder or lower. I also recommend using a potentiometer for volume regulation.

## Running without the I2S DAC

The ESP32 has a built-in 8-Bit DAC that we can use.
Run `make menuconfig` and choose "DAC_BUILT_IN" in the "Audio" submenu. Make sure to enable `DAC_BUG_WORKAROUND`, too.

You can now connect a speaker to ground and the pins 25 and 26 for the left and right channels. You should probably add a resistor to avoid overloading the pins.

The ESP32 also supports PDM mode which should give better sound than the 8 bit DAC, but the SDK doesn't support it yet.

## More Information

There is a thread over at the ESP32 forum:
https://esp32.com/viewtopic.php?f=17&t=1026

## Breadboard Example

I used the Watterott ESP-WROOM-32-Breakout, which is pin-compatible to the Espressif Core Board (DevKitC).
Please note that in this picture, the JTAG header is connected too, but you can safely ignore that.

<img src="doc/breadboard_wiring.jpg" width="50%" height="50%">