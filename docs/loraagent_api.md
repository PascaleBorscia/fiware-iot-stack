
Lora Agent allows you to:

- Send data from the LoRa device to the FIWARE IoT Stack
- Change tracking mode from your application to the LoRa device

Your LoRa Tracker is already registered on the Orion Context Broker and can receive command on the `SET_tracking` attribute. This `SET_tracking` attribute allows to modify the tracking mode:

- `SLEEP`: no tracking
- `SLOWTRACK`: the tracker send 1 position request every one hour
- `FASTTRACK`: the tracker send 1 position request every 10 min
- `MOTIONTRACK`: motion tracking

# Simulating a LoRa device

The tracker LoRa sends the data to the LoRa Agent that forwards it to Orion.
To simulate LoRa Agent, you can send the message directly to Orion with the following HTTP POST request:

```
POST   /cb/v1/contextEntities/{{tracker_id}}
HOST: {{host}}
Content-Type: application/json
X-Auth-Token: {{TOKEN}}
Fiware-Service: {{Fiware-Service}}
Fiware-ServicePath: {{Fiware-ServicePath]}}

{
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
                "metadatas":[
                    {
                    "name":"location",
                    "type":"string",
                    "value":"WGS84"

                    }
                ]
            }
            ]
        }
    ],
    "updateAction": "APPEND"
}
```

# Get Data

Finally, after connecting your IoT devices this way you (or any other developer with the right access permissions) should be able to use the Data API to read the NGSI entity assigned to your device.

```
GET   http://{{HOST}}/cb/v1/contextEntities
Headers: {'content-type': 'application/json’; 'X-Auth-Token' : [TOKEN]; "Fiware-Service: [Fiware-Service]”; "Fiware-ServicePath: [Fiware-ServicePath]"}
Payload:

```

Results

```
{
  "contextResponses": [
    {
      "contextElement": {
        "type": "edison",
        "isPattern": "false",
        "id": "myEdison",
        "attributes": [
          {
            "name": "TimeInstant",
            "type": "ISO8601",
            "value": "2015-12-01T16:45:37.997157"
          }
        ]
      },
      "statusCode": {
        "code": "200",
        "reasonPhrase": "OK"
      }
    },
    {
      "contextElement": {
        "type": "Room",
        "isPattern": "false",
        "id": "Room1",
        "attributes": [
          {
            "name": "pressure",
            "type": "integer",
            "value": "763"
          },
          {
            "name": "temperature",
            "type": "float",
            "value": "26.5"
          }
        ]
      },
      "statusCode": {
        "code": "200",
        "reasonPhrase": "OK"
      }
    },
    {
      "contextElement": {
        "type": "tracker",
        "isPattern": "false",
        "id": "20635F0012000011",
        "attributes": [
          {
            "name": "levelBattery",
            "type": "long",
            "value": "0.75"
          },
          {
            "name": "mode",
            "type": "string",
            "value": "POSITION"
          },
          {
            "name": "position",
            "type": "coords",
            "value": "45.5, -10.5",
            "metadatas": [
              {
                "name": "location",
                "type": "string",
                "value": "WGS84"
              }
            ]
          },
          {
            "name": "temperature",
            "type": "double",
            "value": "25.5"
          },
          {
            "name": "tracking",
            "type": "string",
            "value": "SLOWTRACK"
          }
        ]
      },
      "statusCode": {
        "code": "200",
        "reasonPhrase": "OK"
      }
    }
  ]
}
```

# Change tracking mode

You can change tracking mode (POST request) in the following way:

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
