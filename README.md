Git / github workflow
=====================

Good refs:

- [Pro git](http://git-scm.com/book)
- http://gitref.org/index.html

There is [Help](https://help.github.com/) available.

- Setting up an account
- Creating a new repository

Starting a new project
----------------------------

Most of the time you probably have some initial code or files
under development before you decide to start versioning them
and before you have a github repo.

**Create new project or start versioning an existing one:**

    mkdir ~/git/project  # if needed
    cd ~/git/project
    git init
    touch README
    emacs README
    cat "0.1" > VERSION

**Commit:**

    git add README VERSION  # etc
    git add .  # Add everything in the directory and subdirectories
    git commit -m 'first commit'

**Shortcut to commit all changed files:**

    git commit -a -m 'next commit'
    git commit -a  # Brings up an editor

**Push:**

At this point create a corresponding `project` repo on github.
Do *not* have it automatically create a README file, this new
github repo needs to be empty.

    git remote add origin git@github.com:taldcroft/project.git      # ssh
    git remote add origin https://github.com/taldcroft/project.git  # https
    git push -u origin master

Github now recommends the https protocol but it requires git 1.7.10 (OK for
Ska, not OK for HEAD network).  Just use SSH.  The whole `git remote add ...`
business is only needed when creating a new repo from scratch.  If you had
cloned with `git clone git@github.com/taldcroft/project.git` then that wouldn't
be needed.

This introduces two very important concepts, the remote `origin` and the
branch `master`.

    git remote -v
    git branch -a

Cloning an existing project
---------------------------

    git clone git@github.com:taldcroft/example.git

Development for a simple (not released) project
---------------------------------------------------

If it's really just one person doing a project and it is not yet
released or configured, it might be OK to work on the `master` branch.
This single thread development is not good once a project gets
more complex.

    <edit>
    <test etc>
    git status
    git diff
    git commit -a  # brings up editor
    git log

Write a [good commit message](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)!

Periodically update github:

    git push origin  # master is implicit

Likewise to pull in any changes on github to your current master:

    git pull origin

Development for a mature / complex project
--------------------------------------------

Starcheck example: https://github.com/sot/starcheck.

Go through the Pro Git chapter on
[branching](http://git-scm.com/book/en/Git-Branching-Basic-Branching-and-Merging)

    cd ~/git/project
    git branch new-feature  # make new branch
    git checkout new-feature  # checkout new branch
    touch newfile.py
    git status
    git add newfile.py
    git status
    git diff
    git commit -m "Add newfile.py"
    git checkout master
    git checkout new-branch
    emacs VERSION
    git status
    git diff
    git commit -am "Update version to 0.2"
    gitk --all

Now send this up to github as a new branch:

    git branch  # make sure you are on the correct branch
    git push origin new-feature  # push current branch

You can delete a branch:

    git checkout master
    git branch -d new-feature
    git branch -D new-feature

Now pull a new feature branch from github to your local repo:

    git pull origin new-feature:new-feature

If you just do `git pull origin new-feature` it will merge into your current
branch (`master`).

*Now go to github and issue a pull request.*

Last resync your local repo with github and tag the release:

    git fetch origin
    git checkout master
    git merge origin/master
    git tag -a "0.2" -m "Version 0.2"
    git push origin --tags
