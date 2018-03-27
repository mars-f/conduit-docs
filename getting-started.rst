.. role:: todo
.. raw:: html

    <style>
      .todo { color: red }
    </style>

.. NOTE::

    :todo:`TODO items look like this.`

##############
Original guide
##############

************
Introduction
************

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
  mozreview: Fix that broken thing

  There was a thing that was broken, because we assumed apples, but there were
  actually oranges.  Switch over to oranges.

  $ hg wipshort
  @  4816 tip mozreview: Fix that broken thing
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~

* And then in another commit let's add a test case:

::

  $ hg status
  M hgext/reviewboard/tests/test-push.t
  $ hg commit
  mozreview: mozreview: Add test for apples/oranges

  Ensure we use oranges instead of apples when doing that thing.

  $ hg wipshort
  @  4817 tip mozreview: mozreview: Add test for apples/oranges
  o  4816 mozreview: Fix that broken thing
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
  @  4817 tip mozreview: mozreview: Add test for apples/oranges
  o  4816 mozreview: Fix that broken thing
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~
  $ hg co 4816
  1 files updated, 0 files merged, 0 files removed, 0 files unresolved
  $ hg wipshort
  o  4817 tip mozreview: mozreview: Add test for apples/oranges
  @  4816 mozreview: Fix that broken thing
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
  @  4819 tip mozreview: Fix that broken thing
  | o  4817 mozreview: mozreview: Add test for apples/oranges
  | x  4816 mozreview: Fix that broken thing
  |/
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~

* We need to rebase the orphans onto the updated revision

::

  $ hg rebase -s 4817 -d 4819
  rebasing 4817:32d34909fb2f "mozreview: mozreview: Add test for apples/oranges"
  $ hg wipshort
  o  4820 tip mozreview: mozreview: Add test for apples/oranges
  @  4819 mozreview: Fix that broken thing
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

########
New Work
########

************
Introduction
************

TODO

***************
Getting Started
***************

git: one branch, one revision, multiple commits
git: stacked branches, stacked revisions
hg: one bookmark, one revision, multiple commits
hg: stacked bookmarks, stacked revisions
hg: one bookmark, multiple revisions, one commit per revision

Story 1: hg with one bookmark, multiple commits, one revision
=============================================================

Fixing the code
---------------

Let's start with a clean checkout.

::

  $ hg wipshort
  @  5865 tip @ autoland: configure lando s3 bucket (bug 1448051).
  |
  ~

  $ hg bookmark fix-docstring
  $ vim pylib/mozautomation/mozautomation/commitparser.py
  $ hg status
  M pylib/mozautomation/mozautomation/commitparser.py

Make sure our commit message is well-formatted.

  * `Here is how to write a good commit message <https://chris.beams.io/posts/git-commit/#why-not-how>`_
  * We do not need a bug number or "r?" reviewers list [FIXME check this]
  * The component name is not needed for Firefox [FIXME check this]

::

  $ hg commit
  mozautomation: fix pep8 lint

  Fix some PEP 8 lint in the module level docstrings.

Oops, we found a second file to update.  We can add a fix-up commit to the bookmark that adds the new changes.

::

  $ vim pylib/mozautomation/mozautomation/treestatus.py
  $ hg commit -m 'docstring for the treestatus module'
  $ hg wipshort

  @  5871 tip fix-docstring docstring for the treestatus module
  o  5870 mozautomation: fix pep8 lint
  o  5865 @ autoland: configure lando s3 bucket (bug 1448051).
  o  5864 hgmo: upgrade ZooKeeper to 3.4.11 (bug 1434339) r=sheehan
  o  5863 autoland: configure lando pingback url (bug 1445567) (fixup)
  |


Requesting a Review
-------------------

Before we request a review we should check for changes upstream.

::

  $ hg pull --rebase

* If you want to look before you leap in with ``arc diff`` or ``arc land``, you can run ``arc which`` to get a description of what ``arc`` is going to do next.

We need to include:

* An real BMO Bug #
* Reviewers
* A Test Plan, even if it is just the string 'n/a'

::

  $ arc diff
  mozautomation: fix pep8 lint

  Summary:

  Fix some PEP 8 lint in the module level docstrings.

  Test Plan: $ pep8 thefiles

  Reviewers: glob, imadueme

  Subscribers:

  Bug #: 5556555

  # NEW DIFFERENTIAL REVISION
  # Describe the changes in this new revision.
  #
  # Included commits in branch default:
  #
  #         153ddf055585 docstring for the treestatus module
  #         c7ab40d66585 mozautomation: fix pep8 lint
  #
  # arc could not identify any existing revision in your working copy.
  # If you intended to update an existing revision, use:
  #
  #   $ arc diff --update <revision>


Addressing feedback
-------------------

Our reviewers came back with some changes.  Let's add some fix-up commits for the work.

::

  $ hg checkout fix-docstring
  $ vim pylib/mozautomation/mozautomation/treestatus.py
  $ hg commit -m 'fix lint'

Phabricator has a neat trick where you can check the 'Done' button in the review at the same time as fixing the commit.  The next time you put your changes up for review with ``arc diff``, Phabricator will automagically submit your 'Done' items and bundle them into a nice summary.

.. TODO Screenshot of Done item

::

  $ arc diff

Landing the changes
-------------------

.. FIXME Lando?

Everything looks good, let's land our changes in mainline.

::

  $ hg checkout fix-docstring

We'll check that there are no conflicting changes upstream.

::

  $ hg pull --rebase

``arc land`` is going to squash our bookmark into a single commit before adding the changes to mainline. The Phabricator review fields will become the commit message.

.. TODO checking for "\bwip\b" in the commit message would make a good lint extension to 'arc land'

**NOTE** Make sure you remove any "WIP not ready yet" stuff from the review summary before running ``arc land``!

* If you want to check what commit message ``arc land`` is going to use, you can run ``arc amend`` to update your local changeset's commit message to match Phabricator.

::

  $ arc land

.. FIXME ^ should that use lando instead?  maybe keep command for branches not in mozilla-central.


Story 2: hg with one commit per review
======================================

Telling arc to make one review per changeset
--------------------------------------------

First we need to tell the ``arc diff`` command to only submit the current changeset for review.

::

    $ arc set-config base 'arc:this, arc:prompt'

Everything for pushing up a single commit change is the same as if we used a bookmark.

::

    $ vim pylib/mozautomation/mozautomation/commitparser.py

    $ hg status
    M pylib/mozautomation/mozautomation/commitparser.py

    $ hg commit

    $ hg pull --rebase

    $ arc diff

When it's time to address feedback we use ``hg amend``.

  * ``hg commit --amend`` also works, and allows you to update the commit description while amending the commit

::

    $ hg wipshort
    o  5870 tip mozautomation: fix pep8 lint
    @  5865 @ autoland: configure lando s3 bucket (bug 1448051).
    o  5864 hgmo: upgrade ZooKeeper to 3.4.11 (bug 1434339) r=sheehan
    o  5863 autoland: configure lando pingback url (bug 1445567) (fixup)
    |

    $ hg checkout tip

    $ vim pylib/mozautomation/mozautomation/commitparser.py

    $ hg amend

    $ arc diff

Story 3: hg with stacked branches
=================================

Let's make a complex fix that would be easier to review if it were split into two parts.

We'll use the "One changeset per review" workflow.

.. FIXME do we still need to mention "no merge commits?" from http://mozilla-version-control-tools.readthedocs.io/en/latest/mozreview/commits.html#how-to-structure-commits


Telling arc to make one review per changeset
--------------------------------------------

If you haven't done so already, we need to tell the ``arc diff`` command to only submit the current changeset for review.

::

    $ arc set-config base 'arc:this, arc:prompt'

Fixing the code
---------------

First we'll submit part 1 for review.  Start with a clean branch.

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

::

  $ hg commit
  mozreview: Fix that broken thing

  There was a thing that was broken, because we assumed apples, but there were
  actually oranges.  Switch over to oranges.

  $ hg wipshort
  @  4816 tip mozreview: Fix that broken thing
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~

* And then in another commit let's add a test case:

::

  $ hg status
  M hgext/reviewboard/tests/test-push.t
  $ hg commit
  mozreview: Add test for apples/oranges

  Ensure we use oranges instead of apples when doing that thing.

  $ hg wipshort
  @  4817 tip mozreview: mozreview: Add test for apples/oranges
  o  4816 mozreview: Fix that broken thing
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
  @  4817 tip mozreview: mozreview: Add test for apples/oranges
  o  4816 mozreview: Fix that broken thing
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~
  $ hg co 4816
  1 files updated, 0 files merged, 0 files removed, 0 files unresolved
  $ hg wipshort
  o  4817 tip mozreview: mozreview: Add test for apples/oranges
  @  4816 mozreview: Fix that broken thing
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
  @  4819 tip mozreview: Fix that broken thing
  | o  4817 mozreview: mozreview: Add test for apples/oranges
  | x  4816 mozreview: Fix that broken thing
  |/
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~

* We need to rebase the orphans onto the updated revision

::

  $ hg rebase -s 4817 -d 4819
  rebasing 4817:32d34909fb2f "mozreview: mozreview: Add test for apples/oranges"
  $ hg wipshort
  o  4820 tip mozreview: mozreview: Add test for apples/oranges
  @  4819 mozreview: Fix that broken thing
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~
  $ hg co 4820
  1 files updated, 0 files merged, 0 files removed, 0 files unresolved

* Mercurial's ``histedit`` command allows you to fancy things to commits like reordering or folding (combining).  Read more about Histedit on `the Histedit wiki page <https://www.mercurial-scm.org/wiki/HisteditExtension>`_.

Requesting a Review
-------------------

Tests pass, and you're happy with the change; let's put it up for review in Phabricator.

We need to check out each individual changeset and submit it.

::

  $ hg wipshort
  o  4820 tip mozreview: mozreview: Add test for apples/oranges
  @  4819 mozreview: Fix that broken thing
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~
  $ hg co 4819
  $ arc diff
  $ hg next
  [4820] mozreview: Add test for apples/oranges
  $ arc diff

Now we can go to the Phabricator UI and set the relation between the two reviews.

.. TODO screenshot or link to stacking UI