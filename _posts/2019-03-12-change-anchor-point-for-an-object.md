---
layout: post
title: "How to change the anchor point of an object in OpenFL"
sub_title: "Rotate and scale are two big enemies when working with OpenFL"
excerpt_separator: "<!--more-->"
categories:
  - OpenFL
elements:
  - openfl
  - haxe
  - game development
---
You have your graphic, you just need to rotate it 90º. You set the rotation variable and.... Wait... where did it go?

Well, your playtformer character has to turn arround, just set scaleX to -1 and.... Wait... why did it step left?

If this is you, you are probably suffering from the fact that the anchor point for all objects in OpenFL is located on the top-left-most point of it.  
But... there must be a simple way to set the anchor right?... Right? 😰

<!--more-->

Well, sadly the answer is **no**  
There is no simple way of setting the anchor of a DisplayObject in a simple variable and forget about it.

But you already clicked so let me show you two ways of achieving something similar!
* Matrix transformations. *(The hard and super Math heavy way)*
* Abusing the Parent-Child relationships. *(The simple and no-brainer way)*

### Matrix transformations
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

But I hear you saying: *Hey, there is no "Rotation" field on your matrix. Why are you talking about this thing if it can't rotate?*  
Well, the thing is that rotation is a crazy combination of all the fields. Rotation is achieved with a matrix that looks like this:

$$ \begin{bmatrix}
\cos(\alpha) & \sin(\alpha) & 0\\
-\sin(\alpha) & \cos(\alpha) & 0\\
0 & 0 & 1 
\end{bmatrix}  $$

Whenever you touch the `rotation` variable on your `DisplayObject` OpenFL does all that crazy matrix transformations for you.

Things only get more complicated from here if we want to apply multiple transformations to an object. Luckily the Matrix class has some methods to modify a matrix without knowing the exact matrix math.  
[To learn an awful lot about matrix, check this awesome post. (You will need to enable Flash to see the examples)](http://www.senocular.com/flash/tutorials/transformmatrix/)   

So, let me put a big chunk of code.
```js
public function rotateAroundPoint(object:DisplayObject, center:Point, angleDegrees:Float) {

  // We get the matrix to carry on all the previous transforms that could be going on before this rotation
  var matrix:Matrix = object.transform.matrix;

  //Convert our relative point to absolute
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
This is for the simple reason that we don't know which point was used to rotate the object the time before this one, to fix the rotation and be able to *"set"* an angle

The advantage of this method is that whenever we can choose a new point for rotation and *give it a spin* and it will work.

---

### Abusing the Parent-Child relationships.
*Things got darker than what I expected* 😐

Let's begin with *Display List 101*:
* Everything that is drawn on screen was added as a child of a `Display Object Container`.
  * The Daddy of them all is an object called `stage`
* The class `Sprite` is a wonderful `Display Object Container`
* A `Display Object Container` will transform all his childs relative to itself.
  * Moving, scaling and **rotating** a parent will affect his children.

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

woah code.


