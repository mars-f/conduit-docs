************
Introduction
************

Mercurial Setup
===============

.. TODO link to the main phab doc

Multi-Head vs. bookmarks
========================

While some developers use bookmarks/etc to track changes, it's possible to just create a new "head", which essentially means "just start coding off tip and commit".  The ``hg wip`` alias provides a view of the repository that allows for keeping track of the work.

.. FIXME: wording
.. FIXME: include link to https://mozilla-version-control-tools.readthedocs.io/en/latest/hgmozilla/workflows.html#to-label-or-not-to-label ?

***************
Getting Started
***************

Style 1: One review per head
============================

In this style we use GitHub-style fix-up commits under a single head.  The fix-ups will be squashed into a single commit before landing.

We'll use:
* One review
* Multiple fix-up commits under one head

Fixing the code
---------------

Let's start with a clean checkout.

::

  $ hg wip
  @  5865 tip @ autoland: configure lando s3 bucket (bug 1448051).
  |
  ~

  $ vim pylib/mozautomation/mozautomation/commitparser.py
  $ hg status
  M pylib/mozautomation/mozautomation/commitparser.py

Make sure our commit message is well-formatted.

  * We do not need a bug number or "r?" reviewers list

::

  $ hg commit
  Fix pep8 lint

  Fix some PEP 8 lint in the module level docstrings.

Oops, we found a second file to update.  We can add a fix-up commit to the head that adds the new changes.

::

  $ vim pylib/mozautomation/mozautomation/treestatus.py
  $ hg commit -m 'docstring for the treestatus module'
  $ hg wip
  @  5871 tip docstring for the treestatus module
  o  5870 Fix pep8 lint
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
  Fix pep8 lint

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
  #         c7ab40d66585 Fix pep8 lint
  #
  # arc could not identify any existing revision in your working copy.
  # If you intended to update an existing revision, use:
  #
  #   $ arc diff --update <revision>


Addressing feedback
-------------------

Our reviewers came back with some changes.  Let's add some fix-up commits for the work.

::

  $ hg wip
  o  5871 tip docstring for the treestatus module
  o  5870 Fix pep8 lint
  @  5865 @ autoland: configure lando s3 bucket (bug 1448051).

  $ hg checkout 5871
  $ vim pylib/mozautomation/mozautomation/treestatus.py
  $ hg commit -m 'fix lint'

Check off the Done item in the UI.

[TODO screenshot of Done item]

Now run ``arc diff``.  Phabrictor will automatically submit your Done items in the UI and create a nicely formatted update.

::

  $ arc diff

[TODO] screenshot of revision update

Landing the changes
-------------------

Everything looks good, let's land our changes in mainline.

[TODO requesting landing of your changes into FF - link?]
[TODO Lando]


Style 2: One changeset per review
=================================

In this style we craft just one commit per review.  When we get feedback or fixups we amend our single commit.

We'll use:
* One commit
* One review per commit
* ``hg amend`` to add fix-ups to our commit


Telling arc to make one review per changeset
--------------------------------------------

First we need to tell the ``arc diff`` command to only submit the current changeset for review.

Let's change the setting for just this project.

::

    $ arc set-config --local base 'arc:this, arc:prompt'

* NOTE: If you want to change this setting for all projects under your login, remove the ``--local`` switch.


Fixing the code
---------------

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

    $ hg wip
    o  5870 tip Fix pep8 lint
    @  5865 @ autoland: configure lando s3 bucket (bug 1448051).
    o  5864 hgmo: upgrade ZooKeeper to 3.4.11 (bug 1434339) r=sheehan
    o  5863 autoland: configure lando pingback url (bug 1445567) (fixup)
    |

    $ hg checkout tip

    $ vim pylib/mozautomation/mozautomation/commitparser.py

    $ hg amend

    $ arc diff


Stacked changes with evolve
===========================

.. TODO link to main phabricator doc about this?
.. TODO maybe remove the evolve extension reference.  smacleod says it's iffy to support because it's still experimental.


Let's make a complex fix that would be easier to review if it were split into two parts.

We'll use the "One changeset per review" workflow.

.. FIXME do we still need to mention "no merge commits?" from http://mozilla-version-control-tools.readthedocs.io/en/latest/mozreview/commits.html#how-to-structure-commits


Installing the Evolve extension
-------------------------------

TODO explanation of the `Evolve extension <https://www.mercurial-scm.org/wiki/EvolveExtension>`_


Telling arc to make one review per changeset
--------------------------------------------

If you haven't done so already, we need to tell the ``arc diff`` command to only submit the current changeset for review.

::

    $ arc set-config base 'arc:this, arc:prompt'

Fixing the code
---------------

First we'll submit part 1 for review.  Start with a clean branch.

::

  $ hg wip
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

  $ hg wip
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

  $ hg wip
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

  $ hg wip
  @  4817 tip mozreview: mozreview: Add test for apples/oranges
  o  4816 mozreview: Fix that broken thing
  o  4815 @ Bug 1309644 - Adding Kyle Machulis to WebIDL DOM Peer Hook; r=ted
  |
  ~
  $ hg co 4816
  1 files updated, 0 files merged, 0 files removed, 0 files unresolved
  $ hg wip
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

* ``wip`` shows that the ``amend`` has orphaned all children of the amended revision (4817 in this example)

::

  $ hg wip
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
  $ hg wip
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

  $ hg wip
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
