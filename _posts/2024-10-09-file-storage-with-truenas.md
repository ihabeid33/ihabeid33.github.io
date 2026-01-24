---
layout: post
title: File Storage with TrueNAS
date: 2024-10-09 14:26:00.000000000 -04:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Storage
tags: []
image: 
  path: assets/2024/10/truenas_logo.png
  width: 1200
  height: 630
---
## TrueNAS Scale

There are times that we find ourselves needing a place to store our ever-expanding collection of files, and the simple 1TB hard drive on our workstation just won't cut it anymore. In such cases, we should probably set up a NAS. A **NAS**, or "Network Attached Storage", is a device that is connected to a network and acts as a dedicated file storage service. Users on the network can connect to it via a 'file share' and be able to access, share, or copy their files with other users. NAS's do this job by pooling a large amount of hard drives together in what is called a 'redundant disk array'. These disk array (or **RAID**) configurations allow a network to centralize hundreds of terabytes of storage while also providing redundancy in case a disk fails. Without redundancy, all of the data could be lost if a single hard drive stops working. This is because the data is "striped" across each disk in the pool.

There are multiple ways you can set up a NAS in your organization's network or your own homelab. You could purchase a pre-built, ready-to-go device like a **Synology NAS**, or you could build your own with open-source software. 

A great example is **TrueNAS**. TrueNAS is a family of open-source NAS solutions developed by iXsystems. Its variations include **TrueNAS Core** (Unix-based), **TrueNAS Community Edition** (Linux-based), and **TrueNAS Enterprise** (business version of the Community Edition and not free).

Many hypervisors these days can function as a NAS at some level. For example, **Proxmox** is an open-source and Linux-based hypervisor that is able to pool together multiple disks into a RAID configuration using a file system technology called 'ZFS'.

I initially decided to set up a NAS because I needed a larger file storage for my media server running **Jellyfin** and **Audiobookshelf**. As my media collection grew, I quickly realized the SSD on my media server was going to run out of space. Therefore, in order to add storage to my homelab, I decided to set up TrueNAS Community Edition (formally known as TrueNAS Scale).

---

## Hardware

To install TrueNAS on bare metal, you need a desktop or server that can carry at least three hard drives. One of the disks is for the boot-drive so it only must be at least 32 GB in size.

*Disclaimer: Try not to use a large capacity drive for the boot because you will not be able to use the remaining space for anything other than booting TrueNAS.*

Pretty much any not-so-old i5, i7 or Xeon with 5-6 cores will suffice for processing power. RAM will be more important than the CPU anyways. It is the general consensus that you should ideally have 1 gigabyte of RAM for every terabyte of storage in your NAS. 

For my NAS build, I used an old HP ProDesk 600 that rocks a humble 4th Gen i5 4-core CPU and 16 GB of DDR3 memory. As for my storage, I managed to stuff 2x8 GB HDDs (plus an SSD for the boot drive) inside of it. So far, I think this was enough to do the job, but I can always upgrade if I need to.

---

## Installation

Installation is fairly simple after burning the ISO to a usb stick and booting into it:

1. Enter the Installation Wizard.
2. Select the drive you want TrueNAS to boot from, and confirm.
3. Enter and confirm a password for the admin user.
4. Reboot.

After those simple steps, you should be able to log into the Web UI and begin using TrueNAS.

### Installation Screenshots

![Enter Wizard]({{site.baseurl}}/assets/2024/10/20250702_012440-scaled.jpg)  
![Select Boot Drive]({{site.baseurl}}/assets/2024/10/20250702_012450-scaled.jpg)  
![Confirm you Understand]({{site.baseurl}}/assets/2024/10/20250702_012502-scaled.jpg)  
![Select 1st Option and Choose a Password]({{site.baseurl}}/assets/2024/10/20250702_012519-scaled.jpg)  
![Done.]({{site.baseurl}}/assets/2024/10/20250702_012741-scaled.jpg)  
![Reboot.]({{site.baseurl}}/assets/2024/10/20250702_012757-scaled.jpg)  

**Tips / Caveats for Beginners:**

- Make sure you take out the USB stick after it's done installing. If you don't, it will boot into it again.
- Make sure the boot order is correct. It should be something like:  
  `USB > Boot-Drive > Any_Other_Drive_For_Storage`
- Make sure your ethernet cables are plugged in. (Duh)
- Make sure you have DHCP working on that subnet or VLAN, or just assign one statically.  
  Your TrueNAS can't start the Web UI if it doesn't have an IP address.

---

## Setup and Pool Creation

After logging in, you will be greeted with the **Dashboard**. At this point you should be able to make any tweaks you need to your NAS's settings. For example, I changed the domain, hostname, and nameserver.

![Dashboard]({{site.baseurl}}/assets/2024/10/00_dashboard-1-1024x548.png)

To be able to utilize our storage, we need to first create a "pool". To do that, first make sure TrueNAS is aware of your disks by going to the **Storage Dashboard**, then clicking **Disks**. Your boot drive will be designated as "boot-pool".

![Disks]({{site.baseurl}}/assets/2024/10/05_disks-1024x543.png)

After confirming the disks are available, you can start creating your pool by going back to the **Storage Dashboard** and clicking **Create Pool**. This will start the **Pool Creation Wizard**.

![Create Pool Wizard]({{site.baseurl}}/assets/2024/10/04_create_pool-1024x545.png)

**First Step: Name your pool and choose if you want to enable encryption.**

![Naming Pool]({{site.baseurl}}/assets/2024/10/06_namepool-1024x582.png)

**Step 2 is to choose your disk layout. You have many different options.**

The worst option is to stripe all your data across all your disks. You do not want to do this as you will not have any disk redundancy. This means that if one of your disks fails, you will have no way to recover your data. The only plus side is that you will be able to use every last bit of your storage space.

The better options are one of the RAID schemes. These allow you to have a varied balance between being able to use most of your storage space and still having redundancy. The downside is that you need to have at least three disks for RAIDZ1, at least four for RAIDZ2, and at least five disks for RAIDZ3.

Unfortunately for me, I only had two 8TB hard drives in my NAS, leaving me with only the ‘Mirror’ option. This option uses one disk for storage and mirrors the data to the other drive for redundancy. Only being able to use 50% of my storage space isn’t ideal, but my machine doesn’t fit any more drives and it’s better than risking data loss.

![Layout]({{site.baseurl}}/assets/2024/10/07_choose_disks-1-1024x487.png)

There are several more optional steps that require extra drives. I did not utilize these because they require more drives. Here is a brief explanation of each:

- **Log**: ZFS log device to improve write speeds.
- **Spare**: Drive reserved for automatically replacing a failed drive.
- **Cache**: A read-cache to improve read performance.
- **Metadata**: Speeds up metadata Input/Output.
- **Dedup**: Deduplication tables for performance on certain workloads.

Review your configuration and if you're happy, click **Create Pool**. Your Storage Dashboard should look similar to this when done:

![Pool Created]({{site.baseurl}}/assets/2024/10/08_pool_done-1024x539.png)

The next step is to create datasets to organize and store your files.

---

## Creating Datasets

A dataset is like a file system. You create different datasets for different things, such as one dataset for media files and another for docker containers. Datasets can also have parent-child relationships.

![Datasets]({{site.baseurl}}/assets/2024/10/09_datasets-1024x561.png)

To create a dataset, go to your Dataset Dashboard, select your pool, and click ‘Add Dataset‘.

From there, name your dataset whatever you like, and then select a preset. 
* If you plan to set up an SMB share with this dataset, select **SMB**. 
* If it’s going to be for containers, select **App**. 
* If you’re like me and you’re setting up an NFS share, select **Generic**. 

You can leave the advanced options as default for now and click save.

![Dataset Options]({{site.baseurl}}/assets/2024/10/10_dataset_options.png)

Make sure you re-select the pool for every dataset you create if you want it to be the parent. You might accidentally set the parent of any new dataset you create as the previously created dataset.

![Datasets Created]({{site.baseurl}}/assets/2024/10/11_datasets_created-1024x571.png)

---

## Dataset Permissions

After creating a dataset, it’s important to configure its permissions appropriately. Permissions control how users, and groups of users, are allowed to interact with resources. TrueNAS provides two main ways to fine-tuning the permissions applied to datasets: **Unix Permissions** and **Access Control Lists** (ACLs).

### Unix Permissions

Unix permissions are the most basic way to do access control on a dataset. These permissions function on two main axes: ownership and access. Files and directories are set to be owned by both a user and a group. If a user does not fit into either that user or group ownership, it will be classified as ‘other’. For each of those three types of ownership, there are also three types access controls: read, write, and execute (rwx). Combining both ownership and access controls gives us a matrix of 3×3, as you can see on your dataset’s permissions editor.

To edit a dataset’s permissions, select the dataset to edit, find the “Permissions” box, and click ‘Edit’. This will open the Unix Permissions Editor.

![Unix Permissions]({{site.baseurl}}/assets/2024/10/15_unix_permissions.png)

I found that balancing the accessibility needs of my dataset, while also keeping it secure, was kind of tricky at first. I needed the service accounts on a different machine (media server) to be able to access the file share, but I did not want to grant users in the ‘Other’ group access to it for security purposes. A simple solution that worked for me was to create a new group called ‘media’ and grant it ownership to the dataset. The downside to this is that I also had to recreate the same group on any machine that needs access, then add the necessary users and service accounts to that group.

Unix permissions are traditional and the simpler way of controlling access. They are more suited for smaller Unix/Linux environments that do not need complex user or group access rights. Unix permissions are also mostly used with NFS shares instead of SMB.

### Access Control Lists (ACLs)

ACLs are typically set up when sharing over SMB, and sharing to more complex environments that use both Windows and Linux operating systems. They use a more complex, Windows style permission scheme with attributes like Read, Read/Write, Full Access, or Deny being applied to users and groups. The screenshot below shows how I set up a different dataset’s permissions that is being shared over SMB:

![ACLs]({{site.baseurl}}/assets/2024/10/16_acl-1024x719.png)

---

## Adding Shares

In order to share our datasets to a network, we need to add a file sharing service. TrueNAS gives us three options: NFS, SMB, and iSCSI.

![Shares Menu]({{site.baseurl}}/assets/2024/10/12_shares-1024x511.png)

### NFS

This is the Network File System. It is a Unix-based file share, great for Linux and UNIX environments.

### SMB

Server Message Block. A Windows-based file share commonly used in Windows and Active Directory environments, although Linux machines do have the ability to mount an SMB drive.

### iSCSI

Internet Small Computer Systems Interface. This is another network protocol that shares storage devices over IP. It works by encapsulating SCSI commands within network packets.

### Sharing with NFS

I chose to use NFS to share my datasets. To add an NFS share, first go to the **Shares** menu, and click the **‘Add‘** button within the NFS box. Next, select the path to the dataset you want to share and make sure the **‘Enabled‘** option is checked. All other options can be left as the default. Click **‘Save‘** to launch it.

![NFS Share Options]({{site.baseurl}}/assets/2024/10/13_share_options-1024x585.png)

Now your NFS service should be running and sharing your dataset.

![NFS Share Up]({{site.baseurl}}/assets/2024/10/14_shares_up.png)

#### File Share Permissions

File share access controls allow you to specify which networks, hosts, or IP addresses are authorized to access the file share. Restricting access based on network or IP addresses allow for a more granular way to apply permissions securely, without being to restrictive with traditional Unix permissions or ACLs.

![Network Access Controls]({{site.baseurl}}/assets/2024/10/17_network_access.png)

---

## Mounting NFS

Finally, we get to mount the NFS drives we created, which can be done with a simple command:

```bash
sudo mount -t nfs [ip_of_truenas]:/mount/point/to/share [local mount directory]
```
Example:

```bash
sudo mount -t nfs 10.1.3.5:/mnt/pool0/books /tmp/share/books/
```

## fstab

Maybe you want your system to mount a share at boot. On Linux, you just have to edit a single file:

/etc/fstab

Here’s my media server’s fstab file as an example:

```bash
10.1.1.19:/mnt/pool0/jellyfin /share/jellyfin nfs rw,vers=4.0 0 0

10.1.1.19:/mnt/pool0/books /share/books nfs rw,vers=4.0 0 0
```
The parameters you add will depend on your situation (like the NFS version running on TrueNAS).