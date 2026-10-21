
# Rapport de diagnostic OpenClaw


Genere le 2026-08-04 23:36:13


# 1. Vue d'ensemble


Element

Valeur

Statut

Version OpenClaw

OpenClaw 2026.7.1-2 (0790d9f)

OK

Fichier config trouve

true

OK

Config parsee sans erreur

true

OK

Chemin config

C:\Users\Tech\.openclaw\openclaw.json

Workspace

D:\OpenClaw\workspace


# 2. Gateway


Element

Valeur

Statut

Gateway en cours d'execution

true

OK

Sonde de connectivite

true

OK

Mode de binding

lan

ATTENTION

Port

18789

Mode d'authentification

token

OK

Tailscale

off

Detail technique (openclaw gateway status):

Service: Scheduled Task (registered)

File logs: ~\AppData\Local\Temp\openclaw\openclaw-2026-08-04.log

Command: C:\Program Files\nodejs\node.exe C:\Users\Tech\AppData\Roaming\npm\node_modules\openclaw\dist\index.js gateway --port 18789

Service file: ~\.openclaw\gateway.cmd

Service env: OPENCLAW_GATEWAY_PORT=18789

Config (cli): ~\.openclaw\openclaw.json

Config (service): ~\.openclaw\openclaw.json

Gateway: bind=lan (0.0.0.0), port=18789 (service args)

Probe target: ws://127.0.0.1:18789

Dashboard: http://10.0.0.28:18789/

Probe note: bind=lan listens on 0.0.0.0 (all interfaces); probing via 127.0.0.1.

CLI version: 2026.7.1-2 (~\AppData\Roaming\npm\node_modules\openclaw\openclaw.mjs)

Gateway version: 2026.7.1-2

Runtime: running (pid 15672, state Ready, last run 0, last run time 2026-08-04 11:32:02 PM, Gateway process detected for gateway port 18789.)

Connectivity probe: ok

Capability: connected-no-operator-scope

Listening: 0.0.0.0:18789

Troubles: run openclaw status

Troubleshooting: https://docs.openclaw.ai/troubleshooting


# 3. Modele et memoire


Element

Valeur

Statut

Modele principal

mistral/mistral-large-latest

Nombre de modeles de secours

7

Fournisseur embeddings memoire

openai

OK

Modele embeddings

text-embedding-nomic-embed-text-v1.5

Fichiers indexes

5/6

OK

Purge de contexte (contextPruning)

cache-ttl

OK

Vidage memoire auto (memoryFlush)

true

OK

Seuil de vidage (tokens)

40000

Detail technique (openclaw memory status):

Memory Search (main)

Provider: openai (requested: openai)

Model: text-embedding-nomic-embed-text-v1.5

Sources: memory

Indexed: 5/6 files ┬╖ 87 chunks

Dirty: no

Store: ~\.openclaw\agents\main\agent\openclaw-agent.sqlite

Workspace: D:\OpenClaw\workspace

Dreaming: off

By source:

memory ┬╖ 5/6 files ┬╖ 87 chunks

Vector store: unknown

Vector dims: 768

FTS: ready

Embedding cache: enabled (20 entries)

Batch: disabled (failures 0/2)

Recall store: 0 entries ┬╖ 0 promoted ┬╖ 0 concept-tagged ┬╖ 0 spaced

Recall path: plugin-state:memory-core/short-term-recall/3a0980715b0ece41988f3ffe46520f3d073e3aff12fb324551dbc9a88b024a27

Dreaming artifacts: diary absent ┬╖ 0 corpus files ┬╖ ingestion state absent

Dream corpus: D:\OpenClaw\workspace\memory\.dreams\session-corpus

Dream ingestion: D:\OpenClaw\workspace\memory\.dreams\session-ingestion.json


# 4. LM Studio (embeddings locaux)


Element

Valeur

Statut

LM Studio accessible (localhost:1234)

true

OK

Code de reponse HTTP

200

OK

Modeles disponibles dans LM Studio:Modeles disponibles dans LM Studio:

phi-4-reasoning-plus, gemma4-26b-a4b-instruct-pure, huihui-granite-4.1-3b-abliterated, qwen3-vl-8b-instruct-abliterated, deepseek-r1-distill-qwen-14b, mistral-7b-instruct-v0.3, gemma-4-e4b-it, text-embedding-nomic-embed-text-v1.5-embedding, gemma-4-e4b-it-obliterated, qwen2.5-coder-7b-instruct-ghidra-v2, qwen2.5-7b-instruct, meta-llama-3.1-8b-instruct, meta-llama-3.1-8b-instruct-abliterated, qwen3.5-4b-instruct-revised, llama-3.2-3b-instruct, qwen3.5-4b-kimi-reasoning-luna-lora, text-embedding-nomic-embed-text-v1.5


# 5. Channels (WhatsApp / Telegram)


Element

Valeur

Statut

Telegram

ON

OK

WhatsApp

ON

OK


# 6. Appareils appaires


Paired (2)

+------------------------------+------------+------------------------------------------------+------------+------------+

| Device                       | Roles      | Scopes                                         | Tokens     | IP         |

+------------------------------+------------+------------------------------------------------+------------+------------+

| Redmi Note 14                | node,      | operator.approvals, operator.read, operator.   | node,      | 10.0.0.226 |

|                              | operator   | talk.secrets, operator.write                   | operator   |            |

| 1d3dc994f59af7f2aedad763f9c6 | operator   | operator.admin, operator.read, operator.       | operator   |            |

| 305e965147a909720d0efce40b83 |            | write, operator.approvals, operator.pairing    |            |            |

| 0808c1c5                     |            |                                                |            |            |

+------------------------------+------------+------------------------------------------------+------------+------------+


# 7. Securite


Element

Valeur

Statut

Secrets en clair detectes

true

ATTENTION

Gateway expose sur le LAN

true

ATTENTION

Note: ces deux points sont actuellement acceptes tels quels (machine et reseau prives, usage personnel uniquement).


# 8. Resultat openclaw doctor


Des avertissements ont ete detectes (voir ci-dessous).
