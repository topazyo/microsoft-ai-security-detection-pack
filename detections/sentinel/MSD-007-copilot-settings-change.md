# MSD-007 - Copilot configuration changes in the Sentinel audit stream

| Field | Value |
|---|---|
| **ID** | MSD-007 |
| **Deployment target** | Microsoft Sentinel (Log Analytics) |
| **Primary table** | `CopilotActivity` |
| **Status** | **Requires further validation** - the connector's release state is contested by Microsoft's own page |
| **Last verified** | 2026-08-15 |
| **MITRE ATLAS** | `AML.T0081` Modify AI Agent Configuration · `AML.T0012` Valid Accounts |
| **OWASP LLM 2026** | LLM03:2026 Excessive Agency *(the weakest mapping in this pack - see the cross-walk)* |
| **Requires** | The Microsoft Copilot logs data connector |

## Read this before anything else

**This table is an audit stream, not prompt content.** Microsoft Learn describes `LLMEventData` as
"Parsed LLM event data (for copilot different RecordTypes)" and **does not document that it carries
prompt text**.

**Do not build prompt-injection detection on this table.** There is no documented advanced-hunting
surface for Microsoft 365 Copilot chat prompts - a claim scoped to the pages listed in
[`docs/verification-methodology.md`](../../docs/verification-methodology.md) section 3.1 - and this
table is a Sentinel Log Analytics audit stream rather than an advanced-hunting table in any case.
A pack that treated it as a prompt-hunting surface would be wrong in exactly the way this pack
exists to avoid.

What this table supports is configuration and access monitoring: who changed Copilot settings, from
where, and when.

## Status - why this row is not labelled GA or Preview

The Microsoft Sentinel data-connectors reference contradicts itself, and the conflict was
**re-verified live on 2026-08-15 and still stands**:

- A page-level notice states, verbatim: "Note that Microsoft Sentinel data connectors are currently
  in Preview. The Azure Preview Supplemental Terms include additional legal terms that apply to
  Azure features that are in beta, preview, or otherwise not yet released into general
  availability." That is a blanket claim over the whole class.
- The **same page** tags individual connector entries "(Preview)". **Counting the literal
  `(Preview)` over the rendered page on 2026-08-23 gives six occurrences: five sit in a connector
  entry's own title, four at the end of that title and one before a further parenthetical, and the
  sixth sits inside a numbered setup step in another entry's body. So five entries carry the tag**,
  and the instrument is named here because the two figures differ and a reader re-running Group 7
  has to know which one they are reproducing. If the blanket notice were operative, per-entry
  labelling would be redundant.
- The **Microsoft Copilot** entry carries **no** such tag, and its body contains no release-state
  sentence at all.

The two signals cannot both be authoritative, and which one governs decides whether this connector
is GA or Preview. That is not resolvable from the page, which is what **Requires further
validation** means here.

**On a coverage map, record it as unresolved rather than as either label.** For *planning* purposes
assume preview - not because the evidence favours it, but because preview terms and the absence of
a service-level commitment are the safer assumption when the evidence is ambiguous, and being wrong
in that direction costs you nothing. Verify in your own workspace. **Do not record it as GA**, and
do not let the planning assumption harden into a status.

Separately: the `CopilotActivity` table reference on Microsoft Learn carries **no release-state
label of any kind**.

> **Scope judgement, not a quotation.** The connector is named "**Microsoft Copilot**" and its
> description spans Microsoft Copilot **and Security Copilot**. Matching it to a Microsoft 365
> Copilot control is therefore a judgement. Use the `Workload` column to separate the products, and
> confirm which values your own tenant emits.

## Schema this depends on

Columns quoted from the `CopilotActivity` table reference on Microsoft Learn, read 2026-08-15
(page stamp 2026-07-28, which is the rendered date; the source file's own `ms.date` reads 2026-07-27,
and these generated reference pages carry both). Table description, verbatim: "Audit logs for Copilot
and other AI workloads. Extensible for future AI audit types."

| Column | Data type | Learn description (verbatim) |
|---|---|---|
| `TimeGenerated` | `datetime` | Timestamp of the audit event. |
| `RecordType` | `string` | Normalized record type name (e.g., CopilotInteraction, UpdateCopilotSettings). |
| `RecordId` | `string` | Unique identifier for the audit record. |
| `ActorName` | `string` | User principal name or email address. |
| `ActorUserId` | `string` | Internal user key or GUID. |
| `ActorUserType` | `string` | Type of user (e.g., Regular, Admin, System). |
| `AppHost` | `string` | Application that hosts copilot. |
| `AppIdentity` | `string` | Identity of the application hosting the copilot interaction. |
| `Workload` | `string` | The workload or product (e.g., Copilot, AzureOpenAI). |
| `AgentId` | `string` | The version number or version ID of the agent involved. |
| `AgentName` | `string` | A friendly readable name of the agent. |
| `AIModelName` | `string` | Name of the AI model used (for extensibility). |
| `AIModelVersion` | `string` | Version of the AI model used. |
| `SrcIpAddr` | `string` | IP address of the client. |
| `ClientRegion` | `string` | Region of the client. |
| `LLMEventData` | `dynamic` | Parsed LLM event data (for copilot different RecordTypes). |
| `LogVersion` | `string` | Version of the LLM log format. |
| `Version` | `string` | Version of the audit schema or event. |

> **The `RecordType` value set is not enumerated.** Learn gives two examples behind an "e.g." -
> `CopilotInteraction` and `UpdateCopilotSettings` - and nothing more. The shipped query is built to
> work without knowing the full set.

> **`AgentId` is described as "The version number or version ID of the agent involved."** That
> description does not match the column name, and this pack does not resolve the discrepancy. Do not
> join it to `AgentsInfo.AgentId` without confirming they hold the same thing.

### Table plan constraints, from Learn's own attribute table

| Attribute | Value |
|---|---|
| Categories | Security, Audit |
| Solutions | SecurityInsights |
| Basic table support | Yes |
| Auxiliary / Lake table support | Yes |
| DCR workspace transformation support | **No** |
| Ingestion API support | No |

"Basic table support: Yes" records that the table **can** be set to that plan, not that it is. If
your workspace puts this table on a Basic or Auxiliary plan, confirm what a scheduled analytics rule
can do against that plan before deploying. The connector reference separately records "Data
collection rule support: Not currently supported", which agrees with the "No" above.

## Query

```kusto
// MSD-007 step 1 - discovery. What record types does your tenant actually emit?
// Deployment target: Microsoft Sentinel (Log Analytics).
// Schema verified against Microsoft Learn on 2026-08-15.
// Connector release state is contested - see the Status section.
CopilotActivity
| where TimeGenerated > ago(30d)
| summarize
    Events = count(),
    Actors = dcount(ActorName),
    FirstSeen = min(TimeGenerated),
    LastSeen = max(TimeGenerated)
    by RecordType, Workload, AppHost, ActorUserType
| order by Events desc
```

```kusto
// MSD-007 step 2 - everything that is not an interaction record.
// Written as an exclusion because Learn does not enumerate the RecordType value set:
// a new administrative record type appears here without a rule change.
CopilotActivity
| where TimeGenerated > ago(7d)
| where RecordType != "CopilotInteraction"
| project
    TimeGenerated,
    RecordType,
    ActorName,
    ActorUserType,
    AppHost,
    AppIdentity,
    Workload,
    SrcIpAddr,
    ClientRegion,
    AgentName,
    AIModelName,
    RecordId
| order by TimeGenerated desc
```

```kusto
// MSD-007 step 3 - the documented settings-change record type specifically.
CopilotActivity
| where TimeGenerated > ago(7d)
| where RecordType == "UpdateCopilotSettings"
| summarize
    Changes = count(),
    Hosts = make_set(AppHost, 10),
    SourceIPs = make_set(SrcIpAddr, 10),
    LastSeen = max(TimeGenerated)
    by ActorName, ActorUserType, Workload
| order by Changes desc
```

**Run step 1 once as discovery, then deploy step 2.** Step 3 is narrower and will miss any
administrative record type Microsoft has not documented, so it is a pivot rather than the rule. The
README's index carries the same order in its Deploy-as column.

**Step 3's `==` and step 2's `!=` are both case-sensitive, and Microsoft publishes each on its own
reference page.** The `!=` page opens "Filters a record set for data that doesn't match a
case-sensitive string", and both pages carry the same comparison table marking `==` and `!=` as
case-sensitive. Both compare against `"CopilotInteraction"` or `"UpdateCopilotSettings"` exactly as
written, so a workspace emitting a different casing for either value makes step 3 match nothing and
makes step 2 stop excluding. Step 1 is what tells you the casing your own tenant uses, which is one
more reason to run it first and to read its `RecordType` inventory before deploying either.

**What every query in this file returns stays in your environment, and none of it is `LLMEventData`,
which the workspace verification section handles separately.** Step 3 and the access-pattern view
group named actor principals beside a set of client network addresses, which are two of the
categories this pack's own exclusion list names, materialised side by side. **Step 2 is the query
this file tells you to deploy, and it carries more than either rollup does**: it projects those same
actor principals and client addresses one row at a time rather than grouped, and adds the client
region, the application identity, the record identifier, the agent name and the AI model name, so a
single row of its output identifies more than a whole row of either rollup. **`AgentName` is free
text on the pack's own test**: Learn describes it as a friendly readable name of the agent and
publishes no value set for it, so the value is a name from your own estate rather than a class the
platform assigns, and what travels from it is the column name and the shape rather than the
contents. MSD-003 and MSD-004 apply the same reasoning to the same-named column in `AgentsInfo`.
**That classifies `AgentName` and nothing else here**; the note on `AgentId` above stands unchanged.
**The rule governs step 1 as much as the rest**, and step 1 is the step a reader is most likely to
think it does not, because its output reads as an inventory rather than as rows: its event and
distinct-actor figures are counts, and its first-seen and last-seen timestamps are estate data in
the same way, so all four stay with you. **What travels from this file is a closed list: the
`RecordType`, `Workload` and `AppHost` value strings step 1 reports, which are the three the
checklist's Group 7 step asks you to record and compare, together with the `RecordType` and field
names verification step 3 permits from `LLMEventData`, and nothing else.** Learn enumerates none of
those columns' values, which is why the inventory is worth having and why the value names rather
than the figures beside them are the contribution. **`AppHost` goes out under the checklist's
condition for a column Learn gives neither an example nor a value set, rather than as a plain value
name**: Learn describes it as the application that hosts copilot and settles neither way whose
applications it names, so look at what your own values contain before any of them travels, and send
the shape rather than the value where they name anything specific to your estate, whoever chose the
name. **The same condition gives the set its own rule**, because a complete inventory of the
applications hosting Copilot in your tenant states part of what your estate runs even where every
member of it passes the per-value test: that inventory stays in your environment, and what may
travel from this column is an individual value rather than the list step 1 reports. The checklist
states the per-value test and the set rule in full, under **Recording the result**. The list above
does not shorten; the condition is what reaches it. **Not `ActorUserType`, which step 1 also groups
by** - the checklist's step does not ask for it, so nothing here asks you to send it. **That is the
reach of this file's permission rather than a published prohibition**: Learn illustrates that
column's values behind an "e.g." exactly as it does `RecordType`'s, so no published distinction
separates them. What travels outward is a column name or a value name, never a result row, a row
count, or anything inside one. Checklist Group 0 states the same rule for every step in the pack,
and this file repeats it because a reader who deploys one detection may never open the checklist.

### Access-pattern view

```kusto
// Copilot activity from an unusual client region for a given actor, over a 30-day baseline.
let baseline =
    CopilotActivity
    | where TimeGenerated between (ago(31d) .. ago(1d))
    | distinct ActorName, ClientRegion;
CopilotActivity
| where TimeGenerated > ago(1d)
| join kind=leftanti baseline on ActorName, ClientRegion
| summarize Events = count(), SourceIPs = make_set(SrcIpAddr, 10), LastSeen = max(TimeGenerated)
    by ActorName, ClientRegion, Workload, AppHost
| order by Events desc
```

## What this detection cannot see

- **Prompt content, and therefore prompt injection.** Stated at the top of this file because it is
  the assumption most likely to be made. `LLMEventData` is not documented as carrying prompt text.
- **Microsoft 365 Copilot chat interactions in a form that supports hunting the prompt.** The
  `CopilotInteraction` record type exists, but Learn documents neither its contents nor the shape of
  `LLMEventData` for it.
- **Any Copilot surface not covered by the Office Management API.** Per the connector reference:
  "This connector uses the Office Management API to get your Microsoft Copilot audit logs."
- **Which product a row belongs to, without reading `Workload`.** The connector spans Microsoft
  Copilot and Security Copilot.
- **Anything, if the connector is not deployed.** `CopilotActivity` is populated through the
  Microsoft Copilot logs data connector and nothing else, so an absent connector means an absent
  table rather than a quiet one. Other detections here depend on data reaching the workspace from
  elsewhere and fail the same way for different reasons: MSD-005 needs Defender for Cloud alerts to
  be streaming, MSD-006 needs the service-principal sign-in diagnostic category to be exported, and
  MSD-003 and MSD-004 need the Microsoft 365 app connector to be collecting Agent 365 observability
  data, which MSD-003 quotes from Learn. **Confirm the source in each case before reading an empty
  result as a clean one**, rather than counting how many detections that applies to. Verification
  step 1.
- **Connector state itself.** No query in this pack reads whether the connector is configured; the
  table is the only thing these queries see. An empty result therefore does not discriminate a
  connector that was never deployed from one that is deployed and not producing, and no query here
  can be made to. **Record "cannot tell" rather than choosing between them** - the checklist treats
  that as itself the finding - and note that an observation made tenant wide does not settle a
  question about a single workspace. Verification step 1 is what separates a deployment question
  from a production question.

## False-positive guidance

- **Administrative settings changes are routine.** This is change monitoring, not threat detection.
  Alert on the actor, the source region, and the hour - not on the event class.
- **`ActorUserType` values are examples, not an enumeration.** Learn gives "Regular, Admin, System"
  behind an "e.g.", exactly as it does for `RecordType`. A `System` actor is likely to produce
  steady service-activity volume rather than human action, so separating it before counting is
  usually right - but read the values your own tenant emits from step 1 first, rather than filtering
  on a documented example.
- **The `!= "CopilotInteraction"` exclusion is deliberately broad.** It will surface record types you
  have not seen before, which is the point, and it means the first week of results is mostly
  discovery.
- **Region anomalies follow travel and VPN egress**, not only compromise.
- **Absence of results has three explanations and they look identical**: the connector is not
  deployed, the connector is deployed but the audit source is not producing, or nothing happened.
  Verification step 1 distinguishes the first from the other two.

## Workspace verification before deployment

1. Confirm the connector is deployed and the table's columns match this file:
   `CopilotActivity | getschema | project ColumnName, ColumnType`. **Diff that output against the schema
   table above.** Record as a **schema mismatch** any column the schema table lists which the output
   does not carry, or which the output types differently. **A column the output carries and the schema
   table does not is not a mismatch**, because the schema table is a deliberate subset. Then
   `CopilotActivity | take 10`. Per the connector reference the prerequisite is a tenant role -
   "'Security Administrator' or 'Global Administrator' on the workspace's tenant".
   **No licensing or SKU requirement is stated for the connector** - which is silence rather than a
   statement that none applies, so confirm against your own licensing terms rather than reading the
   absence as a guarantee.
2. Run step 1 and record the full `RecordType`, `Workload` and `AppHost` inventory your tenant
   emits. Compare it against the examples Learn gives behind an "e.g." for `RecordType` and for
   `Workload`, note that it gives `AppHost` neither an example nor a value set, and note the
   difference.
3. Inspect `LLMEventData` on a sample of rows and record what it actually contains, per
   `RecordType`. **Do this before writing any query that reads inside it**, and re-read this file's
   opening warning before drawing a conclusion from what you find. **The values you write down from
   `LLMEventData` stay in your environment.** They are your own tenant's data, and Learn describes
   the column without documenting its shape, so nothing tells you in advance what your own rows will
   carry there. What a public issue or a verification report wants from here is the `RecordType` a
   field appears under and the field's name, and nothing about the values inside it.
4. Confirm the table's plan in your workspace (Analytics, Basic, or Auxiliary) and confirm that a
   scheduled analytics rule can run against it on that plan.
5. Make one Copilot settings change deliberately in a lab tenant and confirm it appears with the
   expected `RecordType`. Without a positive control, an empty result is uninterpretable.
   **If it does not appear, the next question is whether the change was audited at all**, and that
   one is answered outside Sentinel. Four things govern that search.
   - **The unified audit log is reachable through Microsoft Graph**, which matters where the
     Exchange Online module is not available on the machine you are working from:
     `microsoft.graph.security.auditLogQuery`, which Microsoft describes as "a query against the
     Microsoft 365 unified audit log", **on the v1.0 endpoint rather than beta**. It takes
     `filterStartDateTime` and `filterEndDateTime`, with filters including `operationFilters`,
     `recordTypeFilters`, `keywordFilter` and `serviceFilters`, and it runs as an asynchronous job
     whose `status` you poll.
   - **Do not assume access carries across from advanced hunting.** Advanced hunting through Graph
     is a different endpoint under a different scope, and **this pack's reading is that the audit
     log query is a separate resource requiring its own permission**. The resource page cited below
     publishes no permission at all, and the advanced-hunting method page publishes
     `ThreatHunting.Read.All`; neither states that holding one confers the other. **Confirm the
     audit grant before you start** rather than part way through.
   - **Pad the window, and state the timezone you mean.** Audit timestamps are UTC, so a change made
     near a UTC boundary in local time falls on the adjacent UTC day, and a one day window can
     return a true zero while the record sits just outside it. Pad by a day on each side, or a
     boundary artefact is indistinguishable from the absence you are testing for.
   - **Enumerate the operation names in the window rather than matching on terms.** A term search
     excludes only what its terms match: a search on two terms says nothing about an operation named
     with neither, and an operation name need not carry the words the step 3 filter value is built
     from, since one called simply "Update" would carry none of them. **Note also that the audit log
     records operation names while step 3 filters on a record type**, which is a mapping this pack
     infers rather than one Microsoft publishes. Read the list, and give any matcher a control term
     you know is present in the same set.
6. Re-read the connector entry on the Sentinel data-connectors reference and record whether the
   page-level-notice versus per-entry-tag conflict still stands. If Microsoft resolves it, this
   row's status label changes.

## Sources

- [CopilotActivity table (Azure Monitor Logs reference, Microsoft Learn)](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/copilotactivity) - last verified 2026-08-15
- [Find your Microsoft Sentinel data connector (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/sentinel/data-connectors-reference) - last verified 2026-08-15
- [`==` (equals), case-sensitive (Kusto query reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/equals-cs-operator) - read 2026-08-22 for step 3's case sensitivity
- [`!=` (not equals), case-sensitive (Kusto query reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/not-equals-cs-operator) - read 2026-08-22 for step 2's case sensitivity
- [`auditLogQuery` resource type (Microsoft Graph reference, Microsoft Learn)](https://learn.microsoft.com/en-us/graph/api/resources/security-auditlogquery) - read 2026-09-19 for verification step 5's description of the resource and its filter parameters. **Re-read 2026-09-20: the page publishes no permission**, which is why step 5 marks the separate-grant point as this pack's reading rather than as a documented one
- [`security: runHuntingQuery` (Microsoft Graph reference, Microsoft Learn)](https://learn.microsoft.com/en-us/graph/api/security-security-runhuntingquery) - read 2026-09-19 for verification step 5's separate advanced-hunting scope
- MITRE ATLAS technique IDs read from the distributed `atlas-data` dataset, `version: 5.6.0` (release tag `v2026.07`) - verified 2026-08-15
