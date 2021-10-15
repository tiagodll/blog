---
title: "Go"
date: 2014-10-02T11:16:11+02:00
draft: false
tags: ['Go']
featured_image: https://upload.wikimedia.org/wikipedia/commons/2/23/Golang.png
description: My thoughts and experiences with go

---

![Go logo](http://golang.org/doc/gopher/frontpage.png "go logo")

C is fast, but boring,
Ruby is slow, but fun.
Go is right in between

Let's start with the fun bits:
- You don't import just a file, you import a repository.
Reading import "github.com/tiagodll/golib" is beautiful, that alone gets me excited about go.

- Returning multiple values. Yes, finally!
You can set resp, err := MyFunction()
- Ditching the ;. I never got the reason for it, maybe it is because I learned programming with VB, but to me it just doesn't make sense.

Now to the boring
- The name. Seems like nothing, but having a language called go makes google searches useless unless you use golang.
- Not having try/catch. It sucks. Although returning value, error makes up for it quite well.
- Having any not used lib/variable as an error instead of warning. It is ok when you have a final version, but that is really annoying while development phase.
And the worst of all: 
- the insane amount of extra lines to assign a variable such as
{{< highlight go >}}
temp := MyType{id:123}
GetValueFromMongo(&temp)
Printf("%v", temp)
Instead of a simple
puts get_value_from_mongo(123).to_s
{{< / highlight >}}

and the grey area
- Not having classes. I like the idea of having all static methods (unless it is really about the object they are in), but not having classes could have some issues, specially because of inheritance.

Regardless, this is my current language of choice.
But this is just the beginning. 
Let's see next what go is all about. 