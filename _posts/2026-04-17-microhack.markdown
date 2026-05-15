---
layout: post
title:  "Microcontroller Hackathon 2026"
author: "Ryan Wang"
date:   2026-04-17 00:00:00 +1000
categories: competitions hackathon
tags: microhack hackathon 2026
---


2026 brings the 5th continuation of the Microcontroller Hackathon. Members of UQ MARS and QUT robotics now have to tackle the theme of games. Participants could either build some sort of game, or anything else that is gamified.

Our team, Simulating VR Recoil (or VR Gunfighters) decided to go with a VR gunstock that has haptic recoil at the back to make FPS VR games more immersive.

Friday got us looking into OpenVR drivers and starting the CAD modeling for the modified gunstock. Since it was a short day, we got started on everything and started prints running overnight whilst the team recharged for tomorrow.


Saturday was the big grind day. The prints overnight had failed, so we had to bring a different printer in (Bambu labs P1S) to now do all the heavy lifting. We got ESP32 communication over WiFi with OpenVR driver working, so we can get the Meta Quest 3 controller states and send a wifi message when trigger on the controller is pressed. The main way we know when the controller is pressed is;
- if event is a button press
    - if button is right trigger
        - if button trigger is from a valid controller
            - if trigger exceeds some threshold, then send the wifi signal.
Whilst OpenVR is deprecated and replaced by OpenXR, OpenXR has much less flexibility when it comes to trying to access controller data. Trying to build a custom VR Controller/Device is very locked down, and wouldn't normally be possible natively on a Quest device if not for the fact that SteamVR allows custom drivers and services to be able to run.

We then tried to figure out what we wanted to do for our recoil system. Since we couldn't find good solenoids for a clean linear motion, we decided to use an FOC motor to at least produce some form of force.

{% include figure.html img_path="/assets/images/2026/MH-3.jpg"%}


Since the trigger with the Meta controllers had some latency, we also decided to look into making our own trigger that is connected to the ESP32 to resolve this, so instead of having trigger -> gun fires in game -> motor fires, we would have trigger -> motor fires -> gun fires in game.
However, writing the basic drivers proved to be very challenging. I initially tried to follow the instructions [here](https://github.com/finallyfunctional/openvr-driver-example), which involved building the following
- Legacy bindings
- TrackedDeviceServerDriver
- TrackedDeviceServerProvider
- ProviderFactory

Of which I honestly have no clue what they do; they register the controller and advertise what inputs they can offer, and that is about it. There were examples of how to setup an ESP32 microcontroller as a movement tracking device with an IMU, but doing it for something as simple as trigger was more difficult than it seemed, since triggers are only registered as a part of valid controllers. I tried setting up trigger as a tracker, which did register succesfully in the game, but certain games seemed to have disallowed having trackers doing any sort of inputs, so that idea was also out of the window. 

Where the output structure would then be installed in the following format in SteamVR -> Drivers
- example
    - bin
        - win64
            - driver_example.dll
        - resources
            - icons
                -   game_controller.svg
            - input
                - controller_profile.json
                - legacy_binding_example.json
    - driver.vrdrivermanifest

Didn't end up working in the end, but it was worth a shot.


Sunday we finalised everything and recorded a demo video of it working at the last minute. Turns out having a high baud rate with jumper wires wasn't really the best idea (mmm resistance). We also had a backup option using a bomb game in case the VR stuff didn't work, so we utilised it for our presentation instead.

(WIP) You can find a quick demo of our project below, repo [here](https://github.com/Alice-Lesmes/MicroHack2026).




### Team
Huge thanks to the team for their great work over the weekend.
- Ryan Wang - VR Drivers, WiFi-Comms and Logistics
- Alice Lesmes - VR Drivers and Presentation
- Aaron Liu - CAD Designer and Motor Driver
- Xinjiang Zhou - Bomb game maker


{% include figure.html img_path="/assets/images/2026/MH-9.jpg"%}
