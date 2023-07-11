---
title: "Certificate Auto-Enrollment"
tags: [
    "ADCS",
    "Microsoft",
    "Windows"
]
geekdocNav: true
geekdocAnchor: false
weight: 70
---

Enabling the auto-enrollment feature in Group Policy will allow users and workstations within the organization the ability to automatically receive a certificate from the **Active Directory Certificate Authority** server. This level of automation is helpful for large organizations that need to quickly deploy certificates for users or workstations.

{{< toc >}}

## 7.1 User Auto-Enrollment ##

To enable certificate auto-enrollment for user accounts in the **TFS Labs** domain, perform the following steps on the **TFS-DC01** server:

1. On the **TFS-DC01** server, open the **Group Policy Management** console (gpmc.msc).
2. Open the **TFS Labs Certificates** GPO that was created earlier.
3. Open the **User Configuration > Policies > Windows Settings > Security Settings > Public Key Policies** node.
4. Open the **Certificate Services Client - Certificate Enrollment Policy** object.
5. In the **Properties** window, change the **Configuration Model** option to **Enabled**. Click the **OK** button to close the window.
6. Open the **Certificate Services Client - Auto-Enrollment** object.
7. In the **Properties** window, change the **Configuration Model** option to **Enabled**. Select the options for **Renew expired certificates, update pending certificates, and remove revoked certificates** and **Update certificate that use certificate templates** options. Click the **OK** button to close the window.

{{< hint type=note title="Group Policy Propagation" >}}
Once the auto-enrollment options have been added to Group Policy, allow up to 1 hour for the update to be processed in the entire Active Directory Forest.
{{< /hint >}}

## 7.2 Workstation Auto-Enrollment ##

To enable certificate auto-enrollment for workstation accounts in the **TFS Labs** domain, perform the following steps on the **TFS-DC01** server:

1. On the **TFS-DC01** server, open the **Group Policy Management** console (gpmc.msc).
2. Open the **TFS Labs Certificates** GPO that was created earlier.
3. Open the **Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies** node.
4. Open the **Certificate Services Client - Certificate Enrollment Policy** object.
5. In the **Properties** window, change the **Configuration Model** option to **Enabled**. Click the **OK** button to close the window.
6. Open the **Certificate Services Client - Auto-Enrollment** object.
7. In the **Properties** window, change the **Configuration Model** option to **Enabled**. Select the options for **Renew expired certificates, update pending certificates, and remove revoked certificates** and **Update certificate that use certificate templates** options. Click the **OK** button to close the window.

{{< hint type=note title="Group Policy Propagation" >}}
Once the auto-enrollment options have been added to Group Policy, allow up to 1 hour for the update to be processed in the entire Active Directory Forest.
{{< /hint >}}

## AD CS on Windows Server 2019 Guide ##

* [Introduction - AD CS on Windows Server 2019 Guide](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/)
* [Part 1 - Offline Root CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-1/)
* [Part 2 - Subordinate CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-2/)
* [Part 3 - Deploy Root and Subordinate Certificate](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-3/)
* [Part 4 - Certificate Revocation Policies](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-4/)
* [Part 5 - Configure Private Key Archive and Recovery](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-5/)
* [Part 6 - Certificate Template Deployment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-6/)
* **Part 7 - Certificate Auto-Enrollment**
* [Part 8 - AD CS Final Steps](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2019/adcs-windows-server-2019-part-8/)
