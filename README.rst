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


High level chroot configuration
===============================

Instead of running ``nspawn-runner chroot-create``, you can define a chroot
through an ansible playbook placed in ``/etc/nspawn-runner/`` or
``/var/lib/nspawn-runner``.

The name of the playbook, without extension, will be used for the chroot image
name, and made available to ``gitlab-runner``.

``nspawn-runner`` can parse ``vars:`` from the first element in the playbook
(see `issue #3`__), and read configuration from variables starting with
``nspawn_runner_``: this allows to configure chroot creation and layout in a
single place. Currently supported are:

__ https://github.com/Truelite/nspawn-runner/issues/3

* ``nspawn_runner_chroot_suite``: suite to use for debootstrap

If ``nspawn-runner chroot-create`` finds a matching playbook, it will get
creation defaults from it, and run the playbook to customize the chroot after
creation.

You can use ``nspawn-runner chroot-maintenance`` to run all playbooks on all
chroots. It will also chreate chroots if they don't exist in the file system.
You can schedule it to run periodically, to keep chroots up to date.

This also means that you can provision a new gitlab runner by copying over just
the ``.yaml`` files and running ``nspawn-runner chroot-maintenance`` once.


Dependencies
============

``PyYaml`` or ``ruamel.yaml`` are needed to parse playbooks.

``eatmydata`` is an optional dependency: if found, it is used to speed up image
creation.

``btrfs`` is an optional dependency: if ``/var/lib/nspawn-runner`` is on btrfs,
chroots are created as subvolumes, and running CIs will use
``systemd-nspawn``'s ``--ephemeral`` feature.


Copyright
=========

Copyright 2021 Truelite S.r.l.

This software is released under the GNU General Public License 3.0
