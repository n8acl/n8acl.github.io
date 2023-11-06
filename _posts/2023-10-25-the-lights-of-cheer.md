---
layout: post
title: "The Lights of Cheer"
tags: ["Home-Automation","Home-Assistant"]
categories: ["Home-Automation","Home-Assistant"]
---

## Introduction

It is that time of year again. Fall is dropped and Winter is right around the corner. That means all sorts of fun holiday displays for Halloween and Christmas. Most include some sort of cheerful lighting in the display. Alot use some automation to make them unique and really fun and appealing. That is why this post is probably appropriatly timed. 

Over the last few years, I have been very priviledged to contribute some code to an awesome project that helps people to learn some programming, small project building and is just a fun project to play with for everyone of any age. This project has been used all over the world and is very popular in the DIY crowd. The project is called CheerLights. This project was started about 10 years ago by Hans Scharler and is still supported today. I have been lucky to be able to contribute code for the Discord bot that allows people in the Official CheerLights Server to be able to change the lights right from there. 

There have been so many projects and people integrating lighting displays into CheerLights and some of them have been so elaborate. 

I have been working with little lighting projects over the last few years and with some recent changes to my setup, I decided it was time to talk about CheerLights and how I am using it here. 

Maybe this will help inspire you to create your own projects.

As I mentioned, CheerLights does have it's own Discord Server. The invite link is: [https://cheerlights.com/discord](https://cheerlights.com/discord) Come join us and be a part of changing lights in displays ALL over the world!


If you would like to see some examples of code on what I did for my projects, check out my Github Repo: [https://github.com/n8acl/cheerlights_projects](https://github.com/n8acl/cheerlights_projects)

## Materials Used

- Software
  - Python
  - Home Assistant (Version 20231005.0 as of this writing)
    - Home Assistant Community Store (HACS)
      - Virtual Components Integration
      - Chroma integration
  - Eclipse Mosquitto MQTT Broker
  - Razer Synapse Version 3
  - Windows 10 - For the Tartarus since it lives on my gaming machine
  - Linux
- Hardware
  - Raspberry PI Zero W with SenseHat hat
  - Razer Tartarus V2 Gaming Keypad
  - Frankensteined Gaming Machine (it was build by my brother inlaw a number of years ago and I inherited it. It is a custom built machine.)

## Bridging the Gap

So to kick this off, I am going to start from the beginning and work my way out. Meaning, I am going to start from where the data comes into my network and then work down the chain of how it branches out as well as the order of how I built things.

So the first thing I needed to do was get the CheerLights data coming into the network. Since I already run some other home automation, I already had Home Assistant and the Eclipse Mosquitto MQTT broker setup on my network. These both run in Docker. CheerLights has it's own MQTT broker that it uses to send out information that people can tap into to get the data from the system. They also have a web API that can be used, which is what I used to use, but since I had MQTT, I wanted one place on the network for a source of truth. That way I can connect everything to the broker and everything gets the same data stream. I also only wanted one connection out the network to save on Internet Bandwidth. Yes I know it is trivival, but why have multiple connections to the same external source. 

So I started off by using Python to write a script that connects to both the CheerLights Broker as well as my own Broker and it flows the data from CL to my broker. Works like a charm and keeps me from having to work to bridge the two brokers, which I know can be done, but this solution was much easier. Now I have the data on my broker and I can then connect anything I want to it internally. I have the data flow to Home Assistant, a database and a couple of other Python Scripts.

## Raspberry Pi SenseHat

The next part I did was to write another Python Script to subscribe to the CheerLights topic on the broker and then programmed it to react to when data comes across. The SenseHat has an 8x8 RGB LED matrix on it. Using some Python magic, I was able to get the matrix to light up with the correct color. Now when a color comes in, the matrix lights up and this was my first project. 

I have used this in so many ways. As a light on my desk. I have put it in/on some holiday decorations to light those up. There are many other uses as well.

## Automating the Light

This script worked great, but after a while I got tired of SSHing into the pi to start and stop the script when I wanted the lights on or off. So I figured I could use Home Assistant to do it for me. 

I loaded an integration from the Home Assistant Community Store called Virtual Components. This integration allows you to create switchs, lights and other things that are not tied to a physical device IE virtual switches. This allowed me to create switches that would just drive automations.

So what I did was create a power switch and a sensor in Home Assistant along with some automations that would send a signal across my MQTT Broker to the script that runs the CheerLights on my SenseHat. This signal would tell the script to clear the lights or turn them on. Now the script can run all the time and I can turn it off and on at will.

I also tied an automation that turns on my desk lamp to the SenseHat CheerLights Switch that will turn the switch off when I turn off my desk lamp. That way the light is not burning all night.
