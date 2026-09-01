---
layout: post
# DRAFT — raw material for a future part. When publishing, set these:
#   part: <next number>
#   permalink: /part-<n>/
#   date: <yyyy-mm-dd>
#   thumbnail: /assets/img/thumb-skyN.png
title: "A sky that keeps the server's time"
description: "The world had a placeholder sky. This time I rebuilt the real Lineage 2 day/night: its own encrypted config tables, the sky textures pulled out of the client, and a clock that runs on the same schedule as the server. Then I put it next to the official client and fixed everything that did not match."
---

The world was standing on real ground with real names on everything, but the sky above it was
a Godot placeholder. A blue gradient and a procedural sun. It was fine, and it was fake. Lineage 2
has a very specific look at each time of day, and I wanted the real one, not my impression of it.

It turned out the whole day/night system is sitting right there in the client, if you can read it.

## The client already knows what every hour looks like

Two files in the client's `system` folder do all the work: `Env.int` and a set of
`TimeEnv0..3.int`. They are encrypted the same way as the other `.int` files, a header followed
by a body XOR'd with `0xAC`. Decoded, they are plain text config.

`Env.int` is the master switch:

```
[EnvSetup]
IsClock=true
StartTime=12
TimeRatio=6
SKYBOX=SkybackgroundColor
HAZERING=HazeRing_Final
CLOUD1=Cloud_Final
Cloud2=StarField_Final01
Cloud3=StarField_Final02
```

And each `TimeEnv` file is a set of tables keyed by the hour. The client picks two keyframes and
blends between them. Here is the sky colour across a day, straight from the file:

```
[SkyBoxColor]
COLOR1=(T=0,R=40,G=49,B=70)     ; night
COLOR7=(T=6,R=81,G=75,B=146)    ; dawn
COLOR12=(T=12,R=66,G=124,B=176) ; midday
COLOR19=(T=18,R=60,G=111,B=157) ; sunset begins
COLOR25=(T=24,R=40,G=49,B=70)   ; back to night
```

There are tables like this for the sun colour and size, the moon colour and size, the clouds, the
horizon haze, and the ambient light on the ground. Nothing here is my taste. It is the artists'
taste from 2004, and I just have to read it correctly.

## A clock that agrees with the server

`TimeRatio=6` means the in-game clock runs six times faster than real time, so a full day takes
four real hours. The interesting part is that the server runs on the exact same number. In L2J,
`GameTimeTaskManager` has `IG_DAYS_PER_DAY = 6` and derives the time from the real clock. So both
sides compute the time of day from the same wall clock with the same ratio, and they land on the
same value on their own, without anyone sending a packet.

I still wired up the server's time packet, `ClientSetTime`, in case a server ever does send it. But
on this one it never does, and it does not need to: my clock and the server's clock stay locked
because they are the same sum. Night mobs, the Dark Elf night skill, and my sky all change at the
same moment.

## The textures were hiding in the wrong package

I assumed the sky art lived in a package called `lineageenv`. It does not, that one is code, the
ambient birds and butterflies. The sky is in `l2_skies.utx`: the cloud texture, the horizon ring,
two starfield layers, the sun, several moons, and a set of lens flares. I pulled them all out with
umodel. Each one is wrapped in a little material graph that says how it moves, a panner that drifts
the clouds, a rotator that turns the stars very slowly. I read those numbers too and used them as
the motion speeds.

With the tables feeding the colours and the real textures feeding the shape, the sky started
looking like the game. Here it is at noon, at dusk, and deep at night:

![Noon, dusk and night skies in the Godot client]({{ "/assets/img/hero-sky.png" | relative_url }})

## Then I put it next to the real thing

A sky that looks good on its own is not the goal. The goal is a sky that matches. So I ran the
official client and mine side by side on the same server, at the same time, and started listing
every difference. There were a few, and each one had a real cause.

**The world went red at midday.** L2 stores its light colours as hue/saturation/brightness, and at
noon the value is a fully saturated red. In the real engine that only nudges the pre-baked lighting,
but as a single Godot sun it turned the whole ground red, the way an event sky would. The fix was to
stop using that table for the light's colour and take the colour from the sun and moon tables
instead, which are the warm daylight and cool moonlight you would expect.

**The sun moved wrong.** It came up on one side, climbed to straight overhead, then slid back the
way it came. My path maths was wobbling at the top. I replaced it with a proper tilted circle, so the
sun rises due east, crosses the southern sky, and sets due west, the way it should, and the way L2
does.

**The moon was small and orange; theirs was big and white.** I was multiplying the white moon texture
by the warm moon colour and shrinking it. Once I let the moon be white and gave it its real size, it
matched. I also gave it the real current phase, a curved terminator that follows the actual lunar
cycle, so tonight's crescent in my client is tonight's crescent in the sky.

**Their ground was bright at night; mine was black.** This was the best one. L2 lights each kind of
surface with its own set of values, and the terrain's night light is as strong as its midday light,
which is why you can always see the floor in Lineage 2 even at midnight. I had been using the dimmer
light meant for characters. Switching the ground to its own table fixed it in one line.

Left is the official client, right is mine, same server, same night:

![Official client and Godot client side by side at night]({{ "/assets/img/sky-compare-night.png" | relative_url }})

## The last gap

One stubborn moment was left, around a quarter to six in the morning. The moon has set, the sun has
not quite risen, and for a few minutes both are on the horizon. A real sun light at that angle skims
flat ground and lights nothing, so the world went dark right before dawn. L2's terrain light does not
care about that angle, so I clamped my light so it never lies completely flat. The floor stays lit
through the handover, and then the sun takes over.

To check all of this without waiting on the four-hour clock, I put a slider in the debug panel that
scrubs the time of day from 0 to 24, with a toggle to hand control back to the server clock.

## Where this leaves me

The sky is the client's own, running on the server's schedule. The colours, the sun and moon, the
clouds and stars, the light on the ground, all of it comes from the files that shipped with the game
in 2004, and it lines up with the official client hour for hour. The sun even has its lens flare back.

It is still floating capsules walking around under that sky, though. The models are next.
