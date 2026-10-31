# OpenClaw

A documented hybrid AI architecture experiment that compares a local LM Studio workflow with cloud AI services.

## Why this repository exists

OpenClaw records two iterations of the same idea:

- **V1** — a layered system that accumulated patches and fallbacks until it became difficult to maintain.
- **V2** — a clean restart based on machine-readable contracts, a strict separation between the main system and local inference, and a step-by-step runbook.

The main lesson is practical: define the system boundaries before adding more fixes.

## Repository map

```
V1/   Historical reports, architecture notes and diagnostics
V2/   Contracts, reset plan, hardening notes and runbook
```

## What it demonstrates

- How technical debt grows in an AI integration project
- Why local and cloud responsibilities should be explicit
- How contracts and journals make troubleshooting repeatable
- How to reset an unstable design without losing the lessons learned

## Status

This is primarily a documentation and methodology project. Read the V2 contract and action plan first.

## Author

Built by [Malik Belkacemi](https://github.com/MlkBlk-IT).
