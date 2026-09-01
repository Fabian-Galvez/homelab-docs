# Proxmox Host

## Overview

Proxmox VE runs as the bare-metal operating system on the 2011 Dell Inspiron 620. The host boots from a 120 GB SSD, holds media on a 1 TB data drive, and runs the lab's VMs and containers. Administration happens in a browser at `https://<proxmox-ip>:8006`.

Built 12 and 13 January 2026 against Proxmox VE 9.1.1. Every address in this doc is a placeholder.

## Install

The install is four screens and about fifteen minutes. Four decisions on those screens are hard to change later.

| Decision | Choice       | Reason |
| --- | --- | --- |
| Target disk | The 120 GB SSD, with the 1 TB drive physically disconnected | The target disk page wipes the whole selected disk. With one disk connected there is no wrong disk to pick |
| Filesystem | `ext4` | ZFS is the right choice with multiple disks and spare RAM. This box has one system SSD and a hard 16 GB ceiling |
| Hostname | `<hostname>.home.arpa` | `.local` is reserved for multicast DNS and can be answered by any device on the network. `.home.arpa` is reserved for home networks by RFC 8375 |
| Management address | Typed once on the network screen | No later screen sets it. Every URL and command in the lab is downstream of this box |

The installer builds a bridge called `vmbr0` on the first Ethernet card and puts the address on the bridge. That bridge is what every VM and container's network tab points at later.

I verified the ISO hash before writing it to the stick. `certutil -hashfile` on Windows, compared character for character against the SHA256SUM on the download page.

## The Built-In First Failure
A fresh install points at the enterprise package repository, which needs a paid subscription. The very first `apt update` fails with a red banner. Nothing is broken.

| Step | Location |
| --- | --- |
| Disable the `pve-enterprise` entry | Node, Updates, Repositories |
| Add the No-Subscription repository | Same page |
| Refresh and confirm `TASK OK` | Node, Updates |

No-subscription is not a free tier. It is the same software from a repository that gets less testing before release. Fine for a lab, not for anything people depend on.

## Storage Layout

| Storage | Disk | Type | Holds |
| --- | --- | --- | --- |
| `local` | 120 GB SSD | Directory | ISOs, container templates, backups |
| `local-lvm` | 120 GB SSD | LVM-Thin | VM and container root disks |
| `TB-mediaDrive` | 1 TB HDD | Directory | media files only |

The installer creates the first two. I created the third after wiping the 1 TB drive, which still held old partitions: a Dell recovery partition, a Windows install, and an earlier attempt at this same build.

The media drive is a Directory storage on purpose. It holds files a container bind-mounts straight in, not virtual disks. LVM-Thin on that drive would have been the wrong tool.

Proxmox refuses to initialize a disk that still shows partitions. The order that works: destroy the old Directory entry, remove the storage from the datacenter, wipe the disk, then Initialize with GPT. The wipe confirm dialog repeats the drive's serial number. I match the serial against the sticker on the physical drive before clicking. `/dev/sda` can name a different disk on the next boot.

## Patching the Node

The Updates panel is a preview. The Upgrade button opens a live console running `apt-get dist-upgrade`, the command the Proxmox docs require. Plain `apt-get upgrade` will not complete an upgrade that needs a dependency change.

Reading the pending list before pressing Upgrade is the habit worth keeping. Packages are grouped by Origin:

| Origin | Layer | Effect on the plan    |
| --- | --- | --- |
| Debian | The base OS | Nothing, usually. Thirty-two library packages is a routine week |
| Proxmox | The hypervisor, including its own kernel | A kernel entry means a reboot is owed. A `pve-manager` entry moves the management layer and is worth scheduling |

A kernel package in the pending list means the running kernel is no longer the installed one after the upgrade. The node needs a reboot. Guests get shut down first.

## The Task Log

Every action in Proxmox becomes a numbered task. The one-line red summary in the tasks pane is never the actual error. Double-clicking the row opens the full console output.

- A successful run ends with `TASK OK` on its own line. That string is the check, not the absence of red.

- A task showing OK means the command exited cleanly, not that it did what I wanted. An `apt update` that only reached the Debian repositories succeeds while telling me nothing about Proxmox packages. I read which repositories were hit, not just the last line.

## The Subscription Dialog

The "No valid subscription" dialog on every login comes from a check inside `proxmoxlib.js`, a JavaScript file the node serves to the browser. Editing one function removes the dialog. Nothing else changes: paid features stay locked, the enterprise repository stays refused.

| Step | Command |
| --- | --- |
| Back up the file | `cd /usr/share/javascript/proxmox-widget-toolkit && cp proxmoxlib.js proxmoxlib.js.bak` |
| Edit the `checked_command` success handler to call `orig_cmd();` directly | `nano proxmoxlib.js` |
| Restart the service that serves the file | `systemctl restart pveproxy.service` |

Skipping the `pveproxy` restart is why people conclude the edit failed. The browser is still holding the old file. Any update that ships a new `proxmox-widget-toolkit` overwrites the edit. The dialog comes back. Expected, not a failed edit.

## Working Commands

```
# Node shell
pct list                      # containers and their state
pct enter 110                 # shell inside the container, no password
pveversion                    # which release is actually running

# Read back what the installer set
hostname -f
ip -4 addr show vmbr0
cat /etc/network/interfaces
```

Shutdown order at the end of a session: container first, then node, then mains. The web interface Shutdown on a VM without a guest agent installed is closer to holding the power button.

## Lessons
| Lesson | Detail |
| --- | --- |
| First `apt update` | The first `apt update` on a fresh Proxmox install fails by design. Fix the repositories before trusting anything else on the page |
| Disk selection | A disk is selected by serial number, never by `/dev/sda`. Device names can change between boots. The serial is on the sticker |
| Terminal UI installer | The Terminal UI installer is the same installer as the graphical one. A black screen at install is the reason it exists, not a lesser option |
| Ventoy on old hardware | Ventoy never booted on this BIOS-era Dell. Rufus in MBR mode worked first try. On an old machine, skip Ventoy and flash the drive directly |
| The shell prompt | Read the prompt before every command. `root@pve` is the physical Dell. `root@<ct-hostname>` is the container. I ran repository commands on the hypervisor by mistake once. The cleanup taught me to check the prompt every time |
