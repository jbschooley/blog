---
title: "Hacking My Car's Climate Controls, Part 2: Building the Interceptor"
author: Jacob Schooley
excerpt: "I'll walk through the process of building the hardware that will allow me to intercept and modify the CAN bus messages sent by my car's climate control system."
tags:
  - car hacking
  - hardware
  - personal projects
image:
  path: /assets/images/2022-11-30-hacking-my-cars-climate-controls-lin-reverse-engineering/panel.jpg
  width: 1365
  height: 617
---

We’ve had an incredible winter and Utah this year. Storm after storm have been blessing us with an unforgettable season of back to back powder days, making all that money I spent on my ski pass totally worth it.

As much as I would like to have my favorite resort to myself, that isn’t realistic. Everyone in the general Salt Lake area has to compete for their share of untouched pow. And for me, being over an hour away without traffic, that means I have to wake up way too early to even have a chance at a parking spot. If my car is covered in snow, the 10 minutes I’d have to wait for the defroster to kick in and melt it off the windshield can be the difference between parking right next to the lift or parking on the side of the road a mile away. Anyone who has hiked in ski boots before knows that is definitely not an ideal way to start the day.

This has made it imperative that I find time between classes and my job to finish this project. It’s really nice walking out to my car and only having to brush off the already melted snow before I can pick up the boys and head to the mountains.

I’m on a college student budget (and already spent all my money on ski gear), and part of the reason I’m posting this is to show that DIY hardware doesn’t have to be expensive, and that, even on a very limited budget, you can make some really cool stuff. Feel free to replace parts with better more expensive ones, or even design a custom PCB if you have the knowledge for that (and please let me know if you do as I would love to use it)!

### Materials

#### Parts
- Arduino Mega [amazon](https://www.amazon.com/ARDUINO-MEGA-2560-REV3-A000067/dp/B0046AMGW0/) [aliexpress](https://www.aliexpress.us/item/2251832678521697.html) [search](https://www.aliexpress.com/w/wholesale-arduino-mega.html?SearchText=arduino+mega)
- Mikroe LIN Shield for Arduino Mega [mikroe](https://www.mikroe.com/arduino-mega-click-shield) [mouser](https://www.mouser.com/ProductDetail/932-MIKROE-1900)
- 2x Mikroe LIN Click [mikroe](https://www.mikroe.com/lin-click) [mouser](https://www.mouser.com/ProductDetail/Mikroe/MIKROE-3816?qs=Cb2nCFKsA8qi3nKltYdgQw%3D%3D)
  - Alternatively, you can use the ATA663254 click as it includes a 5V regulator. 2 of these in parallel should be able to power the Arduino, eliminating the need for the car charger. [mikroe](https://www.mikroe.com/ata663254-click) [mouser](https://www.mouser.com/ProductDetail/Mikroe/MIKROE-2872?qs=f9yNj16SXrJrtHOXHs2kUA%3D%3D)
- Toyota connectors 90980-12369 and 90980-12370 (get 2 of each because thye're easy to break) [aliexpress](https://www.aliexpress.us/item/3256803593976654.html)
- Protoboard (insert size)
- Dollar store USB car charger
- 18-20 gauge wire
- Gorilla mounting tape (or 3M VHB tape) [amazon](https://www.amazon.com/Gorilla-Heavy-Double-Sided-Mounting/dp/B082TQ3KB5/)

#### Tools
- Soldering kit. Get a good one with a temperature dial; if you try to use a cheap soldering iron from Walmart, you’ll end up very frustrated very quickly. These are expensive so you may want to borrow one. [amazon](https://www.amazon.com/Weller-Digital-Soldering-Station-WE1010NA/dp/B077JDGY1J/)
- Extraction tool for removing wires from connectors (I used a breadboard pin, but they break easily so I wouldn’t recommend it)
- Trim removal tool [amazon](https://www.amazon.com/XBRN-Removal-Window-Fastener-Remover/dp/B07R44VJHN)
- Digital multimeter [amazon](https://www.amazon.com/Crenova-Auto-Ranging-Multimeter-Measuring-Backlight/dp/B00KXX2OYY/)

### Building the Interceptor

#### Step 1: Assemble the Mikroe LIN Shield

The LIN shield comes with header pins that are not attached, so you will need to solder the ones that you’ll be using. You do not need to solder all of them. Solder pins for:

- ground
- 5V
- serial RX and TX
- the enable pins for both LIN clicks

If you aren’t a soldering expert, you may want to use your multimeter to check continuity between the pins and the click headers.

#### Step 2: Solder jumpers on the LIN clicks

The basic LIN click has a jumper for selecting Slave or Master mode. It comes with the Slave position selected. On one of your clicks, you will need to solder the jumper to the Master position. This click will be the one that will be sending the messages to the car. The other click will be the Slave click, and will be receiving the messages from the car. You can also remove the 3.3V jumper and solder it to the 5V position, but this is not necessary as the Arduino will output 3.3V for the clicks to use.

If you get the ATA663254, you will need to solder the 5V jumper on both clicks and the L-PULL jumper on the click that will be used as the Master.

#### Step 3: Take apart your dollar store car charger

If using the ATA663254, you can skip this step. Otherwise, you will need to take apart your car charger and solder wires for 12 V, 5V, and ground to it.

#### Step 4: Prepare the Toyota connectors and wires

Remove wires from your Toyota connectors. You’ll need an extraction tool or a perfectly sized pin to get them out, and these connectors are very finicky and easy to break. I would suggest buying a second set of connectors because they’re cheap.

You will only need 10 wires total. Label 2 wires each as:

- 12V
- Ground
- LIN
- Illumination+
- Illumination-

Don't insert them back into the connectors yet. It's easier to solder them first.

#### Step 5: Solder everything together on the proto board

	4. Solder everything together on the Proto board
	5. Test every connection with your multimeter. Remember you’re dealing with your car is electrical system, which will not be cheap to repair if you fry something.
	6. Flash the Arduino
	7. Plug it in and test it!
Install it behind the dash. You may want to 3-D print a case for it doesn’t touch anything it’s not supposed to, and short itself out.
