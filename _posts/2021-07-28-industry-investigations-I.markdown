---
layout: post
title: "Industry investigations I: DevSecOps sounds convoluted."
date: 2021-07-27 10:00:00 +0100
category: industry investigations
permalink: /blog/:categories/:title/
description: The buzzwords are out of control!

---

Have you been interviewing for technology jobs lately? I have. It's been an experience.

I recently finished my course for systems administration. Mostly it was all what I expected: building virtual machines, service administration, how a datacenter works, data recovery, some security, red-team and blue-team... turns out, times have changed. Times have changed *a lot*.

So the first thing I noticed by scavenging LinkedIn is that sysadmins are out of style. You don't want a systems administrator anymore, or at least not someone who's *only* a sysadmin. That's separated from the development team, what's called a "silo", and "silos" are so last-gen. Now, DevOps is in!

In this search for something new, you've probably come across the word DevOps. DevOps is essentially a philosophy, a way of working in your team, that makes those "silos" merge together and make development and environment management a lot more agile and fast-moving. No more  patch tuesdays, no more fear of deploying on fridays!

Now if you're like me, you might find yourself wanting to follow the security path. It's a growing field, there's plenty of work, entails exciting experiences every day... seems like it's all positive! (as long as your client listens to your recommendations, of course), but pretty quickly you notice that cybersecurity is one of the fastest moving fields in the industry. It needs to keep up with changing in *all* of the IT aspects of a company. You're in charge of protecting every asset around, whatever it happens to do.

If you follow that lead and give the thought a spin, there's a conclusion you may come to pretty fast: DevOps and security are a very good match! integrating everything for automated checks can take work out of your hands, defining security requirements in every part of the process... it just makes sense that if you choose to follow security culture in the entire development process, it will probably take work out of you after the fact.

That thought recently gave way to a new job opening space that integrated security inside the development culture of DevOps, which we named DevSecOps, because the IT industry is about getting work done, not making up fancy names. At least we didn't let AWS name it so we aren't stuck with AWS DSO or something... yet.

So, let's dive into it. DevSecOps, as its easy to recognize name presents, is a culture, a way of working, where we integrate security into the process of DevOps. This is better explained with an example: Imagine you have a pipeline that tests your code before you build and publish it to test if it works as intended: unit testing, code analysis... 

Well, you can do something similar with the rest of the environment, like for example analyzing if the infrastructure the code is gonna run on is well configured and has no security problems by analyzing your Terraform declarations for encryption missing in a volume, or an unrestricted security group. If the alert is raised before the infrastructure is built, you don't have to fix it later!

The entire idea can be summarized into a simple concept: Security isn't a toggle you can click. You don't apply security efficiently by installing that sotfware the vendor recommended to you. Security is a continuous process that is better built into your development process, preferably with automated checks that tell your team that something needs changing. No deployment with security holes on your ops team's watch.

Even if your infrastructure seems too legacy to adapt into the DevOps way of working, DevSecOps could be important. Uploading some Ansible playbooks to GitHub? Maybe pass them through a security analysis module when they get pushed out as a pre-commit hook. Built your own docker images for some specific task? Maybe you want to give them a spin through an image analyzer like Snyk or `docker scan` (did you know Docker had its own [vulnerability analyzer](https://docs.docker.com/engine/scan/)? I didn't!). Are you moving some infrastructure into terraform code? Bridgecrew integrates into pretty much any provider to alert you of that slip of "allow 22/tcp from 0.0.0.0/0".

Any small step is a big move towards making work easier for your developers in the long run in avoiding technical debt and lifting load from your ops and admin team from these janitor tasks so they can focus on the really important tasks on your organization. Also worth noticing these tools are far cheaper than an incident response spending. Certainly in money, maybe in public image. I'd gladly take some cheap AWS Security Hub alerts over a GDPR fine.

Have you been thinking about getting up to speed on your infrastructure security? Well, maybe you should. I certainly will be giving some thought to my docker repos from now on, just in case.
