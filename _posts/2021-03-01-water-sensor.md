---
layout: single
title:  "A cheap water level monitoring system with ESP32 and Home Assistant"
date:   2021-03-01 21:51:16 +0100
categories: ESP32 IOT 3D HomeAssistant
---

How to monitor your rain water tank level for less than 30€?

![Overkill setup for the task !](/assets/images/water_sensor/water_sensor_setup.png)

## Requirements
You need a little bit of hardware:
* An ESP32 NodeMCU boar (I used the one from [AZ Delivery](https://www.az-delivery.de/fr/products/esp32-developmentboard))
* A waterproof ulatrasonic sentor (Used one from [Amazon](https://www.amazon.fr/gp/product/B07YDG53MC/ref=ppx_yo_dt_b_asin_title_o01_s00?ie=UTF8&psc=1))
* A Micro USB cable
* A USB power supply (there are other options [here](https://techexplorations.com/guides/esp32/begin/power/))

I won't cover home assistant installation in this post.

## Introduction
It is very difficult to access my rain water tank. As I already had a home automation setup based on Loxone and Home Assistant, I wanted to integrate a level sensor. However, the official Loxone of is pretty expensive. Therefore, I decided to go for something more DYI.

## ESPHome
First, you install ESPHome on your computer. ESPhome is a framework which produces ESP32 firmware based on yaml configuration files. The framework supports multiple devices and features. After flashing, the ESPHome devices can be integrated with [Home Assistant](https://www.home-assistant.io/).

Install ESPHome following [these instructions](https://esphome.io/guides/getting_started_command_line.html).
In a nutshell, open a terminal and type:
{% highlight bash %}
pip3 install esphome
esphome water.yaml wizard
{% endhighlight %}

Answer the questions like this:
{% highlight bash %}
(name [water_level]): water_sensor
(ESP32/ESP8266): ESP32
(board): nodemcu-32s
(ssid): <your_ssid>
(PSK): <your_wpa_key>
(password): <choose_a_pasword>
{% endhighlight %}
I  highly recommend using python virtual environment and [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) (for convenience).

Connect the ESP32 to the laptop using a USB cable and type:
{% highlight bash %}
esphome water.yaml run
{% endhighlight %}
For the first run, select the option to flash over serial.
If you entered correct information, the ESP32 should connect to your Wi-Fi network. Write down the IP address.
If you have a DHCP server, it's a good idea to create a static lease for your ESP32.

For subsequent runs, you can flash the device over the air using:
{% highlight bash %}
esphome water.yaml run --upload-port XX.XX.XX.XX
{% endhighlight %}

## The sensor
You can now connect the sensor to the board. The one from Amazon came with the necessary wires. I didn't do any soldering yet. There are just 4 wires to connect and it works. SO I'll keep it like that.

The sensor is pretty cheap and came with no documentation.
After a few attempts, I realized that the TX pin should be connected to ...echo and RX to Trigger.
It does not make sense but it works. Connect RX go GPIO26 and TX to GPIO25. If it does not work, swap them.

Edit the water.yaml file and make it look like this one:
{% highlight yaml %}
esphome:
  name: water_sensor
  platform: ESP32
  board: nodemcu-32s

wifi:
  ssid: <your_ssid>
  password: <your_wpa2_key>

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Water Sensor Fallback Hotspot"
    password: <auto_generated>

captive_portal:

# Enable logging
logger:

# Enable Home Assistant API
api:
  password: <complex_password>

ota:
  password: <complex_password>

sensor:
  - platform: ultrasonic
    trigger_pin: GPIO26
    echo_pin: GPIO25
    name: "Water Level"
    icon: "mdi:water"
    update_interval: 300s
    timeout: 3m
    filters:
    - filter_out: nan
    - lambda: return 1.64-x; # 1.64 is the distance between the sensor and the bottom of the tank
{% endhighlight %}

You can start by an update_interval of 1s for troubleshooting.
Flash the device (over the air or serial).
It is very important to place the sensor perpendicularly to the surface you are pointing.
The sensor sends a sounds and measures the time for the echo. If it's not perpendicular, you won't get an echo. If it's not "perpendicular enough", the sensor will often timeout and the measurements won't be reliable.

Point the sensor to a wall (perpendicular) and run the ESPhome. Got a value? Good, let's continue.

It is possible to limit the impact of an improper angle. When the measure does not succeed, the ESP32 sends "nan". Home Assistant will interpret it as "Unknown". Unfortunately, there is no easy way to display the last known value in Home Assistant. It will just display an ugly "Unknown".  I added a filter to ensure that the ESPhome returns a measurement only if it's not "nan". It is this part of the config file:
{% highlight yaml %}
    - filter_out: nan
{% endhighlight %}

## The 3D mount

Placing the sensor very precisely in the tank is not easy. Therefore I design 3D model of a sensor holder.
It works pretty well and it really improves the accuracy.
![The 3D file](/assets/images/water_sensor/water_sensor_mount.png)

You can download the models here:
* [STL file](/assets/3d_printing/Water sensor holder v5.stl)
* [Prusa printer files](/assets/3d_printing/Water sensor holder v5.3mf)
I suggest using PETG for printing. PLA does not support very well humid environment.

Here is a picture of the device placed:
![Device placed](/assets/images/water_sensor/water_sensor_install.png)

## Home Assistant
Finally, go to your Home Assistant web interface and follow [this guide](https://www.home-assistant.io/integrations/esphome/) to add the sensor to Home Assistant.

You should now have a water monitoring system, fully integrated with Home Assistant.
![Device placed](/assets/images/water_sensor/water_sensor_home_assistant.png)

As a next step, I start working on a battery powered version with a Lora ESP32 for a water tank placed where there is no Wi-Fi or power. TBC...
