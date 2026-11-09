# Lab 02 — Systemd

## Objective

Explore systemd, the initialization system and service manager used by
most modern Linux distributions, and reinforce the theoretical concepts
learned through practical exercises.

The goal is to understand how Linux boots and manages services, how
targets and units are organized, and how to create, control and
troubleshoot services using command-line tools such as systemctl,
journalctl and systemd-analyze.

## Environment

- **OS:** CentOS
- **Machine:** Virtual Machine
- **Shell:** Bash

---

## Exercises

1 — systemd as PID 1
Confirm which process is PID 1:

Look at the process tree and find which processes are children of systemd:

Check if /etc/init.d/ exists on your system and what’s inside. Are those scripts still used? (Hint: look for systemd-sysv-generator.)


2 — Targets

Show your default target, then list all active targets:

Run runlevel and compare the number with your default target using the equivalence table in your notes.

View what multi-user.target pulls in, and compare it with graphical.target:What does graphical.target have that multi-user.target doesn’t?

Look at the target file itself and find the Requires=, Wants= and After= lines:

(VM only) If you have a GUI, switch to text mode without rebooting, then back:

(VM only) Change the default target to multi-user.target, reboot, confirm you boot without GUI, then restore it.\

3 — Units

List all unit files and count how many of each type exist:

Find one example of each type: .service, .socket, .timer, .mount, .device, .path.

Pick ssh.service (or sshd.service) and answer: Where is its file? Which target is it WantedBy? Does it Restart?

Compare the contents of /usr/lib/systemd/system/ and /etc/systemd/system/. What kind of files do you find in /etc/systemd/system/? (Hint: look at the *.wants/ directories and what they contain — ls -l.

4 — start vs enable

Install nginx (sudo apt install nginx or sudo dnf install nginx), then:

Check both states at once:

Stop and disable it. Reboot. Is it running?
Only start it (don’t enable). Reboot. Is it running?
Only enable it (don’t start). Is it running now? After reboot?
Run sudo systemctl enable nginx and read the output carefully. What exactly did it create? Find that file with ls -l.


5 — Create your own service (main exercise)
Create a script /usr/local/bin/hello-loop.sh:

Make it executable with sudo chmod +x.
2. Create /etc/systemd/system/hello.service with the three sections [Unit], [Service] and [Install]. It should start after the network, restart on failure and be wanted by multi-user.target.
3. Make systemd see the new file, then enable and start it in one command.
4. Check its status and confirm the messages appear in the logs.
5. Find the PID of the service and kill it with sudo kill -9 <PID>. What happens? Why?
6. Change Restart=on-failure to Restart=no, apply the change correctly, and repeat step 5. What’s different now?


6 — journalctl
Using your hello.service:

Show only its logs, then only the last 10 lines.
Follow its logs in real time (-f) while you restart it in another terminal.
Show logs from the current boot only, then from the previous boot.
Show only errors from the whole system (-p err). Pick one and investigate which unit produced it.
Show logs from the last 30 minutes:

Check how much disk space the journal uses: journalctl --disk-usage.


7 — daemon-reload and overrides
Edit /etc/systemd/system/hello.service (change the Description) and run systemctl status hello without daemon-reload. What warning appears?
Run daemon-reload. Is the service restarted? (Check the “Active: since...” time.)
Change the echo message inside the script. Do you need daemon-reload? Do you need restart? Explain why.
Use an override instead of editing the original file:

Add an environment variable (Environment=GREETING=Hi) and use it in the script. Find where systemd saved the override file.
5. Do the same for nginx.service and explain why overriding is better than editing the file in /usr/lib/systemd/system/.

8 — Timers (bonus)
List existing timers: systemctl list-timers.
Create backup.service (a oneshot that copies /etc/hostname to /tmp/backup-$(date +%s)) and backup.timer that runs it every 2 minutes.
Enable the timer (not the service), wait, and confirm the files appear in /tmp and in the journal.


9 — Parallel startup and boot analysis
See how long the boot took and which units were slowest:

See the chain of units that determined the boot time:

(GUI) Generate a boot chart and open it in a browser:

Look at the bars: which services started at the same time?

4. Use what you saw to answer the question in your notes: How can systemd start services in parallel? (Think about After=, Wants=/Requires=, and socket activation.)

10 — Troubleshooting challenge

Create broken.service with a mistake on purpose, for example ExecStart=/usr/bin/doesnotexist. Then, using only systemctl status and journalctl, find and fix the problem. Repeat with other mistakes: a relative path in ExecStart, a typo in a section name like [Servce], and a script without execute permission. Note what error message each one produces.