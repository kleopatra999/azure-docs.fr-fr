---
title: Utilisation du support de flux de modification dans Azure Cosmos DB | Microsoft Docs
description: "Utilisez le support de flux de modification d’Azure Cosmos DB pour suivre les modifications dans les documents et effectuer des opérations de traitement basées sur les événements tels que des déclencheurs et la mise à jour des systèmes de cache et d’analyse."
keywords: change feed
services: cosmos-db
author: arramac
manager: jhubbard
editor: mimig
documentationcenter: 
ms.assetid: 2d7798db-857f-431a-b10f-3ccbc7d93b50
ms.service: cosmos-db
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: rest-api
ms.topic: article
ms.date: 03/23/2017
ms.author: arramac
ms.translationtype: Human Translation
ms.sourcegitcommit: 7948c99b7b60d77a927743c7869d74147634ddbf
ms.openlocfilehash: d04d1240fb353a973953b2a90eadc65705219edb
ms.contentlocale: fr-fr
ms.lasthandoff: 06/20/2017


---
# <a name="working-with-the-change-feed-support-in-azure-cosmos-db"></a>Utilisation du support de flux de modification dans Azure Cosmos DB
[Azure Cosmos DB](../cosmos-db/introduction.md) est un service de base de données répliqué mondialement, rapide et flexible utilisé pour le stockage d’importants volumes de données transactionnelles et opérationnelles avec une latence prévisible en quelques millisecondes pour les lectures et les écritures. Il est ainsi parfait pour les applications d’IoT, de jeux, de vente au détail et de journalisation des compteurs. Un modèle de conception courant dans ces applications consiste à suivre les modifications apportées aux données Azure Cosmos DB et mettre à jour les vues matérialisées, effectuer des analyses en temps réel, archiver les données vers le stockage à froid et déclencher des notifications pour certains événements en fonction de ces modifications. Le **support de flux de modification** d’Azure Cosmos DB vous permet de créer des solutions efficaces et évolutives pour chacun de ces modèles.

Avec le support de flux de modification, Azure Cosmos DB fournit une liste triée de documents d’une collection Azure Cosmos DB affichés dans l’ordre dans lequel ils ont été modifiés. Ce flux peut être utilisé pour écouter les modifications apportées aux données dans la collection et effectuer des actions telles que :

* déclencher un appel à une API quand un document est inséré ou modifié ;
* effectuer un traitement en temps réel (flux) sur les mises à jour ;
* synchroniser les données avec un cache, le moteur de recherche ou l’entrepôt de données.

Les modifications dans Azure Cosmos DB sont conservées et peuvent être traitées de manière asynchrone et réparties sur un ou plusieurs consommateurs pour un traitement en parallèle. Examinons les API pour le flux de modification et voyons comment vous pouvez les utiliser pour créer des applications évolutives en temps réel. Cet article montre comment utiliser les données spatiales avec l’API DocumentDB d’Azure Cosmos DB. 

![Utilisation du flux de modification d’Azure Cosmos DB pour alimenter les analyses en temps réel et les scénarios de calcul pilotés par les événements](./media/change-feed/changefeed.png)

## <a name="use-cases-and-scenarios"></a>Cas d'utilisation et scénarios
Le flux de modification permet un traitement efficace des jeux de données importants avec un volume d’écritures élevé et offre une alternative à l’interrogation des jeux de données pour identifier les changements apportés. Par exemple, vous pouvez effectuer efficacement les tâches suivantes :

* Mettre à jour un cache, l’index de recherche ou un entrepôt de données avec les données stockées dans Azure Cosmos DB.
* Implémenter la hiérarchisation et l’archivage des données de niveau application, autrement dit, stocker les « données à chaud » dans Azure Cosmos DB et archiver les « données à froid » dans [Stockage Blob Azure](../storage/storage-introduction.md) ou [Azure Data Lake Store](../data-lake-store/data-lake-store-overview.md).
* Implémenter une analyse en mode batch sur les données à l’aide [d’Apache Hadoop](run-hadoop-with-hdinsight.md).
* Implémenter des [pipelines lambda sur Azure](https://blogs.technet.microsoft.com/msuspartner/2016/01/27/azure-partner-community-big-data-advanced-analytics-and-lambda-architecture/) avec Azure Cosmos DB. Azure Cosmos DB fournit une solution de base de données évolutive qui peut gérer l’ingestion et l’interrogation, et implémenter des architectures lambda avec un faible coût total de possession. 
* Effectuer des migrations sans aucun temps d’arrêt vers un autre compte Azure Cosmos DB avec un schéma de partitionnement différent.

**Pipelines lambda avec Azure Cosmos DB pour l’ingestion et l’interrogation :**

![Pipeline lambda Azure Cosmos DB pour l’ingestion et l’interrogation](./media/change-feed/lambda.png)

Vous pouvez utiliser Azure Cosmos DB pour recevoir et stocker des données d’événement d’appareils, de capteurs, de l’infrastructure et des applications, et traiter ces événements en temps réel avec [Azure Stream Analytics](../stream-analytics/stream-analytics-documentdb-output.md), [Apache Storm](../hdinsight/hdinsight-storm-overview.md) ou [Apache Spark](../hdinsight/hdinsight-apache-spark-overview.md). 

Dans les applications web et mobiles, vous pouvez suivre les événements tels que les modifications apportées aux profil, préférences ou emplacement de votre client pour déclencher certaines actions, comme l’envoi de notifications Push à leurs appareils à l’aide [d’Azure Functions](../azure-functions/functions-bindings-documentdb.md) ou [d’App Services](https://azure.microsoft.com/services/app-service/). Si vous utilisez Azure Cosmos DB pour créer un jeu, vous pouvez, par exemple, utiliser le flux de modification pour implémenter des classements en temps réel basés sur des scores de jeux terminés.

## <a name="how-change-feed-works-in-azure-cosmos-db"></a>Fonctionnement du flux de modification dans Azure Cosmos DB
Azure Cosmos DB offre la possibilité de lire de façon incrémentielle les mises à jour apportées à une collection Azure Cosmos DB. Ce flux de modification a les propriétés suivantes :

* Les modifications sont persistantes dans Azure Cosmos DB et peuvent être traitées de façon asynchrone.
* Les modifications apportées aux documents d’une collection sont disponibles immédiatement dans le flux de modification.
* Chaque modification apportée à un document apparaît une seule fois dans le flux de modification. Seule la dernière modification d’un document donné est incluse dans le journal des modifications. Les modifications intermédiaires peuvent ne pas être disponibles.
* Le flux de modification est trié par ordre de modification au sein de chaque valeur de clé de partition. Il n’existe aucun ordre garanti entre les valeurs de clé de partition.
* Les modifications peuvent être synchronisées à partir de n’importe quel moment donné ; autrement dit, il n’existe aucune période de rétention de données fixes pendant laquelle les modifications sont disponibles.
* Les modifications sont disponibles dans des blocs de plages de clés de partition. Cette fonctionnalité permet le traitement en parallèle de modifications de grandes collections par plusieurs consommateurs/serveurs.
* Les applications peuvent demander plusieurs flux de modification simultanément sur la même collection.

Le flux de modification d’Azure Cosmos DB est activé par défaut pour tous les comptes et n’entraîne pas de coûts supplémentaires sur votre compte. Vous pouvez utiliser votre [débit configuré](request-units.md) dans votre région d’écriture ou toute [région de lecture](distribute-data-globally.md) pour lire à partir du flux de modification comme toute autre opération effectuée à partir d’Azure Cosmos DB. Le flux de modification inclut les insertions et les opérations de mise à jour apportées aux documents de la collection. Vous pouvez capturer des suppressions en définissant un indicateur de « suppression réversible » dans vos documents à la place des suppressions. Vous pouvez également définir un délai d’expiration limité pour vos documents via la [fonctionnalité de durée de vie (TTL)](time-to-live.md), par exemple 24 heures, et utiliser la valeur de cette propriété pour capturer les suppressions. Avec cette solution, vous devez traiter les modifications dans un intervalle de temps inférieur à la période d’expiration de durée de vie. Le flux de modification est disponible pour chaque plage de clés de partition dans la collection de documents et peut ainsi être distribué vers un ou plusieurs consommateurs pour un traitement en parallèle. 

![Traitement distribué du flux de modification d’Azure Cosmos DB](./media/change-feed/changefeedvisual.png)

Dans la section suivante, nous décrivons comment accéder au flux de modification à l’aide de l’API REST d’Azure Cosmos DB et des kits de développement logiciel (SDK). Pour les applications .NET, nous vous recommandons d’utiliser la [bibliothèque du processeur de flux de modification]() pour traiter les événements à partir du flux de modification.

## <a id="rest-apis"></a>Utilisation de l’API REST et des Kits de développement logiciel (SDK)
Azure Cosmos DB fournit des conteneurs élastiques de stockage et de débit appelés **collections**. Les données contenues dans les collections sont regroupées logiquement à l’aide de [clés de partition](partition-data.md) à des fins d’évolutivité et de performance. Azure Cosmos DB fournit diverses API pour accéder à ces données, y compris la recherche par ID (Read/Get), les requêtes et les flux de lecture (analyses). Le flux de modification peut être obtenu en remplissant deux nouveaux en-têtes de demande de l’API `ReadDocumentFeed` de DocumentDB et peut être traité en parallèle sur différentes plages de clés de partition.

### <a name="readdocumentfeed-api"></a>API ReadDocumentFeed
Examinons brièvement comment ReadDocumentFeed fonctionne. Azure Cosmos DB prend en charge la lecture d’un flux de documents dans une collection via l’API `ReadDocumentFeed`. Par exemple, la demande suivante retourne une page de documents à l’intérieur de la collection `serverlogs`. 

    GET https://mydocumentdb.documents.azure.com/dbs/smalldb/colls/serverlogs HTTP/1.1
    x-ms-date: Tue, 22 Nov 2016 17:05:14 GMT
    authorization: type%3dmaster%26ver%3d1.0%26sig%3dgo7JEogZDn6ritWhwc5hX%2fNTV4wwM1u9V2Is1H4%2bDRg%3d
    Cache-Control: no-cache
    x-ms-consistency-level: Strong
    User-Agent: Microsoft.Azure.Documents.Client/1.10.27.5
    x-ms-version: 2016-07-11
    Accept: application/json
    Host: mydocumentdb.documents.azure.com

Les résultats peuvent être limités à l’aide de l’en-tête `x-ms-max-item-count`, et les lectures peuvent être reprises en soumettant à nouveau la demande avec un en-tête `x-ms-continuation` retourné dans la réponse précédente. Lors de l’opération à partir d’un seul client, `ReadDocumentFeed` itère dans les résultats sur les différentes partitions en série. 

**Flux de lecture de documents en série**

Vous pouvez également récupérer le flux de documents à l’aide d’un des [kits de développement logiciel (SDK) d’Azure Cosmos DB ](documentdb-sdk-dotnet.md) pris en charge. Par exemple, l’extrait de code suivant montre comment exécuter ReadDocumentFeed dans .NET.

    FeedResponse<dynamic> feedResponse = null;
    do
    {
        feedResponse = await client.ReadDocumentFeedAsync(collection, new FeedOptions { MaxItemCount = -1 });
    }
    while (feedResponse.ResponseContinuation != null);

### <a name="distributed-execution-of-readdocumentfeed"></a>Exécution distribuée de ReadDocumentFeed
Pour les collections qui contiennent des téraoctets de données ou plus, ou qui reçoivent un grand nombre de mises à jour, l’exécution en série de flux de lecture à partir d’un seul ordinateur client n’est pas forcément pratique. Pour prendre en charge ces scénarios Big Data, Azure Cosmos DB propose des API pour distribuer les appels `ReadDocumentFeed` de façon transparente sur plusieurs lecteurs/consommateurs clients. 

**Flux de lecture de documents distribué**

Pour fournir un traitement évolutif des modifications incrémentielles, Azure Cosmos DB prend en charge un modèle d’augmentation de la taille des instances pour l’API de flux de modification en fonction des plages de clés de partition.

* Vous pouvez obtenir une liste des plages de clés de partition d’une collection effectuant un appel `ReadPartitionKeyRanges`. 
* Pour chaque plage de clés de partition, vous pouvez exécuter `ReadDocumentFeed` pour la lecture des documents avec des clés de partition dans cette plage.

### <a name="retrieving-partition-key-ranges-for-a-collection"></a>Récupération des plages de clés de partition d’une collection
Vous pouvez récupérer les plages de clés de partition en demandant la ressource `pkranges` au sein d’une collection. Par exemple, la demande suivante récupère la liste des plages de clés de partition de la collection `serverlogs` :

    GET https://querydemo.documents.azure.com/dbs/bigdb/colls/serverlogs/pkranges HTTP/1.1
    x-ms-date: Tue, 15 Nov 2016 07:26:51 GMT
    authorization: type%3dmaster%26ver%3d1.0%26sig%3dEConYmRgDExu6q%2bZ8GjfUGOH0AcOx%2behkancw3LsGQ8%3d
    x-ms-consistency-level: Session
    x-ms-version: 2016-07-11
    Accept: application/json
    Host: querydemo.documents.azure.com

Cette demande renvoie la réponse suivante contenant les métadonnées des plages de clés de partition :

    HTTP/1.1 200 Ok
    Content-Type: application/json
    x-ms-item-count: 25
    x-ms-schemaversion: 1.1
    Date: Tue, 15 Nov 2016 07:26:51 GMT

    {
       "_rid":"qYcAAPEvJBQ=",
       "PartitionKeyRanges":[
          {
             "_rid":"qYcAAPEvJBQCAAAAAAAAUA==",
             "id":"0",
             "_etag":"\"00002800-0000-0000-0000-580ac4ea0000\"",
             "minInclusive":"",
             "maxExclusive":"05C1CFFFFFFFF8",
             "_self":"dbs\/qYcAAA==\/colls\/qYcAAPEvJBQ=\/pkranges\/qYcAAPEvJBQCAAAAAAAAUA==\/",
             "_ts":1477100776
          },
          ...
       ],
       "_count": 25
    }


**Propriétés des plages de clés de partition** : chaque plage de clés de partition inclut les propriétés de métadonnées indiquées dans le tableau suivant :

<table>
    <tr>
        <th>Nom de l’en-tête</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>id</td>
        <td>
            <p>ID de la plage de clés de partition. Il s’agit d’un ID stable et unique dans chaque collection.</p>
            <p>Il doit être utilisé dans l’appel suivant pour lire les modifications par plage de clés de partition.</p>
        </td>
    </tr>
    <tr>
        <td>maxExclusive</td>
        <td>Valeur de hachage de la clé de partition maximale pour la plage de clés de partition. À usage interne.</td>
    </tr>
    <tr>
        <td>minInclusive</td>
        <td>Valeur de hachage de la clé de partition minimale pour la plage de clés de partition. À usage interne.</td>
    </tr>       
</table>

Vous pouvez effectuer cette opération à l’aide d’un des [kits de développement logiciel (SDK) d’Azure Cosmos DB](documentdb-sdk-dotnet.md) pris en charge. Par exemple, l’extrait de code suivant montre comment récupérer des plages de clés de partition dans .NET.

    string pkRangesResponseContinuation = null;
    List<PartitionKeyRange> partitionKeyRanges = new List<PartitionKeyRange>();

    do
    {
        FeedResponse<PartitionKeyRange> pkRangesResponse = await client.ReadPartitionKeyRangeFeedAsync(
            collectionUri, 
            new FeedOptions { RequestContinuation = pkRangesResponseContinuation });

        partitionKeyRanges.AddRange(pkRangesResponse);
        pkRangesResponseContinuation = pkRangesResponse.ResponseContinuation;
    }
    while (pkRangesResponseContinuation != null);

Azure Cosmos DB prend en charge la récupération de documents par plage de clés de partition en définissant l’en-tête `x-ms-documentdb-partitionkeyrangeid` facultatif. 

### <a name="performing-an-incremental-readdocumentfeed"></a>Exécution d’un ReadDocumentFeed incrémentiel
ReadDocumentFeed prend en charge les scénarios/tâches suivants pour le traitement incrémentiel des modifications dans les collections Azure Cosmos DB :

* Lecture de toutes les modifications apportées aux documents depuis le début, autrement dit, depuis la création de la collection.
* Lecture de toutes les modifications ou futures mises à jour qui seront apportées aux documents à partir de maintenant.
* Lecture de toutes les modifications apportées aux documents à partir d’une version logique de la collection (ETag). Vous pouvez contrôler vos consommateurs en fonction de l’ETag retourné à partir des demandes de flux de lecture incrémentiels.

Les modifications incluent les insertions et les mises à jour apportées aux documents. Pour capturer les suppressions, vous devez utiliser une propriété de « suppression réversible » dans vos documents, ou utiliser la [propriété TTL intégrée](time-to-live.md) pour signaler une suppression en attente dans le flux de modification.

Le tableau suivant répertorie les en-têtes de demande et de réponse pour les opérations ReadDocumentFeed.

**En-têtes de demande pour les opérations ReadDocumentFeed incrémentielles** :

<table>
    <tr>
        <th>Nom de l’en-tête</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>A-IM</td>
        <td>Doit être défini sur « Incremental feed » (Flux incrémentiel), ou omis.</td>
    </tr>
    <tr>
        <td>If-None-Match</td>
        <td>
            <p>Aucun en-tête : retourne toutes les modifications depuis le début (création de la collection).</p>
            <p>« * » : retourne toutes les nouvelles modifications apportées aux données dans la collection.</p>
            <p>&lt;etag&gt; : s’il est défini sur un ETag de la collection, retourne toutes les modifications effectuées depuis cet horodatage logique.</p>
        </td>
    </tr>
    <tr>
        <td>x-ms-documentdb-partitionkeyrangeid</td>
        <td>ID de la plage de clés de partition pour la lecture des données.</td>
    </tr>
</table>

**En-têtes de réponse pour les opérations ReadDocumentFeed incrémentielles** :

<table>
    <tr>
        <th>Nom de l’en-tête</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>etag</td>
        <td>
            <p>Numéro de séquence logique (LSN) du dernier document renvoyé dans la réponse.</p>
            <p>L’opération ReadDocumentFeed incrémentielle peut être reprise en soumettant à nouveau cette valeur dans If-None-Match.</p>
        </td>
    </tr>
</table>

Voici un exemple de demande pour retourner toutes les modifications incrémentielles de la collection à partir de la version logique/ETag `28535` et de la plage de clés de partition = `16` :

    GET https://mydocumentdb.documents.azure.com/dbs/bigdb/colls/bigcoll/docs HTTP/1.1
    x-ms-max-item-count: 1
    If-None-Match: "28535"
    A-IM: Incremental feed
    x-ms-documentdb-partitionkeyrangeid: 16
    x-ms-date: Tue, 22 Nov 2016 20:43:01 GMT
    authorization: type%3dmaster%26ver%3d1.0%26sig%3dzdpL2QQ8TCfiNbW%2fEcT88JHNvWeCgDA8gWeRZ%2btfN5o%3d
    x-ms-version: 2016-07-11
    Accept: application/json
    Host: mydocumentdb.documents.azure.com

Les modifications sont classées par heure dans chaque valeur de clé de partition de la plage. Il n’existe aucun ordre garanti entre les valeurs de clé de partition. Si les résultats sont trop nombreux pour figurer sur une seule page, vous pouvez lire la page de résultats suivante en soumettant à nouveau la demande avec l’en-tête `If-None-Match` et une valeur égale à l’`etag` de la réponse précédente. Si plusieurs documents ont été insérés ou mis à jour via des transactions au sein d’une procédure stockée ou avec un déclencheur, ils seront tous renvoyés dans la même page de réponse.

> [!NOTE]
> Avec le flux de modification, plus d’éléments seront retournés dans une page spécifiée dans `x-ms-max-item-count` dans le cas de plusieurs documents insérés ou mis à jour à l’intérieur des procédures stockées ou des déclencheurs. 

Le Kit de développement logiciel (SDK) .NET fournit les classes d’assistance [CreateDocumentChangeFeedQuery](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.createdocumentchangefeedquery.aspx) et [ChangeFeedOptions](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.changefeedoptions.aspx) pour accéder aux modifications apportées à une collection. L’extrait de code suivant montre comment récupérer toutes les modifications apportées depuis le début à l’aide du SDK .NET à partir d’un seul client.

    private async Task<Dictionary<string, string>> GetChanges(
        DocumentClient client,
        string collection,
        Dictionary<string, string> checkpoints)
    {
        string pkRangesResponseContinuation = null;
        List<PartitionKeyRange> partitionKeyRanges = new List<PartitionKeyRange>();

        do
        {
            FeedResponse<PartitionKeyRange> pkRangesResponse = await client.ReadPartitionKeyRangeFeedAsync(
                collectionUri, 
                new FeedOptions { RequestContinuation = pkRangesResponseContinuation });

            partitionKeyRanges.AddRange(pkRangesResponse);
            pkRangesResponseContinuation = pkRangesResponse.ResponseContinuation;
        }
        while (pkRangesResponseContinuation != null);

        foreach (PartitionKeyRange pkRange in partitionKeyRanges)
        {
            string continuation = null;
            checkpoints.TryGetValue(pkRange.Id, out continuation);

            IDocumentQuery<Document> query = client.CreateDocumentChangeFeedQuery(
                collection,
                new ChangeFeedOptions
                {
                    PartitionKeyRangeId = pkRange.Id,
                    StartFromBeginning = true,
                    RequestContinuation = continuation,
                    MaxItemCount = 1
                });

            while (query.HasMoreResults)
            {
                FeedResponse<DeviceReading> readChangesResponse = query.ExecuteNextAsync<DeviceReading>().Result;

                foreach (DeviceReading changedDocument in readChangesResponse)
                {
                    Console.WriteLine(changedDocument.Id);
                }

                checkpoints[pkRange.Id] = readChangesResponse.ResponseContinuation;
            }
        }

        return checkpoints;
    }

L’extrait de code suivant montre comment traiter les modifications en temps réel avec Azure Cosmos DB à l’aide du support de flux de modification et de la fonction qui précède. Le premier appel retourne tous les documents de la collection, et le second retourne uniquement les deux documents créés depuis le dernier point de contrôle.

    // Returns all documents in the collection.
    Dictionary<string, string> checkpoints = await GetChanges(client, collection, new Dictionary<string, string>());

    await client.CreateDocumentAsync(collection, new DeviceReading { DeviceId = "xsensr-201", MetricType = "Temperature", Unit = "Celsius", MetricValue = 1000 });
    await client.CreateDocumentAsync(collection, new DeviceReading { DeviceId = "xsensr-212", MetricType = "Pressure", Unit = "psi", MetricValue = 1000 });

    // Returns only the two documents created above.
    checkpoints = await GetChanges(client, collection, checkpoints);


Vous pouvez également filtrer le flux de modification à l’aide de la logique côté client pour traiter les événements de manière sélective. Par exemple, voici un extrait de code qui utilise LINQ côté client pour traiter uniquement les événements de modification de température des capteurs de périphérique.

    FeedResponse<DeviceReading> readChangesResponse = query.ExecuteNextAsync<DeviceReading>().Result;

    foreach (DeviceReading changedDocument in 
        readChangesResponse.AsEnumerable().Where(d => d.MetricType == "Temperature" && d.MetricValue > 1000L))
    {
        // trigger an action, like call an API
    }

## <a id="change-feed-processor"></a>Bibliothèque du processeur de flux de modification
La [bibliothèque du processeur de flux de modification Azure Cosmos DB](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/ChangeFeedProcessor) peut être utilisée pour distribuer le traitement des événements du flux de modification sur plusieurs consommateurs d’événements. Vous devez utiliser cette implémentation lors de la création de lecteurs de flux de modification sur la plateforme .NET. La classe `ChangeFeedProcessorHost` fournit un environnement d'exécution sécurisé, multiprocessus, thread-safe pour des implémentations d’événements qui fournissent également une gestion de contrôle et de location de partition.

Pour utiliser la classe [`ChangeFeedProcessorHost`](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/ChangeFeedProcessor/DocumentDB.ChangeFeedProcessor/ChangeFeedEventHost.cs), vous pouvez implémenter [`IChangeFeedObserver`](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/ChangeFeedProcessor/DocumentDB.ChangeFeedProcessor/IChangeFeedObserver.cs). Cette interface contient trois méthodes :

* OpenAsync
* CloseAsync
* ProcessEventsAsync

Pour commencer le traitement des événements, instanciez ChangeFeedProcessorHost en fournissant les paramètres appropriés pour votre collection Azure Cosmos DB. Ensuite, appelez `RegisterObserverAsync` pour enregistrer votre implémentation `IChangeFeedObserver` avec le runtime. À ce stade, l’hôte tente d’acquérir un bail sur chaque partition dans chaque plage de clés de partition dans la collection Azure Cosmos DB à l’aide d’un algorithme « gourmand ». Ces baux sont valables pour une période donnée et doivent ensuite être renouvelés. Étant donné que de nouveaux nœuds (ici, des instances de rôle) sont en ligne, ils placent des réservations de bail et, en même temps, la charge est déplacée entre les nœuds tandis que chaque nœud tente d’acquérir plus de baux.

![Utilisation de l’hôte du processus de flux de modification Azure Cosmos DB](./media/change-feed/changefeedprocessor.png)

Au fil du temps, un équilibre est établi. Cette fonctionnalité dynamique permet d’appliquer aux consommateurs une mise à l'échelle basée sur le processeur pour une augmentation et une diminution d’échelle. Si Azure Cosmos DB affiche plus de modifications que ce que les consommateurs peuvent traiter, l’augmentation du processeur sur les consommateurs peut être utilisée pour effectuer une mise à l’échelle automatique sur le nombre d’instances de travail.

La classe `ChangeFeedProcessorHost` implémente également un mécanisme de point de contrôle à l’aide d’une collection de baux Azure Cosmos DB distincte. Ce mécanisme stocke le décalage sur une base par partition, afin que chaque consommateur puisse déterminer quel était le dernier point de contrôle du consommateur précédent. Étant donné que la transition des partitions entre les nœuds se fait via les baux, il s’agit d’un mécanisme de synchronisation qui facilité le déplacement de la charge.


Voici un extrait de code pour un hôte du processus de flux de modification simple qui imprime les modifications apportées à la console :

```cs
    class DocumentFeedObserver : IChangeFeedObserver
    {
        private static int s_totalDocs = 0;
        public Task OpenAsync(ChangeFeedObserverContext context)
        {
            Console.WriteLine("Worker opened, {0}", context.PartitionKeyRangeId);
            return Task.CompletedTask;  // Requires targeting .NET 4.6+.
        }
        public Task CloseAsync(ChangeFeedObserverContext context, ChangeFeedObserverCloseReason reason)
        {
            Console.WriteLine("Worker closed, {0}", context.PartitionKeyRangeId);
            return Task.CompletedTask;
        }
        public Task ProcessEventsAsync(IReadOnlyList<Document> docs, ChangeFeedObserverContext context)
        {
            Console.WriteLine("Change feed: total {0} doc(s)", Interlocked.Add(ref s_totalDocs, docs.Count));
            return Task.CompletedTask;
        }
    }
```

L’extrait de code suivant montre comment inscrire un nouvel hôte chargé d’écouter les modifications à partir d’une collection Azure Cosmos DB. Ici, nous configurons une collection distincte pour gérer les baux de connexion aux partitions pour plusieurs consommateurs :

```cs
    string hostName = Guid.NewGuid().ToString();
    DocumentCollectionInfo documentCollectionLocation = new DocumentCollectionInfo
    {
        Uri = new Uri("https://YOUR_SERVICE.documents.azure.com:443/"),
        MasterKey = "YOUR_SECRET_KEY==",
        DatabaseName = "db1",
        CollectionName = "documents"
    };

    DocumentCollectionInfo leaseCollectionLocation = new DocumentCollectionInfo
    {
        Uri = new Uri("https://YOUR_SERVICE.documents.azure.com:443/"),
        MasterKey = "YOUR_SECRET_KEY==",
        DatabaseName = "db1",
        CollectionName = "leases"
    };

    ChangeFeedEventHost host = new ChangeFeedEventHost(hostName, documentCollectionLocation, leaseCollectionLocation);
    await host.RegisterObserverAsync<DocumentFeedObserver>();
```

Dans cet article, nous vous avons fourni une procédure pas à pas concernant le support de flux de modification d’Azure Cosmos DB et expliqué comment effectuer le suivi des modifications apportées aux données d’Azure Cosmos DB à l’aide de l’API REST et/ou des kits de développement logiciel (SDK). 

## <a name="next-steps"></a>Étapes suivantes
* Essayez les [exemples de code de flux de modification Azure Cosmos DB sur GitHub](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/code-samples/ChangeFeed).
* Commencez à coder avec [les kits de développement logiciel (SDK) Azure Cosmos DB](documentdb-sdk-dotnet.md) ou [l’API REST](/rest/api/documentdb/).

