---
layout: post
title:  "Devops insights from getting your cable tv to work"
date:   2017-02-20 11:14:49 +0200
categories: devops learn tips troubleshooting
---

## Background

> I will not fix your computer

Says the [ThinkGeek t-shirt](http://www.thinkgeek.com/product/388b/)

Us devops dudes are required to fix all the computer issues for the extended family, and quite rightly, as we are most qualified to figure out what has gone wrong on the PC, home network, or excel sheet. Well, not the excel sheet, but you get the idea. 

Sarcasm aside, I recently encountered issues with our home Internet-based cable TV provider, and resolving these issues brought me to write this post.

Recently one of the Israeli cellular providers, namely Cellcom, has branched into Home TV providing, they have an interesting solution that is based on a combination of DVB-T and Internet connectivity.

My network setup at home is based on a ADSL connection, a router with Wifi for the ground floor, a connection to the top floor through power line adapters. 

On the bottom floor the Cellcom box was connected directly to the ADSL router and on the top floor the second Cellcom box is connected to a router that is connected to the power line adapter.

One more important piece of information is that the telephone line over which the ADSL is carried on the bottom floor was added after we moved into the house, and is a goes through a few junction boxes from the main telephony switch at the house entrance.

At first both Cellcom boxes worked well. The VOD feature worked fine on the box downstairs near the ADSL router and on the one upstairs. 

At some stage, the top floor cellcom box stopped working well over the Internet, the DVB-T reception remained, and the Internet connection test, on the box, showed Good connectivity.

## Getting support

I called Cellcom support, to their credit they answer pretty quickly and are very obliging. The tech-support guy claimed my Internet connection was under par. Cellcom requires we have a 40Mbps connection at least - we have a 100Mbps connection. After a bit of negotiation I was convinced to take the problematic Cellcom box downstairs and switch it with the one that was working. 

At this stage, the tech support escalated the issue and called in his supervisor. 

On my side I realized two things. 

* The box originally from the top floor was not operating well, i.e. no VOD reception, while the other had VOD working on it, with no problems
* The Cellcom tech observation that our Internet connection was under par was correct, running a speed test on a computer connected directly to the ADSL router was showing 16Mbps, when I was expecting something closer to the 100Mbps we are paying for.

## Solutions

After reconnecting with Cellcom support, this time with the a more senior tech, we managed to reset the connection of the 2nd Cellcom box. To be frank, I'm not sure what was done, if it was a reset on the server side, or a remote reset on the client side, but either way, the problem was resolved. Meaning the top-floor box was now showing VOD content as well as the bottom-floor box.
First problem solved.

I was still getting poor Internet connectivity, and as there are more end users at home, as you might imagine, mainly my boys playing endlessly on Minecraft, that needed to be resolved as well.

If you recall, I mentioned above that the actual phone line on the ground floor was a relatively recent addition, so I thought of switching the ADSL router and second router. The top floor had a telephony socket that seemed better connected, so I moved the ADSL router there, and through the same power line adapters connected it downstairs to the ground-floor router. 

Running speed-tests on computers connected to both routers yielded much better speeds and reception in all the house is now has much better Internet connectivity. 

## Insights

So what does all of this have to do with Devops?

Troubleshooting takes a major role in getting the job done in our field. Figuring out what is the problem can be tricky, and doing it right leads to a swifter solution.

**Get support**

There is a lot to know in our field, and asking for help is part of the game, in recent tasks I've been asked to resolve I approached tech support of AWS, Atlassian and many more. Their answers were always helpful and got me to solve the issues swiftly 

**Don't Assume**

There is the famous saying:

> When you assume you make an *Ass* of *U* and *Me*

Base your conclusions on your findings, not your gut feelings. 
Many years back, an senior engineer told me the 90% of Linux issues are those of Permissions and Ownership. Same goes to network connectivity issues in AWS, you'd best start looking for solutions in the Security Groups. These insights are all good and well, and come following a lot of hard earned experience, but still, keep your mind open. 

In this case, there was one symptom, i.e. the VOD feature not working, knowing the Internet connection was poor I could assume this was the issue, but the underlying circumstances were not identical, which brings me to the second insight:

**A/B tests**

Only after bringing the box to the same position in the home network was I able to say to the Cellcom rep:

> Look, box one working, box two not working

A works
B doesn't

When you are troubleshooting an issue, find out when the issue arose, check if you can run the system *without* the issue - and see what changed.

**Follow through**

Any Tennis coach will tell you that you need to follow through on your swing. Same here, so you started of with one symptom (no VOD) and discovered another underlying issue (poor Internet connectivity)? 
Resolve them both, don't leave an open issue dangling.
