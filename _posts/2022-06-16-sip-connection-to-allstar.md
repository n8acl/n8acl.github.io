---
layout: post
title: "Setup Allstar for a SIP Client Connection"
tags: Ham-Radio
categories: Ham-Radio
---

# Introduction

In my last post, I talked about how to setup Allstar to connect to Discord. In this one, I am going to talk about setting up a SIP client to connect to Allstar.

But why would you want to do this? There are alot of softphone clients out there, as well as a bunch of old liquidated IP Hard Phones that use the Session Inititation Protocol, or SIP, to connect to a PBX. IAX is used to connect Asterisk Peers together, but in many applications, it has been used as a way to connect a client. There are some flaws and got-ya's to that, but SIP is a much better, cleaner protocol to use for carrying Audio over a call.

Remember that under the hood, Allstar is really just an Asterisk PBX, so connecting a phone to it is trivial.

In talking about connecting Discord to Allstar, some one asked me about how to setup a SIP client to connect, so hang on to your hats, cause here we go.

## Assumptions

For this article we assume:
* you have a working ASL system already setup
  * The ASL system I am using is built on an Ubuntu 20.04 server using the DVSwitch fork of Allstar. I do not have Hamvoip or the new ASL Beta installed, so things might be a little different.
  * You should have a static IP address for your Allstar System setup. Each router is different and you will need to look that up.
* You have a SIP Client already (be it a hardphone, softphone, whatever)
* You have a fully qualified Domain Name through someone like DDNS or something to your home network. You ISP public IP Address can change so you will need a Dynamic DNS service to route a URL to your IP.
* You are comfortable editing files in Linux

## Configure Allstar

First we need to setup our Allstar system to accept a SIP connection.

You will need to edit 2 files in ```/etc/asterisk``` on your Allstar server: ```sip.conf``` and ```modules.conf```

#### Modules.conf

First we need to edit the ```modules.conf``` file. 

```bash
sudo nano /etc/asterisk/modules.conf
```

Look for the SIP Channel and make sure that is set to load. Change noload to load:

```bash
load => chan_sip.so ;				Session Initiation Protocol (SIP)  
```

Save that (if using nano ```CTRL``` + ```X``` + ```Y```)

#### sip.conf

Next we need to edit the ```sip.conf``` file.

```bash
sudo nano /etc/asterisk/sip.conf
```

You will need to do the following:

* Set the ```bindaddr``` field to 0.0.0.0 (so it doesn't bind to specific address)
* If you have another phone using SIP port 5060, you will need to change ```bindport``` to something else. You could use 5061. Remember this for later.

Then add a stanza similiar to this:

```bash
[2000]
type=friend
host=dynamic
username=2000
secret=<your secret password>  ; make it yours 
dtmfmode=rfc2833
mailbox=2000                  ; Mailbox for message waiting indicator
context=myphone           ; Points to the stanza in extensions.conf
callerid="<Your callsign>"
```

2000 is just the extension number I gave it.. It can be anything you want, but remember that this will be the username for your login and it must be a number. We put that in the stanza header and then also in username and mailbox.

Make sure to set a password you will remember. you will need this later.

Callerid can be anything you want (blank even) since no one will ever see it.

Context is the context stanza you have setup for your Allstar Nodes in the ```extensions.conf``` file. When I initially setup my Allstar Node Extensions I called it ```[myphone]``` for lack of anything better at the time. So whatever your stanza title is, is what you will use.

Save that (if using nano ```CTRL``` + ```X``` + ```Y```)

Once all these are edited and saved, you will need to restart your asterisk service.

```bash
sudo service asterisk restart
```

Once it comes back up, make sure that it comes up correctly.

## Setup your router

Unless you are like me and don't like to punch threw unessesary ports on your router firewall and connect to all you services at home via a VPN (told you I was a geek), you will need to forward the SIP port in your router to the IP address of your Allstar System. You will need to forward it as UDP. This port number will be the same one you just setup in the ```sip.conf``` file.

Each router is different on how to do this, so you will need to look that up.

If you are only going to connect within your network, you will not need to do this step.

## Setup your phone

Now you will need to setup your phone. 

Each phone is a little different. But for the most part when you setup your account, you will need to setup a SIP connection and the following:

* ```username``` - This is the username that you set up in your ```sip.conf``` file
* ```password``` - Same thing
* ```domain``` - This will be your public IP Address or Dynamic DNS URL that you setup. You will also need the port number. 
  * Example: ```my.coolurl.com:5061```

Once you have that all setup it should connect. 

If you are using the phone internally, you would point it at the IP address of your Allstar system instead.

If you are using the Groundwire client, you could use [this walkthrough](https://hamsoverip.github.io/wiki/endpoints/soft_phones/groundwire/groundwire/) as a starting place. Just make sure to use the right username, password and domain for your Allstar System.

## Using this connection

To use this connection, once it connects, all you have to do is dial the node number of your Allstar node and hit send. It should Connect, play an announcement if one is setup, and now you can control it just like if you were on a radio. ```*3+node number``` to connect to a node, ```*1+node Number``` to disconnct, etc.

To Transmit, you would dial ```*99``` to talk and ```#``` to stop. That is your PTT.

# Wrap Up

Unfortunatly there are alot of different types of Hard and Softphones out there, so covering all of them would never happen in this article. This is just a way to show you how to setup your Allstar Node to accept SIP connections. This would help to eliminate the need to have multiple nodes, cellular modems and all sorts of other things.

This is just one of many ways that we can connect Ham Radio and Computers together. Part of why I do what I do :)

Enjoy! Till next we meet!
