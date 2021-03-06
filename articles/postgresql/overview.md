---
title: "Übersicht über den relationalen Datenbankdienst Azure-Datenbank für PostgreSQL | Microsoft-Dokumentation"
description: "Bietet eine Übersicht über den relationalen Datenbankdienst Azure-Datenbank für PostgreSQL."
services: postgresql
author: kamathsun
ms.author: sukamat
manager: jhubbard
editor: jasonwhowell
ms.custom: mvc
ms.service: postgresql
ms.topic: overview
ms.date: 10/20/2017
ms.openlocfilehash: 9aa24dd10ef29c716c05cafeb84e0beb23d50628
ms.sourcegitcommit: 0930aabc3ede63240f60c2c61baa88ac6576c508
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/07/2017
---
# <a name="what-is-azure-database-for-postgresql"></a>Was ist Azure-Datenbank für PostgreSQL?

Azure-Datenbank für PostgreSQL ist ein relationaler Datenbankdienst in Microsoft Cloud für Entwickler, der auf der Communityversion des Open Source-[PostgreSQL](https://www.postgresql.org/)-Datenbankmoduls basiert. Dieser Dienst befindet sich in der öffentlichen Vorschau. Azure-Datenbank für PostgreSQL bietet Folgendes:

- Integrierte Hochverfügbarkeit ohne zusätzliche Kosten.
- Vorhersagbare Leistung inklusive nutzungsbasierter Bezahlung.
- Skalierung bei Bedarf innerhalb von Sekunden.
- Gesicherte Datenbank zum Schutz sensibler Daten in Ruhe und Bewegung.
- Automatische Sicherungen und Point-in-Time-Wiederherstellung für bis zu 35 Tage.
- Sicherheit und Konformität auf Unternehmensniveau.

All diese Funktionen erfordern fast keine Administration und werden alle ohne zusätzliche Kosten bereitgestellt. Mit diesen Funktionen können Sie sich auf die schnelle Entwicklung von Anwendungen und die Verkürzung des Zeitraums bis zur Markteinführung konzentrieren, anstatt wertvolle Zeit und Ressourcen für die Verwaltung von virtuellen Computern und der Infrastruktur aufwenden zu müssen. Darüber hinaus können Sie Ihre Anwendung weiterhin mit den Open Source-Tools und der Plattform Ihrer Wahl entwickeln und mit der Geschwindigkeit und Effizienz liefern, die Ihr Unternehmen verlangt, ohne dass Sie sich dafür neue Fähigkeiten aneignen müssen. 

Dieser Artikel bietet eine Einführung in die wesentlichen Konzepte und Features von Azure-Datenbank für PostgreSQL im Hinblick auf Leistung, Skalierbarkeit und Verwaltbarkeit. Diese Schnellstarts erleichtern Ihnen den Einstieg:

- [Erstellen einer Azure-Datenbank für PostgreSQL mithilfe des Azure-Portals](quickstart-create-server-database-portal.md)
- [Erstellen einer Azure-Datenbank für PostgreSQL mithilfe der Azure-CLI](quickstart-create-server-database-azure-cli.md)

Eine Reihe von Azure CLI-Beispielen finden Sie unter:

- [Azure CLI-Beispiele für Azure-Datenbank für PostgreSQL](./sample-scripts-azure-cli.md)

## <a name="adjust-performance-and-scale-within-seconds"></a>Anpassen von Leistung und Skalierung in nur wenigen Sekunden
Für den Azure Database for PostgreSQL-Dienst gibt es in der Vorschauversion zwei Diensttarife: „Basic“ und „Standard“. Jeder Tarif bietet verschiedene Leistungsstufen und Funktionen zur Unterstützung geringer und hoher Datenbankworkloads. Sie können Ihre erste App mit einer kleinen Datenbank für wenige USD im Monat erstellen und die Skalierung dann so ändern, dass sie den Anforderungen Ihrer Lösung entspricht. Durch die dynamische Skalierung kann Ihre Datenbank transparent auf schnell wechselnde Ressourcenanforderungen reagieren. Sie bezahlen nur für die benötigten Ressourcen und auch nur dann, wenn Sie sie verwenden. Einzelheiten hierzu finden Sie unter [Tarife](concepts-service-tiers.md).

## <a name="monitoring-and-alerting"></a>Überwachung und Warnung
Wie entscheiden Sie, wann Sie nach oben oder nach unten skalieren? Verwenden Sie die integrierten Features für Leistungsüberwachung und Warnungen in Kombination mit den Leistungsbewertungen nach Compute-Einheit. Mit diesen Tools können Sie die Auswirkungen des zentralen Hoch- oder Herunterskalierens von Compute-Einheiten je nach Ihren derzeitigen oder erwarteten Leistungsanforderungen schnell bewerten. Einzelheiten hierzu finden Sie unter [Warnungen](howto-alert-on-metric.md).

## <a name="keep-your-app-and-business-running"></a>Aufrechterhalten von App-Ausführung und Geschäftskontinuität
Durch die Unterstützung eines globalen Netzwerks von Microsoft-verwalteten Rechenzentren stellt die in der Branche führende Verfügbarkeit von Azure mit einer Vereinbarung zum Servicelevel (SLA) von 99,99% (in der Vorschau nicht verfügbar) sicher, dass Ihre Apps rund um die Uhr ausgeführt werden. Mit jedem Azure-Datenbank für PostgreSQL-Server nutzen Sie die integrierte Sicherheit, die Fehlertoleranz und die Datensicherheit, die Sie andernfalls entwerfen oder kaufen, erstellen und verwalten müssten. In Azure-Datenbank für PostgreSQL bietet jeder Diensttarif einen umfassenden Satz von Features für Geschäftskontinuität und Optionen, die sicherstellen, dass Sie den Betrieb aufnehmen können und dieser nicht unterbrochen wird. Über die [Point-in-Time-Wiederherstellung](howto-restore-server-portal.md) können Sie den früheren Status einer Datenbank für bis zu 35 Tage wiederherstellen. Ferner können Sie bei einem Ausfall des Rechenzentrums, in dem Ihre Datenbanken gehostet werden, Ihre Datenbanken aus georedundanten Kopien kürzlich durchgeführter Sicherungen wiederherstellen.

## <a name="secure-your-data"></a>Sichern der Daten
Azure-Datenbank für PostgreSQL führt die Tradition der Datensicherheit von Azure-Datenbankdiensten fort – mit Features zum Einschränken des Zugriffs, zum Schützen ruhender und bewegter Daten und zum Überwachen von Aktivitäten. Informationen zur Sicherheit der Azure-Plattform finden Sie im [Azure Trust Center](https://www.microsoft.com/TrustCenter/Security/default.aspx).

Die Azure-Datenbank für PostgreSQL-Dienst verwendet Speicherverschlüsselung für ruhende Daten. Daten, einschließlich von Sicherungen, werden auf dem Datenträger verschlüsselt (mit Ausnahme von temporären Dateien, die vom Modul während der Ausführung von Abfragen erstellt werden). Der Dienst verwendet das in der Azure-Speicherverschlüsselung inbegriffene AES-256-Bit-Verschlüsselungsverfahren und die Schlüssel werden vom System verwaltet. Die Speicherverschlüsselung ist immer aktiviert und kann nicht deaktiviert werden.

Standardmäßig ist der Azure-Datenbank für PostgreSQL-Dienst so konfiguriert, dass [SSL-Verbindungssicherheit](./concepts-ssl-connection-security.md) für Daten während Übertragungen innerhalb des Netzwerks erforderlich ist. Das Erzwingen von SSL-Verbindungen zwischen dem Datenbankserver und Clientanwendungen trägt zum Schutz vor Man-in-the-Middle-Angriffen bei, indem der Datenstrom zwischen dem Server und der Anwendung verschlüsselt wird.  Wenn Ihre Clientanwendung keine SSL-Konnektivität unterstützt, können Sie optional die Erzwingung von SSL zum Herstellen einer Verbindung mit dem Datenbankdienst deaktivieren.

## <a name="next-steps"></a>Nächste Schritte
- Auf der Seite mit der [Preisgestaltung](https://azure.microsoft.com/pricing/details/postgresql/) finden Sie Kostenvergleiche und Rechner.
- Beginnen Sie mit dem [Erstellen einer ersten Azure-Datenbank für PostgreSQL](./quickstart-create-server-database-portal.md).
- Erstellen Ihrer ersten App in Python, PHP, Ruby, C\#, Java oder Node.js: [Verbindungsbibliotheken](./concepts-connection-libraries.md)
