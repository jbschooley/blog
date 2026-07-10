---
title: "OpenStageRF: Wireless MIDI that Doesn't Suck"
author: Jacob Schooley
excerpt: "You can't buy a wireless MIDI solution that doesn't use 2.4GHz... but you can build one"
tags:
  - hardware
  - wireless
  - music
  - personal projects
image:
  path: /assets/images/2026-06-03-hacking-my-cars-climate-controls-part-3/board.jpg
  width: 836
  height: 520
---

A year and a half ago some friends and I decided to [start a band](https://www.youtube.com/watch?v=dYY5banG1ew). That entails playing live shows, and entertaining a crowd as a band requires that you play good music and look cool doing it. For our guitarist, that means jumping into the crowd during solos. I play keys, so I got myself a keytar so I could join in the fun.

Wireless instrument audio is incredibly common and easy to find (albeit expensive for the good stuff) so our guitarist picked up a Sennheiser kit. I found that no keytar on the market can produce all the sounds I need, so the best solution for me was to use my Ax-Edge as a MIDI controller for my MODX M7. Of course, I have to go wireless too, so I started looking for solutions and found a few, notably CME WIDI, midiBeam, and MIDIJet. I bought a pair of CME WIDI devices and set them up, only to realize it wasn't going to be a reliable system for gigs.

See, all three of those systems use 2.4 GHz. 2.4 GHz is an attractive frequency because it's license-free worldwide, so it's incredibly common and easy to develop with. The problemw with that is everything uses it, WiFi, Bluetooth, microwaves, IoT devices, and all those devices sharing the same frequency range and all trying to get the most out of it means there's a lot of interference. In the basement we practice in, that isn't a big deal because the only interfering devices are a single WiFi network and 5 phones with Bluetooth. A 2000-cap venue with at least 2000 phones and who knows how many smartwatches and other devices constantly pinging each other is a whole different animal and poses a serious problem when the data you're trying to put on the air is extremely latency-sensitive and absolutely must make it to its destination or the entire audience will notice.

Wireless audio has had this figured out for years. Good wireless mics and instrument systems (in the US, I haven't looked into wireless standards in other countries) run on subsets of 470-608 MHz, which are reserved for over-the-air TV and can be used for wireless audio as long as TV stations are avoided. If you're running a wireless system, you arrive at the venue, scan for channels that are not occupied by TV traffic, coordinate with the venue and other bands, and you can be virtually 100% sure that nothing will interfere with your signal during the show.

Beats me why the same doesn't apply to MIDI too. Maybe it's too niche to sell, or too hard to get FCC approval to sell a MIDI system outside of 2.4 GHz. But I was tired of having to stay within ten feet of my MODX or risk dropping notes so I decided to build something better.
