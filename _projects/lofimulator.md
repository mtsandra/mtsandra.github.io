---
layout: page
title: "lofimulator"
description: algorithmic ambient music and soundscape
img: ../assets/img/project-covers/lofimulator_br.gif
importance: 1
category: music tech
date: 2025-09-05
---



<div class="row">
    
    <div class="col-sm mt-3 mt-md-0">
        <p style = "background-color: var(--global-card-bg-color); color: var(--global-text-color); font-size: 14px; padding: 10px; border-radius: 10px;
        font-family: monospace;
        letter-spacing: .08em;
        margin: 0 auto;
        overflow: hidden;
        white-space: nowrap;
        border-right: .17em solid var(--global-theme-color);
        animation: typing 3.5s steps(30, end), blinking-cursor .5s step-end infinite;
        max-width: auto">
        09/05/2025<br>
        Interactive System built with Max and TouchOSC<br>
        Aleksandra Ma and Trenton Wong
        </p>

    </div>
    
</div>





<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <iframe src="https://www.youtube.com/embed/M3qQvuJEjRE?si=tNKUgMlEptPsHc7C"
                allow="autoplay"
                style="width:100%; height:60vh; border:none;"
                allowfullscreen>
        </iframe>
    </div>
</div>

### How It Works
The Lofimulator consists of two parts: ambient music that is generated algorithmically and a soundscape explorer that could be crossfaded. Both parts could be controlled via Max directly or via a mobile device through touchOSC. Users could choose to explore the system one part at a time by lowering the volume of the other, or both at the same time. 

To generate ambient music, the system multiplies 4 types of waves based on the mixture composition specified by users: triangular, rectangular, sinusoidal, and sawtooth waves. Then the system outputs notes from a scale using the waveform mixture with reverb, dampening effects, and EQ processing. Users have the option to choose the key, mode, and octave range they would like the ambient music system to generate notes from, as well as the speed at which the system generates new notes. If they don't choose, Lofimulator will generate notes in a C Major scale covering two octaves with the first note being middle C by default. The ambient music part of the Lofimulator has two modes: automatic waveform mixture composition and manual exploratory waveform composition. If they choose automatic mode, then the system will automatically change the proportion of the 4 waveforms present in the mixture based on user input. Users could specify the speed at which the system changes this mixture. Alternatively, users could select one single waveform mixture proportion based on preference and leave it unchanged. 

The soundscape exploring section includes 4 looped samples: candle burning, beach sounds, crickets singing, and rain sounds. These 4 samples are looped and crossfaded to form an interesting and relaxing soundscape. Users could use the X-Y plane to explore how much of each sample environment sounds they want to hear. The visuals correspond to the environmental sound samples, and the transparency setting of the visuals map to the crossfading of the sounds. This way, users have visual feedback of the auditory scene. In particular, for the rain sounds visual, users could enable water dripping sounds represented in the picture. The water dripping sound sequence is a Markov chain outputting different waterdrop samples controlled by a random metronome. The pitch is then shifted randomly to create a realistic and everchanging sequence of water dripping sounds. 

### How We Implemented It 
Lofimulator is built using Max 8 and touchOSC. The algorithmic ambient music section takes in a JSON file listing all the scale MIDI numbers and transforms them to the key, mode, and octave range users specify. Then the system converts them to frequencies that the 4 waveform oscillators generate sounds at. We added equalizer filters, reverb, and dampening effects to the output signal. Initially, we only had 1 sinusoidal wave generating sound, however, the output sounded a bit dull. Our solution to that is to have 4 different waveforms multiplied together to make the timbre richer. We implemented an X-Y plane to get the 4 ratio each waveform takes up in the final output. Sinusoidal and sawtooth waveform ratios always add up to 1, and rectangular and triangular waveform ratios always add up to 1. In order for the system to automatically change the mixture ratio at a constant speed, we implemented a subpatcher that allows the cursor to oscillate along the X-axis and Y-axis based on the X-speed and Y-speed users specify.

We implemented the soundscape navigator with the node object and scaled the sample amplitude based on how far the cursor is from the node center. This way if the nodes are overlapping on top of each other, the samples will also be crossfaded. One challenge that we ran into during this part was finding an effective way to provide visual feedback to users to inform them of what samples they are listening to. Our solution is to use an image that corresponds to the environment the sound sample represents, and use the alpha/opacity value of the visual to represent the amplitude of the environment sound samples. For the generative waterdrop sounds, we implemented a Markov chain to determine the order in which the 4 different dripping sounds should go in. Then we implemented a random metronome that randomly changes the speed of playing back dripping sounds, and implemented a pitch shifter to change the pitch of those dripping sounds.

In order to add more mobility and flexibility to our system, we implemented mobile device control through touchOSC. We created a custom user interface through the touchOSC Mk1 editor that matches the layout of our Max patcher's presentation mode. This way the system becomes more mobile and does not confine users to their laptops. 

