
# OpenClaw — Architecture HybrideMAIN + LOCAL + LM Studio


Architecture, séparation des responsabilités, délégation et execution locale

Version documentaire : 1.0 — 10 août 2026

Objectif : définir une architecture OpenClaw dans laquelle MAIN est l’orchestrateur cloud (reasoning, recherche, analyse et planification), tandis que LOCAL est le worker local utilisant LM Studio pour l’inférence et les opérations d’exécution sur le poste.


# 1. Objectif architectural


Le besoin n’est pas simplement de basculer entre un modèle cloud et un modèle local. Le besoin est une architecture multi-agent hybride : MAIN reçoit la demande, réalise le reasoning et la recherche, puis délègue une tâche d’exécution explicite à LOCAL. LOCAL utilise son propre modèle LM Studio et ses outils locaux, produit le livrable, le valide et retourne un résultat à MAIN.


# 2. Architecture cible


USER → MAIN → reasoning / research / planning → sessions_spawn(agentId="local") → LOCAL → LM Studio → tools/filesystem → résultat → MAIN → USER

@local → LOCAL → LM Studio → exécution locale


# 3. Rôles des composants


Composant

Responsabilité

Modèle

Accès

MAIN

Orchestration, reasoning, recherche, analyse, planification, validation

Cloud

Web + outils autorisés

LOCAL

Execution locale, génération/modification de fichiers, opérations machine

LM Studio

Tools locaux selon policy

LM Studio

Inference engine local

GGUF/MLX

127.0.0.1:1234

openclaw.json

Runtime configuration : agents, models, subagent permissions, bindings

N/A

Configuration Gateway

AGENTS.md / SOUL.md / knowledge

Instructions, identité, procédures et connaissances

N/A

Contexte agent


# 4. Point critique : Binding ≠ Delegation


Un Agent Binding sert à déterminer quel agent possède une conversation ou un flux entrant. Il ne remplace pas la délégation inter-agent. Pour le scénario MAIN → LOCAL, le mécanisme principal est sessions_spawn avec agentId="local" et une allowlist subagents appropriée.

Ainsi, @local peut servir de chemin direct vers LOCAL, tandis que le mode hybride permet à MAIN de rester l’orchestrateur et de lancer LOCAL comme sub-agent. Les deux mécanismes peuvent coexister.


# 5. Configuration conceptuelle


La configuration runtime doit séparer explicitement les agents et leur modèle effectif :

agents├── main│   └── model = <cloud-primary>└── local    └── model = lmstudio/<model-id>models.providers.lmstudio└── baseUrl = http://127.0.0.1:1234/v1agents.defaults.subagents├── allowAgents = ["local"]└── requireAgentId = true

Important : les identifiants et emplacements exacts doivent être adaptés à la structure de la version OpenClaw installée. Ne pas remplacer aveuglément un openclaw.json existant : faire un backup et valider le schéma/runtime avant modification.


# 6. Flux hybride détaillé


1. Réception — L’utilisateur adresse normalement sa demande à MAIN.

2. Reasoning — MAIN comprend la demande, recherche les sources, analyse les résultats et construit un plan.

3. Décision — MAIN détermine qu’une opération locale est nécessaire.

4. Delegation — MAIN appelle sessions_spawn avec agentId="local".

5. Execution — LOCAL reçoit une tâche structurée et utilise son modèle LM Studio et les tools autorisés.

6. Artifact — LOCAL crée le livrable demandé, par exemple un fichier DOCX.

7. Validation — LOCAL vérifie l’existence, l’intégrité et le résultat attendu du fichier.

8. Return — LOCAL annonce le résultat à la session MAIN.

9. Synthesis — MAIN synthétise le résultat et répond à l’utilisateur.


# 7. Exemple : recherche → DOCX local


Demande utilisateur : « Recherche la documentation officielle de X, analyse-la, puis crée un DOCX sur mon disque. »

MAIN :

effectue la recherche Web ;

sélectionne et vérifie les sources ;

analyse les informations ;

définit la structure du document ;

prépare une tâche d’exécution complète pour LOCAL.

Délégation logique :

sessions_spawn(agentId="local", task="Créer et valider le DOCX à partir des résultats fournis.")

LOCAL :

charge son contexte d’agent ;

utilise le modèle LM Studio ;

effectue la génération/écriture locale ;

valide le fichier ;

retourne path + statut + preuves de validation.


# 8. Mode direct @local


@local doit être considéré comme un raccourci explicite permettant à l’utilisateur de cibler directement l’agent LOCAL. Ce chemin ne nécessite pas que MAIN décide de déléguer.

@local → LOCAL → lmstudio/<model> → tools locaux


# 9. Rôle des fichiers Markdown


Fichier

Doit contenir

Ne doit pas être utilisé pour

SOUL.md

Identité, règles, Kernel, contraintes

Remplacer le runtime routing

AGENTS.md

Rôles, orchestration, procédures

Créer un binding runtime par instructions textuelles

knowledge/*.md

Connaissances techniques et documentation

Forcer un provider/model

openclaw.json

Agents, models, providers, subagent permissions, bindings

Stocker des secrets en clair


# 10. LM Studio : rôle et lifecycle


OpenClaw dispose d’un provider LM Studio. La documentation actuelle indique un endpoint d’inférence par défaut sur http://localhost:1234/v1 et l’utilisation de /api/v1/models pour la découverte. OpenClaw peut aussi utiliser le mécanisme de preload ; si l’on veut laisser LM Studio gérer JIT, TTL et auto-eviction, preload peut être désactivé.

lmstudio/<model-id> → http://127.0.0.1:1234/v1


# 11. Sécurité et principe du moindre privilège


LOCAL ne doit recevoir que les tools nécessaires à l’exécution.

Le filesystem access doit être limité aux répertoires nécessaires.

MAIN ne doit pas avoir besoin d’un accès local complet pour effectuer son reasoning.

La délégation doit cibler explicitement agentId="local".

Conserver une allowlist subagents restrictive plutôt que ["*"] lorsque possible.

Les API keys cloud/LM Studio doivent rester hors des fichiers Markdown et des documents de connaissance.

Toute modification de configuration doit être précédée d’un backup et suivie d’une validation runtime.


# 12. Validation de l’architecture


☐ MAIN utilise bien son modèle cloud primaire.

☐ LOCAL possède un model override LM Studio effectif.

☐ LM Studio expose le model ID exact attendu.

☐ MAIN est autorisé à spawn LOCAL.

☐ requireAgentId est activé si l’on veut empêcher une délégation implicite vers un mauvais agent.

☐ Un test @local arrive directement sur LOCAL.

☐ Un test hybride MAIN → LOCAL crée bien une session child avec agentId=local.

☐ LOCAL retourne une preuve de création/validation du fichier.

☐ MAIN reçoit le résultat du child et peut le synthétiser.


# 13. Anti-patterns à éviter


Mettre toute la logique de routing dans AGENTS.md.

Mettre des instructions de routing critiques dans knowledge/*.md.

Demander au Main de « penser » à utiliser LM Studio sans mécanisme runtime de délégation.

Faire du modèle local le fallback cloud du Main si l’objectif est une séparation de responsabilités.

Confondre Agent Binding et sub-agent delegation.

Modifier la fallback chain cloud pour résoudre un problème de routing local.

Réinstaller ou reconfigurer LM Studio avant d’avoir vérifié le routing OpenClaw.


# 14. Architecture finale recommandée


USER │ ├── @local ───────────────────────────────► LOCAL │                                             │ │                                             ▼ │                                         LM Studio │                                             │ │                                             ▼ │                                        Execution │ └── normal request ─► MAIN (Cloud)                          │                          ├── Research                          ├── Reasoning                          ├── Analysis                          ├── Planning                          │                          └── sessions_spawn(agentId="local")                                      │                                      ▼                                    LOCAL                                      │                                      ├── LM Studio                                      ├── Tools                                      └── Filesystem                                      │                                      ▼                                  Result / Evidence                                      │                                      ▼                                    MAIN                                      │                                      ▼                                    USER


# 15. Sources officielles


OpenClaw — Agent bindings : https://docs.openclaw.ai/concepts/agent-bindings

OpenClaw — Sub-agents / sessions_spawn : https://docs.openclaw.ai/tools/subagents

OpenClaw — LM Studio provider : https://docs.openclaw.ai/providers/lmstudio

OpenClaw — Model providers : https://docs.openclaw.ai/concepts/model-providers


# 16. Conclusion


La séparation recherchée est une architecture Orchestrator → Worker. MAIN est responsable du reasoning, de la recherche et de la planification. LOCAL est responsable de l’exécution locale et utilise LM Studio comme provider d’inférence. Le lien MAIN → LOCAL est réalisé par la capacité native de sub-agent et par une allowlist explicite ; @local constitue un chemin direct distinct. Les fichiers Markdown documentent les rôles et règles, tandis que openclaw.json porte la configuration runtime.
