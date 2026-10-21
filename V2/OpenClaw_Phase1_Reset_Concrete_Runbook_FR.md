
# OPENCLAW V2 — PHASE 1RESET COMPLET + BASELINE PROPRE


Runbook concret • Windows 11 • PowerShell 7 • OpenClaw actuel • Français

VERSION=1.0 | DATE=2026-08-10 | ENVIRONNEMENT=C:\Users\Tech\.openclaw + D:\OpenClaw\workspace


# 1. OBJECTIF EXACT


Cette phase ne cherche pas encore à construire le système MAIN/LOCAL complet. Elle a un seul objectif : obtenir un OpenClaw propre, démarrable et vérifiable, puis poser uniquement le socle minimal nécessaire à la Phase 2. Aucune logique de routing, aucun fallback cloud, aucune intégration Telegram/WhatsApp, aucun gros ensemble de .md et aucun tool policy complexe ne doit être réintroduit ici.

PHASE 1 = BACKUP VERIFIED → RESET VERIFIED → CLEAN RUNTIME → MINIMAL BASELINE → VALIDATE → STOP


# 2. CE QUE LA DOCUMENTATION OFFICIELLE DIT


openclaw reset est la commande native pour réinitialiser la configuration et l'état local sans supprimer la CLI.

Le scope config supprime seulement la configuration.

Le scope config+creds+sessions supprime la configuration, les credentials/OAuth et les sessions des agents, et arrête d'abord le Gateway géré.

Le scope full supprime le state directory, la configuration, les credentials et les workspaces ; c'est donc destructif pour le workspace.

La documentation recommande explicitement une sauvegarde avant reset.

openclaw backup create --verify produit et vérifie une archive avec manifest ; --only-config permet une sauvegarde limitée au fichier de configuration.

openclaw config validate valide la configuration selon le schema actif.

openclaw config schema affiche le JSON Schema réellement utilisé par l'installation.

openclaw doctor sert au diagnostic et aux réparations supportées ; --fix applique les réparations recommandées.

Le Gateway local exige gateway.mode=local pour un démarrage normal ; onboard --mode local et setup peuvent écrire cette valeur.

La configuration est JSON5 et tous les champs sont optionnels ; OpenClaw possède donc des defaults sûrs.

Sources officielles utilisées :

Reset — https://docs.openclaw.ai/fr/cli/reset

Backup — https://docs.openclaw.ai/fr/cli/backup

Configuration — https://docs.openclaw.ai/fr/gateway/configuration

Configuration Reference — https://docs.openclaw.ai/fr/gateway/configuration-reference

Config CLI — https://docs.openclaw.ai/fr/cli/config

Gateway CLI — https://docs.openclaw.ai/fr/cli/gateway

Doctor — https://docs.openclaw.ai/gateway/doctor

Troubleshooting — https://docs.openclaw.ai/fr/gateway/troubleshooting

CLI Reference — https://docs.openclaw.ai/fr/cli


# 3. DÉCISION D'ARCHITECTURE POUR TON LAB


Le reset doit être assez profond pour supprimer les sessions/credentials qui peuvent conserver un ancien état, mais il ne faut pas détruire D:\OpenClaw\workspace avant d'avoir une archive vérifiée.

ACTIVE_CONFIG = C:\Users\Tech\.openclaw\openclaw.jsonSTATE_ROOT      = C:\Users\Tech\.openclawWORKSPACE       = D:\OpenClaw\workspaceBACKUP_ROOT     = D:\OpenClaw\backups\reset-v2

Choix recommandé pour ton reset :

openclaw reset --scope config+creds+sessions --yes --non-interactive

Ce scope est le meilleur point de départ pour la V2 : il supprime précisément la configuration, les credentials et les sessions, sans supprimer ton workspace D:\OpenClaw\workspace. Le scope full ne doit être utilisé que si l'objectif devient réellement de reconstruire aussi le workspace.

Source : documentation officielle Reset. citeturn1search5


# 4. ÉTAPE 0 — PRÉREQUIS ET AUDIT AVANT RESET


Ouvrir PowerShell 7.x en session utilisateur normale. Ne pas lancer le reset depuis une console administrateur sauf nécessité réelle.

Vérifier que la CLI OpenClaw répond.

Identifier la configuration active avec la commande native.

Vérifier l'état du Gateway.

Lister les processus OpenClaw/Node afin d'identifier un éventuel processus fantôme.

Créer le répertoire de backup hors de C:\Users\Tech\.openclaw et hors de D:\OpenClaw\workspace.

Ne supprimer aucun fichier manuellement à ce stade.

$ErrorActionPreference = 'Stop'openclaw --versionopenclaw config fileopenclaw gateway status --jsonGet-Process openclaw,node -ErrorAction SilentlyContinue |    Select-Object Id,ProcessName,Path,StartTime

GATE — La CLI répond, le chemin de config est connu, le Gateway est identifiable et aucun nettoyage manuel n'a encore été effectué.


# 5. ÉTAPE 1 — BACKUP NATIF OPENCLAW


Ne commence pas par zipper manuellement le dossier .openclaw. OpenClaw possède maintenant son propre mécanisme de backup, qui connaît les sessions, SQLite, credentials, configuration et workspace. Il doit être la sauvegarde primaire.

Créer le dossier D:\OpenClaw\backups\reset-v2.

Faire un dry-run pour vérifier ce qui sera sauvegardé.

Créer ensuite le backup avec --verify.

Conserver l'archive hors du state directory.

Vérifier explicitement l'archive après création.

Conserver également une copie séparée des fichiers Markdown de gouvernance si tu veux comparer/reconstruire leur contenu après le reset.

$BackupRoot = 'D:\OpenClaw\backups\reset-v2'New-Item -ItemType Directory -Path $BackupRoot -Force | Out-Nullopenclaw backup create `  --output $BackupRoot `  --dry-run `  --jsonopenclaw backup create `  --output $BackupRoot `  --verifyGet-ChildItem $BackupRoot |  Sort-Object LastWriteTime -Descending |  Select-Object -First 5 Name,Length,LastWriteTime

La documentation officielle précise que l'archive contient un manifest et que --verify effectue une validation d'intégrité. citeturn1search0

Si tu veux une sauvegarde minimale supplémentaire du JSON actif :

openclaw backup create `  --output $BackupRoot `  --only-config `  --verify

GATE — Au moins une archive .tar.gz existe dans D:\OpenClaw\backups\reset-v2 et a été créée avec --verify sans erreur.


# 6. ÉTAPE 2 — ARCHIVE HUMAINE DES .MD


Le backup OpenClaw est la sauvegarde runtime. Cette archive séparée est destinée à ton travail d'architecture et à la comparaison avant/après.

$Stamp = Get-Date -Format 'yyyyMMdd-HHmmss'$MdArchive = Join-Path $BackupRoot "governance-md-$Stamp.zip"$MdFiles = @(  'D:\OpenClaw\workspace\SOUL.md',  'D:\OpenClaw\workspace\AGENTS.md',  'D:\OpenClaw\workspace\TOOLS.md',  'D:\OpenClaw\workspace\USER.md',  'D:\OpenClaw\workspace\MEMORY.md',  'D:\OpenClaw\workspace\IDENTITY.md',  'D:\OpenClaw\workspace\HEARTBEAT.md') | Where-Object { Test-Path $_ }if ($MdFiles.Count -gt 0) {    Compress-Archive -Path $MdFiles -DestinationPath $MdArchive -CompressionLevel Optimal -Force}Get-FileHash $MdArchive -Algorithm SHA256

Les fichiers absents sont ignorés volontairement : ne crée pas artificiellement des fichiers simplement parce qu'ils figurent dans une ancienne liste.


# 7. ÉTAPE 3 — VÉRIFIER LE BACKUP AVANT DE DÉTRUIRE


$Archive = Get-ChildItem $BackupRoot -Filter '*.tar.gz' |    Sort-Object LastWriteTime -Descending |    Select-Object -First 1if (-not $Archive) {    throw "Aucune archive OpenClaw .tar.gz trouvée."}openclaw backup verify $Archive.FullNameGet-FileHash $Archive.FullName -Algorithm SHA256

GATE — openclaw backup verify termine sans erreur. Si la vérification échoue : STOP. Aucun reset.


# 8. ÉTAPE 4 — DRY-RUN DU RESET


Avant toute suppression, demander à OpenClaw ce qu'il va supprimer.

openclaw reset --scope config+creds+sessions --dry-run

Comparer mentalement la portée avec l'objectif : configuration + credentials + sessions. Le workspace D:\OpenClaw\workspace ne doit pas faire partie de ce reset.

GATE — Le dry-run correspond exactement à la portée décidée.


# 9. ÉTAPE 5 — RESET RÉEL


Exécuter maintenant la commande native. Ne pas supprimer manuellement C:\Users\Tech\.openclaw.

openclaw reset --scope config+creds+sessions --yes --non-interactive

Selon la documentation, ce scope arrête d'abord le Gateway géré puis supprime configuration, credentials/OAuth et sessions. citeturn1search5

GATE — La commande se termine sans erreur et OpenClaw indique la prochaine étape d'onboarding.


# 10. ÉTAPE 6 — PROUVER QUE LE RESET A RÉELLEMENT EU LIEU


Relancer openclaw config file.

Vérifier l'existence et le contenu de la configuration active.

Vérifier l'état du Gateway.

Vérifier qu'aucune ancienne session agent ne reste active.

Vérifier qu'aucune ancienne configuration de providers/models n'est réapparue.

Vérifier que le workspace D:\OpenClaw\workspace existe toujours puisque le scope choisi ne devait pas le supprimer.

$ErrorActionPreference = 'Continue'openclaw config fileopenclaw gateway status --jsonopenclaw agents list 2>$nullopenclaw sessions list 2>$nullWrite-Host "`nWorkspace:"Test-Path 'D:\OpenClaw\workspace'Write-Host "`nState root:"Get-ChildItem 'C:\Users\Tech\.openclaw' -Force -ErrorAction SilentlyContinue |    Select-Object Name,Mode,Length,LastWriteTime

GATE — Ancien routing/provider/session state absent ou explicitement expliqué. Workspace conservé. Aucun ghost Gateway.


# 11. ÉTAPE 7 — NE PAS RESTAURER L'ANCIEN OPENCLAW.JSON


C'est le point critique. Le backup sert au rollback et à l'analyse, pas à recopier immédiatement l'ancien fichier. Ton problème historique est précisément lié à une accumulation de configuration, routing, agents et providers. La V2 doit reconstruire le runtime progressivement.

Ne copie pas l'ancien openclaw.json dans le state actif.

Ne restaure pas les anciens providers.

Ne restaure pas les anciens agents.

Ne restaure pas les anciennes sessions.

Ne restaure pas les anciennes policies tools.

Ne réinjecte pas les anciens .md avant d'avoir validé le runtime minimal.


# 12. ÉTAPE 8 — CONSTRUIRE LE MINIMUM SUPPORTED


La documentation actuelle indique que tous les champs sont optionnels et qu'OpenClaw possède des defaults sûrs. Le minimum fonctionnel doit donc rester petit. Pour un Gateway local, il faut cependant que gateway.mode soit défini à local pour un démarrage normal.

La documentation recommande également d'utiliser le schema actif comme source de vérité :

openclaw config schema

et pour les modifications ciblées :

openclaw config set <path> <value>

La documentation recommande de valider après chaque modification :

openclaw config validate

Sources : configuration et CLI config. citeturn0search0turn0search12


# 13. ÉTAPE 9 — BASELINE MINIMALE RECOMMANDÉE


Pour cette phase, ne crée pas encore MAIN + LOCAL, providers, Telegram ou policies complexes. Pose seulement le Gateway local et le workspace.

{  gateway: {    mode: "local",  },  agents: {    defaults: {      workspace: "D:\\OpenClaw\\workspace",    },  },}

Important : cette structure est volontairement minimale. Avant écriture, vérifier le schema réellement exposé par ta version avec `openclaw config schema`. La documentation officielle précise que le schema live est l'autorité pour la validation.

GATE — La configuration est valide et le Gateway accepte la baseline sans provider ni agent spécialisé.


# 14. ÉTAPE 10 — ÉCRITURE SÉCURISÉE DE LA BASELINE


Préférer les commandes natives pour les petits changements plutôt qu'un remplacement aveugle du JSON.

openclaw config set gateway.mode localopenclaw config set agents.defaults.workspace 'D:\OpenClaw\workspace'openclaw config validate

Si une commande `config set` n'accepte pas la valeur ou le chemin dans ta version, ne force pas le JSON : consulte le schema live et adapte la commande à la structure réellement exposée.


# 15. ÉTAPE 11 — DOCTOR APRÈS VALIDATION


openclaw config validateopenclaw doctor

N'utiliser `openclaw doctor --fix` que si Doctor identifie une réparation nécessaire. La documentation indique que `--fix` applique les réparations recommandées ; il ne faut pas l'utiliser comme substitut à une validation systématique.

openclaw doctor --fix

GATE — config validate = PASS et doctor = PASS ou uniquement warnings compris/documentés.


# 16. ÉTAPE 12 — DÉMARRAGE DU GATEWAY


openclaw gateway status --jsonopenclaw gateway startopenclaw gateway status --json

Si le Gateway est déjà installé et géré, utiliser `openclaw gateway restart` plutôt que de chaîner stop/start pour simuler un restart. La CLI officielle documente les commandes de lifecycle du Gateway. citeturn0search3

GATE — Gateway running + mode local + port/bind attendus + aucun Invalid config.


# 17. ÉTAPE 13 — VALIDATION FONCTIONNELLE MINIMALE


ID

TEST

COMMANDE / ACTION

PASS

P1-01

CLI

openclaw --version

☐

P1-02

Config path

openclaw config file

☐

P1-03

Schema

openclaw config schema

☐

P1-04

Config validation

openclaw config validate

☐

P1-05

Doctor

openclaw doctor

☐

P1-06

Gateway

openclaw gateway status --json

☐

P1-07

Workspace

Test-Path D:\OpenClaw\workspace

☐

P1-08

No old provider chain

inspect live config/schema output

☐

P1-09

No old sessions

openclaw sessions list

☐

P1-10

Restart persistence

restart Gateway → validate → status

☐


# 18. CE QUI EST EXPRESSÉMENT HORS PHASE 1


Pas de cloud provider.

Pas de fallback chain.

Pas de LM Studio model binding.

Pas de MAIN/LOCAL routing.

Pas de sessions_spawn.

Pas de @local.

Pas de Telegram.

Pas de WhatsApp.

Pas de tool allow/deny avancé.

Pas de restauration massive des .md.

Pas d'optimisation performance.

Pas de preload de modèles LM Studio.


# 19. GATE FINAL — PHASE 1 → PHASE 2


PHASE_1_PASS =BACKUP_VERIFIEDAND RESET_SCOPE_VERIFIEDAND RESET_COMPLETEDAND NO_UNEXPECTED_RESIDUEAND WORKSPACE_PRESERVEDAND CONFIG_SCHEMA_VALIDAND CONFIG_VALIDAND DOCTOR_OKAND GATEWAY_RUNNINGAND RESTART_PERSISTSAND NO_OLD_ROUTING_RESTORED

Si une condition est FALSE : STOP. On corrige Phase 1 avant de toucher à Phase 2.


# 20. SOURCES OFFICIELLES — LIENS DE RÉFÉRENCE


Reset: https://docs.openclaw.ai/fr/cli/reset

Backup: https://docs.openclaw.ai/fr/cli/backup

Configuration: https://docs.openclaw.ai/fr/gateway/configuration

Configuration Reference: https://docs.openclaw.ai/fr/gateway/configuration-reference

Config CLI: https://docs.openclaw.ai/fr/cli/config

Gateway CLI: https://docs.openclaw.ai/fr/cli/gateway

Doctor: https://docs.openclaw.ai/gateway/doctor

Troubleshooting: https://docs.openclaw.ai/fr/gateway/troubleshooting

CLI Reference: https://docs.openclaw.ai/fr/cli

Documentation vérifiée le 2026-08-10. Les commandes et options doivent toujours être confirmées contre la CLI installée avec `openclaw --help` / la documentation correspondant à la version réellement installée.


# 21. NOTE DE SÉCURITÉ


Le backup OpenClaw peut contenir credentials, profils d'authentification, identifiants de canaux/fournisseurs et état de sessions. Le répertoire D:\OpenClaw\backups\reset-v2 doit donc être protégé comme un emplacement contenant des secrets. Ne partage jamais l'archive brute dans un chat ou un dépôt Git.
