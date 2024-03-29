---
title: "did I run my migrations?"
date: 2015-04-22T16:19:28Z
draft: false
tags: ['ruby', 'db']

featured_image: https://upload.wikimedia.org/wikipedia/commons/6/62/Ruby_On_Rails_Logo.svg
description: migrations are awesome, but only when you run them
---

 Today I had an error message driving me crazy:

```bash
 `method_missing': undefined method `export_format' for XXX model
```

 You see, our model is a gem itself, because it is shared by several projects.
 When we have something in the schema that is not in the database, that is the error we get.


 Now, I knew this error from previous executions, but my model was in synch with the DB.

 I tried to comment out all my changes, without luck.


 Until I realized. This error was in my specs.

 I had run the rake db:migrate, but not RACK_ENG=test

 In this situation I was happy I fixed the issue, but at the same time extremely angry I wasted 1h with that


 Mental note: run both friggin migrations together in the future!


 This might help:
```bash
alias migrate='rake db:migrate; rake db:migrate RAILS_ENV=test'
```

 thanks to Eumir for this [post](http://blog.aelogica.com/tutorial/migrating-development-db-and-test-db-simultaneously/)
