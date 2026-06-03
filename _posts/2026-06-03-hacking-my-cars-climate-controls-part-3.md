---
title: "Hacking My Car's Climate Controls, Part 3: PCB Design"
author: Jacob Schooley
excerpt: "Designing a custom PCB to make my LIN interceptor more compact and easier to install."
tags:
  - car hacking
  - hardware
  - personal projects
image:
  path: /assets/images/2026-06-03-hacking-my-cars-climate-controls-part-3/board.jpg
  width: 836
  height: 520
---

I'll admit, I do not enjoy writing as much as I enjoy building things. Over the last 3 years since my last post on this project, I've started working full time, graduated from BYU with a degree in Cybersecurity, moved across town, [started a band](https://www.instagram.com/tstreet.band), and built a ton of software and hardware to make my life easier.

But I did say I was going to design a custom board for this project, and I presented it at the BYU Cybersecurity Conference in February 2024. It is still in my car and continues to work great. I didn't use it much this last winter, given the extreme lack of snow, but now it's crazy hot outside and I often leave it in L2 mode which sets the AC to max when the car's started.

The board is a much more compact version of the prototype I built in Part 2. It is designed as a backplane that attaches to an Arduino Mega 2560 Pro Mini, the smallest Arduino I could find at the time that had 2 hardware UARTs. It has a power module, 2 LIN transcievers, and a Molex connector that attaches to a custom wiring harness that plugs into both the car's and panel's connectors. (Please leave a comment if you know of a good source for custom built wiring harness as this harness was a nightmare to build by hand.)

I designed the board using EasyEDA and used JLCPCB to manufacture it. As someone who had never manufactured a PCB before this, it turned out to be easier than I thought to turn a schematic into a PCB into a physical, functional product.

![Board](/assets/images/2026-06-03-hacking-my-cars-climate-controls-part-3/board.jpg)

![Arduino](/assets/images/2026-06-03-hacking-my-cars-climate-controls-part-3/arduino.jpg)

And a size comparison of the board to the prototype. The prototype worked but required removing a few panels to get it installed. The new board is small enough to fit through the little hole behind the climate control panel without having to unscrew other stuff.

![Case](/assets/images/2026-06-03-hacking-my-cars-climate-controls-part-3/case.jpg)

[Board](https://github.com/jbschooley/ToyotaLinInterceptor/tree/main/Board) and [case](https://github.com/jbschooley/ToyotaLinInterceptor/tree/main/Case) files can be found on this project's GitHub repo. I can't guarantee all the parts I used are still available, but you are welcome to import the design files into your PCB design software of choice and modify as needed.

I discovered during this process that I am not a fan of hardware design and will probably stick to software development as a career. When developing software, I can compile and run my current code in seconds to see if it works. With hardware, I have to spend money ($50 for this project per iteration, and this was before tariffs), then wait 3 weeks for the board to be built and shipped, then test it, and if it doesn't work, I have to throw it out, spend more money, and wait another 3 weeks. Instant gratification really is my weakness. But I am glad I went through this process and figured it out. Since then, I have built more hardware for other projects, because sometimes it's the only way to get the functionality you need.
