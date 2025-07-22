---
layout: post
title: "Coloring Outside the Lines (in Unreal Engine UMG Shaders)"
sub_title: "What if I told you you can color outside the lines? What if I told you that you can render outside the quad?"
excerpt: "Stop me if you've heard this one before: your UI artist asks, 'Hey, can we add stroke, glow, or shadow in the engine, right?' All of a sudden your head starts racing; you know you only have 2px padding on the edge of the PNG! How are you gonna make it without destroying your layout?"
categories:
  - Unreal Engine
tags:
  - unreal engine
  - user interface
  - game development
---
# Coloring Outside the Lines 
### _(in Unreal Engine UMG Shaders)_

Stop me if you've heard this one before: your UI artist asks, "Hey, can we add stroke, glow, or shadow in the engine, right?" All of a sudden your head starts racing; you know the three shader methods to do that, but you also know you only have 2px padding on the edge of the PNG... maybe 4px tops if you are lucky!

What if I told you... you can color _outside_ the lines? What if I told you that you can **render outside the quad**?

Open your mind
<div align="center">

![](/assets/images/outsidethelines/openyourmind.png)

</div>

## Let's go with the _naive_ approach first

Ok, so our artist asked us to draw, in shader, around an image that goes very close to the edge, leaving us very little space to do so.

Take this sword png as an example.

<div align="center">

![](/assets/images/outsidethelines/originalsword.png)

</div>

The "only way" to do this, I thought, was to _scale down_ the image in shader (`Scale UV by Center`), and use the extra space to draw any cool effects

<div align="center">

![](/assets/images/outsidethelines/swordofperfectlyaveragesize.png)

</div>

(The blue square is another widget in the background to give you a size reference.)

But now, I need to make the UMG Image widget larger to compensate for shrinking the sword!

<div align="center">

![](/assets/images/outsidethelines/brokenlayout.png)

</div>

And now my layout is all wrong!

<div align="center">

![](/assets/images/outsidethelines/thisisyou.jpg)

</div>

## Black magic

It all began one day in the [Unreal Garden Discord server](https://discord.gg/KnWJ2jCSFk), where this image was posted by [Melchior](https://www.linkedin.com/in/melchior-corgie/).

<div align="center">

![](/assets/images/outsidethelines/blackmagic.png)

</div>

If you know a bit about rendering, you know that the allotted space for you to render is **inside** the bounds of the quad that slate will use to show you the material... however, we can clearly see how this is rendering a magenta circle **outside** that space...

This shader is ~~abusing~~ using the `Screen Position` output to _scale up_ the rendering space!

First of all, since we are in a _fragment_ shader, we can effectively think of this as running _for every pixel of the widget._

Now, what `Screen Position` does is modify where we are rendering all of our pixels in screen space.

So, if we just move every pixel to the right by, say, 100px, that would be it; we end up with the widget just translated to the right... but what if we move the left pixels to the left and the right pixels to the right? If we offset them correctly, we would _stretch_ the widget!

We can use the UVs to know what is _left_ and what is _right,_ and we remap the `0` to `1` to `-1` to `1` so we can just multiply this new range by the amount we want to _stretch_ and achieve our desired result.

Using this new cheat, we can make this!

<div align="center">

![](/assets/images/outsidethelines/perfectlayout.png)

</div>

### VICTORY

Look at the green lines. That is the Slate layout bounds!

We are effectively drawing outside the bounds. Our layout is safe, and we can do fancy effects!

## Victory? You just drew a magenta circle!

Well, this is just the proof of concept of the superpower you just unlocked!

With this, you can now do strokes, glows, and drop shadows that won't break your layout!

<div align="center">

![](/assets/images/outsidethelines/animatedexample.webp)

</div>

## Give me something useful!

Let's make a Material Function that allows us to pick a _quad scale_ but still gives us the _original_ UVs in case we still want to render inside the quad like lawful legal shader<i>ers</i>.

<div align="center">

![](/assets/images/outsidethelines/usageexample.png)

</div>

### MF_UI_ScaleQuad
[See in BlueprintUE.com](https://blueprintue.com/blueprint/wdvprf9o/){:target="_blank"}

<div align="center">

[![](/assets/images/outsidethelines/scalesource.png) (Open in new tab)](/assets/images/outsidethelines/scalesource.png){:target="_blank"}

</div>

This one works with a multiplier (e.g., "Make my quad 1.5 times bigger"), but what if you only want to say, "Inflate my quad by 50px"?

### MF_UI_ResizeQuad
[See in BlueprintUE.com](https://blueprintue.com/blueprint/cro2vhg2/){:target="_blank"}

Here, this is a (simpler?) version that takes in the amount of padding (in pixels) to add.

<div align="center">

[![](/assets/images/outsidethelines/paddingsource.png) (Open in new tab)](/assets/images/outsidethelines/paddingsource.png){:target="_blank"}

</div>

## Important note when using pixel size in UI materials

To mess with pixel size, you often need the magical node called `GetUserInterfaceUV` and specifically the `PixelSize` which contains the size **at which your shader is currently being rendered.**

**This means that editor zoom affects this value.**

During design, if your zoom level is not at 1:1, your shader might look broken, but trust that during runtime it will behave nicely (probably).

---

And this is it; you have now learnt how to break the rules of blueprints!

---

Special thanks to

- [Melchior](https://www.linkedin.com/in/melchior-corgie/) for the original implementation (and the screenshot that took my attention)

- [Richard](https://bsky.app/profile/rtm223.me) for the idea to expand the UI in the vertex shader (even if that is not exactly what is going on)

- [Ryan](https://ryandowlingsoka.com/) for demystifying how this would behave around z-order.

- and to everyone in the Unreal Garden Discord server. [Join us!](https://discord.gg/KnWJ2jCSFk) 🌱