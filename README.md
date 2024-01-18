# ESP32C3 Super Mini W5500 Ethernet
Playing around with the amazing ESP32C3 Super Mini board.

Home Assistant integration using ESPHome on a ESP32C3-SuperMini board with W5500 ethernet and I²C sensor:
- connected to the W5500 ethernet module, using the fantastic work of Jeroen Van Oort described in https://github.com/esphome/esphome/pull/4424
- connected to a I²C BMP280 temperature/pressure sensor

# Hardware wiring schema
<img src="/../main/pictures/esp32c3-super-mini-w5500-bmp280.png" width="40%" alt= "Wiring" height="40%">

And even with this small sized board, there is some room for extra connections!

Possible improvements to investigate:
- CLK wire for the W5500 SPI bus is set to GPIO08
- GPIO08 is also the linked to the status led of the ESP32 C3 super mini development board
- possible or better to use GPIO02 instead?
- update 18.01.2024: switching CLK to GPIO02 did not seem to work...

# ESPHome config
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

external_components:
- source:
    type: git
    url: https://github.com/JeroenVanOort/esphome/
    ref: eth-w5500
  components:
  - ethernet

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


# PCB design
Work in progress on a pcb design for a small board with ethernet connection and exposure of all remaining GPIOs.

See PCB folder for first design attempt.

<img src="/../main/pictures/Schematic_ESP32-C3-SuperMini W5500 Ehternet I2C_2024-01-16.png" width="50%" alt= "Schematic" height="50%">

<img src="/../main/pictures/PCB_3D-view.png" width="50%" alt= "Schematic" height="50%">
