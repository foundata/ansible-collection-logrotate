# Ansible role: `foundata.logrotate.run`

The `foundata.logrotate.run` Ansible role (part of the `foundata.logrotate` Ansible collection). It provides automated management of [logrotate](https://github.com/logrotate/logrotate) configuration.




## Table of contents<a id="toc"></a>

- [Features](#features)
- [Example playbooks, using this role](#examples)
- [Supported tags](#tags)<!-- ANSIBLE DOCSMITH TOC START -->
- [Role variables](#variables)
  - [`run_logrotate_state`](#variable-run_logrotate_state)
  - [`run_logrotate_autoupgrade`](#variable-run_logrotate_autoupgrade)
  - [`run_logrotate_service_state`](#variable-run_logrotate_service_state)
  - [`run_logrotate_config_defaults`](#variable-run_logrotate_config_defaults)
  - [`run_logrotate_config_defaults_dropin_file_name`](#variable-run_logrotate_config_defaults_dropin_file_name)
  - [`run_logrotate_config_sections`](#variable-run_logrotate_config_sections)
  - [`run_logrotate_timer_manage`](#variable-run_logrotate_timer_manage)
  - [`run_logrotate_timer_settings`](#variable-run_logrotate_timer_settings)
<!-- ANSIBLE DOCSMITH TOC END -->
- [Dependencies](#dependencies)
- [Compatibility](#compatibility)
- [External requirements](#requirements)



## Features<a id="features"></a>

* Preserve distribution defaults by managing only drop-in configuration files in `/etc/logrotate.d/`:
  * ensuring compatibility across a wide range of distributions
  * including openSUSE-specific behavior.
* Simple rule definition:
  * Optional baseline defaults
  * Log rotation with a simple dictionary per application.



## Example playbooks, using this role<a id="examples"></a>

Basic installation with your operating system's defaults and automatic upgrade on each Ansible run:

```yaml
---

- name: "Initialize the foundata.logrotate.run role"
  hosts: localhost
  gather_facts: false
  tasks:

    - name: "Trigger invocation of the foundata.logrotate.run role"
      ansible.builtin.include_role:
        name: "foundata.logrotate.run"
      vars:
        run_logrotate_autoupgrade: true
```


To define global logrotate defaults and application-specific rotation rules—resulting in configurations such as:

```
rotate 5
weekly
mail recipient@example.org
```

and:

```
/var/log/my_app/*.log "/var/log/with space/"*".log" {  # Path(s) to rotate, automatic quoting preserving globs
    daily                   # Rotation frequency
    rotate 14               # Keep 14 rotated files
    compress                # Compress rotated files
    delaycompress           # Wait one cycle before compressing
    missingok               # Don't error if file missing
    notifempty              # Don't rotate empty files
    create 0640 root adm    # Create new file with these permissions
    sharedscripts           # Run scripts once, not per-file
    postrotate              # Script block
        systemctl reload my-app > /dev/null 2>&1 || true
    endscript
}
```

the following playbook can be used:

```yaml
---

- name: "Initialize the foundata.logrotate.run role"
  hosts: localhost
  gather_facts: false
  tasks:

    - name: "Trigger invocation of the foundata.logrotate.run role"
      ansible.builtin.include_role:
        name: "foundata.logrotate.run"
      vars:

        # Creates /etc/logrotate.d/00-defaults. Optional baseline behavior for
        # all rotation sections and may be overridden by per-section directives
        # defined in run_logrotate_config_sections
        run_logrotate_config_defaults:
          rotate: 5
          weekly: true
          mail: "recipient@example.org"

        run_logrotate_config_sections:
          my_app:                              # Creates /etc/logrotate.d/my_app
            path:                              # Path line(s)
              - "/var/log/my_app/*.log"
              - "/var/log/with space/*.log"
            daily: true
            rotate: 14
            compress: true
            delaycompress: true
            missingok: true
            notifempty: true
            create: "0640 root adm"
            sharedscripts: true
            postrotate: |
              systemctl reload my_app > /dev/null 2>&1 || true
```

Customizing the systemd timer that triggers logrotate (e.g. changing the schedule or randomized delay):

```yaml
---
- name: "Initialize the foundata.logrotate.run role"
  hosts: localhost
  gather_facts: false
  tasks:

    - name: "Trigger invocation of the foundata.logrotate.run role"
      ansible.builtin.include_role:
        name: "foundata.logrotate.run"
      vars:

        # Creates /etc/systemd/system/logrotate.timer.d/00-managed.conf as a
        # systemd drop-in to override the platform's default timer unit settings.
        run_logrotate_timer_manage: true
        run_logrotate_timer_settings:
          RandomizedDelaySec: "30m"
```

Uninstall:

```yaml
---

- name: "Initialize the foundata.logrotate.run role"
  hosts: localhost
  gather_facts: false
  tasks:

    - name: "Trigger invocation of the foundata.logrotate.run role"
      ansible.builtin.include_role:
        name: "foundata.logrotate.run"
      vars:
        run_logrotate_state: "absent"
```



## Supported tags<a id="tags"></a>

It might be useful and faster to only call parts of the role by using tags:

- `run_logrotate_setup`: Manage basic resources, such as packages or service users.
- `run_logrotate_config`: Manage settings, such as adapting or creating configuration files.

There are also tags usually not meant to be called directly but listed for the sake of completeness** and edge cases:

- `run_logrotate_always`, `always`: Tasks needed by the role itself for internal role setup and the Ansible environment.


<!-- ANSIBLE DOCSMITH MAIN START -->

## Role variables<a id="variables"></a>

Main entry point for the foundata.logrotate.run role

The following variables can be configured for this role:

| Variable | Type | Required | Default | Description (abstract) |
|----------|------|----------|---------|------------------------|
| `run_logrotate_state` | `str` | No | `"present"` | Determines whether the managed resources should be `present` or `absent`.<br><br>`present` ensures that required components, such as software packages, are installed and configured.<br><br>`absent` reverts changes as much as possible, such as […](#variable-run_logrotate_state) |
| `run_logrotate_autoupgrade` | `bool` | No | `false` | If set to `true`, all managed packages will be upgraded during each Ansible run (e.g., when the package provider detects a newer version than the currently installed one). |
| `run_logrotate_service_state` | `str` | No | `"enabled"` | Defines the status of the service(s) the role manages. For logrotate this is the systemd `logrotate.timer` unit that triggers periodic log rotation; the term "service" is used generically and also covers such timer units.<br><br>Possible […](#variable-run_logrotate_service_state) |
| `run_logrotate_config_defaults` | `dict` | No | `{}` | Global logrotate default directives for all sections.<br><br>This dictionary defines logrotate directives that apply globally and are written to the drop-in file specified by `run_logrotate_config_defaults_dropin_file_name` in […](#variable-run_logrotate_config_defaults) |
| `run_logrotate_config_defaults_dropin_file_name` | `str` | No | `"00-defaults"` | Filename of the drop-in configuration file placed in `/logrotate.d/` to define global defaults. Defaults to `00-defaults`. The `00-` prefix ensures the file is loaded early, allowing its settings be overridden by later configuration files.<br><br>If […](#variable-run_logrotate_config_defaults_dropin_file_name) |
| `run_logrotate_config_sections` | `dict` | No | `{}` | Log rotation section definitions. Each section creates a drop-in configuration file in `/logrotate.d/`.<br><br>Dictionary structure:<br><br>- Keys: Section name (will create `/logrotate.d/`) - Values: Dictionary containing path(s) and rotation […](#variable-run_logrotate_config_sections) |
| `run_logrotate_timer_manage` | `bool` | No | `true` | Controls whether the role manages the automatic logrotate scheduling.<br><br>When set to `true` (the default), the role configures the systemd timer unit for logrotate according to `run_logrotate_timer_settings`.<br><br>Note: This feature currently […](#variable-run_logrotate_timer_manage) |
| `run_logrotate_timer_settings` | `dict` | No | `{}` | Configuration for the systemd timer that triggers logrotate execution. This dictionary controls when and how often logrotate runs.<br><br>These settings map to systemd timer unit directives and are applied via a drop-in override file. They take […](#variable-run_logrotate_timer_settings) |

### `run_logrotate_state`<a id="variable-run_logrotate_state"></a>

[*⇑ Back to ToC ⇑*](#toc)

Determines whether the managed resources should be `present` or `absent`.

`present` ensures that required components, such as software packages, are
installed and configured.

`absent` reverts changes as much as possible, such as removing packages,
deleting created users, stopping services, restoring modified settings, …

- **Type**: `str`
- **Required**: No
- **Default**: `"present"`
- **Choices**: `present`, `absent`



### `run_logrotate_autoupgrade`<a id="variable-run_logrotate_autoupgrade"></a>

[*⇑ Back to ToC ⇑*](#toc)

If set to `true`, all managed packages will be upgraded during each Ansible
run (e.g., when the package provider detects a newer version than the
currently installed one).

- **Type**: `bool`
- **Required**: No
- **Default**: `false`



### `run_logrotate_service_state`<a id="variable-run_logrotate_service_state"></a>

[*⇑ Back to ToC ⇑*](#toc)

Defines the status of the service(s) the role manages. For logrotate this is
the systemd `logrotate.timer` unit that triggers periodic log rotation; the
term "service" is used generically and also covers such timer units.

Possible values:

- `enabled`: Service is running and will start automatically at boot.
- `disabled`: Service is stopped and will not start automatically at boot.
- `running`: Service is running but will not start automatically at boot.
  This can be used to start a service on the first Ansible run without
  enabling it for boot.
- `unmanaged`: Service will not start at boot, and Ansible will not
  manage its running state. This is primarily useful when services are
  monitored and managed by systems other than Ansible.

The singular form ("service") is used for simplicity. However, the defined
status applies to all services if multiple are being managed by this role.

Note: This is only effective on systemd-managed systems and when
`run_logrotate_state` is `present`. It controls the timer's enablement and
run state; the timer's schedule is configured separately via
`run_logrotate_timer_manage` and `run_logrotate_timer_settings`.

- **Type**: `str`
- **Required**: No
- **Default**: `"enabled"`
- **Choices**: `enabled`, `disabled`, `running`, `unmanaged`



### `run_logrotate_config_defaults`<a id="variable-run_logrotate_config_defaults"></a>

[*⇑ Back to ToC ⇑*](#toc)

Global logrotate default directives for all sections.

This dictionary defines logrotate directives that apply globally and are
written to the drop-in file specified by
`run_logrotate_config_defaults_dropin_file_name` in
`<config dir>/logrotate.d/`.

All standard logrotate directives may be used as keys. Use `true` as the value
for directives that take no arguments. No log file paths must be specified
here.

These defaults provide baseline behavior for all rotation sections and may be
overridden by later drop-in configuration files or by per-section directives
defined in `run_logrotate_config_sections`.

Using a drop-in for also for global defaults avoids modifying
`/etc/logrotate.conf` and provides a safe way to customize or replace platform
defaults.

- **Type**: `dict`
- **Required**: No
- **Default**: `{}`



### `run_logrotate_config_defaults_dropin_file_name`<a id="variable-run_logrotate_config_defaults_dropin_file_name"></a>

[*⇑ Back to ToC ⇑*](#toc)

Filename of the drop-in configuration file placed in
`<config dir>/logrotate.d/` to define global defaults. Defaults to
`00-defaults`. The `00-` prefix ensures the file is loaded early, allowing
its settings be overridden by later configuration files.

If a non-default filename is used, any existing
`<config dir>/logrotate.d/00-defaults` from previous Ansible runs will be
removed automatically to prevent conflicts.

- **Type**: `str`
- **Required**: No
- **Default**: `"00-defaults"`



### `run_logrotate_config_sections`<a id="variable-run_logrotate_config_sections"></a>

[*⇑ Back to ToC ⇑*](#toc)

Log rotation section definitions. Each section creates a drop-in configuration
file in `<config dir>/logrotate.d/`.

Dictionary structure:

- Keys: Section name (will create `<config dir>/logrotate.d/<name>`)
- Values: Dictionary containing path(s) and rotation directives. All standard
  logrotate directives may be used as keys.

Required keys per rule:

- `path`: Log file path(s) or glob to rotate. Can be a string (single path or
  glob) or a list of paths/globs. Paths are automatically quoted when needed
  without preventing glob expansion. `paths` can be used as alias.

Special cases:

- Use `true` as the value for logrotate directives that take no arguments.
- There is a `content` key, allowing you to put raw lines for the drop-in file
  as the value without any special treatment. This is useful if you already have
  existing configuration and do not want to convert their definitions into YAML:
  ```yaml
  my-complex-logrotate:
    content: |
      /var/log/app1/*.log "/var/log/app2 space/"*".log" {
        daily
        rotate 14
        compress
        postrotate
          systemctl reload my-app > /dev/null 2>&1 || true
        endscript
      }

Example:

```yaml
my_app:
  path:
    - "/var/log/app1/*.log"
    - "/var/log/app2 space/*.log"
  daily: true                  # Rotation frequency
  rotate: 14                   # Keep 14 rotated files
  compress: true               # Compress rotated files
  delaycompress: true          # Wait one cycle before compressing
  missingok: true              # Don't error if file missing
  notifempty: true             # Don't rotate empty files
  create: "0640 www-data adm"  # Create new file with these permissions
  sharedscripts: true          # Run scripts once, not per-file
  postrotate: |                # Script block
    systemctl reload my-app > /dev/null 2>&1 || true
```

will create `logrotate.d/my_app` with the content:

```
/var/log/app1/*.log "/var/log/app2 space/"*".log" {
  daily
  rotate 14
  compress
  delaycompress
  missingok
  notifempty
  create 0640 www-data adm
  sharedscripts
  postrotate
    systemctl reload my-app > /dev/null 2>&1 || true
  endscript
}
```

The official documentation provides general configuration guidance:

- [`man logrotate`](https://manpages.debian.org/testing/logrotate/logrotate.8.en.html)
- [Configuration file directives](https://manpages.debian.org/testing/logrotate/logrotate.8.en.html#CONFIGURATION_FILE_DIRECTIVES)

The role marks the resulting files with an ownership marker line and
automatically removes files of keys that were later renamed or removed.
Package-shipped and hand-made files are never touched.

- **Type**: `dict`
- **Required**: No
- **Default**: `{}`



### `run_logrotate_timer_manage`<a id="variable-run_logrotate_timer_manage"></a>

[*⇑ Back to ToC ⇑*](#toc)

Controls whether the role manages the automatic logrotate scheduling.

When set to `true` (the default), the role configures the systemd timer
unit for logrotate according to `run_logrotate_timer_settings`.

Note: This feature currently only supports `systemd` as the service
manager. Other service managers (e.g., SysVinit with cron) are not
supported. On non-systemd systems, a warning is displayed and the
platform's default scheduling mechanism remains unchanged.

- **Type**: `bool`
- **Required**: No
- **Default**: `true`



### `run_logrotate_timer_settings`<a id="variable-run_logrotate_timer_settings"></a>

[*⇑ Back to ToC ⇑*](#toc)

Configuration for the systemd timer that triggers logrotate execution.
This dictionary controls when and how often logrotate runs.

These settings map to systemd timer unit directives and are applied via
a drop-in override file. They take highest priority, overriding internal
defaults (see `__run_logrotate_timer_settings_defaults` in `vars/main.yml`)
and platform-specific overrides.

Use standard systemd `[Timer]` directives as keys with their corresponding
values.

Dictionary structure:

- Keys: Standard systemd `[Timer]` directives.
- Values: Corresponding configuration values. Common options include:
  - `OnCalendar`: Defines the schedule on which the logrotate timer is
     triggered. Defaults to `daily`. An (at least) daily schedule is
     recommended.
  - `RandomizedDelaySec`: Adds a random delay before execution to avoid
    simultaneous runs across systems.  Defaults to `1h`.
  - `Persistent`: Ensures missed timer runs are executed at the next boot
    if the system was powered off. Defaults to `true`. Recommended to keep
    enabled to ensure log rotation occurs even after the system was
    unavailable during a scheduled run.

Value contract for the six repeatable trigger directives (`OnCalendar`,
`OnActiveSec`, `OnBootSec`, `OnStartupSec`, `OnUnitActiveSec`,
`OnUnitInactiveSec`):

- They also accept a YAML list and render one assignment per entry, so
  several expressions of the same directive can be combined:
  ```yaml
  OnCalendar:
    - "Mon..Fri 03:00"
    - "Sat,Sun 05:00"
  ```
- An empty list drops the role's built-in default for that directive
  (e.g. to run a purely monotonic timer without a calendar schedule).
  At least one trigger must remain overall.
- The monotonic trigger directives (all except `OnCalendar`) also accept
  plain numbers, which systemd reads as seconds.
- The boolean event trigger directives `OnClockChange` and
  `OnTimezoneChange` are supported as well and take `true`/`false`
  only. A `true` value counts as a remaining trigger, so an
  event-only timer (all repeatable triggers dropped via empty lists)
  is a valid configuration.
- Anything else is rejected: mappings, booleans, `null` and empty strings
  for repeatable trigger directives, non-booleans for the event trigger
  directives, unusable list entries, and lists for non-repeatable
  directives (those take a single scalar).

Special cases:

- For boolean values, use `true`/`false` (these will be converted to strings
  by the role as needed).

Only effective when `run_logrotate_timer_manage` is `true`.

- **Type**: `dict`
- **Required**: No
- **Default**: `{}`




<!-- ANSIBLE DOCSMITH MAIN END -->

## Dependencies<a id="dependencies"></a>

See `dependencies` in [`meta/main.yml`](./meta/main.yml).



## Compatibility<a id="compatibility"></a>

See `min_ansible_version` in [`meta/main.yml`](./meta/main.yml) and `__run_logrotate_supported_platforms` in [`vars/main.yml`](./vars/main.yml).



## External requirements<a id="requirements"></a>

There are no special requirements not covered by Ansible itself.
