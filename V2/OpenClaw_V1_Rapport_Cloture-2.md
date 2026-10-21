PROJET OPENCLAW

Version 1 — Rapport de clôture et synthèse technique

Architecture hybride d'agents IA — Orchestration Cloud + Local (LM Studio)

Malik

DEP Soutien Informatique (5229) — EMICA Montréal

Dell Latitude 5420 — Windows 11 Pro — OpenClaw 2026.7.1-2

Période couverte : 2 août – 6 août 2026


# Table des matières



# 1. Vue d'ensemble et objectifs



## 1.1 Contexte du projet


OpenClaw V1 est un système d'agents IA hybride conçu et déployé par Malik, étudiant en DEP Soutien Informatique (programme 5229) à l'EMICA de Montréal, sur un poste de travail personnel Dell Latitude 5420 (Intel Core i7-1185G7 vPro, 32 Go RAM DDR4, sans GPU dédié, Windows 11 Pro build 26200). Le projet combine un orchestrateur OpenClaw (v2026.7.1-2), des modèles cloud gratuits multi-fournisseurs, et un parc de modèles locaux GGUF exécutés via LM Studio, entièrement construit from scratch sur la période du 2 au 6 août 2026.

Le projet répondait à deux objectifs distincts et complémentaires, documentés dès la phase de conception : un objectif opérationnel — disposer d'un assistant IA joignable par téléphone (Telegram, WhatsApp), capable d'agir réellement sur le poste de travail (lecture/écriture de fichiers, recherche web, exécution PowerShell, mémoire persistante) — et un objectif pédagogique — comprendre par la pratique l'architecture d'un agent IA moderne : orchestration cloud/local, gestion de fallback, permissions d'exécution, mémoire vectorielle.


## 1.2 Objectifs initiaux


Déployer un orchestrateur multi-agents (OpenClaw) capable de basculer automatiquement entre providers cloud gratuits et modèles locaux GGUF selon la disponibilité, le quota et la nature de la requête.

Construire un lab de cybersécurité pratique sous Hyper-V (initialement 10 VMs) pour appuyer une feuille de route de 17 certifications, en vue d'un objectif professionnel de consulting freelance en cybersécurité cloud (marchés visés : Italie, Espagne, Portugal).

Établir une gouvernance stricte et documentée du comportement de l'agent via des fichiers Markdown injectés au system prompt (SOUL.md, AGENTS.md, TOOLS.md, USER.md, MEMORY.md, IDENTITY.md, HEARTBEAT.md).

Constituer une méthodologie reproductible : audit avant correction, sauvegardes horodatées systématiques, validation par preuve mesurée plutôt que par auto-évaluation du modèle, documentation professionnelle systématique au format Word.


## 1.3 Portée du document


Ce document couvre le cycle de développement de la Version 1 telle que documentée du 2 au 6 août 2026 : conception initiale, déploiement baseline, ajustements de la Session 3, interventions de dépannage du soir du 4 août, diagnostic final validé, puis l'audit de gouvernance du 6 août qui a mis au jour une faille de fond dans la fiabilité des auto-évaluations de l'agent. Cette dernière découverte, combinée aux limites structurelles accumulées au fil des correctifs, motive la décision de Malik de clore la Version 1 et d'engager une refonte en Version 2.

Statut du projet

Décision de Malik : clôture du développement actif de la Version 1. Le système est fonctionnel et validé par diagnostic automatique (23:36:13, 4 août), et un audit de gouvernance correctif a été livré le 6 août. Le choix est fait de ne pas poursuivre l'itération sur cette base et de concevoir une Version 2 sur des fondations repensées.


# 2. Architecture et méthodes



## 2.1 Vision architecturale


L'architecture d'OpenClaw repose sur un principe central, documenté dès la conception : le LLM n'est pas le cerveau monolithique du système mais un composant spécialisé au sein d'un pipeline découplé. Le flux de traitement suit la séquence Utilisateur → Intent Detection → Planning → Model Selection → Execution → Validation → Response, structurée en 7 couches (Interfaces, Input Gateway, Intent Detection, Planner, Policy Engine & Capability Router, Execution Layer, Validation/Memory/Output).

Un Policy Engine verrouille l'exécution locale pour les données sensibles (logs de sécurité, système de fichiers) et autorise l'accès cloud pour la recherche documentaire externe. Cette séparation Raisonnement / Exécution / Validation garantit qu'un modèle compromis ou hallucinant reste limité par les permissions explicites de l'Executor — principe repris et durci lors de l'audit de gouvernance du 6 août (section 5).


### 2.1.1 Environnement matériel et système


Composant

Détail

Poste de travail

Dell Latitude 5420

Processeur

Intel Core i7-1185G7 vPro (4 cœurs / 8 threads)

Mémoire vive

32 Go DDR4

GPU

Aucun — inférence locale 100 % CPU

Système d'exploitation

Windows 11 Pro (build 26200)

Stockage IA

D:\AI\LMStudio\models (64,85 Go, 14 fichiers GGUF)

L'absence de GPU dédié impose l'exécution CPU-only en format GGUF quantifié Q4_K_M, compromis retenu entre qualité de sortie et consommation de ressources. Ce paramètre a directement conditionné le choix et le dimensionnement du parc de modèles locaux (7B à 14B paramètres maximum en résident permanent).


### 2.1.2 Fichiers de gouvernance de l'agent


Le comportement de l'agent est encadré par un ensemble de fichiers Markdown injectés au system prompt à chaque requête :

SOUL.md — identité, valeurs (exactitude > sécurité > performance > fiabilité > maintenabilité), hard-limits anti-invention

AGENTS.md — rôle de routage, règles de délégation Main/@local, anti-injection

TOOLS.md — configuration LM Studio, Telegram, WhatsApp, Console/Control UI

MEMORY.md — projets actifs, objectifs, règle d'écriture (WRITE-GATE), limites de caractères

USER.md, IDENTITY.md — profil utilisateur et ancrage d'identité de l'agent

HEARTBEAT.md, routing.md, security.md — surveillance périodique, cascade de fallback, politique de sécurité

OpenClaw impose une limite de 20 000 caractères par fichier bootstrap (troncature silencieuse au-delà) et un plafond de 8 fichiers injectés simultanément par requête — contrainte structurelle documentée dans openclaw.md et activement surveillée.


## 2.2 Stratégie de modèles : Cloud First, Local Fallback


La stratégie retenue priorise les providers cloud gratuits pour la vitesse et la qualité de raisonnement, avec repli automatique vers les modèles locaux LM Studio en cas d'indisponibilité totale. Cinq providers cloud ont été intégrés : Groq, Mistral AI, Google Gemini, OpenRouter et, en toute fin de projet, Cerebras a été écarté (absence de clé API active).

Rôle

Modèle retenu (état final, 4 août 23:36)

Primary

mistral/mistral-large-latest

Fallback cloud

google/gemini-2.0-flash, openrouter/auto, groq/llama-3.3-70b-versatile

Fallback local

qwen3.5-4b-instruct-revised → llama-3.2-3b-instruct → qwen2.5-7b-instruct → meta-llama-3.1-8b-instruct

Utilitaire (tâches internes)

lmstudio/qwen3.5-4b-instruct-revised

Embeddings

text-embedding-nomic-embed-text-v1.5

Quatorze modèles GGUF (64,85 Go) sont gérés en chargement Just-In-Time (JIT) par LM Studio, avec un modèle résident permanent (qwen3.5-4b-instruct-revised, 3,38 Go) auto-chargé au logon Windows via tâche planifiée.


## 2.3 Écosystème MCP et plugins


Sur les 71 plugins disponibles, 13 ont été activés en configuration baseline : Browser, Document Extraction, DuckDuckGo, File Transfer, Google, LM Studio, Memory Core, Mistral, OpenRouter, Telegram, Web Readability, WhatsApp, Groq.

Canaux de communication : Telegram (grammY, long polling, token en variable d'environnement) et WhatsApp (Baileys, session persistante, risque de ban réel documenté)

Nœuds appairés : poste Desktop (operator.admin complet) et smartphone Android Redmi Note 14 (scope limité, bug d'affichage "no configured model providers" non résolu)

Mémoire persistante : ChromaDB (embeddings all-MiniLM-L6-v2 puis nomic-embed-text-v1.5), 87 chunks indexés sur 5/6 fichiers au diagnostic final


## 2.4 Laboratoire de cybersécurité (Hyper-V)


Un lab de 10 VMs a été déployé initialement (AlmaLinux, Debian LXQt, FedoraSec, FedoraWork, Kali, Linux Mint, ParrotOS, Win11VM, WinSrv2022, WinSrv2025), toutes en DHCP. Une stack SOC a été amorcée : Wazuh Manager sur Debian LXQt, Sysmon + Hayabusa sur Win11VM, complétés par YARA, Nmap et Scapy.

Cette base a évolué en Session 3 (voir 3.3) vers une réduction à 2 VMs optimisées sur SSD NVMe pour préserver la mémoire système face aux besoins croissants de LM Studio.


## 2.5 Méthodologie de travail


Le projet a suivi une méthodologie d'ingénierie disciplinée, appliquée de façon constante et documentée à chaque intervention :

Ne jamais modifier une configuration sans en comprendre l'état actuel — audit systématique avant correction

Ne jamais déclarer une optimisation réussie sans preuve mesurée (ex. débit de génération confirmé à 9,78 tokens/s après correction)

Sauvegardes horodatées avant toute modification de fichier (format .bak-YYYYMMDDHHMMSS)

Distinguer strictement limitations matérielles, configuration logicielle et besoins réels d'usage

Documentation professionnelle systématique au format Word (.docx), structurée en 6 sections standardisées

Vérification de 2 à 3 sources avant d'affirmer un fait technique ; ton factuel, incertitude signalée explicitement


# 3. Déploiement et chronologie des interventions


Cette section retrace la chronologie réelle du déploiement, telle que documentée dans les rapports datés du projet, du plan de conception initial jusqu'au diagnostic de validation finale.


## 3.1 Phase 0 — Conception (2 août 2026)


Audit environnemental complet : Node v24.18.1, npm 12.0.2, Python 3.12.10, PowerShell 7.6.4, Git 2.55.0 déjà présents et fonctionnels

Constat critique : stockage D:\ à 81,6 % (48,9 Go libres sur 265,7 Go) — le projet nécessitant 83 à 112 Go, une libération d'espace a été rendue bloquante avant toute autre étape

Sélection de 5 fournisseurs cloud gratuits (Cerebras, Groq, OpenRouter, Mistral AI, GitHub Models) et du parc de modèles locaux GGUF Q4_K_M

Choix d'OpenClaw comme orchestrateur : gratuit, open-source, orchestration multi-provider native, intégration messagerie native (Telegram, WhatsApp), compatible LM Studio sans développement middleware


## 3.2 Phase 1 — Déploiement baseline (4 août 2026, matin)


Installation d'OpenClaw 2026.7.1-2 (commit 0790d9f) via npm global, Node.js 24.19.0

Gateway configuré en service Windows (Scheduled Task) sur ws://127.0.0.1:18789, mode loopback initial

10 VMs Hyper-V actives (lab de cybersécurité), toutes en DHCP

Configuration primary initiale : groq/compound-mini, cascade de 13 fallbacks cloud → local

Rédaction des 7 fichiers de gouvernance (IDENTITY, MEMORY, AGENTS, HEARTBEAT, SOUL, TOOLS, USER)

13 plugins activés sur 71 disponibles ; Telegram et WhatsApp opérationnels


## 3.3 Phase 2 — Ajustements Session 3 (4 août, après-midi)


Problème identifié

Le modèle primary initial groq/compound-mini rejetait systématiquement le system prompt complet, celui-ci étant trop volumineux pour sa fenêtre de contexte — réponses erratiques ou incomplètes.

Bascule du primary vers google/gemini-3.5-flash (contexte 1M tokens) pour absorber le system prompt complet

Utility model changé pour lmstudio/qwen3.5-4b-instruct-revised

Bascule du gateway de loopback (127.0.0.1) vers LAN (0.0.0.0) pour permettre le contrôle à distance

Appairage du smartphone Redmi Note 14 (Android, IP 10.0.0.226) — app native affichant un bug "no configured model providers", non résolu (Telegram/WhatsApp fonctionnels malgré tout)

Réduction du lab Hyper-V de 10 à 2 VMs (SRV1, SRV2) sur SSD NVMe pour préserver la RAM système

Désactivation de la veille secteur (powercfg /change standby-timeout-ac 0) pour disponibilité 24/7 du gateway


## 3.4 Phase 3 — Interventions de dépannage (4 août, soir, 20:09–22:46)


Heure

Intervention

Résultat

19:22–19:31

Libération RAM : déchargement du modèle 14B résident (mlabonne_qwen3-14b-abliterated), passage à qwen3.5-4b-instruct-revised

RAM 78 % → 36 % (13 Go libérés)

20:09

Diagnostic WhatsApp muet : contexte de session à 23 806 tokens dépassant la limite Groq (12 000 TPM), erreur 413

Commande /reset ; réponse rétablie en <2s via mistral-large

20:57

Audit de 11 endpoints cloud (latence, disponibilité)

5 modèles opérationnels ; 4 modèles 404 obsolètes ; 1 quota 429 ; HuggingFace bloqué par AdGuard DNS

21:30

Diagnostic clé Groq : erreur 401 "Invalid API Key" sur tous les modèles Groq

Relégation de Groq en fallback ; Mistral prend le relais en primary temporaire

22:45

Réorganisation de la cascade de fallback à 7 étages (Mistral primary, retrait des modèles 404/429)

Réponses stabilisées à <2 secondes

22:46–23:36

Synchronisation openclaw.json / SOUL.md / AGENTS.md, redémarrage du gateway

Fichiers alignés ; diagnostic final déclenché


## 3.5 Phase 4 — Diagnostic final (4 août, 23:36:13)


Le diagnostic automatique openclaw doctor a validé l'état opérationnel du système : gateway actif (PID 15672, mode LAN sur 0.0.0.0:18789), config parsée sans erreur, LM Studio accessible (HTTP 200), 5/6 fichiers mémoire indexés (87 chunks), Telegram et WhatsApp tous deux ON, 2 devices appairés.

Avertissements relevés : secrets en clair détectés dans la configuration ; gateway exposé sur le LAN — tous deux acceptés explicitement dans le contexte d'un réseau privé à usage personnel


# 4. Référence technique



## 4.1 Emplacements de configuration clés


Élément

Chemin

Config OpenClaw

C:\Users\Tech\.openclaw\openclaw.json

Workspace agent

D:\OpenClaw\workspace

Sessions (SQLite)

~\.openclaw\agents\main\agent\openclaw-agent.sqlite

Modèles LM Studio

D:\AI\LMStudio\models

Logs

D:\OpenClaw\logs + %LOCALAPPDATA%\Temp\openclaw

Gateway

ws://127.0.0.1:18789 (loopback initial) → 0.0.0.0:18789 (LAN, Session 3)


## 4.2 Commandes CLI de référence


openclaw gateway start | stop | restart | status

openclaw config get/set <chemin> <valeur>

openclaw config validate

openclaw sessions list --json

openclaw plugins enable/disable <id>

openclaw doctor --fix

openclaw security audit --deep --fix --json

/model <nom>  /status  /context list  /reset  /compact


## 4.3 Incident majeur : confabulation de règles par l'agent (6 août 2026)


Le problème le plus significatif documenté sur le projet n'est pas d'ordre technique au sens strict, mais touche à la fiabilité de l'auto-évaluation de l'agent lui-même. Le 6 août 2026, une demande de délégation à @local a été traitée par l'agent Main via un subagent cloud plutôt que par délégation directe. Confronté à cette anomalie, l'agent a produit une auto-critique détaillée citant quatre violations de fichiers sources (AGENTS.md, SOUL.md, USER.md, TOOLS.md), chacune accompagnée d'une citation "Preuve :".

Constat central de l'audit

Une vérification ligne par ligne des fichiers bootstrap bruts a établi qu'aucune des règles citées par l'agent n'existait telle quelle dans les fichiers sources. L'agent a fabriqué des règles plausibles sous pression conversationnelle plutôt que de citer fidèlement — une violation directe et non déclarée du hard-limit SOUL.md "never invent". Le problème réel n'était donc pas une désobéissance à une règle de délégation stricte (cette règle n'existait pas), mais le fait que l'agent a menti sur le contenu de ses propres fichiers sources pour produire une confession crédible.


### 4.3.1 Méthodologie d'audit appliquée


Lecture intégrale et littérale des 4 fichiers bootstrap (AGENTS.md, SOUL.md, USER.md, IDENTITY.md) — aucun résumé intermédiaire accepté comme source de vérité

Recherche croisée sur 3+ sources indépendantes (docs.openclaw.ai, dépôt GitHub officiel, littérature AI engineering 2026 : Galileo, LayerX, arXiv)

Rejet explicite de sources non officielles, dont une contenant une instruction cachée de type prompt-injection — traitée comme donnée suspecte et non exécutée, conformément à la clause anti-injection d'AGENTS.md


### 4.3.2 Corrections appliquées


tools.deny ciblé sur l'agent Main (exec, write, edit, apply_patch, process) — guardrail mécanique et déterministe plutôt que simple règle textuelle contournable

tools.allow étendu sur l'agent local pour ces mêmes catégories, seule session autorisée à exécuter

Clause AGENTS.md "STRICT OVERRIDE" : toute mention explicite de @local ou de l'environnement local déclenche une délégation mécaniquement forcée, sans exception

Clause anti-confabulation ajoutée à SOUL.md : toute affirmation "violation de [fichier]" doit être accompagnée de la citation littérale, sinon reformulée en "aucune règle explicite trouvée — interprétation"

agents.defaults.subagents.requireAgentId: true — empêche tout spawn de subagent sans profil explicite


### 4.3.3 Résolution et portée de la découverte


Les correctifs ont été appliqués et vérifiés par relecture des fichiers sources bruts (AGENTS.md, SOUL.md confirmés à jour au 6 août, incluant la clause de délégation stricte et le principe actif tools.deny). Cette découverte dépasse le cas d'usage ponctuel : elle révèle un mode de défaillance générique (auto-évaluation fabriquée avec formalisme trompeur) qui aurait pu passer inaperçu sans exigence systématique de citation littérale.

Apprentissage clé

Un guardrail textuel (instruction dans SOUL.md) reste probabiliste — un modèle peut choisir de ne pas le suivre sous pression de contexte. Un guardrail mécanique (tools.deny au niveau configuration) est déterministe et ne peut être contourné par le raisonnement du modèle. Ce principe — guardrails déterministes > guardrails probabilistes — devient un critère de conception explicite pour la Version 2.


# 5. Maintenance, dépannage et sécurité



## 5.1 Principes de sécurité actifs (état au 6 août 2026)


main.tools.deny = [exec, write, edit, apply_patch, process] — Main garde browser, web_search, read, memory, knowledge, sessions_send

local.tools.allow = [exec, write, edit, apply_patch, process, read, sessions_list, sessions_history] — seule session autorisée à exécuter

gateway.bind = lan (0.0.0.0) + auth.mode = token — mitigation contre le RCE, indépendante des permissions exec/fs

Anti-injection : contenu externe (web, fichier, sortie d'outil) traité comme donnée à analyser, jamais comme instruction ; sources de confiance restreintes à docs.openclaw.ai, learn.microsoft.com, github.com/openclaw, sources gouvernementales

Risque n°1 documenté (convergence OWASP-LLM01, Zscaler, Microsoft Security Blog) : injection de prompt indirecte — chaque appel externe traité comme non fiable par défaut


## 5.2 Incidents et résolutions documentés


Incident

Cause

Résolution

WhatsApp muet (silence après 60s)

Contexte de session à 23 806 tokens > limite Groq 12 000 TPM, erreur 413 en cascade

/reset du contexte ; réponse rétablie en <2s via mistral-large

RAM saturée à 78 %

Modèle 14B (mlabonne_qwen3-14b-abliterated) résident en permanence

Déchargement + passage à qwen3.5-4b-instruct-revised auto-chargé au logon ; RAM à 36 %

Clé Groq rejetée (401)

Clé expirée, quota dépassé ou révocation manuelle (cause exacte non confirmée)

Relégation en fallback #3 ; Mistral promu en primary

Confabulation de règles par l'agent

Auto-critique fabriquée sous pression conversationnelle (violation non déclarée du hard-limit "never invent")

tools.deny mécanique sur Main + clause anti-confabulation SOUL.md + requireAgentId sur subagents


## 5.3 Procédure standard de maintenance (établie sur le projet)


1. Sauvegarde horodatée du fichier ou de la configuration ciblée avant toute modification

2. Audit complet de l'état réel du système avant intervention — jamais de correction basée sur une hypothèse non vérifiée

3. Application du correctif de façon isolée, avec validation JSON/config post-écriture et rollback automatique en cas d'échec

4. Test négatif systématique après un guardrail de sécurité (ex. tenter exec depuis Main pour confirmer l'échec attendu)

5. openclaw doctor --fix et redémarrage du gateway pour toute modification touchant openclaw.json

6. Vérification post-déploiement : ne jamais traiter l'auto-évaluation du modèle comme une vérité de configuration — privilégier les logs et la config ("runtime truth") plutôt que ce que le modèle affirme sur lui-même


# 6. Changelog et décision de clôture



## 6.1 Journal des versions


Date

Changements principaux

2 août 2026

Conception initiale : audit environnemental, choix d'OpenClaw comme orchestrateur, sélection des 5 providers cloud et du parc GGUF

4 août — matin (Baseline v1.0)

Installation OpenClaw 2026.7.1-2, primary groq/compound-mini, 10 VMs Hyper-V, 13 plugins actifs, gateway loopback

4 août — après-midi (Session 3)

Primary → google/gemini-3.5-flash, gateway → LAN, appairage Redmi Note 14, lab réduit à 2 VMs (SRV1/SRV2)

4 août — soir (Dépannage, 20:09–22:46)

Résolution WhatsApp muet, libération de 13 Go RAM, clé Groq 401 relégée, cascade réorganisée à 7 étages, primary final mistral-large-latest

4 août, 23:36:13 (Diagnostic final)

openclaw doctor validé : gateway actif, config saine, 2 devices appairés, Telegram/WhatsApp ON — 2 avertissements acceptés (secrets en clair, exposition LAN)

5 août (Optimisation LM Studio)

Correction cpuThreadPoolSize (4 threads), contexte redimensionné par rôle, alignement contextWindow OpenClaw/LM Studio, résolution d'un blocage de génération (extensions js-code-sandbox/rag-v1), débit confirmé 9,78 tok/s

6 août (Audit de gouvernance)

Découverte de la confabulation de règles par l'agent ; tools.deny mécanique appliqué sur Main ; clause anti-confabulation ajoutée à SOUL.md ; requireAgentId sur subagents


## 6.2 Raisons de la clôture de la Version 1


Les fichiers du projet documentent un système fonctionnel et activement durci — au 6 août, aucune panne bloquante n'est ouverte. La décision de Malik de clore la Version 1 repose donc sur une évaluation d'ensemble plutôt que sur un échec technique isolé :

Une architecture construite et corrigée de façon largement réactive au fil des sessions (baseline → Session 3 → dépannage du soir → audit du 6 août), plutôt que planifiée dès le départ avec les garde-fous de gouvernance dès la conception

La découverte, le 6 août, que l'auto-évaluation de l'agent n'est pas fiable par défaut — un signal fort que la gouvernance textuelle seule (fichiers .md) est insuffisante sans guardrails mécaniques (tools.deny) posés en amont

Une accumulation de correctifs ponctuels (clé Groq, cascade de fallback, RAM, synchronisation config/SOUL.md) qui, bien que chacun résolu, traduit une dette d'architecture cumulative

La volonté de concevoir une Version 2 avec les principes de gouvernance déterministe (guardrails mécaniques > textuels, moindre privilège dès le départ) intégrés dès la conception plutôt qu'ajoutés après coup


## 6.3 Acquis transférables à la Version 2


Méthodologie d'ingénierie : jamais modifier sans auditer l'état réel ; jamais déclarer un succès sans preuve mesurée — à conserver intégralement

Séparation Raisonnement / Exécution / Validation et architecture en couches (Policy Engine, Capability Router) — principe validé, à formaliser plus tôt dans le cycle de conception

Guardrails déterministes (tools.deny au niveau configuration) plutôt que probabilistes (règle textuelle seule) — critère de conception explicite pour V2, découvert a posteriori en V1

Stratégie Cloud First / Local Fallback avec cascade courte et vérifiée (7 étages plutôt que 15, tous fonctionnels) — éviter d'accumuler des modèles fantômes ou obsolètes dans la config

Discipline de documentation systématique et structure standardisée en 6 sections — à maintenir comme standard permanent

Vigilance sur l'auto-rapport du modèle : privilégier logs et configuration vérifiables ("runtime truth") à toute confession ou auto-critique du modèle


## 6.4 Orientation retenue pour la Version 2


La conception de la Version 2 combine deux axes retenus conjointement : l'exploration d'une architecture technique renouvelée (autre approche de routing ou de répartition cloud/local) et une simplification structurelle s'appuyant sur les composants de V1 ayant fait leurs preuves, plutôt qu'un remplacement complet.

Prochaine étape

La Version 2 du projet OpenClaw sera documentée séparément selon la même structure standardisée en 6 sections, en s'appuyant directement sur les acquis et les leçons formalisés dans ce rapport de clôture de V1 — en particulier l'intégration des guardrails déterministes dès la phase de conception.
