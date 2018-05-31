---
title: "Exporting an Azure App Service Certificate to a pfx file using Powershell"
date: 2018-05-31T09:50:27+03:00
categories: ['Article', 'Tutorials']
tags: ['App Service Certificate', 'Azure', 'Powershell', 'pfx']
author: "Petre"
---

There are a couple of guides out there regarding these steps. This is a simplified version.
<!--more-->

You only requires three variables:

-   A password used to encrypt the private key (which will be used when importing the pfx)
-   The name of the KeyVault used to store the App Service Certificate
-   The name of the Secret which represents the actual certificate

Additionally, if you have multiple subscriptions, you will also need to run `Set-AzureRmContext` after logging in to select the correct sub id.

##### Getting the Key Vault name

1.  Open the App Service Certificate in the portal
2.  Open Certificate Configuration
3.  Click on `Step 1. Store` and note down the Key Vault name

##### Getting the Secret name

4.  Click on "Manage KeyVault"
5.  Open Secrets
6.  The secret name will begin with the App Service Certificate name, followed by a GUID. Also make sure you get the current version by looking at the Expiration Date
    `Example: mycert2b06e7fe-06a9-4363-aed2-6e9b7243574d`

##### Export the certificate

```powershell
$subscriptionId = "yoursubscriptionID"

Login-AzureRmAccount
Set-AzureRmContext -SubscriptionId $subscriptionId

#Set variables for getting and exporting
$pfxpassword = "password"
$keyVaultName = "keyvaultname"
$keyVaultSecretName = "secretName"
 
$secret = Get-AzureKeyVaultSecret -VaultName $keyVaultName -Name $keyVaultSecretName

$pfxCertObject= New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @([Convert]::FromBase64String($secret.SecretValueText),"", [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::Exportable)

$currentDirectory = (Get-Location -PSProvider FileSystem).ProviderPath
[Environment]::CurrentDirectory = (Get-Location -PSProvider FileSystem).ProviderPath
[io.file]::WriteAllBytes(".\appservicecertificate.pfx", $pfxCertObject.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $pfxpassword))

Write-Host "Created an App Service Certificate copy at: $currentDirectory\appservicecertificate.pfx"
Write-Warning "For security reasons, do not store the PFX password. Use it directly from the console as required."
Write-Host "PFX password: $pfxpassword"
```