# OpenClaw — Architecture Agentique IA Hybride

> De la V1 (dette technique) à la V2 (clean slate, contrats machine)
> Projet d'orchestration IA hybride locale (LM Studio) + cloud

## Leçons apprises

Ce projet documente deux itérations complètes d'une architecture IA :

### V1 — L'accumulation
- Architecture hybride 7 couches (LM Studio local + API cloud)
- 14 rapports de dépannage, configuration baseline, ajustements
- **Cause de mort** : correctif sur correctif, 15 fallbacks empilés, aucune réinitialisation
- La séquence "ça marche sur ma machine" s'est accumulée jusqu'à un système impossible à maintenir

### V2 — Le reset
- **Clean Slate** : abandon de V1, redémarrage propre
- Contrats machine avant le code (`Architecture_Contract.docx`, `Orchestration_Contract.docx`)
- Séparation stricte MAIN (système) / LOCAL (LM Studio)
- Runbook en 10 phases, hardening structurel
- Journal de session, plan d'action professionnel

## Structure

```
├── V1/
│   └── Document Project/     # 14 rapports (architecture, déploiement, diagnostic)
├── V2/
│   ├── Architecture_Contract.docx → .md
│   ├── Reset_V2_Action_Plan.md
│   ├── Hardening_Structurel.md
│   └── Session1_Journal.md
└── Rapport_Consolide_OpenClaw.md
```

## Ce que ce projet démontre

1. **Conscience de la dette technique** — identifier et acter l'échec plutôt que patcher à l'infini
2. **Méthodologie** — contrats avant code, séparation des préoccupations
3. **Documentation rigoureuse** — chaque décision tracée, chaque échève analysé

## Auteur

Malik — étudiant DEP Soutien Informatique, autodidacte
Projet : août 2026
