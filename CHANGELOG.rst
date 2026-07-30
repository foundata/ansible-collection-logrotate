===================================================
foundata.logrotate Ansible collection Release Notes
===================================================

.. contents:: Topics

v1.3.0
======

Release Summary
---------------

Release Date: 2026-07-30

Maintenance and bugfix release.

Minor Changes
-------------

- The Molecule ``default`` scenario now selects the test backend per platform via a ``type`` key: ``podman`` (container, the default when omitted) or ``libvirt`` (QEMU/KVM virtual machine from a vendor cloud image via a session libvirt daemon, without root privileges). VM platforms allow tests containers cannot cover; commented ``libvirt`` alternates for every platform are included in ``molecule.yml``. ``molecule login`` now works through a per-instance login command for both backends. See ``extensions/molecule/README.md`` for requirements and usage.
- ``run`` role - Files created by the role in the shared drop-in directory now start with an ownership marker line. Files carrying the marker whose ``run_logrotate_config_sections`` key was renamed or removed (or whose defaults drop-in filename changed) are now removed automatically; package-shipped and hand-made files are never touched. Files created by earlier releases do not carry the marker yet: still-declared ones get it on the next run, already orphaned ones must be removed manually once.
- ``run`` role - The keys of ``run_logrotate_config_sections`` are now checked to yield unique, non-empty drop-in filenames. They are normalized to lowercase with every character outside ``a-z0-9._`` replaced, so keys that differ only in case or in such characters (for example ``My App`` and ``my_app``, or ``a-b`` and ``a_b``) previously ended up in the same file and silently overwrote each other, and a key consisting only of such characters collapsed to an empty filename. Such configurations now fail with a message naming the normalized filenames instead of losing a section.
- ``run`` role - ``run_logrotate_timer_settings`` now accepts a YAML list for the repeatable systemd trigger directives (``OnCalendar``, ``OnActiveSec``, ``OnBootSec``, ``OnStartupSec``, ``OnUnitActiveSec``, ``OnUnitInactiveSec``), rendering one assignment per entry so several expressions of the same directive can be combined. An empty list drops the role's built-in default for that directive (e.g. to run a purely monotonic timer without a calendar schedule); at least one trigger must remain. The monotonic trigger directives also accept plain numbers (seconds). The boolean event trigger directives ``OnClockChange`` and ``OnTimezoneChange`` are supported and count as a trigger when ``true``, so an event-only timer is a valid configuration. Unsupported values - mappings, booleans, ``null`` or empty strings for repeatable trigger directives, non-booleans for the event trigger directives, unusable list entries, and lists for non-repeatable directives - are now rejected with an explanatory message instead of being silently rendered into an invalid unit.

Security Fixes
--------------

- ``run`` role - ``run_logrotate_config_defaults_dropin_file_name`` is now validated to be a plain filename. A value containing path separators (such as ``../logrotate.conf``) could previously make the role write to or remove files outside of the ``logrotate.d`` configuration directory with root privileges.

Bugfixes
--------

- The comment written into neutralized distribution config files contained a stray double quote in the Debian hint (``dpkg -S '<file>'"``), so the suggested command could not be copied and pasted as-is. The quote is removed.
- ``run`` role - A previously managed systemd timer drop-in is now removed when ``run_logrotate_timer_manage`` is set back to ``false`` (on systemd hosts), so an old role-managed schedule no longer keeps applying silently. Before, the drop-in was only removed on a full uninstall.
- ``run`` role - Platform-specific task files are now guaranteed to run before the shared default tasks. The former single include loop did not preserve that order with several platforms in one play: Ansible batches the includes across hosts and the insertion order depends on when results arrive (non-deterministic), so default tasks could run before platform-specific ones. The includes are now two sequential tasks, which is a hard ordering barrier.
- ``run`` role - The documentation of the ``unmanaged`` service state falsely claimed the service "will not start at boot". The role leaves the service completely alone in this state: both the running state and the boot (enablement) state stay exactly as they are. The description now documents the real behavior.
- ``run`` role - The systemd timer drop-in now emits exactly one reset line before the configured trigger directives, so a configured ``OnCalendar`` in ``run_logrotate_timer_settings`` replaces the schedule of the vendor ``logrotate.timer`` instead of accumulating with it. Such directives add up across a unit and its drop-ins (see ``systemd.unit(5)``), so the timer previously kept the vendor ``OnCalendar=daily`` entry next to the configured one and ran on both schedules. Setting a less frequent schedule than the vendor default therefore had no effect. All trigger directives (``OnCalendar``, ``OnActiveSec``, ``OnBootSec``, ...) share a single trigger list, so one reset per directive would make each reset wipe the triggers rendered before it and only the last configured trigger would survive.
- ``run`` role - ``run_logrotate_state: "absent"`` now removes all section files created by the role (identified by the ownership marker). Before, only files derivable from the current ``run_logrotate_config_sections`` value were removed, leaving files of removed or renamed keys behind.

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
