---
title: "Azure CLI-Skriptbeispiel – Herstellen einer Verbindung einer Web-App mit einem Speicherkonto | Microsoft-Dokumentation"
description: "Azure CLI-Skriptbeispiel – Herstellen einer Verbindung einer Web-App mit einem Speicherkonto"
services: appservice
documentationcenter: appservice
author: syntaxc4
manager: erikre
editor: 
tags: azure-service-management
ms.assetid: bc8345b2-8487-40c6-a91f-77414e8688e6
ms.service: app-service
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: web
ms.date: 12/11/2017
ms.author: cfowler
ms.custom: mvc
ms.openlocfilehash: 0f60c4cd03a5728dc81cb43e02c8486b15221851
ms.sourcegitcommit: aaba209b9cea87cb983e6f498e7a820616a77471
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/12/2017
---
# <a name="connect-a-web-app-to-a-storage-account"></a>Herstellen einer Verbindung einer Web-App mit einem Speicherkonto

Dieses Beispielskript erstellt ein Azure-Speicherkonto und eine Azure-Web-App. Anschließend wird das Speicherkonto mithilfe von App-Einstellungen mit der Web-App verknüpft.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Wenn Sie die CLI lokal installieren und verwenden möchten, benötigen Sie die Azure CLI-Version 2.0 oder höher. Führen Sie `az --version` aus, um die Version zu finden. Wenn Sie eine Installation oder ein Upgrade ausführen müssen, finden Sie unter [Installieren von Azure CLI 2.0]( /cli/azure/install-azure-cli) Informationen dazu.


## <a name="sample-script"></a>Beispielskript

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/connect-to-storage/connect-to-storage.sh "Azure Storage")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Erläuterung des Skripts

In diesem Skript werden die folgenden Befehle verwendet, um eine Ressourcengruppe, eine Web-App, ein Speicherkonto und alle zugehörigen Ressourcen zu erstellen. Jeder Befehl in der Tabelle ist mit der zugehörigen Dokumentation verknüpft.

| Befehl | Hinweise |
|---|---|
| [`az group create`](/cli/azure/group?view=azure-cli-latest#az_group_create) | Erstellt eine Ressourcengruppe, in der alle Ressourcen gespeichert sind. |
| [`az appservice plan create`](/cli/azure/appservice/plan?view=azure-cli-latest#az_appservice_plan_create) | Erstellt einen App Service-Plan. |
| [`az webapp create`](/cli/azure/webapp?view=azure-cli-latest#az_webapp_create) | Erstellt eine Azure-Web-App. |
| [`az storage account create`](/cli/azure/storage/account?view=azure-cli-latest#az_storage_account_create) | Erstellt ein Speicherkonto. |
| [`az storage account show-connection-string`](/cli/azure/storage/account?view=azure-cli-latest#az_storage_account_show_connection_string) | Abrufen der Verbindungszeichenfolge für ein Speicherkonto |
| [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az_webapp_config_appsettings_set) | Erstellt oder aktualisiert eine App-Einstellung für eine Azure-Web-App. App-Einstellungen werden als Umgebungsvariablen für Ihre App verfügbar gemacht. |

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zur Azure CLI finden Sie in der [Azure CLI-Dokumentation](https://docs.microsoft.com/cli/azure/overview).

Zusätzliche App Service-CLI-Skriptbeispiele finden Sie in der [Dokumentation zu Azure App Service](../app-service-cli-samples.md).
