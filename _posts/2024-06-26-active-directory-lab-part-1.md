---
layout: post
title: Active Directory Part 1 - Lab Setup
date: 2024-06-26 00:03:10.000000000 -04:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Active Directory
tags: []
image: assets/2024/06/ad_icon.webp
meta:
  _last_editor_used_jetpack: block-editor
  _edit_last: '1'
  bloglo_disable_topbar: ''
  bloglo_disable_header: ''
  bloglo_disable_page_title: ''
  bloglo_disable_breadcrumbs: ''
  bloglo_disable_thumbnail: ''
  bloglo_disable_footer: ''
  bloglo_disable_copyright: ''
  bloglo_disable_blog_card_border: ''
  _thumbnail_id: '240'
permalink: "/2024/06/26/active-directory-lab-part-1/"
---
## Why Active Directory?

Active Directory (AD) is a critical component in many IT environments, especially within organizations using Microsoft technologies. It is estimated that well over 90% of Fortune 1000 companies implement Active Directory services in some form. That is why it is crucial for anyone that wants to enter the IT field, must set up their own AD Lab to practice on. Here are some of the features provided by AD:

* Active Directory allows administrators to centrally manage user accounts, computers, and groups using a single centralized database. Tasks such as user creation, password changes, and permission management can be done from a single machine and applied to the whole domain.
* A feature called Group Policy implements domain wide configurations such as security settings and software installations across all computers.
* Another major part of Active Directory is Directory Services. This is basically like a repository (or database) that maps out all of the network resources on the domain, making them easily accessible. Such resources include printers, computers, and file shares.
* AD can also perform Single Sign-On (SSO) and access control. SSO is a way for users to log in one time and still access more than one resource on the network with specific permissions. Active Directory does this using one of two protocols: Kerberos or LDAP.
* Other features include Federation Services (ADFS), Auditing & Monitoring, and redundancy/fault tolerance.

The next parts of this post are a walk-through on how I set up Active Directory in my Homelab environment.

## Creating Server and Client VMs in Proxmox

Every Active Directory environment will need at least a version of Windows Server for the Domain Controller, and one or more machines running Windows 7/8/10/11 for our workstations. I chose to run Windows Server 2019 and Windows 10 as virtual machines inside of a virtualization environment called Proxmox.

Links to the ISOs:

* [Windows Server 2019](https://info.microsoft.com/ww-landing-windows-server-2019.html)
* [Windows 10](https://www.microsoft.com/en-us/software-download/windows10ISO)

Please note:

* Windows 10 has four different editions: Home, Pro, Education, and Enterprise.
* Windows 10 Home edition CANNOT be used to join a domain.
* In order to use Windows 10 in an AD environment, you must choose any one of the other three editions.
* Windows 7 and 8 are the same way.

Proxmox VE (Virtual Environment) is a Linux-based and open-source virtualization management solution. It is also a "Type 1" hypervisor, meaning it is run directly on a machine's physical hardware as the machine's operating system. "Type 2" hypervisors, such as Virtualbox and VMware Workstation, run as an application on top of an already existing OS.

After downloading my ISO's, I went ahead and created two VMs on Proxmox. The screenshots below show the hardware specifications I used for each VM.

![Server VM Specifications]({{site.baseurl}}/assets/2024/06/0_proxmox_winserver.png)
![Windows 10 VM Specs]({{site.baseurl}}/assets/2024/06/0_proxmox_win10-1024x380.png)

## Setting Up Windows Server

### Installing Windows Server

Once the VM is set up correctly, the next step is to start the VM and install the operating system. (Make sure it is set up to boot the ISO before booting).

Once it has booted, select your preferences and click **Next**.

![Windows Setup]({{site.baseurl}}/assets/2024/06/1_WindowsSetup.png)

Click **Install now**.

![Install Now]({{site.baseurl}}/assets/2024/06/2_InstallNow.png)

Insert CD key if you have one. If not, you may get a free trial but it will expire after a set time. After that click **Next**.

![CD Key]({{site.baseurl}}/assets/2024/06/3_CDKey.png)

You can choose to install with just a command-line environment or with a Desktop environment. I chose the latter (Desktop Experience).

![Desktop Experience]({{site.baseurl}}/assets/2024/06/4_DesktopExperience.png)

Accept the license terms and click **Next**.

![Accept Terms]({{site.baseurl}}/assets/2024/06/5_AcceptTerms.png)

Select the Custom Install option.

![Custom Install]({{site.baseurl}}/assets/2024/06/6_CustomInstall.png)

Select **Drive 0** and click **Next**.

![Select Drive]({{site.baseurl}}/assets/2024/06/7_SelectDrive.png)

The setup wizard will now start the install process. Once it completes it should reboot the virtual machine.

![Wait for Install]({{site.baseurl}}/assets/2024/06/8_WaitforInstall.png)

After the VM reboots, it should prompt you to set up a password. After doing so, click **Finish**.

![Set Password]({{site.baseurl}}/assets/2024/06/9_Password.png)

Windows Server should now be ready.

### Configuring Windows Server

If you are running your server in a virtual machine, it's always helpful to install guest tools on it. Guest tools are software that allow your keyboard and mouse input to pass to the virtual machine. Which tools to install and how to install them depends on what hypervisor you are using. For example, VMware uses "VMware Tools". 

For my lab, I am using "Spice Guest Tools".

![Install Guest Tools]({{site.baseurl}}/assets/2024/06/10_InstallGuestTools.png)

The next step is to rename the machine.

To rename the machine, click the start menu and search **About**. Then click on **About your PC**.

Click on **Rename this PC**, and rename it to whatever you like. Then click **Restart Now** for it to take effect immediately.

![Rename PC]({{site.baseurl}}/assets/2024/06/11_rename.png)

After renaming the machine, the next task is to give it a static IP address.

We need to give it a static IP address because it is a server, and servers usually should always keep the same IP address for long periods of time. Therefore, DHCP should not be in use here.

* Right click on the network icon in the task bar
* Click **Open Network & Internet settings**
* Click the **Ethernet** tab, then click **Change adapter options**
* Right click the adapter you want to configure and click **Properties**
* Highlight the "TCP/IPv4" settings and click **Properties**
* Select "Use the following IP address" and "Use the following DNS server addresses" settings
* Enter the required information then click **OK**
  * For DNS, set the preferred DNS to localhost (127.0.0.1) and the alternative DNS to a public DNS
* After that, restart the VM

![Network Settings]({{site.baseurl}}/assets/2024/06/12_net_settings.png)

After rebooting, we should be ready to finally install Active Directory.

### Installing Active Directory

First, open the Server Manager if it is not already open. Click on the **Manage** tab, then click on **Add Roles and Features**. This should open up an install wizard for setting up server roles. 

At the "Before You Begin" page, click **Next**.

![Before You Begin]({{site.baseurl}}/assets/2024/06/13_beforeyoubegin.png)

For the "Installation Type" page, select **Role-based or feature-based installation**. Then click **Next**.

![Installation Type]({{site.baseurl}}/assets/2024/06/14_installationtype.png)

On the "Server Selection" page, select your server (there should only be one) and click **Next**.

![Server Select]({{site.baseurl}}/assets/2024/06/15_serverselect.png)

Now for the Server Roles. Select the **Active Directory Domain Services** role. A pop-up window should come up. Leave the required features as is and click **Add Features**. Then click **Next**.

![Select Roles]({{site.baseurl}}/assets/2024/06/16_select_roles-1024x642.png)

At the "Features" page, we don't need any more features, so just click **Next** again.

![Features]({{site.baseurl}}/assets/2024/06/17_features.png)

The "AD DS" page will tell us that we will need to install the DNS role for Active Directory to work properly. We will do that later. Click **Next**.

![AD DS]({{site.baseurl}}/assets/2024/06/18_adds.png)

At the "Confirmation" page, confirm your choices and click **Install**.

![Confirmation]({{site.baseurl}}/assets/2024/06/19_confirmation.png)

The wizard will now install AD DS to your server. Once it is successfully finished, click **Close**.

![Results]({{site.baseurl}}/assets/2024/06/20_results.png)

A notification should appear on your dashboard. Click the icon and then click **Promote this server to a domain controller**. 

![Promote to DC]({{site.baseurl}}/assets/2024/06/21_promotetodc.png)

A new window should pop up. Since we have not set up a domain before, select **Add a new forest** and enter a root domain name. Click **Next**.

![Deployment Config]({{site.baseurl}}/assets/2024/06/22_deploymentconfig.png)

On the DC Options page, select the highest possible **Forest** and **Domain** function levels. Make sure the "DNS server" box is checked. The other boxes are grayed out because it is a new forest. Choose a password and click **Next**.

![DC Options]({{site.baseurl}}/assets/2024/06/23_DC_options.png)

On the "DNS Options" page, leave the box unchecked and click **Next**.

![DNS Options]({{site.baseurl}}/assets/2024/06/24_dnsoptions.png)

For "Additional Options", choose a NetBIOS domain name or leave the generated one as is. Click **Next**.

![Additional Options]({{site.baseurl}}/assets/2024/06/25_add_options.png)

For "Paths", you can choose the drives and locations for the Database, Logs, and SYSVOL folders. You can leave them as default and click **Next**.

![Paths]({{site.baseurl}}/assets/2024/06/26_paths.png)

Review the options and click **Next**.

![Review Options]({{site.baseurl}}/assets/2024/06/27_review_opt.png)

At the "Prerequisites Check" page, you should get an "All prerequisite checks passed" notification. You may also get a few warnings; that's okay if they are not critical. Click **Install**.

![Prerequisites Check]({{site.baseurl}}/assets/2024/06/28_prereq_check.png)

Your Active Directory should now install and your machine should reboot. After reboot, AD DS and DNS should be running. You can confirm on the dashboard.

![Services Up]({{site.baseurl}}/assets/2024/06/30_services_up.png)

---

### Installing Active Directory Certificate Services (AD CS)

ADCS allows us to create certificate authorities to manage certificates for protocols and services, e.g., LDAPS.

The steps are almost the same as installing AD DS:

1. Click the **Manage** tab, then **Add Roles and Features**.
2. Leave defaults on the first three pages until "Server Roles".
3. Select **Active Directory Certificate Services Role**.
4. Click **Add Features**, then **Next**.
5. Leave "Features" page default and click **Next**.
6. Click **Next** at the "AD CS" page.
7. At the "Role Services" page, check **Certificate Authority** and click **Next**.
8. Finally, click **Install**.
9. When installation is finished, click **Close** and reboot.

![ADCS Installed]({{site.baseurl}}/assets/2024/06/31_adcs.png)

Now we need to configure AD CS:

1. Open the **Notifications** tab, find the new notification, and click **Configure Active Directory Certificate Services...**
2. On the "Credentials" page, leave defaults and click **Next**.
3. For "Role Services", check **Certificate Authority** and click **Next**.
4. Set the Setup Type as **Enterprise CA**.
5. Set the CA Type as **Root CA**.
6. Create a new private key.
7. Leave cryptography as default and click **Next**.
8. Leave CA Name as default and click **Next**.
9. Change validity period if desired, click **Next**.
10. Leave Certificate Database location as default and click **Next**.
11. Confirm and click **Configure**.
12. Finally, click **Close** and reboot your server.

![ADCS Up]({{site.baseurl}}/assets/2024/06/32_adcs_up.png)

AD CS should now be fully up and running.

## Setting Up Windows 10

### Installing Windows

Now it's time to set up our first workstation.

![Windows 10 Setup]({{site.baseurl}}/assets/2024/06/1_setup_win10-1.png)

After creating a Windows 10 VM, boot it up and start the installation process.

1. Select Language, Time/Date format, and Keyboard Layout.
2. Click **Install Now**.
3. Enter a product key or click **I don't have a product key**.
4. Select your operating system.
   - Note: Only Pro, Education, or Enterprise editions can connect to Active Directory.
5. Accept License Terms.
6. Select **Custom Install → Select Drive**.
7. Wait for installation to finish.

![Windows 10 Install]({{site.baseurl}}/assets/2024/06/2_install.png)

After installing, the machine should reboot and start a second setup wizard.

![Windows 10 Setup 2]({{site.baseurl}}/assets/2024/06/3_setup2.png)

1. Select your region.
2. Setup Keyboard layouts.
3. Instead of signing in with Microsoft account, select **Domain Join instead**.
4. Enter a name and password for a temporary user.
5. Fill out security questions.
6. Select Privacy Settings.
7. Disable Cortana.

After that, the setup should be complete. Now we only need to install guest tools to the VM and we are ready to connect to the Domain Controller.

---

### Network Firewall Configuration

Port Allow List:

![AD Ports]({{site.baseurl}}/assets/2024/06/ports.png)

A lot of times an organization’s network isn’t flat. It could be segmented with multiple subnets, VLANs, and secured with firewall rules. That is sort of how built my Homelab. I designed my network using VLANs to segment it and Pfsense to handle routing and filtering. You can read more about it in my first blog post.

Long story short, I have configured my virtual machines to be on different VLANs. My internal services, including Active Directory, are tagged with VLAN 10. User devices and workstations are tagged with VLAN 20. This also means that they have to be routed through Pfsense in order to connect to each other. Therefore, I need to enable port numbers associated with Active Directory services, so that my workstation doesn’t get blocked from connecting to the DC. The diagram above lists some of the ports required for Active Directory to function.

In Pfsense, I created an alias grouping all of the required port number together:

![Pfsense Alias]({{site.baseurl}}/assets/2024/06/pfsense-alias-1024x771.png)  

After that, I created a firewall rule using that alias to allow AD DS traffic between the UserLAN and the AdminLAN.

![Firewall Rule]({{site.baseurl}}/assets/2024/06/firewall_rule-1.png)

Now that my workstation has access to the DC, it’s ready to connect to the domain.

![Connection Allowed]({{site.baseurl}}/assets/2024/06/connection_allowed.png)

---

### Connecting Windows 10 to Active Directory

The first step is to configure the workstation to use the Domain Controller as its DNS server.

1. On Windows 10, open **Network and Internet Settings**.
2. Click the **Ethernet** tab.
3. Click **Change adapter options**.
4. Choose your adapter, right-click, and select **Properties**.
5. Select **IPv4 Properties**.
6. Choose **Use the following DNS server addresses:**
7. Enter the Domain Controller's IP as the preferred DNS.
8. Enter either a local or public DNS as the alternative.
9. Click **OK**, close settings, and reboot.

![DNS Settings]({{site.baseurl}}/assets/2024/06/4_dns_settings.png)

Next, rename the PC and join it to the Domain:

1. Go to **Start → About your PC**.
2. Click **Rename this PC**, enter a new name, and click **Next → Restart later**.
3. Go to **Connect to work or school** in settings.
4. Click **Connect → Join this device to a local Active Directory domain**.
5. Enter the domain name and click **Next**.
6. Enter Administrator credentials. Click **Skip** if asked to add an account.
7. Restart the workstation.

Now if you check **Active Directory Users and Computers** on the DC, the workstation is part of the domain.

![Workstation Added]({{site.baseurl}}/assets/2024/06/7_WS_added.png)

---

## End of Part 1

So far, we’ve:

- Set up virtual machines.
- Installed Active Directory (AD DS) and Active Directory Certificate Services (AD CS).
- Configured firewall rules for AD traffic.
- Connected a Windows 10 workstation to the domain.

In Part 2, we will:

- Add users and more workstations.
- Configure Group Policy settings.
- Explore PowerShell scripting for AD administration.
