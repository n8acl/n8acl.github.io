---
layout: post
title: "Automating Your Smoking"
tags: ["Home-Automation","Smoking/BBQ"]
categories: ["Home-Automation","Smoking/BBQ"]
---

## Introduction

Tech has always been my number one hobby. I always have liked playing with computers and gadgets. Remember how on my homepage of this blog I mentioned that I have many other hobbies? I have always looked for ways where I can incorporate tech into these hobbies as well, beyond just using a GPS to find a greeat fishing spot or that really cool campsite off in the woods.

Within the last few months, I have started smoking meat. I have always wanted to try it and for Fathers Day of 2022, my wife and son bought me an electric smoker. I've done all sorts of meats in it so far. Brisket, pork butt, chicken. I still have some ribs to try and want to try a ham and a turkey at some point. But one of the biggest pain points that I have run into so far is the constant maintenance of the tempurature of the chamber. Unlike wood fired smokers, I don't have a fire I have to tend to, but I do need to make sure I monitor the temps of the chamber and the meat itself to make sure things are cooking properly. I had a cheap little multi-probe thermometer that I used to monitor it all, but the thing was, I constantly had to get up from what I was doing, go outside, check the temps, adjust and then do it all over again 15-30 minutes later. 

So for my birthday, my loving wife got me a [Tenergy Solis iBBQ Bluetooth Smart Thermometer](https://www.amazon.com/dp/B077821Z4C?psc=1&ref=ppx_yo2ov_dt_b_product_details). This one I can use bluetooth to connect it to my phone and get notifications on temps and also check the temp in real time of the meat and chamber. But since I am a geek, while that adds to the geek factor of smoking meat, I wanted to do more. Enter Home Automation.

I have been using Home Assistant (HA) to run some automations in the house for a few years now. Things like lights for example. So I wanted to tie this new thermometer into HA to be able to see everything on one dashboard, show graphs of the temps so I can see if the temps are on the right course and get notifications on when temps have reached certain things and also set reminders to check on things like wood chips or temps.

By tieing this into HA, I can now leave the house if I need to, leaving my wife here to make sure the house does not burn down of course. But if I need more supplies like wood chips or sides for dinner, this frees me up to be able to do that since I can see the dashboards outside the house and setup notifications to Discord in HA. Once I walk out of the house and get far enough away, my phone will of course disconnect from the Thermometer, so it is useless if I get to far away.

The goal of this article is going to be less a step by step walkthrough like I have done previously, since everything I am doing is already documented online very well, but more of a discussion and the path I took to get to where I am at and links to the things that you will need to be able to do this as well.

One of the other things I want to point out is, the method I used is also useful for other aspects of home automations. As long as a sensor or device uses bluetooth, I can pull the data into HA and use it for other things too. So this method also has uses outside of this, which is awesome.

Also my usual Disclaimer: You modify your systems at your own risk. If something goes sideways, it is not my fault.

## Software Used

* Python
* [Docker/Docker-Compose](https://www.docker.com/) - This is container tools I use to run Home Assistant, Grafana and Eclipse Mosquitto (MQTT Broker)
* [Eclipse Mosquitto](https://mosquitto.org/) - My MQTT Broker
* [Theengs Gateway](https://gateway.theengs.io/) - This is the BLE packet decoder
* [Home Assistant](https://www.home-assistant.io/) - This is my Home Automation Controller
* [Grafana](https://grafana.com/) - This is used for graphing time series data

## Hardware Used

* Raspberry Pi 4
* Whatever my Docker host Server is (an old Dell Optiplex I think)
* [Tenergy Solis iBBQ Bluetooth Smart Thermometer](https://www.amazon.com/dp/B077821Z4C?psc=1&ref=ppx_yo2ov_dt_b_product_details)

## One point to make

I would like to point out one thing before we get started. I have my own Home Lab setup here and have multiple servers and multiple Pis. I run alot of Microservices in Docker on an old Desktop that I converted to a server and that is where my Home Assistant and Grafana live. However, I just wanted to point out that if you are starting from scratch on all of this, the Raspberry PI 4 I have mentioned can run ALL of this. Docker, Home Assistant, Grafana, etc... it can all run on it. In fact, before I got this server a year or so ago and moved all my container infra over to it, I was running everything on this PI 4 that I am using now. So it will handle it.

## Theengs Gateway

I started this project trying to use another project called CloudBBQ. It is supposed to do basically the same thing as Theengs, but it is geared more towards specific uses. However, try as I might, I could not get it to work. Not saying it's a bad project, but after spending 3 days fighting with it, I felt there had to be a better solution.

Theengs Gateway is a Bluetooth Low Energy (BLE) decoder. It has alot of bluetooth devices that it can decode already, and more are being added, but this does read ANY bluetooth packet it can hear too. It just may not be able to decode them is all. Keep that in mind as we go through this. 

The Theengs website documents how to get this going.

## Get your Thermometer Topic

Now that Theengs Gateway is running, you will need to find the topic where the data is being sent.

Find a good MQTT client (I recommend [MQTT Explorer](http://mqtt-explorer.com/) as it organizes things and makes everything easier to see and is cross platform) that you like and connect it to your broker. Once connected, look for the Theengs topic you created and subscribe to it. 

Now watch for a data packet that has the information about your Thermometer. It should be its own topic based on the MAC address of the bluetooth radio of your thermometer. You are going to need to watch everything coming across to identify the correct one for your thermometer. Unless you can look up your MAC address for your Thermometer.

I let the gateway scan at it's default so I can get the temp data is as much real time as I can get it, which I think is every 5 seconds and as often as the thermometer can send the data. Since I use my own broker and the data does not go outside my network, I am ok with that much data hitting the broker all the time.

## Parse the JSON Packet from the Thermometer

Now we need to parse the JSON to get the probe data that is being sent to MQTT. I chose to use Python for this and I put the script up on Github that I created to do the parsing. You can use it as a base to see how I did it for the Tenergy that I have, but of course, you can use whatever you are most comfortable in. Javascript, C, even Node-Red would work with this.

Basically what I have done is my script listens to the thermometer topic that we found from Theengs Gateway, parses the probe data out of it and then sends it back to MQTT on a set of different topics, one for each probe. I also push the data to a MySQL database table. I will explain next why I do this.

The script can be found at: [https://github.com/n8acl/smoking_scripts](https://github.com/n8acl/smoking_scripts)

## Home Assistant

Now that I have the probe data going back out over MQTT on their own topics, I setup sensors in Home Assistant that can see those topics and then displays the temps in 
guages so I can see all the probes. 

The Dashboard I set up shows me the weather, all the reminder and notifications switchs and the probe temps and the graph I created so I have one place to see it all.

## Grafana

Speaking of graphing the temp data, I chose to use Grafana for this. Using the time series data I push to MySQL, I can see the history of the probe temps over time. This allows me to guage at a glance at how fast the meat is cooking so I can adjust cooking temps if I need to. 

I can also later pull the history of the cook to see at the end about how fast a certain temp settings this particular hunk of animal took to cook so I can plan later cooks.

## Notifications

One last thing I needed is to get notifications on things so I can stay on top of the cook. For example, if the chamber temp goes over 275 or under 225 I can get a notification or when the meat hits say 150 and I need to wrap it. 

There are multiple ways that I can accomplish this:

* I can setup notifications through automations in Home Assistant when the temps hit certain values. 
* I can use the same or a different Python Script to send a notification when it gets certain data as well.
* Since Theengs Gateway does not directly connect to the thermometer, I can still use the app on my phone and set alarms there as well.

If you choose to use a different programming platform, you can also choose to use that to send notifications. Whatever you choose to use, you need will need to get notifications to stay on top of things.
