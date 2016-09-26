---
title: Linear Thinking
author: Jason P. Becker
email: jason.paul.becker@gmail.com
date: 2013-06-02
tags: apple, ios7, osx, interface
---

Apple will be revealing new details for both of its major operating systems at WWDC on June 10, 2013. The focus of much speculation has been how Apple will improve multi-tasking and inter-app communication in iOS7. As batteries have grown, CPUs have become increasingly powerful, and the application [ecosystem][] has matured [^viticci], the iOS sandboxing model has felt increasingly limiting and outdated.

I think that there is a simple change that could dramatically increase the effectiveness of multitasking on iOS by re-examining how application switching works.

Scrolling through a long list of applications, either through the basement shelf or via the four-finger gesture on an iPad, is both slow and lacking in contextual cues. In a simple case where I am working with two applications simultaneously, how do I switch between them? The list of open applications always places the current application in the first position. The previously used application sits in the second position. The first time I want to change to another application this is not so bad. I move to the "right" on the list to progress forward into the next application [^next]. 

The trouble comes when I want to return to where I was just working. The most natural mental model for this switch is to move "left" in the list. I moved "right" to get here and millions of years of evolution has taught me the the "undo button" for moving right is to move left [^inform7]. But of course, when I attempt to move "left", I find no destination [^youcant]. I can pop an application from anywhere on the list, but I can only prepend new applications to the list [^ignorant].

Apple needs to move away from its linear thinking and enter the second dimension.

What if I could drag apps in the switcher on top of each other to make a new stack, not unlike the option on the OS X launch bar? Throughout this stack, position is maintained regardless of which application is in use/was last used. I can always move **up** from Chrome to ByWord, and **down** to Good Reader, for example, if I was writing a report. Apple might call this a Stack, mirroring the term in OSX, but I would prefer this to be called a *Flow*.

The goal of this feature is to organize a *Flow* for a particular task that requires using multiple apps. One feature might be saving a "Flow", this way each time I want to write a blog post, I tap the same home screen button and the same four apps in the same order launch in a *Flow*, ready for easy switching using the familiar four-finger swipe gesture up and down. I no longer have to worry about the sequence I have recently accessed applications which is confusing and requires me to look at the app switcher draw or needlessly and repeatedly swipe through applications. I never have to worry about lingering too long on one application while swiping through and switching to that app, changing my position to the origin of the list and starting over again. 

For all the calls for complex inter-app communication or having multiple apps active on the screen at the same time, it seems a simple interface change to application switching could complete change the way we multitask on iOS.

[ecosystem]: |filename|frictionless.md

[^viticci]: And Federico Viticci has either shown us the light or gone completely mad.

[^next]: For now, lets assume the right application is next in the stack. I'll get to that issue with my second change.

[^inform7]: `You are in a green room. > Go east. You are in a red room. > Go west. You are in a green room.`

[^youcant]: `You can't go that way.`

[^ignorant]: I don't know enough about data structures yet to name what's going on here. I am tempted to think that the challenge is they have presented a list to users, with a decidedly horizontal metaphor, when they actually have created something more akin to a stack, with a decidedly vertical metaphor. But a stack isn't quite the right way to understand the app switcher. You can "pop" an app from any arbitrary position on the app switcher, but funny enough can only push a new app on to the top of the switcher.