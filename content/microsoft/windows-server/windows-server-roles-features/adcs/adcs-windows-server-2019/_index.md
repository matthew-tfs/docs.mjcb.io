---
title: "AD CS on Windows Server 2019"
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

{{< hint type=tip title="Archived" >}}
This guide is archived and will no longer be updated. It has been superseded with the [AD CS on Windows Server 2022](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/) guide.
{{< /hint >}}

{{< hint type=note title="Original Guide" >}}
This guide was originally posted on the [https://mjcb.io](https://mjcb.io/) website in March 2020. This guide has received updates to fix minor errors and to improve readability.
{{< /hint >}}

{{< toc >}}

## Goals of this Guide ##

The goal of this guide is to deploy an internal Two-Tier Certificate Authority and a Public Key Infrastructure (PKI) using **Active Directory Certificate Services** in Windows Server 2019. This provides multiple benefits to an organization, including features like:

* Utilizing SSL on internal servers and on internal websites.
* Replacing self-signed certificates on internal servers and devices.
* Increasing security for Remote Desktop Services on internal servers.
* Utilizing internal certificates for applications and services.
* Issuing internal certificates for VPN services.
* Issuing certificates for 802.1X.
* Allow for enhanced security for Active Directory with LDAPS.
* Using internal certificates for SSL decryption on firewalls or dedicated devices.

The procedure for creating a Certificate Authority has not changed considerably since Windows Server 2012 R2. Recent enhancements and changes with some vendors do require a few minor changes to allow for security changes with those vendors.

## Guide Sections ##

Since this is such a complicated subject there are eight parts to this guide. Here are the links to each part of the guide:

* [Introduction - AD CS on Windows Server 2019 Guide](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/)
* [Part 1 - Offline Root CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-1/)
* [Part 2 - Subordinate CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-2/)
* [Part 3 - Deploy Root and Subordinate Certificate](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-3/)
* [Part 4 - Certificate Revocation Policies](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-4/)
* [Part 5 - Configure Private Key Archive and Recovery](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-5/)
* [Part 6 - Certificate Template Deployment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-6/)
* [Part 7 - Certificate Auto-Enrollment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-7/)
* [Part 8 - AD CS Final Steps](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-8/)

## Environment Assumptions ##

All servers in this guide are using **Windows Server 2019 Standard (Desktop Experience)**, but this should work correctly using Windows Server 2016. In this guide, the **Active Directory Domain and Forest functional levels** are set to **Windows Server 2016**, but this should work for Windows Server 2012 R2 functional levels.

This guide should work using Hyper-V, VirtualBox or VMware. This guide does not assume any virtualization platform so there should not be any issues using any Virtualization platform.

{{< hint type=important title="Virtualization Platform" >}}
This guide was created using VMware Workstation 15 Pro (15.5.1 build-15018445) on Windows 10 Pro 1909 (Build 18363.657).
{{< /hint >}}

## Environment Design and Overview ##

This guide uses a simplified and basic server infrastructure which is the bare minimum that is required for Active Directory Certificate Services to operate correctly. It is technically possible to run Active Directory Certificate Services on the same server as a Domain Controller, but this is very bad practice and can have some unintended consequences if there is ever an issue with it. It is also incredibly insecure to always have your Root CA server available.

The example that is used in this guide is the **TFS Labs** domain (**corp.tfslabs.com**). It is basic in design, and there is a total of 3 servers and 1 workstation:

![TFS Labs Certificate Authority Infrastructure Overview](/images/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/ca-infrastructure-overview.png)

The virtual machines that are being used in this guide are using the following specifications:

| **Virtual Machine** | **Operating System** | **CPU** | **Memory** | **Hard Disk** | **IP Address**   |
|:--------------------|:---------------------|:--------|:-----------|:--------------|:-----------------|
| TFS-DC01            | Windows Server 2019  | 1       | 4096 MB    | 60 GB         | 192.168.0.210/24 |
| TFS-CA01            | Windows Server 2019  | 2       | 8192 MB    | 60 GB         | 192.168.0.211/24 |
| TFS-WIN10           | Windows 10 Pro       | 1       | 4096 MB    | 60 GB         | 192.168.0.212/24 |
| TFS-ROOT-CA         | Windows Server 2019  | 1       | 4096 MB    | 60 GB         | N/A              |

Here is breakdown of the servers and workstations in this environment:

* **TFS-DC01** the Domain Controller for the **TFS Labs** domain. It is also needed to allow for certificate distribution and for Group Policy updates to the **TFS Labs** domain. It is also the **CDP and AIA Publishing Location**
* **TFS-ROOT-CA** is the Offline Root CA, and it is only used to issue the Root certificate for the **TFS Labs** domain. It signs the certificate for the Subordinate Certificate Authority only and is left offline unless there is an issue with the Subordinate Certificate Authority. It is not a member of the **TFS Labs** domain and has no additional software or services installed on it. Once the implementation of the Certificate Authority is complete it can be shutdown (but not deleted).
* **TFS-CA01** is the Subordinate Certificate Authority and issues all certificates within the **TFS Labs** domain. It also handles the OCSP Role and CRL roles. It is a **TFS Labs** domain member.
* **TFS-WIN10** is a workstation that is a member of the **TFS Labs** domain, and it is used to ensure certificates that are issued by the two Certificate Authorities are operating correctly. It is also used to ensure that Group Policy deployment of these certificates are working correctly.

{{< hint type=important title="Active Directory Domain Setup" >}}
This guide assumes that you already know how to setup a basic Active Directory domain and a Domain Controller.
{{< /hint >}}

{{< hint type=important title="Windows 10 Pro Workstation" >}}
This guide assumes that you can configure a Windows 10 Pro virtual machine prior to starting this guide.
{{< /hint >}}

## Certificate Hierarchy Overview

For the certificates that will be issued for the **TFS Labs** domain, there will be one Root and one Subordinate certificate in a **Two-Tier Certificate Authority**:

![TFS Labs Certificate Authority Hierarchy](/images/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/ca-certificates-structure.png)

| **Certificate Type** | **Certificate Name**           | **Server Name** | **Validity Period** |
|:---------------------|:-------------------------------|:----------------|:--------------------|
| Root CA              | TFS Labs Certificate Authority | TFS-ROOT-CA     | 10 Years            |
| Subordinate CA       | TFS Labs Enterprise CA         | TFS-CA01        | 5 Years             |
| Certificate          | N/A                            | TFS-CA01        | 1 Years             |

The validity period for the issued certificates in the **TFS Labs** domain is set to the following:

* The Root CA certificate is set to expire after 10 years. This certificate is the Root of the entire PKI at **TFS Labs**. 10 years for the validity period is perfectly acceptable for a Root CA, and that server will need to be brought online once every 52 weeks to update the CRL for the Subordinate CA.
* The Subordinate CA certificate is set to expire after 5 years. This certificate is used to sign all certificates that are issued to the **TFS Labs** domain. Unlike the Root CA, it is always online.
* Any certificates that are issued from the Subordinate CA is limited to 1 year only. A lot of vendors have specifically restricted SSL lifetimes to 1 year only. This forces vendors to keep their SSL certificates up to date, and to make sure that modern security practices and technologies are being used.

## Design Considerations ##

In the deployment of Active Directory Certificate Services on the **TFS Labs** domain, the following design considerations will be made:

* The use of SHA-1 will not be used since it has been deprecated by online Certificate Authorities and by virtually every vendor. The Certificate Authority created at **TFS Labs** will use SHA-2 (SHA256) by default.
* Utilize an Offline Root CA for enhanced security.
* Utilize a Subordinate CA for issuing certificates to the **TFS Labs** domain. This will always be online and will be used to issue all certificates.
* The Root certificate will be valid for 10 years and the Subordinate certificate will be valid for 5 years. All issued certificates from the Subordinate CA will be valid for 1 year only.
* The Offline Root CA is only online for creating the Subordinate CA and is then shutdown and isolated from the network to keep it safe.
* Any files that need to be transferred to and from the Root CA is to be done with a virtual floppy disk. This will be deleted at the end of the implementation phase and when needed in the future a new one should be created.
* Auditing will be enabled on all servers that are performing certificate related tasks. This will write events to the Windows Event Log every time a certificate is issued, revoked, requested, etc.
* CNAME records will be used in the deployment to allow for the **TFS-CA01** server to be split up in the future if needed.

## Why Use an Offline CA? ##

There are a lot of very good reasons to use an Offline Root CA in your environment and it is pretty much expected in a Certificate Authority that is created today. Unauthorized access to your Certificate Authority can put your organization at risk and can cause a lot of headaches to fix it, especially if you depend heavily on a PKI for critical functions in your environment.

The Root CA is critical to your PKI, and you don't want to risk having the Root CA compromised and having your private keys leaked. This would effectively invalidate every single certificate in your organization.

The best way to protect the Root CA is always to have it be completely unavailable to people on the network. It isn't needed in day-to-day operations, so having it online is not necessary and can put it at unnecessary risk. This also means that it is not enough to just have it turned off until needed, it shouldn't be accessible by anyone even when it is temporarily powered on.

A lot of administrators don't even have a network connection to it and use virtual floppy disks to transfer data between it and other servers. It is cumbersome, but this happens so infrequently that it shouldn't be an issue. Some virtualization platforms allow for copy/paste, but that should be disabled for the Root CA to minimize the attack surface on it.

## Registered IANA Number ##

When you are dealing with an internal Certificate Authority you don't really need to worry about utilizing a registered IANA Number. This is only ever required if you are going to be using your Certificate Authority outside of your organization. This is beyond the scope of this guide, but it is an option should it be required for your organization.

## Active Directory Certificate Services Internal URLs ##

The following URLs will be in use once the Active Directory Certificate Services implementation has been completed:

| **Server (CNAME)** | **Role**                                   | **Address**                                |
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

## Disclaimer ##

I cannot guarantee that this guide will work in your environment, and I cannot take responsibility if this guide causes any potential issues in your environment. If you or anyone else has attempted to create a Certificate Authority in the past, you should check your Active Directory setup to see if you have any failed Certificate Authorities in there already. You should remove these first before starting this guide.

I cannot guarantee that there are no errors in this guide as well. I have implemented this exact same setup at multiple organizations without any major issues, but odd issues can always arise in a Windows Server infrastructure. So be prepared to have to "Google" your way out a few error messages in this guide.

As with everything else, you should build this out in a lab at least once prior to attempting this on a production environment. You should not attempt to implement this guide for your organization if you don't have a good understanding of how a Certificate Authority and PKI works.
