# OpenClaw — audit et documentation

Dossier d'audit et de documentation d'un projet **abandonné**. Il n'y a plus de
développement ici — la valeur est documentaire.

## Contenu

| Chemin | Rôle |
|---|---|
| `V1/` | 14 fichiers — rapports, configs, dépannage. Période 2–7 août 2026 |
| `V2/` | 10 fichiers — plans, contrats, runbooks. 10 août 2026 |
| `Rapport_Consolide_OpenClaw.docx` | document unique agrégé, **plafonné à 50 pages** |

## La leçon methodology (le point clé de ce dossier)

**V1 a échoué** : correctif sur correctif, 15 fallbacks empilés, aucune
réinitialisation. La séquence « ça marche sur ma machine » s'est accumulée
jusqu'à un système que personne ne pouvait modifier sans le casser.

**V2 a réussi** : clean slate. `MAIN` / `LOCAL` séparés, contrats machine
écrits avant le code, 10 phases.

**Règle applicable** : quand un système empile ses correctifs, le premier
travail est de **réinitialiser**, pas de corriger. Le nombre de couches de
correctifs est le signal.

## Règles de rédaction

- Concis et rigoureux. **Plafond de 50 pages** sur le rapport consolidé.
- Chaque affirmation sur un CVE ou une faille doit porter sa source et sa date.
- Ne pas mélanger documentation et vie personnelle du propriétaire.
