
Device API is exposed by the IoT agent. It allows you to:

- Register your device to reduce the message footprint and use commands.
- Send data from the device to the FIWARE IoT Stack using UL20 and MQTT protocols.
- Send commands from your application to the device using UL20 and MQTT protocols.

You can use the examples provided in section "Device API", a "MQTT tutorial" and "UL20 tutorial" of the Postman collection to get a good overview of the API.

# Register your IoT device

Remember this step is optional, it is only required if you want to use the MQTT protocole or UL20 commands in order to act upon devices or want to define a mapping to reduce the attributes identifier when you send observations to reduce the message size.

If you simply want to send UL20 observations you can skip this and just go to the "Send observations" section.

## Registering for UL2.0

On this sample a device is registered to send temperature observations using UL2.0 protocol and a PING command:

```
POST /ul/iot/devices
Host: {{HOST}}
Content-Type: application/json
X-Auth-Token: {{TOKEN}}
Fiware-Service: smartcity
Fiware-ServicePath: /teamX

{
  "devices": [
    {
      "device_id": "{{DEV_ID}}",
      "entity_name": "{{ENTITY_ID}}",
      "entity_type": "thing",
      "protocol": "PDI-IoTA-UltraLight",
      "timezone": "Europe/Paris",
	  "endpoint": "http://{{DEVICE_IP}}:{{PORT}}",
      "attributes": [
        {
          "object_id": "t",
          "name": "temperature",
          "type": "int"
        }
      ],
      "commands": [
        {
          "name": "ping",
          "type": "command",
          "value": "[Dev_ID]@ping|%s"
        }
      ]
    }
  ]
}
```

The important parameters to be defined are:

- "device_id": the device identifier.
- "entity_name": the entity ID to be used at the Context Broker
- "entity_type": the entity Type to be used at the Context Broker
- "attributes": Used to map UL2.0 alias to Context Broker entities.
- "commands": Used to indicate which commands the device supports.
- "endpoint": optional field. Commands will be considered as push if endpoint is defined or pull if not defined (see Act upon device section).

## Registering for MQTT

In this case, it will be necessary to provision a device in the MQTT IoT Agent:

```
POST /ul/iot/devices
Host: {{HOST}}
Content-Type: application/json
X-Auth-Token: {{TOKEN}}
Fiware-Service: smartcity
Fiware-ServicePath: /teamX

{
  "devices": [
    {
      "device_id": "[DEV_ID]",
      "entity_name": "[ENTITY_ID]",
      "entity_type": "thing",
      "protocol": "PDI-IoTA-MQTT-UltraLight",
      "timezone": "Europe/Paris",
      "attributes": [
        {
          "object_id": "t",
          "name": "temperature",
          "type": "int"
        }
      ],
      "commands": [
        {
          "name": "ping",
          "type": "command",
          "value": "[Dev_ID]@ping|%s"
        }
      ]
    }
  ]
}
```

The only diffrence with UL2.0 is the the of `PDI-IoTA-MQTT-UltraLight` as the protocol.

# Send observations

## Sending using UL2.0

Ultralight2.0 (UL2.0) is a proposed simplification of the SensorML (SML) standard – and get those devices sending  their measurements (observations) to the ContextBroker. Ultralight2.0 is selected in this example because of its simplicity.

Sending an observation from IoT devices is extremely efficient and simple with the following HTTP POST request:

```
POST  /ul/iot/d?k={{TOKEN}}&i={{device_id}}
Host: {{HOST}}
Content-Type: application/text

t|25
```

The previous example sends an update of the Temperature attribute that is automatically sent by IoT Agent to the corresponding entity at the Context Broker.

Sending multiple observations in the same message is also possible with the following payload:

```
POST  /ul/iot/d?k={{TOKEN}}&i={{device_id}}
Host: {{HOST}}
Content-Type: application/text

t|23#h|80#l|95#m|Quiet
```

Finally, after connecting your IoT devices this way you (or any other developer with the right access permissions) should be able to use the Data API to read the NGSI entity assigned to your device.

## Sending using MQTT

Devices can publish MQTT messages to the IoTAgent. Those messages contain one piece of information each. That means that one message will be translated into one single entity on the Contex Broker domain. The information can be typically sensors' measures.

This is the topic hierarchy that has to be used by devices:

```
{{TOKEN}}/{{device-id}}/{{type}}
```

Where:

-  `TOKEN`: this is a unique value per service.
-  `device-id`: this is typically a sensor id, it has to be unique per `TOKEN`.
- `type`: it is the actual phenomenon being measured, for example: temperature, pressure, etc… this is the name of the attribute being published on ContextBroker.

Example:

```
$ mosquitto_pub -h $HOST -t TOKEN/mydevice/t -m 44.4 -u TOKEN
```

Another scenario can happen when devices send more than one phenomena within the payload. That is to say, one single MQTT message carries all measures. When it comes to ContextBroker, there will be one entity publication (per device) but containing all different attributes as per measures included in the mqtt message (each phenomenon or measure will be a separate attribute). In order to be able to parse the information on the IoT Agent, devices should follow this format:

Topic:

```
{{TOKEN}}/{{device-id}}/mul20
```

Example:

```
$ mosquitto_pub -h $HOST -t TOKEN/mydevice/mul20 -m "t|5.4#o|4.3#n|3.2#c|2.1" -u TOKEN
```

# Act upon devices

In order to send commands to devices, you just need to know which attributes correspond to commands and update them.

You can declare the command related attributes at the registry process (POST request) in the following way:

If you take a look to the previous device example, you can find that a "Ping" command was defined.

Any update on this attribute “Ping” at the NGSI entity in the ContextBroker will send a command to your device.

If the row "endpoint": "http://[DEVICE_IP]:[PORT]" is declared, then your device is supposed to be listening for commands at that URL in a synchronous way.

If that enpoint is not declared (if that row does not exist) then your devices is supposed to work in a polling mode and therefore receiving commands in an asynchronous way (i.e. when the device proactively asks for commands).

For a device working in the polling mode to receive commands, the full pending queue of commands will be received with the following HTTP GET request:

```
GET  /ul/iot/d?k={{TOKEN}}&i={{device-id}}
Host: {{HOST}}
```

# In more detail ...

You can get more information about the FIWARE component providing this functionalty, reference API documentation and source code at Identity Management [Backend Gateway IDAS](device_gateway.md)
