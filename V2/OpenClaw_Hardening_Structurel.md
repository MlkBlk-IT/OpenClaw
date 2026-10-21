
# HARDENING STRUCTUREL OPENCLAW


Au-delà des guardrails textuels — Architecture de défense en profondeur

Date : 8 août 2026 | Version : 1.0 | Classification : Interne


# Table des matières


1. Contexte et diagnostic

2. Couches de défense structurelle

2.1 Docker Sandbox Isolation

2.2 Tool Allowlist / Denylist

2.3 Exec Security Mode

2.4 Consent Mode

2.5 DM Policy & Session Isolation

2.6 Gateway Hardening

2.7 Security Audit

2.8 File Permissions

2.9 Policy-as-Code

2.10 Agentic Zero-Trust

3. Plan d'action priorisé

4. Questions de validation

5. Annexes — Configurations complètes


# 1. Contexte et diagnostic


État actuel du déploiement :

OpenClaw 2026.7.1-2 orchestrant 6 agents locaux + providers cloud

LM Studio 0.4.20 servant 18 modèles locaux via API OpenAI-compatible (port 1234)

Gateway exposé sur ws://0.0.0.0:18789 (mode LAN pour accès mobile)

Channels : Telegram + WhatsApp (bridge Baileys)

HARD-GUARDRAIL répété dans 4 fichiers + 8 fichiers de gouvernance (text-based)

Problème fondamental : Les guardrails textuels reposent sur l'obéissance du LLM. Un prompt injection, un jailbreak ou un model swap bypassent l'ensemble de ces protections. Il faut des contrôles structuraux, non contournables par le LLM.

⚠️  ALERTE : CrowdStrike a décrit le modèle de capacité d'OpenClaw comme "quasi identique à un RAT (Remote Access Trojan) au niveau technique".


# 2. Couches de défense structurelle



## 2.1 Docker Sandbox Isolation


Description : OpenClaw supporte le sandboxing Docker natif. C'est la barrière la plus forte entre l'agent et le host. Chaque session s'exécute dans un container isolé, détruit après usage.

ℹ️  mode: "all" = tout agent, tout le temps, sandboxé | scope: "session" = un container par session | workspaceAccess: "ro" = read-only

Configuration recommandée :

{  agents: {    defaults: {      sandbox: {        mode: "all",           // off | non-main | all        scope: "session",      // shared | session        workspaceAccess: "ro"  // ro | rw      }    }  }}

Impact : Toute exécution d'agent est confinée. Même un agent compromis ne peut pas modifier le filesystem host ni persister entre sessions.


## 2.2 Tool Allowlist / Denylist


Description : Au lieu de dire au LLM "ne fais pas ça", on bloque structuralement l'accès aux outils dangereux. La politique de base est restrictive ; les exceptions sont justifiées par agent.

ℹ️  deny au niveau defaults = politique de base restrictive | allow au niveau agent spécifique = exception justifiée

Configuration recommandée :

{  agents: {    defaults: {      tools: {        allow: ["read", "sessions_list", "search"],        deny: ["exec", "write", "browser", "edit", "apply_patch"]      }    },    list: [      {        id: "local-coding",        tools: {          allow: ["read", "write", "edit", "exec"],          deny: ["browser"]        }      }    ]  }}


## 2.3 Exec Security Mode


Description : Trois modes : deny | allowlist | full. Jamais full en production. Les agents non-codage n'ont pas de shell ; l'agent codage a une allowlist restreinte.

Matrice de configuration :

Agent

Mode

Commandes autorisées

local-fast

deny

Aucune

local-normal

deny

Aucune

local-vision

deny

Aucune

local-coding

allowlist

ls, cat, python3, npm, git

local-heavy

allowlist

ls, cat, python3 (restreint)

{  agents: {    list: [      {        id: "local-coding",        exec: {          security: "allowlist",          allowed_commands: ["ls", "cat", "python3", "npm", "git"]        }      }    ]  }}


## 2.4 Consent Mode


Description : Forcer une approbation explicite avant toute action irréversible. L'utilisateur doit confirmer manuellement chaque opération destructrice.

Configuration recommandée :

{  consent: {    require_for: [      "file_delete",      "package_install",      "service_modify",      "network_request_external"    ]  }}


## 2.5 DM Policy & Session Isolation


Description : Empêcher le context bleed entre utilisateurs et channels. Chaque conversation est isolée. L'approbation explicite (pairing) est requise pour les DM.

ℹ️  dmScope: "per-channel-peer" = isolation stricte par utilisateur | dmPolicy: "pairing" = approbation explicite requise

Configuration recommandée :

{  session: {    dmScope: "per-channel-peer"  },  channels: {    telegram: {      dmPolicy: "pairing",      groups: {        "*": { requireMention: true }      }    },    whatsapp: {      dmPolicy: "pairing",      groups: {        "*": { requireMention: true }      }    }  }}


## 2.6 Gateway Hardening


Description : Le Gateway est exposé sur ws://0.0.0.0:18789. C'est un vecteur d'attaque majeur. Il faut le binder sur loopback, générer un token fort, et désactiver la découverte réseau.

🚨 CRITIQUE : Si le Gateway est accessible depuis Internet, la priorité est absolue.

Commandes et configuration :

# 1. Générer un token fortopenclaw doctor --generate-gateway-token# 2. Binder sur loopback + VPN (Tailscale)openclaw config set gateway.bind localhost# Configuration JSON5 :{  gateway: {    bind: "loopback",      // ou "tailnet" avec Tailscale    port: 18789,    auth: {      token: "TOKEN_FORT_GENERE"    }  },  discovery: {    mdns: { mode: "off" }  // Désactiver la découverte réseau  }}


## 2.7 Security Audit


Description : OpenClaw dispose d'un audit natif de 78 checks couvrant gateway, filesystem, channels, sandbox, et plugins.

Commandes :

# Audit completopenclaw security audit --deep# Avec auto-fix des permissionsopenclaw security audit --deep --fix# Export JSON pour CI/monitoringopenclaw security audit --json > audit-$(date +%Y-%m-%d).json

Catégories couvertes :

gateway.* — auth, binding, token strength

fs.* — permissions fichiers

channels.* — allowlists Telegram/Discord/Slack

sandbox.* — Docker/AppArmor config

plugins.* — sécurité code extensions


## 2.8 File Permissions


Description : Permissions strictes sur le dossier OpenClaw pour empêcher la lecture des credentials et de la configuration par d'autres utilisateurs du système.

Commandes :

# Permissions strictes sur le dossier OpenClawchmod 700 ~/.openclawchmod 600 ~/.openclaw/openclaw.jsonchmod 600 ~/.openclaw/credentials/**/*


## 2.9 Policy-as-Code


Description : Fichiers de policy pour verrouiller la configuration. Les règles empêchent l'activation d'outils élevés, l'exposition du Gateway en remote, et la capture de contenu par la télémétrie.

Fichier policy.yaml :

rules:  - id: no-elevated-tools    level: deny    tools:      elevated: true  - id: no-remote-gateway    level: deny    gateway:      mode: remote  - id: require-group-allowlist    level: deny    channels:      groupPolicy: open  - id: no-telemetry-content    level: deny    diagnostics:      otel:        captureContent: true

Commandes :

openclaw policy checkopenclaw policy compare  # Comparer contre baseline


## 2.10 Agentic Zero-Trust


Description : Appliquer les 5 piliers Zero-Trust aux agents LLM. Chaque pilier a une implémentation concrète dans OpenClaw.

Pilier

Principe

Implémentation OpenClaw

Identity

Vérifier chaque identité

Pairing obligatoire, token auth, dmScope: per-channel-peer

Device

Vérifier chaque device

Sandbox Docker, filesystem denylist

Network

Segmenter et contrôler

gateway.bind: loopback, mdns: off, firewall

Data

Protéger les données

workspaceAccess: ro, PII detection, log redaction

Application

Sécuriser les workloads

Tool allowlist, exec allowlist, consent mode


# 3. Plan d'action priorisé


Le tableau suivant présente les actions par ordre de priorité, avec leur impact et leur complexité.

Priorité

Action

Impact

Complexité

P0

Activer sandbox: { mode: "all", scope: "session" }

Contient tout dommage d'exécution

Moyenne (nécessite Docker)

P0

Passer gateway.bind de 0.0.0.0 à loopback

Ferme le vecteur réseau externe

Faible

P0

Générer token auth fort + mdns: off

Authentification + discrétion

Faible

P1

Configurer tools.deny par défaut + allow par agent

Moindre privilège outils

Moyenne

P1

exec.security: "allowlist" pour agents codage

Contrainte shell structurale

Faible

P1

consent.require_for actions destructrices

Approbation humaine obligatoire

Faible

P2

session.dmScope: "per-channel-peer"

Isolation contexte multi-utilisateur

Faible

P2

openclaw security audit --deep --fix

Baseline + auto-correction

Faible

P2

Fichier policy.yaml + policy check

Gouvernance as-code

Moyenne

P3

chmod 600/700 sur ~/.openclaw/**

Hardening filesystem

Faible

P3

Audit hebdomadaire automatisé (cron + --json)

Détection drift continue

Moyenne


# 4. Questions de validation


Avant de procéder à l'implémentation, les réponses suivantes sont nécessaires :

Ton Gateway en 0.0.0.0:18789 est-il accessible depuis l'extérieur (Internet) ou seulement LAN ? Si Internet → urgence critique.

Quels outils chaque agent doit-il vraiment utiliser ? (permet de générer la config tools.allow/deny exacte par agent)

Veux-tu le fichier policy.yaml complet et le delta de config JSON5 à appliquer ?

As-tu Docker installé/configuré pour le sandboxing ? (Nécessite docker.sock accessible au Gateway)


# 5. Annexes — Configurations complètes



## 5.1 Configuration sandbox complète


{  agents: {    defaults: {      sandbox: {        mode: "all",        scope: "session",        workspaceAccess: "ro",        image: "openclaw/sandbox:latest",        resources: {          memory: "2g",          cpus: "2.0"        }      }    }  }}


## 5.2 Configuration gateway complète


{  gateway: {    bind: "loopback",    port: 18789,    auth: {      type: "token",      token: "${GATEWAY_TOKEN}",  // Référence variable d'env      tokenFile: "~/.openclaw/.gateway_token"    },    tls: {      enabled: true,      certFile: "~/.openclaw/certs/gateway.crt",      keyFile: "~/.openclaw/certs/gateway.key"    }  },  discovery: {    mdns: { mode: "off" },    broadcast: { mode: "off" }  }}


## 5.3 Configuration agents complète


{  agents: {    defaults: {      model: {        primary: "lmstudio/llama-3.2-3b-instruct",        fallbacks: ["lmstudio/qwen2.5-7b-instruct"]      },      thinkingDefault: "medium",      tools: {        allow: ["read", "sessions_list", "search"],        deny: ["exec", "write", "browser", "edit", "apply_patch"]      },      sandbox: {        mode: "all",        scope: "session",        workspaceAccess: "ro"      }    },    list: [      {        id: "local-fast",        model: { primary: "lmstudio/llama-3.2-3b-instruct" },        thinkingDefault: "low",        exec: { security: "deny" }      },      {        id: "local-coding",        model: { primary: "lmstudio/qwen2.5-coder-7b-instruct" },        thinkingDefault: "medium",        tools: {          allow: ["read", "write", "edit", "exec"],          deny: ["browser"]        },        exec: {          security: "allowlist",          allowed_commands: ["ls", "cat", "python3", "npm", "git", "pip"]        }      },      {        id: "local-normal",        model: { primary: "lmstudio/llama-3.2-3b-instruct" },        thinkingDefault: "medium",        exec: { security: "deny" }      },      {        id: "local-vision",        model: { primary: "lmstudio/llava-phi-3-vision" },        thinkingDefault: "medium",        exec: { security: "deny" }      },      {        id: "local-heavy",        model: { primary: "lmstudio/ibm-granite-3.2-8b-instruct" },        thinkingDefault: "high",        tools: {          allow: ["read", "search", "exec"],          deny: ["write", "edit", "browser"]        },        exec: {          security: "allowlist",          allowed_commands: ["ls", "cat", "python3"]        }      }    ]  }}


## 5.4 Script d'audit automatisé (cron)


#!/bin/bash# /etc/cron.weekly/openclaw-audit# Audit hebdomadaire OpenClaw avec alerte sur driftAUDIT_DIR="/var/log/openclaw/audits"mkdir -p "$AUDIT_DIR"DATE=$(date +%Y-%m-%d)REPORT="$AUDIT_DIR/audit-$DATE.json"# Run deep audit with JSON outputopenclaw security audit --deep --json > "$REPORT" 2>/dev/null# Check for critical findingsCRITICAL=$(jq '.findings[] | select(.severity == "critical") | .id' "$REPORT" 2>/dev/null | wc -l)if [ "$CRITICAL" -gt 0 ]; then  echo "[ALERT] $CRITICAL critical findings detected in OpenClaw audit ($DATE)" \    | mail -s "OpenClaw Security Alert" admin@lab.localfi# Rotate logs (keep 30 days)find "$AUDIT_DIR" -name "audit-*.json" -mtime +30 -delete


## 5.5 Checklist de validation post-implémentation


☐ Sandbox activé : openclaw config get agents.defaults.sandbox.mode retourne "all"

☐ Gateway bindé sur loopback : netstat -tlnp | grep 18789 montre 127.0.0.1

☐ Token auth configuré : ~/.openclaw/.gateway_token existe et a les permissions 600

☐ mDNS désactivé : openclaw config get discovery.mdns.mode retourne "off"

☐ Tools denylist active : openclaw config get agents.defaults.tools.deny contient exec/write/browser/edit

☐ Exec allowlist pour local-coding : openclaw config get agents.list[1].exec.security retourne "allowlist"

☐ Consent mode activé : openclaw config get consent.require_for contient file_delete

☐ DM isolation : openclaw config get session.dmScope retourne "per-channel-peer"

☐ Audit passé : openclaw security audit --deep retourne 0 findings critical

☐ Policy check passé : openclaw policy check retourne "all policies satisfied"

☐ Permissions fichiers : ls -la ~/.openclaw/openclaw.json montre -rw-------

☐ Cron audit actif : crontab -l contient le script hebdomadaire

— Document généré le 8 août 2026 —
