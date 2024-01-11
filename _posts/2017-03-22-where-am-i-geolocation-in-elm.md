---
title: Where Am I? Geolocation in Elm
author: Juan Pedro Villa Isaza
tags: elm programming
layout: post
---

[Elm][elm] is a simple and functional language that compiles to
JavaScript. These are my notes about getting started with Elm using a
small but complete application to get the current location of a device
and update it based on movement.

{% assign elm-packages = "http://package.elm-lang.org/packages/" %}

{% assign elm-lang = "elm-lang/" | prepend: elm-packages %}

{% assign core = "core/5.1.1/" | prepend: elm-lang %}
{% assign geolocation = "geolocation/1.0.2/" | prepend: elm-lang %}
{% assign html = "html/2.0.0/" | prepend: elm-lang %}

{% assign attempt = "Task#attempt" | prepend: core %}
{% assign changes = "Geolocation#changes" | prepend: geolocation %}
{% assign error = "Geolocation#Error" | prepend: geolocation %}
{% assign location = "Geolocation#Location" | prepend: geolocation %}
{% assign maybe = "Maybe#Maybe" | prepend: core %}
{% assign never = "Basics#Never" | prepend: core %}
{% assign now = "Geolocation#now" | prepend: geolocation %}
{% assign program = "Platform#Program" | prepend: core %}
{% assign result = "Result#Result" | prepend: core %}
{% assign task = "Task#Task" | prepend: core %}

First of all, we have to install Elm. There are several ways to do that---I used
the [npm package][elm-npm]:

```
$ npm install --global elm
```

And this is the version I got:

```
$ elm --version
0.18.0
```

We also need a directory for the application:

```
$ mkdir whereami
$ cd whereami/
```

And an Elm project:

```
$ elm make --yes
```

These are the packages I got:

- [core 5.1.1]({{ core }})
- [html 2.0.0]({{ html }})

To do geolocation in Elm, we can use the geolocation package:

```
$ elm package install elm-lang/geolocation --yes
```

This is the version I got:

- [geolocation 1.0.2]({{ geolocation }})

Now, let's create an Elm file for the application:

```
$ touch WhereAmI.elm
```

And run Elm:

```
$ elm reactor
elm-reactor 0.18.0
Listening on http://localhost:8000
```

You can go there and open the Elm file, which shouldn't work... To fix
the problem, we can add a list of imports:

```elm
import Geolocation exposing (Location)
import Html exposing (Html)
import Task
```

And an empty model:

```elm
type alias Model =
  {}
```

And reload. Every time we do something, reload.

In short, an [Elm application][elm-architecture] consists of a model,
which represents the state of the application, and ways to update and
view the model. And that's it... Or most of it. Let's take a look at
each part while we build the geolocation application.

The basic idea of the application is to get and update the location of
a device, so the model could be a [location]({{ location }}):

```elm
type alias Model =
  Location
```

But there's no location at the beginning, so the model could be an
[optional]({{ maybe }}) location instead:

```elm
type alias Model =
  Maybe Location
```

Also, there could be an [error]({{ error }}) if the user denies
permission to access the location or if the location is unavailable,
so a better model could be the [result]({{ result }}) of either an
error or an optional location:

```elm
type alias Model =
  Result Geolocation.Error (Maybe Location)
```

A cool thing about this model is that it
makes [impossible states impossible][impossible]: there's no way to
have both an error and a location or more than one location---which
could be possible states of a different application. Also, it's not
the only possible model for this application, but it's good enough for
me.

Now, there's always a model, so we need to declare the initial one:

```elm
init : (Model, Cmd msg)
init =
  ( Ok Nothing
  , Cmd.none
  )
```

There's no location at the beginning and there's no command for Elm
yet, which means that nothing will happen after initializing the
application. Of course, something should happen to get the current
location of the device, but we'll come back to that.

To change the model, we need messages to communicate with the
application:

```elm
type Msg
  = Update (Result Geolocation.Error Location)
```

That is, a message is an update with either an error or a location,
which is almost the same as the model.

But a message by itself is useless: given a message and a model, what
is the new model and should something happen after updating the
model?:

```elm
update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
  case msg of
    Update (Err err) ->
      ( Err err
      , Cmd.none
      )

    Update (Ok location) ->
      ( Ok (Just location)
      , Cmd.none
      )
```

In the case of an error, the new model is the error, and, in the case
of a location, the new model is just the location. There's no command
for Elm in either case. To simplify:

```elm
update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
  case msg of
    Update result ->
      ( Result.map Just result
      , Cmd.none
      )
```

That is, if the result in the update message is a location, the new
model is the location wrapped inside a `Just`. Otherwise, the new
model is the result.

This is still useless, though. The command in the `init` function
should trigger an update after getting the current location of the
device, and we should declare subscriptions to trigger updates based
on movement:

```elm
subscriptions : Model -> Sub Msg
subscriptions _ =
  Sub.none
```

There are no subscriptions yet, but we'll come back to that.

To recap, we now have a model and a way to update it. Let's take a
look at how to view the model as HTML:

```elm
view : Model -> Html Msg
view model =
  Html.div
    []
    [ Html.h1
        []
        [ Html.text "Where Am I?" ]
    , Html.div
        []
        [ Html.text (toString model) ]
    ]
```

This is just a title and the model as a string.

To wrap things up, we need a [program]({{ program }}) to describe the
application to Elm. In other words, we need to explicitly tell Elm
what the model is, what the messages are, how to initialize the model,
and so on:

```elm
main : Program Never Model Msg
main =
  Html.program
    { init = init
    , subscriptions = subscriptions
    , update = update
    , view = view
    }
```

The only thing that shouldn't make sense here is
the [`Never`]({{ never }}), which corresponds to JavaScript flags that
can be used to initialize the model. In this case, we're telling Elm
that the application never takes flags.

If you reload, you should see something like this:

```
# Where Am I?

Ok Nothing
```

OK... But something should happen...

After initializing the application, there should be a command to get
the current location of the device. The geolocation package exposes
a [task]({{ now }}) to do that:

```elm
Geolocation.now : Task Geolocation.Error Location
```

A [task]({{ task }}) describes an operation that could fail. To use a
task, we can [attempt]({{ attempt }}) to do it:

```elm
Task.attempt : (Result x a -> msg) -> Task x a -> Cmd msg
```

That is, given a way to turn a result into a message and a task, we
get a command to attempt to do the task and turn the result into a
message that will then be sent to the application and used to update
it.

Let's create a command to get the current location of the device (note
that the type of commands is now `Cmd Msg` and not `Cmd msg` because
we're now using the message type we created):

```elm
init : (Model, Cmd Msg)
init =
  ( Ok Nothing
  , Task.attempt Update Geolocation.now
  )
```

In other words, after initializing the application, Elm should attempt
to get the current location using the `now` task and then send the
result to the application as an update message.

If you reload, you should see something like this (ignoring the
title):

```
Ok (Just { latitude = 3.1003322, longitude = -43.3334241, ... })
```

Unless you block access to the location:

```
Err (PermissionDenied "User denied Geolocation")
```

If you want to try it on a phone or some other device, set the address
for the server:

```
$ elm reactor --address 0.0.0.0
```

But note that some browsers only allow secure origins when doing
geolocation.

To update the location, reload. But to actually update it based on
movement, we need a subscription to location changes. The geolocation
package exposes a [subscription]({{ changes }}) to do this:

```elm
Geolocation.changes : (Location -> msg) -> Sub msg
```

Given a way to turn a location into a message, we get a subscription
to location changes which will turn locations into messages that will
then be sent to the application and used to update it. Let's do that:

```elm
subscriptions : Model -> Sub Msg
subscriptions _ =
  Geolocation.changes (Update << Ok)
```

Note that this is first putting the location inside an `Ok` and then
turning that into an update message.

If you reload, you should see the same thing, but the location should
change. If it doesn't, try switching tabs... Or moving.

Finally, we can compile the Elm file to JavaScript:

```
$ elm make WhereAmI.elm --output whereami.js
```

And then embed it in HTML:

```html
<div id="whereami"></div>

<script src="whereami.js"></script>
<script>
  Elm.Main.embed(document.getElementById("whereami"));
</script>
```

Now, instead of running Elm, open the HTML file.

[elm]: http://elm-lang.org/
[elm-architecture]: https://guide.elm-lang.org/architecture/
[elm-guide]: https://guide.elm-lang.org/
[elm-npm]: https://www.npmjs.com/package/elm
[impossible]: https://youtu.be/IcgmSRJHu_8
