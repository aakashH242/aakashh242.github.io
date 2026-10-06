---
title: Fixing corrupt NTFS on Linux
description: An utility to diagnose and fix corrupt NTFS drives and mount them in Linux. 
---

It's been months since I permanently switched from Windows to Linux. Fedora 44 with KDE Plasma FTW!!!
Switching was pretty simple as my storage setup was a SATA HDD as the boot drive containing Windows and two NVMe M2 drives containing my work, Games and the rest.
I just replaced the SATA HDD with a SATA SSD and installed Linux on it. I hadn't bothered to reformat the M2 drives with ext4 file system as Linux doesn't play well
with NTFS - the file system Windows uses. It's proprietary, AFAIK. Little did I know this will come back to bite me later.

Since all my work and games are on the M2 drives, my IDEs, game launchers - all mount those drives. I even have MCP servers running from those drives.
I had left it running one night - there was a power cut and when I booted back in, the M2 drives just refused to mount. 
A bit of research brought me to the conclusion - power cut, crash, or unsafe disconnect can leave an NTFS drive marked as dirty or with unfinished filesystem changes. Linux may then refuse to mount it.
And I didn't wanna connect and boot up Windows just to repair them. Linux has the `ntfsfix` utility that can help fix some classes of errors and I looked into that.

The outcome is a small utility app that monitors any NTFS drive you connect and, when it's reported corrupt, allows you to diagnose and fix it, as long as the fix falls within `ntfsfix`'s realm.

You can find it here - https://github.com/aakashh242/mount-medic

To install, just run the below commands:

```bash
curl -fsSL https://raw.githubusercontent.com/aakashH242/mount-medic/main/install.sh | bash -s -- --download
```
