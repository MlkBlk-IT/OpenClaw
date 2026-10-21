PLAN DE DÉPLOIEMENT OPENCLAW

Architecture Hybride IA — Local & Cloud

Dell Latitude 5420 | i7-1185G7 | 32 Go RAM | Windows 11 Pro Build 26200 | 2 août 2026Document basé sur l'audit environnemental réel + sources cross-référencées (npmjs, docs.openclaw.ai, lmstudio.ai, huggingface, promptquorum, freecodecamp, sphereinc)

────────────────────────────────────────────────────────────────────────────────


# 0. État Actuel de l'Environnement (Audit)


⚠️ STOCKAGE D: CRITIQUE: D: à 81.6% (48.9 Go libres / 265.7 Go). Le projet nécessite 83–112 Go. E: est une carte SD → pas de migration possible. Libérer D: manuellement (supprimer VMs inutilisées).

✅ RAM OK: Task Manager confirme 10.3/31.4 Go utilisés (33%). Suffisant pour charger les modèles GGUF (cible 18–22 Go max).

✅ OUTILS DÉJÀ INSTALLÉS: Node v24.18.1, npm 12.0.2, Python 3.12.10, pip 26.2, PowerShell 7.6.4, Git 2.55.0 — tous présents et fonctionnels.


## 0.1 Inventaire complet des packages installés


Packages Python (audit.json confirme) :

chromadb 1.5.9 ✅ — base vectorielle locale

sentence-transformers 5.6.1 ✅ — embeddings all-MiniLM-L6-v2

openai 2.52.0 ✅ — client API OpenAI-compatible

requests 2.34.2 ✅ — HTTP client

httpx 0.28.1 ✅ — async HTTP

fastapi 0.141.1 ✅ — serveur API (optionnel)

uvicorn 0.52.1 ✅ — ASGI server (optionnel)

torch 2.13.0+cpu ✅ — inference CPU PyTorch

Packages npm (audit.json confirme) :

openclaw — ❌ NON INSTALLÉ (à faire Phase 3)

pm2 — ❌ NON INSTALLÉ (optionnel, daemon manager)

nodemon — ❌ NON INSTALLÉ (optionnel, dev reload)

Outils système (audit.json confirme) :

node v24.18.1 ✅ — runtime OpenClaw (requis Node 24.15+ d'après docs.openclaw.ai)

npm 12.0.2 ✅ — package manager

python 3.12.10 ✅ — ChromaDB + scripts

pip 26.2 ✅ — gestion packages Python

git 2.55.0 ✅ — clonage repos

pwsh 7.6.4 ✅ — scripts d'automatisation

go 1.26.5 ✅ — non requis

rustc 1.97.1 ✅ — non requis

cargo 1.97.1 ✅ — non requis

scoop 0.5.3 ✅ — gestionnaire paquets (optionnel)

choco 2.7.3 ✅ — gestionnaire paquets (optionnel)

Outils cybersécurité manquants (non bloquants) :

docker — ❌ non requis (ChromaDB tourne en natif Python)

nmap — ❌ non requis

wireshark — ❌ non requis

hashcat — ❌ non requis

john — ❌ non requis


## 0.2 Vérification LangChain / RAG


L'audit ne liste pas explicitement langchain, langchain-community, ou langchain-chroma. Vérifier avant déploiement :

python -c "import langchain; print(langchain.__version__)"

python -c "import langchain_chroma; print('OK')"

python -c "import langchain_community; print('OK')"

Si manquant, installer :

pip install langchain langchain-chroma langchain-community


## 0.3 Structure dossiers existante


Tous les dossiers suivants existent déjà sur D:\OpenClaw (à conserver sur D:) :

D:\OpenClaw\config

D:\OpenClaw\models

D:\OpenClaw\logs

D:\OpenClaw\cache

D:\OpenClaw\scripts

D:\OpenClaw\vector_db ✅ (ChromaDB déjà initialisé)

D:\OpenClaw\workspace\inputs

D:\OpenClaw\workspace\outputs


## 0.4 LM Studio — État actuel


Serveur actif sur http://127.0.0.1:1234 ✅

Modèles chargés en mémoire :

adi-qwen2.5-coder-7b-kimi2.7-code (4.36 Go GGUF)

kimi-k2.7-code (1.43 Go + 1.76 Go mmproj)

text-embedding-nomic-embed-text-v1.5

Modèles sur disque D:\AI\LMStudio\models :

adi-qwen2.5-coder-7b-kimi-q4_k_m.gguf — 4.36 Go

imatrix.gguf — 1.43 Go

mmproj-Kimi-K2.7-Code-F32.gguf — 1.76 Go

Source validation: LM Studio expose une API OpenAI-compatible sur le port 1234 (docs officiels lmstudio.ai). Le format GGUF Q4_K_M est le standard recommandé pour inference CPU-only (PromptQuorum, juillet 2026).


# PHASE 0 — Libération de l'Espace Disque D: (Bloquante)


Durée estimée : 15–30 min | Statut : IMPÉRATIF avant toute autre étape

⚠️ CONTRAINTE: E: est une carte SD — pas de migration possible. Tout reste sur D:. Libérer manuellement.

Étape 0.1 — Identifier les gros consommateurs d'espace

Lister les dossiers les plus volumineux sur D:

Get-ChildItem "D:\" -Directory | ForEach-Object { $size = (Get-ChildItem $_.FullName -Recurse -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB; [PSCustomObject]@{Folder=$_.Name; SizeGB=[math]::Round($size,2)} } | Sort-Object SizeGB -Descending | Select-Object -First 15

Note: Cibler les VMs Hyper-V, les dossiers de téléchargements, les caches.

Étape 0.2 — Supprimer les VMs Hyper-V inutilisées

Lister les VMs existantes :

Get-VM | Select-Object Name, State, @{N="SizeGB";E={[math]::Round((Get-ChildItem (Get-VMHardDiskDrive -VMName $_.Name).Path -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum/1GB,2)}}

Note: Ne supprimer que les VMs identifiées comme inutiles. Conserver le lab de sécurité.

Étape 0.3 — Nettoyer les caches et fichiers temporaires

Vider les caches Windows et navigateurs :

Remove-Item "$env:TEMP\*" -Recurse -Force -ErrorAction SilentlyContinue

Note: Cible : libérer au minimum 40 Go pour atteindre ~90 Go libres sur D:.

Étape 0.4 — Vérifier l'espace libéré

Confirmer que D: a suffisamment d'espace :

Get-Volume D | Select-Object DriveLetter, @{N="FreeGB";E={[math]::Round($_.SizeRemaining/1GB,2)}}, @{N="TotalGB";E={[math]::Round($_.Size/1GB,2)}}, @{N="UsedPct";E={[math]::Round(($_.Size-$_.SizeRemaining)/$_.Size*100,1)}}

Vérification: Cible : > 90 Go libres (soit < 66% utilisé).


# PHASE 1 — Téléchargement et Configuration des Modèles GGUF


Durée estimée : 45–90 min | Dépend de la bande passante

ℹ️ STRATÉGIE MODÈLES: Conserver adi-qwen2.5-coder-7b (4.36 Go) comme modèle code principal. Remplacer le Qwen3-Coder-14B prévu (9 Go) par ce modèle existant pour économiser ~5 Go RAM. Télécharger les 3 manquants.


## 1.1 Modèles retenus (parc final)


Modèle

Taille GGUF

RAM estimée

Rôle

Source HF

Priorité

adi-qwen2.5-coder-7b-kimi-q4_k_m

4.36 Go

~5 Go

Code, scripting, raisonnement technique

Déjà présent

1 (chargé)

Meta-Llama-3.1-8B-Instruct-GGUF (bartowski)

~5 Go

~6 Go

Usage général, dialogue

huggingface.co/bartowski

2

Qwen3.5-4B-Instruct-GGUF (bartowski)

~3.5 Go

~4 Go

Traitement rapide, résumé

huggingface.co/bartowski

3

Phi-4-mini-instruct-GGUF (bartowski)

~3 Go

~3.5 Go

Secours ultra-léger, toujours chargé

huggingface.co/bartowski

4

Charge mémoire totale estimée : ~18–19 Go (vs 22 Go avec le 14B). Marge confortable sur 32 Go.

Étape 1.2 — Télécharger Llama-3.1-8B-Instruct (Q4_K_M)

Via LM Studio GUI ou CLI lms :

lms get bartowski/Meta-Llama-3.1-8B-Instruct-GGUF --quantization Q4_K_M

Note: Taille ~5 Go. Délai estimé : 10–20 min selon connexion.

Vérification: lms ls | Select-String "Llama-3.1"

Étape 1.3 — Télécharger Qwen3.5-4B-Instruct (Q4_K_M)

Via LM Studio GUI ou CLI lms :

lms get bartowski/Qwen3.5-4B-Instruct-GGUF --quantization Q4_K_M

Note: Taille ~3.5 Go. Modèle rapide pour requêtes simples.

Vérification: lms ls | Select-String "Qwen3.5"

Étape 1.4 — Télécharger Phi-4-Mini-Instruct (Q4_K_M)

Via LM Studio GUI ou CLI lms :

lms get bartowski/Phi-4-mini-instruct-GGUF --quantization Q4_K_M

Note: Taille ~3 Go. Toujours chargé en mémoire comme filet de secours.

Vérification: lms ls | Select-String "Phi-4"

Étape 1.5 — Charger les modèles dans LM Studio

Charger chaque modèle via l'interface ou CLI :

lms load bartowski/Meta-Llama-3.1-8B-Instruct-GGUFlms load bartowski/Qwen3.5-4B-Instruct-GGUFlms load bartowski/Phi-4-mini-instruct-GGUF

Note: LM Studio détecte automatiquement le CPU-only (pas de GPU dédié sur cette machine).

Vérification: Vérifier dans l'onglet 'Developer' que le serveur est actif.

Étape 1.6 — Vérifier l'API LM Studio

Tester que l'API répond et liste tous les modèles :

Invoke-RestMethod http://127.0.0.1:1234/v1/models | ConvertTo-Json -Depth 3

Note: Doit retourner les 4 modèles (adi-qwen2.5 + Llama-3.1 + Qwen3.5 + Phi-4).

Vérification: Vérifier la présence de chaque model 'id' dans la réponse JSON.

Source validation: LM Studio expose une API OpenAI-compatible sur localhost:1234 (docs lmstudio.ai). Le format GGUF Q4_K_M réduit la taille de ~70% avec perte minimale de qualité, idéal pour CPU-only (PromptQuorum, juillet 2026).


# PHASE 2 — Création des Comptes Cloud et Clés API


Durée estimée : 20–30 min | 5 comptes + 1 bot Telegram

⚠️ SÉCURITÉ: Toutes les clés API actuellement manquantes. Ne jamais hardcoder les clés dans les fichiers de config. Utiliser les variables d'environnement Windows.


## 2.1 Tableau récapitulatif des fournisseurs


Priorité

Fournisseur

URL

Limites gratuites

Variable d'env

1

Cerebras

cloud.cerebras.ai

1M tokens/jour, 30 RPM

CEREBRAS_API_KEY

2

Groq

console.groq.com

30 RPM, 1K–14.4K RPD

GROQ_API_KEY

3

OpenRouter

openrouter.ai

20 RPM, 50 RPD

OPENROUTER_API_KEY

4

Mistral AI

console.mistral.ai

1B tokens/mois, 2 RPM

MISTRAL_API_KEY

5

GitHub Models

github.com/settings/tokens

15 RPM, 150–1K RPD

GITHUB_TOKEN

—

Telegram Bot

@BotFather (Telegram)

Illimité

TELEGRAM_BOT_TOKEN

Étape 2.2 — Créer compte Cerebras (Priorité 1)

1. Ouvrir cloud.cerebras.ai → Sign up2. Aller dans API Keys → Create API Key3. Copier la clé (format sk-...)

[System.Environment]::SetEnvironmentVariable("CEREBRAS_API_KEY","sk-votre-cle-ici","User")

Note: Aucune carte de crédit requise. 1 million tokens/jour. Haut débit, idéal pour raisonnement complexe.

Vérification: [System.Environment]::GetEnvironmentVariable("CEREBRAS_API_KEY","User")

Étape 2.3 — Créer compte Groq (Priorité 2)

1. Ouvrir console.groq.com → Sign up2. Aller dans API Keys → Create API Key3. Copier la clé (format gsk_...)

[System.Environment]::SetEnvironmentVariable("GROQ_API_KEY","gsk-votre-cle-ici","User")

Note: Ultra-rapide (inférence < 100ms). Idéal pour le code et les requêtes temps réel.

Vérification: [System.Environment]::GetEnvironmentVariable("GROQ_API_KEY","User")

Étape 2.4 — Créer compte OpenRouter (Priorité 3)

1. Ouvrir openrouter.ai → Sign in2. Aller dans Keys → Create Key3. Copier la clé (format sk-or-...)

[System.Environment]::SetEnvironmentVariable("OPENROUTER_API_KEY","sk-or-votre-cle-ici","User")

Note: Agrégateur 28+ modèles (Qwen, DeepSeek, Llama, Gemma). Une seule clé pour tout.

Vérification: [System.Environment]::GetEnvironmentVariable("OPENROUTER_API_KEY","User")

Étape 2.5 — Créer compte Mistral AI (Priorité 4)

1. Ouvrir console.mistral.ai → Sign up2. Aller dans API Keys → Create API Key3. Copier la clé

[System.Environment]::SetEnvironmentVariable("MISTRAL_API_KEY","votre-cle-ici","User")

Note: 1 milliard tokens/mois gratuit. Attention : entraînement expérimental sur les données.

Vérification: [System.Environment]::GetEnvironmentVariable("MISTRAL_API_KEY","User")

Étape 2.6 — Créer token GitHub Models (Priorité 5)

1. Ouvrir github.com → Settings → Developer settings2. Personal access tokens → Fine-grained tokens → Generate3. Sélectionner 'Models' dans les permissions4. Copier le token (format ghp_...)

[System.Environment]::SetEnvironmentVariable("GITHUB_TOKEN","ghp-votre-token-ici","User")

Note: Accès à GPT-4o, Claude 3.5, Llama, Phi (100+ modèles).

Vérification: [System.Environment]::GetEnvironmentVariable("GITHUB_TOKEN","User")

Étape 2.7 — Créer le bot Telegram

1. Ouvrir Telegram → chercher @BotFather2. Envoyer /newbot3. Choisir un nom et un username (doit finir par 'bot')4. Copier le token (format 123456789:ABC...)

[System.Environment]::SetEnvironmentVariable("TELEGRAM_BOT_TOKEN","123456789:ABCvotretoken","User")

Note: Activer aussi /setcommands pour définir les commandes rapides du bot.

Vérification: [System.Environment]::GetEnvironmentVariable("TELEGRAM_BOT_TOKEN","User")

Étape 2.8 — Vérifier TOUTES les clés en une commande

Script de vérification complet :

@("CEREBRAS_API_KEY","GROQ_API_KEY","OPENROUTER_API_KEY","MISTRAL_API_KEY","GITHUB_TOKEN","TELEGRAM_BOT_TOKEN") | ForEach-Object { $v=[System.Environment]::GetEnvironmentVariable($_,"User"); if($v){ Write-Host "✅ $_ : $($v.Substring(0,4))****" } else { Write-Host "❌ $_ : MANQUANT" } }

Vérification: Toutes les lignes doivent afficher ✅. Si ❌, revenir à l'étape correspondante.

Source validation: OpenClaw supporte nativement Telegram, WhatsApp, Discord, Slack, Signal, iMessage, etc. (npmjs.com/package/openclaw). Le pairing Telegram requiert un allowlist pour la sécurité (Medium, février 2026).


# PHASE 3 — Installation et Configuration d'OpenClaw


Durée estimée : 15–20 min

ℹ️ PRÉREQUIS OPENCLAW: Node 24.15+ requis (docs.openclaw.ai). Machine actuelle : Node v24.18.1 ✅. npm global install est la méthode recommandée pour Windows natif.

Étape 3.1 — Installer OpenClaw globalement via npm

Lancer PowerShell 7 en administrateur :

npm install -g openclaw@latest

Note: Télécharge ~200 Mo de dépendances. Durée : 3–5 min.

Vérification: openclaw --version

Étape 3.2 — Vérifier l'installation

Confirmer que la CLI répond :

openclaw --versionopenclaw doctor

Note: 'doctor' vérifie que toutes les dépendances sont présentes et correctement configurées (freecodecamp, avril 2026).

Vérification: Doit afficher la version et doctor avec ✅ verts.

Étape 3.3 — Lancer l'onboarding interactif

L'assistant configure automatiquement le gateway, les channels et les skills :

cd D:\OpenClawopenclaw onboard --install-daemon

Note: --install-daemon enregistre OpenClaw comme service Windows (démarrage auto). Le port par défaut du gateway est 18789.

Vérification: openclaw gateway status

Étape 3.4 — Configurer le backend LM Studio (local)

Pendant l'onboarding, sélectionner 'OpenAI Compatible' comme provider local.Saisir l'URL : http://127.0.0.1:1234/v1API Key : lm-studio (valeur fictive, LM Studio l'ignore)

Note: LM Studio ignore la clé API mais le champ est requis par le client OpenAI (PromptQuorum, juillet 2026).

Vérification: Test de connexion automatique pendant l'onboarding.

Étape 3.5 — Configurer les providers cloud (fallback cascade)

Dans l'ordre de priorité :1. Cerebras (primary)2. Groq (fallback 1)3. OpenRouter (fallback 2)4. Mistral AI (fallback 3)5. GitHub Models (fallback 4)

Note: OpenClaw bascule automatiquement vers le provider suivant en cas d'indisponibilité (Sphere Inc., avril 2026).

Vérification: Vérifier dans openclaw.json que la section 'agents.defaults.model' contient la liste des fallbacks.

Étape 3.6 — Configurer la mémoire persistante (ChromaDB)

ChromaDB est déjà installé (v1.5.9). Vérifier le chemin dans la config :

python -c "import chromadb; c=chromadb.PersistentClient('D:/OpenClaw/vector_db'); print([col.name for col in c.list_collections()])"

Note: Le client persistant conserve les données entre les redémarrages. Le modèle d'embeddings est all-MiniLM-L6-v2 (sentence-transformers 5.6.1).

Vérification: Doit retourner une liste (vide ou peuplée) sans erreur.

Étape 3.7 — Configurer les permissions (moindre privilège)

Restreindre l'accès fichiers et commandes :

openclaw permissions set --allow-read "D:\OpenClaw\workspace" --allow-read "C:\Users\Tech\Documents" --allow-write "D:\OpenClaw\workspace\outputs" --allow-write "D:\OpenClaw\logs" --allow-exec "powershell" --deny "**\.env" --deny "**\.ssh" --deny "**\id_rsa"

Note: Ne JAMAIS autoriser l'accès au disque complet. Toujours spécifier les répertoires exacts (roadmap.sh/openclaw/security, mai 2026).

Vérification: Get-Content "D:\OpenClaw\config\permissions.json" | ConvertFrom-Json

Étape 3.8 — Créer les fichiers d'identité (SOUL.md, USER.md, AGENTS.md)

Ces fichiers Markdown définissent le comportement de l'agent (freecodecamp, avril 2026) :

notepad "D:\OpenClaw\workspace\SOUL.md"

Note: SOUL.md = personnalité et limites. USER.md = infos sur l'utilisateur. AGENTS.md = règles opérationnelles.

Vérification: Les 3 fichiers doivent exister dans D:\OpenClaw\workspace\

Source validation: OpenClaw utilise une architecture en 3 couches (Channel → Brain → Body) avec une boucle agentique en 7 étapes (freecodecamp, avril 2026). Les fichiers SOUL.md, USER.md et AGENTS.md sont le 'manuel d'opération' de l'agent. La sécurité filesystem est critique : deny explicite sur .env, .ssh, et fichiers système (roadmap.sh, mai 2026).


# PHASE 4 — Configuration Telegram et Tests de Validation


Durée estimée : 20–30 min

Étape 4.1 — Configurer le canal Telegram dans OpenClaw

Pendant l'onboarding ou via la config :

openclaw channels add telegram --token $env:TELEGRAM_BOT_TOKEN

Note: Le bot doit avoir été créé via @BotFather (Phase 2.7).

Vérification: openclaw channels list

Étape 4.2 — Approuver le pairing (sécurité)

1. Envoyer /start au bot depuis Telegram2. Noter le 'Pairing code' affiché3. Approuver dans OpenClaw :

openclaw pairing approve telegram ABCD1234

Note: Sans cette étape, le bot refuse de répondre (Medium, février 2026).

Vérification: Le bot répond 'Hello' dans Telegram.

Étape 4.3 — Passer en mode allowlist (production)

Pour sécuriser à long terme, remplacer le pairing temporaire par un allowlist :

# Éditer D:\OpenClaw\config\openclaw.json# Section channels.telegram :#   "dmPolicy": "allowlist"#   "allowFrom": ["VOTRE_TELEGRAM_USER_ID"]

Note: Récupérer l'user ID via @userinfobot sur Telegram. Empêche toute réponse à des inconnus.

Vérification: Envoyer un message depuis un autre compte Telegram → le bot doit ignorer.


## 4.4 Plan de tests (4 scénarios)


TEST 1 — Action locale sans cloud

Requête Telegram : 'Vérifie l'espace disque sur D: et liste les 5 plus gros fichiers dans Téléchargements'

Attendu : exécution PowerShell locale, réponse instantanée, zéro appel API cloud.

Get-ChildItem "C:\Users\Tech\Downloads" | Sort-Object Length -Descending | Select-Object -First 5 Name, @{N="SizeMB";E={[math]::Round($_.Length/1MB,2)}}

TEST 2 — Recherche web + rédaction + sauvegarde

Requête Telegram : 'Résume les nouveautés OpenClaw cette semaine et sauvegarde dans D:\OpenClaw\workspace\outputs\test_notes.md'

Attendu : appel Cerebras (cloud) → rédaction → écriture locale. Vérifier le fichier créé.

Test-Path "D:\OpenClaw\workspace\outputs\test_notes.md"

TEST 3 — Mémoire persistante

1. Envoyer : 'Rappelle-moi ce qu'on a testé hier'2. Redémarrer OpenClaw3. Renvoyer la même question

Attendu : l'agent interroge ChromaDB et répond sans relancer de recherche.

python -c "import chromadb; c=chromadb.PersistentClient('D:/OpenClaw/vector_db'); cols=c.list_collections(); print([col.name for col in cols])"

TEST 4 — Fallback en cascade

1. Désactiver temporairement CEREBRAS_API_KEY (renommer la variable)2. Envoyer une requête nécessitant le cloud3. Observer les logs

Attendu : OpenClaw bascule automatiquement sur Groq, puis OpenRouter, etc.

Get-Content "D:\OpenClaw\logs\*.log" -Tail 100 | Select-String "fallback|provider|switching"


# PHASE 5 — Sécurisation et Hardening


Durée estimée : 10 min

Étape 5.1 — Restreindre le gateway à localhost

Empêcher tout accès réseau externe au gateway :

# Dans D:\OpenClaw\config\openclaw.json# "gateway": { "bindHost": "127.0.0.1", "port": 18789 }

Note: Par défaut le gateway écoute sur toutes les interfaces. C'est une faille sur un réseau partagé (freecodecamp, avril 2026).

Vérification: netstat -an | Select-String "18789"

Étape 5.2 — Activer l'authentification par token

Protéger le panel admin :

# Dans D:\OpenClaw\config\openclaw.json# "auth": { "token": "votre-token-long-et-aléatoire-64-caractères" }

Note: Sans token auth, n'importe qui sur le réseau local peut contrôler l'agent.

Vérification: Tenter d'accéder à http://localhost:18789 sans token → doit refuser.

Étape 5.3 — Restreindre les permissions fichiers (ACL Windows)

Le dossier .openclaw contient les clés API. Protéger :

icacls "D:\OpenClaw\config" /inheritance:r /grant:r "%USERNAME%:(OI)(CI)F"

Note: Seul l'utilisateur courant a accès. Aucun autre utilisateur Windows ne peut lire les clés.

Vérification: icacls "D:\OpenClaw\config"

Étape 5.4 — Configurer le consent mode pour les commandes destructrices

Bloquer les commandes rm, del, format, etc. :

# Dans D:\OpenClaw\config\permissions.json# "approvals": { "exec": { "mode": "required", "denyPatterns": ["rm -rf", "del /f", "format", "Remove-Item -Recurse"] } }

Note: OpenClaw doit demander confirmation avant toute commande destructrice (roadmap.sh, mai 2026).

Vérification: Envoyer 'supprime le dossier X' → l'agent doit demander confirmation.

Étape 5.5 — Ajouter les règles de sécurité dans SOUL.md

Instructions de sécurité obligatoires en haut du fichier :

notepad "D:\OpenClaw\workspace\SOUL.md"

Note: Copier les règles de sécurité du guide ModernMomPlaybook (février 2026) : jamais accéder aux fichiers hors workspace, jamais exécuter sans approbation, jamais envoyer de messages sans confirmation.

Vérification: Le fichier SOUL.md contient les sections 'What you do', 'What you never do', 'How you communicate'.


# CHECKLIST FINALE DE DÉPLOIEMENT


□ Phase 0 — StockageD: libéré à > 90 Go libres | VMs inutiles supprimées | Caches nettoyés

□ Phase 1 — Modèles4 modèles GGUF téléchargés (adi-qwen2.5 + Llama-3.1-8B + Qwen3.5-4B + Phi-4-Mini) | API LM Studio répond sur :1234

□ Phase 1b — VérificationLangChain installé (pip list | findstr langchain) | sentence-transformers OK | ChromaDB client testé

□ Phase 2 — Clés API6 variables d'environnement définies et vérifiées (Cerebras, Groq, OpenRouter, Mistral, GitHub, Telegram)

□ Phase 3 — OpenClawnpm install -g openclaw@latest ✅ | onboard --install-daemon ✅ | LM Studio configuré comme provider local ✅ | Fallback cascade configuré ✅

□ Phase 3b — MémoireChromaDB persistant sur D:\OpenClaw\vector_db | all-MiniLM-L6-v2 fonctionnel

□ Phase 3c — PermissionsMoindre privilège appliqué | deny sur .env, .ssh, fichiers système | exec restreint à PowerShell

□ Phase 4 — TelegramBot créé via @BotFather ✅ | Token configuré ✅ | Pairing approuvé ✅ | Allowlist activé ✅

□ Phase 4b — TestsTest 1 (local) ✅ | Test 2 (cloud+écriture) ✅ | Test 3 (mémoire) ✅ | Test 4 (fallback) ✅

□ Phase 5 — SécuritéGateway bindé à 127.0.0.1 ✅ | Token auth activé ✅ | ACL fichiers restrictives ✅ | Consent mode exec ✅ | SOUL.md sécurisé ✅


# Sources et Références Cross-Référencées


OpenClaw — Package npm officiel : npmjs.com/package/openclaw — Runtime Node 24.15+, install npm global, onboarding interactif

OpenClaw — Documentation officielle : docs.openclaw.ai — Architecture gateway, channels, skills, memory

Sphere Inc. — Complete OpenClaw Setup Guide : sphereinc.com/blogs/the-complete-openclaw-setup-installation-guide — 26 Tools, 53 Skills, config production-ready, sécurité

Easton Dev — Install OpenClaw 2026 : eastondev.com/blog/en/posts/ai/20260226-openclaw-2026-installation-guide — Script vs npm vs Docker, Windows natif et WSL2

FreeCodeCamp — Build and Secure Personal AI Agent : freecodecamp.org/news/how-to-build-and-secure-a-personal-ai-agent-with-openclaw — 7-stage agentic loop, SOUL.md/USER.md/AGENTS.md, hardening

Medium / Mikhela — Telegram + OpenClaw VPS : medium.com/@mikhela65/how-to-connect-telegram-to-openclaw — Pairing, allowlist, dmPolicy

LM Studio — Local Server Docs : lmstudio.ai/docs/local-server — API OpenAI-compatible localhost:1234, endpoints /v1/chat/completions, /v1/models

PromptQuorum — CPU-Only LLM 2026 : promptquorum.com/local-llms/best-cpu-only-llm — GGUF Q4_K_M, RAM requirements, optimisation CPU

PromptQuorum — OpenAI Compatible API : promptquorum.com/local-llms/local-llm-openai-compatible-api — LM Studio & Ollama, Python/Node.js integration

Sannty.in — Run Your Own OpenAI API : sannty.in/blog/run-your-own-openai-compatible-api-with-lm-studio — lms CLI, load models, server start, verification curl

Reddit r/AIToolsPerformance — LM Studio Private API : reddit.com/r/AIToolsPerformance/comments/1qs3kwc — Performance 35-40 tok/s 70B, context window limits

Roadmap.sh — OpenClaw Security Best Practices : roadmap.sh/openclaw/security-best-practices — 12 pratiques, filesystem hardening, consent mode

Tencent Cloud — OpenClaw Access Control : tencentcloud.com/techpedia/140851 — Admin panel auth, API key management, channel-level ACL

ModernMomPlaybook — OpenClaw Security Guide 2026 : modernmomplaybook.substack.com/p/openclaw-security-guide-2026 — SOUL.md security rules, weekly updates

Meta-Intelligence — OpenClaw Agent Configuration : meta-intelligence.tech/en/insight-openclaw-agent-setup — agents add command, model binding, workspace setup

────────────────────────────────────────────────────────────────────────────────

Plan de déploiement OpenClaw — Version 2.0 — 2 août 2026Basé sur l'audit environnemental TECH + sources cross-référencées (npmjs, docs.openclaw.ai, lmstudio.ai, huggingface, promptquorum, freecodecamp, sphereinc, roadmap.sh)
