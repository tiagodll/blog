---
title: "Hugo blog engine"
date: 2021-06-12T17:50:00+02:00
draft: false
tags: ['Go', 'Hugo', 'Blog']
---

After almost 10 years of blogging in blogger (with a very low frequency), 
I decided to self host my blog. 
So then I had to decide what to go with

## Requirements:
1. **Markdown** - I have been a web developer for more than 20 years. Because of that, my growing hatred towards WYSYWYG editors peaked while I was working in a custom CMS. In 2013 my company started using redmine (a markdown)

    I got used to markdown while using redmine at work. Since then, I have been using it for everything. Even tocobooks mas markdown as its text editor.

2. **self hosted** - I dont want to have my content locked in someone else's database

3. **no database** - it is sad to see the effort the blog engines take to give you version control, when you could just keep your .md files in git. And that is on top of the main reason: you own your text files.

4. **low or no maintenance** - I like ruby, but my experience with it is: server needs constant maintenance. 
Go on the other hand is a lot more stable, and lightweight.
Dotnet would be ideal, as I already keep always up to date in my server.

5. **easy to modify** - I started programming web apps in PHP in the early 2000's. I understand why it is so popular for blogging. the fact that you can just include .php files anywhere gives a flexibility that is very dificult to match.


Considering all this points, Hugo seems like the obvious choice (in 2021).
- markdown - check!
- self hosted - check!
- no database - check!
- low or no maintenance - definitely check!

    hugo generates static html and you can host it anywhere
- easy to modify - yes

    not as it would be with php, but hey, considering the other benefits, it is a no brainer.

## My tips to get it up and running

### - **Hugo should not be installed in your server.**
The installed version is for development and exporting the files.
You will copy the generated static site to nginx, apache or something similar.

If you are impatient like me, you might rush into having it all running directly in the server (I even edited the first posts directly there), and then will be thinking: -Ok, I have it running in the server. How do I run as a service now?

### - **You should host your blog using a simple web server**
Since it is just a static site, you can host it anywhere. 

### - **Everything will hardcoded with the baseurl from your config file**
It is obvious, but sometimes it take a bit too long to process it.

My theme was looking weird. At first I thought the theme was not working, but then I realised it was the baseUrl

### - **Keep your local dev environmnet in source control**
Or at least your config and md files. 

Just in case

## Will I continue with this in the Future
Not sure. 

So far it seems good.

It took some effort to move away from blogger, but now I have it as markdown files, so it could be easy to migrate again.