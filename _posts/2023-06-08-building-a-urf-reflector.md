---
layout: post
title: "Building a URF Reflector System"
tags: Ham-Radio
categories: Ham-Radio
---

## Introduction

Recently a good friend of mine ran across a new way to build an XLX reflector system that has quite a few more features then using the standard XLX reflector system. This newish one is called a URF system and is a fork of the XLX system, but way more updated. 

From the README for the IRF Reflector:

"A urfd supports DStar protocols (DPlus, DCS, DExtra and G3) DMR protocols (MMDVMHost, DMR+ and NXDN), M17, YSF, P25 (using IMBE) and USRP (Allstar). A key part of this is the hybrid transcoder, tcd, which is in a separate repository. You can't interlink urfd with xlxd. This reflector can be built without a transcoder, but clients will only hear other clients using the same codec."

It does still use the XLX infrastructure to add the reflector to radio hotspots and lists, etc, but it addes the ability to do all digital modes out of the box and has the ability to transcode audio between modes much easier. It is also a little simplier to get setup and running.

So being who I am, I decided to tackle setting one of these up and as I started down the rabbit hole, I began to realize that, while still simple and cool, it was not as simple as I first thought. The developer did a good job documenting the process, but there are alot of gotchas that both my friend and I ran across and after talking I decided I was going to document the whole process here for a few reasons. 1) When I need to reload this in a few weeks (cause I will break something no doubt) I will need to remember what I did 2) This gives both my friend and I a place that this is all documented in the future if we need to build one again and 3)I wanted to share the process cause you know.... techy blog. Plus I realized I hadn't really posted anything new since September. Life got busy and I started gaming more, so I hadn't really devoted any time for development or doing any new techy stuff.

My friend built his system on a VM in his Proxmox cluster in his home lab. I don't have Promox installed on anything, but I did have a Raspberry Pi kicking around looking for something to do. So while this system can be built to a certain extent on a regular computer system as documented here, to get the whole experience and to be able to do transcoding via software, you will need a Raspberry Pi. If you have a hardware transcoder kicking about you can use that on a regualr system, but on an x86 system you HAVE to use hardware to do the transcoding. Software transcoding will only work on the Raspberry Pi with this method (according to the developer), using the repos and software I am going to talk about below. 

The foot print and processing power is so small and it runs well on a Raspberry Pi 3B+, which is what I am using this on. So it should handle multiple connections well. So this is a very viable option for yo.


Just keep in mind, I am gearing this walkthrough towards building it on a Raspberry Pi. Your Mileage May Vary on other systems and you will need to work out any kinks from there.

So let's get started.

## Skills needed

For this Project you will need:
* a Raspberry Pi (1,2,3,4 are required. This will not work on a Zero 1 or 2)
* the latest Raspberry Pi OS (the lite version. There is no need for a Desktop)
* to understand how to use the following on Linux:
    * make
    * git
* to be comfortable building software from scratch on Linux
* to be comfortable clonng git repos
* to be comfortable using the Linux terminal and SSH

Also a Disclaimer: You modify your systems at your own risk. I am not responsible if you hose up your own installs

### Setup your Raspberry Pi

First you will need to burn the OS to an SD card and get the pi setuo and updated. There are pletnty of online guides to help you with that. Once you have the pi setup and ready to go, you can proceed to the next step.

Make sure that you run all the latest updates on the pi before moving on.

### Setup URFD

URFD Reflector Repo: [https://github.com/n7tae/urfd](https://github.com/n7tae/urfd)

Our first step is to install and configure the URFD Reflector system. Without this the rest of the process if kind of moot.

SSH into your pi and issue the following commands:

```bash

sudo apt install git apache2 php build-essential nlohmann-json3-dev libcurl4-gnutls-dev

git clone https://github.com/n7tae/urfd.git

cd urfd/reflector
```

#### Choose Your Number

Before you get to far into this you need to make a couple of decisions. 

First, do you want this reflector to be accessed by anyone or are you setting this up for you to either use as a hub for linking some stuff or whatever. This is important to figure out now, cause this will play a roll in how this is setup later.

You will need to choose a 3 character identifier for your reflector. This needs to be a 3 digit number or a 3 alphanumeric digit identifier. So for example... 666 or ABC would work. However, if you are going to put it on the public internet, you will need to choose an identifier that is currently not being used. To find out, you can use Google to look for an XLX Refelctor list and look through that and find an unused number. If you are not putting this out in the public, then you can use whatever number you want. 

If you are putting this out in the public, or are making it private with plans to put it out in the public **DO NOT USE AN ALREADY USED NUMBER**. This will not only cause a conflict on the system, but also piss off **ALOT** of people.

#### Setup Configuration Files

Now we need to setup the configuration files for the reflector. First copy the configuration files to a seperate config folder by running this command:

```bash
cp ../config/* .

```

This will create 7 files in the config folder. We are only concerned with 2 at the moment. 
* urfd.mk
* urfd.ini

```urfd.mk``` controls settings at compile time and ```urfd.ini``` controls setting at run time for the reflector when it is running.

**Edit urfd.mk**

First, edit the ```urfd.mk``` file. We are going to edit this in the ```urfd/reflector``` folder.

```bash
sudo nano urfd.mk
```

There are only a handful of lines in there, but the one we are concerned with is:

```bash
# To disable DHT support, set DHT to false.
DHT = true

```

We want to set that value to ```false``` since we are not going to build DHT Support. DHT is a blockchain method of setting ID's that the system can use for connections. It is really cool, but kind of a pain to get working. We don't have to have it and it is optional to use so.

Once you made that change hit ```Ctrl-X``` then ```Y``` to save and exit the file.

**Edit urfd.ini**

Next we are going to edit the ```urfd.ini``` file. So we are going to do

```bash
cd ..

cd config

sudo nano urfd.ini
 ```
Again this is a longfile, but there are a few lines we need to edit for this.

Let's start at the top and work our way down. I am going to skip most of the lines so if you don't see what I am talking about here right away, make sure you scroll down.


* ```Callsign = URF???``` - Change the ```???``` to the Number you chose above for your reflector.
* ```SysopEmail = ``` - Change this to your email address.
* ```Country = ``` - Change this to your 2 digit country code (ex. US)
* Comment out the line that says ```Transcoder```
* ```Modules =``` - This is where you define the modules that you want to use. You can use any of the 26 from A-Z and they do not have to be contiguious. Use Capital letters and string them all together, no spaces, commas or anything.
    * Example: ABCHTK
* Now each Module needs a Description field. So the Description for each module is ```Description?=``` where the ```?``` is the letter of the module. Next to that just type out a short decription. You may need to change letters or add lines depending on your module setup.
    * example: DescriptionA = My Module A
* Under the ```[Protocols]``` section, you will leave everything alone for right now, but make sure that you scroll through this section and comment out any lines that have ```AutoLinkModule``` in them. We don't want to link any of these.
* Under the database files section, make sure to set any files paths to your ```urfd/config``` directory. They need to be absolute paths.
* Under other file locations do the same thing.

Once you made those changes hit ```Ctrl-X``` then ```Y``` to save and exit the file.

### Build URFD
N
ow we can build the reflceotr system. In the ```urfd/reflecor``` directory, run the following:
```bash
sudo make
sudo make install
```

Once that is completed, issue the following:

```bash
sudo systemctl stop urfd
```

We need to make some additional adjustments to things before we are ready to leave it runni.