---
layout: post
title: "CommonUI Demystified: Focus, Input Routing, and Activatable Widgets"
sub_title: "Disambiguating Focus, Navigation, Input, and Selection"
excerpt: "We all heard that CommonUI will help you make User Interfaces that are easy to control and navigate with a gamepad, but what is it really doing? and, more importantly, why is it not doing what you think it should be doing?"
categories:
  - Unreal Engine
tags:
  - unreal engine
  - user interface
  - game development
---

# CommonUI Demystified: Focus, Input Routing, and Activatable Widgets

So, if you are making a game in Unreal Engine 5, you might have heard that "CommonUI" is a great way to make sure your UI is gamepad friendly and to start from Lyra as a starting point...

The problem comes when something doesn't work as you think it should work and you try to google the answer, but you have no idea where CommonUI starts or ends, so you end up googling the wrong thing or applying a solution that works _against_ CommonUI!

In this post, I will try to explain how CommonUI tries to help you with gamepad-friendly UIs, what it does for you, and what it doesn't do for you.

(By this point I will assume that you already added CommonUI to your project and watched/read a couple of tutorials and kinda know what an Activatable Widget is. Even if you think it's just a fancy way to make some widgets visible and others invisible, that's enough. Here you will learn what _other_ things _Activating an Activatable_ does.)

## Glossary and Disambiguation

Let's start with the Vanilla UMG _-VS-_ CommonUI:
- Vanilla Unreal Engine (has been with us since UE4)
	- Slate: The actual thing in charge of drawing the UI and handling inputs. C++ only and older than UMG by a couple of versions (4.0 vs 4.2, I think)
	- UMG: The UI framework that you can access from the editor. It's just a wrapper on top of slate.
- New, cool stuff
	- CommonUI: A UI framework built on top of UMG for Fortnite and Paragon that Epic gave us to show us how to make UI that doesn't suck (too much)

### Disambiguation
There are very similar words that mean different things. So first check these fancy words...

- **Focus / Focusable**: The light blue box that shows up when you press the _Tab_ key or the _Arrows_ key to move around. (Vanilla stuff, sometimes used by CommonUI. But Vanilla in itself)
- **Navigation**: The fact that pressing the _Tab_ or _Arrow Keys_ or _Gamepad DPad_ moves the blue box around. (**100%** vanilla stuff. CommonUI _never_ uses this)
- **Input Routing**: If you press a button in your gamepad/keyboard... where does it get handled? (Vanilla + CommonUI)
- **Input Config**: Do you want your game to hear the inputs? Only the UI? Both? This "config" is in charge of deciding that. (CommonUI)
- **CommonUI Activatable Widgets**: The main way CommonUI allows you to change and manipulate Focus, Input Routing, and Input Configs.
- **Selected / Selectable**: A special word reserved for CommonUI Buttons (`UCommonButtonBase`). A "Selectable" button is what I would call a "toggleable" button. Only Buttons can be Selectable. Beyond Buttons, this word is meaningless/confusing.

Now, let's tackle things one by one!

## Focus and Navigation.

**Focus** is a concept that exists deep down in Slate. You can think about it as the _light blue box_ that shows up around buttons and other widgets when you start pressing _directions_ or the _Tab_ key.

![](/assets/images/focusnavigationinput/renderfocus.png)

(You can force the blue box to always be visible or hide it entirely)

Not all widgets support focus; they have to be `bFocusable` and properly answer to an event of "Hey, I gave you focus". This answer is done by returning an `FReply` on the method `NativeOnFocusReceived` or the blueprint counterpart `OnFocusReceived`.

It is important to note that **one and only one** widget has focus at **all times**. You never focus _nothing_ nor _more than one thing_.

**Navigation** Is the magical guessing that Slate does so that when you press a direction on the keyboard or gamepad, it knows what widget is in that direction on the screen. This can be altered by setting up the behaviours for how the widget navigate for each direction. This does **not** care which widget is a brother, parent, or half cousin twice removed of the current widget; **only the widget that is spatially in the correct direction**

![](/assets/images/focusnavigationinput/navigationescape.png)

"Escape" means "Go to the widget in that direction"

**This is a Slate/UMG vanilla process. It doesn't care about Activatable widgets and if they are active or not. If a widget is focusable and visible, the focus can go to it!**

Clicking a Focusable Widget is also a good way to set the focus to that element. However, when using a gamepad, moving the mouse pointer around and clicking stuff is not possible, so it's very important that you never get your focus "stuck" somewhere that you can't navigate away from.

### What does CommonUI do for Focus and Navigation?

**Activation Focus**: To avoid the focus getting _stuck_ somewhere when you open/close a dialog or panel (whenever an activatable widget gets activated or deactivated), CommonUI will check the "Activatable widget that is both active and on top of everything else" (internally called as the _leftmost active widget in the tree_) and ask it where the focus should be (`GetDesiredFocusTarget`).
This widget can skip the question (by setting `bSupportsActivationFocus` to false) and in that case, CommonUI will keep asking the active widgets until someone doesn't skip it, in which case you are met with one of many possible outputs:

- `[User %d] Set AutoRestoreTarget`
Your Activatable had the `bAutoRestoreFocus` set to true. So it remembered the Focused widget the last time it was the foremost active widget and focused that.

- `[User %d] Focused desired target %s`
Your Activatable widget has pointed to a valid target with `GetDesiredFocusTarget`

- `[User %d] Leaf-most node [%s] did not set focus through desired methods, but the currently focused widget is acceptable. Doing nothing, but this widget should be updated.`
You did not implement `GetDesiredFocusTarget` but you lucked out because the currently focused widget is a child of your activatable. So it did nothing.

- `[User %d] No focus target for leaf-most node [%s] - setting focus directly to the widget as a last resort.`
You did not implement `GetDesiredFocusTarget` but you lucked out because your Activatable Widget was also a Focusable Widget. So that got focused.

- `[User %d] No focus target for leaf-most node [%s], and the widget isn't focusable - focusing the game viewport.`
The bad ending of Focus. You did not set the focus target, and your Activatable is not focusable. Your focus got lost in the void. You might want to avoid this at all costs to avoid sending it _somewhere_ where it might get _stuck_

**Synthetic Cursor**: This is a fancy name for "CommonUI will teleport an invisible mouse cursor to the focused element". This is so that your buttons and any other widget that wants to animate _on mouse over_ can do the same animations when you are using a gamepad. Sadly, this doesn't work on Keyboard Navigation. You need to hack a bunch of the CommonUI source code if you want it to work that way.

### Debugging Focus

![](/assets/images/focusnavigationinput/debuggingfocus.png)

In the **Widget Reflector** (Tools->Debug->Widget Reflector), you can set the "Pick Hit-Testable Items" to "Show Focus".

Now, the focused element will have a green box around it, and in the tree of widgets that the reflector shows, the green highlighted elements are the _focus path_.

## Input Config and Input Routing

### Input Config

Something very common in games is that when you open a menu, your inputs stop moving your character and move the menu, and when you close this menu, the inputs go back into the game and not the UI.

Before CommonUI you had some methods/nodes to call, things like `Set Input Mode to UI/Game` and `Set Show Mouse Cursor`.

![](/assets/images/focusnavigationinput/dontdothis.png)
**Don't** use these if you are using CommonUI

**Input Config** is a CommonUI tool that helps you with that. **Instead** of calling all those methods to set your inputs exactly how you want them and then remembering to set things back to how they were, Activatable Widgets will ask for a _desired_ `FUIInputConfig` with the method`GetDesiredInputConfig`. 

![](/assets/images/focusnavigationinput/dothis.png)

That struct has all the settings you might want for input: sending input to Game, UI or both, showing or hiding the mouse cursor; and capturing/locking the mouse cursor.

When an Activatable becomes the foremost active widget, it will try to set this config and when it deactivates it will request for the new foremost active widget to activate their config.

This means two things:
- You must **not** handle the input mode by yourself! Do not "Set Input Mode" manually. Trust the Activatables.
- On deactivation, Activatables don't "restore the last config" but "ask the new active widget". If you deactivate your last activatable, this question falls and no input mode is changed. So if you only had one activatable that had a "Menu Only Config", once you deactivate it, you won't get back to a "Game Config"! Keep a bottom "HUD" activatable that you never deactivate with the "Game Config" so the system can apply the right input (this is what Lyra does, btw).

### Input Routing

**Input Routing** is what you can use to listen for _any button that isn't navigation_ and make sure the correct widget hears it. We can set our input mode to menu, great! But now if I say we want to press `Y` to do something... How do we listen for that event? And if two things want to hear `Y`... who would hear it first?

This is where the _activatable tree_ shines.

When you press any key on your keyboard or button on a gamepad (and you are on a "Menu Config"), Slate will ask the currently focused widget if they want to handle that input. If the focused widget doesn't handle it, it will ask the parent of that widget, and the parent of that, and so on until somebody handles it or it reaches the root of the tree and nobody handled it.

If you are using CommonUI, **Nobody** should handle that event.

This is because we want to reach the root of the tree, a thingy called `UCommonGameViewportClient`, here the input will get routed in a different way by checking the Activatable Tree.

The first and foremost active Activatable Widget (called the **Active Root**) will check if any of its Activatable descendants want to handle the input (the descendants will ask the same question recursively) first, and if none did, handle it itself.

(There is a special case, `bIsModal` which makes an activatable handle **all** requests and stop cascading to its children. Effectively eating all inputs that come their way, and pruning the tree)

**It doesn't matter if the foremost active tree doesn't have focus**

In the case of Activatable siblings, descendants of a **Non-Activatable tree**, the _foremost_ Activatable becomes the **Active Root** and the other brother is **never** checked.

In the case of Activatable siblings, descendants of an **Activatable tree**, the _order_ in which the _parent_ iterates the children is the order in which they were added and *not* the foremost first (This was very unintuitive for me). 

**Matryoshka-ing your Activatables is okay, but having Activatable siblings in the matryoshka is a footgun**

### But how do I add an action?

`UCommonButtonBase` can register an action to trigger the button click when a key is pressed. (Focus won't jump to it; just the click callback will trigger)

All `UCommonUserWidget` (and thus, `UCommonActivatableWidget`) can call `RegisterUIActionBinding` to listen for a key (This is only exposed from C++)

All actions you register live in the first Activatable parent they can find. This is so that the whole recursive check I described above doesn't have to check every children and can just look at Activatables.

### Dumping the Activatable Tree

To see all the activatable widgets and which ones are activated or not, you can use the command `CommonUI.DumpActivatableTree` in the console, and you should get something like this in your Output Log:

```
LogUIActionRouter: Display: Dumping ActivatableWidgetTree for LocalPlayer [User 0, ControllerId 0]:
** Active Root **
W_WidgetName: IsActivated? [true]. LayerId=[58]. [0] Normal Bindings.
*****************
W_AnotherWidget: IsActivated? [true]. LayerId=[40]. [1] Normal Bindings.
-DT_UIActions: Owner [ButtonNameHere], Mode [Menu], Displayed? [false]
W_DeactivatedWidget: IsActivated? [false]. LayerId=[58]. [0] Normal Bindings.
```

Let's break it down!

```
** Active Root **
```
Only things in the active root can hear key presses. If your listener is not before the row of `*****************`, then it's not gonna be heard. (Can you guess if this would hear my `DT_UIActions`?)

```
W_WidgetName: IsActivated? [true]. LayerId=[58]. [0] Normal Bindings.  
```

- `W_WidgetName`: The name of the Activatable widget
- `IsActivated? [true]`: If said Activatable is Activated or not
- `LayerId=[58]`: The Layer order. Bigger means foreground-er. The biggest number here becomes the Activatable Root.
- `[0] Normal Bindings`: How many Keys are bound.


## Selected / Selectable and the UCommonButtonGroupBase

**Selectable** is a property of the UCommonButtonBase it's the ability for the button to behave almost like a toggleable button... but the wording can be confusing.

![](/assets/images/focusnavigationinput/selectable.png)

So...

- **Selectable**: Has the ability to "stay selected", or "stay depressed", or "toggled on"
- **Should Select Upon Receiving Focus**: If you want the button to become "toggled on" as soon as the gamepad focuses on it. **This doesn't mean it will automatically toggle off when you move the focus away!**
- **Interactable when Selected**: Broken as of 5.6 - Should prevent the button from changing state and firing events once it has been "toggled on"
- **Toggleable**: Allows you to "toggle off" the button by clicking it again
- **Trigger Clicked After Selection**: By default, it triggers your _OnClicked_ callback and then flips the `bSelected` boolean flag. If you care about the new "toggled" value, this flips the call order, and the `bSelected` bool is flipped first, and then _OnClicked_ is called

You might have noticed that besides _Toggleable_ there is no way to "toggle off" one of these selectables...
  
Except for **CommonButtonGroupBase**!
  
This is a UObject that says:
 
> Manages an arbitrary collection of CommonButton widgets.  
> Ensures that no more (and optionally, no less) than one button in the group is selected at a time
 
That's the missing piece! We can add a bunch of CommonButtons to this _Group_ and the group will make sure that _never more than one_ is active at a time!
On top of that, you can also enforce that _at least one_ is active.

(By the way, this is what the CommonTabListWidget uses to make sure you have one and only one tab selected)

---

You made it! Thanks for reading and I hope you came out of this post a little bit smarter.

Special thanks to [Sharundaar](https://bsky.app/profile/sharundaar.bsky.social) and [Dylan](https://x.com/DoubleDeez) from the [Unreal Garden Discord](https://discord.gg/KnWJ2jCSFk) for having the patience to explain me how things work.