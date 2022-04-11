+++
title = "How to manage your dotfiles"
date = 2022-04-11
description = "How to manage your configs and dotfiles using git bare. It allows to backup dotfiles into a git repository, allowing to track changes and revert/reset if needed."
in_search_index = true
[taxonomies]
tags = ["linux", "git", "short"]
+++
## Intro
Dotfiles - files that start with a dot `.`. They are considered hidden by the __unix__ operating systems and usually are configuration files or directories. While there are many ways of managing dotfiles (_symlinking_, _copying_, _programs_), one of the most versatile method is by using a _git bare_ repository.
## Setup
Using your choice of a terminal, create a directory for storing git files. The directories name is not important, but I recommend choosing something like `dotfiles` or `.dotfiles`. A folder can be made by using the `mkdir` command:
```
mkdir $HOME/.dotfiles
```
After creating the directory, using git, initialize a bare repository:
```
git init --bare $HOME/.dotfiles
```
To use this git repository, we would need to call git with extra flags:
```
/usr/bin/git --git-dir=$HOME/dotfiles/ --work-tree=$HOME
```
Typing this in every time would be inconvenient. To solve this, let's add an alias to our `.bashrc`, be ether calling this command:
```
echo "alias config='/usr/bin/git --git-dir=$HOME/dotfiles/ --work-tree=$HOME'" >> $HOME/.bashrc
```
or just copying it in manually by opening `.bashrc` and pasting in:
```
alias dotfiles='/usr/bin/git --git-dir=$HOME/dotfiles/ --work-tree=$HOME'
```
Then we want to make sure that git doesn't show us files that we are not interested in when checking this repositories status by calling:
```
dotfiles config --local status.showUntrackedFiles no
```
And we are done! Our git bare repository is configured to track our configs.

## Using
To add a file to the repository, simply call:
```
dotfiles add /path/to/file
```
To commit changes, use this:
```
dotfiles commit -m "What was changed?"
```
Our setup git bare repository is like any other repository. You can even add a remote origin and push it to there. Simply instead of _git_ use _dotfiles_ in your commands.
