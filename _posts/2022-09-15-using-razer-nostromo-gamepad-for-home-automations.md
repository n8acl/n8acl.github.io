---
layout: post
title: "Using a Razer Nostromo Gamepad for Home Automations"
tags: ["Home-Automation","Gaming", "Home-Assistant"]
categories: ["Home-Automation","Gaming", Home-Assistant]
---

## Introduction

I used to play alot of online games. And I mean alot! There is about 8 years of my life that I dedicated (read: was addicited) to ***World of Warcraft***. So alot of gaming time. When I was hardcore gaming though, I got my hands on a Razer Nostromo Gamepad. If you don't know what a gamepad is (also called a speedpad or gaming keypad) this is a little device that has a directional thumb wheel to help you move around and any number of keys that are right under your fingers. Basically, this, paired with a mouse, allowed you to cast your spells and run around without having to take your fingers off the keys. It helped with spell rotation and helped to speed up timing for example. You didn;t have to search the keyboard for the keys you needed since they were all right there under your fingertips.

Over time, my priorities shifted and I got away from gaming as much, but stilled played some video games here and there. Recently, my Brother-In-Law convinced me to download and play a game he had found called ***7 Days to Die***. It's a post zombie apocalypic survival crafting game (think Minecraft meets The Walking Dead meets an MMO). It's kind of a fun game, but the controls on the keyboard were a little cumbersome. So I remembered I still had this Nostromo kicking around in a box and dug it out. Now I am playing this new game with this old game pad.

But now I have this additional peripheral living on my desk. I start looking at it the other day and thought to myself "Can I use this for other things other then gaming? Would this be useful in my work?" And then I realized, yes I could. For example. It's been almost 3 years since most of us started working from home. Some of us permanently. But since we also have families and meetings to balance, many have come up with a variant of a meeting status light to tlet the other people in the house know that now is not a good time to come ask Mommy/Daddy for a sandwich. I even built a variant of my own using a Raspberry Pi, Blinkt! hat, MQTT and Home Assistant. While I can make it automatically change using Home Assistant when I get into a Zoom meeting, there are times where I need to manually kick the light when I answer the phone. 

So for a while I was just clicking a button on a computer screen or using my phone or the Alexa sitting on my desk to change the color. But I needed a quicker way. And since I had a hunk of computer equipment with ALOT of programmable buttons on it, I thought it would be nice to be able to hit a button and have the lights change.

So now during the working day, I switch keybind modes on the gamepad and I can use the buttons to fire automations in Home Assistant that let me control not only the meeting light, but my desk lamp, and other lights in the house, all with the push of a button. After work, when it's time to game, I just switch keybinds again to the correct profile for the game I am playing and I am off to killing zombie hordes.

This will work with other gaming pads no doubt, but since I have a Nostromo, that is what I am going to talk to in this article.

## TL;DR

Sadly, the Nostromo I have is not supported on Linux. Newer versions of the pad are, but this particular one is to old to work. Plus, all the open source Linux Drivers/control applications I found did not support programming the keys. They only supported controling the RGB lights on the devices. This Nostromo I have only has one color light and it is usually off unless I am playing late at night. So this for this project, I had to leave my gaming machine living in Windows. But that is a sacrifice I am willing to make for the advancement of cool geeky stuff.

After getting the Razer Synapse software installed on my gaming machine for the Nostromo (which is a seperate machine on my desk from my primary computer and work computers), I started looking at the software and found that you can set the buttons to be programmed to launch applications. This is what gave me the idea to go down this path. Knowing the Home Assistant can use Webhooks as triggers for automations, I created a set of batch files that are triggered based on the button pressed. All these batch files have in them are curl commands to touch the Webhooks in Home Assistant, which in turn fires the automation that toggles the switches.

Lets look at one example of an automation I created for this. The example I will use is turning my desk lamp on and off.

## Software Used

* Home Assistant Version 20220907.0-latest
* Razer Synapse Software Version 2
* Windows 10

## Creating the Automations

The first thing I needed after setting up the Razer Synapse software, was a set of automations with Webhook triggers in Home Assistant. To do this:

* I went into Home Assistant and created a new automation (used a blank one to start)
* I set ```Trigger``` for ```Webhook``` and gave it the ```Webhook ID``` of ```jeff_desk_lamp```
* I then set an action to ```Call Service```, ```Toggle Switch``` and then chose the switch for my desk lamp.
* I saved the automation and called it ```Gamepad - Toggle Jeff Desk Lamp``` so I knew what it was later.
* After saving the automation, I went back in and copied the Webhook URL from the ```Webhook ID``` field. By doing this, you get the whole URL. In the lastest verison of Home Assistant, there is a little copy icon at the right hand side of the field. Clicking this will copy the webhook URL to your clipboard (it will be similiar to http://10.0.0.1/api/webhook/jeff_desk_lamp or whatever ID you gave it.)

Keep this webhook URL you will need it in the next step.

## Create the control Batch File

Now I went in and created a new batch file on my Windows Machine (I called mine toggle_jeff_desk_lamp.bat) and in this file all I have is one line that looks like this:

```bat
curl -X POST http://10.0.0.1/api/webhook/jeff_desk_lamp
```

This is a curl request to post a request to the URL of ```http://10.0.0.1/api/webhook/jeff_desk_lamp``` that I copied from Home Assistant in the Webhook ID Field.

That is all that is needed at a minimum for this to work.

## Configure the Nostromo

Once I had the batch file setup, I went into the Razer Synapse software, set the keybind map to the one I wanted to use for automations, then I selected the key I wanted to assign this new function to, selected ```Launch Application``` and then found the batch file I just created. As soon as I hit save, the button was programmed and ready to go.

At this point I just hit the button on the pad and the light turned off. I hit it again and it turned back on.

## Wrap Up

And that's it. It was a super easy solution for a task that I wanted to complete. After I created that automation, I created more to turn other lights on and off from my game pad. Now I have quick, easy control over things in the house and I am utilizing a piece of computer equipment that is already living on my desk now. I was really excited at the possibilites for this.

This has peaked my interest in trying to find other ways to use this. There are so many possibilities to be able to include this into my work flow. 

Enjoy! Till next time!