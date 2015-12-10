Cepheus provides a Complex Event Processor (CEP) with a NGSI 9/10 interface.

The goal of a Cepheus is to process data in real time.
Frequently implemented features include filtering, aggregating and merging real-time data from your IoT device data.

Thanks to a CEP, it is easy for applications to only subscribe to value-added data which is relevant to them.
CEP technology is sometimes also referred to as event stream analysis, or real time event correlation.

For more information about Esper, the CEP engine used in Cepheus, please refer to its [presentation](http://www.espertech.com/esper/)
and [documentation](http://www.espertech.com/esper/documentation.php).

# Configure CEP

To start, you must configure the CEP. The CEP must know the upcoming events (entities), the upgoing events (new or not new entities)
and the statements.

In order to do so, you have to select the CEP API Add Configuration.  On the following examples, let us consider we have :

 - a set of NGSI-capable temperature sensors across multiple rooms
 - multiple rooms over a set of floors
 - a set of LoRa trackers
 - Geofencing alert that is triggered when a Tracker enters or exits a geofenced zone

You would like to get the average temperature (over the last 10 minutes) on each floor (updated every 1 minute).
For this, you send the configuration:


```
POST /cep/v1/admin/config HTTP/1.1
Host: {{HOST}}
Accept: application/json
Content-Type: application/json
Fiware-Service: {{Fiware-Service}}
Fiware-ServicePath: {{Fiware-ServicePath}}
X-Auth-Token: {{token}}

{
  "host":"http://{{HOST}}/cep",
  "in":[
    {
      "id":"Room.*",
      "type":"Room",
      "isPattern": true,
      "attributes":[
        { "name":"temperature", "type":"double" },
        { "name":"floor", "type":"string" }
      ]
    },
    {
        "id":".*",
        "type":"tracker",
        "isPattern": true,
        "attributes":[
           { "name":"time", "type":"date" },
           { "name":"position", "type":"geo:point" }
        ]
    }
  ],
  "out":[
    {
      "id":"Floor1",
      "type":"Floor",
      "attributes":[
        { "name":"temperature", "type":"double" }
      ],
      "brokers": [
        { "url":"http://{{HOST}}/cb",
          "serviceName": "{{Fiware-Service}}",
          "servicePath": "{{Fiware-ServicePath}}",
          "authToken": "{{TOKEN}}"
        }
      ]
    },
    {
        "id":"Fence1",
        "type":"Alert",
        "attributes":[
            { "name":"time", "type":"date" },
            { "name":"position", "type":"geo:point" },
            { "name":"inside", "type":"boolean" }
        ],
      "brokers": [
        { "url":"http://{{HOST}}/cb",
          "serviceName": "{{Fiware-Service}}",
          "servicePath": "{{Fiware-ServicePath}}",
          "authToken": "{{TOKEN}}"
        }
      ]
    }
  ],
  "statements":[
    "INSERT INTO Floor SELECT floor as id, avg(temperature) as temperature FROM Room.win:time(10 min) GROUP BY floor OUTPUT LAST EVERY 10 sec",
    "CREATE VARIABLE Geometry fence = polygon({point(43.59432, 1.44934), point(43.59508, 1.45156), point(43.5949, 1.45191), point(43.59352, 1.45014), point(43.59432, 1.44934)})",
    "CREATE WINDOW TrackerState.std:unique(id) as Alert",
    "INSERT INTO TrackerInside SELECT *, fence.contains(position) as inside FROM tracker",
    "INSERT INTO FenceCross SELECT a.* FROM pattern [ every a=TrackerInside -> (timer:interval(4 sec) and not TrackerInside(id=a.id, inside!=a.inside)) ]",
    "ON FenceCross fc MERGE TrackerState ts WHERE fc.id = ts.id WHEN NOT MATCHED THEN INSERT SELECT id, time, position, inside WHEN MATCHED AND fc.inside != ts.inside THEN UPDATE SET inside = fc.inside",
    "INSERT INTO Alert SELECT * FROM TrackerState"
  ]
}

```

# Activate the CEP

You have to choose which data from your IoT device is relevant to be perform by the CEP at the FIWARE IoT Stack.

In order to do so, you have to select the Data API entity attributes. On the following examples, the Room entity will be selected:
```
POST /cb/v1/subscribeContext HTTP/1.1
Host: {{HOST}}
Accept: application/json
Content-Type: application/json
Fiware-Service: {{Fiware-Service}}
Fiware-ServicePath: {{Fiware-ServicePath}}
X-Auth-Token: {{token}}

{
   "entities": [
     {
       "id": "Room1",
       "type": "Room",
       "isPattern": "false"
     }
   ],
   "reference": "http://{{HOST}}/cep/notify",
   "duration": "P1D",
   "notifyConditions": [
     {
       "type": "ONCHANGE",
       "condValues": [
         "temperature"
       ]
     }
   ],
   "throttling": "PT5S"
 }

```

For the geofencing example, the tracker entity will be selected:

```
POST /cb/v1/subscribeContext HTTP/1.1
Host: {{HOST}}
Accept: application/json
Content-Type: application/json
Fiware-Service: {{Fiware-Service}}
Fiware-ServicePath: {{Fiware-ServicePath}}
X-Auth-Token: {{token}}

{
  "entities": [
    {
      "id": ".*",
      "type": "tracker",
      "isPattern": "true"
    }
  ],
  "reference": "http://{{HOST}}/cep/notify",
  "duration": "P1D",
  "notifyConditions": [
    {
      "type": "ONCHANGE",
      "condValues": [
        "position"
      ]
    }
  ],
  "throttling": "PT5S"
}

```

If you are familiar with FIWARE components, on this request you are using the Data API subscription operation to notify new data from your device to the FIWARE CEP component providing the CEP API.

# Get results

Your device data is now sended in the CEP, and you can get the average of temperatures in your application.

You can get obtain the average as follows:

```
GET /cb/v1/contextEntities/Floor1 HTTP/1.1
Host: {{HOST}}
Accept: application/json
Content-Type: application/json
Fiware-Service: {{Fiware-Service}}
Fiware-ServicePath: {{Fiware-ServicePath}}
X-Auth-Token: {{token}}
```

You will get the Floor1 entity with the temperature attribute (average) values:

```
HTTP 200 OK
Content-Type : application/json

{
  "contextElement": {
    "type": "Floor",
    "isPattern": "false",
    "id": "Floor1",
    "attributes": [
      {
        "name": "temperature",
        "type": "double",
        "value": "27.5"
      }
    ]
  },
  "statusCode": {
    "code": "200",
    "reasonPhrase": "OK"
  }
}

```

Your tracker data is also sended in the CEP, and you can receive geofencing alert in your application.

You can get obtain the alert as follows:

```
GET /cb/v1/contextEntities/Fence1 HTTP/1.1
Host: {{HOST}}
Accept: application/json
Content-Type: application/json
Fiware-Service: {{Fiware-Service}}
Fiware-ServicePath: {{Fiware-ServicePath}}
X-Auth-Token: {{token}}
```

You will get the Fence1 entity with the inside attribute values. When inside attribut is true, it means that the tracker enters in the geofencing zone.
When inside attribut is false, it means that the tracker exits out the geofencing zone.

```
HTTP 200 OK
Content-Type : application/json

{
  "contextElement": {
    "type": "Alert",
    "isPattern": "false",
    "id": "Fence1",
    "attributes": [
       {
            "name": "time",
            "type": "date",
            "value": "2015-12-09T16:17:57Z"
       },
      {
            "name": "position",
            "type": "geo:point",
            "value": "43.59432, 1.44934"
      },
      {
        "name": "inside",
        "type": "boolean",
        "value": "true"
      }
    ]
  },
  "statusCode": {
    "code": "200",
    "reasonPhrase": "OK"
  }
}

```

# In more detail ...

You can get more information about the FIWARE component providing this functionalty, reference API documentation and source code at [Complex Event Processing (Cepheus)](cep.md)
