---
layout: default
title: Components Logic Level Converter Cylewet
---

# Description
2 Channel Logic Level Converter 5V to 3.3V and 3.3V to 5V.

## Cost
Purchased on [Amazon](https://www.amazon.com/Cylewet-Channel-Converter-Arduino-CYT1070/dp/B073D4DJDC) 10 pcs for $10. 

## Directions for Use
The following was provided by the seller as, at the time, there are no directions for use included with the product, nothing easily found online, or on Amazon. 

![]({{"/assets/images/componentLogicLevelConvertCylewetDiagram.png"}})

```
On Sun, Jan 28, 2018 at 11:11 PM, Aloway - Amazon Marketplace wrote:
Subject: Product details inquiry from Amazon customer Wm Parker Mackenzie IV
Message from 3rd party seller:
Hi, Parker

Please see the schematic in the attached picture.

Connection instruction:
HV pin connects to 5V; LV pin connects to 3.3V; GND pin connects to the negative pole of power supply, the negative poles of 5V and 3.3V should use the same source grounding.
RXI inputs 5v TTL , RXO outputs3.3v TTL; TXI inputs and outputs 3.3V TTL, TXO inputs and outputs 5V TTL; TXI and TXO are mutual transferred.

Hope that I can help you.

Kind Regards
Hestia
Aloway Sales Team
```
## Notes
The logic level converters arrive in a small bag, each 2 channel converter is supplied with 2 6 pin connectors which need to be soldered onto the PCB. 

<img src="{{"/assets/images/componentLlConverter.jpg"}}" style="width: 150px">

![]({{"/assets/images/componentLlConverterSoldering.jpg"}})

Well they seem to work. A little concerned with the lack of any information on line, so, after receiving the response from the seller, some prototyping shows that when 5V is applied to either of the TXI pins the TXO pins provide 3.3V and when 3.3V is applied to either of the RXI pins the RXO pins provide 5V. I did not take the time to determine how fast the input pins could be varied or the response time of the output given an input; for now I will assume it is good enough for the reasonably slow interface to the ultra sonic sensor and relay in the project these were purchased for.  

It would suck to get these physically in place only to find that they do not work...

![]({{"/assets/images/breadBoard1.jpg"}})
![]({{"/assets/images/breadBoardTest.jpg"}})