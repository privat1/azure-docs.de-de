---
title: Azure Cosmos DB-Tabellen-API-.NET-SDK und Ressourcen | Microsoft-Dokumentation
description: "Wichtige Informationen zur Azure Cosmos DB-Tabellen-API einschließlich Veröffentlichungsterminen, Deaktivierungsterminen und Änderungen in den einzelnen Versionen."
services: cosmos-db
documentationcenter: .net
author: rnagpal
manager: jhubbard
editor: cgronlun
ms.assetid: 
ms.service: cosmos-db
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 12/12/2017
ms.author: mimig
ms.openlocfilehash: 02bb5d23ee9468ab1f74396877cdcd6bdd8b8fba
ms.sourcegitcommit: aaba209b9cea87cb983e6f498e7a820616a77471
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/12/2017
---
# <a name="azure-cosmos-db-table-net-api-download-and-release-notes"></a>Azure Cosmos DB-Tabellen-.NET-API: Herunterladen und Versionshinweise
> [!div class="op_single_selector"]
> * [.NET](table-sdk-dotnet.md)
> * [Java](table-sdk-java.md)
> * [Node.js](table-sdk-nodejs.md)
> * [Python](table-sdk-python.md)

|   |   |
|---|---|
|**SDK-Download**|[NuGet](https://aka.ms/acdbtablenuget)|
|**API-Dokumentation**|[.NET API-Referenzdokumentation](https://aka.ms/acdbtableapiref)|
|**Schnellstart**|[Azure Cosmos DB: Build a .NET application using the Table API](create-table-dotnet.md) (Azure Cosmos DB: Erstellen einer .NET-Anwendung mit der Tabellen-API)|
|**Tutorial**|[Azure Cosmos DB: Entwickeln mit der Tabellen-API in .NET](tutorial-develop-table-dotnet.md)|
|**Aktuelles unterstütztes Framework**|[Microsoft .NET Framework 4.5](https://www.microsoft.com/download/details.aspx?id=30653)|

> [!IMPORTANT]
> Wenn Sie während der Vorschauphase ein Tabellen-API-Konto erstellt haben, erstellen Sie ein [neues Tabellen-API-Konto](create-table-dotnet.md#create-a-database-account) für die Verwendung mit den allgemein verfügbaren Table API SDKs.
>

## <a name="release-notes"></a>Versionshinweise

### <a name="a-name100100"></a><a name="1.0.0"/>1.0.0
* Release zur allgemeinen Verfügbarkeit

### <a name="a-name010-preview090-preview"></a><a name="0.1.0-preview"/>0.9.0-preview
* Erste Vorschauversion

## <a name="release-and-retirement-dates"></a>Veröffentlichungs- und Deaktivierungstermine
Wenn Microsoft ein SDK deaktiviert, werden Sie mindestens **12 Monate** vorher benachrichtigt, um einen reibungslosen Übergang zu einer neueren/unterstützten Version zu gewährleisten.

Das Vorschaupaket [WindowsAzure.Storage-PremiumTable](https://www.nuget.org/packages/WindowsAzure.Storage-PremiumTable/0.1.0-preview) ist veraltet und wurde durch das Paket [Microsoft.Azure.CosmosDB.Table](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table) ersetzt. Das SDK WindowsAzure.Storage-PremiumTable wird am 15. November 2018 eingestellt. Ab diesem Zeitpunkt werden Anforderungen an das eingestellte SDK nicht mehr zugelassen.

Neue Features, Funktionen und Optimierungen werden nur dem aktuellen SDK hinzugefügt. Daher wird empfohlen, immer so früh wie möglich auf die neueste SDK-Version zu aktualisieren. 

Anforderungen an Azure Cosmos DB mithilfe eines deaktivierten SDK werden vom Dienst abgelehnt.
<br/>

| Version | Herausgabedatum | Deaktivierungstermine |
| --- | --- | --- |
| [1.0.0](#1.0.0) |15. November 2017|--- |
| [0.9.0-preview](#0.9.0-preview) |11. November 2017 |--- |

## <a name="troubleshooting"></a>Problembehandlung

Wenn Sie die folgende Fehlermeldung erhalten: 

```
Unable to resolve dependency 'Microsoft.Azure.Storage.Common'. Source(s) used: 'nuget.org', 
'CliFallbackFolder', 'Microsoft Visual Studio Offline Packages', 'Microsoft Azure Service Fabric SDK'`
```

und der Versuch, das Microsoft.Azure.CosmosDB.Table NuGet-Paket zu verwenden zugrunde liegt, haben Sie zwei Möglichkeiten, um das Problem zu beheben:

* Verwenden Sie die Paket-Manager-Konsole, um das Microsoft.Azure.CosmosDB.Table-Paket und seine Abhängigkeiten zu installieren. Geben Sie dazu Folgendes in die Paket-Manager-Konsole für Ihre Projektmappe ein. 
    ```
    Install-Package Microsoft.Azure.CosmosDB.Table -IncludePrerelease
    ```
    
* Installieren Sie das NuGet-Paket „Microsoft.Azure.Storage.Common“ mit Ihrem bevorzugten NuGet-Paketverwaltungstool vor der Installation von „Microsoft.Azure.CosmosDB.Table“.

## <a name="faq"></a>Häufig gestellte Fragen

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>Weitere Informationen
Weitere Informationen zur Table-API von Azure Cosmos DB finden Sie in der [Einführung in die Tabellen-API von Azure Cosmos DB](table-introduction.md). 
