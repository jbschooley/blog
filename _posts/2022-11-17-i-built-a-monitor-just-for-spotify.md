---
title: I Built a Monitor Just For Spotify
tags:
  - hardware
  - music
  - personal projects
---

I love music. I listen to music constantly, whether I’m working or doing homework or goofing off. I also really like knowing what I’m listening to at all times, as I spend much of my time listening to other people’s playlists and Spotify radio/auto-generated playlists. Bonus points if it’s easy to skip through a bunch of songs quickly for those times when I’m feeling a specific type of song but not quite sure exactly which one.

### What Didn’t Work

I was particularly intrigued by Spotify’s [Car Thing](https://carthing.spotify.com/). I didn’t need one for my car because I use CarPlay and its dashboard works great. For my office, though, it looked like it could be useful. I have 2 monitors and use all the screen space I have available for work. MusicBee and the Pandora app I used years ago allow me to minimize the player into a tiny window that just shows the album art and track name with player controls, but Spotify does not. A big ol' window to show what I’m listening to on Spotify takes up too much screen real estate.

![Spotify car thing]({{site.url}}/assets/images/2022-11-17-i-built-a-monitor-just-for-spotify/carthing.png)

So I bought one. Unfortunately, it turns out, the firmware it ships with will not allow it to connect to a PC. It was designed to connect only to the Spotify app on a phone. I didn’t like that because it would either drain my phone’s battery or would require me to set up a spare phone just for interfacing with that screen. I gave up and sold it.

### Building Something That Does What I Want

There’s plenty of space under my main monitor, so I settled on putting a little 7” monitor under it. That would leave my options open in case I wanted to eventually display more than just Spotify. An Amazon search revealed that every mini monitor that came with a case had the HDMI and USB ports sticking out the side. Yuck.

![monitor with ports on the side]({{site.url}}/assets/images/2022-11-17-i-built-a-monitor-just-for-spotify/amazon-prebuilt-monitor.png)

I actually bought one, thinking it couldn't be that bad. I was wrong. The cables sticking out the side bothered me a bit too much. If I wanted something that looked nice, I was going to have to go with a caseless one and build a case for it. [This is the one I went with.](https://a.co/d/4XXSw9p) Waveshare makes decent displays, and this one is pretty cheap, even though shipping took 3 weeks. It also comes with a separate HDMI board, so I could put the ports wherever I want.

I then designed and 3D-printed a chassis for this monitor. I’m not an artist and there’s a reason I’m studying cybersecurity rather than mechanical engineering. Prior to this, I had never successfully designed anything physical, much less actually made it real, so trying to design something functional was a whole new learning experience and took a lot longer than I was expecting.

This is what I came up with:

![case in fusion 360]({{site.url}}/assets/images/2022-11-17-i-built-a-monitor-just-for-spotify/case-fusion-360.png)

Definitely not your typical monitor chassis. But it didn't take too long and wasn’t too difficult to design, it attaches to a spare monitor arm I had, the ports point out the back, there's a spot for a brightness knob, and for the most part it looks like it belongs on my desk. I may redesign it in the future because the chin is more noticeable than I thought it would be, but this is good enough for now.

![behind monitor]({{site.url}}/assets/images/2022-11-17-i-built-a-monitor-just-for-spotify/behind-monitor.jpg)

It took 4 prints and revisions to come up with a case that would fit the screen and not fall apart while being printed. Every time I paid for a print, waited 12+ hours, and then realized it wouldn't fit right, I was reminded why I chose a career in software.

### The Insides

The HDMI board has a pin for controlling brightness. Brightness is high at 3.3 V and low when grounded. The waveshare wiki instructions say brightness can be controlled via PWM, but I found that a potentiometer works just fine (and DC brightness control is better on the eyes).

Most monitors automatically turn off when they detect that no video is coming through. This one, as I found out after building it, does not. My PC will cut power to the USB ports when it's shutdown, but I usually put it to sleep when I'm not using it, and USB devices are powered during sleep. This posed a problem, as I would wake up at 2 am and see a glowing black square on the other side of my room.

One possible solution to this would be to detect a difference in voltage on one of the HDMI pins between on and sleep. One of the pins did show a noticeable difference when probed with a multimeter, and I would be able to detect that with an Arduino and use that to control power. That may be an option in the future, but it would require SMD tools and I'm not quite comfortable with those yet.

I found an interesting workaround though. I had an old USB audio interface lying around, and found that when the PC was awake, the mic connector had a positive voltage but was grounded during sleep. Connecting that to the EN (enable) pin on the HDMI port resulted in the monitor turning off during sleep.

![insides]({{site.url}}/assets/images/2022-11-17-i-built-a-monitor-just-for-spotify/insides.jpg)

### Software

For the Spotify interface, I simply modified the Spotify web player. They already did all the complicated stuff, and some simple-ish changes make it look pretty good on its own screen.

[Slimjet](https://www.slimjet.com/) is a minimal Chromium fork that's distributed in a portable version. Any Chromium fork works, as long as you aren't using it for anything else.

[Mini Player for Spotify](https://chrome.google.com/webstore/detail/mini-player-for-spotify/mjjeebakniihklfggnacbigighgildlo?hl=en) is a Chrome extension that adds some CSS to the Spotify player to get rid of the unnecessary stuff.

[This tampermonkey script]() further adjusts the CSS (and some JS to fix low resolution album art) to make it look even better.

A desktop shortcut with this target to open it in full screen:
```
slimjet.exe -kiosk https://open.spotify.com
```

And this is what it looks like:

![interface]({{site.url}}/assets/images/2022-11-17-i-built-a-monitor-just-for-spotify/interface.jpg)

Honestly, I think it looks pretty good.

### My Projects Never End

I feel like I can't ever truly "finish" a project. I'm always thinking of ways I could make it better. For example:

I could replace Slimjet with [Thorium](https://thorium.rocks/) because it's open source.

I at least want to get rid of the Chrome extension and move all that CSS to the tampermonkey script.

Tapping the buttons works fine, but dragging the timebar doesn't. I haven't been able to find a reliable way to fix that, but it works fine on the mobile interface. The mobile interface is more difficult and time consuming to modify so that it looks nice on this screen, but I'm considering it if not being able to drag the timebar bothers me enough.

Or, I could show my Home Assistant dashboard instead. That would let me easily control the lights in my room too. However, the Spotify HA plugin currently makes for a poor user experience. The official player uses a websocket for instant updates whenever another device changes the track or playback status, but the HA plugin just polls the API every few seconds, which means if I pause the music or change the song from the desktop app, it will take a few seconds to update on the screen. I'm not sure if this is fixable because Spotify doesn't provide a websocket API to developers.

We'll see if I ever get around to doing any of that.
