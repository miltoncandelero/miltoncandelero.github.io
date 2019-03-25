---
layout: page
title: "Making a web/mobile game: Resize vs Responsive"
sub_title: "Do we set resize to true and call it a day? or de we make things the right way?"
excerpt_separator: "<!--more-->"
categories:
  - OpenFL
tags:
  - openfl
  - haxe
  - game development
  - responsive
---
If you are making web games you probably already use the "resize" property in your project.xml  
It allows you to simply *stretch* your game to match the target resolution without loosing the original aspect ratio.  
The downside is that you get ugly black bars where the ratio didn't match.  
Wouldn't it be cool if we could have an image to bleed out instead of those black bars?  
Join me in this marvelous journey into the realm of *responsiveness...*  

<!--more-->

Let's begin with a brush up on how to make a game just resizable.   
Go into your `project.xml` and locate (or add) something that looks like this:

```xml
<window resizable="true" />
```

Try playing with this variable and you will see the different results.   
You should get a stretch but with the aspect ratio is preserved by adding black bars to the sides (or top and bottom) of your game.   

And that maybe is enough for you!   
If your game will be played in a iframe inside another website (think Kongregate, NewGrounds, itch.io, etc), you can set a ratio or size of the frame and your game will resize and never look ugly.

**But what if you need your game to run directly on a phone and resize to cover the full screen of the phone?**

That is the question I had to answer and the knowledge I am here to give you today.   
Let's begin with the basics: **How to make the game resize to any resolution without having the black bars?**

Simply set the `height` and `width` to zero. *(I think `resizable` becomes automatically true when those are zero but it can't hurt to add it, right?)*

```xml
<window width="0" height="0" resizable="true" />
```
If we test it we will see that now our game (or what bleeds out of our game) fills the screen... but there is no resizing!

Of course there isn't. This resizes the container. We will need to play with the scale of our game.   
*(It would be really useful if you have your entire game inside a single container instead of just cluttering your `Main.hx`)*

Let's list some bases we need to cover:
- We need to resize things when the screen size changes
  - We need to know the actual size of the screen
- We need to keep our aspect ratio intact and *fit* inside the screen
  - We want a image to *bleed out* where we can't keep up due to our aspect ratio

Let's tackle them one by one and then mix them up all together.

## We need to know when the screen size changed.

Luckily, OpenFL has our back with that one with the `Event.RESIZE` event.

Let's find a stage and add that event. *(Some creative freedoms where taken. Let's pretend your game is a class called `Game`)*

```js
class Main extends Sprite 
{
  var game:Game;

  public function new() 
  {
    super();

    game = new Game();
    addChild(game);

    // Pick ONLY ONE of the following:

    // Add it to the "Main" sprite. This is good enough.
    addEventListener(Event.RESIZE,onResize);

    // The "stage" object is better but only exists after the sprite was addChild()'ed.
    // (Main is an exception to this rule and exists all the time)
    stage.addEventListener(Event.RESIZE,onResize);

    // This is the proper way of reaching the stage from wherever you might be.
    Lib.current.stage.addEventListener(Event.RESIZE,onResize);
  }
  
  function onResize(e:Event):Void 
  {
    trace ("YO BOI, WE RESIZED");
  }
}
```

You can try the 3 different methods and pick the one that you feel more comfortable with. I will go with `Lib.current.stage` since we will need to use the `stage` again.

Now for the next part, **Getting the new size of the screen**

We get our events when we resized but if you ask the size (`height` and `width`) of the stage you might notice that the size reported is a rectangle that contains everything inside the `stage` object and not the size of the screen.   
That's why we need to check the `stageWidth` and `stageHeight`of the stage. Those two variables keep the size of the screen regardless of what we have showing (or not showing) in the screen.

Try this

```js
class Main extends Sprite 
{
  var game:Game;

  public function new() 
  {
    super();

    game = new Game();
    addChild(game);

    Lib.current.stage.addEventListener(Event.RESIZE,onResize);
  }
  
  function onResize(e:Event):Void 
  {
    trace ("YO BOI, current screen size: " + Lib.current.stage.stageWidth + " by " + Lib.current.stage.stageHeight);
  }
}
```

So, we are resizing and getting the values for the size of the screen. Now to the fun part, **scaling our game**

## Scaling our game and preserving the aspect ratio

Ok, first of all: **How do we preserve the aspect ratio?**  
The answer is simple: Keep the `scaleX` and `scaleY` of our game the same value.  
So every time we resize we are going to `scaleX = scaleY = newScale`. We cool? We cool.

And how do we get this magic `newScale`? by doing some hardcore **MATH**  
*(maybe not that hardcore)*

Let's say we envision our game to run in a portrait mode of **360 by 640** but we need to run on a screen that has a resolution of **800 by 600**
*(This would be a super worst case scenario where we developed for a 9:16 ratio and are trying to run in a 4:3 ratio)*

We need to find the `ratio` between the screen and our game. Let's answer this question:  
*How many times my game fits in the screen?*

$$ ratio = \frac{screen}{game} $$

now in code:

```js
var newScaleX:Float = Lib.current.stage.stageWidth/game.width;
var newScaleY:Float = Lib.current.stage.stageHeight/game.height;
```

If we assign directly those values to `scaleX` and `scaleY` we will see our game resize but also deform as the aspect ratio was violently changed from 9:16 to 4:3  
We need to pick only one of those values and use it for both `scaleX` and `scaleY`  
To do so, we answer this question:
*Do I want to **fit** inside of the screen or do I want to **bleed** outside of the screen?*

Think about it: 
- If we pick the *smaller* value, one side of our game will fit perfectly and the other will fall short. Effectively staying inside of the screen.
- If we pick the *bigger* value, one side of our game will fit perfectly but the other side will bleed out of the screen.

Since we want our game to stay inside of the screen, we want the smaller value of the two.  
To do this we could do an `if()...else...` clause or we could use the already done `Math.min()`.

```js
class Main extends Sprite 
{
  var game:Game;

  public function new() 
  {
    super();

    game = new Game();
    addChild(game);

    Lib.current.stage.addEventListener(Event.RESIZE,onResize);
  }
  
  function onResize(e:Event):Void 
  {
    trace ("YO BOI, current screen size: " + Lib.current.stage.stageWidth + " by " + Lib.current.stage.stageHeight);

    game.scaleX = game.scaleY = 1; //This is important if we use the "width" and "height" properties of your game!
    var newScaleX:Float = Lib.current.stage.stageWidth/game.width;
    var newScaleY:Float = Lib.current.stage.stageHeight/game.height;
    game.scaleX = game.scaleY = Math.min(newScaleX,newScaleY);
  }
}
```

Now, the size of your game might not be stored directly into the `width and height` variables of your game, in that case just hardcode the correct numbers and you will be good to go.  
But if you are using those variables, you will need to set the scale to 1 before doing the math or otherwise your width and height will report the size multiplied by the previous scale, throwing off all of our math.

Now you probably want your game to stay centered. You then must simply do:

```js
game.x = (Lib.current.stage.stageWidth - game.width) / 2;
game.y = (Lib.current.stage.stageHeight - game.height) / 2;
```

*Note: If you were hardcoding your values, you will need to multiply your values by the new scale like this*

```js
game.x = (Lib.current.stage.stageWidth - HARDCODED_WIDTH * game.scaleX) / 2;
game.y = (Lib.current.stage.stageHeight - HARDCODED_HEIGHT - game.scaleY) / 2;
```

Now for the final trick, have a **background bleed out** of the screen and cover it entirely.

If you have followed the post to this point you already get how we do it.  
Instead of using `Math.min` we use `Math.max`

```js
class Main extends Sprite 
{
  var game:Game;
  var coolBackground:Bitmap;

  public function new() 
  {
    super();

    coolBackground = new Bitmap(Assets.getBitmapData("assets/img/yourCoolBackground.png"));
    addChild(coolBackground);

    game = new Game();
    addChild(game);

    Lib.current.stage.addEventListener(Event.RESIZE,onResize);
  }
  
  function onResize(e:Event):Void 
  {
    trace ("YO BOI, current screen size: " + Lib.current.stage.stageWidth + " by " + Lib.current.stage.stageHeight);

    game.scaleX = game.scaleY = 1; //This is important if we use the "width" and "height" properties of your game!
    var newScaleX:Float = Lib.current.stage.stageWidth/game.width;
    var newScaleY:Float = Lib.current.stage.stageHeight/game.height;
    game.scaleX = game.scaleY = Math.min(newScaleX,newScaleY);
    game.x = (Lib.current.stage.stageWidth - game.width) / 2;
    game.y = (Lib.current.stage.stageHeight - game.height) / 2;

    coolBackground.scaleX = coolBackground.scaleY = 1;
    var newScaleX:Float = Lib.current.stage.stageWidth/coolBackground.width;
    var newScaleY:Float = Lib.current.stage.stageHeight/coolBackground.height;
    coolBackground.scaleX = coolBackground.scaleY = Math.max(newScaleX,newScaleY); //here we set the max
    coolBackground.x = (Lib.current.stage.stageWidth - coolBackground.width) / 2;
    coolBackground.y = (Lib.current.stage.stageHeight - coolBackground.height) / 2;
  }
}
```

And that would be all on how to make your game truly responsive instead of just resizing and having black bars!  
This can be useful for a lot of things. Just some suggestions for you:

- Keep the `scaleX` and `scaleY` integer (with `Math.floor`) to have a pixel perfect resize
- Instead of just resizing everything, move around your UI and adapt to wider or taller screens
- Maybe you just want to fill *a fraction* of the screen. Try to find where you need to do that extra math

Good luck and see you on the next problem!