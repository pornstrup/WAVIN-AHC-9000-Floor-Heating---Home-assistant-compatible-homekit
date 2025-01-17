# Credits
This fantastic project is not developed by me. The project is forked from this : https://github.com/dkjonas/Wavin-AHC-9000-mqtt

All the credits go to the originally owner "dkjonas". 

I made some modifications to this project, so the system integrates 3 led´s which can be used to indicate power, mqtt and wifi when all the electronics are mounted in a box. 

This project will recieve further updates. 

# Wavin-AHC-9000-mqtt
This is a simple Esp8266 mqtt interface for Wavin AHC-9000/Jablotron AC-116, with the goal of being able to control this heating controller from a home automation system.

## Hardware
The AHC-9000 uses modbus to communicate over a half duplex RS422 connection. It has two RJ45 connectors for this purpose, which can both be used. 
The following schematic shows how to connect an Esp8266 to the AHC-9000:
![Schematic](/electronics/schematic.png)

## Components with links to devices on Aliexpress

* Buck Converter : https://www.aliexpress.com/item/LM2596-LM2596S-LED-Voltmeter-ADJ-DC-DC-Step-down-Step-Down-Adjustable-Power-Supply-Module-With/32827909407.html?spm=a2g0s.9042311.0.0.7edb4c4duCPOwN

* Tripler Base: https://www.aliexpress.com/item/TZT-Tripler-Base-V1-0-0-Module-Board-with-Pins-D1-Mini-Active-Components-Integrated-Circuits/32848786211.html?spm=a2g0s.9042311.0.0.27424c4dOt1jSr

* Protoboard-shield : https://www.aliexpress.com/item/ProtoBoard-Shield-for-WeMos-D1-mini-double-sided-perf-board-Compatible/32823336161.html?spm=a2g0s.9042311.0.0.27424c4dOt1jSr

* LED : https://www.aliexpress.com/item/Free-Ship-100PCS-2-3-4-2-5-7-Square-Transparent-LED-Red-Yellow-Green-Blue/32869433921.html?spm=a2g0s.9042311.0.0.27424c4dls9aGy

* Thread M3 : https://www.aliexpress.com/item/100pcs-m3-3-4-5-6-8-OD-4-2mm-M3-Injection-Molding-Brass-Knurled-Thread/32813496424.html?spm=a2g0s.9042311.0.0.27424c4dhfqKHo

* M3 bolts.

* Modbus IC MAX3072E : https://www.aliexpress.com/item/MAX3072E-MAX3072EEPA-DIP/32859557410.html?spm=a2g0s.9042311.0.0.27424c4d1kt8zR

* DIP Socket : https://www.aliexpress.com/item/High-quality-20pcs-lot-8-Pins-DIP-DIP8-IC-Sockets-Adaptor-Solder-Type-8-PIN-IC/32822549802.html?spm=a2g0s.9042311.0.0.27424c4dFoaid5

* WEMOS D1 mini or PRO or any ESP8266 device for that matter 

* 330R Resisitors

* RJ-45 Breakoutboard : https://www.aliexpress.com/item/Durable-Tap-Electronics-RJ45-Breakout-ModuleBoard-For-Arduino-New/32923930313.html?spm=a2g0s.9042311.0.0.27424c4dYoLJ5s

* RJ-45 Socket : https://www.aliexpress.com/item/10Pcs-set-RJ45-Network-Ethernet-8P-8C-Female-Socket-Connectors-8Pin-PCB-Mount-RJ45-8P8C-Single/32850223640.html?spm=a2g0s.9042311.0.0.27424c4dhfv0iu




## Software

### Configuration
src/PrivateConfig.h contains 5 constants, that should be changed to fit your own setup.

`WIFI_SSID`, `WIFI_PASS`, `MQTT_SERVER`, `MQTT_USER`, and `MQTT_PASS`.

NOTE: If you use MQTT password AND username, remember to change lines in the code according to the description in the code!!

### Compiling
I use [PlatformIO](https://platformio.org/) for compiling, uploading, and and maintaining dependencies for my code. If you install PlatformIO in a supported editor, building this project is quite simple. Just open the directory containing `platformio.ini` from this project, and click build/upload. If you use a different board than nodemcu, remember to change the `board` variable in `platformio.ini`.
You may be able to use the Arduino tools with the esp8266 additions for compiling, but a few changes may be needed, including downloading dependencies manually.

### Testing
Assuming you have a working mqtt server setup, you should now be able to control your AHC-9000 using mqtt. If you have the [Mosquitto](https://mosquitto.org/) mqtt tools installed on your mqtt server, you can execude:
```
mosquitto_sub -u username -P password -t heat/# -v
```
to see all live updated parameters from the controller.

To change the target temperature for a output, use:
```
mosquitto_pub -u username -P password -t heat/floorXXXXXXXXXXXX/1/target_set -m 20.5
```
where the number 1 in the above command is the output you want to control and 20.5 is the target temperature in degree celcius. XXXXXXXXXXXX is the MAC address of the Esp8266, so it will be unique for your setup.

### Integration with HomeAssistant
If you have a working mqtt setup in [HomeAssistant](https://home-assistant.io/), all you need to do in order to control your heating from HomeAssistant is to enable auto discovery for mqtt in your `configuration.yaml`.
```
mqtt:
  discovery: true
  discovery_prefix: homeassistant
```
You will then get a climate and a battery sensor device for each configured output on the controller.

If you don't like auto discovery, you can add the entries manually. Create an entry for each output you want to control. Replace the number 0 in the topics with the id of the output and XXXXXXXXXXXX with the MAC of the Esp8266 (can be determined with the mosquitto_sub command shown above)
```
climate wavinAhc9000:
  - platform: mqtt
    name: floor_kitchen
    current_temperature_topic: "heat/floorXXXXXXXXXXXX/0/current"
    temperature_command_topic: "heat/floorXXXXXXXXXXXX/0/target_set"
    temperature_state_topic: "heat/floorXXXXXXXXXXXX/0/target"
    mode_command_topic: "heat/floorXXXXXXXXXXXX/0/mode_set"
    mode_state_topic: "heat/floorXXXXXXXXXXXX/0/mode"
    modes:
      - "heat"
      - "off"
    availability_topic: "heat/floorXXXXXXXXXXXX/online"
    payload_available: "True"
    payload_not_available: "False"
    qos: 0

sensor wavinBattery:
  - platform: mqtt
    state_topic: "heat/floorXXXXXXXXXXXX/0/battery"
    availability_topic: "heat/floorXXXXXXXXXXXX/online"
    payload_available: "True"
    payload_not_available: "False"
    name: floor_kitchen_battery
    unit_of_measurement: "%"
    device_class: battery
    qos: 0
```
