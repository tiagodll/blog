---
title: "Blazor is awesome, but not for the reason you think"
date: 2023-11-28T18:00:00+02:00
draft: false
tags: ['C#', 'csharp', 'blazor', 'webassembly']
description: Blazor is great, but why?
---

When you think about Blazor, what do you hear?

> This is awesome, I can write C# in the frontend.

Well, today I will show you some reasons why that is not entirely true.
## 1. You will probably still need a bit of JavaScript

As of now, there are still reasons to use JavaScript interop, although this may change in the future.
## 2. But, but, components…

While components are useful, they may not cover all the cases you need. However, you can do most of your needs with simple HTML + CSS.
## 3. JavaScript code is not that bad

If you want to have a full-blown enterprise application and your frontend code is huge, then using C# makes sense. Otherwise, a little JavaScript will not hurt.
## 4. The shared model

One problem I had to deal with for years was having the model from the frontend and backend out of sync. You might add a new property, remove a property, or make a typo, and now you have no clue what bug to expect. Is the field going to be mapped to empty? null? the whole object null? It depends on how you implemented it.

This bothered me so much that I made a little app that would generate a JavaScript model based on my C# code. However, this is not great, as it needs maintenance, overwrites changes you make to the frontend model, and can be hard to deal with some properties.

But fear no more, Blazor to the rescue!

Now you can share a model between the frontend and backend. Just make a separate project, add it there, and Bob’s your uncle.

This is the biggest value for me. The fact that you now have a compiler checking if there is any inconsistency can save you so much time.

It is beautiful.

I know someone is going to say:

>  You can do the same with JavaScript.

And to that, I will reply:

>  Sure, you can, but in a project with a big team I would rather have C# code any day of the week.

Now it comes down to personal opinion, but hey, this is mine. 
How about yours?