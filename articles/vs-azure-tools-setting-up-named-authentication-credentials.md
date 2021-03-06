---
title: "Configuration des informations d’authentification nommées | Microsoft Docs"
description: "Découvrez comment fournir des informations d’identification que Visual Studio pourra utiliser pour authentifier les demandes effectuées auprès d’Azure dans le cadre de la publication d’une application dans Azure à partir de Visual Studio, ou de l’analyse d’un service cloud existant. "
services: visual-studio-online
documentationcenter: na
author: TomArcher
manager: douge
editor: 
ms.assetid: 61570907-42a1-40e8-bcd6-952b21a55786
ms.service: multiple
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 8/17/2017
ms.author: tarcher
ms.translationtype: HT
ms.sourcegitcommit: 847eb792064bd0ee7d50163f35cd2e0368324203
ms.openlocfilehash: 613f17081fcb70b126caaae7ade5739d336662a7
ms.contentlocale: fr-fr
ms.lasthandoff: 08/19/2017

---
# <a name="setting-up-named-authentication-credentials"></a>Configuration des informations d’authentification nommées
Pour publier une application dans Azure à partir de Visual Studio, ou pour analyser un service cloud existant, vous devez fournir des informations d’identification que Visual Studio pourra utiliser pour authentifier les demandes effectuées auprès d’Azure. Il existe plusieurs emplacements dans Visual Studio à partir desquels vous pouvez vous connecter pour fournir ces informations d’identification. Par exemple, à partir de l’Explorateur de serveurs, vous pouvez ouvrir le menu contextuel du nœud **Azure**, puis sélectionner **Se connecter à Abonnement Microsoft Azure...**. Quand vous vous connectez, les informations d’abonnement associées à votre compte Azure sont disponibles dans Visual Studio. Vous n’avez donc rien à faire de plus.

Les outils Azure prennent également en charge une ancienne méthode d’authentification, qui est l’utilisation du fichier d’abonnement (fichier .publishsettings). Cette rubrique explique cette méthode, qui est toujours prise en charge dans le kit de développement logiciel (SDK) Azure 2.2.

Les éléments suivants sont indispensables pour l’authentification dans Azure :

* Votre ID d’abonnement
* Un certificat X.509 v3 valide

> [!NOTE]
> La longueur de la clé du certificat X.509 v3 doit être de 2 048 bits au minimum. Azure rejette les certificats qui ne répondent pas à cette exigence ou qui ne sont pas valides.
>
>

Visual Studio utilise votre ID d’abonnement et les données du certificat comme informations d’identification. Les informations d’identification sont référencées dans le fichier d’abonnement (fichier .publishsettings), qui contient une clé publique pour le certificat. Le fichier d’abonnement peut contenir des informations d’identification pour plusieurs abonnements.

Vous pouvez modifier les informations d’abonnement à partir de la boîte de dialogue **Modifier l’abonnement/Nouvel abonnement** , comme expliqué plus loin dans cette rubrique.

Si vous souhaitez créer vous-même un certificat, vous pouvez consulter les instructions fournies dans [Créer et charger un certificat de gestion pour Azure](https://msdn.microsoft.com/library/windowsazure/gg551722.aspx), puis charger manuellement le certificat vers le [portail Azure Classic](http://go.microsoft.com/fwlink/?LinkID=213885).

> [!NOTE]
> Les informations d’identification dont Visual Studio a besoin pour gérer vos services cloud ne sont pas les mêmes que celles nécessaires pour authentifier une demande effectuée auprès des services de stockage Azure.
>
>

## <a name="modify-or-export-authentication-credentials-in-visual-studio"></a>Modifier ou exporter des informations d’authentification dans Visual Studio
Vous pouvez également configurer, modifier ou exporter vos informations d’authentification à partir de la boîte de dialogue **Nouvel abonnement** qui s’affiche quand vous effectuez l’une des actions suivantes :

* Dans l’**Explorateur de serveurs**, ouvrez le menu contextuel du nœud **Azure**, choisissez **Gérer et filtrer les abonnements...**, sélectionnez l’onglet **Certificats**, puis au choix **Importer**, **Nouveau** ou **Modifier**.
* Quand vous publiez un service cloud Azure à partir de l’Assistant **Publication d’application Azure**, sélectionnez **Gérer** dans la liste **Choisir votre abonnement**. Ensuite, sélectionnez l’onglet Certificats, puis sélectionnez le bouton **Nouveau** ou **Modifier**.

La procédure suivante suppose que la boîte de dialogue **Nouvel abonnement** est ouverte.

### <a name="to-set-up-authentication-credentials-in-visual-studio"></a>Pour modifier ou exporter des informations d’authentification dans Visual Studio
1. Dans la liste **Sélectionnez un certificat existant pour l’authentification** , choisissez un certificat.
2. Cliquez sur le bouton **Copier le chemin d’accès complet**. Le chemin d’accès du certificat (fichier .cer) est copié dans le Presse-papiers.

   > [!IMPORTANT]
   > Pour publier votre application Azure à partir de Visual Studio, vous devrez charger ce certificat vers le [portail Azure Classic](http://go.microsoft.com/fwlink/?LinkID=213885).
   >
   >
3. Pour télécharger le certificat sur le [portail Azure Classic](http://go.microsoft.com/fwlink/?LinkID=213885):

   1. Cliquez sur le lien du portail Azure.

        Le [portail Azure Classic](http://go.microsoft.com/fwlink/?LinkID=213885) s’ouvre.
   2. Connectez-vous au [portail Azure Classic](http://go.microsoft.com/fwlink/?LinkID=213885), puis cliquez sur le bouton **Cloud Services** .
   3. Sélectionnez le service cloud qui vous intéresse.

       La page du service s’ouvre.
   4. Sous l’onglet **Certificats**, choisissez le bouton **Télécharger**.
   5. Collez le chemin d’accès complet du fichier .cer que vous venez de créer, puis entrez le mot de passe que vous avez spécifié.

