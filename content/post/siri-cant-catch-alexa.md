---
title: Siri Won't Catch Alexa
author: Jason P. Becker
date: 2017-01-14
tags: 
---

> "I just got an Amazon Echo and surprisingly, I really love it."

In one form or another, [this story](https://www.macstories.net/reviews/astra-brings-amazons-alexa-voice-assistant-to-the-iphone/) has repeated [again](https://sixcolors.com/post/2015/12/my-favorite-gadget-of-2015-the-amazon-echo/) and [again](https://twitter.com/marcoarment/status/715611484567511040) across the internet. So while the recent headline seemed to be ["Amazon's Alexa is everywhere at CES 2017"](http://www.theverge.com/ces/2017/1/4/14169550/amazon-alexa-so-many-things-at-ces-2017), it really feels like this year was the Amazon Alexa year.

I have an Amazon Echo. I bought around a year ago during a sale as the buzz seemed to have peaked [^peaked]. My experience with the Amazon Echo has mostly been "I don't get it." 

{{< tweet 736174308129705985 >}}
{{< tweet 747087351084683264 >}}
{{< tweet 747087927247777793 >}}

The Echo was kind of fun with Philip's Hue lights or for a timer or unit conversion in the kitchen from time to time, but not much else. I was not much of a Siri user, and it turned out I was not much of an Amazon Echo user.

But I just bought and Echo Dot.

## HomeKit and Siri Can't Compete

Siri and HomeKit should be a match made in heaven, but if I say "Hey Siri, turn off the bedroom lights," the most common response is "Sorry, I cannot find device 'bedroom lights'." I then repeat the same command and the lights go off. This literally has happened once in almost a year of Echo ownership, but it happened nearly every time with Siri.

Apple is miserable at sharing [^sharing], and that means that even if Siri worked perfectly, HomeKit is built on a bad model.

My Philips Hue lights are HomeKit compatible. I use Family Sharing so that my partner Elsa can have access to all the apps I buy. Yet it took months to get the email to invite her to my home to send and work. And really, why should I have to invite someone to anything to turn on the lights in my home? Apple knows all about proximity, with it's [awesome use of iBeacons in its own stores](http://appleinsider.com/articles/13/12/06/first-look-using-ibeacon-location-awareness-at-an-apple-store). If being within reach of my light switches or thermostats were enough security to control my devices before, why is Apple making it so hard for people to access them via HomeKit? [^security]

## A Better Model for HomeKit

HomeKit's insistence that all devices have the same security profile and complex approval has meant that devices are rarely HomeKit compatible while everything is compatible with Alexa's simple skills program.

Imagine if the HomeKit app and excellent Control Center interface
[^controlcenter] existed at the launch of HomeKit. Imagine if instead of a complex encryption setup that required new hardware, Apple had tiered security requirements, where door locks and video surveillance lay on one side with heavy security, but lights and thermostats lay on the other. Imagine if HomeKit sharing was a simple check box interface that the primary account with Family Sharing could click on and off. Imagine if controlling low security profile devices with Siri worked for anyone within a certain proximity using iBeacons.

This is a world where Apple's Phil Schiller [is right](https://backchannel.com/phil-schiller-on-iphones-launch-how-it-changed-apple-and-why-it-will-keep-going-for-50-years-e4412ad2c8f5#.8jw6uh7fe) that "having my iPhone with me" could provide a better experience than a device designed to live in one place.

That's not the product we have. And even if Apple gets into the "tower with microphones plugged into a wall" game, I don't see them producing an Echo Dot like product that makes sure your voice assistant is everywhere. My iPhone might be with me. I may be able to turn on and off the lights with my voice. But without something like iBeacons in the picture, if someone comes to stay in my guest room they're back to using the switch on the wall. If a family member uses Android, they are out of luck. If I have a child under the age where a cell phone is appropriate, they are back to living in a dumb home. The inexpensive Echo Dot means you can sprinkle Alexa everywhere the devices you want to control are for anyone to interact with.[^privilege] Apple doesn't do inexpensive well.

I'm not sure they can resolve the product decisions around HomeKit that make it less appealing to hardware manufacturers. Worse, some of Alexa's best skills are entirely software. Alexa's skills can seemingly shoot off requests to API endpoints all over the place. So instead of needing to buy a Logitech Harmony Hub with complex encryption and specialized SiriKit/HomeKit skills, and tight integration, my existing Harmony Hub that has an API used by Logitech's own application supports Alexa skills. An Alexa skill can be built in a day. Apple is allergic to doing simple services like this well, even though the entire web runs on them.

## My Dot

Our new bedroom in Baltimore does not have recessed lighting much like our old bedroom. We're using one of those inexpensive, black tower lamps in the bedroom. I don't have a switch for the outlets in there. Philips doesn't make any Hue bulbs that provides enough light to light the room with one lamp. 

I needed an instant way to get the light on and off. That's when I remembered I had an old WeMo bought from before the days of HomeKit. I used that WeMo to have a simple nightly schedule (turn some lights on at sundown and off at midnight each night) and never really thought about it. The WeMo was perfect, and lo and behold, it works with Alexa. Our Echo is a bit far from the bedroom though, and I don't want to shout across the house to turn off the lights. 

Not only was the inexpensive Echo Dot perfectly for sprinkling a little voice in the bedroom, it also meant our master bathroom can have Hue lights that are controlled with voice again. And, now I have a way to get Spotify onto my Kanto YU5 speakers in the bedroom without fussing with Bluetooth[^dumpsterfire] from my phone by just connecting an 1/8" phono plug in permanently.

Now we say "Alexa, turn on the bedroom light" and "Alexa, play Jazz for Sleep". It's great. It always works. If we had a guest bedroom with the same setup, anyone who stayed there would be able to use it just as easily. No wonder why the Wynn is [putting Amazon Echo in their hotel](http://www.cnbc.com/2016/12/14/wynn-las-vegas-to-add-amazon-alexa-to-all-hotel-rooms.html). Apple literally can't do that.

## Whither Voice Control

Amazon, Apple, and Google seemed locked in a battle over voice [^microsoft]. I can think of four main times I want to use voice:

1. Walking with headphones
2. Driving in the car
3. Cooking in the kitchen
4. Sharing an interface to shared devices

For Phil Schiller, and by extension, Apple, the killer feature of Siri is you always have it. [^watch] In a [recent interview](https://backchannel.com/phil-schiller-on-iphones-launch-how-it-changed-apple-and-why-it-will-keep-going-for-50-years-e4412ad2c8f5#.8jw6uh7fe), Schiller is quoted as saying:

> Personally, I still think the best intelligent assistant is the one that’s with you all the time. Having my iPhone with me as the thing I speak to is better than something stuck in my kitchen or on a wall somewhere.

Apple AirPods are all about maximizing (1). Siri works pretty well when I'm out with the dogs and need a quick bit of information or to make a call. But the truth is, I don't really need to learn a lot from Siri on those dog walks. When I'm out walking, Siri is better than pulling out my phone, but once I've got a podcast or music going I don't really need anything from Siri in those moments.

Driving is another great context for voice control. I can't look much at a screen and shouldn't anyway. Ignoring CarPlay, Apple's real move here is Bluetooth interfaces, which places Siri in most cars. But again, what is my real use for voice control in this scenario? Reading SMS and iMessages makes for a cool demo, but not really something I need. Getting directions to a location by name is probably the best use here, but Apple's location database is shit for this. Plus, most of the time I choose where I am going when I get in the car, when I can still use my screen and would prefer to. The most important use of voice control in the car is calling a contact, which Voice Control, the on-device precursor to Siri, did just fine. [And now Alexa is entering the car space too](http://www.geekwire.com/2016/ford-working-on-amazon-echo-integration-connecting-cars-to-homes/).

While cooking, it is great to be able to set timers, convert measurement units, and change music or podcast while my hands are occupied. This is why so many people place their Amazon Echo in the kitchen-- it works great for these simple task. "Hey Siri" and a Bluetooth speaker is a terrible solution in comparison. In fact, one thing that the Amazon Echo has done is cause me to wear my headphones less while cooking or doing the dishes, since the Amazon Echo works better and doesn't mean I can't hear Elsa in the other room. This isn't a killer feature though. Early adopters may be all about the $180 kitchen timer with a modest speaker, but the Echo won't be a mass product if this is its best value proposition.

## Shared Interface to Shared Devices

There is a reason why home automation is where the Echo shines. Our homes are full of technology: light switches, appliances, televisions and all the boxes that plug in them, and everyone who enters the home has cause to control them. We expect that basically all home technology is reasonably transparent to anyone inside. Everyone knows how to lock and unlock doors, turn on the TV, turn on and off lights, or adjust the thermostat. Home automation has long been a geek honey pot that angers every cohabitant and visitor, but voice control offers an interface as easy and common as a light switch.

Home automation is the early adopter use case that reveals how and why voice control is a compelling user interface. Turning on the bedroom lights means saying "Alexa, turn on the bedroom lights." There is no pause for Siri to be listening. There is no taking out my phone or lifting up my watch. There is no invite process. There is no worrying about guests being confused. Anyone inside my home has always been able to hit a light switch. Anyone inside my home can say "Alexa, turn on the living room lights." That's why Apple erred by not having a lower security, proximity based way to control HomeKit devices.

Voice control is great because it provides a shared interface to devices that are shared by multiple people. Computers, smartphones, and really most screen-based interfaces that we use are the aberration, pushing toward and suggesting that technology is becoming more personal. The world we actually live in is filled with artifacts that are communal, and as computer and information technology grow to infuse into the material technologies of our lives, we need communal interfaces. Amazon is well positioned to be a part of this future, but I don't think Apple has a shot with its existing product strategy.


[^peaked]: It hadn't.

[^microsoft]: Cortana isn't anywhere that matters, so it doesn't matter yet.

[^sharing]:  We still don't have multi-user iPads or iPhones. I have a new AppleTV, but all the TV app recommendations don't work because two people watch TV. Unlike Netflix, we can't have separate profiles. And the Apple Watch is billed as [their most personal device yet](http://www.apple.com/pr/library/2014/09/09Apple-Unveils-Apple-Watch-Apples-Most-Personal-Device-Ever.html). Where Amazon moves into the world of ever-present, open communal interfaces, Apple is looking toward individual, private worlds.

[^controlcenter]: An unconscionable amount of the time I see "No Response" in Control Center under my devices. Worse, I have to sit and wait for Apple to realize those devices are there because eventually they pop on. Instant interfaces matter, and they matter even more when trying to replace a light switch.

[^dumpsterfire]: Bluetooth is a dumpster fire when you have two phones, two sets of wireless headphones, a Bluetooth speaker in the bathroom, a Bluetooth speaker in the bedroom, and a shared car with Bluetooth. All of these things will choose at various times to ~conveniently~ connect to whatever phone they want if you're not diligent about powering things down all the time. Bluetooth audio is a shit experience.

[^watch]: Apple Watch is about extending Siri wherever you are, but I don't use Siri on my Watch much, because it's not great in any of those four contexts. If I can raise my hand I have hands and I'd rather use my phone.

[^security]: Ok, here comes the critiques about how HomeKit can be used to open door locks or activate video surveillance, etc. Great-- those are cool uses of technology that also have mostly proximity based security but fine, I can see a case for heavy encryption and complex sharing setups for those devices. But the truth is, most of the [internet of things](https://twitter.com/internetofshit) aren't these kinds of devices. A good product would easily scale to different security profiles.

[^privilege]: There's probably a good critique about privilege here, assuming that you have multiple rooms that would need a separate assistant device. But I would like to remind you that we're talking about spending hundreds of dollars on cylinders plugged into walls that you talk to to control things that cost 4x their traditional counterparts. For the foreseeable future, we are addressing a market of rich people and this technology will succeed or fail there long before we get to ubiquity. Plus, who cares what Apple has to say about any of this if we're not talking about rich people? Apple's market is rich people and that isn't going to change. Affordable luxury is the brand and target, where affordable in the global scheme means fairly well off.
