---
title: "Préparation des ressources Azure pour répliquer des machines virtuelles Hyper-V (avec System Center VMM) vers Azure à l’aide d’Azure Site Recovery | Microsoft Docs"
description: "Décrit les éléments à mettre en place dans Azure avant de commencer la réplication de machines virtuelles Hyper-V (avec VMM) vers Azure à l’aide d’Azure Site Recovery"
services: site-recovery
documentationcenter: 
author: rayne-wiselman
manager: carmonm
editor: 
ms.assetid: 1568bdc3-e767-477b-b040-f13699ab5644
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 07/23/2017
ms.author: raynew
ms.translationtype: HT
ms.sourcegitcommit: 74b75232b4b1c14dbb81151cdab5856a1e4da28c
ms.openlocfilehash: 365dd9477f791432c1a92f1b81eb573dbbc6f874
ms.contentlocale: fr-fr
ms.lasthandoff: 07/26/2017

---

# <a name="step-5-prepare-azure-resources-for-hyper-v-replication-with-vmm-to-azure"></a>Étape 5 : Préparer les ressources Azure pour la réplication de Hyper-V (avec VMM) vers Azure

Après avoir vérifié la [configuration réseau requise](vmm-to-azure-walkthrough-network.md), utilisez les instructions fournies dans cet article pour préparer les ressources Azure de manière à pouvoir répliquer des machines virtuelles Hyper-V locales dans les clouds System Center Virtual Machine Manager (VMM) vers Azure à l’aide du service [Azure Site Recovery](site-recovery-overview.md).

Après avoir lu cet article, envoyez vos commentaires en bas ou posez vos questions techniques sur le [Forum Azure Recovery Services](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr).


## <a name="set-up-an-azure-account"></a>Configurer un compte Azure

- Procurez-vous un [compte Microsoft Azure](http://azure.microsoft.com/).
- Vous pouvez commencer par une version d’ [essai gratuit](https://azure.microsoft.com/pricing/free-trial/).
- Vérifiez les régions prises en charge pour Site Recovery dans la section Disponibilité géographique de la page [Tarification d’Azure Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/)
- Lisez les informations relatives à la [tarification de Site Recovery](site-recovery-faq.md#pricing) et prenez connaissance de la [tarification appliquée](https://azure.microsoft.com/pricing/details/site-recovery/).
- Assurez-vous que votre compte Azure dispose des [autorisations](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines) nécessaires pour créer des machines virtuelles Azure. [En savoir plus](../active-directory/role-based-access-built-in-roles.md) sur le contrôle d’accès en fonction du rôle dans Azure.


## <a name="set-up-an-azure-network"></a>Configurer un réseau Azure

- Configurez un [réseau Azure](../virtual-network/virtual-network-get-started-vnet-subnet.md). Les machines virtuelles Azure sont placées dans ce réseau une fois créées après le basculement.
- Ce réseau doit se trouver dans la même région que le coffre Recovery Services.
- Site Recovery dans le portail Azure peut utiliser les réseaux configurés dans [Resource Manager](../resource-manager-deployment-model.md) ou en mode classique.
- Nous vous recommandons de configurer un réseau avant de commencer. Sinon, vous devrez le faire lors du déploiement de Site Recovery.
- Prenez connaissance de la [tarification des réseaux virtuels](https://azure.microsoft.com/pricing/details/virtual-network/).


## <a name="set-up-an-azure-storage-account"></a>Configurer un compte de stockage Azure

- Site Recovery réplique les machines virtuelles locales sur le stockage Azure. Des machines virtuelles Azure sont créées à partir du stockage après le basculement.
- Configurez un [compte de stockage Azure](../storage/storage-create-storage-account.md#create-a-storage-account) Standard ou Premium pour stocker les données répliquées vers Azure.
- Le [stockage Premium](../storage/storage-premium-storage.md) est généralement utilisé pour les machines virtuelles nécessitant des performances d’E/S élevées et une faible latence pour héberger les charges de travail nécessitant beaucoup d’E/S.
- Si vous souhaitez utiliser un compte Premium pour stocker les données répliquées, vous avez aussi besoin d’un compte de stockage standard afin de stocker les journaux de réplication qui capturent les modifications apportées en continu aux données locales.
- Selon le modèle de ressource que vous souhaitez utiliser pour les machines virtuelles Azure ayant fait l’objet d’un basculement, vous allez configurer un compte en [mode Azure Resource Manager](../storage/storage-create-storage-account.md) ou en [mode Classic](../storage/storage-create-storage-account-classic-portal.md).
- Nous vous recommandons de configurer un compte de stockage avant de commencer. Sinon, vous devrez le faire lors du déploiement de Site Recovery. Les comptes doivent se trouver dans la même région que le coffre Recovery Services.
- Vous ne pouvez pas déplacer de comptes de stockage utilisés par Site Recovery entre des groupes de ressources au sein du même abonnement, ou entre différents abonnements.


## <a name="next-steps"></a>Étapes suivantes

Aller à [Étape 6 : Préparer VMM](vmm-to-azure-walkthrough-vmm-hyper-v.md)

