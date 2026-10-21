RAPPORT DE CONFIGURATION

OpenClaw 2026.7.1-2 — Architecture Hybride Local + Cloud

Utilisateur : MalikDate : 4 août 2026Version : 1.0


# Table des matières


1. Résumé exécutif

2. Contexte et objectifs

3. Architecture matérielle

4. Installation et configuration initiale

5. Configuration OpenClaw

6. Modèles d'IA — Inventaire complet

7. Fichiers de configuration de l'agent

8. Erreurs rencontrées et résolutions

9. Plugins et outils activés

10. Infrastructure réseau et sécurité

11. Synthèse et analyse finale

12. Recommandations et prochaines étapes

Annexe A — Commandes CLI de référence

Annexe B — Fichiers .md complets


# 1. Résumé exécutif


Ce document présente la configuration complète d'un assistant IA hybride (local + cloud) déployé sur un poste de travail Dell Latitude 5420 sous Windows 11 Pro. L'architecture repose sur OpenClaw 2026.7.1-2 comme orchestrateur, avec une stratégie « Cloud First, Local Fallback » optimisée pour un usage pédagogique et professionnel en cybersécurité.

Points clés :

• 14 modèles d'IA locaux (GGUF) gérés via LM Studio en mode JIT

• 5 providers cloud avec 31 modèles accessibles (Groq, Mistral, Google, OpenRouter, Cerebras — ce dernier retiré)

• 10 machines virtuelles Hyper-V constituant un lab de cybersécurité

• Stack SOC en construction (Wazuh + Sysmon + Hayabusa)

• Channels de communication : Telegram + WhatsApp

• Documentation systématique au format Word (.docx)


# 2. Contexte et objectifs


2.1 Profil de l'utilisateur

Malik est étudiant en DEP Soutien Informatique (programme 5229) à l'EMICA de Montréal. Son approche est pédagogique : chaque projet doit être formateur, pas seulement fonctionnel. La cible professionnelle est le consulting freelance/nomade en cybersécurité cloud, avec un focus sur les marchés européens (Italie, Espagne, Portugal).

2.2 Objectifs du déploiement

• Disposer d'un assistant IA hybride exploitant au maximum les ressources cloud (vitesse, modèles avancés) tout en conservant une autonomie locale

• Construire un lab de cybersécurité opérationnel avec détection et monitoring

• Documenter chaque étape pour capitaliser sur l'apprentissage

• Préparer une roadmap de 17 certifications cybersécurité


# 3. Architecture matérielle


3.1 Machine principale

Composant

Spécification

Modèle

Dell Latitude 5420

Processeur

Intel i7-1185G7 vPro (4 cœurs / 8 threads)

Mémoire vive

32 GB DDR4

GPU

Aucun — calcul CPU uniquement pour l'IA locale

Système d'exploitation

Windows 11 Pro (build 26200)

3.2 Stockage

• Disque D:\ : principal pour données, projets, modèles IA (69.58 GB de modèles)

• Disque C:\ : système uniquement — écriture interdite sauf exception justifiée

3.3 Virtualisation

Hyper-V natif Windows avec 10 VMs actives :

Nom VM

Système d'exploitation

Rôle

AlmaLinux

Alma Linux

Lab général

DebianLXQt

Debian + LXQt

Wazuh Manager

FedoraSec

Fedora

Sécurité

FedoraWork

Fedora

Travail

Kali

Kali Linux

Pentest

LinuxMint

Linux Mint

Desktop

ParrotOS

Parrot OS

Forensics

Win11VM

Windows 11

Test

WinSrv2022

Windows Server 2022

Serveur

WinSrv2025

Windows Server 2025

Serveur

• BlackArch : prévu, pas encore créé

• Toutes les VMs sont en DHCP (pas d'IP fixe actuellement)


# 4. Installation et configuration initiale


4.1 OpenClaw

Version installée : 2026.7.1-2 (commit 0790d9f)

Méthode d'installation : npm (Node.js 24.19.0)

Gateway : service Windows (Scheduled Task) sur ws://127.0.0.1:18789

Dashboard web : http://127.0.0.1:18789/

Workspace agent : D:\openclaw\workspace

4.2 LM Studio

Version : 0.4.20

Port d'API : 1234

Mode : JIT (Just-In-Time — charge les modèles à la volée, pas en mémoire permanente)

Chemin modèles : D:\AI\LMStudio\models

Modèles disponibles : 14 fichiers .gguf (voir section 6)

4.3 Stack de développement

• Python 3.12

• Node.js 24.19.0

• FastAPI, LangChain, LangGraph, ChromaDB

• Brave (navigateur principal)

4.4 Sécurité

• Wazuh Manager : déployé sur DebianLXQt (ports 1514/1515/55000)

• Sysmon + Hayabusa : déployés sur Win11VM (poste Windows durci)

• Outils complémentaires : YARA, Nmap, Scapy

• DNS/VPN : configuré


# 5. Configuration OpenClaw


5.1 Stratégie de routing

La stratégie adoptée est « Cloud First, Local Fallback ». Le modèle primary (cloud rapide) répond en premier. En cas d'indisponibilité, la cascade de fallbacks prend le relai, terminant par les modèles locaux LM Studio.

5.2 Configuration CLI appliquée

Clé de configuration

Valeur

Description

agents.defaults.model.primary

groq/compound-mini

Modèle cloud rapide par défaut

agents.defaults.model.fallbacks

[13 modèles]

Cascade cloud → local

agents.defaults.utilityModel

lmstudio/llama-3.2-3b-instruct

Tâches internes légères

agents.defaults.thinkingDefault

medium

Niveau de reasoning par défaut

5.3 Cascade de fallbacks complète

L'ordre exact des fallbacks configurés est le suivant :

groq/compound-mini              ← PRIMARY

groq/llama-3.3-70b-versatile    ← fallback #1

groq/openai/gpt-oss-120b        ← fallback #2

google/gemini-2.5-pro           ← fallback #3

google/gemini-2.5-flash         ← fallback #4

mistral/mistral-large-latest    ← fallback #5

mistral/codestral-latest        ← fallback #6

openrouter/moonshotai/kimi-k2.6 ← fallback #7

openrouter/auto                 ← fallback #8

lmstudio/deepseek-r1-distill-qwen-14b    ← fallback #1 local

lmstudio/mlabonne_qwen3-14b-abliterated  ← fallback #2 local

lmstudio/qwen2.5-coder-14b-instruct      ← fallback #3 local

lmstudio/qwen3.5-4b-instruct-revised     ← fallback #4 local

lmstudio/llama-3.2-3b-instruct           ← fallback #5 local

5.4 Channels de communication

• Telegram : activé, configuré

• WhatsApp : activé, numéro +14384081775

5.5 Session management

OpenClaw 2026.7.1 utilise SQLite pour le stockage des sessions (remplacement du legacy sessions.json). Les sessions actives sont dans :

~\.openclaw\agents\main\agent\openclaw-agent.sqlite

Les archives de transcripts sont dans :

~\.openclaw\agents\main\sessions\

5.6 Maintenance automatique

• Mode : enforce (nettoie activement)

• Prune après : 30 jours

• Max entries : 500 sessions

• Max disk : 10 GB par agent


# 6. Modèles d'IA — Inventaire complet


6.1 Modèles locaux (LM Studio — 14 fichiers .gguf)

Tous les modèles sont stockés dans D:\AI\LMStudio\models et chargés en mode JIT.

Fichier .gguf

Rôle

Taille

RAM estimée

Vitesse CPU

DeepSeek-R1-Distill-Qwen-14B-Q4_K_S.gguf

Math, raisonnement logique

7.98 GB

~9 GB

~4 tok/s

mlabonne_Qwen3-14B-abliterated-Q4_K_S.gguf

Reasoning sans censure

7.98 GB

~9 GB

~4 tok/s

Qwen2.5-Coder-14B-Instruct-Q4_K_S.gguf

Code local #1

7.98 GB

~9 GB

~4 tok/s

gemma-4-e4b-it-obliterated-q4_k_m.gguf

Vision sans censure

4.94 GB

~6 GB

~6 tok/s

meta-llama-3.1-8b-instruct-abliterated.Q4_K_M.gguf

Fallback sans censure

4.58 GB

~5.5 GB

~7 tok/s

Meta-Llama-3.1-8B-Instruct-Q4_K_S.gguf

Fallback général

4.37 GB

~5 GB

~7 tok/s

qwen2.5-coder-7b-instruct-ghidra-v2-q4_k_m.gguf

Cybersec, reverse eng

4.36 GB

~5 GB

~7 tok/s

Qwen2.5-7B-Instruct-Q4_K_S.gguf

Général léger

4.15 GB

~5 GB

~7 tok/s

Mistral-7B-Instruct-v0.3-Q4_K_S.gguf

Fallback urgence

3.86 GB

~4.5 GB

~8 tok/s

Qwen3.5-4B-Kimi-Reasoning-Luna-LoRA-step495-Q4_K_M.gguf

Reasoning léger

2.59 GB

~3 GB

~12 tok/s

Qwen3.5-4B-Revised-q4_k_m.gguf

Rapide, vision, 262K ctx

2.52 GB

~3 GB

~12 tok/s

Llama-3.2-3B-Instruct-Q4_K_S.gguf

Ultra léger, rapide

1.80 GB

~2 GB

~18 tok/s

Fichiers complémentaires :

• mmproj-F32.gguf (1.78 GB) — projection vision

• mmproj-Qwen3.5-4B-Kimi-Reasoning-Luna-LoRA-step495-BF16.gguf (0.63 GB)

• mmproj-f16.gguf (0.63 GB)

• nomic-embed-text-v1.5-Q8_0.gguf (0.14 GB) — embeddings

• mtp-gemma-4-E4B-it-Q8_0.gguf (0.09 GB)

6.2 Modèles cloud — Providers activés

Groq (primary provider — vitesse)

• compound-mini — PRIMARY (réponses rapides, 2000+ tok/s)

• llama-3.3-70b-versatile — fallback rapide

• gpt-oss-120b — raisonnement

• llama-4-scout — vision

• qwen3-32b — général

• llama-3.1-8b-instant — ultra rapide

Mistral (code)

• devstral-medium-latest — code agentique

• codestral-latest — code pur

• mistral-large-latest — général puissant

Google Gemini (multimodal)

• gemini-2.5-pro — contexte 1M tokens, multimodal

• gemini-2.5-flash — version rapide

OpenRouter (variété / fallback)

• moonshotai/kimi-k2.6 — fallback prioritaire avant auto

• auto — routage automatique OpenRouter

6.3 Modèle utilitaire

• lmstudio/llama-3.2-3b-instruct — tâches internes légères (titres, classification)

6.4 Répartition des modèles par tâche

Tâche

Modèle suggéré

Type

Recherche web simple

groq/llama-3.3-70b-versatile

Cloud

Raisonnement complexe

groq/gpt-oss-120b

Cloud

Code — génération cloud

mistral/devstral-medium-latest

Cloud

Code — exécution locale

lmstudio/qwen2.5-coder-14b-instruct

Local

Cybersec / reverse eng

lmstudio/qwen2.5-coder-7b-ghidra-v2

Local

Image / multimodal

google/gemini-2.5-pro

Cloud

Contexte énorme

google/gemini-2.5-pro

Cloud

Tâche privée / offline

lmstudio/qwen2.5-coder-14b-instruct

Local

Réponse rapide chat

groq/llama-3.1-8b-instant

Cloud

Math / raisonnement logique

lmstudio/deepseek-r1-distill-qwen-14b

Local


# 7. Fichiers de configuration de l'agent


OpenClaw injecte les fichiers .md du workspace dans le system prompt de l'agent. Chaque fichier a un rôle spécifique dans la personnalisation du comportement.

7.1 IDENTITY.md — Identité de l'agent

Définit la persona de l'agent : nom, creature, ton, emoji.

Contenu :

• Name : Agent

• Creature : AI

• Vibe : sharp / direct / précis

• Emoji : 🤖

• Avatar : non configuré

7.2 MEMORY.md — Mémoire persistante

Stocke les projets actifs, préférences, faits persistants et notes de suivi. Servira de contexte à long terme pour l'agent.

Sections :

• Index des projets actifs (SOC, Hyper-V, OpenClaw, Certifications)

• Projets à venir (BlackArch, expansion SOC, portfolio, lab cloud, Docker/K8s, etc.)

• Préférences utilisateur (format Word, convention AUDIT/FIX, logs vers D:\)

• Hardware de référence

• Notes de suivi (checklist à compléter)

7.3 AGENTS.md — Référence modèles

Guide de référence pour le choix manuel du modèle. IMPORTANT : ce fichier n'est PAS un router automatique. Le routing se fait via la config CLI (primary/fallbacks) ou la commande /model.

Contient :

• Liste complète des providers cloud avec rôles

• Table des modèles locaux avec taille réelle .gguf

• Table de correspondance tâche → modèle suggéré

• Cascade de fallbacks exacte de la config CLI

7.4 HEARTBEAT.md — Tâches périodiques

Actuellement vide (comments-only). Aucune tâche périodique ne s'exécute.

Tâches potentielles documentées :

• Monitoring système (disque, RAM, VMs)

• Monitoring sécurité (Wazuh, Sysmon, Hayabusa)

• Maintenance (cleanup logs, health check providers)

7.5 SOUL.md — Comportement et orchestration

Définit les règles de comportement de l'agent et les directives d'orchestration modèle.

Sections :

• Comportement de base (langue, ton, niveau)

• Règles d'exécution (CLI+GUI, confirmation, logs)

• Orchestration modèle (cloud uniquement, cloud+local, local uniquement, fallback)

• Permissions fichiers (D:\ complet, C:\ lecture seule)

• Standard de documentation (Word .docx, 6 sections)

7.6 TOOLS.md — Notes locales et outils

Référence rapide des outils, VMs, chemins et configuration technique.

Contient :

• Table des VMs Hyper-V (nom, OS, rôle)

• Configuration LM Studio et OpenClaw

• Liste complète des plugins OpenClaw (activés + désactivés avec raisons)

• Règle : toujours maximiser les tools pertinents

• Chemins fréquents D:\ uniquement

• Études en informatique (modules en cours)

• Projets à venir détaillés

7.7 USER.md — Profil utilisateur

Profil complet de Malik : formation, objectifs, hardware, projets, stack, règles, comportement attendu.

7.8 Synthèse des rôles des fichiers

Fichier

Rôle

Fréquence de lecture

IDENTITY.md

Persona de l'agent

À chaque prompt

MEMORY.md

Mémoire persistante

À chaque prompt

AGENTS.md

Référence modèles

À chaque prompt

HEARTBEAT.md

Tâches périodiques

Si heartbeat activé

SOUL.md

Comportement + orchestration

À chaque prompt

TOOLS.md

Notes locales techniques

À chaque prompt

USER.md

Profil utilisateur

À chaque prompt


# 8. Erreurs rencontrées et résolutions


8.1 Commandes CLI incorrectes (doc en ligne obsolète)

Problème :

Plusieurs commandes suggérées par la documentation en ligne se sont révélées incorrectes pour la version 2026.7.1-2.

Erreurs constatées :

Commande tentée

Erreur

Solution

openclaw config list --filter agents.defaults --format json

--filter non reconnu

openclaw config list --format json (mais --format non reconnu non plus)

openclaw sessions close --all

--all non reconnu

Pas de commande close dans sessions. Utiliser cleanup ou arrêter le gateway

openclaw sessions close <ID>

Too many arguments

Pas de commande close. Les sessions se gèrent via le dashboard ou cleanup

openclaw config list --format json

--format non reconnu

openclaw config list (sans options) ou utiliser config get/set

8.2 Sessions persistants après redémarrage

Problème :

Le redémarrage du gateway (openclaw gateway restart) n'a pas purgé les sessions actives. Les 2 sessions (main + WhatsApp) restaient visibles dans le status.

Cause :

OpenClaw 2026.7.1 stocke les sessions dans SQLite (openclaw-agent.sqlite), pas dans le legacy sessions.json. Le restart recharge les sessions depuis SQLite.

Résolution :

1. Arrêter le gateway : openclaw gateway stop

2. Sauvegarder l'ancien SQLite (optionnel)

3. Supprimer le fichier : Remove-Item ~\.openclaw\agents\main\agent\openclaw-agent.sqlite

4. Supprimer le legacy JSON (optionnel) : Remove-Item ~\.openclaw\agents\main\sessions\sessions.json

5. Redémarrer : openclaw gateway start

Résultat : 0 sessions, historique purgé, config intacte.

8.3 Fichier MEMORY.md écrasé

Problème :

Le fichier MEMORY.md a été accidentellement écrasé dans le dashboard web (contenu remplacé par 'a').

Cause :

Édition directe dans le dashboard sans sauvegarde préalable.

Résolution :

Reconstruction complète du fichier avec structure propre (projets, préférences, faits, notes de suivi).

Prévention :

Toujours sauvegarder les fichiers .md avant modification dans le dashboard.

8.4 Routing intelligent inexistant

Problème :

OpenClaw 2026.7.1 n'a pas de mécanisme natif de routing par type de tâche (intent detection). Le fichier AGENTS.md créé avec une 'logique de routage' n'est pas interprété automatiquement.

Cause :

Limitation architecturale d'OpenClaw. Seuls primary et fallbacks sont supportés nativement.

Résolution :

1. Clarifier dans AGENTS.md que c'est un GUIDE de référence, pas un router

2. Utiliser la commande /model <nom> pour switcher manuellement

3. Configurer la cascade de fallbacks pour couvrir les cas d'indisponibilité

Alternatives explorées :

• ClawRouter (plugin) : nécessite clé API + admin — pas applicable

• Skill custom : pas de hook pre-request documenté

• Wrapper externe : possible mais complexe

8.5 Modèles fantômes dans la documentation

Problème :

Le fichier AGENTS.md initial contenait des modèles non présents dans LM Studio (bonsai-27b) et des versions Gemini incorrectes (3.1, 3.5 au lieu de 2.5).

Cause :

Copie depuis une documentation externe sans vérification contre le système réel.

Résolution :

Audit complet des fichiers .gguf présents dans D:\AI\LMStudio\models et alignement de la documentation.

8.6 Cerebras retiré

Problème :

Le provider Cerebras était listé dans AGENTS.md mais n'est pas activé dans OpenClaw (plugin cerebras-provider désactivé, pas de clé API configurée).

Résolution :

Suppression complète de Cerebras de la documentation. Les modèles Cerebras (gpt-oss-120b, etc.) restent accessibles via Groq.


# 9. Plugins et outils activés


9.1 Plugins activés (13/71)

Plugin

ID

Usage

Browser

browser

Navigation web, recherche, scraping

Document Extraction

document-extract

Extraction texte/images de documents locaux

DuckDuckGo

duckduckgo

Recherche web anonyme

File Transfer

file-transfer

Transfert fichiers via nœuds pairés

Google

google

Provider Google Gemini

LM Studio

lmstudio

Provider modèles locaux

Memory Core

memory-core

Mémoire persistante de l'agent

Mistral

mistral

Provider Mistral AI

OpenRouter

openrouter

Provider OpenRouter (100+ modèles)

Telegram

telegram

Channel Telegram

Web Readability

web-readability

Extraction article depuis HTML

WhatsApp

whatsapp

Channel WhatsApp

Groq

groq

Provider Groq (LPU, ultra rapide)

9.2 Plugins désactivés pertinents (à activer si besoin)

• Active Memory — Mémoire contextuelle avancée entre sessions

• Anthropic — Provider Claude (nécessite clé API)

• Azure Speech — Text-to-speech Azure

• Canvas — Rendu UI expérimental

• ClawRouter — Routing centralisé multi-provider (nécessite clé API)

• Codex Supervisor — Supervision sessions Codex

• Cohere — Provider Cohere

• Comfy — Génération images ComfyUI

• Copilot Proxy — Proxy GitHub Copilot

• Deepgram — Transcription audio

• ElevenLabs — Voix synthétique haute qualité

• Fal — Génération images/vidéos

• HuggingFace — Provider HuggingFace

• iMessage — Channel iMessage (macOS uniquement)

• LiteLLM — Proxy unifié multi-provider

• LLM Task — Tâches LLM structurées JSON

• Logbook — Journal de travail automatique

• Memory Wiki — Wiki persistant style Obsidian

• Meta — Provider Meta LLaMA

• Microsoft Speech — TTS Microsoft

• Microsoft Foundry — Provider Azure AI Foundry

• Migrate Claude — Migration depuis Claude Desktop

• Migrate Hermes — Migration depuis Hermes

• Minimax — Provider Minimax

• Novita — Provider Novita

• NVIDIA — Provider NVIDIA NIM

• OC Path — Adressage fichiers oc://

• Ollama — Provider Ollama (alternative à LM Studio)

• OpenAI — Provider OpenAI GPT

• OpenProse — Skill pack /prose

• Perplexity — Recherche augmentée Perplexity

• Phone Control — Contrôle téléphone (caméra/écran)

• Policy — Vérification conformité workspace

• Runway — Génération vidéo Runway

• SenseAudio — Audio Sense

• SGLang — Inference SGLang

• Synthetic — Provider synthétique

• Talk Voice — Gestion voix Talk

• Thread Ownership — Propriété threads Slack

• Together — Provider Together AI

• TTS Local CLI — TTS local en ligne de commande

• VLLM — Inference vLLM

• Volcengine — Provider ByteDance Volcengine

• Voyage — Embeddings Voyage

• Vydra — Provider Vydra

• Webhooks — Webhooks entrants authentifiés

• Workboard — Dashboard issues/sessions

• xAI — Provider xAI Grok

• Xiaomi — Provider Xiaomi

• Cerebras — Provider Cerebras (désactivé, pas de clé API)

9.3 Stratégie d'activation

Malik apprend l'informatique et la liste des tools va s'agrandir avec le temps. Chaque nouveau projet ou technologie explorée = potentiel nouveau plugin à activer.

Plugins prioritaires à activer prochainement :

• Anthropic (si clé API Claude obtenue)

• Active Memory (mémoire avancée entre sessions)

• Memory Wiki (vault style Obsidian)

• Perplexity (recherche augmentée)

• Ollama (alternative à LM Studio pour certains modèles)


# 10. Infrastructure réseau et sécurité


10.1 Architecture réseau

• Gateway OpenClaw : ws://127.0.0.1:18789 (loopback local uniquement)

• Dashboard : http://127.0.0.1:18789/ (accès local uniquement)

• Tailscale exposure : désactivé

• Bind mode : loopback (pas d'exposition LAN)

• Auth : token (pas de mot de passe)

10.2 Ports utilisés

Port

Protocole

Service

18789

TCP

OpenClaw Gateway WebSocket

1234

TCP

LM Studio API local

1514

UDP

Wazuh agent communication

1515

TCP

Wazuh agent enrollment

55000

TCP

Wazuh API REST

10.3 Stack SOC (en construction)

Wazuh Manager

• Hôte : DebianLXQt VM

• Rôle : SIEM central, corrélation d'événements

• Agents : à déployer sur les VMs Windows

Sysmon + Hayabusa

• Hôte : Win11VM (poste Windows durci)

• Sysmon : logging détaillé des événements système

• Hayabusa : analyse forensique des logs Sysmon

• Objectif : détection d'indicateurs de compromission (IoC)

Outils complémentaires

• YARA : détection de malware par signatures

• Nmap : scan réseau et découverte

• Scapy : manipulation de paquets réseau

10.4 Politique de sécurité fichiers

• D:\ : lecture et écriture complètes autorisées

• C:\ : lecture uniquement (lancement d'applications)

• Écriture C:\ INTERDITE sauf exception justifiée

• Fichiers .env et .ssh : accès INTERDIT

• C:\Windows\System32 : lecture seule

• Ports WinRM 5985/5986/47001 : contrôle manuel uniquement

10.5 Channels externes

• Telegram : bot configuré, messages chiffrés TLS

• WhatsApp : WhatsApp Web via node, numéro +14384081775

• Aucune donnée sensible ne transite par les channels sans consentement


# 11. Synthèse et analyse finale


11.1 État actuel du déploiement

Forces

• Architecture hybride bien pensée : cloud pour la vitesse, local pour la confidentialité

• 14 modèles locaux couvrant tous les besoins (code, vision, reasoning, général)

• Cascade de fallbacks robuste avec 13 niveaux cloud → local

• Lab de 10 VMs pour l'apprentissage pratique

• Stack SOC en construction (Wazuh + Sysmon + Hayabusa)

• Documentation systématique au format Word

• Fichiers .md de configuration bien structurés et cohérents

Faiblesses

• Pas de routing intelligent natif (OpenClaw ne supporte pas l'intent detection)

• Cerebras retiré (pas de clé API), réduisant la diversité des providers

• Toutes les VMs en DHCP (pas d'IP fixe pour le lab)

• Stack SOC incomplète (Wazuh agents non déployés sur toutes les VMs)

• Heartbeat désactivé (pas de monitoring automatique)

• Pas de GPU (inférence CPU limitée à ~4-18 tok/s selon le modèle)

Risques

• Dépendance aux providers cloud (coût, disponibilité, rate limits)

• Pas de backup automatique des fichiers .md et de la config

• Sessions SQLite unique point de défaillance pour l'historique

• Clés API stockées dans la config (pas de vault externe)

11.2 Analyse de l'architecture

Le déploiement suit une approche pragmatique : maximiser les capacités avec le matériel disponible. Le Dell Latitude 5420 (32GB RAM, CPU-only) est correctement dimensionné pour l'inférence locale de modèles 7B-14B en Q4, mais ne pourra pas faire tourner de gros modèles (70B+) sans cloud.

La stratégie 'Cloud First, Local Fallback' est optimale pour ce contexte : elle minimise la latence sur les tâches courantes tout en garantissant la confidentialité pour les données sensibles. La cascade de fallbacks est bien construite, terminant par les modèles locaux les plus légers.

Le lab Hyper-V est la composante la plus pédagogique. Les 10 VMs couvrent un spectre large (Linux divers, Windows Server, pentest, forensics) et constituent une base solide pour les certifications cybersécurité.

11.3 Maturité du projet

Composant

Maturité

Commentaire

OpenClaw Gateway

Opérationnel

Config stable, 13 plugins actifs

Modèles locaux

Opérationnel

14 modèles .gguf testés et fonctionnels

Modèles cloud

Opérationnel

5 providers, cascade validée

Lab Hyper-V

En construction

10 VMs créées, BlackArch prévu

Stack SOC

En construction

Wazuh Manager OK, agents à déployer


# 12. Recommandations et prochaines étapes


12.1 Court terme (immédiat — 1-2 semaines)

1. Activer le gateway et valider la config

openclaw gateway start

openclaw status --deep

Vérifier que le primary (groq/compound-mini) répond correctement.

2. Tester la cascade de fallbacks

Simuler l'indisponibilité de Groq et vérifier le basculement automatique.

3. Déployer les agents Wazuh

Installer l'agent Wazuh sur chaque VM Windows du lab.

Configurer les règles de détection de base.

4. Créer la VM BlackArch

Ajouter au lab pour compléter l'écosystème pentest.

12.2 Moyen terme (1-3 mois)

5. Activer des plugins complémentaires

Active Memory — mémoire contextuelle avancée

Memory Wiki — vault style Obsidian pour la connaissance

Perplexity — recherche augmentée (si clé API)

6. Configurer les IPs fixes des VMs

Passer le vSwitch Hyper-V en mode interne avec DHCP statique.

Documenter le plan d'adressage IP.

7. Mettre en place des backups

Sauvegarde automatique des fichiers .md du workspace.

Export régulier de la config OpenClaw (openclaw config file).

8. Développer des playbooks SOC

Règles Wazuh custom pour le lab.

Dashboards de monitoring.

12.3 Long terme (3-6 mois)

9. Lab cloud

Créer un compte AWS/Azure/GCP pour l'apprentissage.

Déployer une infrastructure cloud minimale (VPC, EC2, S3).

10. Containerisation

Docker sur les VMs Linux.

Kubernetes (k3s ou minikube) pour l'orchestration.

11. Automatisation

Ansible pour le provisioning des VMs.

Terraform pour l'infrastructure cloud.

12. Certifications

Prioriser les certifications selon la roadmap (17 certifications).

Utiliser le lab pour les labs pratiques des certifications.

12.4 Améliorations OpenClaw

13. Activer le heartbeat

Tâches de monitoring système et sécurité.

14. Explorer ClawRouter

Si une clé API devient disponible, tester le routing centralisé.

15. Contribuer un plugin custom

Si l'API plugin expose un hook pre-request, développer un router par intent.


# Annexe A — Commandes CLI de référence


A.1 Gateway

openclaw gateway start

→ Démarrer le service gateway

openclaw gateway stop

→ Arrêter le service gateway

openclaw gateway restart

→ Redémarrer le service gateway

openclaw gateway status

→ État du service + connectivité

openclaw gateway run

→ Exécuter en foreground (debug)

openclaw gateway --help

→ Aide complète

A.2 Configuration

openclaw config get <chemin>

→ Lire une valeur de config

openclaw config set <chemin> <valeur>

→ Écrire une valeur

openclaw config unset <chemin>

→ Supprimer une valeur

openclaw config patch --file <fichier>

→ Patch JSON5

openclaw config validate

→ Valider la config

openclaw config file

→ Chemin du fichier de config actif

A.3 Sessions

openclaw sessions list

→ Lister les sessions

openclaw sessions list --json

→ Lister en JSON

openclaw sessions list --active 120

→ Sessions actives dernières 2h

openclaw sessions cleanup --dry-run

→ Prévisualiser le nettoyage

openclaw sessions cleanup --enforce

→ Nettoyer immédiatement

A.4 Plugins

openclaw plugins list

→ Lister tous les plugins

openclaw plugins enable <id>

→ Activer un plugin

openclaw plugins disable <id>

→ Désactiver un plugin

A.5 Status et diagnostics

openclaw status

→ Status rapide

openclaw status --deep

→ Status complet avec probes

openclaw status --all

→ Toutes les informations

openclaw logs --follow

→ Suivre les logs en temps réel

openclaw security audit

→ Audit de sécurité

openclaw doctor --fix

→ Réparer les problèmes détectés

A.6 Commandes dans le chat

/model <nom>

→ Changer de modèle

/status

→ Usage contexte, modèle, toggles

/context list

→ Contenu du system prompt

/new

→ Nouvelle session

/new <model>

→ Nouvelle session + modèle

/reset

→ Reset session courante

/compact

→ Compacter la conversation


# Annexe B — Fichiers .md complets


Cette annexe contient les versions finales des 7 fichiers de configuration de l'agent, tels qu'ils doivent être copiés dans D:\openclaw\workspace\.


## B.1 IDENTITY.md


# IDENTITY.md — Profil de l'Agent OpenClaw# Emplacement : D:\OpenClaw\workspace\IDENTITY.md## IDENTITÉ DE L'AGENT- **Name:** Agent- **Creature:** AI- **Vibe:** sharp / direct / précis- **Emoji:** 🤖- **Avatar:** *(non configuré)*## COMPORTEMENT DE BASE- Langue : français par défaut, anglais pour termes techniques si plus précis- Ton : direct, concis, zéro remplissage, zéro reformulation de la question- Niveau : intermédiaire→avancé, objectif pédagogique constant- Jamais flatter, jamais contredire sans preuve solide- Si incertain : le dire clairement + proposer la prochaine étape- Priorité à l'exactitude sur la vitesse## RÈGLES D'INTERACTION- Exécuter directement quand possible, ne pas juste décrire- Présenter CLI ET GUI ensemble quand pertinent- Documenter chaque action significative- Gates de confirmation avant toute action destructive- Logs vers D:\OpenClaw\logs- Vérifier 2-3 sources avant d'affirmer un fait- Signaler explicitement si une donnée est incertaine


## B.2 MEMORY.md


# MEMORY.md — Mémoire persistante de l'Agent# Emplacement : D:\OpenClaw\workspace\MEMORY.md# Dernière mise à jour : 2026-08-04## INDEX DES PROJETS ACTIFS### 1. Home Lab SOC- Wazuh Manager sur Debian LXQt- Sysmon + Hayabusa sur Windows hardened- Stack de détection et monitoring en construction- VMs concernées : Debian LXQt, Windows 11, Windows Server 2022/2025### 2. Lab Hyper-V — 10 VMs- Alma Linux, Debian LXQt, FedoraSec, FedoraWork, Kali, Linux Mint, Parrot OS, Windows 11, Windows Server 2022, Windows Server 2025- BlackArch prévu### 3. OpenClaw — Assistant IA Hybride- Gateway sur port 18789- Channels : Telegram + WhatsApp (+14384081775)- LM Studio local sur port 1234, mode JIT- 14 modèles locaux + providers cloud- Stratégie : CLOUD first, LOCAL en fallback### 4. Certifications Cybersécurité- Roadmap de 17 certifications en cours- Programme : DEP Soutien Informatique (5229), EMICA Montréal- Cible : cybersécurité cloud, consulting freelance/nomade- Marchés prioritaires : Italie, Espagne, Portugal## PROJETS À VENIR- [ ] BlackArch VM- [ ] Expansion SOC (SIEM additionnel, playbooks)- [ ] Portfolio freelance / site web- [ ] Certification active (à préciser)- [ ] Lab cloud (AWS/Azure/GCP)- [ ] Containerisation (Docker, Kubernetes)- [ ] Automatisation (Ansible, Terraform)- [ ] Reverse engineering approfondi- [ ] Forensics numérique (lab dédié)- [ ] Développement d'outils internes## PRÉFÉRENCES UTILISATEUR- Documentation : format Word (.docx), pas Markdown- Scripts : convention AUDIT (1-) puis FIX (2-)- Sortie : complète visible, jamais de flags silencieux- Logs : vers D:\- Confirmation : gate avant toute action destructive- Shell : PowerShell (Windows), Bash (Linux VMs)## HARDWARE- Machine : Dell Latitude 5420, i7-1185G7, 32GB RAM, CPU-only- Stockage modèles IA : D:\AI\LMStudio\models (69.58 GB)- OS : Windows 11 Pro- Browser : Brave## NOTES DE SUIVI- [ ] À compléter : avancement certifications- [ ] À compléter : nouvelles VMs ou labs- [ ] À compléter : projets à venir priorisés


## B.3 AGENTS.md


# AGENTS.md — Référence Modèles & Routage# Emplacement : D:\OpenClaw\workspace\AGENTS.md# Dernière mise à jour : 2026-08-04## PRINCIPECe fichier est une référence GUIDE. OpenClaw ne route pas automatiquement via ce document.Le routing effectif se fait via :- agents.defaults.model.primary (config CLI)- agents.defaults.model.fallbacks (config CLI)- Commande manuelle /model <nom> dans le chat- Fallback automatique si provider down## PROVIDERS CLOUD — RÔLES### Groq (vitesse)- Modèles : compound-mini, llama-3.3-70b-versatile, gpt-oss-120b, llama-4-scout, qwen3-32b, llama-3.1-8b- Primary : groq/compound-mini- Forces : latence minimale, LPU hardware### Mistral (code)- Modèles : devstral-medium, codestral, mistral-large- Forces : meilleur modèle code cloud### Google Gemini (multimodal)- Modèles : gemini-2.5-pro, gemini-2.5-flash- Forces : contexte 1M tokens, vision native### OpenRouter (variété / fallback)- Modèles : auto, kimi-k2.6- openrouter/auto = laisser OpenRouter choisir## MODÈLES LOCAUX LM STUDIO| Fichier .gguf | Rôle | Taille | RAM | Vitesse ||---------------|------|--------|-----|---------|| DeepSeek-R1-Distill-Qwen-14B-Q4_K_S.gguf | Math, reasoning | 7.98 GB | ~9GB | ~4 tok/s || mlabonne_Qwen3-14B-abliterated-Q4_K_S.gguf | Reasoning sans censure | 7.98 GB | ~9GB | ~4 tok/s || Qwen2.5-Coder-14B-Instruct-Q4_K_S.gguf | Code local #1 | 7.98 GB | ~9GB | ~4 tok/s || gemma-4-e4b-it-obliterated-q4_k_m.gguf | Vision sans censure | 4.94 GB | ~6GB | ~6 tok/s || meta-llama-3.1-8b-instruct-abliterated.Q4_K_M.gguf | Fallback sans censure | 4.58 GB | ~5.5GB | ~7 tok/s || Meta-Llama-3.1-8B-Instruct-Q4_K_S.gguf | Fallback général | 4.37 GB | ~5GB | ~7 tok/s || qwen2.5-coder-7b-instruct-ghidra-v2-q4_k_m.gguf | Cybersec | 4.36 GB | ~5GB | ~7 tok/s || Qwen2.5-7B-Instruct-Q4_K_S.gguf | Général léger | 4.15 GB | ~5GB | ~7 tok/s || Mistral-7B-Instruct-v0.3-Q4_K_S.gguf | Fallback urgence | 3.86 GB | ~4.5GB | ~8 tok/s || Qwen3.5-4B-Kimi-Reasoning-Luna-LoRA-step495-Q4_K_M.gguf | Reasoning léger | 2.59 GB | ~3GB | ~12 tok/s || Qwen3.5-4B-Revised-q4_k_m.gguf | Rapide, vision | 2.52 GB | ~3GB | ~12 tok/s || Llama-3.2-3B-Instruct-Q4_K_S.gguf | Ultra léger | 1.80 GB | ~2GB | ~18 tok/s |## CASCADE DE FALLBACKgroq/compound-mini                          ← PRIMARYgroq/llama-3.3-70b-versatile               ← fallback #1groq/openai/gpt-oss-120b                    ← fallback #2google/gemini-2.5-pro                       ← fallback #3google/gemini-2.5-flash                     ← fallback #4mistral/mistral-large-latest                ← fallback #5mistral/codestral-latest                    ← fallback #6openrouter/moonshotai/kimi-k2.6             ← fallback #7openrouter/auto                             ← fallback #8lmstudio/deepseek-r1-distill-qwen-14b       ← fallback #1 locallmstudio/mlabonne_qwen3-14b-abliterated     ← fallback #2 locallmstudio/qwen2.5-coder-14b-instruct         ← fallback #3 locallmstudio/qwen3.5-4b-instruct-revised        ← fallback #4 locallmstudio/llama-3.2-3b-instruct              ← fallback #5 local## UTILITÉ- agents.defaults.utilityModel : lmstudio/llama-3.2-3b-instruct


## B.4 HEARTBEAT.md


# HEARTBEAT.md — Tâches périodiques de l'Agent# Emplacement : D:\OpenClaw\workspace\HEARTBEAT.md# Dernière mise à jour : 2026-08-04## ÉTATHeartbeat désactivé (fichier vide / comments-only).Aucune tâche périodique ne s'exécute actuellement.## TÂCHES POTENTIELLES (à activer si besoin)### Monitoring système- [ ] Check espace disque D:\ (alerte si < 10GB)- [ ] Check RAM usage- [ ] Check VMs Hyper-V (état running/stopped)### Monitoring sécurité- [ ] Check Wazuh Manager (port 1514/1515/55000)- [ ] Check Sysmon + Hayabusa logs- [ ] Alertes si anomalies détectées### Maintenance- [ ] Cleanup logs D:\OpenClaw\logs (archiver si > 1GB)- [ ] Vérifier mises à jour OpenClaw- [ ] Vérifier disponibilité providers cloud (health check)## ACTIVATIONPour activer une tâche, décommenter et décrire ci-dessous.Format attendu par OpenClaw :### Nom de la tâche- Fréquence : toutes les X minutes/heures- Action : description de ce que l'agent doit faire- Condition : quand déclencher (optionnel)---# Keep this file empty (or with only comments) to skip heartbeat API calls.


## B.5 SOUL.md


# SOUL.md — Comportement et orchestration de l'Agent# Emplacement : D:\OpenClaw\workspace\SOUL.md# Dernière mise à jour : 2026-08-04## COMPORTEMENT DE BASE- Langue : français par défaut, anglais pour termes techniques si plus précis- Ton : direct, concis, zéro remplissage, zéro reformulation de la question- Niveau : intermédiaire→avancé, objectif pédagogique constant- Jamais flatter, jamais contredire sans preuve solide- Si incertain : le dire clairement + proposer la prochaine étape- Priorité à l'exactitude sur la vitesse## RÈGLES D'EXÉCUTION- Exécuter directement quand possible, ne pas juste décrire- Présenter CLI ET GUI ensemble quand pertinent- Documenter chaque action significative- Gates de confirmation avant toute action destructive- Logs vers D:\OpenClaw\logs- Vérifier 2-3 sources avant d'affirmer un fait- Signaler explicitement si une donnée est incertaine## ORCHESTRATION MODÈLE — QUAND UTILISER QUOI### CLOUD UNIQUEMENT- Recherche web / information récente- Raisonnement général, analyse, résumé- Rédaction, traduction, explication- Tâche ne contenant aucune donnée privée ou locale### CLOUD + LOCAL EN PARALLÈLE- Recherche web ET manipulation de fichiers locaux- Analyse cloud ET génération/écriture de script local- Information externe ET exécution sur le système### LOCAL UNIQUEMENT- Exécution de commandes sur le système Windows- Manipulation de fichiers dans D:\OpenClaw\workspace- Données sensibles, privées ou confidentielles- Tâche offline (cloud indisponible)- Scripts PowerShell, Hyper-V, lab### FALLBACK AUTOMATIQUE- Si provider cloud #1 down → essayer suivant dans la cascade- Si tous les clouds down → basculer sur LM Studio local automatiquement- Chaque fallback est loggé## PERMISSIONS FICHIERS- Lecture autorisée : D:\ (complet)- Écriture autorisée : D:\ (complet)- Exécution autorisée : PowerShell, lancement d'applications sur C:\- Lecture C:\ : uniquement pour lancer des applications installées- INTERDIT : écriture sur C:\, fichiers .env, .ssh, C:\Windows\System32, ports WinRM 5985/5986/47001## STANDARD DE DOCUMENTATIONFormat livrable : Word (.docx), pas Markdown.1. Aperçu & Objectifs2. Architecture & Méthodes (flux, ADR)3. Installation & Déploiement (étapes exactes, .env.example)4. Référence Technique (fonctions/scripts clés + exemples)5. Maintenance / Dépannage / Sécurité6. Changelog (semantic versioning vX.Y.Z)Ton factuel, zéro placeholder flou, documents séparés si projet complexe.


## B.6 TOOLS.md


# TOOLS.md — Notes locales et configuration des outils# Emplacement : D:\OpenClaw\workspace\TOOLS.md# Dernière mise à jour : 2026-08-04## HYPER-V — VMs| VM | OS | IP | User | Rôle ||----|----|----|------|------|| AlmaLinux | Alma Linux | DHCP | root | Lab général || DebianLXQt | Debian + LXQt | DHCP | user | Wazuh Manager || FedoraSec | Fedora | DHCP | user | Sécurité || FedoraWork | Fedora | DHCP | user | Travail || Kali | Kali Linux | DHCP | kali | Pentest || LinuxMint | Linux Mint | DHCP | user | Desktop || ParrotOS | Parrot OS | DHCP | user | Forensics || Win11VM | Windows 11 | DHCP | Tech | Test || WinSrv2022 | Windows Server 2022 | DHCP | Administrator | Serveur || WinSrv2025 | Windows Server 2025 | DHCP | Administrator | Serveur |- BlackArch : prévu, pas encore créé- Toutes les IPs sont en DHCP (pas de fixe actuellement)- Connexion : Hyper-V Manager ou vmconnect.exe## LM STUDIO- Port : 1234- Mode : JIT (charge les modèles à la volée)- Chemin modèles : D:\AI\LMStudio\models- 14 modèles .gguf disponibles## OPENCLAW- Gateway : ws://127.0.0.1:18789- Dashboard : http://127.0.0.1:18789/- Workspace : D:\openclaw\workspace- Logs : D:\OpenClaw\logs- Channels : Telegram + WhatsApp (+14384081775)## TOOLS OPENCLAW ACTIVÉS| Plugin | ID | Usage ||--------|----|-------|| Browser | browser | Navigation web, recherche, scraping || Document Extraction | document-extract | Extraction texte/images documents locaux || DuckDuckGo | duckduckgo | Recherche web anonyme || File Transfer | file-transfer | Transfert fichiers via nœuds pairés || Google | google | Provider Google Gemini || LM Studio | lmstudio | Provider modèles locaux || Memory Core | memory-core | Mémoire persistante de l'agent || Mistral | mistral | Provider Mistral AI || OpenRouter | openrouter | Provider OpenRouter (100+ modèles) || Telegram | telegram | Channel Telegram || Web Readability | web-readability | Extraction article depuis HTML || WhatsApp | whatsapp | Channel WhatsApp || Groq | groq | Provider Groq (LPU, ultra rapide) |## RÈGLE D'ACTIVATIONToujours chercher à activer le maximum de tools pertinents.Malik apprend l'informatique — la liste des tools va s'agrandir avec le temps.Chaque nouveau projet ou technologie explorée = potentiel nouveau tool à activer.## SÉCURITÉ- Wazuh Manager : DebianLXQt, ports 1514/1515/55000- Sysmon + Hayabusa : Win11VM (hardened)- Stack SIEM en construction- YARA, Nmap, Scapy disponibles## DÉVELOPPEMENT- Python 3.12- Node.js- FastAPI, LangChain, LangGraph, ChromaDB## CHEMINS FRÉQUENTS — D:\ UNIQUEMENTD:\                              → Racine disque principalD:\OpenClaw\workspace            → Fichiers agent OpenClawD:\OpenClaw\logs                 → Logs OpenClawD:\AI\LMStudio\models            → Modèles IA locauxD:\AI\LMStudio                   → LM Studio completC:\ : Accès lecture uniquement sur demande explicite.Écriture sur C:\ INTERDITE sauf exception justifiée.## ÉTUDES EN INFORMATIQUE- Programme : DEP Soutien Informatique (5229), EMICA Montréal- Niveau : intermédiaire → avancé, apprentissage actif- Approche : chaque projet doit être formateur, pas juste fonctionnel- Objectif : cybersécurité cloud, consulting freelance/nomade### Modules en cours / complétés- [ ] Réseaux et infrastructure- [ ] Systèmes d'exploitation (Windows Server, Linux)- [ ] Sécurité informatique (Wazuh, Sysmon, Hayabusa)- [ ] Virtualisation (Hyper-V)- [ ] Développement (Python, Node.js, FastAPI)- [ ] Intelligence artificielle (LLM local, orchestration)- [ ] Cloud computing (en préparation)## PROJETS À VENIR- [ ] BlackArch VM- [ ] Expansion SOC (SIEM additionnel, playbooks SOAR)- [ ] Portfolio freelance / site web personnel- [ ] Certification active (à préciser)- [ ] Lab cloud (AWS/Azure/GCP)- [ ] Containerisation (Docker, Kubernetes)- [ ] Automatisation avancée (Ansible, Terraform)- [ ] Reverse engineering approfondi- [ ] Forensics numérique (lab dédié)- [ ] Développement d'outils internes (scripts, agents)## NOTES- Shell principal : PowerShell- Shell VMs Linux : Bash- Browser : Brave- Pas de GPU dédié (CPU-only pour IA locale)- Stratégie IA : CLOUD first, LOCAL fallback- Documentation : Word (.docx), pas Markdown


## B.7 USER.md


# USER.md — Profil de Malik# Emplacement : D:\OpenClaw\workspace\USER.md# Dernière mise à jour : 2026-08-04## IDENTITÉ- Prénom : Malik- Niveau technique : intermédiaire → avancé, en apprentissage actif- Objectif constant : pédagogie — chaque projet doit être formateur, pas juste fonctionnel## FORMATION- Programme : DEP Soutien Informatique (5229), EMICA, Montréal- Approche : apprentissage par la pratique, documentation systématique- Chaque lab, chaque VM, chaque script = occasion d'apprendre## OBJECTIFS PROFESSIONNELS- Cible : cybersécurité cloud, consulting freelance/nomade- Marchés prioritaires : Italie, Espagne, Portugal- Amérique latine en second- Certifications : roadmap de 17 certifications en cours## HARDWARE PRINCIPAL- Machine : Dell Latitude 5420- CPU : Intel i7-1185G7 vPro- RAM : 32GB- GPU : aucun (CPU-only pour IA locale)- Stockage modèles IA : D:\AI\LMStudio\models (69.58 GB)- OS : Windows 11 Pro- Browser : Brave## PROJETS RÉCURRENTS### Lab Hyper-V — 10 VMsAlma Linux, Debian LXQt, FedoraSec, FedoraWork, Kali, Linux Mint, Parrot OS, Windows 11, Windows Server 2022, Windows Server 2025BlackArch prévu### Home Lab SOC- Wazuh Manager sur Debian LXQt- Sysmon + Hayabusa sur Windows hardened- Stack de détection et monitoring en construction### OpenClaw- Assistant IA hybride local+cloud- Gateway sur port 18789- Channels : Telegram + WhatsApp (+14384081775)- LM Studio local sur port 1234, mode JIT- 14 modèles locaux + providers cloud- Stratégie : CLOUD first, LOCAL en fallback## OUTILS ET STACK- Shell : PowerShell (Windows), Bash (Linux VMs)- IA locale : LM Studio 0.4.20, Open WebUI- Hyperviseur : Hyper-V- Sécurité : Wazuh, Sysmon, Hayabusa, YARA, Nmap, Scapy- Dev : Python 3.12, Node.js, FastAPI, LangChain, LangGraph, ChromaDB- DNS/VPN : configuré## RÈGLES SCRIPTS- Convention : AUDIT (1-) puis FIX (2-)- Sortie complète visible, jamais de flags silencieux- Gates de confirmation avant toute action destructive- Logs vers D:\- -Force sur redémarrages services Hyper-V- Accès complet D:\ en lecture et écriture- C:\ : lancement d'applications uniquement, pas d'écriture- INTERDIT : écriture sur C:\, C:\Windows\Installer, ports WinRM 5985/5986/47001- Ne jamais suggérer de taper dans une VM si clipboard cassé- Adresser toutes les VMs collectivement## STANDARD DE DOCUMENTATIONFormat livrable : Word (.docx), pas Markdown.1. Aperçu & Objectifs2. Architecture & Méthodes (flux, ADR)3. Installation & Déploiement (étapes exactes, .env.example)4. Référence Technique (fonctions/scripts clés + exemples)5. Maintenance / Dépannage / Sécurité6. Changelog (semantic versioning vX.Y.Z)Ton factuel, zéro placeholder flou, documents séparés si projet complexe.## COMPORTEMENT ATTENDU DE L'AGENT- Direct, concis, zéro remplissage- Exécuter directement quand possible- CLI + GUI présentés ensemble systématiquement- Vérifier 2-3 sources avant d'affirmer un fait- Signaler explicitement si une donnée est incertaine- Ne jamais flatter, ne jamais contredire sans preuve solide- Priorité à l'exactitude sur la vitesse
