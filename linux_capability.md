> Source: https://man7.org/linux/man-pages/man7/capabilities.7.html

| Capability                | Description                                                                                     |
|---------------------------|-------------------------------------------------------------------------------------------------|
| CAP_AUDIT_CONTROL         | Enable and disable kernel auditing; change auditing filter rules; retrieve auditing status.     |
| CAP_AUDIT_READ            | Allow reading the audit log via a multicast netlink socket.                                     |
| CAP_AUDIT_WRITE           | Write records to kernel auditing log.                                                           |
| CAP_BLOCK_SUSPEND         | Employ features that can block system suspend.                                                  |
| CAP_BPF                   | Employ privileged BPF operations.                                                               |
| CAP_CHECKPOINT_RESTORE    | Update /proc/sys/kernel/ns_last_pid; employ the set_tid feature of clone3(2); read map_files.  |
| CAP_CHOWN                 | Make arbitrary changes to file UIDs and GIDs.                                                   |
| CAP_DAC_OVERRIDE          | Bypass file read, write, and execute permission checks.                                         |
| CAP_DAC_READ_SEARCH       | Bypass file read permission checks and directory read and execute permission checks.            |
| CAP_FOWNER                | Bypass permission checks on operations that normally require the filesystem UID to match.      |
| CAP_FSETID                | Don't clear set-user-ID and set-group-ID mode bits when a file is modified.                     |
| CAP_IPC_LOCK              | Lock memory and allocate memory using huge pages.                                               |
| CAP_IPC_OWNER             | Bypass permission checks for operations on System V IPC objects.                                |
| CAP_KILL                  | Bypass permission checks for sending signals.                                                   |
| CAP_LEASE                 | Establish leases on arbitrary files.                                                            |
| CAP_LINUX_IMMUTABLE       | Set the FS_APPEND_FL and FS_IMMUTABLE_FL inode flags.                                           |
| CAP_MAC_ADMIN             | Allow MAC configuration or state changes.                                                       |
| CAP_MAC_OVERRIDE          | Override Mandatory Access Control (MAC).                                                        |
| CAP_MKNOD                 | Create special files using mknod(2).                                                            |
| CAP_NET_ADMIN             | Perform various network-related operations.                                                     |
| CAP_NET_BIND_SERVICE      | Bind a socket to Internet domain privileged ports.                                              |
| CAP_NET_BROADCAST         | Make socket broadcasts and listen to multicasts.                                                |
| CAP_NET_RAW               | Use RAW and PACKET sockets; bind to any address for transparent proxying.                       |
| CAP_PERFMON               | Employ various performance-monitoring mechanisms.                                               |
| CAP_SETGID                | Make arbitrary manipulations of process GIDs and supplementary GID list.                        |
| CAP_SETFCAP               | Set arbitrary capabilities on a file.                                                           |
| CAP_SETPCAP               | Add any capability from the calling thread's bounding set to its inheritable set.               |
| CAP_SETUID                | Make arbitrary manipulations of process UIDs.                                                   |
| CAP_SYS_ADMIN             | Perform a range of system administration operations.                                            |
| CAP_SYS_BOOT              | Use reboot(2) and kexec_load(2).                                                                |
| CAP_SYS_CHROOT            | Use chroot(2); change mount namespaces using setns(2).                                          |
| CAP_SYS_MODULE            | Load and unload kernel modules.                                                                 |
| CAP_SYS_NICE              | Lower the process nice value and change the nice value for arbitrary processes.                 |
| CAP_SYS_PACCT             | Use acct(2).                                                                                    |
| CAP_SYS_PTRACE            | Trace arbitrary processes using ptrace(2).                                                      |
| CAP_SYS_RAWIO             | Perform I/O port operations and access /proc/kcore.                                             |
| CAP_SYS_RESOURCE          | Use reserved space on ext2 filesystems; override disk quota limits.                             |
| CAP_SYS_TIME              | Set system clock and real-time (hardware) clock.                                                |
| CAP_SYS_TTY_CONFIG        | Use vhangup(2); employ various privileged ioctl(2) operations on virtual terminals.              |
| CAP_SYSLOG                | Perform privileged syslog(2) operations.                                                        |
| CAP_WAKE_ALARM            | Trigger something that will wake up the system.                                                 |
