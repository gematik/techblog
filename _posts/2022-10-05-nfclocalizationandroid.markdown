---
layout: post
title:  "Localization of the NFC antenna on Android devices"
date:   2022-10-07 13:00:00 +0200
author: Till Jannik Plewe
categories: tech
excerpt: "<br/>In order to fully use the *E-Rezept-App* the users have to authenticate themselves with their digital identity which is stored on their health card (eGK). NFC-Localization-Android is an open source project which was implemented by the author to gather position data of the NFC-antennas of various Android based mobile phones. The goal is to provide users with better guidance during the authentication process by displaying the correct positioning of the health card relative to the mobile phone and its NFC antenna.<br/><br/>"
tags: NFC Android Phone eGK Gesundheitskarte
---

## Introduction

The modernization (and thus also the digitization) of the German healthcare system is an ongoing process. 
One part of this process is the introduction of the [E-Rezept](https://www.gematik.de/anwendungen/e-rezept)", which shall replace the old paper based prescriptions. 
Users may get access to the *E-Rezept* by using the [E-Rezept-App](https://play.google.com/store/apps/details?id=de.gematik.ti.erp.app&hl=de&gl=US) on their Android device. 
In order to fully use the *E-Rezept-App* the users have to authenticate themselves with their digital identity which is stored on their health card (eGK).
This authentication method uses Near Field Communication (NFC) to utilize the certificate stored on the health card using the mobile phone.
[NFC-Localization-Android](https://github.com/gematik/NfcLocalization-Android) is an open source project which was implemented by the author to gather position data of the NFC-antennas of various Android based mobile phones. The goal is to provide users with better guidance during the authentication process by displaying the correct positioning of the health card relative to the mobile phone and its NFC antenna.

<p align="center">
<img src="{{ site.baseurl }}/assets/img/221006-nfclocalization/loginguidance.png" alt="drawing" width="200"/>
</p>
*Fig. 1: Screenshot: Login guidance for the Huawei P40 in the Android E-Rezept App*

## What is NFC-Localization-Android?

NFC-Localization-Android was implemented because users reported that the authentication process was prematurely terminated. Therefore they were not able to log in to the *E-Rezept-App*.
There are many possible reasons why a NFC process can get terminated. The best solution is to guide the users to place their health card exactly in the center of the NFC-antenna of their mobile phone, since the connection between the health card's and the mobile phone's antenna is most stable in this location. To be able to display this information, a database containing the position data of NFC antennas from various mobile phones was required.
Because such a database did not exist, NFC-Localization-Android was implemented to gather the needed data from the websites of various manufacturers semiautomatically.

## What does NFC-Localization-Android do?

NFC-Localization-Android gathers and evaluates NFC antenna position data stored in images retrieved from websites of different manufacturers. 
The position of the NFC antenna of the corresponding mobile phone gets normalized (percentage of the screen) and saved together with the model names in a JSON-file.
The following list and flowchart roughly display the sequence of the evaluation algorithm per manufacturer:

1. Get the data from the website
2. Gather the model names for this mobile phone from the [Google Play Device List](https://storage.googleapis.com/play_public/supported_devices.html)
3. Find the NFC antenna and the outline of the mobile phone in the image
4. Normalize the found position of the NFC antenna
5. Save the position and model names in a JSON file

> Note: The steps 2-4 are repeated for all mobile phone models extracted in step 1 before jumping to step 5

<p align="center">
<img src="{{ site.baseurl }}/assets/img/221006-nfclocalization/ablauf.png" alt="drawing" width="400"/>
</p>
*Fig. 2: Flowchart of the evaluation algorithm*

## How is NFC-Localization-Android structured?
To be flexible and future-proof, we decided to implement a unique NFC localization function for every manufacturer. 
The implementation of some of the steps listed above varies for different manufacturers since the websites and the way the required data is displayed also varies.
Generally used functions are located in the `baseline_functions.py` file.
The JSON file which contains all the gathered position data is found in the output folder.
### Dependencies
The following tools are required:
* [python](https://www.python.org/): Version 3.9.14
* [beautifulsoup4](https://www.crummy.com/software/BeautifulSoup/bs4/doc/): Version 4.11.1
* [easyocr](https://www.jaided.ai/easyocr/): Version 1.4.1
* [opencv 2](https://opencv.org/): Version 4.5.4.60
* [pandas](https://pandas.pydata.org/): Version 1.4.2
* [requests](https://requests.readthedocs.io/en/latest/): Version 2.27.1

## Where to go from here?

*NFC-Localization-Android* is available as open source software on [GitHub](https://github.com/gematik/NfcLocalization-Android). Check the Readme for installation and running guidance. Feedback and additional NFC-antenna positions are much appreciated.

*E-Rezept-App-Android* is also available as open source software on [GitHub](https://github.com/gematik/E-Rezept-App-Android/)

# About the author

Till Jannik Plewe has just finished his Bachelor of Science in software engineering. During his studies he worked on Java and Android projects for one year. At gematik he is mainly implementing UX-Features for the  Android version of the *E-Rezept-App*.
