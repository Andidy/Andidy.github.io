---
layout: post
title: Engine Moving ady_math improvements to the ady_math repo
---

A common dependency I've been using throughout my C++ projects is what I call ady_math.h.
It is a  linear algebra library that fits the naming and style I find most pleasant to use.
The issue is that for a while ady_math has just been a file floating around in my repos that
I copy into a new project or copy over an old projects implementation.
I want to make it into a library that is easy to include, and update going forward by using
git submodules in conjunction with putting ady_math into its own repo.
This way I can just add the submodule to new projects and update old projects with new changes.

The docs for git submodules could really use an update cause after reading them and a few
tutorials I still couldn't tell how to use them. Eventually, I found one tutorial though that
was more modern and didn't consider very old versions of git from like 2010. It turns out adding
a submodule from an existing repo is quite easy.

First we need to add the submodule:

```
git submodule add <url of submodule repo> <path/to/folder/of/submodule>

git commit -m "Added my submodule"

git push
```

For the place we add the submodule, we're all set, but for cloning the repo later we need to do a
bit more:

```
git clone <repo with submodules in it>

cd path/to/folder/of/submodule

git checkout <branch of submodule you want to use>
```

Now both cases are setup. If we want to get changes from the submodule repo we just do:

```
cd path/to/folder/of/submodule

git pull

cd back/to/main/repo

git add <submodule>

git commit -m "Updated submodule"

git push
```

And if we want to add our own changes to the submodule to the submodule's repo:

```
// you make changes to the submodules files

cd path/to/folder/of/submodule

git add <changes>

git commit -m "<your commit message>"

git push

cd back/to/main/repo

git add <submodule>

git commit -m "Updated submodule"

git push
```

And boom you've pushed your changes to the submodule. You can probably turn these into scripts to automate
them to be just a couple commands instead of 4-5 commands per task.

One note, if you are on windows and you get errors saying something like:

```
fatal: No url found for submodule path 'path/to/submodule'

or

No submodule mapping found in .gitmodules for path 'path/to/submodule'
```

Make sure your .gitmodules file entry for the given submodule looks like this:

```
[submodule "path/to/submodule"]
	path = path/to/submodule
	url = submodule_url
```

And not:

```
[submodule "path\\to\\submodule"]
	path = path\\to\\submodule
	url = submodule_url

// notice the "/" vs "\\"
```

Hopefully this is helpful to you.