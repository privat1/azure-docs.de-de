---
title: "Azure CLI-Skriptbeispiel – Erstellen einer Windows Server 2016-VM mit NLB | Microsoft-Dokumentation"
description: "Azure CLI-Skriptbeispiel – Erstellen einer Windows Server 2016-VM mit NLB"
services: virtual-machines-Windows
documentationcenter: virtual-machines
author: rickstercdn
manager: timlt
editor: tysonn
tags: 
ms.assetid: 
ms.service: virtual-machines-Windows
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/23/2017
ms.author: rclaus
ms.openlocfilehash: 9662e9a2ea128b609b76edfc4ec7b57016909761
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 10/11/2017
---
# <a name="load-balance-traffic-between-highly-available-virtual-machines"></a>Lastenausgleich für den Datenverkehr zwischen hoch verfügbaren virtuellen Computern

Dieses Beispielskript erstellt alle Komponenten, die zum Ausführen mehrerer Ubuntu-VMs in einer Konfiguration mit hoher Verfügbarkeit und Lastenausgleich benötigt werden. Nach dem Ausführen dieses Skripts verfügen Sie über drei virtuelle Computer, die in einer Azure-Verfügbarkeitsgruppe zusammengefasst und über eine Azure Load Balancer-Instanz zugänglich sind.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Beispielskript

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/create-vm-nlb/create-windows-vm-nlb.sh "Quick Create VM")]

## <a name="clean-up-deployment"></a>Bereinigen der Bereitstellung 

Führen Sie den folgenden Befehl aus, um die Ressourcengruppe, den virtuellen Computer und alle zugehörigen Ressourcen zu entfernen.

```azurecli-interactive 
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>Erläuterung des Skripts

In diesem Skript werden die folgenden Befehle verwendet, um eine Ressourcengruppe, einen virtuellen Computer, eine Verfügbarkeitsgruppe, einen Load Balancer und alle zugehörigen Ressourcen zu erstellen. Jeder Befehl in der Tabelle ist mit der zugehörigen Dokumentation verknüpft.

| Befehl | Hinweise |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az_group_create) | Erstellt eine Ressourcengruppe, in der alle Ressourcen gespeichert sind. |
| [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet#az_network_vnet_create) | Erstellt ein virtuelles Azure-Netzwerk und ein Subnetz. |
| [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip#az_network_public_ip_create) | Erstellt eine öffentliche IP-Adresse mit einer statischen IP-Adresse und zugeordnetem DNS-Namen. |
| [az network lb create](https://docs.microsoft.com/cli/azure/network/lb#az_network_lb_create) | Erstellt einen Azure Network Load Balancer (NLB). |
| [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe#az_network_lb_probe_create) | Erstellt einen NLB-Test. Mithilfe eines NLB-Tests wird jede VM in der NLB-Gruppe überwacht. Falls eine VM nicht verfügbar ist, wird der Datenverkehr nicht an diese VM geroutet. |
| [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule#az_network_lb_rule_create) | Erstellt eine NLB-Regel. In diesem Beispiel wird eine Regel für den Port 80 erstellt. Wenn HTTP-Datenverkehr beim NLB eingeht, wird dieser an Port 80 einer der VMs in der NLB-Gruppe geroutet. |
| [az network lb inbound-nat-rule create](https://docs.microsoft.com/cli/azure/network/lb/inbound-nat-rule#az_network_lb_inbound_nat_rule_create) | Erstellt eine NLB-Regel für die Netzwerkadressübersetzung (NAT).  Mithilfe von NAT-Regeln wird ein NLB-Port einem Port auf einer VM zugeordnet. In diesem Beispiel wird eine NAT-Regel für den SSH-Datenverkehr an jede VM in der NLB-Gruppe erstellt.  |
| [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg#az_network_nsg_create) | Erstellt eine Netzwerksicherheitsgruppe (NSG), die als Sicherheitsgrenze zwischen dem Internet und dem virtuellen Computer fungiert. |
| [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule#az_network_nsg_rule_create) | Erstellt eine NSG-Regel zum Zulassen von eingehendem Datenverkehr. In diesem Beispiel wird Port 22 für SSH-Datenverkehr geöffnet. |
| [az network nic create](https://docs.microsoft.com/cli/azure/network/nic#az_network_nic_create) | Erstellt eine virtuelle Netzwerkkarte und verbindet diese mit dem virtuellen Netzwerk, dem Subnetz und der NSG. |
| [az vm availability-set create](https://docs.microsoft.com/cli/azure/network/lb/rule#az_network_lb_rule_create) | Erstellt eine Verfügbarkeitsgruppe. Verfügbarkeitsgruppen stellen die Verfügbarkeit von Anwendungen sicher, indem sie die virtuellen Computer auf physische Ressourcen verteilen. So wirken sich Ausfälle nicht auf die gesamte Gruppe aus. |
| [az vm create](https://docs.microsoft.com/cli/azure/vm/availability-set#az_vm_availability_set_create) | Erstellt den virtuellen Computer und verbindet diesen mit der Netzwerkkarte, dem virtuellen Netzwerk, dem Subnetz und der NSG. Dieser Befehl legt außerdem das zu verwendende VM-Image und die Administratoranmeldeinformationen fest.  |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#az_vm_extension_set) | Löscht eine Ressourcengruppe einschließlich aller geschachtelten Ressourcen. |

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zur Azure CLI finden Sie in der [Azure CLI-Dokumentation](https://docs.microsoft.com/cli/azure/overview).

Zusätzliche VM-CLI-Skriptbeispiele finden Sie in der [Dokumentation zu Windows-VMs in Azure](../windows/cli-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
