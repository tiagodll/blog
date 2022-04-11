---
title: PureScript web app for dummies - part 4
date: 2022-04-11T17:30:00.000Z
draft: false
tags:
  - PureScript
  - purs
  - spork
featured_image: https://upload.wikimedia.org/wikipedia/commons/6/64/PureScript_Logo.png
description: A dead simple guide of how to make a web app using purescript
date updated: 2022-04-11 17:30
---

![https://www.purescript.org/img/logo.svg](https://www.purescript.org/img/logo.svg)

# recap

[In the previous post](https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-2/), we created a little client side app to create, update and delete notes
It works well, but it is not very useful because nothing is being persisted

Now we are going to add a database to the server

# index for the PureScript web app for dummies series

| Date        | Description                         | Link                                                                        |
| ----------- | ----------------------------------- | --------------------------------------------------------------------------- |
| 02 Mar 2022 | set up a basic HTTPure server       | <https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-1/> |
| 07 Mar 2022 | add a client side app               | <https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-2/> |
| 31 Mar 2022 | add a database, API, and unit tests | <https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-3/> |
| 11 Apr 2022 | connect frontend client to the api  | <https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-4/> |

# Helpers

[The ajax method is a bit verbose, so it make sense to have some helpers (ajax/json)

```haskell
request :: ∀ a b. WriteForeign a => Decode b => Method.Method -> Request a -> Aff (M.Response b)
request method req = 
  liftAff $ 
    Affjax.request 
      (Affjax.defaultRequest 
        { method = Left method
        , headers = maybe [] (\t -> [RequestHeader.RequestHeader "auth.sig" t]) req.token
        , url = req.url
        , content = Just $ RequestBody.string $ writeJSON req.content
        , responseFormat = ResponseFormat.string
        }
      ) >>= case _ of

    Left error -> 
      pure $ M.ResponseError (ApiTransportError (Affjax.printError error))
    Right response -> pure case response.status of
      StatusCode 200 -> case readResponse (JSON response.body) of
        Just content -> content
        Nothing -> M.ResponseError (ApiError response.body)
      _ -> M.ResponseError (ApiError if response.body == "" then response.statusText else response.body)
```

This, combined with the json helper, will automate the parsing, so you just focus on sending/receiving your objects and forget about the serialization.

# Model

To work with our automated serialization, we should have the types that support it

```haskell
-- The response type is being use by the helper to parse the api results
data Response a
  = ResponseError ApiError
  | Response a
 
-- to make it even more readable, we just create an alias for each api response
type GetNotesResponse = Response (Array Note)

derive instance Generic (Response a) _
derive instance Functor Response
instance Encode a => Encode (Response a) where
  encode s = genericEncode defaultOptions s
instance Decode a => Decode (Response a) where
  decode s = genericDecode defaultOptions s
instance Decode a => ReadForeign (Response a) where
  readImpl = decode
instance Encode a => WriteForeign (Response a) where
  writeImpl = encode 
```

# Api

The changes in the api are minimal
We are just going to set the Response type for our serialization/deserialization.

```haskell
getNotes :: String -> HTTPure.ResponseM
getNotes token = do
  notes <- DB.getNotes token
  HTTPure.ok $ JSON.writeJSON (M.Response notes)
```

# Client

Now we want our client to load the notes from our api.

To be able to do that, 3 changes are required:

- add an actionAff, because loading from API is a side effect
- add an action to handle the return
- call this actionAff in the application load

With our new helpers, the code is fairly straightforward

```haskell
-- add to the ActionAff the request action
| GetNotes String (M.GetNotesResponse -> next)
-- just call the ajax helper
GetNotes token next -> do
    notes <- Ajax.get { url: "/api/notes", token: Just token, content: { } }
    pure $ next notes
	
-- add the return to the the Actions
| NotesLoaded M.GetNotesResponse
-- the loaded action will be matched as an error, or a response
NotesLoaded reponse -> case reponse of
    M.ResponseError e -> purely model { error = Just (show e) }
    M.Response r -> purely model { notes = r }

-- add the new ActionAff event to load the notes in the init
app ∷ App.App ActionAff (Const Void) Model Action
app = { update, render, subs: const mempty, init: {model: initModel, effects: App.lift ((GetNotes "token") NotesLoaded)} }
```

Now the next steps are to do the same to save and delete notes.

The code is available in [the tag v6 of our pnotes repo](https://github.com/pnorco/pnotes/releases/tag/v6).

# Conclusion

This concludes are very basic guide of how to make a webapp using purescript.

I know, many things are missing, there are no users, the token is just the hardcoded token string, but the point is: all this features are build on top of what we already have here.

The objective of this guide was to give you the minimum to have a real world website up and running, because starting with a new language can be very frustrating when you are stuck with technical details in the basics.

Functional programming can be very challenging, not because it is hard, but because object oriented being our current standard, it takes a lot of effort to change the way you approach problem solving. 

But it is worth the effort. Even if you will never switch to a functional language, the mainstream languages (such as java, C#, javascript...) are adding more and more functional features.

### Special thanks
This posts would not be possible without the help from my good friend, and purs mentor [Jesse](https://github.com/j-nava). Take a look on his github.
