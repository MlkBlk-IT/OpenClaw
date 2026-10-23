<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:e10600,50:1a0505,100:00ff9c&height=140&section=header&text=OpenClaw&fontSize=42&fontColor=e10600&fontAlignY=32&desc=Hybrid%20local%20%2B%20cloud%20agentic%20AI%20architecture%20%E2%80%94%20V1%20to%20V2%2C%20machine%20contracts&descAlignY=53&descSize=15&descColor=00ff9c" width="100%" alt="OpenClaw" />

<img src="https://img.shields.io/badge/LM%20Studio-e10600?style=for-the-badge&labelColor=0d0d0d" /> <img src="https://img.shields.io/badge/Agentic%20AI-00ff9c?style=for-the-badge&labelColor=0d0d0d" /> <img src="https://img.shields.io/badge/Methodology%20%2B%20Docs-e10600?style=for-the-badge&labelColor=0d0d0d" />

</div>

> From V1 (technical debt) to V2 (clean slate, machine contracts)
> Hybrid local (LM Studio) + cloud AI orchestration project

## Lessons learned

This project documents two complete iterations of an AI architecture:

### V1 — The accumulation
- 7-layer hybrid architecture (local LM Studio + cloud API)
- 14 troubleshooting reports, baseline configuration, tuning
- **Cause of death**: patch on patch, 15 stacked fallbacks, no reset
- The "works on my machine" sequence piled up into an unmaintainable system

### V2 — The reset
- **Clean Slate**: V1 abandoned, clean restart
- Machine contracts before code (`Architecture_Contract.docx`, `Orchestration_Contract.docx`)
- Strict MAIN (system) / LOCAL (LM Studio) separation
- 10-phase runbook, structural hardening
- Session journal, professional action plan

## Structure

```
├── V1/
│   └── Document Project/     # 14 reports (architecture, deployment, diagnostics)
├── V2/
│   ├── Architecture_Contract.docx → .md
│   ├── Reset_V2_Action_Plan.md
│   ├── Hardening_Structurel.md
│   └── Session1_Journal.md
└── Rapport_Consolide_OpenClaw.md
```

## What this project demonstrates

1. **Technical-debt awareness** — identifying and recording the failure rather than patching forever
2. **Methodology** — contracts before code, separation of concerns
3. **Rigorous documentation** — every decision traced, every failure analyzed

## Author

Malik — IT Support student (DEP), self-taught
Project: August 2026

---

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:00ff9c,50:03150c,100:e10600&height=100&section=footer" width="100%" alt="" />

**⚡ Built on CPU. Proven on disk. ⚡**

</div>
