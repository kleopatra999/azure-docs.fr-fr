---
title: "Exemple de script Azure PowerShell - Créer un cluster Service Fabric | Microsoft Docs"
description: "Exemple de script Azure PowerShell - Créer un cluster Service Fabric"
services: service-fabric
documentationcenter: 
author: rwike77
manager: timlt
editor: 
tags: azure-service-management
ms.assetid: 0f9c8bc5-3789-4eb3-8deb-ae6e2200795a
ms.service: service-fabric
ms.workload: multiple
ms.devlang: na
ms.topic: sample
ms.date: 06/20/2017
ms.author: ryanwi
ms.custom: mvc
ms.translationtype: HT
ms.sourcegitcommit: 9afd12380926d4e16b7384ff07d229735ca94aaa
ms.openlocfilehash: 7322570c90b3c83fc79d428a7ac0b57ecb2e4b75
ms.contentlocale: fr-fr
ms.lasthandoff: 07/15/2017

---

# <a name="create-a-service-fabric-cluster"></a>Créer un cluster Service Fabric

Cet exemple de script crée un cluster Service Fabric à cinq nœuds sécurisé avec un certificat X.509.  La commande crée un certificat auto-signé et le charge vers un nouveau coffre de clés. Le certificat est également copié dans un répertoire local.  Définissez le paramètre *-OS* pour choisir la version de Windows ou Linux qui s’exécute sur les nœuds de cluster.  Personnalisez les paramètres, le cas échéant.

Si nécessaire, installez Azure PowerShell à l’aide des instructions figurant dans le [Guide Azure PowerShell](/powershell/azure/overview), puis exécutez `Login-AzureRmAccount` pour créer une connexion avec Azure. 

## <a name="sample-script"></a>Exemple de script

[!code-powershell[main](../../../powershell_scripts/service-fabric/create-secure-cluster/create-secure-cluster.ps1 "Créer un cluster Service Fabric")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement 

Une fois l’exemple de script exécuté, la commande suivante permet de supprimer le groupe de ressources, le cluster et toutes les ressources associées.

```powershell
$groupname="mysfclustergroup"
Remove-AzureRmResourceGroup -Name $groupname -Force
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Remarques |
|---|---|
| [New-AzureRmServiceFabricCluster](/powershell/module/azurerm.servicefabric/New-AzureRmServiceFabricCluster) | Crée un cluster Service Fabric. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le module Azure PowerShell, consultez [Documentation Azure PowerShell](/powershell/azure/overview).

Vous trouverez des exemples supplémentaires de scripts Azure PowerShell pour Azure Service Fabric sur la page [Azure PowerShell Samples](../service-fabric-powershell-samples.md) (Exemples Azure PowerShell).

