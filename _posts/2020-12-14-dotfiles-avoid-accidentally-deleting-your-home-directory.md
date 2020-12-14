---
layout: post
title: ".dotfiles: avoid accidentally deleting your home directory"
permalink: /2020/12/14/dotfiles-avoid-accidentally-deleting-your-home-directory/
tags:
  - git
  - dotfiles
update: 2020-12-14
---

> **TL;DR** If you're using git to track your dotfiles, make sure to separate
> the git directory using worktrees. Being able to call git directly in your
> home directory can have unexpected and potentially destructive consequences.
>
> git responsibly.

A few years ago, I've started tracking my configuration files in my home
directory with git. I was happy with my solution, since it was simple and
convenient. By doing so, I've also unwittingly planted a nasty landmine to step
onto later: an equivalent of `rm -rf ~/`. Today, I've became the victim of muscle
memory and my lousy dotfiles setup. I'll explain how it happened and how you
can avoid it.

## The Lousy Dotfiles Setup

My dotfiles setup was very basic. I treated `~/` as any other git repository. When on a
new machine, I'd initialize it as a git repo, add my remote, fetch and checkout
and I'd be done. I used the same setup as described
[here](https://drewdevault.com/2019/12/30/dotfiles.html).

This is what my file structure looked like and if yours looks the same, you
should definitely read on to save yourself some trouble!

```
/home/tkrizek
├── .bashrc
├── .config
├── .git        <-- DON'T track your dotfiles like this!
└── .vim
```

## The Landmine

I regularly deal with many different projects tracked in git. There are
countless way the git directory can get polluted by different build systems,
caches, runtime environments and whatnot. Some of it you'll see in `git
status`, some of it you won't, because it's excluded in `.gitignore`.

These various artifacts can sometime cause unexpected failures or behaviour.
That's why I've learned to love a very powerful command: `git clean -dfx`. It
recursively removes any untracked files, including those that are ignored with
`.gitignore` to truly extinguish all those pesky files that might have sneaked
in e.g. during a build.

I find the command so ubiquitously useful, I probably use it at least once a
day. It's encoded in my muscle memory as the go-to command when something seems
off or I just want a fresh state of the repository. I typically never think
twice about using it, since it only acts in the scope of the git repository.
Unlike the `rm -rf` command, which I tend to double or triple check to avoid
disastrous typos.

## The Explosion

It was one of these days when I juggle three different tasks I want to work on,
but all of a sudden, I just need to make a quick commit in the repository. I
knew I didn't have any ongoing work in that project and I just wanted to make
sure the repository is in a pristine state and doesn't contain any left-over
files. My muscle memory fired up and I issued the fatal `git clean -dfx`
command. The last thing I can remember is being annoyed that it's taking more
time than usual...

```console
/home/tkrizek           <-- my *actual* working directory
├── .git
└── projects
    └── knot-resolver   <-- what I *think* is my working directory
        └── .git

$ git clean -dfx  # don't try this at $HOME
```

By the time the command started logging, it only took me a split-second to
suddenly understand why the command is taking such a long time. The command
started to recursively delete all files in my home directory that weren't
tracked in my dotfiles repository, which is, you know, ALMOST EVERYTHING!

Even though I've immediately canceled the execution, the damage was already
done. I've lost my private keys, certificates and other important
files. It also removed some seemingly inconsequential files like
`~/.Xauthority`, which suddenly rendered my system unable to launch new
windows in my current X session. Luckily, I've had a few terminals open, so I
still had a semi-working system I could try to fix.

## The Aftermath

Disaster recovery is one of the few events that push me to "test" my backups. I
use [duplicity](http://duplicity.nongnu.org/) and [systemd
timers](https://wiki.archlinux.org/index.php/Systemd/Timers) for a fully
automated backup with no user interaction. The downside is that sometimes
weeks pass by before I notice it's broken for some reason. The upside is that I
usually get regular backups I don't have to pay any attention to.

This time, my backup just a few hours old. Great! Now, I just need my private
keys and certificates to connect to the VPN and then SSH to the server that
stores my backups... I suppose this is the reason you should really test your
backups in a "clean" environment before you really need them. I had these
credentials backed up locally on a different medium. After a bit of fiddling
with duplicity, I was able to recover everything.

In hindsight, I could've expected something like this to happen eventually. It
wasn't the first time I was accidentally issuing git commands in `$HOME`. There
were at least a few times when I was surprised why the project repository seems
totally alien, until I finally looked into `git log` and realized I'm
in my dotfiles repository. Most of them were benign like `git status` or `git
commit` and I've never realized the dangers until today.

## The Solution: Git Worktree

Git has a feature called
[worktrees](https://schustudios.com/blog/git-worktree). It allows you to check
out the code in a separate directory. They seemed useful, but I just never
found a place for them in my development workflow. However, my colleague
suggested to use them as a solution to this problem. Specifically, you can
separate the checked-out state, such the dotfiles (which can remain directly in
`$HOME/`) and the `.git` directory which contains the entire repository and
place it somewhere else.

When running the `git` command, it looks for `$PWD/.git` and if it doesn't find
it, it'll refuse to operate. Let's suppose I'd store the bare git repository in
`$HOME/.config/conf.git` instead of `$HOME/.git`.  The `git` command wouldn't
be usable in `$HOME` and nothing would have happened. Instead of deleting my
precious files, I would've ended up with this error message:

```console
$ git clean -dfx
fatal: not a git repository (or any parent up to mount point /)
Stopping at filesystem boundary (GIT_DISCOVERY_ACROSS_FILESYSTEM not set).
```

When you want to actually interact with the dotfiles repository, you instruct
git which repository and worktree you're using. This can be done with
`--git-dir` and `--work-tree` options. Instead of calling `git` in `$HOME`, you
have to use `git --git-dir $HOME/.config/conf.git --work-tree $HOME` instead.
Everything else remains the same as you're used to. Using this command is a bit
unwieldy, but a simple alias can fix that.

## The Proper Dotfiles Setup

The goal is make the regular `git` command benign in your home directory and
use a special command to interact with your dotfiles instead. In order to do
that, you must move the bare git repository from `$HOME/.git` somewhere else,
e.g. `$HOME/.config/conf.git`:

```
/home/tkrizek       <-- worktree for ~/.config/conf.git
├── .bashrc
├── .config
│   └── conf.git    <-- bare git repository with dotfiles
└── .vim
```

If you already have the lousy dotfiles setup, you can simply move the
directory. If you're starting out from scratch, you can clone your dotfiles
repository with `--bare` option. (If you have any issues, you may need to use a
temporary directory [like this](https://news.ycombinator.com/item?id=11079145)
instead.)

```console
# existing installations
$ mv $HOME/.git $HOME/.config/conf.git

# setup on new machines
$ git clone --bare https://example.com/user/dotfiles "$HOME/.config/conf.git"
$ git --git-dir "$HOME/.config/conf.git" --work-tree "$HOME" checkout master
```

You double-check that you're *not able* to sucessfully use the simple `git`
command from `$HOME`. Afterwards, you may want to create an alias you can use
for interacting with the dotfiles repository (unless you enjoy typing really
long and tedious commands).  I've put
[this](https://github.com/tomaskrizek/dotfiles/commit/2949613716363ae342ccde289128590d25b02768)
in my `~/.bashrc`:

```bash
# ~/.bashrc
alias confgit="git --git-dir $HOME/.config/conf.git --work-tree $HOME"
```

Then you can work with your dotfiles as usual and simply use a dedicated
`confgit` command:

```console
$ confgit status
```

That's it! Now you can spend your time on something better than recovering
files from backup (you have backups, right?) when you inevitably make a
mistake.

### What about submodules?

I use submodules to pull in different repositories, that contain e.g. vim
plugins. The good news is that git supports that. I've had to remove the
existing modules from the worktree, re-initialize them and then they worked as
expected.

```console
$ rm path/to/module
$ confgit submodule update --init
```

The bad news is that in order to use worktrees along with submodules, you need
sufficiently modern git. The feature was introduced sometime in 2020.  git
2.29.2 works fine. [I use Arch
btw](https://justin.duch.me/post/i_use_arch_btw/).
