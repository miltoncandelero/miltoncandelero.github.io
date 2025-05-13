---
layout: post
title: "How to develop an update without breaking production"
sub_title: "The Branches Iceberg, or Stability Branches, or Spicy Gitflow"
excerpt: "Have you ever found yourself in the need of hiding a feature with some boolean flags because these features are not for this delivery / patch? Or even worse, you broke a build because you left a half-baked feature for the next delivery in the current one?"
categories:
  - Git
tags:
  - git
  - game development
---

# The Branches Iceberg

Have you ever found yourself in the need of "hiding" a feature with some boolean flags because these features are not for this delivery / patch? Or even worse, you broke a build because you left a half-baked feature for the next delivery in the current one?

What's the solution? Twiddle your thumbs while you wait for the current patch to be delivered before starting work on what's coming for the next one?

If we only had a way to split work and continue working on unstable features that will be needed in the future while some other part of your team works on the final changes for the stable release coming soon...

![](/assets/images/stabilitybranches/iceberg.png)

Introducing: **Stability Branches** (or "Spicy Gitflow")

**This is a part of my "git gud or git rekt" series.**

---

## Stability Branches

Back in 2020, Epic released a white paper called ["Workflow on Fortnite: Collaboration on Large Teams with UnrealGameSync"](https://cdn2.unrealengine.com/workflow-on-fortnite-whitepaper-final-181633758.pdf?utm_source=chatgpt.com), and among a lot of things, they dropped the concept of "**Stability Branches**".

A stability branch is a branch you make from your current development branch when a new release approaches. The idea (and name) for that branch is to make sure that your game must always be buildable (no build/compilation errors) and it should be crash-free (or as crash-free as possible). 

On this branch you should only fix bugs and make balance patches without introducing new features. This way you build towards a very stable release, and you release a very stable and bug-free update.

But... this sounds all too familiar.

## Spicy Gitflow

If you are a hardcore, old-school, silverback developer, you might have already noticed that what I describe as *Stability branches* are just `release/*` branches in a Gitflow... flow, and you would be correct!

Let me just bring people up to speed on Gitflow:

- You have a `develop` branch where you integrate all changes.
- Your changes happen on `feature/*` branches that then integrate into `develop`.
- You branch from `develop` with `release/*` when you start ~~stabilizing~~ polishing a release.
- You release by pushing the `release/*` into `main` or `production`.
- If any critical bug happens, you hotfix it into `main` (or sometimes a `hotfix/*` branch immediately merged into main).
- All changes from a more ~~stable~~ _releas-y_ branch are merged down immediately for history's sake.

With this approach, thinking of live games (updates every two~three months), we can use our `release/*` branches as _stability branches_ to stage the next immediate update.

This assumes you branch from your _unstable_ `develop` branch when all the features are already in and you are ready for a feature freeze, leaving this `release/*` branch to be only for QA, bugfixing and mayyyyybe some polish and juice.

In an ideal world, hell yeah! In reality... we will have to bend that rule.

## Bending the law, bending the law!

Unless you are _exactly_ staffed, where you finish the update the day before release and only then you start with the next update (in which case I would say you might be understaffed), you can find yourself in a position where you can't collaborate on the upcoming update because there aren't any open tasks you can grab, but you know what is coming for the update-after-this-one. How can you start working on that without risking the current, upcoming update?

Well, `feature/*` could be your choice, making yourself an _unstable_ branch to work with and waiting until the update was delivered before integrating your changes into the `develop` branch, but if we bend the rules of _stability_ branches, instead of keeping `development` stable, we make it inherently unsafe, and everything that needs to become more _stable_ branches away from it. This means the part of your team that is getting the update ready branches away from `develop` into their own `release/*` branch where they are still adding features but hopefully can hit the feature freeze ASAP and _stabilize_ that branch so QA can come in and get ready for the final push to `production`.

We bend the law of "stability branches are always feature frozen," and we start seeing our branching strategy more like a gradient.

![](/assets/images/stabilitybranches/gradient.png)


## End of life of a `release/*`

This is an easy one. Once a `release/*` is merged into `production` (released to the public), that branch is officially dead. No further changes can happen on it. Any change happens directly on `production` and is considered a hotfix.

## Always merge down, Merge up only on milestones

### Always down
If the feature for this update was finished in `release/*`, when the hell does it get back into `develop`? What if we had an emergency hotfix? When does that get back to `develop`?

Well, the idea is that all changes that happen in a _more stable_ branch trickle down _immediately_ to their more unstable brethren.

- Hotfixes directly on `production` must be merged back to any active (unreleased) `release/*` and back to `develop`.
  - Even if the hotfix is an ugly hack that `develop` will fix in a better way, merge the fix back for the sake of keeping history, then fix it properly.
- Features, fixes, content, etc. that gets added to any `release/*` should be merged back into `develop` at the earliest convenience.
  - In case it wasn't obvious, when a `release/*` branch dies (a.k.a. is merged into `production` and released to the masses), that final state must also be merged back into `develop`.

### Almost never up
The only moment when you merge up, is when you reach the end of the life of a `release/*` branch and you merge it into `production` and release the update.

However, I know that _no plan survives contact with the enemy_, so it might happen that a very critical bug was fixed in `develop`. In this case, a surgical cherry-pick could be performed to try to get this very important fix to the active `release/*` branch or even to `production`.

## Auxiliary branches

Adding to these concepts, I add two more branch families to this mess:

- `content/*` branches: The stable cousin of `feature/*`. These branch off `develop` at a point where the game was stable enough to open the editor (and maybe even build the game). These are made and used by level designers, artists, and other "non-programmers" to add content to the game without ever compromising the "buildability" of the game. This allows less techy people to work in a somewhat stable environment. While these branches are usually more _stable_ than `develop`, they are not considered to be more _stable_ for other purposes.

- `patch/*` branches: The less crashy cousin of `hotfix/*`. These branches, the same as `hotfix/*` split off of `production` instead of the unstable `develop`. These are changes that will only tweak values or adjust the game very minimally. If your game has some sort of A/B testing and feature delivery, these branches don't need to exist since that system should handle those changes.

## Summary of branches

In stability order, with children underneath them

- `production`: What your players can see. This is what gets built and delivered.
  - `hotfix/*`: "oh f*ck" branches. Branch from `production`, fix something, test it, merge back to `production`. Then merge into everything below.
  - `patch/*`: For balance patches/gameplay patches. If you have a feature delivery system, A/B Testing or something like that, you won't need these.
- `release/*`: Born from `develop` once you start _stabilizing_ a release, they "die" once they get merged into `production` and released to the public. Merge down frequently.
- `develop`: The unstable core where everything that is being developed gets merged.
  - `feature/*`: The _very_ unstable changes. If you have a small team with _very_ good communication, you can avoid these branches and just commit everything into `develop`
  - `content/*`: The _stable_ unstable changes. Mostly useful in engines where a half-baked piece of code can keep you from opening the editor (ejehm... Unreal Engine... ehjem). It's useful for non-programmers to have access to a non-crashing build where they can experiment and test.

Finally, a very spooky graph of how stuff can look with this system.

(Arrows point forward in time, this is not a proper git graph! [In a git graph, arrows point to the parent](./git-time-travel))

![](/assets/images/stabilitybranches/graph.png)

Now, listen here to the important part:

### You don't have to use them all!

I know that I speak (and thus, I write) like I am laying truth _granted onto me from the gods of project management above and that thou should'st do as I say or ill things shall befall thee_

But that ain't how this works.

You know your team better than I do (and if you don't, you probably should start trying to get to know 'em).

Steal what you find useful; don't steal what you don't find useful. Bend, twist, break, and adapt the system to work for you, not against you.

Thank you for your time <3

![](/assets/images/stabilitybranches/full.png)
