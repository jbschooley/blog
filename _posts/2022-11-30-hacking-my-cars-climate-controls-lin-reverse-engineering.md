---
title: "Hacking My Car's Climate Controls: LIN Reverse Engineering"
author: Jacob Schooley
excerpt: "I've been working on reverse engineering the LIN bus in my car's climate control system. This post is a summary of what I've learned so far."
tags:
  - car hacking
  - hardware
  - personal projects
image:
  path: /assets/images/2022-11-30-hacking-my-cars-climate-controls-lin-reverse-engineering/lin_connected_to_computer.jpg
  width: 3024
  height: 4032
---

It's not a great feeling to walk out to your car in the morning and find that your windshield is completely covered in ice. Scraping it off is a pain, and it's even worse when you're in a hurry.

I used to drive a 20-year-old Honda Accord. It had a barebones climate control system, with buttons to choose where the air would blow, and dials to choose the temperature and fan speed. It was simple, but it worked. If I set the mode to defrost and then turned off the car, the defroster would still be on when I started the car again.

Not so in newer cars. My current car, a 5th generation Toyota RAV4, won't keep the front or rear defrosters on after restarting the car. I installed a remote start system, but without being able to keep the defroster on, it's not very useful. I could just as easily start the car myself and then turn on the defroster, but that would require me to go outside earlier than I want to.

I've been told this is a safety feature (and common on many newer cars, not just my Toyota). However, according to [Toyota's website](https://support.toyota.com/s/article/Can-I-control-the-AC-10212?language=en_US), when you remote start the car, "if the outside temperature is less than 41°F, the front and rear defrosters will turn on." Ah, so you _can_ remotely defrost the windshield—if you bought a trim level that comes with Toyota's remote start subscription service. I didn't. (I did want the larger display though, so [I bought and installed one from a junkyard Corolla](https://www.rav4world.com/threads/cheap-audio-plus-8-display-upgrade-with-corolla-radio.322164/#post-2962820).)

So I've been thinking about how I could hack my car's climate control system to keep the defroster on after restarting the car.

### The CAN Bus

With almost everything in modern cars being computer-controlled, CAN is the most common way to communicate between different systems. The CAN bus is a serial bus that uses a twisted pair of wires to carry data. It's a multi-master bus, meaning that multiple devices can be connected to the bus and transmit data at the same time. Each device has a unique ID, and the bus uses arbitration to determine which device gets to transmit data at any given time.

CAN hacking is not new. There are many tools and a plethora of documentation and tutorials available for sniffing, intercepting, and modifying CAN traffic. [Comma.ai](https://comma.ai/), which I use every day, uses a CAN interceptor to modify communications between the car's lane keeping camera and the car's computer to send commands to drive the car semi-autonomously.

I figured that if CAN messages could _drive the car_, then they could probably control the climate control system, too. I started by connecting my computer to the [panda](https://comma.ai/shop/panda) that I'd installed already for openpilot and opening [cabana](https://cabana.comma.ai/) to view the raw CAN traffic. Unfortunately, nothing changed when I messed with the climate control buttons. Cars have multiple CAN buses, so I tried sniffing other buses, including the one connected to the remote starter and the one on the back of the radio. Still nothing.

Simply put, CAN is not used for controlling the climate system in my car. And if there's no CAN traffic, there's nothing to sniff. I needed to find another way to control the climate system.

(Actually, there might be a way to control it through CAN. Newer Highlanders and Venzas with the 12-inch screen can control the climate system through the touchscreen, and some RAV4 Prime and Prius Prime models can control it through the app. If you have one of those, you're welcome to try sniffing the CAN traffic and see if you can figure out how to control the climate system. I'd love to hear about it.)

### The LIN Bus

I started looking for other ways to control the climate system. Studying the wiring diagrams for my car, I found that the control panel on the dash is connected to the climate control unit through a LIN bus. (I also found that there was a CAN line connected to the control unit directly to a CAN gateway, but I'd have to rip apart most of my car's dash to get to it.)

LIN is a serial bus that uses a single wire to carry data. It is different from CAN in that it is a single-master bus, meaning that only one device can transmit data at a time. The master node sends data to up to 16 slave nodes, and the slave devices can respond to requests sent by the master. [More information about LIN can be found here](https://www.csselectronics.com/pages/lin-bus-protocol-intro-basics).

<div class="image-row">
  <img src="/assets/images/2022-11-30-hacking-my-cars-climate-controls-lin-reverse-engineering/connector_car_pinout.jpg" alt="car connector" />
  <img src="/assets/images/2022-11-30-hacking-my-cars-climate-controls-lin-reverse-engineering/connector_panel_pinout.jpg" alt="panel connector" />
</div>

| Circle color | Wire color  | Description  |
|--------------|-------------|--------------|
| Green        | Green       | LIN          |
| Yellow       | White/Black | 12V          |
| White        | White       | Ground       |
| Blue         | Blue        | Illumination |

Because LIN is much newer than CAN, well-documented hardware is scarce and open-source utilities for sniffing and intercepting traffic are practically nonexistent. I decided to start by reverse engineering the LIN data transmitted between the control panel and the climate control unit. This was a lot easier than I expected. A [LIN bus transciever with a TJA1020 chip](https://www.amazon.com/dp/B0895WQ5VM) converts LIN data to TTL serial data, which can be read by a [USB-to-UART adapter](https://www.amazon.com/dp/B00LODGRV8. I ordered one and connected it to my computer, with the LIN terminal connected to the bus alongside both the car and the panel. (It's a mess, but a functional mess.)

![LIN bus connected to computer](/assets/images/2022-11-30-hacking-my-cars-climate-controls-lin-reverse-engineering/lin_connected_to_computer.jpg)

Opening a serial connection using PuTTY gave me this:
![putty serial output](/assets/images/2022-11-30-hacking-my-cars-climate-controls-lin-reverse-engineering/putty_serial_output.png)

Looks ugly, but there's a pattern. A hex editor was a bit more useful. This sequence of bytes was repeated over and over:
```
55 B1 00 40 00 00 38 38 00 00 9D 00 55 32 00 00 00 00 38 38 00 00 5D 00 55 39 40 00 00 00 10 90 00 00 E5 00 55 BA 00 00 00 00 00 00 00 00 45 00 55 F5 00 00 00 00 38 00 00 00 D1 00 55 76 00 55 78 90 00 00 00 00 00 00 00 F6 00
```

A LIN frame begins with a sync byte, followed by a 6-bit ID, a 2-bit parity bit, 8 data bytes, and a 1-byte checksum. The transceiver returns `0x55` to represent the sync byte and `0x00` at the end of the frame. Separating the frames gives us this:

```
55 B1 00 40 00 00 38 38 00 00 9D 00
55 32 00 00 00 00 38 38 00 00 5D 00
55 39 40 00 00 00 10 90 00 00 E5 00
55 BA 00 00 00 00 00 00 00 00 45 00
55 F5 00 00 00 00 38 00 00 00 D1 00
55 76 00
55 78 90 00 00 00 00 00 00 00 F6 00
```

Disconnecting the control panel from the bus changed the output:

```
55 B1 00 40 00 00 38 38 00 00 9D 00
55 32 00 00 00 00 38 38 00 00 5D 00
55 39 00
55 BA 00
55 F5 00 00 00 00 38 00 00 00 D1 00
55 76 00
55 78 00
```

### Deciphering the Data

Immediately we can determine that `0xB1`, `0x32`, and `0xF5` are frames of data sent by the master node (the climate control unit) to the slave node (the control panel). `0x39`, `0xBA`, and `0x78` are requests, which are responded to by the control panel. I have no clue what `0x76` is. To figure out which frames do what, I simply pressed buttons and observed the changes in the data.

#### Status Messages

Frames with the ID `0xB1` contain the current status of the climate system.

<table>
  <thead>
    <tr>
      <th>Function</th>
      <th>Status</th>
      <th>Data (8 bytes + checksum)</th>
      <th>Bits (excluding checksum)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="2">power (fan bit)</td>
      <td>off</td>
      <td><code class="language-plaintext highlighter-rouge">80 00 22 00 38 38 00 80 <em>ba</em></code></td>
      <td rowspan="2"><code>&gt;&gt; 48 &amp; 7</code></td>
    </tr>
    <tr>
      <td>on</td>
      <td><code class="language-plaintext highlighter-rouge">80 23 13 00 2c 2c 00 81 <em>bd</em></code></td>
    </tr>
    <tr>
      <td rowspan="2">auto</td>
      <td>off</td>
      <td><code>80 <strong>0</strong>3 13 00 2c 2c 00 81 <em>dd</em></code></td>
      <td rowspan="2"><code>&gt;&gt; 53 &amp; 1</code></td>
    </tr>
    <tr>
      <td>on</td>
      <td><code>80 <strong>2</strong>3 13 00 2c 2c 00 81 <em>bd</em></code></td>
    </tr>
    <tr>
      <td rowspan="7">fan</td>
      <td>1</td>
      <td><code>80 0<strong>1</strong> 13 00 2c 2c 00 81 <em>df</em></code></td>
      <td rowspan="7"><code>&gt;&gt; 48 &amp; 7</code></td>
    </tr>
    <tr>
      <td>2</td>
      <td><code>80 0<strong>2</strong> 13 00 2c 2c 00 81 <em>de</em></code></td>
    </tr>
    <tr>
      <td>3</td>
      <td><code>80 0<strong>3</strong> 13 00 2c 2c 00 81 <em>dd</em></code></td>
    </tr>
    <tr>
      <td>4</td>
      <td><code>80 0<strong>4</strong> 13 00 2c 2c 00 81 <em>dc</em></code></td>
    </tr>
    <tr>
      <td>5</td>
      <td><code>80 0<strong>5</strong> 13 00 2c 2c 00 81 <em>db</em></code></td>
    </tr>
    <tr>
      <td>6</td>
      <td><code>80 0<strong>6</strong> 13 00 2c 2c 00 81 <em>da</em></code></td>
    </tr>
    <tr>
      <td>7</td>
      <td><code>80 0<strong>7</strong> 13 00 2c 2c 00 81 <em>d9</em></code></td>
    </tr>
    <tr>
      <td rowspan="5">mode</td>
      <td>face</td>
      <td><code>80 02 1<strong>1</strong> 00 2c 2c 00 81 <em>e0</em></code></td>
      <td rowspan="5"><code>&gt;&gt; 40 &amp; F</code></td>
    </tr>
    <tr>
      <td>face/feet</td>
      <td><code>80 02 1<strong>2</strong> 00 2c 2c 00 81 <em>df</em></code></td>
    </tr>
    <tr>
      <td>feet</td>
      <td><code>80 02 1<strong>3</strong> 00 2c 2c 00 81 <em>de</em></code></td>
    </tr>
    <tr>
      <td>feet/defrost</td>
      <td><code>80 02 1<strong>4</strong> 00 2c 2c 00 81 <em>dd</em></code></td>
    </tr>
    <tr>
      <td>defrost</td>
      <td><code>80 02 1<strong>9</strong> 00 2c 2c 00 81 <em>d8</em></code></td>
    </tr>
    <tr>
      <td rowspan="2">rear defrost</td>
      <td>off</td>
      <td><code>80 03 13 <strong>0</strong>0 2c 2c 00 81 <em>dd</em></code></td>
      <td rowspan="2"><code>&gt;&gt; 38 &amp; 1</code></td>
    </tr>
    <tr>
      <td>on</td>
      <td><code>80 03 13 <strong>4</strong>0 2c 2c 00 81 <em>9d</em></code></td>
    </tr>
    <tr>
      <td rowspan="2">eco</td>
      <td>off</td>
      <td><code>8<strong>0</strong> 03 13 00 2c 2c 00 81 <em>dd</em></code></td>
      <td rowspan="2"><code>&gt;&gt; 59 &amp; 1</code></td>
    </tr>
    <tr>
      <td>on</td>
      <td><code>8<strong>8</strong> 03 13 00 2c 2c 00 81 <em>d5</em></code></td>
    </tr>
    <tr>
      <td rowspan="2">recycle</td>
      <td>off</td>
      <td><code>80 02 <strong>1</strong>2 00 2c 2c 00 81 <em>df</em></code></td>
      <td rowspan="2"><code>&gt;&gt; 45 &amp; 1</code></td>
    </tr>
    <tr>
      <td>on</td>
      <td><code>80 02 <strong>2</strong>2 00 2c 2c 00 81 <em>cf</em></code></td>
    </tr>
    <tr>
      <td rowspan="2">s-mode</td>
      <td>off</td>
      <td><code><strong>0</strong>0 02 12 00 2c 2c 00 81 <em>60</em></code></td>
      <td rowspan="2"><code>&gt;&gt; 63 &amp; F</code></td>
    </tr>
    <tr>
      <td>on</td>
      <td><code><strong>8</strong>0 02 12 00 2c 2c 00 81 <em>df</em></code></td>
    </tr>
    <tr>
      <td rowspan="2">sync</td>
      <td>off</td>
      <td><code>00 02 12 <strong>0</strong>0 2c 2c 00 81 <em>60</em></code></td>
      <td rowspan="2"><code>&gt;&gt; 37 &amp; 1</code></td>
    </tr>
    <tr>
      <td>on</td>
      <td><code>00 02 12 <strong>2</strong>0 2c 2c 00 81 <em>40</em></code></td>
    </tr>
    <tr>
      <td rowspan="2">a/c</td>
      <td>off</td>
      <td><code>00 02 12 00 2c 2c 00 8<strong>0</strong> <em>61</em></code></td>
      <td rowspan="2"><code>&amp; 1</code></td>
    </tr>
    <tr>
      <td>on></td>
      <td><code>00 02 12 00 2c 2c 00 8<strong>1</strong> <em>60</em></code></td>
    </tr>
    <tr>
      <td rowspan="2">illumination</td>
      <td>off</td>
      <td><code>00 06 14 00 35 36 00 <strong>8</strong>1 <em>47</em></code></td>
      <td rowspan="2"><code>&gt;&gt; 6 &amp; 1</code></td>
    </tr>
    <tr>
      <td>on</td>
      <td><code>00 06 14 00 35 36 00 <strong>c</strong>1 <em>07</em></code></td>
    </tr>
  </tbody>
</table>

#### Button Presses

Pressing buttons on the control panel to change the temperature, fan speed, or blower mode causes a change in the response to `0x39`. Its data with nothing pressed is
`40 00 00 00 10 90 00 00 e5`. When a button is pressed, a bit or bits in the response change for a minimum of one frame before returning to the response. If the button is held down, the response continues to reflect the pressed button until it is released.

For the most part, multiple buttons can be pressed simultaneously. There are two exceptions.

1. If the fan up/down buttons are pressed simultaneously, the panel will send the response for the fan up button and the fan speed will increase.
2. Temperature up/down can be triggered multiple times per command. For example, if I quickly spin the driver temperature knob clockwise, the panel will respond to the next `0x39` request with <code>40 00 00 00 <strong>12</strong> 90 00 00</code> or <code>40 00 00 00 <strong>13</strong> 90 00 00</code>. The temperature will increase by 2 or 3 degrees, respectively.

| Button              | Data (8 bytes + checksum)                                         | Operation                                                    |
|---------------------|-------------------------------------------------------------------|--------------------------------------------------------------|
| none                | <code>40 00 00 00 10 90 00 00 <em>e5</em></code>                  |                                                              |
| off                 | <code>4<strong>2</strong> 00 00 00 10 90 00 00 <em>e3</em></code> | <code>&#124; 0<strong>2</strong> 00 00 00 00 00 00 00</code> |
| auto                | <code>4<strong>8</strong> 00 00 00 10 90 00 00 <em>dd</em></code> | <code>&#124; 0<strong>8</strong> 00 00 00 00 00 00 00</code> |
| driver temp down    | <code>40 00 00 00 <strong>0f</strong> 90 00 00 <em>e6</em></code> | <code>- 00 00 00 00 0<strong>1</strong> 00 00 00</code>      |
| driver temp up      | <code>40 00 00 00 <strong>11</strong> 90 00 00 <em>e4</em></code> | <code>+ 00 00 00 00 0<strong>1</strong> 00 00 00</code>      |
| passenger temp down | <code>40 00 00 00 10 <strong>8f</strong> 00 00 <em>e6</em></code> | <code>- 00 00 00 00 00 0<strong>1</strong> 00 00</code>      |
| passenger temp up   | <code>40 00 00 00 10 <strong>91</strong> 00 00 <em>e6</em></code> | <code>+ 00 00 00 00 00 0<strong>1</strong> 00 00</code>      |
| sync                | <code>40 00 00 <strong>2</strong>0 10 90 00 00 <em>c5</em></code> | <code>&#124; 00 00 00 <strong>2</strong>0 00 00 00 00</code> |
| a/c                 | <code>40 <strong>8</strong>0 00 00 10 90 00 00 <em>65</em></code> | <code>&#124; 00 <strong>8</strong>0 00 00 00 00 00 00</code> |
| front defrost       | <code>40 00 00 <strong>8</strong>0 10 90 00 00 <em>65</em></code> | <code>&#124; 00 00 00 <strong>8</strong>0 00 00 00 00</code> |
| rear defrost        | <code>40 00 00 <strong>4</strong>0 10 90 00 00 <em>a5</em></code> | <code>&#124; 00 00 00 <strong>4</strong>0 00 00 00 00</code> |
| eco                 | <code>40 <strong>4</strong>0 00 00 10 90 00 00 <em>a5</em></code> | <code>&#124; 00 <strong>4</strong>0 00 00 00 00 00 00</code> |
| fan down            | <code>40 <strong>3d</strong> 00 00 10 90 00 00 <em>a8</em></code> | <code>&#124; 00 <strong>3d</strong> 00 00 00 00 00 00</code> |
| fan up              | <code>40 <strong>3c</strong> 00 00 10 90 00 00 <em>a9</em></code> | <code>&#124; 00 <strong>3c</strong> 00 00 00 00 00 00</code> |
| mode                | <code>40 00 <strong>1c</strong> 00 10 90 00 00 <em>c9</em></code> | <code>&#124; 00 <strong>1c</strong> 00 00 00 00 00 00</code> |
| recycle air         | <code>40 00 00 00 10 90 <strong>c</strong>0 00 <em>25</em></code> | <code>&#124; 00 00 00 00 00 00 <strong>c</strong>0 00</code> |
| s-mode              | <code>40 00 <strong>8</strong>0 00 10 90 00 00 <em>65</em></code> | <code>&#124; 00 <strong>8</strong>0 00 00 00 00 00 00</code> |

#### Everything Else

During my testing, `0x32`, `0xBA`, `0xF5`, and `0x78` messages never changed, so I'm not sure what they do. My car doesn't have the winter package, which includes the heated seats, heated steering wheel, and wiper defroster. Those messages may be related to those features and would be used on cars with those options.

`0x76` is still a mystery, as it is continually requested by the master node and never receives a response. Remember how I mentioned some cars, like the newest generation Venza and Highlander with the 12 inch display, can control the climate system from the touchscreen? They might use CAN, or they might be connected to the climate LIN bus and respond to `0x76` requests. I tried responding to `0x76` requests with the same data as `0x39` and nothing happened. I also tried sending messages with 1 bit changed at a time (`0x0000000000000001`, `0x0000000000000002`, `0x0000000000000004` etc.) and observed no changes.

### Sending LIN Data

Because the TJA1020 transceiver operates as a slave node, it can respond to requests sent by the master. Responses, as with data sent by the master, must end with a [checksum byte](https://onlinedocs.microchip.com/pr/GUID-FA73463A-EB90-493D-A558-196E2EA36553-en-US-1/index.html?GUID-E40384CE-AADE-42A4-B529-F5522A91CAF8). A Python script can be used to listen for requests and transmit responses.

```python
import serial

ser = serial.Serial(
  port='COM9', # the serial port of the UART to USB adapter
  baudrate=19200,
  parity=serial.PARITY_NONE,
  stopbits=serial.STOPBITS_ONE,
  bytesize=serial.EIGHTBITS,
  timeout=None
)

def calculate_checksum(frame_id, bytes_data):
  checksum = frame_id
  for b in bytes_data:
    checksum += b
    if checksum > 255:
      checksum -= 255
  checksum = ~checksum & 0xFF  # invert; & 0xFF is unnecessary in C when using uint8_t
  return checksum.to_bytes(1, 'big', signed=False)

def respond():
  increase_fan_speed = b'\x40\x3c\x00\x00\x10\x90\x00\x00'
  while 1:
    b = ser.read()
    if b.hex() == '55':
      b = ser.read()
      if b.hex() == '39':
        ser.write(increase_fan_speed + calculate_checksum(0x39, increase_fan_speed))
```

Running this script will tell the car that the fan up button is being held down, causing the fan speed to increase. However, the control panel must be unplugged from the car for this to work. If the control panel is plugged in, both nodes will be transmitting responses at the same time, resulting in a collision. The car sees this as invalid data and ignores it.

Operating as a master node is a bit more complex. To start, most LIN chips require another resistor placed between two pins or something of the sort to enable them to transmit in master mode. When I tried sending messages to the panel as a master, nothing happened. I think Python with the USB-UART adapter might just be too slow to send messages fast enough to be recognized by the transceiver as a single message. SoftwareSerial on an Arduino also did not send messages quickly enough. I was finally able to get it to work with an Arduino Mega, which has 3 hardware serial interfaces in addition to its USB serial interface. More detail on this will have to wait for another post.

### Next Steps

My goal is to send messages to turn on the defrosters if they were on before shutting off the car. This is going to require building an interceptor with 2 LIN transceivers, one to act as a slave on the car's network and another to act as a master for the panel. I bought some parts to build a prototype, but turning this into something that reliably works in my car is going to take a while.

![prototype LIN interceptor](/assets/images/2022-11-30-hacking-my-cars-climate-controls-lin-reverse-engineering/prototype_interceptor.jpg)
