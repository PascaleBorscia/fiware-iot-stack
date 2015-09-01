The Complex Event Processing API allows you to analyze data from your IoT device and trigger actions.

This API allows you for example to define rules to trigger email notifications based on the data value thresholds or the the lack of updates from a certain device.

These rules are expressed as an EPL sentence. [EPL](http://www.espertech.com/esper/index.php) is a domain language of Esper, the engine for processing events used. This EPL sentence matches an incoming event if satisfies the conditions and generates an "action-event" that will be sent back to FIWARE IoT Stack to execute the associated action. 

Remember you can always use the [Management Portal](portal.md) to define basic rules, which are a perfect fit for most frequent use cases.


# Activate rules

You have to choose which data from your IoT device is relevant to be evaluated by your rules at the FIWARE IoT Stack.

In order to do so, you have to select the Data API entity attributes. On the following example, the temperature attribute will be selected:

```
POST /NGSI10/subscribeContext HTTP/1.1
Host: test.ttcloud.net:1026
Accept: application/json
Content-Type: application/json
Fiware-Service: {{Fiware-Service}} 
Fiware-ServicePath: {{Fiware-ServicePath}} 
X-Auth-Token: {{user-token}}

{
    "entities": [
        {"type": "device",
        "isPattern": "false",
        "id": "mydevice"
        }
    ],
    "reference": "http://test.ttcloud.net:xxxx/notify", 
    "duration": "P1Y",
    "notifyConditions": [
           {
                "type": "ONCHANGE", 
                "condValues": ["TimeInstant" ]
} ],
"throttling": "PT1S" }
```

If you are familiar with FIWARE components, on this request you are using the Data API subscription operation to notify new data from your device to the FIWARE CEP component providing the Complex Event Processing API.

# Create rule

# In more detail ...

