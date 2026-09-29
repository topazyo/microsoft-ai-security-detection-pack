# Scope and out of scope - v0.1

Scoping this deliberately is the point. A detection pack that spread across every Microsoft AI
surface would rest on schema its author could not verify, and unverified schema is worse than no
schema for the audience this pack is for.

---

## In scope for v0.1

- **Eight detections** across six documented tables, split by deployment target: Microsoft Defender
  XDR advanced hunting, and Microsoft Sentinel Log Analytics.
- **Per detection:** the exact table and columns with their data types as this pack read them, a
  Microsoft Learn citation URL, a last-verified date, a release-status label, MITRE ATLAS technique
  IDs where a technique genuinely applies, one or more OWASP LLM 2026 items, false-positive
  guidance, an explicit statement of what the detection cannot see, and workspace verification
  steps. MSD-004 carries no ATLAS technique and says why; MSD-005 carries six.
- **A framework cross-walk** at item level, with ATLAS IDs read from MITRE's distributed dataset.
- **A verification methodology** that records what was checked and, in its own section, what was
  not.
- **A workspace verification checklist** that is a release gate rather than a suggestion.

## Out of scope for v0.1

Each of these was considered and excluded on purpose, except the last two, which record gaps found
after this scope was set. None is an oversight.

### Microsoft 365 Copilot chat-prompt hunting

**No detection here assumes such a surface exists.** Across the pages listed in
[`verification-methodology.md`](verification-methodology.md), no table or column documents Microsoft
365 Copilot user prompts. Two adjacent surfaces do exist - email-borne prompt
injection through Defender for Office 365, and AI-agent telemetry - and the pack builds on both.
It does not extrapolate from either to a chat-prompt surface.

The Sentinel `CopilotActivity` table carries Copilot *audit* activity and is used for configuration
monitoring in MSD-007. Microsoft does not document that its `LLMEventData` column carries prompt
text, and MSD-007 says so in its opening lines.

### Anything not backed by a public primary source

No detection is built on a product blog, a launch announcement, a conference talk, a community post,
or recalled schema. Microsoft Tech Community is treated as official-adjacent context and can never
set a status here.

### Tenant-specific content of any kind

No captured telemetry, no screenshots, no sample data drawn from a real environment, and no
environment-specific values in any query **except the single member of the observed-value class**
that [`verification-methodology.md`](verification-methodology.md) section 1 names. That one value
is labelled as observed at both of the points where a query uses it, which is the condition the
class carries; nothing else of the kind ships here.

### Detections on tables verified only partially

Two surfaces were read during this pass and deliberately not built on:

- **`CloudAppEvents` for AI-agent activity - and this is the pack's largest structural gap, not a
  tidy deferral.** Microsoft documents it as carrying "Agent 365 observability data for AI agent
  activity, including agent actions, tool invocations, and data access events", in the
  advanced-hunting table list on the
  [AI agent detection and protection page](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/ai-agent-detection-protection),
  read 2026-08-23. **That sentence is not on the `CloudAppEvents` table reference cited below**,
  which is this section's source for the six identifiers and not for this description. That makes it **the
  only documented surface in this pack's evidence set that records what an agent did.** `BehaviorInfo`,
  which MSD-008 reads, carries runtime records too, but they are protection audit and block events -
  what a control did about an agent, not what the agent did. Microsoft documents five `ActionType`
  values for agent activity in that table: "`ActionType` reflects the operation (`InvokeAgent`,
  `InferenceCall`, `ExecuteToolBySDK`, `ExecuteToolByGateway`, `ExecuteToolByMCPServer`) and the
  per-span fields are inside `RawEventData`", on the
  [Agent 365 observability concepts page](https://learn.microsoft.com/en-us/microsoft-agent-365/developer/observability-concepts),
  `ms.date` 2026-09-02, read 2026-09-27. That page does not say which span type produces which
  value, for example which of the three `ExecuteTool*` values an `execute_tool` span yields.

  > **Correction, 2026-09-27.** An earlier version of this passage said `CloudAppEvents`
  > `ActionType` values for agent activity are not documented, so a narrowing filter would be a
  > guess. The page above documents five. `CHANGELOG.md` is the record.

  **The argument against the deferral is accepted, and it is recorded here rather than answered.**
  MSD-008 presents the same problem, and this pack solves it there by shipping a discovery query with
  no narrowing filter and asking for a positive control. As it stands, **two detections tell you what
  agents are configured to do - MSD-003 and MSD-004 - and none tells you what any agent did.**
  Refusing to apply the pack's own pattern here is inconsistency rather than discipline, and calling
  it discipline would be the more comfortable of the two descriptions rather than the accurate one.

  **It is deferred to v0.2 anyway, and the reason is the cost of the change rather than the merits
  of the argument.** A ninth detection moves every inline count in this pack: the eight-detection
  figure wherever this pack states it, the two cross-walk tables a detection appears in, the
  positive-control list, the discovery-first trio. Each of those is a claim this pack has to keep
  true, and moving all of them at once, to add a query that would itself still be waiting on a
  positive control, is the wrong order to do the work in. The query is drafted below so that the
  decision stays a small one whenever it is taken. Every column identifier in it is in the
  documented `CloudAppEvents` schema, the other names in it being the query's own `summarize`
  output, `Events`, `Accounts`, `FirstSeen` and `LastSeen`. This file cites that schema itself
  because it is the only place in the pack that prints a runnable `CloudAppEvents` query:

  ```kusto
  // v0.2 candidate - MSD-009 discovery. No narrowing filter, by design.
  CloudAppEvents
  | where Timestamp > ago(30d)
  | summarize Events = count(), Accounts = dcount(AccountObjectId),
      FirstSeen = min(Timestamp), LastSeen = max(Timestamp)
      by ActionType, Application, ActivityType, AuditSource
  | order by Events desc
  ```

  Source for those six identifiers, read 2026-08-19:
  [`CloudAppEvents` table, Microsoft Defender XDR advanced hunting
  reference](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-cloudappevents-table).

  | Column | Data type | Learn description |
  |---|---|---|
  | `Timestamp` | `datetime` | Date and time when the event was recorded |
  | `AccountObjectId` | `string` | Unique identifier for the account in Microsoft Entra ID |
  | `ActionType` | `string` | Type of activity that triggered the event |
  | `Application` | `string` | Application that performed the recorded action |
  | `ActivityType` | `string` | Type of activity that triggered the event |
  | `AuditSource` | `string` | Audit data source, with three values published beneath it as a list |

  **That header does not say "verbatim", and the reason is `AuditSource`.** Its description is
  followed on the page by a bulleted value list, and flattening that list into a table cell would be
  this pack's reformatting rather than Microsoft's text. The three values it publishes are Defender
  for Cloud Apps access control, Defender for Cloud Apps session control, and Defender for Cloud Apps
  app connector. The other five descriptions are as published. **`ActionType` and `ActivityType`
  carry the identical published description**, so which of the two a narrowing filter should key on
  is not settled by the reference, which is one more reason the query above ships as discovery. For
  agent activity, the Agent 365 observability concepts page places the operation in `ActionType`.

- **`BehaviorEntities`.** Named by Microsoft as a companion to `BehaviorInfo`, and named in MSD-008
  for that reason. Its reference page was read for the table's description and status, which MSD-008
  quotes. **No query here joins to it.** Its published column set has been checked against a
  workspace once, in one environment, and three columns were read; nothing further about it has
  been, which is what "verified only partially" means for this table.

### Automated deployment tooling

No ARM templates, no Bicep, no Sentinel repositories pipeline, no `deploy-to-azure` buttons in v0.1.
Deployment automation for detections that have not been verified in a workspace would encourage
exactly the behaviour this pack is arguing against.

### Detection tuning thresholds

No detection here ships a numeric threshold as a recommendation. The time windows in the queries are
starting points, stated as such. Thresholds require measurement in a specific environment, and this
pack has no measurements.

### Coverage scoring

No maturity score, no coverage percentage, no heat map. Any such number would be a claim about a
denominator nobody has, and the **primary** queries of this pack's three inventory and posture
detections detect nothing at all - which any scoring scheme would obscure.

### Non-Microsoft AI security telemetry

Third-party model gateways, non-Microsoft agent platforms, and non-Microsoft SIEMs are out of scope.

### Response and remediation

This pack detects and scopes. Playbooks, automated response, and containment guidance are not
included.

### GitHub Copilot and Secure SDLC detection content

The companion capability-status matrix covers GitHub Copilot policy management and enterprise MCP
governance as *controls*. Neither has a documented detection surface identified in this pass, so
neither produces a detection here.

### Microsoft Purview, in its entirety

**Named because its absence is otherwise silent, and silence is the defect.** No DSPM for AI, no DLP
for Microsoft 365 Copilot, no sensitivity-label or oversharing surface appears anywhere in this
pack, even though those are core to the positioning this body of work sits inside and the companion
matrix carries four Purview rows.

The reason is the no-invented-schema constraint: **no Purview detection surface was verified in this
pass**, so shipping a detection would breach the pack's first rule. That is a defensible reason not
to ship one and not a reason to leave the gap unstated. A reader building an AI-security coverage
map from this pack would otherwise conclude that Purview has no detection surface, which is a claim
this pack has not established and does not make.

### Shadow-AI and unsanctioned generative-AI app discovery

Defender for Cloud Apps generative-AI app discovery is a documented capability - the companion
matrix carries it as a GA row - and it is the second thing a reader of a Microsoft-stack AI-security
detection pack is likely to look for. **No detection here covers it.** Its discovery data does not
land in the advanced hunting tables this pack verified, and the Sentinel-side surface was not
verified in this pass. A v0.2 candidate, recorded rather than left implicit.

### Local AI agent posture

Recorded on 2026-09-27, when this pack first read the page cited here, so it is a coverage gap in
MSD-003 and MSD-004 rather than one of the exclusions above that were weighed when this scope was
set.
Defender for Endpoint discovers local AI agents on onboarded devices and reports them in
`AgentsInfo`, where Microsoft's own queries select them with `Platform == "LocalAgents"`. It states
that "Local AI agent posture is nested in the `RawAgentInfo` column, under `localAgentMetadata`",
including whether an agent acts without prompting the user for approval and the local MCP servers it
uses, and that "Local MCP servers are reported only in `AgentsInfo`"
([Discover local AI agents with Microsoft Defender for Endpoint](https://learn.microsoft.com/en-us/defender-endpoint/discover-local-ai-agents),
`ms.date` 2026-09-16, read 2026-09-27). MSD-003 and MSD-004 do not read `RawAgentInfo`, so no
detection here covers that posture.

### Foundry runtime traces

Recorded on 2026-09-29, so it is also a gap found after this scope was set. Microsoft states that
"Microsoft Foundry traces can capture sensitive information such as prompts, model responses,
system instructions, and tool calls", lists access to "the Application Insights resource connected
to your project" among the prerequisites for protecting them, and names September 30, 2026 as the
migration date for routing that content to a dedicated table, with a temporary opt-out that is
discontinued on September 30, 2027
([Restrict access to sensitive content in Microsoft Foundry traces](https://learn.microsoft.com/en-us/azure/foundry/observability/how-to/traces-sensitive-content),
`ms.date` 2026-07-21, read 2026-09-29). That table is `AppGenAIContent`: "Generative AI content
captured from an OpenTelemetry source, including input and output messages, system instructions,
and tool interactions"
([Azure Monitor Logs reference - AppGenAIContent](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/appgenaicontent),
`ms.date` 2026-07-27, read 2026-09-29). No detection here reads Application Insights telemetry or
`AppGenAIContent`. MSD-005 reads Defender for Cloud alerts and MSD-008 reads protection behaviours,
and neither is a trace of what a Foundry agent was asked or did.

---

## Confidentiality exclusions

Built entirely from public primary sources. **Never include** in this repository:

- organisation names, or project and system names used privately
- non-public domain names or URLs
- IP addresses, hostnames
- tenant, workspace, or subscription GUIDs
- user names or email addresses
- captured logs, telemetry, or result rows and counts from a real environment
- screenshots of any kind
- real incident details
- vendor evaluation findings, commercial details, licence counts, or cost data
- undisclosed security gaps or policy exceptions
- material under NDA, or from a Microsoft preview programme that is not publicly documented

**Run outcomes are not on that list, and the omission is deliberate rather than an oversight.**
What a run established is published throughout this pack: whether a table or a column resolved,
what shape a function returned, which serialisation a column emitted, and a value a tenant emitted
where no page this pack cites publishes one. **That is a class of its own and it is not a result
row, a row count, or anything inside one**, which is what the bullet above bars. It is published
because the hedges in every detection file rest on it and because the release gate is stated in
terms of it. **It is not the only thing here that describes the author's own environment rather
than a reader's**: `docs/verification-methodology.md` sets out an observed-value class that a
shipped query acts on, and reports further values a tenant emitted that no query acts on.

**The asymmetry the issue templates set out is about identification, not about outcomes.** This
pack does not identify any environment it ran against and it does not ask a contributor to identify
theirs. **It does ask a contributor for outcomes about their own environment**, directly and in the
forms: whether a query ran, whether a table or a connector or a plan was there, and which
serialisation a column emitted. `.github/ISSUE_TEMPLATE/workspace-verification-report.md` states
the same point where a contributor meets it.

**The rule about example values is stated here as a property rather than as a list of sites**, because
a list of sites goes stale on the next query edit. **No example value
anywhere in this pack carries anything from anyone's environment, and that property is what the
boundary rests on** rather than any count of where the literals sit.

The classes those values fall into are set out below rather than summarised, because a summary would
fix their number and this pack ships values that sit outside any fixed set of classes. **Each class
is tagged twice**: whether it sits inside the rule in
[`verification-methodology.md`](verification-methodology.md) section 1, which is what that section's
pointer here is about, and whether an inbound contributor may send one, which is what the
contribution templates' example-value attestation is about. **Those are different questions**, and
only the first class answers yes to both. **The classes up to the last one are classes of value this
pack ships, and their second tag answers the inbound question about that value rather than about
everything a contributor might hold. The last one is the class a contributor originates**, and it is
here because a taxonomy of what the pack ships cannot on its own answer a question about inbound
content.

- **Inside section 1's rule; a contributor may send one.** A value Microsoft publishes on a page
  this pack cites.
- **Outside the rule; a contributor may send one.** An obvious placeholder for the reader to fill.
- **Outside the rule; not a contributor's to send.** A literal this pack wrote for a check.
- **Outside the rule; not a contributor's to send.** A value this pack invented and marks as
  invented where it is used. The `ActionType`
  illustration in the verification-report issue template and in [`SECURITY.md`](../SECURITY.md) is
  the case this class exists for. It is deliberately **not** an obvious placeholder: both files say
  it follows the spelling convention of real Defender values precisely because that is the shape a
  report should send.
- **Outside the rule; not a contributor's to send.** A construction shipped inside a predicate and
  offered for the reader to extend, whether bound to a name or written inline. The empty-forms
  list is the case here. `""`, `[]` and `{}` follow from behaviour Microsoft publishes for
  `tostring()` and `isempty()`, the last of them through the published `isempty(parsejson("{}"))`
  case; `null` is this pack's own addition to the list, with no published derivation. Neither the
  `tostring()` nor the `isempty()` reference enumerates the empty forms a platform may emit, which
  is why the checklist tells you to add a fifth form if your platforms emit one. **The tag bars
  sending these literals back as though they were something you observed; it does not reach the
  fifth form you did observe**, which is the class below.
- **Outside the rule; a contributor may send one, and it is the contribution this pack most wants.**
  A value your own workspace emits that no page this pack cites publishes. It is outside the rule
  because no cited page carries it, and it is not in the classes above because this pack does not
  ship it: you are where it comes from. What travels is the value string or the column name and
  never the rows it matched, which is the line the verification report template's own inbound
  attestation draws. **The value string travels only where the column carries a closed set the
  platform defines. Where a column carries free text, the value is itself estate data and this
  class does not reach it**, because there is no value name to send that is not also the content.
  The checklist states that rule in full and names the free-text columns this pack touches. **That
  list and this one name the same columns**, which they have to: `EmailEvents.Subject`,
  `BehaviorInfo.Title`, `BehaviorInfo.Description`, the `SampleLabels` rollup built from whichever
  of those two a tenant carries, `CopilotActivity.LLMEventData`,
  `AADServicePrincipalSignInLogs.Agent`, `AgentsInfo.McpServers`, `AgentsInfo.AgentName` (`Name` on
  two Microsoft pages) and `CopilotActivity.AgentName`. **The two agent-name columns are different
  columns that share a name in this pack's queries**, returned by MSD-003 and MSD-004 on the one
  hand and by MSD-007 on the other, and the test above reaches the same answer for both because
  Microsoft publishes no value set for either.
  **`AgentsInfo.McpServers` is barred outright rather than routed**: Microsoft describes it as
  holding "server URLs and credential configuration", and MSD-003 says never to paste its contents
  anywhere. **A custom analytics-rule name your own workspace emits is your estate's naming**, so
  what MSD-005's verification step 2 returns is a shape to describe rather than a list to send.

Several of the written-for-a-check literals deliberately ship at more than one site: a check stated
in a detection file and the same check in the checklist have to be the same check, so a serialisation
form, a hash input, an operator-test operand or a state label can appear in both, and does. **The
marking rule in the fourth class above is a property of that class rather than of every literal this
pack wrote**: a check operand is not marked as invented at every point of use, and is not meant to be,
because it is not written in the position a documented schema value would occupy.

The operator test uses three distinct left-hand-side strings, each appearing in more than one column
of its statement. Two of the three join two published detection-technology values with a comma, and
the third re-orders the terms of one published value; that third arrangement cannot be assembled from
published values at all. **That is a summary of the checklist's own explanation at the step that runs
the test, which is the canonical copy**, and if the two ever differ the checklist governs.

---

## What would change this scope

| Trigger | Likely v0.2 change |
|---|---|
| `CloudAppEvents` agent `ActionType` values become documented, or a workspace pass enumerates them | An agent tool-invocation detection. The first half of this trigger was met by the [Agent 365 observability concepts page](https://learn.microsoft.com/en-us/microsoft-agent-365/developer/observability-concepts), read 2026-09-27; the deferral stands on the cost of the change, which is its stated reason |
| Microsoft resolves the Sentinel connector labelling conflict | MSD-007's status label moves off Requires further validation, and its existing access-anomaly query becomes defensible to *schedule* rather than to run once |
| A documented hunting surface for Copilot chat prompts appears | A new detection lane, and a correction to the README's opening claim |
| The `BehaviorEntities` schema is verified **against a workspace** beyond the single column-set reading one run recorded, and a join is exercised against a generated behaviour | An entity join in MSD-008 |
| The workspace verification pass finds a schema mismatch | A citation correction, which ranks above any new detection |
