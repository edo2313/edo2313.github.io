---
title: Touch sensing is HARD
tags: [Electronics, ESP8266]
style: border
color: warning
description: You think that touch sensing is simple because you saw a video of a 13-year old turning on an LED with an arduino connected to a wire and some aluminium foil? Think again.
---

Some time ago I printed [this lamp](https://www.thingiverse.com/thing:4129249) and put it on my bedside table to give a nice touch to my room. I can control it with [Home Assistant](https://www.home-assistant.io/), but sometimes I switch off my phone before turning it off and so I'm left with 2 possibilities:

- Turning on my phone again only to switch it off

or

- get out of bed, unlock the iPad on my desk, turn it off, return to bed

Who wants to do that? Wouldn't a good old physical button be enough?
The answer is yes, but come on, we live in 2021. **Touch buttons** are much more modern!

## The plan

I knew a bit about touch sensing, in theory you only need a wire, a resistor and 2 free pins on your board. Simple enough, right? So I started soldering, luckily the center of the lamp is hollow so I could run a wire upwards and make the top cover touch sensitive. Cool!

I then started coding the simplest sketch possible using the `CapacitiveSensor` Arduino library so that I could see the readings via serial. It looked fine, but obviously it showed lower values when putting the plastic cover between my hand and the wire. No big deal, just increase the threshold. Everything works, nice. Time to do something more than raw numbers on a USB interface.

## All of this just for a RGB light???

On the lamp's ESP8266 I had the awesome [WLED](https://github.com/Aircoookie/WLED) and it supports *usermods* which are basically plugins. After a bit of copypasting from other usermods and some help from a friend that just finished a university course on C++ programming (he also gave the title to this section), I created something usable.

It reports an *ON/OFF* value to MQTT so that Home Assistant can react based on it and turn off all the room lights when I touch it. Oh, it also works with the automatic discovery feature! Cool right? Sure, **if it works**.

## The problem

I flash everything to the board and when trying it I see the value flickering like crazy. Some sort of interference it seems. Basically if the threshold is too low it picks up the change in current from the LEDs turning on and then it goes on like that. The fact that the algorithm calibrates itself when running doesn't help, as it only lowers the value.

After experimenting with thresholds, disabling the autocalibration feature, comparing the current state with the previous one and a lot more I decided to keep it like that. 

## The conclusion
