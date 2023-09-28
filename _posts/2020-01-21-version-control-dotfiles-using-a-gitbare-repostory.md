---
title: Version control dotfiles using a git bare repository
categories: [Git, Guide]
tags: [howto, git, dotfiles]
---

I keep all of my dotfiles versioned controlled using a git bare repository. There are loads of alternate solutions out there helping you to store your dotfiles under some sort of source control.

I have found this method the most useful in terms of keeping your dotfiles up-to-date with source control with minimal number of steps to be performed.

<!--more-->

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

### Setup

```sh
# Create a new git bare repository to track your dotfiles
git init --bare $HOME/.dotfiles.git

# Create an alias `dotfiles` to be used instead of `git` for working with the repository
# Add this alias to your shell configuration file
echo 'alias dotfiles="/usr/bin/git --git-dir=$HOME/.dotfiles.git/ --work-tree=$HOME"' >> $HOME/.bashrc

# reload the shell settings
source $HOME/.bashrc

# Prevent untracked files from showing up when we call dotfiles status
dotfiles config --local status.showUntrackedFiles no
```

That's it. You're now ready to use your dotfiles repository for managing your configuration files.

### Usage

For interacting with your repository, use `dotfiles` instead of using the normal `git` command from anywhere.

Following is an example of adding your `.vimrc` configuration file to your dotfiles repository:

```sh
dotfiles status
dotfiles add .vimrc
dotfiles commit -m "Add vimrc"
dotfiles remote add origin git@github.com:username/dotfiles.git
dotfiles push origin master
```

## Installing dotfiles onto a new system

Now that we have a dotfiles repository, lets take a look at how we can install this onto a new system.

1. Clone dotfiles into a bare repository in a ".dotfiles" folder of your $HOME directory

```sh
git clone --bare git@github.com:username/dotfiles.git $HOME/.dotfiles
```

2. Create an alias `dotfiles` to be used instead of `git` for working with the repository

```sh
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
```

3. Checkout the content from the bare repository to your $HOME

```sh
dotfiles checkout
```

4. Finally, set the flag `showUntrackedFiles` to no on this specific (local) repository
```
dotfiles config --local status.showUntrackedFiles no
```

You should now have the dotfiles installed onto a new system.

Hopefully, this has helped you understanding how to easily manage and source control your configuration files.

Let me know about your own dotfiles in the comments below!
