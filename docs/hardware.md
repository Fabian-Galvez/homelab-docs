# Hardware

## Overview

One physical machine carries every build in this lab: a 2011 Dell Inspiron 620 desktop tower. The Dell arrived as a stock office PC with 6 GB of memory, one hard disk and ten years of dust. This doc records the refurbishment that made it a lab machine and the drive intake process behind its storage.

## The Refurbishment

| Part | 2011 stock | After | Reason |
| --- | --- | --- | --- |
| Memory | 6 GB, a 2 GB and a 4 GB stick | 16 GB, two matched 8 GB sticks | 6 GB runs one VM badly and two not at all |
| Boot drive | one 1 TB HDD carrying Windows 7 | 120 GB SSD | a hypervisor workload is mostly small random reads, the access pattern hard disks handle worst |
| Data drive | none | the original 1 TB HDD, data only | a rebuild wipes the SSD and leaves the data alone |
| CMOS battery | the 2011 coin cell | fresh CR2032 | a clock that resets on every power-off breaks domain logins |
| Thermal paste | the 2011 layer | fresh paste | the cooler was already off for the dust clean |
| Dust | ten years of it | cleaned out | dust stops a heatsink from moving heat |

Work order: unplug and hold the power button five seconds, strap to bare chassis metal, cooler off and dust out, battery, memory, repaste, fit the SSD, set boot order, then diagnostics before any OS goes on.

The second network card is not part of the refurbishment. It went in later, for the current build's two-NIC perimeter.

## The 16 GB Ceiling
| Limit | Number | Source                     |
| --- | --- | --- |
| Memory slots on the board | 2 | the Inspiron 620 board |
| Largest DDR3 non-ECC unbuffered desktop stick | 8 GB | the DDR3 standard. Bigger DDR3 modules exist only as ECC or registered server parts |
| Maximum installable | 16 GB | 2 slots times 8 GB |
| What Dell documented | 8 GB | the board runs 16 GB anyway, common for boards of this era |

Dell's spec sheet said 8 GB. The real wall is the DDR3 standard. Every architecture decision in this lab, containers over VMs, ext4 over ZFS, flows from this one number.

## Hardware Checks First
Diagnostics ran on the refurbished machine before any operating system went on. A diagnostic on a bare machine tests the hardware and nothing else.

| Diagnostic | Runs from | Unique proof                          |
| --- | --- | --- |
| Dell built-in diagnostics, F12 at power-on | the firmware | the vendor's own test of memory, drives and board. Ran a full night |
| MemTest86, from USB | its own tiny OS | nearly all of the memory, because no OS is holding any of it |
| CrystalDiskInfo | inside Windows | the drive's own SMART error log |

## Drive Intake

Two more 1 TB drives came out of donated broken laptops and were fitted after the current build was already running. A drive out of a dead laptop is untrusted until it passes intake.

| #   | Step | Proof |
| --- | --- | --- |
| 1 | Physical clean | the drive arrives dusty from the donor laptop |
| 2 | Connect over a SATA-to-USB adapter to a bench machine | an unknown drive never touches a machine holding data |
| 3 | Read SMART | the errors the previous owner never saw |
| 4 | Full-surface pass over every sector | every sector still accepts data |
| 5 | Read SMART again | whether the pass forced the drive to retire sectors |

Step 4 does the real work. A bad sector is only found when something tries to use it. A quick format never touches the sectors at all, only the index that says where files live.

Reallocated Sector Count is the value compared across steps 3 and 5. A count that rises during the pass is the drive reporting wear.

The passes ran overnight. A 5400 rpm laptop drive behind a USB adapter writes 1 TB in about five hours. The first pass had to be paused mid-day and rerun. The two drives took three nights in total.

## OS History
| Operating system | Reason |
| --- | --- |
| Windows 7 | shipped with it. Booting the original OS proved the hardware baseline |
| Windows 10 | the supported upgrade path, practiced both in-place and clean |
| Windows 11 | installed with workarounds. The Dell fails the UEFI, TPM 2.0 and CPU generation requirements and passes the memory and storage ones |
| Ubuntu and Debian | Linux practice alongside the Windows work |
| Proxmox VE | the hypervisor it runs today |
| Windows Server 2025 with Hyper-V | the enterprise lab build |

The Windows 11 workaround machine is a lab machine and never one to hand to a user. The three failed requirements protect Secure Boot and hardware-backed encryption, not speed. More memory was never going to help. Knowing which requirement fails and why is the point of keeping the record.

## Lessons
| Lesson | Detail |
| --- | --- |
| Memory clips | Both memory retention clips must snap shut on their own. A clip I had to push closed was a stick that was not seated |
| Standby power | The power supply keeps a standby rail live whenever the machine is plugged in. Unplug and hold the power button before touching anything |
| Thermal paste | A cooler that has been lifted always gets fresh paste. There is no putting a ten-year-old layer back |
| SATA hot-swap | Consumer boards from this era handle hot-swap badly. The data drive's SATA cable was connected with the machine powered down and unplugged |
