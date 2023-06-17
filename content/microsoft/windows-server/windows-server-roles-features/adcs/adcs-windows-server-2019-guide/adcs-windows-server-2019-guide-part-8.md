---
title: "8 - AD CS Final Steps"
tags: [
    "ADCS",
    "Microsoft",
    "Windows"
]
geekdocNav: true
geekdocAnchor: false
weight: 80
---

{{< toc >}}

## 8.1 Implementation File Cleanup ##

Once the Certificate Authority Implementation has been successfully implemented and completed there are a few files that should be deleted.

### 8.1.1 TFS-CA01 Server ###

Delete the following files on the **TFS-CA01** Server:

* C:\TFS-CA01.corp.tfslabs.com_corp-TFS-CA01-CA.req
* C:\TFS Labs Certificate Authority.cer
* C:\TFS Labs Enterprise CA.cer
* C:\TFS Labs Enterprise CA.p7b

These files should all be present on the **C:\RootCA** folder on the **TFS-ROOT-CA** Server. Those files don’t need to be deleted.

### 8.1.2 Virtual Floppy Disk ###

Depending on your Virtualization platform, the location of the **RootCAFiles** Virtual Floppy Disk will vary. This file also needs to be deleted. Ensure that if you setup BitLocker on the **TFS-ROOT-CA** Server that you backup up the recovery key.

## 8.2 Recurring Tasks ##

The only major task that you should need to perform on your PKI Infrastructure is that you will need to renew the CRL from the Root CA at least once a year. It is best that once the implementation is completed that you setup a yearly recurring task in order to make sure this task is not forgotten.

## 8.3 Root CA Shutdown ##

Once the Certificate Authority has been successfully implemented, the Root CA needs to be powered off as it is no longer needed. The **TFS-ROOT-CA** Virtual Machine will need to be powered on at least once every 52 weeks in order to update the CRL.

Ensure that you do not delete this Virtual Machine. If you do it will break your entire PKI and there will be no way of recovering from this.

## AD CS on Windows Server 2019 Guide ##

* [Introduction - AD CS on Windows Server 2019 Guide](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019-guide/)
* [Part 1 - Offline Root CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019-guide/adcs-windows-server-2019-guide-part-1/)
* [Part 2 - Subordinate CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019-guide/adcs-windows-server-2019-guide-part-2/)
* [Part 3 - Deploy Root and Subordinate Certificate](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019-guide/adcs-windows-server-2019-guide-part-3/)
* [Part 4 - Certificate Revocation Policies](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019-guide/adcs-windows-server-2019-guide-part-4/)
* [Part 5 - Configure Private Key Archive and Recovery](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019-guide/adcs-windows-server-2019-guide-part-5/)
* [Part 6 - Certificate Template Deployment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019-guide/adcs-windows-server-2019-guide-part-6/)
* [Part 7 - Certificate Auto-Enrollment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019-guide/adcs-windows-server-2019-guide-part-7/)
* **Part 8 - AD CS Final Steps**
