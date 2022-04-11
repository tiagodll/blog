---
title: "PureScript web app for dummies - part 3"
date: 2022-03-31T20:00:00+02:00
draft: false
tags: ['PureScript', 'purs', 'spork']
featured_image: https://upload.wikimedia.org/wikipedia/commons/6/64/PureScript_Logo.png
description: A dead simple guide of how to make a web app using purescript # 3
---

![https://www.purescript.org/img/logo.svg](https://www.purescript.org/img/logo.svg)

# recap

[In the previous post](https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-2/), we created a little client side app to create, update and delete notes

It works well, but it is not very useful because nothing is being persisted

Now we are going to add a database to the server

# PureScript web app for dummies series
| Date | Description | Link |
|------|-------------|------|
| 02 Mar 2022 | set up a basic HTTPure server | [https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-1/](https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-1/) |
| 07 Mar 2022 | add a client side app | [https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-2/](https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-2/) |
| 31 Mar 2022 | add a database, API, and unit tests | [https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-3/](https://tiago.dalligna.com/2022/03/purescript-web-app-for-dummies-part-3/) |
| 11 Apr 2022 | connect frontend client to the api  | <https://tiago.dalligna.com/2022/04/purescript-web-app-for-dummies-part-4/> |

# Prepare your project to sqlite

step 1: add sqlite to spago AND npm

```haskell
spago install sqlite3
npm install sqlite3
```

step 2: add *.db to your gitignore (you dont want to save your database in git). 

# Add a database access file

I will not paste the whole file here, but only the most important bits instead.

We start with some helper functions.

```haskell
module Server.Database where
...
-- This type is just a to add the Aff monad
type InitDBType a = forall m. MonadAff m => m a

-- So now we return our InitDBType together with the connection because of the Aff
openDb :: String -> InitDBType Sqlite.DBConnection
openDb filename = liftAff $ Sqlite.newDB filename

-- just a helper to make queries (return values)
query :: ∀ a. ReadForeign a => String -> Array String -> Aff (Array a)
query sql params = do
  db <- init
  liftAff (Sqlite.queryDB db sql (unsafeToForeign <$> params))
    <#> read
    >>> hush
    >>> fromMaybe []
	
-- same as query, but it ignores the response
run :: String -> Array String -> InitDBType Unit
run sql params = do
  db <- init
  liftAff $ void $ Sqlite.queryDB db sql (unsafeToForeign <$> params)

-- this is a poor man's migration runner
init :: InitDBType Sqlite.DBConnection
init = do
  let dbFilename = "notes.db"
  (liftAff $ FS.exists dbFilename) >>= case _ of
    false -> createDb dbFilename *> openDb dbFilename
    true -> openDb dbFilename
	
-- and here the poor man's migrations
createDb :: String -> InitDBType Unit
createDb filename = do
  db <- liftAff $ Sqlite.newDB filename
  liftAff $ Sqlite.closeDB db
  run "CREATE TABLE IF NOT EXISTS notes( title text, content text, createdAt text PRIMARY KEY);" []
```

This comments are not saved in the github. 
Should I put it there?
It feels a bit too much...

anyway, this is how to use it:
```haskell
-- the query result will be automatically parsed to the return type (Array Note)
getNotes :: String -> Aff (Array Note)
getNotes str =
  query "SELECT * FROM notes" [ ]

-- to run a command is similar, but the return type is just Unit
saveNote :: Note -> Aff Unit
saveNote note = 
  run
    """
      INSERT INTO notes (title, content, createdAt) VALUES (@title, @content, @createdAt)
      ON CONFLICT(createdAt) DO UPDATE SET title=@title, content=@content WHERE createdAt=@createdAt
    """
    [ note.title, note.content, note.createdAt ]
```

Create an api file, nothing special, just parse the parameters and call the db, such as:
```haskell
saveNote :: String -> String -> HTTPure.ResponseM
saveNote token body = case JSON.readJSON body of
  Right (note :: Note) -> do
    notes <- DB.saveNote note
    HTTPure.ok "ok"
  Left err -> HTTPure.internalServerError (show err)
```

Now we have to modify our Router to include this new Api endpoints

```haskell
router { body, query, headers, method, path } = do

  bodyString <- HTTPure.toString body

  case method, path of
    HTTPure.Get, [ "api", "notes" ] -> API.getNotes "token"
    HTTPure.Post, [ "api", "note" ] -> API.saveNote "token" bodyString
    HTTPure.Delete, [ "api", "note", id ] -> API.deleteNote "token" id
```

Pretty straightforward, just call the api

Except, while creating this, the new version of HTTPure is out, and body is not a string anymore.

So that is why we had to create the **bodyString** value

This is not too bad right?
but how do we test it?

well, we start with a simple Request using VS Code rest client:

```haskell
POST http://localhost:8080/api/note

{ "title": "the test note"
, "content": "bla bla bla"
, "createdAt": "2022-03-19T10:20:30"
}


GET http://localhost:8080/api/notes


DELETE http://localhost:8080/api/note/2022-03-19 
```

cool right?

well, not really... it works but by now we should start thinking about something that has been so neglected so far

# Unit testing

Unit testing in purs is quite simple. You add a lib, describe is just to group the similar tests.

For each test include an it section describing the test.

```haskell
main :: Effect Unit
main = launchAff_ $ runSpec [console reporter] do
describe "DB" do
it "insert" do 
let note = { title: "the test note", content: "bla bla bla", createdAt: "2022-03-19T10:20:30"} 
DB.saveNote note 
dbNotes <- DB.getNotes "" 
dbNotes `shouldContain` note
```
I will just leave this test code here and link to the github

It is pretty self explanatory

PS: a function surrounded by backticks can be place in between 2 parameters for improved readability.
ex:

```haskell
dbNotes `shouldContain` note
-- is the same as
shouldContain dbNotes note
```

# conclusion

This post thought you how to add an sqlite database to your app, adapt the api to serve it, and create unit tests.

The code is available in [the tag v5 of our pnotes repo](https://github.com/pnorco/pnotes/releases/tag/v5).

So far, so good.

But... why is my app is still not saving/loading/deleting in/from the database?
Well, our client side is not calling our API yet.
But this will be fixed in the next post.