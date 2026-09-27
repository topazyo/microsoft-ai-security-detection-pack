# Microsoft AI Security Detection Pack

**Cited, status-labelled KQL detections for AI-security signals in Microsoft Sentinel and Microsoft
Defender XDR - built only on tables and columns Microsoft documents.**

---

## What this is

Eight detections across six advanced hunting and Log Analytics tables. Each one carries the exact
table and columns it depends on, a Microsoft Learn citation, a last-verified date, a release-status
label, a MITRE ATLAS technique mapping, and explicit guidance on its false positives and its blind
spots.

The point is not volume. It is that **every schema element a detection file or a shipped query relies
on is one of three things: quoted from a Microsoft Learn page read on a stated date, or marked
Provisional or Requires further validation with the reason stated and a workspace step that resolves
it, or labelled at the point of use as observed in a tenant on a stated date rather than as
documented.** **The third class has one member and it is named here rather than left for a reader to
find**: MSD-002's filter carries a `DeliveryLocation` spelling that a tenant emitted on
2026-08-24 and that Microsoft Learn does not publish, kept beside the published spelling so the
filter matches either, and labelled as observed inside both of that file's query blocks. **A member
of that class may widen a filter and may never narrow one**, and that bound reaches an aggregation on
the column as well as a predicate on it. **That constraint is the load-bearing half of the class**,
because without it an observed value could be pasted into a filter that removes rows, which is the
silent failure this pack argues against. **There is no fourth category.** A rule with an
unstated exception is worth less than a rule with a stated one.
[`docs/verification-methodology.md`](docs/verification-methodology.md) section 1 is the canonical
statement of the rule and names one of the classes that sit outside it, which
[`docs/scope-and-out-of-scope.md`](docs/scope-and-out-of-scope.md) enumerates in full; if this
README and that section ever disagree, that section governs.

## Why that rule matters more than the queries

AI-security detection content has a specific failure mode: plausible schema. A column name like
`AbnormalCopilotBehavior` reads exactly like real Defender schema. **Invent the
column outright and the query fails loudly**: Microsoft classes a reference to a nonexistent column
as a syntax error, so the query never runs
([advanced hunting errors](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-errors),
read 2026-09-21). **The silent case is the expensive one** - a real column filtered on a guessed
value, or a placeholder left in place, returns zero rows and reads as a clean environment. The
reader has then lost time and learned nothing, and that is the case this pack is built against.
[`SECURITY.md`](SECURITY.md) ranks that class first for the same reason.

So several queries here are less convenient than they could be. Where Microsoft does not publish a
value set, this pack ships a **discovery query** that enumerates what your tenant actually emits,
and leaves the narrowing filter for you to complete. That is deliberate. A tidier rule built on a
guessed value string would be the defect, not the improvement.

## Who this is for

- **Primary:** detection engineers and SOC leads building AI-security coverage on Microsoft Sentinel
  and Microsoft Defender XDR.
- **Secondary:** security architects mapping AI risk to telemetry; anyone who needs to know what
  Microsoft's AI-security surfaces can and cannot see before drawing them on a diagram.

## Status legend

The canonical definitions live in
[`docs/verification-methodology.md`](docs/verification-methodology.md) section 2. **This table
reproduces those five rows verbatim**, so that a difference between the two is a defect to report
rather than a nuance to interpret; if they ever do disagree, that one governs.

| Label | Rule |
|---|---|
| **GA (stated)** | Microsoft Learn asserts general availability outright - a release-state row, or a release-note sentence naming the GA date. |
| **GA (no preview qualifier)** | The documenting page carries no preview qualifier and no release-state sentence of any kind. **This is inference from absence and it is a weaker class of evidence.** |
| **Public Preview** | Learn carries a `(Preview)` qualifier in the page or table title, or an explicit public-preview sentence. |
| **Provisional** | The table or capability has a status, but a specific column, value set, or serialisation used by the query is undocumented. Scoped to the element, not the detection. |
| **Requires further validation** | Microsoft's own sources conflict, or the status cannot be established from primary sources. Never guessed, never averaged. |

Two things that table does not carry, because they are commentary on the labels rather than part of
them. **Preview terms apply to anything labelled Public Preview**, and the preview surfaces in this
pack are named in [`disclaimer.md`](disclaimer.md). And **one detection here carries GA (stated)**:
the distinction between the first two rows costs two words per label, and it is the distinction the
pack exists to make.

## The detections

### Microsoft Defender XDR advanced hunting

| ID | Detection | Table | Status | Deploy as |
|---|---|---|---|---|
| [MSD-001](detections/defender-xdr/MSD-001-prompt-injection-email-detected.md) | Prompt-injection email detected by Defender for Office 365 | `EmailEvents` | **GA (no preview qualifier)** | Hunting query or low-volume rule |
| [MSD-002](detections/defender-xdr/MSD-002-prompt-injection-email-delivered.md) | Prompt-injection email that still reached a mailbox | `EmailEvents` | **GA (no preview qualifier)** | **Custom detection rule** |
| [MSD-003](detections/defender-xdr/MSD-003-agents-with-mcp-servers.md) | AI agents with MCP servers attached | `AgentsInfo` | **Public Preview** | Change-detection variant only - run its verification step 7 first |
| [MSD-004](detections/defender-xdr/MSD-004-broad-agents-without-guardrails.md) | Broadly available agents with declared tools and no reported guardrails | `AgentsInfo` | **Public Preview** | Posture review; change variant once baselined |
| [MSD-008](detections/defender-xdr/MSD-008-agent-runtime-protection-behaviours.md) | AI-agent real-time protection block and audit behaviours | `BehaviorInfo` | **Public Preview**, not available for GCC | Discovery first; no shipped filter |

### Microsoft Sentinel (Log Analytics)

| ID | Detection | Table | Status | Deploy as |
|---|---|---|---|---|
| [MSD-005](detections/sentinel/MSD-005-defender-for-cloud-ai-alerts.md) | Defender for Cloud AI-workload alerts arriving in Sentinel | `SecurityAlert` | **GA (stated)** for the plan; 2 of 17 alerts Preview | **Hunting query**, or an analytics rule with incident creation off - see the duplicate-incident caveat |
| [MSD-006](detections/sentinel/MSD-006-workload-identity-signin-conditional-access.md) | Workload-identity sign-in where Conditional Access did not apply | `AADServicePrincipalSignInLogs` | **GA (no preview qualifier)**; 5 elements Provisional | Change-detection variant only |
| [MSD-007](detections/sentinel/MSD-007-copilot-settings-change.md) | Copilot configuration changes in the Sentinel audit stream | `CopilotActivity` | **Requires further validation** | Discovery first; step 2 once step 1 is read |

The split is not cosmetic. Defender XDR advanced hunting queries deploy as Defender custom detection
rules; Sentinel queries deploy as Sentinel analytics rules. The schemas are different and so are the
prerequisites.

### What "eight detections" actually gets you

Read the Deploy-as column before you plan around this pack. Not all eight are rules, and saying so
is more useful than the headline number:

- **Deployable after verification: MSD-001, MSD-002, MSD-005.** MSD-002 is the strongest thing here
  and the only one producing a per-message action item on a non-preview surface.
- **Deployable after verification *and* baselining: MSD-003, MSD-004 and MSD-006**, change-detection
  variants only. Each of those three files says which variant to schedule and why. Their primary
  queries are posture reviews rather than rules. **MSD-003's variant carries one extra condition**:
  it is the only query here meant for deployment that depends on a hashing function. **The
  2026-08-24 run ran that check in one tenant**: `hash_sha256()` ran in Defender XDR
  advanced hunting there without error and returned a digest of the documented shape, which is a
  smaller result than a match against a published value and is all the cited page supports. It did not make the
  variant deployable, because the table did not resolve there. **Every other use of
  one in this pack is a check you run once rather than a rule you schedule.** **MSD-003's verification step 7
  opens with a one-line check that settles availability**, and the answer decides whether MSD-003 has
  a deployable form at all; the step then goes on to a second query that measures how much the
  fingerprint moves on its own.
- **Neither produces an alert: MSD-007 and MSD-008.** MSD-007's step 2 emits an audit stream rather
  than an alert, and it is still the step the Deploy-as column above tells you to deploy once step 1
  has been read; MSD-008 ships no narrowing filter because Microsoft publishes no value set for it.
  **Read this bucket as what the two emit rather than as whether to deploy them**, which is what the
  Deploy-as column is for.

**Three primary queries detect nothing** - MSD-003, MSD-004 and MSD-006 are inventory and posture
queries that rank candidates for human review. Their change-detection variants alert on a
transition, which is a different job from detection, and the bucket above is about those variants
rather than about the census queries above them. That is a reasonable shape for a v0.1 built without
workspace access, which is how every query here was first written, and it is not what "eight
detections" implies on its own.

## Three things to know before you read the queries

### 1. There is no documented advanced-hunting surface for Microsoft 365 Copilot chat prompts

This is an assumption a coverage map can easily get wrong, so it is stated first. Across the seven
Microsoft Learn pages listed in
[`docs/verification-methodology.md`](docs/verification-methodology.md) section 3.1, **no table or
column documents Microsoft 365 Copilot user prompts.** Two of those seven carry
the weight: the advanced hunting overview contains zero occurrences of "Copilot" and zero of
"prompt", and the schema table list - which enumerates every native Defender XDR advanced hunting
table - contains one occurrence of "Copilot", referring to Copilot **Studio** agents.

Three things bound that claim. It is about the native Defender XDR schema. Advanced hunting in the
Defender portal can also query a connected Microsoft Sentinel workspace, where Microsoft says "you
can find many of that workspace's tables"
([advanced hunting with Microsoft Sentinel data](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-microsoft-defender),
read 2026-09-27) without naming `CopilotActivity`; that Sentinel table carries Copilot interaction
audit records, though not documented prompt text
([`CopilotActivity` table reference](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/copilotactivity),
re-read 2026-09-27). And queries published outside Microsoft filter `CloudAppEvents` on
`ActionType == "CopilotInteraction"`, a value no Microsoft Learn page read for this pack documents
for that column: the
[`CloudAppEvents` reference](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-cloudappevents-table),
re-read 2026-09-27, lists no `ActionType` values, and no page the Microsoft Learn site search
returned for the two terms on that date carries both.

What exists is **two adjacent surfaces on different release states**, and this pack builds on both:
prompt injection carried in *email* through Defender for Office 365 (which does reach advanced
hunting, and carries no preview qualifier), and *AI-agent* telemetry (explicitly public preview).

That claim is bounded to the pages listed in
[`docs/verification-methodology.md`](docs/verification-methodology.md). It is strong evidence, not
proof of absence across Microsoft's whole documentation set.

### 2. Defender for Cloud AI threat protection covers Azure-hosted workloads

Per Microsoft Learn, the supported services are Azure OpenAI supported models and Azure AI Model
Inference service supported models. Neither "Microsoft 365 Copilot" nor "GitHub Copilot" appears
anywhere on that page. Note the shape of that evidence: the page **enumerates what it supports** and
neither Copilot is listed. It does not state an exclusion.

### 3. The Sentinel Copilot connector's release state is contested by Microsoft's own page

A page-level notice says all Sentinel data connectors are currently in Preview. The same page
suffixes only some entries "(Preview)", and the Microsoft Copilot entry carries none. Both signals
cannot be authoritative. MSD-007 is therefore labelled **Requires further validation** and will stay
there until Microsoft's documentation stops contradicting itself.

## How to use this

1. Read [`docs/verification-methodology.md`](docs/verification-methodology.md) so you know what the
   status labels are worth.
2. Work through
   [`checklists/workspace-verification-checklist.md`](checklists/workspace-verification-checklist.md)
   in your own workspace. **This is not optional.** Every detection in this pack carries at least
   one element Microsoft does not document, and **none of the eight can separate a clean environment
   from a broken filter without a lab-generated positive control.** For five of them - MSD-003,
   MSD-004, MSD-005, MSD-007 and MSD-008 - that gap is widest, **a judgement about how much else
   there is to fall back on when the result comes back empty rather than a category derived from a
   rule.** [`docs/verification-methodology.md`](docs/verification-methodology.md) section 6 is the
   canonical statement of that judgement; if the two ever disagree, that one governs.
3. Fill in the placeholder value sets in **MSD-006 and MSD-008** from your own observed data.
   **A rule deployed with a placeholder in place returns nothing and looks healthy.** *(MSD-007
   ships a discovery query but no placeholder - its narrowing queries are complete.)*
4. Deploy the ones that verified. Record the ones that did not, and why.

## What is out of scope

See [`docs/scope-and-out-of-scope.md`](docs/scope-and-out-of-scope.md) for the full statement.
Briefly: Microsoft 365 Copilot chat-prompt hunting, anything not backed by a public primary source,
tenant-specific values beyond the single labelled observed value section 1 of the methodology names,
automated deployment tooling, and any detection built on a table this
author could not verify against Microsoft Learn.

## Relationship to the capability-status matrix

This repository is the deferred half of a pair.
[`microsoft-ai-security-control-plane`](https://github.com/topazyo/microsoft-ai-security-control-plane)
maps Microsoft AI-security capabilities to their release status with primary-source citations, and
explicitly scopes runnable detections out. This pack is that deferral discharged: **the matrix's**
rows 4, 9, 10, 11 and 13, and its cross-walk ATLAS column, are this pack's seed. Those row numbers
refer to `microsoft-ai-security-control-plane` **v0.1.2**, read 2026-08-15; row numbers shift
between versions, so check the version before following them.

Read them together. The matrix tells you whether a capability exists and in what state; this pack
tells you what you can query and what you cannot see.

## How each detection was verified

Every element traces to a named page and a date. See
[`docs/verification-methodology.md`](docs/verification-methodology.md), which also records - in its
own section - what verification did **not** include.

**The author has worked through this checklist three times, on 2026-08-24, 2026-08-26 and
2026-09-11**, against a Microsoft Sentinel workspace and Microsoft Defender XDR advanced hunting.
**This pack does not identify any environment it ran against.** **No run has
completed the checklist, so the release condition the checklist sets is not met.** What they did not
reach goes wider than the queries: the steps asking for a generated event, the `AgentsInfo` steps,
the `BehaviorEntities` join, the plan checks and the `SecurityAlert` identifier question are all
recorded as not completed on at least one run. Of the queries submitted, two detections could not
resolve their table and one could not resolve a column.
[`CHANGELOG.md`](CHANGELOG.md) records the three run dates, and the corrections those runs produced
are applied here. **No detection here has yet been observed firing.** Every query remains a
schema-verified construction rather
than an observed result. That is why the checklist is still a release gate, and why three incomplete
runs do not close it.

## Framework mappings

[`crosswalk/detection-framework-crosswalk.md`](crosswalk/detection-framework-crosswalk.md) maps each
detection to MITRE ATLAS technique IDs and OWASP LLM 2026 items at item level. ATLAS IDs were read
from MITRE's distributed `atlas-data` dataset (`version: 5.6.0`), not recalled. The mapping is the
author's synthesis, MITRE and OWASP did not produce it, and the cross-walk names its own weakest
cell.

## Confidentiality

Every file in this repository is built from public primary sources, and carries nothing on the
confidentiality exclusion list in
[`docs/scope-and-out-of-scope.md`](docs/scope-and-out-of-scope.md). **Run outcomes are published
about this pack's own environment rather than withheld**, and that file says why they are not on the
list. **They are not the only category here that describes the author's own environment rather than
a reader's**: the same file points at an observed-value class set out in
[`docs/verification-methodology.md`](docs/verification-methodology.md), which a shipped query acts
on, and at further values a tenant emitted that no query acts on.

**That list is a rule for contributions too**, and it is enforced rather than requested. An issue or
comment containing any of it will be **deleted rather than edited**, because editing leaves the
content in the edit history. **Deletion is not retraction either**: an issue is public the moment it
exists, so deleting it reduces exposure rather than undoing it. **For a pull request, a commit message or a branch name there is no
equivalent remedy**: the diff lives under refs nobody can rewrite, so such a pull request is closed
unmerged and the content stays public. That asymmetry is the reason to check before you open one
rather than after. If explaining a finding requires environment detail, use the private route in
[`SECURITY.md`](SECURITY.md) instead of a public issue.

## Disclaimer

See [`disclaimer.md`](disclaimer.md). In short: statuses change, preview surfaces change faster,
and every query here must be validated in your own environment before you rely on it.

## Licence

**Everything in this repository that is the author's own work is licensed MIT**, under
[`LICENSE`](LICENSE) - the KQL and the prose around it alike. One licence rather than two, because
the queries and the documentation are interleaved inside the same markdown files: a boundary drawn
between them would have to be adjudicated fence by fence, and a practitioner pasting a query into
an analytics rule needs a notice requirement they can actually satisfy where the query ends up.

**Four classes of material are not covered by that grant:** quoted Microsoft Learn text, including
the table, column and value descriptions reproduced verbatim in the detection files; Microsoft
product names, table names, column names, alert names and alert identifiers; MITRE ATLAS technique
identifiers and titles; and OWASP Top 10 for LLM Applications item identifiers and titles. They
remain their publishers', are reproduced under their own terms, and are cited to a source and a read
date in the Sources section of the file that establishes each one. They are not the author's to
re-license, and an unqualified grant over "documentation content" would have read as exactly that.
[`LICENSE`](LICENSE) carves out the same four classes in the same terms. **Alert names and alert
identifiers are named rather than folded into "quoted text" because MSD-005 reproduces 17 of each**,
which is the largest single block of other people's material in this pack.

> **One thing worth knowing before you reuse the quoted material.** Microsoft's documentation is not
> published under a single licence, and which licence reaches a quotation depends on which repository
> the page is published from. Four repositories are in play here. Each licence file below was read on
> 2026-08-17:
>
> - [`MicrosoftDocs/defender-docs`](https://github.com/MicrosoftDocs/defender-docs) - `LICENSE` is
>   the **MIT License**.
> - [`MicrosoftDocs/entra-docs`](https://github.com/MicrosoftDocs/entra-docs) - `LICENSE` is the
>   **MIT License**.
> - [`MicrosoftDocs/azure-monitor-docs`](https://github.com/MicrosoftDocs/azure-monitor-docs) -
>   `LICENSE` is **Creative Commons Attribution 4.0 International**, and `LICENSE-CODE` is the
>   **MIT License**. **This is the repository the `AADServicePrincipalSignInLogs`,
>   `CopilotActivity` and `SecurityAlert` table references are published from.**
> - [`MicrosoftDocs/dataexplorer-docs`](https://github.com/MicrosoftDocs/dataexplorer-docs) - the
>   same split, `LICENSE` **CC BY 4.0** and `LICENSE-CODE` MIT. **This is the repository the Kusto
>   Query Language reference pages are published from**, and this pack cites several of them.
>
> A URL path does not name the repository. Several of these pages sit under
> `learn.microsoft.com/en-us/azure/`, and none of them is published from `azure-docs`: the Defender
> for Cloud pages and the Microsoft Sentinel connector reference come from `defender-docs`, and the
> Azure Monitor table references from `azure-monitor-docs`. Each repository above was identified from
> that repository's own file tree rather than from the URL.
>
> Separately, the rendered `learn.microsoft.com` Terms of Use carry a personal and non-commercial use
> limitation that mentions no open licence at all. A repository licence covers the source markdown
> rather than settling the terms of the rendered page, and that is the part this note cannot resolve
> for you. **Check the terms that apply to your own reuse rather than inheriting an assumption from
> here.**

## Contributing

Corrections are the most valuable contribution here, and a schema mismatch found in a real workspace
is worth more than a new query. Every proposed change must carry a primary-source URL, the date that
source was read, and a status label from the canonical legend in
[`docs/verification-methodology.md`](docs/verification-methodology.md) section 2, which the table
above reproduces.
