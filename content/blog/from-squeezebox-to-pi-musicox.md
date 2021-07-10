---
slug: /blog/from-squeezebox-to-pi-musicbox
title: "From Squeezebox to Pi MusicBox"
description: The journey from using a Logitech Squeezebox to using a Rasberry Pi for music streaming
banner: Music_Streaming_Services.jpg
date: 2014-07-28
---

For a number of years I have been using Logitech Squeezebox devices to stream music to the various rooms of my house, in particular to a set of outdoor speakers via a Logitech Squeezebox Duet.  

Originally I had issues with my old Billion wireless G router/ADSL modem, issues like setting up a playlist with my favourite songs, sitting down to relax only to be up in five minutes having to reset the router and the Squeezebox as the streaming had stopped. Turns out there was an incompatibility with the Squeezebox and router. Solution...a new router. This was a great setup for a number of years, streaming digital formats like mp3 and ogg, though there were still occasional dropouts - usually when I was attempting to demonstrate my music system to friends and family.

<!-- More -->

# Bye Bye Squeezebox

<img src="squeezebox-1.jpg" class="center-block img-responsive padded" >

So a few years went by and my setup was working well. I even bought a Squeezebox Boom and Squeezebox Radio to play music in more rooms of the house. All was well, with regular firmware updates seemingly making things more stable. The plugin model meant there were a range of interesting extensions available, however, in September 2012 Logitech announced they were discontinuing the Squeezebox range in an open letter to squeezebox fans.

http://blog.logitech.com/2012/09/28/an-open-letter-to-squeezebox-fans/

Great. All of the effort put into converting CDs to mp3 and wrangling with wireless networks seemed wasted.

# Enter Music Streaming Services

<img src="Music_Streaming_Services.jpg" class="center-block img-responsive padded">

Some time went by and things began to evolve within the music industry. Music streaming services started to take off. Despite originally having a fairly small catalogue of music they have recently struck deals with the majority of record companies and now have millions of songs available. After trialling Spotify for a while I decided that this would be my new source of music for these reasons:

* Vast catalogue (there have only been a few albums I missed)
* Well priced for premium ($11.99 AUD per month)
* Lot of devices supported and offline support on mobile devices for listening on the go
* Easy to discover new music via recommendation engine and artist radio features

# Raspberry PI

<img src="RaspberryPi_Logo-1.png" class="center-block img-responsive padded">

Now I was finding all of this great music on Spotify but could not listen to it on my home stereo. I remembered reading about people using Raspberry PIs to build a Squeezebox like device that is controlled via their mobile phones. They are very versatile devices. If you want to find out more about what a Raspberry PI can do check out http://www.raspberrypi.org/. I did some research and found that by getting a Raspberry PI and add-on soundcard I could then install some software to build a new music system to replace the ageing and discontinued Squeezebox. Here's what I got:

* [Raspberry PI B+ with wireless adaptor bundle](http://www.buyraspberrypi.com.au/shop/model-b-wifi-package/)
* [HiFiBerry DAC](https://www.hifiberry.com/dacplus)
* [A case to fit it all in](https://www.hifiberry.com/product/hifiberry-caseplus/)

<img src="case-plus-600x600-441x441-1.jpg" class="center-block img-responsive">

# Pi MusicBox
So now all I needed was to get a linux distribution installed, install some software and start streaming music. My Google searches led me to [https://www.mopidy.com/](Mopidy) - a music server that supports Spotify, Soundcloud, Google Play exposing them as an [http://en.wikipedia.org/wiki/Music_Player_Daemon](MPD). This is great software, however, the installation is complicated and time consuming. There are many linux commands, plugins to pick from and configure and then there is trying to get the soundcard to work. I thought this sounded doable but would require more effort than I anticipated so I put it off for a few days. Then I came across [Pi MusicBox](http://www.woutervanwijk.nl/pimusicbox/). This is a OS (Raspbian) image that comes with Mopidy and a bunch of other useful things pre-installed and with minimal configuration it just works! Well the second release I tried worked; the first had an incompatibility with my wireless adaptor. This brought back bad Squeezebox memories. Basically the WiFi adaptor would not restart when the Pi MusicBox scripts would restart it. This was fixed in version 0.5.2 and described [here](https://github.com/woutervanwijk/PiMusicBox/commit/fd466fd12a4b0f0a5348ff1f0e24b8c6602a2f90). After I installed the new version everything worked great. I plugged in my network credentials and spotify username and password (both in clear text yikes!), fired up the web client and could search, play and create playlists of songs from spotify.

# The Good and the Bad

The things I love about the Pi MusicBox music system:

* Open source and updated frequently. Plus it is all on github so I can look at how the various shell scripts work.
* Support for many streaming services and support for music stored on your network share.
* The web client works well on mobile form factors. I sometimes forget it is an angular application not a native iOS or Android application.
* Keeps on streaming. So far I have tested streaming for 48 hours straight. No interruptions.
* A small case that can be easily hidden behind something. The Squeezebox was also compact little black box. However it had a light on the front whose colour would change based on its status. Blue means disconnected and time for a reset. I hated that blue light.
* The ability to create a playlist of songs from many music streaming services and from your own files.

Things that could be better:

* No support for Spotify Connect. Only available on licensed device at the moment.
* When using the client application it gets confusing interacting the play queue. Still getting used to the application I guess.
* Web client volume control not optimised for mobile form factor. I wish I could control the volume with the volume control on the phone.
