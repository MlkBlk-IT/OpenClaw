OPENCLAW

Rapport Consolidé du Projet

Malik — DEP Soutien Informatique (EMICA)Dell Latitude 5420 | i7-1185G7 | 32 GB RAM | CPU-onlyMontréal, 2026


# Table des matières


Préface ····· 3

1. Vision & Architecture ····· 4

2. Infrastructure & Stack Technique ····· 5

3. Parc de Modèles ····· 6

4. Déploiement — Chronologie Complète ····· 7

5. Hardening & Sécurité ····· 8

6. Reset V2 — Reconstruction ····· 9

7. Incidents & Résolutions ····· 10

8. Évolution Méthodologique ····· 11

Conclusion ····· 12

Annexe A — Glossaire ····· 13

Annexe B — Annotations ····· 14

Annexe C — Sources ····· 15


# Préface



## Contexte


Ce document consolide l'intégralité du projet OpenClaw — un assistant IA hybride local + cloud déployé sur un Dell Latitude 5420 (i7-1185G7, 32 GB RAM, CPU-only). Le projet s'est déroulé sur deux phases principales : V1 (2-7 août 2026), marquée par des correctifs incrémentaux, et V2 (10 août 2026), marquée par un reset complet et une reconstruction architecturale.


## Objectif


Agréger 24 fichiers de documentation (rapports, plans, contrats, journaux) en un document unique, professionnel et concis. Chaque section a été synthétisée pour ne retenir que l'essentiel : décisions, justifications, résultats mesurés.


## Méthode


Lecture exhaustive des 24 fichiers sources. Croisement avec les sources officielles (docs.openclaw.ai, GitHub, OWASP). Vérification des faits et des chiffres. Rédaction en français, style technique direct.


## Structure


8 chapitres + 4 annexes. Chaque chapitre suit la progression chronologique du projet. Les annexes fournissent glossaire, annotations pédagogiques et sources.


# 1. Vision & Architecture



## 1.1 Principe fondateur


Le LLM n'est pas le cerveau monolithique du système. Il devient un composant spécialisé dans un pipeline d'exécution séquentiel : User → Intent Detection → Planning → Model Selection → Execution → Validation → Response.


## 1.2 Architecture v1 — 4 Niveaux


La première conception découplait le raisonnement de l'exécution en 4 niveaux :

Intent Router — triage strict, ne répond jamais directement

Planner — raisonnement, décomposition en sous-tâches

Executor — exécution via MCP, PowerShell, Python

Validator — contrôles qualité, checksums, logs


## 1.3 Architecture v2 — 7 Couches


L'évolution vers 7 couches a introduit un Policy Engine et un Capability Router pour répondre aux exigences de sécurité :

Interfaces — points d'entrée (Telegram, WhatsApp, Web UI)

Input Gateway — authentification, session, contexte

Intent Detection — classification de la demande

Planner — analyse et décomposition

Policy Engine & Capability Router — règles + sélection modèle/outil

Execution Layer — exécution via outils désignés

Validation, Memory & Output — contrôle qualité + réponse


## 1.4 Policy Engine


Le Policy Engine applique des règles strictes avant tout routage. Exemples : interdiction d'utiliser le cloud pour les fichiers sensibles, blocage des credentials, validation humaine pour le code de production, préférence locale pour l'analyse de sécurité.


## 1.5 Capability Router


Chaque modèle est enregistré dans un Model Registry avec ses attributs quantifiés (reasoning, coding, filesystem, latency, cost). Le routeur sélectionne la combinaison optimale selon la tâche, pas selon un ordre statique.


# 2. Infrastructure & Stack Technique



## 2.1 Hardware


Composant

Spécification

Machine

Dell Latitude 5420

Processeur

Intel i7-1185G7 vPro (4C/8T)

RAM

32 GB DDR4

GPU

Aucun — CPU-only (Iris Xe intégré)


## 2.2 OpenClaw


Version 2026.7.1-2 (commit 0790d9f). Orchestrateur multi-provider. Gateway sur ws://127.0.0.1:18789. Installation via npm (Node.js 24.19.0). Sessions stockées dans SQLite (openclaw-agent.sqlite).


## 2.3 LM Studio


Version 0.4.20. API OpenAI-compatible sur localhost:1234. Mode JIT (Just-In-Time) — chargement à la demande, TTL 60 min. 14 modèles GGUF sur disque (64.85 Go). Optimisation : GPU offload 32, KV cache GPU, 6 threads CPU → 7.1 tokens/sec (+70%).


## 2.4 Virtualisation


Hyper-V natif Windows. 10 VMs initiales, réduites à 2 VMs optimisées sur SSD NVMe (SRV1, SRV2) pour préserver la RAM. Lab de cybersécurité isolé.


## 2.5 Stack SOC


Wazuh Manager (DebianLXQt), Sysmon + Hayabusa (Win11VM), YARA, Nmap, Scapy. Détection et monitoring.


# 3. Parc de Modèles



## 3.1 Modèles locaux (LM Studio — 14 GGUF)


Modèle

Rôle

Taille

Vitesse CPU

DeepSeek-R1-Distill-Qwen-14B

Math, raisonnement

7.98 GB

~4 tok/s

Qwen2.5-Coder-14B-Instruct

Code local #1

7.98 GB

~4 tok/s

Qwen3.5-4B-Instruct-Revised

Rapide, 262K ctx

2.52 GB

~12 tok/s

Llama-3.2-3B-Instruct

Ultra léger

1.80 GB

~18 tok/s

Meta-Llama-3.1-8B-Instruct

Fallback général

4.37 GB

~7 tok/s

Qwen2.5-Coder-7B-Ghidra-v2

Cybersec, reverse

4.36 GB

~7 tok/s

Phi-4-Mini-3.8B-Instruct

Secours ultra-léger

~3 GB

~15 tok/s


## 3.2 Providers cloud


Provider

Modèles

Rôle

Groq

compound-mini, llama-3.3-70b, gpt-oss-120b

Primary (vitesse)

Google

gemini-2.5-pro, gemini-2.5-flash

Fallback #3-4

Mistral

mistral-large, codestral, devstral

Code agentique

OpenRouter

kimi-k2.6, auto

Variété / fallback

Cerebras

gpt-oss-120b

Retiré (pas de clé)


## 3.3 Cascade de fallbacks (7 étages)


Mistral Large → Gemini 2.0 Flash → OpenRouter/auto → Groq llama-3.3-70b → LM Studio qwen3.5-4b → llama-3.2-3b → qwen2.5-7b → meta-llama-3.1-8b


# 4. Déploiement — Chronologie Complète



## 4.1 Phase 0 — Initialisation (2 août)


Script PowerShell créant l'arborescence D:\OpenClaw (config, workspace, logs, scripts, models, vector_db). Audit environnemental : Node v24.18.1, Python 3.12.10, ChromaDB 1.5.9, LM Studio actif sur :1234.


## 4.2 Phase 1 — Conception (2-4 août)


Architecture v1 (4 niveaux) → v2 (7 couches). Plan de déploiement en 5 phases. Choix des modèles : 4 GGUF locaux + 5 providers cloud. Budget : 100 Go stockage, 18-22 Go RAM pour les modèles.


## 4.3 Phase 2 — Déploiement baseline (4 août)


OpenClaw 2026.7.1-2 installé. 14 GGUF téléchargés. 5 providers configurés. Telegram + WhatsApp activés. 10 VMs Hyper-V. Stack SOC en place.


## 4.4 Session 3 — Ajustements (4 août)


Changement de primary : groq/compound-mini → google/gemini-3.5-flash (rejet du system prompt). Gateway switch loopback → LAN. Appairage Redmi Note 14. SOUL.md réécrit avec routing 4 étapes.


## 4.5 Session 4 — Interventions (4 août, 20h-22h)


Résolution WhatsApp (23K tokens → /reset). Audit providers (5 OK sur 11). Libération RAM (78% → 36%, gain 13 Go). Autostart Qwen 4B. Synchronisation config complète.


## 4.6 Session 5 — Hardening (6 août)


Firewall zero-trust créé. Classification DATA/INSTRUCTION. HARD-GUARDRAIL redondant dans 4 fichiers. 8 fichiers de gouvernance. Méthode : backup → patch → validate → rollback si échec.


## 4.7 Session 6 — Dépannage LM Studio (7 août)


Clé API corrigée (401 → 200). GPU offload 0.125 → 32. preload true → false. Fallbacks 6 → 1. Vitesse 4.2 → 7.1 tok/s (+70%). 6 agents spécialisés créés.


# 5. Hardening & Sécurité



## 5.1 Firewall zero-trust


knowledge/firewall.md créé. Frontière de confiance unique couvrant contenu externe (web, email, tool_result) et frontière interne Main↔local. Classification binaire DATA/INSTRUCTION. L'appartenance au système ne donne aucune immunité.


## 5.2 HARD-GUARDRAIL


Règle absolue sur les informations sensibles, répétée dans SOUL.md, AGENTS.md, firewall.md, USER.md. Liste explicite des tentatives de contournement (impersonation, ordre direct, invocation de policy, bypass graduel).


## 5.3 Limite honnête


Un guardrail textuel reste probabiliste, jamais un hard-stop. Hardening structurel = hors-scope, à évaluer séparément.


## 5.4 CVEs OpenClaw (2026)


543 CVEs trackés en septembre 2026 (138 en avril). 2 CVEs à CVSS 9.9. Claw Chain : 4 CVEs chaînées (TOCTOU, allowlist bypass, MCP loopback). ClawHavoc : 1,400+ skills malveillants sur ClawHub. 12% des uploads malveillants en février 2026.


# 6. Reset V2 — Reconstruction



## 6.1 Pourquoi un reset


V1 avait accumulé 15 fallbacks, 4 tâches redondantes, SOUL.md désynchronisé, RAM saturée. La dette technique était devenue ingérable. V2 = repartir de zéro.


## 6.2 Architecture MAIN/LOCAL


Séparation orchestrateur/worker. MAIN = cloud (reasoning, recherche, analyse). LOCAL = local (exécution, fichiers, outils). LM Studio = inference engine. @local = chemin direct vers LOCAL.


## 6.3 Plan en 10 phases


Phase 1 — Backup & evidence baseline

Phase 2 — Reset / clean baseline

Phase 3 — Minimal runtime skeleton

Phase 4 — Cloud providers

Phase 5 — LM Studio / LOCAL

Phase 6 — Tool policy & delegation

Phase 7 — Telegram

Phase 8 — Governance files

Phase 9 — Total validation

Phase 10 — Real-world test matrix


## 6.4 Contrats machine


AI_Machine_Language_Architecture_Contract.docx — spécification lisible par machine. Universal_AI_Architecture_and_Task_Orchestration_Contract.docx — contrat universel indépendant de tout outil spécifique.


## 6.5 Résultat de la session V2


Reset complet réussi (contournement EBUSY sur SQLite). Réinstallation propre. MAIN configuré et validé. LOCAL non encore créé. Phases 3-10 non commencées.


# 7. Incidents & Résolutions


Incident

Cause

Résolution

WhatsApp silencieux

23K tokens > limite TPM Groq 12K

/reset + fallback Mistral

RAM saturée (78%)

Modèle 14B résident

Déchargement + JIT 4B

Crashes Channel Error

preload=true + 6 fallbacks

preload=false + 1 fallback

Clé API 401

Clé LM Studio incorrecte

Correction openclaw.json

Tâches planifiées échouent

-LogonType S4U incompatible

Remplacement par Interactive

Groq 401

Clé expirée/quota

Fallback Mistral

Fabrication agent

Hallucination sous pression

Firewall zero-trust + DATA/INSTRUCTION


# 8. Évolution Méthodologique



## 8.1 V1 — "Faire fonctionner"


Approche réactive. Correctifs un par un. Documentation après coup. Configuration par essais-erreurs. Markdown comme mécanisme de sécurité. Résultat : système fonctionnel mais instable.


## 8.2 V2 — "Comprendre et reconstruire"


Approche proactive. Reset complet. Architecture d'abord. Documentation avant l'action. Runtime comme mécanisme de sécurité. Résultat : architecture propre, extensible, maintenable.


## 8.3 Le tournant


Le passage V1 → V2 marque une maturité : comprendre que les correctifs ne suffisent pas, adopter une méthodologie structurée, séparer les couches, documenter pour la postérité.


# Conclusion


Le projet OpenClaw a permis de passer d'un système instable à une architecture hybride propre et documentée. Les leçons apprises sont précieuses : l'IA peut fabriquer des résultats, les correctifs empilés créent de la dette invisible, le Markdown n'est pas un mécanisme de sécurité, la configuration runtime est la source de vérité.

La sécurité d'OpenClaw (543 CVEs en 8 mois) a motivé la transition vers Hermes, une approche plus structurée. Le projet V2 reste en attente de ses phases 3-10.

Ce document servira de référence pour tout futur développement d'agent IA local.


# Annexe A — Glossaire


Agent — Configuration indépendante avec nom, workspace, modèle et fallbacks propres.

BFCL — Berkeley Function-Calling Leaderboard — benchmark standard pour le tool calling.

Cascade de fallbacks — Chaîne de modèles de secours activés en cas d'échec du primary.

Channel Error — Erreur LM Studio coupant la génération en plein milieu.

ClawHavoc — Campagne supply chain via skills malveillants sur ClawHub.

Claw Chain — 4 CVEs chaînées permettant un compromis complet d'OpenClaw.

DATA/INSTRUCTION — Classification binaire séparant les données des instructions.

Firewall zero-trust — Frontière de confiance traitant tout contenu comme non fiable.

GGUF — Format de modèle quantifié pour inference CPU/GPU via llama.cpp.

HARD-GUARDRAIL — Règle de sécurité absolue redondante dans plusieurs fichiers.

Hyper-V — Virtualisation native Windows 11 Pro.

JIT — Just-In-Time — chargement modèle à la demande, pas en permanence.

LM Studio — Serveur d'inférence local, API OpenAI-compatible sur :1234.

MAIN — Agent orchestrateur cloud (reasoning, recherche, analyse).

MCP — Model Context Protocol — standard pour outils IA.

Model Registry — Registre des modèles avec capacités quantifiées.

OpenClaw — Orchestrateur IA open-source multi-provider.

Policy Engine — Moteur de règles appliquant contraintes avant routage.

Preload — Préchargement forcé d'un modèle au démarrage.

Q4_K_M — Quantification 4 bits avec optimisation K-means — bon compromis qualité/taille.

SOUL.md — Fichier définissant identité, règles et orchestration de l'agent.

Sysmon — System Monitor — collecte d'événements Windows.

Token — Unité de texte (~0.7-0.75 mot).

TTL — Time To Live — durée avant déchargement automatique d'un modèle JIT.

Zero-trust — Principe : tout contenu est traité comme potentiellement non fiable.


# Annexe B — Annotations


📌 Pourquoi tester l'hypothèse la plus simple en premier ?

Le serveur LM Studio fonctionnait — le problème était la clé API. Tester le serveur en premier a évité de perdre du temps sur une fausse piste.

📌 Pourquoi le mode JIT plutôt que preload ?

Le preload force le chargement en RAM, ce qui entre en conflit avec la gestion JIT de LM Studio. Le JIT permet de charger à la demande et de décharger automatiquement.

📌 Pourquoi classifier DATA/INSTRUCTION en binaire ?

Le standard académique recommande des catégories typées. Le binaire est plus lisible et maintenable pour un système solo. Les sous-types servent de tags de traçabilité.

📌 Pourquoi répéter le HARD-GUARDRAIL dans 4 fichiers ?

La même règle depuis plusieurs points d'entrée réduit la probabilité qu'un chemin d'exécution la contourne. Redondance voulue, pas duplication à nettoyer.

📌 Pourquoi un guardrail textuel n'est-il pas un hard-stop ?

Un LLM peut toujours être amené à interpréter une règle de manière inattendue. Le guardrail réduit le risque, ne l'élimine pas. Le hardening structurel reste nécessaire.

📌 Pourquoi le reset V2 était nécessaire ?

V1 avait accumulé 15 fallbacks, 4 tâches redondantes, une config désynchronisée. La dette technique rendait toute évolution risquée.

📌 Pourquoi séparer MAIN et LOCAL ?

MAIN (cloud) pour le reasoning, LOCAL (local) pour l'exécution. Cette séparation permet de garder les données sensibles localement tout en bénéficiant de la puissance cloud.

📌 Pourquoi 543 CVEs est un signal d'alarme ?

OpenClaw est un projet jeune avec une surface de sécurité immature. Les CVEs montrent que même les concepts de base (sandbox, allowlist, auth) ont des failles.


# Annexe C — Sources


• docs.openclaw.ai — Documentation officielle OpenClaw

• github.com/openclaw/openclaw — Dépôt officiel

• genai.owasp.org — OWASP LLM01 / GenAI Top 10

• cheatsheetseries.owasp.org — LLM Prompt Injection Prevention

• arxiv.org/pdf/2606.31639 — Instruction-Data Separation survey

• kiteworks.com — AI Agent Security Incidents 2026

• lmstudio.ai/docs — LM Studio documentation

• huggingface.co — Modèles open-weight

• github.com/jgamblin/OpenClawCVEs — CVE tracker

• docs.openclaw.ai/fr/cli/reset — Reset documentation

• docs.openclaw.ai/fr/cli/backup — Backup documentation

• docs.openclaw.ai/fr/gateway/configuration — Configuration reference
