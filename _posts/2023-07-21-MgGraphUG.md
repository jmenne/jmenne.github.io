---
title: "Demo-Benutzer und -Gruppen  mit dem Microsoft.Graph Modul erstellen"
categories:
  - Blog
tags:
  - Azure
  - Microsoft.Graph
  - Microsoft Entra ID
  - PowerShell
  - csv
---

# Microsoft.Graph: Benutzer und Gruppen erstellen

## So leer hier

Wenn man ein neues *Entra ID* Verzeichnis erstellt bekommt, wie es z.B. beim Nutzen des [kostenlosen Azure Kontos](https://aka.ms/azure-free-trial) passiert, ist nur ein Benutzer im Verzeichnis. Das ist das Konto, mit dem man sein kostenloses Konto eingerichtet hat.

Bisher konnte man das *AzureAD Modul* und PowerShell benutzen, um schnell ein paar Demo-Benutzer und -Gruppen zu erstellen. Wie man in dem Artikel [Important: Azure AD Graph Retirement and Powershell Module Deprecation](https://techcommunity.microsoft.com/t5/microsoft-entra-azure-ad-blog/important-azure-ad-graph-retirement-and-powershell-module/ba-p/3848270) nachlesen kann, ist damit aber ab dem 30. März 2024 Schluß. Bis dahin müssen alte Azure AD PowerShell Skripte auf Microsoft Graph PowerShell migriert sein!

Hilfe zur Migration, insbesondere zu den erforderlichen Berechtigungen und eine Cmdlet Map, bietet dieser [Artikel](https://learn.microsoft.com/en-us/powershell/microsoftgraph/migration-steps?view=graph-powershell-1.0)

Nun soll gezeigt werden, wie man mit einer csv Datei, der PowerShell und dem Microsoft.Graph Modul Benutzer und Gruppen im Entra ID Verzeichnis erstellt.

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

Wir geben also den Vor- und Nachnamen, einen Job Titel, die Abteilung und den Standort der Nutzung für den Benutzer in der Datei an. Die Angabe der UsageLocation ist notwendig, um dem Benutzer später eine Lizenz - z.B. eine Microsoft Entra ID P2 - zuordnen zu können.

## Das PowerShell Skript

Vorausgesetzt wird, dass das *Microsoft.Graph Modul* in der PowerShell installiert ist. Dann kann man das folgende Skript verwenden, um ein paar Demo Benutzer und Gruppen zu erstellen. Die Namen der Gruppen entsprechen der Abteilung der Benutzer.

```powershell
### These are vairable you need to update to reflect your environment

$Directory = "xxxxx.onmicrosoft.com"
$NewUserPassword = "put_in_your_Password"
$CsvFile = "users.csv"

### Create a PowerShell connection to my directory.

Connect-MgGraph -Scopes 'User.ReadWrite.All','Directory.ReadWrite.All','Group.ReadWrite.All'

### Create a new Password Profile for the new users. We'll be using the same password for all new users in this example

$PasswordProfile = @{
  Password = $NewUserPassword
  ForceChangePasswordNextSignIn = $false
  }

### Import the csv file. You will need to specify the path and file name of the CSV file in this cmdlet

$NewUsers = import-csv -Path $CsvFile

### Loop through all new users in the file. We'll use the ForEach cmdlet for this.

Foreach ($NewUser in $NewUsers) { 

### Construct the UserPrincipalName, the MailNickName and the DisplayName from the input data in the file 


    $UPN = $NewUser.GivenName + $NewUser.Surname[0] + "@" + $Directory
    $DisplayName = $NewUser.GivenName + " " + $NewUser.Surname
    $MailNickName = $NewUser.GivenName + "." + $NewUser.Surname
    
### Now that we have all the necessary data to create the new user, we can execute the New-MgUser cmdlet  

    New-MgUser -UserPrincipalName $UPN -AccountEnabled `
    -DisplayName $DisplayName -GivenName $NewUser.GivenName -Surname $NewUser.Surname -MailNickName $MailNickName `
    -Department $Newuser.Department -JobTitle $NewUser.JobTitle `
    -PasswordProfile $PasswordProfile -UsageLocation $NewUser.UsageLocation
}

### Create new Securitygroups and add Members

$GroupNames = @("Forschung","IT","HelpDesk","Management","Vertrieb","Buchhaltung")
foreach ($GroupName in $GroupNames) {
    $NewGroup=New-MgGroup -DisplayName $GroupName -MailEnabled:$false -SecurityEnabled -MailNickName "NotSet"
    $userIDs = (Get-MgUser -Property ID,Department| Select-Object ID,Department | Where-Object Department -eq $GroupName).ID 
    foreach ($userID in $userIDs){
       New-MgGroupMember -DirectoryObjectId $userID -GroupId $NewGroup.ID
    }
}
```
