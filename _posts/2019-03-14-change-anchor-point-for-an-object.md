---
layout: post
title: "How to change the anchor point of an object in OpenFL"
sub_title: "Rotate and scale are two big enemies when working with OpenFL"
excerpt_separator: "<!--more-->"
categories:
  - OpenFL
tags:
  - openfl
  - haxe
  - game development
---
You have your graphic, you just need to rotate it 90º. You set the rotation variable and.... Wait... where did it go?

Your platformer character has to turn arround, just set scaleX to -1 and.... Wait... why did it step left?

If this is you, you are probably suffering from the fact that the anchor point for all objects in OpenFL is located on the top-left-most point of it.  
But... there must be a simple way to set the anchor right?... Right? 😰

<!--more-->

Well, sadly the answer is **no**  
There is no simple way of setting the anchor of a DisplayObject in a simple variable and forget about it.

But you already clicked so let me show you two ways of achieving something similar!
* Matrix transformations. *(The hard and super Math heavy way)*
* Abusing the Parent-Child relationships. *(The simple and no-brainer way)*
  * Bonus track: Creating a class to easy set the anchor of Bitmaps

## Matrix transformations
All DisplayObjects in OpenFL have a 3x3 Transformation Matrix associated to them.  
The matrix looks like this:  

$$ \begin{bmatrix}
a & b & u \\
c & d & v \\
tx & ty & w 
\end{bmatrix}  $$

*Nice letters... but what do they mean?*  
Well, I kinda lied since `u` `v` and `w` don't really exist and can't be used, so the matrix that you will use looks like this:

$$ \begin{bmatrix}
Scale X & Skew Y & 0\\
Skew X & Scale Y & 0\\
Position X & Position Y & 1 
\end{bmatrix}  $$

*(I have actually a post-it with this reference matrix on my screen)*

Let's tackle a simple problem: You want to rotate your sprite arround a point that is not the default `0,0` top-left anchor.  
But there is no field *rotation* on that matrix!  
The rotation is achieved with a matrix that looks like this:

$$ \begin{bmatrix}
\cos(\alpha) & \sin(\alpha) & 0\\
-\sin(\alpha) & \cos(\alpha) & 0\\
0 & 0 & 1 
\end{bmatrix}  $$

Whenever you touch the `rotation` variable on your `DisplayObject` OpenFL does all that crazy matrix transformations for you.

Things only get more complicated from here if we want to apply multiple transformations to an object. Luckily the Matrix class has some methods to modify a matrix without knowing the exact matrix math.  
[To learn a lot about matrices, check this awesome post. (You will need to enable Flash to see the examples)](http://www.senocular.com/flash/tutorials/transformmatrix/)   

So, let me put a big chunk of code.
```js
public function rotateAroundPoint(object:DisplayObject, center:Point, angleDegrees:Float) {

  // We get the matrix to carry on all the previous transforms that could be going on before this rotation
  var matrix:Matrix = object.transform.matrix;

  //This fixes what does that point mean if you have scaled the object.
  center = matrix.transformPoint(center);

  // We "move" the center
  matrix.translate(-center.x, -center.y);

  // We rotate the object (converting degrees to radians)
  matrix.rotate((angleDegrees / 180) * Math.PI);
  
  // We "restore" the center
  matrix.translate(center.x, center.y);
  
  // the ".matrix" attribute has a setter that only applies a matrix when a new matrix is set.
  object.transform.matrix = matrix;
}
```

The fact that the Matrix object has the `translate` and `rotate` methods saves us from having to do Matrix Math.

Testing this you might realize something: **This doesn't *set* an angle, this just adds that angle to whatever angle we had before!**  
On top of that, this only works for rotation! What if I want to scale or move using an anchor point?   
Well, matrix math will get more and more complicated. That's why I use and recommend the next method...

---

## Abusing the Parent-Child relationships.
*Things got darker than what I expected* 😐

We are going to *abuse* the fact that a parent transformation cascades to all his childrens. Think of it as stacking trays or pinning together squares of paper with thumbtacks.

Let's begin with *Display List 101*:
* Everything that is drawn on screen was added as a child of a `Display Object Container`.
  * The Daddy of them all is an object called `stage`.
* The class `Sprite` is a wonderful `Display Object Container`.
  * The class `Bitmap` is **not** a container.
* A `Display Object Container` will transform all his childs relative to itself.
  * Moving, scaling and **rotating** a parent will affect his children.

*(If this whole "DisplayList" thing is confusing you, I might suggest seeing a [basic tutorial](http://www.republicofcode.com/tutorials/flash/as3displaylist/) to understand how it works)*

With those basics, have a look at this code.

```js
var a:Sprite = new Sprite();
a.graphics.lineStyle(1,0xFF0000);
a.graphics.drawRect(0,0,10,10);

var b:Sprite = new Sprite();
b.graphics.lineStyle(1,0x0000FF);
b.graphics.drawRect(0,0,10,10);

a.addChild(b);
stage.addChild(a);

a.x = a.y = 50;
```

We make two sprites, add some vector graphics to tell them appart and add `b` as a child of `a`.  
We add `a` to the stage and move it away from the top left corner of the screen.  
`b` is stuck to `a`, so it will follow it and they will look overlapping.

And here comes the magic:
```js
b.x = -b.width /2;
b.y = -b.height /2;
```

I just moved `b` to the left and up by *half his own size*.  
This means that the `0,0` point of object `a` (The inmovable anchor point) is now visualy **in the same place as the center of square `b`!**  
What do you think it will happen if we rotate `a`?

```js
a.rotation = 45;
```

Well, what we expected: `a` rotated arround that stupid top left corner... but... wait...  
The transformation of `a` that *cascaded* to `b` had him rotating arround `a`'s anchor... and as we saw earlier, that point is the center of `b`!  
**We just found a way to create our own anchor point for `b`!**

The formula looks like this:
```js
b.x = - (anchorX);
b.y = - (anchorY);
```

And then when we move the parent of `b` our *anchor* will be respected!

---

## Bonus track: Creating a class to easy set the anchor of Bitmaps
Let's idealize a new `Bitmap` that has two magic variables: `anchorX` and `anchorY`.  
We saw earlier, we just need to put our object inside a container and move the object inside to the left and up to match our desired anchor point to the `0,0` coordinate of the container.

So, let's make a new class that extends `Sprite`. This `Sprite` will be the "container" object.  
Since it has to feel like `Bitmap` we will ask for the same parameters as `Bitmap` and create one and add it to the scene.  
Also, we are going to take the two `anchor` parameters and apply them to the child bitmap.

```js
class Image extends Sprite
{
  //The child object. Our image
  public var bitmap:Bitmap;

  public function new (bitmapData:BitmapData = null, pixelSnapping:PixelSnapping = null, smoothing:Bool = false, anchorX:Float = 0, anchorY:Float = 0)
  {
    super();
    //We create the image
    bitmap = new Bitmap(bitmapData, pixelSnapping, smoothing);

    //Set the anchor point to align with the parent 0,0 point
    bitmap.x = -anchorX;
    bitmap.y = -anchorY;

    //And add it to the scene.
    addChild(bitmap);
  }
}
```

And that could probably be all you need.  
You create a new `Image` and set the anchor point of your image there and you are good-to-go!

But... What if you want to change the anchor *after* you make the object?

Well, we could make a function that takes the new anchor and moves the internal image to align again to the new anchor point...   
Or we could use **setters** 😎


```js
class Image extends Sprite
{
  public var bitmap:Bitmap;
  public var anchorY(default, set):Float;
  public var anchorX(default, set):Float;

  public function new (bitmapData:BitmapData = null, pixelSnapping:PixelSnapping = null, smoothing:Bool = false, anchorX:Float = 0, anchorY:Float = 0)
  {
    super();
    bitmap = new Bitmap(bitmapData, pixelSnapping, smoothing);

    //This is no longer needed since the setters will trigger from the next assign.
    //bitmap.x = -anchorX;
    //bitmap.y = -anchorY;
    
    //This will trigger our setters!
    this.anchorX = anchorX;
    this.anchorY = anchorY;

    addChild(bitmap);
  }

  public var set_AnchorX(value:Float):Float
  {
    //Save the new value.
    anchorX = value;
    //Move the child to set the anchor.
    bitmap.x = -anchorX;
    //Setters must return the value assigned.
    return value;
  }

  public var set_AnchorY(value:Float):Float
  {
    anchorY = value;
    anchor.y = -anchorY;
    return value;
  }
}
```

This is looking good. But you might note something: If we change the anchor point, the asset on screen will move since the anchor point now is in a different place.  
Let's fix that by moving the container to counteract the change.  
Our final class will look like this:

```js
class Image extends Sprite
{
  public var bitmap:Bitmap;

  //We initialize this two values to zero as we will need the difference between the old anchor and the new anchor.
  public var anchorY(default, set):Float = 0;
  public var anchorX(default, set):Float = 0;

  public function new (bitmapData:BitmapData = null, pixelSnapping:PixelSnapping = null, smoothing:Bool = false, anchorX:Float = 0, anchorY:Float = 0)
  {
    super();
    bitmap = new Bitmap(bitmapData, pixelSnapping, smoothing);

    //The setter will take care of applying the anchor
    this.anchorX = anchorX;
    this.anchorY = anchorY;

    //we want the initial position of our object to be 0,0
    x = y = 0;

    addChild(bitmap);
  }

  public var set_AnchorX(value:Float):Float
  {
    if (anchorX == value) return value; //If the anchor didn't change, we don't do anything.

    //Ok, read this next line carefully
    x += value - anchorX
    //We "MOVE" the parent container. Not just set X but increment it.
    //The value - anchorX is "The amount we are moving to the right"
    //If it is >0, we move the parent to the right since we will move the child to the left.
    //If it is <0, we move the parent to the left since we will move the child to the right.

    //Save the new value.
    anchorX = value;
    //Move the child to set the anchor.
    bitmap.x = -anchorX;
    //Setters must return the value assigned.
    return value;
  }

  public var set_AnchorY(value:Float):Float
  {
    if (anchorY == value) return value;
    y += value - anchorY;
    anchorY = value;
    anchor.y = -anchorY;
    return value;
  }
}
```

So... by now you should be an expert rotator.  

Hope you enjoyed this simple problem made complex. See you in the next one!
