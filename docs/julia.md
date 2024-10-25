# Starting with Julia

## Overview

__Julia__ is a programming language with a strong focus on numerical computing. 
One of its main features is the use of _just-in-time_ (JIT) compilation, which allows it to reach 
performances cose to those of strongly-typed compiled languages like C, while keeping the 
feel of an interpreted language such as python or matlab. 

Julia is ___young___! Its conception started in 2009 with a first beta version released in 
2012 (`0.1`) and first stable version released in 2018 (`1.0`). While the language is 
stable and quite refined, the ecosystem is much less mature than those of more 
established languages (Python was born in 1991, Matlab in 1979, C++ in 1985 and C in 1972). 
This means that you will stumble on packages that are unstable, unmaintained, or both. 
This is particularly the case for neuroimaging applications, where Julia hasn't picked up 
the same momentum as Python.

Julia is particularly appealing for applications that are cannot be massively parallelized 
(think tractography) or where they are but each operation is computationally and mathematicaly 
involved (think Monte Carlo approaches for _e.g._ micostructure modeling). Julia is both 
easy to learn (its syntax is matlab-like by design) and difficult to master (it relies on 
patterns and concepts, such as multiple dispatch and type inference, that are not common). 
While is is not a strongly typed languages, it only reaches C-like performance if types 
can be properly inferred by the JIT engine. If they are not, it essentially falls back 
to a slow interpreted mode.

## Installation

Simply follow the instructions on [`julialang.org/downloads'](https://julialang.org/downloads/), which 
we recpliate here (but do check that they are still up-to-date!)

```shell
curl -fsSL https://install.julialang.org | sh
```

You will be prompted the following options
```
> Proceed with installation
  Customize installation
  Cancel installation
```
On MGH workstations, the home directory has very little space, it is therefore advised to select the
custom installation mode to install JuliaUp in an alternative location (_e.g._, one of your hard drives).
```
  Proceed with installation
> Customize installation ⏎
  Cancel installation
```
You will then be prompted the following questions:
```
? Enter the folder where you want to install Juliaup > /PATH/TO/MY/LOCAL/juliaup ⏎
? Do you want to add the Julia binaries to your PATH by manipulating various shell startup scripts? > ⏎
? Do you want to add channel specific symlinks? > ⏎
? Enter minutes between check for new version at julia startup, use 0 to disable > ⏎
? Enter minutes between check for new version by a background task, use 0 to disable > ⏎
```
Note that I've kept the default value for each question, except for the first one.

You will then be asked to source your `.bashrc` or `.profile` startup file to update your `PATH`
and thereby get access to the `julia` command.
```
. ~/.bashrc
```

However, while the julia installer (`juliaup`) and startup command (`julia`) are installed where we've asked them
to (`/PATH/TO/MY/LOCAL/juliaup`), the actuall julia binaries and packages are installed in `~/.julia` by default.
Again, we may want to avoid using up space on our `/home` partition, so let's move this. First, move the existing
`~/.julia` directry to a different location
```shell
mv ~/.julia /PATH/TO/MY/LOCAL/julia
```
Then, edit your `~/.bashrc` or `~/.profile` or other startup script and enter the following lines
```
export JULIA_DEPOT_PATH=/PATH/TO/MY/LOCAL/julia
export JULIAUP_DEPOT_PATH=/PATH/TO/MY/LOCAL/julia
```
and source your startup script again
```
. ~/.bashrc
```

## VSCode

The recommended IDE for programming in Julia is [VSCode](https://code.visualstudio.com) with the `Julia` extension. 

We'll assume that you have installed VSCode. To install the extension, select the extension pannel 
(or type `⌘` `⇧` `P` or `CTRL` `⇧` `P`, then `>Extensions: Install Extensions`) and search for the `Julia` extension.

## REPL

## Creating a project

Let's start a new small project. I've created a folder `/PATH/TO/MY/PROJECTS/smallproject` (or even better, I've create a github repo at `https://github.com/<USERNAME>/smallproject` and cloned it at `/PATH/TO/MY/PROJECTS/smallproject`) and opened it in vscode (`File > Open Folder...`). We will know initialize this folder as a Julia project. Using well defined projects rather than free standing collections of scripts is advantageous because it allows julia to automatically keep track of our dependencies (_i.e._, the packages we used and their versions).
