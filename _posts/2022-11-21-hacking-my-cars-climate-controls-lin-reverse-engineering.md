---
title: "Hacking My Car's Climate Controls: LIN Reverse Engineering"
author: Jacob Schooley
excerpt: "I've been working on reverse engineering the LIN bus in my car's climate control system. This post is a summary of what I've learned so far."
tags:
  - car hacking
  - hardware
  - personal projects
image:
  path: /assets/images/2022-11-17-i-built-a-monitor-just-for-spotify/monitor.jpg
  width: 1200
  height: 630
---

It's not a great feeling to walk out to your car in the morning and find that your windshield is completely covered in ice. Scraping it off is a pain, and it's even worse when you're in a hurry.

I used to drive a 20-year-old Honda Accord. It had a barebones climate control system, with buttons to choose where the air would blow, and dials to choose the temperature and fan speed. It was simple, but it worked. If I set the mode to defrost and then turned off the car, the defroster would still be on when I started the car again.

Not so in newer cars. My current car, a 5th generation Toyota RAV4, won't keep the front or rear defrosters on after restarting the car. I installed a remote start system, but without being able to keep the defroster on, it's not very useful. I could just as easily start the car myself and then turn on the defroster, but that would require me to go outside earlier than I want to.

I've been told this is a safety feature (and common on many newer cars, not just my Toyota). However, according to [Toyota's website](https://support.toyota.com/s/article/Can-I-control-the-AC-10212?language=en_US), when you remote start the car, "if the outside temperature is less than 41°F, the front and rear defrosters will turn on." Ah, so you _can_ remotely defrost the windshield—if you bought a trim level that comes with Toyota's remote start subscription service. I didn't.

So I've been thinking about how I could hack my car's climate control system to keep the defroster on after restarting the car.

### The CAN Bus

With almost everything in modern cars being computer-controlled, CAN is the most common way to communicate between different systems. The CAN bus is a serial bus that uses a twisted pair of wires to carry data. It's a multi-master bus, meaning that multiple devices can be connected to the bus and transmit data at the same time. Each device has a unique ID, and the bus uses arbitration to determine which device gets to transmit data at any given time.

CAN hacking is not new. There are many tools and a plethora of documentation and tutorials available for sniffing, intercepting, and modifying CAN traffic. [Comma.ai](https://comma.ai/), which I use every day, uses a CAN interceptor to modify communications between the car's lane keeping camera and the car's computer to send commands to drive the car semi-autonomously.

I figured that if CAN messages could _drive the car_, then they could probably control the climate control system, too. I started by connecting my computer to the [panda](https://comma.ai/shop/panda) that I'd installed already for openpilot and opening [cabana](https://cabana.comma.ai/) to view the raw CAN traffic. Unfortunately, nothing changed when I messed with the climate control buttons. Cars have multiple CAN buses, so I tried sniffing other buses, including the one connected to the remote starter and the one on the back of the radio. Still nothing.

Simply put, CAN is not used for controlling the climate system in my car. And if there's no CAN traffic, there's nothing to sniff. I needed to find another way to control the climate system.

(Actually, there might be a way to control it through CAN. Newer Highlanders and Venzas with the 12-inch screen can control the climate system through the touchscreen, and some RAV4 Prime and Prius Prime models can control it through the app. If you have one of those, you're welcome to try sniffing the CAN traffic and see if you can figure out how to control the climate system. I'd love to hear about it.)

### The LIN Bus

I started looking for other ways to control the climate system. Studying the wiring diagrams for my car, I found that the control panel on the dash is connected to the climate control unit through a LIN bus.

LIN is a serial bus that uses a single wire to carry data. It is different from CAN in that it is a single-master bus, meaning that only one device can transmit data at a time. The master node sends data to up to 16 slave nodes, and the slave devices can respond to requests sent by the master. [More information about LIN can be found here](https://www.csselectronics.com/pages/lin-bus-protocol-intro-basics).

Because LIN is much newer than CAN, well-documented hardware is scarce and open-source utilities for sniffing and intercepting traffic are practically nonexistent. I decided to start by reverse engineering the LIN protocol.
