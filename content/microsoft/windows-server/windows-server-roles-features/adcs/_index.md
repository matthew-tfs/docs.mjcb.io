---
title: "AD CS"
geekdocNav: true
geekdocAnchor: false
---

Active Directory Certificate Services is a role that has been available in Windows Server 2008 and later versions that is responsible for creating Certificate Authorities using Windows Server.

{{< toc >}}

## Active Directory Certificate Services Features ##

Active Directory Certificate Services is split up into six separate roles, which provides additional options for various deployment scenarios:

* **Certification Authority** - This role is used to issue and manage certificates, with the ability to create Root and Subordinate Certificate Authorities. Multiple Certificate Authorities can be linked together and tiered to form a complete PKI. Currently the AD CS role allows for the creation of Certificate Authorities with or without the use of Active Directory:
    * **Standalone CA** - These are servers that may or may not be members of an Active Directory domain and can operate without it entirely. A Standalone CA can be used in an online or offline state and is most often used as a Root Certificate Authority.
    * **Enterprise CA** - These are servers that are a member of an Active Directory domain and are typically used as a Subordinate or Issuing Certificate Authority. These types of servers are typically always online and available.
* **Certificate Enrollment Policy Web Service** - The Certificate Enrollment Policy Web Service role allows users and computers to request and obtain certificates when they are not members of an Active Directory domain or are located outside of the Active Directory network. It is used together with the Certificate Enrollment Web Service to issue certificates.
* **Certificate Enrollment Web Service** - The Certificate Enrollment Web Service role allows users and computers to enroll and renew certificates when they are not members of an Active Directory domain or are located outside of the Active Directory network. It is used together with the Certificate Enrollment Policy Web Service.
* **Certification Authority Web Enrollment** - The Certification Authority Web Enrollment role adds a simple web interface using Internet Information Services over the HTTPS protocol that allows users to request and manage certificates as well as retrieve Certificate Revocation Lists (CRLs).
* **Network Device Enrollment Service** - The Network Device Enrollment Service (NDES) role gives the AD CS role the ability to issue and manage certificates for network devices such as switches, routers, and firewalls. These types of devices typically do not have Active Directory accounts associated with them, so they are not able to automatically request certificates.
* **Online Responder** - The Online Responder role adds the OCSP functionality to Active Directory Certificate Services, which allows for rapid revocation of certificates in large environments.

## History of Active Directory Certificate Services ##

The predecessor to **Active Directory Certificate Services** was known as **Microsoft Certificate Server**. This version was originally released for Windows NT 4.0 Server as an optional feature in 1998 (with the Windows NT 4.0 Option Pack). It offered basic and limited Certificate Authority features to Windows, and was not easy to install or manage. There were subsequent versions of the **Microsoft Certificate Server** application released for Windows 2000 Server, Windows Server 2003 and finally for Windows Server 2003 R2.

When Windows Server 2008 was released, the **Microsoft Certificate Server** application was retired in favour of the new **Active Directory Certificate Services** role, and the role has continued to be updated since that initial release.

## AD CS Guides ##

I have created a few guides on this subject, as well as a few publications.

* [AD CS on Windows Server 2019 Guide](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/) - Free
* [AD CS on Windows Server 2022 Guide](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/) - Free
* [Building a Certificate Authority in Windows Server 2019](https://mjcb.io/publications/building-a-certificate-authority-in-windows-server-2019/) - Free
* [Practical Guide to PKI with Windows Server](https://mjcb.io/publications/practical-guide-to-pki-with-windows-server/) - Paid
