
NAME
====

git-reparent - Recommit HEAD with a new set of parents.


SYNOPSIS
========

``git reparent [OPTIONS] ((-p <parent>)... | --no-parent)``


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

-h, --help                show the help
-e, --edit                edit the commit message in an editor first
-m, --message <message>   use the given message instead of that of HEAD
-p, --parent <commit>     new parent to use; may be given multiple times
--no-parent               create a parentless commit
-q, --quiet               be quiet; only report errors
--no-reset                print the new object id instead of updating HEAD


INSTALLATION
============

Make executable and place somewhere in your $PATH.


EXAMPLES
========

Reparenting the tip of a branch
-------------------------------

Suppose we create some commit *B* and then accidentally pass the ``--amend``
flag when creating new commit *C*, resulting in the following history::

                B
               /
        ...---A---C (HEAD)

What we really wanted was one linear history, ``...---A--B--C``.  If we
were to use ``git rebase`` or ``git cherry-pick`` to reconstruct the history,
this would try to apply the *diff* of *A..C* onto *B*, which might fail.
Instead, what we really want to do is use the exact message and tree from *C*
but with parent *B* instead of *A*.  To do this, we run ::

        $ git reparent -p B

where the name *B* can be found by running ``git reflog`` and looking for the
first "commit (amend)".  The resulting history is now just what we wanted::

                C
               /
        ...---A---B---C' (HEAD)


Reparenting an inner commit
---------------------------

We can also update the parents of a commit other than the most recent.
Suppose that we want to perform a rebase-like operation, moving *master* onto
*origin/master*, but we want to completely ignore any changes made in the
remote branch.  That is, our history currently looks like this::

                B---C (master, HEAD)
               /
        ...---A---D---E (origin/master)

and we want to make it look like this::

                B---C   (origin/master)
               /       /
        ...---A---D---E---B'---C' (master, HEAD)

We can accomplish this by using ``git rebase --interactive`` along with ``git
reparent``::

        $ git rebase -i A
        # select the "edit" command for commit B
        # git rebase will dump us out at commit B
        $ git reparent -p origin/master
        $ git rebase --continue

Now the history will look as desired, and the trees, commit messages, and
authors of *B'* and *C'* will be identical to those of *B* and *C*,
respectively.


SEE ALSO
========

git-filter-branch(1) combined with either git grafts or git-replace(1) can be
used to achieve the same effect

git-rebase(1) can be used to re-apply the *diffs* of the current branch to
another


AUTHOR
======

Mark Lodato <lodatom@gmail.com>
