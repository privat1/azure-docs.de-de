---
title: Zum Herstellen einer Verbindung mit Azure Analysis Services erforderliche Clientbibliotheken | Microsoft-Dokumentation
description: "Beschreibt für Clientanwendungen und Tools erforderliche Clientbibliotheken zum Herstellen einer Verbindung mit Azure Analysis Services"
services: analysis-services
documentationcenter: 
author: minewiskan
manager: kfile
editor: 
tags: 
ms.assetid: 
ms.service: analysis-services
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: na
ms.date: 12/14/2017
ms.author: owend
ms.openlocfilehash: 870d430d1926859894f452e0af812d794272a9e6
ms.sourcegitcommit: 0e4491b7fdd9ca4408d5f2d41be42a09164db775
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/14/2017
---
# <a name="client-libraries-for-connecting-to-azure-analysis-services"></a>Clientbibliotheken zum Herstellen einer Verbindung mit Azure Analysis Services

Damit Clientanwendungen und Tools eine Verbindung mit Analysis Services-Servern herstellen können, sind Clientbibliotheken erforderlich. 

## <a name="download-the-latest-client-libraries"></a>Herunterladen der neuesten Clientbibliotheken  

|Herunterladen  |Version  | 
|---------|---------|
|[MSOLAP (amd64)](https://go.microsoft.com/fwlink/?linkid=829576)    |    14.0.801.241      |
|[MSOLAP (x86)](https://go.microsoft.com/fwlink/?linkid=829575)     |    14.0.801.241      |
|[AMO](https://go.microsoft.com/fwlink/?linkid=829578)     |   14.0.800.117      |
|[ADOMD](https://go.microsoft.com/fwlink/?linkid=829577)     |    14.0.801.241      |

## <a name="understanding-client-libraries"></a>Grundlegendes zu Clientbibliotheken

Für Analysis Services werden drei Clientbibliotheken verwendet, die auch als Datenanbieter bezeichnet werden. ADOMD.NET und Analysis Services Management Objects (AMO) sind verwaltete Clientbibliotheken. Der Analysis Services OLE DB-Anbieter (MSOLAP DLL) ist eine native Clientbibliothek. In der Regel werden alle drei gleichzeitig installiert. **Für Azure Analysis Services werden die neuesten Versionen aller drei Bibliotheken benötigt**. 

Bei Microsoft-Clientanwendungen wie Power BI Desktop und Excel werden alle drei Clientbibliotheken installiert und dann entsprechend aktualisiert, wenn neue Versionen verfügbar sind. Abhängig von der Version oder der Updatehäufigkeit weisen einige Clientbibliotheken dabei unter Umständen nicht die neuesten Versionen auf, die von Azure Analysis Services benötigt werden. Dies gilt auch für benutzerdefinierte Anwendungen oder andere Schnittstellen wie AsCmd, TOM, ADOMD.NET. Für diese Anwendungen müssen die Bibliotheken manuell oder programmgesteuert installiert werden. Die Clientbibliotheken für die manuelle Installation sind in SQL Server-Funktionspaketen wie verteilbaren Paketen enthalten. Allerdings sind diese Clientbibliotheken an die SQL Server-Version gebunden und möglicherweise nicht auf dem neuesten Stand.  

Clientbibliotheken für Clientverbindungen unterscheiden sich von Datenanbietern, die für die Verbindung eines Azure Analysis Services-Servers mit einer Datenquelle erforderlich sind. Weitere Informationen zu den Datenquellenverbindungen finden Sie unter [Datenquellenverbindungen](analysis-services-datasource.md).

## <a name="client-library-types"></a>Typen von Clientbibliotheken

### <a name="analysis-services-ole-db-provider-msolap"></a>Analysis Services OLE DB-Anbieter (MSOLAP) 

 Der Analysis Services OLE DB-Anbieter (MSOLAP) ist die native Clientbibliothek für Analysis Services-Datenbankverbindungen. Er wird indirekt von ADOMD.NET und AMO genutzt, und Verbindungsanforderungen werden an den Datenanbieter delegiert. Sie können den OLE DB-Anbieter auch direkt über den Anwendungscode aufrufen.  
  
 Der Analysis Services OLE DB-Anbieter wird von den meisten Tools und Clientanwendungen, die zum Zugreifen auf Analysis Services-Datenbanken verwendet werden, automatisch installiert. Er muss auf Computern installiert sein, die zum Zugreifen auf Analysis Services-Daten verwendet werden.  
  
 OLE DB-Anbieter werden häufig in Verbindungszeichenfolgen angegeben. Für eine Analysis Services-Verbindungszeichenfolge wird eine andere Nomenklatur genutzt, um auf den OLE DB-Anbieter zu verweisen: MSOLAP.\<version>.dll.

### <a name="amo"></a>AMO  

 AMO ist eine verwaltete Clientbibliothek, die für die Serververwaltung und Datendefinition verwendet wird. Sie wird von Tools und Clientanwendungen installiert und genutzt. Für SQL Server Management Studio (SSMS) wird AMO beispielsweise zum Herstellen einer Verbindung mit Analysis Services genutzt.  
  
 Eine Verbindung, für die AMO genutzt wird, ist meist eine minimale Verbindung, die aus `“data source=\<servername>”` besteht. Nach dem Herstellen einer Verbindung verwenden Sie die API, um mit Datenbanksammlungen und größeren Objekten zu arbeiten. AMO wird sowohl von SSDT als auch von SSMS verwendet, um eine Verbindung mit einer Analysis Services-Instanz herzustellen.  

  
### <a name="adomd"></a>ADOMD

 ADOMD.NET ist eine verwaltete Datenclientbibliothek, die zum Abfragen von Analysis Services-Daten eingesetzt wird. Sie wird von Tools und Clientanwendungen installiert und genutzt. 
  
 Beim Herstellen einer Verbindung mit der Datenbank sind die Eigenschaften der Verbindungszeichenfolgen für alle drei Bibliotheken ähnlich. Fast jede Verbindungszeichenfolge, die Sie für ADOMD.NET per [Microsoft.AnalysisServices.AdomdClient.AdomdConnection.ConnectionString](https://msdn.microsoft.com/library/microsoft.analysisservices.adomdclient.adomdconnection.connectionstring.aspx) definieren, funktioniert auch für AMO und den Analysis Services OLE DB-Anbieter (MSOLAP). Weitere Informationen finden Sie unter [Verbindungszeichenfolgen-Eigenschaften (Analysis Services)](https://docs.microsoft.com/sql/analysis-services/instances/connection-string-properties-analysis-services).  

  
##  <a name="bkmk_LibUpdate"></a> Ermitteln der Version einer Clientbibliothek   
  
### <a name="oleddb-msolap"></a>OLEDDB (MSOLAP)  
  
1.  Wechseln Sie zur Adresse `C:\Program Files\Microsoft Analysis Services\AS OLEDB\140`. Wählen Sie den Ordner mit der höheren Zahl, falls Sie mehr als einen Ordner haben.
  
2.  Klicken Sie mit der rechten Maustaste auf **msolap140.dll** > **Eigenschaften** > **Details**.  
    
    ![Details zur Clientbibliothek](media/analysis-services-data-providers/aas-msolap-details.png)
  
### <a name="amo"></a>AMO

1. Wechseln Sie zur Adresse `C:\Windows\Microsoft.NET\assembly\GAC_MSIL\Microsoft.AnalysisServices\v4.0_14.0.0.0__89845dcd8080cc91`.
2. Klicken Sie mit der rechten Maustaste auf **Microsoft.AnalysisServices** > **Eigenschaften** > **Details**.  

### <a name="adomd"></a>ADOMD

1. Wechseln Sie zur Adresse `C:\Windows\Microsoft.NET\assembly\GAC_MSIL\Microsoft.AnalysisServices.AdomdClient\v4.0_14.0.0.0__89845dcd8080cc91`.
2. Klicken Sie mit der rechten Maustaste auf **Microsoft.AnalysisServices.AdomdClient** > **Eigenschaften** > **Details**.  


## <a name="next-steps"></a>Nächste Schritte
[Herstellen einer Verbindung mit Excel](analysis-services-connect-excel.md)    
[Herstellen einer Verbindung mit Power BI](analysis-services-connect-pbi.md)
