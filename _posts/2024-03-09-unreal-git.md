---
layout: post
title: "Git for Unreal Engine Projects"
sub_title: "I've heard git is free!"
excerpt: "This is my collection of thoughts and research about how to organize and carry an Unreal Engine Project with Git as your source control solution. I will try to keep this entertaining and give you the theory needed to understand how to survive."
categories:
  - Unreal Engine
  - Git
tags:
  - unreal engine
  - git
  - game development
---
# Git for Unreal Engine Projects
This is my collection of thoughts and research about how to organize and carry an Unreal Engine Project with Git as your source control solution. I will try to keep this entertaining and give you the theory needed to understand how to survive.

**This is _(kinda?)_ part of my "git gud or git rekt" series**

## TL;DR
If you ain't got time for reading this entire thing
- Always `rebase`, never `merge`
- You use Git-LFS in one of these two cases
	- You want locking support
	- Your Git provider demands you stop pushing heavy files.
- Get the [Unreal Engine Git plugin from Project Borealis](https://github.com/ProjectBorealis/UEGitPlugin) instead of the included one.
- If you have C++ code, you need a way to deliver binaries to your artists
	- or you force all your artists to install Visual Studio
- ~~Stop using GitHub Desktop~~

## Git Basics
You might look at this and say, "I know everything there is to know, I have my GitHub profile set up, GitHub desktop installed, and I even drew a little Pac-Man with the contributions in my GitHub profile. There is nothing you can teach me about it"...  
_Oh boy, please stay._

### Git is not GitHub
You might think they are the same, you might even say one in place of the other, but the first step in this journey will be to understand that **Git** is a software in charge of keeping track of what changed in your project in a shape that is called a Repository.  
**GitHub,** on the other hand, is just a _service_ that hosts a copy of your Git Repository and you treat this as the "source of truth" for your project.
However, GitHub is not the _only_ provider of such _service_. Popular names are GitLab and Bitbucket.  

#### Git is not GitHub is not GitHub Desktop
Worse than mixing Git and GitHub is thinking that **GitHub Desktop** is _the_ Git.  
_GitHub Desktop_ is what is called a _Git Client._  
While it is true that you _can_ use GitHub Desktop with pretty much any Git Repository hosting providers... maybe you shouldn't.  
I think I get the idea behind GitHub Desktop: It tries to lower the barrier of entry as much as possible, however, it ends up hiding information, and not letting you learn what is truly going on with your project.  
Please consider switching to something that gives you more control and information, I recommend [Fork](https://git-fork.com/). It has an endless free trial and if you want to support the creators it's a one time purchase.

## Unreal Engine is _very_ binary
If you already created your Unreal Engine project, imported a few meshes and textures, maybe added some blueprint logic here and there you might have realized that everything you import or make ends up in a `.uasset` file, well _almost everything_ is an UAsset and UAssets are **binary**.  
What does that mean? Well, you see, Git is excellent at understanding text files and seeing how they changed from version to version and, more importantly, understanding how to blend two different versions into one when your coworker didn't listen to you when you told them that you were modifying that code file, and they went in and edited it anyway.  
With binary, Git can no longer understand how to blend (or _merge_) those two clashing (or _conflicting_) versions that you and your coworker made: You will be forced to pick one over the other and somebody will have to do their stuff again.

### If we only could _lock_ out our messy coworker from touching our files
Well, if you are on a team where your coworkers are a mess, they are in different time zones, or you are working with 100s of people, sometimes communication is not enough and people will start messing with each other files. For this, the Unreal Editor has Check-out support, this means that you can use it to tell our coworkers "hey, I'm editing this file" and Unreal try to prevent accidental edits on that file.  
For a bit of naming:
- "to **check-out** a file" `==` "to **lock** a file"
- "to **check-in** a file" `==` "to **unlock** a file"
_if it helps, think of it as calling "dibs" on the right to modify a file._

However, the _real_ locking of files is not an Unreal Editor thing but a Git thing (The editor only shows the lock state that git has)  
Except Git doesn't have locking support... Git-LFS has locking support.

### Wait... Git LFS?
Remember how I said that Git is excellent at understanding text files? Well, in the past, Git was also _really bad_ at Binary files. Things did get a lot better and now Git can handle large binary files like a champ, however before that happened, Git LFS (Large File Storage) was invented.  
Large File Storage is an elegant hack on top of Git that replaces your large (binary) files with "pointers" to your files (A pointer is a text file that says "hey, the file that was here can be found at XXXX"). However, the hack is smart enough, and you _should_ never see these text pointer files since they get magically replaced with the correct file in your local copy of the project (this is called "smudging").  
But... why? Well, these pointers are what gets really uploaded to your Repository hosting provider and the actual heavy files get uploaded to some other place where they wait for the magical process in which they get downloaded during a "smudging" process in somebody's local copy.  
All of this hiding heavy files into magical _smudgeable_ files and uploading to multiple servers and other things also come with another side benefit: Locking.  
Since you will have to process all your binary files to check the ones that have already been smudged and the ones that need a new pointer and the whole deal, LFS adds the option to prevent people from sending changes to that file.

### Should I always use Git LFS and Locks then?

Nope.

Remember, you can have Git LFS without locking (but you can't have Locking without Git LFS)

Think about this: Do you have any of the following problems?

#### Your Git remote provider is very angry that you keep pushing big files

Most available Git hosting providers (GitHub, GitLab, Azure, Bitbucket?) will not take like you pushing big files without using Git LFS. This is because their infrastructure is not designed for it.  
If you set up your own git server (for example with Gitea) you maybe don't need LFS since you can commit and push your big binary files without any concern!

**You will need Git LFS if your Git hosting demands it**

#### Your team is very big, and it's impossible to communicate who is doing what

When you reach _very_ big team sizes (or you suck at communicating) you will end up with two (or more) people modifying the same file and losing changes due to the binary nature of the files.  
If you keep failing to signal to each other what files are _up for grabs,_ then **LFS File Locking** acts as an authority on who is modifying what file.

**You may want LFS File Locking (or a training on communicating)** 

## `commit`, `pull`, `push`, right? 

**The following section explains how to avoid a problem that will only happen if you are using Git LFS with Locks.**  
_(It's still interesting to read)_

Ok, let's say you went with LFS and locks, you lock your files, your coworker locks their files, everything is amazing, they commit and push.  
You commit, pull, push and... wait... Why can't I push? It says I modified a file that my coworker had locked? But... I am sure I didn't...  
See? My commit is here, I didn't modify any of their files, it's this commit that appeared out of nowhere that says "merge commit branch..."

Well... when you said `git pull` (or you pressed that "pull" button in your Git UI that by this point I hope it's not GitHub Desktop) you actually ran two commands without knowing what they do: `git fetch` and `git merge`.   
`git fetch` is the first command, and it's the one that actually downloads new changes. This doesn't apply those changes to your current project, but it has them ready for when you are ready to apply them.  
`git merge` applies the changes that you downloaded with `fetch` and to do so it creates a dreaded "Merge Commit".

A Merge Commit is a commit with TWO parents. It's a singularity point in which git collapses divergent timelines.

Look at this graph.
![](/assets/images/unrealgit/MergeBad1.png)
(Commits always point to their **Parent**. So they point to the **past**)

So, you and your coworker both started from a shared point in your Repository.
Since your coworker committed and pushed their changes first, they didn't have to get any changes and pushed their changes... Now it's your turn, and you are "parallel" to the current "truth" of the Repository (remember we said that we consider what is uploaded as the "true state").
You have diverged! There are two commits in the upstream branch but one in the local branch!

There are two ways you can join: either you mash (_merge_) the two lanes together, and you end up with a merge commit that reports everything that was modified in both lanes (bad ending) or you add your changes at the end of the other lane (or from another perspective, you put the existing changes below you, _rebase_), without any need for crashing or conflicts (good ending).

This is what a merge looks like
![](/assets/images/unrealgit/MergeBad2.png)

See that _eeeeviiiiiill_ merge commit? That will probably prevent you from pushing and give you an error saying that you modified a file that was locked by somebody else.

Now, look at a rebase
![](/assets/images/unrealgit/MergeBad3.png)
No extra commits, your timeline is not a _big ball of wibbly wobbly, timey wimey stuff_

To achieve this beautiful timeline we need to use the command:  
`git pull --rebase`  
This command tells Git that we want to **rebase** and not **merge**. 

Since you will be doing this a lot, it would be a good idea to set up this repo to always rebase when you pull by running:  
`git config pull.rebase true`  
And if you want to be extra ~~paranoid~~ safe, you can set it globally for all your repos from now on by using the global flag: `git config --global pull.rebase true` 

There is a third thing that can happen: A Fast-Forward. This happens when you don't have any commits to make, and you just need to get the latest changes. In this case, there is no lane to mash, and you just _fast-forward_ to the current state of the Repo.

What did we learn from this? **ALWAYS REBASE**

### AAAAAAAAA I already did a merge now I can't push please heeeelp!

Take a deep breath, you are gonna make it.  
You have two options.

- The safe one: Ask for your coworkers to release the files you can't push. You will be able to push.
- The slightly dangerous one: You can rewind time and undo that ugly merge commit, allowing you to rebase properly.  
I would **strongly** suggest that you first read [my guide on git time traveling](./git-time-travel){:target="_blank"} but the gist of what you need is:
  - `git reset --hard HEAD~` will erase the last commit (the merge one) from existance. (If your last commit is not the merge one, this will destroy it anyway, that's why this is the _slightly dangerous_ method)
  - `git pull --rebase` since you already removed the eeeevil commit, now you can merge. 

## What about C++ Files? Those are code, right? So it's easier... right?
Well, yes and no.  
Are they easier to track with git? Absolutely...  
But C++ files are source code and source code needs to be _compiled_ into binary files...  
Are you sending those binary files? (you probably aren't...) Does everybody in your team have Visual Studio and can build those C++ files? (they probably can't...)

Maybe you are working alone, or you somehow forced your entire team to install Visual Studio and to build the C++ project every time you change something, however most of the time artists and game designers don't want to have those tools, they want to double-click the thingy and the editor opens and to achieve this you need to provide them with the compiled binaries of your C++ code.

There is an easy way: track your Binaries folder into your repo. Make sure you add it to Git LFS and you are good to go.   
- **Pro**: Everything just works automagically. 
- **Con**: Binaries can get heavy

There is a jankier way: Zip your `Binaries` folder every time you change something in C++ and share the zip via Google Drive or something.   
- **Pro**: Uhm... it's quicker than setting up something? 
- **Con**: everything.

There is a sophisticated way: Uploading binaries to servers for storage is a science all in itself, and people smarter than me have already solved it. I have tried a tool called [LongTail](https://github.com/DanEngelbrecht/golongtail) that is meant to do incremental backups of binary files that can work very well, however this requires a storage bucket like Google Cloud, Amazon S3 or some sort of NAS accessible to your entire team.  
- **Pro**: It's _the_ correct answer if you have big projects and a big team
- **Con**: It's definitely not trivial to set up

#### What about my custom engine from source?
If you are working directly with the source code for Unreal Engine, your problem is pretty much the same as the previous section, but now instead of a couple of minutes to build the code it can take a couple of hours. So you will definitely need a way to deliver the "Rocket" build of the engine. (Unreal Engine Editor is called "Rocket". I didn't come up with the name)

## This is a lot, aren't there any tools to make it simpler?

_Well yes, but actually no._

This section is _very_ opinionated. You will need to find the setup that works for **Your** team.

With that being said, this is what I would suggest you look into:

- [**Fork**](https://git-fork.com/) An AMAZING Git client. It's a one time pay if you want to support the creators or if you can't afford it then you can use the **endless free trial**.
- [**Unreal Engine Git Plugin** ](https://github.com/ProjectBorealis/UEGitPlugin): This is waaaay better than the one that comes included in your Unreal installation. It is made by the Project Borealis team, but I think this deserves its own spot in this list.
- **Project Borealis**: Project Borealis is a group of people fixated on making Half Life Episode 3 in Unreal Engine. In the process, they created tools and procedures on how to work in an Unreal Engine Project with **Git LFS and File Locking**
	- [**.gitIgnore**](https://github.com/ProjectBorealis/PBCore/blob/main/.gitignore)
	- [**.gitattributes](https://github.com/ProjectBorealis/PBCore/blob/main/.gitattributes)
	- [**.gitconfig??**](https://github.com/ProjectBorealis/PBCore/blob/main/.gitconfig) If you are savvy enough with git, you will realize that this file is not part of the git ecosystem. This shouldn't work... right?... Well, you can use `git config include.path .gitconfig` to link your local config with this shareable file. The caveat is that every person has to run the command after cloning.
	- [**PBSync**](https://github.com/ProjectBorealis/PBSync) and [**PBCore**](https://github.com/ProjectBorealis/PBCore): PBSync is a series of python scripts that automate the main tasks like pulling, building and solving merge commits. PBCore is some sort of "boilerplate" project to start working with PBSync.
- [**LongTail CLI**](https://github.com/DanEngelbrecht/golongtail): This is an _incremental asset delivery tool_. This means that it's designed to upload and download files efficiently from GCS or S3 bucket (or a local folder). This tool is what Project Borealis ended up using to deliver binaries so their artists and game designers don't need to have Visual Studio and compile the C++ project each time they want to open the project.
- [**Gitea**](https://github.com/go-gitea/gitea): Gitea is a _Git Server_ written in Go. If you want to self-host your git server and stop relying on GitHub or GitLab, then Gitea is what I would recommend.
- [**Azure Git Repositories**](https://azure.microsoft.com/en-us/products/devops/repos): Ok, we all know GitHub and GitLab and Bitbucket and many more, but Azure offers 250GB of free storage as long as you are a team of 5 or fewer people.


---

As always, all my drawings are made using excalidraw and the PNG itself are editable, or you can download my project file [here](/assets/images/unrealgit/MergeBad.excalidraw)

Special thanks to people from Project Borealis that had the patience to explain me their ways.

> _"I heard git is free tho"_