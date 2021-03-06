---
title: "Connecter Excel à Hadoop à l’aide du pilote ODBC Hive - Azure HDInsight | Documents Microsoft"
description: "Découvrez comment configurer et utiliser le pilote ODBC Microsoft Hive pour Excel afin d’interroger des données dans des clusters HDInsight à partir de Microsoft Excel."
keywords: hadoop excel,hive excel,hive odbc
services: hdinsight
documentationcenter: 
author: mumian
manager: jhubbard
tags: azure-portal
editor: cgronlun
ms.assetid: a7665a14-0211-4521-b3e7-3b26e8029cc0
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 08/22/2017
ms.author: jgao
ms.translationtype: HT
ms.sourcegitcommit: 25e4506cc2331ee016b8b365c2e1677424cf4992
ms.openlocfilehash: 7818093e42c34ee671a035cde783a6622fb2a798
ms.contentlocale: fr-fr
ms.lasthandoff: 08/24/2017

---
# <a name="connect-excel-to-hadoop-in-azure-hdinsight-with-the-microsoft-hive-odbc-driver"></a>Connecter Excel à Hadoop dans Azure HDInsight avec le pilote ODBC Microsoft Hive

[!INCLUDE [ODBC-JDBC-selector](../../includes/hdinsight-selector-odbc-jdbc.md)]

La solution de données volumineuses de Microsoft intègre des composants BI (Business Intelligence) à des clusters Apache Hadoop qui ont été déployés par Azure HDInsight. Un exemple de cette intégration est la possibilité de connecter Excel à l’entrepôt de données Hive d’un cluster Hadoop dans HDInsight au moyen du pilote Open Database Connectivity (ODBC) Microsoft Hive.

Il est également possible de connecter les données associées à un cluster HDInsight et d'autres sources de données, y compris d'autres clusters Hadoop (non HDInsight), à partir d'Excel au moyen du complément Microsoft Power Query pour Excel. Pour plus d’informations sur l’installation et l’utilisation de Power Query, consultez [Connexion d’Excel à HDInsight à l’aide de Power Query][hdinsight-power-query].

> [!NOTE]
> Même si les étapes décrites dans cet article peuvent être utilisées avec un cluster HDInsight basé sur Linux ou Windows, Windows est requis pour le poste de travail client.
> 
> 

**Conditions préalables**:

Avant de commencer cet article, vous devez disposer des éléments suivants :

* **Un cluster HDInsight**. Pour en créer un, consultez [Prise en main d’Azure HDInsight][hdinsight-get-started].
* **Une station de travail** avec Office Professionnel Plus 2013, Office 365 ProPlus, l'édition autonome d'Excel 2013 ou Office Professionnel Plus 2010.

## <a name="install-microsoft-hive-odbc-driver"></a>Installation du pilote ODBC Microsoft Hive
Téléchargez et installez le pilote ODBC Microsoft Hive à partir du [Centre de téléchargement][hive-odbc-driver-download].

Ce pilote peut être installé sur les versions 32 bits ou 64 bits de Windows 7, Windows 8, Windows 10, Windows Server 2008 R2 et Windows Server 2012. Le pilote autorise la connexion à Azure HDInsight (versions 1.6 et ultérieures) et à Azure HDInsight Emulator (v.1.0.0.0 et versions ultérieures). Vous allez installer la version correspondant à celle de l’application dans laquelle vous utilisez le pilote ODBC. Pour les besoins de ce didacticiel, le pilote utilisé est celui d’Office Excel.

## <a name="create-hive-odbc-data-source"></a>Création d’une source de données ODBC Hive
La procédure suivante explique comment créer une source de données ODBC Hive.

1. Sur Windows 8 ou Windows 10, appuyez sur la touche Windows pour ouvrir l’écran d’accueil, puis tapez **sources de données**.
2. Cliquez sur **Configurer les sources de données ODBC (32 bits)** ou **Configurer les sources de données ODBC (64 bits)** selon votre version d'Office. Si vous utilisez Windows 7, choisissez **Sources de données ODBC (32 bits)** ou **Sources de données ODBC (64 bits)** dans **Outils d'administration**. La boîte de dialogue **Administrateur de sources de données ODBC** s’affiche.
   
    ![Administrateur de sources de données ODBC](./media/hdinsight-connect-excel-hive-ODBC-driver/HDI.SimbaHiveOdbc.DataSourceAdmin1.png "Configurer un DSN à l’aide de l’Administrateur de sources de données ODBC")

3. Dans DNS Utilisateur, cliquez sur **Ajouter** pour ouvrir l'Assistant **Créer une nouvelle source de données**.
4. Sélectionnez **Microsoft Hive ODBC Driver**, puis cliquez sur **Terminer**. La boîte de dialogue de **configuration de DNS de pilote ODBC Microsoft Hive** s’affiche.
5. Tapez ou sélectionnez les valeurs suivantes :
   
   | Propriété | Description |
   | --- | --- |
   |  Data Source Name |Donnez un nom à votre source de données |
   |  Host |Entrez &lt;HDInsightClusterName>.azurehdinsight.net. Par exemple, myHDICluster.azurehdinsight.net |
   |  Port |Utilisez <strong>443</strong>. (ce port est passé de 563 à 443). |
   |  Base de données |Utilisez <strong>Default</strong>. |
   |  Mechanism |Sélectionnez <strong>Azure HDInsight Service</strong>. |
   |  User Name |Entrez le nom de l’utilisateur HTTP du cluster HDInsight. Le nom d’utilisateur par défaut est <strong>admin</strong>. |
   |  Mot de passe |Entrez le mot de passe du cluster HDInsight. |
   
    </table>
   
    Certains paramètres importants sont à prendre en compte lorsque vous cliquez sur **Options avancées**:
   
   | Paramètre | Description |
   | --- | --- |
   |  Use Native Query |Une fois sélectionné, le pilote ODBC ne tente PAS de convertir TSQL en HiveQL. À utiliser uniquement si vous êtes sûr à 100 % que vous envoyez des instructions HiveQL pures. Si vous effectuez une connexion à SQL Server ou Base de données SQL Azure, ne sélectionnez pas cette option. |
   |  Rows fetched per block |Lors de l’extraction d’un grand nombre d’enregistrements, la définition de ce paramètre peut être nécessaire pour garantir des performances optimales. |
   |  Default string column length, Binary column length, Decimal column scale |Les précisions et longueurs des types de données peuvent affecter la façon dont les données sont renvoyées. Elles entraînent le renvoi d’informations incorrectes en raison d’une perte de précision et/ou de troncations. |

    ![Options avancées](./media/hdinsight-connect-excel-hive-ODBC-driver/HDI.HiveOdbc.DataSource.AdvancedOptions1.png "Options avancées de configuration de DSN")

1. Cliquez sur **Tester** pour tester la source de données. Une fois que la source de données est configurée correctement, le message suivant apparaît *TESTS COMPLETED SUCCESSFULLY!*.
2. Cliquez sur **OK** pour fermer la boîte de dialogue de test. La nouvelle source de données doit apparaitre dans **l’administrateur des sources de données ODBC**.
3. Cliquez sur **OK** pour quitter l'Assistant.

## <a name="import-data-into-excel-from-hdinsight"></a>Importation de données dans Microsoft Excel à partir de HDInsight
Les étapes ci-dessous décrivent comment importer des données d’une table Hive dans un classeur Excel à l’aide de la source de données ODBC que vous avez créée dans la section précédente.

1. Ouvrez un nouveau classeur ou un classeur existant dans Excel.
2. À partir de l’onglet **Données**, cliquez sur **Obtenir des données**, sur **Depuis d’autres sources** puis sur **Depuis ODBC** pour lancer **l’Assistant Connexion de données**.
   
    ![Assistant Ouvrir la connexion de données](./media/hdinsight-connect-excel-hive-ODBC-driver/HDI.SimbaHiveOdbc.Excel.DataConnection1.png "Assistant Ouvrir la connexion de données")
4. Sélectionnez le nom de la source de données que vous avez créé dans la dernière section, puis cliquez sur **OK**.
5. Entrez le nom d’utilisateur Hadoop (le nom par défaut est admin) et le mot de passe, puis cliquez sur **Connecter**.
6. Dans le navigateur, développez **HIVE**, développez **par défaut**, cliquez sur **hivesampletable**, puis cliquez sur **Charger**. Patientez quelques secondes pour que les données soient importées dans Excel.

    ![Navigateur ODBC HDInsight Hive](./media/hdinsight-connect-excel-hive-ODBC-driver/hdinsight.hive.odbc.navigator.png "Assistant Ouvrir la connexion de données")


## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez appris à utiliser le pilote ODBC Microsoft Hive pour extraire des données du service HDInsight dans Excel. De la même façon, vous pouvez extraire les données du service HDInsight dans la base de données SQL. Il est également possible de télécharger des données dans un service HDInsight. Pour plus d'informations, consultez les rubriques suivantes :

* [Analyse des données sur les retards de vol avec HDInsight][hdinsight-analyze-flight-data]
* [Téléchargement de données vers HDInsight][hdinsight-upload-data]
* [Utilisation de Sqoop avec HDInsight][hdinsight-use-sqoop]

[hdinsight-use-sqoop]: hdinsight-use-sqoop.md
[hdinsight-analyze-flight-data]: hdinsight-analyze-flight-delay-data.md
[hdinsight-use-hive]: hdinsight-use-hive.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-power-query]: hdinsight-connect-excel-power-query.md
[hdinsight-get-started]: hdinsight-hadoop-tutorial-get-started-windows.md

[hive-odbc-driver-download]: http://go.microsoft.com/fwlink/?LinkID=286698



