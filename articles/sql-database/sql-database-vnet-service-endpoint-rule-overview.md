---
title: "Dienstendpunkte und Regeln eines virtuellen Netzwerks für Azure SQL-Datenbank | Microsoft-Dokumentation"
description: "Markieren Sie ein Subnetz als Dienstendpunkt eines virtuellen Netzwerks. Nutzen Sie dann den Endpunkt als Regel für ein virtuelles Netzwerk für die Zugriffssteuerungsliste Ihrer Azure SQL-Datenbank-Instanz. Ihre Azure SQL-Datenbank-Instanz akzeptiert anschließend Nachrichten von allen virtuellen Computern und anderen Knoten im Subnetz."
services: sql-database
documentationcenter: 
author: MightyPen
manager: jhubbard
editor: 
tags: 
ms.assetid: 
ms.service: sql-database
ms.custom: VNet Service endpoints
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: On Demand
ms.date: 11/13/2017
ms.author: genemi
ms.openlocfilehash: ce223fbd6a69bc789f902f9478b5255edfd44844
ms.sourcegitcommit: b5c6197f997aa6858f420302d375896360dd7ceb
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/21/2017
---
# <a name="use-virtual-network-service-endpoints-and-rules-for-azure-sql-database"></a>Verwenden von Dienstendpunkten und Regeln eines virtuellen Netzwerks für Azure SQL-Datenbank

*Regeln für ein virtuelles Netzwerk* sind eine Firewallsicherheitsfunktion, die steuert, ob Ihr Azure SQL-Datenbankserver Nachrichten akzeptiert, die von bestimmten Subnetzen in virtuellen Netzwerken gesendet werden. In diesem Artikel wird erklärt, warum Regeln für ein virtuelles Netzwerk mitunter die beste Möglichkeit darstellen, Nachrichten an Ihre Azure SQL-Datenbank-Instanz sicher zuzulassen.

Damit eine Regel für ein virtuelles Netzwerk erstellt werden kann, muss zuerst ein [Dienstendpunkt eines virtuellen Netzwerks][vm-virtual-network-service-endpoints-overview-649d] vorhanden sein, auf den die Regel verweisen kann.


> [!NOTE]
> Für Azure SQL-Datenbank ist dieses Feature in allen Regionen der öffentlichen Azure-Cloud in der Vorschau verfügbar.

#### <a name="how-to-create-a-virtual-network-rule"></a>Erstellen einer Regel für ein virtuelles Netzwerk

Wenn Sie nur eine Regel für ein virtuelles Netzwerk erstellen, können Sie mit den Schritten und der Erklärung [weiter unten in diesem Artikel](#anchor-how-to-by-using-firewall-portal-59j) fortfahren.






<a name="anch-terminology-and-description-82f" />

## <a name="terminology-and-description"></a>Terminologie und Beschreibung

**Virtuelles Netzwerk:** Sie können mit Ihrem Azure-Abonnement virtuelle Netzwerke verknüpfen.

**Subnetz:** Ein virtuelles Netzwerk enthält **Subnetze**. Ihre virtuellen Azure-Computer (VMs) sind Subnetzen zugewiesen. Ein Subnetz kann mehrere VMs oder andere Computeknoten enthalten. Computeknoten, die sich außerhalb Ihres virtuellen Netzwerks befinden, können nicht auf Ihr virtuelles Netzwerk zugreifen, es sei denn, Sie konfigurieren für sie den sicheren Zugriff.

**Dienstendpunkt eines virtuellen Netzwerks:** Ein [Dienstendpunkt eines virtuellen Netzwerks][vm-virtual-network-service-endpoints-overview-649d] ist ein Subnetz, dessen Eigenschaftswerte mindestens einen formalen Azure-Diensttypnamen enthalten. In diesem Artikel beschäftigen wir uns mit dem Typnamen **Microsoft.Sql**, der auf einen Azure-Dienst mit dem Namen „SQL-Datenbank“ verweist.

**Regel für ein virtuelles Netzwerk:** Eine Regel für ein virtuelles Netzwerk für Ihren Azure SQL-Datenbankserver ist ein Subnetz, das in der Zugriffssteuerungsliste des SQL-Datenbankservers aufgeführt ist. Um in die Zugriffssteuerungsliste für Ihre SQL-Datenbank-Instanz zu gelangen, muss das Subnetz den Typnamen **Microsoft.Sql** enthalten.

Eine Regel für ein virtuelles Netzwerk weist Ihren Azure SQL-Datenbankserver an, Nachrichten von jedem Knoten anzunehmen, der zum Subnetz gehört.







<a name="anch-benefits-of-a-vnet-rule-68b" />

## <a name="benefits-of-a-virtual-network-rule"></a>Vorteile einer Regel für ein virtuelles Netzwerk

Bis Sie Maßnahmen ergreifen, können die virtuellen Computer in Ihren Subnetzen nicht mit Ihrer Azure SQL-Datenbank-Instanz kommunizieren. Eine Aktion zum Herstellen der Kommunikation stellt die Erstellung einer Regel für virtuelle Netzwerke dar. Die Begründung der Entscheidung für eine VNET-Regel erfordert eine Erörterung der Vor- und Nachteile, die die von der Firewall gebotenen konkurrierenden Sicherheitsoptionen berücksichtigt.

#### <a name="a-allow-access-to-azure-services"></a>A: Zugriff auf Azure-Dienste erlauben

Der Firewallbereich enthält eine Schaltfläche des Typs **EIN/AUS** mit der Bezeichnung **Zugriff auf Azure-Dienste erlauben**. Die Einstellung **EIN** lässt Nachrichten von allen Azure IP-Adressen und aus allen Azure-Subnetzen zu. Diese Azure-IP-Adressen oder -Subnetze gehören möglicherweise nicht Ihnen. Die Einstellung **EIN** lässt wahrscheinlich einen umfassenderen Zugriff auf Ihre SQL-Datenbank-Instanz zu, als von Ihnen gewünscht. Eine Regel für ein virtuelles Netzwerk ermöglicht eine präzisere Steuerung.

#### <a name="b-ip-rules"></a>B: IP-Regeln

Die Firewall von SQL-Datenbank ermöglicht Ihnen das Angeben von IP-Adressbereichen, aus denen Nachrichten an die SQL-Datenbank-Instanz gesendet werden dürfen. Dieser Ansatz eignet sich gut für statische IP-Adressen, die sich außerhalb des privaten Azure-Netzwerks befinden. Doch viele Knoten innerhalb des privaten Azure-Netzwerks sind mit *dynamischen* IP-Adressen konfiguriert. Dynamische IP-Adressen können sich ändern, z.B. wenn Ihre VM neu gestartet wird. Es wäre töricht, eine dynamische IP-Adresse in einer Firewallregel in einer Produktionsumgebung anzugeben.

Sie können die IP-Option weiter nutzen, indem Sie eine *statische* IP-Adresse für Ihre VM abrufen. Einzelheiten finden Sie unter [Konfigurieren von privaten IP-Adressen für einen virtuellen Computer über das Azure-Portal][vm-configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-portal-321w].

Der Ansatz mit statischen IP-Adressen kann jedoch schwierig zu handhaben und aufwendig sein, wenn er in großem Maßstab befolgt wird. Regeln für ein virtuelles Netzwerk sind einfacher einzurichten und zu verwalten.

#### <a name="c-cannot-yet-have-sql-database-on-a-subnet"></a>C. Azure SQL-Datenbank noch nicht in einem Subnetz unterstützt

Wenn sich Ihr Azure SQL-Datenbankserver auf einem Knoten in einem Subnetz in Ihrem virtuellen Netzwerk befände, könnten alle Knoten innerhalb des virtuellen Netzwerks mit Ihrer SQL-Datenbank-Instanz kommunizieren. In diesem Fall könnten Ihre virtuellen Computer mit der SQL-Datenbank-Instanz kommunizieren, ohne dass Regeln für ein virtuelles Netzwerk oder IP-Regeln erforderlich sind.

Doch im September 2017 gehört der Azure SQL-Datenbankdienst noch nicht zu den Diensten, die einem Subnetz zugewiesen werden können.






<a name="anch-details-about-vnet-rules-38q" />

## <a name="details-about-virtual-network-rules"></a>Details zu Regeln für ein virtuelles Netzwerk

In diesem Abschnitt werden verschiedene Details zu Regeln für ein virtuelles Netzwerk beschrieben.

#### <a name="only-one-geographic-region"></a>Nur eine geografische Region

Jeder Dienstendpunkt eines virtuellen Netzwerks gehört nur zu einer Azure-Region. Der Endpunkt ermöglicht anderen Regionen nicht das Akzeptieren von Nachrichten aus dem Subnetz.

Eine Regel für ein virtuelles Netzwerk ist auf die Region beschränkt, zu der der zugrunde liegende Endpunkt gehört.

#### <a name="server-level-not-database-level"></a>Auf Serverebene, nicht auf Datenbankebene

Jede Regel für ein virtuelles Netzwerk gilt für den gesamten Azure SQL-Datenbankserver und nicht nur für eine bestimmte Datenbank auf dem Server. Das heißt, dass Regeln für ein virtuelles Netzwerk auf Server- und nicht auf Datenbankebene gelten.

- Im Gegensatz dazu können IP-Regeln auf beiden Ebenen gelten.

#### <a name="security-administration-roles"></a>Sicherheitsverwaltungsrollen

Bei der Verwaltung der Dienstendpunkte des virtuellen Netzwerks erfolgt eine Trennung von Sicherheitsrollen. Die folgenden Rollen müssen Aktionen ausführen:

- **Netzwerkadministrator:**&nbsp; Aktivieren des Endpunkts.
- **Datenbankadministrator:**&nbsp; Aktualisieren der Zugriffssteuerungsliste durch Hinzufügen des angegebenen Subnetzes zum Azure SQL-Datenbankserver.

*Alternative zur rollenbasierten Zugriffssteuerung:*

Die Rollen „Netzwerkadministrator“ und „Datenbankadministrator“ haben mehr Zugriffsrechte, als für die Verwaltung von Regeln für ein virtuelles Netzwerk erforderlich ist. Es wird nur eine Teilmenge der Zugriffsrechte benötigt.

Sie können mit der [rollenbasierten Zugriffssteuerung (RBAC)][rbac-what-is-813s] in Azure arbeiten, um eine einzelne benutzerdefinierte Sicherheitsrolle zu erstellen, die nur über die benötigte Teilmenge von Zugriffsrechten verfügt. Die benutzerdefinierte Rolle kann definiert werden, anstatt den Netzwerk- oder Datenbankadministrator einzubeziehen. Die auf die Sicherheit bezogene Angriffsfläche ist kleiner, wenn Sie einen Benutzer einer benutzerdefinierte Rolle hinzufügen und ihn nicht den beiden anderen wichtigen Administratorrollen hinzufügen.

> [!NOTE]
> In einigen Fällen befinden sich die Azure SQL-Datenbank und das VNET-Subnetz in unterschiedlichen Abonnements. In diesen Fällen müssen Sie folgende Konfigurationen sicherstellen:
> - Beide Abonnements müssen demselben Azure Active Directory-Mandanten zugeordnet sein.
> - Der Benutzer muss über die erforderlichen Berechtigungen zum Initiieren der Vorgänge verfügen. Dazu gehören z.B. das Aktivieren von Dienstendpunkten und das Hinzufügen eines VNET-Subnetzes auf dem angegebenen Server.

## <a name="limitations"></a>Einschränkungen

Bei Azure SQL-Datenbank gelten für Regeln für ein virtuelles Netzwerk folgende Einschränkungen:

- Derzeit funktionieren Azure-Web-Apps in Subnetzen mit aktivierten **Dienstendpunkten** noch nicht wie erwartet. Wir arbeiten an der Bereitstellung dieser Funktion.
    - Solange die Funktion noch nicht vollständig implementiert wurde, empfehlen wir, dass Sie Ihre Web-App auf ein anderes Subnetz verschieben, für das keine Dienstendpunkte für SQL aktiviert sind.

- In der Firewall für Ihre SQL-Datenbank verweist jede Regel für ein virtuelles Netzwerk auf ein Subnetz. Alle Subnetze, auf die verwiesen wird, müssen in derselben geografischen Region gehostet werden, in der die SQL-Datenbank gehostet wird.

- Für jeden Azure SQL-Datenbankserver können für ein angegebenes virtuelles Netzwerk maximal 128 Einträge in der Zugriffssteuerungsliste vorhanden sein.

- Regeln für ein virtuelles Netzwerk gelten nur für virtuelle Netzwerke gemäß dem Azure Resource Manager-Modell und nicht gemäß dem [klassischen Bereitstellungsmodell][arm-deployment-model-568f].

- Durch das Aktivieren virtueller Netzwerkdienst-Endpunkte für Azure SQL-Datenbank werden auch die Endpunkte für die Azure-Dienste MySQL und Postgres aktiviert. Jedoch schlagen Verbindungen von den Endpunkten zu den MySQL- oder Postgres-Instanzen fehl, wenn Endpunkte aktiviert sind.
    - Der Grund dafür ist, dass ACLing derzeit von MySQL und Postgres nicht unterstützt wird.

- In der Firewall gelten zwar IP-Adressbereiche für die folgenden Netzwerkelemente, Regeln für virtuelle Netzwerke jedoch nicht:
    - [Virtuelles privates Netzwerk zwischen Standorten][vpn-gateway-indexmd-608y]
    - Lokal über [ExpressRoute][expressroute-indexmd-744v]

#### <a name="expressroute"></a>ExpressRoute

Wenn Ihr Netzwerk über [ExpressRoute][expressroute-indexmd-744v] mit Ihrem Azure-Netzwerk verbunden ist, wird jede Verbindung mit zwei öffentlichen IP-Adressen im Microsoft-Edgebereich konfiguriert. Die beiden IP-Adressen werden zum Herstellen der Verbindung mit Microsoft-Diensten, z.B. Azure Storage, mithilfe von öffentlichem Azure-Peering verwendet.

Um die Kommunikation von Ihrer Verbindung mit Azure SQL-Datenbank zu ermöglichen, müssen Sie IP-Adressregeln für die öffentlichen IP-Adressen Ihrer Verbindungen erstellen. Öffnen Sie über das Azure-Portal ein Supportticket für ExpressRoute, um die öffentlichen IP-Adressen Ihrer ExpressRoute-Verbindung zu ermitteln.


<!--
FYI: Re ARM, 'Azure Service Management (ASM)' was the old name of 'classic deployment model'.
When searching for blogs about ASM, you probably need to use this old and now-forbidden name.
-->

## <a name="impact-of-removing-allow-all-azure-services"></a>Auswirkungen des Entfernens von „Alle Azure-Dienste zulassen“

Viele Benutzer möchten **Alle Azure-Dienste zulassen** von ihren Azure SQL-Servern entfernen und durch eine VNET-Firewallregel ersetzen.
Das Entfernen wirkt sich jedoch auf die folgenden Azure-SQLDB-Features aus:

#### <a name="import-export-service"></a>Import/Export-Dienst
Der Dienst Azure SQLDB-Import/Export wird auf virtuellen Computern in Azure ausgeführt. Diese virtuellen Computer befinden sich nicht in Ihrem VNET und erhalten daher beim Verbinden mit Ihrer Datenbank eine Azure-IP-Adresse. Beim Entfernen von **Alle Azure-Dienste zulassen** können diese virtuellen Computer nicht mehr auf Ihre Datenbanken zugreifen.
Sie können das Problem umgehen. Führen Sie den BACPAC-Import oder -Export mithilfe der mithilfe der DACFx-API direkt im Code aus. Stellen Sie sicher, dass die Bereitstellung auf einem virtuellen Computer in dem VNET-Subnetz erfolgt, für das Sie die Firewallregel festgelegt haben.

#### <a name="sql-database-query-editor"></a>Abfrage-Editor für SQL-Datenbank
Der Abfrage-Editor für Azure SQL-Datenbank wird auf virtuellen Computern in Azure bereitgestellt. Diese virtuellen Computer befinden sich nicht in Ihrem VNET. Aus diesem Grund erhalten die virtuellen Computer beim Verbinden mit Ihrer Datenbank eine Azure-IP-Adresse. Beim Entfernen von **Alle Azure-Dienste zulassen** können diese virtuellen Computer nicht mehr auf Ihre Datenbanken zugreifen.

#### <a name="table-auditing"></a>Tabellenüberwachung
Derzeit stehen Ihnen zwei Möglichkeiten zum Aktivieren der Überwachung für Ihre SQL-Datenbank-Instanz zur Verfügung. Die Tabellenüberwachung führt zu einem Fehler, nachdem Sie Dienstendpunkte in Ihrer Azure SQL Server-Instanz aktiviert haben. Um dieses Problem zu umgehen, wechseln Sie zur Blobüberwachung.


## <a name="impact-of-using-vnet-service-endpoints-with-azure-storage"></a>Auswirkungen der Verwendung von VNET-Dienstendpunkten mit Azure Storage

In Azure Storage ist dasselbe Feature implementiert, mit dem Sie die Konnektivität zu Ihrem Speicherkonto beschränken können.
Wenn Sie dieses Feature mit einem Speicherkonto verwenden, das von einer Azure SQL Server-Instanz verwendet wird, können Probleme auftreten. Im Folgenden finden Sie eine Liste mit Erläuterungen der Azure SQLDB-Features, die davon betroffen sind.

#### <a name="azure-sqldw-polybase"></a>Azure SQLDW PolyBase
PolyBase wird häufig verwendet, um Daten aus Speicherkonten in Azure SQLDW zu laden. Wenn das Speicherkonto, aus dem Sie Daten laden, den Zugriff auf einen Satz von VNET-Subnetzen beschränkt, wird die Konnektivität zwischen PolyBase und dem Konto unterbrochen.

#### <a name="azure-sqldb-blob-auditing"></a>Azure SQLDB-Blobüberwachung
Bei der Blobüberwachung werden die Überwachungsprotokolle in Ihr eigenes Speicherkonto gepusht. Wenn dieses Speicherkonto das Feature für VNET-Dienstendpunkte verwendet, wird die Konnektivität zwischen Azure SQLDB und dem Speicherkonto unterbrochen.


## <a name="errors-40914-and-40615"></a>Fehler 40914 und 40615

Der Verbindungsfehler 40914 bezieht sich auf *Regeln für ein virtuelles Netzwerk*, die im Azure-Portal im Bereich „Firewall“ angegeben werden. Beim Fehler 40615 verhält es sich ähnlich, nur dass sich dieser Fehler auf *IP-Adressregeln* in der Firewall bezieht.

#### <a name="error-40914"></a>Fehler 40914

*Meldungstext:* Der bei der Anmeldung angeforderte Server „*[Servername]*“ kann nicht geöffnet werden. Der Client hat keine Zugriffsberechtigung für den Server.

*Fehlerbeschreibung:* Der Client befindet sich in einem Subnetz, das Endpunkte des virtuellen Netzwerkservers enthält. Der Azure SQL-Datenbankserver enthält jedoch keine Regeln für ein virtuelles Netzwerk, die dem Subnetz die Berechtigung zur Kommunikation mit der SQL-Datenbank gewähren.

*Fehlerbehebung:* Verwenden Sie im Azure-Portal im Bereich „Firewall“ die Steuerung von Regel für ein virtuelles Netzwerk, um [eine Regel für ein virtuelles Netzwerk](#anchor-how-to-by-using-firewall-portal-59j) für das Subnetz hinzuzufügen.

#### <a name="error-40615"></a>Fehler 40615

*Meldungstext:* Der bei der Anmeldung angeforderte Server „{0}“ kann nicht geöffnet werden. Der Client mit der IP-Adresse „{1}“ hat keine Zugriffsberechtigung für den Server.

*Fehlerbeschreibung:* Der Client versucht, eine Verbindung über eine IP-Adresse herzustellen, die nicht zum Herstellen einer Verbindung mit dem Azure SQL-Datenbankserver autorisiert ist. Die Serverfirewall enthält keine IP-Adressregel, die einem Client die Kommunikation mit der SQL-Datenbank über eine bestimmte IP-Adresse erlaubt.

*Fehlerbehebung:* Geben Sie als IP-Regel die IP-Adresse des Clients ein. Tun Sie dies im Bereich „Firewall“ des Azure-Portals.


Eine Liste verschiedener Fehlermeldungen der SQL-Datenbank ist [hier][sql-database-develop-error-messages-419g] dokumentiert.





<a name="anchor-how-to-by-using-firewall-portal-59j" />

## <a name="portal-can-create-a-virtual-network-rule"></a>Erstellen einer Regel für ein virtuelles Netzwerk über das Portal

In diesem Abschnitt wird veranschaulicht, wie Sie im [Azure-Portal][http-azure-portal-link-ref-477t] eine *Regel für ein virtuelles Netzwerk* in Ihrer Azure SQL-Datenbank-Instanz erstellen. Die Regel weist Ihre SQL-Datenbank-Instanz an, Nachrichten von einem bestimmten Subnetz zu akzeptieren, das als *Dienstendpunkt des virtuellen Netzwerks* gekennzeichnet ist.

#### <a name="powershell-alternative"></a>PowerShell-Alternative

Regeln für ein virtuelles Netzwerk können auch mit einem PowerShell-Skript erstellt werden. Dazu dient das Cmdlet **New-AzureRmSqlServerVirtualNetworkRule**. Lesen Sie bei Interesse [Verwenden von PowerShell zum Erstellen eines Endpunkts und einer Regel für den virtuellen Dienst für Azure SQL-Datenbank][sql-db-vnet-service-endpoint-rule-powershell-md-52d].

#### <a name="prerequisites"></a>Voraussetzungen

Falls Sie bereits ein Subnetz haben, das mit dem bestimmten Dienstendpunkt für das virtuelle Netzwerk gekennzeichnet ist, *geben Sie den Namen ein*, der zur Azure SQL-Datenbank-Instanz gehört.

- Der Typname dieses Endpunkts ist **Microsoft.Sql**.
- Wenn Ihr Subnetz nicht mit dem Typnamen gekennzeichnet werden kann, siehe [Überprüfen, dass Ihr Subnetz ein Endpunkt ist][sql-db-vnet-service-endpoint-rule-powershell-md-a-verify-subnet-is-endpoint-ps-100].

<a name="a-portal-steps-for-vnet-rule-200" />

### <a name="azure-portal-steps"></a>Schritte im Azure-Portal

1. Melden Sie sich beim [Azure-Portal][http-azure-portal-link-ref-477t] an.

2. Navigieren Sie dann im Portal zu **Computer mit SQL Server**&gt;**Firewall/Virtuelle Netzwerke**.

3. Legen Sie das Steuerelement **Zugriff auf Azure-Dienste erlauben** auf AUS fest.

    > [!IMPORTANT]
    > Wenn Sie das Steuerelement auf ON festgelegt lassen, akzeptiert der Azure SQL-Datenbank-Server Kommunikation von beliebigen Subnetzen. Das Steuerelement auf ON festgelegt zu lassen, führt also möglicherweise aus Sicht der Sicherheit zu einem übermäßigen Zugriff. Mithilfe des Microsoft Azure Virtual Network-Dienstendpunkts und Regeln für ein virtuelles Netzwerk von SQL-Datenbank können Sie die sicherheitsbezogene Angriffsfläche verkleinern.

4. Klicken Sie im Abschnitt **Virtuelle Netzwerke** auf **+ Vorhandene hinzufügen**.

    ![Klicken Sie auf „Vorhandene hinzufügen“ (Subnetzendpunkt, als SQL-Regel).][image-portal-firewall-vnet-add-existing-10-png]

5. Füllen Sie im Bereich **Erstellen/Aktualisieren** die Steuerelemente mit den Namen Ihrer Azure-Ressourcen aus.

    > [!TIP]
    > Sie müssen das richtige **Adresspräfix** für Ihr Subnetz hinzufügen. Den Wert finden Sie im Portal.
    > Navigieren Sie zu **Alle Ressourcen**&gt;**Alle Typen**&gt;**Virtuelle Netzwerke**. Der Filter zeigt Ihre virtuellen Netzwerke an. Klicken Sie auf das virtuelle Netzwerk und dann auf **Subnetze**. Die Spalte **ADRESSBEREICH** weist das Adresspräfix auf, das Sie benötigen.

    ![Füllen Sie Felder für die neue Regel aus.][image-portal-firewall-create-update-vnet-rule-20-png]

6. Klicken Sie am unteren Rand des Bereichs auf die Schaltfläche **OK**.

7.  Die resultierende Regel für ein virtuelles Netzwerk wird im Firewallbereich angezeigt.

    ![Sie finden die neue Regel im Firewallbereich.][image-portal-firewall-vnet-result-rule-30-png]


> [!NOTE]
> Für die Regeln gelten die folgenden Status oder Zustände:
> - **Bereit:** gibt an, dass der von Ihnen initiierte Vorgang erfolgreich war
> - **Fehlerhaft:** gibt an, dass der von Ihnen initiierte Vorgang zu einem Fehler geführt hat
> - **Gelöscht:** gilt nur für den Löschvorgang und gibt an, dass die Regel gelöscht wurde und nicht mehr angewendet wird
> - **InBearbeitung:** gibt an, dass der Vorgang ausgeführt wird. In diesem Zustand wird die alte Regel weiter angewendet.


<a name="anchor-how-to-links-60h" />

## <a name="related-articles"></a>Verwandte Artikel

- [Dienstendpunkte von virtuellen Netzwerken (Vorschauversion)][vm-virtual-network-service-endpoints-overview-649d]
- [Firewallregeln auf Server- und Datenbankebene für Azure SQL-Datenbank][sql-db-firewall-rules-config-715d]

Das Feature für Regeln für virtuelle Netzwerke für Azure SQL-Datenbank wurde Ende September 2017 eingeführt.

## <a name="next-steps"></a>Nächste Schritte

- [Verwenden von PowerShell zum Erstellen eines Dienstendpunkts und einer Regel für das virtuelle Netzwerk für Azure SQL-Datenbank][sql-db-vnet-service-endpoint-rule-powershell-md-52d]


<!-- Link references, to images. -->

[image-portal-firewall-vnet-add-existing-10-png]: media/sql-database-vnet-service-endpoint-rule-overview/portal-firewall-vnet-add-existing-10.png

[image-portal-firewall-create-update-vnet-rule-20-png]: media/sql-database-vnet-service-endpoint-rule-overview/portal-firewall-create-update-vnet-rule-20.png

[image-portal-firewall-vnet-result-rule-30-png]: media/sql-database-vnet-service-endpoint-rule-overview/portal-firewall-vnet-result-rule-30.png



<!-- Link references, to text, Within this same Github repo. -->

[arm-deployment-model-568f]: ../azure-resource-manager/resource-manager-deployment-model.md

[expressroute-indexmd-744v]: ../expressroute/index.md

[rbac-what-is-813s]: ../active-directory/role-based-access-control-what-is.md

[sql-db-firewall-rules-config-715d]: sql-database-firewall-configure.md

[sql-database-develop-error-messages-419g]: sql-database-develop-error-messages.md

[sql-db-vnet-service-endpoint-rule-powershell-md-52d]: sql-database-vnet-service-endpoint-rule-powershell.md

[sql-db-vnet-service-endpoint-rule-powershell-md-a-verify-subnet-is-endpoint-ps-100]: sql-database-vnet-service-endpoint-rule-powershell.md#a-verify-subnet-is-endpoint-ps-100

[vm-configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-portal-321w]: ../virtual-network/virtual-networks-static-private-ip-arm-pportal.md

[vm-virtual-network-service-endpoints-overview-649d]: https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview

[vpn-gateway-indexmd-608y]: ../vpn-gateway/index.md



<!-- Link references, to text, Outside this Github repo (HTTP). -->

[http-azure-portal-link-ref-477t]: https://portal.azure.com/




<!-- ??2
#### Syntax related articles
- REST API Reference, including JSON

- Azure CLI

- ARM templates
-->
