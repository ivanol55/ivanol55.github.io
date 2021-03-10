---
layout: post
title: "Dev Diaries - PCAPAnalyzer II: uncharted territory!"
date: 2021-03-11 10:00:00 +0100
category: dev diaries
permalink: /blog/:categories/:title/
description: I've never really built a backend before

---

The whole concept of asynchronous programming or service development is something I never got to learn on my classes. I did try self-learning, but this really felt like I'd need a tutor. To my surprise, this backend thing ended up working out!

The word "backend" to me speaks like I am out here building an actual tool. One of those concepts you hear in actual open source projects that we all use literally every day. "frontend", "backend", "stack"... stuff apparently you should have for "scalability" or something. And thanks to this project, I got to learn how important, and how complicated, a good backend architecture is about. If you build it *well*, but you build it *once*.

My happy self, Unbenownkst of the actual difficulty of these tasks, did not follow neither of those words while wholeheartedly believing I was building good architecture. But I would find out about this much, much later, so we'll address that in a future post detailing just how much I learnt while rewriting my own bad code.

So, let's remember what we dedided we needed for this application backend.

We wanted to capture interface communication data on a machine with industry standard tooling. This task was accomplished with `tshark`, the incredibly versatile command line interface for `wireshark`, the absolute standard for network data analysis. There were other considerations, like `tcpdump`, but I've always seen that one as a "quick and dirty" tool for a basic live monitoring job, no analysis. Both of them use the `libpcap` standard library under the hood anyways, so it's up to interface usage.

Next, we need to store that big amount of data in one single place, and we need to retrieve it fast. Here enters `mysql`, the biggest relational database management system out there by far. Or, at least, its open source counterpart `mariadb` because there will be no Oracle licensing in this house. Not on my watch, at the very least. Well, I do use VirtualBox, but that kinda doesn't count, because that's not production-designed software. I mean, it's *VirtualBox*. It's the toy equivalent of virtualization software. But, I'm losing track, moving on.

Unfortunately, both of these use their own binary file writing standards, incompatible with each other, because sometimes you decide to prioritize internal reading optimization at the cost of application interoperability, but this is not for us. So, we need `csv`'s. We generate these `csv`'s with the programming language I am the most comfortable with using thanks to my classes, `python3`. Human-readable, me-writable!

Okay, so, tools chosen, next it's infrastructure time. First, the easy part: copying (and slightly modifying) a `tcpdump` command for continuous `.pcap` generation on a folder. That one was easy. Then, move the completed ones to a new directory with some `bash script` dark magic. Slightly less easy, but `man` and StackOveflow were the usual lifesavers, so it got to a working state soon enough. script it in python with some help of importing the `os` library for command execution and filesystem management, and the first part was set!

Second step: convert these `.pcap` files into a universally readable `csv` file. After some DuckDuckGo-ing I got `tshark` to read these files into rows of text separated by commas, each one being one packet from the system. Then I noticed that this was gonna be a bit harder, because this wasn't enough of a challenge and it was getting boring, right? Well, apparently, someone over at the god forsaken lands of Wireshark development space decided that who needs ISO compliant date formats? What for? not like interoperability is important, right? especially who would need well formatted *dates*, eh?

Well, thankfully, someone on that team has earned their angel wings by force of feature implementation with something called `epoch time`, a format that counts the seconds that have passed since January 1st, 1970 (does this date ring a bell? I'm sure you've seen it when some time error happens), universally used as standard time on UTC, so we'll use that and be off the races. One epoch to datetime conversion and some string joining on the script, and we write it off to our next step!

Step three was kind of weird. See, `mysql` is an Oracle product. So the one to one compatibility needs are transported to `mariadb`. This means we have a really good database connection driver written for *java*. I am not using java. After some investigation I quickly found out that on our case, library choice is a bit more of a mess. There's `mysql.connector`, `MySQLdb`... everyone claims to be the standard and better than everyone else. [Relevant xkcd](https://xkcd.com/927/). But anyways, I ended up with `MySQLdb` just to choose one of the competent ones, and get doing. It was simple enough to read a file for lines, and run an `INSERT` query for every entry into the database.

Yes, I know. We'll talk about that on the mistakes post. Good thing I had a database class teacher near me!

now, at this point, I could run through the entire process to make it work, fixing rough edges here ad there. But it was *me* running the scripts on the system. That doesn't sound very backend service-y, does it? So, it was time for the newest kid in my town: process management. This is something that, to me, as a last year vocational studies systems administration student, sounded *spooky*. Fortunatrly, it ended up being way easier than I expected it to be! I just imported the `os` library, tried simply running the scripts as subprocesses of this `start.py` script, and after some playing around with relative paths (the `os` library does some weird things to relative paths depending on where you run it) and it was working!

Next was to control it from the system, because I can't go around putting "just kill `start.py`" on the documentation. I had never written a custom service file. Hell, I had to google where they were stored! but again, GNU/Linux is amazing like that, so with some documentation and some observing the already made `.service` files, the thing was just working. Intuitive and human-readable as anything I've ever used. As much fun as I like to poke at `systemd`, I don't think I can use anything but service files anymore.

So, apparently, this was done! a bit more tweaking in script sleeping times on the main script launcher, and a lot more tweaking to maken actually valid `csv` for edge cases that creeped up on me, and the backend was fully operative and chugging along! Job well done! (writer's note: I did not know about the fact that I had enough material for a mistakes blogpost back at that time. I was young, happy and naïve.)

So I had a backend! it fully worked, and I could `SELECT` away all my network packets away however big the thing was, filer with anything I wanted... Wireshark (mostly) meets SQL!

As a usual habit, let's list what we ended up with in a bullet point list for summarizing my text blocks

- We start capturing data with an unending `tshark` task, from a `python` script with the `os` library
- When the files are done being written to, we move them with a bash incantation I cannot fully decipher
- When files are moved to the new location, we run them through a python script that
	1. Reads the `.pcap` file into standard input
	2. we modify the `epoch time` entries into UTC time
	3. we join empty fields into consistent entries that `tshark` separates by default
	4. We write it all into a new processed `csv` in a standard usable format for our database scheme
- The new CSV is fed into the `python` library called `MySQLdb` that connects to our `mysql` instance and runs an `INSERT` query for each row on the file

Working infrastructure!

But a backend is that. *back*. Like the back of a refrigerator, it's pretty damn ugly to look at if you don't know what you're working on. So, on the next blogpost: frontend! we get to see why I am even less of a designer than I am a developer! responsive can't be that hard of a task, can it?

We'll see how that train of thought ended as the inevitable full scale railway accident it was meant to be since the moment I said "responsive design".
