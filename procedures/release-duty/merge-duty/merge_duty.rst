Merge Duty
==========

All code changes to Firefox land in the
`mozilla-central <https://hg.mozilla.org/mozilla-central>`__ repository
while the Fennec counterpart land to
`mozilla-esr68 <https://hg.mozilla.org/releases/mozilla-esr68>`__.

* The ``nightly`` releases are built from that repo twice a day.
* DevEdition and Beta releases are built from the `beta <https://hg.mozilla.org/releases/mozilla-beta/>`__ repository
* Extended Support Releases follow-up from the relevant ESR repo, such as `mozilla-esr68 <https://hg.mozilla.org/releases/mozilla-esr68/>`__
* Release and Release Candidates are built from `mozilla-release <https://hg.mozilla.org/releases/mozilla-release/>`__ repository

How are those repositories kept in sync? That's ``MergeDuty`` and is
part of the ``releaseduty`` responsibility.

Overview of Procedure
---------------------

``MergeDuty`` consists of multiple separate days of work. Each day you
must perform several sequential tasks. The days are spread out over
nearly three weeks, with *three* major days of activity:

-  Do the prep work a week before the merge

   -  `File tracking migration bug <#file-tracking-migration-bug>`__
   -  `Turn off beta l10n bumper on RC day <#turn-off-beta-l10n-bumper-on-rc-day>`__
   -  `Do migration no-op trial runs <#do-migration-no-op-trial-runs>`__
   -  `Sanity check no blocking migration
      bugs <#sanity-check-no-blocking-migration-bugs>`__
   -  `Land whatsnewpage list of
      locales <#land-whatsnewpage-list-of-locales>`__

-  On Merge day:

   -  `Merge beta to release <#merge-beta-to-release>`__
   -  `Reply migrations are
      complete <#reply-to-relman-migrations-are-complete>`__

-  A week after Merge day, bump mozilla-central:

   -  `Merge central to beta <#merge-central-to-beta>`__
   -  `Re-open trees <#re-opening-the-trees>`__
   -  `Verify the l10n bumper output <#verify-the-l10n-bumper-output>`__
   -  `Tag central and bump versions <#tag-central-and-bump-versions>`__
   -  `Bump mozilla-esr <#bump-esr-version>`__
   -  `Reply to RelMan that procedure is
      completed <#reply-to-relman-central-bump-completed>`__
   -  `Update wiki versions <#update-wiki-versions>`__
   -  `Bump Nightly version in ShipIt <#bump-nightly-shipit>`__

Historical context of this procedure:

Originally, the ``m-c`` -> ``m-b`` was done a week after ``m-b`` ->
``m-r``. Starting at ``Firefox 57``, Release Management wanted to ship
DevEdition ``b1`` week before the planned mozilla-beta merge day. This
meant Releng had to merge both repos at the same time. With 71.0, we're
back to the initial workflow with merging ``m-b`` -> ``m-r`` in the
first week and then ``m-c`` -> ``m-b`` in the follow-up week.

Do the prep work a week before the merge
----------------------------------------

File tracking migration bug
~~~~~~~~~~~~~~~~~~~~~~~~~~~

File a tracking migration bug if there isn't one (e.g. `bug
1412962 <https://bugzilla.mozilla.org/show_bug.cgi?id=1412962>`__)

Turn off beta l10n bumper on RC day
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Write a patch against mozilla-beta that removes `this line <https://hg.mozilla.org/releases/mozilla-beta/file/5e8e24da4af86d6f67e6145397bcb5e27dc09d89/.cron.yml#l287>`__. This patch should only land on mozilla-beta, not mozilla-central. This patch should land before RC GTB.

The rationale: we no longer have Elmo for localization signoffs. Let's turn off the automatic bumper on first RC, and only update l10n-changesets manually if needed. By leaving central untouched, we will reenable beta l10n-bumper on central-to-beta merge day.

Do migration no-op trial runs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doing a no-op trial run of each migration has one major benefit these
days: you ensure that the migrations themselves work prior to Merge day.

General steps
^^^^^^^^^^^^^

1. Go to
   `Treeherder <https://treeherder.mozilla.org/>`__.
2. Select the repo depending on the merge you want to perform (central, beta or the ESR one).
3. On the latest push, click on the down arrow at the top right corner.
4. Select “Custom push action…”
5. Choose ``merge-automation``

mozilla-beta->mozilla-release migration no-op trial run
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Follow the `general steps <#general-steps>`__ hopping on `beta <https://treeherder.mozilla.org/#/jobs?repo=mozilla-beta>`__
2. Insert the following payload and click submit.

.. code:: yaml

   force-dry-run: true
   behavior: beta-to-release
   push: true

mozilla-central->mozilla-beta migration no-op trial run
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Follow the `general steps <#general-steps>`__ hopping on `central <https://treeherder.mozilla.org/#/jobs?repo=mozilla-central>`__
2. Insert the following payload and click submit.

.. code:: yaml

   force-dry-run: true
   behavior: central-to-beta
   push: true

mozilla-esr bump no-op trial run
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Follow the `general steps <#general-steps>`__ hopping on `esr <https://treeherder.mozilla.org/#/jobs?repo=mozilla-esr78>`__
2. Insert the following payload and click submit.

.. code:: yaml

   force-dry-run: true
   behavior: bump-esr
   push: true

Diff should be similar to `this esr68
one <https://hg.mozilla.org/releases/mozilla-esr68/rev/bf17c381b0615fba955f8998c89593b103f32ba1>`__ or `this esr78 one <https://hg.mozilla.org/releases/mozilla-esr78/rev/5024137054922f8f9565a04a2fa4c5326ee1f190>`__.

Sanity check no blocking migration bugs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Make sure the bug that tracks the migration has no blocking items.

Land whatsnewpage list of locales
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**TODO** - this needs to change, as the process no longer assumes this,
but apply them; the l10n drivers provide the final list of locales to
receive the WNP on the Tuesday prior to the ship date.

1. For each release, there should already be a bug flying around named
   ``Setup WNP for users coming from < X and receiving the X release``.
   Find it for the current release. e.g. `Bug
   1523699 <https://bugzilla.mozilla.org/show_bug.cgi?id=1523699>`__. We
   should always aim to chain this bug to our main mergeduty tracking
   bug. That is, block the WNP bug against the
   ``tracking XXX migration day``. If not already, please do so. This
   way, it's easier to find deps and navigate via bugs.
2. By the Friday prior to merge day, the l10n (most likely
   ``Peiying Mo [:CocoMo]``) team will have posted the final list of
   locales for whatsnewpage. Double-check with them again to make sure
   that is the final list. The list of locales comes in two forms:
   attachment in bug directly to be ``hg import``\ ed, but also as a
   comment. Make sure to double-check they match as that's generated
   automatically and sometimes there could be fallout resulting in
   mismatches.
3. Update the `in-tree whatsnewpage list of
   locales <https://hg.mozilla.org/mozilla-central/file/tip/browser/config/whats_new_page.yml>`__
   on central and request an uplift of that to beta. Similar to `this
   patch <https://hg.mozilla.org/mozilla-central/rev/55c218c9489b>`__.
   It will uplift to release when the merge happens on Monday

   1. On development machine, update
      ``browser/config/whats_new_page.yml`` with the list of locales
      from the bug
   2. Commit the change and create Phabricator patch request as usual
   3. Once the patch request is approved, land the patch via lando
   4. In Bugzilla edit the phabricator attachment and add a
      approval-mozilla-beta? flag similar to
      `this <https://bugzilla.mozilla.org/show_bug.cgi?id=1616636#c7>`__
   5. ensure someone from sheriffs or relman uplift this to Beta before
      Monday's merge and RC go-to-build

Release Merge Day - part I
--------------------------

**When**: Wait for go from relman to release-signoff@mozilla.com. Relman
might want to do the migration in two steps. Read the email to
understand which migration you are suppose to do, and then wait for
second email. For date, see `Release Scheduling
calendar <https://calendar.google.com/calendar/embed?src=bW96aWxsYS5jb21fZGJxODRhbnI5aTh0Y25taGFiYXRzdHY1Y29AZ3JvdXAuY2FsZW5kYXIuZ29vZ2xlLmNvbQ>`__
or check with relman

Merge beta to release
~~~~~~~~~~~~~~~~~~~~~

1. `Close
   mozilla-beta <https://treestatus.mozilla-releng.net/static/ui/treestatus/show/mozilla-beta>`__.
   Check *“Remember this change to undo later”*. Please enter a good
   message as the reason for the closure, such as “Mergeduty - closing
   beta for $VERSION RC week”.
2. Run the ``m-b -> m-r`` `no-op trial
   run <#do-migration-no-op-trial-runs>`__ one more time, and show the
   diff to another person on releaseduty.
3. The diff for ``release`` should be fairly similar to
   `this <https://hg.mozilla.org/releases/mozilla-release/rev/0eae18af659f087056bce0f62a325e5e595fff72>`__,
   with updated the version change.
4. Submit a new task with ``force-dry-run`` set to false:

.. code:: yaml

   force-dry-run: false
   behavior: beta-to-release
   push: true

:warning:
   It's not unlikely for the push to take between 10-20 minutes to complete.

:warning:
   If an issue comes up during this phase, you may not be able to run
   this command (or the no-op one) correctly. You may need to publicly
   backout some tags/changesets to get back in a known state.

1. Upon successful run, ``mozilla-release`` should get a version bump
   and branding changes consisting of a ``commit`` like
   `this <https://hg.mozilla.org/releases/mozilla-release/rev/0eae18af659f087056bce0f62a325e5e595fff72>`__
   and a ``tag`` like
   `this <https://hg.mozilla.org/releases/mozilla-release/rev/be8c618fd8ad921642e04e1552fbad46a044fe9e>`__
2. In the same time ``mozilla-beta`` should get a tag like
   `this <https://hg.mozilla.org/releases/mozilla-beta/rev/d87f9b66ddd19a973ec3ef26a9163bab9383c438>`__
3. Verify changesets are visible on `hg
   pushlog <https://hg.mozilla.org/releases/mozilla-release/pushloghtml>`__
   and
   `Treeherder <https://treeherder.mozilla.org/#/jobs?repo=mozilla-release>`__.
   It may take a couple of minutes to appear.

:warning:
   The decision task of the resulting pushlog in the ``mozilla-release``
   might fail in the first place with a timeout. A rerun might solve
   the problem which can be caused by an unlucky slow instance.

Reply to relman migrations are complete
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Reply to the migration request with the template:

.. code:: text

   This is now complete:
   * mozilla-beta is merged to mozilla-release, new version is XX.Y
   * beta will stay closed until next week

Release Merge Day - part II - a week after Merge day
----------------------------------------------------

**When**: Wait for go from relman to release-signoff@mozilla.com. For
date, see `Release Scheduling
calendar <https://calendar.google.com/calendar/embed?src=bW96aWxsYS5jb21fZGJxODRhbnI5aTh0Y25taGFiYXRzdHY1Y29AZ3JvdXAuY2FsZW5kYXIuZ29vZ2xlLmNvbQ>`__
or check with relman

Merge central to beta
~~~~~~~~~~~~~~~~~~~~~

1. Run the ``m-c -> m-b`` `no-op trial
   run <#do-migration-no-op-trial-runs>`__ one more time, and show the
   diff to another person on releaseduty.
2. The diff generated by the task should be fairly similar to
   `this <https://hg.mozilla.org/releases/mozilla-beta/rev/13d947f127a76828e19d8bb7f8f6353a7b3a0f6e>`__.
3. Submit a new task with ``force-dry-run`` set to false:

.. code:: yaml

   force-dry-run: false
   behavior: central-to-beta
   push: true

:warning:
   It's not unlikely for the push to take between 10-20 minutes to complete.

1. Upon successful run, ``mozilla-beta`` should get a version bump and
   branding changes consisting of a ``commit`` like
   `this <https://hg.mozilla.org/releases/mozilla-beta/rev/13d947f127a76828e19d8bb7f8f6353a7b3a0f6e>`__
   and a ``tag`` like
   `this <https://hg.mozilla.org/releases/mozilla-beta/rev/a6981603097c54950b3a00a6e7aa95f532947482>`__
2. In the same time ``mozilla-central`` should get a tag like
   `this <https://hg.mozilla.org/mozilla-central/rev/6d98cc745df58e544a8d71c131f060fc2c460d83>`__
3. Verify changesets are visible on `hg
   pushlog <https://hg.mozilla.org/releases/mozilla-beta/pushloghtml>`__
   and
   `Treeherder <https://treeherder.mozilla.org/#/jobs?repo=mozilla-beta>`__.
   It may take a couple of minutes to appear.

:warning:
   The decision task of the resulting pushlog in the ``mozilla-beta``
   might fail in the first place with a timeout. A rerun might solve
   the problem which can be caused by an unlucky slow instance.

Re-opening the tree(s)
~~~~~~~~~~~~~~~~~~~~~~

`Reopen the mozilla-beta tree <https://treestatus.mozilla-releng.net/static/ui/treestatus/show/mozilla-beta>`__
by restoring the previous state so that **l10n bumper can run**.

Verify the l10n bumper output
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In theory, this happened during central-to-beta merges after 2020.09.30.
Verify that ``browser/locales/l10n-changesets.json`` has revisions, not
``default``, and/or verify that the merge task has l10n-bump in the logs.
The diff should look like `this <https://hg.mozilla.org/releases/mozilla-beta/rev/7564379b690bb9c24cb9a7a4bbb2552c9724c147>`__

Tag central and bump versions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**What happens**: A new tag is needed to specify the end of the nightly
cycle. Then clobber and bump versions in ``mozilla-central`` as
instructions depict.

1. Follow the `general steps <#general-steps>`__
2. Insert the following payload and click submit.

.. code:: yaml

   force-dry-run: false
   push: true
   behavior: bump-central

1. Upon successful run, ``mozilla-central`` should get a version bump
   consisting of a ``commit`` like
   `this <https://hg.mozilla.org/mozilla-central/rev/b00860a2a28336267070c6fd882f0f5feabcebad>`__
   and a ``tag`` like
   `this <https://hg.mozilla.org/mozilla-central/rev/0ab2bba66188606446c37868f4b01cdffebd0acc>`__
2. Verify changesets are visible on `hg
   pushlog <https://hg.mozilla.org/mozilla-central/pushloghtml>`__ and
   `Treeherder <https://treeherder.mozilla.org/#/jobs?repo=mozilla-central>`__.
   It may take a couple of minutes to appear.

Bump ESR version
~~~~~~~~~~~~~~~~

Note: You could have one ESR to bump, or two. If you are not sure, ask.

Run the bump-esr `no-op trial run <#do-migration-no-op-trial-runs>`__
one more time, and show the diff to another person on releaseduty.

Diff should be similar to `this
one <https://hg.mozilla.org/releases/mozilla-esr78/rev/5024137054922f8f9565a04a2fa4c5326ee1f190>`__.

Push your changes generated by the no-op trial run:

1. Follow the `general steps <#general-steps>`__ - (As of 2020/04 this
   action hasn't yet been uplifted to release or esr68, consider
   using ``mozilla-central``\ 's action, as the payload controls where
   the effects land)
2. Insert the following payload and click submit.

.. code:: yaml

   force-dry-run: false
   push: true
   behavior: bump-esr

*Note* This is currently set to ``esr78``, the defaults can be
overridden in-tree in ``taskcluster/ci/config.yml`` or specified here as
using an action payload such as:

.. code:: yaml

   force-dry-run: false
   push: true
   behavior: bump-esr
   to-branch: esr88
   to-repo: https://hg.mozilla.org/releases/mozilla-esr88

1. Upon successful run, ``mozilla-esr${VERSION}`` should get a
   ``commit`` like
   `this <https://hg.mozilla.org/releases/mozilla-esr78/rev/5024137054922f8f9565a04a2fa4c5326ee1f190>`__.
2. Verify new changesets popped on
   https://hg.mozilla.org/releases/mozilla-esr78/pushloghtml

Reply to relman central bump completed
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Reply to the migration request with the template:

.. code:: text

   This is now complete:
   * mozilla-central is merged to mozilla-beta, new version is XX.Y
   * mozilla-central has been tagged and version bumped
   * mozilla-esr has been version bumped
   * newly triggered nightlies will pick the version change on cron-based schedule

Update wiki versions
~~~~~~~~~~~~~~~~~~~~

1. Edit the new values manually:

-  `NEXT_VERSION <https://wiki.mozilla.org/Template:Version/Gecko/release/next>`__
-  `CENTRAL_VERSION <https://wiki.mozilla.org/Template:Version/Gecko/central/current>`__
-  `BETA_VERSION <https://wiki.mozilla.org/Template:Version/Gecko/beta/current>`__
-  `RELEASE_VERSION <https://wiki.mozilla.org/Template:Version/Gecko/release/current>`__
-  `Next release
   date <https://wiki.mozilla.org/index.php?title=Template:NextReleaseDate>`__.
   This updates

   -  `The next ship
      date <https://wiki.mozilla.org/index.php?title=Template:FIREFOX_SHIP_DATE>`__
   -  `The next merge
      date <https://wiki.mozilla.org/index.php?title=Template:FIREFOX_MERGE_DATE>`__
   -  `The current
      cycle <https://wiki.mozilla.org/index.php?title=Template:CURRENT_CYCLE>`__

Bump Nightly version and release dates in ShipIt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ShipIt currently hard-codes the version of Nightly that's being released, as
well as the release dates.

It doesn't get automatically updated because it would need to know when a new
nightly was available, not just when the version had been updated in-tree.
Everything up to merging this pull request can be done early, but the PR must
not be merged before the first nightly has been built and published with the
new version.

1. ``git clone git@github.com:mozilla-releng/shipit.git``
2. ``git checkout -b nightly_version_bump_${version}``
3. Edit FIREFOX_NIGHTLY's major version in
   https://github.com/mozilla-releng/shipit/blob/f3d45d1dd1cc08cc466865f7d39305f1b2edbcf7/api/src/shipit_api/common/config.py#L49
4. Edit the `LAST` and `NEXT` known dates (all 6 of them) at
   https://github.com/mozilla-releng/shipit/blob/f3d45d1dd1cc08cc466865f7d39305f1b2edbcf7/api/src/shipit_api/common/config.py#L54-L59
5. Commit, and submit a pull request
6. Merge the pull request *after* a new nightly version has been pushed
   to CDNs
