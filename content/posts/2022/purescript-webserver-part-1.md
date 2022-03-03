---
title: "PureScript web app for dummies"
date: 2022-03-02T18:00:00+02:00
draft: false
tags: ['PureScript', 'purs', 'spork']
description: A dead simple guide of how to make a web app using purescript
featured_image: https://upload.wikimedia.org/wikipedia/commons/6/64/PureScript_Logo.png
---

![https://www.purescript.org/img/logo.svg](https://www.purescript.org/img/logo.svg)

# setup

Hello folks, are you interested in purescript?
Yes?
Then I have something you will definitely enjoy: HTTPure + spork = web application
but just in case you dont have it installed in your machine yet

```bash
npm install purescript spago --global
```

# Init

you start with a basic package:
- pacakge.json => the npm package usual stuff
- packages.dhall => list of purescript packages (it can contain direct reference to the repository)
- spago.dhall => purescript dependency file (well, spago, but you understand my point right?)
- src/Main.purs => entry point for our webserver
- test/Main.purs => tests file

you can download this files here: [in our github repo](https://github.com/pnorco/pnotes/tree/256f5671f9cc3618201087b3dbebd8341d6f9730)
but you don't have to.

Here are the steps to get there:

```bash
spago init
spago run Main
```

now you have a purescript project
but it doesnt do anything
so we modify our main to listen to the port 8080

```haskell
module Server.Main where

import Prelude

import Effect.Class.Console (log)
import HTTPure (ServerM, serve, ok)

main :: ServerM
main = serve 8080 router $ log "Server now up on port 8080"
  where
    router _ = ok "hello world!"
```

now add   "httpure" in your list of dependencies:

```bash
spago install httpure
spago run Main
```

browse http://localhost:8080 and you have a web server
not a very useful one though...

# Adding a Router

Lets start improving this a bit by adding a rounter:

```haskell
main :: Effect Unit 
main = launchAff_ do
  let options = {hostname: "localhost", port: 8080, backlog: Nothing}
  liftEffect 
    $ HTTPure.serve' options router
    $ log ("Server now up @ " <> options.hostname <> ":" <> show options.port)


router :: HTTPure.Request -> HTTPure.ResponseM
router { body, headers, method, path } = case method, path of
  
  HTTPure.Get, [ ] -> HTTPure.ok $ "hello world"
  HTTPure.Get, [ "ping" ] -> HTTPure.ok $ "pong"

  _, _ -> do
    log $ "Not found: " <> show path
    HTTPure.unauthorized
```

now you will need 
```bash
spago install node-fs-aff
spago install arrays
spago install maybe
spago install tuples
```

# Rendering static assets

ok, things are starting to get more interesting now
but still it is not that great
lets render some assets (images/css/js/etc)

This means reading the files

```haskell
serveFile :: String -> HTTPure.ResponseM
serveFile path = (liftAff $ FS.exists path) >>= case _ of
  false -> HTTPure.notFound
  true -> (liftAff $ FS.readFile path) >>= (HTTPure.ok)
```

and sending to the browser
```haskell
HTTPure.Get, [ "static", filename ] -> serveFile ("src/wwwroot/" <> filename)
```


now Main is starting to get too big, so lets split router into it's own file

```haskell
module Server.Router where

import Prelude

import Data.Array (length, take)
import Data.Tuple.Nested ((/\))
import Effect.Aff.Class (liftAff)
import Effect.Class.Console (log)
import Node.FS.Aff (exists, readFile) as FS
import HTTPure as HTTPure


router :: HTTPure.Request -> HTTPure.ResponseM
router { body, headers, method, path } = case method, path of
  
  HTTPure.Get, [ "ping" ] -> HTTPure.ok $ "pong"
  HTTPure.Get, [ ] -> serveFile' "text/html" "src/wwwroot/index.html"
  HTTPure.Get, [ "static", filename ] -> serveFile ("src/wwwroot/" <> filename)

  HTTPure.Get, _ 
    | startsWith path [ "../" ] -> HTTPure.unauthorized

  _, _ -> do
    log $ "Not found: " <> show path
    HTTPure.unauthorized

startsWith :: Array String -> Array String -> Boolean
startsWith s t = t == (take (length t) s)

serveFile' :: String -> String -> HTTPure.ResponseM
serveFile' contentType path = (liftAff $ FS.exists path) >>= case _ of
  false -> HTTPure.notFound
  true -> (liftAff $ FS.readFile path) >>= (HTTPure.ok' (HTTPure.headers [ "Content-Type" /\ contentType ]))

serveFile :: String -> HTTPure.ResponseM
serveFile path = (liftAff $ FS.exists path) >>= case _ of
  false -> HTTPure.notFound
  true -> (liftAff $ FS.readFile path) >>= (HTTPure.ok)
  
```


[here you see the updated repo](https://github.com/pnorco/pnotes/tree/a2313f7975566b3d18c91a0473c321f0424866e6)

# Wrap up

In this post I tried to help you go through the very basics of how to make a purescript web server 
It is the first post of a series that will take this into a functional note taking app, with sqlite database and spork as the client side

I hope you have enjoyed and till next one
