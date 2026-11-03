# Lab 01 — Filesystem Exploration

## Objective

Explore the Linux Filesystem Hierarchy Standard (FHS) and reinforce the theoretical concepts learned through practical exercises.

The goal is to understand how Linux organizes its directories, where system files are stored, and how to investigate filesystem resources using command-line tools.

## Environment

- **OS:** CentOS
- **Machine:** Virtual Machine
- **Shell:** Bash

---

## Exercises

### 1. Explore the Root Directory

**Objective:** Explore the root directory (`/`), list its contents, and identify at least 10 directories and their purposes.

#### Navigation

First, navigate to the root directory:

```bash
cd /
```

*Screenshot: Navigating to the root directory.*

#### Listing files and directories

Use the `ls` command with the following flags:

```bash
ls -lah
```

- `-l`: Displays detailed information about files and directories.
- `-a`: Displays all entries, including hidden files.
- `-h`: Displays file sizes in a human-readable format.

*Screenshot: Root directory listing.*

#### Identified Directories

| Directory | Purpose |
|---|---|
| `/boot` | Contains files required to boot the operating system, such as the kernel and bootloader files. |
| `/etc` | Stores system-wide configuration files and application configurations. |
| `/bin` | Contains essential system command binaries. |
| `/sbin` | Contains system administration and maintenance binaries. |
| `/media` | Common mount point for removable media. |
| `/root` | Home directory of the root user. |
| `/home` | Stores the personal directories of regular users. |
| `/var` | Stores variable data generated during system operation, such as logs, caches, and application data. |
| `/proc` | A virtual filesystem that exposes information about running processes and kernel resources. |
| `/srv` | Stores site-specific data served by the system and its services. |

---

### 2. Navigate Through the Filesystem

**Objective:** Explore `/etc`, `/var/log`, and `/home`, identify relevant files and directories, and understand their purposes.

#### Exploring `/etc`

Navigate to `/etc`:

```bash
cd /etc
```

List its contents:

```bash
ls
```

*Screenshot: Contents of `/etc`.*

#### Identified Files and Directories

- **`/etc/yum`**: Contains configuration files related to YUM, a package management tool used by CentOS.
- **`/etc/ssh`**: Contains SSH configuration files, including settings for the SSH server and client.

#### Exploring `/var/log`

Navigate to `/var/log`:

```bash
cd /var/log
```

List its contents:

```bash
ls
```

*Screenshot: Contents of `/var/log`.*

#### Identified Files

- **`boot.log-20261029`**: A boot log file containing information about the system startup process on October 29, 2026.

To inspect the file contents, use:

```bash
cat boot.log-20261029
```

*Screenshot: Displaying boot log contents.*

#### Exploring `/home`

Navigate to `/home`:

```bash
cd /home
```

List its contents:

```bash
ls
```

*Screenshot: Contents of `/home`.*

#### Identified Directories

- **`lfcandido`**: The home directory of the regular user `lfcandido`. It stores the user's personal files and user-specific data.

---

### 3. Find Configuration Files

**Objective:** Locate the `hostname`, `hosts`, and `fstab` configuration files and understand their purposes.

Since these are system configuration files, they are located in `/etc`.

The following files were identified:

```bash
ls -l /etc/hostname /etc/hosts /etc/fstab
```

To understand their purposes, I used the `man` command to access their manual pages.

```bash
man fstab
man hostname
man hosts
```

#### Identified Files

**`/etc/fstab` — Filesystem Table**

Contains descriptive information about filesystems that can be mounted by the system.

It defines filesystems, their mount points, filesystem types, and mount options. It is commonly used to configure filesystems that should be mounted during system startup.

**`/etc/hostname` — System Hostname**

Contains the system's hostname.

The hostname identifies the machine within the system and network environment.

**`/etc/hosts` — Local Hostname Resolution**

A local configuration file that associates IP addresses with hostnames.

It allows the system to resolve specific hostnames locally without necessarily querying a DNS server.

---

### 4. Explore System Binaries

**Objective:** Locate the `ls`, `bash`, `grep`, and `find` executables and identify their directories.

Since these are executable system commands, I initially investigated the `/bin` and `/sbin` directories.

The following commands were used:

```bash
which ls
which bash
which grep
which find
```

*Screenshots: Locating system binaries.*

#### Results

All four executables were found under `/bin` on the investigated system.

This means they are available through the system's command search path and can be executed by regular users, subject to their permissions.

---

### 5. Investigate Symbolic Links

**Objective:** Determine whether `/bin`, `/sbin`, and `/lib` are actual directories or symbolic links.

First, navigate to the root directory:

```bash
cd /
```

Then inspect the directory entries:

```bash
ls -lah
```

*Screenshots: Inspecting symbolic links.*

#### Results

After analyzing the listing, I identified that all three paths are symbolic links:

```text
/bin  -> usr/bin
/sbin -> usr/sbin
/lib  -> usr/lib
```

This filesystem organization is part of the merged `/usr` approach used by many modern Linux distributions.

Instead of maintaining separate directory trees, these paths point to their corresponding locations inside `/usr`.

---

### 6. Explore Virtual Filesystems

**Objective:** Investigate `/proc`, `/sys`, and `/run` and understand how they expose system and kernel information.

#### Exploring `/proc`

The `/proc` directory is a virtual filesystem that exposes information about running processes and kernel resources.

First, navigate to the directory and inspect its contents:

```bash
cd /proc
ls -lah
```

*Screenshot: Contents of `/proc`.*

One interesting detail is that directories named with numbers represent process IDs (PIDs).

Each process directory contains files with information about the corresponding running process.

For example, I investigated process `2141`:

```bash
cd /proc/2141
ls -lah
```

*Screenshots: Exploring process information.*

The files inside this directory expose information about process `2141`, such as its command-line arguments, environment, memory maps, and file descriptors.

#### Exploring `/sys`

The `/sys` directory, also known as **sysfs**, is a virtual filesystem provided by the Linux kernel.

It exposes information about hardware devices, device drivers, and kernel subsystems through a structured filesystem interface.

*Screenshot: Contents of `/sys`.*

#### Main Subdirectories in `/sys`

| Directory | Purpose |
|---|---|
| `class/` | Groups devices by functional category, such as network interfaces and input devices. |
| `block/` | Contains information about block devices, such as disks and partitions. |
| `bus/` | Exposes information about system buses, such as PCI and USB. |
| `devices/` | Represents the system's device hierarchy. |
| `dev/` | Contains links that associate device numbers with their corresponding device objects. |
| `firmware/` | Exposes information and interfaces related to system firmware. |
| `fs/` | Contains interfaces related to filesystem subsystems. |
| `kernel/` | Exposes kernel-related attributes and configuration interfaces. |
| `module/` | Provides information about loaded kernel modules. |

#### Exploring `/run`

The `/run` directory stores volatile runtime data generated by the system and its services.

It is generally stored in memory using a temporary filesystem (usually `tmpfs`) and is cleared during reboot.

*Screenshot: Contents of `/run`.*

Unlike `/tmp`, which is intended for general temporary files, `/run` is specifically designed for runtime information that services and processes need while the system is running.

A common use case is storing PID files, which contain process identifiers used by applications and services.

---

### 7. Analyze Disk Usage

**Objective:** Investigate filesystem capacity and identify which directories consume the most disk space.

#### Checking filesystem usage

Before investigating individual directories, I used the `df` command to inspect mounted filesystems and their disk usage.

```bash
df -h
```

The `-h` flag displays sizes in a human-readable format.

This command provides information about:

- Total filesystem capacity.
- Used space.
- Available space.
- Usage percentage.
- Mount points.

*Screenshot: Filesystem usage.*

#### Investigating directory sizes

To identify which directories consume the most space, I used the `du` command:

```bash
sudo du -sh /var /usr /home /opt
```

- `-s`: Displays a summary for each specified directory.
- `-h`: Displays sizes in a human-readable format.

To investigate all directories under the root directory and sort them by size:

```bash
sudo du -sh /* 2>/dev/null | sort -h
```

- `/*`: Matches entries directly under the root directory.
- `2>/dev/null`: Redirects error messages to `/dev/null`.
- `sort -h`: Sorts sizes according to their human-readable values, from smallest to largest.

*Screenshots: Directory disk usage analysis.*

#### Results

The investigation allowed me to identify which directories were consuming the most disk space.

This is useful when troubleshooting disk space issues and monitoring storage consumption.

---

## What I Learned

Throughout this laboratory, I gained practical experience exploring the Linux filesystem hierarchy and investigating how the operating system organizes its files and resources.

I learned how to navigate the filesystem, locate configuration files, identify system binaries, inspect symbolic links, and explore virtual filesystems such as `/proc`, `/sys`, and `/run`.

I also practiced using tools such as `ls`, `cd`, `cat`, `man`, `which`, `df`, and `du` to investigate the system.

One of the most interesting discoveries was understanding how virtual filesystems expose kernel and process information without storing that information as regular files on disk.

Finally, I learned how to analyze disk usage and identify directories that consume the most storage space.

These exercises improved my ability to navigate Linux systems, investigate system behavior, and troubleshoot common administration tasks.

## References

- **Linux Professional Institute (LPI):** [Linux Filesystem Hierarchy](https://learning.lpi.org/pt/learning-materials/101-500/104/104.7/104.7_01/)
- **Filesystem Hierarchy Standard:** [FHS Specification](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)
- **Linux Training:** Formação Linux Completa — Mateus Muller ([Udemy](https://www.udemy.com/course/formacao-linux-completa-do-basico-ao-avancado/))