---
layout: post
title: "Dev Diaries - PCAPAnalyzer IV: Mistakes and manuals"
date: 2021-05-18 10:00:00 +0200
category: dev diaries
permalink: /blog/:categories/:title/
description: Ah right, people might use this

---

Let's be honest with ourselves for a moment here: when I decided that I was going to develop a tool from scratch, it was bound to have some bad choices in it. That's how learning works, right?

So, both to avoid people from making these in the future and for your amusement and entertainment, let's speak about those mistakes I made when developing this project.

The first one stemmed from a lack of experience with the tools at my disposal, in this case, specifically about the relational database mananagement system I chose to implement into our application here. If you remember our post about builduing the backend, I chose to implement `MySQL` as the database management program for our data storage and querying. This turned out to be my first mistake.

See, in the beginning, there was a general access with `admin` to all databases. Turns out, this is not a really good idea, so that needed to change for production-grade releases. This turned me to look into some IAM, or Identity and Access management, for our product. This, in turn, showed me a comparison between user management systems in different relational databases. Turns out, when compared with the rest of the options, `MySQL` is, let's say, *less good*. It's not *bad* per se, but having seen `PostgreSQL`'s user, role, permission and ownership management system makes the former one look half-baked at best.

So, it came time to port the entire ecosystem to `PostgreSQL`. This got me mixed results. On the frontend side, it was mostly simple in the `php` and `sql` side, even though I needed to make some slight modifications, mostly for time calculation. On the backend side, however, it was a mixed bag. Choosing the new `python3` driver for database management system interaction was dead simple: the tried and true standard is just using [psycopg2](https://www.psycopg.org/) and running with it, since it implements all of the `python` database interaction standards as the specification requires, so it's better standardized.

But in contrast, I found the problems when I tried to insert data into the database. It was kind of simple in the local server capture, since it was a `COPY FROM` command that grabs a `CSV` file that is read into the database. But then I discovered a problem: `COPY FROM` is a SQL command that reads server files. But it wasn't reading CSV files from agents. After a couple of hours of scratching my head, I noticed the problem: It was looking for *local folders* that didn't exist in the server. Because agents are *remote*.

That said, `psycopg2` has its grounds covered for you in this, thanks to being a more standard and complete database driver than the `MySQL` counterparts. You can just use the `copy_expert()` function and use a `COPY FROM STDIN` query, that will just read the function parameter with the entire file. Presto, it works. It just inserts a really long string over the network. (bonus points since `postgreSQL` has ssl-enabled connections by default!)

Good, now all of our system is running against `PostgreSQL`! Everyone can come in and check!

Which brings us to our second problem on the line. *everyone* can come in and check. There's no access control system. The frontend and API are a wild west free-for-all. That needs to change.

Here I chose to learn something new to implement here. I've never done user storage and retrieval, and I wanted some testing done. So I investigated a bit. Seemed like I needed two things: API access control, and user login control. The API control was easy: just generate 32 character alphanumeric strings, and check that the strings exist in the newly created credentials database for every API request. Then, the user and password management system. 

This, as it turns out, was easier than I expected it to be. Inserting the user is just as easy as transforming the provided password into a `bcrypt blowfish` hash and inserting it into the database, then using another function built ito php to check if it is correct. No need for my own salting or anything. Then it was just a matter of checking if the user was logged in on every page, and done, the system now has Identity and Access Management. Just a page to create users, and some code to check the login status.

Now, that last bit brings us to our next problem. I was just pasting the code into the `php` file headers, and remembered a line in the `to-do.txt` on my Documents folder

- Refactor PCAPAnalyzer into using functions

Yeah, I was just running `.php` scripts when coming into a folder. Not a single function on sight. Wanna change something in the menu? change it on the 9-12 files where it's stored. This is something that needed a substantial logic rewrite. Thankfully all my code was mostly the same in every page check, so after a bit of investigation into how `include`s worked in `php` and some variable passing logic, the app was complete. One change in the function, and the menu was different in every single page. One line of security login checking, and the entire function check took care of the needed session checks. Structure!

Now, writing this I noticed a problem. I had kind of forgotten how my old code worked. Well, okay, some parts I didn't remember *at all*. Past Ivan decided that a couple thousand lines of code didn't really need code comments or documentation, since I remembered it then, didn't I?

Well, since this was going to be a public project, I needed some actual documentation, both for code cleanup and clarity, and for usage documentation and feature discoverability. The first one was simple once I discovered how my old code actually worked, just write out the logic of the functions and HTML generation, and it was a much easier to understand project for anyone that came in without knowing the project.

On the other hand, user/administration documentation was a different beast altogether. This is something that I had never done before, so it was a chance to get to know a new tool. I asked around for recommendations, and I got one that I liked a lot: on the [SelfHosted podcast](https://selfhosted.show/) Discord server, [RealOrangeOne](https://github.com/realorangeone) pointed me towards `MKDocs`. Turns out, I had seen `MKDocs` hundreds of times before, I just never knew. Everyone seems to be building their user docs on it, so it's an active project that, upon inspection, basically told me "Jekyll for documentation", so I was sold. Downloaded the packages into my machine, wrote the documentation structure, pushed into github, ran `mkdocs gh-deploy`, boom, [the docs were up](https://ivanol55.github.io/pcapanalyzer-docs/). No wonder this is the standard out there.

The project worked. It was fully documented. It was usable by other people. We were ready for lauch, except for one last detail

I don't trust my code security. I haven't ever done security-oriented programming, and I would be far more comfortable with the project with some pentesting done to it. So, thanks to this being a class project, I enlisted the help of [Sergi](https://github.com/hondas04). Sergi is a systems administrator and classmate of mine focusing in cybersecurity and penetration testing, so we decided that part of the project would be to pentest the program.

Long story short is we did end up discovering a couple high-priority CVE's on the program, basically some stored cross-site scripting and session stealing, which enabled an attacker to access the application. These vulnerabilities have since been fixed on the application thanks to the HTTPOnly flags set on a the `.htaccess` file.

As always, let's recap what we ended up changing and fixing

- Rewrote the interaction with the database management system from `MySQL` to `PostgreSQL`
1. Changed the frontend to postgresql queries and php variables
2. Changed the backend to interact using the `psycopg2` interaction library
- Implemented identity and access management for the web frontend of the ecosystem
- Implemented authentication for our API endpoint requests
- Rewrote the frontend implementation to work using functions
- Documented our newly refactored code
- Documented the usage and management of our application ecosystem with MKDocs
- Ran a penetration testing session with some friendly help
- Fixed the vulnerabilities we found on the system

So, with that, the project was finally complete! So this two and a half year development journey is complete, and you can now check out the [PCAPAnalyzer project](https://ivanol55.github.io/pcapanalyzer/) and the accompanying [Project documentation](https://ivanol55.github.io/pcapanalyzer-docs/)! Now I just need to figure out what our next project is gonna be...

We'll see!
