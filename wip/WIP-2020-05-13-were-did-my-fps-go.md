---
layout: page
title: "Where did my FPS go and how do I get them back?"
sub_title: "How to use chrome tools to detect major performance flaws"
excerpt_separator: "<!--more-->"
categories:
  - Javascript
elements:
  - javascript
  - pixi
  - game development
---
Sometimes we code things in a rush and they end up sloppy and laggy. Some other times we are pushing the renderer to it's limits and we need every extra fraction of fps we can get.  
Join me on this marvelous travel where we learn how to read the chrome debugger tools to squeeze blood from the stones.

<!--more-->

After reading the part talking about _blood from the stones_ you are still here? Great!  
Let me enumerate the 3 big bad problems we will tackle today:
- High frame times (AKA thinking too much) 
- Memory leaks (AKA remembering too much)
- Garbage collects (AKA forgetting too much)

# Step 0: Know where you lay
Before we start you must be able to see how many fps you are rocking to begin with. Otherwise we are trying to find a black cat that might or might not be in an unlit room.

### Method 1: Chrome rendering tools
- Open your chrome dev console (F12)
- Find the vertcal ellipsis (...) menu
- In the menu, go to More Tools -> Rendering
- In the window that popped, find the "FPS meter"

//! TODO: Add the screenshot here.

### Method 2: Use something cool like stats.js
[stats.js](https://github.com/mrdoob/stats.js/) is really good since it can show you not only fps but frametime and memory consumption. If your engine supports it you might want to go for [gstatsjs](https://github.com/eXponenta/gstatsjs) which includes GL drawcalls.

### Method 3: Cavaman delta time
At the beginning of your update loop check how many time passed since the last frame
```js
let lastTime = Date.now();
function update()
{
  const now = Date.now(); 
  const dt = now - lastTime;
  lastTime = now;
  console.log("oogabooga frametime was",dt)

  // your update code...
}
```

---
I know consider that you have a way to know how many fps are you getting and we are not just _feeling it_. We are scientist for code's sake!

# Frametime and you
Definitions can get a bit fuzzy but if FPS is Frames Per Second, then Frametime is Seconds per Frame.  
Since we are master race and we target 60 fps, our frametime becomes 16.666<sup>6<sup>6<sup>6</sup></sup></sup> miliseconds (You can't go higher than 60 fps on web kiddo, sorry)  
Now the thing becomes purely mathematic: If we want more frames we need to think less than 16 ms per frame.

To test how does a mean, ugly function behaves we have this bad boy here:
```js
function calculatePi() {
    //Leibniz formula for Pi
    let pi = 0;
    let n = 1;
    const loops = Math.random() * 10000000;
    for (i = 0; i <= loops; i++) {
        pi = pi + (4 / n) - (4 / (n + 2))
        n = n + 4
    }
    return pi;
}
```
This is a formula to calculate PI I stole from the internet. It is not important, the important part is that it has a random loop that can skip the loop or loop ten million times. I made it random so we can see some nice dips in the framerate.

Now, this is our framerate after this slow boy was called once every frame
//! TODO Add screenshot here!

Here we start the profiling!


https://www.pixiplayground.com/#/edit/yTrjVgznIY_wi1mKiLKir


I love it even if it is ugly

# Lets do something big here.

aaaand we are done.