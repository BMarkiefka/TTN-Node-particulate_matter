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

This sketch will send Battery Voltage (in mV), Temperature (in Celsius), Humidity (in %) and PM10/PM2.5 counts using the lora-serialization library matching setttings have to be added to the payload decoder funtion in the The Things Network console/backend.

The Application will 'sleep' 75x8 seconds (10 minutes) and then run the SDS011 sensor for 30 seconds to get a good reading on the pm2.5 and pm10 count. You can adjust those sleep and uptimes with the variables

```c++
int sleepcycles = 75;
#define sdsSamples 30
```

This uses OTAA (Over-the-air activation), where where a DevEUI and application key is configured, which are used in an over-the-air activation procedure where a DevAddr and session keys are assigned/generated for use with all further communication.

To use this sketch, first register your application and device with The Things Network, to set or generate an AppEUI, DevEUI and AppKey. Multiple devices can use the same AppEUI, but each device has its own DevEUI and AppKey. Do not forget to adjust the payload decoder function.

In the payload function change the decode function, by adding the code from https://github.com/thesolarnomad/lora-serialization/blob/master/src/decoder.js to the function right below the "function Decoder(bytes, port) {" and delete everything below exept the last "}". Right before the last line add this code

```javascript
return decode(bytes, [uint16, uint16, uint16, temperature, humidity], ['battery', 'pm25', 'pm10', 'temp', 'humi']);
```

and you get a json containing the stats for battery, pm25, pm10, temp and humi.

Hardware:
---------

To build the node please follow the exellent labs story from Frank on TTN (https://www.thethingsnetwork.org/labs/story/workshop-creating-a-ttn-node) but skip all the sensors. Instead connect a AM2302 to Pin 7 and the SDS011 to Pins 12 & 13 and the PDWN from the Pololu StepUp U1V11F5 to Pin 6. Now make sure you have all the grounds together and the power (3.3V) everywhere it is needed. Don't forget the power (5V) from the StepUp to the SDS011. Have a look at the following picture to see all the components connected, exept the solar panel for which you need a adapter cable to connect it to the adafruit solar controller.

![Hardware assembled](https://github.com/Freifunk-Hennef/TTN-Node-particulate_matter/tree/master/images/ttn_node_pm001.jpg "Hardware assembled")

Together with some 6mm hose and 2 75mm 87° tubes you can build a very compact and water proof case.

![Hardware assembled](https://github.com/Freifunk-Hennef/TTN-Node-particulate_matter/tree/master/images/ttn_node_pm002.jpg "Hardware assembled")

Links:
------
- original code from the workshop: https://github.com/galagaking/ttn_nodeworkshop
- lora-serialization library: https://github.com/thesolarnomad/lora-serialization
- http://luftdaten.info/