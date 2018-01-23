% Git 201
% Michał Zając
% Akademickie Stowarzyszenie Informatyczne

# Agenda

## Git 201

#. Authentication methods
#. Reverting changes
#. Cherry picking
#. Rebasing
#. Branching models
#. Some Git internals

# Authentication methods

## HTTPS

Git actually has two modes, Smart and Dumb HTTPS

## SSH

## Generating keys

`ssh-keygen -t rsa -b 4096 -C "spock@star.trek"`

## Where are my keys?

`$HOME/.ssh/id_rsa` and `$HOME/.ssh/id_rsa.pub`

## Your PRIVATE key is really private

Don't do dumb shit like this:

![Dumb person on the internet sharing his private key](images/private_key.png){.stretch}\ 

## Put your PUBLIC key on GitHub

![How to put your keys on GitHub](images/github_ssh_keys.png){.stretch}\ 

# Reverting changes

## Reverting local changes

You can do whatever you want, `git checkout --`, `git reset --soft/--hard` or even `git rebase -i`.

## Reverting published changes

If you use something else than `git revert` then your fellow programmers will find you and kill you.

## Main problem with `revert`

People don't do atomic changes so reverting stuff is painful.

# Cherry picking

## How?

```
git cherry-pick 09c063
```

# Rebasing

## Integrating changes from other branches

In Git 101 I covered merging. This time around I will talk about rebasing and when you don't want to use it.

## Before merge

![Git graph before merge](images/basic-merge-before.png){.stretch}\ 

## After merge

![Git graph after merge](images/basic-merge-after.png){.stretch}\ 

## Enter rebase

We are basically going to take the patch of the change from C4 and reapply it on top of C3.

```
$ git checkout experiment
$ git rebase master
```

## After rebase

![Git graph after rebase](images/basic-rebase-after.png){.stretch}\ 

## Merging

```
$ git checkout master
$ git merge experiment
```

## After merge

![Git graph after merging a rebased branch](images/basic-rebase-after-merge.png){.stretch}\ 

## Rebase vs. merge

>(...) rebasing makes for a cleaner history. If you examine the log of a rebased branch, it looks like a linear history: it appears that all the work happened in series, even when it originally happened in parallel.

## More convoluted stuff

![Git graph with topic branch branched from a topic branch](images/interesting-rebase-1.png){.stretch}\ 

## Scenario

Let's say we want to apply changes from `client` branch to `master` branch but not all the changes from `server` branch.

## gg ez

`git rebase --onto master server client`

>This basically says, “Take the client branch, figure out the patches since it diverged from the server branch, and replay these patches in the client branch as if it was based directly off the master branch instead.”

## After rebase

![Git graph with topic branch rebased onto master](images/interesting-rebase-2.png){.stretch}\ 

## After merging

![Git graph with topic branch merged into master after rebasing](images/interesting-rebase-3.png){.stretch}\ 

## Done with the server?

```
git rebase master server
```

## After rebase

![Git graph with base topic branch rebased onto master](images/interesting-rebase-4.png){.stretch}\ 

## After merge

![Git graph with base topic merged to master after rebasing](images/interesting-rebase-5.png){.stretch}\ 

## One simple rule

>Do not rebase commits that exist outside your repository.

## Example of stuff breaking down

## Scenario

We clone a repository and do some work on it.

## State

![State after cloning and doing some basic work](images/perils-of-rebasing-1.png){.stretch}\ 

## Scenario

Someone else did some work including a merge and pushed it to the remote server. You pulled the changes and merged them.

## State

![State after merging upstream changes](images/perils-of-rebasing-2.png){.stretch}\-

## Scenario

Now this smartass decided to rebase his work and force pushed it to the server.

## State

![State after merging upstream changes](images/perils-of-rebasing-3.png){.stretch}\-

## Scenario

If you `git pull` this you will get something like that

## State

![State after merging upstream changes](images/perils-of-rebasing-4.png){.stretch}\-

# Branching models

## Local branches only

Not sure why you would want this but why not.

## Topic branches only

Dead simple, might not work in some extreme cases. If you are not sure then go with this.

## Git Flow

Presented by Vincent Driessen in 2010, makes my head hurt.

## Seriously?

![Git Flow image](images/gitflow.png){.stretch}\ 

## Explain this madness!

Sure mate, give me two hours or so.

## OneFlow

Apparently GitFlow is [harmful](http://endoflineblog.com/gitflow-considered-harmful) so Adam Ruka proposed something simpler.

I didn't use that so I won't comment on it.

# Git internals

## .git directory

Tell me your secrets .git directory!

```
ls -F1 .git
branches/
config
description
HEAD
hooks/
info/
objects/
refs/
```

## What's what?

* `config` - file contains your project-specific configuration options
* `HEAD` - points to the branch we are on
* `index` - stores your staging area information
* `hooks` - directory contains your client- or server-side hook scripts
* `info` - keeps a global exclude file for ignored patterns that you don’t want to track in a `.gitignore` file
* `objects` - stores all the content
* `refs` - stores pointers into commit objects

## What are Git objects?

Object is basically anything you put into Git. Once something is stored Git
gives you a unique key that you can use to retrive it. We can use
`git hash-object` to create new data object and manually store it

## Example

```
$ git init test
Initialized empty Git repository in /tmp/test/.git/
$ cd test
$ find .git/objects
.git/objects
.git/objects/info
.git/objects/pack
$ find .git/objects -type f
```

## Creating a new entry

```
$ echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
$ find .git/objects -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```

## Inspecting the file

It's going to be garbage. Use `git cat-file` instead.

## Example

```
$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content
```

## Writing some more

```
$ echo 'version 1' > test.txt
$ git hash-object -w test.txt
83baae61804e65cc73a7201a7252750c76066a30

$ echo 'version 2' > test.txt
$ git hash-object -w test.txt
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a

find .git/objects -type f
.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```

# Finishing up

## Questions?

## Further reading

#. https://danielkummer.github.io/git-flow-cheatsheet/
#. http://endoflineblog.com/oneflow-a-git-branching-model-and-workflow
#. https://barro.github.io/2016/02/a-succesful-git-branching-model-considered-harmful/
#. https://git-scm.com/book/en/v2/Git-Branching-Branching-Workflows
#. https://git-scm.com/book/en/v2/Git-Branching-Branch-Management
#. https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell
#. http://nvie.com/posts/a-successful-git-branching-model/
#. https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain

# Thanks for your attention
