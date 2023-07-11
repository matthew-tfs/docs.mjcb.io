---
title: "Deploy Certificates"
tags: [
    "ADCS",
    "Microsoft",
    "Windows"
]
geekdocNav: true
geekdocAnchor: false
weight: 40
---

The easiest and most efficient method to deploy the Root and Subordinate certificates to your organization is to use Group Policy to deploy them automatically to your devices. There are also methods of deploying those same certificates to devices manually, should automated processes be unavailable.

{{< toc >}}

## 4.1 Export the Root and Subordinate Certificates ##

The Root and Subordinate certificates can be easily exported into a format that can be used for ease of deployment. Once the certificate files are exported, they can be deployed using Group Policy. To export the certificate files, perform the following steps on the **TFS-CA01** server:

1. On the root of the **C:\ Drive** on the **TFS-CA01** server, create a folder called **Certificates** (C:\Certificates).
2. Open the **Certificates** console under the **Local Computer Account** on the **TFS-CA01** server.
3. Open the **Trusted Root Certification Authorities > Certificates** Store and export the **TFS Labs Certificate Authority** certificate as a **DER encoded binary** to the **C:\Certificates** folder. Save it as **C:\Certificates\TFS Labs Certificate Authority.cer** when exporting the certificate.
4. Open the **Intermediate Certification Authorities > Certificates** Store and export the **TFS Labs Enterprise CA** certificate as a **DER encoded binary** to the **C:\Certificates** folder. Save it as **C:\Certificates\TFS Labs TFS Labs Enterprise CA.cer** when exporting the certificate.

Once the certificate files have been exported, perform the following steps on the **TFS-DC01** server:

1. On the root of the **C:\ Drive** on the **TFS-DC01** server, create a folder called **Certificates** (C:\Certificates).
2. Copy the **C:\Certificates\TFS Labs Certificate Authority.cer** and **C:\Certificates\TFS Labs TFS Labs Enterprise CA.cer** files from the **TFS-CA01** server to the **C:\Certificates** folder on the **TFS-DC01** server.

## 4.2 Deploy the Root and Subordinate Certificates to the Domain ##

For initial deployment of the Root and Subordinate certificates to the **TFS Labs** domain, it will be applied to the root of **TFS Labs** OU that was created earlier. This is something that can be refined later depending on your requirements. If an OU structure wasn't created earlier, it can be applied to the root of the Active Directory domain.

To create the Group Policy Object for deploying the Root and Subordinate certificates, perform the following steps on the **TFS-DC01** server:

1. Open the **Group Policy Management** console on the **TFS-DC01** server.
2. In the root of the **corp.tfslabs.com** domain, open the **TFS Labs** OU, and create a new **GPO** called **TFS Labs Certificates**.
3. Right-click on the **TFS Labs Certificates** GPO and select the **Edit** option.
4. In the **Group Policy Management Editor** window, open the **Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies** node.
5. Right-click on the **Trusted Root Certification Authorities** folder and select the **Import** option. Select the **C:\Certificates\TFS Labs Certificate Authority.cer** file and import it.
6. Right-click on the **Intermediate Certification Authorities** folder and select the **Import** option. Select the **C:\Certificates\TFS Labs Enterprise CA.cer** file and import it.
7. Close the **Group Policy Management** console.
8. Once the certificates have been added to Group Policy, allow up to **1 hour** for the certificates to be deployed to the entire **Active Directory** Forest.

## 4.3 Enable IIS for Certificate File Deployment ##

**This section is optional. If you are not planning to deploy certificates without using Group Policy, then you can skip this section.**

Since there is already a folder present on the **TFS-CA01** server that contains the Root and Subordinate certificates used for Group Policy deployment, it is quite easy to make that folder available through IIS for other clients who require those files as well. This is useful for clients that are not managed through Active Directory.

On the **TFS-CA01** server, perform the following steps to allow for directory browsing for the Root and Subordinate certificate files:

1. Open the **Internet Information Services (IIS) Manager** console.
2. On the **Connections** pane, expand **TFS-CA01** and then expand **Sites**.
3. Right-click on **Default Web Site** and select **Add Virtual Directory**.
4. On **Add Virtual Directory** page, in **Alias**, enter **Certificates**. For the **Physical path**, enter **C:\Certificates** and then click **OK**.
5. In the **Connections** pane, under the **Default Web Site**, ensure the **Certificates** virtual directory is selected.
6. In the **Certificates Home** pane, double-click on **Directory Browsing**.
7. In the **Actions** pane, click the **Enable** button.
8. Close the **Internet Information Services (IIS) Manager** console.

## 4.4 Deploy Root and Subordinate Certificates to iOS ##

**This section is optional. If you are not planning to deploy certificates to iOS devices, then you can skip this section.**

If there is a need to deploy the Root and Subordinate certificates to iOS devices, there are only a few steps required to install those certificates.

On the **TFS-CA01** server, duplicate the **C:\Certificates\TFS Labs Certificate Authority.cer** and **C:\Certificates\TFS Labs TFS Labs Enterprise CA.cer** files in the **C:\Certificates** folder. You will need to change the extension on these files from **.cer** to **.crt** for iOS to be able to recognize and download them.

Once the certificates have been renamed, perform the following steps on an iOS device to install the Root and Subordinate certificates:

1. Open **Safari** on the iOS device and go to **http​://tfs-ca01.corp.tfslabs.com/Certificates** to access the Root and Subordinate certificates.
2. Click on the **TFS Labs Certificate Authority.crt** file. You should get a popup stating **This website is trying to download a configuration profile. Do you want to allow this?** and click the **Allow** button to confirm it. You will then get a prompt that the **SSL Profile** was downloaded. Click the **Close** button to continue.
3. Open the **Settings** on your iOS device and select the **Profile Downloaded** menu option. A prompt should appear with the **Install Profile** options for the **TFS Labs Certificate Authority**. Click the **Install** button. You may be prompted to enter your credentials. On the **Warning** prompt, click the **Install** button to continue. Click **Done** to finish the installation.
4. Go to **Settings > General > About > Certificate Trust Settings**. Under **Enable Full Trust For Root Certificates**, enable the **TFS Labs Certificate Authority** certificate. A prompt for enabling a **Root Certificate** will appear, click **Continue** to complete the trust.
5. Open **Safari** on the iOS Device and go to **http​://tfs-ca01.corp.tfslabs.com/Certificates** to access the **Root and Subordinate Certificates**.
6. Click on the **TFS Labs Enterprise CA.crt** file. You should get a popup stating **This website is trying to download a configuration profile. Do you want to allow this?** and click the **Allow** button to confirm it. You will then get a prompt that the **SSL Profile** was downloaded. Click the **Close** button to continue.
7. Open the **Settings** on your iOS device and select the **Profile Downloaded** menu option. A prompt should appear with the **Install Profile** options for the **TFS Labs Enterprise CA**. Click the **Install** button. You may be prompted to enter your credentials. On the **Warning** prompt, click the **Install** button to continue. Click **Done** to finish the installation.

## AD CS on Windows Server 2022 Guide ##

* [Introduction - AD CS on Windows Server 2022](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/)
* [Part 1 - Domain Controller and Workstation Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-1/)
* [Part 2 - Offline Root CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-2/)
* [Part 3 - Subordinate CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-3/)
* **Part 4 - Deploy Certificates**
* [Part 5 - Online Responder Role Configuration](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-5/)
* [Part 6 - Private Key Archive and Recovery](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-6/)
* [Part 7 - Certificate Template Deployment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-7/)
* [Part 8 - Certificate Auto-Enrollment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-8/)
