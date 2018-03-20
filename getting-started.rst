.. role:: todo
.. raw:: html

    <style>
      .todo { color: red }
    </style>

.. NOTE::

    :todo:`TODO items look like this.`

Introduction
============

Mercurial Setup
---------------

This post assumes that you already have a functional Mercurial setup configured using ``mach mercurial-setup``.  Full instructions can be found in the `Configuring Your Machine to Use MozReview <http://mozilla-version-control-tools.readthedocs.io/en/latest/mozreview/install.html>`_ and `Configuring Mercurial to Use MozReview <http://mozilla-version-control-tools.readthedocs.io/en/latest/mozreview/install-mercurial.html>`_ documents.  You'll also need an active `bugzilla.mozilla.org (BMO) <https://bugzilla.mozilla.org/>`_ account.

#. `Create an API-Key <https://bugzilla.mozilla.org/userprefs.cgi?tab=apikey>`_ named ``mercurial`` on BMO.
#. Ensure `Your real name <https://bugzilla.mozilla.org/userprefs.cgi?tab=account>`_ on BMO contains your IRC nick prefixed by a colon (eg. my ``real name`` field has ``Byron Jones :glob``).
#. If you have a Mozilla LDAP account, `associate your LDAP account with MozReview <http://mozilla-version-control-tools.readthedocs.io/en/latest/mozreview/install.html#manually-associating-your-ldap-account-with-mozreview>`_.
#. `Install the mercurial extension <http://mozilla-version-control-tools.readthedocs.io/en/latest/mozreview/install-mercurial.html#installing-the-mercurial-extension>`_ by cloning the ``version-control-tools`` repository and adding it to your ``.hgrc`` file.
#. `Add your BMO email address and API key <http://mozilla-version-control-tools.readthedocs.io/en/latest/mozreview/install-mercurial.html#bugzilla-credentials>`_ to the ``[bugzilla]`` section in ``.hgrc``.
#. `Add your IRC nick <http://mozilla-version-control-tools.readthedocs.io/en/latest/mozreview/install-mercurial.html#irc-nickname>`_ to the ``[mozilla]`` section in ``.hgrc``.
#. Install the :ref:`extensions-aliases`. Note: the `Evolve extension <https://www.mercurial-scm.org/wiki/EvolveExtension>`_ is required.

Microcommits and Multi-Head
---------------------------

.. TODO switch to phabricator many-small-commits doc?  https://secure.phabricator.com/book/phabflavor/article/writing_reviewable_code/#many-small-commits
.. FIXME Does the "everything in a small commit" workflow still work for phabricator?

We recommend reading the `How to Structure Commits <http://mozilla-version-control-tools.readthedocs.io/en/latest/mozreview/commits.html#how-to-structure-commits>`_ documentation.

It is recommend to author more, smaller commits than fewer, larger commits.  This practice is sometimes referred to as *microcommits*. In general, a commit should be as small as possible but no smaller.  Here are some guidelines:

* If you need to perform some cleanup before a refactor, put the cleanup in its own commit separate from the refactor.
* If you need to fix a typo, put that in its own commit.
* If you need to make a wide-sweeping change such as adding an argument to a commonly-called function, update function declarations one at a time (1 per commit) or use 1 commit to introduce the new interface and another for implementing it

While some developers use bookmarks/etc to track changes, it's possible to just create a new "head", which essentially means "just start coding off tip and commit".  The ``wipshort`` alias provides a view of the repository that allows for keeping track of the work.

Getting Started
===============

Updating Code
-------------

Let's start working on a new bug.

* ``wipshort`` shows we have a clean repository to work on

::

  $ hg wipshort
  @  4815 tip @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~

* After updating the files:

::

  $ hg status
  M pylib/mozreview/mozreview/extension.py

* You need to commit the changes

.. FIXME remove this

  * You have to specify the Bug number in the first line of the commit message
  * You should also set my reviewers here, using "r?name", specifying their IRC nickname (you can comma separate multiple reviewers)

::

  $ hg commit
  mozreview: Fix that broken thing (Bug 123456) r?smacleod

  There was a thing that was broken, because we assumed apples, but there were
  actually oranges.  Switch over to oranges.

  $ hg wipshort
  @  4816 tip mozreview: Fix that broken thing (Bug 123456) r?smacleod
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~

* And then in another commit let's add a test case:

::

  $ hg status
  M hgext/reviewboard/tests/test-push.t
  $ hg commit
  mozreview: Add test for apples/oranges (Bug 123456) r?smacleod

  Ensure we use oranges instead of apples when doing that thing.

  $ hg wipshort
  @  4817 tip mozreview: Add test for apples/oranges (Bug 123456) r?smacleod
  o  4816 mozreview: Fix that broken thing (Bug 123456) r?smacleod
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~

.. _updating-commits:

Updating Commits
----------------

Oops, while working on the tests I found an issue with a change, let's fix that.

* First, `checkout` the revision that needs to be updated

::

  $ hg wipshort
  @  4817 tip mozreview: Add test for apples/oranges (Bug 123456) r?smacleod
  o  4816 mozreview: Fix that broken thing (Bug 123456) r?smacleod
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~
  $ hg co 4816
  1 files updated, 0 files merged, 0 files removed, 0 files unresolved
  $ hg wipshort
  o  4817 tip mozreview: Add test for apples/oranges (Bug 123456) r?smacleod
  @  4816 mozreview: Fix that broken thing (Bug 123456) r?smacleod
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~

* Make the changes, and ``amend``

  * ``hg commit --amend`` also works, and allows you to update the commit description while amending the commit

::

  $ vi pylib/mozreview/mozreview/extension.py
  $ hg status
  M pylib/mozreview/mozreview/extension.py
  $ hg amend
  1 new unstable changesets

* ``wipshort`` shows that the ``amend`` has orphaned all children of the amended revision (4817 in this example)

::

  $ hg wipshort
  @  4819 tip mozreview: Fix that broken thing (Bug 123456) r?smacleod
  | o  4817 mozreview: Add test for apples/oranges (Bug 123456) r?smacleod
  | x  4816 mozreview: Fix that broken thing (Bug 123456) r?smacleod
  |/
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~

* We need to rebase the orphans onto the updated revision

::

  $ hg rebase -s 4817 -d 4819
  rebasing 4817:32d34909fb2f "mozreview: Add test for apples/oranges (Bug 123456) r?smacleod"
  $ hg wipshort
  o  4820 tip mozreview: Add test for apples/oranges (Bug 123456) r?smacleod
  @  4819 mozreview: Fix that broken thing (Bug 123456) r?smacleod
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~
  $ hg co 4820
  1 files updated, 0 files merged, 0 files removed, 0 files unresolved

* Mercurial's ``histedit`` command allows you to fancy things to commits like reordering or folding (combining).  Read more about Histedit on `the Histedit wiki page <https://www.mercurial-scm.org/wiki/HisteditExtension>`_.

Requesting a Review
-------------------

Tests pass, and you're happy with the change; let's push it to MozReview.

* First, let's ensure we're working off the latest revision of the code :todo:`screenshot: hg pull -u; rebase`
* I like to use my ``hg-outgoing`` shell function to read through my changes before pushing :todo:`maybe don't include this`

  * This is almost the same as checking out each revision we're going to push, and running ``hg diff -c .``

* Pushing it for review is simple: ``hg push review``.  Pay attention to any warnings about reviewers. :todo:`screenshot: hg push review`

Working on Something Else
-------------------------

Reviews can take some time, let's work on another bug.

* Checkout the last public revision :todo:`screenshot: wip; hg pull -u; hg co @`
* Then make changes and commit :todo:`find another good example bug to work on.` :todo:`screenshot: vi, commit, wip`

Dealing with Review Feedback
----------------------------

The reviewer wanted some changes, let's make those and push for another review.  This is generally the same steps as :ref:`updating-commits`.

* Checkout the revision we want to change :todo:`screenshot: wip; hg co`
* Make the changes, amend, and rebase :todo:`screenshot: vi,amend, rebase`
* To re-request a review, push again to MozReview :todo:`screenshot: hg push`

  * MozReview will carry forward existing r+'s, and request new reviews from the review where required :todo:`link to docs describing rules, write if missing!`

Landing with Autoland
---------------------

:todo:`the rest of this document is a TODO`

* requirements
* fix-on-commit

  * basically update as normal, push, then autoland

* pruning landed revisions

  * don't do it immediately; you can't unprune with evolve

Reviewing Code
--------------

* update your repo
* ``hg pull`` from ui url

  * wip screenshot

* review
* don't prune changes until bug fixed, and has stabilised

  * ``abort: 00changelog.i@5e746b6fb9dd41bdc2c9ab9c49f27e68f52e0bb9: filtered node!``

Mercurial Extensions and Aliases
================================

Evolve
------

You need the evolve extension: https://www.mercurial-scm.org/wiki/EvolveExtension#Setup

This extension allows for correct editing of history.  Make sure you add it to your user ``.hgrc`` file (ie. ``~/.hgrc``), not the hgrc file within the cloned repository.  Running ``hg config -e`` will open the correct file in your editor.

Aliases
-------

I have a few Mercurial tweaks that make my life slightly easier.  Most of these require editing your user ``.hgrc`` file with ``hg config -e``.

autoreview repo
^^^^^^^^^^^^^^^

There is a special repository called the ``autoreview`` repository that will automatically see what you are pushing and redirect your push to the appropriate code review repository.  See `Configuring The Auto Review Repository <http://mozilla-version-control-tools.readthedocs.io/en/latest/mozreview/install-mercurial.html#configuring-the-auto-review-repository>`_ for installation instructions.

wipshort
^^^^^^^^

An alternative to Mozilla's `wip` alias, which uses more concise and colourised output.

::

    [alias]
    wipshort = log --graph --rev=wip --template=wipshort

    [templates]
    wipshort = '{label(ifeq(graphnode,"x","custom.rev_obsolete",ifeq(phase,"draft","custom.rev_draft","custom.rev_public")),rev)}{label("custom.tags", if(tags," {tags}"))}{label("custom.bookmarks", if(bookmarks," {bookmarks}"))} {label(ifcontains(rev, revset("parents()"), "custom.here"), desc|firstline)}'

    [color]
    custom.bookmarks = magenta
    custom.here = red
    custom.rev_draft = green
    custom.rev_obsolete = none
    custom.rev_public = blue
    custom.tags = yellow

    [experimental]
    graphshorten = true

.. image:: wip-wipshort.png

This document uses ``wipshort`` in its output, however due to limitations of ReST colour cannot be used.

:todo:`either talk with gps about getting this added to mach, or don't mention it in the official docs. going to push for inclusion and i find it much nicer than mozilla's "wip" alias`

ls
^^

:todo:`probably going to remove this from this document`

Lists files modified by the current revisions.

::

    [alias]
    ls = !if [[ \"$1\" == \"\" ]]; then $HG log_files_draft | sort -u ; else $HG log_files -r "$@" | sort -u; fi
    log_files = log --template '{join(files,"\n")}\n'
    log_files_draft = log --template '{join(files,"\n")}\n' -r 'children(last(public()))::.'

grab
^^^^

:todo:`probably going to remove this from this document`

Simple alias for rebasing.  Note this overrides the ``grab`` alias defined by the ``evolve`` extension (``evolve``'s ``grab`` picks up a single revision, while this version grabs the revision and all children).

::

    [alias]
    grab = !$HG rebase --dest . --source $@ && $HG up tip

hg-outgoing
^^^^^^^^^^^

:todo:`probably going to remove this from this document`

Unlike the previous aliases, this is a *shell alias*, and should be added to the appropriate shell startup file (eg. ``~/.bash_aliases``).

::

    function hg-outgoing {
        local rev
        for rev in $(hg log -r 'children(last(public()))::.' --template '{rev}\n'); do
            hg --color always export -r $rev | less -R
        done
    }


########
New Work
########

git: one branch, one revision, multiple commits
git: stacked branches, stacked revisions
hg: one bookmark, one revision, multiple commits
hg: stacked bookmarks, stacked revisions
hg: one bookmark, multiple revisions, one commit per revision

Story 1: hg with one bookmark, multiple commits, one revision
-------------------------------------------------------------------

::

    $ hg wipshort

    $ hg bookmark easy-fix

    # hack hack

    $ hg status
    $ hg commit

    # Link to well-formatted commit messages
    # Mention convention of putting component first in subject

    $ hg status

    # Add tests

    $ hg wipshort

    # Oops, found a problem

    # hack hack

    $ hg commit

    $ hg wipshort

    # Request the review

    $ hg pull --update --rebase

    $ arc diff

    # Note the needed bug #, reviewers

    # Addressing feedback

    $ hg wipshort

    # hack hack
    # checkbox in UI

    $ hg commit -m 'fix lint'

    # hack hack
    # checkbox in UI

    $ hg commit -m 'fix complex problem foo'

    $ arc diff

    # ^ should "just work"

    # TODO landing changes

    $ hg pull --update --rebase

    $ arc land

    # FIXME ^ should that use lando instead?  maybe keep command for branches not in mozilla-central.


Story 2: hg with multiple bookmarks and stacked branches
--------------------------------------------------------


::

    $ hg wipshort

    $ hg bookmark complex-fix-part-1

    # hack hack

    $ hg status
    $ hg commit

    # Link to well-formatted commit messages
    # Mention convention of putting component first in subject

    $ hg status

    $ hg bookmark complex-fix-part-2

    # Add tests

    $ hg commit

    $ hg wipshort

    # Oops, found a problem

    $ hg checkout 4816

    $ hg wipshort

    # hack hack

    $ hg amend / hg commit --amend

    $ hg wipshort

    $ hg evolve (maybe hg rebase)

    # Request the review

    $ hg wipshort
    $ arc feature

    $ hg bookmark complex-fix-part-1 / arc feature complex-fix-part-1

    $ hg pull --update --rebase

    # FIXME ^^^ check these commands, is this how we evolve a bookmark stack?

    $ arc diff @

    # Note the needed bug #, reviewers

    $ hg bookmark complex-fix-part-2 / arc feature complex-fix-part-2

    # Note that the user needs to specify the diff target or it will default to @ for stacked bookmarks
    $ arc diff complex-fix-part-1

    # Might want to note the user can run $ arc diff --browse complex-diff-part-1
    # Might want to mention tab completion

    # Set stack relation

    # Addressing feedback

    $ hg wipshort
    $ arc feature

    $ hg bookmark complex-fix-part-1 / arc feature complex-fix-part-1

    # hack hack

    $ hg commit -m 'fix but in function()'

    $ arc diff

    $ hg wipshort
    $ arc feature

    $ hg evolve

    $ hg wipshort
    $ arc feature

    # Note that the commit<->revision link is gone.  Maybe?  Does it store the link by bookmark or by commit?

    $ arc diff --update D9999 complex-feature-part-1

    # Landing changes

    $ arc feature
    $ arc feature complex-fix-part-1
    $ hg pull --update --rebase
    $ arc land

    $ hg evolve (maybe)
    $ arc feature
    $ arc land


Story blah
----------

::

    $ hg wipshort

    # hack hack

    $ hg status
    $ hg commit

    # Link to well-formatted commit messages
    # Mention convention of putting component first in subject

    $ hg status

    # Add tests

    $ hg wipshort

    # Oops, found a problem

    $ hg checkout 4816

    $ hg wipshort

    $ hg amend / hg commit --amend

    $ hg wipshort

    $ hg rebase / hg evolve

    $ hg wipshort

    # Request the review

    $ hg pull --update --rebase

    $ arc diff ????

    # Note the needed bug #, reviewers
