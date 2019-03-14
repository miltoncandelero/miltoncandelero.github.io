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