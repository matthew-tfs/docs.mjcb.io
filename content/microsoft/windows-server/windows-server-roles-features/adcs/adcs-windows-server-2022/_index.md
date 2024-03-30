---
title: "AD CS on Windows Server 2022"
tags: [
    "ADCS",
    "Guides",
    "Microsoft",
    "Windows"
]
geekdocNav: true
geekdocAnchor: false
geekdocCollapseSection: true
---

This is an updated version of the [AD CS on Windows Server 2019](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019-guide/) guide that is already available on this website. This guide reflects any changes that are present in Active Directory Certificate Services, Windows Server 2022, and Windows 11.

{{< toc >}}

## Goals of this Guide ##

The goal of this guide is to deploy an internal Two-Tier Certificate Authority (CA) and a Public Key Infrastructure (PKI) using **Active Directory Certificate Services (AD CS)** in **Windows Server 2022**.

An internal Certificate Authority provides multiple benefits to an organization, providing features such as:

* Utilizing SSL on internal servers and on internal websites.
* Replacing self-signed certificates on internal servers and devices.
* Increasing security for Remote Desktop Services on internal servers.
* Utilizing internal certificates for applications and services.
* Issuing internal certificates for VPN services.
* Issuing certificates for 802.1X.
* Allow for enhanced security for Active Directory with LDAPS.
* Using internal certificates for SSL decryption on firewalls or dedicated devices.

This guide aims to deploy a complete Two-Tier Certificate Authority solution, using as much functionality as possible using Active Directory Certificate Services, and other services that are already included with Windows Server.

Not all elements of this guide are required, and sections that are optional will be marked as such. A Certificate Authority has varying requirements, and those requirements vary based on the organization where one is deployed.

For ease of deployment of Active Directory Certificate Services, most of the configuration in this guide is performed using the GUI in Windows Server. There are only a few configuration tasks that are performed through the command line. It is possible to deploy AD CS entirely through the command line, but that is not covered in this guide.

## AD CS Guide Sections ##

Since deploying a Certificate Authority is such an in-depth subject, this guide has been split into multiple parts:

* [Part 1 - Domain Controller and Workstation Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-1/)
* [Part 2 - Offline Root CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-2/)
* [Part 3 - Subordinate CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-3/)
* [Part 4 - Deploy Certificates](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-4/)
* [Part 5 - Online Responder Role Configuration](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-5/)
* [Part 6 - Private Key Archive and Recovery](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-6/)
* [Part 7 - Certificate Template Deployment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-7/)
* [Part 8 - Certificate Auto-Enrollment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-8/)

Deploying a Certificate Authority is not a "one size fits all" kind of deployment. The needs will vary for every single organization. This is also why is it important to test the deployment of a Certificate Authority in a controlled environment prior to deploying it in production.

## Windows Versions and Virtualization ##

All servers in this guide are using **Windows Server 2022 Standard (Desktop Experience)**, but this guide should work correctly using Windows Server 2016 or [Windows Server 2019](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019-guide/). There is no difference in using the Datacenter version of Windows Server for the deployment. As stated earlier, the instructions provided in this guide assumes that there is a GUI available on the server, as the majority of the configuration uses various MMC consoles within Windows.

The workstation in this guide is using **Windows 11 Pro**, but this guide should work correctly using any of the currently supported versions of **Windows 10 Pro** as well. This guide is untested on any earlier client versions of Windows, and any legacy or retired versions of Windows.

In this guide, the **Active Directory Domain and Forest functional levels** for the test domain are set to **Windows Server 2016**, but this guide should also work for **Windows Server 2012 R2 Domain and Forest functional levels** as well.

{{< hint type=note title="Windows Server and Windows Client Versions" >}}
These instructions were created using Windows Server 2022 Standard (21H2 Build 20348.1726) and Windows 11 Pro (22H2 Build 22621.1702).
{{< /hint >}}

This guide should work perfectly fine using Hyper-V, VirtualBox, VMware or whatever virtualization solution you use. This guide does not assume any virtualization platform so there should not be any issues using any virtualization platform. The only system requirements for a virtualization platform for this guide are:

* x64 architecture support for Windows Server 2022 and Windows 11.
* Virtual floppy disk support (Hyper-V supports virtual floppy disks with Generation 1 virtual machines).
* Secure Boot and TPM 2.0 support (for Windows 11).

The system that is being used for testing should have sufficient disk space and memory as well (the requirements are listed below).

{{< hint type=note title="Virtualization Platform" >}}
The virtualization platform that was used to create this guide was Hyper-V on Windows Server 2022 Standard (21H2 Build 20348.1726).
{{< /hint >}}

## Environment Design and Overview ##

This guide uses a simplified and basic server infrastructure which is the bare minimum that is required for a Two-Tier Certificate Authority using AD CS. It is technically possible to run AD CS on the same server as a Domain Controller, but this is very bad practice and can have some unintended consequences if there is ever an issue with it.

It is also incredibly insecure to always have your Root CA server available. Security is an important consideration with AD CS, as having the Root certificate compromised will affect your entire PKI.

The example that is going to be used in this guide is the **TFS Labs** domain (**corp.tfslabs.com**). It is very basic in design, and there are a total of three servers and one workstation:

![TFS Labs Certificate Authority Infrastructure Overview](/images/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/ca-infrastructure-overview.png)

The virtual machines that are being used in this guide are using the following specifications:

| **Virtual Machine** | **Operating System** | **CPU** | **Memory** | **Hard Disk** | **IP Address**   |
|:--------------------|:---------------------|:--------|:-----------|:--------------|:-----------------|
| TFS-DC01            | Windows Server 2022  | 2       | 4096 MB    | 80 GB         | 192.168.0.210/24 |
| TFS-CA01            | Windows Server 2022  | 2       | 4096 MB    | 80 GB         | 192.168.0.211/24 |
| TFS-WIN11           | Windows 11 Pro       | 2       | 4096 MB    | 80 GB         | 192.168.0.212/24 |
| TFS-ROOT-CA         | Windows Server 2022  | 2       | 4096 MB    | 80 GB         | N/A              |

You can use whatever specifications for the virtual machines that you want, but be aware that provisioning the virtual machines with lower specifications can result in issues with installation and general performance. Windows 11 also has much more strict requirements than earlier versions of Windows, so you should be aware that provisioning it incorrectly can cause issues with installation.

Here is breakdown of the servers and workstations in this environment:

* **TFS-DC01** the Domain Controller for the **TFS Labs** domain. It is also needed to allow for certificate distribution and for Group Policy updates to the **TFS Labs** domain. It is also the **CDP and AIA Publishing Location**.
* **TFS-ROOT-CA** is the Offline Root Certificate Authority and is used to issue the Root certificate for the **TFS Labs** domain. It also signs the certificate for the Subordinate Certificate Authority and is left offline unless there is an issue with the Subordinate Certificate Authority or if the Root CRL needs to be renewed. It is not a member of the **TFS Labs** domain and has no additional software or services installed on it. Once the implementation of the Certificate Authority is complete it can be shutdown (but not deleted).
* **TFS-CA01** is the Subordinate Certificate Authority and Enterprise CA, and it issues all certificates within the **TFS Labs** domain. It also handles the **Online Responder (OCSP)** role and CRL distribution roles. It is a **TFS Labs** domain member.
* **TFS-WIN11** is a workstation that is a member of the **TFS Labs** domain, and it is used to ensure certificates that are issued by the Certificate Authority are operating correctly. It is also used to ensure that Group Policy deployment of these certificates is working correctly.

## Certificate Hierarchy Overview

For the certificates that will be issued for the **TFS Labs** domain, there will be one Root and one Subordinate certificate in a **Two-Tier Certificate Authority**:

![TFS Labs Certificate Authority Hierarchy](/images/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/ca-certificates-structure.png)

| **Certificate Type** | **Certificate Name**           | **Server Name** | **Validity Period** |
|:---------------------|:-------------------------------|:----------------|:--------------------|
| Root CA              | TFS Labs Certificate Authority | TFS-ROOT-CA     | 10 Years            |
| Subordinate CA       | TFS Labs Enterprise CA         | TFS-CA01        | 5 Years             |
| Issued Certificate   | N/A                            | TFS-CA01        | 1 Years             |

The validity period for the issued certificates in the **TFS Labs** domain is set to the following:

* The Root CA certificate is set to expire after 10 years. This certificate is the Root of the entire PKI at **TFS Labs**. 10 years for the validity period is perfectly acceptable for a Root CA, and that server will need to be brought online once every 52 weeks to update the CRL for the Subordinate CA.
* The Subordinate CA certificate is set to expire after 5 years. This certificate is used to sign all certificates that are issued to the **TFS Labs** domain. Unlike the Root CA, it is always online.
* Any certificates that are issued from the Subordinate CA is limited to 1 year only. A lot of vendors have specifically restricted SSL lifetimes to 1 year only. This forces vendors to keep their SSL certificates up to date, and to make sure that modern security practices and technologies are being used.

## Certificate Authority Design Considerations ##

In the deployment of Active Directory Certificate Services on the **TFS Labs** domain, the following design considerations will be made:

* The use of SHA-1 will not be used since it has been deprecated by online Certificate Authorities and by virtually every vendor. The Certificate Authority created at **TFS Labs** will use SHA-2 (SHA256) by default.
* Utilize an Offline Root CA for enhanced security.
* Utilize a Subordinate CA for issuing certificates to the **TFS Labs** domain. This will always be online and will be used to issue all certificates.
* The Root certificate will be valid for 10 years and the Subordinate certificate will be valid for 5 years. All issued certificates from the Subordinate CA will be valid for 1 year only.
* The Offline Root CA is only online for creating the Subordinate CA and is then shutdown and isolated from the network to keep it safe.
* Any files that need to be transferred to and from the Root CA is to be done with a virtual floppy disk. This will be deleted at the end of the implementation phase and when needed in the future a new one should be created.
* Auditing will be enabled on all servers that are performing certificate related tasks. This will write events to the Windows Event Log every time a certificate is issued, revoked, requested, etc.
* CNAME records will be used in the deployment to allow for the **TFS-CA01** server to be split up in the future if needed.
* A basic OU structure is used, as it simplifies certificate deployment using Group Policy.

## Why Use an Offline CA? ##

There are a lot of very good reasons to use an Offline Root CA in your environment and it is pretty much expected in a Certificate Authority that is created today. Unauthorized access to your Certificate Authority can put your organization at risk and can cause a lot of headaches to fix it, especially if you depend heavily on a PKI for critical functions in your environment.

The Root CA is critical to your PKI, and you don't want to risk having the Root CA compromised and having your private keys leaked. This would effectively invalidate every single certificate in your organization.

The best way to protect the Root CA is always to have it be completely unavailable to people on the network. It isn't needed in day-to-day operations, so having it online is not necessary and can put it at unnecessary risk. This also means that it is not enough to just have it turned off until needed, it shouldn't be accessible by anyone even when it is temporarily powered on.

A lot of administrators don't even have a network connection to it and use virtual floppy disks to transfer data between it and other servers. It is cumbersome, but this happens so infrequently that it shouldn't be an issue. Some virtualization platforms allow for copy/paste, but that should be disabled for the Root CA to minimize the attack surface on it.

## Registered IANA Number ##

When you are dealing with an internal Certificate Authority you don't really need to worry about utilizing a registered IANA Number. This is only ever required if you are going to be using your Certificate Authority outside of your organization. This is beyond the scope of this guide, but it is an option should it be required for your organization.

## Active Directory Certificate Services Internal URLs ##

The following URLs will be in use once the Active Directory Certificate Services implementation has been completed:

| **Server (CNAME)** | **Server Role**                            | **Address**                                |
|:-------------------|:-------------------------------------------|:-------------------------------------------|
| TFS-CA01 (N/A)     | IIS 10.0 HTTP Server Instance              | http​://tfs-ca01.corp.tfslabs.com/          |
| TFS-CA01 (N/A)     | Active Directory Web Enrollment Service    | https​://tfs-ca01.corp.tfslabs.com/CertSrv/ |
| TFS-CA01 (PKI)     | Certificate Practice Statement             | http​://pki.corp.tfslabs.com/cps.html       |
| TFS-CA01 (PKI)     | Root CA CRL                                | http​://pki.corp.tfslabs.com/CertData/      |
| TFS-CA01 (PKI)     | Enterprise CA CRL                          | http​://pki.corp.tfslabs.com/CertEnroll/    |
| TFS-CA01 (OCSP)    | Online Certificate Status Protocol         | http​://ocsp.corp.tfslabs.com/ocsp/         |
| TFS-CA01 (PKI)     | Root and Subordinate Certificates Download | http​://pki.corp.tfslabs.com/Certificates/  |

{{< hint type=note title="SSL Enabled Services" >}}
SSL is not used for securing many of the websites that a Certificate Authority uses. The one exception is the Active Directory Web Enrollment Service since it is used to securely submit a certificate request. This is because you cannot always assume that the device connecting to the HTTPS service has your certificate chain on it, and therefore the connection would not be secure anyways.
{{< /hint >}}

## Testing AD CS on an Existing Domain ##

There are a few ways to test an Active Directory Certificate Services deployment on an existing Active Directory domain without making changes to the production network. Since Active Directory Certificate Services makes changes to Active Directory, it is good practice to test the deployment prior to putting it into production. All you need for testing purposes is a Domain Controller that is a member of your Active Directory domain. This is not difficult to do, and there are a few options for an existing Active Directory domain:

* Clone an existing Domain Controller to a new virtual machine.
* Provision a new Domain Controller in a virtual machine and allow all Active Directory data to synchronize with it. 

Regardless of which method you choose to use for getting a Domain Controller for your existing Active Directory domain, ensure that you perform the following steps prior to attempting to deploy Active Directory Certificate Services:

1. Place the Domain Controller on a network segment that has no connections to your production network.
2. Update the DNS records on the Domain Controller to allow it to handle its own DNS resolution.
3. If the Domain Controller does not have all FSMO roles available, seize those roles to ensure that all FSMO roles are on that Domain Controller.

Once the Domain Controller has been properly isolated and setup for proper authentication and DNS resolution, you can then proceed to testing the deployment of Active Directory Certificate Services on your domain. This guide uses the TFS Labs domain as an example, and you can easily modify the deployment steps to fit with your existing Active Directory domain.

Testing the deployment of Active Directory Certificate Services in a controlled manner is strongly recommended prior to putting it into production. Active Directory domains that have been in production for extended periods of time tend to accumulate technical issues as they are upgraded and migrated over time. Testing the deployment of Active Directory Certificate Services in a controlled manner on a copy of a Domain Controller is something that you should seriously consider doing prior to deploying it in production.

## Disclaimer ##

I cannot guarantee that this guide will work in your environment, and I cannot take responsibility if this guide causes any potential issues in your environment. If you or anyone else has attempted to create a Certificate Authority in the past, you should check your Active Directory setup to see if you have any failed Certificate Authorities in there already. You should remove these first before starting this guide.

I cannot guarantee that there are no errors in this guide as well. I have implemented this exact same setup at multiple organizations without any major issues, but odd issues can always arise in a Windows Server infrastructure. So be prepared to have to "Google" your way out a few error messages in this guide.

As with everything else, you should build this out in a lab at least once prior to attempting this on a production environment. You should not attempt to implement this guide for your organization if you don't have a good understanding of how a Certificate Authority and PKI works.
