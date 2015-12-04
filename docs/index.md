FIWARE IoT Stack allows you to connect devices and receive data, integrating all 
device protocols and connectivity methods, understanding and interpreting relevant information. 
It isolates data processing and application service layers from the device and network complexity, 
in terms of access, security and network protocols.

These are the main benefits of solutions 'powered by the FIWARE IoT Stack':

- Simple sensor data integration 
- Device-independent APIs for quick app development & lock-in prevention
- Modular
- Scalable. High available
- Open & standards based. FIWARE compliant

## APIs available 

FIWARE IoT Stack provides the following APIs:

- [Device API](device_api.md): allows managing devices, sending data from the device to the cloud and receiving commands.
- [Lora Agent API](loraagent_api.md): allows sending data from the LoRa device to the cloud and receiving commands.
- [Data API](data_api.md): allows querying and subscribing to data stored at the cloud.
- [Historic Data API](historicdata_api.md): allows querying historic data series and statistics stored at the cloud.

## FIWARE Components

FIWARE IoT Stack is based on the following [FIWARE components](walkthrough.md) in order to provide its functionality:

- Device Backend Gateway (IDAS)
- Protocol Adapter LoRa (Lora Agent)
- Context Broker (Orion)
- Short Term Historic (STH)

