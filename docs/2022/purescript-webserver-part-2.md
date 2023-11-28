---
title: "PureScript web app for dummies - part 2"
date: 2022-03-07T23:00:00+02:00
draft: false
tags: ['PureScript', 'purs', 'spork']
description: A dead simple guide of how to make a web app using purescript - part 2
featured_image: https://upload.wikimedia.org/wikipedia/commons/6/64/PureScript_Logo.png
---

![https://www.purescript.org/img/logo.svg](https://www.purescript.org/img/logo.svg)

# recap

[In the previous post](https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-1/), we set up a basic HTTPure server

Now we are going to add a client side for this app using spork

# PureScript web app for dummies series
| Date | Description | Link |
|------|-------------|------|
| 02 Mar 2022 | set up a basic HTTPure server | [https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-1/](https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-1/) |
| 07 Mar 2022 | add a client side app | [https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-2/](https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-2/) |
| 31 Mar 2022 | add a database, API, and unit tests | [https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-3/](https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-3/) |
| 11 Apr 2022 | connect frontend client to the api  | <https://tiago.dalligna.com/2022/04/purescript-web-app-for-dummies-part-4/> |

# preparing

Move the previous purs files to new folder called Server (inside src)
Create a new folder called Client, and paste [this file](https://github.com/natefaubion/purescript-spork/blob/master/examples/counter/src/Main.purs) from spork's examples:

```haskell
module Client.Main where

import Prelude

import Effect (Effect)
import Spork.Html (Html)
import Spork.Html as H
import Spork.PureApp (PureApp)
import Spork.PureApp as PureApp

type Model = Int

data Action = Inc | Dec

update ∷ Model → Action → Model
update i = case _ of
  Inc → i + 1
  Dec → i - 1

render ∷ Model → Html Action
render i =
  H.div []
    [ H.button
        [ H.onClick (H.always_ Inc) ]
        [ H.text "+" ]
    , H.button
        [ H.onClick (H.always_ Dec) ]
        [ H.text "-" ]
    , H.span []
        [ H.text (show i)
        ]
    ]

app ∷ PureApp Model Action
app = { update, render, init: 0 }

main ∷ Effect Unit
main = void $ PureApp.makeWithSelector app "#app"
```

# basics of spork (elm architecture)

Lets now look at this code and we can understand the elm architecture

- **main**: this is just to call the app, and assign it to the div with the ID "app"
- **app**: this defines our app, and it uses 3 parameters:
	- **model**: all data of the app should be inside this model
	- **render**: the function that generates the html based on the model
	- **update**: the function handles any event in the app
		- the data Action is like an enum for the possible events in the app
		- the type Model represents the data in the app, usually a record, but in this case just an Int

and there you have it
it is this simple
If you would like to see more on a similar topic, [check out this post I wrote about bolero](https://tiago.dalligna.com/2021/08/bolero/), that is also an elmish architecture, but in F#

Many times a new library will be required, I will not write here every time
The error message from spago will show you exactly the install command you have to run like this:

```bash
spago install [the_lib_name]
```


## build

Now since we have 2 projects, our build script is a little more complex.

Purs compiler generates javascript modules of the CommonJS type,
thus a bundler is required to run it in the browser.

Our bundler of choice is esbuild.
Here is how you can set up the build for the project.

```json
"build": "spago build && npx esbuild build/client.js --bundle --minify --outfile=src/wwwroot/index.js",
"server": "npm run build && spago run -m Server.Main",
```

then create a folder called build and add this files:

client:
```javascript
var main = require("../output/Client.Main/index.js");
main.main();
```
server:
```javascript
var main = require("../output/Server.Main/index.js");
main.main();
```

and lastely run:
```bash
npm run server
```

now you have a server and a client
hurray!

[this is the github commit](https://github.com/pnorco/pnotes/tree/a98abee76f9028a7231c6c81df675f1287033d43) with current version of the code

# adding some functionality to the client

we can start by updating our model to have a list of notes
```haskell
type Model = 
  { currentNote :: Maybe Note
  , notes :: Array Note
  }

type Note =
    { title :: String
    , content :: String
    , createdAt :: String
    }
```

now remember, our GUI is just reflecting our model
so that means if we want to have a form to add/edit notes, that should be in the model too.

But we can't just have a note value, otherwise something would be being edited all the time.
Instead we are going to add a Maybe Note
and this way, we only show the form, when there is a value. 

We only show the form to add/edit a note when currentNote is not Nothing
```haskell
renderNote :: Note -> Html Action
renderNote note =
  H.li [H.classes ["note"]] 
    [ H.i [H.classes ["mdi", "mdi-delete", "small", "delete"], H.onClick (H.always_ $ Delete note)] []
    , H.div [H.classes ["createdAt"]] [H.text note.createdAt]
    , H.div [H.classes ["title"]] [H.text note.title]
    , H.div [H.classes ["content"]] [H.text note.content]
    ]

-- and this is how we can show the form or a button to add when there is nothing being edited

case model.currentNote of
      Nothing -> 
        H.button [ H.onClick (H.always_ Add) ] [ H.text "+" ]
      Just note -> 
        H.div [ H.classes ["note"] ] 
        [ H.input [ H.type_ H.InputText, H.classes ["content"], H.onValueInput (H.always UpdateNoteTitle) ]
        , H.input [ H.type_ H.InputText, H.classes ["date"], H.onValueInput (H.always UpdateNoteDate) ]
        , H.textarea [ H.classes ["content"], H.onValueInput (H.always UpdateNoteContent)]
        , H.button
          [ H.onClick (H.always_ Save) ]
          [ H.text "save" ]
        ]
```

Once the model and the view are done, all you need is the update method to handle the events fired by the user interacting with the GUI:
```haskell
data Action 
  = Initialize
  | Add -- will create a new note as current
  | Save -- will add currentNote to the list of notes
  | Delete Note -- remove note from list
  | UpdateNoteTitle String
  | UpdateNoteContent String
  | UpdateNoteDate String
  
  
update method:
  UpdateNoteTitle str -> 
    let note = fromMaybe {title:"", content:"", createdAt: "0"} model.currentNote
    in model { currentNote = Just note{ title=str } }
  UpdateNoteContent str ->
    let note = fromMaybe {title:"", content:"", createdAt: "0"} model.currentNote
    in model { currentNote = Just note{ content=str } }
  UpdateNoteDate str ->
    let note = fromMaybe {title:"", content:"", createdAt: ""} model.currentNote
    in model { currentNote = Just note{ createdAt=str } }
```

This is all great, and it works
**BUT** we still dont have the creation date
and this is a much bigger deal than it sounds

You see, in purescript functions are pure by default.
If you want to introduce **any** side effect, you have to change **the whole call stack** to the Aff or Effect monad.

>Why is this a good thing? you may ask
Because pure functions, for a given input will **always** return the same outputs
That means your unit tests are simpler and more reliable.
There is also a little bonus: based on your code, in some cases, pure function can be replaced by values at compiled time (that is called referencial transparency).

In our case, the creation date will be the primary key of our note.
Yeah, I know, this is not good
**BUT** this is just a simple app for educational purposes
dont judge me!

Anyway, we are now using a feature that returns an Effect
therefore the update method must be an Effect
therefore the app should be an Effect
therefore main should be an Effect as well

so you see, that is why it changes everything
the good news is that once this is done, you can now introduce as many effects as you want

Well, there is one more thing:
In order to keep the update function pure, this events should be handled in a separate update code. Like this:

```haskell
data Action 
  = Initialize
  | Add
  | Added String
  | Focused String
  | Edit Note
...

update ∷ Model -> (Action -> Transition ActionAff Model Action)
update model = case _ of
...
  Add ->
    { model: model, effects: App.lift (AddAff Added) }
  Added date ->
    let current = fromMaybe {title:"", content:"", createdAt: "0"} model.currentNote
    in { model: model {currentNote = Just current {createdAt=date}}, effects: App.lift ((Focus "newNoteTitle") Focused) }
  Focused _ ->
    purely model
  Edit note ->
    { model: model {currentNote = Just note}, effects: App.lift ((Focus "newNoteTitle") 
	
...

data ActionAff next
  = AddAff (String -> next)
  | Focus String (String -> next)

updateAff ∷ ActionAff ~> Aff
updateAff aff = case aff of
  AddAff next -> do
    createdAt <- liftEffect nowDateTime
    let createdStr = displayDatetime createdAt
    pure $ next createdStr
  Focus id next -> do
    _ <- liftEffect $ FFI.focus id
    pure $ next id

displayDatetime :: DateTime -> String
displayDatetime d = 
    fromRight "no date" (formatDateTime "YYYY-MM-DD HH:mm:ss" d)

app ∷ App.App ActionAff (Const Void) Model Action
app = { update, render, subs: const mempty, init: purely initModel }

main ∷ Effect Unit
main = do
  let interpreter = throughAff updateAff handleErrors
  inst <- App.makeWithSelector (interpreter `merge` never) app "#app"
  inst.run
```

Lets see what happened here:

to allow effects, we changed our main function, and now we use an App.App

we modified our update method to update ∷ Model -> (Action -> Transition ActionAff Model Action)

that means we now have to return "purely model", or a record that has {model: x, effects: y }
where the effects are a tuple containing the **ActionAff** to be ran, and the callback **Action**.

This **ActionAff**s are handled in a special update method, called **updateAff**.
In this function, every ActionAff has a next parameter
this next is the callback
also, every return has a pure in front
this pure is the purescript way to say: 
- Now you take the response type monad of this funciona, and apply it to the result

but more on this in a future post.

# conclusion

At this point you have a basic server and a client.
The client is being compiled into a js file, dropped inside the wwwroot folder, and loaded into a div with id app inside the index page.

The code is available in [the tag v4 of our pnotes repo](https://github.com/pnorco/pnotes/releases/tag/v4).

So far, so good.

But shouldn't we save this notes somewhere?

Yes we should.
But that is part of the next part of our serie.

# side note

But what do you think, was this complicated?
I know it seems like a lot at first,
but it gets easier overtime

Unfortunately the error messages from purs are not very helpful,
and are extremely confusing for beginers

> This makes it even more important to have a helpful comunity.
> I was lucky enough to have a close friend helping me with every step
> still a beginer, I often still need some support

> And this is why I started this serie of posts
> I feel there is a lot of articles exlaining what is a monad
> but not enough explaining how to build something like a webapp

I hope this is being helpful and see you in the next post
