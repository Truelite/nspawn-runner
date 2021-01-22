=============
nspawn-runner
=============

Backend implementation for a custom gitlab runner based on systemd-nspawn.

A detailed discussion on the design and implementation details in at
https://www.enricozini.org/blog/2021/debian/gitlab-runners-with-nspawn

Usage
=====

Command line summary::

  usage: nspawn-runner [-h] [-v] [--debug]
                       {chroot-create,chroot-login,prepare,run,cleanup,gitlab-config,toml}
                       ...
  
  Manage systemd-nspawn machines for CI runs.
  
  positional arguments:
    {chroot-create,chroot-login,prepare,run,cleanup,gitlab-config,toml}
                          sub-command help
      chroot-create       create a chroot that serves as a base for ephemeral
                          machines
      chroot-login        enter the chroot to perform maintenance
      prepare             start an ephemeral system for a CI run
      run                 run a command inside a CI machine
      cleanup             cleanup a CI machine after it's run
      gitlab-config       configuration step for gitlab-runner
      toml                output the toml configuration for the custom runner
  
  optional arguments:
    -h, --help            show this help message and exit
    -v, --verbose         verbose output
    --debug               verbose output

Steps:

1. Use ``nspawn-runner chroot-create`` to setup a chroot.
2. Use ``nspawn-runner toml`` to output a configuration snippet for ``/etc/gitlab-runner/config.toml``.
3. ???
4. Profit!

You can use ``nspawn-runner chroot-login`` to enter the base chroot to customize it.

The ``gitlab-config``, ``prepare``, ``run``, ``cleanup`` commands match what
`gitlab-runner custom executors`__ expect. By default they take the run ID from
``$CUSTOM_ENV_CI_JOB_ID`` as passed by gitlab-runner, but you can also use
``--id`` to pass it manually. You can use this as a basis for a CI-like system
of your own.

__ https://docs.gitlab.com/runner/executors/custom.html


Copyright
=========

Copyright 2021 Truelite S.r.l.

This software is released under the GNU General Public License 3.0
