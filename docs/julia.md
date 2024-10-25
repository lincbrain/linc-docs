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

Simply follow the instructions on [`julialang.org/downloads`](https://julialang.org/downloads/), which 
we replicate here (but do check that they are still up-to-date!)

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

I will redirect you to the tutorial [___From Zero to Julia___](https://techytok.com/from-zero-to-julia/).

## Creating a project

Let's start a new small project. I've created a folder `/PATH/TO/MY/PROJECTS/smallproject` (or even better, I've create a github repo at `https://github.com/<USERNAME>/smallproject` and cloned it at `/PATH/TO/MY/PROJECTS/smallproject`) and opened it in vscode
```
File > Open Folder...
```

### Instantiate the project

We will know initialize this folder as a Julia project. Using well defined projects rather than free standing collections of scripts is advantageous because it allows julia to automatically keep track of our dependencies (_i.e._, the packages we used and their versions).

Let's open a terminal
```
Terminal > New Terminal
```
and launch the Julia REPL
```shell
$ julia
```
```julia
               _
   _       _ _(_)_     |  Documentation: https://docs.julialang.org
  (_)     | (_) (_)    |
   _ _   _| |_  __ _   |  Type "?" for help, "]?" for Pkg help.
  | | | | | | |/ _` |  |
  | | |_| | | | (_| |  |  Version 1.11.1 (2024-10-16)
 _/ |\__'_|_|_|\__'_|  |  Official https://julialang.org/ release
|__/                   |

julia> 
```

We can now press `]` to enter the Pkg REPL mode
```julia
pkg>
```
and add the library [`PkgTemplates.jl`](https://github.com/JuliaCI/PkgTemplates.jl) to our project
```julia
pkg> add PkgTemplates
```
Using the pkg mode avoids having to manually use the `Pkg.jl` library. Otherwise, we'd have to do
```julia
julia> using Pkg; Pkg.add("PkgTemplates")
```
To exist the Pkg REPL mode, type `CTRL` `C` (or `^` `C` on mac).

We can now use `PkgTemplates` and instantiate our project
```julia
using PkgTemplates
tpl = Template(dir=".", plugins=[Git(; manifest=true, ssh=true)])
tpl("MyProject.jl")
```
The `Template()` constructor assumes the existence of some preexisting 
Git configuration (show configuration using git config --list and set with git config --global):

- `user.name`: Your real name, e.g. John Smith.
- `user.email`: Your email address, eg. john.smith@acme.corp.
- `github.user`: Your GitHub username: e.g. john-smith.

There are other options available, described in the [user guide](https://juliaci.github.io/PkgTemplates.jl/stable/user/)

> [!WARNING]
>
> On Julia 1.11, when using a user-defined `JULIA_DEPOT_PATH` which is our case),
> the call to `using PkgTemplates` throws the following error:
> ```julia
> julia> using PkgTemplates
> ERROR: InitError: MethodError: no method matching repl_init(::REPL.LineEditREPL)
> The function `repl_init` exists, but no method is defined for this combination of argument types.
> ```
> This is [a known bug](https://github.com/JuliaLang/julia/issues/56216) reported in Oct 2024.
> A second call to
> ```julia
> julia> using PkgTemplates
> ```
> is succesful.
> 
> A reported alternative solution is to insert
> ```julia
> using Pkg
> ```
> in the file `${JULIA_DEPOT_PATH}/config/startup.jl`

At this point, the following directory and file structure has been created
```
.
└── MyProject.jl
    ├── LICENSE
    ├── Manifest.toml
    ├── Project.toml
    ├── README.md
    ├── src
    │   └── MyProject.jl
    └── test
        └── runtests.jl
```

### Activate the environment

At the bottom of the vscode window, you will see a little box
```
Julia env: <ENV NAME>
```
Click on this box and navigate to the location of the newly created project (_i.e._, the directory that contains the files `Manifest.toml` and `Project.toml`) and click `OK`.

### Start coding

If we were to write a ___Package___, our code would go under `MyProject.jl/src` and would be included in the module file `MyProject.jl/src/MyProject.jl`. However, here we will only write a small ___Project___, so we'll create a `MyProject.jl/scripts` folder inside which we'll create a file `MyProject.jl/scripts/myscript.jl`.

