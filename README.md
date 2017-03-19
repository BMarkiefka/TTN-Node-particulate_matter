Code for a particulate matter sensor based on hardware which is a crossover from the loratracker node workshop and luftdaten.info
=================================================================================================================================

Code adapted from the Node Building Workshop using a modified LoraTracker board
-------------------------------------------------------------------------------

Copyright (c) 2015 Thomas Telkamp and Matthijs Kooijman

Copyright (c) 2017 Caspar Armster for the modifications concerning 
- the sensor support for the AM2302 and SDS011
- the battery voltage measurement
- integration of the lora-serialization library
- the Pololu 5V StepUp U1V11F5 support
- integration of the adafruit solar controller and the solar panel
 
Permission is hereby granted, free of charge, to anyone obtaining a copy of this document and accompanying files, to do whatever they want with them without any restriction, including, but not limited to, copying, modification and redistribution.

NO WARRANTY OF ANY KIND IS PROVIDED.
------------------------------------

This sketch will send Battery Voltage (in mV), Temperature (in Celsius), Humidity (in %) and PM10/PM2.5 counts using the lora-serialization library matching settings have to be added to the payload decoder function in the The Things Network console/backend.

The Application will 'sleep' 75x8 seconds (10 minutes) and then run the SDS011 sensor for 30 seconds to get a good reading on the pm2.5 and pm10 count. You can adjust those sleep and uptimes with the variables

```c++
int sleepcycles = 75;
#define sdsSamples 30
```

This uses OTAA (Over-the-air activation), where where a DevEUI and application key is configured, which are used in an over-the-air activation procedure where a DevAddr and session keys are assigned/generated for use with all further communication.

To use this sketch, first register your application and device with The Things Network, to set or generate an AppEUI, DevEUI and AppKey. Multiple devices can use the same AppEUI, but each device has its own DevEUI and AppKey. Do not forget to adjust the payload decoder function.

In the payload function change the decode function, by adding the code from https://github.com/thesolarnomad/lora-serialization/blob/master/src/decoder.js to the function right below the "function Decoder(bytes, port) {" and delete everything below except the last "}". Right before the last line add this code

```javascript
var values = decode(bytes, [uint16, uint16, uint16, temperature, humidity], ['battery', 'pm25', 'pm10', 'temp', 'humi']);
values["pm25"] = values["pm25"]/10;
values["pm10"] = values["pm10"]/10;
return values;
```

and you get a json containing the stats for battery, pm25, pm10, temp and humi.

Node-RED
--------

To connect your sensor to the luftdaten.info map you can run your data through a Node-RED installation. Node-RED needs the "The Things Network Node-RED Nodes"-Package from this site: https://www.npmjs.com/package/node-red-contrib-ttn

![Hardware assembled](https://raw.githubusercontent.com/Freie-Netzwerker/TTN-Node-particulate_matter/master/images/Node-RED_screenshot001.jpg "Hardware assembled")

Now build a flow with the "ttn message" node first (tell the node your application id, device id), connect this to a function node and use the following code, replace "TTN-Hennef-" and "TTN-Hennef-v1" with the credentials from your community:

```javascript
msg.headers = {
    "X-Pin": "1",
    "X-Sensor": "TTNHennef-" + parseInt(msg["hardware_serial"], 16)
};
msg.payload = {
    "software_version": "TTNHennef-v1",
    "sensordatavalues": [
        {"value_type": "P1", "value": parseFloat(msg.payload_fields["pm10"])},
        {"value_type": "P2", "value": parseFloat(msg.payload_fields["pm25"])}
    ]
};
return msg;
```

![Hardware assembled](https://raw.githubusercontent.com/Freie-Netzwerker/TTN-Node-particulate_matter/master/images/Node-RED_screenshot002.jpg "Hardware assembled")

Now connect a "http request"-node to your flow and set the Method to "POST" and the URL to "http://api.luftdaten.info/v1/push-sensor-data/" and you are set.

![Hardware assembled](https://raw.githubusercontent.com/Freie-Netzwerker/TTN-Node-particulate_matter/master/images/Node-RED_screenshot003.jpg "Hardware assembled")

If your sensor does take longer breaks than 5 minutes between the data transmissions, you should proxy the data and send it in an intervall shorter than 5 minutes again, else your sensor will not be visible on the map all the time. In my example the sensor is sending every 12 minutes (brutto = pause + sensor reading + transmitting), so i send the data again after 4 minutes and after 8 minutes, to have no gap longer than 5 minutes. You can best use the "delay"-node and split the flow:

![Hardware assembled](https://raw.githubusercontent.com/Freie-Netzwerker/TTN-Node-particulate_matter/master/images/Node-RED_screenshot004.jpg "Hardware assembled")

Hardware:
---------

To build the node please follow the excellent labs story from Frank on TTN (https://www.thethingsnetwork.org/labs/story/workshop-creating-a-ttn-node) but skip all the sensors. Instead connect a AM2302 to Pin 7 and the SDS011 to Pins 12 & 13 and the PDWN from the Pololu StepUp U1V11F5 to Pin 6. Now make sure you have all the grounds together and the power (3.3V) everywhere it is needed. Don't forget the power (5V) from the StepUp to the SDS011. Have a look at the following picture to see all the components connected, except the solar panel for which you need an adapter cable to connect it to the adafruit solar controller.

![Hardware assembled](https://raw.githubusercontent.com/Freie-Netzwerker/TTN-Node-particulate_matter/master/images/ttn_node_pm001.jpg "Hardware assembled")

Together with some 6mm hose and 2 75mm 87° tubes you can build a very compact and water proof case.

![Hardware assembled](https://raw.githubusercontent.com/Freie-Netzwerker/TTN-Node-particulate_matter/master/images/ttn_node_pm002.jpg "Hardware assembled")

Part list
---------

- Arduino Pro Mini 3.3V / 8Mhz (5V will not work)
- RFM Module RFM95W (868 Mhz)
- FT232RL-3.3v-5v-TTL-USB-Serial-Port-Adapter (has to have a 3.3V option!)
- USB cable for FTDI
- 1x Led 3mm red
- 1x resistor 1k (brown-black-red)
- 1x resistor 4k7 (yellow-violet-red)
- 1x resistor 10k (brown-black-orange)
- 1x resistor 100k (brown-black-yellow)
- Header 6p female (Arduino programming)
- Header 4p female (sensor)
- Header 4p male 90 degrees (gps / optional) (cut 2p from the header included with the Pro Mini)
- PCB Loratracker RFM98 including special headers (2x 8p 2mm, 2x 12p 2.54mm, 1x 2p 2.54mm) for RFM and Arduino
- SDS011 pm10 & pm2.5 sensor
- AM2302 sensor
- 1x resistor 4k7 (yellow-violet-red)
- Popolu 5V StepUp U1V11F5
- Adafruit USB / DC / Solar Lithium Ion/Polymer charger - v2
- Lithium Ion Battery Pack - 3.7V 6600mAh
- Medium 6V 2W Solar panel - 2.0 Watt
- 3.5 / 1.3mm or 3.8 / 1.1mm to 5.5 / 2.1mm DC Jack Adapter Cable
- bunch of cables
- cable ties
- 6mm hose
- 2x Marley Silent HT Bogen (DN 75 87°)

Links:
------
- original code from the workshop: https://github.com/galagaking/ttn_nodeworkshop
- lora-serialization library: https://github.com/thesolarnomad/lora-serialization
- http://luftdaten.info/