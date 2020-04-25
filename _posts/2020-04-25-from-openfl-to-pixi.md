---
layout: page
title: "OpenFL and PixiJS: mind the gap"
sub_title: "How and why did I change?"
excerpt_separator: "<!--more-->"
categories:
  - PixiJS
elements:
  - openfl
  - haxe
  - pixijs
  - typescript
  - vscode
  - game development
---
When I reached Killabunnies I was handled OpenFL 3 and had to start making games... and making games I did.  
That was 3 years ago.  
Earlier this year I was thrown into a scary world called Babylonjs and threw the rest of my team into another scary world called PixiJS.  
Join me on this magic carpet ride as I show you the good, the bad, and the ugly of this technological leap.  

<!--more-->

Let's start from the bottom:

# Haxe vs Typescript

## The elephant in the room
*Let's kill it!*  
Why not raw Javascript.

```js
0=="0" //true
0==[] //true
"0" == [] // false
[] == "0" // false (one guy once told me that this might be different. It wasn't)
```
meanwhile on typescript...
```ts
0 == "0" // ERROR: This condition will always return 'false' since the types 'number' and 'string' have no overlap. (TS2367)
```
Coming from C++, C#, and Haxe, strongly typing has become second nature for me. The desition for typescript was obvious.

## The target
The biggest difference between Haxe and Typescript is that the former is meant to transpile into a lot of different languages and platforms while Typescript only becomes Javascript.  
This is not a small feat, however, we were mostly making HTML5 games and when the time to put one of our games into native came, instead of using Haxe's native android/windows hxcpp export, we used Cordova and NWJS.

## Making the leap

Let's make a table

| Haxe | Typescript |
|---|---|
| `var` | `let` & `const`</br>*(`var` exists but it's evil and you shouldn't use it)* |
| `typedef` | `interface` |
| `for(e in arr)` | `for(e of arr)`</br>*(the `for in` version exists but does a different thing!)* |
| `switch` cases never fall through</br>*(the keyword `break` isn't needed)* | `switch` cases do fall through and you need to put your `break`! |
| `Int` & `Float` | `number` |
| `Array<Thingy>` | `Thingy[]`  |
| `()->{}` | `()=>{}` |
| `new(){}` | `constructor(){}` |
| only `null` exists</br>*(anything that you don't set, is null)* | `null` is the null value and `undefined` means you didn't set it |
| everything by default is `private` | everything by default is `public` |
| overriding methods need the keyword `override` | you just declare the new method with the same name |

That is pretty much all there is to it! (I am, of course, lying but with this, you should be ready to hit the ground running).


# OpenFL vs Pixi
*with the programming linguistics out of the way, let's get into the main course*

Frist some history: When Apple decided to kill flash by not putting it into his new phone, people saw the need for a new 2d engine to do animations and games. A lot of things came and went (some are still alive) but today we focus on OpenFL and PixiJS  
While **OpenFL** holds a purist approach to what was the Flash API and recreates it as best as possible (You can oftentimes read the as3 reference and almost use it verbatim), **Pixi**, on the other hand, is a Canvas2D and WebGL API *unificator* heavily inspired by flash but without trying to recreate it.

OpenFL was (and still is) great if you come from AS3 Flash because everything is done exactly as it was done there. `Sprites` and `Movieclips` are empty containers and `Bitmaps` are your raster images.  
However, if you are not an OpenFL user you already see a problem.  
The confused face and question I get when I talk to a new guy: 
> `Sprite`... is just a container? and can't show a sprite on his own?.

Pixi went the more sensible way (or at least, reached it at some point in time) where `Container` is an empty container, `Graphic` is a vector graphic and `Sprite` is a sprite.

Besides the naming convention, the great equalizer between the two is the _Display List_ (a scene graph) way of rendering things.

## The Display List
The display list is actually a display tree (sneaky little lier). It has a root and that root contains one or more objects that can themselves contain one or more objects that can themselves contain... you get the idea.  
The objects are rendered by Post-Ordering traversing the tree and the transformations passed from parent to children stack.  
Parents drag children, children don't know there is anything outside their parents.

For me, the display list is THE best way of handling objects in a 2D space. It does have a major flaw and that is that since a tree can't properly represent the Penrose Stairs (sorry, a tree with loops ain't a tree in my book) the display list can't draw them either.

## Why did I force everyone to make the jump?
*(Remember I am the de facto CTO. I said "we pixi now" and we are Pixi now)*

We have been using OpenFL for years shielding ourselves on the "we can export to any platform with a single codebase" but things started happening...

### We need to go to a single place, fast.
While OpenFL focused on multi-platform export (thanks to Haxe) and keeping the Flash API as intact as possible, PixiJS strapped a rocket to its back and did its best impression of Sonic the Hedgehog. Without 90s references: PixiJS just wants to go fast.

So, since I never used Flash (and the number of people who did is steadily going down), OpenFL's greatest advantage is the ability to export natively to multiple platforms and I was holding my fort with that claim until the day we actually had to make some native builds (windows and android) aaaaaand we used Cordova and NWJS.  
The fact that we weren't taking advantage of the native export and that we were going through massive hoops to get good performance on the only target that mattered (HTML5) made Pixi more and more enticing.

### For some reason, clients love Pixi
PixiJS is famous among the clients (not Phaser famous, but we will get to that a bit later) so clients were like:
> It's nice and all but we only tried Pixi and Phaser on our site, I don't know if your thing will work with our thing.

### PixiJS v5 was officially out of beta  
Before that, I was hesitant to jump, I didn't want to jump to something that was just about to be superseded by a new version nor I wanted to jump *production-first* into a Beta tool.  
Catching Pixi v5 early out of beta allowed us to stay in front of the ever pushing wall of deprecation and but not too far that we fall into the darkness of *"nobody ever tried that before"*

### Something that I didn't know I wanted
Even though with Haxe 4 and the huge efforts of everyone involved the *Haxe Completion Server* the code completion has been pushed to a really good, usable, and reliable state, it can't match the crazy lightning speed of what Typescript can do. The speed at which Typescript suggests you things and the black magic that is Microsoft's *IntelliCode* it's just amazing.

### The good thing about using the popular kid's language
Due to javascript and Pixi being the popular web game development framework there are lots of libraries waiting for you at npm. Whatever you need you probably don't have to make it, there is one on npm for grabs.

---
As I said, It was not a single thing that happened but a sum of all the little things.

## Addendum: Why not Phaser?
*The quick answer is: for the same reason we didn't use HaxeFlixel.*

When you use a game engine a lot of the food is chewed by you, and this can be great but as soon as you want to move outside of the game engine's zone of comfort you hit a really hard wall. Things are meant to do one way: the engine's way.  
By working with a "graphic framework" like OpenFL or Pixi you have the freedom to do things however you see fit for every use case. I learnt to enjoy that freedom and find really difficult to accept that some things "can't be done".

---

As a final note, I learnt a lot and got help from a lot of wonderful people in the OpenFL community. I miss you all and I might return one day...