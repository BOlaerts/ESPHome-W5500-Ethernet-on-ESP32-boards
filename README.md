# ESP32C3 Super Mini
Playing around with the amazing ESP32C3 Super Mini board.

Home Assistant integration using ESPHome on a ESP32C3-SuperMini board with W5500 ethernet and I²C sensor:
- connected to the W5500 ethernet module, using the fantastic work of Jeroen Van Oort described in https://github.com/esphome/esphome/pull/4424
- connected to a I²C BMP280 temperature/pressure sensor

# Hardware wiring schema
<img src="/../main/pictures/esp32c3-super-mini-w5500-bmp280.png" width="40%" alt= "Schematic" height="40%">

And even with this small sized board, there is some room for extra connections!

# ESPHome config
I didn't get it to work using the esp-idf framework, as presented in Jeroens config, but arduino does the trick for me.
So far, connection seems to be ligthning fast and super stable.
In my experience, the easiest way of flashing the new ESP32C3 boards is by using the [Adafruit ESPTool](https://adafruit.github.io/Adafruit_WebSerial_ESPTool/).
``` yaml
esphome:
  name: esp32c3-super-mini
  friendly_name: esp32c3-super-mini

esp32:
  board: esp32-c3-devkitm-1
  framework:
    type: arduino

web_server:
  port: 80

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
# PCB design
Working on a pcb design for a small board with ethernet connection and exposure of all remaining GPIOs.

See PCB folder for first design attempt.
