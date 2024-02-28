---
layout: post
title: "Return to Monke (Learn to use git branches)"
sub_title: "Git is a time machine, no need to bring your own!"
excerpt: "I used to be like you: <i>Branches are for monkeys</i>, I would say, but then I understood that even when you think you are always working on the trunk, on your main and only branch, you are still using more than once branch, and you should return to monke and learn to swing on those branches."
categories:
  - Unreal Engine
  - Git
tags:
  - unreal engine
  - git
  - game development
---
# Return to Monke
_how you should stop merging everything and learn to rebase from time to time_

Recently I've seen how people are scared of using git branches and how rebase seems like this incomprehensible tool to destroy your history and reenact the [Minitrue from 1984](https://en.wikipedia.org/wiki/Ministries_in_Nineteen_Eighty-Four#Ministry_of_Truth).

I used to be like you: _"Branches are for monkeys"_, I would say, but then I understood that even when you think you are always working on the trunk, on your _main_ and only branch, you are still using more than once branch, and you should return to monke and learn to swing on those branches.

**This is a part of my "git gud or git rekt" series**

## WTF Is a branch anyway

Git is a marvelous tool, but it's not as scary as you might think, for the scope of this article, we can consider that a branch is just a pointer to the last commit of that branch.

And if we remember that a commit is just a pointer to their previous or parent commit, this graph should make sense to you

![](/assets/images/branches/Branches1.png)

Just in case: remember that commits point to the **previous** commit in history. In this graph the **past is at the left** and the **future is at the right**.

## You use branches every day and you don't realize

You might think that you don't, after all you are a trunk merging connoisseur, obviously you would know if your delicate hands touched a branch... right?

There is the "uploaded thing", you probably call it "GitHub" or "the pushed repo" but git calls this `remote` and the default one is usually called `origin`.  

To handle the fact that this `remote` has different changes to what you have locally, git tracks the remote upstream branch as `origin/main` _(name of remote/name of branch)_

Your local repo isn't always _in sync_ with "the uploaded thing", sometimes you _pull_, sometimes you _push_.

Well, have you ever `git pull` before? What if I tell you that what you did was use a shorthand for two commands: `git fetch` and... `git merge`! **dun dun duuuuun**

Let's break it down:
- `git fetch` means "go to the online thing and download the branch to my pc."
- `git merge` means "now, grab that branch you download it and slam it against my local branch until something happens."
  
> But wait... so you _actually download the remote branch before merging it with the local things?_ **MADNESS!**
  
Yes, you do. Now, look at this graph:
  
![](/assets/images/branches/Branches2.png)
Here you might say _"There are no branches, it's just a straight line with no bifurcations"_.  
If you do, please read the previous section, where I told you: **"a branch is just a pointer to the last commit of that branch"**. 

![](/assets/images/branches/Branches3.png)
_This is the same graph as before, it just looks different, but the arrows point to the same place so it's the same._

This graph has two boxes: `main` and `origin/main`. There are **two branches**.  

Git will call this something along the lines of "You are 1 commit behind".

## Your first branch swing

Let's take baby steps, let's say you just cloned a new repo, you have `main` up to date, and you now need to branch out to make a new feature because some _eeeeevil_ CTO forces you to use branches.

`git checkout -b my-cool-feature`
Again, let's break it down
- `git checkout`: It's the git command to go places: branches, back in time, to alternate universes, etc. [Read more about time travel in git](./git-time-travel)
- `-b` : You won't find the branch I'm about to tell you. Create it pls.
- `my-cool-feature`: The name of the branch you want to create. Your _evil_ CTO should have some rules on how to name these.
  
The graph now looks like this, pretty uninteresting
![](/assets/images/branches/Branches4.png)
_If you push your new branch, you would create a new branch called `origin/my-cool-feature` that would also point to the same commit_

Now, let's make some commits in our feature (we will mark them with `F`) and some commits in main so we **diverge**

![](/assets/images/branches/Branches5.png)

_if you are confused by the `HEAD` thingy, you should check the [How to Time Travel in Git](./git-time-travel) but for now, consider it the "you are here" of git_

Now, we see that not only we have _diverged_ with our `my-cool-feature` from `origin/main` but `main` is _outdated_!  
The easy step first, with `git checkout main` (we no longer need the `-b` because `main` already exists) we move `HEAD` to `main` and we can `git pull` and we have main up to date.

Now, since we are still not illuminated monkeys, we can just `git merge my-cool-feature`.  
Let's break it down:

- We are in `main`
- `git merge`: "grab a branch and slam it against the branch I am in until something happens"
- `my-cool-feature`: The name of the branch you will slam against the one you are in

This will look like this, and `M` is a **Merge Commit**

![](/assets/images/branches/Branches6.png)

At this point, you are "1 commit ahead" and can `git push` to send your changes.

But look at that commit... That **Merge Commit**... What makes it a Merge? The name? The contents? You probably didn't even make it, it... just kinda happened :|

**A Merge Commit is a commit with TWO previous commits**

You see now that it has two arrows pointing back? Your commit is now a quantum collapsed multiverse child. An absolute point in time in which two timelines crashed together, and somehow a single line continues forward.

(In this case, we only merged two timelines, but there is something called an "octopus merge" that can merge many timelines in a single commit.)

To sum up, to fix the timeline we had to create one of these "Merge" commits thingies... but we had no choice... right?

## Rebase without rewriting the past

Usually, when people think of rebase they think of destroying the history, rewriting it 1984 style and then `git push --force` to erase all proof that stuff ever existed...

And while you _can_ use rebase for that... you can also format your hard drive, cut your internet wire, put a fork in a toaster... but you don't _have to_. (please, don't put forks in the toaster)

### What does rebase do?
It basically means "Change from where in time this branch split off"
Let's go back to this example:

![](/assets/images/branches/Branches5.png)

`HEAD` is in `my-cool-feature` and we can see that we branched the timeline at `C2`.  
`C3` happened sometime after we did our `F1`, but since the `origin/main` timeline doesn't know our `F1` happened, it really doesn't care if it happened after `C2` or `C3`... so what if we could do this

`git rebase origin/main`

![](/assets/images/branches/Branches7.png)
(Remember, our head is on `my-cool-feature`)

Let's break it down:
- `HEAD` is on `my-cool-feature`
- `git rebase`: Let's change where we split off from the timeline.
- `origin/master`: This is the point where I want to split off from the timeline.

Now, hopefully you can see it, but we are actually "1 commit ahead" of `origin/main`. This means we can merge without a pesky **Merge Commit**

By now, I hope you can follow these commands:
- `git checkout main`: Move to `main`... but `main` is outdated
- `git pull`: Sync `main` to `origin/main`
- `git merge my-cool-feature`: But... wouldn't this create the dreaded **Merge Commit**?

In this case, Git is smart enough to know that the time split happens from now on and into the future, and performs a **Fast Forward Merge** that ends up looking like this

![](/assets/images/branches/Branches8.png)

Isn't this pretty? We didn't lose any commits, we didn't rewrite any history, we did not create a totalitarian state, and we have a flat timeline without these quantum anomalies that have many pasts.

## So is rebase "just better"?
Nope. They are just different.  

You probably have used `merge` all your life and didn't even know it, and probably can continue to do so.  

The downsides of `merge` are that they muddy the water on who modified what and where given that these **Merge Commits** have two parents.

**The biggest downside from merge is when working with binary files and LFS locks. If you use LFS Locks you MUST use REBASE**.

## AAAAAAA I did something wrong, and I am stuck in a rebase and I don't know how to fix it, and I am scared!!!11!1one

`git rebase --abort`

This will stop the rebase and undo everything to exactly like it was when you ran `git rebase`

## But rebase _can_ rewrite history, right? 
Yes, it can. The topics are "squashing" and "interactive rebase". I won't cover those here.  
Unless you know exactly what you are doing or your _evil_ CTO asked you to do them, I advice to not squash.

---

Thanks for reaching this far into the article!  
All my drawings are excallidraw embeded so you can download the png and edit it directly or if you want the full editable file you can find it [here](/assets/images/branches/Branches.excalidraw)

See you in the next one!