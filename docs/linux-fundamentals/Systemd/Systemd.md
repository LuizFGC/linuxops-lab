# Systemd

## 1. SysVinit vs systemd

**SysVinit** was one of the traditional initialization systems used in Unix and Linux operating systems.

**systemd** was introduced later and replaced SysVinit in many modern Linux distributions.

The main difference is how they initialize the system and manage services.

```text
SysVinit → Runlevels + Initialization Scripts
systemd  → Targets + Units
```

## 2. What is SysVinit?

SysVinit is a traditional initialization system responsible for starting the operating system and executing the scripts required to initialize system services.

Initialization scripts are traditionally stored in:

```bash
/etc/init.d/
```

SysVinit uses **Runlevels** to represent different system operating states.

Its initialization model is primarily based on scripts and sequential execution.

**Important:** SysVinit does not mean that Linux is incapable of multitasking or multiprocessing. Linux supports multiple processes and CPU cores regardless of the initialization system being used.

The main difference between SysVinit and systemd is how services, dependencies, and the initialization process are managed.

## 3. What is systemd?

systemd is a modern initialization system and service manager used by many Linux distributions.

It is normally the first user-space process started by the kernel:

```text
PID = 1
```

systemd is responsible for initializing the user-space environment and managing several system resources, including:

- Services
- Processes
- Logs
- Timers
- Mount points
- Devices
- Sessions
- Targets
- Service dependencies

Network management can also be handled by components from the systemd ecosystem, such as `systemd-networkd`, depending on the distribution and configuration.

systemd can start services in parallel when their dependencies and ordering requirements allow it.

## 4. Runlevels

Runlevels are a concept used by SysVinit to represent different system operating states.

Each runlevel is represented by a number.

Traditional examples:

| Runlevel | Description |
|---|---|
| 0 | System shutdown |
| 1 | Single-user mode / maintenance |
| 2 | Multi-user mode (varies by distribution) |
| 3 | Multi-user mode without graphical interface |
| 4 | User-defined (varies by distribution) |
| 5 | Multi-user mode with graphical interface |
| 6 | System reboot |

The main concept is:

```text
Runlevel = System operating state
```

## 5. Targets

In systemd, **Targets** are used to group units and represent specific system states.

They are conceptually similar to SysVinit Runlevels.

Instead of using numbers, systemd uses descriptive names.

Examples:

```text
poweroff.target
rescue.target
multi-user.target
graphical.target
reboot.target
```

Approximate equivalence:

| SysVinit | systemd |
|---|---|
| Runlevel 0 | poweroff.target |
| Runlevel 1 | rescue.target |
| Runlevel 3 | multi-user.target |
| Runlevel 5 | graphical.target |
| Runlevel 6 | reboot.target |

Targets can have dependencies on other units.

For example:

```text
graphical.target
       |
       v
Required and wanted units
       |
       v
Services and dependencies
```

Useful commands:

```bash
# Display the default target
systemctl get-default

# List available targets
systemctl list-units --type=target

# Display target dependencies
systemctl list-dependencies graphical.target
```

## 6. Units

systemd organizes the resources it manages through **Units**.

A Unit represents a resource or configuration object managed by systemd.

Main unit types:

| Unit type | Description |
|---|---|
| `.service` | System services |
| `.target` | Groups of units and system states |
| `.timer` | Scheduled tasks |
| `.mount` | Filesystem mount points |
| `.socket` | Socket-based activation |
| `.device` | Devices managed by systemd |

Example:

```text
nginx.service
```

This unit represents the Nginx service.

## 7. systemctl

`systemctl` is the main command-line utility used to manage systemd and its units.

### Check service status

```bash
systemctl status nginx
```

### Start a service

```bash
sudo systemctl start nginx
```

### Stop a service

```bash
sudo systemctl stop nginx
```

### Restart a service

```bash
sudo systemctl restart nginx
```

### Reload service configuration

```bash
sudo systemctl reload nginx
```

### Enable a service at boot

```bash
sudo systemctl enable nginx
```

### Disable a service at boot

```bash
sudo systemctl disable nginx
```

### Enable and start a service immediately

```bash
sudo systemctl enable --now nginx
```

It is important to understand the difference:

```text
start
→ Starts the service immediately.

enable
→ Configures the service to start automatically during boot.
```

Enabling a service does not necessarily start it immediately. Similarly, starting a service does not automatically enable it at boot.

## 8. journalctl

`journalctl` is used to query logs stored by `systemd-journald`.

It is particularly useful for troubleshooting service failures and boot problems.

### Display all available logs

```bash
journalctl
```

### Follow logs in real time

```bash
journalctl -f
```

### Display logs from a specific service

```bash
journalctl -u nginx
```

### Display the last 50 log entries

```bash
journalctl -u nginx -n 50
```

### Display logs from the current boot

```bash
journalctl -b
```

### Display logs from the previous boot

```bash
journalctl -b -1
```

### Display logs with higher priority

```bash
journalctl -p err
```

## 9. Important systemd Directories

### `/usr/lib/systemd/system/`

Contains system unit files provided by installed packages and the operating system.

Examples:

```text
/usr/lib/systemd/system/ssh.service
/usr/lib/systemd/system/nginx.service
```

It is not recommended to edit these files directly because package updates may overwrite local modifications.

### `/etc/systemd/system/`

Contains locally configured units and administrative customizations.

It is commonly used to:

- Create custom services.
- Configure unit overrides.
- Customize existing units.
- Define dependencies and startup behavior.

Example:

```text
/etc/systemd/system/myapp.service
```

**Best practice:** Use `/etc/systemd/system/` for local customizations instead of modifying vendor-provided unit files.

## 10. Service Unit Files

A `.service` file defines how systemd should manage a service.

Example:

```ini
[Unit]
Description=My Application
After=network.target

[Service]
ExecStart=/usr/bin/node /var/www/app/server.js
Restart=always

[Install]
WantedBy=multi-user.target
```

### `[Unit]`

Contains general information about the unit and its dependencies.

Example:

```ini
[Unit]
Description=My Application
After=network.target
```

- `Description`: Provides a description of the service.
- `After`: Defines startup ordering relative to another unit.

**Note:** `After=network.target` establishes ordering. It does not guarantee that network connectivity is available.

### `[Service]`

Defines how the service should be executed and managed.

Example:

```ini
[Service]
ExecStart=/usr/bin/node /var/www/app/server.js
Restart=always
```

- `ExecStart`: Defines the command used to start the service.
- `Restart`: Defines the service restart behavior.

### `[Install]`

Defines how the unit is connected to other units when `systemctl enable` is executed.

Example:

```ini
[Install]
WantedBy=multi-user.target
```

This configures the service to be started as part of `multi-user.target` when enabled.

## 11. daemon-reload

When a unit file is created or modified, systemd must reload its configuration.

Use:

```bash
sudo systemctl daemon-reload
```

After reloading, the service can be started or enabled:

```bash
sudo systemctl start myapp
```

Or:

```bash
sudo systemctl enable --now myapp
```

**Important:** `daemon-reload` does not restart all services. It makes systemd reload unit files and rebuild its dependency configuration.

If a service is already running, you may still need to restart it for changes to its execution configuration to take effect.

## 12. Useful systemd Commands

### Identify the PID 1 process

```bash
ps -p 1 -o pid,comm,args
```

### List loaded units

```bash
systemctl list-units
```

### List services

```bash
systemctl list-units --type=service
```

### List installed unit files

```bash
systemctl list-unit-files
```

### List targets

```bash
systemctl list-units --type=target
```

### Display the default target

```bash
systemctl get-default
```

### Display dependencies of a target

```bash
systemctl list-dependencies graphical.target
```

## 13. Summary

### SysVinit

```text
SysVinit
 |
 +-- Initialization scripts
 +-- /etc/init.d/
 +-- Runlevels
 +-- Traditional service startup
```

### systemd

```text
systemd
 |
 +-- PID 1
 +-- Units
 +-- Targets
 +-- Dependencies
 +-- Service management
 +-- systemd-journald
 +-- systemctl
 +-- journalctl
```

### Main Differences

| Concept | SysVinit | systemd |
|---|---|---|
| System states | Runlevels | Targets |
| Service configuration | Initialization scripts | Unit files |
| Service management | Service scripts | systemctl |
| Log management | Traditional log files and syslog | journald (among other logging options) |
| Dependencies | Primarily script ordering | Unit dependencies and ordering |
| Startup | Traditionally sequential | Supports parallel startup |

## 14. What I Learned

After completing this topic, answer the following questions:

- What is the main difference between SysVinit and systemd?

-> Systemd was introduced to replace the SysV that was one of the traditional initialization systems on linux, the mais diference is how they initialize and manage services, 
SysV works with RunLevels + initialization scripts and the Systemd Targets + units 

- What is a Runlevel?

-> RunLevel in a concept used to represent different system operating states, the states are represented by numbers

0	System shutdown

1	Single-user mode / maintenance

2	Multi-user mode (varies by distribution)

3	Multi-user mode without graphical interface

4	User-defined (varies by distribution)

5	Multi-user mode with graphical interface

6	System reboot


- What is a Target?

-> Targets are used by the systemd to group units and represent system sates similar to the Runlevels 
Instead of numbers systemd uses descriptive names 

* poweroff.target

* rescue.target

* multi-user.target

* graphical.target

* reboot.target



- What is a Unit?

-> A unit represents a resource that systemd can manage.
Every unit is described by a configuration file, and its type
is indicated by the file extension:

* .service  -> a service/daemon (e.g. sshd.service, nginx.service)
* .target   -> groups other units, represents a system state
* .mount    -> a filesystem mount point
* .socket   -> starts a service on demand when a connection arrives
* .timer    -> schedules tasks (alternative to cron)
* .device   -> a device detected by the kernel/udev
* .path     -> activates a unit when a file/directory changes

-> Unit files are stored in 3 main places (lowest to highest priority):

* /usr/lib/systemd/system/   -> units installed by packages (don't edit)

* /run/systemd/system/       -> runtime units (temporary)

* /etc/systemd/system/       -> units created/changed by the administrator

(highest priority, overrides the others)

- What are they used for?

-> Both define which services and resources are active in the system,
allowing the administrator to control how the system behaves.

* Define how the system boots
  e.g. servers usually boot into multi-user.target (no GUI),
  saving memory and CPU, while desktops use graphical.target

* Maintenance and recovery
  rescue.target / emergency.target start only the essentials,
  making it possible to fix problems such as a forgotten root password,
  a corrupted filesystem or a service that breaks the boot

* Switch system states without rebooting
  e.g. stop the graphical interface temporarily to do maintenance

* Shut down or reboot the system in a controlled way
  (poweroff.target / reboot.target)

* Organize service dependencies (targets only)
  systemd can start services in parallel and in the right order,
  and administrators can create custom targets

- What is the difference between `systemctl start` and `systemctl enable`?

-> Start - starts the service immediately, enable - configures the service to start automatically during boot 

- What is the purpose of `journalctl`?

-> The journalctl is used to see the systemctl logs, useful for troubleshooting service failures

- What is the difference between `/usr/lib/systemd/system/` and `/etc/systemd/system/`?

-> /usr/lib/systemd/system/ -> units installed by packages, /etc/systemd/system/ -> units created/changed by the administrator

- What does `daemon-reload` do?

-> The daemon-reaload is used to reload systemd configuration when a unit file is created or modified

- What is the purpose of a `.service` file?

-> The .service file defines how systemd should manage a service 

- How can systemd start services in parallel?

-> systemd reads all unit files and builds a dependency graph.
Units that don't depend on each other are started at the same time,
instead of one after another like SysVinit scripts.

## 15. References

- [systemd Documentation](https://www.freedesktop.org/software/systemd/man/)
- [Red Hat Enterprise Linux Documentation](https://docs.redhat.com/)
- [systemd GitHub Repository](https://github.com/systemd/systemd)
