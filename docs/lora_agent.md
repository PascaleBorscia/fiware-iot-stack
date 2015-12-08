
Lora Agent allows to stock Tracker LoRa informations to the cloud.

The provided device is a geolocation tracker with a temperature captor and accelerometer.
Depending on the type of request, Lora agent stocks the informations:

 - temperature (celsius)
 - geolocation (latitude, longitude)
 - level of battery (percent)
 - tracking mode
 - mode (type of request)

 There are 4 types of mode:

 - INIT : when the tracker starts
 - LORALIVE : heartbeat all the 10 minutes
 - POSITION : send location depending on tracking mode
 - OFF : when the tracker stops

There are 4 tracking modes:

- SLEEP : no tracking
- SLOWTRACK : the tracker send 1 position request every one hour
- FASTTRACK : the tracker send 1 position request every 10 min
- MOTIONTRACK: motion tracking
