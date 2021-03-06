---
title: "App Service-Übersicht: Azure Stack | Microsoft-Dokumentation"
description: "Übersicht über App Service in Azure Stack"
services: azure-stack
documentationcenter: 
author: apwestgarth
manager: stefsch
editor: 
ms.assetid: 
ms.service: azure-stack
ms.workload: app-service
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/10/2017
ms.author: anwestg
ms.openlocfilehash: fd2d355b2556faddb06acf2998b54ffcc9aa7919
ms.sourcegitcommit: b07d06ea51a20e32fdc61980667e801cb5db7333
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/08/2017
---
# <a name="app-service-on-azure-stack-overview"></a>Übersicht über App Service in Azure Stack

Azure App Service ist ein Platform-as-a-Service-Angebot (PaaS) von Microsoft Azure in Azure Stack. Mit diesem Dienst können Ihre Kunden – intern und extern – Web-, API- und Azure Functions-Anwendungen für beliebige Plattformen oder Geräte erstellen. Sie können Ihre Apps in lokale Anwendungen integrieren und ihre Geschäftsprozesse automatisieren. Azure Stack-Cloudoperatoren können Kunden-Apps auf vollständig verwalteten virtuellen Computern (VMs) mit beliebigen freigegebenen VM-Ressourcen oder dedizierten VMs ausführen.

Azure App Service enthält Funktionen zum Automatisieren von Geschäftsprozessen und Hosten von Cloud-APIs. Als einzelner integrierter Dienst können Sie mit Azure App Service verschiedene Komponenten – Websites, RESTful-APIs und Geschäftsprozesse – in einer zentralen Lösung zusammenstellen.

## <a name="why-offer-azure-app-service-on-azure-stack"></a>Gründe für das Angebot von Azure App Service in Azure Stack

Hier sind einige wichtige Features und Funktionen von App Service aufgeführt:
- **Mehrere Sprachen und Frameworks:** App Service verfügt über erstklassige Unterstützung für ASP.NET, Node.js, Java, PHP und Python. Außerdem können Sie Windows PowerShell und andere Skripts oder ausführbare Dateien auf App Service-VMs ausführen.
- **DevOps-Optimierung:** Richten Sie Continuous Integration und Continuous Deployment mit GitHub, einem lokalen Git oder Bitbucket ein. Stufen Sie Updates über Test- und Stagingumgebungen herauf. Verwalten Sie Ihre Apps in App Service mithilfe von Azure PowerShell oder der plattformübergreifenden Befehlszeilenschnittstelle (Command-Line Interface, CLI).
- **Visual Studio-Integration:** Dedizierte Tools in Visual Studio optimieren das Erstellen und Bereitstellen von Anwendungen.

## <a name="app-types-in-app-service"></a>App-Typen in App Service

App Service verfügt über mehrere App-Typen, die jeweils zum Hosten einer bestimmten Workload bestimmt sind:

- [Web-Apps:](https://docs.microsoft.com/azure/app-service-web/app-service-web-overview) Hosten von Websites und Webanwendungen
- [API-Apps:](https://docs.microsoft.com/azure/app-service-api/app-service-api-apps-why-best-platform) Hosten von RESTful-APIs
- Azure Functions zum Hosten ereignisgesteuerter, serverloser Workloads

Der Begriff „App“ bezieht sich hier auf die Hostingressourcen, die dediziert für die Ausführung einer Workload bestimmt sind. Wenn wir „Web-App“ als Beispiel nehmen, verstehen Sie darunter wahrscheinlich sowohl die Computeressourcen als auch die Anwendungscodeelemente, die zusammen für einen Browser die Funktionalität bereitstellen. In App Service handelt es sich bei einer Web-App aber um die Computeressourcen, die von Azure Stack zum Hosten Ihres Anwendungscodes bereitgestellt werden.

Die Anwendung kann aus mehreren App Service-Apps unterschiedlicher Art bestehen. Wenn Ihre Anwendung also beispielsweise aus einem Web-Front-End und einem RESTful-API-Back-End besteht, haben Sie folgende Möglichkeiten:
- Sie können beides (Front-End und API) für eine einzelne Web-App bereitstellen.
- Sie können Ihren Front-End-Code für eine Web-App und Ihren Back-End-Code für eine API-App bereitstellen.

   ![](media/azure-stack-app-service-overview/image01.png)

## <a name="what-is-an-app-service-plan"></a>Was ist ein App Service-Plan?

Der App Service-Ressourcenanbieter verwendet den gleichen Code, der für Azure App Service genutzt wird. Daher ist es sinnvoll, einige gemeinsame Konzepte zu beschreiben. In App Service wird der Preiscontainer für Anwendungen als App Service-Plan bezeichnet. Er steht für die Gruppe von dedizierten virtuellen Computern, die zum Vorhalten Ihrer Apps verwendet werden. Ein bestimmtes Abonnement kann mehrere App Service-Pläne enthalten.

In Azure werden freigegebene und dedizierte Worker verwendet. Ein freigegebener Worker unterstützt das mehrinstanzenfähige App-Hosting mit hoher Dichte, und es gibt nur eine Gruppe von freigegebenen Workern. Dedizierte Server werden nur von einem Mandanten verwendet und können drei Größen haben: Klein, Mittel und Groß. Die Anforderungen von lokalen Kunden können mit diesen Begriffen nicht immer beschrieben werden. In App Service in Azure Stack können Ressourcenanbieter-Administratoren die Workerebenen definieren, die sie verfügbar machen möchten. Sie können mehrere Gruppen mit freigegebenen Workern oder verschiedene Gruppen mit dedizierten Workern definieren. Dies richtet sich nach den jeweiligen speziellen Hostinganforderungen. Mit diesen Definitionen der Workerebenen können dann eigene Preis-SKUs definiert werden.

## <a name="portal-features"></a>Features im Portal

Für App Service in Azure Stack wird die gleiche Benutzeroberfläche wie für Azure App Service verwendet. Dies gilt auch für das Back-End. Einige Features sind deaktiviert und funktionieren in Azure Stack nicht. Die Azure-spezifischen Erwartungen bzw. Dienste, die für diese Features erforderlich sind, sind in Azure Stack noch nicht verfügbar.

## <a name="next-steps"></a>Nächste Schritte


- [Vor den ersten Schritten mit App Service in Azure Stack](azure-stack-app-service-before-you-get-started.md)
- [Hinzufügen eines App Service-Ressourcenanbieters zu Azure Stack](azure-stack-app-service-deploy.md)

Sie können auch andere [PaaS-Dienste (Platform-as-a-Service)](azure-stack-tools-paas-services.md) ausprobieren, z.B. den [SQL Server-Ressourcenanbieter](azure-stack-sql-resource-provider-deploy.md) und den [MySQL-Ressourcenanbieter](azure-stack-mysql-resource-provider-deploy.md).
