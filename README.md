# Build a Directory Service Server with Active Directory

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Network Topology](#network-topology)
3. [Active Directory Overview](#active-directory-overview)
    - [What is Active Directory?](#what-is-active-directory)
    - [Why is Active Directory Used?](#why-is-active-directory-used)
    - [Active Directory Core Concepts](#active-directory-core-concepts)
4. [Security Implications](#security-implications)
5. [Setup Windows Server 2025](#setup-windows-server-2025)
    - [Step 1](#step-1)
    - [Step 2](#step-2)
6. [Disable Default Logoff](#disable-default-logoff)
7. [Disable CTRL + ALT + DEL](#disable-ctrl-alt-del)
8. [Assign Static IP Address](#assign-static-ip-address)
    - [Step 1](#step-1)
9. [Promote Active Directory to a Domain Controller](#promote-active-directory-to-a-domain-controller)
    - [Step 1](#step-1-1)
10. [Setup DNS For Internet Access](#setup-dns-for-internet-access)
11. [Setup DHCP](#setup-dhcp)
12. [Add User Accounts in Active Directory](#add-user-accounts-in-active-directory)

---

## Prerequisites
1. VirtualBox installed.
2. Virtual Machine with Windows 11 Server 2025 ISO has been configured and provisioned (the ISO should be attached to the new VM).

---

## Network Topology

---

## Active Directory Overview

### What is Active Directory?
Active Directory (AD) is a directory service developed by Microsoft that manages and organizes resources in a network. It acts as a centralized database to authenticate and authorize users and devices, making it the backbone of most Windows-based enterprise environments.

Key components:
- **Authentication**: Verifies user identity using credentials like username and password.
- **Authorization**: Grants or denies access to network resources based on permissions.
- **Management**: Centralizes control over users, computers, and other resources.

### Why is Active Directory Used?
Active Directory is widely used in enterprise environments to streamline and secure network management. It serves multiple purposes:
1. **Centralized Resource Management**: Enables administrators to manage users, devices, and permissions from a single location, reducing complexity.
2. **Scalability**: Handles environments ranging from small businesses to multinational corporations.
3. **Authentication and Authorization**: Provides a framework for verifying users and granting access using security protocols like Kerberos and LDAP.
4. **Group Policy Management**: Enforces security settings, deploys software, and manages updates across the network.
5. **Integration with Other Services**: Integrates seamlessly with services like Microsoft Exchange and Azure AD.

### Active Directory Core Concepts
1. **Domains**: A domain is a logical grouping of objects (users, devices, etc.) that share the same database and security policies. Example: `corp.project-x-dc.com` (used in this project).
2. **Domain Controllers (DCs)**: Servers that host the Active Directory database and perform authentication, authorization, and replication.
3. **Organizational Units (OUs)**: Containers within a domain used to organize objects logically.
4. **Objects**: Entities in AD, such as users, computers, printers, and groups.
5. **Groups**: 
   - Security Groups: Manage permissions to resources.
   - Distribution Groups: Used for email distribution.
6. **Forest and Trees**: A forest is the highest-level container encompassing multiple domains; a tree is a hierarchy of domains within a forest.
7. **Global Catalog (GC)**: A distributed data repository providing information about all objects in the forest for faster lookups.
8. **Trust Relationships**: Enable users in one domain to access resources in another domain.

---

## Security Implications
Active Directory is a prime target for attackers due to its central role in managing network resources. Misconfigurations or vulnerabilities can lead to significant security risks. Common threats include:
- **Credential Theft**: Techniques like Pass-the-Hash or Kerberoasting.
- **Privilege Escalation**: Exploiting misconfigured permissions to gain higher access levels.
- **Lateral Movement**: Attackers using AD to move through the network and target valuable systems.

Many organizations are transitioning to hybrid environments using Microsoft Entra ID (formerly Azure Active Directory), but this project will focus on on-premises infrastructure to fully control the setup.

---

## Setup Windows Server 2025

### Step 1
Select “Next” → “Install Windows 11” → Check the box → “Next”.

Select “Desktop Experience”.

Accept Microsoft’s End User License Agreement (EULA) → “Next”.  
Select “Disk 0 Unallocated Space” → “Create Partition”. Use the default “Size in MB” setting → “Apply”. Wait for three partitions to show up.

Select **Disk 0 Partition 3** (with the largest free space). Select “Install”. Wait for Windows Server 2025 to fully install. The VM should restart.

### Step 2
Set a password for the default Administrator account. Password is `(@Deeboodah1!)`.  
Refer to the “Project Overview” guide for more information on default usernames and passwords.

The login screen will appear. Navigate to the top of VirtualBox, go to **Input → Keyboard → Insert Ctrl-Alt-Del** to open the login prompt.

Choose “Required only” for sending diagnostic data to Microsoft.

After signing in, you should see the “Server Manager” Window. You can exit out of the dialog box to try Azure Arc.

---

## Disable Default Logoff
The default time for signing out of Windows Server 2025 is 5 minutes. Let’s change this.

Look up “Settings” in the Search bar → “System” → “Power” → Select the toggle under “Screen timeout” → Select “Never”.

---

## Disable CTRL + ALT + DEL
If you do not want to use **Input → Keyboard → Insert CTRL + ALT + DEL** each time, you can disable this setting.

Look up “Local Security Policy”.  
Navigate to the following folder tree → Look for **“Interactive logon…”** → Toggle from Disabled to Enabled → “Apply” → “OK”.

---

## Assign Static IP Address

### Step 1
Navigate to the **Control Panel** (Shortcut: **Windows+X**).  
Select “Network and Sharing Center”.  
Select “Change adapter settings”.

A window will pop up with a computer icon named “Ethernet”. Right-click this icon → “Properties”.  
Another box will open. Select **Internet Protocol Version 4 (TCP/IPv4)** → “Properties”.

Set this device to a static IP address:
- IP address: `10.0.0.05`
- Subnet mask: `255.255.255.0`
- Default gateway: `10.0.0.1`

---

## Promote Active Directory to a Domain Controller

### Step 1
Go back to **Server Manager** → “Add roles and features”.

Select “Next” for the next 3 boxes.  
Select **“Active Directory Domain Services”**, **“DHCP Server”**, **“DNS Server”**, **File and Storage Services**, and **Web Server (IIS)**.

Leave the defaults, select **Next**.  
Select **Next** until you get to the **Confirmation** tab. Select **Install**.

You can close the dialogue box while the features are installed.

A message notification will appear for configuring Active Directory. Select **“More”**.

Select **“Promote this server to a domain”**.  
Select **“Add a new forest”**, then enter a root domain name: `corp.project-x-dc.com`.  
Leave the default options, use the Administrator password for the **Directory Services Restore Mode (DSRM)**. Select **Next**.

Leave the **Create DNS delegation** box blank → **Next**.  
Leave **NetBIOS** as CORP, proceed with all other defaults until you reach the check screen.  
Allow the wizard to finish, then select **Install**. Let the server restart.

---

## Setup DNS For Internet Access

### Step 1
Go to **Server Manager** → **DNS** → Select the Server → Right-click → “DNS Manager”.

DNS Manager will appear → Right-click the domain → “Properties”.

Select the **“Forwarders”** tab → “Edit”.  
Add in **“8.8.8.8”** → **OK**.

---

## Setup DHCP

### Step 1
Navigate to **DHCP** → “DHCP Manager”.

Navigate to **IPv4** → “New Scope”.  
Add **project-x-scope**.

Enter the following addresses for leasing:
- Start IP address: `10.0.0.100`
- End IP address: `10.0.0.200`
- Subnet mask: `255.255.255.0`

---

## Add User Accounts in Active Directory

### Step 1
Navigate to **Server Manager** → “Tools” → “Active Directory Users and Computers”.

Navigate to **Users** → “New” → “User”.  
Add in the user information.  
Refer to the “Project Overview” guide for more information on default usernames and passwords.

Select **“User cannot change password”** → “Next”.  
Run through all default configuration settings.

---

### Take Snapshot!
