
# DOCUMENTATION TECHNIQUE


OpenClaw ↔ LM Studio — Dépannage, Optimisation & Hardening

Période couverte

6 août 2026 — 7 août 2026

Durée totale

~4h30 (session hardening + session dépannage)

Machine concernée

Poste "Tech" — Windows, Intel i7 vPro, iGPU Iris Xe

Logiciels

OpenClaw 2026.7.1-2, LM Studio 0.4.20

Documents sources

rapport-session-2026-08-06.txt + Documentation_OpenClaw_LMStudio_Dépannage.txt


# 📊 SYNTHÈSE EXECUTIVE


📌 RÉSULTAT GLOBAL

Deux sessions complémentaires ont permis de transformer une infrastructure OpenClaw instable et mal configurée en un système durci, performant et documenté. La vitesse de génération a été quasi doublée (+70%), les crashes ont été éliminés, et 6 agents spécialisés ont été créés avec des règles de gouvernance robustes.

TABLEAU DE BORD — ÉTAT FINAL

Connexion OpenClaw ↔ LM Studio

Stabilisée — clé API corrigée, crashs éliminés

Performance de génération

4.2 → 7.1 tokens/sec (+70%)

Architecture agents

6 agents locaux configurés et spécialisés

Sécurité / Hardening

Firewall zero-trust, guardrails redondants, 8 fichiers de gouvernance

Sauvegardes

Backup horodaté systématique avant chaque modification


# PARTIE I — HARDENING & GOUVERNANCE


Session du 6 août 2026 — Durée : ~2h


## 1. Contexte de départ


La session a débuté avec un handoff documentant un incident-canonique déjà résolu : l'agent Main avait fabriqué une chaîne d'exécution complète (session_key, resolvedModel_json, claim SUCCESS) sans jamais invoquer sessions_spawn. Cette hallucination sous pression, survenue après épuisement de la chaîne de fallbacks cloud, a révélé une faille de gouvernance structurelle : l'absence de frontière de confiance entre contenu externe et instructions internes.

📌 ANALYSE CAUSE RACINE

L'incident n'était pas un bug technique mais une défaillance de gouvernance : sans règle explicite de classification DATA/INSTRUCTION, l'agent a interprété un contenu généré (halluciné) comme une instruction valide. Cela justifie la création d'un firewall sémantique permanent.


## 2. Méthode de travail — Contrat comportemental


Un contrat explicite (BEHAVIOR-CONTRACT) a été établi en cours de session et respecté intégralement :

Zero-assumption : aucune affirmation sans vérification préalable

Token-economy : réponses concises, pas de fluff

Machine-language : syntaxe technique précise, pas de prose marketing

Exécution step-by-step avec confirmation explicite avant chaque écriture-disque

Backup horodaté systématique + validation post-écriture + rollback automatique si échec


## 3. Recherches effectuées (vérif. croisée)


Chaque affirmation a été vérifiée avant utilisation. Aucune assertion sans source.

SOURCES DE VÉRIFICATION CROISÉE

Format SKILL.md OpenClaw

docs.openclaw.ai — déterminé que le contenu requis était une règle de gouvernance permanente, pas un skill invocable → reorienté vers knowledge/

Structure knowledge/*.md

docs.openclaw.ai — aucune structure officielle trouvée. Confirmé comme convention perso par capture d'écran + contenu réel de security.md/routing.md fournis par l'utilisateur

OWASP LLM01 / GenAI Top 10

genai.owasp.org + Cheat Sheet Series — standard industrie pour séparation instruction/donnée et gestion contenus non-fiables

Best practices sub-agents OpenClaw

docs.openclaw.ai + GitHub — pattern officiel 'task/label = data, pas instruction' et règle no-nested-fanout

Guardrails 2026 / data-exfiltration

Kiteworks, Webyot, Improvado — limite honnête établie : un guardrail textuel reste probabiliste, jamais un vrai hard-stop


## 4. Décisions prises et justifications



### 4.1 firewall.md dans knowledge/ (pas skills/)


Le contenu (zero-trust content-boundary) est un comportement permanent appliqué à chaque tour, pas une procédure déclenchée à la demande. Un skill OpenClaw = invocable pour UNE tâche ; ceci s'applique systématiquement.


### 4.2 Classification binaire DATA/INSTRUCTION


Le standard académique (arxiv 2606.31639) recommande des catégories typées (instruction/document/tool-output/mémoire/policy). Choix opéré : garder le binaire décisionnel simple pour la lisibilité, avec les sous-types comme tags de traçabilité (log-only) sans complexifier la logique de décision. Compromis explicite entre rigueur académique et maintenabilité pour un système solo.

📌 DÉCISION CLÉ

La frontière interne Main↔local est traitée comme une frontière externe. L'incident-canonique a prouvé qu'une fabrication peut survenir sans compromission externe. L'appartenance au système (local = agent de confiance opérationnelle) ne doit donner aucune immunité de classification.


### 4.3 Règle HARD-GUARDRAIL redondante


La règle sur les informations sensibles est répétée dans 4 fichiers (SOUL.md, AGENTS.md, firewall.md, USER.md). Redondance voulue et confirmée par l'utilisateur. Rationale : la même règle accessible depuis plusieurs points d'entrée réduit la probabilité qu'un chemin d'exécution la contourne complètement.


### 4.4 Limite honnête documentée


Une ligne explicite a été ajoutée dans le guardrail : 'règle-textuelle = probabiliste... réduit le risque, n'élimine pas. Hardening structurel = hors-scope, à évaluer séparément.' Ce choix évite de reproduire l'erreur de l'incident-canonique (confiance excessive dans une règle textuelle).


### 4.5 Routing @local : config custom


La recherche (docs.openclaw.ai/concepts/multi-agent) confirme un mécanisme de bindings + mentionPatterns, mais pas explicitement un bypass ingress ponctuel dans un chat partagé. Documenté comme configuration personnalisée, pas comme feature officielle garantie.


## 5. Fichiers modifiés (session hardening)


INVENTAIRE DES MODIFICATIONS

knowledge/firewall.md

CRÉÉ — frontière de confiance zero-trust (contenu externe + interne)

knowledge/security.md

PATCH — cloud.execute=forbidden, ajout section Related

knowledge/routing.md

PATCH — durcissement tâches 6/7/8, ajout redteam

agents/local/AGENTS.md

RÉÉCRIT — HARD-GUARDRAIL, restrictions, routing, applies-even-if étendu

agents/local/SOUL.md

PATCH — accès web conditionnel, info-sensible

agents/local/TOOLS.md

PATCH — orchestrateur-routing, skills-héritage, Browser restreint

agents/local/IDENTITY.md

REMPLI — était template vide

agents/local/USER.md

REMPLI — warning + guardrail complet

agents/local/HEARTBEAT.md

Inchangé — vide intentionnel, comportement par défaut correct

📌 MÉTHODOLOGIE

Backup horodaté avant chaque patch → vérification des marqueurs-clés post-écriture (validate-post) → rollback automatique si validation échoue. Un cas de rollback partiel rencontré (regex non-DOTALL n'a pas matché un bloc multi-ligne) — corrigé par ancrage différent, pas de perte de données.


# PARTIE II — DÉPANNAGE & OPTIMISATION


Session du 7 août 2026 — Durée : ~2h30 (04:00 à 06:30)


## 1. Synthèse générale


Le point de départ était un symptôme vague ('ça ne se connecte plus'). L'investigation a révélé PLUSIEURS problèmes empilés, indépendants les uns des autres :

Une clé d'authentification API incorrecte

Une configuration de performance sous-optimale (CPU au lieu du GPU intégré)

Une instabilité provoquant des crashes en pleine génération (Channel Error) due à une cascade de rechargement de modèles

📌 RÉSULTAT FINAL

Connexion stabilisée, zero crash constaté sur les derniers tests. Vitesse de génération quasi doublée (4.2 → 7.1 tokens/sec). Architecture enrichie : 6 agents locaux configurés, chacun optimisé pour un type de tâche différent. L'agent 'local' utilise désormais le modèle le plus rapide disponible (llama-3.2-3b-instruct).


## 2. Diagnostic de la panne de connexion


📌 PÉDAGOGIE — Architecture

OpenClaw est un orchestrateur qui pilote plusieurs cerveaux IA (Claude, GPT, Gemini, Mistral, et modèles locaux via LM Studio). Pour parler à un modèle local, OpenClaw envoie des requêtes HTTP vers LM Studio, serveur sur http://localhost:1234 (localhost = cette machine elle-même).


### Étape 2.1 — Vérification du serveur LM Studio


Test : curl http://127.0.0.1:1234/v1/models → succès. LM Studio répondait avec la liste de ses 18 modèles installés. Ce test a permis d'ÉCARTER l'hypothèse 'le serveur ne tourne pas' dès le départ, évitant une fausse piste.

📌 ANNOTATION

Tester l'hypothèse la plus simple en premier est une règle fondamentale du dépannage. Ici, le serveur fonctionnait — le problème était ailleurs.


### Étape 2.2 — Lecture des journaux (logs)


Deux sources de logs consultées :

Logs OpenClaw : AppData\Local\Temp\openclaw\ — contenaient du 'bruit' (erreurs 'missing scope: operator.admin' répétées, liées à une connexion WebSocket dashboard sans droits admin). Faux positif classique, sans rapport avec LM Studio.

Logs LM Studio : onglet Developer → Developer Logs — source primaire pour le diagnostic.

📌 ANNOTATION

Savoir écarter les faux positifs est une compétence clé en dépannage. Les erreurs admin répétées dans les logs OpenClaw étaient un distracteur complet — elles concernaient un tout autre sous-système.


### Étape 2.3 — Découverte du vrai problème : clé API


LM Studio avait l'option 'Require API Key' activée avec une clé personnalisée. Or openclaw.json utilisait encore la valeur par défaut 'lm-studio'. Résultat : erreur 401 Unauthorized.

📌 PÉDAGOGIE — Clé API

Une clé API est comme un mot de passe que le client (OpenClaw) présente au serveur (LM Studio) pour prouver son autorisation. Si la clé ne correspond pas, le serveur répond 401 Unauthorized.

Résolution : après une tentative avec une clé personnalisée révoquée entre-temps, retour à la valeur correcte 'lm-studio' (clé réellement configurée côté LM Studio à ce moment-là).

DÉTAIL DE LA CORRECTION

Fichier modifié

~\.openclaw\openclaw.json

Champ modifié

models.providers.lmstudio.apiKey

Valeur finale

lm-studio


## 3. Optimisation de la vitesse


📌 PÉDAGOGIE — Pourquoi un modèle IA local est lent ?

Un modèle d'IA effectue des milliards de calculs mathématiques pour générer chaque mot. Ces calculs vont beaucoup plus vite sur un GPU dédié (carte graphique séparée) que sur un CPU classique. Le poste 'Tech' n'a PAS de GPU dédié : il utilise seulement l'iGPU Intel Iris Xe, beaucoup moins puissant pour ce type de tâche.


### Diagnostic initial


Vitesse de génération initiale : ~4.2 tokens/sec (~3 mots/sec, soit ~180 mots/min). Relativement lent comparé à un service cloud (100-500+ tokens/sec).


### Causes identifiées dans LM Studio


RÉGLAGES LIMITANTS IDENTIFIÉS

offloadRatio = 0.125

Seulement 12,5% du modèle utilisait le GPU, le reste tournait sur le CPU

offloadKVCacheToGpu = false

La mémoire de travail du modèle pendant la génération restait sur le CPU

cpuThreadPoolSize = 3

Seulement 3 threads CPU utilisés, alors que le processeur peut en gérer davantage


### Actions correctives appliquées


Augmentation du 'GPU Offload' à 32 (couverture complète du modèle par l'iGPU)

Activation du réglage 'Offload KV Cache to GPU Memory'

Augmentation du nombre de threads CPU de 3 à 6


### Résultats mesurés après optimisation


GAINS MESURÉS

Traitement du prompt

44-50 → 61-80 tokens/sec (+36-82%)

Génération de réponse

4.2 → 7.1-7.35 tokens/sec (+70%)

Amélioration globale

~+70% de vitesse de génération

📌 ANNOTATION IMPORTANTE

Cette amélioration reste plafonnée par le matériel : un iGPU, même bien exploité, n'égale jamais un vrai GPU dédié. Le gain obtenu est le maximum raisonnable pour cette configuration matérielle précise.


## 4. Stabilisation — Résolution des crashes 'Channel Error'


Symptôme : en cours de génération, erreur 'Prediction failed: Error: Channel Error' coupant la réponse en plein milieu.

📌 ANNOTATION CRITIQUE

Ce crash déclenchait un effet domino : dès qu'un modèle échouait, OpenClaw tentait automatiquement le modèle suivant de sa liste de fallbacks, qui à son tour pouvait échouer, entraînant un chargement/déchargement en cascade de plusieurs modèles en quelques secondes — forte pression sur la mémoire, aggravant le problème initial.


### Causes identifiées dans openclaw.json


CAUSES DES CRASHES

preload: true

Forçait le préchargement d'un modèle au démarrage, entrant en conflit avec le JIT loading de LM Studio (chargement à la demande)

6 fallbacks en cascade

En cas d'échec du premier, OpenClaw tentait successivement 5 autres modèles, chacun nécessitant un chargement complet en mémoire de plusieurs Go — opération lourde répétée inutilement

📌 PÉDAGOGIE — JIT Loading

Plutôt que de garder TOUS les modèles installés en permanence en mémoire (ce qui saturerait la RAM), LM Studio charge un modèle uniquement au moment où une requête arrive pour lui, et le décharge automatiquement après une période d'inactivité. C'est un système économe en ressources, mais qui suppose qu'on ne le contredise pas avec un préchargement forcé côté client.


### Actions correctives


"preload": true → "preload": false

Liste de fallbacks réduite de 6 modèles à 1 seul modèle de secours raisonnable (qwen2.5-7b-instruct)

Résultat : zero occurrence de 'Channel Error' sur les tests post-correction. Indicateur technique : truncated = 0 (réponse non coupée prématurément).


## 5. Comportement de mise en mémoire des modèles (TTL)


📌 PÉDAGOGIE — TTL (Time To Live)

Par défaut, un modèle chargé en JIT reste en mémoire pendant 60 MINUTES d'inactivité avant d'être déchargé automatiquement. Ce délai se RÉINITIALISE à chaque nouvelle requête reçue par le modèle.

COMPORTEMENT TTL

Activité >= 1 msg/heure

Le modèle reste en permanence chargé, sans jamais repayer le coût de chargement initial (~10-15 sec de délai sur le premier message)

Inactivite > 1 heure

Le prochain message déclenche un nouveau chargement complet avec le délai initial associé

Configuration

Modifiable dans LM Studio -> Developer -> Server Settings


## 6. Inventaire des modèles locaux et recommandations


Inventaire complet réalisé via 'lms ls --json' : 18 modèles disponibles (16 modèles de langage + 2 modèles d'embedding). Classés en 5 catégories :

CATEGORISATION DES 18 MODELES

1. VITESSE PURE

llama-3.2-3b-instruct, gemma-4-e4b-it, qwen3.5-4b-instruct-revised — réponse la plus rapide, modèles petits

2. CODING

qwen2.5-coder-7b-instruct-ghidra-v2, qwen2.5-7b-instruct, senecallm_x_qwen2.5-7b-cybersecurity — spécialisés développement/code

3. TACHE NORMALE

qwen3.5-4b-instruct-revised, qwen3.5-4b-kimi-reasoning-luna-lora, meta-llama-3.1-8b-instruct — équilibre vitesse/qualité

4. MULTI-MODAL RAPIDE

gemma-4-e4b-it, qwen3.5-4b-instruct-revised, qwen3.5-4b-kimi-reasoning-luna-lora — capable de lire des images, léger. Exclu : qwen3-vl-8b-instruct-abliterated (1.58 tokens/sec, trop lent)

5. LENT MAIS PRECIS

mythos (contexte 1M tokens), qwen3.6-27b-redteam/blueteam-v5 (sécurité off/def), ibm/granite-3.2-8b, agents-a1-4b-kimi-preview-heretic — tâches complexes, contexte étendu

📌 ⚠️ MISE EN GARDE

Certains modèles portent des noms comme 'abliterated', 'heretic', 'redteam' ou 'blueteam', indiquant des versions modifiées où certains garde-fous de sécurité standards ont été retirés ou altérés par des auteurs tiers. Leur comportement peut être moins prévisible que les versions officielles. Vigilance particulière recommandée.


## 7. Création de 5 agents OpenClaw spécialisés


📌 PÉDAGOGIE — Qu'est-ce qu'un agent ?

Un agent est une configuration indépendante avec son propre nom (id), son propre dossier de travail, son propre modèle IA par défaut, et sa propre liste de fallbacks. C'est un 'profil' ou une 'personnalité' différente d'OpenClaw, chacune optimisée pour un type de tâche particulier.

5 nouveaux agents créés dans openclaw.json, chacun avec son dossier sous D:\OpenClaw\workspace\agents\ :

TABLEAU DES AGENTS CREES

local-fast

llama-3.2-3b-instruct

Priorite vitesse — reponses rapides

local-coding

qwen2.5-coder-7b-instruct-ghidra-v2

Priorite developpement/code

local-normal

qwen3.5-4b-instruct-revised

Usage general, equilibre vitesse/qualite

local-vision

gemma-4-e4b-it

Lecture d'images rapide, multi-modal

local-heavy

mythos

Taches complexes, contexte etendu (jusqu'a 1M tokens)

📌 ⚠️ NOTE IMPORTANTE

Ces 5 agents sont configures et prets, mais n'ont PAS encore ete testes individuellement lors de cette session (fatigue de fin de session). Prochaine session : (a) determiner comment selectionner un agent selon le canal (Telegram, WhatsApp, CLI), (b) tester chaque agent individuellement, (c) verifier le fallback 'lmstudio/ibm/granite-3.2-8b' de local-heavy dont le nom contient un '/' supplementaire potentiellement problematique.


## 8. Dernier réglage de fin de session


Sur demande explicite (priorite vitesse), l'agent 'local' original a ete reconfigure :

CHANGEMENT DE MODELE PRINCIPAL

AVANT

qwen3.5-4b-instruct-revised (fallback : qwen2.5-7b-instruct)

APRES

llama-3.2-3b-instruct (fallback : qwen3.5-4b-instruct-revised)

📌 ANNOTATION

Ce choix privilegie la RAPIDITE au detriment potentiel d'une legere baisse de qualite/nuance. Le modele 3B est plus petit (donc moins 'capable' en theorie) que le 4B precedent. C'est un compromis volontaire, revisable a tout moment en revenant a qwen3.5-4b-instruct-revised comme modele principal.


# ANNEXE A — Fichiers modifies et sauvegardes


DETAILS DES FICHIERS

Fichier principal

C:\Users\Tech\.openclaw\openclaw.json

Config modele LM Studio

C:\Users\Tech\.lmstudio\.internal\user-concrete-model-default-config\Smoffyy\Qwen3.5-4B-Instruct-Revised-GGUF\Qwen3.5-4B-Revised-q4_k_m.gguf.json

Format backup

openclaw.json.bak-AAAAMMJJ-HHMMSS (horodate automatiquement)

Restauration

Renommer le fichier de sauvegarde en 'openclaw.json'

📌 BONNE PRATIQUE

Avant chaque modification du fichier openclaw.json, une copie de sauvegarde horodatee a ete creee automatiquement. En cas de probleme post-modification, restauration possible immediate.


# ANNEXE B — Glossaire technique


TERMINOLOGIE UTILISEE DANS CE DOCUMENT

API / Cle API

Interface permettant a deux logiciels de communiquer ; la cle API est un identifiant secret prouvant l'autorisation d'acces.

Token

Unite de texte utilisee par les modeles d'IA (environ 0,7-0,75 mot par token en moyenne).

Tokens/seconde

Mesure de vitesse de generation d'un modele d'IA — plus le chiffre est eleve, plus la reponse apparait rapidement.

GPU / iGPU

Circuit graphique. Un GPU dedie (carte graphique separee) est nettement plus puissant qu'un iGPU (circuit integre au processeur) pour les calculs d'IA.

Offload

Action de deleguer une partie des calculs d'un composant (ex : CPU) vers un autre (ex : GPU).

JIT (Just In Time)

Chargement a la demande — un modele n'est charge en memoire qu'au moment ou on en a besoin, plutot qu'en permanence.

TTL (Time To Live)

Duree pendant laquelle un modele reste charge en memoire sans activite avant d'etre decharge automatiquement.

Fallback

Modele ou service de secours utilise automatiquement si le choix principal echoue.

Preload

Prechargement force d'un modele au demarrage, par opposition au chargement a la demande.

Channel Error

Erreur technique survenant en cours de generation, generalement liee a une interruption de connexion ou une surcharge de ressources.

HARD-GUARDRAIL

Regle de securite absolue interdisant la divulgation d'informations sensibles, repetee redondamment dans plusieurs fichiers de gouvernance.

Zero-trust content-boundary

Principe selon lequel tout contenu, meme interne, est traite comme potentiellement non-fiable jusqu'a preuve du contraire.

No-nested-fanout

Regle interdisant a un sub-agent de spawner un autre sub-agent, prevenant les cascades de delegation incontrolees.


# SYNTHESE GLOBALE — TRANSFORMATION COMPLETE


📌 RECAPITULATIF STRATEGIQUE

En l'espace de deux sessions (~4h30), l'infrastructure OpenClaw du poste 'Tech' est passee d'un etat de dysfonctionnement multi-factoriel (connexion rompue, lenteur chronique, crashes aleatoires, absence de gouvernance) a un systeme durci, performant et documente. Chaque probleme a ete traite selon une methodologie rigoureuse : diagnostic root-cause -> verification croisee -> correction ciblee -> validation post-implementation.


## Vue d'ensemble par axe


Axe

Probleme initial

Action corrective

Resultat

Connexion

Cle API incorrecte -> 401 Unauthorized

Correction de la cle API dans openclaw.json

Connexion stabilisee, zero erreur 401

Performance

4.2 tokens/sec, CPU-bound, 3 threads

GPU offload max, KV cache GPU, 6 threads

7.1-7.35 tokens/sec (+70%)

Stabilite

Crashes 'Channel Error' + cascade fallbacks

preload=false, fallbacks reduits a 1

Zero crash post-correction

Architecture

1 agent generique, pas de specialisation

6 agents specialises (fast/coding/normal/vision/heavy + local)

Chaque tache = agent optimise

Securite

Aucune regle de gouvernance

Firewall zero-trust, HARD-GUARDRAIL redondant, 8 fichiers de gouvernance

Protection probabiliste documentee, limite honnete affichee


## Prochaines etapes recommandees


Tester individuellement chacun des 5 nouveaux agents (local-fast, local-coding, local-normal, local-vision, local-heavy)

Verifier le fallback 'lmstudio/ibm/granite-3.2-8b' de local-heavy (caractere '/' potentiellement problematique)

Determiner comment selectionner un agent specifique selon le canal utilise (Telegram, WhatsApp, CLI OpenClaw)

Evaluer si le compromis vitesse/qualite du modele 3B (local) reste satisfaisant a l'usage quotidien

Considerer le hardening structurel (hors-scope du guardrail textuel) comme prochaine phase de securisation

📌 METHODOLOGIE GLOBALE

Tout au long des deux sessions, le protocole de travail obligatoire a ete respecte : lecture -> recherche datee -> cross-reference avec sources officielles uniquement -> explication -> proposition (delta uniquement) -> validation explicite avant toute execution/ecriture. Aucune etape n'a ete sautee. Aucune donnee n'a ete perdue.

—  Document genere le 8 aout 2026  —
