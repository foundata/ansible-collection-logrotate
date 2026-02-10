# Ansible role: `foundata.logrotate.run`

The `foundata.logrotate.run` Ansible role (part of the `foundata.logrotate` Ansible collection).



## Table of contents<a id="toc"></a>

- [Features](#features)
- [Example playbooks, using this role](#examples)
- [Supported tags](#tags)<!-- ANSIBLE DOCSMITH TOC START -->
- [Role variables](#variables)<!-- ANSIBLE DOCSMITH TOC END -->
- [Dependencies](#dependencies)
- [Compatibility](#compatibility)
- [External requirements](#requirements)



## Features<a id="features"></a>

* Preserve distribution defaults by managing only drop-in files in `/etc/logrotate.d/`.
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


To create a logrotate configuration like

```
rotate 5
weekly
mail recipient@example.org
```

and

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

the following playbook could be used:

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

See [`defaults/main.yml`](./defaults/main.yml) for all available role parameters and their description. [`vars/main.yml`](./vars/main.yml) contains internal variables you should not override (but their description might be interesting).

Additionally, there are variables read from other roles and/or the global scope (for example, host or group vars) as follows:

- None right now.

<!-- ANSIBLE DOCSMITH MAIN END -->

## Dependencies<a id="dependencies"></a>

See `dependencies` in [`meta/main.yml`](./meta/main.yml).



## Compatibility<a id="compatibility"></a>

See `min_ansible_version` in [`meta/main.yml`](./meta/main.yml) and `__run_logrotate_supported_platforms` in [`vars/main.yml`](./vars/main.yml).



## External requirements<a id="requirements"></a>

There are no special requirements not covered by Ansible itself.
