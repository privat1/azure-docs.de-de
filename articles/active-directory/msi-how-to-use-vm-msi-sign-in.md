---
title: "Verwenden der verwalteten Dienstidentität eines virtuellen Azure-Computers für die Anmeldung"
description: "Schrittanweisungen und Beispiele zur Verwendung des MSI-Dienstprinzipals eines virtuellen Azure-Computers für die Anmeldung und den Zugriff auf Ressourcen eines Skriptclients."
services: active-directory
documentationcenter: 
author: bryanla
manager: mtillman
editor: 
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/01/2017
ms.author: bryanla
ms.openlocfilehash: 2e4da8cd02a1d07a3225a0c1fda4c60928dba8a4
ms.sourcegitcommit: be9a42d7b321304d9a33786ed8e2b9b972a5977e
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/19/2018
---
# <a name="how-to-use-an-azure-vm-managed-service-identity-msi-for-sign-in"></a>Verwenden der verwalteten Dienstidentität (Managed Service Identity, MSI) eines virtuellen Azure-Computers für die Anmeldung 

[!INCLUDE [preview-notice](../../includes/active-directory-msi-preview-notice.md)]  
Dieser Artikel enthält PowerShell- und CLI-Skriptbeispiele für die Anmeldung mit einem MSI-Dienstprinzipal sowie eine Anleitung zu wichtigen Themen, z.B. zur Fehlerbehandlung.

## <a name="prerequisites"></a>Voraussetzungen

[!INCLUDE [msi-qs-configure-prereqs](../../includes/active-directory-msi-qs-configure-prereqs.md)]

Wenn Sie die Azure PowerShell- oder Azure CLI-Beispiele in diesem Artikel verwenden möchten, müssen Sie die neueste Version von [Azure PowerShell](https://www.powershellgallery.com/packages/AzureRM) oder [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli) installieren. 

> [!IMPORTANT]
> - Bei allen Beispielskripts in diesem Artikel wird vorausgesetzt, dass der Befehlszeilenclient auf einem MSI-fähigen virtuellen Computer ausgeführt wird. Verwenden Sie die Funktion „Verbinden“ im Azure-Portal zum Herstellen einer Remoteverbindung mit Ihrem virtuellen Computer. Weitere Informationen zur Aktivierung von MSI auf einer VM finden Sie unter [Konfigurieren einer VM-MSI (Managed Service Identity, verwaltete Dienstidentität) über das Azure-Portal](msi-qs-configure-portal-windows-vm.md) oder in einem der verwandten Artikel (PowerShell, CLI, Vorlage oder Azure SDK). 
> - Um Fehler beim Zugriff auf Ressourcen zu verhindern, muss die MSI des virtuellen Computers mindestens über den Zugriff „Leser“ im entsprechenden Bereich (virtueller Computer oder höher) verfügen, um Azure Resource Manager-Vorgänge auf dem virtuellen Computer zu ermöglichen. Weitere Informationen finden Sie unter [Zuweisen des MSI-Zugriffs (Managed Service Identity, verwaltete Dienstidentität) auf eine Ressource über das Azure-Portal](msi-howto-assign-access-portal.md).

## <a name="overview"></a>Übersicht

Eine MSI enthält einen [Dienstprinzipal](develop/active-directory-dev-glossary.md#service-principal-object), der [beim Aktivieren von MSI](msi-overview.md#how-does-it-work) auf dem virtuellen Computer erstellt wird. Der Dienstprinzipal kann Zugriff auf Azure-Ressourcen erhalten und von Skript- oder Befehlszeilenclients als Identität für die Anmeldung und den Zugriff auf Ressourcen verwendet werden. Bislang war für einen Skriptclient für den Zugriff auf geschützte Ressourcen unter seiner eigenen Identität Folgendes erforderlich:  

   - Registrierung und Zustimmung als vertrauliche und Webclientanwendung in Azure AD
   - Anmeldung unter dem zugehörigen Dienstprinzipal mit den Anmeldeinformationen der Anwendung (wahrscheinlich im Skript eingebettet)

Mit MSI ist für den Skriptclient beides nicht mehr erforderlich, da er unter dem MSI-Dienstprinzipal angemeldet werden kann. 

## <a name="azure-cli"></a>Azure-Befehlszeilenschnittstelle

Mit diesem Skript wird Folgendes veranschaulicht:

1. Anmelden bei Azure AD unter dem MSI-Dienstprinzipal des virtuellen Computers  
2. Aufrufen von Azure Resource Manager und Abrufen der Dienstprinzipal-ID des virtuellen Computers. Die Verwaltung des Tokenabrufs und der Tokenverwendung erfolgt automatisch über die Befehlszeilenschnittstelle. Achten Sie darauf, dass Sie `<VM-NAME>` durch den Namen Ihres virtuellen Computers ersetzen.  

   ```azurecli
   az login --msi
   
   spID=$(az resource list -n <VM-NAME> --query [*].identity.principalId --out tsv)
   echo The MSI service principal ID is $spID
   ```

## <a name="azure-powershell"></a>Azure PowerShell

Mit diesem Skript wird Folgendes veranschaulicht:

1. Abrufen eines MSI-Zugriffstokens für den virtuellen Azure-Computer  
2. Verwenden des Zugriffstokens zum Anmelden bei Azure AD unter dem entsprechenden MSI-Dienstprinzipal   
3. Aufrufen eines Azure Resource Manager-Cmdlets zum Abrufen von Informationen zu dem virtuellen Computer. Die Verwaltung der Tokenverwendung erfolgt automatisch über PowerShell.  

   ```azurepowershell
   # Get an access token for the MSI
   $response = Invoke-WebRequest -Uri http://localhost:50342/oauth2/token `
                                 -Method GET -Body @{resource="https://management.azure.com/"} -Headers @{Metadata="true"}
   $content =$response.Content | ConvertFrom-Json
   $access_token = $content.access_token
   echo "The MSI access token is $access_token"

   # Use the access token to sign in under the MSI service principal. -AccountID can be any string to identify the session.
   Login-AzureRmAccount -AccessToken $access_token -AccountId "MSI@50342"

   # Call Azure Resource Manager to get the service principal ID for the VM's MSI. 
   $vmInfoPs = Get-AzureRMVM -ResourceGroupName <RESOURCE-GROUP> -Name <VM-NAME>
   $spID = $vmInfoPs.Identity.PrincipalId
   echo "The MSI service principal ID is $spID"
   ```

## <a name="resource-ids-for-azure-services"></a>Ressourcen-IDs für Azure-Dienste

Eine Liste mit Ressourcen, die Azure AD unterstützen und mit MSI getestet wurden, und die jeweiligen zugehörigen Ressourcen-IDs finden Sie unter [Azure-Dienste, die die Azure AD-Authentifizierung unterstützen](msi-overview.md#azure-services-that-support-azure-ad-authentication).

## <a name="error-handling-guidance"></a>Anleitung zur Fehlerbehandlung 

Antworten wie beispielsweise die folgenden können angeben, dass die MSI des virtuellen Computers nicht ordnungsgemäß konfiguriert wurde:

- PowerShell: *Invoke-WebRequest: Es konnte keine Verbindung mit dem Remoteserver hergestellt werden.*
- CLI: *MSI: Failed to retrieve a token from 'http://localhost:50342/oauth2/token' with an error of 'HTTPConnectionPool(host='localhost', port=50342)* [MSI: Fehler beim Abrufen eines Tokens aus 'http://localhost:50342/oauth2/token' mit dem Fehler 'HTTPConnectionPool(host='localhost', port=50342)] 

Wenn einer dieser Fehler angezeigt wird, wechseln Sie zu dem virtuellen Azure-Computer im [Azure-Portal](https://portal.azure.com) und:

- Stellen Sie auf der Seite **Konfiguration** sicher, dass „Verwaltete Dienstidentität“ auf „Ja“ festgelegt ist.
- Vergewissern Sie sich auf der Seite **Erweiterungen**, dass die MSI-Erweiterung erfolgreich bereitgestellt wurde.

Wenn einer dieser Punkte nicht zutrifft, müssen Sie die MSI möglicherweise erneut für die Ressource bereitstellen oder den Fehler bei der Bereitstellung beheben. Hilfe zur Konfiguration des virtuellen Computers finden Sie unter [Konfigurieren einer VM-MSI (Managed Service Identity, verwaltete Dienstidentität) über das Azure-Portal](msi-qs-configure-portal-windows-vm.md).

## <a name="related-content"></a>Verwandte Inhalte

- Informationen zum Aktivieren von MSI auf einem virtuellen Azure-Computer finden Sie unter [Konfigurieren einer VM-MSI (Managed Service Identity, verwaltete Dienstidentität) mit PowerShell](msi-qs-configure-powershell-windows-vm.md) oder [Konfigurieren einer VM-MSI (Managed Service Identity, verwaltete Dienstidentität) mit der Azure CLI](msi-qs-configure-cli-windows-vm.md).

Verwenden Sie den folgenden Kommentarabschnitt, um uns Feedback zu senden und uns bei der Verbesserung unserer Inhalte zu unterstützen.








