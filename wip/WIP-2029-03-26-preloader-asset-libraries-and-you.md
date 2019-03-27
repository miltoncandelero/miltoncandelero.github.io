---
layout: page
title: "Preloader, Assets Libraries and you!"
sub_title: "A short guide on how to load your assets"
excerpt_separator: "<!--more-->"
categories:
  - OpenFL
elements:
  - openfl
  - haxe
  - game development
---
Do you want a cool preloader for your web game?  
Do you want to pack your assets into a single bundle file for your desktop release?  
Do you want to load your game one part at a time instead of all at the beginning?

Well *totally real person and not a figment of my imagination*, I hear you loud and clear.  
Come with me and we shall tackle these issues...

<!--more-->
But before we start, a list of what are we going to cover.
*(just because I love making lists of things)*

- Preloader for Web games
- The `<assets>` tag and you:
 - The include and ignore attributes
 - The library attribute and `<library>` tag
 - The preload attribute

---

# Making a Preloader for Web games

*(A preloader is the progress bar that shows how much is left before your entire game is loaded and ready to play)*

If you create a new, blank OpenFL project (`openfl create project NewProjectName`) you might notice that there is no "default" preloader.  
If you are smart you might realize a sample for custom preloader exists in `openfl create CustomPrealoader`... 😐

*Why are you doing this?* you might ask, and the answer is both *because the sample doesn't use assets* and *because I started writing this before realizing that there was a sample*

But since there is a sample, let's quickly glance the mechanics of a preloader:  
you need to add in your `project.xml` a line that says something like:

```xml
<app preloader="Preloader" />
```

And then have a `Preloader` class that inherits sprite ready to go. The default created by the sample looks like this:

```js
package;

import haxe.Timer;
import openfl.display.Sprite;
import openfl.events.Event;
import openfl.events.ProgressEvent;

class Preloader extends Sprite {
 
 public function new () {
  super ();
  addEventListener (Event.COMPLETE, this_onComplete);
  addEventListener (ProgressEvent.PROGRESS, this_onProgress);
 }
 
 private function update (percent:Float):Void {
  graphics.clear ();
  graphics.beginFill (0x1F9DB2);
  graphics.drawRect (0, 0, stage.stageWidth * percent, stage.stageHeight);
 }
 
 private function this_onComplete (event:Event):Void {

  update (1);
  
  // optional
  
  event.preventDefault ();
  
  Timer.delay (function () {
   
   dispatchEvent (new Event (Event.UNLOAD));
   
  }, 2000);
  
 }
 
 private function this_onProgress (event:ProgressEvent):Void {
  
  if (event.bytesTotal <= 0) {
   
   update (0);
   
  } else {
   
   update (event.bytesLoaded / event.bytesTotal);
   
  }
  
 }
 
}
```

Ok, that will work. It will make out of the entire screen a progress bar that will fill from left to right to show the load progress.

Now, if you want images you might be tempted to go `Assets.getBitmapData(...)` but wait, how are you going to get that BitmapData if the image is still downloading?  
The answer is that you can't get that file. You need to embed your assets for the preloader *in the preloader* 🤯

We can embed 4 kinds of files:

- Bitmaps (They must extend BitmapData) with `@:bitmap`
- Fonts (They must extend Font) with `@:font`
- Sounds (They must extend Sound) with `@:sound`
- Binary data (They must extend ByteArray) with `@:file`

We are going to focus on the Bitmap one but the other types should be as straightforward as bitmap.  

The way to declare the asset is: `@:bitmap("preloader/bg_image.png") class Background extends BitmapData {}`  
Breaking it down:
- `@:bitmap` We are embedding a bitmap
- `("preloader/bg_image.png")` The path to the image to embed
 - You might realize it is not in our `assets` folder. That is on purpose since we don't want the asset to be copied to the final build as we are embedding the image inside the code itself
- `class Background` We give it a unique name that we will use later to reference this image.
 - It might be odd that we make it a class instead of just a variable but it's necessary.
- `extends BitmapData {}` When we embed bitmaps we must extend the BitmapData class.

Ok, we created a new class that contains all the information to make our image. How do we make an actual image?  
We create our new bitmap like this: `new Bitmap(new Background(0, 0));`  
If you are curious you might wonder what are those `0, 0` values?  
Those are the width and height of your image. **If you want to use the width and height value of your new `Bitmap` object you will need to set this values correctly by hand. There is no way to read those values from the assets. You must hardcode them!**

Now that we created our `Bitmap` object, we just add it to the display list

```js
var background:Bitmap = new Bitmap(new Background(0, 0));
addChild(background);
```

That should be all we need to add an image to our preloader.  
Let's take a look at this new Loader.hx class, I removed the code to draw the primitive progress bar and added a background image:

```js
package;

import haxe.Timer;
import openfl.display.Sprite;
import openfl.events.Event;
import openfl.events.ProgressEvent;
import openfl.display.Bitmap;
import openfl.display.BitmapData;

//Asset declarations go outside the preloader class
@:bitmap("preloader/bg_image.png") class Background extends BitmapData {}

class Preloader extends Sprite {
 
 public function new () {
  super ();
  //we make a new bitmap using our embeded asset
  var background:Bitmap = new Bitmap(new Background(0, 0));
  //we add it to the scene
  addChild(background);

  //Listen for the "when we finish loading" event
  addEventListener (Event.COMPLETE, onComplete);

  //Listen for the "progress moved forward" event
  addEventListener (ProgressEvent.PROGRESS, onProgress);
 }
 
 //Small function to report back the load progress
 private function update (factor:Float):Void {
   //Simple trace the percent of what we loaded.
   trace ("Loaded " + Std.int(factor*100) + "%");
 }
 
 //When the load is done
 private function onComplete (event:Event):Void {
  // updateone last time.
  update (1);
  event.preventDefault ();

  //This closes our preloader and loads our game.
  //If you want to do a cool animation do it and then call this
  dispatchEvent (new Event (Event.UNLOAD));
 }
 
 // when the load moves forward
 private function onProgress (event:ProgressEvent):Void {
  // safety check to not divide by zero.
  if (event.bytesTotal <= 0) {
   update (0);
  } else {
   // call the method that reports back the load.
   update (event.bytesLoaded / event.bytesTotal);
  }
 }
}
```

And that is all that there is. With some creativity you might make some really cool looking preloaders!  
Now for the second part...

---

# The `<assets>` and you

AaAAAAND I have to finish this.

This is my post

I love it even if it is ugly

# Lets do something big here.

aaaand we are done.