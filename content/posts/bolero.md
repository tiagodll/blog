 What do you think about blazor?

what about Elm?

what if Elm architecture was running in blazor, but in F#?

Well, that is what bolero is about. 

 ![Fsharp](https://upload.wikimedia.org/wikipedia/commons/5/57/Fsharp_logo.png)

 ## Getting started
1. Install dotnet if you haven't already
2. go to https://fsbolero.io/ and follow the 3 easy steps to have your app up and running
3. Continue following this guide.


## Understanding the Elm architecture

Bolero is based on the elm architecture.

this is divided in 3 items:
- **Model**: it represents the current state of the app.
- **update**: this function the only way to change your model. 
- **render**: it is how your html will be rendered. 

Well, there is also the init function, where you give the initial state of the application.

But that is it. Anything else is built on top of this 3 items.

Simple, no?

![Elm architecture](https://guide.elm-lang.org/architecture/buttons.svg)


## Model
there is only one model, 
it is global (inside the component) and it contains every detail in the application.
your working data, cached values, state of the dropdowns and popups... everything

Example:
```
type Model =
    {
				page: Page
				date: DateTime
				error: string option
				items: Item list
				others: Other list
				touchInfo: TouchInfo
    }
```

## Update
Nothing can change in the model without being part of this method.
It is a blessing and a curse.
It is much easier to find out where something is being updated, but you end up with huge update methods.
Then you can start passing the update to another function.
This helps with the size, and it is ok if you are well organized. but it could also mess up your codebase.

```
let update message model =
    match message with
    | SetPage page -> { model with page = page }, Cmd.none
    | Error exn -> { model with error = Some exn.Message }, Cmd.none
    | ClearError -> { model with error = None }, Cmd.none
		...
```
## View
The view is called on every update to the model.
You can use functions to represent the html, and also include some logic to hide or display items based on the model.

Example:
```
let view model dispatch =
    div [attr.``class`` "columns"] [
        div[][text " x "]
        div[attr.``class`` "has-text-centered"][
            button[attr.classes ["button"; "is-small"]; on.click (fun _ -> dispatch PreviousDay)][text "<"]
            span[attr.classes ["dashboard-date"]][text (model.date.ToString("yyyy-MM-dd"))]
            button[attr.classes ["button"; "is-small"]; on.click (fun _ -> dispatch NextDay)][text ">"]
        ]
```


## Understanding monads

Nah, just kidding
I believe having this basics of functional programming everywhere is one of the reasons of why people never go deep into it.
Nobody needs yet another "let me explain functional for dummies"
If you are like me, you rather start a project and learn along with it.

## aha moments

There are a few moments where I got stuck, but then it all make sense

### Any update to the model is automatically reflected in the GUI
yes, you dont have to add any function

### The ONLY way to update the GUI is by changing the model
That means anything you want to change **must** be in the model. Even state of your dropdown, popup, etc

### You can use anything from C#
But please don't overuse it. Try to keep consistent using F# types and functions.

### I can modify the default functions
Just because the update function doesn't have a parameter I want, it doesn't mean it cannot have.

In this example I was stuck for 2 days trying to access the JavaScript runtime from the client side.

Then I found out I can pass my own update method with that parameter:
```
let update = update this.JSRuntime Program.mkProgram init update view
```

### I can easily interop with javascript from bolero client
The interop will not only call functions passing integers/strings
It will pass objects between javascript and bolero.

```FSharp
let LoadCheckins (js:IJSRuntime) (date:DateTime) = Cmd.OfJS.either js "FromStorage" [| "checkins_" + date.ToString("yyyy-MM-dd") |] CheckinsLoaded Error
```
