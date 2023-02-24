---
layout: post
title: "Water Tank Height Sensor"
date: 2021-08-21
description: 
author: 
image: /assets/images/watertank-sensorcovered.png
tags: 
    - programming
    - electronics
---
My grandparents live out in the woods, and are grappling with the [largely anthropogenic](linktopaper) drought in CA. They've been rationing their water, but haven't had a good sense of how much water is available at a given time besides knocking on the side of the well-water tank and exclaiming "we've got water!" or the dour alternative. I looked into buying pre-built water tank sensors, but they were prohibitively expensive, so I figured we could make our own.

We wanted a continuous stream of water tank data, easily accessible from our phones. To do this, we needed to build sensors for detecting the water levels in the various tanks, and a smarthome server to receive all of the sensor outputs, aggregate them, and make them accessible on our phones/computers.

We'll start with the sensor, then setup the homeassistant server and integrate the sensor

## Table of Contents
1. [The Sensor](#the-sensor)
2. [Smarthome Server](#smarthome-server)



An exhaustive list of all materials used in this project (total = $77, $61 of which is the raspberry pi home server)
* [ESP-12E Development Board ($3)](https://www.aliexpress.com/item/32782666206.html?spm=a2g0o.productlist.0.0.70eb15bfyGLMbN&algo_pvid=3678810f-1bbc-47ad-ad60-4bfd83280a1b&algo_exp_id=3678810f-1bbc-47ad-ad60-4bfd83280a1b-0&pdp_ext_f=%7B%22sku_id%22%3A%2263087734925%22%7D)
* [DuPont Wires ($3 for a ton)](https://www.aliexpress.com/item/1005002046765371.html?spm=a2g0o.productlist.0.0.6bce128bHvV1Xy&algo_pvid=45b12b05-9dfa-4b86-974a-0db7bec9f996&algo_exp_id=45b12b05-9dfa-4b86-974a-0db7bec9f996-2&pdp_ext_f=%7B%22sku_id%22%3A%2212000018543604511%22%7D) (socket-socket, otherwise known as female-female)
* [Ultrasonic HC-SR04 Distance Sensor ($1)](https://www.aliexpress.com/item/2041955317.html?spm=a2g0o.productlist.0.0.236168e5u3hRd9&algo_pvid=10dc0f68-7836-48e1-8b24-d182653bf385&algo_exp_id=10dc0f68-7836-48e1-8b24-d182653bf385-0&pdp_ext_f=%7B%22sku_id%22%3A%2212000018273195310%22%7D)
* [microUSB charger ($1)](https://www.monoprice.com/product?p_id=4867)
* [Raspberry Pi 4, 2 GB memory ($45)](https://www.pishop.us/product/raspberry-pi-4-model-b-2gb/)
* [microSD ($8)](https://www.banggood.com/Mini-128GB-CLASS10-Memory-TF-Card-Flash-Card-Smart-Card-16GB-32GB-64GB-for-Mobile-Phone-Laptop-p-1727878.html?cur_warehouse=CN&ID=3150&rmmds=search)
* [microSD --> SD/USB converter ($8)](https://www.banggood.com/USB-2_0-Multi-Card-Reader-TF-Card-OTG-Reader-USB2_0-Micro-USB-Interface-480MB-or-S-for-Smartphone-p-1553700.html?cur_warehouse=CN&rmmds=search)
* [USB-C charger ($8)](https://www.monoprice.com/product?p_id=31201)


## The Sensor
Materials:
* ESP-12E Development Board
* DuPont Wires (socket-socket, otherwise known as female-female)
* Ultrasonic HC-SR04 Distance Sensor
* microUSB charger

These are all the parts you'll need for the sensor:
<p align="center"><img src="/assets/images/watertank-sensorparts.png" width="50%"/></p>

Here's the wiring diagram:
<p align="center"><img src="/assets/images/watertank-wirediagram.png" width="30%"/></p>

Here's what it looks like wired up:
<p align="center"><img src="/assets/images/watertank-sensorwired.png" width="55%"/></p>

When I 'water proof' it, it is unexpectedly and impossibly cute (the nodemcu board is detached so I can feed it through the hole I put in the tank):
<p align="center"><img src="/assets/images/watertank-sensorcovered.png" width="40%"/></p>

You can see the white ballon mounted next to the tank input pipe here, carefully such that the sensor is perpindicular to the water.
<p align="center"><img src="/assets/images/watertank-mounted.png" width="50%"/></p>

Here's the hole and the wires out the back.
<p align="center"><img src="/assets/images/watertank-mountwires.png" width="30%"/></p>

Here's our little conjuction of power cable and distance sensor wires and the nodemcu board, mounted to the side of the tank
<p align="center"><img src="/assets/images/watertank-assembly.png" width="30%"/></p>

Now we'll setup the software for the sensor

I'm assuming you have a unix system (linux or mac)
open Terminal or some other terminal emulator and run the following

I use pip as a python package manager. It should be installed by default with python3 

download the esphome package
```bash
pip install esphome
```
Now we're going to make a .yaml configuration file that will be used to flash the software onto the sensor.

We're gonna assume we name the config file 'water.yaml'
```bash
esphome water.yaml wizard
```
Answer the questions like this if you got the exact sensor hardware I did:
```bash
(name [water_level]): <choose_a_name_for_water_sensor>
(ESP32/ESP8266): ESP8266
(board): nodemcu
(ssid): <your_wifi_network_name(ssid)>
(PSK): <your_wifi_password_(wpa_key)>
(password): <choose_a_password_for_flashing_sensor_if_you_want>
```

Once this is done, ensure that your sensor is plugged in via USB and run 

```bash
esphome water.yaml run
```

It compiled, but I received the following error
```bash
INFO Successfully compiled program.
INFO Resolving IP address of barntank_sensor.local
ERROR Error resolving IP address of barntank_sensor.local. Is it connected to WiFi?
ERROR (If this error persists, please set a static IP address: https://esphome.io/components/wifi.html#manual-ips)
ERROR Error resolving IP address: Error resolving address with mDNS: Did not respond. Maybe the device is offline., [Errno 8] nodename nor servname provided, or not known
```
After about an hour of trouble shooting I realized it was because the microUSB cable I was using does not transfer data, it only charges devices, which is the case with [some cheap cables](https://www.quora.com/Are-some-USB-cables-for-power-only-no-data-Is-there-a-quick-way-to-tell-by-looking?share=1). After replacing the cable with another microUSB I had lying around, it worked.

It's gonna compile and take a few minutes. When prompted, select the option to flash over 'serial' i.e. USB.

If you change the configuration and flash it again you can use the following command to flash it over wifi
```bash
esphome water.yaml run --upload-port XX.XX.XX.XX
```
where xx.xx.xx.xx is the IP address of the sensor (the IP of the sensor is reported after you flash it over USB the first time. Might be worth writing it down.)

This is what my .yaml looks like
```yaml
esphome:
  name: barntank_sensor
  platform: ESP8266
  board: nodemcu

# Enable logging
logger:

# Enable Home Assistant API
api:
  password: <choose_a_password_for_HomeAssistant>

ota:
  password: <choose_a_password_for_HomeAssistant>

wifi:
  ssid: <your_wifi_network_name(ssid)>
  password: <your_wifi_password_(wpa_key)>

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Barntank Sensor Fallback Hotspot"
    password: <auto_generated>


captive_portal:

sensor:
  - platform: ultrasonic
    trigger_pin: D6
    echo_pin: D5
    name: "Water Level"
    icon: "mdi:water"
    update_interval: 20s
    timeout: 3m
    filters:
    - filter_out: nan
    - lambda: return x * (-444.693) + 5000; # See Below for the Math I used
    - sliding_window_moving_average: # This takes the mean of 3 measurements and sends the 3rd measurement to home assistant. 
        window_size: 3
        send_every: 3
    unit_of_measurement: "gallons"

```

OK so the math to calculate the gallons (or whatever unit of volume) is going to be particular to your use.

Our tank was a cylinder, so the formula for volume was 

$V = pi * r^2 * h$

We measured:

circumference = 8.33m

diameter = 2.65

height (when full) = 3.43m

$r = diameter / (2 * pi) $

So 

$V = pi * (diameter / (2 * pi))^2 * h$

We're looking for the Volume as a function of height, so we reduce the equation to volume and height. 

Because we're putting the sensor above the water in the tank, $Volume_{Current} = Volume_{Max} - Volume_{Sensor\ Distance}$

We know the volume of our tank is $8.93m^3$ gallons, so

$Volume_{Current} = 8.93m^3 - pi * (diameter / (2 * pi))^2 * sensor\ distance$

Your sensor distance is going to be some fixed distance above the water, so when the tank is full figure out what the $sensor\ distance$ is and adjust the constant such that both sides of the equation are equal when the tank is full.

For me this simplifies to the values seen in the above code block.

Flash/update the software on the sensor as often as you change the .yaml configuration.
Now the sensor should be set up

## Smarthome Server
Materials:
* Raspberry Pi 4, 2 GB memory
* microSD
* microSD --> SD/USB converter
* USB-C power cord

This part is pretty easy, but opens up pandora's box. There are so many things you *can* do. But to integrate our sensor it's pretty easy.

So we're gonna install Home Assistant on a raspberry pi 4. You can install Home Assistant software on a bunch of different machines/computers, but raspberry pis are cheap and effective.

You're going to start by taking your microSD card and using the microSD converter and plugging it into your computer.

Once you've got the microSD in, follow [this tutorial](https://www.home-assistant.io/installation/raspberrypi/) for installing Home Assistant. It says to use Balena Etcher but you can also use [Raspberry Pi Imager](https://www.raspberrypi.org/software/), which is what I used.

Once you set up home assistant you should find the sensor under the 'configuration' menu.

<p align="center"><img src="/assets/images/watertank-sensor_configure.png" width="60%"/></p>

Now you can customize how to display the sensor data in your Home Assistant dashboard!

