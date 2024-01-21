---
title: "Command Listing"
geekdocNav: true
geekdocAnchor: false
---

Listed on this page are all commands that are included in the print edition of [Practical Guide to PKI with Windows Server - First Edition](https://mjcb.io/publications/practical-guide-to-pki-with-windows-server-first-edition/). This page is intended for anyone who purchased the physical copy of the book so that they don't need to manually type any of the entries.

Not included on this page are any commands that are listed in the Preface, Introduction, Glossary and Commands sections of the book.

Commands are organized based on what page they are printed on, from top to bottom.

{{< hint type=note title="Command Entry Modifications" >}}
These commands are displayed on this page exactly as they appear in the 12 chapters in the book. These commands have not been modified in any way.
{{< /hint >}}

{{< toc >}}

## Chapter 1 - Public Key Infrastructure Overview ##

There are no commands in this chapter.

## Chapter 2 - Certificate Authority Test Environment ##

### Page 41 ###

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

### Page 42 ###

```cmd
DISM.exe /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V
```

## Chapter 3 - Domain Controller and Workstation Setup ##

### Page 59 ###

```powershell
Install-WindowsFeature AD-Domain-Services, RSAT-ADDS
```

### Page 67 ###

```powershell
Import-Module ADDSDeployment
```

### Page 68 ###

```powershell
Install-ADDSForest ` 
-CreateDnsDelegation:$false ` 
-DatabasePath "C:\Windows\NTDS" ` 
-DomainMode "WinThreshold" ` 
-DomainName "corp.tfslabs.com" ` 
-DomainNetbiosName "TFSLABS" ` 
-ForestMode "WinThreshold" ` 
-InstallDns:$true `
-LogPath "C:\Windows\NTDS" ` 
-NoRebootOnCompletion:$false ` 
-SysvolPath "C:\Windows\SYSVOL" ` 
-Force:$true
```

### Page 73 ###

```powershell
New-ADOrganizationalUnit `
-Name "TFS Labs" -Path "DC=corp,DC=tfslabs,DC=com"

New-ADOrganizationalUnit `
-Name "TFS Servers" -Path "OU=TFS Labs,DC=corp,DC=tfslabs,DC=com"

New-ADOrganizationalUnit `
-Name "TFS Users" -Path "OU=TFS Labs,DC=corp,DC=tfslabs,DC=com"

New-ADOrganizationalUnit `
-Name "TFS Workstations" -Path "OU=TFS Labs,DC=corp,DC=tfslabs,DC=com"
```

### Page 78 ###

```powershell
New-ADUser `
-Name "Mary Smith" `
-GivenName "Mary" `
-Surname "Smith" `
-SamAccountName "msmith" `
-EmailAddress "msmith@corp.tfslabs.com" `
-Path "OU=TFS Users,OU=TFS Labs,DC=corp,DC=tfslabs,DC=com" `
-AccountPassword (Read-Host -AsSecureString "User Password") `
-ChangePasswordAtLogon $false `
-Enabled $true
```

### Page 79 ###

```powershell
Set-ADUser `
-Identity Administrator `
-EmailAddress "administrator@corp.tfslabs.com"
```

### Page 81 ###

```powershell
Get-ADComputer "TFS-WIN10" | `
Move-ADObject `
-TargetPath "OU=TFS Workstations,OU=TFS Labs,DC=corp,DC=tfslabs,DC=com"
```

## Chapter 4 - Offline Root CA Setup ##

### Page 96 ###

```powershell
Install-WindowsFeature BitLocker `
-IncludeAllSubFeature `
-IncludeManagementTools `
-Restart
```

### Page 105 ###

```cmd
manage-bde.exe -protectors -get C: > A:\BitLocker-TFS-ROOT-CA.txt
```

```powershell
sconfig.cmd
```

### Page 110 ###

```ini
[Version]
Signature="$Windows NT$"

[PolicyStatementExtension]
Policies=AllIssuancePolicy,InternalPolicy
Critical=FALSE

; Enables all Certificate Templates.
[AllIssuancePolicy]
OID=2.5.29.32.0

[InternalPolicy]
OID=1.2.3.4.1455.67.89.5
Notice="The TFS Labs Certification Authority is internal only."
URL=http://pki.corp.tfslabs.com/cps.html

[Certsrv_Server]
; Renewal information for the Root CA.
RenewalKeyLength=4096
RenewalValidityPeriod=Years
RenewalValidityPeriodUnits=10

; Disable support for issuing certificates using RSASSA-PSS.
AlternateSignatureAlgorithm=0

; The CRL publication period is the lifetime of the Root CA.
CRLPeriod=Years
CRLPeriodUnits=10

; The option for Delta CRL is disabled since this is a Root CA.
CRLDeltaPeriod=Days
CRLDeltaPeriodUnits=0
```

### Page 118 ###

```powershell
Add-WindowsFeature -Name ADCS-Cert-Authority -IncludeManagementTools
```

### Page 126 ###

```powershell
Install-AdcsCertificationAuthority `
-CAType StandaloneRootCA `
-CACommonName "TFS Labs Certificate Authority" `
-KeyLength 4096 `
-HashAlgorithm SHA256 `
-CryptoProviderName "RSA#Microsoft Software Key Storage Provider" `
-ValidityPeriod Years `
-ValidityPeriodUnits 10 `
-DatabaseDirectory $(Join-Path $env:SystemRoot "System32\CertLog") `
-Force
```

### Page 130 ###

```powershell
certutil.exe -setreg `
CA\DSConfigDN "CN=Configuration,DC=corp,DC=tfslabs,DC=com"
```

```powershell
certutil.exe -setreg CA\ValidityPeriodUnits 5
certutil.exe -setreg CA\ValidityPeriod "Years"
```

```powershell
certutil.exe -setreg CA\CRLPeriodUnits 52
certutil.exe -setreg CA\CRLPeriod "Weeks"
```

```powershell
certutil.exe -setreg CA\CRLOverlapPeriodUnits 12
certutil.exe -setreg CA\CRLOverlapPeriod "Hours"
```

```powershell
net stop CertSvc
net start CertSvc
```

### Page 132 ###

```powershell
auditpol.exe /set /category:"Object Access" `
/failure:enable /success:enable
```

```powershell
certutil.exe -setreg CA\AuditFilter 127
```

```powershell
net stop CertSvc
net start CertSvc
```

### Page 136 ###

```plain
http://pki.corp.tfslabs.com/CertData/<CaName><CRLNameSuffix><DeltaCRLAllowed>.crl
```

```plain
http://pki.corp.tfslabs.com/CertData/<ServerDNSName>_<CaName><CertificateName>.crt
```

### Page 137 ###

```powershell
certutil.exe -getreg CA\CRLPublicationURLs
```

```powershell
certutil.exe -getreg CA\CACertPublicationURLs
```

### Page 139 ###

```powershell
certutil.exe -setreg CA\CRLPublicationURLs "65:C:\Windows\system32\
CertSrv\CertEnroll\%3%8%9.crl\n8:ldap:///CN=%7%8,CN=%2,CN=CDP,
CN=Public Key Services,CN=Services,%6%10\n0:http://%1/CertEnroll/
%3%8%9.crl\n6:http://pki.corp.tfslabs.com/CertData/%3%8%9.crl"
```

```powershell
certutil.exe -setreg CA\CACertPublicationURLs "1:C:\Windows\system32\
CertSrv\CertEnroll\%1_%3%4.crt\n0:ldap:///CN=%7,CN=AIA,
CN=Public Key Services,CN=Services,%6%11\n0:http://%1/CertEnroll/
%1_%3%4.crt\n2:http://pki.corp.tfslabs.com/CertData/%1_%3%4.crt"
```

```powershell
net stop CertSvc
net start CertSvc
```

```powershell
certutil.exe -crl
```

```powershell
certutil.exe -getreg CA\CRLPublicationURLs
```

```powershell
certutil.exe -getreg CA\CACertPublicationURLs
```

## Chapter 5 - Subordinate CA Setup ##

### Page 143 ###

```powershell
Get-ADComputer "TFS-CA01" | `
Move-ADObject -TargetPath "OU=TFS Servers,OU=TFS Labs,DC=corp,DC=tfslabs,DC=com"
```

### Page 146 ###

```powershell
Add-DnsServerResourceRecordCName `
-Name "OCSP" `
-HostNameAlias "TFS-CA01.corp.tfslabs.com" `
-ZoneName "corp.tfslabs.com"
```

```powershell
Add-DnsServerResourceRecordCName `
-Name "PKI" `
-HostNameAlias "TFS-CA01.corp.tfslabs.com" `
-ZoneName "corp.tfslabs.com"
```

### Page 147 ###

```ini
[Version]
Signature="$Windows NT$"

[PolicyStatementExtension]
Policies=AllIssuancePolicy,InternalPolicy
Critical=FALSE

; Enables all Certificate Templates.
[AllIssuancePolicy]
OID=2.5.29.32.0

[InternalPolicy]
OID=1.2.3.4.1455.67.89.5
Notice="The TFS Labs Certification Authority is internal only."
URL=http://pki.corp.tfslabs.com/cps.html

[Certsrv_Server]
; Renewal information for the Subordinate CA.
RenewalKeyLength=4096
RenewalValidityPeriod=Years
RenewalValidityPeriodUnits=5

; Disable support for issuing certificates using RSASSA-PSS.
AlternateSignatureAlgorithm=0

; Load all certificate templates by default.
LoadDefaultTemplates=1
```

### Page 157 ###

```powershell
Add-WindowsFeature `
-Name ADCS-Cert-Authority, ADCS-Web-Enrollment, Web-Mgmt-Service `
-IncludeManagementTools
```

### Page 165 ###

```powershell
Install-AdcsCertificationAuthority `
-CAType EnterpriseSubordinateCA `
-CACommonName "TFS Labs Enterprise CA" `
-KeyLength 4096 `
-HashAlgorithm SHA256 `
-CryptoProviderName "RSA#Microsoft Software Key Storage Provider" `
-Force
```

### Page 166 ###

```powershell
Install-AdcsWebEnrollment -Force
```

### Page 172 ###

```cmd
cd C:\Windows\System32\inetsrv\
```

```cmd
appcmd.exe add vdir /app.name:"Default Web Site/" ^
/path:/CertData /physicalPath:C:\CertData
```

```cmd
appcmd.exe set config "Default Web Site/CertData" ^
/section:directoryBrowse /enabled:true
```

### Page 173 ###

```cmd
cd C:\Windows\System32\inetsrv\
```

```cmd
appcmd.exe set config "Default Web Site" ^
/section:system.webServer/Security/requestFiltering ^
-allowDoubleEscaping:True
```

### Page 183 ###

```powershell
certutil.exe –setreg ca\CRLFlags +CRLF_REVCHECK_IGNORE_OFFLINE
```

```powershell
net stop CertSvc
net start CertSvc
```

```powershell
certutil.exe -setreg CA\ValidityPeriodUnits 1
certutil.exe -setreg CA\ValidityPeriod "Years"
```

```powershell
net stop CertSvc
net start CertSvc
```

### Page 186 ###

```cmd
cd C:\Windows\System32\inetsrv\
```

```cmd
appcmd.exe set config "Default Web Site/CertEnroll" ^
/section:directoryBrowse /enabled:true
```

### Page 188 ###

```powershell
auditpol.exe /set /category:"Object Access" `
/failure:enable /success:enable
```

```powershell
certutil.exe -setreg CA\AuditFilter 127
```

```powershell
net stop CertSvc
net start CertSvc
```

### Page 191 ###

```plain
http://pki.corp.tfslabs.com/CertEnroll/<CaName><CRLNameSuffix><DeltaCRLAllowed>.crl
```

```plain
http://pki.corp.tfslabs.com/CertEnroll/<ServerDNSName>_<CaName><CertificateName>.crt
```

### Page 192 ###

```powershell
certutil.exe -getreg CA\CRLPublicationURLs
```

```powershell
certutil.exe -getreg CA\CACertPublicationURLs
```

### Page 194 ###

```powershell
certutil.exe -setreg CA\CRLPublicationURLs "65:C:\Windows\system32\
CertSrv\CertEnroll\%3%8%9.crl\n79:ldap:///CN=%7%8,CN=%2,CN=CDP,CN=
Public Key Services,CN=Services,%6%10\n0:http://%1/CertEnroll/%3%8%9.crl\
n6:http://pki.corp.tfslabs.com/CertEnroll/%3%8%9.crl"
```

```powershell
certutil.exe -setreg CA\CACertPublicationURLs "1:C:\Windows\system32\
CertSrv\CertEnroll\%1_%3%4.crt\n3:ldap:///CN=%7,CN=AIA,
CN=Public Key Services,CN=Services,%6%11\n0:http://%1/CertEnroll/
%1_%3%4.crt\n2:http://pki.corp.tfslabs.com/CertEnroll/%1_%3%4.crt"
```

```powershell
net stop CertSvc
net start CertSvc
```

```powershell
certutil.exe -crl
```

```powershell
certutil.exe -getreg CA\CRLPublicationURLs
```

```powershell
certutil.exe -getreg CA\CACertPublicationURLs
```

### Page 195 ###

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>TFS Labs Certification Practice Statement</title>
</head>
<body>

<h1>TFS Labs Certification Practice Statement</h1>

<p>The TFS Labs Certification Authority is internal only.</p>

<p>All issued certificates are for internal usage only.</p>

</body>
</html>
```

### Page 199 ###

```powershell
certutil.exe -dspublish `
-f "C:\CertData\TFS-ROOT-CA_TFS Labs Certificate Authority.crt" RootCA
```

### Page 200 ###

```powershell
certutil.exe -addstore `
-f root "C:\CertData\TFS-ROOT-CA_TFS Labs Certificate Authority.crt"
```

```powershell
certutil.exe -addstore `
-f root "C:\CertData\TFS Labs Certificate Authority.crl"
```

## Chapter 6 - Deploy Root and Subordinate Certificates ##

### Page 208 ###

```cmd
certutil.exe -encode der-certificate.cer base64-certificate.cer
```

```cmd
certutil.exe -decode base64-certificate.cer der-certificate.cer
```

### Page 213 ###

```cmd
gpupdate.exe /force
```

### Page 216 ###

```cmd
cd C:\Windows\System32\inetsrv\
```

```cmd
appcmd.exe add vdir /app.name:"Default Web Site/" ^
/path:/Certificates /physicalPath:C:\Certificates
```

```cmd
appcmd.exe set config "Default Web Site/Certificates" ^
/section:directoryBrowse /enabled:true
```

## Chapter 7 - Online Responder Role Configuration ##

### Page 221 ###

```powershell
Add-WindowsFeature Adcs-Online-Cert, RSAT-Online-Responder
```

### Page 225 ###

```powershell
Install-AdcsOnlineResponder -Force
```

### Page 226 ###

```powershell
certutil.exe -vocsproot
```

```powershell
iisreset.exe
```

### Page 228 ###

```powershell
certutil.exe -setreg CA\CACertPublicationURLs "1:C:\Windows\system32\
CertSrv\CertEnroll\%1_%3%4.crt\n3:ldap:///CN=%7,CN=AIA,
CN=Public Key Services,CN=Services,%6%11\n0:http://%1/CertEnroll/
%1_%3%4.crt\n2:http://pki.corp.tfslabs.com/CertEnroll/%1_%3%4.crt\n
32:http://ocsp.corp.tfslabs.com/ocsp"
```

```powershell
net stop CertSvc
net start CertSvc
```

### Page 247 ###

```powershell
certutil.exe -cainfo xchg
```

### Page 248 ###

```powershell
certutil.exe -URL "C:\TFS Labs Enterprise CA.cer"
```

## Chapter 8 - Private Key Archive and Recovery ##

There are no commands in this chapter.

## Chapter 9 - Certificate Template Customization ##

### Page 272 ##

```powershell
certutil.exe -getkey 3c0000000946c30a9587d470b9000000000009 msmith
```

```powershell
certutil.exe -recoverkey .\msmith msmith.pfx
```

### Page 282 ###

```powershell
iisreset.exe
```

## Chapter 10 - Certificate Enrollment ##

### Page 292 ###

```plain
[req]
default_bits           = 2048
distinguished_name     = req_distinguished_name
req_extensions         = req_ext

[req_distinguished_name]
commonName             = TFS-LINUX-WWW.corp.tfslabs.com
countryName            = CA
stateOrProvinceName    = Ontario
localityName           = Toronto
organizationName       = TFS Labs
organizationalUnitName = IT
emailAddress           = administrator@corp.tfslabs.com

[req_ext]
subjectAltName         = @alt_names

[alt_names]
DNS.1                  = TFS-LINUX-WWW.corp.tfslabs.com
DNS.2                  = WWW.corp.tfslabs.com
```

```bash
openssl req \
-out tfs-linux-www.csr \ -newkey rsa:2048 \
-nodes \
-keyout tfs-linux-www.key \ -config tfs-linux-www.cnf
```

### Page 295 ###

```bash
openssl pkcs7 -print_certs -in certnew.p7b -out tfs-labs-ca
```

```bash
cp certnew.cer tfs-linux-www.cer
```

```bash
sudo apt-get install apache2
```

```bash
sudo a2enmod ssl
```

```plain
ServerName TFS-LINUX-WWW.corp.tfslabs.com

SSLCertificateFile /etc/ssl/TFS-LINUX-WWW/tfs-linux-www.cer
SSLCertificateKeyFile /etc/ssl/TFS-LINUX-WWW/tfs-linux-www.key
SSLCertificateChainFile /etc/ssl/TFS-LINUX-WWW/tfs-labs-ca
```

### Page 296 ###

```bash
sudo a2ensite default-ssl.conf
```

```bash
sudo service apache2 restart
```

```bash
sudo systemctl enable apache2
```

### Page 297 ###

```bash
sudo apt-get install nginx
```

```plain
listen 443 ssl default_server;
listen [::]:443 ssl default_server;

ssl_certificate /etc/ssl/TFS-LINUX-WWW/tfs-linux-www.cer;
ssl_certificate_key /etc/ssl/TFS-LINUX-WWW/tfs-linux-www.key;
```

```bash
sudo service nginx restart
```

```bash
sudo systemctl enable nginx
```

## Chapter 11 - AD CS Post-Implementation Tasks ##

### Page 313 ###

```powershell
certutil.exe -crl
```

```powershell
certutil.exe -addstore `
-f root "C:\CertData\TFS Labs Certificate Authority.crl"
```

### Page 321 ###

```cmd
certutil.exe -dump
```

## Chapter 12 - AD CS Quick Start ##

### Page 330 ###

```powershell
Install-WindowsFeature AD-Domain-Services, RSAT-ADDS
```

### Page 331 ###

```powershell
Import-Module ADDSDeployment
```

```powershell
Install-ADDSForest `
-CreateDnsDelegation:$false `
-DatabasePath "C:\Windows\NTDS" `
-DomainMode "WinThreshold" `
-DomainName "corp.tfslabs.com" `
-DomainNetbiosName "TFSLABS" `
-ForestMode "WinThreshold" `
-InstallDns:$true `
-LogPath "C:\Windows\NTDS" `
-NoRebootOnCompletion:$false `
-SysvolPath "C:\Windows\SYSVOL" `
-Force:$true
```

### Page 332 ###

```powershell
New-ADOrganizationalUnit `
-Name "TFS Labs" `
-Path "DC=corp,DC=tfslabs,DC=com"

New-ADOrganizationalUnit `
-Name "TFS Servers" `
-Path "OU=TFS Labs,DC=corp,DC=tfslabs,DC=com"

New-ADOrganizationalUnit `
-Name "TFS Users" `
-Path "OU=TFS Labs,DC=corp,DC=tfslabs,DC=com"

New-ADOrganizationalUnit `
-Name "TFS Workstations" `
-Path "OU=TFS Labs,DC=corp,DC=tfslabs,DC=com"
```

```powershell
Set-ADUser `
-Identity Administrator `
-EmailAddress "administrator@corp.tfslabs.com"
```

### Page 334 ###

```ini
[Version]
Signature="$Windows NT$"

[PolicyStatementExtension]
Policies=AllIssuancePolicy,InternalPolicy
Critical=FALSE

; Enables all Certificate Templates.
[AllIssuancePolicy]
OID=2.5.29.32.0

[InternalPolicy]
OID=1.2.3.4.1455.67.89.5
Notice="The TFS Labs Certification Authority is internal only."
URL=http://pki.corp.tfslabs.com/cps.html

[Certsrv_Server]
; Renewal information for the Root CA.
RenewalKeyLength=4096
RenewalValidityPeriod=Years
RenewalValidityPeriodUnits=10

; Disable support for issuing certificates using RSASSA-PSS.
AlternateSignatureAlgorithm=0

; The CRL publication period is the lifetime of the Root CA.
CRLPeriod=Years
CRLPeriodUnits=10

; The option for Delta CRL is disabled since this is a Root CA.
CRLDeltaPeriod=Days
CRLDeltaPeriodUnits=0
```

### Page 335 ###

```powershell
Add-WindowsFeature -Name ADCS-Cert-Authority -IncludeManagementTools
```

### Page 336 ###

```powershell
Install-AdcsCertificationAuthority `
-CAType StandaloneRootCA `
-CACommonName "TFS Labs Certificate Authority" `
-KeyLength 4096 `
-HashAlgorithm SHA256 `
-CryptoProviderName "RSA#Microsoft Software Key Storage Provider" `
-ValidityPeriod Years `
-ValidityPeriodUnits 10 `
-DatabaseDirectory $(Join-Path $env:SystemRoot "System32\CertLog") `
-Force
```

### Page 338 ###

```powershell
certutil.exe -setreg `
CA\DSConfigDN "CN=Configuration,DC=corp,DC=tfslabs,DC=com"
```

```powershell
certutil.exe -setreg CA\ValidityPeriodUnits 5
certutil.exe -setreg CA\ValidityPeriod "Years"
```

```powershell
certutil.exe -setreg CA\CRLPeriodUnits 52
certutil.exe -setreg CA\CRLPeriod "Weeks"
```

```powershell
certutil.exe -setreg CA\CRLOverlapPeriodUnits 12
certutil.exe -setreg CA\CRLOverlapPeriod "Hours"
```

```powershell
net stop CertSvc
net start CertSvc
```

### Page 340 ###

```powershell
certutil.exe -setreg CA\CRLPublicationURLs "65:C:\Windows\system32\
CertSrv\CertEnroll\%3%8%9.crl\n8:ldap:///CN=%7%8,CN=%2,CN=CDP,
CN=Public Key Services,CN=Services,%6%10\n0:http://%1/CertEnroll/
%3%8%9.crl\n6:http://pki.corp.tfslabs.com/CertData/%3%8%9.crl"
```

```powershell
certutil.exe -setreg CA\CACertPublicationURLs "1:C:\Windows\system32\
CertSrv\CertEnroll\%1_%3%4.crt\n0:ldap:///CN=%7,CN=AIA,
CN=Public Key Services,CN=Services,%6%11\n0:http://%1/CertEnroll/
%1_%3%4.crt\n2:http://pki.corp.tfslabs.com/CertData/%1_%3%4.crt"
```

```powershell
net stop CertSvc
net start CertSvc
```

```powershell
certutil.exe -crl
```

### Page 342 ###

```powershell
Get-ADComputer "TFS-CA01" | `
Move-ADObject -TargetPath "OU=TFS Servers,OU=TFS Labs,DC=corp,DC=tfslabs,DC=com"
```

### Page 343 ###

```powershell
Add-DnsServerResourceRecordCName `
-Name "PKI" -HostNameAlias "TFS-CA01.corp.tfslabs.com" -ZoneName "corp.tfslabs.com"
```

```ini
[Version]
Signature="$Windows NT$"

[PolicyStatementExtension]
Policies=AllIssuancePolicy,InternalPolicy
Critical=FALSE

; Enables all Certificate Templates.
[AllIssuancePolicy]
OID=2.5.29.32.0

[InternalPolicy]
OID=1.2.3.4.1455.67.89.5
Notice="The TFS Labs Certification Authority is internal only."
URL=http://pki.corp.tfslabs.com/cps.html

[Certsrv_Server]
; Renewal information for the Subordinate CA.
RenewalKeyLength=4096
RenewalValidityPeriod=Years
RenewalValidityPeriodUnits=5

; Disable support for issuing certificates using RSASSA-PSS.
AlternateSignatureAlgorithm=0

; Load all certificate templates by default.
LoadDefaultTemplates=1
```

### Page 344 ###

```powershell
Add-WindowsFeature `
-Name ADCS-Cert-Authority, ADCS-Web-Enrollment, Web-Mgmt-Service `
-IncludeManagementTools
```

### Page 345 ###

```powershell
Install-AdcsCertificationAuthority `
-CAType EnterpriseSubordinateCA `
-CACommonName "TFS Labs Enterprise CA" `
-KeyLength 4096 `
-HashAlgorithm SHA256 `
-CryptoProviderName "RSA#Microsoft Software Key Storage Provider" `
-Force
```

```powershell
Install-AdcsWebEnrollment -Force
```

### Page 347 ###

```cmd
cd C:\Windows\System32\inetsrv\
```

```cmd
appcmd.exe add vdir /app.name:"Default Web Site/" ^
/path:/CertData /physicalPath:C:\CertData
```

```cmd
appcmd.exe set config "Default Web Site/CertData" ^
/section:directoryBrowse /enabled:true
```

### Page 348 ###

```cmd
cd C:\Windows\System32\inetsrv\
```

```cmd
appcmd.exe set config "Default Web Site" ^
/section:system.webServer/Security/requestFiltering ^
-allowDoubleEscaping:True
```

### Page 350 ###

```powershell
certutil.exe -setreg ca\CRLFlags +CRLF_REVCHECK_IGNORE_OFFLINE
```

```powershell
net stop CertSvc
net start CertSvc
```

### Page 351 ###

```powershell
certutil.exe -setreg CA\ValidityPeriodUnits 1
certutil.exe -setreg CA\ValidityPeriod "Years"
```

```powershell
net stop CertSvc
net start CertSvc
```

```cmd
cd C:\Windows\System32\inetsrv\
```

```cmd
appcmd.exe set config "Default Web Site/CertEnroll" ^
/section:directoryBrowse /enabled:true
```

### Page 352 ###

```powershell
certutil.exe -setreg CA\CRLPublicationURLs "65:C:\Windows\system32\
CertSrv\CertEnroll\%3%8%9.crl\n79:ldap:///CN=%7%8,CN=%2,CN=CDP,CN=
Public Key Services,CN=Services,%6%10\n0:http://%1/CertEnroll/%3%8%9.crl\
n6:http://pki.corp.tfslabs.com/CertEnroll/%3%8%9.crl"
```

```powershell
certutil.exe -setreg CA\CACertPublicationURLs "1:C:\Windows\system32\
CertSrv\CertEnroll\%1_%3%4.crt\n3:ldap:///CN=%7,CN=AIA,
CN=Public Key Services,CN=Services,%6%11\n0:http://%1/CertEnroll/
%1_%3%4.crt\n2:http://pki.corp.tfslabs.com/CertEnroll/%1_%3%4.crt"
```

```powershell
net stop CertSvc
net start CertSvc
```

### Page 353 ###

```powershell
certutil.exe -crl
```

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>TFS Labs Certification Practice Statement</title>
</head>
<body>

<h1>TFS Labs Certification Practice Statement</h1>

<p>The TFS Labs Certification Authority is internal only.</p>

<p>All issued certificates are for internal usage only.</p>

</body>
</html>
```

### Page 355 ###

```powershell
certutil.exe -dspublish `
-f "C:\CertData\TFS-ROOT-CA_TFS Labs Certificate Authority.crt" RootCA
```

```powershell
certutil.exe -addstore `
-f root "C:\CertData\TFS-ROOT-CA_TFS Labs Certificate Authority.crt"
```

```powershell
certutil.exe -addstore `
-f root "C:\CertData\TFS Labs Certificate Authority.crl"
```
