---
layout: post
title: "Lights....Meetings.....Gaming?"
tags: ["Home-Automation","Gaming", "Home-Assistant"]
categories: ["Home-Automation","Gaming", "Home-Assistant"]
---

## Introduction

First off, let me apologize. It has been over a year since I put together a post for this blog. But I have been busy with life and some projects have fallen to the side just due to time. That is the thing about this blog. While I am not totally dedicating myself to it, it is more of a place for me to document things that I think are fun and neat and some projects that will help others out. However, I have been developing some smaller things and I do have one that I think is cool and would like to share. 

As you can see by the title, this project is the combination of 2 projects that I have worked on over time and really turned out to be something cool.

## Backstory

I have been gaming off and on since about 2006 when I started playing World of Warcraft (yes I am that old school Warcraft). About a year after I started, I purchased a gaming keypad. This is a device that you plug into the computer and it is basically a shrunk down keyboard that you can map the keys on to do certain functions. This has numerious keys and buttons and a direction joystick on it that allows you to keep one hand on it and one hand on your mouse. That way you are not searching around to keys on your keyboard during a big battle or when flying a plane for example. It let's you focus more on your screen versus having to rememeber where all the keys you want are. At the time I had a Razer Nostromo and it served me well over the years up till recently. I noticed a few months ago that some of the keys were starting to stick and some would not reliably work and that was a problem. 

So in August, I purchased a new keypad and upgraded to a Razer Tartarus V2. I say upgraded as the Tartarus has 5 more keys and RGB LED lighting. Basically, it is the upgraded version of the Nostromo. The lighting on the Tartarus is awesome. I can set the gaming profiles for different games and the keys will light different colors based on the game. I can even go so far as to program the lighting to make each individual key or set of keys different colors. The Nostromo had 2 light settings on it. On or off. This Tartarus lets me do all sorts of things.

Last year, I wrote about how I started using the Nostromo to help with Home Automation and one of the things that I did was setup a keybind to toggle my meeting status light. I have a light that I put together with a Raspberry Pi, a Blinkt! LED light hat and Python that I use to let the rest of my family know when I am in a meeting or not. Since working from home, this has proved itself invaluable time after time. When I got the Tartarus, I setup a key map to do the same thing. But I always felt I could do something more with the lighting on the keypad. 

And that is what brings us to now.

Using a combination of Python, Home Assistant and time, I have been able to not only mirror the light setting for the meeting light with the lights on the Tartarus, I also have been able to incorporate the CheerLights project into this project so that the lights are always changing. It's pretty cool during the work day to see the light change from green to red automatically as I am in and out of meetings and then change colors in the evening and on the weekends while gaming.

So let's jump right into how I did all this.

One thing to note. This is more of a description on how I did it then a walk through to help you make it. Everything will be different for other people so I am going to try to keep these as generic as possible in this post. I may look at doing a deeper dive into how I setup HA to do all the things with the CheerLights Project. 

## Software Used

- Home Assistant (Version 20231005.0 as of this writing)
- Home Assistant Community Store (HACS)
- Eclipse Mosquitto MQTT Broker
- Razer Synapse Version 3
- Windows 10 (Since the Tartarus lives on my gaming computer)

## First Steps

There were some steps that were already done and a few others I needed to do to make this work.

- Install and configure the Razer Synapse 3 Software. Make sure it is working with your keypad and you are able to configure settings on your keys.
- Make sure that you have Home Assistant installed and configured.
- Make sure that the Mosquitto Broker is up and running and connected to Home Assistant. 
- Make sure that you have HACS installed into Home Assistant (we need to use a community integration for this to work)
- Install the Virtual Component Integration from HACS
- Install the Chroma integration from HACS

Since I already had all of this done, this part was easy.

## Meeting Light

Using some Python I wrote, A few years ago I created a meeting light, as I mentioned earlier. This light uses a Raspberry PI Zero W and a Blinkt! LED hat and is mounted to the door frame outside the room (our bedroom). What this light does is if it is green it means that I am not in a meeting or on a phone call and it is ok for anyone to come in and red means that I am in a meeting and can't be disturbed. This is really useful over the summer when my little one is home, so that he knows if it is ok to come in to ask me something or whatever. 

Using Home Assistant, I can automatically turn the light on and off and change the color based on when I am in a meeting or not. This has been working well for about 3-4 years. To do this, I ingerated my iCloud meeting calendar into HA and the light will change when I have an event schduled at a time during the day. So when the meeting event starts on the calendar, the light changes to red. When the meeting event is over, the light will sutomatically change back to red.

But there are times where I am pulled into a meeting or a phone call and do not have it scheduled. I have a button setup on my gaming pad that when I hit the button, it toggles the light color.

## Chroma HA Integration

However, one of the things that up till recently I wish I had was a way to have an indicator on my desk other then the dashboard in Home Assistant to indicate what state the light is in. While looking through HACS a few days ago, I came across a new integration that someone wrote that allows Home Assistant to integrate with the Razer Chroma API that runs on all the Razer devices that support the Chroma API. Razer Chroma is a way for your Razer gaming devices to set colors based on events or profiles or games. 

I loaded this integration into Home Assitant and was able to get it to connect to my device. Once that happened, all I needed to do was to create 2 new Scenes in HA. One for Green and one for Red. Now when Home Assistant tells the Meeting light to change depending on meeting status, it will also activate the appropriate scene to set the color on the Keypad.

Now when I am in a meeting, the whole keypad is lit red and when I am out of a meeting it is lit green, mirroring what the meeting status light outside the door is doing. It also turns the lights off at night and automatically in the morning on working Days.

But what about in the evening or on weekends? Well that is where it gets cooler.

## The Lights of Cheer

One of the projects I have been privledged to contribute code to is called CheerLights. CheerLights was started about 10 years ago by Hans Scharler and I have been playing with the project for a while myself. I love the idea of anyone anywhere being able to send a color on Mastodon or Discord and everyone's lights who has built a CheerLights project will change to the color. Basically, when someone chooses a color, lights all over the world will change to that same color. It is fastinating to think about.

For a while I have been using Home Assistant with the Virtual Components integation to turn the CheerLights devices I use on and off, but that was the extent of it. 

Using the Chroma Integration, I now have added the ability to read what CheerLights colors are coming in and react and send the correct color to the Tartarus and the keypad changes to that color.

What I did was create a few more scenes in HA to cover all the possible colors that CheerLights supports. I have had a sensor in HA for CheerLights that tells me there what the current CheerLights color is. This information is pulled off my MQTT Broker that I setup earlier. This allows me to have one source of truth on my network to control what all the lights are doing. I used that sensor along with an automation that uses a choice action to determine what scene should be activated. When CheerLights changes, as long as it is not in the middle of the work day, the color of the keypad then changes to the current CheerLights color. It is kind of fun and trippy to be gaming in the evening and see the keypad change color right under my hand. It really has added a new level to my gaming. 

## Wrap Up

And there you go! Using all these integrations and scenes, I now not only have a visual indicator on my desk for my meeting light, but I also have elevated my gaming by tying in CheerLights to the system as well.

I am planning on doing a post about all my CheerLights work and how I make it all work together in the future. 