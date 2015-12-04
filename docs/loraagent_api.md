
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

Sending an observation from LoRa IoT devices is extremely efficient and simple with the following HTTP POST request:

```
POST   http://{{HOST}}:8090/notifications/
Headers: {'content-type': 'application/xml’; 'X-Auth-Token' : [TOKEN]; "Fiware-Service: smartcity”; "Fiware-ServicePath: /team1"}
Payload: <?xml version="1.0" encoding="UTF-8"?>
         <DevEUI_uplink xmlns="http://uri.actility.com/lora"><Time>2015-11-24T10:50:24.823+01:00</Time>
         <DevEUI>20635F0012000011</DevEUI><FPort>2</FPort><FCntUp>15831</FCntUp><FCntDn>8</FCntDn>
         <payload_hex>22fe0a834413ffff1a01ec00d1f50000</payload_hex><mic_hex>95a64f6a</mic_hex>
         <Lrcid>00000065</Lrcid><LrrRSSI>-57.000000</LrrRSSI><LrrSNR>9.250000</LrrSNR>
         <SpFact>10</SpFact><SubBand>G2</SubBand><Channel>LC5</Channel><DevLrrCnt>1</DevLrrCnt>
         <Lrrid>5363c981</Lrrid><LrrLAT>43.634010</LrrLAT><LrrLON>1.374207</LrrLON><Lrrs><Lrr><Lrrid>5363c981</Lrrid>
         <LrrRSSI>-57.000000</LrrRSSI><LrrSNR>9.250000</LrrSNR></Lrr></Lrrs><CustomerID>100000695</CustomerID>
         <CustomerData>{"alr":{"pro":"ABEE/APY","ver":"2"}}</CustomerData><ModelCfg>0</ModelCfg></DevEUI_uplink>
```

The previous example sends a Position that is automatically sent by Lora Agent as an updateContext to the corresponding entity at the ContextBroker.

Finally, after connecting your IoT devices this way you (or any other developer with the right access permissions) should be able to use the Data API to read the NGSI entity assigned to your device.

The corresponding updateContext sended by LoRa Agent:

```
POST   http://{{HOST}}/cb/v1/updateContext/
Headers: {'content-type': 'application/xml’; 'X-Auth-Token' : [TOKEN]; "Fiware-Service: smartcity”; "Fiware-ServicePath: [Fiware-ServicePath]"}
Payload: <updateContextRequest>
         <updateAction>APPEND</updateAction>
         <contextElementList>
            <contextElement>
                <entityId type="tracker" isPattern="false">
                    <id>20635F0012000011</id>
                </entityId>
                <contextAttributeList>
                    <contextAttribute>
                        <name>temperature</name>
                        <type>double</type>
                        <contextValue>25.5</contextValue>
                        <metadata/>
                    </contextAttribute>
                    <contextAttribute>
                        <name>mode</name>
                        <type>string</type>
                        <contextValue>POSITION</contextValue>
                        <metadata/>
                    </contextAttribute>
                    <contextAttribute>
                        <name>tracking</name>
                        <type>string</type>
                        <contextValue>SLEEP</contextValue>
                        <metadata/>
                    </contextAttribute>
                    <contextAttribute>
                        <name>levelBattery</name>
                        <type>long</type>
                        <contextValue>0.75</contextValue>
                        <metadata/>
                    </contextAttribute>
                    <contextAttribute>
                        <name>position</name>
                        <type>coords</type>
                        <contextValue>45.5, -10.5</contextValue>
                        <metadata>
                            <name>location</name>
                            <type>string</type>
                            <value>WGS84</value>
                        </metadata>
                    </contextAttribute>
                </contextAttributeList>
            </contextElement>
         </contextElementList>
         </updateContextRequest>

```


# Act upon devices 

In order to send commands to devices, you just need to know which attributes correspond to commands and update them.

You can declare the command related attributes at the registry process (POST request) in the following way:

```
POST   http://{{HOST}}/cb/v1/updateContext
Headers: {'content-type': 'application/xml’; 'X-Auth-Token' : [TOKEN]; "Fiware-Service: smartcity”; "Fiware-ServicePath: [Fiware-ServicePath]"}
Payload:
        <updateContextRequest>
            <updateAction>UPDATE</updateAction>
            <contextElementList>
                <contextElement>
                    <entityId type="tracker" isPattern="false">
                        <id>20635F0012000011</id>
                    </entityId>
                    <contextAttributeList>
                        <contextAttribute>
                            <name>SET_tracking</name>
                            <type>string</type>
                            <contextValue>FASTTRACK</contextValue>
                            <metadata/>
                        </contextAttribute>
                    </contextAttributeList>
                </contextElement>
            </contextElementList>
        </updateContextRequest>

```

