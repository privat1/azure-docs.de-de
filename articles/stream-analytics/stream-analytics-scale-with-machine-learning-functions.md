---
title: Auftragsskalierung mit Azure Stream Analytics und Azure ML-Funktionen | Microsoft-Dokumentation
description: "Erfahren Sie, wie Sie Stream Analytics-Aufträge (Partitionierung, SU-Menge usw.) richtig skalieren, wenn Sie Azure Machine Learning-Funktionen verwenden."
keywords: 
documentationcenter: 
services: stream-analytics
author: samacha
manager: jhubbard
editor: cgronlun
ms.assetid: 47ce7c5e-1de1-41ca-9a26-b5ecce814743
ms.service: stream-analytics
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: data-services
ms.date: 03/28/2017
ms.author: samacha
ms.openlocfilehash: 1e1c3724462a4d2a67eab3ef42867d2aeb5d3fa1
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 10/11/2017
---
# <a name="scale-your-stream-analytics-job-with-azure-machine-learning-functions"></a>Skalieren eines Stream Analytics-Auftrags mit Azure Machine Learning-Funktionen
Häufig ist es einfach, einen Stream Analytics-Auftrag einzurichten und Beispieldaten dafür auszuführen. Was ist zu tun, wenn derselbe Auftrag mit einem höheren Datenvolumen ausgeführt werden soll? Hierfür müssen wir wissen, wie der Stream Analytics-Auftrag konfiguriert werden muss, damit er skaliert werden kann. In diesem Dokument liegt der Schwerpunkt auf den speziellen Aspekten der Skalierung von Stream Analytics-Aufträgen mit Machine Learning-Funktionen. Allgemeine Informationen zum Skalieren von Stream Analytics-Aufträgen finden Sie im Artikel [Skalieren von Aufträgen](stream-analytics-scale-jobs.md).

## <a name="what-is-an-azure-machine-learning-function-in-stream-analytics"></a>Was ist in Stream Analytics eine Azure Machine Learning-Funktion?
Eine Machine Learning-Funktion kann in Stream Analytics wie ein normaler Funktionsaufruf in der Stream Analytics-Abfragesprache verwendet werden. Im Hintergrund handelt es sich bei den Funktionsaufrufen aber um Azure Machine Learning-Webdienstanforderungen. Machine Learning-Webdienste unterstützen das „Batching“ mehrerer Zeilen zu einem so genannten Mini-Batch in demselben Webdienst-API-Aufruf, um den Gesamtdurchsatz zu verbessern. Weitere Details finden Sie im Blogbeitrag [Azure ML Now Available as a Function in Azure Stream Analytics](https://blogs.technet.microsoft.com/machinelearning/2015/12/10/azure-ml-now-available-as-a-function-in-azure-stream-analytics/) (Azure ML jetzt als Funktion in Azure Stream Analytics verfügbar) und im Artikel [Nutzen eines Azure Machine Learning-Webdiensts, der von einem Machine Learning-Experiment aus bereitgestellt wurde](../machine-learning/studio/consume-web-services.md).

## <a name="configure-a-stream-analytics-job-with-machine-learning-functions"></a>Konfigurieren eines Stream Analytics-Auftrags mit Machine Learning-Funktionen
Beim Konfigurieren einer Machine Learning-Funktion für einen Stream Analytics-Auftrag müssen zwei Parameter berücksichtigt werden: die Batchgröße der Machine Learning-Funktionsaufrufe und die Streamingeinheiten (Streaming Units, SUs), die für den Stream Analytics-Auftrag bereitgestellt werden. Zum Bestimmen der richtigen Werte hierfür muss zuerst eine Entscheidung zwischen der Latenz und dem Durchsatz getroffen werden, also der Latenz des Stream Analytics-Auftrags und des Durchsatzes der einzelnen Streamingeinheiten. SUs können einem Auftrag immer hinzugefügt werden, um den Durchsatz einer gut partitionierten Stream Analytics-Abfrage zu erhöhen. Zusätzliche SUs führen aber zu einer Erhöhung der Kosten für die Ausführung des Auftrags.

Daher ist es wichtig, die *Toleranz* in Bezug auf die Latenz beim Ausführen eines Stream Analytics-Auftrags zu bestimmen. Die zusätzliche Latenz aufgrund der Ausführung von Azure Machine Learning-Dienstanforderungen steigt logischerweise an, wenn sich die Batchgröße erhöht, und somit auch die Gesamtlatenz des Stream Analytics-Auftrags. Andererseits ermöglicht eine höhere Batchgröße auch, dass vom Stream Analytics-Auftrag *mehr Ereignisse mit der *gleichen Anzahl* von Machine Learning-Webdienstanforderungen verarbeitet werden. Häufig ist die Erhöhung der Machine Learning-Webdienstlatenz sublinear zum Anstieg der Batchgröße. Daher ist es wichtig, für einen Machine Learning-Webdienst immer die Batchgröße mit der größten Kosteneffizienz zu ermitteln. Die Standardbatchgröße für die Webdienstanforderungen lautet 1000 und kann geändert werden, indem Sie entweder die [Stream Analytics-REST-API](https://msdn.microsoft.com/library/mt653706.aspx "Stream Analytics-REST-API") oder den [PowerShell-Client für Stream Analytics](stream-analytics-monitor-and-manage-jobs-use-powershell.md "PowerShell-Client für Stream Analytics") verwenden.

Nachdem eine Batchgröße bestimmt wurde, kann die Menge an Streamingeinheiten (Streaming Units, SUs) basierend auf der Anzahl von Ereignissen ermittelt werden, die von der Funktion pro Sekunde verarbeitet werden müssen. Weitere Informationen zu Streamingeinheiten finden Sie unter [Stream Analytics-Skalierungsaufträge](stream-analytics-scale-jobs.md).

Im Allgemeinen sind 20 gleichzeitige Verbindungen mit dem Machine Learning-Webdienst für jeweils 6 SUs vorhanden, aber mit der Ausnahme, dass auch Aufträge mit 1 SU und Aufträge mit 3 SUs 20 gleichzeitige Verbindungen erhalten.  Wenn die Eingangsdatenrate beispielsweise 200.000 Ereignisse pro Sekunde beträgt und für die Batchgröße die Standardeinstellung von 1.000 beibehalten wird, beträgt die sich ergebende Webdienstlatenz für 1.000 Ereignisse pro Mini-Batch 200 ms. Dies bedeutet, dass jede Verbindung pro Sekunde fünf Anforderungen an den Machine Learning-Webdienst durchführen kann. Mit 20 Verbindungen kann der Stream Analytics-Auftrag 20.000 Ereignisse in 200 ms verarbeiten, also 100.000 Ereignisse pro Sekunde. Zum Verarbeiten von 200.000 Ereignissen pro Sekunde benötigt der Stream Analytics-Auftrag also 40 gleichzeitige Verbindungen und somit 12 SUs. Im folgenden Diagramm ist der Verlauf der Anforderungen vom Stream Analytics-Auftrag zum Machine Learning-Webdienst-Endpunkt dargestellt. Für 6 SUs sind jeweils maximal 20 gleichzeitige Verbindungen mit dem Machine Learning-Webdienst vorhanden.

![Skalieren von Stream Analytics mit Machine Learning-Funktionen – Beispiel mit zwei Aufträgen](./media/stream-analytics-scale-with-ml-functions/stream-analytics-scale-with-ml-functions-00.png "Skalieren von Stream Analytics mit Machine Learning-Funktionen – Beispiel mit zwei Aufträgen")

Wenn ***B*** die Batchgröße und ***L*** die Webdienstlatenz bei Batchgröße B in Millisekunden ist, beträgt der Durchsatz eines Stream Analytics-Auftrags mit ***N*** SUs:

![Skalieren von Stream Analytics mit Machine Learning-Funktionen – Formel](./media/stream-analytics-scale-with-ml-functions/stream-analytics-scale-with-ml-functions-02.png "Skalieren von Stream Analytics mit Machine Learning-Funktionen – Formel")

Ein weiterer zu berücksichtigender Aspekt kann auch der Wert für die „maximalen gleichzeitigen Aufrufe“ auf der Seite des Machine Learning-Webdiensts sein. Es wird empfohlen, hierfür den maximalen Wert festzulegen (derzeit 200).

Weitere Informationen zu dieser Einstellung finden Sie im [Artikel zur Skalierung für Machine Learning-Webdienste](../machine-learning/studio/scaling-webservice.md).

## <a name="example--sentiment-analysis"></a>Beispiel: Stimmungsanalyse
Das folgende Beispiel enthält einen Stream Analytics-Auftrag mit der Machine Learning-Funktion für die Stimmungsanalyse, die im [Tutorial zur Machine Learning-Integration für Stream Analytics](stream-analytics-machine-learning-integration-tutorial.md)beschrieben ist.

Die Abfrage umfasst eine einfache vollständig partitionierte Abfrage gefolgt von der Funktion **sentiment**. Dies ist nachstehend dargestellt:

    WITH subquery AS (
        SELECT text, sentiment(text) as result from input
    )

    Select text, result.[Score]
    Into output
    From subquery

Stellen Sie sich das folgende Szenario vor: Bei einem Durchsatz von 10.000 Tweets pro Sekunde muss ein Stream Analytics-Auftrag erstellt werden, um die Stimmungsanalyse der Tweets (Ereignisse) durchzuführen. Kann dieser Stream Analytics-Auftrag den Datenverkehr mit einer SU verarbeiten? Bei Verwendung der standardmäßigen Batchgröße von 1.000 sollte der Auftrag dazu in der Lage sein, den Dateneingang zu verarbeiten. Außerdem sollte für die hinzugefügte Machine Learning-Funktion nicht mehr als eine Sekunde Latenz generiert werden, also die allgemeine Standardlatenz des Machine Learning-Webdiensts für die Stimmungsanalyse (mit einer Standardbatchgröße von 1.000). Die **gesamte** Latenz bzw. End-to-End-Latenz des Stream Analytics-Auftrags würde normalerweise einige Sekunden betragen. Es ist ratsam, sich diesen Stream Analytics-Auftrag genauer anzusehen, *vor allem* die Machine Learning-Funktionsaufrufe. Bei einer Batchgröße von 1.000 werden bei einem Durchsatz von 10.000 Ereignissen ca. 10 Anforderungen an den Webdienst geleitet. Auch bei nur einer SU sind genügend gleichzeitige Verbindungen vorhanden, um diesen Datenverkehrseingang abzudecken.

Aber was passiert, wenn sich die Eingangsrate der Ereignisse um das Hundertfache erhöht und mit dem Stream Analytics-Auftrag nun 1.000.000 Tweets pro Sekunde verarbeitet werden müssen? Es gibt zwei Optionen:

1. Erhöhen Sie die Batchgröße, oder
2. Partitionieren Sie den Eingabedatenstrom, um die Ereignisse parallel zu verarbeiten

Bei der ersten Option erhöht sich die **Latenz** des Auftrags.

Bei der zweiten Option müssen mehr SUs bereitgestellt werden und daher mehr gleichzeitige Machine Learning-Webdienstanforderungen generiert werden. Dies bedeutet, dass die **Kosten** für den Auftrag steigen.

Angenommen, die Latenz des Machine Learning-Webdiensts für die Stimmungsanalyse beträgt 200 ms für Batches mit 1.000 Ereignissen oder weniger, 250 ms für Batches mit 5.000 Ereignissen, 300 ms für Batches mit 10.000 Ereignissen oder 500 ms für Batches mit 25.000 Ereignissen.

1. Bei der ersten Option (**ohne** Bereitstellung von mehr SUs) kann die Batchgröße auf **25.000** erhöht werden. Mit dem Auftrag könnten dann wiederum 1.000.000 Ereignisse mit 20 gleichzeitigen Verbindungen mit dem Machine Learning-Webdienst verarbeitet werden (bei einer Latenz von 500 ms pro Aufruf). Die zusätzliche Latenz des Stream Analytics-Auftrags würde sich also aufgrund der Anforderungen der sentiment-Funktion an die Machine Learning-Webdienstanforderungen von **200 ms** auf **500 ms** erhöhen. Aber die Batchgröße kann **nicht** unendlich erhöht werden, da für die Machine Learning-Webdienste erforderlich ist, dass die Nutzlastgröße einer Anforderung maximal 4 MB beträgt. Für Webdienstanforderungen tritt nach 100 Sekunden eine Zeitüberschreitung auf.
2. Bei der zweiten Option wird die Batchgröße von 1.000 beibehalten. Bei einer Webdienstlatenz von 200 ms können mit 20 gleichzeitigen Verbindungen mit dem Webdienst also jeweils Ereignisse in folgendem Umfang verarbeitet werden: 1.000 * 20 * 5 Ereignisse = 100.000 pro Sekunde. Es sind also 60 SUs für den Auftrag erforderlich, damit 1.000.000 Ereignisse pro Sekunde verarbeitet werden können. Verglichen mit der ersten Option fallen für den Stream Analytics-Auftrag mehr Webdienst-Batchanforderungen an, sodass sich die Kosten erhöhen.

Unten ist eine Tabelle mit Informationen zum Durchsatz des Stream Analytics-Auftrags für unterschiedliche SUs und Batchgrößen angegeben (Anzahl von Ereignissen pro Sekunde).

| Batchgröße (ML-Latenz) | 500 (200 ms) | 1.000 (200 ms) | 5.000 (250 ms) | 10.000 (300 ms) | 25.000 (500 ms) |
| --- | --- | --- | --- | --- | --- |
| **1 SU** |2.500 |5.000 |20.000 |30.000 |50.000 |
| **3 SUs** |2.500 |5.000 |20.000 |30.000 |50.000 |
| **6 SUs** |2.500 |5.000 |20.000 |30.000 |50.000 |
| **12 SUs** |5.000 |10.000 |40.000 |60.000 |100.000 |
| **18 SUs** |7.500 |15.000 |60.000 |90.000 |150.000 |
| **24 SUs** |10.000 |20.000 |80.000 |120.000 |200.000 |
| **…** |… |… |… |… |… |
| **60 SUs** |25.000 |50.000 |200.000 |300.000 |500.000 |

Sie sollten nun bereits über gute Grundlagenkenntnisse verfügen und wissen, wie Machine Learning-Funktionen in Stream Analytics funktionieren. Sie wissen vermutlich auch, dass bei Stream Analytics-Aufträgen Daten aus Datenquellen abgerufen werden („Pull“) und bei jedem Vorgang dieser Art ein Batch mit Ereignissen zur Verarbeitung durch den Stream Analytics-Auftrag zurückgegeben wird. Wie wirkt sich dieses Abrufmodell auf die Machine Learning-Webdienstanforderungen aus?

Normalerweise lässt sich die Batchgröße, die wir für Machine Learning-Funktionen festlegen, nicht genau durch die Anzahl von Ereignissen teilen, die bei jedem Abrufvorgang eines Stream Analytics-Auftrags zurückgegeben werden. In diesem Fall wird der Machine Learning-Webdienst mit „Teilbatches“ aufgerufen. Dies wird praktiziert, damit es nicht zu zusätzlichem Mehraufwand aufgrund von Auftragslatenz kommt, wenn Ereignisse von Abrufvorgang zu Abrufvorgang zusammengefügt werden.

## <a name="new-function-related-monitoring-metrics"></a>Neue funktionsbezogene Überwachungsmetriken
Im Überwachungsbereich eines Stream Analytics-Auftrags wurden drei zusätzliche funktionsbezogene Metriken hinzugefügt. Dies sind FUNKTIONSANFORDERUNGEN, FUNKTIONSEREIGNISSE und FEHLER BEI FUNKTIONSANFORDERUNGEN, wie in der Grafik dargestellt.

![Skalieren von Stream Analytics mit Machine Learning-Funktionen – Metriken](./media/stream-analytics-scale-with-ml-functions/stream-analytics-scale-with-ml-functions-01.png "Skalieren von Stream Analytics mit Machine Learning-Funktionen – Metriken")

Diese sind wie folgt definiert:

**FUNKTIONSANFORDERUNGEN**: Anzahl der Funktionsanforderungen.

**FUNKTIONSEREIGNISSE**: Anzahl der Ereignisse in den Funktionsanforderungen.

**FEHLER BEI FUNKTIONSANFORDERUNGEN**: Anzahl der Funktionsanforderungen mit Fehlern.

## <a name="key-takeaways"></a>Wesentliche Punkte
Zusammenfassend lässt sich festhalten, dass die folgenden Elemente berücksichtigt werden müssen, um einen Stream Analytics-Auftrag mit Machine Learning-Funktionen zu skalieren:

1. Eingangsrate der Ereignisse
2. Tolerierte Latenz für den ausgeführten Stream Analytics-Auftrag (und somit die Batchgröße der Machine Learning-Webdienstanforderungen)
3. Bereitgestellte Stream Analytics-Streamingeinheiten (SUs) und die Anzahl von Machine Learning-Webdienstanforderungen (zusätzliche funktionsbezogene Kosten)

Als Beispiel wurde eine vollständig partitionierte Stream Analytics-Abfrage verwendet. Falls Sie eine komplexere Abfrage benötigen, können Sie das [Azure Stream Analytics-Forum](https://social.msdn.microsoft.com/Forums/en-US/home?forum=AzureStreamAnalytics) als hervorragende Ressource nutzen, um vom Stream Analytics-Team weitere Hilfe zu erhalten.

## <a name="next-steps"></a>Nächste Schritte
Weitere Informationen zu Stream Analytics finden Sie unter:

* [Erste Schritte mit Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Skalieren von Azure Stream Analytics-Aufträgen](stream-analytics-scale-jobs.md)
* [Stream Analytics Query Language Reference (in englischer Sprache)](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Referenz zur Azure Stream Analytics-Verwaltungs-REST-API](https://msdn.microsoft.com/library/azure/dn835031.aspx)
