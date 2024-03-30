---
title: "Offline Root CA Setup"
tags: [
    "ADCS",
    "Microsoft",
    "Windows"
]
geekdocNav: true
geekdocAnchor: false
weight: 20
---

The **TFS-ROOT-CA** server will be used for hosting the Offline Root Certificate Authority. The **TFS-ROOT-CA** server is only ever used for issuing Subordinate certificates to other **TFS Labs** domain servers and is also used to revoke or add new Subordinate certificates if necessary. It is also used to refresh the Root CRL at least once a year, which means it needs to be powered on at least once a year to complete that task.

The **TFS-ROOT-CA** server is setup as a standalone Windows Server and is never meant to be a member of an Active Directory domain, or even have any network connections to it. This means that it requires some local security modifications that are normally handled through Group Policy from Active Directory. Since there is no connection to Active Directory, these changes will need to be applied locally.

Since there are no network connections to the server, it is recommended to use a virtual floppy disk to transfer files to and from the server.

{{< toc >}}

## 2.1 Virtual Floppy Disk Creation ##

Since there are no network connections to and from this virtual machine, create a virtual floppy disk that will be used for transferring files to and from the **TFS-ROOT-CA** server. Name the file **RootCAFiles** (the file extension will vary depending on the virtualization platform being used) and store it in a location that will be available for all virtual machines. The first time that the virtual floppy disk is inserted into one of the virtual machines it will need to be formatted with the default settings.

{{< hint type=note title="Hyper-V Virtual Floppy Drives" >}}
In Hyper-V, a virtual floppy drive is automatically added to a virtual machine if it is created as a **Generation 1 VM**. A virtual floppy disk is stored using the **.vfd** extension.
{{< /hint >}}

## 2.2 Root Certificate Authority Server Setup ##

Provision and configure a new virtual machine for the **TFS-ROOT-CA** server using the following settings:

1. Create a new virtual machine with the following settings:
   * Virtual CPU - **2**
   * Virtual Memory - **4096 MB**
   * Virtual Hard Disk - **80 GB**
   * Virtual Floppy Drive - **1**
   * Virtual Network Adapters - **0**
2. Install **Windows Server 2022 Standard (Desktop Experience)** with the default options.
3. Set a password for the local Administrator account. The local Administrator account will optionally be renamed to **CA-Admin** in a further step in this section.
4. Set the hostname of the server to **TFS-ROOT-CA**. Restart the server to apply the change.
5. Set the server to be a member of the **TFS-CA** workgroup instead of a domain. Restart the server to apply the change.
6. There are no network adapters on the **TFS-ROOT-CA** server, so there is no configuration required.

{{< hint type=note title="Windows Server Activation" >}}
If activation is required for this server, the telephone option would be needed to accomplish this since there is no network connection available.
{{< /hint >}}

Once the server has been installed and configured, the **TFS-ROOT-CA** server needs to be prepared to allow for files to be transferred to and from the server. Perform the following steps on the **TFS-ROOT-CA** server:

1. On the root of the **C:\ Drive** create a folder called **RootCA** (C:\RootCA). This folder will store the Root certificate, Subordinate certificate and other necessary certificate files that are needed during the entire implementation process.
2. Insert the **RootCAFiles** virtual floppy disk into the virtual floppy drive. Format the floppy disk with the default settings. Eject the virtual floppy disk when completed.

### 2.2.1 Root Certificate Authority Security Modifications ##

{{< hint type=note title="Local Security Modifications" >}}
If you are not interested in modifying any of the local security settings on the Root CA server, this section can be skipped.
{{< /hint >}}

Since the **TFS-ROOT-CA** server is not a member of an Active Directory domain, there are several local security settings that should be configured to secure the server:

1. On the **TFS-ROOT-CA** server, open the **Local Group Policy Editor** console.
2. Modify the **Local Computer Policy > Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options** settings to match the following options:
   * **Accounts: Administrator account status** - Enabled
   * **Accounts: Block Microsoft accounts** - Users can't add or log on with Microsoft Accounts
   * **Accounts: Guest account status** - Disabled
   * **Accounts: Limit local account use of blank passwords to Console logon only** - Enabled
   * **Accounts: Rename administrator account** - CA-Admin
   * **Accounts: Rename guest account** - Administrator
   * **Interactive Logon: Don't display last signed-in** - Enabled
   * **Interactive Logon: Don't display username at sign-in** - Enabled
   * **Network security: Do not store LAN Manager hash value on next password change** - Enabled
   * **Network security: LAN Manager authentication level** - Send NTLMv2 response only. Refuse LM & NTLM

Additionally, the password settings should be configured to be more secure from the default options:

1. On the **TFS-ROOT-CA** server, open the **Local Security Policy** console.
2. Modify the **Security Settings > Account Policies > Password Policy** settings to match the following options:
   * **Enforce password history** - 24 passwords remembered
   * **Maximum password age** - 90 days
   * **Minimum password age** - 1 days
   * **Minimum password length** - 14 characters
   * **Password must meet complexity requirements** - Enabled
   * **Store passwords using reversible encryption** - Disabled
3. Modify the **Security Settings > Account Policies > Account Lockout Policy** settings to match the following options:
   * **Account lockout threshold** - 5 invalid login attempts
   * **Account lockout duration** - 30 minutes
   * **Reset account lockout counter after** - 30 minutes

Restart the **TFS-ROOT-CA** server to apply the updated policy and security settings.

{{< hint type=important title="Local Administrator Account" >}}
Once this step is completed the local Administrator account is renamed to **CA-Admin**. Use that account to login to the server and complete the installation of AD CS.
{{< /hint >}}

### 2.2.2 Root Certificate Authority BitLocker Encryption ###

{{< hint type=note title="BitLocker Encryption" >}}
If you are not interested in enabling BitLocker on the Root CA server, this section can be skipped.
{{< /hint >}}

Enable BitLocker to secure the **TFS-ROOT-CA** virtual machine while it is powered off or being stored in another location. To install the BitLocker role, perform the following steps on the **TFS-ROOT-CA** server:

1. Open the **Server Manager** console, click on the **Manage** menu, and click on **Add Roles and Features** to start the installation wizard.
2. On the **Before you begin** screen, click the **Next** button to continue.
3. On the **Select installation type** screen, select the option for **Role-based or feature-based installation** and click the **Next** button to continue.
4. On the **Select destination server** screen, verify that the **TFS-ROOT-CA** server is selected and click the **Next** button to continue.
5. On the **Select server roles** screen, click the **Next** button to continue.
6. On the **Select features** screen, select the **BitLocker Drive Encryption** Feature. The installation wizard will ask to install the necessary management tools for the role. Click the **Add Features** button to continue.
7. On the **Select features** screen, click the **Next** button to continue.
8. On the **Confirm installation selections** screen, select the option to **Restart the destination server automatically if required**. When prompted with a warning about restarting the server, click the **Yes** button (the server must restart to continue). Click the **Install** button to continue.
9. Once the **TFS-ROOT-CA** server has restarted click the **Close** button on the **Installation Progress** screen.

Once the BitLocker role has been installed, there are a few local policy changes that need to be made to allow for BitLocker to operate in a virtual environment.

Open the **Local Group Policy Editor** console and modify the **Computer Configuration > Administrative Templates > Windows Components > BitLocker Drive Encryption > Operating System Drives** settings to match these settings:

   * **Require additional authentication at startup** - Enabled
       * **Allow BitLocker without a compatible TPM (requires a password or a startup key on a USB flash drive)** - Enabled
       * **Configure TPM startup:** Allow TPM
       * **Configure TPM startup PIN:** Allow startup PIN with TPM
       * **Configure TPM startup key:** Allow startup key with TPM
       * **Configure TPM startup key and PIN:** Allow startup key and PIN with TPM

Once the local policy changes have been made, BitLocker can be enabled on the **TFS-ROOT-CA** server. Perform the following steps to enable BitLocker:

1. Insert the **RootCAFiles** virtual floppy disk.
2. Open **File Explorer** and go to **This PC**. Right-click on the **C:\ Drive** and select **Turn on BitLocker**.
3. On the **BitLocker Drive Encryption setup** screen click the **Next** button to continue.
4. On the **Preparing your drive for BitLocker** screen click the **Next** button to continue.
5. On the **BitLocker Drive Encryption setup** screen, click the **Next** button to continue.
6. On the **Choose how to unlock your drive at startup** screen, select the option to **Enter a password**.
7. On the **Create a password to unlock this drive** screen, enter the password that you want to use to unlock the drive at boot up. Make sure that you do not forget this password as it will require the recovery keys to get back into the **TFS-ROOT-CA** virtual machine. Ensure that you use a complex password for this and make it at least 14 characters in length. Click the **Next** button to continue.
8. On the **How do you want to back up your recovery key?** screen, select the option to **Save to a file**. Save the file to the **A:\ Drive** (virtual floppy disk). Click the **Next** button to continue.
9. Eject the **RootCAFiles** virtual floppy disk and then backup the recovery key to another device before you continue. **This is critical in case there is an issue with the BitLocker password.**
10. On the **Choose how much of your drive to encrypt** screen, select the option to **Encrypt entire drive (slower but best for PCs and drives already in use)** and click the **Next** button.
11. On the **Choose which encryption mode to use** screen, select the option for **New encryption mode (best for fixed drives on this devices)** and click the **Next** button to continue.
12. On the **Are you ready to encrypt this drive?** screen, ensure that the **Run BitLocker system check box** is selected and then click the **Continue** button.
13. When prompted, restart the **TFS-ROOT-CA** server. If prompted, ensure that the virtual floppy disk has been removed. Also ensure that the **Windows Server 2022** install media has also been removed.
14. Enter the BitLocker password that you set for the drive to ensure that it is working correctly.
15. Login to the **TFS-ROOT-CA** server, you should receive a notification that the drive is encrypting. You can also check the status of BitLocker at any time by going to **File Explorer**, then to **This PC**, right-clicking on the **C:\ Drive** and selecting the **Manage BitLocker** option.

The amount of time required for encrypting the drive will vary, but you can continue with the configuration of the Root CA server while it is encrypting.

{{< hint type=important title="BitLocker Recovery Key" >}}
Ensure that the BitLocker Recovery Key is properly backed up onto a different device before the end of the Certificate Authority implementation. The virtual floppy disk will be deleted once it is no longer required at the end of this guide.
{{< /hint >}}

## 2.3 Root CA CAPolicy.inf Installation ##

The **CAPolicy.inf** file is used to add configuration details to the certificate at the time of creation. On the **TFS-ROOT-CA** server, create a file in the **C:\Windows** folder called **CAPolicy.inf** (ensure that it is saved with the **inf** extension and not the **txt** extension, otherwise these settings will be ignored).

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
; Renewal information for the Root CA.
RenewalKeyLength = 4096
RenewalValidityPeriod = Years
RenewalValidityPeriodUnits = 10

; Disable support for issuing certificates with the RSASSA-PSS algorithm.
AlternateSignatureAlgorithm = 0

; The CRL publication period is the lifetime of the Root CA.
CRLPeriod = Years
CRLPeriodUnits = 10

; The option for Delta CRL is disabled since this is a Root CA.
CRLDeltaPeriod = Days
CRLDeltaPeriodUnits = 0
```

{{< hint type=note title="OID Number" >}}
You can update the OID number in the **InternalPolicy** section for your deployment if it is required. The OID number in this example is used in Microsoft examples, but it should work for your organization if it is only ever going to be used internally. You can register for one if you would like to through IANA, and this is beyond the scope of this guide.
{{< /hint >}}

{{< hint type=important title="Signature Algorithm Support Issues" >}}
The **AlternateSignatureAlgorithm = 0** flag in the CAPolicy.inf file explicitly uses SHA256 for the algorithm instead of RSASSA-PSS. This can cause issues with some platforms (especially macOS and iOS) and by ensuring that it is disabled you shouldn't have issues with those certificates.
{{< /hint >}}

## 2.4 Active Directory Certificate Services Role Installation ##

Once the **TFS-ROOT-CA** server has been installed and configured properly, the **Active Directory Certificate Services** role needs to be installed. To install the **AD CS** role, perform the following steps on the **TFS-ROOT-CA** server:

1. Open the **Server Manager** console, click on the **Manage** menu, and click on **Add Roles and Features** to start the installation wizard.
2. On the **Before you begin** screen, click the **Next** button to continue.
3. On the **Select installation type** screen, select the option for **Role-based or feature-based installation** and click **Next** to continue.
4. On the **Select destination server** screen, verify that the **TFS-ROOT-CA** server is selected and click Next.
5. On the **Select server roles** screen, select the **Active Directory Certificate Services** option. The installation wizard will ask to install the necessary management tools for the role. Click the **Add Features** button to continue.
6. On the **Select server roles** screen, click the **Next** button to continue.
7. On the **Select features** screen, click the **Next** button to continue.
8. On the **Active Directory Certificate Services** screen, click the **Next** button to continue.
9. On the **Role Services** screen, select the option for **Certification Authority** and click the **Next** button to continue.
10. On the **Confirmation** screen, select the option to **Restart the destination server automatically if required**. When prompted with a warning about restarting the server, click the **Yes** button. Click the **Install** button to continue.
11. Once the installation is completed, click the **Close** button.

## 2.5 Active Directory Certificate Services Role Configuration ##

Once the **Active Directory Certificate Services** role has been added, it will need to be configured. In the process of configuring the role for the **TFS Labs** Domain, the following Root certificate will be created:

| **Root Certificate Setting** | **Value**                                   |
|:-----------------------------|:--------------------------------------------|
| **Cryptographic Provider**   | RSA#Microsoft Software Key Storage Provider |
| **Key Length**               | 4096 Bits                                   |
| **Signature Algorithm**      | SHA256RSA                                   |
| **Signature Hash Signature** | SHA256                                      |
| **CA Common Name**           | TFS Labs Certificate Authority              |
| **Validity Period**          | 10 Years                                    |

To configure the **AD CS** role, perform the following steps on the **TFS-ROOT-CA** server:

1. To begin the configuration of **Active Directory Certificate Services** on **TFS-ROOT-CA**, open the **Server Manager** console. Click the **Notifications** icon in the upper-right hand corner and click the **Configure Active Directory Certificate Services on the destination server** link in the **Post-deployment Configuration** box.
2. On the **Credentials** screen, verify that the Administrator credentials is set to **TFS-ROOT-CA\CA-Admin** and click **Next** to continue. If you did not change the default Administrator account name, then it should be set to **TFS-ROOT-CA\Administrator**.
3. On the **Select Role Services to configure** screen, select the option for **Certification Authority** and click the **Next** button to continue.
4. On the **Specify the setup type of the CA** screen, the option for **Standalone CA** should be selected. The option for Enterprise CA is not available since this server is not a domain server. Click the **Next** button to continue.
5. On the **Specify the type of the CA** screen, ensure that the **Root CA** option is selected (the Subordinate CA option will be used later for the Enterprise CA). Click the **Next** button to continue.
6. On the **Specify the type of the private key** screen, verify that the **Create a new private key** option is selected. This is because this a new CA installation and the private key is not being restored from a previous server. Click the **Next** button to continue.
7. On the **Cryptography for CA** screen, make the following changes and then click the **Next** button to continue:
   * **Cryptographic Provider:** RSA#Microsoft Software Key Storage Provider
   * **Key Length:** 4096
   * **Hash Algorithm:** SHA256
8. On the **Specify the name of the CA** screen, set the **Common Name (CN)** for the CA to **TFS Labs Certificate Authority** and click the **Next** button to continue.
9. On the **Specify the validity period** screen, set the validity period to **10 Years** and click the **Next** button to continue.
10. On the **Specify the database locations** screen, make no changes to the database location and click the **Next** button to continue.
11. On the **Confirmation** screen, verify that the options are correct and click the **Configure** button to commit the changes.
12. On the **Results** screen, click the **Close** button.

{{< hint type=important title="Certificate Validity Period" >}}
It is not advised to have the Root Certificate and the Subordinate Certificate set to have the same validity period. For example, if both certificates have a 5 year expiration date, then it is possible that the Root Certificate will expire before the Subordinate Certificate since it was signed first. If this happens it will be extremely difficult to re-sign both certificates because they will both be invalid at the same time.
{{< /hint >}}

## 2.6 Root Certificate Authority CRL Configuration ##

The CRL configuration for the Root CA is configured in this step to give greater control over when this takes place, and the time is extended to 52 weeks since the CRL does not need to be updated often on the Root CA. It also ensures that the Subordinate CA lifetime is extended from 1 year to 5 years, as the Subordinate certificate is issued from the Root CA.

{{< hint type=tip title="Active Directory Configuration Partition Distinguished Name" >}}
The Active Directory Configuration Partition Distinguished Name is required to configure settings for the Root CA since it is configured from the command line. To determine what the correct format of this name would be for your domain you can check it in only a few steps:

1. On one of your Domain Controllers, open the **Active Directory Users and Computers** console.
2. Go to the **View** menu and select **Advanced Features**.
3. Right-click on the root of your **Domain** and select **Properties**.
4. Go to the **Attribute Editor** tab.
5. Scroll down until you find the **distinguishedName** attribute field and click the **View** button.
6. Copy the value in the **Attribute Field**, this is the information needed for **step 2** below.
{{< /hint >}}

To configure the CRL settings for the Root CA, perform the following steps on the **TFS-ROOT-CA** server:

1. Open an **Administrative Command Prompt**.
2. To define the **Active Directory Configuration Partition Distinguished Name**, run the following command:

```cmd
certutil.exe -setreg CA\DSConfigDN "CN=Configuration,DC=corp,DC=tfslabs,DC=com"
```

3. To define **Validity Period Units** for all issued certificates by this CA, run following commands:

```cmd
certutil.exe -setreg CA\ValidityPeriodUnits 5
certutil.exe -setreg CA\ValidityPeriod "Years"
```

4. To define **CRL Period Units** and **CRL Period**, run the following commands:

```cmd
certutil.exe -setreg CA\CRLPeriodUnits 52
certutil.exe -setreg CA\CRLPeriod "Weeks"
```

5. To define **CRL Overlap Period Units** and **CRL Overlap Period**, run the following commands:

```cmd
certutil.exe -setreg CA\CRLOverlapPeriodUnits 12
certutil.exe -setreg CA\CRLOverlapPeriod "Hours"
```

6. Restart the **Active Directory Certificate Services** service:

```cmd
net stop CertSvc
net start CertSvc
```

7. Close the **Administrative Command Prompt**.

{{< hint type=important title="CRL Renewal Period" >}}
As defined in this section, the CRL period on the Root CA is set to 52 weeks. This means that every 52 weeks you will need to power on the **TFS-ROOT-CA** server and renew the CRL. You should set a reminder in your calendar to do perform this task every 50 weeks to ensure that it is renewed in time. You will require the use of a virtual floppy disk to copy the updated files to the Subordinate CA.
{{< /hint >}}

## 2.7 Enable Auditing on the Root Certificate Authority ##

Auditing is recommended on any server running **Active Directory Certificate Services**. This will write logs to the Windows Event Log whenever a certificate is issued or revoked.

To enable auditing for the Root CA server, perform the following steps on the **TFS-ROOT-CA** server:

1. Open the **Local Security Policy** console and modify the **Security Settings > Local Policies > Audit Policy > Audit object access** setting to audit **Success** and **Failure**.
2. Enable auditing for the Certificate Authority by running the following command from an **Administrative Command Prompt**:

```cmd
certutil.exe -setreg CA\AuditFilter 127
```

3. Restart the **Active Directory Certificate Services** service:

```cmd
net stop CertSvc
net start CertSvc
```

4. Close the **Administrative Command Prompt**.

## 2.8 Root Certificate Authority CDP and AIA Configuration ##

Before the **Subordinate Certificate Authority** can be properly configured, the **Certificate Revocation List** needs to be configured for the Root CA certificate. This configuration will be present in the Subordinate certificate that will be issued on the **Enterprise CA** which will be installed on the **TFS-CA01** server.

To configure the CDP and AIA settings, perform the following steps on the **TFS-ROOT-CA** server:

1. Open the **Certification Authority** console for the **TFS-ROOT-CA** server.
2. Right-click on **TFS Labs Certificate Authority** server and select **Properties**.
3. On the **Extensions** tab, verify that the **CRL Distribution Point (CDP)** extension is selected and click the **Add** button.
4. Under the **Location** field, enter the following address and click the **OK** button:

```
http://pki.corp.tfslabs.com/CertData/<CaName><CRLNameSuffix><DeltaCRLAllowed>.crl
```

5. Back on the **Extensions** tab, verify that the **Include in CRLs. Clients use this to find Delta CRL locations.** and **Include in the CDP extension of issued certificates** options are selected for the location that was just entered.
6. Select the **file://<ServerDNSName>/CertEnroll/TFS%20Labs%20Certificate%20Authority.crl** list item and click the **Remove** button. Click the **Yes** button to confirm that you want to remove the location.
7. On the **Extensions** tab, verify that the **Authority Information Access (AIA)** extension is selected and click the **Add** button.
8. Under the **Location** field, enter the following address and click the **OK** button:

```
http://pki.corp.tfslabs.com/CertData/<ServerDNSName>_<CaName><CertificateName>.crt
```

9. Back on the **Extensions** tab, verify that the **Include in the AIA extension of issued certificates** option is selected for the location that was just entered.
10. Select the **file://<ServerDNSName>/CertEnroll/<ServerDNSName>_<CaName><CertificateName>.crt** list item and click the **Remove** button. Click the **Yes** button to confirm that you want to remove the location.
11. Click the **OK** button to commit the changes. When prompted to restart **Active Directory Certificate Services**, click the **Yes** button.
12. Verify that the settings are correct by running the following commands in an **Administrative Command Prompt**:

```cmd
certutil.exe -getreg CA\CRLPublicationURLs
certutil.exe -getreg CA\CACertPublicationURLs
```

13. Close the **Administrative Command Prompt**.
14. In the **Certification Authority** console, right-click on **Revoked Certificates** under **TFS Labs Certificate Authority** and select **All Tasks > Publish**.
15. On the **Publish CRL** window, verify that **New CRL** is selected and click the **OK** button.
16. Close the **Certification Authority** console.

## 2.9 Root Certificate and CRL List Export ##

Exporting the **Root Certificate CRL List** is needed to make it available on the **TFS-CA01** server. The links to these files were referenced in the CA configuration, so they will need to be copied to the **Subordinate CA** server for users to access these files.

To copy the necessary files, perform the following steps on the **TFS-ROOT-CA** server:

1. Add the **RootCAFiles** virtual floppy disk to the **TFS-ROOT-CA** virtual machine.
2. Copy the contents of the **C:\Windows\System32\CertSrv\CertEnroll** folder to the **C:\RootCA** folder.
3. Open the **Certificates** console under the **Local Computer Account** and export the **TFS Labs Certificate Authority** certificate from the **Trusted Root Certification Authorities** Store as a **DER encoded binary**. Save the file as **C:\RootCA\TFS Labs Certificate Authority.cer**.
4. Copy the contents of the **C:\RootCA** folder to the **A:\ Drive**. The contents of the **A:\ Drive** should be the following:

```
A:\TFS Labs Certificate Authority.cer
A:\TFS Labs Certificate Authority.crl
A:\TFS-ROOT-CA_TFS Labs Certificate Authority.crt
```

5. Eject the **RootCAFiles** virtual floppy disk.

## 2.10 Renewing the Root CA CRL ##

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

## AD CS on Windows Server 2022 Guide ##

* [Introduction - AD CS on Windows Server 2022](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/)
* [Part 1 - Domain Controller and Workstation Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-1/)
* **Part 2 - Offline Root CA Setup**
* [Part 3 - Subordinate CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-3/)
* [Part 4 - Deploy Certificates](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-4/)
* [Part 5 - Online Responder Role Configuration](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-5/)
* [Part 6 - Private Key Archive and Recovery](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-6/)
* [Part 7 - Certificate Template Deployment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-7/)
* [Part 8 - Certificate Auto-Enrollment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-8/)
