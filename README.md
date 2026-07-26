# AI-patrimoine
# Architecture cible de la plateforme agentique

## 1. Retour d’expérience sur la première version

Une première version du projet avait été réalisée principalement avec n8n.

Cette approche a permis de construire rapidement un prototype, mais plusieurs limites sont apparues lorsque le workflow est devenu plus complexe :

* faible visibilité sur le comportement des agents ;
* absence de traçabilité détaillée des décisions ;
* difficulté à comprendre les erreurs ;
* consommation excessive de tokens ;
* appels répétés aux modèles d’intelligence artificielle ;
* workflows complexes et difficiles à maintenir ;
* faible maîtrise de l’état global d’un dossier ;
* difficulté à mesurer les performances ;
* sécurité et gestion des accès insuffisamment centralisées ;
* coût d’exploitation difficile à contrôler.

n8n était utilisé à la fois pour les intégrations, la logique métier et l’orchestration des agents. Ce regroupement a rapidement rendu la solution difficile à optimiser et à faire évoluer.

La nouvelle architecture doit donc clairement séparer les responsabilités.

# 2. Principe général de la nouvelle architecture

La plateforme repose sur trois niveaux principaux :

* n8n pour les intégrations et les automatisations simples ;
* LangChain pour construire les agents ayant accès au système d’information ;
* LangGraph pour orchestrer l’ensemble du processus agentique.

À ces trois composants s’ajoutent :

* une couche d’observabilité ;
* une couche de sécurité ;
* une base de données métier ;
* un système de gestion des droits ;
* une interface de validation humaine.

La logique générale peut être résumée ainsi :

```text
Applications de l’entreprise
Teams, GED, calendrier, patrimoine, finance, comptabilité
                         │
                         ▼
              Couche de sécurité
        Authentification, droits, middleware
                         │
                         ▼
                     LangGraph
          Orchestration et état du dossier
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
       Agents simples          Agents métier
           n8n                 LangChain
             │                       │
             └───────────┬───────────┘
                         ▼
             Systèmes de l’entreprise
                         │
                         ▼
                 Observabilité
       Traces, coûts, erreurs, statistiques
```

# 3. Rôle de n8n

n8n est conservé dans l’architecture, mais son rôle est limité aux tâches d’intégration simples et clairement délimitées.

Un agent ou un workflow peut être développé avec n8n lorsque sa responsabilité est réduite, peu risquée et principalement déterministe.

Exemples :

* récupérer une transcription Teams ;
* déplacer un document vers un espace de stockage ;
* envoyer une notification ;
* transmettre une demande de validation ;
* générer un fichier à partir d’un modèle ;
* créer un événement dans un calendrier ;
* envoyer un courriel ;
* déclencher un webhook ;
* appeler une API externe ;
* mettre à jour un statut dans une base de données.

n8n agit donc comme une couche d’automatisation et de connexion entre les différents services.

Il ne doit pas porter seul :

* la mémoire complète d’un dossier ;
* les décisions complexes ;
* les règles d’accès au patrimoine ;
* les raisonnements longs ;
* les boucles de contrôle ;
* les stratégies multi-agents ;
* la gestion des erreurs métier complexes.

# 4. Rôle de LangChain

LangChain est utilisé pour construire les agents métier ayant besoin d’interagir avec les données et les outils de l’entreprise.

Il peut notamment être utilisé lorsqu’un agent doit :

* interroger le système de gestion du patrimoine ;
* rechercher des informations dans la GED ;
* analyser plusieurs documents ;
* consulter des données financières ;
* comparer une transcription avec un historique de décisions ;
* appeler plusieurs outils successivement ;
* produire une réponse structurée ;
* sélectionner l’outil adapté à une situation ;
* exploiter une base vectorielle ;
* appliquer des règles métier spécifiques.

Exemples d’agents LangChain :

* agent d’analyse des transcriptions ;
* agent de classification rénovation ou acquisition ;
* agent d’analyse du patrimoine ;
* agent spécialisé en rénovation ;
* agent spécialisé en acquisition ;
* agent d’analyse documentaire ;
* agent de contrôle des décisions ;
* agent de préparation des comptes rendus ;
* agent de recherche des personnes habilitées.

Les agents LangChain doivent être construits comme des services indépendants, avec des outils strictement définis.

Chaque agent ne doit avoir accès qu’aux fonctions nécessaires à sa mission.

Par exemple, l’agent d’analyse documentaire peut consulter des documents, mais ne doit pas pouvoir modifier un patrimoine ou créer un engagement financier.

# 5. Rôle de LangGraph

LangGraph constitue le cerveau et l’orchestrateur principal de la plateforme.

Il contrôle :

* les étapes du processus ;
* l’enchaînement des agents ;
* les conditions de passage d’une étape à une autre ;
* la mémoire du dossier ;
* les validations humaines ;
* les erreurs ;
* les reprises après interruption ;
* les branches rénovation et acquisition ;
* les interactions avec n8n ;
* les interactions avec les agents LangChain.

LangGraph doit maintenir un état central pour chaque dossier.

Exemple :

```text
Dossier
├── Identifiant
├── Type d’opération
├── Patrimoine concerné
├── Transcriptions
├── Documents
├── Décisions détectées
├── Décisions validées
├── Actions à réaliser
├── Responsables
├── Participants nécessaires
├── Validations attendues
├── Réunions organisées
├── Niveau de confiance
├── Étape actuelle
└── Historique des événements
```

LangGraph décide ensuite quel composant doit intervenir.

Exemple :

```text
Réception de la transcription
             │
             ▼
Analyse de la transcription
             │
             ▼
Une décision a-t-elle été prise ?
        ┌────┴────┐
        │         │
       Non       Oui
        │         │
        ▼         ▼
Validation     Classification
humaine       du dossier
                  │
          ┌───────┴────────┐
          ▼                ▼
     Rénovation        Acquisition
          │                │
          ▼                ▼
 Agent rénovation    Agent acquisition
          │                │
          └───────┬────────┘
                  ▼
       Vérification des résultats
                  │
                  ▼
       Génération du compte rendu
                  │
                  ▼
      Recherche des disponibilités
                  │
                  ▼
        Validation de l’utilisateur
                  │
                  ▼
      Création de la prochaine réunion
```

# 6. Règle de choix entre n8n et LangChain

Le choix de la technologie dépend de la responsabilité de l’agent.

## Utiliser n8n lorsque :

* la tâche est simple ;
* le workflow est déterministe ;
* le nombre d’étapes est limité ;
* l’agent ne manipule pas de données critiques ;
* aucune mémoire complexe n’est nécessaire ;
* les actions sont facilement vérifiables ;
* l’objectif principal est de connecter des applications.

## Utiliser LangChain lorsque :

* l’agent doit raisonner sur plusieurs sources ;
* l’agent doit sélectionner dynamiquement des outils ;
* l’agent interagit avec le patrimoine de l’entreprise ;
* l’agent manipule des informations sensibles ;
* l’agent doit analyser plusieurs documents ;
* l’agent doit respecter des règles métier complexes ;
* une sortie structurée et contrôlée est nécessaire ;
* les permissions doivent être gérées avec précision.

## Utiliser LangGraph lorsque :

* plusieurs agents doivent collaborer ;
* le processus contient plusieurs branches ;
* le dossier évolue sur plusieurs réunions ;
* l’exécution peut être interrompue puis reprise ;
* une validation humaine est requise ;
* un état persistant doit être conservé ;
* la décision dépend du résultat d’étapes précédentes ;
* le système doit gérer des erreurs et des reprises.

# 7. Couche d’observabilité

L’observabilité est un composant central de la nouvelle architecture.

Elle doit permettre de tracer chaque étape exécutée par le système.

Pour chaque traitement, la plateforme doit pouvoir enregistrer :

* l’identifiant du dossier ;
* l’agent appelé ;
* le modèle utilisé ;
* le prompt utilisé ou sa version ;
* les outils appelés ;
* les documents consultés ;
* la durée du traitement ;
* le nombre de tokens consommés ;
* le coût estimé ;
* le résultat produit ;
* le niveau de confiance ;
* les erreurs rencontrées ;
* les tentatives de relance ;
* les validations humaines ;
* la personne ayant validé ;
* la date et l’heure de chaque action.

## Exemple de trace

```json
{
  "trace_id": "TRACE-2026-00482",
  "dossier_id": "PAT-2026-0017",
  "workflow": "analyse_reunion",
  "agent": "agent_classification",
  "modele": "modele-llm",
  "date_debut": "2026-07-23T10:02:15",
  "duree_ms": 2350,
  "tokens_entree": 4120,
  "tokens_sortie": 620,
  "cout_estime": 0.034,
  "resultat": "RENOVATION",
  "confiance": 0.93,
  "statut": "SUCCES"
}
```

# 8. Statistiques et pilotage

Les traces collectées permettront de construire des tableaux de bord.

La plateforme pourra notamment mesurer :

## Performances techniques

* durée moyenne d’une analyse ;
* taux d’erreur par agent ;
* nombre de relances ;
* disponibilité des services ;
* temps de réponse des outils ;
* taux d’échec des workflows.

## Performances économiques

* coût moyen par dossier ;
* coût moyen par réunion ;
* consommation de tokens par agent ;
* coût par type de document ;
* coût par modèle ;
* coût des traitements échoués ;
* économies réalisées grâce au cache.

## Performances métier

* nombre de décisions extraites ;
* nombre de décisions validées ;
* nombre de corrections humaines ;
* taux de décisions ambiguës ;
* délai moyen entre deux réunions ;
* délai moyen de validation ;
* nombre de dossiers rénovation ;
* nombre de dossiers acquisition ;
* nombre d’actions réalisées dans les délais.

## Qualité des agents

* niveau de confiance moyen ;
* taux de classification correcte ;
* taux de faux positifs ;
* taux de faux négatifs ;
* nombre de comptes rendus corrigés ;
* fréquence des erreurs par type de document ;
* outils les plus utilisés ;
* étapes nécessitant le plus souvent une intervention humaine.

Ces données permettront ensuite de mettre en place des stratégies d’optimisation.

# 9. Stratégies d’optimisation futures

Grâce à l’observabilité, plusieurs optimisations pourront être appliquées.

## Optimisation des coûts

* utiliser un petit modèle pour les tâches simples ;
* réserver les modèles avancés aux décisions complexes ;
* éviter de renvoyer l’intégralité des transcriptions ;
* résumer les anciennes réunions ;
* utiliser un cache pour les résultats déjà calculés ;
* limiter le nombre d’appels successifs ;
* regrouper certaines analyses ;
* arrêter un workflow lorsqu’une information essentielle manque.

## Optimisation des performances

* paralléliser les analyses indépendantes ;
* précharger les documents nécessaires ;
* indexer les documents dans une base vectorielle ;
* utiliser des sorties JSON structurées ;
* réduire la taille des prompts ;
* sélectionner uniquement les passages pertinents ;
* utiliser des règles programmées avant d’appeler un modèle.

## Optimisation de la qualité

* comparer les résultats de plusieurs agents ;
* mettre en place un agent de vérification ;
* vérifier chaque décision avec un extrait de la transcription ;
* mesurer les corrections humaines ;
* améliorer les prompts à partir des erreurs observées ;
* créer des jeux de tests métier ;
* versionner les agents et les prompts.

# 10. Gestion renforcée des accès

L’accès aux outils et aux données doit être contrôlé par une couche de middleware.

Le middleware intervient avant chaque appel à un outil ou à un service.

Il vérifie notamment :

* l’identité de l’utilisateur ;
* l’identité de l’agent ;
* le rôle de l’utilisateur ;
* les permissions de l’agent ;
* le type de dossier ;
* le périmètre du patrimoine concerné ;
* le niveau de sensibilité des données ;
* l’action demandée ;
* la nécessité d’une validation humaine.

## Exemple de contrôle

```text
Agent acquisition
        │
        ▼
Demande d’accès aux données financières
        │
        ▼
Middleware de sécurité
        │
        ├── L’agent est-il autorisé ?
        ├── L’utilisateur est-il habilité ?
        ├── Le dossier est-il dans son périmètre ?
        ├── Les données sont-elles nécessaires ?
        └── Une validation est-elle requise ?
                │
          ┌─────┴─────┐
          ▼           ▼
       Autorisé     Refusé
          │           │
          ▼           ▼
      Appel outil   Journalisation
```

# 11. Principe du moindre privilège

Chaque agent possède uniquement les permissions nécessaires à son rôle.

Exemples :

| Agent                  | Permissions                                                      |
| ---------------------- | ---------------------------------------------------------------- |
| Agent de transcription | Lire et structurer une transcription                             |
| Agent documentaire     | Consulter les documents du dossier                               |
| Agent rénovation       | Consulter les données techniques du patrimoine                   |
| Agent acquisition      | Consulter les données d’acquisition autorisées                   |
| Agent calendrier       | Lire les disponibilités et proposer des créneaux                 |
| Agent de compte rendu  | Générer un document, sans pouvoir le publier seul                |
| Agent financier        | Consulter certaines données financières, sans engager de dépense |

Un agent ne doit jamais recevoir un accès général à l’ensemble du système d’information.

# 12. Middleware complémentaires

Plusieurs middleware peuvent être placés autour des agents.

## Middleware d’authentification

Il vérifie l’identité de l’utilisateur et de l’application.

## Middleware d’autorisation

Il contrôle les rôles et les permissions.

## Middleware de filtrage des données

Il masque ou supprime les données que l’agent n’est pas autorisé à consulter.

## Middleware de validation humaine

Il bloque les opérations sensibles jusqu’à l’accord d’une personne habilitée.

## Middleware de limitation

Il limite :

* le nombre d’appels ;
* la consommation de tokens ;
* le coût maximal par dossier ;
* le temps maximal d’exécution ;
* le nombre de tentatives.

## Middleware d’audit

Il enregistre toutes les actions sensibles.

## Middleware de protection des outils

Il vérifie les paramètres avant d’autoriser un agent à utiliser un outil.

# 13. Actions nécessitant une validation humaine

Les actions suivantes ne doivent pas être exécutées automatiquement :

* validation définitive d’une acquisition ;
* validation d’un budget ;
* engagement d’une dépense ;
* lancement officiel de travaux ;
* modification d’une donnée patrimoniale critique ;
* envoi d’un compte rendu officiel ;
* invitation de personnes externes ;
* transmission d’un document confidentiel ;
* clôture d’un dossier ;
* suppression d’une information.

L’agent peut préparer l’action, mais la décision finale reste humaine.

# 14. Architecture technique synthétique

```text
┌──────────────────────────────────────────────┐
│                Interfaces                    │
│ Application métier, portail, Teams, e-mails │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│         API Gateway et middleware            │
│ Authentification, rôles, filtrage, quotas    │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│                  LangGraph                   │
│ Orchestration, état, branches, validations   │
└──────────────┬─────────────────┬─────────────┘
               │                 │
               ▼                 ▼
┌──────────────────────┐ ┌─────────────────────┐
│ Agents LangChain     │ │ Workflows n8n       │
│ Raisonnement métier  │ │ Intégrations simples│
└───────────┬──────────┘ └──────────┬──────────┘
            │                       │
            └───────────┬───────────┘
                        ▼
┌──────────────────────────────────────────────┐
│ Outils et systèmes d’information             │
│ Patrimoine, GED, finance, calendriers, Teams │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│ Observabilité et audit                       │
│ Traces, coûts, qualité, erreurs, statistiques│
└──────────────────────────────────────────────┘
```

# 15. Résumé de la stratégie

La nouvelle architecture repose sur une séparation claire des responsabilités :

```text
n8n
Automatisation et intégration des tâches simples

LangChain
Construction des agents métier et accès contrôlé aux outils

LangGraph
Orchestration, mémoire, décisions et cycle de vie des dossiers

Middleware
Sécurité, permissions, validation et contrôle des actions

Observabilité
Traçabilité, coûts, performances, qualité et optimisation
```

Cette architecture permet de dépasser les limites rencontrées lors du premier prototype.

Elle rend la plateforme :

* plus maintenable ;
* plus sécurisée ;
* plus observable ;
* plus économique ;
* plus facile à optimiser ;
* plus adaptée aux processus métier complexes ;
* plus fiable pour interagir avec le patrimoine de l’entreprise.
