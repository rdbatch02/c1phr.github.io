---
layout: post
title: Programmer's Block
comments: true
---
I've run into a wall. Over the past two weeks, I've been trying to finish up the data access layer for my app so I can start working on actually making something I can see and touch. 

For the past two weeks, I've been nose deep in documentation and tutorials on Core Data and Alamofire (a slick Swift HTTP library) and my nights have been spent making sure I can get content from a server into a user's device. I've frustrated myself over the asynchronous details of how HTTP requests happen on iOS and tried to comprehend how to setup a callback to process my data (actually, it only took about an hour after reading the documentation, I should have started there). After two weeks of getting discouraged, I finally understand how these little details work together and I feel a little bit more comfortable programming for iOS.

But now I have a new problem... _what's next?_

After spending all of that time swimming in the implementation details of my app that by the time I came up for air, I had entirely lost sight of what I was setting out to build.

Yes, I've started using tools like [Trello](http://trello.com) to keep organized, and for the most part that's worked wonders on keeping me on track. The problem with organization tools though, is they only work as well as the end goal is defined. For this period of time that I was working on implementing my system, the end goal was just to move some data from one place to another. Now that I've tried re-aligning my end goal with my original plans, it seems that even Trello can't help me.

My initial idea seemed simple: build an application that I would want to use to interact with fellow fans at a sporting event (starting with hockey, of course). This seemed like a great idea to just start running with and adding features as I go. I wanted to start simple, so I worked on gathering arena and team data from various sources and putting a database and API together. At the time, these just seemed like useful pieces of data. As I started working, I focused on this data being the most important part of the app, without ever stopping to think about how I would use this data. 

_This was my first problem._

Focusing on the data just in front of my nose made me blind to what other things I might need to work with. After all, I built an API around this data, and I wrote some Python to scrape and gather this data, what else could I possibly need? When I started looking at my idea again and trying to figure out what to do next, I had a total rejection of everything that I had worked on. I couldn't see where my work and my vision were even on the same track. Team and arena data really had nothing to do with social interaction, so why haven't I built anything related to social interaction yet? Have I just wasted the last two weeks building something useless (of course not, I learned something, but I didn't think about this at the time)? 

_This was my second problem._

With these combined, I had a recipie for what I _want_ to call "programmer's block". What I've been experiencing is actually more similar to writer's block than [some people would define programmer's block](http://programmers.stackexchange.com/questions/34867/is-there-such-a-thing-as-programmers-block), because ultimately I've run into a lack of inspiration. I got to this strange point where every time I tried to think about my original idea, I was finding problems with it that were based around not knowing how to proceed based on what I had built initially.

And then I went to a baseball game.

Major League Baseball has an [app dedicated to the ballpark experience](http://mlb.mlb.com/mobile/ballpark/) and being the tech nerd that I am, I downloaded it before taking a trip to Chase Field yesterday. Honestly, I forgot all about it until my phone decided to befriend an [iBeacon](https://en.wikipedia.org/wiki/IBeacon) just after passing through the gate and I was prompted to check into the game to unlock some additional features in the app. While I didn't choose to check in and unlock the ability to upgrade to better seats, this app re-inspired what I was hoping to design for a hockey setting.

No, it's not a clone. The Ballpark app is actually missing the major features that I want to see in my app, and I don't think that I would be able to offer most of the same services that the Ballpark app can (such as ticket upgrades and in-seat food ordering) given that it's first-party. But on the very first screen of the app I, as a new user, was greated with a list of **teams** and asked to choose my favorites. After choosing a team, I was shown their **arena** (well, ballpark), and everything came full-circle again. I started to identify where my features could find their home among these bits of data that I had gathered, and ultimately, I regained sight of my goal.

Now, before I open XCode again, I think that I need to make sure this doesn't happen again. My next steps are to do a better job at fleshing out what my overall idea is in Trello (and focus on organizing the implementation details later) and I'm finally at a point where I think that [prototyping](http://www.appcooker.com) might help me keep my head up and eyes forward. Given that I've never prototyped beyond a piece of paper, this will be an interesting experience, and I look forward to pushing this entire learning experience along.