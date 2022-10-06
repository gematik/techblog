---
layout: post
title:  "NFC-Localization-Android"
date:   2022-10-05 11:28:16 +0200
author: Till Jannik Plewe
categories: tech
tags: NfcLocalization-Android
---

## Introduction

The modernization (and thus also the digitization) of the German healthcare system is an ongoing process. 
One part of this modernization process is the introduction of the "[E-Rezept](https://www.gematik.de/anwendungen/e-rezept)", which shall replace the old paper based prescriptions. 
Users may get access to the "E-Rezept" by using the "[E-Rezept-App](https://play.google.com/store/apps/details?id=de.gematik.ti.erp.app&hl=de&gl=US)". 
In order to fully use the "E-Rezept-App" the users have to authenticate themselves with their digital identity which is stored on their health card.
This authentication method uses Near Field Communication (NFC) to read the needed data from the health card using the mobile phone.
[NFC-Localization-Android](https://github.com/gematik/NfcLocalization-Android) is an open source project which was implemented by the author to gather position data of NFC-antennas from various android based mobile phones to provide users with better login guidance (display the correct positioning of the health card relative to the mobile phone).

<p align="center">
<img src="{{ site.baseurl }}/assets/img/221006-nfclocalization/loginguidance.png" alt="drawing" width="200"/>
</p>
*Fig. 1: Screenshot: login guidance for the Huawei p40 in the "E-Rezept-App"*

## What is NFC-Localization-Android?

NFC-Localization-Android was implemented since users reported that the authentication process was prematurely terminated, and therefore they were not able to log in to the "E-Rezept-App".
There are many possible reasons why a NFC-process can get terminated. The best solution is to guide the users to place their health card exactly in the center of the NFC-antenna of their mobile phone,
since the connection between the health card and the mobile phone is the strongest at this location. To be able to display this information a database was needed, which contains the positions of NFC-antennas from various mobile phones.
Because such a database did not exist NFC-Localization-Android was implemented to gather the needed data from the websites of different manufacturers semiautomatically.

## What does NFC-Localization-Android do?

NFC-Localization-Android gathers and evaluates NFC-position data stored in images from websites of different manufacturers. 
The position of the NFC-antenna of the corresponding mobile phone gets normalized (percentage of the screen) and saved together with the model names in a JSON-file.
The following list and flowchart roughly display the sequence of the evaluation algorithm per manufacturer:

1. get the data from the website
2. gather the model names for this mobile phone from the [Google Play Device List](https://storage.googleapis.com/play_public/supported_devices.html)
3. find the NFC-antenna & the outline of the mobile phone in the image
4. normalize the found position of the NFC-antenna
5. save the position and model names in a JSON-file

> Note: The steps 2-4 are repeated for all in step 1 extracted mobile phones before jumping to step 5

<p align="center">
<img src="{{ site.baseurl }}/assets/img/221006-nfclocalization/ablauf.png" alt="drawing" width="400"/>
</p>
*Fig. 2: flowchart of the evaluation algorithm*

## How is NFC-Localization-Android structured?
There is a unique NFC-localization function for every manufacturer. 
The implementation of some steps listed above varies since the websites and the way the needed data is displayed also varies.
Generally used functions are located in the baseline_functions.py file.
The JSON-file which contains all the gathered position data is found in the output folder.
### Dependencies
The following tools are required:
* [python](https://www.python.org/): Version 3.9.14
* [beautifulsoup4](https://www.crummy.com/software/BeautifulSoup/bs4/doc/): Version 4.11.1
* [easyocr](https://www.jaided.ai/easyocr/): Version 1.4.1
* [opencv 2](https://opencv.org/): Version 4.5.4.60
* [pandas](https://pandas.pydata.org/): Version 1.4.2
* [requests](https://requests.readthedocs.io/en/latest/): Version 2.27.1

## Where to go from here?

NFC-Localization-Android is available as open source software on [GitHub](https://github.com/gematik/NfcLocalization-Android). Check the Readme for installation and running guidance. Feedback and additional NFC-antenna positions are much appreciated.

# About the author

Till Jannik Plewe has just finished his Bachelor of Science in software engineering. During his studies he worked on java and android projects for one year. At gematik he is mainly implementing UX-Features for the "E-Rezept-App"-Android version.





