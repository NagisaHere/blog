---
layout: post
title:  "Microcontroller Hackathon 2025"
author: "Ryan Wang"
date:   2025-04-17 00:00:00 +1000
categories: competitions hackathon
tags: microhack hackathon 2025
---
2025 marked the second year of my degree, and my first participation of the Microcontroller Hackathon.

Since it was difficult to train a small model that could fit onto the microcontroller's flash memory, a lot of the Optical Character Recognition (OCR) was done on a computer, and an ESP32-S3-CAM was used to send image messages over. I still decided to go a full No-AI route when doing this, so it was a painful Saturday night of constantly trying to fight the complexities of what functions I should use, as ESP-IDF was in a "migratory" state at this time with many of its wi-fi functions being deprecated.

Overall, this project taught me quite a bit about using the ESP-IDF tool, and also programming the ESP32 in C. It's definitely not as friendly as using Arduino Core in Arduino IDE (in fact Arduino Core is just all wrappers for ESP-IDF), but it does allow finer control over the microcontroller. 

Interested people can go take a look at the [repo](https://github.com/NagisaHere/2025-MCU-HACK).

### Team
Credits to the team!
- Ryan Wang: HTTP Web server and Camera
- [Cielo Nicolosi](https://www.linkedin.com/in/cielo-nicolosi/): Text Recognition, translation and text-to-speech
