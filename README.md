Gerda
=====

Gerda is a command-line utility written in Python that combines
several gerrit-related hacks in one tool. It automates parts
of my gerrit workflow and provides a convenient place to write
even more hacks and automate even more parts of it.

Gerda should work with any gerrit installation, but some of
its parts might be more or less tied to what OpenStack CI provides.



Common arguments
----------------

  -h, --help            show this help message and exit
  --verbose, -v         verbose operation
  --host HOST, -H HOST  Gerrit hostname
  --user USER, -U USER  Gerrit username
  --port PORT, -P PORT  Gerrit SSH port


Host, user and port and project for subcommands that need it
are sometimes automagically derived from current git repo
and git-review configuration.


Subcommands
-----------

### gerda list

List open changeset for project in gerrit. Project and gerrit URL
is taken from current git repository -- it has to have gerrit
remote named 'gerrit' configured (like git-review does it).

If any command-line arguments are given they are interpreted as
regular expressions and only changesets with matching subjects
or topics are output.

Example:

    $ gerda list -p taskflow zook
    Some zookeeper persistence improvements/adjustments[zk] by Joshua Harlow  https://review.openstack.org/69814
      NEW  refs/changes/14/69814/11
    Add zookeeper job/jobboard impl[bp/job-reference-impl] by Joshua Harlow  https://review.openstack.org/54220
      NEW  V+1(jenkins)  refs/changes/20/54220/70
    Run zookeeper tests if localhost has a compat. zookeeper server[zk] by Joshua Harlow  https://review.openstack.org/70017
      NEW  V+1(jenkins)  refs/changes/17/70017/12


### gerda fetch

Fetch given (latest by default) patchset from gerrit. The commit
is put to FETCH_HEAD for farther use.

Example:
    $ gerda fetch https://review.openstack.org/63155
    INFO: Fetching change 'Message-oriented worker-based flow with kombu' from ref refs/changes/55/63155/38 into FETCH_HEAD

### gerda reset-to

Hard-reset current branch to the change from gerrit. Runs several
check before hard-reset to ensure that no changes are lost.

Example:

    $ gerda -v reset-to https://review.openstack.org/63155
    DEBUG: 1. Test current repository is clean
    DEBUG: Executing git status --porcelain
    DEBUG: 2. Check that current change were put to gerrit
    DEBUG: Executing git rev-parse HEAD
    DEBUG: Executing ssh -x -p 29418 imelnikov@review.openstack.org gerrit query --format=JSON 32e8c3da61b0dea2ffbbdd96f2c97c11fe8f3323
    DEBUG: 3. Check if we are resetting to branch
    DEBUG: Fetch changeset
    DEBUG: Executing ssh -x -p 29418 imelnikov@review.openstack.org gerrit query --format=JSON --current-patch-set change:63155
    INFO: Fetching change 'Message-oriented worker-based flow with kombu' from ref refs/changes/55/63155/38 into FETCH_HEAD
    DEBUG: Executing git fetch gerrit refs/changes/55/63155/38
    INFO: Doing HARD RESET to FETCH_HEAD
    DEBUG: Executing git reset --hard FETCH_HEAD
    HEAD is now at 32e8c3d Message-oriented worker-based flow with kombu

### gerda open

Open changeset in browser. Uses xdg-open by default. For example

    $ gerda open refs/changes/55/63155/38

will open https://review.openstack.org/63155 in your default browser. It also works
with plain change numbers (gerda open 63155) and (possibly partial) SHA-1 commit ids
(gerda open 913bc37b), and without arguments (plain 'gerda open' will open
change for current HEAD if such change exists).

### gerda spam

Review several changes with same comment. Useful for mass rechecks.

### gerda ls-projects

List gerrit projects which names contain given substring, or
all of them if no substring given.

Example:

    $ gerda ls-projects cinder
    openstack/cinder
    openstack/python-cinderclient
    stackforge/puppet-cinder

### gerda license

Prints gerda licensing information

### gerda dump-config

Print current configuration. Useful to see which gerrit instance
and project gerda will use by default

Example:

    $ gerda dump-config
    host: review.openstack.org
    port: 29418
    project: None
    user: imelnikov
