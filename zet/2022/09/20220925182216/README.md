# Flash Shelly Plug S with ESPHome

## Flashing process
1. Connect iPhone to the WIFI network created by shelly plug
2. Add Shelly device to the iOs app
3. Flash Tasmota OTA by calling http://IP/ota?url=http://dl.dasker.eu/firmware/mg2tasmota-ShellyPlugS.zip
4. Remember: The device creates a new wifi network now. Don't wait for it to be on the existing wifi.
5. Connect to the newly created wifi network and put in wifi credentials when asked for
6. Head to the devices IP address with browser
7. Configure device "Configure -> Configure Other" and paste in the following template: `{"NAME":"Shelly Plug S","GPIO":[56,255,158,255,255,134,0,0,131,17,132,21,255],"FLAG":2,"BASE":45i}`
8. Using another device (one you trust to give right measurements) in series to measure a high load (e.g. a kettle) and write down both values
9. Adapt the measured values in the script and let ESPHome create an image
10. Donwload latest Tasmota minimal image from https://github.com/arendst/Tasmota/releases/download/v12.1.1/tasmota-minimal.bin
11. Flash minimal image to have enough space for ESPhome images
12. Now flash the image generated by ESPHome

## Configuration
```YAML
substitutions:
  devicename: kühlschrank
  channel_1: Relay
  
  # Higher value gives lower watt readout
  current_res: "0.000943"
  
  # Lower value gives lower voltage readout
  voltage_div: "2066"
  
  # measure a relatively strong load and enter values measured by the device vs the values your reference measurement provided here
  power_cal_meas: "2220.0"
  power_cal_real: "2102.0"

  max_power: "2500"
  max_temp: "70.0"

esphome:
  name: ${devicename}

esp8266:
  board: esp8285

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: ""

ota:
  password: ""

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: ${devicename}
    password: ""

captive_portal:

web_server:
  port: 80

time:
  - platform: sntp
    id: my_time

binary_sensor:
  - platform: gpio
    pin:
      number: GPIO13
      inverted: True
    name: "${devicename}_button"
    on_press:
      - switch.toggle: relay

status_led:
  pin:
    number: GPIO02
    inverted: True

output:
  - platform: gpio
    pin: GPIO00
    inverted: true
    id: led

switch:
  - platform: gpio
    pin: GPIO15
    id: relay
    name: "${channel_1}"
    on_turn_on:
      - output.turn_on: led
    on_turn_off:
      - output.turn_off: led

sensor:
  - platform: wifi_signal
    name: "${devicename} WiFi Signal"
    update_interval: 300s

  # NTC Temperature
  - platform: ntc
    sensor: temp_resistance_reading
    name: ${devicename} temperature
    unit_of_measurement: "°C"
    accuracy_decimals: 1
    icon: "mdi:thermometer"
    calibration:
      b_constant: 3350
      reference_resistance: 10kOhm
      reference_temperature: 298.15K
    on_value_range:
      - above: ${max_temp}
        then:
          - switch.turn_off: relay
          - homeassistant.service:
              service: persistent_notification.create
              data:
                title: Message from ${devicename}
              data_template:
                message: Switch turned off because temperature exceeded ${max_temp}°C
  - platform: resistance
    id: temp_resistance_reading
    sensor: temp_analog_reading
    configuration: DOWNSTREAM
    resistor: 32kOhm
  - platform: adc
    id: temp_analog_reading
    pin: A0

  - platform: hlw8012
    model: BL0937
    sel_pin:
      number: GPIO12
      inverted: true
    cf_pin: GPIO05
    cf1_pin: GPIO14
    current_resistor: ${current_res}
    voltage_divider: ${voltage_div}
    current:
      name: "${channel_1} current"
      unit_of_measurement: "A"
      accuracy_decimals: 3
      icon: mdi:flash-outline
    voltage:
      name: "${channel_1} voltage"
      unit_of_measurement: "V"
      icon: mdi:flash-outline
    power:
      name: "${channel_1} power"
      id: power
      unit_of_measurement: "W"
      filters:
        - calibrate_linear:
          - 0.0 -> 0.0
          - ${power_cal_meas} -> ${power_cal_real}
      icon: mdi:flash-outline
      on_value_range:
        - above: ${max_power}
          then:
            - switch.turn_off: relay
            - homeassistant.service:
                service: persistent_notification.create
                data:
                  title: Message from ${devicename}
                data_template:
                  message: Switch turned off because power exceeded ${max_power}W
    update_interval: 10s

  - platform: total_daily_energy
    name: "${channel_1} daily energy"
    power_id: power
    filters:
      # Multiplication factor from W to kW is 0.001
      - multiply: 0.001
    unit_of_measurement: kWh
```