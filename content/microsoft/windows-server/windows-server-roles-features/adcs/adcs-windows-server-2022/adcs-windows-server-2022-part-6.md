---
title: "Private Key Archive and Recovery"
tags: [
    "ADCS",
    "Microsoft",
    "Windows"
]
geekdocNav: true
geekdocAnchor: false
weight: 60
---

The **Key Recovery Agent** feature of **Active Directory Certificate Services** allows for the archival of private keys that are generated by the Certificate Authority. This is very important if a certificate is deleted and needs to be restored.

{{< hint type=note title="Private Key Archive and Recovery" >}}
This entire section is optional. Not implementing private key archive and recovery will have no impact on the functionality of your Certificate Authority, nor will it interfere with any later steps. This functionality can be added at any time in the future if needed.
{{< /hint >}}

{{< toc >}}

## 6.1 Create the Key Recovery Agent Template ##

1. In the **Certification Authority** console on the **TFS-CA01** server, ensure that the **TFS Labs Enterprise CA** server is expanded in the console tree.
2. Right-click on **Certificate Templates** and then click **Manage**. The **Certificate Templates Console** will open and display the Certificate Templates stored in **Active Directory**.
3. In the details pane, right-click on the **Key Recovery Agent** Certificate Template and then click **Duplicate Template**.
4. On the **Properties of New Template** window, click on the **General** tab. Change the name of the template to **TFS Labs Key Recovery Agent**. Ensure that the **Validity Period** is set to **1 year**.
5. On the **Issuance Requirements** tab, uncheck the option for **CA certificate manager approval**.
6. On the **Security** tab verify that **Authenticated Users** do not have the **Enroll** or **Autoenroll** permissions enabled.
7. On the **Security** tab select **Domain Admins** and **Enterprise Admins** and enable the **Enroll** permission. Click **OK** to close the window.
8. Close the **Certificate Templates Console** window.
9. In **Certification Authority** console, right-click on **Certificate Templates**, then select **New** and then select **Certificate Template to Issue**.
10. In the **Enable Certificate Templates** dialog box, click **TFS Labs Key Recovery Agent** and then click **OK**.

## 6.2 Create the Key Recovery Agent Certificate ##

Once the **Certificate Template** has been created it can now be requested for the **Domain Administrator** account.

1. On the **TFS-CA01** Server, open the **Certificates** console under the Domain Administrator account.
2. Right-click on the **Personal** folder and select the **All Tasks > Request New Certificate...** option.
3. On the **Before You Begin** screen, click the **Next** button to continue.
4. On the **Select Certificate Enrollment Policy** screen, click the **Next** button to continue.
5. On the **Request Certificates** screen, check the box beside the **TFS Labs Key Recovery Agent** certificate and click the **Enroll** button.
6. On the **Certificate Installation Results** screen, click the **Finish** button.

## 6.3 Configure the Certificate Authority to Allow Key Recovery ##

The option to archive keys that are generated by the Subordinate CA will need to be explicitly activated for it work correctly.

To configure the Subordinate CA to allow for Key Recovery, perform the following steps on the **TFS-CA01** server:

1. In the **Certification Authority** console on the **TFS-CA01** server, ensure that the **TFS Labs Enterprise CA Server** is expanded in the console tree.
2. Right-click on the **TFS Labs Enterprise CA** server and select the **Properties** option.
3. On the **Recovery Agents** tab, select the option to **Archive the Key**. Click the **Add...** button and select the **Key Recovery Agent Certificate** that was just requested.
4. Click the **OK** button. When prompted to restart **Active Directory Certificate Services**, click the **Yes** button.

## 6.4 Configure the Certificate Template for Archiving Keys ##

Now that the **Key Archive** feature has been enabled, the certificate can now be published to **Active Directory** and the **Certificate Authority**.

To configured the Certificate Template for Key Archival, perform the following steps on the **TFS-CA01** server:

1. In the **Certification Authority** console on the **TFS-CA01** server, ensure that the **TFS Labs Enterprise CA** server is expanded in the console tree.
2. Right-click on **Certificate Templates** and then click **Manage**. The **Certificate Templates Console** window will open and display the **Certificate Templates** that are currently stored in **Active Directory**.
3. In the details pane, right-click on the **User** Certificate Template and then click **Duplicate Template**.
4. On the **Properties of New Template** window, click on the **General** tab. Change the name of the template to **TFS Labs Key Archive**. Ensure that the **Validity Period** is set to **1 Year**.
5. On the **Subject Name** tab, uncheck the options for **Include e-mail name in the subject name** and the **E-mail Name** from the **Active Directory** settings.
6. On the **Request Handling** tab, select the option for **Archive subject's encryption private key**. When the **Changing Key Archival Property** box opens, click the **OK** button to continue.
7. Click the **OK** button to close the template window.
8. Close the **Certificate Templates Console** window.
9. In the **Certification Authority** console, right-click on **Certificate Templates**, then select **New** and then select **Certificate Template to Issue**.
10. In the **Enable Certificate Templates** dialog box, click the **TFS Labs Key Archive** option and then click **OK**.

## AD CS on Windows Server 2022 Guide ##

* [Introduction - AD CS on Windows Server 2022](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/)
* [Part 1 - Domain Controller and Workstation Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-1/)
* [Part 2 - Offline Root CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-2/)
* [Part 3 - Subordinate CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-3/)
* [Part 4 - Deploy Certificates](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-4/)
* [Part 5 - Online Responder Role Configuration](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-5/)
* **Part 6 - Private Key Archive and Recovery**
* [Part 7 - Certificate Template Deployment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-7/)
* [Part 8 - Certificate Auto-Enrollment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-8/)
