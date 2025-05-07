---
layout: post
title: "What can we learn (and steal) from Unreal Engine GameplayTags"
sub_title: "Stealing is bad"
excerpt: "If you ever worked long enough with Unreal Engine, you should've come across _Gameplay Tags_. I don't mean the Name tags that Unreal uses (although those are pretty cool too) but the Damage.Elemental.Fire kinda thingy."
categories:
  - Unity
tags:
  - unity
  - unreal engine
  - game development
---
# What can we learn (and steal) from Unreal Engine GameplayTags
If you ever worked long enough with Unreal Engine, you should've come across _Gameplay Tags_. I don't mean the "Name" tags that Unreal uses (although those are pretty cool too) but the `Damage.Elemental.Fire` kinda thingy.

It might seem like an unholy mix between gameplay and a version number, but if we look closer, we find a tool that should replace booleans, enums, and hardcoded strings to improve on performance, DX, UX, FX, and _almost*_ any other X you can think of.

(* I don't think we can help Twitter by now.)

So... let's see **what** and **why** we want to ~~steal~~ borrow before diving into the **how.**

---

## What is a GameplayTag?

![](/assets/images/gameplaytags/unreal.webp)

Well a gameplay tag is just a **hierarchical** way of handling our tags and we can use that to **semantically** represent our game.

This means we can have _nested tags_ (that's the hierarchical part) that represent our game _as is_ instead of having to adapt our game to an unholy amount of booleans and enums.

Look at these tags

```
Damage.Elemental.Fire
Damage.Elemental.Cold

Damage.Piercing.Arrow
Damage.Piercing.Bullet

Damage.Slashing.Sword
Damage.Slashing.Axe
```

You may think that we could easily do this with an `enum` (Please tell me you didn't say a `string`)

```
EnumDamage {
	Elemental_Fire
	Elemental_Cold

	Piercing_Arrow
	Piercing_Bullet

	Slashing_Sword
	Slashing_Axe
}
```

But you didn't notice something really cool of the gameplay tags I showed before: `Damage`, `Damage.Elemental`, `Damage.Piercing` and `Damage.Slashing` **are also tags**

But Milton, I hear you ask, why does this matter?

## Asking the right questions

Thanks to the hierarchy of the tags, we can not only ask questions like "Is this tag equal to `Damage.Elemental.Fire`?" but we can also ask "Is this tag in the `Damage.Elemental` family?" or even "Is this tag even a `Damage` to begin with?"

But what if we could have _more_ tags...

## More than just an `Array<GameplayTag>`

Well, if we want more of one thing, we just put it into an array or list and call it a day, right?

Let's see, an array can ask one way:

- Does this container have this tag?

But thanks to the GameplayTagContainer we can ask the other way around

- Does this tag match _Exactly_ with any/all of the tags in the container?

- Does this tag match any of the _Families_ found in this container?

And here you start to see the potential...

Complex filters that could once span many booleans and enums can now read as "Do you have a tag in your container for weaknesses that matches with any of my damage tags?

```
if (WeakTags.MatchesAny(DamageTags)) {
	// Massive Damage
}
```

## Expand and extend without fear

Right before release, your game designer comes to you and says "I've invented a new elemental damage type: **Emotional Damage** "

_You are not entirely sure if that kind of damage should be Elemental, but you follow along for the example._

Well, if your entire code depends on enums, switches, hopes and dreams you are going to play a very precise game of wack-a-bug, checking every single path where you ever checked for damage, but with gameplay tags you just create a new one, `Damage.Elemental.Emotional` and that's it!

Everything that was weak/strong to `Elemental` before, now responds accordingly to this new kind of `Elemental`, everything that was only weak/strong to a particular kind of elemental (like `Damage.Elemental.Fire`) ignores it properly. No bug hunting needed, everything just kinda works.

## Status, Counters and Identities

GameplayTags not only refer to what something **is** but they can also refer to **How** they are.

Thanks to english using the verb to-be for _ser_ and _estar_, this is easier explained with examples, trust me.

- Identity: Your weapon is a `Weapon.Simple.ShortSword` you are a `Class.Barbarian.Berserker`

- Status: You currently are `Status.Debuff.Stun`, you are immune to `Status.Debuff.Fear`, the enemy is `Status.Buff.Haste`

But now, the more esoteric use: Status Counters!

Thanks to a particular kind of `GameplayTagContainer`, we can have a `GameplayTagCountContainer`! That remembers how many times you added and how many times you removed a tag!

- You currently have 3 `Damage.Elemental.Poison` Tags. You will lose 1 `Damage.Elemental.Poison` Tag every 24 hours. A poisoned blade, adds 1 `Damage.Elemental.Poison` Tag on hit.

## Okay, I'm sold! How can we steal this?

Ok, we could implement this with just strings. You just need to implement the `GameplayTag` data structure that is a string that splits by the `.` and those are the families and compare ahead...

But somebody already did it and it's waaaaay better.

![](/assets/images/gameplaytags/unity.png)

One wonderful gentleman from Brazil made [https://github.com/BandoWare/GameplayTags](https://github.com/BandoWare/GameplayTags) an extremely good implementation that goes beyond just splitting strings, these are performant and easy to use thanks to the editor integration!

---

### That is it for now friends

I hope you enjoyed it and I'll see you next time we pick something to ~~steal~~ borrow!