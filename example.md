# INC0584729 - Problématique avec Kafka

Titre de l'incident: Problématique avec Kafka
Description de l'incident: Problématique avec Kafka depuis hier dans monitoring-dev-01 ce qui empêche le fonctionnement normal de l'ensemble des logging operator de dev.
Le délai de l'ES Sink vers elastic est rendu énorme 2758607377

Date d'ouverture de l'incident: 2022-06-09 08:32:21

## Tables des matières

- [INC0584729 - Problématique avec Kafka](#inc0584729---problématique-avec-kafka)
  - [Tables des matières](#tables-des-matières)
  - [Chronologie des événements](#chronologie-des-événements)
  - [Corrections appliquées](#corrections-appliquées)
    - [Validation de base et constat que la problématique est vers 1 topics Kafka (Corp_InfraOper_k8splatform_client_system_log_DEV) et 1 flow (Systems Logs)](#validation-de-base-et-constat-que-la-problématique-est-vers-1-topics-kafka-corp_infraoper_k8splatform_client_system_log_dev-et-1-flow-systems-logs)
    - [Correction de l'ES Sink vers Elastic qui était hors service](#correction-de-les-sink-vers-elastic-qui-était-hors-service)
    - [ElasticSearch était hors service et en problème. Travaux afin de régulariser le tout.](#elasticsearch-était-hors-service-et-en-problème-travaux-afin-de-régulariser-le-tout)
    - [Supprimer à la source (Sur les fluentD) les logs du flow (Systems Logs) afin d'effectuer de l'espace pour éviter une panne.](#supprimer-à-la-source-sur-les-fluentd-les-logs-du-flow-systems-logs-afin-deffectuer-de-lespace-pour-éviter-une-panne)
    - [Supprimer le contenu du topic afin de libérer de l'espace.](#supprimer-le-contenu-du-topic-afin-de-libérer-de-lespace)
    - [Recréer le topic Kafka](#recréer-le-topic-kafka)
  - [Solutions appliquées](#solutions-appliquées)

## Chronologie des événements

<dl>
<dt>2022-06-08 12h20</dt>
<dd> Début des alertes courriel pour l'ensemble de la DEV</dd>

<dd> Validation de base et constat que la problématique est vers 1 topics Kafka (Corp_InfraOper_k8splatform_client_system_log_DEV) et 1 flow (Systems Logs)</dd>

<dt>2022-06-08 en après-midi</dt>
<dd> Validation avec équipe plateforme si changement</dd>

<dd> Validation avec équipe plateforme si la panne de réseau sur l'heure du diner est la cause</dd>

<dd> Correction de l'ES Sink vers Elastic qui était hors service </dd>

<dt>2022-06-08 en soirée</dt>
<dd> ElasticSearch était hors service et en problème. Travaux afin de régulariser le tout.</dd>

<dt>2022-06-09</dt>
<dd>Le topic Kafka est énorme (+ de 10TB). Le buffer de logging operator est dangereusement élevé sur quelques clusters</dd>
<dd>Supprimer à la source (Sur les fluentD) les logs du flow (Systems Logs) afin d'effectuer de l'espace pour éviter une panne </dd>

<dt>2022-06-09</dt>
<dd> Supprimer le contenu du topic afin de libérer de l'espace.</dd>

<dt>2022-06-09</dt>
<dd> Enlever kong du cluster flow system-logs</dd>
<dd> Ceci afin d'enlever 50% de la journalisation en trop</dd>
<dd> Stabilisation des buffers mais aucune résolution du problème suite à cela</dd>

<dt>2022-06-10</dt>
<dd> Rebalancer le topic Kafka </dd>
<dd> Recréer le topic Kafka</dd>
<dd> Constater que tout revient à la normale</dd>
<dd> Valider l'ensemble des logging operator de dev </dd>

## Corrections appliquées

### Validation de base et constat que la problématique est vers 1 topics Kafka (Corp_InfraOper_k8splatform_client_system_log_DEV) et 1 flow (Systems Logs)

Valider dans le Grafana de Rancher sous /Plateforme Conteneurs/Logging Operator Dashboard/Fluentd Output Buffer

![Images](images/Logging-Operator-Dashboard-Grafana.png)

### Correction de l'ES Sink vers Elastic qui était hors service 

L'ES Sink vers Elastic était down suite à des problématiques réseaux.

Nous l'avons simplement redémarré à l'aide des liens suivants
![Images](images/akhq-restart.png)

[URL AKHQ](https://akhq-monitoring.app.dev.integration.ia.iafg.net/ui/dev/connect/connect-journalisation-k8splatform/definition/es-sink-journalisation-client-system/tasks)

### ElasticSearch était hors service et en problème. Travaux afin de régulariser le tout.

La problématique était le nombre de fields autorisé dans l'index.

![Images](images/jsonaddfields.png)

Il faut ajouter ceci dans "Edit settings" de l'index.

### Supprimer à la source (Sur les fluentD) les logs du flow (Systems Logs) afin d'effectuer de l'espace pour éviter une panne.

Commande pour supprimer à la source directement dans le container fluentd :

```console
kubectl exec -i -t -n cattle-logging-system rancher-logging-fluentd-2 -c fluentd -- sh -c "clear; (bash || ash || sh)"
```

```console
rm -rf /buffers/clusterflow:cattle-logging-system:system-logs:clusteroutput*
```

**Ne jamais effectuer la commande !!! Demander à plateforme si jamais.**

### Supprimer le contenu du topic afin de libérer de l'espace.

Dans AKHQ : [Lien](https://akhq-monitoring.app.dev.integration.ia.iafg.net/ui/dev/topic/Corp_InfraOper_k8splatform_client_system_log_DEV/data?sort=Oldest&partition=All)

![Images](images/emptytopic.png)

### Recréer le topic Kafka

Effectué par l'équipe COE Intégration

## Solutions appliquées

La solution appliquée fut de recréer le topic Kafka et effectuer un rebanlancement de celui-ci.

Suite à cela, le problème fut résolu définitivement
