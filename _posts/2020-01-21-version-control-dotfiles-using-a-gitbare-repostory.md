---
layout: post
current: post
navigation: True
title:  Version control dotfiles using a git bare repository
date:   2020-01-21 09:00:00 +0000
tags: [guide]
class: post-template
subclass: 'post tag-guide'
author: haroon
---

I keep all of my dotfiles versioned controlled using a git bare repository. There are loads of alternate solutions out there helping you to store your dotfiles under some sort of source control. 

I have found this method the most useful in terms of keeping your dotfiles up-to-date with source control with minimal number of steps to be performed.
    
### Why version-control dotfiles?

* useful when starting up a new job
* easy to setup new development machines
* Can share with other people
* You can keep a list of things that you need installing on a new machine

### What is a git bare repository?

A standard git repository contains 2 working directories `.git/` and `project files` within the project directory itself. On other hand, git `bare` repsitories have no working directories but store the git history files at the root of a project directory.

A standard git repository is created with just running `git init` from inside the project directory and a git bare repository is created with `git init --bare <path/to/project/directory.git>`


## Setting up a dotfiles bare repository

All you need is to make sure is that you have `git` installed on your machine.


