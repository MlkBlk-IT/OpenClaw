
# Document de Synthèse : Architecture de l'Assistant Hybride "OpenClaw"



# 1. Vision Stratégique et Changement de Paradigme


L'architecture d'OpenClaw repose sur un principe fondamental : le modèle de langage (LLM) n'est plus le cerveau monolithique du système, mais devient un composant spécialisé au sein d'un pipeline d'exécution. L'objectif est de structurer la réflexion de l'agent selon le flux séquentiel suivant :

User → Intent Detection → Planning → Model Selection → Execution → Validation → Response


# 2. L'Architecture en 4 Niveaux



## Niveau 1 — Intent Router (Routage d'Intention)


Ce niveau agit comme un point de triage strict et ne répond jamais directement à l'utilisateur. Il détermine la nature exacte de la requête :

• S'agit-il d'une question simple ou d'une recherche ?

• S'agit-il d'une tâche locale, de l'écriture de code ou du contrôle de l'environnement Windows ?

• Nécessite-t-il l'utilisation du protocole MCP (Model Context Protocol) ou l'orchestration de plusieurs modèles ?


## Niveau 2 — Planner (Raisonnement & Planification)


C'est à ce niveau que le modèle le plus performant (en environnement Cloud ou Local) intervient. Son rôle exclusif est de :

• Comprendre la requête et découper la tâche en sous-tâches logiques.

• Choisir les outils et modèles appropriés.

• Produire un plan d'action séquentiel (ex. 1. Lire les logs, 2. Résumer, 3. Générer DOCX, 4. Sauvegarder, 5. Retour utilisateur).


## Niveau 3 — Executor (Exécution)


Ce module exécute le plan défini par le Planner en utilisant les outils appropriés selon le contexte, par exemple : PowerShell, Python, Microsoft Word, LibreOffice, ou MCP Filesystem.


## Niveau 4 — Validator (Validation)


Avant de formuler la réponse finale, ce composant effectue des contrôles de qualité (gestion d'erreurs, vérification de création de fichiers, validation des commandes, checksums et logs). Ce n'est qu'après validation que la mission est déclarée terminée.


# 3. Matrice de Sélection des Modèles (Cloud vs Local)


La sélection du modèle s'effectue dynamiquement en fonction de la tâche à accomplir, garantissant l'utilisation du meilleur outil pour chaque besoin.


### Environnement Cloud


• Raisonnement : GPT OSS 20B → GPT OSS 120B → Gemini Pro

• Général : Groq Compound → Llama 70B → Gemini Flash

• Code : Devstral → Codestral → Mistral Large


### Environnement Local


• Raisonnement : mlabonne/qwen3-14b-abliterated → deepseek-r1-distill-qwen-14b

• Général : Meta Llama 3.1 8B → Gemma → Qwen3.5

• Code : Qwen2.5 Coder 14B → Qwen2.5 Coder 7B Ghidra


# 4. Flux d'Exécution Global


Le processus complet de bout en bout suit ce flux logique et cohérent :

WhatsApp / Telegram↓OpenClaw↓Intent Router↓Planner↓Choix Cloud ou Local↓Choix du meilleur modèle↓Choix des outils (MCP, PowerShell, Python, navigateur, etc.)↓Exécution↓Validation↓Réponse


# 5. Conclusion


Cette approche unifie la stratégie de décision. La différence entre le Cloud et le Local réside uniquement dans le lieu d'exécution du travail, et non dans la méthode de réflexion. C'est le fondement de l'architecture d'un assistant hybride robuste, modulaire et hautement cohérent.
