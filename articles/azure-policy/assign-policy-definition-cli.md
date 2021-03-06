---
title: "Verwenden der Azure-Befehlszeilenschnittstelle für die Erstellung einer Richtlinienzuweisung zum Identifizieren nicht kompatibler Ressourcen in Ihrer Azure-Umgebung | Microsoft-Dokumentation"
description: Erstellen Sie mithilfe von PowerShell eine Azure Policy-Zuweisung zum Identifizieren nicht kompatibler Ressourcen.
services: azure-policy
keywords: 
author: bandersmsft
ms.author: banders
ms.date: 12/06/2017
ms.topic: quickstart
ms.service: azure-policy
ms.custom: mvc
ms.openlocfilehash: 88ceb47d46b66e716c6c263098d5b9458e4aff22
ms.sourcegitcommit: cc03e42cffdec775515f489fa8e02edd35fd83dc
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/07/2017
---
# <a name="create-a-policy-assignment-to-identify-non-compliant-resources-in-your-azure-environment-with-the-azure-cli"></a>Erstellen einer Richtlinienzuweisung zum Identifizieren nicht kompatibler Ressourcen in Ihrer Azure-Umgebung mit der Azure-Befehlszeilenschnittstelle

Zum Verständnis der Konformität in Azure müssen Sie zunächst wissen, wo Sie derzeit mit Ihren eigenen Ressourcen stehen. Diese Schnellstartanleitung führt Sie schrittweise durch die Erstellung einer Richtlinienzuweisung zur Identifizierung von virtuellen Computern, die keine verwalteten Datenträger verwenden.

Am Ende dieses Prozesses haben Sie erfolgreich identifiziert, welche virtuellen Computer keine verwalteten Datenträger verwenden und somit *nicht konform* sind.

Wenn Sie kein Azure-Abonnement besitzen, können Sie ein [kostenloses Konto](https://azure.microsoft.com/free/) erstellen, bevor Sie beginnen.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Wenn Sie die CLI lokal installieren und verwenden möchten, müssen Sie für diesen Schnellstart die Azure CLI-Version 2.0.4 oder höher ausführen. Führen Sie `az --version` aus, um die Version zu finden. Wenn Sie eine Installation oder ein Upgrade ausführen müssen, finden Sie unter [Installieren von Azure CLI 2.0]( /cli/azure/install-azure-cli) Informationen dazu.

## <a name="create-a-policy-assignment"></a>Erstellen einer Richtlinienzuweisung

In dieser Schnellstartanleitung erstellen wir eine Richtlinienzuweisung und weisen die Definition „Audit Virtual Machines without Managed Disks“ (Virtuelle Computer ohne verwaltete Datenträger überwachen) zu. Mit dieser Richtliniendefinition werden Ressourcen identifiziert, die die in der Richtliniendefinition festgelegten Bedingungen nicht erfüllen.

Führen Sie die folgenden Schritte aus, um eine neue Richtlinienzuweisung zu erstellen.

Zeigen Sie alle Richtliniendefinitionen an, und suchen Sie die Richtliniendefinition „Audit Virtual Machines without Managed Disks“ (Virtuelle Computer ohne verwaltete Datenträger überwachen):

```azurecli
az policy definition list
```

Azure Policy umfasst bereits erstellte Richtliniendefinitionen, die Sie verwenden können. Ihnen werden integrierte Richtliniendefinitionen wie die folgenden angezeigt:

- Enforce tag and its value (Tag und zugehörigen Wert erzwingen)
- Apply tag and its value (Tag und zugehörigen Wert anwenden)
- Require SQL Server Version 12.0 (SQL Server-Version 12.0 erforderlich)

Geben Sie als Nächstes folgende Informationen an, und führen Sie folgenden Befehl zum Zuweisen der Richtliniendefinition aus:

- **Anzeigename** für die Richtlinienzuweisung. In diesem Fall verwenden wir *Audit Virtual Machines without Managed Disks* (Virtuelle Computer ohne verwaltete Datenträger überwachen).
- **Richtlinie**: Die Richtliniendefinition, auf deren Grundlage Sie die Zuweisung erstellen. In diesem Fall ist es die Richtliniendefinition: *Audit Virtual Machines without Managed Disks* (Virtuelle Computer ohne verwaltete Datenträger überwachen).
- **Bereich**: Ein Bereich bestimmt, für welche Ressourcen oder Ressourcengruppe die Richtlinienzuweisung erzwungen wird. Er kann von einem Abonnement bis zu Ressourcengruppen reichen.

  Verwenden Sie das zuvor registrierte Abonnement (oder die Ressourcengruppe). In diesem Beispiel verwenden wir die Abonnement-ID **bc75htn-a0fhsi-349b-56gh-4fghti-f84852** und **FabrikamOMS** für den Namen der Ressourcengruppe. Stellen Sie sicher, dass Sie diese in die Abonnement-ID und den Namen der Ressourcengruppe ändern, mit denen Sie arbeiten.

Der Befehl sollte wie folgt aussehen:

```azurecli
az policy assignment create --name Audit Virtual Machines without Managed Disks Assignment --policy Audit Virtual Machines without Managed Disks --scope /subscriptions/
bc75htn-a0fhsi-349b-56gh-4fghti-f84852/resourceGroups/FabrikamOMS
```

Eine Richtlinienzuweisung ist eine zugewiesene Richtlinie, die innerhalb eines bestimmten Bereichs angewendet werden soll. Ein solcher Bereich kann sich von einer Verwaltungsgruppe bis hin zu einer Ressourcengruppe erstrecken.

## <a name="identify-non-compliant-resources"></a>Identifizieren nicht kompatibler Ressourcen

So zeigen Sie die Ressourcen an, die in dieser neuen Zuweisung nicht kompatibel sind:

1. Navigieren Sie zurück zur Azure Policy-Seite.
2. Wählen Sie im linken Bereich **Konformität** aus, und suchen Sie nach der von Ihnen erstellten **Richtlinienzuweisung**.

   ![Richtlinienkonformität](media/assign-policy-definition/policy-compliance.png)

   Wenn Ressourcen vorhanden sind, die mit dieser neuen Zuweisung nicht kompatibel sind, werden diese wie oben dargestellt auf der Registerkarte **Non-compliant resources** (Nicht kompatible Ressourcen) angezeigt.

## <a name="clean-up-resources"></a>Bereinigen von Ressourcen

Andere Anleitungen in dieser Sammlung bauen auf diesem Schnellstart auf. Wenn Sie mit den nachfolgenden Tutorials fortfahren möchten, sollten Sie die in diesem Schnellstart erstellten Ressourcen nicht bereinigen. Wenn Sie nicht fortfahren möchten, löschen Sie die erstellte Zuweisung mit dem folgenden Befehl:

```azurecli
az policy assignment delete –name  Assignment --scope /subscriptions/ bc75htn-a0fhsi-349b-56gh-4fghti-f84852 resourceGroups/ FabrikamOMS
```

## <a name="next-steps"></a>Nächste Schritte

In diesem Schnellstart haben Sie eine Richtliniendefinition zum Identifizieren nicht kompatibler Ressourcen in Ihrer Azure-Umgebung zugewiesen.

Wenn Sie mehr über das Zuweisen von Richtlinien erfahren möchten, um sicherzustellen, dass **zukünftige** von Ihnen erstellte Ressourcen kompatibel sind, finden Sie im folgenden Tutorial weitere Informationen:

> [!div class="nextstepaction"]
> [Erstellen und Verwalten von Richtlinien](./create-manage-policy.md)
