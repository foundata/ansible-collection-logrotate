===================================================
foundata.logrotate Ansible collection Release Notes
===================================================

.. contents:: Topics

v1.2.0
======

Release Summary
---------------

Release Date: 2026-05-31

Feature release.

Minor Changes
-------------

- ``run`` - Added the ``run_logrotate_service_state`` parameter (``enabled``/``disabled``/``running``/``unmanaged``, default ``enabled``) to explicitly manage the enablement and run state of the systemd ``logrotate.timer``. Previously the role relied on the timer's vendor preset and provided no way to disable it or leave it unmanaged.

v1.1.0
======

Release Summary
---------------

Release Date: 2026-05-10

Maintenance release.

Minor Changes
-------------

- Added Fedora 44 as supported platform for all collection roles and Molecule test scenarios.
- Added Ubuntu 26.04 LTS (Resolute Raccoon) as supported platform for all collection roles and Molecule test scenarios.

Removed Features (previously deprecated)
----------------------------------------

- Removed Fedora 42 support (End of Life, EOL) from collection roles and Molecule scenarios. The collection may still work on Fedora 42, but no testing or bugfixes will be provided. A warning will be displayed when used on unsupported platforms.

v1.0.0
======

Release Summary
---------------

Release Date: 2026-02-18

First public release, providing all functionality and files.
