
Lora Agent API allows you to:

- Send data from the LoRa device to the FIWARE IoT Stack
- Send commands from your application to the LoRa device

Your LoRa Tracker is already registered to receive command on the SET_tracking.
SET_tracking allow to modify the tracking mode:

- SLEEP : no tracking
- SLOWTRACK : the tracker send 1 position request every one hour
- FASTTRACK : the tracker send 1 position request every 10 min
- MOTIONTRACK: motion tracking

# Send observations 

The tracker LoRa sends the data to the LoRa Agent that forwards to Orion.
To simulate LoRa Agent, you can send the message directly to Orion with the following HTTP POST request:

```
POST   http://{{HOST}}/cb/v1/updateContext
Headers: {'content-type': 'application/json’; 'X-Auth-Token' : [TOKEN]; "Fiware-Service: [Fiware-Service]”; "Fiware-ServicePath: [Fiware-ServicePath]"}
Payload: {
             "contextElements": [
                 {
                     "type": "tracker",
                     "isPattern": "false",
                     "id": "20635F0012000011",
                     "attributes": [
                     {
                         "name": "temperature",
                         "type": "double",
                         "value": "25.5"
                     },
                     {
                         "name": "mode",
                         "type": "string",
                         "value": "POSITION"
                     },
                     {
                         "name": "tracking",
                         "type": "string",
                         "value": "SLOWTRACK"
                     },
                     {
                         "name": "levelBattery",
                         "type": "long",
                         "value": "0.75"
                     },
                     {
                         "name": "position",
                         "type": "coords",
                         "value": "45.5, -10.5",
                         "metadata":{
                             "name":"location",
                             "type":"string",
                             "value":"WGS84"
                         }
                     },
                     ]
                 }
             ],
             "updateAction": "APPEND"
         }
```


Finally, after connecting your IoT devices this way you (or any other developer with the right access permissions) should be able to use the Data API to read the NGSI entity assigned to your device.

```
GET   http://{{HOST}}/cb/v1/contextEntities
Headers: {'content-type': 'application/json’; 'X-Auth-Token' : [TOKEN]; "Fiware-Service: [Fiware-Service]”; "Fiware-ServicePath: [Fiware-ServicePath]"}
Payload:

```


# Act upon devices 

In order to send commands to devices, you just need to know which attributes correspond to commands and update them.

You can declare the command related attributes at the registry process (POST request) in the following way:

```
POST   http://{{HOST}}/cb/v1/updateContext
Headers: {'content-type': 'application/json’; 'X-Auth-Token' : [TOKEN]; "Fiware-Service: [Fiware-Service]”; "Fiware-ServicePath: [Fiware-ServicePath]"}
Payload:
        {
            "contextElements": [
                {
                    "type": "tracker",
                    "isPattern": "false",
                    "id": "20635F0012000011",
                    "attributes": [
                        {
                        "name": "SET_tracking",
                        "type": "string",
                        "value": "FASTTRACK"
                        }
                    ]
                }
            ],
            "updateAction": "UPDATE"
        }

```

