# mowas-openhab
Holt vom MoWas JSON Api die Warndaten und durchsucht nach dem festgelegten Gebiet. Wenn dies gefunden wird, werden Typ und Titel der ersten gefundenen Warnung an die Items übergeben.
![alt text](https://www.bbk.bund.de/SharedDocs/Bilder/BBK/DE/Logos/National/MoWas_Logo.jpg;jsessionid=4A71025FBAD46B19FA764159754070E0.2_cid345?__blob=normal&v=5 "Logo Title Text 1")
![alt text](https://community-openhab-org.s3-eu-central-1.amazonaws.com/original/2X/7/7d388a86c95471f89b1bb911d96d7609a3e3a059.svg "Logo Title Text 1")

> JSONPath transformation must be installed (PaperUI)

## Item file *mowas.items*
```Openhab
Switch Warnungen
String Json "Raw JSON Warnungen" { http="<[https://warnung.bund.de/bbk.mowas/gefahrendurchsagen.json:30000:JS(returnjson.js)]" }
String Headline "Warnungs headline"
String MsgTypeMowas "Message Typ"

```
## Rule file *rules/mowas.rules*
```
rule "Convert JSON to Item String Mowas"
  when
    Item Json changed or
    System started
 then
    // use the transformation service to retrieve the value
    if(Json.state.toString != "Fehler" && Json.state.toString != 0){
        val newValue = transform("JSONPATH", "$.[0].info.[0].headline", Json.state.toString)
        val newValue2 = transform("JSONPATH", "$.[0].msgType", Json.state.toString)
        Headline.postUpdate( newValue )
        MsgTypeMowas.postUpdate (newValue2)
        Warnungen.sendCommand(ON)
    }
    if(Json.state.toString == "0"){
            Headline.postUpdate("")
            MsgTypeMowas.postUpdate ("")
            Warnungen.sendCommand(OFF)
    }
 end
```

## Javascript Transformation *transform/returnjson.js*
```Javascript
//#######################
//>>>>>>>>>>
var target = "Bielefeld"; //<=== <=== <===
//>>>>>>>>>>
//#######################
var logger = Java.type("org.slf4j.LoggerFactory").getLogger("WARNUNGEN MOWAS");
function f2() {
    try {
        var newJson = JSON.parse(input);
    } catch (err) {
        logger.error("JSON Fehler. Fehler bei dem angeforderten JSON von warnung.bund.de");
        return "Fehler";
    }
    for (var i = 0; i < newJson.length; i++) {
        var typ = newJson[i]["msgType"];
        var hdln = newJson[i]["info"][0]["headline"]
        var sub = newJson[i]["info"][0]["area"][0]["geocode"]
        for (var j = 0; j < sub.length; j++) {
            var str = sub[j]["valueName"];
            if (str == target) {
                logger.warn("Erfolgreiche Auswertung von warnung.bund.de -- Min. eine Warnung AKTIV!");
                return "[" + JSON.stringify(newJson[i]) + "]";
            }
        }
    }
    logger.warn("Erfolgreiche Auswertung von warnung.bund.de -- Keine Warnung für "+ target);
    return 0;
};
f2();
```
