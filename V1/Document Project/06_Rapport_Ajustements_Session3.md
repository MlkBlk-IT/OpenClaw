RAPPORT DE DELTA — SESSION 3

OpenClaw 2026.7.1-2 — Changements depuis la configuration v1.0

Utilisateur : Malik  |  Date : 4 août 2026  |  Version : 3.0-delta

📋  Ce document couvre UNIQUEMENT les changements intervenus après la rédaction du Rapport de Configuration v1.0.

Pour l'architecture complète, se référer au document : Rapport_Configuration_OpenClaw_Malik.docx (v1.0).


# 1. Changements de configuration



## 1.1 Modèles — Mise à jour


Les deux changements suivants ont été effectués via CLI après le v1.0 :

Paramètre

Avant (v1.0)

Après (Session 3)

Statut

primary model

groq/compound-mini

google/gemini-3.5-flash

✅

utilityModel

lmstudio/llama-3.2-3b-instruct

lmstudio/qwen3.5-4b-instruct-revised

✅

gateway.bind

loopback (127.0.0.1)

lan (accès réseau local)

✅

⚠️  Raison du changement de primary : groq/compound-mini rejetait le system prompt complet (trop gros).

google/gemini-3.5-flash dispose d'un contexte de 1M tokens — solution définitive.


## 1.2 Tâches planifiées Windows — Ajouts


Deux tâches planifiées ont été créées dans le Planificateur de tâches Windows :

Nom de la tâche

Déclencheur

Action

LMStudio_Autostart

Au logon de l'utilisateur Tech

Lance LM Studio (interface graphique)

LMStudio_LoadOrchestrator

20 secondes après logon

Charge qwen3.5-4b-instruct-revised via lms.exe (orchestrateur JIT permanent, ~3GB RAM)


## 1.3 PATH système — Ajout lms.exe


lms.exe (CLI LM Studio) a été ajouté au PATH au niveau Machine :

Chemin ajouté : C:\Users\Tech\AppData\Local\Programs\LM Studio\resources\app\.webpack

Portée         : Machine (tous les utilisateurs, toutes les sessions PowerShell)

Usage          : lms load <model-id>  |  lms status  |  lms list


# 2. Nœuds pairés — Ajout



## 2.1 Redmi Note 14


Le smartphone Redmi Note 14 a été ajouté comme nœud OpenClaw lors de la Session 3 :

Nom du nœud

Redmi Note 14

Type

Mobile (Android)

Scope accordé

node (lecture de base)

operator.admin

NON — aucune commande CLI nodes grant disponible

App mobile

Installée — affiche 'no configured model providers' (bug d'affichage)

Fonctionnel

Telegram ✅  |  WhatsApp ✅  |  App native ⚠️ (bug affichage)


# 3. SOUL.md — Routing intelligent reécrit


Le fichier SOUL.md a été reécrit pour inclure un routing explicite en 4 étapes. Ce routing est interprété par le LLM lors de chaque requête (pas un router natif OpenClaw — cf. limitation connue section 8.4 du v1.0).


## 3.1 Logique de routing en 4 étapes


▸ Étape 1 — Classifier l'intention

Analyser la requête : web/info ? code ? local ? données privées ?

▸ Étape 2 — Choisir le spécialiste local

Si local requis : sélectionner le modèle LM Studio adapté (code → Qwen2.5-Coder-14B, reasoning → DeepSeek-R1, rapide → Qwen3.5-4B)

▸ Étape 3 — Flux cloud + local

Cloud pour la recherche/analyse, local pour l'exécution et les données sensibles

▸ Étape 4 — Gestion RAM

Vérifier la RAM disponible avant de charger un modèle 14B. Si < 12GB libre → utiliser un modèle 4B ou déléguer au cloud

⚠️  Rappel : ce routing est une directive comportementale dans le system prompt, pas un mécanisme natif.

Le routing effectif reste contrôlé par primary/fallbacks CLI + commande /model manuelle.

Voir section 8.4 du Rapport v1.0 pour le contexte complet.


# 4. Infrastructure — Changements



## 4.1 Veille AC désactivée


• La veille secteur a été désactivée sur le Dell Latitude 5420.

• Résultat : laptop joignable 24/7 sur secteur — gateway OpenClaw toujours actif.

• Commande appliquée : powercfg /change standby-timeout-ac 0


## 4.2 Clés API — État


Provider

Statut

Note

GROQ

✅ Configuré (niveau Machine)

Primary provider actif

GOOGLE

✅ Configuré (niveau Machine)

gemini-3.5-flash primary

MISTRAL

✅ Configuré (niveau Machine)

Fallbacks #5-#6

OPENROUTER

✅ Configuré (niveau Machine)

Fallbacks #7-#8

GitHub Models

❌ Manquant

À ajouter — Session 4

NVIDIA NIM

❌ Manquant

À ajouter — Session 4

SambaNova

❌ Manquant

À ajouter — Session 4

DeepSeek

❌ Manquant

À ajouter — Session 4

HuggingFace

❌ Manquant

À ajouter — Session 4

Cloudflare

❌ Manquant

À ajouter — Session 4


# 5. Problèmes en suspens



## 5.1 App mobile — no configured model providers


Symptôme   : L'app native OpenClaw sur le Redmi Note 14 affiche 'no configured model providers'.

Canaux OK  : Telegram et WhatsApp fonctionnent normalement depuis le mobile.

Diagnostic : Bug d'affichage probable dans l'app mobile — les providers sont bien configurés côté gateway.

Hypothèse  : L'app mobile requiert peut-être operator.admin pour afficher les providers.

Prochaine étape : Consulter docs.openclaw.ai/cli/nodes pour la commande nodes grant.


## 5.2 operator.admin — Node mobile


Aucune commande CLI nodes grant n'a été trouvée dans la version 2026.7.1-2 pour accorder le scope operator.admin à un nœud mobile.

Piste : vérifier si la commande existe dans une version plus récente ou via le dashboard web.


# 6. Prochaines étapes — Session 4


#

Action

Commande / Note

Priorité

1

Tester depuis Telegram : créer un fichier Word dans D:\Test

Envoyer : 'Crée un doc Word D:\Test\test.docx avec une phrase'

🔴 Haute

2

Vérifier le routing 4 étapes de SOUL.md

Observer dans les logs quel modèle exécute la tâche

🔴 Haute

3

Résoudre operator.admin mobile

docs.openclaw.ai/cli/nodes → chercher 'nodes grant'

🟡 Moyenne

4

Ajouter 6 clés API manquantes

GitHub Models, NVIDIA NIM, SambaNova, DeepSeek, HuggingFace, Cloudflare

🟡 Moyenne

5

Mettre à jour Rapport v1.0 → v2.0

Intégrer tous les changements de ce delta dans le rapport complet

🟢 Basse

6

Activer Active Memory plugin

openclaw plugins enable active-memory

🟢 Basse

Rapport généré automatiquement • OpenClaw 2026.7.1-2 • Dell Latitude 5420 • Malik • Montréal
