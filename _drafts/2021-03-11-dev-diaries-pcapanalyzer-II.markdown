---
layout: post
title: "Dev Diaries - PCAPAnalyzer II: File rotation is kind of ugly"
date: 2021-03-11 10:00:00 +0100
category: dev diaries
permalink: /blog/:categories/:title/
description: I've never really built a backend before

---

Async programming is a concept I never got to learn when I had to do this project.

The word "backend" to me speaks like I am out here building an actual tool. One of those concepts you hear in actual open source projects that we all use literally every day. "frontend", "backend", "stack"... stuff apparently you should have for "scalability" or something. And thanks to this project, I got to learn how important, and how complicated, a good backend architecture is about. You build it *well*, but you build it *once*.

I, unkowingly, did not follow neither of those words. But I would find out quite later, so we'll address that in a future post.

So, let's remember what we dedided we needed for this application backend.

We wanted to capture interface communication data on a machine with industry standard tooling. This task was accomplished with `tshark`, the incredibly versatile command line interface for `wireshark`, the absolute standard for network data analysis. There were other considerations, like `tcpdump`, but I've always seen that one as a "quick and dirty" tool for a basic live monitoring job, no analysis. Both of them use the `libpcap` standard library under the hood anyways, so it's up to interface usage.

Next, we need to store that big amount of data in one single place, and we need to retrieve it fast. Here enters `mysql`, the biggest relational database management system out there by far. Or, at least, its open source counterpart `mariadb` because there will be no Oracle licensing in this house. Not on my watch, at the very least.

Unfortunately, both of these use their own binary file writing standards, incompatible with each other, because sometimes you decide to prioritize internal reading optimization at the cost of application interoperability, but this is not for us. So, we need `csv`'s. We generate these `csv`'s with the programming language I am the most comfortable with using thanks to my classes, `python3`. Human-readable, me-writable!

Okay, so, tools chosen, next it's infrastructure time. First, the easy part: copying (and slightly modifying) a `tcpdump` command for continuous `.pcap` generation on a folder. That one was easy. Then, move the completed ones to a new directory with some `bash script` dark magic. Slightly less easy, but `man` and StackOveflow were the usual lifesavers, so it got to a working state soon enough. script it in python with some help of importing the `os` library for command execution and filesystem management, and the first part was set!

Second step: convert these `.pcap` files into a universally readable `csv` file. After some DuckDuckGo-ing I got `tshark` to read these files into rows of text separated by commas, each one being one packet from the system. Then I noticed that this was gonna be a bit harder, because this wasn't enough of a challenge and it was getting boring, right? Well, apparently, someone over at the god forsaken lands of Wireshark development space decided that who needs ISO compliant date formats? What for? not like interoperability is important, right? especially who would need well formatted *dates*, eh?

Well, thankfully, someone on that team has earned their angel wings by force of feature implementation with something called `epoch time`, a format that counts the seconds that have passed since January 1st, 1970 (does this date ring a bell? I'm sure you've seen it when some time error happens), universally used as standard time on UTC, so we'll use that and be off the races. One epoch to datetime conversion and some string joining on the script, and we write it off to our next step!

Step three was kind of weird. See, `mysql` is an Oracle product. So the one to one compatibility needs are transported to `mariadb`. This means we have a really good database connection driver written for *java*. I am not using java. After some investigation I quickly found out that on our case, library choice is a bit more of a mess. There's `mysql.connector`, `MySQLdb`... everyone claims to be the standard and better than everyone else. [Relevant xkcd](https://xkcd.com/927/). But anyways, I ended up with `MySQLdb` just to choose one of the competent ones, and get doing. It was simple enough to read a file for lines, and run an `INSERT` query for every entry into the database.

Yes, I know. We'll talk about that on the mistakes post. Good thing I had a database class teacher near me!

So, apparently, this was done! a bit more tweaking in scripting times on the main script launcher, and a lot more tweaking to maken actually valid `csv` for edge cases, and the backend was working! Job well done! (writer's note: I did not know about the fact that I had enough material for a mistakes blogpost back at that time.)

So I had a backend! it fully worked, and I could `SELECT` away all my network packets away however big the thing was!

But a backend is that. *back*. Like the back of a refrigerator, it's pretty damn ugly to look at if you don't know what you're working on. So, on the next blogpost: frontend! we get to see why I am even less of a designer than I am a developer! responsive can't be that hard of a task, can it?

We'll see how that train of thought ended as the inevitable rail accident it was meant to be.
