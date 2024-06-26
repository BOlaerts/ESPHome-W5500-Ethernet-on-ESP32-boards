# Introduction
Playing around with the ESPHOME and W5500 ethernet module on different ESP32 boards.
Goal is to be able to use SPI for ethernet and I²C for other sensors or extensions.

# Table of contents
1. [ ESP32C3 Super Mini ](#esp32c3supermini)
2. [ SEEED XIAO ESP32C3 ](#seeedxiaoesp32c3)
3. [ LILYGO T-ETH-Lite ](#lilygotethlite)

<a name="esp32c3supermini"></a>
# ESP32C3 Super Mini

## Hardware wiring schema
<img src="/../main/pictures/esp32c3-super-mini-w5500-bmp280.png" width="40%" alt= "Wiring" height="40%">

And even with this small sized board, there is some room for extra connections!

Possible improvements to investigate:
- CLK wire for the W5500 SPI bus is set to GPIO08
- GPIO08 is also the linked to the status led of the ESP32 C3 super mini development board
- possible or better to use GPIO02 instead?
- update 18.01.2024: switching CLK to GPIO02 did not seem to work...

## ESPHome config
At first, I only got it to work with the arduino framework:
``` yaml
esphome:
  name: esp32c3-super-mini
  friendly_name: esp32c3-super-mini

esp32:
  board: esp32-c3-devkitm-1
  framework:
    type: arduino

# Enable logging
logger:
  level: DEBUG

# Enable Home Assistant API
api:

ota:

ethernet:
  type: W5500
  mosi_pin: GPIO10
  miso_pin: GPIO09
  clk_pin: GPIO08
  cs_pin: GPIO5
  reset_pin: GPIO04
  interrupt_pin: GPIO03
  clock_speed: 25MHz

i2c:
  scl: GPIO07
  sda: GPIO06
  scan: True
  id: bus_a

sensor:
  - platform: bmp280
    temperature:
      name: "Temperature"
      unit_of_measurement: °C
      accuracy_decimals: 1
    pressure:
      name: "Pressure"
      unit_of_measurement: hPa
      accuracy_decimals: 0
    i2c_id: bus_a
    address: 0x76
```

After some iterations, it also works with the esp-idf framework; some tweaking of the platformio options was necessary:
``` yaml
esphome:
  name: esp32c3-super-mini-idf
  friendly_name: esp32c3-super-mini-idf
  platformio_options:
    board_build.flash_mode: dio
    board_build.mcu: esp32c3
    board_build.f_cpu: 160000000L

esp32:
  board: esp32-c3-devkitm-1
  framework:
    type: esp-idf
```

So far, connection seems to be lightning fast and super stable.

In my experience, the easiest way of flashing the new ESP32C3 boards is by using the [Adafruit ESPTool](https://adafruit.github.io/Adafruit_WebSerial_ESPTool/).


## PCB design
Work in progress on a pcb design for a small board with ethernet connection and exposure of all remaining GPIOs.

See PCB folder for first design attempt.

<img src="/../main/pictures/Schematic_ESP32-C3-SuperMini W5500 Ehternet I2C_2024-01-16.png" width="50%" alt= "Schematic" height="50%">

<img src="/../main/pictures/PCB_3D-view.png" width="50%" alt= "Schematic" height="50%">

<a name="seeedxiaoesp32c3"></a>
# SEEED XIAO ESP32C3

## Hardware wiring schema
<img src="/../main/pictures/seeed-xiao-esp32c3-w5500-bmp280.png" width="40%" alt= "Wiring" height="40%">

Two more pins left for other purposes!

## ESPHome config
I didn't manage to get the board working with the arduino framework.
Esp-idf framework works fine as follows:
``` yaml
esphome:
  name: seeed-xiao-c3-w5500
  friendly_name: seeed-xiao-c3-w5500
  platformio_options:
    board_build.flash_mode: dio
    board_build.mcu: esp32c3
    board_build.f_cpu: 160000000L

esp32:
  board: seeed_xiao_esp32c3
  variant: esp32c3
  framework:
    type: esp-idf

# Enable logging
logger:

# Enable Home Assistant API
api:

ota:

ethernet:
  type: W5500
  mosi_pin: 10
  miso_pin: 9
  clk_pin: 8
  cs_pin: 4
  reset_pin: 2
  interrupt_pin: 3
  clock_speed: 25MHz

i2c:
  scl: 7
  sda: 6
  scan: True
  id: bus_a

sensor:
  - platform: bmp280
    temperature:
      name: "Temperatuur"
      unit_of_measurement: °C
      accuracy_decimals: 1
    pressure:
      name: "Luchtdruk"
      unit_of_measurement: hPa
      accuracy_decimals: 0
    i2c_id: bus_a
    address: 0x76

binary_sensor:
  - platform: status
    name: "Status"

text_sensor:
  - platform: ethernet_info
    ip_address:
      name: IP Address

switch:
  - platform: restart
    name: "Restart"
```

<a name="lilygotethlite"></a>
# LILYGO T-ETH-Lite ESP32S3
## Hardware wiring schema
<img src="/../main/pictures/LILYGO-T-ETH-LITE-ESP32-S3.jpg" width="40%" alt= "Wiring" height="40%">

Flashing: see https://github.com/Xinyuan-LilyGO/LilyGO-T-ETH-Series?tab=readme-ov-file

I managed to get the board operational, but I had to upload the code from within ESPHome in Home Assistant; uploading a bin file did not work.

## ESPHome config
``` yaml
esphome:
  name: lilygo-t-eth-lite-w5500
  friendly_name: lilygo-t-eth-lite-w5500
  platformio_options:
    board_build.flash_mode: dio

esp32:
  board: esp32-s3-devkitc-1
  framework:
    type: esp-idf

# Enable logging
logger:

# Enable Home Assistant API
api:

ota:

ethernet:
  type: W5500
  mosi_pin: 12
  miso_pin: 11
  clk_pin: 10
  cs_pin: 9
  reset_pin: 14
  interrupt_pin: 13
  clock_speed: 25MHz

binary_sensor:
  - platform: status
    name: "Status"

text_sensor:
  - platform: ethernet_info
    ip_address:
      name: IP Address

switch:
  - platform: restart
    name: "Restart"
```
