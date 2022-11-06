---
title: "Az-104 Linux Edition"
categories:
  - Blog
tags:
  - Azure
  - Az-104
  - Linux
  - ARM Template 
---

# Az-104 mal anders - Linux statt Windows

In den offiziellen Lab Anleitungen zum Kurs *AZ-104 Microsoft Azure Administrator* werden ausschließlich Windows VMs verwendet. Im Kapitel 8 wird ganz kurz erwähnt, dass man auch Linux VMs erstellen und sich per SSH mit den Maschinen verbinden kann. In den Labs 4, 5, 6 und 8 lassen sich aber auch sehr gut Linuxcomputer statt Windows Server verwenden. Statt eines Passworts für die Anmeldung an der VM kommt hier ein SSH Schlüsselpaar zum Einsatz.

## 1. Erstellen der SSH Schlüssel

Selbst Windows 10/11 kommt ohne Zusatzsoftware aus, wenn es um SSH geht. Zunächst startet man eine Powershellsitzung. Das Erstellen des Schlüsselpaares erledigt der Befehl

```sh
ssh-keygen -m PEM -t rsa -b 4096
```

Führt man den Befehl mit den Standardwerten aus, wird ein neues Verzeichnis **.ssh** im Homeverzeichnis des Benutzers erstellt. Hierin finden sich zwei Dateien:

- Der private Schlüssel **id_rsa**
- Der öffentliche Schlüssel **id_rsa.pub**

## 2. Verwenden der SSH Schlüssel

Aus der Datei **id_rsa.pub** kopiert man sich den Schlüssel und fügt ihn z.B. in eine Parameter Datei ein

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmSize": {
            "value": "Standard_D2s_v3"
        },
        "adminUsername": {
            "value": "Student"
        },
        "adminPublicKey": {
            "value": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC0cj4xTv98ZeJKVxjdKywpr+jwM8EtUU5nhOP/w+It+OSsToqyNwI8ZkjJAREReKmUxf/tweYXC0HsvK3zThwJ0EUNiCfYsJYJIHHsVQNrOYE/7f/oCqSOacD6OZ0zo3czfeAG9ZUMaCnFOGmHBF6Al7+J5kHxmY9cbR/eEfsf1SEtNFO30R9noUfc5sKnk38cd/CuUBYrNOQ5M+fyPyQl6cGGw//bsRqTHzRls7srAOK474CSxI7712VTYALRVhqcI9eUQdN9bNqO/mGMLCQ4Tf6XCcQnzVKznQP73xK0FeKf78f6CzNZcgJHFKWHEbfK7Bnc4eDowinEm9XMjJZFXpneYAV6pnRRrpIXff4bhVPEnGFxU2u7q2JLklYXTwrGg0u9WfOiZooT9d/ZVvtzDfRVvO2RCv/9TW1Um4JH20Gen8VXrI6c0UM2QkKfZVxcjljRKbQ7/rFSerphvZfS55V3qdgJ4IeGbM09rMAuWf5ch/m1Lx7j203eVuAskhK0gsrPyZCHst+4EUQZcYkUIocH1hcHXSAeMfQjWTkCAtD+c6sy8QZuTDApFnaofvf23KmDUFEI1oR8kl7PL+kNH7e2uSUmXsrHUFSG/Lc1IPwRlXginZiV7IOqqCglVogXmTm0AYMPtFR7cDM2QQc1hidGPM4xo+mRwUoiz++6bw=="
        }
    }
}
```

Erstellt man im Azure Portal eine Linux VM, wählt man unter *SSH public key source* **Use existing public key** und kopiert den Schlüssel in das Feld **SSH public key**.

## 3. Verbinden mit der Linux VM

Im Azure Portal gibt es detailierte Anweisungen, wie und unter welchen Bedingungen man sich per SSH verbinden kann. Selbstverständlich muss der Port 22 über eine öffentlich IP (z.b. 11.22.13.14) erreichbar sein ;-)

Das Kommando

```sh
ssh Student@11.22.13.14
```

stellt die Verbindung her, führt man es auf dem gleichen PC und mit dem gleichen Benutzer aus, der die Schlüssel erzeugt hat.

## 4. Downloads

Die Lab Anleitungen, Templates, Parameterdateien und sonstige Dateien gibt es hier:

- Labfiles [allfiles-v221106.zip](https://jmenne.github.io/assets/Az104Linux/allfiles-v221106.zip)
- Anleitungen als docx [lab_instructions-v221106.zip](https://jmenne.github.io/assets/Az104Linux/lab_instructions-v221106.zip)
- Anleitungen als pdf [lab_instructions_pdf-v221106.zip](https://jmenne.github.io/assets/Az104Linux/lab_instructions_pdf-v221106.zip)
