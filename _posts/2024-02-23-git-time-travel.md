---
layout: post
title: "How to Time Travel in Git"
sub_title: "Git is a time machine, no need to bring your own!"
excerpt: "In this brief article, we will go over the two main ways to time travel in Git.<br>I will make parallelism to TV, Books, Movies, Comics and pretty much any kind of media that has time travel to explain this.<br>Join us as we create paradoxes, become our own grandpas, and save Barry's Mom one more time!"
categories:
  - Unreal Engine
  - Git
tags:
  - unreal engine
  - git
  - game development
---
# How to Time Travel in Git

In this brief article, we will go over the two main ways to time travel in Git.  
I will make parallelism to TV, Books, Movies, Comics and pretty much any kind of media that has time travel to explain this.  
Join us as we create paradoxes, become our own grandpas, and save Barry's Mom one more time!

**This is _probably_ the first in my "git gud or git rekt" series**

## Before we start, let's get HEAD
_in Git! I MEANT IN GIT!!! Real mature guys..._

In Git, **HEAD** is a _thingy_ that points to the current checked out commit and branch.  
So we can see it as a "You are here" arrow through time and branches.

![](/assets/images/timetravel/TimeTravel1.png)

Just in case: remember that commits point to the **previous** commit in history. In this graph the **past is at the left** and the **future is at the right**.

### Small detour: How to find the point you want to go to
This wasn't part of the initial article, but I realized that you might have no idea what a **Commit Hash** (Sometimes called **Commit SHA**) even is.

Try running the "A DOG" command: `git log --all --decorate --oneline --graph` or seeing it in your visual git client. (I recommend you [Fork](https://git-fork.com/) that has an endless free trial WinRAR style)

It will look like one of these two:

![](/assets/images/timetravel/TimeTravel0.png)

Those funny numbers and letters like `4da2e22` and `9ee8352` are your **Commit Hash**. The unique identifier of that particular commit and how git expects you refer to them.

## Now, let's talk about space-time continuum

Time travel in media either by a machine, a magical phone booth or just running really fast usually has its own rules on what you can and can't do in the past. If you want to go off in that tangent, you can always check [TV Tropes](https://tvtropes.org/pmwiki/pmwiki.php/Main/TimeTravel)

For our analogies we will have these:

- **Flashback**: specifically the [Pensieve Flashback](https://tvtropes.org/pmwiki/pmwiki.php/Main/PensieveFlashback "/pmwiki/pmwiki.php/Main/PensieveFlashback") where the present-day version of the character shows up inside their own memories of the past, in order to provide snarky commentary or to inexplicably interact with the past. (Not really time traveling since this is a memory, but we can then create an _Alternate Timeline_ from that, like in ["The Butterfly Effect" Movie](https://tvtropes.org/pmwiki/pmwiki.php/Film/TheButterflyEffect))
   - [**Alternate Timeline**](https://tvtropes.org/pmwiki/pmwiki.php/Main/AlternateTimeline "/pmwiki/pmwiki.php/Main/AlternateTimeline"): The heroes go back to the past and change it such that the universe splits in twain. (We will go back in time and be able to change the past in a different branch)
- [**Set Right What Once Went Wrong**](https://tvtropes.org/pmwiki/pmwiki.php/Main/SetRightWhatOnceWentWrong "/pmwiki/pmwiki.php/Main/SetRightWhatOnceWentWrong"): The heroes go to the past, this time because things went wrong in the past, and they want to change it to make a better "present". (We will rewind time to an earlier point and create a new future, stomping over what once was the present.)
- [**Fish out of Temporal Water**](https://tvtropes.org/pmwiki/pmwiki.php/Main/FishOutOfTemporalWater): A character is placed in a situation completely unfamiliar that results from them being placed in an unfamiliar time period. (We will bring a file from the past right into the present)

Let's start with the flashback!
## Flashback and "Detached HEAD"

In a flashback you can see how things were in the past, but you can't touch anything, it's just a memory after all.

You can achieve this by doing:
`git checkout CommitHashHere`

Let's break this down:
- `git checkout`: checkout is the command to move between commits and branches (it does more than that, but that's enough for now)
- `CommitHashHere`: All commits have a funky string that uniquely identifies them. Put it here

![](/assets/images/timetravel/TimeTravel2.png)
If you are observant, now `HEAD` points to a commit and not to a branch and git told you something spooky like **You are in 'detached HEAD' state.**

This **detached HEAD** is Git way of saying: You can look at this as much as you want, but you can't change it because this is the past.

You can try your code, make small changes, make a build, rescue a file, whatever you want... but you **can't make any new commits** as that would mean you would change the past!

But... what if we make an alternate universe...

## Alternate universes and Branches

We are going to create an alternate universe from this point, to do this we will do a `git checkout -b alternate-main`

Again, let's break it down

- `git checkout` to move to a different place
- `-b` will create a branch because we are moving to a new universe
- `alternate-main` can be any name you want to give to your new branch.

From here you can make a new commit, since your head is now reattached into your new branch.

![](/assets/images/timetravel/TimeTravel3.png)

Now it is up to you how you want to merge back your changes. (An article on how to manage branches is coming...)

But what if I want to remove C2 and C3 from history and rewrite the `main` timeline?
## Go to the past and change the future forever

**We are entering the destructive kind of time travel. You have been warned**

You are going to learn the use of the command `git reset`

This command will rewind time to a specific point in time and forget that the rest of the commits ever existed.

`git reset --hard CommitHashHere`

Let's break it down:

- `git reset`: Reset the state of this repo to a previous state
- `--hard`: I don't want to keep anything between now and the old state
- `CommitHashHere`: To where (when?) I want to travel.

It would look something like this.

![](/assets/images/timetravel/TimeTravel4.png)

Some things you might notice:
- We not only brought the HEAD arrow but the entire branch came back
	- `C2` and `C3` are not actually deleted, but have no way of knowing where they are since arrows only point back (That's why I added the `???`)
- Git might tell you that you now are **Behind** `origin/main` and if you think about it, you are! You are now in the past!
	- If you just wanted to erase stuff from the record* right now, you can `git push --force` and stomp those commits that were already pushed.
	  
> \* Those commits and their files might still exist somewhere, if you are trying to delete something sensitive that you shouldn't have pushed (like a password or private key) you need something like [git obliterate](https://gist.github.com/brianloveswords/7545299)

You are now ready to rewrite history! **Just start making commits!**  
(see how the dotted arrow still exists, but there is no way to reach it)

![](/assets/images/timetravel/TimeTravel5.png)

You now can send your changes but if the changes were already there you will need to use the `--force` flag

`git push --force`

> _I say the whole world must learn of our peaceful ways. By force!_

\- Bender, Futurama

### Do we have less destructive alternatives?

Yes, two.

#### Take the present back with you
If you are curious, you are wondering 

> _What if we don't use the `--hard` flag in the command?_

\- You, hopefully


Well, a _soft_ reset (that is, `git reset CommitHashHere` without the `--hard` part) would leave the files exactly as they are now while rewinding the history. This means Git now says you have a ton of files changed, ready to commit.

This still rewinds time and allows you to rewrite history, but it leaves you everything as it was in the present in case you just want to make a somewhat small change.

#### I just need to undo one commit in particular

If you don't need to rewrite the entire history, but you just need to stop a commit from being born, you can `git revert` it.

This is **not destructive** as it will create an exact inverse of the target commit. Think of it as creating the anti-commit, it's a **new commit** that deletes what you added and recreates what you deleted. Effectively, undoing the target commit without rewriting history but appending to it.

`git revert CommitHashHere`

Let's break it down:
- `git revert`: This is not time travel. This is "Please undo this commit"
- `CommitHashHere`: The commit you want to undo

Optionally, you can add the `--no-commit` option (`git revert --no-commit CommitHashHere`) and it will prepare everything but stop right before committing, allowing you to sneak extra changes or review everything.

## Teleport a file through time to the present

Sometimes you don't know how you messed up a file, but you know when was the last time that file did work. You just need to teleport that single file into the present!

Well, look at this:

`git checkout CommitHashHere -- YourFileHere`

Okay, you know the drill:
- `git checkout`: is for time traveling but usually ends up with a detached `HEAD`...
- `CommitHashHere`: from where in time you want your file
- `-- YourFileHere`: The two dashes, **the space**, and then the filename means that we don't want to rewind the whole repository but that single file. So instead of detaching the `HEAD` this brings the file into the present!

This allows you to bring a file from the past right into the present!

---

Thanks for reaching this far into the article!  
All my drawings are excallidraw embeded so you can download the png and edit it directly or if you want the full editable file you can find it [here](/assets/images/timetravel/TimeTravel.excalidraw)

See you in the next one!