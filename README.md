# Home Lab Documentation

This repository documents a home lab running on a single 2011 Dell Inspiron 620. The first build ran inside VirtualBox on a laptop.

These docs record each build, what changed between them, and why.

## Purpose

The lab exists to practice infrastructure work on real hardware. Each build is documented with its configuration.

| Area             | Coverage                                                                                                   |
| ---------------- | ---------------------------------------------------------------------------------------------------------- |
| Active Directory | Domain controller promotion, DNS, DHCP, OUs and groups, bulk user creation, home folders, roaming profiles |
| Virtualization   | Type 1 and Type 2 hypervisors, VMs against containers, storage pools, memory budgeting                     |
| Networking       | Gateways, NAT and double NAT, port forwarding, guest network isolation, DNS filtering                      |
| Remote access    | SSH key authentication, agent-held keys, mesh VPN, remote desktop                                          |
| Storage          | Drive intake testing, partition wiping, bind mounts, SMB shares                                            |
| Troubleshooting  | Isolating one variable per change and recording the result                                                 |

## The hardware

| Part      | Detail                                                                                              |
| --------- | --------------------------------------------------------------------------------------------------- |
| Machine   | Dell Inspiron 620, 2011                                                                             |
| CPU       | Intel Core i5                                                                                       |
| Memory    | 2 x 8 GB DDR3, upgraded from 6 GB                                                                   |
| Boot disk | 120 GB SSD, Patriot Burst Elite                                                                     |
| Storage   | 3 x 1 TB hard disks. Two were recovered from donated laptops and health-checked before being fitted |
| Network   | Two NICs, the second added for the current build                                                    |

## The three builds

| Build | Hypervisor | Server | Network boundary |
| --- | --- | --- | --- |
| 1 | VirtualBox, Type 2 hosted, on a Lenovo laptop | Windows Server 2019 | Inside VirtualBox. No lab traffic reached a physical port |
| 2 | Proxmox VE, Type 1 bare metal, on the Dell | Windows Server 2022 | The Proxmox bridge on the Dell's single NIC |
| 3 | Proxmox VE, Type 1 bare metal, on the Dell | Windows Server 2022/2025 | A pfSense VM between two physical NICs, with Pi-hole filtering DNS ahead of the router |

<sub>Build 3 is the current lab. Builds 1 and 2 are documented because the progression between them is the record of what was learned.</sub>

<br>

## Documentation index

| Doc                                                                              | Covers                                                                     |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| [Active Directory Lab](docs/active-directory-lab.md)                             | Three builds, domain controller, DNS, DHCP, OUs, bulk users, profiles      |
| [Remote Access and Security](docs/remote-access-and-security.md)                 | SSH keys, KeePassXC agent, Tailscale, WireGuard, Pi-hole, router hardening |
| [Proxmox Host](docs/hypervisor-host.md)                                             | Bare-metal install, storage pools, repositories, node upgrades             |
| [Media Server](docs/media-server.md)                                             | Jellyfin in an LXC container, bind mounts, Samba share                     |
| [Hardware](docs/hardware.md)                                                     | All the hardware used in the 3 builds                                      |
| [Troubleshooting](docs/troubleshooting.md)                                       | All issues, their cause and fix. One table per build area                  |
| [Install Proxmox on an Old PC](https://github.com/Fabian-Galvez/old-pc-to-hypervisor) | The walkthrough for the install as an interactive live app                 |

<br>

## Key lessons

| Lesson                                                                          | Problem                                                                                             |
| ------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| A second DHCP server reachable from the home network splits that network in two | Half the devices lease from the router, half from the lab, and the two halves cannot see each other |
| Double NAT breaks inbound port forwarding                                       | When a service works on the LAN and fails from the internet, count the routers first                |
| Change one setting, then test                                                   | I once changed several settings at once and could not locate the fault afterwards                   |
| A drive out of a donated laptop is untrusted                                    | It stays untrusted until it passes a health check and a full-surface test                           |
| Document the fix when it is made                                                | The builds I did not write up at the time are the ones I cannot explain now                         |
<br>

## In progress

- Windows Server 2025 with Hyper-V and Sophos build doc

## Notice

Proxmox is a registered trademark of Proxmox Server Solutions GmbH. Microsoft,
Windows Server and Hyper-V are trademarks of Microsoft Corporation. Dell is a
trademark of Dell Inc. Jellyfin, Pi-hole, pfSense, Tailscale, WireGuard,
KeePassXC, Samba, VirtualBox and Sophos are trademarks of their respective
owners. These docs record one home lab and are not affiliated with, endorsed
by, or sponsored by any of them.
