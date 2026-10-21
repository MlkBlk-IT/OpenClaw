
# Document de Synthèse : Architecture de l'Assistant Hybride "OpenClaw"



## 1. Vision Stratégique et Changement de Paradigme


L'architecture d'OpenClaw repose sur un principe fondamental : le modèle de langage (LLM) n'est plus le cerveau monolithique du système, mais devient un composant spécialisé au sein d'un pipeline d'exécution. L'objectif est de structurer la réflexion de l'agent selon un flux séquentiel robuste, où la prise de décision est découplée de l'exécution.


## 2. L'Architecture Décisionnelle en 7 Couches


Pour garantir un comportement cohérent et extensible, le système est structuré en sept couches distinctes :

Interfaces : Points d'entrée de l'utilisateur (WhatsApp, Telegram, Web UI).

Input Gateway : Gestion de l'authentification, de la session et du contexte.

Intent Detection : Classification initiale de la demande (Question, Recherche, Tâche locale, Code, etc.).

Planner : Analyse et décomposition de la mission en sous-tâches logiques. Identification des capacités requises.

Policy Engine & Capability Router : Moteur de règles appliquant les contraintes de sécurité et de confidentialité, suivi du routeur qui sélectionne la combinaison optimale d'outils et de modèles (Cloud ou Local).

Execution Layer : Exécution de la tâche via les outils désignés (MCP, Python, PowerShell, Navigateur, Système de fichiers).

Validation, Memory & Output : Contrôle qualité des résultats (erreurs, logs, checksums), mise à jour de la mémoire et formulation de la réponse finale.


## 3. Séparation Critique : Raisonnement vs Exécution


Un piège fréquent dans la conception d'agents est de confier toutes les tâches (réflexion, manipulation de fichiers, exécution, vérification) au même modèle. L'architecture d'OpenClaw instaure une séparation stricte :

Le modèle n'a pas les droits directs : Il analyse la requête et propose un plan (Reasoning Model).

L'Executor agit : Un composant dédié (Tool Executor) exécute concrètement les commandes et manipule les fichiers.

Le Validator contrôle : Un vérificateur indépendant s'assure du succès de la tâche (erreurs, logs, création de fichiers) avant la génération de la réponse finale par le Writer.


## 4. Schéma des Capacités et Registre des Modèles


Plutôt que de router statiquement vers un modèle, OpenClaw utilise un Capability Schema. Chaque modèle est enregistré dans un Model Registry avec ses attributs quantifiés, permettant au système de choisir le profil le plus adapté.


### Exemple de Schéma des Capacités (Capability Schema)


capabilities:  reasoning: "Analyse complexe, architecture, décision"  coding: "Création et modification de code"  filesystem: "Lecture écriture fichiers"  powershell: "Administration Windows"  python: "Automation et data processing"  browser: "Recherche web"  vision: "Analyse image"  long_context: "Documents volumineux"  documentation: "Word, PDF, Markdown"  security_analysis: "Logs, malware, forensic"


### Exemple Conceptuel de Registre (Model Registry)


qwen2.5-coder-14b:  capabilities:    coding: 95    reasoning: 80    documentation: 75  location: local  cost: 0  latency: low


## 5. Le Rôle Clé du Policy Engine (Moteur de Règles)


L'introduction du Policy Engine entre le Planner et le Capability Router transforme le chatbot en un agent autonome et contrôlé. Il applique des règles métier et de sécurité strictes avant tout routage :

Système de fichiers local : Interdiction stricte d'utiliser des modèles Cloud.

Détection d'identifiants (credentials) : Blocage immédiat de tout upload externe.

Code de production : Obligation de demander une validation humaine préalable.

Analyse de sécurité : Préférence absolue pour le traitement en local.


## 6. Matrice de Sélection : Capacités, Modèles et Outils


Capacité (Capability)

Stratégie Locale (Local Model)

Stratégie Cloud (Cloud Model)

Outil d'Exécution (Tool)

Reasoning (Raisonnement)

mlabonne/qwen3-14b-abliterated → deepseek-r1-distill-qwen-14b

GPT OSS 20B → GPT OSS 120B → Gemini Pro

Aucun

General (Polyvalence)

Meta Llama 3.1 8B → Gemma → Qwen3.5

Groq Compound → Llama 70B → Gemini Flash

Aucun

Coding (Développement)

Qwen2.5 Coder 14B → Qwen2.5 Coder 7B Ghidra

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

Modèle Local (Qwen, Llama)

Modèle Cloud

Python-docx, Word, PDF

Research (Recherche)

Non applicable (hors ligne)

Gemini Pro → Kimi K2.6

Browser, API Web


## 7. Études de Cas Pratiques



### Cas 1 : Analyse de Sécurité Locale


Requête : "Analyse mes logs Hayabusa et crée un rapport Word."

Le Planner identifie des besoins élevés en Raisonnement, Fichiers, Analyse de Sécurité et Documentation (DOCX), mais aucun besoin Cloud. Le Policy Engine verrouille l'exécution en local pour des raisons de confidentialité.

Reasoning : deepseek-r1-distill-qwen-14b (Local)

Filesystem : MCP

Security : Local Knowledge

PowerShell : Local Executor

Document : Python-docx

Cloud : NO

Pipeline : CSV Hayabusa → Python Parser → Local LLM Analysis → Report Generator → DOCX → Validation.


### Cas 2 : Recherche Documentaire Externe


Requête : "Compare Windows Server 2025 et 2022 avec les dernières docs Microsoft."

Le Planner identifie un besoin de Raisonnement, de Web Search, de Contexte Long et de Données Fraîches issues d'une source d'autorité (Microsoft).

Stratégie : Cloud

Modèle : Gemini Pro ou GPT OSS 120B

Outils : Browser Search + Document Retrieval

Pipeline : Search (Microsoft Learn) → RAG → Reasoning Model → Answer.


## 8. Conclusion


L'architecture d'OpenClaw repose sur une orchestration intelligente où la décision stratégique précède l'exécution. En intégrant un Policy Engine et un Capability Router s'appuyant sur un Model Registry détaillé, l'assistant devient véritablement hybride, contrôlé et sécurisé. Cette fondation garantit que le traitement est systématiquement orienté vers l'outil et le modèle les plus appropriés, tout en respectant scrupuleusement les contraintes de l'environnement (Local vs Cloud).
