# MSD-004 - Broadly available agents with declared tools and no reported guardrails

| Field | Value |
|---|---|
| **ID** | MSD-004 |
| **Deployment target** | Microsoft Defender XDR advanced hunting |
| **Primary table** | `AgentsInfo` |
| **Status** | **Public Preview**. Five Provisional elements below: the internal shapes of `Guardrails`, `DeclaredTools`, `McpServers` and `Endpoints`, and the `Availability` value list. The `AgentName` column name is Requires further validation. |
| **Last verified** | 2026-08-15 |
| **MITRE ATLAS** | *Not mapped to a technique.* This detection reads configuration and observes no adversary behaviour - see the note below. |
| **OWASP LLM 2026** | LLM03:2026 Excessive Agency |

> **Why the ATLAS cell is empty.** The two techniques a reader is most likely to expect here are
> `AML.T0053` AI Agent Tool Invocation and `AML.T0086` Exfiltration via AI Agent Tool Invocation.
> Neither belongs in this cell. Both are runtime-execution techniques, and this query reads a
> **configuration** table that cannot observe a single invocation - so printing either one in a
> column headed MITRE ATLAS would claim coverage from a detection structurally incapable of seeing
> it. **No technique in the 5.6.0 parse this pack verified covers agent-misconfiguration posture**,
> and saying so is more useful than reaching for the nearest one. **The positive control for that
> absence is in this same paragraph**: the search that returned nothing for posture is the search
> that returned `AML.T0053` and `AML.T0086` for agent tool invocation, so the parse was read and the
> absence is a property of the dataset rather than of the search. The techniques this posture is a
> **precondition** for are recorded in the cross-walk's Notes column instead.

## Purpose

Find published agents that can invoke tools and reach data, are available broadly, and report no
guardrails - the shape that turns an agent from a feature into a lateral path.

This is a posture query. It ranks agents for review; it does not assert that any of them is
misconfigured.

## Schema this depends on

Columns quoted from the `AgentsInfo` table reference on Microsoft Learn, read 2026-08-15 (page
stamp 2026-06-03). Page title: **`AgentsInfo (Preview)`**.

| Column | Data type | Learn description (verbatim) |
|---|---|---|
| `Timestamp` | `datetime` | Date and time the agent information was recorded |
| `AgentId` | `string` | Unique identifier for the agent |
| `AgentName` | `string` | Display name of the agent |
| `Platform` | `string` | The platform that provided the information about the agent |
| `PublishedStatus` | `string` | The agent's publication status; possible values: `Draft`, `Published` |
| `LifecycleStatus` | `string` | The agent's current operational state in the tenant; possible values: `Active`, `Blocked`, `Uninstalled`, `Deleted` |
| `Availability` | `string` | The deployment scope of the agent (that is, whether deployed to all users, specific groups, or individual users) |
| `Owners` | `dynamic` | Primary owners of the agent |
| `Instructions` | `string` | The agent's system prompt that defines its default behavior, persona, and operating boundaries |
| `DeclaredTools` | `dynamic` | Functional tools the agent can invoke at runtime |
| `DeclaredDataSources` | `dynamic` | The data repositories and knowledge sources the agent can access |
| `McpServers` | `dynamic` | The Model Context Protocol (MCP) servers connected to the agent, including server URLs and credential configuration |
| `Permissions` | `dynamic` | Permissions record of the agent, including those that have been requested and granted, their approval state, and consent enumeration |
| `Guardrails` | `dynamic` | Guardrails attached to the agent and their coverage |
| `Endpoints` | `dynamic` | List of agent runtime endpoints, including URL, transport type, and external connectivity flag |

> **Provisional: `Guardrails`, `DeclaredTools`, `McpServers` and `Endpoints` are typed `dynamic` and
> their internal shapes are not documented.** The queries test each column's string form rather than
> indexing into it, so they make no assumption about field names. `Endpoints` belongs on this list
> for the same reason as the other three: the posture rollup below counts agents by whether it is
> empty, and Learn describes what the column holds - "List of agent runtime endpoints, including
> URL, transport type, and external connectivity flag" - without publishing the field names inside
> it. The four shapes are undocumented on the table reference. The Defender for Endpoint page on
> discovering local AI agents reads `name`, `type` and `endpoint` from `DeclaredTools` and
> `McpServers` in queries scoped to `Platform == "LocalAgents"` (read 2026-09-27), which says
> nothing about other platforms.

> **Provisional: `Availability` has no documented value list.** Learn describes what the column
> means but does not enumerate its values. This query therefore projects it rather than filtering on
> it. Add a filter only after reading the values your own workspace emits.

> Requires further validation: the name `AgentName`. The table reference quoted above lists it,
> re-read 2026-09-27. The Azure Monitor Logs reference for `AgentsInfo` lists `Name` with the same
> description (`ms.date` 2026-07-31), and the Defender for Endpoint page on discovering local AI
> agents uses `Name` in its advanced-hunting queries against this table (`ms.date` 2026-09-16), both
> read 2026-09-27. Microsoft's own sources conflict on this file's deployment target, which is what
> that label means in the canonical legend. The queries keep `AgentName`, so a wrong name fails
> loudly as a syntax error rather than returning an empty column, and `column_ifexists()` is
> deliberately not used for that reason. Verification step 1 settles it.

> **`isempty()` alone would have made this detection fail silently, in the direction that matters.**
> **The `isempty()` reference settles this outright, in its own example table on the page this file
> already cites**: it publishes `isempty(parsejson("[]"))` as **false** and
> `isempty(parsejson("{}"))` as **false**, against `isempty(parsejson(""))` as true. Read
> 2026-08-19. So if an agent platform emits `[]` for "no guardrails attached", a bare
> `where isempty(Guardrails)` matches **nothing**: the query returns zero rows and an unguarded
> estate reads as clean. That is not workspace variance and it is not this pack's inference; it is
> published behaviour, and it is why the predicate below tests the string form against the
> empty-JSON forms explicitly.
>
> **The coercion chain below explains why the published rows come out that way, and it is
> explanation rather than the evidence.** `isempty()` is documented as true only "if the argument is
> an empty string or is null". A
> `dynamic` argument is coerced through `tostring()`, whose reference states: "If value is non-null,
> the result is a string representation of value. If value is null, the result is an empty string."
> **The string representation of an empty JSON array is `"[]"`** - neither empty nor null. The `""`
> entry in `EmptyForms` carries the null case,
> because `tostring()` renders null as an empty string rather than returning null.
> `array_length()` is deliberately not used - its reference states it "Returns the number of
> elements in array, or `null` if array isn't an array", which would reintroduce the shape
> assumption the note above rules out. Verification step 2 enumerates which forms your own platform
> emits, and step 3 demonstrates the `isempty()` behaviour itself.

> **Outcome: `AgentsInfo` did not resolve on any run this pack records.** The runs are dated
> 2026-08-24, 2026-08-26 and 2026-09-11, and `docs/verification-methodology.md` states the outcome
> for this table in exactly those terms. **Every query block in this file names Microsoft Defender
> XDR advanced hunting as its deployment target and was submitted there, and none of them parsed
> against the table**, because the table was not there to parse against. **Submitted is not the
> same as ran, and this file has only the first.**
> [MSD-003](MSD-003-agents-with-mcp-servers.md) records the same outcome for the same table, and
> that file's `hash_sha256()` note is the one place a run on this target reached past it.
> **So every statement here about what these queries return is a schema-verified construction
> rather than an observed result**, which is what the hedges throughout this file rest on and why
> none of them is written as a claim about behaviour.
> **What this does not tell you is why the table was absent**, and this pack does not establish it:
> the table is in preview, and whether it resolves for you is what the workspace verification below
> is for. **One environment's answer on one date is not yours.**

## Query

```kusto
// MSD-004 - published, active agents that can invoke tools but report no guardrails.
// Deployment target: Microsoft Defender XDR advanced hunting.
// Schema verified against Microsoft Learn on 2026-08-15. AgentsInfo is in Public Preview.
// The 30-day window is deliberate: Learn does not document how often AgentsInfo writes a
// row per agent, so a shorter window can drop stable agents - which are the ones this
// detection most wants to see. See "The window is a load-bearing choice" below.
// in~ and !in~ rather than in and !in: "null" is the one empty form carrying a case, no value
// list is published, and in is case-sensitive. The same choice applies to the empty-form lists
// in the blocks below. It does not reach the tabular in (HadGuardrails) in the removal variant,
// which matches an agent identifier against a let-bound subquery rather than against a value
// list. That is the let-bound tabular-expression right-hand side the case-sensitive in reference
// documents with a worked example, and the same page states that the search considers up to
// 1,000,000 distinct values - a ceiling, and the page does not say what happens when it is
// exceeded, so what a larger estate does here is not settled.
let EmptyForms = dynamic(["", "[]", "{}", "null"]);
AgentsInfo
| where Timestamp > ago(30d)
| summarize arg_max(Timestamp, *) by AgentId
// == is case-sensitive, so both filters match only the casing written here. A workspace
// emitting a different casing on either column returns nothing, and an empty result reads as
// no broad agents rather than as a failed filter. Both literals are used as Learn publishes
// them. MSD-007 cites the operator pages for == and !=.
| where LifecycleStatus == "Active"
| where PublishedStatus == "Published"
| extend
    ToolsRaw = tostring(DeclaredTools),
    GuardrailsRaw = tostring(Guardrails)
| where ToolsRaw !in~ (EmptyForms)
| where GuardrailsRaw in~ (EmptyForms)
| project
    Timestamp,
    AgentId,
    AgentName,
    Platform,
    Availability,
    Owners,
    DeclaredTools,
    DeclaredDataSources,
    McpServers,
    Endpoints,
    Permissions
| order by AgentName asc
```

**The window is a load-bearing choice, not a default.** The `AgentsInfo` table reference does not
document whether the table writes rows on a schedule or only on change. The Defender for Endpoint
page on discovering local AI agents states that "AgentsInfo adds a record each time an agent profile
is updated", which names one trigger and no schedule. If rows are change-driven, an agent whose
configuration has been stable for longer than the window produces no row and **drops out of the
result** - and a stable, broadly deployed agent with tools and no guardrails is precisely this
detection's target. Verification step 4 establishes which behaviour your workspace has.

**Thirty days is also the ceiling on this target, so the floor and the ceiling are the same number.**
The advanced hunting overview states, verbatim: "Each query can look up native Defender XDR data
from up to the past 30 days", and extends that range only where a Microsoft Sentinel workspace is
onboarded and its analytics-tier retention is longer. If the cadence turns out to be slower than the
window, widening the window is not a remedy available here.

**On `Timestamp` and `ReportId`.** Microsoft recommends projecting both when a Defender query becomes
a custom detection rule, and MSD-001 records that recommendation and the enrichment it names.
`Timestamp` is projected here. **`ReportId` was not read on the `AgentsInfo` reference and does not
appear in the schema table above**, so this file does not project it; confirm against the `getschema`
diff before building a rule from any query here. The primary query is a posture census meant to be
read rather than scheduled; the guardrail-removal variant is the one this file says to schedule.

**The custom-detection-rules page also says not to filter on `Timestamp` or `TimeGenerated`, and sets
the rule's lookback from its frequency rather than from the query**, which MSD-001 records in full
along with the page's own exception for narrowing inside the lookback. It bears on the
guardrail-removal variant below, whose baseline leg reaches back 30 days exactly as MSD-003's does,
so the same conversion-time check applies and checklist Group 8 carries it. **The consequence here is
the opposite of MSD-003's, and it is the quieter of the two.** MSD-003 excludes its baseline with a
`leftanti` join, so truncating the baseline makes that rule over-report. This variant includes its
baseline, with `where AgentId in (HadGuardrails)`, so truncating it **shrinks the inclusion list**:
an agent whose guardrail evidence fell outside the surviving slice is dropped, and a genuine
guardrail removal on that agent is not reported at all. **A rule failing this way returns nothing and
reads as a clean estate**, which is why the check matters here even though nothing looks wrong.

**Truncation is not the only thing that shrinks that inclusion list, and it is the conditional one
of the two.** Lookback truncation arises only where the rule's frequency is shorter than the
baseline leg reaches back. **The collapse inside the leg shrinks the list unconditionally and at
every frequency**, because `HadGuardrails` holds each agent's last pre-window state rather than
every state it held, which the paragraph under the change-detection query sets out in full. Both
produce the same silent result, so **a check for one is not a check for the other**.

> **Correction, 2026-09-20.** An earlier version of this passage attributed the shrinking of the
> inclusion list entirely to lookback truncation, and named no other cause. **The collapse inside
> the baseline leg produces the same shrink unconditionally**, which the `arg_max()` reference read
> on the same date settles. The truncation case above is unchanged and still holds; what was wrong
> was presenting it as the only one. `CHANGELOG.md` is the record.

### Posture rollup - run this first

```kusto
// How the estate is distributed before you decide what to alert on.
let EmptyForms = dynamic(["", "[]", "{}", "null"]);
AgentsInfo
| where Timestamp > ago(30d)
| summarize arg_max(Timestamp, *) by AgentId
| summarize
    Agents = count(),
    WithTools = countif(tostring(DeclaredTools) !in~ (EmptyForms)),
    WithoutGuardrails = countif(tostring(Guardrails) in~ (EmptyForms)),
    WithMcp = countif(tostring(McpServers) !in~ (EmptyForms)),
    WithEndpoints = countif(tostring(Endpoints) !in~ (EmptyForms))
    by Platform, LifecycleStatus, PublishedStatus, Availability
| order by Agents desc
```

**That rollup groups by four dimensions, so no single row answers "does this platform report
guardrails at all?"** Collapse it to one row per platform when that is the question:

```kusto
// The same rollup, one row per platform. This is the row that separates a posture finding
// from a platform that does not report guardrails.
let EmptyForms = dynamic(["", "[]", "{}", "null"]);
AgentsInfo
| where Timestamp > ago(30d)
| summarize arg_max(Timestamp, *) by AgentId
| summarize
    Agents = count(),
    WithoutGuardrails = countif(tostring(Guardrails) in~ (EmptyForms))
    by Platform
| extend NoneReportGuardrails = (WithoutGuardrails == Agents)
| order by Agents desc
```

`NoneReportGuardrails` being true for a platform means every agent on that platform reports no
guardrails. **That is equally consistent with a platform that does not populate the column and with
a platform whose agents genuinely have none, and nothing in this table separates the two** - it is
the limitation named first in the next section. So the flag is the question to put to whoever owns
that platform, not an answer in either direction. Reading it as a reporting artefact would be the
more expensive mistake of the two, because it tells you not to look.

### Change detection - guardrail removal is the event worth an alert

```kusto
// An agent that had guardrails and no longer does. Fingerprinting the column needs no
// knowledge of its internal shape.
let EmptyForms = dynamic(["", "[]", "{}", "null"]);
let HadGuardrails =
    AgentsInfo
    | where Timestamp between (ago(30d) .. ago(1d))
    | summarize arg_max(Timestamp, *) by AgentId
    | where tostring(Guardrails) !in~ (EmptyForms)
    // arg_max(...) by AgentId returns one row per agent, so this distinct removes nothing as
    // the leg stands, and HadGuardrails is each agent's LAST pre-window state rather than
    // every state it held. The paragraph below this query states what that costs.
    | distinct AgentId;
AgentsInfo
| where Timestamp > ago(1d)
| summarize arg_max(Timestamp, *) by AgentId
// == is case-sensitive here too. A workspace emitting a different casing returns nothing from
// this leg, which empties the variant rather than reporting no removals. See the note on the
// primary query above.
| where LifecycleStatus == "Active"
| where tostring(Guardrails) in~ (EmptyForms)
| where AgentId in (HadGuardrails)
| project Timestamp, AgentId, AgentName, Platform, Availability, Owners, DeclaredTools, McpServers
```

The primary query is a posture census. This variant is the one to schedule: `arg_max` shows current
state only, so an agent whose guardrails were removed looks identical to one that never had any
unless you compare against a baseline.

**`HadGuardrails` is each agent's last state in the baseline window, not every state it held there,
and that gives this variant a one-run detection window.** `arg_max(Timestamp, *) by AgentId`
returns one row per agent: its reference page states that it "Returns a row in the table that
maximizes the specified expression". So once a removal has appeared in a baseline-window snapshot,
that agent's last pre-window state no longer shows guardrails, it drops out of `HadGuardrails`, and
`where AgentId in (HadGuardrails)` excludes it from the current side. **The removal is then
unreportable rather than merely reported late.** The variant catches a removal on the run that
follows it and not afterwards, so **a missed run is a missed removal**: a paused rule, an ingestion
delay that shifts a row across the window boundary, or an emission cadence slower than daily each
produce a real removal that is never alerted, and the rule stays silent and reads as a clean
estate. **That is the failure class `SECURITY.md` ranks worst**, and it is a property of this leg
rather than of any workspace. Verification step 7 below says what to do after a scheduling gap.

**What these queries return stays in your environment, and some of the columns they project need
more care than the rest.** The query above and the change-detection variant both project
`McpServers`, which Microsoft describes as holding "server URLs and credential configuration", so a
populated value is environment data of the most sensitive kind this pack asks you to look at. The
posture rollup reduces that column to a count and the per-platform rollup does not reference it, so
this paragraph is about the two blocks that project it. `Endpoints` and `Permissions`, which the
query above also projects, carry the same weight for the same reason: Microsoft describes the first
as listing runtime endpoints "including URL, transport type, and external connectivity flag" and the
second as a permissions record "including those that have been requested and granted, their approval
state, and consent enumeration". **`DeclaredTools` and `DeclaredDataSources` belong with them**,
described as "Functional tools the agent can invoke at runtime" and as "The data repositories and
knowledge sources the agent can access": both name an organisation's own configuration. `Owners` is
described only as "Primary owners of the agent", with no value list published. **That an owner may
be a person, a group or a service principal is this pack's reading of what an unenumerated owner
column can hold rather than something that page states**, and the values are estate data on any of
those readings. Their values
render on your own screen wherever you run them, which is unavoidable in your own portal. **The
variant above is the one this file tells you to schedule, and a scheduled rule does more than
render**: it puts whatever it projects into alert storage, which is the same argument this file
makes below for not projecting `Instructions`. **Never paste the contents of `Instructions`,
`McpServers`, `Endpoints`, `Permissions`, `DeclaredTools`, `DeclaredDataSources` or `Owners`
anywhere** - not into an issue, not into a verification report, not into a pull request, and not
privately either. What may leave them is the field names and the empty forms your platforms emit,
never a value. More
generally, what travels outward from these queries is a column name or a value name, never a result
row, a row count, or anything inside one. **That default holds where a column carries a closed set
the platform defines; where a column carries free text the value is itself estate data and the
default does not reach it**, which is why `McpServers` is named above. **Apply that reasoning to any
other free-text column these queries return rather than looking for it on a list**: `AgentName` is
the clearest of them, because Microsoft describes it as the agent's display name and publishes no
value set for it, so it is a name from your own estate rather than a class the platform assigns.
**`Availability` is neither case and is not on the list above**: Learn publishes no value list for it, the queries here project
it, and its serialisation is what the pack wants back. Send the shape you found. A value travels
only if you have looked at what it contains and it carries nothing of your own organisation's; if it
carries anything of yours, the shape travels and the value does not. Verification step 6 below
states the same rule.
Checklist Group 0 states the same rule for every step in the pack, and this file repeats it because a
reader who deploys one detection may never open the checklist.

**It deliberately drops two filters the primary query applies**, and you should know which. There is
no `PublishedStatus == "Published"` filter, because guardrails being removed from a draft agent is
still worth seeing, and no `DeclaredTools` filter, because an agent can lose its guardrails before it
gains its tools. Add either filter if your triage capacity says so; the effect is a narrower result,
not a more accurate one.

## Status evidence

**Public Preview**, on the same evidence as [MSD-003](MSD-003-agents-with-mcp-servers.md):

- The `AgentsInfo` table reference page title reads "AgentsInfo (Preview)" and carries the standing
  prerelease disclaimer.
- The feature page is titled "Detect and investigate threats to AI agents using Microsoft Defender
  (Preview)" and states: "This feature is currently in public preview. The Microsoft Defender
  preview terms apply to features that are in public preview."

## What this detection cannot see

- **Whether an agent has guardrails that its platform does not report.** Learn describes
  `Guardrails` as "Guardrails attached to the agent and their coverage" and **does not state how a
  platform with no guardrail concept populates the column.** An empty value is therefore ambiguous
  between "none attached" and "not reported". This is the single largest limitation of this
  detection.
- **Agents that have not changed inside the query window, if `AgentsInfo` writes rows only on
  change.** The table reference does not document the emission cadence, and the Defender for
  Endpoint page names profile updates as one trigger without a schedule. This limits the
  *population* the query sees rather than what it observes about any agent, which is what makes it
  easy to miss.
- **Whether a declared tool was ever invoked.** `DeclaredTools` is configuration. Runtime tool use
  is a different surface - `CloudAppEvents` per the feature page, and behaviours per MSD-008.
- **Whether the granted permissions are excessive.** The `Permissions` column is projected for
  review, not evaluated. No query in this pack scores a permission set.
- **Agents the table does not report, and the posture of local AI agents it does.** Same boundary as
  MSD-003. The Defender for Endpoint page on discovering local AI agents states that "Many
  AgentsInfo columns describe cloud agents and are empty for local AI agents" and lists the columns
  that carry local-agent data, and `Guardrails` is not among them, so on this pack's reading a
  published, active local agent with declared tools reaches the primary query's result for want of
  a column its platform does not populate. Its own posture, including whether it acts without
  prompting the user for approval, sits in `RawAgentInfo`, which this file does not read.

  > **Correction, 2026-09-27.** An earlier version of this bullet said agents outside Microsoft
  > Agent 365 management are outside this detection. Local AI agents that Defender for Endpoint
  > discovers are reported in this table. `CHANGELOG.md` is the record.
- **What the system prompt says.** `Instructions` is deliberately **not projected** by the primary
  query. Agent system prompts can carry organisation-specific content, and pulling them into a
  scheduled rule's results puts that content into alert storage. Read it on demand, per agent,
  when an investigation needs it.

## False-positive guidance

- **Read the posture rollup before you alert on anything.** If an entire platform reports empty
  `Guardrails`, alerting on it would generate one alert per agent forever - whether the cause is a
  reporting gap or a genuinely unguarded platform. Settle which it is with the platform owner before
  you wire an alert to it, rather than through this query.
- **A guardrail can live outside the agent.** Conditional Access, DLP policy, and network controls
  do not appear in this column. An agent with no reported guardrails may still be well constrained.
- **Broad availability is often correct.** A published helpdesk agent deployed to all users is the
  intended design, not a defect. This query ranks candidates for review; the review is human.
- **Draft agents are excluded by the primary query's `PublishedStatus == "Published"` filter.** That
  is a deliberate choice to keep the result actionable, and it means unpublished agents with tools do
  not appear. Run the rollup without that filter periodically. The guardrail-removal variant carries
  no such filter, for the reason stated under it.
- **`arg_max` shows current state only.** An agent whose guardrails were removed and re-added
  inside the window looks unchanged. The change-detection variant above is the answer to that, and
  it is the variant to schedule.

## Workspace verification before deployment

1. Complete the MSD-003 verification steps first, including the `getschema` diff. This detection
   uses the same table.
2. **Enumerate what your platforms actually emit for an empty column**, because every predicate here
   depends on it:
   `AgentsInfo | where Timestamp > ago(30d) | summarize Rows = count() by
   Platform, GuardrailsRaw = tostring(Guardrails) | order by Rows desc | take 30`. If a form
   appears that is not `""`, `[]`, `{}` or `null`, add it to the `EmptyForms` list in every query.
   Repeat for `DeclaredTools`, `Endpoints` and `McpServers`.
   **That query does not collapse to one row per agent, and that is deliberate.** This file's own
   primary queries open with `summarize arg_max(Timestamp, *) by AgentId` to count agents; for this
   question that collapse hides a form, because an empty form a platform emitted in an earlier
   snapshot inside the window is not in the collapsed set and is still a form every predicate has to
   allow for. **The alias is `Rows` and not `Agents` for the same reason**, so the figure reads as
   snapshot rows rather than as agents. MSD-003 verification step 3 ships that shape over its own
   column and states the reason there.
   **The ordering is a convenience rather than the answer.** `order by ... desc` with a `take`
   surfaces the common forms, and an unanticipated fifth form is by construction a rare one, so the
   truncation hides exactly the case this step exists to find. **Run this as well:**
   `AgentsInfo | where Timestamp > ago(30d) | summarize Rows = count() by GuardrailsRaw =
   tostring(Guardrails) | where GuardrailsRaw !in~ ("", "[]", "{}", "null") | order by Rows asc |
   take 20`. It drops the four forms you already account for and brings the rarest of what is left
   to the top. **It also groups by the value alone rather than by platform and value**, so its
   `Rows` figures are across platforms and are not comparable with the block above, where the same
   alias counts rows per platform and value. A form several platforms emit shows as one row here,
   which is what keeps it inside `take 20`. **That is a
   candidate filter and not a definition of emptiness**: some of what it returns will be genuinely
   populated values rather than a fifth empty form, and telling those apart is the reading this step
   asks of you. **When you repeat it for `McpServers`,
   exactly two things from that column are findings: its field names, and the empty forms your
   platforms emit for it.** This step gives you the empty forms; MSD-003 verification step 2 gives you
   the field names, and the workspace verification checklist and the verification-report template
   state the same two-item rule. Learn describes that column as holding "server URLs and credential
   configuration", so nothing else from it belongs in an issue, a verification report, a pull request,
   or a private message. **This query groups by the whole value string, so for that column it renders
   populated entries on your own screen.** That is unavoidable in your own portal and it changes
   nothing about what leaves it: read the empty forms out of the output and leave everything else
   there.
3. **Confirm the failure mode this query is written around**, once, so you understand why it does
   not use `isempty()` directly: `print BareIsEmpty = isempty(dynamic([])), StringForm =
   tostring(dynamic([]))`. Expect `false` and `"[]"`.
4. Establish the emission cadence, per MSD-003 verification step 5. It decides whether the 30-day
   window is safe.
5. Run the posture rollup and record the per-platform baseline. Without it you cannot tell a
   posture finding from a reporting gap.
6. Confirm the `Availability` values your workspace emits before adding any filter on that column.
   As MSD-003 verification step 4 puts it: Learn's description covers a specific-groups case and
   does not publish the serialisation, so a platform that serialises that case by naming the groups
   puts your own organisation's naming in the column. **Send the shape you found. A value travels
   only if you have looked at what it contains and it carries nothing of your own organisation's; if
   it carries anything of yours, the shape travels and the value does not.**
7. **After any gap in the change-detection variant's schedule, re-run it once over a wider
   current-side window before trusting its silence.** The paragraph under that query explains why:
   `HadGuardrails` holds each agent's last pre-window state, so a removal that fell in the gap has
   already left the inclusion list and no later scheduled run will report it. Widen
   `Timestamp > ago(1d)` on the current side to cover the gap and compare the result against the
   posture rollup from step 5. **A silent run is the expected output of this variant most days, so
   silence after a gap tells you nothing on its own.**

## Sources

- [AgentsInfo table in the advanced hunting schema (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-agentsinfo-table) - last verified 2026-08-15, re-read 2026-09-27 for the `AgentName` row, which is unchanged, at an unchanged rendered date of 2026-06-03
- [Azure Monitor Logs reference - AgentsInfo (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/agentsinfo) - read 2026-09-27, `ms.date` 2026-07-31, for the `Name` row the Requires further validation note sets against this file's schema table
- [Discover local AI agents with Microsoft Defender for Endpoint (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-endpoint/discover-local-ai-agents) - read 2026-09-27, `ms.date` 2026-09-16, for its advanced-hunting queries on `AgentsInfo`, which use `Name` and read `name`, `type` and `endpoint` from `DeclaredTools` and `McpServers`, for the sentences on which columns carry local AI agent data, and for its sentence on when the table adds a record
- [Handle advanced hunting errors (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-errors) - read 2026-09-27, `ms.date` 2026-05-18, for the syntax-error row, whose cause includes "references to nonexistent operators, columns, functions, or tables"
- [Detect and investigate threats to AI agents using Microsoft Defender (Preview) (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/ai-agent-detection-protection) - last verified 2026-08-15
- [`isempty()` (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/isempty-function) - last verified 2026-08-15, for the empty-`dynamic` behaviour the predicates work around, and re-read 2026-08-19 for the published example table quoted above. **Each date is kept rather than collapsed**, because each later read is what added the material recorded against it
- [`tostring()` (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/tostring-function) - last verified 2026-08-16, for the coercion step and for the null case the `""` entry covers
- [`arg_max()` (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/arg-max-aggregation-function) - read 2026-09-20, for the Returns statement that it "Returns a row in the table that maximizes the specified expression", which is what makes `HadGuardrails` each agent's last pre-window state rather than every state it held, and is therefore what gives the change-detection variant its one-run detection window. **This page was argued from in this file before it was cited**, which the entry records rather than leaves as a gap
- [`array_length()` (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/array-length-function) - last verified 2026-08-16, for the null-on-non-array behaviour that rules it out here
- [`in` operator (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/in-cs-operator) - last verified 2026-08-15, which documents dynamic-array expansion and the case-sensitive `in` / `!in` and case-insensitive `in~` / `!in~` variants; **re-read 2026-08-22** for its "Tabular expression" worked example, which is the `let`-bound form the removal variant's `in (HadGuardrails)` ships and which publishes its own output, and for the parameter note stating that "The search considers up to 1,000,000 distinct values". **Each date is kept rather than collapsed**, because the later read is what added the material recorded against it
- [`in~` operator (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/in-operator) - last verified 2026-08-18, for the "Dynamic array" section, whose example passes a `dynamic([...])` literal directly to `in~` and publishes its output. It does not exemplify the `let`-bound form this file's `in~` predicates ship, and the `let` variant beside it substitutes another operator; [`docs/verification-methodology.md`](../../docs/verification-methodology.md) section 6 states what that leaves open
- [`!in~` operator (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/not-in-operator) - last verified 2026-08-18, for its own "Dynamic array" section, which is where the dynamic-array right-hand side **the shipped queries in this file** put behind `!in~` is exemplified for that operator, including the variant that binds the array with a `let` as those queries do. **Verification step 2 above ships a different form of the same operator**, a parenthesised scalar list rather than a dynamic array; that form is exemplified on this same page's "List of scalars" section and in the comparison table shared across the `in` pages, and not in the "Dynamic array" section this entry points at
- [Advanced hunting overview (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-overview) - last verified 2026-08-16, for the 30-day query date range and how a Microsoft Sentinel workspace extends it
- MITRE ATLAS technique IDs read from the distributed `atlas-data` dataset, `version: 5.6.0` (release tag `v2026.07`) - verified 2026-08-15
- OWASP Top 10 for LLM Applications item numbering carried from the companion capability-status matrix cross-walk, verified there by SHA-256 against OWASP's published download on 2026-08-09; the 2026 edition and its publication date re-confirmed 2026-08-15
