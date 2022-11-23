# esp-pc-control
Turning on/off PC with esp-01 board, ESPHome and Home Assistant

## Scheme
 
![SCHEME](https://github.com/eulampy/esp-pc-control/blob/main/Schematic_esp%20PC%20control.png)

## ESPHome config file

```yaml
esphome:
  name: $devicename
  platform: ESP8266
  board: esp01_1m

substitutions:
  devicename: esp_phenom_control

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  manual_ip:
    static_ip: 10.10.10.70
    gateway: 10.10.10.10
    subnet: 255.255.255.0

# Enable logging
logger:

# Enable Home Assistant API
api:
  password: !secret ha_api_password

ota:

web_server:
  port: 80

# GPIO Binary Sensor
# turn in internal pullup and set name
binary_sensor:
  - platform: gpio
    pin:
      number: GPIO03
      mode: INPUT_PULLUP
      inverted: True
    name: ${devicename} input
    id: ${devicename}_binary_sensor

# output GPIO0
switch:
  - platform: gpio
    name: ${devicename} output
    pin: GPIO0
    inverted: True
    restore_mode: ALWAYS_OFF
    id: ${devicename}_switch

# virtual switch turn on
  - platform: template  
    name: ${devicename} turn on
    turn_on_action:
      then:
        - if:
            condition:
              binary_sensor.is_off: ${devicename}_binary_sensor
            then:
              - switch.turn_on: ${devicename}_switch
              - delay: 1s
              - switch.turn_off: ${devicename}_switch
              - delay: 1s

# virtual switch turn off
  - platform: template  
    name: ${devicename} turn off
    # lambda: |-
    #   return false;
    turn_on_action:
      then:
        - if:
            condition:
              binary_sensor.is_on: ${devicename}_binary_sensor
            then:
              - switch.turn_on: ${devicename}_switch
              - delay: 1s
              - switch.turn_off: ${devicename}_switch
              - delay: 5s   # delay, if PC in save mode...
              #.. and "press" button again
              - switch.turn_on: ${devicename}_switch
              - delay: 1s
              - switch.turn_off: ${devicename}_switch
              - delay: 1s
```
