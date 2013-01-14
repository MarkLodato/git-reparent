
NAME
====

git-reparent - Recommit HEAD with a new set of parents.


SYNOPSIS
========

``git reparent [--no-reset] [-q] ((-p <parent>)... | --no-parent)``


DESCRIPTION
===========

Create a new commit object that has the same tree and commit message as HEAD
but with a different set of parents.  If ``--no-reset`` is given, the full
object id of this commit is printed and the program exits; otherwise, ``git
reset`` is used to update HEAD and the current branch to this new commit.

This command can be used to manually shape the history of a repository.
Whereas ``git rebase`` moves around *diffs*, ``git reparent`` moves around
*snapshots*.  See EXAMPLES for a reason why you might want to use this
command.


OPTIONS
=======

-h, --help              show the help
-e, --edit              edit the commit message in an editor first
-m, --message <message> use the given message instead of that of HEAD
-p, --parent <commit>   new parent to use; may be given multiple times
--no-parent             create a parentless commit
-q, --quiet             be quiet; only report errors
--no-reset              print the new object id instead of updating HEAD


INSTALLATION
============

Make executable and place somewhere in your $PATH.


EXAMPLES
========

Reparenting the tip of a branch
-------------------------------

Suppose we created a commit on branch *master* on our local repository, and
meanwhile someone else created a commit on *master* and pushed it.  Now, our
local branch and the upstream branch have diverged. ::

                B (master, HEAD)
               /
        ...---A---C---D (origin/master)

We looked at the contents of the upstream branch and realized that we want to
throw all that way and use the state of *B* as-is, but we want to keep *C* and
*D* in the history.  If we were to run ``git rebase origin/master``, this
would try to apply the *diff* of *A..B* onto *D*; in other words, it would
include both the upstream changes on ours.  This is not what we want.
Instead, we want to create a new commit on top of *D* that contains the same
exact *tree* as *B*.  To do this, we run::

        $ git reparent -p origin/master

After running the command, the history will look like this::

                B       (origin/master)
               /       /
        ...---A---C---D---E (master, HEAD)

However, the tree, commit message, and author information of *B* and *E* will
be identical.

Reparenting an inner commit
---------------------------

Now suppose we have the same situation, but we have more than one commit on
our local branch and we want to keep them all::

                B---C (master, HEAD)
               /
        ...---A---D---E (origin/master)

We can do this by using ``git rebase --interactive`` along with ``git
reparent``::

        $ git rebase -i origin/master
        # select the "edit" command for commit B
        # git rebase will dump us out at commit B
        $ git reparent -p origin/master
        $ git rebase --continue

Now the history will look like we want it::

                B---C   (origin/master)
               /       /
        ...---A---D---E---F---G (master, HEAD)

As before, *F* will be identical to *B*, except for the parent and the
committer information, and similiarly for *G* and *C*.


SEE ALSO
========

git-filter-branch(1) combined with either git grafts or git-replace(1) can be
used to achieve the same effect

git-rebase(1) can be used to re-apply the *diffs* of the current branch to
another


AUTHOR
======

Mark Lodato <lodatom@gmail.com>
