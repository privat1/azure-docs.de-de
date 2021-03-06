---
title: Replizieren von Anwendungen (Azure zu Azure) | Microsoft-Dokumentation
description: "Dieser Artikel beschreibt, wie Sie die Replikation von virtuellen Computern, die in einer Azure-Region ausgeführt werden, in einer anderen Azure-Region einrichten."
services: site-recovery
documentationcenter: 
author: asgang
manager: rochakm
editor: 
ms.assetid: 
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 12/08/2017
ms.author: asgang
ms.openlocfilehash: 209ec47388ee7291f8107df022e0c2bb202ba6b5
ms.sourcegitcommit: 094061b19b0a707eace42ae47f39d7a666364d58
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/08/2017
---
# <a name="replicate-azure-virtual-machines-to-another-azure-region"></a>Replizieren von virtuellen Azure-Computern in einer anderen Azure-Region



>[!NOTE]
>
> Die Site Recovery-Replikation für virtuelle Azure-Computer ist derzeit als Vorschauversion verfügbar.

Dieser Artikel beschreibt, wie Sie die Replikation von virtuellen Computern, die in einer Azure-Region ausgeführt werden, in einer anderen Azure-Region einrichten.

## <a name="prerequisites"></a>Voraussetzungen

* In diesem Artikel wird davon ausgegangen, dass Sie bereits mit Site Recovery und dem Recovery Services-Tresor vertraut sind. Sie müssen bereits einen Recovery Services-Tresor erstellt haben.

    >[!NOTE]
    >
    > Es wird empfohlen, dass Sie den Recovery Services-Tresor an dem Speicherort erstellen, an dem Ihre virtuellen Computer repliziert werden sollen. Wenn Ihr Zielspeicherort beispielsweise „USA, Mitte“ ist, erstellen Sie den Tresor „USA, Mitte“.

* Wenn Sie NSG-Regeln (Netzwerksicherheitsgruppen) oder einen Firewallproxy verwenden, um den Zugriff für ausgehende Internetverbindungen auf den Azure-VMs zu steuern, stellen Sie sicher, dass Sie die erforderlichen URLs oder IP-Adressen in die Whiteliste aufnehmen. Weitere Informationen finden Sie unter [Richtliniendokument für Netzwerke](./site-recovery-azure-to-azure-networking-guidance.md).

* Wenn eine ExpressRoute- oder VPN-Verbindung zwischen dem lokalen Speicherort und dem Quellspeicherort in Azure vorliegt, folgen Sie den Informationen im Dokument [Überlegungen zur Site Recovery für Azure bei einer lokalen ExpressRoute-/VPN-Konfiguration](site-recovery-azure-to-azure-networking-guidance.md#guidelines-for-existing-azure-to-on-premises-expressroutevpn-configuration).

* Ihr Azure-Benutzerkonto benötigt bestimmte [Berechtigungen](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines) zum Aktivieren der Replikation eines virtuellen Azure-Computers.

* Ihr Azure-Abonnement sollte zum Erstellen von virtuellen Computer an dem Zielspeicherort aktiviert sein, den Sie als Region für die Notfallwiederherstellung verwenden möchten. Sie können sich an den Support wenden, um das erforderliche Kontingent zu aktivieren.

## <a name="enable-replication-from-azure-site-recovery-vault"></a>Aktivieren der Replikation über den Azure Site Recovery-Tresor
Für diese Veranschaulichung werden virtuelle Computer, die am Standort „Asien, Osten“ ausgeführt werden, am Standort „Asien, Südosten“ repliziert. Die Schritte lauten wie folgt:

 Klicken Sie im Tresor auf **+Replizieren**, um die Replikation für die virtuellen Computer zu aktivieren.

1. **Quelle:** Dies bezieht sich auf den Ursprungsort der Computer, in diesem Fall also **Azure**.

2. **Quellstandort:** Dies ist die Azure-Region, aus der Sie Ihre virtuellen Computer schützen möchten. In diesem Beispiel ist der Quellspeicherort „Asien, Osten“.

3. **Bereitstellungsmodell:** Dies bezieht sich auf das Azure-Bereitstellungsmodell der Quellcomputer. Sie können entweder das klassische oder das Resource Manager-Modell auswählen. Die Computer, die zu dem jeweiligen Modell gehören, werden im nächsten Schritt zum Einrichten des Schutzes aufgeführt.

      >[!NOTE]
      >
      > Sie können einen replizierten klassischen virtuellen Computer nur als klassischen virtuellen Computer wiederherstellen. Eine Wiederherstellung als virtueller Ressourcen-Manager-Computer ist nicht möglich.

4. **Ressourcengruppe:** Dies ist die Ressourcengruppe, zu der Ihre virtuellen Quellcomputer gehören. Alle virtuellen Computer der ausgewählten Ressourcengruppe werden im nächsten Schritt für den Schutz aufgeführt.

    ![Replikation aktivieren](./media/site-recovery-replicate-azure-to-azure/enabledrwizard1.png)

Klicken Sie auf **Virtuelle Computer > Virtuelle Computer auswählen**, und wählen Sie die Computer aus, die Sie replizieren möchten. Sie können nur Computer auswählen, für die die Replikation aktiviert werden kann. Klicken Sie dann auf „OK“.
    ![Replikation aktivieren](./media/site-recovery-replicate-azure-to-azure/virtualmachine_selection.png)


Im Abschnitt „Einstellungen“ können Sie die Eigenschaften des Zielspeicherorts konfigurieren.

1. **Zielstandort:** Dies ist der Speicherort, am dem die Daten der virtuellen Quellcomputer repliziert werden. Abhängig vom ausgewählten Computerstandort, stellt Site Recovery eine Liste der geeigneten Zielregionen bereit.

    > [!TIP]
    > Es wird empfohlen, den Zielspeicherort des Recovery Services-Tresors zu verwenden.

2. **Zielressourcengruppe:** Dies ist die Ressourcengruppe, zu der alle replizierten virtuellen Computer gehören. Azure Site Recovery erstellt standardmäßig in der Zielregion eine neue Ressourcengruppe, deren Name das Suffix „asr“ aufweist. Falls bereits eine von Azure Site Recovery erstellte Ressourcengruppe vorhanden ist, wird diese wiederverwendet. Sie können die Gruppe auch anpassen, wie im Abschnitt unten gezeigt.    
3. **Virtuelles Zielnetzwerk:** Standardmäßig erstellt Azure Site Recovery in der Zielregion ein neues virtuelles Netzwerk, dessen Name das Suffix „asr“ aufweist. Dieses wird Ihrem Quellnetzwerk zugeordnet und für alle zukünftigen Schutzaktivitäten verwendet werden.

    > [!NOTE]
    > [Überprüfen Sie die Netzwerkdetails](site-recovery-network-mapping-azure-to-azure.md), um weitere Informationen zur Netzwerkzuordnung zu erhalten.

4. **Zielspeicherkonten:** Standardmäßig erstellt Azure Site Recovery ein neues Zielspeicherkonto, wobei die Konfiguration Ihres VM-Quellspeichers kopiert wird. Falls bereits ein von Azure Site Recovery erstelltes Speicherkonto vorhanden ist, wird dieses wiederverwendet.

5. **Cachespeicherkonten:** Azure Site Recovery benötigt als zusätzliches Speicherkonto ein sogenanntes Cachespeicherkonto in der Quellregion. Alle Änderungen an den virtuellen Quellcomputern werden nachverfolgt und vor der Replikation dieser Computer am Zielspeicherort an das Cachespeicherkonto gesendet.

6. **Verfügbarkeitsgruppe:** Standardmäßig erstellt Azure Site Recovery in der Zielregion eine neue Verfügbarkeitsgruppe, deren Name das Suffix „asr“ aufweist. Falls bereits eine von Azure Site Recovery erstellte Verfügbarkeitsgruppe vorhanden ist, wird diese wiederverwendet.

7.  **Replikationsrichtlinie:** Diese definiert die Einstellungen für den Aufbewahrungsverlauf des Wiederherstellungspunkts und die App-konsistente Momentaufnahmenhäufigkeit. Standardmäßig erstellt Azure Site Recovery eine neue Replikationsrichtlinie mit der Standardeinstellung „24 Stunden“ für den Aufbewahrungszeitraum des Wiederherstellungspunkts und „60 Minuten“ für die App-konsistente Momentaufnahmenhäufigkeit.

    ![Replikation aktivieren](./media/site-recovery-replicate-azure-to-azure/enabledrwizard3.PNG)

## <a name="customize-target-resources"></a>Anpassen der Zielressourcen

Sie können die von Azure Site Recovery verwendeten Standardeinstellungen nach Bedarf ändern.

1. **Anpassen:** Klicken Sie auf diese Option, um die von Azure Site Recovery verwendeten Standardeinstellungen zu ändern.

2. **Zielressourcengruppe:** Sie können die Ressourcengruppe aus der Liste aller Ressourcengruppen auswählen, die innerhalb des Abonnements am Zielspeicherort vorhanden sind.

3. **Virtuelles Zielnetzwerk:** Hier finden Sie die Liste aller virtuellen Netzwerke am Zielspeicherort.

4. **Verfügbarkeitsgruppe:** Sie können Einstellungen für Verfügbarkeitsgruppen nur zu den virtuellen Computern hinzufügen, die der Verfügbarkeitsgruppe in der Quellregion angehören.

5. **Zielspeicherkonto:**

![Replikation aktivieren](./media/site-recovery-replicate-azure-to-azure/customize.PNG) Klicken Sie auf **Create target resource** (Zielressource erstellen) und auf „Replikation aktivieren“.


Nachdem die virtuellen Computer geschützt wurden, können Sie deren Integrität unter **Replizierte Elemente** überprüfen.

>[!NOTE]
>Im Verlauf der ersten Replikation besteht die Möglichkeit, dass die Statusaktualisierung lange und vorübergehend kein Fortschritt angezeigt wird. Sie können oben auf dem Blatt auf die Schaltfläche „Aktualisieren“ klicken, um den neuesten Status abzurufen.
>

![Replikation aktivieren](./media/site-recovery-replicate-azure-to-azure/replicateditems.PNG)


## <a name="next-steps"></a>Nächste Schritte
- [Erfahren Sie mehr](site-recovery-test-failover-to-azure.md) über die Ausführung von Testfailovervorgängen.
- [Informieren Sie sich](site-recovery-failover.md) über die unterschiedlichen Failoverarten und deren Durchführung.
- Erfahren Sie mehr über die [Verwendung von Wiederherstellungsplänen](site-recovery-create-recovery-plans.md) zur RTO-Verringerung.
- Erfahren Sie mehr über das [erneute Schützen von virtuellen Azure-Computern](site-recovery-how-to-reprotect.md) nach einem Failover.
