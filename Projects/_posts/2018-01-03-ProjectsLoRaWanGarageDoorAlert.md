---
layout: default
title: Projects LoRaWAN Garage Door Sensor and Remote Control - 1st Post
---

Once upon a time on a very frigid day in January a boy borrows his father's car only to leave the garage door open for hours after his return... ugh... 

![]({{"/assets/images/frozenGarage.jpg"}})

If necessity is the mother of all invention then parent's should be the most inventive resource on the planet. After finding the garage door had been left open for several hours on a frigid New England January day, the idea to build a contraption which would alert those that pay for the heat in the house to it being open was born. After several more moments of thought, having the ability to shut the garage door remotely would be a cool feature to have as well; what happens if the garage door was left open after leaving...

After mildly venting a father's rage (I have been told the politically correct term for this is "discussing a learning moment"), I thought about what some of the constraints would be for this project. 
1. The device would have to not be a significant power sucking alien. I don't want to spend more on powering the device in a year then it saves me. 
2. The device needs to work in the absence of internet connectivity. Although, internet connectivity usually is very good; I don't want to have to mess with one more device when I change my WiFi credentials due to the kiddos sharing them with every Tom, Dick, and Harry who visit the house. 
3. I don't want to pay a fortune in operating expenses. 

These constraints ruled out using my favorite go to, the WiFi enabled Raspberry PI. Cellular connectivity costs an arm and a leg and is way over kill for the few bytes that will be occasionally sent. As it turns out the answer was right in my backyard, our town is blessed with fantastic public IoT access via LoRaWAN from [Senet](https://www.senetco.com). Also, working for Senet is a plus... ;-)

My kiddos asked what I was working on, and I said the Magic Box. The name kind of stuck. 

# Best Laid Plans
Well the best part of planning is making the plan, at the very least it provides some general directions. After some white board brain storming, a rough plan came together.

![]({{"/assets/images/magicBox_UseCaseAndBlockDiagram.jpg"}})

I have settled on using the ST Micro Nucleo developer kit with the LRWAN-1 shield. It is a little more money then I really wanted to spend. However, the micro-controller and developer board have everything I need, the LoRaWAN shield is based on a well known design using a driver I can make changes to, if necessary, and the programming environment is well known in the industry. With a few lines of well placed code, I should be able to make this device sip energy as it sits in wait to tattle on the kids.

Sure, could I go out and buy a LoRaWAN device with an accelerometer, connect it to one of the popular IoT services via the Senet network, and have a sensor to tell me when the garage door is open. Yep, I could, matter of fact [Tektelic](https://tektelic.com/iot/lorawan-sensors/) makes a great device. However, I want to be able to control the door remotely and even more importantly where would the fun be in that. 

In order to control the function of the garage door, I will be mounting the unit on the ceiling. This will allow a cable to be run to the garage door's motor controller, mounting the unit to the garage door would mean the cable would need to retract in some way. With the unit mounted to the ceiling, I needed something to determine that the garage door is up without being able to use an accelerometer; for this I am going to try out the HY-SRF05 ultra sonic range tester. The range tester is a favorite for those people building cool Arduino devices; looking at the specs I should be able to get it precise enough to know if it is a garage door or something else lower to the ground.

## Component Wiring Diagram
As I wait for the ST Nucleo developer module, the following is the current plan for wiring it up. 
![]({{"/assets/images/MagicBox_GarageDoorOpener_WiringDiagram.png"}})

## BOM Cost
Rough estimate is I can build this thing for less than $90.

|QTY|Mfg Part No         |Description                        |Unit Cost|Line Cost|Manufacturer      |Supplier             |
|:--|:-------------------|:----------------------------------|:--------|:--------|:-----------------|:--------------------|
| 1 |P-NUCLEO LRWAN1     |Nucleo 64 L073RZ & LoRaWan Radio   |$70.00   |$70.00   |ST-Electronics    |ST-Electronics       |
| 1 |HY-SRF05            |Ultra Sonic Range Tester           |$7.00    |$7.00    |SMAKN             |Amazon               |
| 1 |CYT1070             |Logic Level Converter - Cylewet    |$1.00    |$1.00    |Qianxin           |Amazon               |
| 1 |JQC-3F(T73)         |5V Relay                           |$1.00    |$1.00    |Baomain           |Amazon               |
| 1 |                    |5V Power Supply                    |$8.00    |$8.00    |                  |Amazon               |
| 1 |                    |LuzBot Filament Enclosure          |$5.00    |$5.00    |                  |LuzBot               |

## Where to Next
Next stop is to start building pieces of this up and wrapping the pieces in some software.

## References
* ST Nucleo
  * [Software Development Tools](http://www.st.com/content/ccc/resource/technical/document/user_manual/1b/03/1b/b4/88/20/4e/cd/DM00105928.pdf/files/DM00105928.pdf/jcr:content/translations/en.DM00105928.pdf)
  * [Hardware Users Manual](http://www.st.com/content/ccc/resource/technical/document/user_manual/98/2e/fa/4b/e0/82/43/b7/DM00105823.pdf/files/DM00105823.pdf/jcr:content/translations/en.DM00105823.pdf)
  * [Data Sheet](http://www.st.com/content/ccc/resource/technical/document/datasheet/d8/b3/7b/dc/18/22/44/f5/DM00141036.pdf/files/DM00141036.pdf/jcr:content/translations/en.DM00141036.pdf)
* SRF05 Range Sensor
  * [Manual](https://www.robot-electronics.co.uk/htm/srf05tech.htm)
* JQC-3F Relay
  * [Data Sheet](https://voron.ua/files/pdf/relay/JQC-3F(T73).pdf)



