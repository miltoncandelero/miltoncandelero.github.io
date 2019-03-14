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
* Matrix transformations.
* Abusing the Parent-Child relationships.

### Matrix transformations
All DisplayObjects in OpenFL have a 3x3 Transformation Matrix associated to them.  
The matrix looks like this:  

```|  a |  b | u |
|  c |  d | v |
| tx | ty | w |```

*Nice letters... but what do they mean?*  
Well, I kinda lied since `u` `v` and `w` don't really exist and can't be used, so the matrix that you will use looks like this:

```|    a ScaleX   |    b Skew Y   |
|    c Skew X   |    d ScaleY   |
| tx Position X | ty Position Y |```

*(I have actually a post-it with this reference matrix on my screen)*

But I hear you saying: *Hey, there is no "Rotation" field on your matrix. Why are you talking about this thing if it can't rotate?*  
Well, the thing is that rotation is a crazy combination of all the fields. Rotation is achieved with a matrix that looks like this:

```| cosine(angle) |  sine(angle)  |
|  -sine(angle) | cosine(angle) |
|       0       |       0       |```

Whenever you touch the `rotation` variable on your `DisplayObject` OpenFL does all that crazy matrix transformations for you.

Things only get more complicated from here if we want to apply multiple transformations to an object. Luckily the Matrix class has some methods to modify a matrix without knowing the exact matrix math.  
[To learn an awful lot about matrix, check this awesome post. (You will need to enable Flash to see the examples)](http://www.senocular.com/flash/tutorials/transformmatrix/)   

So, let me put a big chunk of code and then explain to you what it does.
```js
public function rotateAroundPoint(object:DisplayObject, center:Point, angleDegrees:Float)
{
    //We get the matrix to carry on all the previous transforms that could be going on before this rotation
    var matrix:Matrix = object.transform.matrix; 

    //We "move" the center
    matrix.translate(-(object.x + center.x), -(object.y + center.y));
    //We rotate the object (converting degrees to radians)
    matrix.rotate((angleDegrees / 180) * Math.PI);
    //We "restore" the center
    matrix.translate(object.x + center.x, object.y + center.y);
    //the ".matrix" attribute has a setter that only applies a matrix when a new matrix is set.
    object.transform.matrix = matrix;
}
```

woah code.


