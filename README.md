Git / github workflow
=====================

Good refs:

- [Pro git](http://git-scm.com/book)
- [Visual guide to git](http://marklodato.github.io/visual-git-guide/index-en.html)

There is [Help](https://help.github.com/) available.  In particular you should read through
and follow the steps in these pages:

- [Join GitHub](https://github.com/join)
- [Setting up git](https://help.github.com/articles/set-up-git#platform-linux)
- [Creating an SSH key](https://help.github.com/articles/generating-ssh-keys#platform-linux)
- [Creating a new repository](https://help.github.com/articles/create-a-repo)

When you join GitHub and set up git it is important to use a consistent email address
since this is the ultimately the way you are identified to Git / GitHub when making
commits.

The SSH key section will allow you to push to repositories without having to enter
your password every time.

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
    git push -u origin master

Github now recommends the https protocol but it requires git 1.7.10 (OK for
Ska, not OK for HEAD network).  Just use SSH as this works reliably on HEAD
machines.  Note that the whole `git remote add ...`
business is only needed when creating a new repo from scratch.  If you had
cloned with `git clone git@github.com/taldcroft/project.git` then that wouldn't
be needed.

In the case that your local repo was already cloned from somewhere else
(perhaps a local disk repo) and you now want to push to github, you need
 to change the URL associated with the `origin` remote:

     git remote set-url origin git@github.com:taldcroft/project.git

This introduces two very important concepts, the remote `origin` and the
branch `master`.

    git remote -v
    git branch -a

Cloning an existing project on GitHub
-------------------------------------

If you the project exists on GitHub but not locally, do the following:

    cd ~/git
    git clone git@github.com:taldcroft/example.git
    cd example

Updating an existing project
-----------------------------

If the project exists both on GitHub and locally, but you haven't worked on
it for a bit, you typically want to make sure that the master branch is
synced with what is on GitHub:

    cd ~/git/example
    git checkout master
    git fetch origin  # fetch any new commits / branches on github
    git merge origin/master  # merge the github master into your local (on-disk) master

Development for a simple project with no review
---------------------------------------------------

If it's really just one person doing a project and changes do not require review,
it might be OK to work on the `master` branch.
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

    git push origin master

Here `origin` is the name of the remote and `master` is the name of the local branch you are pushing.

Likewise to pull in any changes on github to your current master:

    git fetch origin

This will indicate if there is new content in any remote origin branch.  If there is you can either
merge it into an existing branch or checkout as a new branch.  For instance:

    git checkout master  # if you are not already on master
    git merge origin/master

    # OR

    git checkout origin/new-feature  # if it doesn't already exist


Development for a project with review required
------------------------------------------------

Any configured project (particularly anything related to flight ops) should
have secondary review of changes.  Using github pull requests makes this
process easy and produces a permanent legacy of the review.

Eng archive example: https://github.com/sot/eng_archive

The fundamental difference in this process is consistent use of branches
to isolate development of a new "feature set".  The goal is to make
each feature branch contain a specific and independent improvement.
That isn't always precisely possible, but most of the time it is.

Starcheck example: https://github.com/sot/starcheck.

Go through the Pro Git chapter on
[branching](http://git-scm.com/book/en/Git-Branching-Basic-Branching-and-Merging)

The following example assumes that `project` already exists as a git repository,
both locally and on GitHub.  It also assumes that you have synced the local
repo to GitHub as shown previously:

    cd ~/git/project
    git checkout master  # make sure you start from master
    git checkout -b new-feature  # create and checkout new branch

At this point you should see a mesage saying that you are on branch `new-feature`.
Now you can start doing your edits and commits:

    touch newfile.py
    git status
    git add newfile.py
    git status
    git diff
    git commit -m "Add newfile.py"

You can always go back to the `master` branch and then back to your new `new-feature`:

    git checkout master
    git checkout new-feature

Make some more edits and finally use `gitk` to get a graphical view of what's happening:

    emacs VERSION
    git status
    git diff
    git commit -am "Update version to 0.2"
    gitk --all

Now send this up to github as a new branch:

    git branch  # make sure you are on the correct branch
    git push origin new-feature  # push current branch to origin

**The all important Pull Request process**

- *Go to github and issue a pull request.*
- Send email to relevant parties requesting review
- If you get some comments that need to be addressed, do the following
  on your local `new-feature` branch:

  - Edit files as needed
  - `git status`
  - `git diff`
  - `git commit -am "Next commit message"`
  - `git push origin new-feature`

- When approval for the final pull request is obtained, then merge using the
  GitHub green merge button.  *Do NOT merge locally on the command line!*

- If needed make a version release tag on Github via the `Releases` button.
  E.g. put in a Version of 0.2 and give a title of `Version 0.2`.  This
  happens *after* merging the pull request, but note that any embedded
  version info should be done *as part* of the pull request.

**Resync your local repo**

    git fetch origin  # fetch all changes since no branch name provided
    git checkout master
    git merge origin/master

**More management**

You can delete a branch:

    git checkout master
    git branch -d new-feature
    git branch -D new-feature

Now pull a new feature branch from github to your local repo:

    git pull origin new-feature:new-feature

If you just do `git pull origin new-feature` it will merge into your current
branch (`master`).


### Fetch vs. Pull

This is a little confusing at first.

**fetch**: fetches all the changes (updates to existing branches or new branches) from the remote but
  *does not touch your local repo*!  This is a good thing because it avoids surprises.

**pull**: fetches the changes (exactly as for fetch) and then automatically merges changes
  into your current branch.

Best practice is to use `fetch` and then see what came down, then examine what was fetched, and
then merge.
Otherwise it's possible to make mistakes.  The remote branches are fetched into branches named
`origin/<remote-branch>`.  You cannot directly checkout or inspect these remote branches.  (Note
that `origin` is the most common remote name, but this is arbitrary and could be anything you want).

To merge changes into your current branch:

    git merge origin/<remote-branch>

If you fetch and there is a new branch that you want to work on, do:

    git branch new-branch origin/new-branch
    git checkout new-branch

Big picture: what is master
----------------------------

The workflow described here may be slightly different than you are used to in one important regard.
In this case the `master` repository generally contains the most recent *development* version
of the code which has been tested and is acceptable for release, but is not necessarily installed yet.

So feature branches get merged to master after test and review, but that is independent of
release and installation.  The tracking of *released* code takes place via git tags.  In this
workflow it is important to tag releases.  Then github automatically makes a link available to
download a tarball of that release.

This workflow is quite common in open source projects.

There are many other workflows possible, and one is to explicitly have a `development` branch that
holds the latest development version.  Then `master` is updated only when code is installed.  In
that way `master` is always the installed release.  In this workflow tagging is not as essential but
there is still the required step of pushing to master at the end.

In practice for us the distinction is not so big because most of the time we install `master` fairly
quickly after a feature branch gets merged because we want the new functionality out on the floor.
So in this way life is simplified and we don't need to deal with an extra long lived branch.

Tips / tricks
--------------

- Read up on .gitignore to learn how to have `git status` ignore certain files that are not versioned.
- Every time you change directory to a git repo after a break, do `git status` to
  see where things stand.  This says what branch you are on and what files are modified.
- Examine `/home/aldcroft/.gitconfig` for some useful shortcuts.

### Try things out on the side first

If you are worried about doing a merge or aren't sure what will happen if you do some operation,
try it on a test copy of a branch first.

    git checkout master
    git branch master-test  # Make a new branch master-test that is the same as master
    git checkout master-test
    # Now try whatever merge or other operation.  Master is safe!
