---
layout: post
title: "Connecting to AllStarLink with 3CX PBX"
tags: Ham-Radio
categories: Ham-Radio
---

## Introduction

I have had a PBX at home for a while now for various uses and experimentation. Up till a few weeks ago, I was using FreePBX/Asterisk as the base for my PBX. But I wanted to try something new and I had heard about 3CX a while ago and I know that there are alot of people who are using it. I decided to explore this new system and just see what it had. I have been pleaseantly surprised with it's capablities. However, diving into 3CX is a completely different topic. In this article, we will be looking at connecting 3CX to your AllStarLink System.

Keep in mind that 3CX only works using SIP as it is not an Asterisk based PBX system. Yet most connections to AllStar involve using an IAX based connection. If you remember from reading a previous article that I wrote about [Setting Up AllStar for a SIP Connection](https://n8acl.github.io/ham-radio/2022/06/16/sip-connection-to-allstar.html), we walked through setting up AllStar to accept a connection from a SIP Client. Well, let's use that same connection, but instead of using it for a client like Zoiper or Groundwire, we are going to use 3CX PBX to connect.

### Assumptions

For this article we assume:
* You have a working ASL system already setup and already configured as in the article [Setting Up AllStar for a SIP Connection](https://n8acl.github.io/ham-radio/2022/06/16/sip-connection-to-allstar.html).
* You already have a 3CX PBX setup and working and have a client phone connected to it.

### Configure 3CX SIP Trunk

We are going to use the credentials that we setup on AllStar as the provider for a 3CX SIP Trunk.

* Log into your 3CX PBX Management Console.
* Click on ```SIP Trunks``` in the left hand column.
* Click ```Add SIP Trunk```.
* Select ```Generic``` in the ```Select Country``` dropdown.
* Then select ```Asterisk``` in the ```Select Provider in your Country``` dropdown.
* Enter the extension number that you configured in the ```sip.conf``` file in AllStar in the ```Main Trunk No``` field. (In the example article, we used extension 2000.)
  * Note that if you setup an extension number shorter then 4 digits in the AllStar SIP Config, you will need to adjust it up to 4 digits. 3CX does not allow you to use a number less then 4 digits.
* Click ```Ok```. This should drop you into the edit screen for the trunk.
* Now give it a name in the ```Enter name for Trunk``` field (Suggested "Allstar Nodes")
* In the ```Registrar/Server/Gateway Hostname or IP``` field, enter the IP Address of your Allstar System box. In the box at the end of that line, you will need to put the port in.
  * If you have the port on Allstar set as 5060, then leave this as the default. If not, you will need to change it to the port number you defined in the ```sip.conf``` file in Allstar.
* Scroll down to the ```Authentication``` section
* In the ```Type of Authentication``` drop down select ```Register/Account based```
* Put the extension (username) in the ```Authentication ID (aka SIP User ID)``` field. (In the example article we used 2000, the same as the extension number).
* Put the password you defined in the ```sip.conf``` file in the ```Authentication Password``` field.
* Leave the rest as default.
* Click ```OK``` at the top to save the settings. Wait a minute or two for the trunk to register. It should register, unless you entered something wrong. If it does not register, go back and check all your settings.

### Configure 3CX Outbound Rule

Once we have the Trunk setup and registered correctly, now all we need is an outbound route. Since we are just using the PBX as a jumping off point for the 3CX app on our phone or a Hard Phone on our desk, we do not need an inbound route. If you were going to setup an autopatch from AllStar, then you would need an inbound rule. But for this article, we are not doing that.

* Click on ```Outbound Rules``` on the left hand menu.
* Click ```Add```.
* Give it a name in ```Rule Name``` (Suggested: "Allstar Outbound")
* In the ```Calls to numbers starting with prefix``` give it a prefix for dialing. This would be a number that the PBX would see at the beginning of the dialed number that it would know to use this new route. I used 7 for mine. If you know of or remember the old "Dial 9 for an outside line" thing in offices, this is the same thing. This way you don't accidently dial your Allstar Node if you are trying to make a different call.
* Scroll Down to the ```Make outbound calls on``` Section.
* For ```Route 1``` select the Allstar Trunk we configured above. 
* In the ```Strip Digits``` dropdown, choose the number 1. This will strip off the prefix we defined above and then send the rest of the number dialed down the "trunk".
* Click ```OK``` at the top to save your settings.

### Test Our New Trunk

Now we need to test our new trunk. Grab your Cellphone/Tablet that has the 3CX app already configured or grab your HardPhone and pick it up.

Dial one of your Allstar Nodes as ```prefix+node number```. So using our example from above, you would dial "712345" and hit send.

If you have your node configured to anounce your connection, it should play the announcement and you are now on your node. Now you should be able to connect your node to other ASL nodes just like before. 

Now any phone on your 3CX PBX can call your Allstar System and use the radio. 

Note that you may want to restrict the Trunk to only extensions that you or another Licensed Ham in the house use in order to keep someone from accidently dialing your Allstar node. If you are the only one using it, then don't worry.

## Wrap Up

And there you have it. You now have 3CX setup properly for using it with Allstar. Now just like the folks using FreePBX/Asterisk, you too can have one app to use to call your Allstar Nodes.

This gives you alot of flexibility when out and about and way from home. Since 3CX allows you to connect from outside the house, you can be connected all the time if needed. 