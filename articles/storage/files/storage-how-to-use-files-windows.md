---
title: Einbinden einer Azure-Dateifreigabe und Zugreifen auf die Freigabe unter Windows | Microsoft-Dokumentation
description: Binden Sie eine Azure-Dateifreigabe unter Windows ein, und greifen Sie darauf zu.
services: storage
documentationcenter: na
author: RenaShahMSFT
manager: aungoo
editor: tysonn
ms.assetid: 
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/19/2017
ms.author: renash
ms.openlocfilehash: 5134fab447f1d1842369aeda4ebc1948a5d78262
ms.sourcegitcommit: cf4c0ad6a628dfcbf5b841896ab3c78b97d4eafd
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 10/21/2017
---
# <a name="mount-an-azure-file-share-and-access-the-share-in-windows"></a>Einbinden einer Azure-Dateifreigabe und Zugreifen auf die Freigabe unter Windows
[Azure Files](storage-files-introduction.md) ist das benutzerfreundliche Clouddateisystem von Microsoft. Azure-Dateifreigaben können in Windows und in Windows Server eingebunden werden. Dieser Artikel zeigt drei verschiedene Methoden zum Einbinden einer Azure-Dateifreigabe unter Windows: über die Benutzeroberfläche des Explorers, mithilfe von PowerShell und über die Eingabeaufforderung. 

Wenn Sie eine Azure-Dateifreigabe außerhalb der Azure-Region einbinden möchten, in der sie gehostet wird (beispielsweise lokal oder in einer anderen Azure-Region), muss das Betriebssystem SMB 3.0 unterstützen. 

Sie können Azure-Dateifreigaben unter einer Windows-Installation einbinden, die entweder auf einer Azure-VM oder lokal ausgeführt wird. In der folgenden Tabelle wird veranschaulicht, welche Betriebssystemversionen das Einbinden von Dateifreigaben in den einzelnen Umgebungen unterstützen:

| Windows-Version        | SMB-Version | Einbindung in Azure-VM möglich | Lokale Einbindung möglich |
|------------------------|-------------|-----------------------|----------------------|
| Halbjährlicher Windows Server-Kanal<sup>1</sup> | SMB 3.0 | Ja | Ja |
| Windows 10<sup>2</sup>  | SMB 3.0 | Ja | Ja |
| Windows Server 2016    | SMB 3.0     | Ja                   | Ja                  |
| Windows 8.1            | SMB 3.0     | Ja                   | Ja                  |
| Windows Server 2012 R2 | SMB 3.0     | Ja                   | Ja                  |
| Windows Server 2012    | SMB 3.0     | Ja                   | Ja                  |
| Windows 7              | SMB 2.1     | Ja                   | Nein                   |
| Windows Server 2008 R2 | SMB 2.1     | Ja                   | Nein                   |

<sup>1</sup>Windows Server Version 1709.  
<sup>2</sup>Windows 10, Versionen 1507, 1607, 1703 und 1709.

> [!Note]  
> Es empfiehlt sich, immer die neueste KB für Ihre Windows-Version zu verwenden.

## <a name="aprerequisites-for-mounting-azure-file-share-with-windows"></a></a>Voraussetzungen für das Einbinden von Azure-Dateifreigaben mit Windows 
* **Name des Speicherkontos:** Zum Einbinden einer Azure-Dateifreigabe benötigen Sie den Namen des Speicherkontos.

* **Speicherkontoschlüssel:** Zum Einbinden einer Azure-Dateifreigabe benötigen Sie den primären (oder sekundären) Speicherschlüssel. SAS-Schlüssel können derzeit nicht zum Einbinden verwendet werden.

* **Sicherstellen, dass Port 445 geöffnet ist:** Azure Files verwendet das SMB-Protokoll. SMB kommuniziert über den TCP-Port 445. Vergewissern Sie sich, dass der TCP-Port 445 des Clientcomputers nicht durch die Firewall blockiert wird.

## <a name="mount-the-azure-file-share-with-file-explorer"></a>Einbinden der Azure-Dateifreigabe über den Explorer
> [!Note]  
> Beachten Sie, dass sich die folgenden Anweisungen auf Windows 10 beziehen und bei älteren Versionen unter Umständen geringfügig unterscheiden können. 

1. **Öffnen Sie den Explorer:** Verwenden Sie dazu entweder das Startmenü, oder drücken Sie WINDOWS-TASTE+E.

2. **Navigieren Sie im linken Fensterbereich zu „Dieser PC“. Dadurch ändern sich die verfügbaren Menüs auf dem Menüband. Wählen Sie im Menü „Computer“ die Option „Netzlaufwerk verbinden“ aus.**
    
    ![Screenshot des Dropdownmenüs „Netzlaufwerk verbinden“](./media/storage-how-to-use-files-windows/1_MountOnWindows10.png)

3. **Kopieren Sie im Azure-Portal den UNC-Pfad aus dem Bereich „Verbinden“:** Falls Sie nicht genau wissen, wie Sie diese Information finden, lesen Sie [hier](storage-how-to-use-files-portal.md#connect-to-file-share) weiter.

    ![UNC-Pfad aus dem Verbindungsbereich von Azure Files](./media/storage-how-to-use-files-windows/portal_netuse_connect.png)

4. **Wählen Sie den Laufwerkbuchstaben aus, und geben Sie den UNC-Pfad ein.** 
    
    ![Screenshot des Dialogfelds „Netzlaufwerk verbinden“](./media/storage-how-to-use-files-windows/2_MountOnWindows10.png)

5. **Verwenden Sie den Speicherkontonamen mit vorangestelltem `Azure\` als Benutzername und einen Speicherkontoschlüssel als Kennwort.**
    
    ![Screenshot des Dialogfelds für die Netzwerkanmeldeinformationen](./media/storage-how-to-use-files-windows/3_MountOnWindows10.png)

6. **Verwenden Sie die Azure-Dateifreigabe wie gewünscht.**
    
    ![Die Azure-Dateifreigabe ist nun eingebunden.](./media/storage-how-to-use-files-windows/4_MountOnWindows10.png)

7. **Wenn Sie die Einbindung der Azure-Dateifreigabe wieder aufheben (also die Verbindung trennen) möchten, klicken Sie im Explorer unter „Netzwerk“ mit der rechten Maustaste auf den Eintrag für die Freigabe, und wählen Sie „Trennen“ aus.**

## <a name="mount-the-azure-file-share-with-powershell"></a>Einbinden der Azure-Dateifreigabe mithilfe von PowerShell
1. **Verwenden Sie zum Einbinden der Azure-Dateifreigabe den folgenden Befehl.** Ersetzen Sie dabei `<storage-account-name>`, `<share-name>`, `<storage-account-key>` und `<desired-drive-letter>` durch die entsprechenden Angaben.

    ```PowerShell
    $acctKey = ConvertTo-SecureString -String "<storage-account-key>" -AsPlainText -Force
    $credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\<storage-account-name>", $acctKey
    New-PSDrive -Name <desired-drive-letter> -PSProvider FileSystem -Root "\\<storage-account-name>.file.core.windows.net\<share-name>" -Credential $credential
    ```

2. **Verwenden Sie die Azure-Dateifreigabe wie gewünscht.**

3. **Wenn Sie die Azure-Dateifreigabe nicht mehr benötigen, heben Sie die Einbindung mithilfe des folgenden Befehls wieder auf:**

    ```PowerShell
    Remove-PSDrive -Name <desired-drive-letter>
    ```

> [!Note]  
> Mit dem Parameter `-Persist` für `New-PSDrive` können Sie die eingebundene Azure-Dateifreigabe für den Rest des Betriebssystems sichtbar machen.

## <a name="mount-the-azure-file-share-with-command-prompt"></a>Einbinden der Azure-Dateifreigabe über die Eingabeaufforderung
1. **Verwenden Sie zum Einbinden der Azure-Dateifreigabe den folgenden Befehl.** Ersetzen Sie dabei `<storage-account-name>`, `<share-name>`, `<storage-account-key>` und `<desired-drive-letter>` durch die entsprechenden Angaben.

    ```
    net use <desired-drive-letter>: \\<storage-account-name>.file.core.windows.net\<share-name> <storage-account-key> /user:Azure\<storage-account-name>
    ```

2. **Verwenden Sie die Azure-Dateifreigabe wie gewünscht.**

3. **Wenn Sie die Azure-Dateifreigabe nicht mehr benötigen, heben Sie die Einbindung mithilfe des folgenden Befehls wieder auf:**

    ```
    net use <desired-drive-letter>: /delete
    ```

> [!Note]  
> Sie können die Azure-Dateifreigabe so konfigurieren, dass die Verbindung nach einem Neustart des Systems automatisch wiederhergestellt wird, indem Sie die Anmeldeinformationen in Windows speichern. Verwenden Sie zum Speichern der Anmeldeinformationen den folgenden Befehl:
>   ```
>   cmdkey /add:<storage-account-name>.file.core.windows.net /user:AZURE\<storage-account-name> /pass:<storage-account-key>
>   ```

## <a name="next-steps"></a>Nächste Schritte
Weitere Informationen zu Azure Files erhalten Sie über diese Links.

* [Häufig gestellte Fragen](../storage-files-faq.md)
* [Troubleshoot Azure File storage problems in Windows](storage-troubleshoot-windows-file-connection-problems.md) (Beheben von Problemen mit Azure File Storage unter Windows)      

### <a name="conceptual-articles-and-videos"></a>Konzeptionelle Artikel und Videos
* [Azure Files: a frictionless cloud SMB file system for Windows and Linux (Azure Files: ein reibungsloses Cloud-SMB-Dateisystem für Windows und Linux)](https://azure.microsoft.com/documentation/videos/azurecon-2015-azure-files-storage-a-frictionless-cloud-smb-file-system-for-windows-and-linux/)
* [Verwenden von Azure Files mit Linux](../storage-how-to-use-files-linux.md)

### <a name="tooling-support-for-azure-files"></a>Toolunterstützung für Azure Files
* [Verwenden von AzCopy mit Microsoft Azure Storage](../common/storage-use-azcopy.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)
* [Verwenden der Azure-Befehlszeilenschnittstelle mit Azure-Speicher](../common/storage-azure-cli.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json#create-and-manage-file-shares)
* [Behandeln von Azure Files-Problemen unter Windows](storage-troubleshoot-windows-file-connection-problems.md)
* [Troubleshooting Azure Files problems – Linux](storage-troubleshoot-linux-file-connection-problems.md) (Behandeln von Azure Files-Problemen – Linux)

### <a name="blog-posts"></a>Blogbeiträge
* [Azure Files is now generally available](https://azure.microsoft.com/blog/azure-file-storage-now-generally-available/) (Azure Files ist jetzt allgemein verfügbar)
* [Inside Azure Files](https://azure.microsoft.com/blog/inside-azure-file-storage/) (Einblick in Azure Files)
* [Einführung in den Microsoft Azure-Dateidienst](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx)
* [Migrating Data to Microsoft Azure Files](https://azure.microsoft.com/blog/migrating-data-to-microsoft-azure-files/) (Migrieren von Daten zu Microsoft Azure Files)

### <a name="reference"></a>Referenz
* [Referenz zur Storage-Clientbibliothek für .NET](https://msdn.microsoft.com/library/azure/dn261237.aspx)
* [Referenz zur REST-API des Dateidiensts](http://msdn.microsoft.com/library/azure/dn167006.aspx)
