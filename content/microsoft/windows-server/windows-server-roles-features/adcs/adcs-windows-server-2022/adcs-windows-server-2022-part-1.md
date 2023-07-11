---
title: "Domain Controller and Workstation Setup"
tags: [
    "ADCS",
    "Microsoft",
    "Windows"
]
geekdocNav: true
geekdocAnchor: false
weight: 10
---

The **TFS Labs** domain that is used in this guide is configured using a single Domain Controller in the **corp.tfslabs.com (TFSLABS)** domain. This Active Directory domain is used for critical functions in the TFS Labs domain and is necessary for a proper Active Directory Certificates Services (AD CS) deployment to be completed. The **TFS-DC01** server will be used as a Domain Controller, and this section will focus on how to set it up correctly.

A test workstation will also be created which will be used to test certificate deployment. The **TFS-WIN11** workstation will be running Windows 11 Pro.

{{< toc >}}

## 1.1 Active Directory Server Setup ##

Provision and configure a new virtual machine for the **TFS-DC01** server using the following settings:

1. Create a new virtual machine with the following settings:
   * Virtual CPU - **2**
   * Virtual Memory - **4096 MB**
   * Virtual Hard Disk - **80 GB**
   * Virtual Floppy Drive - **0**
   * Virtual Network Adapters - **1**
2. Install **Windows Server 2022 Standard (Desktop Experience)** with the default options.
3. Set a password for the local Administrator account. **This password will later be used for the Domain Administrator account**.
4. Set the hostname of the server to **TFS-DC01**. Restart the server to apply the changes.
5. Configure the network settings for the **TFS-DC01** server. For the DNS settings, configure at least one external DNS server (such as Cloudflare, Google or OpenDNS).

Once the **TFS-DC01** server has been installed and configured, you can proceed to installing the Active Directory Domain Services role.

## 1.2 Active Directory Domain Services Role Installation ##

The **Active Directory Domain Services** role needs to be installed on the **TFS-DC01** server before the role can be configured.

To install the Active Directory Domain Services role, perform the following steps on the **TFS-DC01** server:

1. Open the **Server Manager** console, click on the **Manage** menu, and click on **Add Roles and Features** to start the installation wizard.
2. On the **Before you begin** screen, click the **Next** button to continue.
3. On the **Select installation type** screen, select the option for **Role-based or feature-based installation** and click **Next** to continue.
4. On the **Select destination server** screen, verify that the **TFS-DC01** server is selected and click **Next** to continue.
5. On the **Select server roles** screen, select the **Active Directory Domain Services** option.
6. The installation wizard will ask to install any additional features required for the **Active Directory Domain Services** role. Select the option to **Include management tools (if applicable)** to install the management tools for the role. Click the **Add Features** button to continue.
7. On the **Select server roles** screen, click the **Next** button to continue.
8. On the **Select features** screen, click the **Next** button to continue.
9. On the **Active Directory Domain Services** screen, click the **Next** button to continue.
10. On the **Confirm installation selections** screen, select the option to **Restart the destination server automatically if required**. When prompted with a warning about restarting the server, click the **Yes** button. Click the **Install** button to continue.
11. Once the installation is completed, click the **Close** button.

## 1.3 Active Directory Domain Services Role Configuration ##

The Active Directory Domain Services role will be configured with the following settings:

| **AD DS Attribute**         | **TFS Labs Setting** |
|-----------------------------|----------------------|
| **Forest Name**             | corp.tfslabs.com     |
| **NetBIOS Domain Name**     | TFSLABS              |
| **Forest Functional Level** | Windows Server 2016  |
| **Domain Functional Level** | Windows Server 2016  |

At the end of this section, a functioning Domain Controller for the TFS Labs domain will be setup.

To configure the AD DS role, perform the following steps on the **TFS-DC01** server:

1. To begin the configuration of **Active Directory Domain Services**, open the **Server Manager** console. You will notice that **AD DS** now shows up as being installed on the **TFS-DC01** server. Click the **Notifications** icon in the upper-right hand corner and click the **Promote this server to a domain controller** link in the **Post-deployment Configuration box**.
2. On the **Deployment Configuration** screen, select the option for **Add a new forest**. For the **Root domain name**, enter **corp.tfslabs.com** and click **Next** to continue.
3. On the **Domain Controller Options** screen, ensure that the **Forest functional level** and **Domain functional level** are both set to **Windows Server 2016**. Ensure that the options for **Domain Name System (DNS) server** and **Global Catalog (GC)** are both selected. For the **Directory Services Restore Mode (DSRM)** password, enter a complex password in both fields (ensure that you record this password somewhere). Once completed, click **Next** to continue.
4. On the **DNS Options** screen you can ignore the warning message about **DNS Delegation** (this is due to there being no existing DNS infrastructure at this point) and click **Next** to continue.
5. On the **Additional Options** screen, for the **NetBIOS domain name** enter **TFSLABS** and click **Next** to continue.
6. On the **Paths** screen, do not modify the default folder paths and click **Next** to continue.
7. On the **Review Options** screen, click **Next** to continue.
8. On the **Prerequisites Check** screen you can ignore any warning messages and click **Install** to continue.
9. On the **Installation** screen, **Active Directory Domain Services** will be configured, no actions are required.
10. On the **Results** screen, click the **Close** button to exit the wizard.
11. The **TFS-DC01** server will automatically restart when the configuration has completed.

The **TFS-DC01** server should restart automatically to complete the configuration of Active Directory. Once the Active Directory Domain Services configuration is complete, you will have a functioning Domain Controller for the TFS Labs domain. You should be able to login to the **TFS-DC01** server as the Domain Administrator using the same password set earlier for the local Administrator account.

## 1.4 Active Directory OU Creation ##

{{< hint type=note title="Active Directory OU Structure" >}}
If you are aware of how Group Policy works and know how to work with OUs, this section can be skipped.
{{< /hint >}}

A proper **Organizational Unit (OU)** structure makes deploying certificates using Group Policy considerably easier, and it makes Active Directory easier to manage. In this section, this basic OU structure will be created on the **TFS-DC01** server:

| **OU Folder**                 | **Purpose**                      |
|:------------------------------|:---------------------------------|
| **TFS Labs**                  | TFS Labs Root OU                 |
| **TFS Labs\TFS Servers**      | Stores all TFS Labs Servers      |
| **TFS Labs\TFS Users**        | Stores all TFS Labs Users        |
| **TFS Labs\TFS Workstations** | Stores all TFS Labs Workstations |

To create the OU structure using the **Active Directory Users and Computers** console, perform the following steps on the **TFS-DC01** server:

1. Open the **Active Directory Users and Computers** console.
2. Click on the **corp.tfslabs.com** domain in the left side of the window.
3. Right-click on the **corp.tfslabs.com** domain, select the **New** option, and select **Organizational Unit**.
4. In the **New Object - Organizational Unit** window, for the **Name**, enter **TFS Labs**. Ensure that the option to **Protect container from accidental deletion is selected** and click the **OK** button.
5. The **TFS Labs** OU should now appear in the **corp.tfslabs.com** domain.
6. Right-click on the **TFS Labs** OU, go to **New**, and select **Organizational Unit**.
7. In the **New Object - Organizational Unit** window, for the **Name**, enter **TFS Servers**. Ensure that the option to **Protect container from accidental deletion is selected** and click **OK**.
8. Right-click on the **TFS Labs** OU, go to **New**, and select **Organizational Unit**.
9. In the **New Object - Organizational Unit** window, for the **Name**, enter **TFS Users**. Ensure that the option to **Protect container from accidental deletion is selected** and click **OK**.
10. Right-click on the **TFS Labs** OU, go to **New**, and select **Organizational Unit**.
11. In the **New Object - Organizational Unit** window, for the **Name**, enter **TFS Workstations**. Ensure that the option to **Protect container from accidental deletion is selected** and click **OK**.
12. There should now be three OUs within the **TFS Labs** OU.
13. Close the **Active Directory Users and Computers** console.

## 1.5 Active Directory Test Account Creation ##

{{< hint type=note title="Active Directory Test Accounts" >}}
If you do not require any test accounts and are comfortable using the Domain Administrator account for testing, this section can be skipped. Ensure that the Administrator account has an e-mail address configured.
{{< /hint >}}

To create a test user account using the Active Directory Users and Computers console, perform the following steps on the **TFS-DC01** server:

1. Open the **Active Directory Users and Computers** console.
2. Click on the **corp.tfslabs.com** domain, and then click the arrow to the left of the domain to expand the domain structure.
3. Expand the **TFS Labs** OU and select the **TFS Users** OU.
4. Right-click on the **TFS Users** OU, select **New**, and then select **User**.
5. On the **New Object - User** window, enter the details for the user account and click **Next** to continue.
6. On the next window, enter a password for the user account. You can uncheck the option for **User must change password at next logon** option, as this is not necessary for testing purposes. Click the **Next** button to continue.
7. On the next window, ensure that the account details are correct and click the **Finish** button to continue.
8. To ensure that there are no issues generating user certificates later in this book, the email addresses of the users will need to be configured on each account. Right-click on one of the user accounts and select the **Properties** option.
9. On the **User Properties** window, on the **General** tab, enter the **E-mail** address for the user account. Click the **Apply** button and then click the **OK** button. Repeat this step for all user accounts until the email address is added to all accounts.
10. Close the **Active Directory Users and Computers** console.

## 1.6 Workstation Creation and Domain Join ##

{{< hint type=note title="Windows 11 Test Workstation" >}}
If you do not require a test workstation and are comfortable using the servers for testing, this section can be skipped. If you created any test user accounts they may encounter issues when logging into any of the servers.
{{< /hint >}}

A test workstation is recommended for validating the deployment of AD CS, as well as the automatic deployment of certificates. Running Windows 11 on Hyper-V requires additional configuration options due to changes to the system requirements for that Operating System. For information on how to install Windows 11 on Hyper-V, refer to this [guide](/microsoft/windows-client/windows-11/windows-11-hyper-v/).

{{< hint type=important title="Windows 11 System Requirements" >}}
Windows 11 is strict on the types of accounts that can be created when the Operating System is installed and configured. Windows 11 will try to force the creation of an account using a Microsoft account, which requires additional configuration. The method that you use to create a local account is beyond the scope of this guide.
{{< /hint >}}

Provision and configure a new virtual machine for the **TFS-WIN11** workstation using the following settings:

1. Create a new virtual machine with the following settings:
   * Virtual CPU - **2**
   * Virtual Memory - **4096 MB**
   * Virtual Hard Disk - **80 GB**
   * Virtual Floppy Drive - **0**
   * Virtual Network Adapters - **1**
2. Depending on the virtualization platform that you are using, perform any additional configuration that is needed to enable **Secure Boot** and **TPM 2.0**.
3. Install **Windows 11 Pro** with the default options.
4. Set the hostname of the workstation to **TFS-WIN11**. Restart the workstation to apply the changes.
5. Configure the network settings for the **TFS-WIN11** workstation. Ensure that the DNS server is set to the **TFS-DC01** server, otherwise it will not be possible to join it to the **TFSLABS** domain.
6. Join the **TFS-WIN11** workstation to the **TFSLABS** domain. Restart the workstation to apply the change.
7. Login to a domain account on the **TFS-WIN11** workstation to validate that it is working correctly.

Once the **TFS-WIN11** workstation has been joined to the **TFS Labs** domain, the computer object will need to be moved to the **TFS Labs\TFS Workstations** OU. This task can be performed using the **Active Directory Users and Computers** console on the **TFS-DC01** server.

## AD CS on Windows Server 2022 Guide ##

* [Introduction - AD CS on Windows Server 2022](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/)
* **Part 1 - Domain Controller and Workstation Setup**
* [Part 2 - Offline Root CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-2/)
* [Part 3 - Subordinate CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-3/)
* [Part 4 - Deploy Certificates](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-4/)
* [Part 5 - Online Responder Role Configuration](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-5/)
* [Part 6 - Private Key Archive and Recovery](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-6/)
* [Part 7 - Certificate Template Deployment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-7/)
* [Part 8 - Certificate Auto-Enrollment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-8/)
