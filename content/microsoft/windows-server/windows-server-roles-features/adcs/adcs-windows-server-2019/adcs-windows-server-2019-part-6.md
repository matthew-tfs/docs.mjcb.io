---
title: "Certificate Template Deployment"
tags: [
    "ADCS",
    "Microsoft",
    "Windows"
]
geekdocNav: true
geekdocAnchor: false
weight: 60
---

The following **Certificate Templates** will need to be created in the **Certification Authority** console on the **TFS-CA01** server:

| **Template Name**                | **Validity** | **Publish in ADDS** | **Additional Security**                                                             |
|:---------------------------------|:-------------|:--------------------|:------------------------------------------------------------------------------------|
| TFS Labs User Certificate	       | 1 Year       | Yes                 | <ul><li>TFS-CA01 (Enroll)</li><li>Domain Users (Read, Enroll, Autoenroll)</li></ul> |
| TFS Labs Workstation Certificate | 1 Year       | Yes                 | <ul><li>TFS-CA01 (Enroll)</li><li>Domain Computers (Enroll, Autoenroll)</li></ul>   |
| TFS Labs Web Server Certificate  | 1 Year       | No	                | <ul><li>TFS-CA01 (Enroll)</li></ul>                                                 |

These **Certificate Templates** will be used for issues **Certificates** to the organization. Some will be issued automatically, and the others can be requested by users or devices. The procedure for creating these **Certificate Templates** is mostly the same.

The settings for these templates will vary based on the needs of your organization, and the settings being used in this guide may or may not work for you. Ensure that you understand what the template settings are and how they will affect the users and devices in your organization.

{{< toc >}}

## 6.1 User Certificate Template Creation ##

1. In the **Certification Authority** console on the **TFS-CA01** server, ensure that the **TFS Labs Enterprise CA** server is expanded in the console tree.
2. Right-click on **Certificate Templates** and then click **Manage**. The **Certificate Templates Console** window will open and display the **Certificate Templates** that are currently stored in **Active Directory**.
3. In the details pane, right-click on the **User** Certificate Template and then click **Duplicate Template**.
4. On the **Properties of New Template** window, click on the **General** tab. Change the name of the template to **TFS Labs User Certificate**. Ensure that the **Validity Period** is set to **1 Year**.
5. Ensure that the **Publish certificate in Active Directory option** checkbox is selected, as well as **Do not automatically reenroll if a duplicate certificate exists in Active Directory is selected** option is selected as well.
6. On the **Security** tab, click the **Add** button and add the **TFS-CA01** server. Give it the **Enroll** permission.
7. Select the **Domain Users** group and add the **Read**, **Enroll** and **Autoenroll** permissions.
8. Click the **OK** button to close the template window.
9. Close the **Certificate Templates** console window.
10. In the **Certification Authority** console, right-click on **Certificate Templates**, then select **New** and then select **Certificate Template to Issue**.
11. In the **Enable Certificate Templates** dialog box, click **TFS Labs User Certificate** and then click **OK**.

{{< hint type=important title="User Certificate Template" >}}
Ensure that the user has their e-mail address entered into Active Directory, otherwise this certificate will not deploy correctly to the user account.
{{< /hint >}}

## 6.2 Workstation Certificate Template Creation ##

1. In the **Certification Authority** console on the **TFS-CA01** server, ensure that the **TFS Labs Enterprise CA** server is expanded in the console tree.
2. Right-click on **Certificate Templates** and then click **Manage**. The **Certificate Templates Console** window will open and display the **Certificate Templates** that are currently stored in **Active Directory**.
3. In the details pane, right-click on the **Computer** Certificate Template and then click **Duplicate Template**.
4. On the **Properties of New Template** window, click on the **General** tab. Change the name of the template to **TFS Labs Workstation Certificate**. Ensure that the **Validity Period** is set to **1 Year**.
5. Ensure that the **Publish certificate in Active Directory option** checkbox is selected.
6. On the **Security** tab, click the **Add** button and add the **TFS-CA01** server. Give it the **Enroll** permission.
7. Select the **Domain Computer** group and add the **Enroll** and **Autoenroll** permissions.
8. Click the **OK** button to close the template window.
9. Close the **Certificate Templates** console window.
10. In the **Certification Authority** console, right-click on **Certificate Templates**, then select **New** and then select **Certificate Template to Issue**.
11. In the **Enable Certificate Templates** dialog box, click **TFS Labs Workstation Certificate** and then click **OK**.

## 6.3 Web Server Certificate Template Creation ##

1. In the **Certification Authority** console on the **TFS-CA01** server, ensure that the **TFS Labs Enterprise CA** server is expanded in the console tree.
2. Right-click on **Certificate Templates** and then click **Manage**. The **Certificate Templates Console** window will open and display the **Certificate Templates** that are currently stored in **Active Directory**.
3. In the details pane, right-click on the **Web Server** Certificate Template and then click **Duplicate Template**.
4. On the **Properties of New Template** window, click on the **General** tab. Change the name of the template to **TFS Labs Web Server Certificate**. Ensure that the **Validity Period** is set to **1 Year**.
5. On the **Security** tab, click the **Add** button and add the **TFS-CA01** server. Give it the **Enroll** permission.
6. Select the **Domain Admins** group and add the **Enroll** and **Autoenroll** permissions.
7. Click the **OK** button to close the template window.
8. Close the **Certificate Templates** console window.
9. In the **Certification Authority** console, right-click on **Certificate Templates**, then select **New** and then select **Certificate Template to Issue**.
10. In the **Enable Certificate Templates** dialog box, click **TFS Labs Web Server Certificate** and then click **OK**.

## 6.4 Active Directory Certificate Services Web Enrollment

The **Active Directory Certificate Services Web Enrollment** website is a feature that allows authenticated users in an organization the ability to submit certificate requests and download the completed certificates. It can be found by going to the following URL:

**http​://tfs-ca01.corp.tfslabs.com/CertSrv/**

The only issue with this website is that SSL should be enabled to allow for the secure transmission of certificate requests and files. Fortunately, the Certificate Authority is now able to issue certificates and a proper certificate can now be requested and applied to this website.

To enable SSL on the **Active Directory Certificate Services Web Enrollment** website, perform the following steps on the **TFS-CA01** server:

1. Open the **Certificates** console under the **Local Computer Account** on the **TFS-CA01** server.
2. Go to the **Certificates > Personal > Certificates** Store.
3. Right-click on the **Certificates** folder, go to **All Tasks** and select **Request New Certificate...**.
4. On the **Before You Begin** screen, click the **Next** button to continue.
5. On the **Select Certificate Enrollment Policy** screen, click the **Next** button to continue.
6. On the **Request Certificates** screen, select the **TFS Labs Workstation Certificate** and click the **Enroll** button.
7. On the **Certificate Installation Results** screen, click the **Finish** button to close the wizard.
8. Close the **Certificates** console.

Once the SSL certificate has been created for the **TFS-CA01** server, it needs to be configured on the IIS server. Perform the following steps to configure IIS to use the SSL certificate:

1. Open the **Internet Information Services (IIS) Manager** console on the **TFS-CA01** server.
2. In the **Connections** pane, select the **TFS-CA01** server and expand **Sites**.
3. Select the **Default Web Site** and in the **Actions** pane, select **Bindings...**.
4. In the **Site Bindings** window, click the **Add...** button.
5. In the **Add Site Binding** window, use the following settings and then click the **OK** button:
   * **Type:** https
   * **IP Address:** All Unassigned
   * **Port:** 443
   * **SSL Certificate:** TFS-CA01.corp.tfslabs.com
6. Expand **Default Web Site** and select the **CertSrv** folder.
7. In the **/CertSrv Home** pane, double-click on the **SSL Settings** icon.
8. In the **SSL Settings** window, select the option for **Require SSL** and then click the **Apply** button in the **Actions** pane.

You can easily verify that SSL is working correctly on the **Active Directory Certificate Services Web Enrollment** page by opening the **https​://tfs-ca01.corp.tfslabs.com/CertSrv** page (you can login with a Domain Administrator account). It should be secured with SSL using the correct certificate. If you go to the SSL certificate properties of the web page, you will be able to see that the certificate has been issued by the **TFS Labs Enterprise CA** and is valid for only 1 year.

## AD CS on Windows Server 2019 Guide ##

* [Introduction - AD CS on Windows Server 2019 Guide](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/)
* [Part 1 - Offline Root CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-1/)
* [Part 2 - Subordinate CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-2/)
* [Part 3 - Deploy Root and Subordinate Certificate](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-3/)
* [Part 4 - Certificate Revocation Policies](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-4/)
* [Part 5 - Configure Private Key Archive and Recovery](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-5/)
* **Part 6 - Certificate Template Deployment**
* [Part 7 - Certificate Auto-Enrollment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-7/)
* [Part 8 - AD CS Final Steps](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-8/)
