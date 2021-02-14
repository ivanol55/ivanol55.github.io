---
layout: post
title: "Headbang development I: What's this Jekyll thing?"
date: 2021-02-14 10:00:00 +0100
category: headbang development
permalink: /blog/:categories/:title/
description: Let's start a blog, they said, "It's quite simple" they said...

---

Blogging is a good way to practice your technical writing soft-skills. You get to learn the really important ability of writing understandable content for a broad audiences, with the perk of building up portfolio. Win-win, right? well, that is, if you can find a topic. Development diaries? client work stories? software reviews? Choice paralysis is a thing, and picking a consistent theme and structure is complicated. 

Then I started to build this blog while I thought about dev diaries in a completely new platform to me, and suddenly saw my first post series in front of me, in the form of cryptic error codes and stacked browser tabs: self-learning stories.

Self-learning sounds fun at first when you look at it. That's something that recruiters like, right? independence, knowing how to set yourself up in new environments without much one-on-one help, something-something pull your own bootstraps. It *is* a really useful skill to pick up, especially as a junior, but it can sometimes be an... interesting process.

I like to compare this way of learning to [Answer in Progress](https://www.answerinprogress.com/)'s way of investigating a topic: search what you want to do on Google (Bonus points for using DuckDuckGo) and see where the algorithm takes you. Documentation, courses, "getting started" tutorials, github repos... I usually spend the first 15 to 30 minutes just reading up and making notes. That's how I started this, really! I just googled "build a site with Jekyll" and went knee deep into tabs.

Then problems started to appear. The first problem was what I like to call "store fatigue". Everyone has their own package management system. I just `apt install`ed jekyll, why is it not launching? what do you mean unset variables? well, when this happens, we go back to our *headbang development* model. Google the error code and start opening tabs!

What do I find with this? thanks to the best place in the internet StackOverflow, I find out that apt does not set up jekyll correctly, and that I need to use something called *gems* from the ruby repos. I didn't know what Ruby was until this morning, so I already got something out of the project. (Partial) success! and even better, using the ruby install process just fixed my issue, so the terminal tells me I have a working Jekyll site!

I open the URL that the program spit out, and I open it on Firefox. What I find in front of my eyes is some form of practical joke.

Who designed these Jekyll size standards? 60% of the page is literally empty! Well, with this, I remember having seen the words "Jekyll themes" on the google search suggestions when starting my investigations. May the gods bless the almighty algorithm. So, I start looking at these themes on whatever recommended pages there is, like the default Github Pages repos, or third party aggregators.

Is it some kind design convention? Like an inside joke, written in stone, that a Jekyll theme must waste at least 50% of screen real-estate on empty white space? who designed these, Twitter? Does Jekyll not work in any computers connected to a screen that is not a 4:3 CRT TV from 2002 or a phone? THis needs fixing!

But when I start looking at the folders that Jekyll helpfully generated, it seems there is no css source anywhere to edit. There *is* a css file on the generated site under `_site`, but it source of configuration seems, at first sight, to be fed entirely with magic and unicorns, invisible to the human eye to access. DuckDuckGo it is then.

Turns out, Jekyll *used to* have the theme files on your site folder, but that is (truthfully, I'll give them that) hard to mantain and update. So their solution was to put them out as easily installable and upgradable packages (yay!) on the Ruby repositories (ugh, *store fatigue*). Apparently, the gem setup already pulled in the default theme as a gem, and the files are somewhere else. A couple tab openings later, I found a bash command incantation that tells me that the theme files are located on `/var/lib/gems/2.5.0/gems/minima-2.5.1/`, so I go take a look and see how they work.

I find out that you can override these by creating the folders yourself in the Jekyll file folder, so I copy the defaults and start implementing some *actually sane* design configurations. I set the theme content to be 80% of the window's width, easy enough, then I try to create more posts to check out the layout.

That move was about to open me up about 6 hours of work.

Someone apparently decided that, as the content has to be 33% of the screen width, the *only* logical answer is of course set them to be one big unclassified column. Because of course that is the most logical design decision for a modern website. So, change time it is! I decide the best choice for space optimization is a resizing flexbox that shows the posts. Not all of them, more on that later. So, I `vim home.html` the template on the `_templates` folder and see the problem. It is, apparently, a list, so I need to change this to a css flexbox grid.

Problem is, I have only headbang learned php in terms of WebDev, which is very much not a client-side static language. So, after opening a bunch of new sites to figure out what even `Liquid Markup` is, I find out it is apparently decently easy to figure out, at least with the theme's premade varable structure. The naming structure was good, fortunately (at least something had a good design team on it!) so I configure it to limit it to the last 5 posts, and make it display as a flexbox after some tinkering. Well, a lot of tinkering. I'm not precisely a CSS Wizard.

So I reload the `jekyll serve` command, and it's finally working! Now I can investigate how creating a blogpost works. You know, for the *blog* part of this blog project that started 6 hours ago? Well, this is when the easy part comes in: the blogposts are auto-detected on the `_posts` folder as long as you name them on the correct way (good thing the docs exist so I see what that is), and it auto-picks the name, post excerpt, SEO tags and permalink from the file header. Wordpress could learn a thing or two about post simplicity from Jekyll at the end of the day.

Speaking about post simplicity: post tagging. This is great! no database queries, no storage, no consiscency errors... You just write the tag and it parses it for you! So, as I want to write post *series*, each series being a category (more on those series coming soon!), I need to make a template for the markdown file to show the categorized post. The best option, according to StackOverflow, is a "Liquid Markup for loop that iterates over every category, with a nested for loop listing the category posts".

As easy as that turned out to be, this industry has a word simplicity problem.

So, after checking that works, it all seems to work fine. So, all the confusing stuff done, now I gotta put it up on a Github Pages site. Easy enough, right? I mean, I've pushed stuff to repos before! I have already released [open source projects](https://github.com/ivanol55?tab=repositories) before! This is gonna be real easy! Well, turns out, Github Pages is not Github straightforward.

I push the contents to the site and open `ivanol55.github.io`, the expected url. *Nada*. Empty. Well, my disappointment is immesurable, my day is ruined, and my DuckDuckGo tab is revving its search *engine*. I find out a couple of errors I didn't assume there:

1. I need to push the `_site` folder, not the whole thing
2. I need to tell Github what branch it needs to read as the root of the `_site` folder

So, I figure out how to make a new branch, setup the .git folder to push the `_site` contents to said branch by default, and make a test push. I find the tab that I left with the github page, press Fn + 5 (60% keyboard gang) to reload the previous 404 and... there's a site! Jekyll did it, it's *aliiiive!*... wait, wrong book reference?

Now the blog is live! And after that initial rocky setup, the results are in:

1. I learned that ruby exists
2. I learned what static site generators are, and that there's more of them
3. Some more CSS tinkering
4. Some basics on Liquid Markup logic and function
5. How to setup Github Pages
6. A blog where I can write posts. Like this post! Hello!

So, it was an interesting weekend project that has opened quite the new road to me about content creation. Maybe even go back to [Twitch live development](https://www.twitch.tv/ivanol55) at some point?

To many new blog posts to come!
