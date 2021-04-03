---
layout: post
title: "Dev Diaries - PCAPAnalyzer III: What do you mean CSS is turing-complete?"
date: 2021-04-03 10:00:00 +0200
category: dev diaries
permalink: /blog/:categories/:title/
description: ugh, CSS

---

I think that, if you take a look around this site, even though it's built upon a Jekyll theme, you will notice a fact that is impossible to deny: apart from not being a programmer, I am *also* very much not a designer. 

I have had HTML (and some CSS) experiences in class. Granted, it was stay at home classes because there was a *pandemic* happening, so I didn't get a lot of explaining done to me at the time. Then, to make it not boring, I had some self-taught PHP under my now less-fitting belt (again, pandemic. Did we all bake our own bread at the time?) and some PHP classes on an actual school this time, because the curve was flattened. Kinda.

So what I learnt about all of the earlier steps of this process in terms of good practice in software development, and in general, is to plan before I do. So I decided I needed to plan the feature scheme for the frontend first, then implement it bit by bit later. It's very much a lot easier to build something that is clearly outlined, that to jump in and just start writing PHP without proper forethought!

So, I gave it a lot of thought, built it, and did it wrong. Learning!

Let's go through what my planning scheme was. So, first, let's talk about the landing page. We enter the URL and we should land on a page that should act like a *dashboard* of sorts. I like what [Grafana](https://grafana.com/) does, just simple numbers at a quick look, so I kind of mimicked that. I display, on a table, every database accessible and show the total number of stored packets, the data sources it has, the total number of external IPs, MAC addresses count, the multicast packets, and how many protocols were found on the capture. Just quick numbers in case something looks abnormal.

Then we get to probably the hardest part of the project to engineer: packetStream. packetStream is what this project initially was thought for. Here, we display every single packet that has gone through the capture interface. Any data you captured, a query away. Now, because this is a lot of data, it is imperative that I built in a filtering system in place, which lets you filter by any combination of columns in the system, like finding all of the packets that are SSH auth attempts from a certain IP. Or finding everything a certain machine has done (with their MAC address). All behind a pretty interface, in 50-packet lists.

Is this an overwhelming amount of data? It probably is. Going packet by packet is not efficient when searching for intrusion attempts in a large-scale network. That's why I came up with dataGlance. dataGlance, as its name explains, is *data* at a *glance*. Yeah, don't call me to name Amazon Web Services products, I'm incapable of coming up with something convoluted like *lightsail* or *kinesis*. Names are supposed to be helpful ever before they accomplish a PR task. 

So, here in this tab we get useful information that we may be interested on, pre-calculated for us. All of the found private and public IP addresses. Is there a DHCP lease that is not accounted for? Is someone connecting to somewhere in an IP block ythey shouldn't be? It's here. Any MAC address that is not from a known machine is running around? here's a full list of the seen ones for you to cross-reference. What protocols have gone through a capture point? what ports are being used? All that info is here. just a click away for you to account for and investigate.

Now it feels like this might need more. Yes, a quick glance at data has shown you some potential need for investigation, But then, simple filtering through dataGlance is not cutting it anymore. You need absolute control. That's where queryRunner comes in.

Again, the feature is named in the anti-Amazon Web Services naming scheme: it does exactly what it is named like. You write a SQL query on the text box, it runs against the database, and you get your data. In this case, you get your data as a CSV text file, because as we established in an earlier blog post, yay for standard formats, and you do whatever you want with it from then on, locally.

All of these seem like good features to have on a piece of forensics networking software, but having it only capture traffic from some source is not that interesting for a lot of use cases. Maybe the big incident already happened and you need to run an analysis on already captured data?

This is where dbCreator and databasePicker join the game.

The first one is a simple form: you provide a name, you provide some `.pcap` files, click on the button, wait a bit and boom, you have a database to work as, like you did in your network capture environment, with almost the same features as we did for the packetStream backend. (dataSources aren't identifiable from pcap files, at least as I understand it now, unfortunately).

Then, changing to your new database is dead simple. Just click the menu button displaying your current database and you will be taken to databaseChanger, which lets you choose your newly created database and set it as active. From then on until your session ends or you switch it again, you'll run all your data queries and page browsing checks against the database you selected. Then, when you're done with it, you can just drop the database if you want to save space on your server.

Then on the About page you manage access to the program. You can both create ysers in the user management console to access the web frontend, and you can create API keys for the API query system.

Which takes me to the fact that there is an API query system! You have the option to retrieve data, like all of the available databases, the data from the frontpage, or the data from dataGlance, so you can integrate it in a custom client, like perhaps a desktop app or an external monitoring tool?

Now you many notice that this was all too simple. Too few mistakes along the way, too few rewriting decisions in our process. And I thought so too. And then, the two afterthoughts came upon me, one from getting to know a new tool and one from procastination. The first one happened when I discovered that PostgreSQL existed thanks to my classes, and how the user control system was *way* nicer for my plans (and also, less related to Oracle), best defined python connector drivers for the backend... It was worth a move!

Unfortunately, this automatically revealed to me that I had a roadblock based on procastination: my php code had exactly zero (0) functions in it. It was just a pack of scripts running some queries, because I did very much not expect the project to get this large.

Let's recap what we have gotten to do so far:
- a front page that shows us numeric data for each of our databases
- packetStream, a multi-source table system with filtering options for finding some specific types of communication on the logs
- dataGlance, a page that shows you important info on simple tables, visible on just a fast look for sususpicious data
- queryRunner, just run any SQL query you want against your database and get a handy `csv` file to put into any other system that will swallow it
- dbCreator and databaseChanger, for after-the-fact large scale analysis tasks for any `.pcap` files that you may want to upload into the system and get the frontend treatment for their data
- Identity and Access Management for the frontend system with user management and API key creation and dropping
- An API system where you can query the available database list, and any dataGlance and frontpage data entries in handy universal `json` format

So, the frontend worked! For like, a week. Then it was time to move to `postgresql`, and the entire project was halted because backend compatibility connectors were mad that that the `mysql` connector was not pointing to a `mysql` relational database management system anymore. So, in the next post, my naïve and dumb decisions mean I get to learn code refactoring *the hard way*!

See you then!
