# Detection framework cross-walk - v0.1

> **This cross-walk is the author's interpretive synthesis. It is not Microsoft's mapping, and not
> an official mapping by MITRE or OWASP.** It maps each detection in this pack to specific framework
> items at item level, never category-to-product.

## Framework versions cited

| Framework | Version / edition | How it was verified | Last verified |
|---|---|---|---|
| MITRE ATLAS | ATLAS knowledge base, distributed `atlas-data` dataset `version: 5.6.0` (release tag `v2026.07`) | `dist/ATLAS.yaml` parsed directly; 170 technique id/name pairs extracted; every ID this pack cites matched against that parse, where an identifier named only to illustrate the v6 restructure is not a citation and the methodology's section 4 states the same scope | 2026-08-15 |
| OWASP Top 10 for LLM Applications | **2026 edition** (v1.0, published 2026-08-03 by the OWASP GenAI Security Project) | Edition existence and publication date re-confirmed against the OWASP GenAI Security Project resource page on 2026-08-15. The per-item numbering is carried from the companion capability-status matrix repository, which verified the artefact by SHA-256 against OWASP's published download - see the caveat below | 2026-08-15 (edition) · 2026-08-09 (item numbering) |

> **One caveat about checking this yourself.** The `genai.owasp.org/llm-top-10/` landing page still
> presented the **2025** edition when re-read on 2026-08-23, listing its ten items as `LLM01:2025`
> through `LLM10:2025`.
> The 2026 edition is published and dated on the project's resource page. If you check the landing
> page and conclude 2025 is current, that is the reason - and it is why the item numbering here is
> attributed to a SHA-256-verified read of the published artefact rather than to a web page.
>
> **The version and date in the row above are OWASP's own distribution labels**, carried from the
> resource page and the published file name. **The document labels itself `Version 2026`**, and its
> cover leaves the publication date unset above `August 4th, 2026`. Section 5 of
> [`docs/verification-methodology.md`](../docs/verification-methodology.md) records both readings
> rather than reconciling them.

**ATLAS IDs were read from the dataset, not recalled.** That is a deliberate control: technique IDs
are exactly the kind of detail a plausible guess gets almost right. **The dataset cited in the row
above declares itself deprecated, and the same release tag ships a successor emission under a newer
schema version.** Section 4 of [`docs/verification-methodology.md`](../docs/verification-methodology.md)
states what that does and does not affect here.

**OWASP 2026 numbering only.** Only LLM01 Prompt Injection and LLM02 Sensitive Information
Disclosure kept the numbers they held in the 2025 edition. Every other slot changed occupant, so a
2025-era `LLM0x` ID must never be carried over unchanged.

---

## The cross-walk

| Detection | Surface | Status | MITRE ATLAS | OWASP LLM 2026 | Notes (synthesis) |
|---|---|---|---|---|---|
| **MSD-001** Prompt-injection email detected | `EmailEvents` | GA (no preview qualifier) | `AML.T0051` LLM Prompt Injection · `AML.T0051.001` Indirect | LLM01:2026 Prompt Injection | The indirect sub-technique is the right one: the instruction arrives inside content the model ingests, not from the operator. Detective, and scoped to one channel. |
| **MSD-002** Prompt-injection email delivered | `EmailEvents` | GA (no preview qualifier) | `AML.T0051` · `AML.T0051.001` Indirect | LLM01:2026 Prompt Injection | Same technique as MSD-001. The mapping does not change; the response does. |
| **MSD-003** Agents with MCP servers attached | `AgentsInfo` | Public Preview | `AML.T0010.005` AI Supply Chain Compromise: AI Agent Tool · `AML.T0011.002` User Execution: Poisoned AI Agent Tool | LLM03:2026 Excessive Agency · LLM04:2026 Supply Chain | **Visibility, not mitigation.** An MCP server is a third-party component entering the agent's tool surface (LLM04) and an extension of what the agent can reach (LLM03). `AML.T0010.005` is the compromised-tool case; `AML.T0011.002` is the poisoned-tool case an allowlist is meant to keep out. This query detects neither - it tells you what is attached. |
| **MSD-004** Broad agents, tools, no reported guardrails | `AgentsInfo` | Public Preview | ***Not mapped*** - see the note below | LLM03:2026 Excessive Agency | **Posture, not detection, and the ATLAS cell is deliberately empty.** The two techniques a reader is most likely to expect here are `AML.T0053` AI Agent Tool Invocation and `AML.T0086` Exfiltration via AI Agent Tool Invocation. Both are runtime-execution techniques and this query reads a **configuration** table that cannot observe a single invocation, so printing either in an ATLAS column would claim coverage from a detection structurally incapable of seeing it. The posture is a **precondition** for both; it is not either of them. |
| **MSD-005** Defender for Cloud AI alerts | `SecurityAlert` | GA (stated) for the plan; 2 of 17 alerts Preview | `AML.T0054` LLM Jailbreak · `AML.T0051.001` Indirect · `AML.T0057` LLM Data Leakage · `AML.T0034` Cost Harvesting · `AML.T0053` AI Agent Tool Invocation · `AML.T0010` AI Supply Chain Compromise | LLM01:2026 · LLM02:2026 Sensitive Information Disclosure · LLM06:2026 Unbounded Consumption · LLM04:2026 Supply Chain · LLM05:2026 Data and Model Poisoning | **The only row where Microsoft supplies the detection and this pack supplies the correlation.** Six techniques because the 17-alert set genuinely spans six. **Jailbreak maps to `AML.T0054`, not `AML.T0051`** - ATLAS carries them as distinct techniques, and MSD-008 uses `AML.T0054` for the same concept, so mapping the jailbreak alerts to `AML.T0051` would leave one pack using two IDs for one thing. ASCII smuggling stays on `AML.T0051.001` on Learn's own wording. `AML.T0053` covers the anomalous-tool-invocation alert; `AML.T0010` and LLM04 cover the malicious-uploaded-model alert. **LLM05 Data and Model Poisoning is the stretch in that last cell**, and MSD-005 records it as one: the alert reports malicious content found in an uploaded model, which is the supply-chain event LLM04 names directly, while LLM05 describes poisoning of training or model data that the alert does not itself establish. **Azure-hosted workloads only** - see the row's scope boundary before mapping it to any Copilot control. |
| **MSD-006** Workload-identity sign-in without applied CA | `AADServicePrincipalSignInLogs` | GA (no preview qualifier); 5 elements Provisional | `AML.T0012` Valid Accounts | LLM03:2026 Excessive Agency | **The only row here that measures a preventive control.** `AML.T0012` is the technique it defends against - an adversary operating with a legitimate service-principal identity. **`AML.T0083` Credentials from AI Agent Configuration is deliberately *not* in this cell**, though it is the technique a reader might expect: an agent authenticating with a key held in its configuration bypasses Entra ID entirely and produces no row, so it sits **outside** what this detection can see. A technique ID in a column headed MITRE ATLAS reads as coverage, and using one cell to mean the opposite would defeat the item-level precision this artefact exists for. It is recorded in the not-mapped table below instead. |
| **MSD-007** Copilot configuration changes | `CopilotActivity` | **Requires further validation** | `AML.T0081` Modify AI Agent Configuration · `AML.T0012` Valid Accounts | LLM03:2026 Excessive Agency **(weakest cell in this table - see below)** | **A telemetry surface, not a control.** `AML.T0081` fits a settings change directly; `AML.T0012` covers the case where the change is made by a legitimately-authenticated but unexpected actor. |
| **MSD-008** AI-agent real-time protection behaviours | `BehaviorInfo` | Public Preview, not available for GCC | `AML.T0051` LLM Prompt Injection · `AML.T0054` LLM Jailbreak | LLM01:2026 Prompt Injection | Detective, and **one of two detections here that reflect an enforcement event** rather than an observation - MSD-005's `BlockedAttempt` alert is the other, and MSD-005 makes the blocked-versus-detected distinction a centrepiece of its own triage guidance. Audit-mode and block-mode behaviours both land in this table and the mapping does not distinguish them; your triage must. |

---

## The weakest cell, named

**MSD-007's OWASP mapping.** On the 2026 item list as carried in the companion cross-walk - item
numbering verified there on 2026-08-09, not independently re-derived here - **no item covers
configuration tampering against an AI platform's administrative surface.** LLM03:2026 Excessive
Agency is the closest fit, on the reading that a settings change is how agent scope gets widened,
but that is an inference about *what a change might do* rather than about what the detection
observes. The ATLAS cell (`AML.T0081` Modify AI Agent Configuration) is a clean fit; the OWASP cell
is a stretch, and it is recorded as one rather than presented as settled.

**MSD-004's ATLAS cell was the other candidate, and it was resolved rather than ranked.** Citing
either runtime-execution technique on a detection that reads a configuration table would have been
weaker than a fit-stretch: it would have been a category error, and two cells of it. An empty cell
with a stated reason claims less and survives a hostile read, which is why that cell no longer
competes for this title.

Naming the weakest cell is the point. A cross-walk with no weak cells has usually stopped checking.

---

## What this cross-walk deliberately does not claim

- **It does not claim coverage.** A technique appearing in the ATLAS column means one detection in
  this pack touches that technique on one surface. It does not mean the technique is covered. Three
  of the eight detections have a **primary** query that detects nothing at all - MSD-003, MSD-004
  and MSD-006 are inventory and posture queries. Their change-detection variants alert on a
  transition, which is a different job.
- **It does not claim Microsoft agrees.** Where a Microsoft table publishes its own framework column
  - `BehaviorInfo.AttackTechniques` carries MITRE **ATT&CK** techniques, and `SecurityAlert` carries
  `Tactics` and `Techniques` - those are Microsoft's values and this table's ATLAS mapping is
  separate. Do not read the two as agreeing.
- **It does not map to NIST AI 600-1 or the CSA AICM.** The companion capability-status matrix
  repository maps Microsoft *controls* to those two frameworks. This pack maps *detections* to
  adversary techniques, which is a different axis, and adding two half-considered columns to look
  complete would be worse than leaving them out.
- **It has no coverage score.** Any number here would be a claim about a denominator nobody has.

## Techniques deliberately not mapped

Named so the gaps are visible rather than implied.

| ATLAS technique | Why nothing here maps to it |
|---|---|
| `AML.T0057` LLM Data Leakage, on the Microsoft 365 Copilot path | No documented advanced-hunting surface exists for Copilot chat prompts. MSD-005 maps it on the Azure path only. |
| `AML.T0083` Credentials from AI Agent Configuration | The condition it describes is **outside** MSD-006's view: an agent authenticating with a key in its configuration bypasses Entra ID and produces no sign-in row. Recorded here rather than in MSD-006's ATLAS cell, so a technique ID never reads as coverage of the thing it is the blind spot for. |
| `AML.T0053` AI Agent Tool Invocation, on the agent-posture path | Mapped in MSD-005 for the Azure alert that observes it. **Not** mapped to MSD-004, which reads configuration and cannot see an invocation. The surface that would observe it across agents is `CloudAppEvents`, deferred to v0.2. |
| `AML.T0086` Exfiltration via AI Agent Tool Invocation | Same reason as the row above. No detection in v0.1 observes agent tool invocation. |
| `AML.T0056` Extract LLM System Prompt | **Partially reachable and deliberately not claimed, and the reason is the alert's own scope rather than its release state.** MSD-005 ships `AI.Azure_LLMReconnaissance`, whose Learn description reads: "A threat actor is interacting with your AI application in a way that resembles reconnaissance behavior, including attempts to extract system instructions, model capabilities, or bypass safety guardrails." System-prompt extraction is **one of three behaviours the alert bundles**, and the alert says the activity *resembles* reconnaissance rather than that extraction occurred. A cell reading `AML.T0056` would therefore claim item-level precision the alert does not carry, which is the one thing this artefact exists not to do. **Its `(Preview)` tag is not the reason** - `AI.AIModelScan_MalwareDetected` is the other preview-tagged alert and it **is** mapped, on `AML.T0010` in MSD-005's row of the main table above rather than anywhere in this table. Nor does preview status empty a cell here: of the three Public Preview detections in the main table, MSD-003 and MSD-008 both carry ATLAS techniques, all three carry an OWASP item, and the one empty ATLAS cell among them is MSD-004's, which its own row explains by what the query reads rather than by its release state. Read live 2026-08-17. On the agent path, `AgentsInfo.Instructions` holds the system prompt but records configuration, not extraction attempts. |
| `AML.T0024` Exfiltration via AI Inference API | Maps to shadow-AI discovery, which is a Defender for Cloud Apps surface this pack does not build on in v0.1. |
| `AML.T0080` AI Agent Context Poisoning | No documented surface identified. |
| `AML.T0110` AI Agent Tool Poisoning | Adjacent to MSD-003, which sees configuration rather than poisoning. |

## Sources

- MITRE ATLAS: [atlas.mitre.org](https://atlas.mitre.org/) · dataset [github.com/mitre-atlas/atlas-data](https://github.com/mitre-atlas/atlas-data), release `v2026.07`, `dist/ATLAS.yaml` `version: 5.6.0`
- OWASP Top 10 for LLM Applications 2026 edition: [genai.owasp.org/resource/owasp-genai-llm-top-10-2026/](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/)
- Microsoft Learn citations are recorded per detection, in each detection file's Sources section.
