---
title: "Css expandabox"
date: 2013-08-15T22:21:43+02:00
tags: ['css']
draft: false
---

Those of you following my posts for a while might know that I like javascript.
I like javascript as much as I like not using javascript when I can use css instead.

So trying to make an expandable box I realised its behaviour is very similar to a checkbox. you have the small excerpt and when you click it you get the full text.

That being said, it is possible not only to change the label properties according to its state, but to set its state by clicking the label.

{{< highlight html >}}
<style>
  input[type="checkbox"].expand-content{ display:none; }

  input[type="checkbox"].expand-content + label .collapsed { display:block; }
  input[type="checkbox"].expand-content + label .expanded { display:none; }

  input[type="checkbox"].expand-content:checked + label .collapsed { display:none; }
  input[type="checkbox"].expand-content:checked + label .expanded { display:block; }
</style>

<input class="expand-content" id="theradio" type="checkbox" />
<label for="theradio">
    <div class="collapsed">
this is the small content</div>
<div class="expanded">
this is the big content</div>
</label>
{{< / highlight >}}

This is also available on [my gist profile](https://gist.github.com/tiagodll/6164290#file-css-expandbox-html)