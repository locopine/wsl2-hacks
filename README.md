# wsl2-hacks
Useful snippets / tools for using WSL2 as a development environment

---

**Auto-start/services** (systemd)
1. Install deps

    ```shell
    $ sudo apt update
    $ sudo apt install dbus policykit-1 daemonize
    ```

2. Create a `.bash_profile` for the `root` user

    `$ sudo editor ~root/.bash_profile`
    
    Add the following, be sure to replace `<YOURUSER>` with your WSL2 Linux username

    ```sh
    # Start systemd if not started
    /usr/sbin/daemonize -l "${HOME}/.systemd.lock" /usr/bin/unshare -fp --mount-proc /lib/systemd/systemd 2>/dev/null
    # Enter systemd namespace
    exec /usr/bin/nsenter -t "$(pgrep -xo systemd)" -m -p /bin/login -p -f <YOURUSER>
    ```

3. Exit out of / close the WSL2 shell

    In a PowerShell terminal run:
    
    ```
    > wsl --shutdown
    > ubuntu1804.exe config --default-user root
    ```
    
4. Re-open WSL2

    You should be logged in as your normal user and `systemd` should be running
    
    You can test by running the following in WSL2:
    
    ```sh
    $ systemctl is-active dbus
    active
    ```

5. Create `/etc/rc.local` (optional)

    If you want to run certain commands when the WSL2 VM starts up, this is a useful file that's automatically ran by systemd.
    
    ```shell
    $ sudo touch /etc/rc.local
    $ sudo chmod +x /etc/rc.local
    $ sudo editor /etc/rc.local
    ```
    
    Add the following:
    ```sh
    #!/bin/sh -e
    
    # your commands here...
    
    exit 0
    ```

`/etc/rc.local` is only run on "boot", so only when you first access WSL2 (or it's shutdown due to inactivity/no-processes).
To test you can shutdown WSL via PowerShell/CMD `wsl --shutdown` then start it back up with `wsl`.

---

**Access localhost ports from Windows**

Many development servers default to binding to `127.0.0.1` or `localhost`. It can be cumbersome and frustrating to get it to bind to `0.0.0.0` to make it accessible via Windows using the IP of the WSL2 VM.

> Take a look at https://github.com/shayne/go-wsl2-host to have `wsl.local` automatically resolve to the WSL2 VM

To make these dev servers / ports accessible you can run the following commands, or add them to the `/etc/rc.local` if you have `systemd` running:

```shell
# /etc/rc.local runs as root by default
# if you run these yourself add 'sudo' to the beginning of each command

$ sysctl -w net.ipv4.conf.all.route_localnet=1
$ iptables -t nat -I PREROUTING -p tcp -j DNAT --to-destination 127.0.0.1 
```

---

**Increase `max_user_watches`**

If devtools are watching for file changes, the default is too low.

```
# /etc/rc.local runs as root by default
# if you run these yourself add 'sudo' to the beginning of each command

sysctl -w fs.inotify.max_user_watches=524288
```

---

**Open MS Terminal to home directory by default**

Open your MS Terminal configuration <kbd>Ctrl+,</kbd>

Find the `"commandLine":...` config for the WSL profile.

Change to something like:

```json
"commandline": "wsl.exe ~ -d Ubuntu-18.04",
```
