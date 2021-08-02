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

