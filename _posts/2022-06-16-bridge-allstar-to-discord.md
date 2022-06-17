---
layout: post
title: "Bridge Allstar to Discord"
categories: Ham-Radio
---

## Introduction
There are any number of VOIP based radio linking networks out there, but for this article, Allstarlink is the one we are looking at. It is an internet based VOIP radio linking system that uses the Asterisk PBX as the backbone to link radios, repeaters and even phones to allow Ham's to talk all around the world. Why is this important?

Because of it's flexibility, Allstar is used at the heart of many multi-mode, bridge systems that allow any number of Ham Radio digital modes to talk to one another. We are able to bridge in DMR, D-Star, P25, Yeasu System Fusion, M-17 and even "Analog" Modes like Echolink, IRLP, Allstar and (since Asterisk is a PBX) we can even connect SIP based phone clients like Cisco IP phones and softphones like on the computer or our cellphone. So this really gives ALOT of flexibility to really be able to choose a mode and be able to talk with others. 

There have been some other attempts in the past to connect other VOIP chat systems to Ham Radio as well. Some have used TeamSpeak or Ventrilo or Mumble, but these usually involve using an Audio mixer program like Virtual Audio Cables or Jack or something like that. However, I wanted a more elegant way to bridge a Discord audio channel to Allstar and then rest of the world. And I finally came up with a solution.

There is a project that I found that was designed to bridge Discord to a DMR Talkgroup called [dmr-bridge-discord](https://github.com/jess-sys/dmr-bridge-discord). This project connects to Discord via a bot and then uses Analog_Bridge and MMDVM_Bridge to connect to a DMR network and link to a DMR Talkgroup. The project is written in RUST. 

I had been sitting around playing with some ideas and then one late Friday night, at about 1:30 in the morning, I had an epiphany! I was going about it all the hard way.

Digging into the project, you can see that dmr-bridge-discord uses USRP to connect to Analog_Bridge. Then I realized.... Allstar uses USRP to connect to Analog_Bridge. THAT'S IT! 

So here is how I did it.... 

This is a long article, but I wanted to take you through all the steps for this process. 

## Skills needed

For this Project you will need:
* to have an Allstar system setup and know how to change your config files
* a Discord Server already setup and know how to add bots, create channels and set permissions.
* be familiar with Git and cloning a Git repo
* be familiar with building code from source in Linux

## Building a Bridge

Keep in mind, this assumes:
* that you already have a working ASL System setup. We are not going to go through that process. 
  * The ASL system I am using is built on an Ubuntu 20.04 server using the DVSwitch fork of Allstar. I do not have Hamvoip or the new ASL Beta installed, so things might be a little different.
* that you already have a Discord Server setup that you administrate and know how to setup voice channels.

Also a Disclaimer: You modify your ASL setup at your own risk. If something goes sideways, it's not my fault. 

### Setup Discord Bot/Application

First you will need to setup a Discord Bot and a voice channel in order for the application to connect to Discord. In order to do this, do the following:

* Go to: [https://discord.com/developers/applications](https://discord.com/developers/applications)
* Click ```New Application```
* Give it a name (I called mine ASL471940 (since my node I am linking is ASL node 471940))
* On the next screen, you can upload an avatar for the bot.
* Click the ```bot``` selection under settings
* Click ```Add Bot```
* Give it a Name (I used the same name)
* Then Copy the Bot Token (you will need this for the .env file later)
* Turn off ```Public Bot```
* Make sure to turn on the ```Message Intents``` Setting under ```Privileged Gateway Intents```.
* Save Settings
* Click ```OAuth2``` and then ```URL Generator```
* For Scope Choose ```bot``` and ```appplications.commands```
* For Permissions, choose the following:
    - Under General Permissions:
        - Read Messages/View Channels
    - Under Text Permissions:
        - Send messages
        - Send Messages in threads
        - Embed Links
        - Attach Files
        - Read message History
        - Use Slash Commands
    - Under Voice Permissions:
        - Connect
        - Speak
        - Use Voice Activites
        - Priority Speaker
* Copy the generated URL
* Paste it into a browser window Address bar
* Choose the Server you want to authorize it to and then click authorize.
* It should pop into the server.

### Setup Allstar

Now we need to setup a private Allstar node in order for the application to send audio to and from Allstar.

If you have setup private nodes before (for DMR/D-Star/etc or for bridging in your phone) you can skip this step. If not, or if you need a review.

You will need to edit 3 files in ```/etc/asterisk``` on your Allstar server: ```extensions.conf```, ```rpt.conf``` and ```modules.conf```

#### Extensions.conf

Let's start with ```extensions.conf``` since it is the easiest change to make. Open your ```extensions.conf``` file in your favorite editor or use nano:

```bash
sudo nano /etc/asterisk/extensions.conf
```

Look for the ```[radio-secure]``` stanza near the top. It will look something like this:

```conf
[radio-secure]
;exten => ${NODE},1,rpt,${NODE}
exten => 471940,1,rpt,471940
exten => 1896,1,rpt,1896
```

You will need to add a line here for your new private node. In this case, I used 1896 as my private node number. You can use any number from 1000 - 2000 for your private node number, but I started with 1899 (I have other private nodes) for mine since I read somewhere that since everyone is using the 1900 series of numbers, it can cause problems for Asterisk to connect. So we add a line like this:

```bash
exten => 1896,1,rpt,1896
```

Save that (if using nano ```CTRL``` + ```X``` + ```Y```)

#### Modules.conf

Next we need to edit the ```modules.conf``` file. 

```bash
sudo nano /etc/asterisk/modules.conf
```

Look for the USRP Channel and make sure that is set to load. Change noload to load:

```bash
load => chan_usrp.so ;			GNU Radio interface USRP Channel Driver
```

Save that (if using nano ```CTRL``` + ```X``` + ```Y```)

#### Rpt.conf

Next we need to edit the ```rpt.conf``` file. 

```bash
sudo nano /etc/asterisk/rpt.conf
```

Here we are going to add a bunch of stuff. First, scroll down through that file to the bottom of the stanza that has your last node configured. You are going to add the following underneath that:

```bash
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

[1896] ; Discord to ASL 403730

rxchannel = USRP/10.0.0.4:34002:32002  ; Use the USRP channel driver. Must be enabled in modules.conf
					; 127.0.0.1 = IP of the target application
					; 34001 = UDP port the target application is listening on
					; 32001 = UDP port ASL is listening on

duplex = 0				; 0 = Half duplex with no telemetry tones or hang time. Ah, but Allison STILL talks!

hangtime = 0				; squelch tail hang time 0

althangtime = 0				; longer squelch tail hang time 0

holdofftelem = 1			; Hold off all telemetry when signal is present on receiver or from connected nodes
					; except when an ID needs to be done and there is a signal coming from a connected node.

telemdefault = 0			; 0 = telemetry output off. Don't send Allison to DMR !!!!!!!!!!!!!!!!! Trust me.

telemdynamic = 0			; 0 = disallow users to change the local telemetry setting with a COP command,

linktolink = no				; disables forcing physical half-duplex operation of main repeater while
					; still keeping half-duplex semantics (optional)

nounkeyct = 1				; Set to a 1 to eliminate courtesy tones and associated delays.

totime = 180000				; transmit time-out time (in ms) (optional, default 3 minutes 180000 ms)

idrecording = |ie			; id recording or morse string see http://ohnosec.org/drupal/node/87

idtalkover = |ie			; Talkover ID (optional) default is none see http://ohnosec.org/drupal/node/129

startup_macro = *934
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
```

Here we will need to edit the ```rxchannel``` to set the IP ADDRESS of where the dmr-bridge-discord application will be running. In this example, I have it running on another server on my Home Lab, so I have it pointing there. If you are going to run the bot locally, you will change that to 127.0.0.1. Leave the ports as they are if you like. You will need to remember these for the ```dmr-bridge-discord``` step.

Next you will need to add the node to be registered with asterisk. Scroll down that file and look for the ```[nodes]``` stanza. It should look like this:

```bash
[nodes]
; Note, if you are using automatic update for allstar link nodes,
; no allstar link nodes should be defined here. Only place a definition
; for your local nodes, and private (off of allstar link) nodes here.

471940 = radio@127.0.0.1:4568/471940,NONE	; This must be changed to your node number
                                        ; and iax port number if not the default
1896 = radio@127.0.0.1:4568/1896,NONE
```

You will need to add the line for the 1896 node we just added. Make sure that you use the correct port. In my example, I am using 4568 as my ASL port. Standard is 4569. 

Finally, go to the node stanza of your main node that you want this to connect to and in the startmacro, set it so that this new private node will connect on startup.

Example:
```bash
startup_macro = *934*1896
```

Save that (if using nano ```CTRL``` + ```X``` + ```Y```)

Once all these are edited and saved, you will need to restart your asterisk service.

```bash
sudo service asterisk restart
```

Once it comes back up, make sure that it comes up correctly and your new node is connected.

### Setup dmr-bridge-discord

This will be the hardest part, but once this is done, you will be able to use this multiple times.

First go to [the project repo](https://github.com/jess-sys/dmr-bridge-discord) and you will need to walk through the steps on the README of the project to build the application from source. 

Once you have the release built, you will need to copy the ```.env``` file and the ```dmr-bridge-discord``` file to wherever you will be running this from.

Once you have them copied, you will need to edit the ```.env``` file. It will look like this:

```bash
# Analog Bridge configuration
TARGET_RX_ADDR="10.0.0.2:32002"
LOCAL_RX_ADDR="10.0.0.4:34002"

# Discord Bot configuration
BOT_TOKEN=<YOUR BOT TOKEN HERE>
BOT_PREFIX="!"

```
Where:
* ```TARGET_RX_ADDR``` is the IP address of your ASL system (if running this on the ASL system it will be 127.0.0.1) and the second port number from the private node stanza we setup earlier.
* ```LOCAL_RX_ADDR``` is the IP Address of where this application is going to run (again if on the ASL system 127.0.0.1) and the first port number from the private stanza that we setup earlier.
* ```BOT_TOKEN``` is the bot token from where we setup the Discord Bot earlier.
* ```BOT_PREFIX``` is the prefix that will trigger the bot to respond to a command. You can leave this alone, unless you are planning on running a second bot in the same server, but that is up to you.

Save the ```.env``` file.

To run the application and get audio flowing, in the directory where you have the files type :

```bash
./dmr-bridge-discord
```

Your bot should connect to ASL and Discord.

You may want this to run on boot up. There are numerous ways online that you can search for on how to make that happen.

### Create Discord Channel

The last part of the setup now is setting up a voice channel on your Discord Server. Add the channel with the following suggestions:
* make sure to set the permissions for the channel so that everyone cannot use Voice Activity and No priority Speaker. This will force people to have to use the mute button like a PTT or use a Keybind. 
* Add the bot as a user that can use Voice Activiey and Prioirty Speaker for the channel so it can send and receive audio.
* if this is a famliy server, make sure that ONLY licensed Hams have access to the channel

## Testing

Now we need to test to make sure that everything is working. You will need a way to control Allstar outside of Discord (I have a way to control ASL from Discord commands, but I have not pushed it to github yet. Once I have all the kinks out, I will update this post.)

On Discord, connect to the voice channel you want the bot in. Then in a text channel (or in the voice text channel) type ```!join``` This will cause the bot to join the channel.

Connect your ASL node to another node and see if it has traffic. You should be able to hear the audio from the nodes in Discord. Audio should flow back as well. 

A good one to use to test (if you have Echolink setup on your node) is the Echolink Test Server (from ASL 3009999). Since this will record and play audio back and forth, it's a good way to test the bridge.

# Wrap up

And there you have it. You now can talk on Allstar Via Discord. 

Just some final thoughts:
* When using this to check into nets or other systems, do not mention that you are on Discord. All you need to say is you are on Allstar, which is not a lie.
  * because most people will have no idea what you are talking about
  * some system administrators will ban you. Some systems do not like having external voice chat systems where "anyone" could get access to Ham Radio Frequencies, regardless of how secure you have it setup.
  * So sometimes discretion is in order.
* You will need a way to be able to control where your Allstar System can connect. As I previously mentioned, I am putting the final touches on a Bot Project that will allow you to control your ASL system from Discord via text commands. Once I have that up I will post about it.

Enjoy! Till next time!
