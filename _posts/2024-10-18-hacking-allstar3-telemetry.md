---
layout: post
title: "Hacking Allstar3 - Disable Statistics Telemetry"
tags: Ham-Radio
categories: Ham-Radio
---

# Introduction

Well... Appraently I did it again. It's been almost exacly a year since I posted anything. To be fair, this blog was designed from the beginning to really be a place that would not get many posts, just mostly stuff that I think would help others or things I did that I thought were really cool. That and I have been pretty busy over the last year, so while I have done some cool geeky stuff, I might come up with something that I have learned or done that I could write about.

Anyway. Apologies aside, as you can tell from the title, today we are going to talk about hacking the new Allstar 3 project. And by hacking, since it is an open source project, I mean "Let's dig into somethings about it that I want to change to make it fit my needs/wants and fix some issues that I see in it." That's the beauty of Open Source and why I like it.

A few of my friends have asked me about ASL3 and my thoughts, so I decided it was time to take a look at it. I am perfectly happy with my current install, but if I can upgrade it, that would be awesome. If I could get it on a Raspberry Pi and have it stable, that would be better still. That way I could free up a box for something else.

So Allstar itself has been around for MANY MANY years and it was starting to age out. Thankfully some of the people involved decided it was time to moderize it and update it to be more compatible with newer operating systems. There were alot of things broken with it prior to now, like for example, if you updated the OS of the box you had it running on, it would break the dahdi kernel and you had to go in and rebuild it and other things. It was a pain, so many people just never updated the box (for example, my production system is still running on Ubuntu 20.04). So the fact that they took it on and have brought it up to modern standards is absolutly fantastic. However, there were some decisions made that I don't agree with, but since I don't contribute to the project, I can't complain too much about it. All I can do is make changes to my own implementation of the system since I control that. I can also make decisions about how I handle the installation process.

For example. It was decieded that what they call their PI Appliance is a Rapsberry Pi image that runs ASL in a VM.... I'm sorry.... what? There is ZERO reason that a VM should be run on a Raspberry Pi. Thankfully they have a way to install it on bare metal as well, but I don't understand the thought process behind the VM thing.

But that is a small issue compared to something else I found that I REALLY don't agree with and take a bigger issue with. 

# Anonymous Data

While reading through the documentation, I found a small blurb buried on a page that they are collecting data from your ASL system and sending it back to them. Yep. It was about a sentence long that says they collect "Anonymous" data for the purpose of "Adoption and Performance" statistics. And then give a link to a github repo, which BTW is not the actual implementation of the stats gathering.

So I dug a little more into what "anonymous" data they were collecting and turns out that it is NOT actually anonymous. Truly anonymous data is data that does not have identifying information in it that can link the data back to a person. For example, a survey at work. If the first thing they ask for is your employee id, it's not anonymous. "Well we won't see it". Well someone can and it can be traced back... so yeah. 

In the case of ASL, this is an example of the data that they are collecting:
```bash
RANDOM_UUID = d56da682-f200-4307-a97c-e67ab393c69c
ASL_AST_VER = Asterisk 20.7.0+asl3-1.3-1.deb12
ASL_NODES = [ 460181 ]
ASL_HTTP_NODES = [ 460181 ]
ASL_IAX_NODES = [  ]
ASL_CHANS = [{ "460181" : "SimpleUSB" }]
ASL_UPTIME = 1:8:22
ASL_RELOAD_TIME= 1:8:22
OS_OS = Linux
OS_DISTRO = Debian
OS_RELEASE = 12
OS_KERNEL = 6.6.31+rpt-rpi-v8
OS_ARCH = aarch64
PKGS = [{ "allmon3" : "1.2.1-2" , "asl-apt-repos" : "1.4-1.deb12" , "asl3" : "3.0.0-3.deb" , "asl3-asterisk" : "2:20.7.0+asl3-1.3-1.deb12" , "asl3-asterisk-config" : "2:20.7.0+asl3-1.3-1.deb12" , "asl3-asterisk-config-custom" : "" , "asl3-asterisk-dev" : "" , "asl3-asterisk-doc" : "2:20.7.0+asl3-1.3-1.deb12" , "asl3-asterisk-modules" : "2:20.7.0+asl3-1.3-1.deb12" , "asl3-menu" : "1.3-1.deb12" , "asl3-pi-appliance" : "1.5-1.deb12" , "asl3-update-nodelist" : "1.2-4.deb12" , "dahdi" : "1:3.1.0-2" , "dahdi-dkms" : "1:3.3.0-5+asl" , "dahdi-linux" : "1:3.3.0-5+asl" , "dahdi-source" : ""  }]
```

Do you see the issue with the "anonymous"part? Yep. They are collecting your Node numbers from your system. Sure they are putting a Unique random GUID in the payload, but your node numbers are in the payload too. Those numbers can then be used to trace back to who owns the system AND not only that.... the data is being posted online! Yep. And it shows all the needed and good information about your system like kernel version, os version and many other things that a bad actor would love to have to get another machine on their bot network. And the kicker. No option is provided to opt out of this data collection. I had to dig into through some code to figure out how the data was being sent to figure out how to disable it myself.

Now, of course, we all know you need to make sure to secure your own system as best you can. Have a firewall, don't expose too much of it to the world, only what is needed for connections and other things. But this kind of really pisses me off. Granted, in the grander scheme of life, this data collection is not something that is sending REALLY important information about anything, like passwords or bank accounts. What I am upset about is the principle of the idea.

What upsets me about this is:
* a ) it was just a short blurb about it, buried in the documentation
* 2 ) The claim that it is anonymous data is very much false
* c ) the fact that there is no way offered to opt out of this data collection

Yes many other pieces of software collect this kind of data. But in most all cases, you have the ability to opt out of it. If you don't there are other ways to block that data from being sent home. But in this case, there is a simple way to do this. 

And now we get to the crux of this post. I am offering a way for you to opt out of sending this data.

# System Description

Since I only have one x86 box that I can use for ASL and it is currently running my production ASL system, I installed ASL3 to a Raspberry Pi. So the platform I used for this testing is:

- Raspberry Pi 3B+
- Raspbian 64 bit OS (basically Debian 12 Bookworm)
- 16 GB SD Card
- ASL3 installed "bare metal" (IE directly to the card, not running their VM appliance)

Along with other various pieces of software to ssh, burn the card, etc.

# Disclaimer

You modify any system at your own risk. If something goes sideways, I am not responsible for it. This is here for informational purposes, so make sure to verify things as you go.

# Pre-Process

Once I had all the components together, I burnt the most recent Raspbian OS to the card, stuck it in the Pi and let it boot, updated the OS and then Installed ASL3. I followed the instructions in their documentaion to install on Debian and those instructions were pretty simple to follow.

Once I had ASL3 installed, I walked through the configuration process that they recommend, which is using the asl-menu script. For basic configuration, it works well. For me, it's too basic. Since I can edit the conf files directly, I will do that on my next install. I have some more advanced things that I do with my system, so configuring it myself is not that big a deal. For regular Joe Ham who doesn't want to take the time to learn Linux, the asl-menu is perfectly fine.

Once I had everything configured and rebooted and had the system up, I tested to make sure it was connecting to the ASL network and I could send and receive connections. That confirmed I had an operational system. Then it was time to break it...

Note that you can do the process below before you setup the connection to the ASL network. That insures that none of your data is sent once you have configured your connection.

# Opting out

I will not bore you with the process I took to find out how it was sending the data, but needless to say, I finally found the script and service files that are run to send the data.

Btw the script if you are looking for it is called ```asl-telemetry``` in the ```/usr/bin``` directory. Take a gander at it. It's actually pretty slick on what they did.

I started looking at the script, but found how they were validating some things and decided it would be to much work to rip out the code and change it. It is just easier to disable the script.

So. To do that. There are two files in the ```/usr/lib/systemd/system``` directory called ```asl-telemetry.service``` and ```asl-telemetry.timer```. We need to make these go away.

Create a new folder to hold the service files in, incase you would want to go back to sending data at some point:

```bash
mkdir asl_telem_service_files
```

Next we are going to move the files from the ```systemd/system``` directory to our new directory

```bash
sudo cp /usr/lib/systemd/system/asl* asl_telem_service_files
sudo rm /usr/lib/systemd/system/asl*
```

Now check to make sure they are gone and in the new directory:

```bash
ls /usr/lib/systemd/system/asl*
ls asl_telem_service_files
```

On my installation, these were the only two files in the systemd directory that started with ```asl``` (I checked first) so I was able to use the wild card to move them all at once. You may want to check to make sure this is the case on your system as well.

# Wrap Up

And there you have it. With the 2 service files moved out of the systemd folder, systemd will not be able to fire the script and your data will not be sent to the servers.

Yes, I understand that this all seems petty, but the principle of it is what is important. I strive to make sure my data and information is only shared with places that I want it shared, and I have strived to make sure that I am in control of that data. I choose not to use services like Google (gmail) or Yahoo for example so they don't have my data. In this case, the fact it was buried and that the anonymous part of it was false really kind of torqued me. So yes I decied to find a way to make sure that I am in control of the data. 

This will not effect the functioanlity of your ASL system. It will just allow you to opt out of "anonymous" data collection. Which is a privlege that everyone has to keep their data private.

Till we meet again......