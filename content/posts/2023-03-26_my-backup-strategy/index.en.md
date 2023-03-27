---
title: A Backup Strategy for 2023
date: 2023-03-26T14:29:54.905Z
---
After lucking out and not loosing any of my data saved to my nextcloud after the OVH fire in Strasbourg. I did however loose the VPS that hosted the website for [www.evansmedia.fr](https://www.evansediafr)

> The shoemaker always wears the worst shoes, right?

Since then I may have become a bit paranoid about backups, especially for anything stored in my Nextcloud which hold just about anything to do with my personal life, including photos of our recent newborn. 

## The 3-2-1 Rule
I decided to base my backup strategy on the 3-2-1 rule (and you should too)
>The idea that a minimal backup solution should include three copies of the data, including two local copies and one remote copy.

So that means I need: Two local backups at home (althougth the virtual machine I'm backing up is offsite) and one backup that is remote. 

## Local Backups

*(We are pretending that we are backing up a local server here but the Virtual Machine I'm backing up is actually hosted in a DC so I'm backing it up locally at home)*

To perform my backups I am using two tools, Proxmox Backup Server^pbs which is a backup appliance built by the folks that build proxmox; and Restic, a modern app that backup files to loads of different services.

### Proxmox Backup Server (PBS)

> Proxmox Backup Server is an enterprise backup solution, for backing up and restoring VMs, containers, and physical hosts. By supporting incremental, fully deduplicated backups.

Using PBS appliance that is installed on a server at home, I run a daily backup an incremential snapshot whole virtual machine hosting my nextcloud to my NAS. Of these backupsI keep two incrementials and one full backup.

{{<figure src="./pbs_settings_600.png">}}