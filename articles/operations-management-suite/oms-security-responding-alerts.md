---
title: "Überwachen von und Reagieren auf Sicherheitswarnungen in der Sicherheits- und Überwachungslösung der Operations Management Suite | Microsoft Docs"
description: "Dieses Dokument unterstützt Sie bei der Verwendung der Option Threat Intelligence in der Sicherheits- und Überwachungslösung von OMS für das Überwachen von und Reagieren auf Sicherheitswarnungen."
services: operations-management-suite
documentationcenter: na
author: YuriDio
manager: mbaldwin
editor: 
ms.assetid: 7d45a32b-1341-4bb5-a436-1f42a8a2590a
ms.service: operations-management-suite
ms.custom: oms-security
ms.topic: article
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/13/2017
ms.author: yurid
ms.openlocfilehash: df82afab2c38431e134146143524edc080ee38f9
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 10/11/2017
---
# <a name="monitoring-and-responding-to-security-alerts-in-operations-management-suite-security-and-audit-solution"></a>Überwachen von und Reagieren auf Sicherheitswarnungen in der Sicherheits- und Überwachungslösung von Operations Management Suite
Dieses Dokument unterstützt Sie bei der Verwendung der Option Threat Intelligence in der Sicherheits- und Überwachungslösung von OMS für das Überwachen von und Reagieren auf Sicherheitswarnungen.

## <a name="what-is-oms"></a>Was ist OMS?
Microsoft Operations Management Suite (OMS) ist die cloudbasierte IT-Verwaltungslösung von Microsoft, die Ihnen das Verwalten und Schützen Ihrer lokalen und Cloudinfrastruktur erleichtert. Weitere Informationen zu OMS finden Sie im Artikel [Operations Management Suite](https://technet.microsoft.com/library/mt484091.aspx).

## <a name="threat-intelligence"></a>Threat Intelligence
In einer Unternehmensumgebung mit umfassenden Zugriff der Benutzer auf das Netzwerk und der Verwendung vieler Geräte für Verbindungen mit Unternehmensdaten ist es zwingend erforderlich, dass Sie die Ressourcen aktiv überwachen und schnell auf Sicherheitsvorfälle reagieren können. Dies ist insbesondere wichtig aus der Perspektive des Sicherheitslebenszyklus, da einige Bedrohungen der Cybersicherheit möglicherweise keine Warnungen auslösen oder verdächtige Aktivitäten, die durch herkömmliche technische Sicherheitskontrollen identifiziert werden können. 

Mithilfe der Option **Informationen zu Bedrohungen** in der Sicherheits- und Überwachungslösung von OMS können IT-Administratoren Sicherheitsrisiken für die Umgebung erkennen. So können sie z.B. ermitteln, ob ein bestimmter Computer Teil von einem [Botnet](https://www.microsoft.com/security/sir/story/default.aspx#!botnetsection) ist. Computer können Knoten in einem Botnet werden, wenn Angreifer illegal Schadsoftware installieren, die diesen Computer heimlich mit einem Steuerknoten verbindet. Die Option ermöglicht auch die Erkennung potenzieller Bedrohungen aus verdeckten Kommunikationskanälen wie dem [Darknet](https://www.microsoft.com/security/sir/story/default.aspx#!botnetsection_honeypots_darkents). 

Zum Erstellen von Threat Intelligence verwendet die Sicherheits- und Überwachungslösung von OMS Daten aus mehreren Quellen bei Microsoft. Die Sicherheits- und Überwachungslösung von OMS nutzt diese Daten zum Erkennen potenzieller Bedrohungen für Ihre Umgebung.

Der Bereich Threat Intelligence umfasst drei wichtige Optionen:

* Server mit ausgehendem böswilligem Datenverkehr
* Typen der erkannten Bedrohungen
* Threat Intelligence-Karte

> [!NOTE]
> Eine Übersicht über alle Optionen finden Sie unter [Erste Schritte mit der Sicherheits- und Überwachungslösung von Operations Management Suite](oms-security-getting-started.md).
> 
> 

### <a name="responding-to-security-alerts"></a>Reagieren auf Sicherheitswarnungen
Einer der Schritte bei der [Reaktion auf Sicherheitsvorfälle](https://technet.microsoft.com/library/cc512623.aspx) ist die Ermittlung des Schweregrads der Gefährdung für die Systeme. In dieser Phase sollten Sie die folgenden Aufgaben durchführen:

* Bestimmen der Art des Angriffs
* Bestimmen des ersten Angriffspunkts
* Bestimmen der Absicht des Angriffs. Ist der Angriff speziell auf Ihre Organisation ausgerichtet, um bestimmte Informationen zu erlangen, oder erfolgte er zufällig?
* Ermitteln der betroffenen Systeme
* Identifizieren der Dateien, auf die zugegriffen wurde, und Bestimmen der Vertraulichkeit dieser Dateien

Zur Unterstützung dieser Aufgaben können Sie die Informationen aus **Threat Intelligence** in der Sicherheits- und Überwachungslösung von OMS nutzen. Führen Sie die folgenden Schritte aus, um auf diese Optionen von **Threat Intelligence** zuzugreifen:

1. Klicken Sie im Hauptdashboard der **Microsoft Operations Management Suite** auf die Kachel **Sicherheit und Überwachung**.
   
    ![Security and Audit](./media/oms-security-responding-alerts/oms-security-responding-alerts-fig1.png)
2. Im Dashboard **Sicherheit und Überwachung** finden Sie die Optionen von **Informationen zu Bedrohungen** auf der rechten Seite, wie unten dargestellt:
   
    ![Threat Intelligence](./media/oms-security-responding-alerts/oms-security-responding-alerts-fig2-ga.png)

Über diese drei Kacheln erhalten Sie eine Übersicht über die aktuellen Bedrohungen. Unter **Server with outbound malicious traffic** (Server mit ausgehendem böswilligem Datenverkehr) können Sie ermitteln, ob ein von Ihnen überwachter Computer (innerhalb oder außerhalb des Netzwerks) böswilligen Datenverkehr an das Internet sendet. 

Die Kachel **Ermittelte Bedrohungstypen** zeigt eine Zusammenfassung der Bedrohungen an, die derzeit „in der Praxis“ vorhanden sind. Wenn Sie auf diese Kachel klicken, sehen Sie weitere Details zu diesen Bedrohungen, wie unten gezeigt:

![Arten der erkannten Bedrohungen](./media/oms-security-responding-alerts/oms-security-responding-alerts-fig3.png)

Sie erhalten weitere Informationen zu den einzelnen Bedrohungen, indem Sie darauf klicken. Das folgende Beispiel zeigt weitere Details zum Botnet:

![Weitere Details zur Bedrohung](./media/oms-security-responding-alerts/oms-security-responding-alerts-fig4.png)

Wie am Anfang dieses Abschnitts beschrieben, können diese Informationen bei der Reaktion auf Sicherheitsvorfälle sehr nützlich sein. Sie können auch bei einer forensischen Untersuchung wichtig sein, bei der Sie die Quelle des Angriffs, die betroffenen Systeme und die Zeitachse ermitteln müssen. In diesem Bericht können Sie sehr einfach einige wichtige Details über den Angriff herausfinden. Dazu gehören die Quelle des Angriffs, die lokal betroffene IP-Adresse und der aktuelle Sitzungszustand der Verbindung. 

Die Karte **Informationen zu Bedrohungen** hilft Ihnen beim Auffinden der aktuellen Orte mit böswilligem Datenverkehr auf der ganzen Welt. Die Karte gibt die Richtung des Datenverkehrs mit orangefarbenen (eingehend) und roten (ausgehend) Pfeilen an. Wenn Sie auf einen dieser Pfeile klicken, werden die Art der Bedrohung und die Datenrichtung angezeigt, wie unten dargestellt:

![Threat Intelligence-Karte](./media/oms-security-responding-alerts/oms-security-responding-alerts-fig5.png)

> [!NOTE]
> Die Verwendung dieser Funktion bei der Reaktion auf einen Incident wird in der Präsentation [Mitigate datacenter security threats with guided investigation using Operations Management Suite](https://myignite.microsoft.com/videos/5000) (Minimieren von Sicherheitsrisiken im Rechenzentrum per geführter Untersuchung mithilfe der Operations Management Suite) demonstriert, die Sie auf Microsoft Ignite finden.
> 

### <a name="responding-to-distinct-malicious-ip-accessed"></a>Reagieren auf bestimmte böswillige IP-Adresse, auf die zugegriffen wurde
In einigen Szenarien sehen Sie möglicherweise eine potenziell böswillige IP-Adresse, auf die einer der überwachten Computer zugegriffen hat:

![Threat Intelligence-Karte](./media/oms-security-responding-alerts/oms-security-responding-alerts-fig6.PNG)

Diese Warnung und andere derselben Kategorie werden über die OMS-Sicherheit generiert, wobei [Microsoft Threat Intelligence](https://youtu.be/O4WtxgUrDc8) verwendet wird. Die Threat Intelligence-Daten werden von Microsoft gesammelt und von führenden Threat Intelligence-Anbietern erworben. Diese Daten werden regelmäßig aktualisiert und an sich schnell entwickelnde Bedrohungen angepasst. Es empfiehlt sich eine Kombination mit anderen Quellen von Sicherheitsinformationen, wenn eine Sicherheitswarnung [untersucht](https://blogs.technet.microsoft.com/msoms/2016/12/08/investigating-suspicious-activity-in-a-hybrid-cloud-with-oms-security/) wird. 

## <a name="customize-alerts-received-via-e-mail"></a>Anpassen von per E-Mail empfangenen Warnungen

Sie können anpassen, welche Benutzer in Ihrer Organisation benachrichtigt werden, wenn durch die Sicherheitslösung von OMS Sicherheitswarnungen ausgelöst werden. Diese Option ist im OMS-Dashboard unter „Überblick“ > „Einstellungen“ verfügbar:

![E-Mail](./media/oms-security-responding-alerts/oms-security-responding-alerts-fig7.png)

## <a name="see-also"></a>Weitere Informationen
In diesem Dokument haben Sie erfahren, wie Sie die Option **Threat Intelligence** in der Sicherheits- und Überwachungslösung von OMS für das Reagieren auf Sicherheitswarnungen verwenden. Weitere Informationen zur Sicherheitslösung von OMS finden Sie in den folgenden Artikeln:

* [Operations Management Suite (OMS) – Übersicht](operations-management-suite-overview.md)
* [Erste Schritte mit der Sicherheits- und Überwachungslösung von Operations Management Suite](oms-security-getting-started.md)
* [Überwachen von Ressourcen in der Sicherheits- und Überwachungslösung der Operations Management Suite](oms-security-monitoring-resources.md)

