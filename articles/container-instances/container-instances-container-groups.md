---
title: Groupes de conteneurs Azure Container Instances
description: "Découvrez comment fonctionnent les groupes de conteneurs dans Azure Container Instances"
services: container-instances
documentationcenter: 
author: seanmck
manager: timlt
editor: 
tags: 
keywords: 
ms.assetid: 
ms.service: container-instances
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/08/2017
ms.author: seanmck
ms.custom: mvc
ms.translationtype: HT
ms.sourcegitcommit: f9003c65d1818952c6a019f81080d595791f63bf
ms.openlocfilehash: 5b2abb76fc2bfcf719292db1fdafc8b353d13a3c
ms.contentlocale: fr-fr
ms.lasthandoff: 08/09/2017

---

# <a name="container-groups-in-azure-container-instances"></a>Groupes de conteneurs dans Azure Container Instances

La ressource de niveau supérieur dans Azure Container Instances est un groupe de conteneurs. Cet article décrit les groupes de conteneurs et les types de scénarios associés.

## <a name="how-a-container-group-works"></a>Fonctionnement d’un groupe de conteneurs

Un groupe de conteneurs est une collection de conteneurs qui sont planifiés sur le même ordinateur hôte et qui partagent un cycle de vie, un réseau local et des volumes de stockage. Il s’apparente conceptuellement à un *pod* dans [Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/pod/) et [DC/OS](https://dcos.io/docs/1.9/deploying-services/pods/).

Le diagramme suivant montre un exemple de groupe de conteneurs.

![Exemple de groupe de conteneurs][container-groups-example]

Notez les points suivants :

- Le groupe est planifié sur un ordinateur hôte unique.
- Le groupe expose une adresse IP publique unique, avec exposition d’un seul port.
- Le groupe se compose de deux conteneurs. Un conteneur écoute le port 80, l’autre le port 5000.
- Le groupe inclut deux partages de fichiers Azure en tant que montages de volume, et chaque conteneur monte l’un des partages localement.

### <a name="networking"></a>Mise en réseau

Les groupes du conteneur partagent une adresse IP et un espace de noms de port sur cette adresse IP. Pour que les clients externes puissent atteindre un conteneur au sein du groupe, vous devez exposer le port sur l’adresse IP et à partir du conteneur. Étant donné que les conteneurs au sein du groupe partagent un espace de noms de port, le mappage de port n’est pas pris en charge. Les conteneurs au sein d’un groupe peuvent s’atteindre les uns les autres via localhost sur les ports qu’ils ont exposés, même si ces ports ne sont pas exposés en externe sur l’adresse IP du groupe.

### <a name="storage"></a>Stockage

Vous pouvez spécifier des volumes externes à monter dans un groupe de conteneurs. Vous pouvez mapper ces volumes à des chemins spécifiques dans les conteneurs individuels d’un groupe.

## <a name="common-scenarios"></a>Scénarios courants

Avec les groupes de conteneurs, vous pouvez répartir une seule tâche fonctionnelle sur un petit nombre d’images conteneur, qui peuvent être fournies par différentes équipes et présenter des exigences spécifiques en ressources. Exemples d’utilisation :

- Un conteneur d’applications et un conteneur de journalisation. Le conteneur de journalisation collecte les journaux et les métriques générés par l’application principale, puis les écrit dans le stockage à long terme.
- Une application et un conteneur de surveillance. Le conteneur de surveillance émet régulièrement une demande à destination de l’application pour vérifier qu’elle s’exécute et répond correctement, et déclenche une alerte si ce n’est pas le cas.
- Un conteneur servant une application web et un conteneur extrayant le contenu le plus récent du contrôle de code source.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [déployer un groupe de conteneurs](container-instances-multi-container-group.md) avec un modèle Azure Resource Manager.

<!-- IMAGES -->

[container-groups-example]: ./media/container-instances-container-groups/container-groups-example.png
