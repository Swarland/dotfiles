# My dotfiles

This repostory helps me keep the dotfile settings I like across different computers. The way I organize and install these is inspired by [this blog post by Alex Pearce](https://alexpearce.me/2016/02/managing-dotfiles-with-stow/) about using git and GNU stow to easily version track and install dotfiles. This repo essentially acts as a stow directory, making it easy to create symlinks in `$HOME` that point to the dotfiles in this repositroy (assuming this repo is in your `$HOME` directory).

I also picked up some tips from [this post](https://www.anishathalye.com/2014/08/03/managing-your-dotfiles/) about dotfile organization and tips.

## Contents

```
.
├── bash
├── bin
│   └── bin
├── git
├── install
├── local_dotfiles_MyMacbookAir
├── ohmyzsh
├── other
├── sh
├── tmux
├── vim
└── zsh

12 directories
```

- `bash` : my bash specific settings (eg: my `.bashrc` and `.bash_profile`)
- `bin/bin` : a couple custom scripts which are generally useful to me. The nested structure is so that stow will create a symlink in `$HOME` to the child `bin/bin` instead of `bin`
- `git` : git settings
- `local_dotfiles_*` : settings with computer-specific dotfiles that will be sourced in the general dotfiles. See installation step3.
- `ohmyzsh` : git-submodule of [oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh). This includes some zsh themes and plugins sourced in my `.zshrc`. So there is no need to install oh-my-zsh independently; it is included in these dotfiles.
- `other` : other random dotfiles
- `sh` : shell settings that I like in both zsh and bash. Both my `.zprofile` and `.bash_profile` source `sh/.profile`
- `tmux` : tmux settings. Note the `tmux/.tmux/*` files for version specific tmux settings
- `vim` : My vim settings. A git submodule of [my .vim repo](https://github.com/bfairkun/.vim) which contains its own README to help with installation of plugins
- `zsh` : zsh specific settings. In addition to `.zshrc`, this also includes non oh-my-zsh plugins as a git submodule

## Installation

#### Step1: clone this repo

First, clone this repo to your home directory, using git commands that will also clone the git submodules nested in this repo:

```
cd ~
git clone --recurse-submodules --remote-submodules
```

#### Step2: Putting desired dotfiles (or symlinks to dotfiles) in `$HOME``

Now you can just copy or create symlinks from the relevant dotfile to your home directory:

```bash
cd ~/dotfiles

# Create a single symlink
ln -s bash/.bashrc ~/.bashrc
``````

To do this for all my dotfiles easily, you can use [GNU stow](https://www.gnu.org/software/stow/). But I have found this to be a pain to install where I don't have root privelages because recent versions require recent PERL version which itself was difficult to install without root privelages. An easier to install alternative for systems without root privelages and without up-to-date PERL versions (RCC Midway) is [Xstow](http://xstow.sourceforge.net).


```bash
# Use stow to create symlinks for everything in bash
# Use the dry-run -n flag first.
# Remove -n when you feel comfortable you won't mess anything up
stow -v -n bash

# Or use xstow
xstow -v -n bash
```

Stow can also create create symlinks for all the files in multuple folders, like this example that uses a glob pattern with stow. The `local_dotfiles_*` should obviously only be for the desired computer.

```bash
#Use extended globbing to ignore the local dotfiles and README
setopt extended_glob
stow -v -n ^(local|README)*
```

Stow will not overwrite existing files in `$HOME`. So if you want stow to creat a `$HOME/.vim` symlink to `$HOME/dotfiles/vim/.vim` but you already have a `$HOME/.vim` folder, you need to get rid of it (or move it somewhere else) or else stow will not write a symlink and it will complain of conflicts. I have created a helper script `bin/bin/MoveStowConflicts.cli.py` to systematically put all of the stow conflicts in `$HOME` into a new folder, which you might want to call `MyOldDotfilesThatCreateConflicts/`:

```bash
bin/bin/MoveStowConflicts.cli.py MoveStowConflictFilesToDir -n -v --NewDir ~/MyOldDotfilesThatCreateConflicts/ --SubtreeDirs *
```

The script contains a dry-run (`-n`) parameter so you can preview what will happen before actually doing it. Use the `-h` help flag on that script for more.

Once you have gotten rid of your stow conflicts, you can consider revisiting the stow commands to create symlinks for all the dotfiles

#### Step3: Add local_dotfiles to override my general settings as needed.

When I need computer specific settings, just create an additional local dotfile that gets sourced in main dotfile. For example, a snippet at the end of my `.bashrc`:

```bash
if [ -f ~/.bashrc_local ]; then
    source ~/.bashrc_local
fi
```

Then you can create a `~/.bashrc_local` to contain computer specific settings. I have included the local dotfiles I use on a few computers in `local_dotfiles_*` which can be stowed like this example:

```bash
cd ~/dotfiles
stow -n -v local_dotfiles_MidwayRCC
```

#### Step4: All done!

But note that my vim settings may need some plugins installed, which you can do with `:PlugInstall` once you open vim. The readme in my [.vim repo](https://github.com/bfairkun/.vim) has a little more instructions if needed. Also, as I change things in my .vim repo (a nested submodule in this repo), pulling in changes can be kind of tricky if you don't know about git submodules. I read [this primer on git submodules](https://www.vogella.com/tutorials/GitSubmodules/article.html)
