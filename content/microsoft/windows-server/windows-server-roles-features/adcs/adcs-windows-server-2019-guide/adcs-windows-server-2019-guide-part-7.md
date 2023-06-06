---
title: "7 - Certificate Auto-Enrollment"
tags: [
    "ADCS",
    "Microsoft",
    "Windows"
]
geekdocNav: true
geekdocAnchor: false
weight: 70
---

{{< toc >}}

## 7.1 User Auto-Enrollment ##

Enabling the User Auto-Enrollment feature will allow Users within the organization the ability to automatically receive a Certificate from the Active Directory Certificate Authority Server when they login to a Workstation.

1. On the **TFS-DC01** Server, open the **Group Policy Management** Console (gpmc.msc).
2. Open the **TFS Labs Certificates** GPO that was created earlier.
3. Open the **User Configuration > Policies > Windows Settings > Security Settings > Public Key Policies** node.
4. Open the **Certificate Services Client - Certificate Enrollment Policy** object.
5. In the **Properties** window, change the **Configuration Model** option to **Enabled**. Click the **OK** button to close the window.
6. Open the **Certificate Services Client - Auto-Enrollment** object.
7. In the **Properties** window, change the **Configuration Model** option to **Enabled**. Select the options for **Renew expired certificates, update pending certificates, and remove revoked certificates** and **Update certificate that use certificate templates** options. Click the **OK** button to close the window.

### 7.1.1 Group Policy Propagation ###

Once the Auto-Enrollment options have been added to Group Policy, allow up to 1 hour for the update to be processed in the entire Active Directory Forest.

## 7.2 Workstation Auto-Enrollment ##

Enabling the Workstation Auto-Enrollment feature will allow Workstations within the organization the ability to automatically receive a Certificate from the Active Directory Certificate Authority Server when they come online.

1. On the **TFS-DC01** Server, open the **Group Policy Management** Console (gpmc.msc).
2. Open the **TFS Labs Certificates** GPO that was created earlier.
3. Open the **Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies** node.
4. Open the **Certificate Services Client - Certificate Enrollment Policy** object.
5. In the **Properties** window, change the **Configuration Model** option to **Enabled**. Click the **OK** button to close the window.
6. Open the **Certificate Services Client - Auto-Enrollment** object.
7. In the **Properties** window, change the **Configuration Model** option to **Enabled**. Select the options for **Renew expired certificates, update pending certificates, and remove revoked certificates** and **Update certificate that use certificate templates** options. Click the **OK** button to close the window.

### 7.2.1 Group Policy Propagation ###

Once the Auto-Enrollment options have been added to Group Policy, allow up to 1 hour for the update to be processed in the entire Active Directory Forest.

## 7.3 Auto-Enrollment Verification ##

Confirm that the Auto-Enrollment of the **TFS Labs User Certificate** and **TFS Labs Workstation Certificate** is working correctly by running the **gpupdate /force** command on the **TFS-WIN10** and restarting it. If Auto-Enrollment is working correctly, there should be an additional Certificate in the **Personal** Store belonging to the User Account and the Workstation Account.
