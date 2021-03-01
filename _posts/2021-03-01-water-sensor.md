---
layout: single
title:  "A cheap water level monitoring with ESP32 and Home Assistant"
date:   2018-12-13 21:51:16 +0100
categories: ESP32 IOT
---

How to monitor your rain water tank level with cheap for less than 30€?

You need a little bit of hardware:
* An ESP32 NodeMCU boar (I used the one from [AZ Delivery](https://www.az-delivery.de/fr/products/esp32-developmentboard))
* A waterproof ulatrasonic sentor (Used one from [Amazon](https://www.amazon.fr/gp/product/B07YDG53MC/ref=ppx_yo_dt_b_asin_title_o01_s00?ie=UTF8&psc=1))
* A Micro USB cable
* A Home Assistant local instance

I won't cover home assistant installation in this psot.

First, you have to install ESPHome on your computer. ESPhome is a framework which produces ESP32 firmware based on yaml configuration files. The framework supports multiple devices and features and the after flashing, the ESP32 are compatible with [Home Assistant](https://www.home-assistant.io/).

Install ESPHome following [these instructions](https://esphome.io/guides/getting_started_command_line.html).
I also highly recommend using python virtual environment and [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) (for convenience).
In a nutshell, open a terminal and type:
{% highlight bash %}
pip3 install esphome
esphome water.yaml wizard
{% endhighlight %}
You will have to answer a few questions:
{% highlight bash %}
(name [water_level]): water_sensor
(ESP32/ESP8266): ESP32
(board): nodemcu-32s
(ssid): <your_ssid>
(PSK): <your_wpa_key>
(password): <choose_a_pasword>
{% endhighlight %}

If you entered correct information, your ESP32 should connect to your Wi-Fi network.
Connect it to the laptop using a USB cable and type:
{% highlight bash %}
esphome water.yaml run
{% endhighlight %}
For the first run, select the option to flash over serial.
If the device successfully connects to the Wi-Fi, write down the IP address.
If you have a DHCP server, it's time to create a static lease for your ESP32.
For subsequent runs, you can flash the device over the air using:

{% highlight bash %}
esphome water.yaml run --upload-port XX.XX.XX.XX
{% endhighlight %}

We can now connect the sensor to the board. The one from Amazon came with the necessary wires.
The sensor is pretty cheap and comes with no documentation.
After a few attempts, I realized that the TX pin should be connected to ...echo and RX to Trigger.
It does not make sense but it works. So connects RX go GPIO26 and TX to GPIO25. If it does not work, swap them.

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

You can start by an update_interval of 1s for trouble shooting purpose.
Flash the device (over the air or serial).
It is very important to place the sensor perpendicularly to the surface you are pointing.
The sensor sends a sounds and measure the time for the echo. If it's not perpendicular, you won't get an measure. If it's not perpendicular enough, it will often timout.

However, I took care of that already. When the measure does not succeed, the ESP32 sends "nan". Home Assistant will interpret it as "Unknown". Unfortunately, there is no easy way to display the last known value in Home Assistant. So I added a filter to ensure that the ESPhome returns a value only if it's not "nan". It is that part of the config file:
{% highlight yaml %}
    - filter_out: nan
{% endhighlight %}

Point the sensor to a wall (perpendicular) and run the ESPhome. Got a value? Good, let's continue.

Placing the sensor very precisely in the tank is not easy. So I design a sensor holder for 3D printers.
It works pretty well and it really improves the measures.
You can download it here:
* STL file
* Prusa printer files

Here is a picture of the device placed:

Finally, go to your Home Assistant web interface and follow [this guide](https://www.home-assistant.io/integrations/esphome/).
