---
title: "Autopilot VM Hardware Hash ohne OOBE"
categories:
  - Blog
tags:
  - Autopilot
  - Intune
  - Powershell
  - VM 
---

# Autopilot testen oder demonstrieren

Möchte man die Autopilot Bereitstellung demonstrieren oder benötigt man zu Testzwecken neue Windows PCs bietet sich eine virtuelle Maschine geradezu an. In dem Artikel [*Veranschaulichen der Autopilot-Bereitstellung*](https://learn.microsoft.com/de-de/windows/deployment/windows-autopilot/demonstrate-deployment-on-vm) stellt Microsoft eine ausführliche Anleitung dazu bereit. Der Knackpunkt hier: Die VM wird mit *Diesen PC zurücksetzen* auf die *Out-of-Box-Experience (OOBE)* zurückgesetzt, nachdem der Hardware Hash erfasst und in einer csv gespeichert wurde. Dieser Prozess dauert sehr lange, lässst sich aber deutlich abkürzen, wie in den folgenden Schritten gezeigt wird.

## 1. Eine neue VM erstellen

Ich gehe davon aus, dass man bereits eine neue VM mit Windows 10/11 installiert hat und man sich nun zu Beginn des OOBE Prozesses befindet. Wichtig ist, dass die VM eine Internetverbindung hat!

![Startseite des OOBE Prozesses]({{site.url}}{{site.baseurl}}/assets/images/Autopilot/Bild1.png)

## 2. Vorbereitungen im Intune admin center

- eine Gerätegruppe z.b. *Autopilopt Devices* mit dynamischer Zuweisung und folgender Regel erstellen. (device.devicePhysicalIDs -any (_ -contains "[ZTDId]"))

- ein Bereitstellungsprofil z.B. *AP Profile* erstellen und die Gruppe einschliessen

## 3. Powershell ist dein Freund

Man öffnet sich nun durch die Tastenkombination **SHIFT + F10** ein Kommandozeilenfenster und tippt **powershell**. Im Powershell Fenster geht es nun weiter mit

```powershell
Set-ExecutionPolicy -ExecutionPolicy Bypass

Install-Script Get-WindowsAutopilotInfo
```

Die drei nachfolgenden Fragen beantwortet man jeweils mi **J**

![Installation des Get-WindowsAutopilotInfo Skripts]({{site.url}}{{site.baseurl}}/assets/images/Autopilot/Bild2.png)

Der nächste Befehl extrahiert den Hardware Hash und registriert die VM in Intune.

```powershell
Get-WindowsAutopilotInfo.ps1 -Online
```

Hierdurch wird zunächst das Modul *WindowsAutopilotIntune* installiert, dann erscheint ein Anmeldefenster für den AzureAD Tenant. Nach der Anmeldung beginnt die Registrierung der VM als Autopilot Device in Intune. Dieser Vorgang dauert ca. 3-4 Minuten

![Registrirung des Geräts]({{site.url}}{{site.baseurl}}/assets/images/Autopilot/Bild4.png)

Im Intune admin Center erscheint das Gerät mit dem *Profile Status* *Not Assigned*. Nach einer kurzen Wartezeit ist das Gerät dann Mitglied der dynamischen Gerätegruppe und das Bereitstellungsprofil ist zugewiesen.
Der *Profile Status* zeigt nun *Assigned*.

## 4. VM ausschalten und neu starten

Nach dem Neustart der VM beginnt dann die Autopilot Bereitstellung.

![Beginn der Autopilot Bereitstellung]({{site.url}}{{site.baseurl}}/assets/images/Autopilot/Bild5.png)

Viel Spaß beim selbst ausprobieren!
