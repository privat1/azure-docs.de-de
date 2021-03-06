---
title: "Ausführen von benutzerdefinierten Skripts auf Linux-VMs in Azure | Microsoft-Dokumentation"
description: "Automatisieren Sie Konfigurationsaufgaben für virtuelle Linux-Computer mithilfe der benutzerdefinierten Skripterweiterung."
services: virtual-machines-linux
documentationcenter: 
author: danielsollondon
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: cf17ab2b-8d7e-4078-b6df-955c6d5071c2
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 04/26/2017
ms.author: danis
ms.openlocfilehash: 53a241f12373acdb5d40575915d8d6c2f3c86b9a
ms.sourcegitcommit: 6fb44d6fbce161b26328f863479ef09c5303090f
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/10/2018
---
# <a name="using-the-azure-custom-script-extension-with-linux-virtual-machines"></a>Verwenden der benutzerdefinierten Skripterweiterung von Azure mit virtuellen Linux-Computern
Die benutzerdefinierte Skripterweiterung lädt Skripts auf virtuelle Azure-Computer herunter und führt sie aus. Diese Erweiterung ist hilfreich bei der Konfiguration nach der Bereitstellung, bei der Softwareinstallation oder bei anderen Konfigurations-/Verwaltungsaufgaben. Skripts können aus Azure Storage oder von einem anderen zugänglichen Speicherort im Internet heruntergeladen oder zur Laufzeit für die Erweiterung bereitgestellt werden. Die benutzerdefinierte Skripterweiterung kann in Azure Resource Manager-Vorlagen integriert und auch mithilfe der Azure-Befehlszeilenschnittstelle, mithilfe von PowerShell, über das Azure-Portal oder unter Verwendung der REST-API für virtuelle Azure-Computer ausgeführt werden.

In diesem Dokument erfahren Sie, wie Sie die benutzerdefinierte Skripterweiterung über die Azure-Befehlszeilenschnittstelle ausführen. Außerdem erfahren Sie, wie Sie eine Azure Resource Manager-Vorlage verwenden, und Sie erhalten Informationen zu Problembehandlungsschritten für Linux-Systeme.

## <a name="extension-configuration"></a>Konfiguration der Erweiterung
In der Konfiguration der benutzerdefinierten Skripterweiterung werden Aspekte wie der Skriptspeicherort und der auszuführende Befehl angegeben. Die Konfiguration kann in Konfigurationsdateien gespeichert oder in der Befehlszeile oder in einer Azure Resource Manager-Vorlage angegeben werden. Sensible Daten können in einer geschützten Konfiguration gespeichert werden. Diese ist verschlüsselt und wird nur auf dem virtuellen Computer entschlüsselt. Die geschützte Konfiguration ist hilfreich, wenn der Ausführungsbefehl vertrauliche Informationen (beispielsweise ein Kennwort) enthält.

### <a name="public-configuration"></a>Öffentliche Konfiguration
Schema:

**Hinweis**: Bei Eigenschaftennamen wird zwischen Groß- und Kleinschreibung unterschieden. Verwenden Sie die Namen wie hier angegeben, um Probleme bei der Bereitstellung zu vermeiden.

* **commandToExecute**(erforderlich; Zeichenfolge): Das auszuführende Skript für den Einstiegspunkt.
* **fileUris**(optional; Zeichenfolgenarray): Die URLs für die herunterzuladenden Dateien.
* **timestamp** (optional; ganze Zahl): Durch Ändern dieses Felds können Sie eine erneute Ausführung des Skripts auslösen.

```json
{
  "fileUris": ["<url>"],
  "commandToExecute": "<command-to-execute>"
}
```

### <a name="protected-configuration"></a>Geschützte Konfiguration
Schema:

**Hinweis**: Bei Eigenschaftennamen wird zwischen Groß- und Kleinschreibung unterschieden. Verwenden Sie die Namen wie hier angegeben, um Probleme bei der Bereitstellung zu vermeiden.

* **commandToExecute**(optional; Zeichenfolge): Das auszuführende Skript für den Einstiegspunkt. Verwenden Sie dieses Feld, falls Ihr Befehl vertrauliche Informationen (beispielsweise Kennwörter) enthält.
* **storageAccountName**(optional; Zeichenfolge): Der Name des Speicherkontos. Wenn Sie Speicheranmeldeinformationen angeben, muss es sich bei allen Datei-URIs um URLs für Azure-Blobs handeln.
* **storageAccountKey**(optional; Zeichenfolge): Der Zugriffsschlüssel des Speicherkontos.

```json
{
  "commandToExecute": "<command-to-execute>",
  "storageAccountName": "<storage-account-name>",
  "storageAccountKey": "<storage-account-key>"
}
```

## <a name="azure-cli"></a>Azure-Befehlszeilenschnittstelle
Wenn Sie die benutzerdefinierte Skripterweiterung über die Azure-Befehlszeilenschnittstelle ausführen, erstellen Sie eine Konfigurationsdatei oder Dateien mit mindestens dem Datei-URI und dem Skriptausführungsbefehl.

```azurecli
az vm extension set --resource-group myResourceGroup --vm-name myVM --name customScript --publisher Microsoft.Azure.Extensions --settings ./script-config.json
```

Optional können die Einstellungen im Befehl als JSON-formatierte Zeichenfolge angegeben werden. Dadurch kann die Konfiguration während der Ausführung und ohne separate Konfigurationsdatei angegeben werden.

```azurecli
az vm extension set '
  --resource-group exttest `
  --vm-name exttest `
  --name customScript `
  --publisher Microsoft.Azure.Extensions `
  --settings '{"fileUris": ["https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"],"commandToExecute": "./config-music.sh"}'
```

### <a name="azure-cli-examples"></a>Beispiele für die Azure-Befehlszeilenschnittstelle

**Beispiel 1:** Öffentliche Konfiguration mit Skriptdatei.

```json
{
  "fileUris": ["https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"],
  "commandToExecute": "./config-music.sh"
}
```

Befehl für die Azure-Befehlszeilenschnittstelle:

```azurecli
az vm extension set --resource-group myResourceGroup --vm-name myVM --name customScript --publisher Microsoft.Azure.Extensions --settings ./script-config.json
```

**Beispiel 2:** Öffentliche Konfiguration ohne Skriptdatei.

```json
{
  "commandToExecute": "apt-get -y update && apt-get install -y apache2"
}
```

Befehl für die Azure-Befehlszeilenschnittstelle:

```azurecli
az vm extension set --resource-group myResourceGroup --vm-name myVM --name customScript --publisher Microsoft.Azure.Extensions --settings ./script-config.json
```

**Beispiel 3:** Angabe des Skriptdatei-URIs mithilfe einer öffentlichen Konfigurationsdatei und Verwendung einer geschützten Konfigurationsdatei zum Angeben des auszuführenden Befehls.

Datei für die öffentliche Konfiguration:

```json
{
  "fileUris": ["https://gist.github.com/ahmetalpbalkan/b5d4a856fe15464015ae87d5587a4439/raw/466f5c30507c990a4d5a2f5c79f901fa89a80841/hello.sh"]
}
```

Datei für die geschützte Konfiguration:  

```json
{
  "commandToExecute": "./hello.sh <password>"
}
```

Befehl für die Azure-Befehlszeilenschnittstelle:

```azurecli
az vm extension set --resource-group myResourceGroup --vm-name myVM --name customScript --publisher Microsoft.Azure.Extensions --settings ./script-config.json --protected-settings ./protected-config.json
```

## <a name="resource-manager-template"></a>Resource Manager-Vorlage
Die benutzerdefinierte Skripterweiterung von Azure kann beim Bereitstellen virtueller Computer mithilfe einer Resource Manager-Vorlage ausgeführt werden. Zu diesem Zweck muss der Bereitstellungsvorlage ordnungsgemäß formatierter JSON-Code hinzugefügt werden.

### <a name="resource-manager-examples"></a>Resource Manager-Beispiele
**Beispiel 1:** Öffentliche Konfiguration.

```json
{
    "name": "scriptextensiondemo",
    "type": "extensions",
    "location": "[resourceGroup().location]",
    "apiVersion": "2015-06-15",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', parameters('scriptextensiondemoName'))]"
    ],
    "tags": {
        "displayName": "scriptextensiondemo"
    },
    "properties": {
        "publisher": "Microsoft.Azure.Extensions",
        "type": "CustomScript",
        "typeHandlerVersion": "2.0",
        "autoUpgradeMinorVersion": true,
      "settings": {
        "fileUris": [
          "https://gist.github.com/ahmetalpbalkan/b5d4a856fe15464015ae87d5587a4439/raw/466f5c30507c990a4d5a2f5c79f901fa89a80841/hello.sh"
        ],
        "commandToExecute": "sh hello.sh"
      }
    }
}
```

**Beispiel 2:** Ausführungsbefehl in der geschützten Konfiguration.

```json
{
  "name": "config-app",
  "type": "extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2015-06-15",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "https://gist.github.com/ahmetalpbalkan/b5d4a856fe15464015ae87d5587a4439/raw/466f5c30507c990a4d5a2f5c79f901fa89a80841/hello.sh"
      ]              
    },
    "protectedSettings": {
      "commandToExecute": "sh hello.sh <password>"
    }
  }
}
```

Ein vollständiges Beispiel finden Sie in der [Music Store-Demo für .NET Core](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-linux).

## <a name="troubleshooting"></a>Problembehandlung
Beim Ausführen der benutzerdefinierten Skripterweiterung wird das Skript erstellt oder wie im folgenden Beispiel in ein Verzeichnis heruntergeladen. Die Ausgabe des Befehls wird ebenfalls in diesem Verzeichnis gespeichert (in den Dateien `stdout` und `stderr`).

```bash
/var/lib/waagent/custom-script/download/0/
```

Das von der Azure-Skripterweiterung erzeugte Protokoll finden Sie hier:

```bash
/var/log/azure/custom-script/handler.log
```

Der Ausführungsstatus der benutzerdefinierten Skripterweiterung kann auch über die Azure-Befehlszeilenschnittstelle abgerufen werden.

```azurecli
az vm extension list -g myResourceGroup --vm-name myVM
```

Die Ausgabe sieht in etwa wie folgt aus:

```azurecli
info:    Executing command vm extension get
+ Looking up the VM "scripttst001"
data:    Publisher                   Name                                      Version  State
data:    --------------------------  ----------------------------------------  -------  ---------
data:    Microsoft.Azure.Extensions  CustomScript                              2.0      Succeeded
data:    Microsoft.OSTCExtensions    Microsoft.Insights.VMDiagnosticsSettings  2.3      Succeeded
info:    vm extension get command OK
```

## <a name="next-steps"></a>Nächste Schritte
Informationen zu anderen Skripterweiterungen für virtuelle Computer finden Sie unter [Informationen zu Erweiterungen und Features für virtuelle Computer](extensions-features.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

