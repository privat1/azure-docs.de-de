---
title: "Schützen von Daten und Vorgängen in Azure Search | Microsoft-Dokumentation"
description: "Die Sicherheit von Azure Search basiert auf SOC 2-Konformität, Verschlüsselung, Authentifizierung und Identitätszugriff durch Benutzer- und Gruppensicherheits-IDs in Azure Search-Filtern."
services: search
documentationcenter: 
author: HeidiSteen
manager: jhubbard
editor: 
ms.assetid: 
ms.service: search
ms.devlang: 
ms.workload: search
ms.topic: article
ms.tgt_pltfrm: na
ms.date: 12/14/2017
ms.author: heidist
ms.openlocfilehash: 23616c70a5fd336b743f5acfad2601a6c3e23fc4
ms.sourcegitcommit: 357afe80eae48e14dffdd51224c863c898303449
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/15/2017
---
# <a name="data-security-and-controlled-access-to-azure-search-operations"></a>Datensicherheit und kontrollierter Zugriff auf Azure Search-Vorgänge

Azure Search ist [SOC 2-konform](https://servicetrust.microsoft.com/ViewPage/MSComplianceGuide?command=Download&downloadType=Document&downloadId=93292f19-f43e-4c4e-8615-c38ab953cf95&docTab=4ce99610-c9c0-11e7-8c2c-f908a777fa4d_SOC%20%2F%20SSAE%2016%20Reports) und verfügt über eine umfassende Sicherheitsarchitektur mit physischer Sicherheit, verschlüsselter Übertragung, verschlüsselter Speicherung und plattformweitem Softwareschutz. Im laufenden Betrieb akzeptiert Azure Search ausschließlich authentifizierte Anforderungen. Optional kann eine benutzerspezifische Zugriffssteuerung für Inhalte hinzugefügt werden. Dieser Artikel geht zwar auf die einzelnen Sicherheitsebenen ein, beschäftigt sich aber hauptsächlich mit dem Schutz von Daten und Vorgängen in Azure Search.

![Blockdiagramm der Sicherheitsebenen](media/search-security-overview/azsearch-security-diagram.png)

Azure Search liegen zwar die Sicherheitsmaßnahmen und Absicherungen der Azure-Plattform zugrunde, der primäre Mechanismus des Diensts ist jedoch die schlüsselbasierte Authentifizierung, wobei die Art des Schlüssels über die Zugriffsebene entscheidet. Bei dem Schlüssel handelt es sich entweder um einen Administratorschlüssel oder um einen Abfrageschlüssel für schreibgeschützten Zugriff.

Der Zugriff auf Ihren Dienst basiert auf einem Querschnitt von Berechtigungen, die durch den Schlüssel angegeben werden (Vollzugriff oder schreibgeschützter Zugriff), sowie auf einem Kontext, der einen Vorgangsbereich definiert. Jede Anforderung setzt sich aus einem obligatorischen Schlüssel, einem Vorgang und einem Objekt zusammen. In Kombination lässt sich durch die beiden Berechtigungsstufen und den Kontext ein vollumfängliches Sicherheitsspektrum für Dienstvorgänge erreichen. 

## <a name="physical-security"></a>Physische Sicherheit

Microsoft-Rechenzentren sind mit branchenführenden physischen Sicherheitsmaßnahmen ausgestattet und erfüllen eine breite Palette von Standards und Bestimmungen. Weitere Informationen finden Sie auf der Seite [Global Datacenters](https://www.microsoft.com/cloud-platform/global-datacenters) sowie im folgenden kurzen Video zur Sicherheit von Rechenzentren:

> [!VIDEO https://www.youtube.com/embed/r1cyTL8JqRg]

## <a name="encrypted-transmission-and-storage"></a>Verschlüsselte Übertragung und Speicherung

Azure Search lauscht an HTTPS-Port 443. Verbindungen mit Azure-Diensten sind plattformweit verschlüsselt. 

Im für Indizes und andere Konstrukte verwendeten Back-End-Speicher nutzt Azure Search die Verschlüsselungsfunktionen dieser Plattformen. In allen Rechenzentren, die Azure Search anbieten, steht für alle Suchdienste uneingeschränkte [AICPA SOC 2-Konformität](https://www.aicpa.org/interestareas/frc/assuranceadvisoryservices/aicpasoc2report.html) zur Verfügung. Den vollständigen Bericht finden Sie unter [Azure - and Azure Government SOC 2 Type II Report](https://servicetrust.microsoft.com/ViewPage/MSComplianceGuide?command=Download&downloadType=Document&downloadId=93292f19-f43e-4c4e-8615-c38ab953cf95&docTab=4ce99610-c9c0-11e7-8c2c-f908a777fa4d_SOC%20%2F%20SSAE%2016%20Reports) (SOC 2Type II-Bericht für Azure und Azure Government).

Im Zuge der transparenten Verschlüsselung werden Verschlüsselungsschlüssel intern verwaltet und universell angewendet. Sie können weder die Verschlüsselung für bestimmte Suchdienste oder -indizes deaktivieren noch Schlüssel direkt verwalten oder eigene Schlüssel verwenden. 

## <a name="azure-wide-logical-security"></a>Azure-weite logische Sicherheit

Für die gesamte Azure Stack-Umgebung sind verschiedene Sicherheitsmechanismen verfügbar – und dadurch automatisch auch für die von Ihnen erstellten Azure Search-Ressourcen.

+ [Sperren auf Abonnement- oder Ressourcenebene zur Verhinderung von Löschvorgängen](../azure-resource-manager/resource-group-lock-resources.md)
+ [Rollenbasierte Zugriffssteuerung (Role-based Access Control, RBAC) zur Steuerung des Zugriffs auf Informationen und Verwaltungsvorgänge](../active-directory/role-based-access-control-what-is.md)

Alle Azure-Dienste unterstützen die rollenbasierte Zugriffssteuerung (Role-based Access Control, RBAC) zum einheitlichen Festlegen von Zugriffsebenen für alle Dienste. So ist beispielsweise das Anzeigen sensibler Daten (etwa des Administratorschlüssels) auf die Rollen „Besitzer“ und „Mitwirkender“ beschränkt, der Dienststatus kann dagegen von Mitgliedern aller Rollen angezeigt werden. RBAC bietet die Rollen „Besitzer“, „Mitwirkenden“ und „Leser“. Administratoren sind standardmäßig Mitglieder der Rolle "Besitzer".

## <a name="service-authentication"></a>Dienstauthentifizierung

Azure Search verfügt über eine eigene Authentifizierungsmethode. Die Authentifizierung erfolgt bei jeder Anforderung und basiert auf einem Zugriffsschlüssel, der über den Umfang der Vorgänge entscheidet. Ein gültiger Zugriffsschlüssel gilt als Beleg dafür, dass die Anforderung von einer vertrauenswürdigen Entität stammt. 

Die dienstspezifische Authentifizierung ist auf zwei Ebenen verfügbar: Vollzugriff und reiner Abfragezugriff. Die Art des Schlüssels entscheidet, welche Zugriffsebene zur Anwendung kommt.

|Schlüssel|Beschreibung|Einschränkungen|  
|---------|-----------------|------------|  
|Administrator|Gewährt Vollzugriff auf alle Vorgänge. Dazu zählen die Dienstverwaltung sowie das Erstellen und Löschen von **Indizes**, **Indexern** und **Datenquellen**.<br /><br /> Für Administratoren werden zwei **API-Schlüssel** generiert, wenn der Dienst erstellt wird. Diese werden im Portal als *primärer* bzw. *sekundärer* Schlüssel bezeichnet und können bei Bedarf einzeln neu generiert werden. Durch die Verwendung von zwei Schlüsseln kann weiterhin auf den Dienst zugegriffen werden, während für einen der Schlüssel ein Rollover durchgeführt wird.<br /><br /> Administratorschlüssel werden nur in HTTP-Anforderungsheadern angegeben. Ein **API-Schlüssel** für Administratoren kann nicht in einer URL platziert werden.|Maximal zwei pro Dienst|  
|Abfrage|Gewährt Lesezugriff auf Indizes und Dokumente. Diese werden in der Regel auf Clientanwendungen verteilt, die Suchanfragen ausgeben.<br /><br /> Abfrageschlüssel werden bei Bedarf erstellt. Sie können manuell über das Portal oder programmgesteuert über die [Verwaltungs-REST-API](https://docs.microsoft.com/rest/api/searchmanagement/) erstellt werden.<br /><br /> Abfrageschlüssel können in einem HTTP-Anforderungsheader für Such-, Vorschlags- oder Recherchevorgänge angegeben werden. Sie können einen Abfrageschlüssel aber auch als Parameter in einer URL übergeben. Je nachdem, wie Ihre Clientanwendung die Anforderung formuliert, ist es unter Umständen einfacher, den Schlüssel als Abfrageparameter zu übergeben:<br /><br /> `GET /indexes/hotels/docs?search=*&$orderby=lastRenovationDate desc&api-version=2016-09-01&api-key=A8DA81E03F809FE166ADDB183E9ED84D`|50 pro Dienst|  

 Administrator- und Abfrageschlüssel sind rein optisch nicht zu unterscheiden. Bei beiden Schlüsseln handelt es sich um eine Zeichenfolge mit 32 nach dem Zufallsprinzip generierten alphanumerischen Zeichen. Sollten Sie nicht mehr wissen, welche Art von Schlüssel in Ihrer Anwendung angegeben ist, können Sie [die Schlüsselwerte im Portal überprüfen](https://portal.azure.com) oder über die [REST-API](https://docs.microsoft.com/rest/api/searchmanagement/) den Wert und die Art des Schlüssels zurückgeben.  

> [!NOTE]  
>  Aus Sicherheitsgründen sollten im Anforderungs-URI keine vertraulichen Daten wie etwa ein API-Schlüssel (`api-key`) übergeben werden. Aus diesem Grund akzeptiert Azure Search als `api-key` in der Abfragezeichenfolge nur einen Abfrageschlüssel. Es empfiehlt sich, diese Richtlinie einzuhalten – es sei denn, der Inhalt Ihres Index soll öffentlich verfügbar sein. Wir empfehlen generell, `api-key` als Anforderungsheader zu übergeben.  

### <a name="how-to-find-the-access-keys-for-your-service"></a>Ermitteln der Zugriffsschlüssel für Ihren Dienst

Zugriffsschlüssel stehen über das Portal oder über die [Verwaltungs-REST-API](https://docs.microsoft.com/rest/api/searchmanagement/) zur Verfügung. Weitere Informationen finden Sie unter [Verwalten von API-Schlüsseln](search-manage.md#manage-api-keys).

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com) an.
2. Listen Sie die [Suchdienste](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) für Ihr Abonnement auf.
3. Wählen Sie den Dienst aus, und suchen Sie auf der Dienstseite nach **Einstellungen** >**Schlüssel**, um Administrator- und Abfrageschlüssel anzuzeigen.

![Portalseite > „Einstellungen“ > Abschnitt „Schlüssel“](media/search-security-overview/settings-keys.png)

## <a name="index-access"></a>Indexzugriff

In Azure Search ist ein einzelner Index kein sicherungsfähiges Objekt. Stattdessen wird der Zugriff auf einen Index durch eine Kombination aus Dienstebene (Lese- oder Schreibzugriff) und Vorgangskontext bestimmt.

Für den Endbenutzerzugriff können Sie Abfrageanforderungen in Ihrer Anwendung so strukturieren, dass die Verbindungsherstellung unter Verwendung eines Abfrageschlüssels erfolgt (wodurch alle Abfragen nur schreibgeschützten Zugriff erhalten), und den spezifischen, von Ihrer App verwendeten Index einschließen. Da es in einer Abfrageanforderung kein Konzept für die Verknüpfung von Indizes oder den gleichzeitigen Zugriff auf mehrere Indizes gibt, sind alle Anforderungen definitionsgemäß auf einen einzelnen Index ausgerichtet. Folglich wird die Sicherheitsgrenze durch die Struktur der Abfrage (Kombination aus Schlüssel und einzelnem Zielindex) definiert.

Der Indexzugriff von Administratoren und Entwicklern ist undifferenziert: Beide benötigen Schreibzugriff, um vom Dienst verwaltete Objekte erstellen, löschen und aktualisieren zu können. Ein Benutzer mit einem Administratorschlüssel für Ihren Dienst kann jeden beliebigen Index in diesem Dienst lesen, ändern und löschen. Zum Schutz vor der versehentlichen oder böswilligen Löschung von Indizes können Sie Ihre interne Quellcodeverwaltung für Coderessourcen verwenden und unerwünschte Lösch- oder Änderungsvorgänge für Indizes rückgängig machen. Zur Gewährleistung der Verfügbarkeit verfügt Azure Search über ein clusterinternes Failover-Feature. Ihr proprietärer Code zum Erstellen oder Laden von Indizes wird jedoch nicht gespeichert oder ausgeführt.

Mehrinstanzenfähige Lösungen, die Sicherheitsgrenzen auf der Indexebene erfordern, enthalten in der Regel eine mittlere Ebene, die Kunden für die Indexisolierung verwenden. Weitere Informationen zum mehrinstanzenfähigen Anwendungsfall finden Sie unter [Entwurfsmuster für mehrinstanzenfähige SaaS-Anwendungen und Azure Search](search-modeling-multitenant-saas-applications.md).

## <a name="admin-access-from-client-apps"></a>Administratorzugriff über Client-Apps

Die Verwaltungs-REST-API von Azure Search ist eine Erweiterung von Azure Resource Manager und verfügt über die gleichen Abhängigkeiten. Folglich wird für die Verwaltung des Azure Search-Diensts Active Directory vorausgesetzt. Jegliche aus Clientcode stammende Verwaltungsanforderung muss mithilfe von Azure Active Directory authentifiziert werden, bevor die Anforderung Resource Manager erreicht.

Bei Datenanforderungen für den Azure Search-Dienstendpunkt – beispielsweise „Index erstellen“ (REST-API des Azure Search-Diensts) oder „Dokumente suchen“ (REST-API des Azure Search-Diensts) – wird ein API-Schlüssel im Anforderungsheader verwendet.

Falls Ihr Anwendungscode sowohl Dienstverwaltungsvorgänge als auch Datenvorgänge für Suchindizes oder Dokumente umfasst, implementieren Sie darin zwei Authentifizierungsmethoden: den für Azure Search nativen Zugriffsschlüssel und die für Resource Manager benötigte Active Directory-Authentifizierungsmethode. 

Informationen zum Strukturieren einer Anforderung in Azure Search finden Sie unter [Azure Search Service REST](https://docs.microsoft.com/rest/api/searchservice/) (Azure Search-Dienst: REST). Weitere Informationen zu Authentifizierungsanforderungen für Resource Manager finden Sie unter [Verwenden der Resource Manager-Authentifizierungs-API für den Zugriff auf Abonnements](../azure-resource-manager/resource-manager-api-authentication.md).

## <a name="user-access-to-index-content"></a>Benutzerzugriff auf den Indexinhalt

Benutzerspezifischer Zugriff auf den Inhalt eines Index wird über Sicherheitsfilter für Ihre Abfragen implementiert, sodass mit einer bestimmten Sicherheitsidentität verknüpfte Dokumente zurückgegeben werden. Anstelle vordefinierter Rollen und Rollenzuweisungen wird eine identitätsbasierte Zugriffssteuerung als Filter implementiert, die die Suchergebnisse von Dokumenten und Inhalten auf der Grundlage von Identitäten einschränken. In der folgenden Tabelle werden zwei Ansätze zur Einschränkung von Suchergebnissen für nicht autorisierte Inhalte beschrieben.

| Vorgehensweise | Beschreibung |
|----------|-------------|
|[Einschränkung aus Sicherheitsgründen auf der Grundlage von Identitätsfiltern](search-security-trimming-for-azure-search.md)  | Beschreibt den grundlegenden Workflow für die Implementierung der Zugriffssteuerung auf der Grundlage der Benutzeridentität. Der Artikel behandelt das Hinzufügen von Sicherheitsbezeichnern zu einem Index sowie das Filtern nach dem entsprechenden Feld, um unzulässige Inhalte aus den Ergebnissen zu entfernen. |
|[Einschränkung aus Sicherheitsgründen auf der Grundlage von Azure Active Directory-Identitäten](search-security-trimming-for-azure-search-with-aad.md)  | Dieser Artikel baut auf dem vorherigen Artikel auf und enthält Schritte zum Abrufen von Identitäten aus Azure Active Directory (AAD) – einem der [kostenlosen Dienste](https://azure.microsoft.com/free/) der Azure-Cloudplattform. |

## <a name="table-permissioned-operations"></a>Tabelle: Berechtigungsbasierte Vorgänge

Die folgende Tabelle enthält eine Zusammenfassung der in Azure Search zulässigen Vorgänge sowie Informationen dazu, welcher Schlüssel für den jeweiligen Vorgang erforderlich ist.

| Vorgang | Berechtigungen |
|-----------|-------------------------|
| Erstellen von Diensten | Azure-Abonnementinhaber|
| Skalieren von Diensten | Administratorschlüssel, RBAC-Rolle „Besitzer“ oder „Mitwirkender“ für die Ressource  |
| Löschen von Diensten | Administratorschlüssel, RBAC-Rolle „Besitzer“ oder „Mitwirkender“ für die Ressource |
| Erstellen/Ändern/Löschen von Objekten für den Dienst: <br>Indizes und Komponententeile (einschließlich Analysedefinitionen, Bewertungsprofile und CORS-Optionen), Indexer, Datenquellen, Synonyme und Vorschläge. | Administratorschlüssel, RBAC-Rolle „Besitzer“ oder „Mitwirkender“ für die Ressource  |
| Abfragen von Indizes | Administrator- oder Abfrageschlüssel (RBAC nicht verfügbar) |
| Abfragen von Systeminformationen (etwa zum Zurückgeben von Statistiken, Anzahlen und Objektlisten) | Administratorschlüssel, RBAC-Rolle für die Ressource („Besitzer“, „Mitwirkender“ oder „Leser“) |
| Verwalten von Administratorschlüsseln | Administratorschlüssel, RBAC-Rolle „Besitzer“ oder „Mitwirkender“ für die Ressource |
| Verwalten von Abfrageschlüsseln |  Administratorschlüssel, RBAC-Rolle „Besitzer“ oder „Mitwirkender“ für die Ressource. Mit der RBAC-Rolle „Leser“ können Abfrageschlüssel angezeigt werden. |


## <a name="see-also"></a>Weitere Informationen

+ [Erste Schritte mit .NET (veranschaulicht das Erstellen eines Index unter Verwendung eines Administratorschlüssels)](search-create-index-dotnet.md)
+ [Erste Schritte mit REST (veranschaulicht das Erstellen eines Index unter Verwendung eines Administratorschlüssels)](search-create-index-rest-api.md)
+ [Identitätsbasierte Zugriffssteuerung mit Azure Search-Filtern](search-security-trimming-for-azure-search.md)
+ [Auf der Active Directory-Identität basierende Zugriffssteuerung mit Azure Search-Filtern](search-security-trimming-for-azure-search-with-aad.md)
+ [Filter in Azure Search](search-filters.md)