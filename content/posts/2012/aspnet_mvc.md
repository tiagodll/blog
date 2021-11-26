---
title: "Aspnet_mvc"
date: 2012-03-23T17:37:27+02:00
tags: ['aspnet', 'dotnet']
draft: false
---

So seems like asp.net mvc is finally popular?
That's perfect, I mean, this is how it should have been since the beginning.

Let's start with what is mvc:
Mvc is to build the application as model, view and controller.
What does that means?
It means that the view should be only responsible for showing the interface, the data access should be in the model layer, and everything else should be done by the controller.

Sounds nice, right? Old but gold... But the thing is... and how about the web?
 Usually we had a asp / aspx / php / jsp / etc file. This file is requested, and then, from this point on you can call your controller logic.
But this isn't the proper way to do it. This way your view is mixed with the controller.

And then, there was the light! true MVC to the web applications.
Is this hard to implement? off course not, my little grasshopper....
You just have to think your application in a different way. and what is that way?
Routes!

And your routes file should look like this:

{{< highlight csharp >}}
public static void RegisterRoutes(RouteCollection routes)
    {
        routes.IgnoreRoute("{resource}.axd/{*pathInfo}");

        routes.MapRoute(
            "Default",                                              // Route name
            "{controller}/{action}/{id}",                           // URL with parameters
            new { controller = "Home", action = "Index", id = "" }  // Parameter defaults
        );

    }
{{< / highlight >}}

And how do you use it?
 Easy: you set it as your controller name, and action, so every time the user types that address the method defined as action is called.
so when the user types website/home/index, the method index inside the controller home is called.

Ok, we've got the code, but how to display it to the user?
In the controller method, there should be a return View(). You can return View("name-of-the-view")

Now you have the request processed in the right place, and you have your view. Sweet! :D
Buuuuuuuuut, something is still missing... and that's because you have to connect those 2.
And this is called the mighty ViewBag!

View bag - speaking like mr jobs would say - is a magical, fantastic... the better ever created object to carry values between controller and view.
So how do you use this thing?
Easy:
{{< highlight csharp >}}
ViewBag.YourVariable = "The title";
{{< / highlight >}}

and in the view you can use it like this:
<title>@ViewBag.YourVariables</title>

By the way, this brings up another topic:RAZOR

Once again: this is magical, fantastic, amazingly simple. you can put C# code in the view just by adding a @ in front of it.
And, if you want to put a chunk of code, just use
{{< highlight csharp >}}
@{
  //several lines of code
}
{{< / highlight >}}

and how about the model?
Well... the model is the same as before, I won't talk about it here.
But that's it for now.
I'll write more about razor.