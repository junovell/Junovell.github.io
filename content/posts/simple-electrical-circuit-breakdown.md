---
title: "A Simple Electrical Circuit Breakdown"
description: "The start of my electrical engineering journey"
date: 2026-09-24
author: Junovell
toc: true
math: false
isStarred: true
tags: ["guides", "electrical circuit"]
---

# Introduction

I recently joined a two-year program to become an electronics technician, and things are getting hands-on from the start **(ó﹏ò｡)**. In one of my very first classes, our electrical teacher handed us a practical assignment: *break down and describe a simple switchboard from scratch*. Since I’m documenting my journey into the world of electronics, I figured—why not turn my homework into a quick guide? So here is my breakdown of the task, starting with a look at each component in the exact order that power flows through the system.

# The Circuit

![The circuit diagram for a simple switchboard](simple-electrical-circuit-breakdown/sample_switchboard_circuit+irl_image-fr.png)

To get a clearer picture of what we are dealing with, I recreated my teacher's paper using [QElectroTech](https://qelectrotech.org/) so it's readable and next to it is the image of the switchboard it's representing. I also put the ID of each breaker as it was defined in the diagram's table.

This specific circuit represents a simple classroom switchboard layout. Here is the general layout of what it has:

- Incoming Power Supply: A main 3-phase circuit breaker (the big red breaker at the top) that acts as the primary switch and protection for the whole board.

- Distribution: Power flows down from the main breaker to 11 single-phase breakers, which split the load into smaller individual circuits.

- Standardization: All of this is drawn using standard IEC symbols, which are the global standard for electrical, electronic, and architectural schematics.


## Components

To break this down for my own notes, I’m going to follow the exact path of the electricity starting from the main incoming supply at the top and working my way down to the individual components.

### 0. The Power Source

Before electricity can do anything in the switchboard, it has to enter the system from the main grid, but since this is a classroom's dashboard, then then electricity is coming from the building's main electrical room.

Which is what this triangle down here represents:

![power source symbol](simple-electrical-circuit-breakdown/power-source.png)

"DEPUIS TGBT" is french for "From the Main Low Voltage Switchboard", it means that our switchboard isn't an independent power source; rather it's being fed upstream from the building's main electrical distribution panel.

### 1. The Diagnostic Circuit

Right below the main supply line and on it's left, you'll notice a branch coming off to the side like a little electrical appendix:

![alt text](simple-electrical-circuit-breakdown/fuses-lamps.png)

On the diagram, this is symbolized by a single fuse (a rectangle with a line through it) attached in series with three lamps (circles with an X inside).

Honestly tho? From my beginner's perspective, this part of the diagram is pretty misleading.

Why? Because when you look at the actual physical switchboard, the reality looks a bit different (excuse the poor camera quality):

![alt text](simple-electrical-circuit-breakdown/fuses-irl.png)

Instead of just one fuse and a shared string, the real board actually has three separate fuses, one for each of the three incoming phases. Each phase feeds into its own fuse and then outputs to its own individual indicator lamp.

This creates a practical diagnostic tool: if one phase drops out or experiences a fault, only the corresponding lamp for that specific phase turns off. It gives us an instant visual warning of which line is having issues without needing to test each phase manually.

### 3. The Main 3-Phase Circuit Breaker

Now we can finally talk about the big RED elephant in the closet: the main 3-phase circuit breaker.

![a picture of a 3 phase circuit breaker next to the diagram symbol of a normal circuit breaker](simple-electrical-circuit-breakdown/3-phase-breaker.png)

On the diagram, it's drawn using a standard circuit breaker symbol, but seeing it on paper doesn't quite do justice to what it actually handles.

first what does a **3-phase circuit breaker** even mean? well it's actually very simple, it's just 3 live wires (each carrying power from the grid) tied together into a single breaker, which is the red one we see in the image above.

But how much power can it deliver tho? is a question i had too, at first i had the naive idea of just summing them up together and calling it a day so 230V x 3 = 690V and voila **◝(ᵔᗜᵔ)◜**...

Yeah, well, that's not quite how it works **(ᵕ—ᴗ—)**.

while my naive idea might seem intuitive at first, we need to understand how 3 phases actually work. Instead of stacking up, they deliver power all at the same time but with their phase shifted by 120°, something like this:

![an image of a 3 phase oscilloscope graph](simple-electrical-circuit-breakdown/3-phase-oscilloscope.png)

Each wave represents a different live wire. Because they peak at different times, the maximum voltage difference between any two of them is calculated as: `230V x √3 = ~398V` or 400V (we'll discuss how that rounding up happens in the future).

Looking at the table at the bottom of the diagram, in the "ARRIVE GENERALE" column, we can read these specs for the breaker:

- Brand & Model: Legrand (which is a French industrial group specializing in electrical parts)
- Reference: DX series / dx40
- Rating: 40A (this is the maximum current intensity the breaker can allow before it shuts down to protect the circuit)
- Power Rating: 15 kW (the maximum power limit it can handle)
- Number of Poles: 4P4D (4 poles, meaning it switches and protects all three live phases plus the neutral.)
- Cable Section: 5x10 mm² (meaning a 5-conductor cable with three phases, neutral, and earth with a 10mm² cross-section to handle that 15 kW load safely)

### 4. The Council of the 11 Single-Phase Breakers
