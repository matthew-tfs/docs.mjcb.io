---
title: "Subordinate CA Setup"
tags: [
    "ADCS",
    "Microsoft",
    "Windows"
]
geekdocNav: true
geekdocAnchor: false
weight: 30
---

The **TFS-CA01** server will be used for hosting the Subordinate Certificate Authority. The Subordinate CA server is used for issuing certificates to any device that requests one, whether it be automatically or manually requested. It will also be used to host all files that are required for the complete PKI for the domain, since the Offline Root CA has no network connections, as well as host the **OCSP** service for the domain.

{{< toc >}}

## 3.1 Subordinate Certificate Authority Server Setup ##

Provision and configure a new virtual machine for the **TFS-CA01** server using the following settings:

1. Create a new virtual machine with the following settings:
   * Virtual CPU - **2**
   * Virtual Memory - **4096 MB**
   * Virtual Hard Disk - **80 GB**
   * Virtual Floppy Drive - **1**
   * Virtual Network Adapters - **1**
2. Install **Windows Server 2022 Standard (Desktop Experience)** with the default options.
3. Set a password for the local Administrator account.
4. Set the hostname of the server to **TFS-CA01**. Restart the server to apply the change.
5. Configure the network settings for the **TFS-CA01** server. Ensure that you set the DNS settings to point to the **TFS-DC01** server, otherwise you will be unable to join the TFS Labs domain.
6. Join the **TFS-CA01** virtual machine to the **TFS Labs** domain. Restart the server to apply the change.

Once the **TFS-CA01** server has been joined to the **TFS Labs** domain, the computer object will need to be moved to the **TFS Labs\TFS Servers** OU. This task can be performed using the **Active Directory Users and Computers** console on the **TFS-DC01** server.

## 3.2 Create CNAME Records in DNS ##

By splitting the individual **AD CS** services into separate **CNAME** records, it would make it possible to split up the role in the future if needed. On the **TFS-DC01** server, create the following **CNAME** record pointing the **TFS-CA01** server:

1. Open the **DNS Manager** console.
2. Under the **DNS** Node, expand the **TFS-DC01** server and then expand **Forward Lookup Zones**. Select and the **corp.tfslabs.com** Zone. Right-click **New Alias (CNAME)**.
3. In **Alias name (uses parent domain if left blank)**, enter **PKI** as the name. In the **Fully qualified domain name (FQDN)** field, enter **tfs-ca01.corp.tfslabs.com.** and then click **OK**.
4. Close the **DNS Manager** console.

To validate that the **CNAME** record was created correctly, you should be able to ping the address. The ping request should fail because by default the Windows Firewall will deny the request, but the name should still resolve.

## 3.3 Subordinate CA CAPolicy.inf Installation ##

The **CAPolicy.inf** file is used to add configuration details to the certificate at the time of creation. On the **TFS-CA01** server, create a file in the **C:\Windows** folder called **CAPolicy.inf** (ensure that it is saved with the **inf** extension and not the **txt** extension, otherwise these settings will be ignored).

Copy the following contents into the **C:\Windows\CAPolicy.inf** file:

```ini
[Version]
Signature = "$Windows NT$"

[PolicyStatementExtension]
Policies = AllIssuancePolicy,InternalPolicy
Critical = FALSE

; AllIssuancePolicy is set to the OID of 2.5.29.32.0 to ensure all certificate templates are available.
[AllIssuancePolicy]
OID = 2.5.29.32.0

[InternalPolicy]
OID = 1.2.3.4.1455.67.89.5
Notice = "The TFS Labs Certification Authority is an internal resource. Certificates that are issued by this Certificate Authority are for internal usage only."
URL = http://pki.corp.tfslabs.com/cps.html

[Certsrv_Server]
; Renewal information for the Enterprise CA.
RenewalKeyLength = 4096
RenewalValidityPeriod = Years
RenewalValidityPeriodUnits = 5

; Disable support for issuing certificates with the RSASSA-PSS algorithm.
AlternateSignatureAlgorithm = 0

; Load all of the templates by default.
LoadDefaultTemplates = 1
```

{{< hint type=note title="OID Number" >}}
You can update the OID number in the **InternalPolicy** section for your deployment if it is required. The OID number in this example is used in Microsoft examples, but it should work for your organization if it is only ever going to be used internally. You can register for one if you would like to through IANA, and this is beyond the scope of this guide.
{{< /hint >}}

{{< hint type=important title="Signature Algorithm Support Issues" >}}
The **AlternateSignatureAlgorithm = 0** flag in the CAPolicy.inf file explicitly uses SHA256 for the algorithm instead of RSASSA-PSS. This can cause issues with some platforms (especially macOS and iOS) and by ensuring that it is disabled you shouldn't have issues with those certificates.
{{< /hint >}}

## 3.4 Active Directory Certificate Services Role Installation ##

The **Active Directory Certificate Services** role needs to be installed on the **TFS-CA01** server now that the **CAPolicy.inf** file is in place and ready to be used. To install the **AD CS** role, perform the following steps on the **TFS-CA01** server:

1. Open the **Server Manager** console, click on the **Manage** menu, and click on **Add Roles and Features** to start the installation wizard.
2. On the **Before you begin** screen, click the **Next** button to continue.
3. On the **Select installation type** screen, select the option for **Role-based or feature-based installation** and click the **Next** button to continue.
4. On the **Select destination server** screen, verify that the **TFSCA-01.corp.tfslabs.com** server is selected and click **Next**.
5. On the **Select server roles** screen, select the **Active Directory Certificate Services** option.
6. The installation wizard will ask to install the necessary management tools for the role. Click the **Add Features** button to continue.
7. On the **Select server roles** screen, click the **Next** button to continue.
8. On the **Select features** screen, click the **Next** button to continue.
9. On the **Active Directory Certificate Services** screen, click the **Next** button to continue.
10. On the **Select role services** screen, select the option for **Certification Authority** and **Certificate Authority Web Enrollment**. The installation wizard will ask to install the necessary features for **Certification Authority Web Enrollment**. Click the **Add Features** button to continue. Once that has been completed, click the **Next** button to continue.
11. On the **Web Server Role (IIS)** screen, click the **Next** button to continue.
12. On the **Select role services** screen, click the **Next** button to continue.
13. On the **Confirmation** screen, select the option to **Restart the destination server automatically if required**. When prompted with a warning about restarting the server, click the **Yes** button. Click the **Install** button to continue.
14. Once the installation is completed, click the **Close** button.

## 3.5 Active Directory Certificate Services Role Configuration ##

Once the **Active Directory Certificate Services** role has been added, it will need to be properly configured. In the process of configuring the role for the **TFS Labs** Domain, the following will be configured:

| **Subordinate Certificate Setting** | **Value**                                   |
|:------------------------------------|:--------------------------------------------|
| **Cryptographic Provider**          | RSA#Microsoft Software Key Storage Provider |
| **Key Length**                      | 4096 Bits                                   |
| **Signature Algorithm**             | SHA256RSA                                   |
| **Signature Hash Signature**        | SHA256                                      |
| **CA Common Name**                  | TFS Labs Enterprise CA                      |
| **Validity Period**                 | 5 Years (Configured from Root CA)           |

To configure the **AD CS** role, perform the following steps on the **TFS-CA01** server:

1. To begin the configuration of **Active Directory Certificate Services**, open the **Server Manager**. Click the **Notifications** icon in the upper-right hand corner and click the **Configure Active Directory Certificate Services on the destination server** link in the **Post-deployment Configuration** box.
2. On the **Credentials** screen, verify that the Administrator credentials is set to a **Domain Administrator Account** and click the **Next** button to continue. If you do not use a Domain Administrator account, then the installation will not allow you to install the **Active Directory Certificate Services** service correctly.
3. On the **Role Services** screen, select the options for **Certification Authority** and **Certification Authority Web Enrollment** and click **Next** to continue.
4. On the **Setup Type** screen, select the option for **Enterprise CA** and click the **Next** button to continue.
5. On the **CA Type** screen, ensure that the **Subordinate CA** option is selected and click the **Next** button to continue.
6. On the **Private Key** screen, verify that the **Create a new private key** option is selected. This is because this a new CA installation and the private key is not being restored from a previous server. Click the **Next** button to continue.
7. On the **Cryptography for CA** screen, make the following changes and then click the **Next** button to continue:
   * **Cryptographic Provider:** RSA#Microsoft Software Key Storage Provider
   * **Key Length:** 4096
   * **Hash Algorithm:** SHA256
8. On the **CA Name** screen, set the **Common Name (CN)** for the CA to **TFS Labs Enterprise CA** and click the **Next** button to continue.
9. On the **Certificate Request** screen, accept the default location for saving the **Certificate Request** file. It will be saved as **C:\TFS-CA01.corp.tfslabs.com_corp-TFS-CA01-CA.req**. Click the **Next** button to continue.
10. On the **CA Database** screen, make no changes to the database location and click the **Next** button to continue.
11. On the **Confirmation** screen, verify that the options are correct and click the **Configure** button to commit the changes.
12. On the **Results** screen, click the **Close** button.

Once the request file has been successfully generated, it will need to be copied to the **RootCAFiles** virtual floppy disk since the Root CA on **TFS-ROOT-CA** needs the request file to issue the **Subordinate Certificate**. Perform the following steps to copy the request file:

1. Add the **RootCAFiles** virtual floppy disk to the **TFS-CA01** virtual machine.
2. Browse to the **C:\ Drive** and copy the **TFS-CA01.corp.tfslabs.com_corp-TFS-CA01-CA.req** to the **A:\ Drive**.
3. Leave the **RootCAFiles** virtual floppy disk inserted.

## 3.6 Install the Root Certificate ##

On the **TFS-CA01** server, the TFS Labs Root certificate needs to be installed to complete the certificate chain after the Subordinate certificate has been issued:

1. Open the **A:\ Drive** folder.
2. Right-click on the **TFS Labs Certificate Authority.cer** file and select the **Install Certificate** option.
3. On the **Certificate Import Wizard** screen, select the **Local Machine** for the **Store Location** and click the **Next** button.
4. When prompted for the **Certificate Store** location, click the **Browse** button and select the **Trusted Root Certification Authorities** store and click the **OK** button. Click the **Next** button to continue.
5. Click the **Finish** button to complete the wizard.
6. Click the **OK** button to close the wizard.

## 3.7 Create the CertData Virtual Directory ##

On the **TFS-CA01** server, create a folder that will be used to host important certificate files for the domain users, workstations, and servers:

1. On the Root of the **C:\ Drive**, create a folder called **CertData** (C:\CertData).
2. Open the **A:\ Drive** and copy the **TFS Labs Certificate Authority.crl** and **TFS-ROOT-CA_TFS Labs Certificate Authority.crt** files to the **C:\CertData** folder.
3. Eject the **RootCAFiles** virtual floppy disk.
4. Open the **Internet Information Services (IIS) Manager** console.
5. On the **Connections** pane, expand **TFS-CA01** and then expand **Sites**.
6. Right-click on **Default Web Site** and select **Add Virtual Directory**.
7. On **Add Virtual Directory** page, in **Alias**, enter **CertData**. For the **Physical path**, enter **C:\CertData** and then click **OK**.
8. In the **Connections** pane, under the **Default Web Site**, ensure the **CertData** virtual directory is selected.
9. In the **CertData Home** pane, double-click on **Directory Browsing**.
10. In **Actions** pane click **Enable**.
11. Close the **Internet Information Services (IIS) Manager** console.

## 3.8 Enable Double Escaping ##

On the **TFS-CA01** server, enable Double Escaping in IIS to allow for proper CRL publication on the **TFS Labs** Domain.

1. Open an **Administrative Command Prompt**.
2. Type **cd C:\Windows\System32\inetsrv** and press **ENTER**.
3. Type following command and press **ENTER**:

```cmd
appcmd.exe set config /section:requestfiltering /allowdoubleescaping:true
```

4. Restart IIS service by typing **iisreset** and pressing **ENTER**.
5. Close the **Administrative Command Prompt**.

## 3.9 Subordinate Certificate Creation ##

Once the Subordinate CA has been configured and the request successfully generated, it is now time to complete the Subordinate CA Certificate by using the **TFS-ROOT-CA** server.

1. On the **TFS-ROOT-CA** server insert the **RootCAFiles** virtual floppy disk.
2. Copy the **A:\TFS-CA01.corp.tfslabs.com_corp-TFS-CA01-CA.req** file to the **C:\RootCA** folder.
3. On the **TFS-ROOT-CA** server open **Certification Authority** console.
4. Right-click on the **TFS Labs Certificate Authority** server, select **All Tasks** and click on **Submit new request...**.
5. Browse to the **C:\RootCA** folder and select the **TFS-CA01.corp.tfslabs.com_corp-TFS-CA01-CA.req** file that was copied from **TFS-CA01**. Click the **Open** button to continue.
6. Once the request has been submitted, go to the **Pending Requests** folder to see the certificate. It should be identified as **Request ID 2**, the first request being the self-signed Root certificate (not shown).
7. To issue the certificate, right-click on the request, select **All Tasks** and click on **Issue**.
8. Once the certificate has been issued, go to the **Issued Certificates** folder to see the certificate. It is still identified as **Request ID 2**. Double-click on the Certificate to open the **Certificate Properties** window.
9. On the **Details** tab, click the **Copy to File...** button.
10. On the first screen of the **Certificate Export Wizard**, click the **Next** button to continue.
11. On the **Export File Format** screen, select the **Cryptographic Message Syntax Standard - PKCS #7 Certificate (.P7B)** format. Select the option to **Include all certificates in the certification path if possible** and click the **Next** button.
12. For the file name, enter **C:\RootCA\TFS Labs Enterprise CA.p7b** and click **Next** to continue.
13. Click the **Finish** button to complete the wizard.
14. Copy the **C:\RootCA\TFS Labs Enterprise CA.p7b** file to the **A:\ Drive**.
15. Eject the **RootCAFiles** virtual floppy disk.
16. On the **TFS-CA01** server insert the **RootCAFiles** virtual floppy disk. Copy the **A:\TFS Labs Enterprise CA.p7b** file to the root of the **C:\ Drive**.
17. On the **TFS-CA01** server, open the **Certification Authority** console.
18. Right-click on the **TFS Labs Enterprise CA** server, go to **All Tasks** and select the option to **Install CA Certificate...**.
19. Browse to the root of the **C:\ Drive** and select the **TFS Labs Enterprise CA.p7b** file and click **Open**.
20. If there were no errors in installing the certificate, right-click on the **TFS Labs Enterprise CA** server, go to **All Tasks** and click the **Start Service** option.
21. The Subordinate certificate has now been installed successfully, and the Subordinate CA is now running.

Eject the **RootCAFiles** virtual floppy disk.

## 3.10 Set Maximum Certificate Age ##

Since all certificates that will be created by the Subordinate CA will only be valid for 1 year, the setting can be forced so that a Certificate Template does not attempt to sign a certificate for a longer time period. To adjust the validity period for issued certificates, perform the following steps on the **TFS-CA01** server:

1. To define the maximum age of any certificate that the Subordinate CA issues, run the following commands from an **Administrative Command Prompt**:

```cmd
certutil.exe -setreg CA\ValidityPeriodUnits 1
certutil.exe -setreg CA\ValidityPeriod "Years"
```

2. Once that is completed, restart the **Active Directory Certificate Services** service.
3. Close the **Administrative Command Prompt**.

## 3.11 Subordinate Certificate Authority CDP and AIA Configuration ##

Before the **Subordinate CA CDP and AIA Configuration** can be added to the **Subordinate Certificate**, the **CertEnroll** folder in IIS will need to have **Directory Browsing** enabled:

1. Open the **Internet Information Services (IIS) Manager** console.
2. On the **Connections** pane, expand **TFS-CA01** and then expand **Sites**.
3. In the **Connections** pane, under the **Default Web Site**, ensure the **CertEnroll** virtual directory is selected.
4. In the **CertEnroll** pane, double-click on **Directory Browsing**.
5. In **Actions** pane click **Enable**.
6. Close the **Internet Information Services (IIS) Manager** console.

Once the **Directory Browsing** option has been enabled, the **CDP and AIA entries** can now be added:

1. Open the **Certification Authority** console.
2. Right-click on **TFS Labs Enterprise CA** server and select **Properties**.
3. On the **Extensions** tab, verify that the **CRL Distribution Point (CDP)** extension is selected and click the **Add** button.
4. Under the **Location** field, enter the following address and click the **OK** button:

```
http://pki.corp.tfslabs.com/CertEnroll/<CaName><CRLNameSuffix><DeltaCRLAllowed>.crl
```

5. Back on the **Extensions** tab, verify that the **Include in CRLs. Clients use this to find Delta CRL locations.** and **Include in the CDP extension of issued certificates** options are selected for the location that was just entered.
6. On the **Extensions** tab, verify that the **Authority Information Access (AIA)** extension is selected and click the **Add** button.
7. Under the **Location** field, enter the following address and click the **OK** button:

```
http://pki.corp.tfslabs.com/CertEnroll/<ServerDNSName>_<CaName><CertificateName>.crt
```

8. Back on the Extensions tab, verify that the **Include in the AIA extension of issued certificates** option is selected for the location that was just entered.
9. Click the **OK** button to commit the changes. When prompted to restart **Active Directory Certificate Services**, click the **Yes** button.
10. Verify that the settings are correct by running the following commands in an **Administrative Command Prompt**:

```cmd
certutil.exe -getreg CA\CRLPublicationURLs
certutil.exe -getreg CA\CACertPublicationURLs
```
11. Close the **Administrative Command Prompt**.
12. In the **Certification Authority** console, right-click on **Revoked Certificates** under **TFS Labs Enterprise CA** and select **All Tasks > Publish**.
13. On the **Publish CRL** window, verify that **New CRL** is selected and click the **OK** button.

## 3.12 Enable Auditing on the Subordinate Certificate Authority ##

Auditing is recommended on any server running **Active Directory Certificate Services**. This will write logs to the Windows Event Log whenever a certificate is issued or revoked.

To enable auditing for the Subordinate CA server, perform the following steps on the **TFS-CA01** server:

1. Open the **Local Security Policy** console and modify the **Security Settings > Local Policies > Audit Policy > Audit object access** setting to audit **Success** and **Failure**.
2. Enable auditing for the CA by running the following command from an **Administrative Command Prompt**:

```cmd
certutil.exe -setreg CA\AuditFilter 127
```

3. Restart the **Active Directory Certificate Services** service:

```cmd
net stop CertSvc
net start CertSvc
```

4. Close the **Administrative Command Prompt**.

## 3.13 CPS Document Placeholder ##

The **Certification Practice Statement** is a document that is available to users who are using the Certificate Authority to inform them of important policies regarding the Certificate Authority. It's usage is optional in most cases, but it is good practice to have something in place for users.

To create a placeholder for the **CPS** document, perform the following steps on the **TFS-CA01** server:

1. Open the **C:\inetpub\wwwroot** folder and create a file called **cps.html** (C:\inetpub\wwwroot\cps.html).
2. Copy the following into the **cps.html** file:

```html
<html>
<head>
<title>TFS Labs Certification Practice Statement</title>
</head>
<body>
TFS Labs Certification Practice Statement
</body>
</html>
```

3. Test that the document is working correctly by opening a browser on the **TFS-CA01** server and going to the **http://pki.corp.tfslabs.com/cps.html** address.

## 3.14 Verify PKI Infrastructure ##

Before continuing with the **Online Responder** role configuration and the deployment of the Root and Subordinate certificates to the **TFS Labs** domain, verify that there are no issues with the **Active Directory Certificate Services** Configuration:

1. On the **TFS-CA01** server, open the **Enterprise PKI** console.
2. Under the **Enterprise PKI** node, click on the **TFS Labs Certificate Authority** server and check that the status of the **CA**, **AIA** and **CDP** is **OK**.

If there are no issues with the Certificate Authority configuration, then the initial Certificate Authority deployment is now complete.

## 3.15 Implementation File Cleanup ##

Once the initial phase of the Certificate Authority has been successfully implemented and completed, there are a few files that should be deleted as they are no longer required.

### 3.15.1 TFS-CA01 Server ###

Delete the following files on the **TFS-CA01** server:

* C:\TFS-CA01.corp.tfslabs.com_corp-TFS-CA01-CA.req
* C:\TFS Labs Certificate Authority.cer
* C:\TFS Labs Enterprise CA.cer
* C:\TFS Labs Enterprise CA.p7b

These files should all be present on the **C:\RootCA** folder on the **TFS-ROOT-CA** server and can remain on that server.

### 3.15.2 Virtual Floppy Disk Deletion ###

Depending on your virtualization platform, the location of the **RootCAFiles** virtual floppy disk will vary. This file also needs to be deleted. Ensure that if you setup BitLocker on the **TFS-ROOT-CA** server that you backup up the recovery key.

## 3.16 Root CA Shutdown ##

At this point in the Certificate Authority implementation the **TFS-ROOT-CA** server is no longer required to be powered on since the Subordinate certificate has been issued to the **TFS-CA01** server. The **TFS-ROOT-CA** server is no longer required and it should be powered shutdown. The Root CA server only needs to be powered on to renew the Root CRL once a year, and the instructions on how to renew the CRL can be found [here](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-2/#210-renewing-the-root-ca-crl).

{{< hint type=warning title="Root Certificate Authority Server" >}}
Ensure that you do not delete this virtual machine and ensure that it is being backed up correctly. If the Root CA server is deleted without the necessary PKI files 
{{< /hint >}}

## AD CS on Windows Server 2022 Guide ##

* [Introduction - AD CS on Windows Server 2022](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/)
* [Part 1 - Domain Controller and Workstation Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-1/)
* [Part 2 - Offline Root CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-2/)
* **Part 3 - Subordinate CA Setup**
* [Part 4 - Deploy Certificates](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-4/)
* [Part 5 - Online Responder Role Configuration](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-5/)
* [Part 6 - Private Key Archive and Recovery](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-6/)
* [Part 7 - Certificate Template Deployment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-7/)
* [Part 8 - Certificate Auto-Enrollment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-8/)
