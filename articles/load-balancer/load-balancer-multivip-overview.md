---
title: "Mehrere VIPs für Azure Load Balancer | Microsoft Docs"
description: "Übersicht über die Verwendung mehrerer VIPs in Azure Load Balancer"
services: load-balancer
documentationcenter: na
author: chkuhtz
manager: narayan
editor: 
ms.assetid: 748e50cd-3087-4c2e-a9e1-ac0ecce4f869
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: chkuhtz
ms.openlocfilehash: 1045a18f5fd9739a6028198deea129e9e621f127
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 10/11/2017
---
# <a name="multiple-vips-for-azure-load-balancer"></a>Mehrere VIPs für Azure Load Balancer

[!INCLUDE [load-balancer-basic-sku-include.md](../../includes/load-balancer-basic-sku-include.md)]

Mit Azure Load Balancer können Sie für Dienste an mehreren Ports, mehreren IP-Adressen oder beidem einen Lastenausgleich vornehmen. Sie können öffentliche und interne Load Balancer-Definitionen verwenden, um einen Lastenausgleich für Datenflüsse innerhalb einer VM-Gruppe durchzuführen.

Dieser Artikel beschreibt die Grundlagen dieser Funktion, wichtige Konzepte und Einschränkungen. Wenn Sie Dienste mit einer IP-Adresse verfügbar machen möchten, finden Sie vereinfachte Anweisungen für öffentliche Load Balancer-Konfigurationen [hier](load-balancer-get-started-internet-portal.md) und für interne Load Balancer-Konfigurationen [hier](load-balancer-get-started-ilb-arm-portal.md). Mehrere VIPs können einer Konfiguration mit einer einzelnen VIP inkrementell hinzugefügt. Mit den Konzepten in diesem Artikel können Sie eine vereinfachte Konfiguration jederzeit erweitern.

Wenn Sie einen Azure Load Balancer definieren, sind eine Front-End- und eine Back-End-Konfiguration über Regeln verbunden. Mit dem Integritätstest, auf den von der Regel verwiesen wird, wird bestimmt, wie neue Datenflüsse an einen Knoten im Back-End-Pool gesendet werden. Das Front-End wird durch eine virtuelle IP (VIP) definiert. Dabei handelt es sich um ein 3-Tupel bestehend aus einer IP-Adresse (öffentlich oder intern), einem Transportprotokoll (UDP oder TCP) und einer Portnummer. Eine DIP ist eine IP-Adresse einer virtuellen Azure-NIC, die einer VM im Back-End-Pool zugeordnet ist.

Die folgende Tabelle enthält einige Beispielkonfigurationen des Front-Ends:

| VIP | IP-Adresse | protocol | port |
| --- | --- | --- | --- |
| 1 |65.52.0.1 |TCP |80 |
| 2 |65.52.0.1 |TCP |*8080* |
| 3 |65.52.0.1 |*UDP* |80 |
| 4 |*65.52.0.2* |TCP |80 |

Die Tabelle enthält vier verschiedene Front-Ends. Die Front-Ends 1, 2 und 3 sind eine einzelne VIP mit mehreren Regeln. Die gleiche IP-Adresse wird verwendet, aber der Port oder das Protokoll ist für jedes Front-End anders. Die Front-Ends 1 und 4 sind ein Beispiel für mehrere VIPs, wobei das gleiche Front-End-Protokoll und der gleiche Port für mehrere VIPs verwendet werden.

Azure Load Balancer bietet Flexibilität beim Definieren der Lastenausgleichsregeln. Eine Regel deklariert, wie eine Adresse und ein Port auf dem Front-End der Zieladresse und dem Port auf dem Back-End zugeordnet werden. Ob Back-End-Ports in mehreren Regeln wiederverwendet werden, hängt vom Typ der Regel ab. Für jeden Regeltyp gelten bestimmte Anforderungen, die die Hostkonfiguration und den Testentwurf beeinflussen können. Es gibt zwei Regeltypen:

1. Die Standardregel ohne Wiederverwendung des Back-End-Ports
2. Die Floating IP-Regel, bei der Back-End-Ports wiederverwendet werden

Mit Azure Load Balancer können Sie beide Regeltypen in einer Load Balancer-Konfiguration mischen. Der Load Balancer kann sie gleichzeitig für eine bestimmte VM oder eine beliebige Kombination verwenden, sofern Sie die Einschränkungen der Regel beachten. Welchen Regeltyp Sie auswählen, hängt von den Anforderungen der Anwendung und der Komplexität bei der Unterstützung der Konfiguration ab. Sie sollten bewerten, welche Regeltypen für Ihr Szenario am besten geeignet sind.

Wir untersuchen diese Szenarien näher und beginnen mit dem Standardverhalten.

## <a name="rule-type-1-no-backend-port-reuse"></a>Regeltyp 1: Keine Wiederverwendung von Back-End-Ports

![Abbildung zur Verwendung mehrerer VIPs](./media/load-balancer-multivip-overview/load-balancer-multivip.png)

In diesem Szenario werden die Front-End-VIPs wie folgt konfiguriert:

| VIP | IP-Adresse | protocol | port |
| --- | --- | --- | --- |
| ![VIP](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) 1 |65.52.0.1 |TCP |80 |
| ![VIP](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) 2 |*65.52.0.2* |TCP |80 |

Die DIP ist das Ziel des eingehenden Datenflusses. Im Back-End-Pool macht jede VM den gewünschten Dienst an einem eindeutigen Port einer DIP verfügbar. Dieser Dienst ist dem Front-End über eine Regeldefinition zugeordnet.

Wir definieren zwei Regeln:

| Regel | Front-End-Zuordnung | Im Back-End-Pool |
| --- | --- | --- |
| 1 |![VIP](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) VIP1:80 |![Back-End](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) DIP1:80, ![Back-End](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) DIP2:80 |
| 2 |![VIP](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) VIP2:80 |![Back-End](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) DIP1:81, ![Back-End](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) DIP2:81 |

Die vollständige Zuordnung in Azure Load Balancer sieht jetzt wie folgt aus:

| Regel | VIP-IP-Adresse | protocol | port | Ziel | port |
| --- | --- | --- | --- | --- | --- |
| ![Regel](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) 1 |65.52.0.1 |TCP |80 |DIP-IP-Adresse |80 |
| ![Regel](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) 2 |65.52.0.2 |TCP |80 |DIP-IP-Adresse |81 |

Jede Regel muss einen Datenfluss mit einer eindeutigen Kombination aus IP-Zieladresse und Zielport erzeugen. Durch die Änderung des Zielports für den Datenfluss können mehrere Regeln Datenflüsse an die gleiche DIP an unterschiedlichen Ports übermitteln.

Integritätstests werden immer an die DIP einer VM weitergeleitet. Sie müssen sicherstellen, dass der Test den Zustand der VM widerspiegelt.

## <a name="rule-type-2-backend-port-reuse-by-using-floating-ip"></a>Regeltyp 2: Wiederverwendung von Back-End-Ports mit Floating IP

Azure Load Balancer bietet die Flexibilität, den Front-End-Port unabhängig vom verwendeten Regeltyp für mehrere VIPs wiederzuverwenden. Darüber hinaus ist es in einigen Anwendungsszenarien eine Priorität bzw. eine Anforderung, den gleichen Port für mehrere Anwendungsinstanzen auf einer einzelnen VM im Back-End-Pool zu verwenden. Gängige Beispiele für die Portwiederverwendung sind das Clustering für hohe Verfügbarkeit, virtuelle Netzwerkgeräte und die Bereitstellung mehrerer TLS-Endpunkte ohne erneute Verschlüsselung.

Wenn Sie den Back-End-Port in mehreren Regeln wiederverwenden möchten, müssen Sie in der Regeldefinition Floating IP aktivieren.

Floating IP ist ein Bestandteil von Direct Server Return (DSR). DSR besteht aus zwei Teilen: einer Datenflusstopologie und einem Zuordnungsschema für IP-Adressen. Auf Plattformebene wird Azure Load Balancer immer in einer DSR-Datenflusstopologie betrieben, unabhängig davon, ob Floating IP aktiviert ist. Dies bedeutet, dass der ausgehende Teil eines Datenflusses immer ordnungsgemäß so umgeschrieben wird, dass er direkt wieder an den Ursprung übermittelt wird.

Um die Nutzung zu vereinfachen, stellt Azure mit dem Standardregeltyp ein herkömmliches Zuordnungsschema für IP-Adressen für den Lastenausgleich bereit. Durch das Aktivieren von Floating IP ändert sich das Zuordnungsschema für IP-Adressen, sodass größere Flexibilität möglich ist, wie im Folgenden beschrieben.

Das folgende Diagramm veranschaulicht diese Konfiguration:

![Abbildung zur Verwendung mehrerer VIPs](./media/load-balancer-multivip-overview/load-balancer-multivip-dsr.png)

In diesem Szenario hat jede VM im Back-End-Pool drei Netzwerkschnittstellen:

* DIP: eine mit der VM verknüpfte virtuelle NIC (IP-Konfiguration der NIC-Ressource von Azure)
* VIP1: eine Loopback-Schnittstelle im Gastbetriebssystem, die mit der IP-Adresse von VIP1 konfiguriert ist
* VIP2: eine Loopback-Schnittstelle im Gastbetriebssystem, die mit der IP-Adresse von VIP2 konfiguriert ist

> [!IMPORTANT]
> Die Konfiguration der logischen Schnittstellen erfolgt innerhalb des Gastbetriebssystems. Diese Konfiguration wird nicht von Azure vorgenommen oder verwaltet. Ohne diese Konfiguration funktionieren die Regeln nicht. Integritätstestdefinitionen verwenden die DIP der VM statt der logischen VIP. Deshalb muss Ihr Dienst Testantworten an einem DIP-Port bereitstellen, die den Status des Diensts wiedergeben, der an der logischen VIP angeboten wird.

Gehen wir von der gleichen Front-End-Konfiguration wie im vorherigen Szenario aus:

| VIP | IP-Adresse | protocol | port |
| --- | --- | --- | --- |
| ![VIP](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) 1 |65.52.0.1 |TCP |80 |
| ![VIP](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) 2 |*65.52.0.2* |TCP |80 |

Wir definieren zwei Regeln:

| Regel | Front-End-Zuordnung | Im Back-End-Pool |
| --- | --- | --- |
| 1 |![Regel](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) VIP1:80 |![Back-End](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) VIP1:80 (in VM1 und VM2) |
| 2 |![Regel](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) VIP2:80 |![Back-End](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) VIP2:80 (in VM1 und VM2) |

Die folgende Tabelle veranschaulicht die vollständige Zuordnung im Load Balancer:

| Regel | VIP-IP-Adresse | protocol | port | Ziel | port |
| --- | --- | --- | --- | --- | --- |
| ![VIP](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) 1 |65.52.0.1 |TCP |80 |wie VIP (65.52.0.1) |wie VIP (80) |
| ![VIP](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) 2 |65.52.0.2 |TCP |80 |wie VIP (65.52.0.2) |wie VIP (80) |

Das Ziel des eingehenden Datenflusses ist die VIP-Adresse der Loopback-Schnittstelle der VM. Jede Regel muss einen Datenfluss mit einer eindeutigen Kombination aus IP-Zieladresse und Zielport erzeugen. Durch die Änderung der IP-Zieladresse für den Datenfluss ist die Portwiederverwendung auf der gleichen VM möglich. Ihr Dienst wird für den Load Balancer verfügbar gemacht, indem er an die VIP-IP-Adresse und den Port der jeweiligen Loopback-Schnittstelle gebunden wird.

Beachten Sie, dass in diesem Beispiel der Zielport nicht geändert wird. Obwohl dies ein Floating IP-Szenario ist, unterstützt Azure Load Balancer auch das Definieren einer Regel, um den Back-End-Zielport zu ändern, damit er sich vom Front-End-Zielport unterscheidet.

Der Floating IP-Regeltyp bildet die Grundlage für mehrere Load Balancer-Konfigurationsmuster. Ein Beispiel, das derzeit verfügbar ist, ist die [Konfiguration von SQL AlwaysOn mit mehreren Listenern](../virtual-machines/windows/sql/virtual-machines-windows-portal-sql-ps-alwayson-int-listener.md) . Mit der Zeit werden weitere dieser Szenarien dokumentiert.

## <a name="limitations"></a>Einschränkungen

* Konfigurationen mit mehreren VIPs werden nur für IaaS-VMs unterstützt.
* Bei der Floating IP-Regel muss die Anwendung die DIP für ausgehende Datenflüsse verwenden. Wenn Ihre Anwendung an die VIP-Adresse gebunden ist, die an der Loopback-Schnittstelle im Gastbetriebssystem konfiguriert ist, ist SNAT nicht verfügbar, um den ausgehenden Datenfluss umzuschreiben, und der Datenfluss schlägt fehl.
* Öffentliche IP-Adressen haben Auswirkungen auf die Abrechnung. Weitere Informationen finden Sie unter [Preise für IP-Adressen](https://azure.microsoft.com/pricing/details/ip-addresses/)
* Es gelten Grenzwerte für Abonnements. Weitere Informationen finden Sie unter [Einschränkungen für Dienste](../azure-subscription-service-limits.md#networking-limits) .
