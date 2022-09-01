---
layout: post
title: "Create Auto Do Not Disturb in FreePBX"
tags: ["VOIP-Telephony","Ham-Radio"]
categories: ["VOIP-Telephony","Ham-Radio"]
---

## Introduction

Remember the days before cellphones of having an actual phone in the home connected to the wall and you had to stretch all the cables so you could make phone calls that your parents couldn't hear? Well with the market for liquidated IP phones becoming popular, those days are back. There are so many uses for a VOIP phone system in the house now. Not only can you bring in your own home phone number now and put as many extensions as you have ports on your switch, you can even tie in your cellphone so you can actually get calls while out and about away from home. Another cool feature you can have is room to room intercom or even whole house paging. Dinner ready? Pick up the phone, activate the intercom function and you can let everyone one know it's time to eat.

There are so many uses even out side of that. Use in the Ham Radio space has become very popular of the last few years. There are multiple services now being run by many volunteers strictly for Hams and by using your own PBX at home, you can tie not only any incoming commercial VOIP systems into your PBX, you can also tie these Ham Radio ones into it as well. This lets you pick up your extension on your desk and make a call to work, make a call to your family or even make a call to a fellow Ham Radio Operator all with the push of a button and you don't need multiple lines to do it. 

But what if you don't want your phone ringing off the hook all the time. If you work from home for example, it's completely understandable if you don't want one of your Ham Radio buddies being able to call you, or you don't want Aunt Matilda the Gossip to call you at 11:00 at night when you are laying in bed and getting ready to fall asleep or you want to make sure that no work calls come through on the weekends. Instead of having to remember to hit the Do Not Disturb button on all the phones, you can set up FreePBX to automatically route all incoming calls during certain times to voicemail or some other destination. And the cool part about this is you can do it based on where the call is coming from so you can basically disbale all Ham Radio calls during work but allow work calls and home calls through. I call this "Auto Do Not disturb" (herein after referred to as AutoDND).

Let's take a look at how to do it on FreePBX.

## Skills needed

For this Project you will need:
* to have a FreePBX system setup and configured 
* at least one incoming trunk connection
* at least one inbound route configured for that trunk connection
* at least one extension with voicemail box configured (this could even be a virtual extension, as long as it has voicemail enabled or it could be a Ring Group).

Also a Disclaimer: You modify your FreePBX setup at your own risk. If something goes sideways, it's not my fault. 

### Install Needed Modules in FreePBX

In order to do this, we need some modules installed in FreePBX to add this functionality we are talking about. Depending on your setup or the distro spin of FreePBX you are using, you may already have the needed modules installed, may have some of them, or have none of them. In any case, you will need to install them if you don't.

In order to install these modules, you will need to log into your FreePBX admin GUI and click on ```Admin``` and then ```Module Admin```.

You will then need to install the following modules, in the following order (some of these modules depend on others being there before they can be installed):

* Calendar and CEL
* Time Conditions

### Check Server Time

For this to work for you, you will need to double check to make sure that your server time is set correct for your location. This will make sure that things are compared to the correct time for your time zone and also make sure that it follows Daylight Savings Time.

* Click on ```Settings```
* Click on ```Advanced Settings```
* Find ```PHP Timezone```
* Set this to your correct time zone (for me it is ```America/New_York``` for example)

### Setup Time Groups

After getting all of these new modules installed and verified your server time zone is set, you will have two new selections under the ```Applications``` menu: ```Time Groups``` and ```Time Conditions```.

First we need to setup a new ```Time Group``` for our AutoDND.

* Click ```Applications```
* Select ```Time Groups```
* Click on ```Add Time Group```

First we need to give it a description so that we know what this is for. I would suggest ```AutoDND``` since that is what we are using it for.

Now comes thr tricky part. How do we set up the time groups properly. This tripped me up at first trying to figure out how this worked. This is not like setting a recurring event in a calendar app. So if you set from say 12:00 PM - 3:00 PM but you want it every day, if you try to set it for Sunday to Saturday, it won't work.

So what you will need to do is set it for every 24 hour period you want within a week. And you can set a whole week in one Time Group... Let me give you an example:

I want my AutoDND to be in effect from 10:00 PM at night to 5:00 PM the next evening. Basically, I only want to receive calls between 5:00 PM and 10:00 PM each day. SO for my first time group, I set:

```Time to Start```: 22:00<br/>
```Time to FInish```: 17:00<br/>
```Week Day Start```: Monday<br/>
```Week Day End```: Tuesday<br/>

So from 10:00 PM Monday night to 5:00 PM Tuesday evening, this would be in effect. Now I can hit ```Add Time``` near the bottom and do the same thing for Tuesday:

```Time to Start```: 22:00<br/>
```Time to FInish```: 17:00<br/>
```Week Day Start```: Tuesday<br/>
```Week Day End```: Wednesday<br/>

And so on for every day of the week. If you want a different set of times for the weekend, you can change it... for example:

```Time to Start```: 22:00<br/>
```Time to FInish```: 07:00<br/>
```Week Day Start```: Friday<br/>
```Week Day End```: Saturday<br/>

```Time to Start```: 22:00<br/>
```Time to FInish```: 07:00<br/>
```Week Day Start```: Saturday<br/>
```Week Day End```: Sunday<br/>

and so on. Once you have your week setup click ```Submit``` at the bottom and then ```Apply Changes``` at the top.

### Configure Time Conditions

Now we need to configure a ```Time Condition```. This is what we will have FreePBX do the comparing with the ```Time Group``` we just setup. This is where the logic happens as this is where we point the destination of the inbound routes to.

* Click ```Applications```
* Click ```Time Conditions```
* Click ```Add Time Condition```

Now we need to add our logic for the ```Time Condition``` to know how to route the call for us.

Set the following fields at a minimum:

* ```Time Condition name```: This is the name/description of the Time Condition. I suggest ```AutoDND``` since that is what we are using it for.
* ```Time Zone```: Set this to your local time zone. For me it is ```America/New_York```
* ```Time Group```: Select the AutoDND group we created above
* ```Destination Matches```: So this is the place where we send this call if the current time of the incoming call is within the Time Group. In other words, if the incoming call is within the time frame we set for the Time Groups, send the call here. For me I set ```Voicemail``` and then the ```Unavailable Message``` for my main phone extension.
* ```Destination non-matches```: If the incoming call is outside the times we sent, send it here. For me I chose ```Extension``` and my main extension.

When this is done, click ```Submit``` and then ```Apply Changes```.

### Configure Inbound Route 

Last but not least, we need to set the Inbound Routes that we want to use this for. 

* Click ```Connectivity```
* Click ```Inbound Routes```
  
Now edit the Inbound Route you want to set this for.

Scroll down to ```Set Destination```, click the drop down and select ```Time Conditions``` and then ```AutoDND``` (or whatever you called it).

Now click ```Submit``` and then ```Apply Changes```.

Repeat this for each Inbound Route you want this to be used for.

## Wrap up

And there you have it. Now when Aunt Sally tries to call at Midnight to tell you about Uncle Walt's Gout flaring up, she will be sent to voicemail and you can sleep easy knowing you won't be awakened.

Just some final thoughts:
* The logic I used to setup my time groups may seem backwards. You could reverse it and say set the times from 5 - 10 at night and then in the Time Conditions set the Destination Matches to your extension and non-matches to voicemail. This would work as well. 
* You can also use a Ring Group instead of a specific extension for the final destination as well. This would let you set multiple extensions to be rung instead of just one. I have just one extension setup on my personal PBX (my cellphone) so I don't need a Ring Group.
* You could even route to different extensions based on time as well. For example, during work hours, send your work number to your desk phone, but outside of hours, to your cellphone.

Enjoy! Till next time!
