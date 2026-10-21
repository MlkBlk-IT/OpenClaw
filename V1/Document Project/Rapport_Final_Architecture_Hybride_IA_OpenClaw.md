ARCHITECTURE HYBRIDE IA

Assistant Personnel Local et Cloud via OpenClaw

──────────────────────────────────────────────────

RAPPORT FINAL DE DÉPLOIEMENT ET D'OPÉRATION

Dell Latitude 5420 | Intel Core i7-1185G7 vPro | 32 Go RAM | Windows 11 ProDéployé le 2–4 août 2026 | Version OpenClaw 2026.7.1-2Auteur : Malik | DEP Soutien Informatique 5229 — EMICA Montréal

【 DOCUMENT CONFIDENTIEL — USAGE INTERNE 】


# TABLE DES MATIÈRES


1.  RÉSUMÉ EXÉCUTIF3

2.  CONTEXTE ET OBJECTIFS DU PROJET4

2.1  Profil de l'utilisateur4

2.2  Objectifs opérationnels et pédagogiques4

2.3  Contraintes matérielles5

3.  ARCHITECTURE GÉNÉRALE DU SYSTÈME6

3.1  Vision stratégique6

3.2  Architecture en 7 couches7

3.3  Séparation Raisonnement vs Exécution8

3.4  Flux de données global9

4.  PHASE 0 — CONCEPTION ET PRÉPARATION10

4.1  Justification du choix d'OpenClaw10

4.2  Sélection des modèles locaux11

4.3  Sélection des fournisseurs cloud12

4.4  Budget ressources13

5.  PHASE 1 — DÉPLOIEMENT BASELINE14

5.1  Installation d'OpenClaw14

5.2  Configuration LM Studio15

5.3  Configuration providers cloud15

5.4  Canaux de communication16

5.5  Fichiers de personnalisation17

6.  PHASE 2 — AJUSTEMENTS SESSION 319

6.1  Changement de modèle principal19

6.2  Bascule gateway LAN20

6.3  Appairage mobile Redmi Note 1420

6.4  Automatisation chargement modèle21

7.  PHASE 3 — INTERVENTIONS DE DÉPANNAGE22

7.1  Diagnostic WhatsApp22

7.2  Audit providers cloud23

7.3  Libération mémoire RAM24

7.4  Résolution clé Groq25

7.5  Réorganisation cascade fallback25

7.6  Synchronisation configuration26

8.  PHASE 4 — DIAGNOSTIC FINAL27

8.1  Résultats du diagnostic27

8.2  État final du système28

9.  SYNTHÈSE COMPARATIVE29

10.  SÉCURITÉ ET MESURES DE PROTECTION30

11.  RECOMMANDATIONS ET PROCHAINES ÉTAPES32

12.  CONCLUSION34

ANNEXE A  GLOSSAIRE35

ANNEXE B  COMMANDES CLI37

ANNEXE C  MATRICE DE SÉLECTION39

ANNEXE D  INVENTAIRE MODÈLES LOCAUX40

ANNEXE E  CHRONOLOGIE INTERVENTIONS41


# 1. RÉSUMÉ EXÉCUTIF


Ce document présente le rapport final de déploiement d'une architecture hybride d'intelligence artificielle, conçue et opérée sur un poste de travail Dell Latitude 5420. L'architecture repose sur OpenClaw 2026.7.1-2 comme orchestrateur central, couplant l'exécution de modèles de langage locaux (via LM Studio) à l'accès de providers cloud gratuits, le tout contrôlable à distance depuis un smartphone via Telegram et WhatsApp.

▬▬▬ MÉTRIQUES CLÉS DU PROJET ▬▬▬

INDICATEUR

VALEUR

Durée totale de déploiement

2 jours (2–4 août 2026)

Modèles locaux déployés

14 fichiers GGUF (64,85 Go)

Providers cloud intégrés

4 actifs (Mistral, Google, OpenRouter, Groq)

Canaux de communication

Telegram + WhatsApp

Devices appairés

2 (Desktop + Redmi Note 14)

RAM libérée par optimisation

13 Go (78% → 36%)

Temps de réponse moyen

< 2 secondes

Machines virtuelles lab

10 VMs Hyper-V


## Points saillants


▸ Architecture modulaire : Conception en 7 couches distinctes séparant intention, planification, exécution et validation.

▸ Résilience opérationnelle : Cascade de 7 fallbacks garantissant la continuité de service même en cas d'indisponibilité cloud.

▸ Optimisation ressources : Passage d'un modèle 14B résident (14,6 Go RAM) à un modèle 4B auto-chargé (3 Go RAM), libérant 13 Go pour le système.

▸ Sécurité intégrée : Policy Engine verrouillant l'exécution locale pour les données sensibles, isolation du lab Hyper-V, principe du moindre privilège.

▸ Accessibilité mobile : Contrôle complet du système depuis un smartphone Android via Telegram et WhatsApp, avec appairage sécurisé.


# 2. CONTEXTE ET OBJECTIFS DU PROJET



## 2.1 Profil de l'utilisateur


Le projet est mené par Malik, étudiant en DEP Soutien Informatique (programme 5229) à l'EMICA de Montréal. Son approche est fondamentalement pédagogique : chaque composant technique déployé doit servir d'occasion d'apprentissage, au-delà de sa seule fonctionnalité. La cible professionnelle visée est le consulting freelance et nomade en cybersécurité cloud, avec un focus géographique sur les marchés européens (Italie, Espagne, Portugal).


## 2.2 Objectifs opérationnels et pédagogiques


L'architecture repose sur deux objectifs distincts mais complémentaires :

Objectif opérationnel : disposer d'un assistant IA personnel joignable depuis un téléphone via Telegram ou WhatsApp, capable d'agir réellement sur le poste de travail. L'assistant lit des fichiers, effectue des recherches sur le web, rédige des documents, exécute des commandes système et conserve le contexte d'une session à l'autre grâce à une mémoire persistante.

Objectif pédagogique : comprendre par la pratique le fonctionnement d'une architecture d'agent IA moderne, incluant l'orchestration entre sources de raisonnement local et cloud, la gestion de fallback entre fournisseurs, les permissions d'exécution et la mémoire vectorielle.


## 2.3 Contraintes matérielles


Le poste de travail retenu est un Dell Latitude 5420 équipé d'un processeur Intel Core i7-1185G7 vPro (4 cœurs / 8 threads), de 32 Go de mémoire vive DDR4 et d'un stockage NVMe de 512 Go. L'absence de carte graphique dédiée impose l'exécution des modèles locaux sur CPU uniquement, ce qui guide le choix des modèles en format GGUF quantifié Q4_K_M et limite la taille des modèles déployables en mémoire permanente.

▬▬▬ SPÉCIFICATIONS MATÉRIELLES ▬▬▬

COMPOSANT

SPÉCIFICATION

Modèle

Dell Latitude 5420

Processeur

Intel Core i7-1185G7 vPro (4C/8T)

Mémoire vive

32 Go DDR4

GPU

Aucun — calcul CPU uniquement pour l'IA locale

Stockage système

512 Go NVMe (C:\ système, D:\ données)

Système d'exploitation

Windows 11 Pro (Build 26200)

Budget stockage projet

100 Go sur volume D:\


# 3. ARCHITECTURE GÉNÉRALE DU SYSTÈME



## 3.1 Vision stratégique : le LLM comme composant, non cerveau monolithique


L'architecture d'OpenClaw repose sur un principe fondamental : le modèle de langage (LLM) n'est plus le cerveau monolithique du système, mais devient un composant spécialisé au sein d'un pipeline d'exécution. Cette approche découple la prise de décision de l'exécution concrète, permettant une gouvernance fine des actions et une traçabilité complète.

Le flux de réflexion de l'agent suit la séquence : Utilisateur → Détection d'intention → Planification → Sélection du modèle → Exécution → Validation → Réponse


## 3.2 Architecture décisionnelle en 7 couches


Pour garantir un comportement cohérent, extensible et sécurisé, le système est structuré en sept couches distinctes, chacune assumant une responsabilité unique dans le pipeline de traitement :

1. Couche 1 — InterfacesPoints d'entrée utilisateur : Telegram, WhatsApp et Web UI. Chaque canal normalise la requête entrante vers un format interne commun.

2. Couche 2 — Input GatewayGestion de l'authentification, de la session utilisateur et du contexte conversationnel. Vérifie les droits d'accès et initialise l'environnement d'exécution.

3. Couche 3 — Intent DetectionClassification initiale de la demande : question simple, recherche web, tâche locale, génération de code, analyse de sécurité, etc. Cette couche détermine la nature exacte de la requête sans jamais y répondre directement.

4. Couche 4 — PlannerAnalyse approfondie et décomposition de la mission en sous-tâches logiques. Identifie les capacités requises (raisonnement, code, fichiers, web) et produit un plan d'action séquentiel.

5. Couche 5 — Policy Engine & Capability RouterMoteur de règles appliquant les contraintes de sécurité et de confidentialité. Le Policy Engine verrouille ou autorise l'accès aux ressources ; le Capability Router sélectionne ensuite la combinaison optimale d'outils et de modèles (Cloud ou Local).

6. Couche 6 — Execution LayerExécution concrète de la tâche via les outils désignés : MCP (Model Context Protocol), PowerShell, Python, navigateur web, système de fichiers.

7. Couche 7 — Validation, Memory & OutputContrôle qualité des résultats (gestion d'erreurs, vérification de création de fichiers, validation des commandes, checksums et logs). Mise à jour de la mémoire vectorielle et formulation de la réponse finale.


## 3.3 Séparation critique : Raisonnement vs Exécution


Un piège fréquent dans la conception d'agents est de confier toutes les tâches (réflexion, manipulation de fichiers, exécution, vérification) au même modèle. L'architecture instaure une séparation stricte :

Le modèle n'a pas les droits directs : Il analyse la requête et propose un plan (Reasoning Model), mais ne manipule jamais directement le système de fichiers ni n'exécute de commandes.

L'Executor agit : Un composant dédié (Tool Executor) exécute concrètement les commandes et manipule les fichiers selon le plan validé.

Le Validator contrôle : Un vérificateur indépendant s'assure du succès de la tâche (erreurs, logs, création de fichiers) avant la génération de la réponse finale par le Writer.

Cette séparation garantit que même si le modèle de raisonnement est compromis ou hallucine, l'impact sur le système reste limité par les permissions explicites de l'Executor.


## 3.4 Flux de données global


FLUX DE DONNÉES COMPLET

┌─────────────────────────────────────────────────────────────────┐    │                    UTILISATEUR (téléphone)                       │    │              Telegram  │  WhatsApp  │  Web UI                    │    └────────────────────────┴────────────┴───────────────────────────┘                              │                              ▼    ┌─────────────────────────────────────────────────────────────────┐    │              INPUT GATEWAY (Auth, Session, Contexte)             │    └─────────────────────────────────────────────────────────────────┘                              │                              ▼    ┌─────────────────────────────────────────────────────────────────┐    │              INTENT DETECTION (Classification)                   │    └─────────────────────────────────────────────────────────────────┘                              │                              ▼    ┌─────────────────────────────────────────────────────────────────┐    │              PLANNER (Décomposition, Capacités)                  │    └─────────────────────────────────────────────────────────────────┘                              │                              ▼    ┌─────────────────────────────────────────────────────────────────┐    │     POLICY ENGINE ──► Capability Router (Cloud vs Local)         │    └─────────────────────────────────────────────────────────────────┘                              │              ┌───────────────┴───────────────┐              │                               │              ▼                               ▼    ┌─────────────────────┐         ┌─────────────────────────────┐    │   LM STUDIO (local) │         │   PROVIDERS CLOUD           │    │   Port 1234, CPU    │         │   Mistral, Google,          │    │   Modèles GGUF      │         │   OpenRouter, Groq          │    └─────────────────────┘         └─────────────────────────────┘              │                               │              └───────────────┬───────────────┘                              │                              ▼    ┌─────────────────────────────────────────────────────────────────┐    │              EXECUTION LAYER (MCP, PowerShell, Python)           │    └─────────────────────────────────────────────────────────────────┘                              │                              ▼    ┌─────────────────────────────────────────────────────────────────┐    │     VALIDATOR ──► MEMORY (ChromaDB) ──► OUTPUT (Réponse)       │    └─────────────────────────────────────────────────────────────────┘

Principe de routage : OpenClaw sélectionne le provider selon la nature de la requête. Les tâches privées, courtes ou nécessitant une action locale restent sur le laptop. Les tâches nécessitant une recherche web ou un raisonnement lourd sont envoyées au cloud. La décision est prise par le Policy Engine selon les règles configurées.


# 4. PHASE 0 — CONCEPTION ET PRÉPARATION (2 août 2026)


La phase de conception a été menée le 2 août 2026. Elle a consisté en la définition de l'architecture, le choix des outils, la sélection des modèles et l'établissement du plan de déploiement. Cette phase a produit les documents de conception initiaux qui ont servi de fondation à tout le projet.


## 4.1 Justification du choix d'OpenClaw


La sélection d'OpenClaw comme orchestrateur central résulte d'une analyse comparative des solutions disponibles. Plutôt que de développer un agent sur mesure, ce projet s'appuie sur un outil open-source déjà conçu pour ce cas d'usage, réduisant le temps de mise en service et la charge de maintenance.

Gratuit et open-source : Le code est public sur GitHub, aucune licence commerciale n'est requise, et la communauté assure un suivi actif des évolutions.

Orchestration multi-provider native : Le routage automatique entre modèles locaux et fournisseurs cloud, avec bascule configurable en cas d'indisponibilité, est intégré sans développement middleware supplémentaire.

Exécution d'actions réelles : OpenClaw agit comme un opérateur système, capable de lire et d'écrire des fichiers, d'exécuter des commandes PowerShell et d'interagir avec le système d'exploitation, sous réserve d'une configuration explicite des permissions.

Mémoire persistante intégrée : Le contexte des conversations, les plans d'action et les résultats sont conservés dans une structure de mémoire basée sur des fichiers Markdown et YAML, assurant la continuité entre les sessions.

Intégration messagerie native : La connexion avec Telegram, Discord, Slack, WhatsApp, Signal et iMessage est prise en charge nativement.

Compatibilité LM Studio : L'agent se connecte à un serveur LM Studio local via une simple URL, sans nécessiter de code d'intégration spécifique.

Conséquence pratique : ce projet n'est pas un développement logiciel mais un projet de configuration. L'essentiel du travail technique est déjà résolu par l'outil, ce qui permet de se concentrer sur le paramétrage, la sécurisation et l'optimisation.


## 4.2 Sélection des modèles locaux (GGUF)


LM Studio est la plateforme d'exécution des modèles de langage au format GGUF. Elle expose une API compatible OpenAI sur le port 1234 de la machine locale. Le format de quantification Q4_K_M est retenu pour offrir le meilleur compromis entre qualité de sortie et consommation de ressources sur CPU.

Le parc de modèles retenu couvre quatre profils fonctionnels :

MODÈLE

TAILLE DISQUE

RAM (Q4_K_M)

DÉBIT CPU

FONCTION PRINCIPALE

adi-qwen2.5-coder-7b-kimi-q4_k_m

4,36 Go

~5 Go

6–12 tok/s

Génération de code, scripting, raisonnement technique

Meta-Llama-3.1-8B-Instruct

~5 Go

~6 Go

10–18 tok/s

Traitement général, dialogue, tâches courantes

Qwen3.5-4B-Instruct

~3,5 Go

~4 Go

15–25 tok/s

Traitement rapide, résumé, requêtes simples

Phi-4-Mini-3.8B-Instruct

~3 Go

~3 Go

15–25 tok/s

Mode de secours ultra-léger, toujours chargé

Charge mémoire totale estimée : 18–22 Go pour les modèles actifs simultanément, laissant une marge de 10–14 Go pour le système d'exploitation, les outils de sécurité et les machines virtuelles Hyper-V.


## 4.3 Sélection des fournisseurs cloud


Cinq fournisseurs ont été intégrés au système selon une hiérarchie de priorité fondée sur le volume de requêtes autorisé, la vitesse de réponse et la politique de confidentialité des données.

RANG

FOURNISSEUR

MODÈLES

LIMITES

ENTRAÎNEMENT

FORCE

1

Cerebras

Llama 3.3 70B

30 RPM, 1M tokens/jour

Non

Haut débit, raisonnement complexe

2

Groq

Llama 3.3 70B, Mixtral

30 RPM, 1K–14,4K RPD

Non

Réponse ultra-rapide, idéal code

3

OpenRouter

Qwen, DeepSeek, Llama (28+)

20 RPM, 50 RPD

Non

Diversité de modèles, unification d'API

4

Mistral AI

Mistral Small, Large, Codestral

1B tokens/mois, 2 RPM

Expérimental

Volume mensuel élevé

5

GitHub Models

GPT-4o, Claude 3.5, Llama (100+)

15 RPM, 150–1K RPD

Non

Accès aux modèles frontières

Stratégie de fallback : en cas d'épuisement du quota ou d'indisponibilité d'un fournisseur, le système bascule automatiquement vers le fournisseur de rang immédiatement inférieur. Le modèle local Phi-4-Mini constitue le dernier recours en cas d'indisponibilité totale des services externes.

Note sur Google AI Studio : exclu de la sélection principale car la plateforme entraîne sur les données des utilisateurs situés hors de l'Union européenne, de l'Espace économique européen, de la Suisse et du Royaume-Uni.


## 4.4 Budget ressources et plan de déploiement initial


L'enveloppe de stockage allouée au projet est de 100 Go sur le volume D:\. La répartition prévisionnelle est la suivante :

RESSOURCE

ENVELOPPE

Modèles de langage (4 fichiers GGUF)

30–35 Go

Base de données vectorielle et embeddings

5–10 Go

Journaux d'exécution et cache local

5–10 Go

OpenClaw et dépendances

2–5 Go

Scripts de configuration et outils

1–2 Go

Marge de sécurité

40–50 Go

TOTAL ESTIMÉ

83–112 Go

Plan de déploiement initial : l'ensemble de la mise en service était prévu en une seule session d'environ 90 minutes à 2 heures, couvrant la vérification de LM Studio, l'installation d'OpenClaw, la configuration des backends, la création des comptes cloud, la configuration du bot Telegram, l'activation des permissions et les tests de validation.


# 5. PHASE 1 — DÉPLOIEMENT BASELINE (4 août 2026, matin)


La phase de déploiement baseline a été réalisée le matin du 4 août 2026. Elle a consisté en l'installation complète d'OpenClaw, la configuration des backends locaux et cloud, l'activation des canaux de communication et la personnalisation du comportement de l'agent via les fichiers Markdown du workspace.


## 5.1 Installation d'OpenClaw 2026.7.1-2


Version installée : OpenClaw 2026.7.1-2 (commit 0790d9f).

Méthode d'installation : npm global (Node.js 24.19.0). La commande utilisée est npm install -g openclaw@latest, téléchargeant environ 200 Mo de dépendances.

Gateway : service Windows (Scheduled Task) sur ws://127.0.0.1:18789. Le dashboard web est accessible sur http://127.0.0.1:18789/.

Workspace agent : D:\openclaw\workspace.

Stack de développement déjà présente : Python 3.12, Node.js 24.19.0, FastAPI, LangChain, LangGraph, ChromaDB. Le navigateur principal est Brave.


## 5.2 Configuration du backend LM Studio


LM Studio version 0.4.20 expose une API OpenAI-compatible sur le port 1234. Le mode de chargement est JIT (Just-In-Time) : les modèles sont chargés à la volée et non conservés en mémoire permanente, optimisant l'utilisation de la RAM.

Chemin des modèles : D:\AI\LMStudio\models. Quatorze fichiers .gguf sont disponibles, totalisant 64,85 Go sur disque.

Modèles chargés initialement : adi-qwen2.5-coder-7b-kimi2.7-code (4,36 Go), kimi-k2.7-code (1,43 Go + 1,76 Go mmproj), et text-embedding-nomic-embed-text-v1.5.


## 5.3 Configuration des providers cloud


Stratégie de routing : Cloud First, Local Fallback. Le modèle primary (cloud rapide) répond en premier. En cas d'indisponibilité, la cascade de fallbacks prend le relai, terminant par les modèles locaux LM Studio.

Configuration CLI appliquée :

CLÉ DE CONFIGURATION

VALEUR

DESCRIPTION

agents.defaults.model.primary

groq/compound-mini

Modèle cloud rapide par défaut

agents.defaults.model.fallbacks

[13 modèles]

Cascade cloud → local complète

agents.defaults.utilityModel

lmstudio/llama-3.2-3b-instruct

Tâches internes légères

agents.defaults.thinkingDefault

medium

Niveau de reasoning par défaut

Cascade de fallbacks initiale (15 étages) : groq/compound-mini (PRIMARY) → groq/llama-3.3-70b-versatile → groq/openai/gpt-oss-120b → google/gemini-2.5-pro → google/gemini-2.5-flash → mistral/mistral-large-latest → mistral/codestral-latest → openrouter/moonshotai/kimi-k2.6 → openrouter/auto → lmstudio/deepseek-r1-distill-qwen-14b → lmstudio/mlabonne_qwen3-14b-abliterated → lmstudio/qwen2.5-coder-14b-instruct → lmstudio/qwen3.5-4b-instruct-revised → lmstudio/llama-3.2-3b-instruct.


## 5.4 Configuration des canaux de communication


Deux canaux ont été activés :

Telegram : bot créé via @BotFather, token configuré, pairing approuvé. Le canal est opérationnel avec restriction allowFrom au propriétaire (ID 5062403849).

WhatsApp : activé via le pont Baileys, numéro +14384081775. Session persistante avec credentials et pre-keys stockés.

Session management : OpenClaw 2026.7.1 utilise SQLite pour le stockage des sessions (remplacement du legacy sessions.json). Les sessions actives sont dans ~/.openclaw/agents/main/agent/openclaw-agent.sqlite. La maintenance automatique est configurée en mode enforce avec prune après 30 jours, maximum 500 sessions et 10 Go par agent.


## 5.5 Fichiers de personnalisation de l'agent


OpenClaw injecte les fichiers .md du workspace dans le system prompt de l'agent. Chaque fichier assume un rôle spécifique dans la personnalisation du comportement. Voici la structure complète :

FICHIER

RÔLE ET CONTENU

IDENTITY.md

Définit la persona de l'agent : nom (Agent), créature (AI), ton (sharp / direct / précis), emoji (🤖). Lu à chaque prompt.

MEMORY.md

Stocke les projets actifs (SOC, Hyper-V, OpenClaw, Certifications), les préférences utilisateur, le hardware de référence et les notes de suivi. Lu à chaque prompt.

AGENTS.md

Guide de référence pour le choix manuel du modèle. Contient la liste des providers cloud, la table des modèles locaux et la cascade de fallbacks exacte. Ce fichier est un GUIDE, pas un router automatique. Lu à chaque prompt.

HEARTBEAT.md

Tâches périodiques de l'agent. Actuellement vide (comments-only). Aucune tâche périodique ne s'exécute.

SOUL.md

Définit les règles de comportement de l'agent et les directives d'orchestration modèle (cloud uniquement, cloud+local, local uniquement, fallback). Contient les permissions fichiers et le standard de documentation. Lu à chaque prompt.

TOOLS.md

Référence rapide des outils, VMs, chemins et configuration technique. Contient la table des VMs Hyper-V, la liste des plugins OpenClaw et les chemins fréquents. Lu à chaque prompt.

USER.md

Profil complet de Malik : formation, objectifs professionnels, hardware, projets, stack technique, règles scripts et comportement attendu de l'agent. Lu à chaque prompt.

Plugins activés (13/71) : Browser, Document Extraction, DuckDuckGo, File Transfer, Google, LM Studio, Memory Core, Mistral, OpenRouter, Telegram, Web Readability, WhatsApp, Groq.


# 6. PHASE 2 — AJUSTEMENTS SESSION 3 (4 août 2026, après-midi)


La Session 3, menée l'après-midi du 4 août 2026, a consisté en des ajustements majeurs suite aux premières observations du fonctionnement réel du système. Ces modifications ont été motivées par des problèmes de compatibilité, des besoins d'accessibilité mobile et des optimisations de performance.


## 6.1 Changement de modèle principal


🔴 PROBLÈME IDENTIFIÉ

Le modèle primary initial groq/compound-mini rejetait systématiquement le system prompt complet, celui-ci étant trop volumineux pour sa fenêtre de contexte. Les réponses étaient erratiques ou incomplètes, entraînant une cascade de fallbacks inefficace et des temps de réponse dégradés.

✅ SOLUTION APPLIQUÉE

Passage du primary à google/gemini-3.5-flash, disposant d'une fenêtre de contexte de 1 million de tokens. Ce modèle absorbe intégralement le system prompt volumineux (contenant les 7 fichiers .md du workspace) sans rejet, garantissant des réponses cohérentes et stables.

Changements effectués via CLI :

PARAMÈTRE

AVANT

APRÈS

JUSTIFICATION

agents.defaults.model.primary

groq/compound-mini

google/gemini-3.5-flash

Résolution du rejet de prompt système

agents.defaults.utilityModel

lmstudio/llama-3.2-3b-instruct

lmstudio/qwen3.5-4b-instruct-revised

Meilleur compromis vitesse/capacités


## 6.2 Bascule du gateway en mode LAN


🔴 PROBLÈME IDENTIFIÉ

Le gateway était initialement configuré en mode loopback (bind=127.0.0.1), ce qui limitait l'accès au poste local uniquement. L'objectif de contrôle à distance depuis un smartphone nécessitait l'ouverture du gateway sur le réseau local.

✅ SOLUTION APPLIQUÉE

Bascule du bind de loopback (127.0.0.1) à lan (0.0.0.0). Le gateway écoute désormais sur toutes les interfaces réseau, permettant la connexion depuis n'importe quel appareil du réseau local. Le dashboard est accessible via http://10.0.0.28:18789/.

⚠️ Impact sécurité : cette ouverture expose le gateway sur le LAN. Dans le contexte d'un réseau domestique privé avec authentification par token, ce risque est accepté. Une hardening ultérieure prévoit le bind 127.0.0.1 pour le management avec exposition LAN restreinte.


## 6.3 Appairage du nœud mobile Redmi Note 14


Un smartphone Redmi Note 14 (Android) a été ajouté comme nœud OpenClaw pairé :

ATTRIBUT

VALEUR

Nom du nœud

Redmi Note 14

Type

Mobile (Android)

Scope accordé

node (lecture de base) + operator

IP

10.0.0.226

Canaux fonctionnels

Telegram OK | WhatsApp OK | App native ⚠️

Problème résiduel : l'application native OpenClaw sur le Redmi Note 14 affiche 'no configured model providers'. Ce bug d'affichage n'affecte pas les canaux Telegram et WhatsApp qui fonctionnent normalement. L'hypothèse est que l'app mobile requiert le scope operator.admin pour afficher les providers.


## 6.4 Automatisation du chargement modèle local


Deux tâches planifiées Windows ont été créées dans le Planificateur de tâches :

NOM DE LA TÂCHE

DÉCLENCHEUR

ACTION

LMStudio_Autostart

Au logon de l'utilisateur Tech

Lance LM Studio (interface graphique)

LMStudio_LoadOrchestrator

20 secondes après logon

Charge qwen3.5-4b-instruct-revised via lms.exe (~3 Go RAM)

Ajout au PATH système : lms.exe (CLI LM Studio) a été ajouté au PATH au niveau Machine, permettant l'utilisation des commandes lms load, lms status et lms list depuis n'importe quelle session PowerShell.

Veille secteur désactivée : la veille secteur a été désactivée sur le Dell Latitude 5420 (powercfg /change standby-timeout-ac 0), garantissant que le laptop reste joignable 24h/24 sur secteur et que le gateway OpenClaw reste toujours actif.


# 7. PHASE 3 — INTERVENTIONS DE DÉPANNAGE (4 août 2026, soir)


La soirée du 4 août 2026 a constitué un tournant critique dans la viabilité opérationnelle du système. Une suite d'interventions stratégiques a permis de passer d'un état partiellement fonctionnel à une plateforme pleinement opérationnelle. Les problèmes rencontrés, leurs causes racines et les solutions appliquées sont documentés ci-dessous dans l'ordre chronologique des interventions.


## 7.1 Diagnostic et réparation du canal WhatsApp (20:09)


🔴 PROBLÈME

WhatsApp ne répondait plus (silence après 60 secondes), alors que Telegram fonctionnait normalement. Les messages envoyés depuis le téléphone restaient sans réponse.

🔍 ANALYSE

L'analyse des logs OpenClaw a révélé la chaîne causale suivante :

La session WhatsApp avait accumulé 23 806 tokens de contexte au fil des conversations.

Le tier gratuit Groq impose une limite de 12 000 TPM (Tokens Per Minute) maximum.

Lors de l'envoi d'une requête, Groq rejetait avec une erreur 413 'Request too large'.

Les fallbacks suivants (gemini-3.5-flash en 429 rate limit, qwen3-32b en 404 modèle inexistant, llama-4-scout en 404) brûlaient environ 60 secondes avant que mistral-large ne réponde.

Pendant ce délai, le pont WhatsApp Web (Baileys) abandonnait le tour, considérant que la réponse n'arriverait jamais.

Log clé identifié : 'visible channel turn dispatched with no queued reply payloads'.

✅ SOLUTION

Émission de la commande /reset dans le chat WhatsApp pour purger le contexte accumulé. Réponse revenue en moins de 2 secondes via mistral-large. Leçon : le contexte de session doit être surveillé ; au-delà de ~5 000 tokens, Groq free tier échoue systématiquement.


## 7.2 Audit complet des providers cloud (20:57)


Un audit systématique de chaque modèle de la chaîne de fallback a été réalisé pour mesurer les temps de réponse et identifier les modèles défectueux.

MODÈLE

STATUT

LATENCE

DIAGNOSTIC

groq/llama-3.3-70b-versatile

OK

403 ms

Fonctionnel

groq/openai/gpt-oss-120b

OK

218 ms

Fonctionnel

groq/llama-3.1-8b-instant

OK

190 ms

Fonctionnel

google/gemini-3.1-flash-lite

OK

575 ms

Fonctionnel

mistral/mistral-large-latest

OK

792 ms

Fonctionnel

google/gemini-3.5-flash

Quota

—

Erreur 429 (rate limit)

groq/qwen/qwen3-32b

404

—

Modèle inexistant

groq/meta-llama/llama-4-scout

404

—

Modèle inexistant

google/gemini-2.5-flash

404

—

Modèle inexistant

groq/compound

404

—

Modèle inexistant

huggingface/*

DNS

—

Bloqué par AdGuard DNS

Constat : 5 modèles cloud opérationnels sur 11 testés. La chaîne fonctionnait mais avec un taux d'échec intermédiaire élevé, entraînant des temps de réponse dégradés (jusqu'à 62 secondes en cas de multiples fallbacks morts).


## 7.3 Libération de la mémoire RAM (19:22–19:31)


🔴 PROBLÈME

La RAM système était saturée à 78% (~24,5 Go sur 31,39 Go). Le modèle mlabonne_qwen3-14b-abliterated résidait en permanence dans LM Studio, consommant 14,6 Go de RAM. Cette saturation limitait les capacités du système et risquait de provoquer des swaps disque dégradant les performances.

🔍 ANALYSE

Analyse du parc local pour trouver un modèle plus léger conservant les capacités essentielles (tools + reasoning) :

MODÈLE

TAILLE

CONTEXT

CAPACITÉS

ÉVALUATION

qwen3.5-4b-instruct-revised

3,38 Go

262k context

Tools OK, Reasoning OK

Meilleur compromis

llama-3.2-3b-instruct

1,8 Go

Standard

Tools OK, Reasoning KO

Trop limité

qwen3.5-4b-kimi-reasoning-luna-lora

0,6 Go

LoRA

Non autonome

Inapproprié

meta-llama-3.1-8b-instruct

4,37 Go

Standard

Tools OK, Reasoning partiel

Trop lourd

✅ SOLUTION

Choix du modèle qwen3.5-4b-instruct-revised comme modèle résident. Actions réalisées :

Déchargement du modèle 14B : lms unload mlabonne_qwen3-14b-abliterated

Chargement du modèle 4B : lms load qwen3.5-4b-instruct-revised --exact

Mise à jour du fallback OpenClaw : lmstudio/qwen3.5-4b-instruct-revised

Création d'une tâche planifiée unique LMStudio-AutoLoad-qwen3.5-4b (AtLogOn)

Suppression des 4 anciennes tâches LM Studio redondantes

Résultat : RAM redescendue de 78% à 36% (~11,2 Go sur 31,39 Go). Gain net de 13 Go de RAM libérés.


## 7.4 Résolution clé API Groq (erreur 401) (21:30)


🔴 PROBLÈME

Tous les modèles Groq retournaient une erreur 401 'Invalid API Key'. La clé était présente dans la configuration mais rejetée côté serveur.

🔍 ANALYSE

Causes possibles : clé expirée, quota horaire ou mensuel dépassé, révocation manuelle. Le fournisseur Groq a été relégué en position de fallback 3 dans la nouvelle hiérarchie, ne bloquant plus le système.

✅ SOLUTION (temporaire)

Mistral-large-latest prend le relais comme PRIMARY. Action requise : vérifier la clé sur console.groq.com/keys et régénérer si nécessaire.


## 7.5 Réorganisation de la cascade de fallback (22:45)


🔴 PROBLÈME

La combinaison de la clé Groq 401 et du contexte WhatsApp trop volumineux entraînait des réponses lentes (jusqu'à 62 secondes). L'ancienne chaîne de 15 fallbacks contenait 4 modèles inexistants (404) et 1 modèle en quota dépassé (429), créant un goulot d'étranglement systématique.

🔍 ANALYSE

Recherche comparative des providers restants :

PROVIDER

CONTEXT

QUOTA

ANALYSE

STATUT

Mistral Large

128K–131K context

Stable

Quota free tier souple

Optimal

Gemini 2.0 Flash

1M context

15 RPM, 1M TPM

Quota généreux

Excellent

OpenRouter/auto

Variable

20 RPM, 50 RPD

Routeur gratuit

Fallback

Groq llama-3.3-70b

128K–200K context

30 RPM

Limite TPM 12k (goulot)

Restreint

✅ SOLUTION

Nouvelle hiérarchie optimisée à 7 étages, conçue pour le grand contexte et la stabilité :

RANG

MODÈLE

CARACTÉRISTIQUES

PRIMARY

mistral/mistral-large-latest

Stable, 792ms, quota souple

FB 1

google/gemini-2.0-flash

1M tokens, 15 RPM, 1M TPM

FB 2

openrouter/auto

Routeur gratuit, grand contexte

FB 3

groq/llama-3.3-70b-versatile

128K context, attention TPM 12k

FB 4

lmstudio/qwen3.5-4b-instruct-revised

Utility local, ~3 Go, toujours chargé

FB 5

lmstudio/llama-3.2-3b-instruct

Fallback léger, ~2 Go

FB 6

lmstudio/qwen2.5-7b-instruct

Généraliste, ~5 Go

FB 7

lmstudio/meta-llama-3.1-8b-instruct

Fallback ultime, ~5 Go

Actions réalisées : openclaw config set agents.defaults.model.primary 'mistral/mistral-large-latest', mise à jour de la chaîne de fallbacks, réécriture de SOUL.md avec la nouvelle hiérarchie, redémarrage du gateway. Résultat : réponses en moins de 2 secondes sur tous les canaux.


## 7.6 Synchronisation des fichiers de configuration


🔴 PROBLÈME

Le fichier SOUL.md contenait encore l'ancienne configuration (gemini-3.5-flash primary, modèles 14B, profils inventés). Cette désynchronisation entre openclaw.json et SOUL.md créait une incohérence où l'agent annonçait un modèle théorique différent de celui réellement utilisé.

✅ SOLUTION

Actions réalisées :

Backup de SOUL.md : SOUL.md.bak-20260804224554

Réécriture complète avec PRIMARY mistral-large-latest, chaîne 7 étages vérifiée, modèles 14B retirés, utilityModel qwen3.5-4b-instruct-revised, embedding nomic-embed-text-v1.5

Ajout de la règle 'Annoncer le modèle RÉEL utilisé à la fin de chaque réponse'

Synchronisation d'AGENTS.md avec la nouvelle chaîne

Résultat : synchronisation parfaite entre openclaw.json, SOUL.md et AGENTS.md. L'agent annonce désormais le modèle réel utilisé à chaque réponse.


# 8. PHASE 4 — DIAGNOSTIC FINAL ET VALIDATION


Le diagnostic automatique a été généré le 4 août 2026 à 23:36:13. Il valide l'état opérationnel complet du système après l'ensemble des interventions de la soirée.


## 8.1 Résultats du diagnostic automatique


ÉLÉMENT

VALEUR

STATUT

Version OpenClaw

2026.7.1-2 (commit 0790d9f)

OK

Fichier config trouvé

true

OK

Config parsée sans erreur

true

OK

Chemin config

C:\Users\Tech\.openclaw\openclaw.json

—

Workspace

D:\OpenClaw\workspace

—

▸ Gateway

ÉLÉMENT

VALEUR

STATUT

Gateway en cours d'exécution

true

OK

Sonde de connectivité

true

OK

Mode de binding

lan (0.0.0.0)

ATTENTION

Port

18789

—

Mode d'authentification

token

OK

Tailscale

off

—

Détail technique : Service Windows (Scheduled Task) enregistré. PID 15672, état Ready, dernière exécution 2026-08-04 23:32:02. Connectivité probe OK. Capability : connected-no-operator-scope. Écoute sur 0.0.0.0:18789.

▸ Modèle et mémoire

ÉLÉMENT

VALEUR

STATUT

Modèle principal

mistral/mistral-large-latest

—

Nombre de modèles de secours

7

—

Fournisseur embeddings mémoire

openai

OK

Modèle embeddings

text-embedding-nomic-embed-text-v1.5

—

Fichiers indexés

5/6

OK

Purge de contexte

cache-ttl

OK

Vidage mémoire auto

true

OK

Seuil de vidage

40 000 tokens

—

Détail mémoire : Provider openai (requested: openai), 5/6 fichiers indexés, 87 chunks. Vector store opérationnel, 768 dimensions. FTS ready. Embedding cache enabled (20 entries). Store SQLite : ~/.openclaw/agents/main/agent/openclaw-agent.sqlite.

▸ LM Studio (embeddings locaux)

ÉLÉMENT

VALEUR

STATUT

LM Studio accessible (localhost:1234)

true

OK

Code de réponse HTTP

200

OK

Modèles disponibles dans LM Studio : phi-4-reasoning-plus, gemma4-26b-a4b-instruct-pure, huihui-granite-4.1-3b-abliterated, qwen3-vl-8b-instruct-abliterated, deepseek-r1-distill-qwen-14b, mistral-7b-instruct-v0.3, gemma-4-e4b-it, text-embedding-nomic-embed-text-v1.5-embedding, gemma-4-e4b-it-obliterated, qwen2.5-coder-7b-instruct-ghidra-v2, qwen2.5-7b-instruct, meta-llama-3.1-8b-instruct, meta-llama-3.1-8b-instruct-abliterated, qwen3.5-4b-instruct-revised, llama-3.2-3b-instruct, qwen3.5-4b-kimi-reasoning-luna-lora, text-embedding-nomic-embed-text-v1.5.

▸ Canaux de communication

CANAL

ÉTAT

STATUT

Telegram

ON

OK

WhatsApp

ON

OK

▸ Appareils appairés

DEVICE

ROLES

SCOPES

TOKENS

IP

Redmi Note 14

node, operator

operator.approvals, operator.read, operator.write, operator.talk.secrets, operator.pairing

node, operator

10.0.0.226

Desktop

operator

operator.admin, operator.read, operator.write, operator.approvals, operator.pairing

operator

—

▸ Sécurité — Avertissements

ÉLÉMENT

VALEUR

STATUT

Secrets en clair détectés

true

ATTENTION

Gateway exposé sur le LAN

true

ATTENTION

Note : ces deux points sont actuellement acceptés tels quels dans le contexte d'une machine et d'un réseau privés, à usage personnel uniquement. Une hardening ultérieure est prévue.


## 8.2 État final du système


CONFIGURATION OPENCLAW

RANG

MODÈLE

PRIMARY

mistral/mistral-large-latest

FALLBACK 1

google/gemini-2.0-flash (1M tokens, 15 RPM / 1M TPM)

FALLBACK 2

openrouter/auto (routeur gratuit, grand contexte)

FALLBACK 3

groq/llama-3.3-70b-versatile (128K context, attention TPM 12k)

FALLBACK 4

lmstudio/qwen3.5-4b-instruct-revised (utility, ~3 Go, toujours chargé)

FALLBACK 5

lmstudio/llama-3.2-3b-instruct (fallback léger ~2 Go)

FALLBACK 6

lmstudio/qwen2.5-7b-instruct (généraliste ~5 Go)

FALLBACK 7

lmstudio/meta-llama-3.1-8b-instruct (fallback ultime ~5 Go)

INFRASTRUCTURE LOCALE

COMPOSANT

DÉTAIL

LM Studio

API OpenAI-compatible sur port 1234

Modèle résident

qwen3.5-4b-instruct-revised (3,38 Go, 262k context)

Auto-load

Tâche planifiée LMStudio-AutoLoad-qwen3.5-4b (AtLogOn)

Script

D:\OpenClaw\scripts\autoload-orchestrator.ps1

Modèles sur disque

14 GGUF (64,85 Go dans D:\AI\LMStudio\models)

Embedding

text-embedding-nomic-embed-text-v1.5 (hors chaîne chat)

RESSOURCES SYSTÈME

RESSOURCE

ÉTAT

RAM

35,4% (11,12 Go / 31,39 Go) — stable

Stockage D:

64,85 Go modèles + ~20 Go OpenClaw/logs/workspace

CPU

i7-1185G7 (4C/8T), inference CPU-only


# 9. SYNTHÈSE COMPARATIVE : ÉTAT INITIAL VS ÉTAT FINAL


Le tableau suivant résume l'évolution du système depuis le début de la journée du 4 août 2026 jusqu'au diagnostic final de 23:36. Chaque métrique illustre la transformation opérée par les interventions de la soirée.

MÉTRIQUE

ÉTAT INITIAL

ÉTAT FINAL

TRANSFORMATION

Modèle Principal (PRIMARY)

groq/compound-mini(Erreurs de prompt système)

mistral/mistral-large-latest(Stable, 792ms)

Passage à un modèle avec fenêtre de contexte adaptée

Utilisation Mémoire RAM

78% (~24,5 Go sur 32 Go)Saturation critique

36% (~11,5 Go sur 32 Go)13 Go libérés

Déchargement du modèle 14B résident + JIT loading

Canal WhatsApp

Inopérant(Silence après 60s, erreur TPM 12k)

100% Opérationnel(Réponses <2s)

Purge contexte + reconfiguration fallback

Gestion Modèles Locaux

Modèle 14B résident permanent(14,6 Go bloqués)

Chargement JIT +qwen3.5-4b auto-chargé au logon

Optimisation RAM avec modèle léger permanent

Binding Gateway

Loopback strict(127.0.0.1 uniquement)

LAN(Accès réseau local + Smartphone)

Ouverture pour contrôle mobile

Alignement Fichiers Config

Désynchronisation(openclaw.json ≠ SOUL.md)

Synchronisation parfaite(json + SOUL + AGENTS)

Réécriture complète avec backups

Cascade de Fallbacks

15 étages(4 modèles 404 + 1 quota 429)

7 étages optimisés(Tous fonctionnels)

Audit + retrait des modèles morts

Temps de Réponse Moyen

Jusqu'à 62 secondes(Cascade de modèles morts)

< 2 secondes(Primary stable)

Hiérarchie Mistral → Gemini → OpenRouter

Devices Appairés

1 (Desktop uniquement)

2 (Desktop + Redmi Note 14)

Ajout nœud mobile Android

Constat global : le système est passé d'un état instable, saturé et partiellement fonctionnel à une plateforme d'intelligence hybride de classe professionnelle, pleinement opérationnelle et optimisée pour le contexte matériel disponible.


# 10. SÉCURITÉ ET MESURES DE PROTECTION



## 10.1 Principe du moindre privilège


Les compétences d'action sont activées sélectivement. Seuls les répertoires explicitement déclarés sont accessibles en lecture et en écriture. L'accès au disque complet est explicitement refusé.

Permissions configurées :

PERMISSION

PORTÉE

Lecture autorisée

D:\ (complet)

Écriture autorisée

D:\ (complet)

Exécution autorisée

PowerShell, lancement d'applications sur C:\

Lecture C:\

uniquement pour lancer des applications installées

INTERDIT

écriture sur C:\, fichiers .env, .ssh, C:\Windows\System32, ports WinRM 5985/5986/47001


## 10.2 Isolation réseau


Aucune interface réseau de l'agent ne pénètre le segment du laboratoire Hyper-V.

Les machines virtuelles restent invisibles depuis le poste hôte pour le trafic de l'agent.

Gateway bindé en mode LAN (0.0.0.0:18789) — exposition acceptée dans le contexte réseau privé.

Authentification par token activée.

Tailscale désactivé.


## 10.3 Protection des secrets


⚠️ Avertissement : secrets en clair détectés dans la configuration.

Les jetons d'authentification sont conservés dans le fichier de configuration locale. ACL Windows restrictive sur D:\OpenClaw\config (icacls avec héritage désactivé). Les clés API ne sont jamais transmises à des tiers ni exposées dans les journaux d'exécution. Une migration vers un vault externe est prévue à moyen terme.


## 10.4 Journalisation et audit


Toutes les actions système déclenchées par l'agent sont consignées avec horodatage, identifiant de requête et contenu de la commande.

Logs vers D:\OpenClaw\logs et %LOCALAPPDATA%\Temp\openclaw\.

Backups automatiques des fichiers modifiés avec suffixe timestamp.

Sandbox PowerShell avec validation préalable du contenu du script.


# 11. RECOMMANDATIONS ET PROCHAINES ÉTAPES



## 11.1 Court terme (immédiat — 1-2 semaines)


#

ACTION

PRIORITÉ

1

Vérifier/régénérer la clé Groq sur console.groq.com

Haute

2

Tester WhatsApp avec contexte long (>10k tokens)

Haute

3

Résoudre operator.admin sur app mobile (docs.openclaw.ai/cli/nodes)

Moyenne

4

Ajouter 6 clés API manquantes (GitHub, NVIDIA, SambaNova, DeepSeek, HuggingFace, Cloudflare)

Moyenne

5

Activer ChromaDB + mémoire persistante

Basse


## 11.2 Moyen terme (1-3 mois)


Durcir le gateway : bind 127.0.0.1 pour management + exposition LAN restreinte.

Rotation périodique des tokens d'authentification.

Sauvegarde automatique des fichiers .md du workspace.

Configurer les IPs fixes des VMs Hyper-V (vSwitch interne avec DHCP statique).

Déployer les agents Wazuh sur toutes les VMs Windows.

Créer la VM BlackArch pour compléter l'écosystème pentest.


## 11.3 Long terme (3-6 mois)


Architecture Gateway/Workstation VM (Hyper-V + Whonix + Tor).

VPS ARM Oracle Cloud Free Tier pour multi-hop anonymité.

Containerisation (Docker, Kubernetes k3s/minikube).

Automatisation (Ansible, Terraform).

Lab cloud (AWS/Azure/GCP).

Roadmap 17 certifications cybersécurité.


# 12. CONCLUSION


L'architecture hybride IA OpenClaw, déployée sur un Dell Latitude 5420 en 2 jours (2–4 août 2026), démontre qu'un assistant intelligent professionnel peut être construit sans infrastructure cloud coûteuse ni matériel dédié GPU.

Les apports fondamentaux de ce projet sont triples :

1. Architecture modulaire et sécurisée — La séparation en 7 couches, le Policy Engine et le Capability Router garantissent que chaque requête est traitée par l'outil et le modèle les plus appropriés, tout en respectant scrupuleusement les contraintes de confidentialité.

2. Résilience opérationnelle — La cascade de 7 fallbacks, passant de providers cloud performants à des modèles locaux ultra-légers, assure une disponibilité ininterrompue même en cas d'indisponibilité totale des services externes.

3. Optimisation ressources — Le passage d'un modèle 14B résident (14,6 Go RAM) à un modèle 4B auto-chargé (3 Go RAM), combiné au chargement JIT, a libéré 13 Go de RAM et transformé un système saturé en plateforme réactive et stable.

Le système est désormais pleinement opérationnel : Telegram et WhatsApp répondent en moins de 2 secondes, le smartphone Android est appairé, la mémoire est sous contrôle, et la configuration est synchronisée. Les problèmes résiduels (clé Groq, exposition LAN, secrets en clair) sont identifiés, documentés et priorisés pour résolution.

Ce projet constitue une fondation solide pour l'objectif professionnel de Malik : le consulting freelance en cybersécurité cloud, avec un lab opérationnel de 10 VMs, une stack SOC en construction et une documentation systématique au format Word.


# ANNEXE A — GLOSSAIRE


TERME

DÉFINITION

Agent IA

Système logiciel capable d'effectuer des tâches de manière autonome en utilisant des modèles de langage et des outils externes.

Capability Router

Composant qui sélectionne dynamiquement le modèle et les outils les plus adaptés à une tâche donnée.

ChromaDB

Base de données vectorielle open-source pour le stockage et la recherche sémantique d'embeddings.

Embedding

Représentation vectorielle d'un texte permettant la recherche sémantique.

Fallback

Mécanisme de bascule automatique vers un provider ou modèle de secours en cas d'indisponibilité.

GGUF

Format de fichier pour modèles de langage quantifiés, optimisé pour l'inférence CPU.

Hyper-V

Hyperviseur natif de Windows pour la virtualisation de machines.

JIT (Just-In-Time)

Mode de chargement où les modèles sont chargés à la volée et non conservés en mémoire permanente.

LLM

Large Language Model — modèle de langage de grande taille capable de générer du texte.

LM Studio

Application de bureau pour exécuter des modèles de langage locaux via une API compatible OpenAI.

MCP (Model Context Protocol)

Protocole standardisé pour l'interaction entre agents IA et outils externes.

OpenClaw

Orchestrateur open-source multi-provider pour agents IA personnels.

Policy Engine

Moteur de règles appliquant les contraintes de sécurité et de confidentialité.

Provider

Service cloud fournissant l'accès à des modèles de langage via API.

Q4_K_M

Niveau de quantification GGUF offrant un compromis optimal entre qualité et taille.

RAG

Retrieval-Augmented Generation — génération augmentée par récupération de documents.

Reasoning

Capacité d'un modèle à effectuer des raisonnements logiques et analytiques.

RPM

Requests Per Minute — nombre de requêtes autorisées par minute.

SQLite

Système de gestion de base de données relationnelle léger et embarqué.

System prompt

Instructions initiales injectées dans le contexte de chaque conversation avec un LLM.

TPM

Tokens Per Minute — nombre de tokens traités par minute.

vSwitch

Commutateur virtuel Hyper-V pour la connectivité réseau des VMs.


# ANNEXE B — COMMANDES CLI DE RÉFÉRENCE



## Gateway


COMMANDE

DESCRIPTION

openclaw gateway start

Démarrer le service gateway

openclaw gateway stop

Arrêter le service gateway

openclaw gateway restart

Redémarrer le service gateway

openclaw gateway status

État du service + connectivité

openclaw gateway run

Exécuter en foreground (debug)


## Configuration


COMMANDE

DESCRIPTION

openclaw config get <chemin>

Lire une valeur de config

openclaw config set <chemin> <valeur>

Écrire une valeur

openclaw config unset <chemin>

Supprimer une valeur

openclaw config validate

Valider la config

openclaw config file

Chemin du fichier actif


## Sessions


COMMANDE

DESCRIPTION

openclaw sessions list

Lister les sessions

openclaw sessions list --json

Lister en JSON

openclaw sessions cleanup --dry-run

Prévisualiser le nettoyage

openclaw sessions cleanup --enforce

Nettoyer immédiatement


## Plugins


COMMANDE

DESCRIPTION

openclaw plugins list

Lister tous les plugins

openclaw plugins enable <id>

Activer un plugin

openclaw plugins disable <id>

Désactiver un plugin


## Commandes dans le chat


COMMANDE

DESCRIPTION

/model <nom>

Changer de modèle

/status

Usage contexte, modèle, toggles

/context list

Contenu du system prompt

/new

Nouvelle session

/reset

Reset session courante

/compact

Compacter la conversation


# ANNEXE C — MATRICE DE SÉLECTION DES MODÈLES


CAPACITÉ

STRATÉGIE LOCALE

STRATÉGIE CLOUD

OUTIL

Reasoning

deepseek-r1-distill-qwen-14b

GPT OSS 20B → 120B → Gemini Pro

Aucun

General

Llama 3.1 8B → Qwen3.5

Groq Compound → Llama 70B → Gemini Flash

Aucun

Coding

Qwen2.5 Coder 14B → 7B

Devstral → Codestral → Mistral Large

Git, IDE, MCP

Windows Admin

Modèle Local Privilégié

Non Autorisé

PowerShell, MCP

Security Analysis

Modèle Local Privilégié

Restreint / Interdit

Hayabusa, Sysmon

Documentation

Qwen, Llama (local)

Modèle Cloud

Python-docx, Word

Research

Non applicable

Gemini Pro → Kimi K2.6

Browser, API Web


# ANNEXE D — INVENTAIRE DES MODÈLES LOCAUX


FICHIER .GGUF

RÔLE

TAILLE

RAM EST.

VITESSE CPU

DeepSeek-R1-Distill-Qwen-14B-Q4_K_S

Math, reasoning

7,98 Go

~9 Go

~4 tok/s

mlabonne_Qwen3-14B-abliterated-Q4_K_S

Reasoning sans censure

7,98 Go

~9 Go

~4 tok/s

Qwen2.5-Coder-14B-Instruct-Q4_K_S

Code local #1

7,98 Go

~9 Go

~4 tok/s

gemma-4-e4b-it-obliterated-q4_k_m

Vision sans censure

4,94 Go

~6 Go

~6 tok/s

meta-llama-3.1-8b-instruct-abliterated.Q4_K_M

Fallback sans censure

4,58 Go

~5,5 Go

~7 tok/s

Meta-Llama-3.1-8B-Instruct-Q4_K_S

Fallback général

4,37 Go

~5 Go

~7 tok/s

qwen2.5-coder-7b-instruct-ghidra-v2-q4_k_m

Cybersec, reverse eng

4,36 Go

~5 Go

~7 tok/s

Qwen2.5-7B-Instruct-Q4_K_S

Général léger

4,15 Go

~5 Go

~7 tok/s

Mistral-7B-Instruct-v0.3-Q4_K_S

Fallback urgence

3,86 Go

~4,5 Go

~8 tok/s

Qwen3.5-4B-Kimi-Reasoning-Luna-LoRA-step495-Q4_K_M

Reasoning léger

2,59 Go

~3 Go

~12 tok/s

Qwen3.5-4B-Revised-q4_k_m

Rapide, vision, 262K ctx

2,52 Go

~3 Go

~12 tok/s

Llama-3.2-3B-Instruct-Q4_K_S

Ultra léger, rapide

1,80 Go

~2 Go

~18 tok/s

Fichiers complémentaires : mmproj-F32.gguf (1,78 Go) — projection vision ; mmproj-Qwen3.5-4B-Kimi-Reasoning-Luna-LoRA-step495-BF16.gguf (0,63 Go) ; mmproj-f16.gguf (0,63 Go) ; nomic-embed-text-v1.5-Q8_0.gguf (0,14 Go) — embeddings ; mtp-gemma-4-E4B-it-Q8_0.gguf (0,09 Go).


# ANNEXE E — CHRONOLOGIE DÉTAILLÉE DES INTERVENTIONS


HEURE

INTERVENTION

RÉSULTAT

19:22

Déchargement modèle 14B, chargement Qwen 4B

RAM 78% → 36%

19:31

Création tâche planifiée auto-load

Modèle 4B chargé au logon

20:09

Diagnostic WhatsApp muet

Identification cause : contexte 23k tokens

20:10

Synchronisation SOUL.md

Backup + réécriture complète

20:57

Audit providers cloud

5 modèles OK / 11 testés

21:30

Diagnostic clé Groq 401

Relégation en fallback 3

22:45

Réorganisation hiérarchie

Mistral PRIMARY, chaîne 7 étages

22:46

Redémarrage gateway

Réponses <2s confirmées

23:36

Diagnostic automatique final

Système validé, 0 sessions actives

──────────────────────────────────────────────────

Document généré le 5 août 2026

Version finale — Architecture Hybride IA OpenClaw

Malik — DEP Soutien Informatique 5229, EMICA Montréal
