---
layout: post
title: "How to migrate from FlashDevelop to VSCode without dying"
sub_title: "Moving away from FlashDevelop is spooky"
excerpt_separator: "<!--more-->"
categories:
  - OpenFL
elements:
  - openfl
  - haxe
  - vscode
last_modified_at: 2017-03-09T14:25:52-05:00
---

In the beginning there was only Tool to develop, to **FlashDevelop**.

But sadly, what makes FlashDevelop great is also his biggest downfall.

Join me on this adventure where we compare **FlashDevelop** and **VSCode** and I give you my tips and tricks on how to make the leap.

<!--more-->

Ok. You made it into the post, good. Let's begin with pros and cons of both tools.

*Note: I won't list all the features of each tool since they share a lot of functionality.*

## FlashDevelop
> HaxeDevelop offers first class support for Haxe development: great and fast code completion & code generation, refactoring, projects compilation, debugging, plenty of project templates, etc.

**PROS**
* Simple, out of the box support for Haxe/OpenFL.
* Class Wizard: Generates all your boilerplate when you create a new class.
* Massive refactoring tool.
* Has a project file that you can double click to launch the tool easily.
* Has a super cool icon `{>` that looks like `FD`

**CONS**
* Uses his own webserver for HTML5 builds (and is prone to get stuck when you are trying to make a clean build).
* Lacks the `-release` setting and there is no easy way to add it.
* No multi-line editing.
* No HTML5 debug and watch tool *(or at least I couldn't find it)*
* Dark theme is clunky due to the windows form UI.
* No easy way to restart the Haxe completition server.

---

## VSCode
> Visual Studio Code is a code editor redefined and optimized for building and debugging modern web and cloud applications.

**PROS**
* Whatever you can think of, there is an extension for that.
* Git client included.
* Multi-line editing.
* Included system shell and task system.
* Really good dark themes out of the box.
* An easy way to restart the Haxe completition server.
* CodeLenses. A quick way to know how many times a variable/class/method was used and where

**CONS**
* Doesn't support Haxe/OpenFL out of the box.
* Rocky to have it build and serve an HTML5.
* No class wizard. *(There is an extension, but it falls short)*
* You have to be handy with a system shell.
  * There is no "project file". You load up folders with ´code path/to/folder/´

---

So you are ready to take a leap of faith and come into the VSCode side of the force, or maybe you are taking your first steps in haxe... or maybe you are just bored... Anyway, let's begin with this trip.

### **Use Haxe 4**
*Wait... what?*
Yes, even though Haxe 4 is still RC at the time of writting this, the VSCode extensions for Haxe work better with version 4 than with the 3.x.
Fear not if you must use haxe 3.x for it will still work.

### **Set up the Extensions to work with Haxe/OpenFL**
*I am assuming that you already [installed Lime/OpenFL from haxelib](https://www.openfl.org/download/)*
You will need the Haxe and Lime extensions for everything to work and while we are at it, why not install some extras?
* [Haxe extension pack](https://marketplace.visualstudio.com/items?itemName=vshaxe.haxe-extension-pack): Includes the Haxe extension and some debuggers for Flash, HashLink and HXCPP
* [Lime](https://marketplace.visualstudio.com/items?itemName=openfl.lime-vscode-extension): OpenFL runs on top of lime. You need this.
* *optional* [Chrome Debugger](https://marketplace.visualstudio.com/items?itemName=msjsdiag.debugger-for-chrome): If you are exporting to HTML5, this will be a lifechanger.
* *optional and questionable* [HaxeManager](https://marketplace.visualstudio.com/items?itemName=jarrio.hxmanager): It attempts to fill the space that the Class Wizard from FlashDevelop left but it could be more annoying than helpful

*(while you are at it, why don't you take a look at the available extensions? Here you can find my personal recomendations)*

### How to create a project without the "Create project" assistant?
*Where is your god now?*
Ok, take a deep breath and keep calm. You will need to use the system shell.
If you are on anything UNIX-y you might think this isn't a big deal but for us windows users it can be really spooky.
We need to open a shell in the folder that contains all of our projects. In my case is a folder called "Projects" in my "Documents" folder.
The fastest way to achieve this is to hold `Ctrl + Shift` and right click on an empty space inside that folder.  You should see an option that says `Open CMD window here` (or maybe it says `Open PowerShell window here`). If this doesn't work you will need to open a console and navigate to the right folder. [Such explanation escapes the idea of this post. So I will just link you to a random tutorial of the internet.](https://www.bleepingcomputer.com/tutorials/windows-command-prompt-introduction/)
*(If you want to be a power user you can try to use PowerShell or any other replacement. I personally use [Cmder](https://cmder.net/))*

Are you still with me? Good. Lets create a project by typing
`openfl create project <ProjectName>`
then we just move into the directory that got created for our project with
`cd <ProjectName>`
from here we can open VSCode by typing `code .` *(We can also open vsCode and go to `File->Open Folder` for this last thing)*

### Build Once
*Wait, I've opened VSCode but I can't get the autocomplete on things. What's going on?*
You won't get the Completition server up and running if you don't build once. If you opened vscode you might have already seen an error that the haxe server had an error.
You need to build once first. The build command is this: `lime build <Platform>` [(How to become an expert on CLI-fu)](https://lime.software/docs/command-line-tools/basic-commands/). If you understand what that means and can run it, yay you did it! If not, let's use the tools we have...

From inside VSCode (If you haven't already go to `File->Open Folder` and select your project folder) we are going to press `Ctrl + P` to get the command bar. There we type `>Tasks:Run Build Task` *(Where the `>` should already be written and the autocomplete will kick in to help you)*.
Lime will ask you a few questions and finally it will run the command `lime build`. When this is done you should see `Terminal will be reused by tasks, press any key to close it.`.
After that we will need to restart the haxe completition server (This is something useful to learn since you can do it whenever the code completition breaks): Inside VSCode hit `Ctrl + Shift + P` and write `>Haxe: Restart Language Server` *(where the `>` will be already written and halfway through your typing it will suggest you what you were trying to do)*

Code completition should be working now.

### Testing
*Can I just hit f5?*
What do you want to test on? 


### Quick Guide of shortcuts that changed from FlashDevelop to VSCode
*Remeber you can always make your own hotkeys.*
|                                        | FlashDevelop           | VSCode                                                                                                                 |
|----------------------------------------|------------------------|------------------------------------------------------------------------------------------------------------------------|
| Go To Definition                       | `F4`                   | `F12` (`Alt+F12` to "peek")                                                                                            |
| Magic Autocomplete and code suggestion | `Ctrl + Shift + F1`    | `Ctrl + .`                                                                                                             |
| Duplicate Line                         | `Ctrl + D`             | `Ctrl + C -> Ctrl +V` (no need to let go of `Ctrl`) `Shift + Alt + Down` (you can use `Up` if you want the copy above) |
| Move selected lines up or down         | `Ctrl + Alt + Up/Down` | `Alt + Up/Down`                                                                                                        |

This is my post

I love it even if it is ugly

# Lets do something big here.

aaaand we are done.