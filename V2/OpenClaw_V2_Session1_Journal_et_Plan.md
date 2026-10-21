OPENCLAW V2 — JOURNAL DE SESSION & PLAN DE SUITE

Rebuild complet — Clean rebuild, séparation MAIN/LOCAL

VERSION=1.0 | DATE=2026-08-10 | OPÉRATEUR=Malik | MACHINE=Dell Latitude 5420

[ASSUMPTION] Ce document reflète ce qui a été fait et dit dans cette session de chat. Les commandes/résultats retranscrits proviennent des sorties collées par l'opérateur ; ils n'ont pas été ré-exécutés ou re-vérifiés indépendamment par un tiers.


# 1. RÉSUMÉ EXÉCUTIF


Cette session a exécuté la Phase 0 (backup, réalisée hors session par l'opérateur), la Phase 1 (reset complet) et la Phase 2 (squelette runtime minimal) du plan OpenClaw Reset V2. L'ancienne installation V1 a été effacée intégralement et une nouvelle installation propre a été mise en place avec un agent MAIN unique, sans provider cloud actif, sans channels configurés, et sans clés API saisies.

Point d'arrêt de cette session : configuration validée par `openclaw status` et `openclaw doctor`, prête pour la Phase 3 (ajout des providers cloud).


# 2. CE QUI A ÉTÉ FAIT — JOURNAL DÉTAILLÉ



## 2.1 Pré-requis (hors session, confirmé par l'opérateur)


Backup manuel du workspace V1 : compression ZIP de D:\OpenClaw\workspace, déplacé vers carte SD 512 Go.

[VERIFIED] Confirmé directement par l'opérateur. Ne couvre PAS C:\Users\Tech\.openclaw (config/state/credentials) — l'opérateur a jugé cette partie de la config trop défectueuse pour valoir un backup.


## 2.2 Recherche documentation officielle — Reset


Source consultée : docs.openclaw.ai/cli/reset (page officielle).

3 scopes confirmés : config (config seule), config+creds+sessions (+ OAuth/credentials/sessions, arrête le gateway), full (+ state dir incluant la base SQLite partagée + tous les workspaces, arrête le gateway).

Recommandation officielle : exécuter openclaw backup create avant reset (non suivie ici — décision opérateur, voir 2.1).

[VERIFIED] Page officielle unique, cohérente sur 3 sources croisées (docs.openclaw.ai, GitHub miroir, openclawskills.dev).


## 2.3 Tentatives de reset --scope full


#

Commande

Résultat

Cause

1

openclaw reset --scope full --yes --non-interactive

ÉCHEC — EBUSY sur openclaw.sqlite

Lock fichier au moment du unlink

2

Relance identique après vérif handle.exe (aucun handle trouvé)

ÉCHEC — EBUSY identique

Lock transitoire, non capturé par handle.exe

3

Relance après Disable-ScheduledTask

ÉCHEC — EBUSY identique

Hypothèse Scheduled Task invalidée

4

Relance après redémarrage complet Windows

ÉCHEC — EBUSY identique

Bug Windows connu (sqlite rename/swap), pas un process orphelin

Diagnostic confirmé : le workspace D:\OpenClaw\workspace a été supprimé avec succès à chaque tentative (comportement normal). Seul C:\Users\Tech\.openclaw (state dir + sqlite) a résisté au unlink interne du CLI.

Recherche croisée : deux issues GitHub officielles du dépôt openclaw/openclaw confirment un pattern EBUSY connu — #39446 (sync cloud verrouille les fichiers) et #64187 (bug Windows spécifique au moment du rename/swap sqlite).

[VERIFIED] Deux issues GitHub du dépôt officiel openclaw/openclaw corroborent le symptôme observé. Diagnostic OneDrive écarté : $env:OneDrive pointe vers C:\Users\Tech\OneDrive - CSSDM, chemin distinct de C:\Users\Tech\.openclaw — la synchronisation cloud n'était donc pas la cause directe ici.


## 2.4 Résolution — contournement manuel


Commande appliquée : Remove-Item "C:\Users\Tech\.openclaw" -Recurse -Force

Résultat : suppression réussie, confirmée par Test-Path retournant False.

[ACTION] Solution de contournement documentée par une source tierce (ClawTank) pour ce type de blocage. A fonctionné du premier coup.


## 2.5 Réinstallation — openclaw onboard --install-daemon


Choix effectués pendant le wizard, dans l'ordre :

Étape

Choix retenu

Justification

Disclaimer sécurité

Yes (compris)

Confirmation standard

Setup mode

Manual setup

Contrôle total, cohérent avec la méthodologie "une couche à la fois" du plan V2

Type de setup

Local gateway (this machine)

Correspond à la config V1 (ws://127.0.0.1:18789)

Workspace directory

D:\openclaw

Changé depuis le défaut C:\Users\Tech\.openclaw\workspace, aligné avec la mémoire projet

Model/auth provider

Skip for now

Clean slate — clés ajoutées plus tard via PowerShell pour éviter la saisie manuelle masquée

Default model

nvidia/nvidia/nemotron-3-super-120b-a12b (saisie manuelle)

Corrigé automatiquement par openclaw doctor vers nvidia/nemotron-3-super-120b-a12b

Gateway port

18789 (défaut)

Inchangé

Gateway bind address

Loopback (127.0.0.1)

Aucune exposition réseau à ce stade, cohérent LEAST_PRIVILEGE

Gateway access protection

Token (recommended)

Défaut, cohérent avec l'ancienne config V1

Token method

Generate/store plaintext token

Généré automatiquement par le wizard

Set up a chat channel now

No

Reporté à la Phase 6 du plan (après validation MAIN/LOCAL)

Search provider

Skip for now

Reporté, aucune clé à ce stade

Optional plugins (install)

Skip for now

Défaut conservé

Optional plugins (configure)

Skip for now

Défaut conservé

Gateway service runtime

Node (recommended)

Seule option viable (node:sqlite requis)

Gateway service already installed

Reinstall

Résidu de l'ancien Scheduled Task pointait vers un fichier supprimé

[VERIFIED] Onboarding complet confirmé par le message final "Onboarding complete" et par openclaw status.

[UNVERIFIED] Le format exact du modèle NVIDIA en sortie (nvidia/nemotron-3-super-120b-a12b, préfixe simple) n'a pas encore été testé avec un vrai appel API — aucune clé n'était configurée pendant cette session. Deux issues GitHub contradictoires existent sur le format correct (préfixe simple vs double) ; à confirmer au premier test réel en Phase 3.


## 2.6 Incident post-installation — Gateway ECONNREFUSED


Health check du wizard : échec, connect ECONNREFUSED 127.0.0.1:18789 (Scheduled Task fraîchement installé, pas encore démarré).

Résolution : Restarted Scheduled Task: OpenClaw Gateway — succès.

openclaw status confirme : Gateway reachable, pid 12284, state Ready, 1 agent (main), 0 sessions.

[VERIFIED] Confirmé par sortie openclaw status complète, collée par l'opérateur.


## 2.7 openclaw doctor — audit et corrections


Point détecté

Sévérité

Action prise

Modèle NVIDIA mal formaté (double préfixe)

Correctif

Corrigé automatiquement vers nvidia/nemotron-3-super-120b-a12b

Command owner non configuré

Avertissement

Non corrigé — à faire avant Phase 6 (Telegram)

Token gateway en clair (plaintext) dans openclaw.json

Avertissement sécurité

Non corrigé — migration SecretRefs recommandée avant exposition réseau

21 skills non disponibles (binaires/clés manquants)

Nettoyage

Désactivées avec succès (1Password, Whisper, Trello, Spotify, etc.)

Memory search provider = openai, sans clé

Avertissement

Non tranché — décision reportée à la session suivante

[ACTION] Doctor complete confirmé, config mise à jour avec backup automatique (~\.openclaw\openclaw.json.bak).


## 2.8 Recherche rate limits providers (2026-08-10)


Recherche effectuée pour choisir le meilleur provider gratuit de démarrage, dans un contexte où Mistral est déjà utilisé à 80% jusqu'au 23 août.

Provider

RPM

TPM

RPD/TPD

Confiance

Google Gemini (Flash)

10-15

250K-1M (variable)

250-1500 (variable)

CONTESTÉ — sources divergentes

NVIDIA NIM

40 (confirmé forums NVIDIA)

Non publié

~1000 crédits au départ, pas de RPD fixe

Élevée sur RPM

Cerebras

15-30 selon source

30K-64K

1 000 000 tokens/jour (cohérent)

Élevée sur TPD

Mistral

Déjà à 80% d'usage

—

—

N/A — à ménager

Cloudflare AI Gateway

N/A — proxy, pas un provider de modèles propre

—

—

Non pertinent comme "primary"

[CONTESTED] Les chiffres Gemini varient fortement selon la source et la date (certaines citent une réduction de 50-80% en décembre 2025, d'autres citent 1500 RPD/1M TPM en avril 2026). Le seul chiffre fiable est celui visible dans le tableau AI Studio du projet réel de l'opérateur — non consulté durant cette session.

Décision finale de l'opérateur : NVIDIA NIM retenu comme candidat, mais AUCUN provider n'a été activé — l'opérateur a choisi de garder un clean slate complet pour cette session et d'ajouter tous les providers via PowerShell dans une session ultérieure.


# 3. ÉTAT ACTUEL CONFIRMÉ (fin de session)


Composant

État

Workspace

D:\openclaw — OK

Config

~\.openclaw\openclaw.json — écrite, backup automatique présent

Gateway

Local, loopback 127.0.0.1:18789, reachable, auth token actif

Gateway service

Scheduled Task "OpenClaw Gateway" — installé, running (pid 12284)

Agents

1 (MAIN uniquement) — LOCAL non encore créé

Sessions

0 — repart à zéro, confirmé

Channels

Aucun configuré (Telegram/WhatsApp non liés, reporté Phase 6)

Modèle par défaut

nvidia/nemotron-3-super-120b-a12b — SANS credentials, non testé

Providers cloud actifs

Aucun — clean slate voulu

Skills

21 désactivées (dépendances manquantes), 26 éligibles restantes

Command owner

NON CONFIGURÉ — à faire avant tout channel

Token gateway

Plaintext dans openclaw.json — migration SecretRefs recommandée

Memory search

Provider openai sans clé — semantic recall non fonctionnel


# 4. CORRESPONDANCE AVEC LE PLAN V2 OFFICIEL


Phase

Titre

État

Phase 0

Backup & evidence baseline

PARTIEL — workspace sauvegardé par l'opérateur (méthode manuelle, hors openclaw backup create) ; config .openclaw non sauvegardée (décision opérateur, jugée défectueuse)

Phase 1

Reset / clean baseline

COMPLET — reset full réussi après contournement EBUSY, .openclaw et workspace supprimés, aucun résidu confirmé

Phase 2

Minimal runtime skeleton

PARTIEL — MAIN configuré et validé (agent addressable, Gateway up). LOCAL non créé. Critère de sortie officiel du plan ("MAIN and LOCAL are independently addressable") NON ATTEINT

Phase 3

Cloud providers

NON COMMENCÉ — clean slate maintenu volontairement, à faire en prochaine session via PowerShell

Phase 4

LM Studio / local

NON COMMENCÉ

Phase 5

Tool policy & delegation

NON COMMENCÉ

Phase 6

Telegram

NON COMMENCÉ — explicitement reporté après validation delegation

Phase 7

Governance files

NON COMMENCÉ

Phase 8

Total validation

NON COMMENCÉ

Phase 9

Real-world test matrix

NON COMMENCÉ

Phase 10

Closure & documentation

EN COURS — ce document

[ASSUMPTION] L'évaluation "PARTIEL / COMPLET / NON COMMENCÉ" est basée sur les critères de sortie explicites ("EXIT CRITERIA") du document OpenClaw_Reset_V2_Professional_Action_Plan fourni par l'opérateur. Elle n'a pas été validée par un test automatisé ou un tiers indépendant.


# 5. PLAN DÉTAILLÉ POUR LA SUITE



## 5.1 Avant Phase 3 — corrections en attente


Configurer commands.ownerAllowFrom (actuellement vide) — nécessaire avant tout channel, mais peut être fait dès maintenant sans dépendance.

Décider du sort de agents.defaults.memorySearch — désactiver, ou reconfigurer avec un provider gratuit supportant les embeddings (à vérifier : NVIDIA/Cerebras/Google proposent-ils un endpoint embeddings compatible ?).

[ACTION] Ces deux points ont été identifiés par openclaw doctor mais leur résolution a été explicitement reportée par l'opérateur à la session suivante.


## 5.2 Phase 3 — Providers cloud (prochaine étape principale)


Objectif du plan : injecter les credentials via le mécanisme secret/environnement supporté, jamais en clair dans un fichier Markdown.

Provider retenu pour amorcer : NVIDIA (déjà configuré comme modèle par défaut, sans clé).

Providers additionnels prévus par l'opérateur, dans l'ordre à définir : Google, Cerebras, Cloudflare AI Gateway, Mistral (déjà à 80% d'usage — à ménager, ne pas mettre en primary).

Méthode confirmée par l'opérateur : injection des clés via PowerShell, en clair au moment de la saisie (évite le champ masqué du wizard qui empêche le copier-coller).

[ACTION] La commande exacte (openclaw config set vs variable d'environnement vs openclaw secrets configure) n'a PAS encore été vérifiée dans la documentation officielle pour cette session. Elle devra être confirmée avant exécution, conformément à la règle de la session : ne jamais deviner une commande sans vérification documentaire préalable.

Règle du plan V2 à respecter : tester chaque provider individuellement avant de l'ajouter à la chaîne de fallback ; ne pas réordonner la chaîne sans validation explicite.

Point de vigilance technique : format de modèle NVIDIA contesté entre deux issues GitHub (préfixe simple vs double) — à observer au premier test réel, ajuster si erreur HTTP 404 model_not_found.


## 5.3 Phase 4 — LM Studio / LOCAL


Créer l'agent LOCAL distinct de MAIN (actuellement seul MAIN existe).

Démarrer LM Studio Server sur le port 1234, valider /v1/models.

Sélectionner 1 à 2 modèles seulement parmi les 13 disponibles en mémoire projet — ne pas précharger tous les modèles téléchargés.

Assigner explicitement le modèle sélectionné à LOCAL (pas d'héritage implicite du modèle de MAIN).

Tester une inférence réelle via OpenClaw, pas seulement un test API brut LM Studio.


## 5.4 Phase 5 — Tool policy & délégation


Restreindre les outils d'exécution de MAIN après vérification des noms d'outils effectifs de la version installée.

Donner à LOCAL uniquement les outils d'exécution minimum requis.

Configurer l'allowlist de sous-agents de MAIN pour permettre explicitement LOCAL.

Activer requireAgentId=true pour une sélection de cible déterministe.

Configurer MAIN → LOCAL via sessions_spawn.

Garder @local comme route directe vers LOCAL, séparée de la délégation MAIN→LOCAL.


## 5.5 Phase 6 — Telegram (WhatsApp après stabilisation de Telegram)


Pré-requis : command owner configuré (voir 5.1), tests agent/modèle/délégation validés localement d'abord.

Stocker le token Telegram via le mécanisme environnement/secret supporté.

Tester la route @local directe, puis le routing MAIN normal, puis l'exécution hybride MAIN→LOCAL depuis Telegram.

WhatsApp seulement après stabilité confirmée de Telegram.


## 5.6 Phases 7 à 10 — Gouvernance, validation totale, tests réels, clôture


Réécrire SOUL.md / AGENTS.md / TOOLS.md comme documentation du runtime vérifié, jamais comme mécanisme de sécurité.

Valider les 16 critères d'acceptation (AC01 à AC16) du plan V2 avant de considérer le rebuild terminé.

Exécuter la matrice de tests réels (@local direct, recherche, tâche hybride, tests négatifs MAIN/LOCAL, Telegram, redémarrage).

Produire le rapport de clôture V2 final avec hash de configuration, matrice agent/modèle/provider, et chaîne de fallback testée.


# 6. RAPPEL — RÈGLES DE PROCÉDURE POUR LA SUITE


AUDIT → CHANGE → VALIDATE → EVIDENCE → PASS/STOP à chaque phase.

Aucune commande n'est proposée sans vérification préalable dans la documentation officielle.

Aucun saut de phase — la Phase 3 ne débute pas avant que les points en attente de la section 5.1 soient tranchés par l'opérateur.

Toute correction automatique (ex. openclaw doctor) est présentée avant application, jamais appliquée silencieusement.

Un échec de phase bloque la progression ; on ne compense jamais une couche runtime défaillante en ajoutant plus de Markdown.

Fin du document — généré à partir du journal de conversation de cette session uniquement. Aucune information n'a été ajoutée au-delà de ce qui a été observé, dit, ou confirmé par l'opérateur pendant cette session.
