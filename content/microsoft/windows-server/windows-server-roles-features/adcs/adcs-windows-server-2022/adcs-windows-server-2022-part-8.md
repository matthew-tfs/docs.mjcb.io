---
title: "Certificate Auto-Enrollment"
tags: [
    "ADCS",
    "Microsoft",
    "Windows"
]
geekdocNav: true
geekdocAnchor: false
weight: 80
---

Enabling the auto-enrollment feature in Group Policy will allow users and workstations within the organization the ability to automatically receive a certificate from the **Active Directory Certificate Authority** server. This level of automation is helpful for large organizations that need to quickly deploy certificates for users or workstations.

{{< hint type=note >}}
**Certificate Auto-Enrollment**\
This entire section is optional. Not implementing certificate auto-enrollment will have no impact on the functionality of your Certificate Authority, nor will it interfere with any later steps. This functionality can be added at any time in the future if needed.
{{< /hint >}}

{{< toc >}}

## 8.1 User Auto-Enrollment ##

To enable certificate auto-enrollment for user accounts in the **TFS Labs** domain, perform the following steps on the **TFS-DC01** server:

1. On the **TFS-DC01** server, open the **Group Policy Management** console.
2. Open the **TFS Labs Certificates** GPO that was created earlier.
3. Open the **User Configuration > Policies > Windows Settings > Security Settings > Public Key Policies** node.
4. Open the **Certificate Services Client - Certificate Enrollment Policy** object.
5. In the **Properties** window, change the **Configuration Model** option to **Enabled**. Click the **OK** button to close the window.
6. Open the **Certificate Services Client - Auto-Enrollment** object.
7. In the **Properties** window, change the **Configuration Model** option to **Enabled**. Select the options for **Renew expired certificates, update pending certificates, and remove revoked certificates** and **Update certificate that use certificate templates** options. Click the **OK** button to close the window.

{{< hint type=note title="Group Policy Propagation" >}}
Once the auto-enrollment options have been added to Group Policy, allow up to 1 hour for the update to be processed in the entire Active Directory Forest.
{{< /hint >}}

## 8.2 Workstation Auto-Enrollment ##

To enable certificate auto-enrollment for workstation accounts in the **TFS Labs** domain, perform the following steps on the **TFS-DC01** server:

1. On the **TFS-DC01** server, open the **Group Policy Management** console.
2. Open the **TFS Labs Certificates** GPO that was created earlier.
3. Open the **Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies** node.
4. Open the **Certificate Services Client - Certificate Enrollment Policy** object.
5. In the **Properties** window, change the **Configuration Model** option to **Enabled**. Click the **OK** button to close the window.
6. Open the **Certificate Services Client - Auto-Enrollment** object.
7. In the **Properties** window, change the **Configuration Model** option to **Enabled**. Select the options for **Renew expired certificates, update pending certificates, and remove revoked certificates** and **Update certificate that use certificate templates** options. Click the **OK** button to close the window.

{{< hint type=note title="Group Policy Propagation" >}}
Once the auto-enrollment options have been added to Group Policy, allow up to 1 hour for the update to be processed in the entire Active Directory Forest.
{{< /hint >}}

## AD CS on Windows Server 2022 Guide ##

* [Introduction - AD CS on Windows Server 2022](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/)
* [Part 1 - Domain Controller and Workstation Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-1/)
* [Part 2 - Offline Root CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-2/)
* [Part 3 - Subordinate CA Setup](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-3/)
* [Part 4 - Deploy Certificates](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-4/)
* [Part 5 - Online Responder Role Configuration](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-5/)
* [Part 6 - Private Key Archive and Recovery](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-6/)
* [Part 7 - Certificate Template Deployment](/microsoft/windows-server/windows-server-roles-features/adcs/adcs-windows-server-2022/adcs-windows-server-2022-part-7/)
* **Part 8 - Certificate Auto-Enrollment**
