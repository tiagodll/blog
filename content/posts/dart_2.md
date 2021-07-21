---
title: "Dart + start + polymer = ?"
date: 2014-08-10T17:06:06+02:00
draft: false
---


So as I said in my <a href="/2014/07/yeah-about-that.html">Dart, what do we think about it?</a> post, it is an interesting language to look into. It is lacking good libraries, but a problem can also be seen as an opportunity: it is a great time build your own.

Back to the topic, because of the pub system it is very easy to have dart + start <b>or</b> dart + polymer, but not as easy to have both.

There is a teeny-tiny detail it took me a couple of days to realize, but let me save this to the end of the post.

Start is simple, lets begin with that.
first, the pubspec

{{< highlight dart >}}
<code file="pubspec.yaml">
name: onemore
description: A sample Polymer application
dependencies:
  polymer: '>=0.11.0'
  start: any
transformers:
- polymer:
    entry_points: web/index.html
</code>
{{< / highlight >}}


Then all you need to do is to import the package and create a server side dart file to start it.
In this example I am only serving static files, it would be just as easy to handle dynamic content with the get(url).
{{< highlight dart >}}
<code file="app.dart">
import 'package:start/start.dart';

main(args) {
  start(port: 3000).then((Server app) { app.static('build/web', jail: false); });
}
</code>
{{< / highlight >}}

Done, you have a web server running in http://localhost:3000 as soon as you run this code.

Now, the polymer part
to have a custom element we need 3 files: the dart, the html and the hosting html file.
this custom element expects a String list, and publish (sends to the client) both lst and plist.
{{< highlight dart >}}
<code file="elementlist.dart">
import 'package:polymer/polymer.dart';
import 'dart:convert';

@CustomTag('element-list')
class ElementsList extends PolymerElement {
  @published String list;
  @published List plist;

  ElementsList.created() : super.created(){
    try{
      plist = JSON.decode(list);
      plist.add({"id":"test"});
    }catch(e){
      print(e.toString());
    }
    polymerCreated();
  }
}
</code>
{{< / highlight >}}

Here in the html we can use mustache style expressions
{{< highlight html >}}
<code file="elementlist.html">
<link rel="import" href="packages/polymer/polymer.html">
<polymer-element name="element-list" attributes="list">
  <template>
    <div>the list: {{ list }}</div>
    <ul>
      <template repeat="{{elmt in plist}}">
        <li>{{elmt["id"]}}</li>
      </template>
      </ul>
  </template>
  <script type="application/dart" src="elementlist.dart"></script>
</polymer-element>
</code>
{{< / highlight >}}

and then your index file you just have to add some references and include the custom element element-list.
{{< highlight html >}}
<code file="index.html">
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sample app</title>
    <script src="packages/web_components/platform.js"></script>
    <script src="packages/web_components/dart_support.js"></script>
    <!-- import the click-counter -->
    <link rel="import" href="elementlist.html">
    <script type="application/dart">export 'package:polymer/init.dart';</script>
    <script src="packages/browser/dart.js"></script>
  </head>
  <body unresolved touch-action="auto">
    before list<br>
    ---------------------<br>
    <element-list id="the_list" list='[{"id":1},{"id":2},{"id":3}]'></element-list>
    ---------------------<br>
  </body>
</html>
</code>
{{< / highlight >}}

Now, let me explain the little detail that made me lose so much time: start is just serving the folder specified above. The problem is that all the polymer javascript is generated in the build, and stored there. So our start must serve that folder instead.
to do that, all we have to do is to change the web to build/web in the start method:
{{< highlight dart >}}
app.static('build/web', jail: false);
{{< / highlight >}}

That is all folks, hope you enjoy it.
