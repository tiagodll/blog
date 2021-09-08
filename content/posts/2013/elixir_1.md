---
title: "So, what is elixir?"
date: 2013-06-23T17:15:00+02:00
draft: false
tags: ['elixir']
---

![Elixir logo](https://lh5.googleusercontent.com/proxy/pDPq3cxOcMm97pAzmpbtcCsaSuq9iTcrJZuUH2YOQ7ggJAsCt8Q7nmpZJq5DFEaCOlnozSSQZBBwGbmkEIKN=s0-d)

A bit of background first:
In my masters I had a module called Programming Paradigms and Languages.
What this module is about is the different paradigms, and one language for each of them.

The chosen languages were:
Object Oriented: Ruby
Functional: Clojure
Logical: Prolog
Process Oriented: Erlang
Mathematical Programming: Matlab

Out of that what I can say is:

    Ruby syntax is awesome. So elegant but there are many performance issues
    Clojure is very nice for the academic world, but I cant imagine anyone using it for commercial software building.
    Prolog. Ah, prolog... I have had a love/hate relationship with this since my undergrad. Powerful but very specific.
    Erlang: blazing fast. handle processes like no one else, but it is probably the worst syntax I've ever seen. Ok, not the worst, but close to that (Yes mind fuck, I didn't forget you)
    Matlab... this doesn't even had to be in this list



So, after that my conclusion is: I want a language able to handle many processes like erlang, but with a nice syntax like ruby and maybe add there some cool recursion features from clojure if you can.

That is elixir.

"Elixir is a functional meta-programming aware language built on top of the Erlang VM. It is a dynamic language with flexible syntax with macros support that leverages Erlang's abilities to build concurrent, distributed, fault-tolerant applications with hot code upgrades."- http://elixir-lang.org/

So, just like that it has become my favorite language.
But just like all open source, making it work is always a hassle. But this is for the next post.

Lets have an easy start: how to add Elixir to your path?
I am new to the mac world, for those of you that like me are not great at this, let's go step by step:

    check if it is already there: type "echo $PATH" on bash (capital letters)
    if elixir is there, you are good to go. if it is not, add it by typing
    echo 'PATH=$PATH:/Software/elixir/bin' >> ~/.bash_profile (I put it in the folder Software under root, use your path)
    close your bash and open it again. type iex, if you get the interactive shell, go ahead and play with it :)
