# Quick Git for Bioinformatics

## Introduction

This demo has the parts: 

 1. Create a git repository, both from a directory and from
 cloning. Add and commit files.
 2. Working with remotes.
 3. Branch and merge.

Unless you're working with collaborators, you'll mostly use the first
and third parts.

Think this is just useful for code? Think again! You ~~can~~ should
version control papers, and small datasets (e.g. an RNA-seq counts
table, not a 5GB SAM file).

But first, tell Git who you are with: 

    git config --global user.name "Vince Buffalo"
    git config --global user.email vsbuffalo@ucdavis.edu

## Creating repositories, adding files, and committing

### Creating a repository from a directory, adding files, and committing

    mkdir vince
    cd vince
    echo "This is a test repository." > test.txt
    git init
    git add test.txt # this is called "staging"
    git commit -m "initial import" # this commits all staged files only. # use a commit message
    cd .. # get to where we were

Note something very important about Git: when you add a new file,
you're not just telling Git it should be under version control, but
you're *staging* it in the next commit. When a file already exists,
you've made changes to it, and you want it to be in the next commit
you need to add it with git add, or tell git commit to commit all
files using -a. This allows AWESOME incremental commits. You can also
unstage a file for commit using git reset (that's advanced, we can
talk about that later, but to unstage a file, you do git reset
FILENAME).

### Cloning a repository

If you have an existing repository you want to clone, you use git
clone. This essentially initializes a empty local git repository, and
pulls a clone from the remote repository.

    git clone ./vince joe
    ls
    cd joe
    git log

### Working with remotes

Let's mimic the workflow we use most often: a remote "bare"
repository. The issue with the current workflow is that if Joe commits
changes, and pushes them to my remote, he would be asked if really
wants to push to my working repository. In fact, he probably doesn't:
this is where I am working and I want to be able to pull in his
changes whenever I like, not have them pushed to me.

Consequently, a very effective git workflow is to have a remote "bare"
reposistory. The "bare" means it's not a real working directories,
which we have to explicitly state by default all git repositories are
working directories (i.e. you can edit files). This is in contrast to
CVS and SVN. 

We set up a server that is frequently backed up that has a /git
directory full of our project's bare reposistories. In this example,
we'll create a bare repository named "remote". Remember, Git works
with SSH!

    rm -rf joe # sorry Joe.

    mkdir remote # this is our pretend remote; in real life, put this on a server everyone has access to
    cd remote
    git init --bare # older versions of Git require this to be git --bare init, but use the newest version!
    ls # see all the Git internal stuff? Not a place to do work!
    cd ..

Now, we're going to tell my local repository about this remote one.

    cd vince
    git remote add origin ../remote
    git remote ## or cat .git/config, SVN users, note that there is only ONE, YES ONE directory!

Now, let's push our commits to remote. 
    
    git log # what commit are we at?
    git push origin master

Why origin master? We want to tell git push (push is the subcommand)
where we want to push (origin, which is the name of the remote we
added; "origin" is used by convention) and which branch we're on
(master is the default).


Now, let's pretend we're Joe again. Joe wants to get my work, add
something to it, and check it in.

    cd .. # get to where we were
    git clone remote joe
    cd joe
    ls 
    git log
    echo "new gibberish" >> test.txt
    git commit -a -m "some gibberish added"
    git push origin master

Ok, so Joe added something to test.txt, commited it, and pushed it to
the remote. Now, Vince wants to pull in Joe's changes:

   cd ../vince
   git pull origin master
   git diff HEAD^

This means take the HEAD (basically a pointer in Git's history to
where your current working directory is), go back one commit in
history (the ^), and diff the current working directory with the last
commit.

We can also look at specific files from older commits using git show:

    git show HEAD^:test.txt

## Git branching

So suppose you're writing an analysis in Python, but someone says it
would be 10000x easier in R. You want to keep your data organized in
the existing directories. You think to create a repo_in_R copy of your
directory, but Keith tells you that this could lead to the innocent
drowning of a kitten (see
http://bitsandbites.posterous.com/details-of-next-meeting-12th-january-2012).

What do you do? If you're from the land of SVN and CVS, you're pulling
out hair, flipping through pages of documentation, and practicing
branching on fake repositories. If you're using Git, you just sip some
tea, lean back, and branch. Branches in Git are super easy, because
they're virtual, not physical. 


    git branch # see, we're in master now
    git checkout -b new-branch
    git branch

    echo "more gibberish" >> test.txt
    git commit -a -m "more gibberish added on a dev branch"
    
    git checkout master # go back to master
    echo "different gibberish added. Uh oh." >> test.txt
    git commit -a -m "different gibberish added, clash anyone? "
 
We can look at this situation with:

    gitk --all &
    # or 
    git log --graph --all

Now, let's merge that new-branch into master:

   git branch # make sure you're in the branch you want to merge the other into.
   git merge new-branch

Uh oh, a merge conflict!! It's easy to resolve. Look what's up with
git status, then manually resolve the conflict. When you're down add
the files, and commit. That easy! And normally you won't find merge
conflicts.

    emacs test.txt
    git add test.txt
    git commit -m "merge conflict resolved"
    git push origin master # might as well?

## What's next?

 - Following remote branches
 - git refspecs
 - git stash
 - Github
 - git blame
