
# OPENCLAW RESET V2PROFESSIONAL ACTION PLAN


Clean rebuild • MAIN/LOCAL separation • Hybrid orchestration • LM Studio • Evidence-driven validation

VERSION=2.1 | DATE=2026-08-10 | STATUS=PLANNED | SCOPE=OpenClaw runtime + agents + providers + LM Studio + Telegram


# 1. EXECUTIVE OBJECTIVE


Rebuild OpenClaw from a known-good baseline instead of continuing incremental repairs. The target architecture separates MAIN and LOCAL at runtime: MAIN is the cloud orchestrator/reasoner; LOCAL is the execution worker using LM Studio. Runtime configuration enforces routing, permissions and delegation; Markdown files provide policy/identity/knowledge but are not the primary security boundary.

TARGET = USER → MAIN → reasoning/research/planning → explicit delegation → LOCAL → authorized tools → validation → MAIN → USER


# 2. DESIGN PRINCIPLES


Clean baseline before optimization.

One architectural layer at a time; validate before proceeding.

Runtime configuration is authoritative for agents, models, routing and tool policy.

SOUL.md / AGENTS.md / knowledge are complementary policy/knowledge layers, never substitutes for runtime enforcement.

MAIN and LOCAL have explicit responsibilities.

Cloud fallback remains short, deterministic and individually tested.

LM Studio is the inference provider for LOCAL, not the routing controller.

Every mutation has pre-change backup and post-change validation.

Unknown is a valid state; never infer success from configuration alone.

Do not modify unrelated working components.


# 3. TARGET ARCHITECTURE


USER ├─ @local ───────────────> LOCAL ──> LM STUDIO ──> authorized execution │ └─ normal request ───────> MAIN (Cloud)                              ├─ research                              ├─ reasoning                              ├─ analysis                              ├─ planning                              └─ sessions_spawn(agentId="local")                                         │                                         v                                       LOCAL                                         │                              tools / filesystem / automation                                         │                                         v                                     validation                                         │                                         v                                       MAIN                                         │                                         v                                       USER


# 4. PHASE 0 — BACKUP & EVIDENCE BASELINE


Inventory OpenClaw version, workspace path, config path, active processes and gateway state.

Stop/quiesce OpenClaw Gateway before copying mutable runtime files.

Archive all current governance Markdown files: SOUL.md, AGENTS.md, TOOLS.md, USER.md, MEMORY.md, IDENTITY.md, HEARTBEAT.md and any other active instruction files.

Archive the complete current openclaw.json.

Preserve knowledge/, skills/, agents/ and relevant workspace configuration for comparison/restore.

Store backup outside the active configuration/workspace tree.

Use an ISO-8601 timestamp and SHA-256 hashes for backup artifacts.

Record version, paths, timestamp and hashes in a manifest.

EXIT CRITERIA: backup exists + hash manifest exists + restore path known + no active config modified.


# 5. PHASE 1 — RESET / CLEAN BASELINE


Verify exact installed OpenClaw version and consult the matching official reset/reinitialization documentation.

Use the native supported reset/reinitialization mechanism for that release; do not improvise deletion of undocumented runtime paths.

Inspect configuration, sessions, locks, gateway state and generated runtime state for residue.

Run the supported diagnostic/doctor command and preserve the output.

Confirm no stale agent IDs, stale subagent allowlist entries, orphaned gateway state or unexpected configuration survives.

Do not restore the old configuration yet.

EXIT CRITERIA: clean baseline proven by runtime inspection and diagnostics.


# 6. PHASE 2 — MINIMAL RUNTIME SKELETON


Create/validate the smallest supported openclaw.json.

Define MAIN and LOCAL as distinct configured agents.

Do not configure complex routing, fallback chains, tools or channels yet.

Do not use Markdown instructions to compensate for missing runtime configuration.

Validate JSON/schema and start the Gateway.

Confirm both agent IDs resolve independently before adding providers.

EXIT CRITERIA: MAIN and LOCAL are independently addressable runtime agents.


# 7. PHASE 3 — CLOUD PROVIDERS


Inject credentials using the supported secret/environment mechanism; never commit keys to Markdown.

Configure the cloud primary model.

Add only the required fallback providers; prefer a short tested chain.

Test every provider independently before placing it in the fallback chain.

Record provider ID, model ID, authentication result, latency/error class and validation date.

Validate the exact fallback order.

Do not proceed with an untested fallback entry.

EXIT CRITERIA: primary and every fallback tested independently and chain behavior verified.


# 8. PHASE 4 — LM STUDIO / LOCAL


Start LM Studio Server on port 1234.

Validate /v1/models and, where useful, LM Studio native /api/v1/models.

Inventory available model IDs and classify them by role.

Select only 1–2 models: e.g. utility/execution plus optional stronger local model.

Do not preload all downloaded models.

Assign the selected LM Studio model explicitly to LOCAL.

Confirm exact provider/model resolution at runtime.

Test a real inference request through OpenClaw, not only a raw LM Studio API request.

Define JIT/TTL/load behavior deliberately.

EXIT CRITERIA: LOCAL resolves to lmstudio/<model-id> and completes a real inference test.


# 9. PHASE 5 — TOOL POLICY & DELEGATION (CORE)


Define MAIN as orchestrator/reasoner; restrict execution tools after verifying the exact effective tool names in the installed release.

Define LOCAL with the minimum required execution tools.

Use agent-specific tool allow/deny policies where appropriate; deny wins and restrictive layers cannot be re-granted later.

Configure MAIN's subagent allowlist to explicitly permit LOCAL.

Set requireAgentId=true for deterministic target selection.

Configure MAIN → LOCAL via sessions_spawn.

Prevent uncontrolled recursive delegation unless explicitly required.

Keep @local as a direct route to LOCAL, separate from MAIN→LOCAL delegation.

Verify effective tools from the runtime/session, not JSON text alone.

EXIT CRITERIA: MAIN cannot perform prohibited execution; MAIN can explicitly spawn LOCAL; LOCAL has only authorized execution tools.


# 10. PHASE 6 — TELEGRAM


Bind Telegram only after agent/model/delegation tests pass locally.

Store the Telegram token using the supported environment/secret mechanism.

Start with local/loopback gateway validation.

Test authentication and inbound/outbound flow.

Test @local direct routing.

Test normal MAIN routing.

Test MAIN→LOCAL hybrid execution from Telegram.

Do not introduce WhatsApp until Telegram is stable.

EXIT CRITERIA: Telegram passes direct, hybrid and failure-path tests.


# 11. PHASE 7 — GOVERNANCE FILES


Rewrite SOUL.md as identity/kernel/policy only.

Rewrite AGENTS.md as role/orchestration guidance.

Create/refresh TOOLS.md from actual runtime capabilities.

Rebuild USER/IDENTITY/MEMORY/HEARTBEAT only where supported and necessary.

Rebuild knowledge files from verified technical facts.

Include anti-confabulation/evidence rules.

State explicitly that runtime configuration has authority over Markdown for routing, permissions and model selection.

Never encode critical security controls exclusively in natural language.

EXIT CRITERIA: Markdown layer documents the verified runtime; it does not substitute for runtime enforcement.


# 12. PHASE 8 — TOTAL VALIDATION


☐ AC01 MAIN model resolves to intended cloud primary.

☐ AC02 LOCAL model resolves to LM Studio.

☐ AC03 @local targets LOCAL directly.

☐ AC04 MAIN can spawn LOCAL with explicit agentId.

☐ AC05 requireAgentId blocks ambiguous spawn.

☐ AC06 MAIN effective tools exclude prohibited execution capabilities.

☐ AC07 LOCAL effective tools include only required execution capabilities.

☐ AC08 LM Studio endpoint and selected model are reachable.

☐ AC09 LOCAL executes an authorized real task.

☐ AC10 LOCAL returns evidence/result to MAIN.

☐ AC11 MAIN synthesizes the returned result.

☐ AC12 Cloud fallback order remains unchanged and validated.

☐ AC13 No unintended secrets exist in configuration.

☐ AC14 No stale gateway/session/config residue remains.

☐ AC15 Backup and rollback procedure is usable.

☐ AC16 Restart/reboot preserves the architecture.


# 13. PHASE 9 — REAL-WORLD TEST MATRIX


TEST

FLOW

EXPECTED

EVIDENCE

@local direct

USER→LOCAL→LM Studio

No MAIN invocation

Agent/model resolution + response

Research

USER→MAIN→Web

MAIN researches/reasons

Sources + result

Hybrid task

USER→MAIN→LOCAL

LOCAL executes

Result + validation evidence

System task

MAIN→LOCAL→tool

LOCAL executes authorized tool

Runtime output

Negative MAIN

MAIN→forbidden tool

Execution denied

Denial evidence

Negative LOCAL

LOCAL→unauthorized tool

Access denied

Denial evidence

Telegram

Telegram→MAIN

MAIN responds

Gateway evidence

Telegram hybrid

Telegram→MAIN→LOCAL

LOCAL executes and returns

End-to-end evidence

Restart

Gateway restart→tests

Architecture remains valid

Post-restart evidence


# 14. PHASE 10 — CLOSURE & DOCUMENTATION


Create V2 success report.

Record final OpenClaw version and configuration hash.

Record final agent/model/provider matrix.

Record effective tool policies.

Record tested cloud fallback chain.

Record LM Studio model IDs and relevant runtime settings.

Record Telegram secret-management method without exposing secrets.

Record every acceptance criterion and result.

Archive final validated configuration.

Update the Universal AI Architecture Contract with implementation-specific decisions.


# 15. PHASE GATE


EVERY_PHASE = AUDIT → CHANGE → VALIDATE → EVIDENCE → PASS/STOP

A failed phase blocks progression. Never compensate for a failed runtime layer by adding more Markdown instructions. Fix the layer where the failure exists.


# 16. ROLLBACK STRATEGY


P0=restore configuration backupP1=restore governance/workspace backupP2=remove only newly added provider/agent/tool/channel configurationP3=restore last validated openclaw.jsonP4=restart gateway + diagnosticsFAILURE=STOP + PRESERVE_EVIDENCE + ROLLBACK_WHEN_DEFINED


# 17. OFFICIAL SOURCES


OpenClaw — Agents configuration — https://docs.openclaw.ai/gateway/config-agents

OpenClaw — Sub-agents — https://docs.openclaw.ai/tools/subagents

OpenClaw — Tools/configuration — https://docs.openclaw.ai/gateway/config-tools

OpenClaw — Multi-agent tool policy — https://docs.openclaw.ai/tools/multi-agent-sandbox-tools

LM Studio — OpenClaw integration — https://lmstudio.ai/docs/integrations/openclaw

LM Studio — REST API — https://lmstudio.ai/docs/developer/rest

LM Studio — List models — https://lmstudio.ai/docs/developer/rest/list

LM Studio — Load model — https://lmstudio.ai/docs/developer/rest/load

LM Studio — OpenAI compatibility — https://lmstudio.ai/docs/developer/openai-compat


# 18. IMPORTANT IMPLEMENTATION NOTES


Follow the configuration schema of the exact installed OpenClaw version; do not blindly paste fragments from another release.

Verify actual effective tool names before constructing deny/allow lists.

Sub-agent model inheritance can follow the caller unless an explicit subagent model or explicit spawn model is configured; LOCAL's model must therefore be explicit.

LM Studio currently documents its native /api/v1 REST API as the current native API, while OpenClaw integration uses the OpenAI-compatible endpoint; distinguish these layers during diagnostics.

A successful raw LM Studio API test proves LM Studio availability, not correct OpenClaw LOCAL model resolution.

Do not reorder the cloud fallback chain without explicit validation.


# 19. DEFINITION OF DONE


DONE =BACKUP_VERIFIEDAND RESET_VERIFIEDAND MAIN_CONFIGUREDAND LOCAL_CONFIGUREDAND CLOUD_PRIMARY_TESTEDAND CLOUD_FALLBACKS_TESTEDAND LMSTUDIO_TESTEDAND LOCAL_MODEL_EXPLICITAND MAIN_TOOL_RESTRICTIONS_VALIDATEDAND LOCAL_TOOL_PERMISSIONS_VALIDATEDAND MAIN_TO_LOCAL_DELEGATION_VALIDATEDAND @LOCAL_VALIDATEDAND TELEGRAM_VALIDATEDAND GOVERNANCE_REBUILTAND NEGATIVE_TESTS_PASSAND REAL_WORLD_TESTS_PASSAND ROLLBACK_VERIFIEDAND FINAL_REPORT_CREATED


# 20. NEXT SESSION — EXECUTION ORDER


Start at Phase 0. Do not jump directly to delegation because it is the desired end state. The clean rebuild is intentionally sequential: backup → reset → minimal runtime → cloud → LM Studio → delegation → Telegram → governance → full validation → real tests → closure.
