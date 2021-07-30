---
layout: post
title: "Headbang development II: Containers are weird"
date: 2021-07-30 10:00:00 +0100
category: headbang development
permalink: /blog/:categories/:title/
description: I was used to VMs already, and we moved from that?

---

I've always done my testing on virtual machines.

I mean, it's what I always got to learn. You test new apps, new servers, new services and such things on top of an operating system. Controlling the whole stack from kernel to application. You need an update? you back it up and update it. You need to add capacity? You add hardware allocation to this virtual machine. Simple enough approach, and perfectly fine for some deployments like having just a website with a database for example.

Problem is, me having a new job in the Operations team at [Geko Cloud](https://geko.cloud/) has led me to the realization that this mindset and procedure is not very DevOps of me. So it was time for a brush-up on my own environment, since I destroyed my raspberry pi server's SD card with database writes. Turns out some static image storage media doesn't like integrity checking nine hundred thousand database rows of data. Who would've thought, eh?

So now I face some problems with environment migration:

1. Installing the services on bare metal is a no-go, since I move my environment a lot and clear my drive from time to time.
2. I can't use virtual machines. This computer is a 4-core machine with 8 gigabytes of ddr3 ram. If I try to run four or five VM's on it, I better have some off-site backup strategy to recover from the fire my machine might spin up.

Additionally, it just so happens that my service backups were, let's say, sub-par, so I could use this chance to rebuild it better than last time with better technology, with my current infrastructure renewal budget of zero euros to invest, I came to this new (to me) incredible technology stack buzzword: *containers*

Turns out there's a lot more container stuffs than I initially expected there to be. I ended up deciding on Docker, because, it's Docker. But A lot of other names were thrown at me on research:

- LXC, the de-facto linux container system on the linux kernel
- LXD, which is LXC but with built-in improvements and some added features
- Podman, which I am still genuinely unable to decipher in terms of the need of existing or what it does

I had done *some* work with Docker for class. I had ran some `docker run` commands to run some hello world nginx containers, I blindly copy-pasted some docker-compose files from GitHub which did some things, and I installed Portainer in a couple of virtual machines for class, which barely worked due to lack of resources. You know, *stuffs*

At first I tried to run some containers using `docker run` commands. Did it work? yes. Was it reproducible and scalable? as it turns out, no, and as always, there's a better way. As I was told over at the [Self hosted podcast](https://selfhosted.show/)'s Discord server, this better way is currently `docker compose`. Docker compose is a declarative way of setting up your entire container stack. In bold words, Docker Compose files are yml-ified `docker run`s. Which is perfect for the situation, because those fit perfectly into a GitHub repo! More projects to show and write about! As if I didn't already make up enough of those, eh?

So, let's analyze the problems I had that I could try to tackle in this new infrastructure:

- My reverse proxy was a single `apache2` install with a lot of `.conf` files
- My password manager was a local `KeePassXC` file I copied into google drive whenever I felt like it
- Google Drive was cutting my free forever picture storage in a month and a half
- There was literally zero environment monitoring
- I had port 22 forwarded from the open internet to access my network (Yes, I am aware that I was hired as a security operator for DevOps)

Let's see how docker can solve these problems.

First, reverse proxy. I had never used one of these, so my first guess was nginx. It *can* do the job if you want to handle it no problem, but as again it was pointed out by [TheOrangeOne](https://theorangeone.net/) at the [Self hosted podcast](https://selfhosted.show/)'s Discord (seriously, give these guys a listen), docker's current solution for this is [Traefik](https://traefik.io/). 

Traefik is a cloud-native solution for reverse proxy tasks that has a very useful tool for my use case: You configure the entry points in each docker-compose file for that individual service, and Traefik just auto-discovers it and forwards it when it is up. If you configure it to generate SSL certificates through Let's Encrypt, it automagically stores them in its own json file and manages it for you. Port pairing with internal ports like sending https to grafana's port 3000, special rules for specific folders in subdomains... everything is easily managed once you get the hang of it!

Next, what to put behind this reverse proxy. Solving the password management problem, it seems like the new big boy in town was Bitwarden. It also had an official self-hostable docker image! neat, right!? Well, turns out, this is an image for enterprise stuff, apparently. Because I certainly don't have a place to run a separate ingress, a big SQL server database, and a heavy frontend implementation made to handle hundreds or maybe thousands of clients. I ended up picking a much simpler solution named bitwarden_rs at the time then renamed to [vaultwarden](https://github.com/dani-garcia/vaultwarden). It's a rewriting of Bitwarden to work in Rust, much simpler in structure, it even runs on sqlite if you're not putting much load in it like I am.

Picture storage was a hard one. I had several solutions lined up on this one that never quite got the job done. I went through Librephotos which felt too complex, lychee which was beautiful but lacking some features, piwigo which didn't quite catch me in... I ended up landing in [Photoprism](https://photoprism.app/) (thanks to a recommendation coming from a certain podcast. Yeah, they know the stuff), which is an interesting solution for two specific reasons. First, it never touches your original pictures, only indexes them (by being a cpu hog for a few hours initially), and second: it has *built-in local neural net learning and image recognition*. I never expected this outside of a paid product, so by all means, it's staying.

Next was monitoring. This ended up being an entire stack out of some specific necessities. First, data gathering. To gather data from a lot of sources at scale, it seems like the current big guy in the scene is [Prometheus](https://prometheus.io/), simple enough and with a lot of support. Then to send data to prometheus I decided on two entries. First, [node_expporter](https://github.com/prometheus/node_exporter) for physical machine data and then, so I could just ping my websites to log if they were up or down and handle some delay checking I stumbled upon [blackbox_exporter](https://github.com/prometheus/blackbox_exporter) which basically lets you check the responses on mostly anything http, tcp, icmp, imap or pop3, so a perfect fit. Visualizing all this was clearly handled by all-trusted Grafana, because sometimes standard is more than good enough for home projects!

Finally, when something like this fails and my monitoring system alerts me, I have to go in and fix it. This is a problem if I'm out of home. Before this I just port forwarded 22/tcp to my pc, which, yeah, I know. I wanted to put in some remote access and host it in docker for the ease of rebuilding, and [linuxserver.io](https://www.linuxserver.io/) had (among a lot of other very useful tools) a wireguard image that just creates as many wireguard clients as you tell it to on boot and runs with it. Simple, reliable, using a new technology stack, wonderful results!

So this solves all of my immediate problems with the nice plus side of teaching me about a very useful tech stack (or rather several of them) I can use on my work too, and for blog posts! win-win situation!

And of course, all of this infrastructure is available and easy to rebuild, modify and tweak quickly thanks to its declarative nature, and you can find all of its (partly inefficiently written) glory over at my [infrastructure repository](https://github.com/ivanol55/docker-infra). I'm keeping up to date with the new services I find that I wanna add into my stack! I wonder what's in [the linuxserver](https://fleet.linuxserver.io/) repo that I can try out next...

Whatever it is, I may speak about it on here, so stay tuned to the blog for future postings!
