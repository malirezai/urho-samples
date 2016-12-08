Inspecting UrhoSharp Game Demo
============

(run on Windows, Visual Studio 2015 Enterprise, VS Android Emulator)
---------

Thanks! Miguel showed you some interactive documentation that demonstrated how
easy it is to manipulate a 3D scene with UrhoSharp. What I'd like to show you
is how we can use the Xamarin Inspector to dive in and modify a running 3D game
built using UrhoSharp.

I've only been playing with UrhoSharp for a little while, but
it's really cool. I've been wanting to learn game programming, and this was the
perfect opportunity to get my feet wet.

I have here a sample game straight out of the UrhoSharp documentation. It runs
on Android, iOS, and Windows and Mac desktops. The UrhoSharp team has done a
really great job. Aside from initializing the game engine, 100% of the code is
shared. And it's really easy to use.

So this is how it works. I fly around in this plane, bad guys shoot at me,
and I try to collect as many coins as I can before I die.

But you know, I keep losing. And the more I play, I think maybe the problem is
that my plane is a bit too big. So I want to try adjusting the size. But I am
not a fan of rebuilding and restarting this app every time I want to adjust one
little number. So let's see what we can do...

***Clicks Inspector button in toolbar, Inspector window appears***

So here I have launched the Xamarin Inspector.

In a normal Android app, you might use this to inspect your views. But the
Inspector has no special knowledge of Urho, so it doesn't know how to populate
the visual tree or anything like that. So what good is it?

It turns out that just having access to a live C# REPL connected to your app
allows you to get a lot done in a single build cycle.

The Inspector has a reference to the same things as your app, so we can make
UrhoSharp API calls with a few using statements.

Also, game engines tend to have their own mainloop outside the native one, so
we can give the Inspector a hint to use Urho's instead of Android's.

***If demo starts with game already running, inspector could already be running
   too! And it could already have these using statements and stuff!***

```csharp
// Set up using statements and main thread
using Urho;
using Urho.Gui;
using SamplyGame;

SetSynchronizationContextHandler (Application.InvokeOnMain);
```

So let's see what we can find out...

***Start another game run before doing this, so that there is a `game.Player`***

```csharp
// Get a reference to the game
var game = (SamplyGameApp)Application.Current
```

***Expand interactive object table, drill down to `game.Player.Node.Scale`***

So here we are looking at the members that make up our game object. This is the
actual game that's running right now. Our game has a `Player`, and if we keep
expanding down here, we'll find the Urho `Node` for my airplane. And look, we
can see that the scale is `0.85f`.

Actually it's getting that from some game settings that I have set up.

***Go back to `game` interactive object table, look at
`game.Settings.PlayerNodeScale`***

So let's just change that...

```csharp
// Set scale via settings. Note that this only takes effect at the beginning
// of each run. Kind of annoying.
game.Settings.PlayerNodeScale = 0.75f;
```

And then if we start a new run...okay, still too big. Let's change it again.
But waiting for a new game to start is still too slow. Let's work faster!

First I'll bump up my health so I don't die while we're testing:

```csharp
game.Settings.PlayerMaxHealth = 700;
```

Now we can directly manipulate the airplane node in the game:

```csharp
// Too small!
game.Player.Node.SetScale (0.1f);
// Too big!
game.Player.Node.SetScale (0.9f);
// Just right!
game.Player.Node.SetScale (0.35f);
```

Yes, I like this much better. I can dodge these weapons but it's not *too* easy.
So now I'll set that value in the game settings.

```csharp
game.Settings.PlayerNodeScale = 0.35f;
```

And I'll reset the starting health to the old value:

```csharp
game.Settings.PlayerMaxHealth = 70;
```

Another feature I thought of to let me play the game longer would be if coins
actually gave me more health. But actually, this sample doesn't give me any way
to know what my health is. So let's fix that...

```csharp
// Add a new UI element to the game
var text = game.CreateText ("0 life");
game.UI.Root.AddChild(text);
```

Does that look good there? Maybe move it to the center?

```csharp
text.HorizontalAlignment = HorizontalAlignment.Center;
```

No it was better before.

```csharp
text.HorizontalAlignment = HorizontalAlignment.Left;
```

Okay let's hook it up to the player's `HealthChanged` event and see if we like
it...

***Start game run, then run this code (probably easier to type it,
then start game run, then quickly go back and submit)***

```csharp
// Make UI element update according to game state
// NOTE: Only works for current run (new Player generated for each run)
game.Player.HealthChanged += (o,e) =>
	text.Value = $"{game.Player.Health} life";
```

Normally, trying out all of these little tweaks would have meant a lot of
quiting, editing, rebuilding, and redeploying, which really interrupts my flow
when I'm doing something creative like working on a game or another UI.

The Xamarin Inspector can be really handy for quickly adjusting bits and pieces
of your app. And it lets me spend more time playing my game, or, if Miguel asks,
"working". Thanks!