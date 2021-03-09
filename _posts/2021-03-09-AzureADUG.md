---
title: "Demo-Benutzer und -Gruppen im Azure Active Directory erstellen"
categories:
  - Blog
tags:
  - Azure
  - Azure AD
  - Powershell
  - csv
---

# Azure Active Directory: Benutzer und Gruppen erstellen

## So leer hier

Wenn man ein neues *Azure AD* Verzeichnis erstellt bekommt, wie es z.B. beim Nutzen des [kostenlosen Azure Kontos](https://aka.ms/azure-free-trial) passiert, ist nur ein Benutzer im Verzeichnis. Das ist das Konto, mit dem man sein kostenloses Konto eingerichtet hat.

Das wollen wir nun ändern, indem wir Powershell in Kombination mit einer .csv Datei verwenden.

## Die csv Datei

So könnte eine Datei mit dem Namen **users.csv** aussehen

```sh
GivenName,Surname,JobTitle,Department,UsageLocation
Ellen,Bogen,Leitung,IT,DE
Erkan,Nichtanders,Mitarbeiter,IT,DE
Ansgar,Ragentor,Mitarbeiter,Forschung,DE
Anna,Bolika,Leitung,Forschung,DE
Ben,Utzer,Leitung,Helpdesk,DE
Lasse,Reden,Mitarbeiter,Helpdesk,DE
Claudia,Manten,Leitung,Vertrieb,DE
Theo,Retisch,Mitarbeiter,Vertrieb,DE
Ed,Was,Mitarbeiter,Buchhaltung,DE
Gesa,Melte-Werke,Leitung,Buchhaltung,DE
Heinz,Ellmann,Mitarbeiter,Management,DE
Jack,Pott,Leitung,Management,DE
```

Wir geben also den Vor- und Nachnamen, einen Job Titel, die Abteilung und den Standort der Nutzung für den Benutzer in der Datei an. Die Angabe der UsageLocation ist notwendig, um dem Benutzer später eine Lizenz - z.B. eine AD Premium P2 - zuordnen zu können.

## Das Powershell Skript

Vorausgesetzt wird, dass das AzureAD Modul in der Powershell installiert ist. Dann kann man das folgende Skript verwenden, um ein paar Demo Benutzer und Gruppen zu erstellen. Die Namen der Gruppen entsprechen der Abteilung der Benutzer.

```powershell
### These are vairable you need to update to reflect your environment

$Directory = "xxxxx.onmicrosoft.com"
$NewUserPassword = "put_in_your_Password"
$CsvFile = "users.csv"

### Create a PowerShell connection to my directory.

$myCred = Get-Credential
Connect-AzureAD -credential $myCred

### Create a new Password Profile for the new users. We'll be using the same password for all new users in this example

$PasswordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
$PasswordProfile.Password = $NewUserPassword
$PasswordProfile.ForceChangePasswordNextLogin = $false

### Import the csv file. You will need to specify the path and file name of the CSV file in this cmdlet

$NewUsers = import-csv -Path $CsvFile

### Loop through all new users in the file. We'll use the ForEach cmdlet for this.

Foreach ($NewUser in $NewUsers) { 

### Construct the UserPrincipalName, the MailNickName and the DisplayName from the input data in the file 


    $UPN = $NewUser.GivenName + $NewUser.Surname[0] + "@" + $Directory
    $DisplayName = $NewUser.GivenName + " " + $NewUser.Surname
    $MailNickName = $NewUser.GivenName + "." + $NewUser.Surname
    
### Now that we have all the necessary data for to create the new user, we can execute the New-AzureADUser cmdlet  

    New-AzureADUser -UserPrincipalName $UPN -AccountEnabled $true `
    -DisplayName $DisplayName -GivenName $NewUser.GivenName -Surname $NewUser.Surname -MailNickName $MailNickName `
    -Department $Newuser.Department -JobTitle $NewUser.JobTitle `
    -PasswordProfile $PasswordProfile -UsageLocation $NewUser.UsageLocation
}

### Create new Securitygroups and add Members

$GroupNames = @("Forschung","IT","HelpDesk","Management","Vertrieb","Buchhaltung")
foreach ($GroupName in $GroupNames) {
    $NewGroup=New-AzureADGroup -DisplayName $GroupName -MailEnabled $false -SecurityEnabled $true -MailNickName "NotSet"
    $userIDs = (Get-AzureADUser | Where-Object Department -eq $GroupName).ObjectID 
    foreach ($userID in $userIDs){
       Add-AzureADGroupMember -RefObjectId $userID -ObjectId $NewGroup.ObjectID
    }
}
```
