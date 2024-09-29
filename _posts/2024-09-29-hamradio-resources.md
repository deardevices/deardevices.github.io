---
layout: post
title: My Ham Radio Resources
image: assets/hamradio.png
---

I got into a new hobby recently: Amateur (a.k.a. Ham) Radio. This article will be a collection of resources I believe are worth sharing in that context: YouTube channels, books, and links to study material. I will also present some things I've experimented with in the last few months.

There is a variety of topics folks get involved with in Ham Radio. I believe many of those align well with what is usually covered on this blog: Embedded Systems, Software, and, maybe, a bit more of Electrical Engineering in future articles.

Being a new operator, I'm trying to take a look at just one item at a time. That way I'm hoping to identify one or more disciplines that would be most fun for me. The following is a *loose collection of aspects I've learned about* since I got my license a few months ago. I'll probably use it as a reference going forward, hoping it might be useful for other people out there as well.

This list will probably keep growing as I'm getting more into the hobby. Therefore, I'll update it occasionally. **Last Update: 2024-09-29**.

Source of title image: [Wikipedia](https://de.m.wikipedia.org/wiki/Datei:International_amateur_radio_symbol.svg).

## Youtube Channels
* [Josh's Ham Radio Crash Course](https://www.youtube.com/@HamRadioCrashCourse): great pleasure to watch, for anything Ham Radio-related.
* [Walt, K4OGO](https://www.youtube.com/@COASTALWAVESWIRES): I like watch Walt's channel for content on antennas and general shortwave experiments.
* [Michael (German)](https://www.youtube.com/@DL2YMR): I watch this channel for local (German) specifics.

## Exam Training (in German, most of it)
* Moltrecht's books: [Klasse E](https://www.amazon.de/Amateurfunk-Lehrgang-Amateurfunkzeugnis-Klasse-allen-Pr%C3%BCfungsfragen/dp/3881803645) / [Klasse A](https://www.amazon.de/Amateurfunk-Lehrgang-Technik-Amateurfunkzeugnis-Erl%C3%A4uterungen-Pr%C3%BCfungsfragen/dp/3881803890) (I've used both).
* [HamRadioTrainer](http://www.hamradiotrainer.de/) software (Windows; supports wine)
* [Anki](https://ankiweb.net/) flashcards: I've created cards for it on my own so I could study on the go.
* [DARC Online-Lehrgang](https://www.darc.de/der-club/referate/ajw/darc-online-lehrgang/) - but note that new training material is already available on [50ohm.de](https://www.50ohm.de/) (there are three classes by now: N, E, and A, N being the latest addition).

## Antennas
Building antennas seems to be one super interesting aspect of Ham Radio. It looks like one can do a lot using simple wire and (optionally) a fiberglass pole. The following antennas I've built so far:
1. Dipole antennas for 10m and 20m, mostly in an Inverted-V style.
1. End-fed dipole for 20m: easier to mount than a regular dipole but needs 49:1 Un-Un (transformer) for impedance conversion as impedance is very high at the end position of the antenna. I've mounted it both horizontally as well as vertically, using a 10m Spiderbeam mast.
1. Groundplane for 70cm.

All of these are portable setups; I've yet to find a good solution for mounting things permanently.

For mobile use, I've had success with an old 420 MHz antenna that gets clamped to my car's window. Together with a handheld transceiver ([RT3S](https://www.retevis.com/rt3s-dual-band-dmr-radio-built-in-gps-us)), I can talk on local repeaters as well as on local frequencies (145.400 MHz and 434.800 MHz).

### Antenna Literature
I like [Rothammel's Antenna Book](https://rothammel.com/Rothammels-Antenna-Book) which is extremely comprehensive and also available in English language. I would occasionally read it as an addition to the various online resources available.

### Antenna Analyzer
For analyzing antenna impedance there's a great little device available: the [nanoVNA](https://nanovna.com/), a small and cheap [Vector Network Analyzer](https://en.wikipedia.org/wiki/Network_analyzer_(electrical)
). Among other things, it measures the Voltage Standing Wave Ratio (VSWR) which is important to know to make sure the antenna is matched to the transceiver. Matching the two is important for maximum power transmission: transceivers have a 50 ohms output impedance and therefore we need to ensure 50 ohms at its output (see [Wikipedia: Impedance Matching](https://en.wikipedia.org/wiki/Impedance_matching)).

### Antenna Tuner
After experimenting with resonant antennas (see [Wikipedia: Resonant Antennas](https://en.wikipedia.org/wiki/Antenna_(radio)#Resonant_antennas)) I've started looking into wire antennas that are not exactly 1/2 wavelength long.

After working with a borrowed manual tuner for a while I ordered an [ATU-100](https://github.com/Dfinitski/N7DDC-ATU-100-mini-and-extended-boards) automatic antenna tuner kit and assembled it.

### Club Antennas
I've signed up with the local DARC club, [OV B04](https://www.b04.info/), and therefore can use the club's antennas. Just recently I got an introduction by one of the club members on what antennas are available and how to make best use of them. The place even offers a solar-powered 12V supply so all we need is to take a transceiver up there and we're good to go for making contacts across Europe and around the world.

Those antennas are at an outdoor location so that mode of working will probably work out well throughout summer. Later this year I might go back to some kind of home setup then.

## Outlook and further ideas
* Transceivers
* Shortwave Propagation
* Learning Morse Code
* Band Plans
* Logging
* Digital Modes
* Electronics
