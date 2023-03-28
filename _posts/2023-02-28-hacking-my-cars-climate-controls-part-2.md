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

The basic LIN click has a jumper for selecting Responder or Commander mode. It comes with the Responder position selected. On one of your clicks, you will need to solder the jumper to the Commander position. This click will be the one that will be sending the messages to the car. The other click will be the Responder click, and will be receiving the messages from the car. You can also remove the 3.3V jumper and solder it to the 5V position, but this is not necessary as the Arduino will output 3.3V for the clicks to use.

If you get the ATA663254, you will need to solder the 5V jumper on both clicks and the L-PULL jumper on the click that will be used as the Commander.

*Note: While the official terms for LIN device roles have been changed to Commander and Responder, most documentation, including the documentation for these LIN clicks, still uses the terms Master and Slave.*

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

The best way I found to set up the proto board is by soldering pins to it so that it takes up the 3rd click slot on the shield. This way, you can keep everything attached to the Arduino rather than having to find another place to mount the proto board.

Some connections can be made without the board, but having a central connection area simplifies things a bit.

You want to connect the following:

- All grounds, including the Arduino ground, both LIN grounds, the car charger ground, and ground from both connectors
- All 12V, including the Arduino 12V, both LIN 12V, the car charger 12V, and 12V from both connectors
- The car charger 5V to a 5V pin on the Arduino

The rest of the connections can be made directly, but connecting them through the proto board allows for cleaner wiring:

- Illumination+ and Illumination- from both connectors (these are direct and do not pass through the Arduino)
- LIN from the car connector to the Responder LIN click
- LIN from the panel connector to the Commander LIN click

#### Step 6: Test every connection with your multimeter

Remember, you’re dealing with your car's electrical system. Electrical problems in cars, especially newer ones, can be very expensive to fix, so it's best to be safe. Make sure that you have continuity between all of your connections and that you have no shorts. If you have a short, you will need to find it and fix it before continuing.

#### Step 7: Flash the Arduino

Clone [jbschooley/ToyotaLinInterceptor](https://github.com/jbschooley/ToyotaLinInterceptor) and push it to your Arduino. I use IntelliJ IDEA with the PlatformIO plugin. You may be able to use another IDE, but I haven’t tried it.

#### Step 8: Plug it in and test it!

[Remove the climate control panel](https://www.youtube.com/watch?v=eL141juGyrM). Unplug it from the car and attach the interceptor between the panel and the car. Plug it back in and turn the car on. If everything is working, you should see the panel light up and the climate control buttons should work. By default, there won't be a preset configured, so hold down the S-Mode button for a second, then turn the driver's temperature knob to the right to set a preset.

If everything works, install it behind the dash. You may want to [3D print a case](https://github.com/jbschooley/ToyotaLinInterceptor/tree/main/Case) so it doesn’t touch anything it’s not supposed to and short itself out. This case *barely* fits behind the dash, and it was a pain to get it in, but it does fit.

### Software

The software is written in C++ and uses the [Arduino framework](https://www.arduino.cc/). I've tried to keep things as simple as possible and keep things modular so that it's easy to add new features. You'll notice that all code is kept in .h files. No, this isn't best practice, but it made development a bit easier and the difference in compile time by keeping implementation code in .cpp files was negligible.

- `Button.h` - handles state for a button on the climate control panel and triggers actions on press, hold, and release
  - `Menu.h` - holding the S-Mode button will replace the driver's temperature display with a preset menu. Turning the driver's temperature knob will cycle between presets, and the preset will be saved when the button is released. Presets are displayed as L1, L2, etc. on the display. (Why L#? Because I had to pick from the text the panel was configured to display, and I figured those were the least likely to be seen during normal operation. I think the car can display up to L32.) If new presets are added, increment the `numPresets` constant.
  - `Toggle.h` - holding the Eco button will toggle the preset on and off. The display will toggle between the preset number and `OFF` when the button is held. (I don't want it always cranking the heat to high when I'm running errands, so I turn it off for that and turn it back on when I park for the night or know it'll be snowing. I might be able to eliminate this if I can figure out a way to detect when the remote starter is used.)
  - `OffButton.h` - turns off the rear defroster when the Off button is pressed.
- `LINController.h` - handles low-level sending and receiving messages on the LIN bus. The same code can be used for reading messages as both the Commander and Responder. `sendFrame` is used for sending data as the Master, `sendRequest` is used to request data from the Responder, and `sendResponse` is used for responding to messages as the Responder.
- `Handler.h` - handles state for reading and writing messages on the LIN bus and ensures received messages are validated before being processed.
  - `CarHandler.h` - handles reading, forwarding, and responding to messages from the car
  - `PanelHandler.h` - handles sending data and requests and responding to messages from the panel
- `DataStore.h` - stores data received from the car and the panel and handles reading and writing to EEPROM
- `Modifier.h` - used to modify intercepted data before it's forwarded. This includes modifying temperature, fan speed, and buttons pressed.
- `Preset.h` - defines how a preset is applied
  - `PresetMaxDefrost.h` - turns on front and rear defrosters, sets fan speed to high, sets temperature to max, and turns off eco mode and S-Mode
- `PresetController.h` - handles applying the correct presets. This is where you can add new presets.
- `Timer.h` - a debug timer that can be used to measure how long it takes to process a message. It's not used in the final code, but it's useful for debugging.

I've tried my best to keep things organized, but my code is definitely not perfect. I'm open to suggestions and pull requests.
