---
title: "Azure Active Directory Domain Services: Einbinden eines virtuellen CentOS-Computers in eine verwaltete Domäne | Microsoft-Dokumentation"
description: Einbinden eines virtuellen CentOS Linux-Computers in Azure AD Domain Services
services: active-directory-ds
documentationcenter: 
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 16100caa-f209-4cb0-86d3-9e218aeb51c6
ms.service: active-directory-ds
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/16/2017
ms.author: maheshu
ms.openlocfilehash: 65a9e4267c8883db5c8d8bfc5e0167577cd969d3
ms.sourcegitcommit: e266df9f97d04acfc4a843770fadfd8edf4fa2b7
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/11/2017
---
# <a name="join-a-centos-linux-virtual-machine-to-a-managed-domain"></a>Einbinden eines virtuellen CentOS Linux-Computers in eine verwaltete Domäne
Dieser Artikel beschreibt, wie ein virtueller CentOS Linux-Computer einer durch Azure AD Domain Services verwalteten Domäne beitritt.

## <a name="before-you-begin"></a>Voraussetzungen
Um die in diesem Artikel beschriebenen Aufgaben auszuführen, benötigen Sie Folgendes:
1. Ein gültiges **Azure-Abonnement**.
2. Ein **Azure AD-Verzeichnis** – entweder synchronisiert mit einem lokalen Verzeichnis oder als reines Cloud-Verzeichnis
3. **Azure AD Domain Services** müssen für das Azure AD-Verzeichnis aktiviert sein. Wenn dies noch nicht der Fall ist, führen Sie alle Aufgaben im Leitfaden [Erste Schritte](active-directory-ds-getting-started.md)aus.
4. Stellen Sie sicher, dass Sie die IP-Adressen der verwalteten Domäne nicht als DNS-Server für das virtuelle Netzwerk konfiguriert haben. Weitere Informationen finden Sie unter [Aktualisieren der DNS-Einstellungen für das virtuelle Azure-Netzwerk](active-directory-ds-getting-started-dns.md).
5. Führen Sie die Schritte aus, die zum [Synchronisieren der Kennwörter für Ihre mit Azure AD Domain Services verwaltete Domäne](active-directory-ds-getting-started-password-sync.md) erforderlich sind.


## <a name="provision-a-centos-linux-virtual-machine"></a>Bereitstellen eines virtuellen CentOS Linux-Computers
Stellen Sie mithilfe einer der folgenden Methoden einen virtuellen CentOS-Computer in Azure bereit:
* [Azure-Portal](../virtual-machines/linux/quick-create-portal.md)
* [Azure-Befehlszeilenschnittstelle](../virtual-machines/linux/quick-create-cli.md)
* [Azure PowerShell](../virtual-machines/linux/quick-create-powershell.md)

> [!IMPORTANT]
> * Stellen Sie den virtuellen Computer in **demselben virtuellen Netzwerk bereit, in dem Sie Azure AD Domain Services aktiviert haben**.
> * Wählen Sie ein **anderes Subnetz** als das aus, in dem Sie Azure AD Domain Services aktiviert haben.
>


## <a name="connect-remotely-to-the-newly-provisioned-linux-virtual-machine"></a>Herstellen einer Remoteverbindung mit dem neu bereitgestellten virtuellen Linux-Computer
Der virtuelle CentOS-Computer wurde in Azure bereitgestellt. Die nächste Aufgabe besteht darin, über das lokale Administratorkonto, das während der Bereitstellung des virtuellen Computers erstellt wurde, eine Remoteverbindung mit dem virtuellen Computer herzustellen.

Befolgen Sie die Anweisungen im Artikel [Anmelden bei einem virtuellen Computer mit Linux](../virtual-machines/linux/mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).


## <a name="configure-the-hosts-file-on-the-linux-virtual-machine"></a>Konfigurieren der Hostdatei auf dem virtuellen Linux-Computer
Bearbeiten Sie in Ihrem SSH-Terminal die Datei „/etc/hosts“, und aktualisieren Sie die IP-Adresse und den Hostnamen Ihres Computers.

```
sudo vi /etc/hosts
```

Geben Sie in der Hostdatei die folgenden Wert ein:

```
127.0.0.1 contoso-centos.contoso100.com contoso-centos
```
„contoso100.com“ steht für den DNS-Domänennamen Ihrer verwalteten Domäne. „contoso-centos“ steht für den Hostnamen des virtuellen CentOS-Computers, den Sie in die verwaltete Domäne einbinden.


## <a name="install-required-packages-on-the-linux-virtual-machine"></a>Installieren der erforderlichen Pakete auf dem virtuellen Linux-Computer
Als Nächstes installieren Sie Pakete, die für den Domänenbeitritt auf dem virtuellen Computer erforderlich sind. Geben Sie die folgenden Befehl in Ihrem SSH-Terminal ein, um die erforderlichen Pakete zu installieren:

    ```
    sudo yum install realmd sssd krb5-workstation krb5-libs oddjob oddjob-mkhomedir samba-common-tools
    ```


## <a name="join-the-linux-virtual-machine-to-the-managed-domain"></a>Einbinden der virtuellen Linux-Computers in die verwaltete Domäne
Nachdem die erforderlichen Pakete auf dem virtuellen Linux-Computer installiert sind, besteht die nächste Aufgabe darin, den virtuellen Computer in die verwaltete Domäne einzubinden.

1. Ermitteln Sie die durch Azure AD-Domänendienste verwaltete Domäne. Geben Sie in Ihrem SSH-Terminal folgenden Befehl ein:

    ```
    sudo realm discover CONTOSO100.COM
    ```

      > [!NOTE]
      > **Problembehandlung:** Wenn Ihre verwaltete Domäne über *Realm discover* (Bereich ermitteln) nicht gefunden werden kann:
        * Ensure that the domain is reachable from the virtual machine (try ping).
        * Check that the virtual machine has indeed been deployed to the same virtual network in which the managed domain is available.
        * Check to see if you have updated the DNS server settings for the virtual network to point to the domain controllers of the managed domain.
      >

2. Initialisieren Sie Kerberos. Geben Sie in Ihrem SSH-Terminal folgenden Befehl ein:

    > [!TIP]
    > * Geben Sie einen Benutzer an, der zur Gruppe „AAD DC-Administratoren“ gehört.
    > * Geben Sie den Domänennamen in Großbuchstaben an. Andernfalls führt kinit zu einem Fehler.
    >

    ```
    kinit bob@CONTOSO100.COM
    ```

3. Binden Sie den Computer in die Domäne ein. Geben Sie in Ihrem SSH-Terminal folgenden Befehl ein:

    > [!TIP]
    > Verwenden Sie das gleiche Benutzerkonto, das Sie im vorherigen Schritt („kinit“) angegeben haben.
    >

    ```
    sudo realm join --verbose CONTOSO100.COM -U 'bob@CONTOSO100.COM'
    ```

Wenn der Computer erfolgreich in die verwaltete Domäne eingebunden wurde, sollte eine Meldung angezeigt werden, dass der Computer erfolgreich im Bereich registriert wurde.


## <a name="verify-domain-join"></a>Überprüfen des Domänenbeitritts
Überprüfen Sie, ob der Computer der verwalteten Domäne erfolgreich beigetreten ist. Stellen Sie über eine andere SSH-Verbindung eine Verbindung mit der Domäne her, der der virtuelle CentOS-Computers beigetreten ist. Verwenden Sie ein Domänenbenutzerkonto, und überprüfen Sie anschließend, ob das Benutzerkonto ordnungsgemäß aufgelöst wurde.

1. Geben Sie in Ihrem SSH-Terminal den folgenden Befehl ein, um über SSH eine Verbindung mit dem der Domäne beigetretenen virtuellen CentOS-Computer herzustellen. Verwenden Sie ein Domänenkonto, das zu der verwalteten Domäne gehört (in diesem Fall z.B. 'bob@CONTOSO100.COM').
    ```
    ssh -l bob@CONTOSO100.COM contoso-centos.contoso100.com
    ```

2. Geben Sie in Ihrem SSH-Terminal den folgenden Befehl ein, um zu ermitteln, ob das Basisverzeichnis ordnungsgemäß initialisiert wurde.
    ```
    pwd
    ```

3. Geben Sie in Ihrem SSH-Terminal den folgenden Befehl ein, um zu ermitteln, ob die Gruppenmitgliedschaften ordnungsgemäß aufgelöst wurden.
    ```
    id
    ```


## <a name="troubleshooting-domain-join"></a>Problembehandlung beim Domänenbeitritt
Informationen finden Sie im Artikel [Problembehandlung beim Domänenbeitritt](active-directory-ds-admin-guide-join-windows-vm-portal.md#troubleshooting-domain-join) .

## <a name="related-content"></a>Verwandte Inhalte
* [Azure AD-Domänendienste – Leitfaden zu den ersten Schritten](active-directory-ds-getting-started.md)
* [Einbinden eines virtuellen Windows Server-Computers in eine verwaltete Domäne der Azure AD-Domänendienste](active-directory-ds-admin-guide-join-windows-vm.md)
* [Anmelden bei einem virtuellen Computer unter Linux](../virtual-machines/linux/mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Installing Kerberos (Installieren von Kerberos)](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Managing_Smart_Cards/installing-kerberos.html)
* [Red Hat Enterprise Linux 7 - Windows Integration Guide (Red Hat Enterprise Linux 7 – Windows-Integrationsleitfaden)](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Windows_Integration_Guide/index.html)
