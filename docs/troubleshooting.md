# Troubleshooting

## Overview

Every fault the lab produced, with its symptom, its cause, and the fix that worked. One table per build area. Each fix was verified on the machine before being written down.

The method behind all of them: change one thing, test, then change the next thing. The faults that took a whole evening were the ones where several settings changed at once.

## Proxmox Host

| Symptom | Cause | Fix |
| --- | --- | --- |
| First `apt update` fails, red banner saying no repository is enabled | A fresh install points at the paid enterprise repository | Disable `pve-enterprise`, add the No-Subscription repository, refresh until `TASK OK` |
| Initialize Disk with GPT grayed out | The disk still carries partitions, or Proxmox still holds the disk as a storage | Destroy the old Directory entry, remove the storage, Wipe Disk, then initialize |
| Graphical installer black screen | Installer video compatibility on old hardware | The Terminal UI entry in the installer menu. Same installer, text interface |
| Installer cannot find the hard disk | SATA mode not AHCI | Switch to AHCI in firmware setup and boot the installer again |
| Ventoy stick never boots on the Dell | BIOS-era firmware and Ventoy's loader do not get along | Rufus in MBR mode, worked first try |
| "No valid subscription" dialog on every login | A check in `proxmoxlib.js` served to the browser | Edit `checked_command` to call `orig_cmd();`, restart `pveproxy`, hard-refresh |
| Edit to `proxmoxlib.js` seems to do nothing | `pveproxy` not restarted, browser still holds the old file | `systemctl restart pveproxy.service`, then a hard refresh |

## Containers

| Symptom | Cause | Fix |
| --- | --- | --- |
| Console tab stays black on a running container | The web console emulates a screen the container may never write to | `pct enter 110` from the node shell. No password, straight to a root shell |
| Console tab black, Start button available | The container is stopped. A stopped container has nothing to print | Start it first |
| Template dropdown empty in the Create wizard | No template downloaded yet | Download one to `local` from the storage page. The fix is not in the wizard |
| 1 TB drive shows 7.8 GB inside the container | The GUI mount point carved an 8 GiB image instead of passing the drive | Bind mount from the node shell: `pct set 110 -mp0 /mnt/pve/TB-mediaDrive,mp=/mnt/media` |
| Third-party repository ended up on the hypervisor | Commands run in the node Shell instead of inside the container | `rm` the repository file on the node, `pct enter`, run them again inside. Read the prompt first |

## Jellyfin

| Symptom | Cause | Fix |
| --- | --- | --- |
| `syntax error near unexpected token` from `curl ... \| bash` | The URL served an HTML page, and bash tried to run `<html>` | The script no longer exists at that URL. Do the four steps by hand: key, repository file, update, install |
| Library adds but stays empty | Folders created as root, and Jellyfin runs as the `jellyfin` user | `chown -R jellyfin:jellyfin /mnt/media`, then Scan Library |
| New files do not appear | Jellyfin does not watch the filesystem in real time | Dashboard, Libraries, Scan Library after a copy |
| Files copied over the share vanish from the library again | The Samba share writes as root, re-owning files inside the tree | Re-run the `chown` after a large import |

## Windows and the Network

| Symptom | Cause | Fix |
| --- | --- | --- |
| "Your organization's security policies block unauthenticated guest access" | Windows blocks guest SMB logons by default since version 1709. Nobody set a policy | Password version of the share: `guest ok = no`, `valid users`, and `smbpasswd -a` |
| Credential box keeps returning on correct credentials | Samba holds no password for that account, or Windows cached a failed logon | `smbpasswd -a` for the account, then `net use * /delete` on Windows |
| Mapped drive dead after a while | The container leases by DHCP and the lease changed | Static address on the container, or the hostname in the UNC path |
| Domain client cannot find the domain | Client DNS not pointing at the domain controller | The DC points at itself, every other machine points at the DC |
| Home LAN split in two, devices cannot see each other | A second DHCP server reached the house network | Isolate the lab network before activating any scope |
| Service works on the LAN, fails from the internet | Double NAT. The address is rewritten twice on the way out | Count the routers first. See the Active Directory doc's Build 3 diagram |

## SSH

| Symptom | Cause | Fix |
| --- | --- | --- |
| PuTTY connects silently, `ssh` in PowerShell still prompts | KeePassXC supplies keys to Pageant by default, and Pageant and Windows ssh-agent do not share keys | Tick "Use OpenSSH for Windows instead of Pageant" in KeePassXC's SSH Agent settings |
| Key login refused, falls back to password | Loose permissions. sshd silently ignores a writable `authorized_keys` | `chmod 700 ~/.ssh` and `chmod 600 ~/.ssh/authorized_keys` |
| "Permission denied" with the right password | The username case is wrong. Linux names are case sensitive and SSH will not say which half failed | `whoami` on the box, retype the name exactly |
| SSH server dead after a config edit | A one-character typo. An unknown keyword is fatal, not skipped | `sudo sshd -t` before every restart. Silence is a clean file |

## Lessons
| Lesson | Detail |
| --- | --- |
| The error names the wrong thing | Most of these faults printed an error that named the wrong thing. The guest-access message blames a policy nobody set, the `curl \| bash` error looks like a shell bug, the SSH login failure will not say whether the name or the password was wrong. The listed symptom is where to start, not what to trust |
| The fix sits beside the fault | The fix that worked is written next to the fault because I had to scroll back and forth once too often in other people's guides |
| A fault hit twice | A fault I hit twice got its own note in the vault the second time. This table is the collected output of that habit |
