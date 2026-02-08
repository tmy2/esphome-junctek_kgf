# esphome-junctek_kgf
Component for esphome to read status from a Junctek KG-F coulometer/battery monitor via UART. It also reads settings so it is able correctly calculate battery percentage left. But settings are not exposed to HomeAssistant.

## Features
Connects to the Junctek KGF series battery monitor via UART (RS-485 *adapter NOT needed*) and retrieves the following values:

* Battery Voltage
* Battery Percent
* Battery Current
* Power
* Temperature
* Battery Ah Charged Total
* Battery Ah Remaining
* Battery Ah Used Total
* Battery Charging Power
* Battery Discharging Power
* Battery life remaining
* Battery Resistance
* Output Status

## Requirements
* ESPHome

## Known problems
* If could call it a problem - when data not available from Junctek interface - you keep seeing the latest value in sensors. Which may misslead. Sensors will not be updated nor they be set to Unavailable.

## Usage
### Connect hardware.
The ESP32 TX and RX needs to be connected via Junctek LINK port using a 4cp4 connector. Monitor SHOULD BE connected too and active. If you dont have monitor you should edit component loop method and uncomment old code, and comment out new one. Monitor makes calls to Junctek asking for data, ESP32 just reads the response and parses it. So you keep the monitor + will have data comming to your IoT system.

## ESPHOME Config
The applicable config for the device should look something like:

```yaml
substitutions:
  name: "shunt"
  friendly_name: Shunt

esphome:
  name: ${name}
  friendly_name: ${friendly_name}

external_components:
  - source:
      type: git
      url: https://github.com/Tommixoft/esphome-junctek_kgf
      ref: main
    components: [ junctek_kgf ]

esp32:
  board: esp32dev
  framework:
    type: arduino

uart:
  tx_pin: TX #43
  rx_pin: RX #44
  baud_rate: 115200


sensor:
  - platform: junctek_kgf
    address: 1
    invert_current: true
    update_stats_interval: 3000 #3 seconds
    voltage:
      name: "Battery Voltage"
    current:
      name: "Battery Current"
    power:
      name: "Battery Power"
    battery_level:
      name: "Battery Level"
    amp_hour_remain:
      name: "Battery Ah Remaining"
    amp_hour_used_total:
      name: "Battery Ah Used Total"
    amp_hour_charged_total:
      name: "Battery Ah Charged Total"            
    temperature:
      name: "Temperature"

    # usless sensor, seems not to work and always returns 1 mili ohm.
    battery_ohm:
      name: "Battery Resistance"
    
    # 0 = ON, 1 = OVP, 2 = OCP, 3 = UVP/LVP, 5 = OPP, 6 = OTP, 7 = UTP??, 99 = OFF (relay is off, if you connected it to Junctek device)
    # i think if there is multiple problems code can be combined. Lets say 7 = OVP + OTP.. but im not sure is this really the case, manual sucks.
    output_status:
      name: "Output Status"

    # Shows how much time left at current discharge rate to go to 0%, or how much time left to go to 100% in case of charge. 
    battery_life:
      name: "Battery life remaining"

    # It just multiply of amps and voltage in case of charging. With this you HA you can add custom sensor to calculate Kwh and integrate into ENERGY dashboard.   
    battery_charged_energy:
      name: "Battery Charging Power"

    #  It just multiply of amps and voltage in case of discharging. With this you HA you can add custom sensor to calculate Kwh and integrate into ENERGY dashboard.   
    battery_discharged_energy:
      name: "Battery Discharging Power"   


logger:
  level: ERROR
  baud_rate: 0 #in this example we use UART that is also used by the logger. We dont need serial logger so we just disable it here
```

Not all sensors need to be added.
Address is assumed to be 1 if not provided. (this is configured on the monitor)
invert_current: This inverts the reported current, it's recommended to include this option with either true or false (which ever makes the current make more sense for your setup). The default is currently false (and false will match previous behaviour), but may change to true in future updates.

