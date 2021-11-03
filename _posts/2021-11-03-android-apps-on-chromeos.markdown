---
layout: post
title:  "Android Apps on ChromeOS"
date:   2021-11-03 12:01:00
categories: blog entry
---
I haven't blogged in a couple of days; why? Mainly because I'm a little bit lazy but also because something exciting
caught my attention. I try to run one of our apps inside ChromeOS; that's not something new since I did the same a couple of years ago.
The thing that is different this time is that I think I have enough experience to achieve the desired performance.

First things first, which is my test device? Well, it's an [HP 14 Chromebook][hp-14-chromebook] with an AMD A4 CPU, which is an x86 CPU; this shouldn't be a problem, but there is something that slows down our app. My current theory is that we are running too many tasks simultaneously, which is causing a bottleneck.

The second potential problem is that this particular model only has 4GB of ram, which can also be another bottleneck since our app is not really memory efficient.

My testing plan is to start profiling the app in the next couple of days to see if I can figure it out.

I'll keep you posted on what I can find out.

[hp-14-chromebook]: https://www.amazon.com/HP-Chromebook-180-degree-14-db0020nr-Chalkboard/dp/B07M8QVNKG