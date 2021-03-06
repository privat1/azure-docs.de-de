---
title: "Entscheidungen zu mit Azure verbundenen Bereitstellungen für in Azure Stack integrierte Systeme | Microsoft-Dokumentation"
description: "Erfahren Sie, wie Sie Entscheidungen bei der Bereitstellungsplanung für mit Azure verbundene Azure Stack-Bereitstellungen mit mehreren Knoten treffen."
services: azure-stack
documentationcenter: 
author: jeffgilb
manager: femila
editor: 
ms.assetid: 
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/16/2018
ms.author: jeffgilb
ms.reviewer: wfayed
ms.openlocfilehash: c1a3b2107abdc3ef19a314616518c494687d81bf
ms.sourcegitcommit: 5108f637c457a276fffcf2b8b332a67774b05981
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/17/2018
---
# <a name="azure-connected-deployment-planning-decisions-for-azure-stack-integrated-systems"></a>Planungsentscheidungen zu mit Azure verbundenen Bereitstellungen für in Azure Stack integrierte Systeme
Nachdem Sie entschieden haben, [wie Sie Azure Stack in Ihre Hybrid Cloud-Umgebung integrieren](azure-stack-deployment-decisions.md), können Sie Ihre Azure Stack-Bereitstellungsentscheidungen abschließen.

Die Bereitstellung von mit Azure verbundenem Azure Stack bedeutet, dass Sie entweder Azure Active Directory (Azure AD) oder Active Directory-Verbunddienste (AD FS) als Identitätsspeicher verwenden können. Sie können auch ein Abrechnungsmodell wählen: nutzungsabhängig oder kapazitätsabhängig. Eine mit Azure verbundene Bereitstellung ist die Standardoption, da sie Kunden ermöglicht, den größtmöglichen Nutzen aus Azure Stack zu ziehen, insbesondere bei Hybrid Cloud-Szenarien, die sowohl Azure als auch Azure Stack umfassen. 

## <a name="choose-an-identity-store"></a>Wählen eines Identitätsspeichers
Bei einer mit Azure verbundenen Bereitstellung haben Sie die Wahl zwischen Azure AD oder AD FS als Identitätsspeicher. Bei einer nicht mit Azure verbundenen Bereitstellung (ohne Internetverbindung) kann nur AD FS verwendet werden.

Ihre Wahl des Identitätsspeichers hat keinen Einfluss auf die virtuellen Computer (VMs) der Mandanten. Die VMs der Mandanten können wählen, mit welchem Identitätsspeicher sie die Verbindung herstellen möchten, was davon abhängt, wie sie konfiguriert werden: Azure AD, in Windows Server Active Directory-Domäne eingebunden, Arbeitsgruppe usw. Dies ist unabhängig von der Entscheidung zum Azure Stack-Identitätsanbieter. 

Wenn Sie z.B. VMs von IaaS-Mandanten auf Azure Stack aufsetzend bereitstellen und möchten, dass sie in eine Active Directory-Domäne des Unternehmens eingebunden werden und die Konten in dieser verwenden, können Sie dies nach wie vor tun. Sie sind nicht verpflichtet, den Azure AD-Identitätsspeicher, den Sie hier auswählen, für diese Konten zu verwenden.

### <a name="azure-ad-identity-store"></a>Azure AD-Identitätsspeicher
Bei Verwendung von Azure AD als Identitätsspeicher werden zwei Azure AD-Konten benötigt: ein globales Administratorkonto und ein Abrechnungskonto. Diese Konten können entweder dasselbe oder verschiedene Konten sein. Während die Verwendung desselben Benutzerkontos einfacher und praktischer sein kann, wenn Sie nur eine begrenzte Anzahl von Azure-Konten haben, können Ihre geschäftlichen Anforderungen die Verwendung zweier Konten nahelegen:

1. **Globales Administratorkonto** (nur für mit Azure verbundene Bereitstellungen erforderlich). Hierbei handelt es sich um ein Azure-Konto, das zum Erstellen von Anwendungen und Dienstprinzipalen für Azure Stack-Infrastrukturdienste in Azure Active Directory verwendet wird. Dieses Konto muss über Administratorrechte für das Verzeichnis verfügen, in dem Ihr Azure Stack-System installiert werden soll. Es wird zum globalen Administratorkonto des „Cloudbetreibers“ für den Azure AD-Mandanten und für die folgenden Aufgaben verwendet: 
    - Bereitstellen und Delegieren von Anwendungen und Dienstprinzipalen für alle Azure Stack-Dienste, die mit Azure Active Directory und Graph-API interagieren. 
    - Als Dienstadministratorkonto. Dies ist der Besitzer des Standardanbieterabonnements (den Sie später jedoch ändern können). Mit diesem Konto können Sie sich beim Azure Stack-Verwaltungsportal anmelden und damit Angebote und Pläne erstellen, Kontingente festlegen und andere administrative Aufgaben in Azure Stack erledigen.
2. **Abrechnungskonto** (sowohl für mit Azure verbundene als auch nicht verbundene Bereitstellungen erforderlich). Dieses Azure-Konto wird verwendet, um die Abrechnungsbeziehung zwischen Ihrem in Azure Stack integrierten System und dem Azure-Commerce-Back-End herzustellen. Dies ist das Konto, dem Azure Stack-Gebühren in Rechnung gestellt werden. Dieses Konto wird auch für die Marketplace-Syndikation und andere Hybridszenarien verwendet. 

### <a name="ad-fs-identity-store"></a>AD FS-IDENTITÄTSSPEICHER
Wählen Sie diese Option, wenn Sie Ihren eigenen Identitätsspeicher, wie z.B. Ihr unternehmenseigenes Active Directory, für Ihre Dienstadministratorkonten verwenden möchten.  

## <a name="choose-a-billing-model"></a>Wählen eines Abrechnungsmodells
Als Abrechnungsmodell können Sie entweder **Nutzungsbasiert** oder **Kapazität** wählen. Bereitstellungen mit nutzungsbasiertem Abrechnungsmodell müssen in der Lage sein, die Nutzung über eine Verbindung mit Azure mindestens einmal alle 30 Tage zu melden. Wenn keine Verbindung verfügbar ist, ist daher das kapazitätsbasierte Abrechnungsmodell die einzige Option. 

### <a name="pay-as-you-use"></a>Nutzungsbasierte Abrechnung
Beim nutzungsbasierten Abrechnungsmodell wird die Nutzung einem Azure-Abonnement in Rechnung gestellt. Sie zahlen nur, wenn Sie die Azure Stack-Dienste verwenden. Wenn Sie sich für dieses Modell entscheiden, benötigen Sie ein Azure-Abonnement und die Konto-ID, die diesem Abonnement zugeordnet ist (z.B. serviceadmin@contoso.onmicrosoft.com). EA-, CSP- und CSL-Abonnements werden unterstützt. Die Erstellung von Nutzungsberichten wird während der [Azure Stack-Registrierung](azure-stack-registration.md) konfiguriert.

> [!NOTE]
> In den meisten Fällen werden EA-Abonnements von Enterprise-Kunden und CSP- oder CSL-Abonnements von Dienstanbietern verwendet.

Wenn Sie ein CSP-Abonnement verwenden möchten, konsultieren Sie die folgende Tabelle, um herauszufinden, welches CSP-Abonnement Sie verwenden sollten, da der richtige Ansatz vom jeweiligen CSP-Szenario abhängt:

|Szenario|Domänen- und Abonnementoptionen|
|-----|-----|
|Sie sind direkter oder indirekter CSL-Partner und betreiben den Azure Stack|Verwenden Sie ein CSL-Abonnement (Common Service Layer).|
|Sie sind direkter oder indirekter CSP-Partner und betreiben den Azure Stack|Erstellen Sie im Partner Center einen Azure AD-Mandanten mit einem aussagekräftigen Namen, z. B. <your organization>CSPAdmin, und einem damit verbundenen Azure CSP-Abonnement.|
|Sie sind indirekter CSP-Wiederverkäufer und betreiben den Azure Stack|Bitten Sie Ihren indirekten CSP-Anbieter im Partner Center einen Azure AD-Mandanten für Ihr Unternehmen und ein damit verbundenes Azure CSP-Abonnement zu erstellen.|

### <a name="capacity-based-billing"></a>Kapazitätsbasierte Abrechnung
Wenn Sie sich für das kapazitätsabhängige Abrechnungsmodells entscheiden, müssen Sie eine Azure Stack-Kapazitätsplan-SKU kaufen, die auf der Kapazität Ihres Systems basiert. Sie müssen die Anzahl der physischen Kerne in Ihrem Azure Stack kennen, um die richtige Menge zu kaufen. 

Für die kapazitätsbezogene Abrechnung ist für die Registrierung ein Azure-Abonnement des Typs EA (Enterprise Agreement) erforderlich. Der Grund dafür ist, dass die Registrierung eine Syndikation einrichtet, die ein Azure-Abonnement erfordert. Das Abonnement wird nicht zur Azure Stack-Nutzung verwendet.

## <a name="learn-more"></a>Weitere Informationen
- Informationen zu Anwendungsfällen, zum Erwerb, zu Partnern und zu OEM-Hardwareanbietern finden Sie auf der Produktseite von [Azure Stack](https://azure.microsoft.com/overview/azure-stack/).
- Informationen zur Roadmap und geografischen Verfügbarkeit für integrierte Azure Stack-Systeme finden Sie im Whitepaper: [Azure Stack: Eine Erweiterung von Azure](https://azure.microsoft.com/resources/azure-stack-an-extension-of-azure/). 
- Weitere Informationen zu Microsoft Azure Stack-Paketen und -Preisen finden Sie im Datenblatt zu [Paketerstellung und Preisen](https://azure.microsoft.com/mediahandler/files/resourcefiles/5bc3f30c-cd57-4513-989e-056325eb95e1/Azure-Stack-packaging-and-pricing-datasheet.pdf). 
