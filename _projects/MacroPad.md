---
name: MacroPad
tools: [Electronics, Arduino, MIDI]
image: https://i.imgur.com/FPKdLqj.png
description: Small PCB with some buttons and rotary encoders to do whatever you want
---

![MacroPad](https://i.imgur.com/FPKdLqj.png)

# MacroPad

Yeah, I know. That name **sucks**. But whatever, it's not like I need to sell this thing. :shrug:

First of all, a bit of context. I set up [Voicemeeter](https://vb-audio.com/Voicemeeter/banana.htm) on my PC so I can control different audio streams separately and so I could improve the quality of my microphone that is connected to an external audio interface. Having to tab out of what I was doing to lower the Spotify volume when someone was talking on Discord became so annoying that I started thinking of something else.

First, I tried using an app on my old iPad that could send MIDI messages over the network. It worked, but I still had to unlock the iPad to use it, and it was also a mess of 3 more apps running on my pc in the background (at startup!).

Then I started considering macros on my keyboard, but having to press 3 buttons while you are doing something else isn't a very good idea either. Also, it lacks the granular control you can have with a physical/virtual slider.

So, after a bit I started thinking of a hardware solution. An Arduino, some encoders and some buttons. Perfect! But I wanted something a bit more polished and that I could show instead of needing to hide a bunch of cables behind a 3D printed enclosure.

Time to learn some PCB design I guess. I did my research and I settled on the open source program [KiCad](https://www.kicad.org/) because there were a lot of tutorials and **it's free**.<br/>
It has everything, from the schematic editor to the PCB editor, it doesn't use proprietary file formats and there is no stupid subscription. Maybe it doesn't have some fancy features, but I don't need them (for now).

## The board

The board layout is pretty simple.

![Board](https://i.imgur.com/0NBBxmx.png)
- 1 Arduino Pro Micro (you know, those chinese clones that cost 3€, not some fancy stuff)
- 4 rotary encoders (with a button)
- 4 mechanical keyboard switches ([Wow!](https://www.youtube.com/watch?v=KlLMlJ2tDkg))
- 4 **RGB LEDs**
- a connector to add other LEDs (I used it for an underglow effect)

The encoders are routed in such a way that every one of them is connected to an interrupt-enabled pin. This improves performance, as the Arduino doesn't need to poll the state as often.

As it was my first time designing a PCB, I used an autorouter instead of doing everything on the PCB by hand. It was a pretty simple process with [FreeRouting](https://freerouting.org/), but I would love to have it automated so I don't have to export a file, load into FreeRouting, route, export and import again in KiCad.

Oh and for me the KiCad keyboard shortcut are absolutely not intuitive, so I wasted a lot of time on simple things.
I guess it's something you learn with time.

## PCB Manufacturing

So, I took a copper plate and some acid and then... No, just kidding.

PCB manufacturing is now **really** cheap.<br/>
I used JLCPCB because it had good reviews (and I had a coupon code :P): I paid something like 10€ for 5 PCBs, including shipping, and it took about 3 weeks to arrive to my doorstep, which isn't bad at all. I *obviously* chose the black PCB, matte black everything!

{% capture carousel_images %}
https://i.imgur.com/SDB1ueZ.jpg
https://i.imgur.com/Ts2mykz.jpg
https://i.imgur.com/Lie9DKI.jpg
{% endcapture %}
{% include elements/carousel.html %}

## The code

The awesome [Control Surface](https://github.com/tttapa/Control-Surface) library makes writing MIDI programs on Arduino a breeze.
The only thing I need to add is a way for the LEDs to turn off automatically after a set period of time, right now they stay on after I turn off my pc so I need to disconnect and reconnect the USB cable to power cycle the circuit. Not a big problem but it can be improved.

```arduino
#include <Encoder.h>
#include <FastLED.h>

#include <Control_Surface.h>

// Define LEDs 
Array<CRGB, 4> leds = {};
constexpr uint8_t ledpin = 4;

//External LED strip
Array<CRGB, 4> extleds = {};
constexpr uint8_t extledpin = 5;

// Instantiate a MIDI over USB interface.
USBMIDI_Interface midi;

using namespace MIDI_Notes;

// Rotary encoders
CCRotaryEncoder encoders[] = {
  { {9, 1}, MCU::V_POT_1, 5},
  { {8, 0}, MCU::V_POT_2, 5},
  { {7, 2}, MCU::V_POT_3, 5},
  { {6, 3}, MCU::V_POT_4, 5},
};

// Encoder buttons
CCButton encoderButtons[] = {
  { A3, {MIDI_CC::General_Purpose_Controller_1, CHANNEL_5}},
  { A2, {MIDI_CC::General_Purpose_Controller_1, CHANNEL_6}},
  { A1, {MIDI_CC::General_Purpose_Controller_1, CHANNEL_7}},
  { A0, {MIDI_CC::General_Purpose_Controller_1, CHANNEL_8}},
};

// Mechanical switches
CCButton switches[] = {
  { 10, {MIDI_CC::General_Purpose_Controller_1, CHANNEL_9}},
  { 16, {MIDI_CC::General_Purpose_Controller_1, CHANNEL_10}},
  { 14, {MIDI_CC::General_Purpose_Controller_1, CHANNEL_11}},
  { 15, {MIDI_CC::General_Purpose_Controller_1, CHANNEL_12}},
};

NoteRangeFastLED<leds.length> midiled = {leds, note(C, 4)};
NoteRangeFastLED<extleds.length> midiextled = {extleds, note(E, 4)};


void setup() {
  // Setup LEDs
  FastLED.addLeds<WS2812B, ledpin, GRB>(leds.data, leds.length);
  midiled.setBrightness(128);
  FastLED.addLeds<WS2812B, extledpin, GRB>(extleds.data, extleds.length);
  midiextled.setBrightness(128);
  
  RelativeCCSender::setMode(relativeCCmode::BINARY_OFFSET);
  Control_Surface.begin(); // Initialize Control Surface
}

void loop() {
  Control_Surface.loop(); // Update the Control Surface
  FastLED.show();         // Update the LEDs
}
```