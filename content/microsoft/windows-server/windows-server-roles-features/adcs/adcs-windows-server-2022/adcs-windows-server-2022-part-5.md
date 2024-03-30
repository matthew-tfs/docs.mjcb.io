---
title: "Online Responder Role Configuration"
tags: [
    "ADCS",
    "Microsoft",
    "Windows"
]
geekdocNav: true
geekdocAnchor: false
weight: 50
---

The **Online Responder** role is a component of **Active Directory Certificate Services** that is used to reduce the overhead with CRLs on a network. It can check for revoked certificates much faster than with regular CRLs and can update clients of their status. This type of functionality is entirely dependent on the size of the network that you have and how often you need to revoke certificates.

{{< hint type=note title="Online Responder Role Configuration" >}}
This entire section is optional. Not implementing the Online Responder role will have no adverse affect on the functionality of your Certificate Authority, nor will it interfere with any later steps. This role can be added at any time in the future if needed.
{{< /hint >}}

{{< toc >}}

## 5.1 Create CNAME Records in DNS ##

1. Open the **DNS Manager** console.
2. Under the **DNS** Node, expand the **TFS-DC01** server and then expand **Forward Lookup Zones**. Select and the **corp.tfslabs.com** Zone. Right-click **New Alias (CNAME)**.
3. In **Alias name (uses parent domain if left blank)**, enter **OCSP** as the name. In the **Fully qualified domain name (FQDN)** field, enter **tfs-ca01.corp.tfslabs.com.** and then click **OK**.
4. Close the **DNS Manager** console.

To validate that the **CNAME** record was created correctly, you should be able to ping the address. The ping request should fail because by default the Windows Firewall will deny the request, but the name should still resolve.

## 5.2 Install the Online Responder Role ##

1. On the **TFS-CA01** server, open the **Server Manager** console, click on the **Manage** menu, and click on **Add Roles and Features** to start the installation wizard.
2. On the **Before you begin** screen, click the **Next** button to continue.
3. On the **Select installation type** screen, select the option for **Role-based or feature-based installation** and click the **Next** button to continue.
4. On the **Select destination server** screen, verify that the **TFS-CA01.corp.tfslabs.com** server is selected and click **Next** to continue.
5. On the **Select server roles** screen, expand the **Active Directory Certificate Services** option, and select the **Online Responder** role.
6. The installation wizard will ask to install the necessary management tools for the role. Click the **Add Features** button to continue.
7. On the **Select server roles** screen, click the **Next** button to continue.
8. On the **Select features** screen, click the **Next** button to continue.
9. On the **Confirm installation selections** screen, select the option to **Restart the destination server automatically if required**. When prompted with a warning about restarting the server, click the **Yes** button. Click the **Install** button to continue.
10. Once the installation is completed, click the **Close** button.
11. Restart the **TFS-CA01** server if it did not restart automatically as part of the installation.

## 5.3 Configure the Online Responder Role ##

Once the **Online Responder** role has been added to the **TFS-CA01** server, it can now be configured.

1. To begin the configuration of the **Online Responder** service, open the **Server Manager** console. Click the **Notifications** icon in the upper-right hand corner and click the **Configure Active Directory Certificate Services on the destination server** link in the **Post-deployment Configuration** box.
2. On the **Credentials** screen, verify that the credentials are set to a Domain Administrator account and click **Next** to continue.
3. On the **Role Services** screen, select the option for **Online Responder** and click **Next** to continue.
4. On the **Confirmation** screen, click on the **Configure** button to continue.
5. Once the configuration is completed, click the **Close** button.

Once the OCSP role has been installed, verify that the OCSP folder is present in the **IIS Manager** console. To do this, open the **Internet Information Services (IIS) Manager** console and expand the **TFS-CA01 > Sites > Default Web Site** folder. The **ocsp** folder should be listed as a folder within IIS.

If the **ocsp** folder is not present, run the following command in an **Administrative Command Prompt**:

```cmd
certutil.exe -vocsproot
```

The IIS service will need to be restarted afterwards if this command is run, otherwise the folder will not be available.

## 5.4 Add the OCSP URL to the Enterprise CA ##

Once the **Online Responder** role has been installed, the URL can now be added to the **Subordinate CA** certificate.

1. In the **Certification Authority** console on the **TFS-CA01** server, in the console tree right-click on **TFS Labs Enterprise CA**, and then select **Properties**.
2. On the **Extensions** tab, under **Select extension**, select **Authority Information Access (AIA)**, and then click **Add**.
3. In the **Location** field, enter in **http​://ocsp.corp.tfslabs.com/ocsp** and then click **OK**.
4. Select **Include in the online certificate status protocol (OCSP) extension** and click the **OK** button (ensure that the **Include in the AIA extension of issued certificates** is not selected).
5. When prompted by the **Certification Authority** dialog box to restart **Active Directory Certificate Services**, click **Yes**.

## 5.5 Configure and Publish the OCSP Response Signing Certificate ##

The **OCSP Response Signing Certificate** will be used by the **OCSP Responder Service** to manage the OCSP services for the organization.

1. Open the **Certification Authority** console on the **TFS-CA01** server, ensure that the **TFS Labs Enterprise CA** server is expanded in the console tree.
2. Right-click on **Certificate Templates** and then click **Manage**. The **Certificate Templates Console** window will open and displays the certificate templates stored in **Active Directory**.
3. In the details pane, right-click on the **OCSP Response Signing** template and then click **Duplicate Template**.
4. On the **Properties of New Template** window, click on the **General** tab. Change the name of the template from **Copy of OCSP Response Signing** to **TFS Labs OCSP Response Signing**.
5. On the **Security** tab select **Authenticated Users** and enable the **Enroll** permission. Click **OK**.
6. On the **Security** tab add the **TFS-CA01** server and enable the **Read** and **Enroll** permission. Click **OK**.
7. Close the **Certificate Templates Console**.
8. In the **Certification Authority** console, right-click **Certificate Templates**, then select **New > Certificate Template to Issue**.
9. In the **Enable Certificate Templates** dialog box, click **TFS Labs OCSP Response Signing** and then click **OK**.

## 5.6 Configure Revocation Configuration on the Online Responder ##

Once the **OCSP Certificates** have been configured, the **OCSP Responder Role** can now be configured.

1. Open the **Online Responder Management** console on the **TFS-CA01** Server.
2. Right-click on **Revocation Configuration** and then click **Add Revocation Configuration**.
3. On the **Getting started with adding a revocation configuration** screen, click **Next** to continue.
4. On the **Name the Revocation Configuration** screen, enter **TFS Labs Enterprise CA** in the **Name** field, and then click **Next** to continue.
5. On the **Select CA Certificate Location** screen, ensure that **Select a certificate for an Existing enterprise CA** is selected, then click **Next** to continue.
6. On the **Choose CA Certificates** screen, ensure that **Browse CA certificates published in Active Directory** is selected, and then click **Browse**.
7. On the **Select Certification Authority** dialog box, ensure that **TFS Labs Enterprise CA** is selected, and then click **OK**. Click the **Next** button to continue.
8. On the **Select Signing Certificate** screen, use the default options and then click **Next** to continue.
9. On the **Revocation Provider** screen, click the **Provider** button.
10. For the **LDAP and HTTP locations** in the **Base CRLs** window, clear the **Refresh CRLs based on their validity periods** option. In **the Update CRLs at this refresh interval (min)** field, enter **15**, and then click **OK**.
11. Click **Finish** to close the window.
12. In the **Online Responder Management** console, expand **Array Configuration** and then click **TFS-CA01.corp.tfslabs.com**.
13. Review the **Revocation Configuration Status** in the middle pane to ensure there is a signing certificate present and the status reports as **OK**. If so, the provider is successfully using the current configuration.

## 5.7 Add the OCSP URL to Group Policy ##

Once the **Online Responder** role has been installed and configured, the URL can now be added to the **Subordinate CA** certificate in Group Policy to ensure that it is being deployed to the organization.

1. Open the **Group Policy Management** console on the **TFS-DC01** Server.
2. In the **corp.tfslabs.com** domain, right-click on the **TFS Labs Certificates** GPO and select the **Edit** option.
3. In the **Group Policy Management Editor** window, open the **Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies** node.
4. Open the **Intermediate Certification Authorities** folder, right-click on the **TFS Labs Enterprise CA** certificate and select **Properties**.
5. In the **TFS Labs Enterprise CA** properties window, click on the **OCSP** tab and in the text field to the left of the **Add URL** button, enter **http​://ocsp.corp.tfslabs.com/ocsp** and click the **Add URL** button. Click the **OK** button to close the window.
6. Close the **Group Policy Management** console.
7. Once the certificates have been added to Group Policy, allow up to 1 hour for the certificates to be deployed to the entire **Active Directory** Forest.

## 5.8 Verify OCSP Status ##

Once the **OCSP Role** has been configured, you can now verify if it is working correctly within the PKI Infrastructure.

1. On the **TFS-CA01** server, open the **Enterprise PKI** console.
2. Under the **Enterprise PKI** node, click on the **TFS Labs Certificate Authority** server and check that the status of **OCSP** is **OK**.

If there are no issues being reported with OCSP then you can proceed to the next step. If there is an issue with OCSP, then the most likely issue is with the **CA Exchange Certificate**. It most likely does not know about OCSP since it was generated before the OCSP role was configured, so it will need to be recreated.

1. Open the **Certification Authority** console on the **TFS-CA01** server, ensure that the **TFS Labs Enterprise CA** server is expanded in the console tree.
2. Under the **Issued Certificates** folder, look for a certificate using the **CA Exchange (CAExchange)** template.
3. Right-click on the certificate and select the **All Tasks > Revoke Certificate** option.
4. On the **Certificate Revocation** window, click the **Yes** button to revoke the certificate.
5. Open an **Administrative Command Prompt** and run the following command request a new certificate:

```cmd
certutil.exe -cainfo xchg
```

6. Close the **Administrative Command Prompt**.
7. On the **TFS-CA01** server, open the **Enterprise PKI** console.
8. Under the **Enterprise PKI** node, click on the **TFS Labs Certificate Authority** server and check that the status of **OCSP** is **OK**.

## 5.9 Verify OCSP Connectivity ##

To verify that the **Online Responder** server can communicate with devices on the **TFS Labs** domain a certificate will need to be exported and run through the **URL Retrieval Tool**. This can be easily accomplished on the **TFS-CA01** Server.

1. On the **TFS-CA01** server, export the **TFS Labs Enterprise CA** certificate as a **DER Encoded Binary** the root of the **C:\ Drive** (C:\TFS Labs Enterprise CA.cer).
2. Run the following command in an **Administrative Command Prompt** to launch the **URL Retrieval Tool**:

```cmd
certutil.exe -URL "C:\TFS Labs Enterprise CA.cer"
```

3. Close the **Administrative Command Prompt**.
4. In the **Retrieve** box, select the option for **OCSP (from AIA)** and then click the **Retrieve** button.
5. If there is no issue with the OCSP service, the result returned should be **Verified OCSP (0.0) http​://ocsp.corp.tfslabs.com/ocsp**.

{{< hint type=important title="OCSP Status Results" >}}
This step may not return a result immediately and that is nothing to be concerned about. If you run this test and nothing is returned in the results, you may want to run this test later using the **TFS-WIN11** workstation once certificate deployments has been completed.
{{< /hint >}}

## AD CS on Windows Server 2022 Guide ##

* [Introduction - AD CS on Windows Server 2022](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/)
* [Part 1 - Domain Controller and Workstation Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-1/)
* [Part 2 - Offline Root CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-2/)
* [Part 3 - Subordinate CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-3/)
* [Part 4 - Deploy Certificates](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-4/)
* **Part 5 - Online Responder Role Configuration**
* [Part 6 - Private Key Archive and Recovery](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-6/)
* [Part 7 - Certificate Template Deployment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-7/)
* [Part 8 - Certificate Auto-Enrollment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-8/)
