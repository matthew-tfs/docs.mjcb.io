---
title: "AD CS Final Steps"
tags: [
    "ADCS",
    "Microsoft",
    "Windows"
]
geekdocNav: true
geekdocAnchor: false
weight: 80
---

Once the Certificate Authority has been successfully implemented and completed, there are a few files that should be deleted and a few tasks that will need to be performed now and in the future.

{{< toc >}}

## 8.1 TFS-CA01 Server Cleanup ##

Delete the following files on the **TFS-CA01** server:

* C:\TFS-CA01.corp.tfslabs.com_corp-TFS-CA01-CA.req
* C:\TFS Labs Certificate Authority.cer
* C:\TFS Labs Enterprise CA.cer
* C:\TFS Labs Enterprise CA.p7b

These files should all be present on the **C:\RootCA** folder on the **TFS-ROOT-CA** server and can remain in place.

## 8.2 Virtual Floppy Disk ##

Depending on your virtualization platform, the location of the **RootCAFiles** virtual floppy disk will vary. This file also needs to be deleted.

Ensure that if you setup BitLocker on the **TFS-ROOT-CA** server, that you backup up the recovery key in the event that is required in the future.

{{< hint type=important title="BitLocker Recovery Key" >}}
Ensure that the BitLocker Recovery Key is properly backed up onto a different device before the end of the Certificate Authority implementation. The virtual floppy disk will be deleted once it is no longer required at the end of this guide.
{{< /hint >}}

## 8.3 Root CA Shutdown ##

Once the Certificate Authority has been successfully implemented, the Root CA needs to be powered off as it is no longer needed at this time. The **TFS-ROOT-CA** virtual machine will need to be powered on at least once every 52 weeks in order to update the CRL.

{{< hint type=warning title="Root Certificate Authority Server" >}}
Ensure that you do not delete this virtual machine and ensure that it is being backed up correctly. If the Root CA server is deleted without the necessary PKI files 
{{< /hint >}}

## 8.4 Renewing the Root CA CRL ##

{{< hint type=important title="Root CA CRL Renewal" >}}
Do not perform this step during the implementation of your Certificate Authority. This step is to be completed at least 2 weeks before the CRL expires as a maintenance task.
{{< /hint >}}

{{< hint type=note title="Root CA CRL Renewal Period" >}}
The Root CA CRL can be renewed at any point prior to the expiration date. In this guide the Root CA CRL is set to expire every 52 weeks, and it is recommended to give yourself at least a 2 week buffer for that renewal to be completed.

Unless there is a serious issue with your PKI, such as a Subordinate CA that needs to be revoked, there is no reason to renew and redistribute the Root CA CRL months prior to the expiration date.
{{< /hint >}}

One of the maintenance tasks that you will need to perform on your CA is that you will need to renew the Root CA CRL at least once a year. Once the implementation is completed you should setup a yearly recurring task to make sure this task is not forgotten. You should put in a recurring task in your calendar every 50 weeks to ensure that you renew the CRL in time.

Renewing a CRL before it expires is perfectly fine and will not affect the operation of a Certificate Authority.

Since the Root CA is an Offline CA, renewing the CRL file is the only time you will turn on the **TFS-ROOT-CA** server as there is no other tasks that are needed to be performed on the server. Once the CRL has been renewed, the server can be shutdown until the next time it is needed. You will need to create a virtual floppy disk to move the file, which will also need to be formatted before it can be used.

To renew the CRL on the Root CA, perform the following steps on the **TFS-ROOT-CA** server:

1. Open the **Certification Authority** console.
2. In the **Certification Authority** console, expand the **TFS Labs Certificate Authority**, right-click on **Revoked Certificates** under **TFS Labs Certificate Authority** and select **All Tasks > Publish**.
3. On the **Publish CRL** window, verify that **New CRL** is selected and click the **OK** button.
4. Close the **Certification Authority** console.

Once the CRL file has been renewed, use a virtual floppy disk to copy the file to the **TFS-CA01** server. To copy the CRL file to the **TFS-CA01** server, perform the following steps on the **TFS-ROOT-CA** server:

1. Add the virtual floppy disk to the **TFS-ROOT-CA** virtual machine (format the virtual floppy disk if necessary).
2. Copy the **C:\Windows\System32\CertSrv\CertEnroll\TFS Labs Certificate Authority.crl** file to the **C:\RootCA** folder. Overwrite the existing file that is already in that directory.
3. Copy the **C:\RootCA\TFS Labs Certificate Authority.crl** file to the **A:\ drive**.
4. Eject the virtual floppy disk.

Once the CRL file has been copied from the Root CA server, it can then be copied to the **TFS-CA01** server so that it can be utilized:

1. Add the virtual floppy disk to the **TFS-CA01** virtual machine.
2. Copy the **A:\TFS Labs Certificate Authority.crl** file to the **C:\CertData** folder. Overwrite the existing file that is already in that directory.
3. Eject the virtual floppy disk.

Now that the CRL file has been copied to the **TFS-CA01** server, the virtual floppy disk can be deleted as it is no longer required.

The final task is to publish the renewed Root CA CRL to Active Directory. This can be completed by running the following command from an **Administrative Command Prompt** on the **TFS-CA01** server:

```cmd
certutil.exe –addstore –f root "C:\CertData\TFS Labs Certificate Authority.crl"
```

Renewing the Root CA CRL file is a trivial process that should only take a few minutes to complete. The most difficult step is ensuring that you do not forget to renew the CRL.

## AD CS on Windows Server 2019 Guide ##

* [Introduction - AD CS on Windows Server 2019 Guide](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/)
* [Part 1 - Offline Root CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-1/)
* [Part 2 - Subordinate CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-2/)
* [Part 3 - Deploy Root and Subordinate Certificate](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-3/)
* [Part 4 - Certificate Revocation Policies](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-4/)
* [Part 5 - Configure Private Key Archive and Recovery](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-5/)
* [Part 6 - Certificate Template Deployment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-6/)
* [Part 7 - Certificate Auto-Enrollment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-7/)
* **Part 8 - AD CS Final Steps**
