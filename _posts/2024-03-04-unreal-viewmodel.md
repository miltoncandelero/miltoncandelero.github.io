---
layout: post
title: "Model View ViewModel for Game Devs"
sub_title: "UI Doesn't have to be hard!"
excerpt: "How to turn something meant for the boring app world into something useful for programming our game UIs"
categories:
  - Unreal Engine
tags:
  - unreal engine
  - user interface
  - game development
---
# What is MVVM?
Ok, this comes from the boring app world. It means **Model View ViewModel**. Let's define the first two concepts and how they apply to video games and then tackle the third one

- **View**: The User Interface (UI) layer. Only the visual part, the UI elements, controls, components, whatever you want to call them. Nothing more, not the information they show, only enough code to render on screen and visually behave as you want them.  
This is the easy concept to port to video games.
  
- **Model**: The data layer. Here lie the backends, the databases, the "business logics", the complex server calculations of very serious big boy stuff. (While some hardcore thinkers would split the data from the functions that modify the data, I will group both of them as the Model.)  
In video games, this is what I will call our capital G, "Game". Everything that is not the UI for our game. The levels, the amount of health a player has and how to hurt/heal them, if your character is wearing a hat, which kind of hat, which color is the hat and if the hat has a hat. Basically, the data and functions that make your Game be a game.

Then, if **View** is my UI layer, and **Model** is the data and functions that make a game be a game... what the hell is a **ViewModel**?

- **ViewModel**: It's the **Model** for the **View**. Let me say it again: It's the **Data and functions** for the **UI Layer**.  
It's a magical _thing_ that collects, computes, calculates, remembers and or invents all the information that your UI needs to show.

The MVVM _pattern_ is then a way of _decoupling_ your **Model** (Game) from your **View** (User Interface)

## The secret that nobody tells you about MVVM

Everybody that talks about MVVM will skip a **crucial** detail: There is a 4th part that makes the magic happen.

- **Binder**: A magical piece of code that is usually already written by somebody else (part of your engine or UI framework) in charge of connecting the UI elements of the **View** to the data of the **ViewModel**.  
This "Binder" works on an _event_ basis, each time something changes inside your **ViewModel** the Binder _programagically_ knows to update the UI element.  
Unreal Engine has a new shiny one, Microsoft had XAML with WPF (and now the community project Avalonia), Android has one (I haven't used it, but I know it exists.)

This magical piece of code lets you set the value of your object to a variable and link them together, when one changes the binder updates the other.

If you have no binder, MVVM gets _very_ hard to justify.

[![](/assets/images/viewmodel/MVVM.png) (Open in new tab)](/assets/images/viewmodel/MVVM-Background.png){:target="_blank"}

# Reasons to use MVVM

- Testability: If you are the kind of person to write Unit Tests, this is for you (I am not that kind of person)
- Reusability: You can make different UI elements that show the same data, since the data is collected and calculated in a ViewModel.
- Mockupeability: (The ability to make mockups) If your UI depends on an enemy to exist and the enemy isn't finished yet... you can't make the UI.  
If your UI depends on a ViewModel, you can make a "Mockup" ViewModel that instead of finding the Enemy answers random values and _voilà_, you can make the UI!
- Separation of Concerns: Your game should only worry about gaming and your UI about looking pretty. If you can avoid mixing those two concerns then your project will be easier to make and maintain.
- Discourage polling: As mentioned before, all MVVM work thanks to a magical thingy that _binds_ UI elements to the ViewModel. This magical thingy changes your UI when **and only when** the information you need to show changes.  
Another way of updating UI is called _polling_, and it means "asking every frame if the data changed". It's the "Are we there yet?" of UIs and should be avoided whenever possible. (Without the ViewModel, Unreal Engine default "Bindings" for UMG use polling. It's quite bad)

## Some examples of where ViewModel can save you

Does your PlayerHealthComponent (or wherever you store it) has a `GetHealthPercent()` method? Why though? How does it affect the Game? Do enemies care percentage? Do guns care?  
If you have these kinds of functions only for UI stuff, you are boggling down your character with things that don't matter for the Game.  
**The ViewModel should compute this percent value from the current and max health**

Does your UI button know all the possible types of weapon, with the associated mesh (even when they never show the mesh) so they can spawn the correct mesh in the player character hands when they buy it?  
Now you're giving too much responsibility to your UI. The only concern for your UI would be to look pretty.  
**The ViewModel should know who is in charge of spawning weapons and ping them with the information on which weapon should spawn**

All your UI elements need to flash red if your player health is `< 25`.
Do you connect each UI element to your player? What if we need to change that `25`? Do we need to go into each UI element to change the number?  
**The ViewModel has this `< 25` logic. All elements of the UI use the ViewModel computed result**

Your shop button needs to check the amount of money an item cost, how much money the user has, subtract the money, and spawn the item in the players hand.  
How many points of contact with the _Game_ did that single purchase button had?  
**The ViewModel is the one in contact with all the parts of the Game. The UI is only in contact with the ViewModel**

# Theoretical Example: Tower defense upgrade dialog

We are making the upgrade dialog for one of our towers in our Tower Defense Game.

The user can see the selected tower, the current stats of the tower and the stats for the next level. The user can also see the cost for the upgrade and there is a button to upgrade.

Happy path: The user has enough money. When the user clicks the upgrade button we subtract the money, we change the model on the world and increase the stats of the tower.

Unhappy path: The user doesn't have enough money. The upgrade button is grayed out. Clicking the button does nothing.

## View

We use the UI elements at our disposal to create the nice looking UI
- Player coin counter
- Tower upgrade dialog
	- Name of the tower
	- Cost of the upgrade
	- Current and next level
	- Current and next stats
	- Upgrade button

[![](/assets/images/viewmodel/View.png) (Open in new tab)](/assets/images/viewmodel/View-Background.png){:target="_blank"}

### Binder
We use the binder layer to connect our view to our ViewModel so our View updates automagically

## Model
We have the following objects:
- Player Wallet: 
	- Knows how many coins the player owns
	- Has the functions to subtract coins.
	- Has an Event that fires when the amount of coins was changed.
- Tower:
	- Knows it's Type
	- Knows it's Current Asset
	- Knows it's Current Level
	- Knows it's Current Damage and Range (stats)
	- Has an Event that fires when the tower was upgraded.
- Tower Shop:
	- Knows all tower types with their stats and assets per level.
	- Has the function to modify a tower asset, level and stats

[![](/assets/images/viewmodel/Model.png) (Open in new tab)](/assets/images/viewmodel/Model-Background.png){:target="_blank"}

## ViewModel
Let's glue things together!  
I will try to briefly explain where the data comes from and where it goes to.  
"Computed" fields are fields that don't quite exist as we need them, so we _craft_ them from the data we have
- Player coins
	- Read from the player when the Game triggers a "money changed" event
	- Shown in the player coin counter
- Computed tower name
	- Crafted from the tower type and the tower datatable for a display name
	- Shown in the upgrade dialog
- Computed tower cost
	- Crafted from the tower type, level and the tower datatable
	- Shown in the upgrade dialog
- Computed has enough coins to upgrade?
	- Crafted internally from ViewModel data: Player coins and tower cost
	- Not everything you show is text, this boolean value converts to
		- A text color: The price turns red if you can't afford the upgrade
		- The state of the button: If you can't afford it, the button is disabled
- Current level stats
	- Read from the tower and the DPS is crafted
	- Shown in the upgrade dialog
- Next level stats
	- Crafted the next level from the previous
	- Read from the datatable the stats
	- Crafted the DPS
	- Shown in the upgrade dialog
- Upgrade button callback
	- This is the information coming back from the UI. In this case it is just a clicked event
	- It gets forwarded to the Shop in the Game with the correct parameters


[![](/assets/images/viewmodel/ViewModel.png) (Open in new tab)](/assets/images/viewmodel/ViewModel-Background.png){:target="_blank"}

_(remember, dotted arrows are "events-driven" connections and full arrows are direct calls)_


# Now, let's make it using Unreal Engine's ViewModel

(I will do it in blueprints as it's more graphical to explain, but you can find how to do it in C++ in the [official unreal documentation](https://docs.unrealengine.com/5.3/en-US/umg-viewmodel/))

## Activating the plugin

Ok, the first step is to enable the plugin for ViewModel  
**Note: ViewModel is still in BETA as the time of writing. The looks can change, but the concepts should remain useful**

![](/assets/images/viewmodel/ViewmodelPlugin.jpg)

Something that it's not _necessary_ but I advise would be to prevent the _old_ (the default) binding system from showing up in the UI.  
This can be changed in Project Settings -> Widget Designer -> Property Binding Rule

![](/assets/images/viewmodel/preventbindings.png)

## Making the View

Now, let's ~~crudely~~ quickly make this UI  
This isn't a full UMG tutorial, I assume you have basic knowledge on how to make UIs  
(If you have _advanced_ knowledge, please don't hate me for not making a reusable component for the stats)

![](/assets/images/viewmodel/uglyUI.png)


## "Making" the Model

And these are the Game things  
This isn't a full "How to make a tower defense game". These are just husks of what a true TD game needs.  
The overall idea:
- The Player pawn keeps the amount of coins (this could be a component or a subsystem)
- The Shop has access to the datatable and triggers an upgrade on a tower (this could be a subsystem)
- The Tower can pew pew things
- DT_Towers is a _probably ugly_ Excel file your Game Designer _hopefully_ balances

![](/assets/images/viewmodel/fakegame.png)

## Creating the ViewModel

Now, for the good part! Let's make our **ViewModel**  
We _probably should_ split the player coins ViewModel from the rest of the tower things. For this example I will do everything together, but there is no hard rule on _how many ViewModels_ you need. Somewhere less than _one per textfield_ and more than _one for the entire game_ is the right amount.  
Some swear by "One VM per Model object", some by "One VM per UI _block_ (dialog, toolbar, etc.)" and some just find a middle point. It all comes from experience.

ViewModels are a base class for a blueprint

![](/assets/images/viewmodel/newviewmodel.png)

And inside... they look like a blueprint

![](/assets/images/viewmodel/emptyviewmodel.png)

Let's add a variable and see the cool things

### Field Notify

![](/assets/images/viewmodel/babysfirstnotify.png)

The little _eye_ button shouldn't surprise you... but what is that _bell_?  
That is the "Field Notify" button, and clicking it means that the ViewModel will emit an event every time that variable changes.  
Turning on that checkbox will change our **Set** blueprint nodes to a **Set with broadcast** nodes

### Computed Values
What about _Computed values_? Those aren't variables but functions!  
You need to make your function both **const** and **pure,** and the **Field Notify** will become available.  
**const** is hidden in the **advanced** dropdown!
![](/assets/images/viewmodel/computedfield.png)

### Chained notifications

Think about this... The "Has enough coins" needs to be re-computed every time one of this happens
- The player coins changed
- The tower cost changed (Tower got upgraded, or we select a different tower)

To connect one variable changing to another we:
- Select the Triggering variable (When _Player Coins_ change...)
- Open the Field Notify dropdown (... notify ...)
- Select who shall be notified when this variable changes (... _Computed Has Enough Coins_)

![](/assets/images/viewmodel/chainnotify.png)

So, whenever a value is modified, all the _chained_ values are modified.

### Finished ViewModel
This is our finished viewmodel.  

>But Milton, it doesn't look like the one in the graph above.  

Indeed, it doesn't, imaginary voice in my head! _No plan survives first contact with the enemy_  
That's why I am a strong believer in planning less and doing more, but that's an article for another day.

![](/assets/images/viewmodel/finalviewmodel.png)

### The _easy_ part, connecting our View and our ViewModel

Open the Viewmodels window from our Widget

![](/assets/images/viewmodel/openviewmodelwindow.png)

and add our ViewModel  
(if nothing happens after pressing select, close and reopen the window, remember this is a beta feature)

![](/assets/images/viewmodel/viewmodelwindowthatneverbreaks.png)

Now, the Bindings buttons should show up and offer you the view model aaaaand

![](/assets/images/viewmodel/yourfirstbinding.png)

**DENIED!**  
That thing is a number! You can't set a number to a text, you dummy!

![](/assets/images/viewmodel/denied.png)

Let's open the Binding windows to see our mistake and an overview of the elements

![](/assets/images/viewmodel/Bindings window.png)
To fix this issue, we don't need to make any changes to our ViewModel, Unreal's magical binding layer has converting functions...

![](/assets/images/viewmodel/usingfunctions.png)

And then we click on the Chain icon to connect again to our viewmodel

![](/assets/images/viewmodel/bindingparameters.png)

We can now connect all our properties to all our fields

### The hard part, who creates the ViewModel and how it connects to the Game?

#### Who creates the ViewModel?
For the ViewModel creation, unreal gives us some options...  
(Select your VM in the Viewmodels window and check the details panel)

![](/assets/images/viewmodel/vmcreationmode.png)

- **Manual**: The widget initializes with the Viewmodel as null, and you need to manually create an instance and assign it.
- **Create Instance**: The widget automatically creates its own instance of the Viewmodel.
- **Global Viewmodel Collection**: Refers to a globally-available Viewmodel that can be used by any widget in your project.
- **Property Path**: At initialization, execute a function to find the Viewmodel. The Viewmodel Property Path uses member names separated by periods.
- **Resolver**: An object with a function in charge of providing the Viewmodel. (This is not documented, but it looks pretty simple)

For our example, we will pick manual and let the HUD actor do the linking...

In our HUD actor, we create our ViewModel instance and store it.  
We also have a function for when the user clicks a tower in the world (this isn't handled by our View since we are clicking a 3D mesh in the level and not a Widget Button)

![](/assets/images/viewmodel/createvminhud.png)

That **Initialize** Function is **not** usually part of a ViewModel... We will look at it in just a second... but now, we need to feed this ViewModel into the Widget

![](/assets/images/viewmodel/createwidget.png)

Simply creating the Widget asks for the `VM Tower Upgrade` reference, since we set it to manual creation.

#### How do you connect the ViewModel to the rest of the game?

We have two ways of changing a ViewModel:
- **Event Driven**: The ViewModel listens for events in the Game.
- **Direct**: The game finds the ViewModel and punches it into shape.

I prefer the **Event Driven** approach but from time to time we need to do a good ol' punching.

Let's look again at how we create our ViewModel:

![](/assets/images/viewmodel/createvmzoom.png)

**Initialize** and **Select New Tower** are not part of the default viewmodel but are what I use to connect all my events.

- **Initialize** is used to feed Game objects into the viewmodel so it can subscribe to events
- **Select New Tower** is my way of punching the viewmodel and letting him know that the tower is now completely different.

Let's see how I made those

![](/assets/images/viewmodel/initializevm.png)

**Initialize** is simple, I just store a ref for the Shop, so I can ask for information (mostly because the Shop isn't a Subsystem, thus I don't have a good way of getting a ref) and I use the player to subscribe to the coins changed event. I use this event to update my local coin variable which in turn broadcasts the change to the UI.


![](/assets/images/viewmodel/selecttower.png)

**Select New Tower** is trickier: I want to listen for the "Tower Changed" event that will fire when a tower gets upgraded, but I need to listen to the right tower (since there might be more than one in the world.)

So when the "Select new tower" event gets punched externally, I unsuscribe from the previously selected tower (if any) and subscribe to the new tower.

But what do I subscribe? Well I fire the same event as if the entire tower had changed. We can manually fire "Pretend this value changed and update everything" events by using the "Broadcast Field Value Changed".

So, in this case, when the tower gets upgraded we tell our viewmodel "Pretend we just selected an entire new tower and update whatever is needed".

---

# We are done!

And there it is, we defined MVVM, we made a theoretical example, and we implemented it on Unreal Engine!

> But where is the tower defense game? I wanna play it!

Oh, you silly voice in my head, there never was a tower defense game!

Thanks for making it to the end, and I hope you learned something!

---

As always, all my drawings are made using excalidraw and the PNG itself are editable, or you can download my project file [here](/assets/images/viewmodel/MVVM.excalidraw)

Special thanks to everyone at [BenUI's Discord](https://discord.benui.ca/)
