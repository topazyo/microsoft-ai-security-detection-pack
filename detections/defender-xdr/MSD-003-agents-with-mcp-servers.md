# MSD-003 - AI agents with MCP servers attached

| Field | Value |
|---|---|
| **ID** | MSD-003 |
| **Deployment target** | Microsoft Defender XDR advanced hunting |
| **Primary table** | `AgentsInfo` |
| **Status** | **Public Preview**. Two Provisional elements below: the internal shape of `McpServers`, and the `Availability` value list. Two column names are Requires further validation: `AgentName` and `EntraAgentId`. |
| **Last verified** | 2026-08-15 |
| **MITRE ATLAS** | `AML.T0010.005` AI Supply Chain Compromise: AI Agent Tool · `AML.T0011.002` User Execution: Poisoned AI Agent Tool |
| **OWASP LLM 2026** | LLM03:2026 Excessive Agency · LLM04:2026 Supply Chain |

## Purpose

Enumerate the AI agents in your tenant that have Model Context Protocol servers connected to them,
with the owner, the availability scope, and the declared tool set alongside.

MCP is where an agent's blast radius stops being a policy question and becomes a list of endpoints.
This detection produces that list from telemetry rather than from a configuration review.

Run it as a **scheduled hunting query and alert on change**, not on presence. An MCP server
attachment is not a finding.

## Schema this depends on

Columns quoted from the `AgentsInfo` table reference on Microsoft Learn, read 2026-08-15 (page
stamp 2026-06-03). The page title reads **`AgentsInfo (Preview)`**.

| Column | Data type | Learn description (verbatim) |
|---|---|---|
| `Timestamp` | `datetime` | Date and time the agent information was recorded |
| `AgentId` | `string` | Unique identifier for the agent |
| `AgentName` | `string` | Display name of the agent |
| `Platform` | `string` | The platform that provided the information about the agent |
| `EntraAgentId` | `string` | The agent's unique enterprise application object identifier by Microsoft Entra ID |
| `PublishedStatus` | `string` | The agent's publication status; possible values: `Draft`, `Published` |
| `LifecycleStatus` | `string` | The agent's current operational state in the tenant; possible values: `Active`, `Blocked`, `Uninstalled`, `Deleted` |
| `Availability` | `string` | The deployment scope of the agent (that is, whether deployed to all users, specific groups, or individual users) |
| `Owners` | `dynamic` | Primary owners of the agent |
| `DeclaredTools` | `dynamic` | Functional tools the agent can invoke at runtime |
| `McpServers` | `dynamic` | The Model Context Protocol (MCP) servers connected to the agent, including server URLs and credential configuration |
| `Guardrails` | `dynamic` | Guardrails attached to the agent and their coverage |

> **Provisional: the internal shape of `McpServers` is not documented.** Learn types it `dynamic`
> and says what it holds, but publishes no field names inside it. The query below therefore tests
> the column's string form rather than indexing into it. **Do not write a query that indexes into
> this column until you have inspected it in your own workspace** - verification step 2 below.
>
> **Both shipped queries project this column, and the inspection queries in the verification steps
> surface it as well, so its populated values render on your own screen wherever you run them.** That
> is what makes them useful for review and it is unavoidable in your own portal. **It changes nothing
> about what may leave it**: the field names and the empty forms your platforms emit are the finding,
> and verification step 2 states that rule in full.
>
> **Rendering on your own screen is not the only exposure, and the sentence above covers only that
> one.** This file tells you to run its change-detection variant as a scheduled hunting query, and a
> scheduled rule puts whatever it projects into alert storage rather than only in front of you. That
> is the same argument MSD-004 makes for deliberately not projecting `Instructions`. Weigh it before
> you schedule either query as written. **What these queries return stays in your environment as
> well, and the egress statement for this file sits after the two queries below** rather than inside
> this block.

> **Provisional: `Availability` has no documented value list.** Learn describes what the column
> means but does not enumerate its values, so the query projects it and never filters on it.

> Requires further validation: the names `AgentName` and `EntraAgentId`. The table reference quoted
> above lists both, re-read 2026-09-27 at an unchanged page stamp. Two later Microsoft Learn pages
> name `AgentName` differently, and one of them also names `EntraAgentId` differently. The Azure
> Monitor Logs reference for `AgentsInfo` lists `Name`
> and `EntraAgentID` with the same descriptions (`ms.date` 2026-07-31). The Defender for Endpoint
> page on discovering local AI agents queries this table in advanced hunting with `Name` and states
> that "The columns that carry local AI agent data are `AgentId`, `Name`, `Version`,
> `PublishedStatus`, `LifecycleStatus`, `LastUpdatedDateTime`, `McpServers`, `DeclaredTools`, and
> `RawAgentInfo`" (`ms.date` 2026-09-16). Both were read 2026-09-27. Microsoft's own sources
> conflict, which is what that label means in the canonical legend, and for `AgentName` they
> conflict on this file's own deployment target. The queries keep the names this table's reference
> publishes. A wrong name fails loudly, because a reference to a column the table does not carry is
> a syntax error, so `column_ifexists()` is deliberately not used: it would turn that error into an
> empty column. Verification step 1 settles which names your workspace carries.

> **Why the query does not use `isnotempty()` on its own, and why that matters more than it looks.**
> **The `isempty()` reference settles it outright, in its own example table on the page this file
> already cites**: it publishes `isempty(parsejson("[]"))` as **false** and
> `isempty(parsejson("{}"))` as **false**, against `isempty(parsejson(""))` as true. Read
> 2026-08-19. So a bare `isnotempty(McpServers)` returns
> **true for every agent whose platform emits an empty array**, which would hand you the entire
> agent estate and look exactly like the standing population this detection tells you to expect.
> **The coercion chain explains why those rows come out that way, and it is explanation rather than
> the evidence.**
> `isempty()` is documented as true only "if the argument is an empty string or is null", per the
> `isempty()` reference. A `dynamic` argument is coerced through `tostring()`, and the `tostring()`
> reference states: "If value is non-null, the result is a string representation of value. If value
> is null, the result is an empty string." **The string representation of an empty JSON array is
> `"[]"`** - two characters, neither empty nor null.
> The predicate below tests the string form against the empty-JSON forms explicitly. It is
> shape-independent, so it does not reintroduce the assumption the note above rules out.
> `array_length()` is deliberately not used: its reference states it "Returns the number of elements
> in array, or `null` if array isn't an array", so a non-array value produces null rather than a
> comparable count. Both pages are cited in Sources.

> **Table migration, and the date has passed.** Learn states: "The `AIAgentsInfo` table is
> transitioning to the `AgentsInfo` table. Microsoft Agent 365 customers should use the
> `AgentsInfo` table today. The `AIAgentsInfo` table remains accessible until **July 1, 2026**."
> That date is in the past as of this pack's verification date. Any query you hold that still
> names `AIAgentsInfo` needs re-checking.
>
> **And Microsoft's own two pages disagree about it right now.** The advanced-hunting schema table
> list still lists `AIAgentsInfo (Preview)` as a current table, described there as "Information
> about AI agents created with Microsoft Copilot Studio, including agent configuration and ownership
> details" - read 2026-08-15, six weeks after the date the `AgentsInfo` reference gives for its
> removal. Do not read the schema list's continued listing as evidence the table is supported.
> Confirm which of the two your workspace actually returns.

## Query

```kusto
// MSD-003 - active agents with an MCP server attached.
// Deployment target: Microsoft Defender XDR advanced hunting.
// Schema verified against Microsoft Learn on 2026-08-15. AgentsInfo is in Public Preview.
// The 30-day window is deliberate: Learn does not document how often AgentsInfo writes a
// row per agent, so a shorter window can drop agents whose configuration has not changed.
// !in~ rather than !in: "null" is the one empty form carrying a case, no value list is
// published, and in is case-sensitive. The same choice applies in the variant below.
AgentsInfo
| where Timestamp > ago(30d)
| summarize arg_max(Timestamp, *) by AgentId
| extend McpServersRaw = tostring(McpServers)
| where isnotempty(McpServersRaw) and McpServersRaw !in~ ("[]", "{}", "null")
// == is case-sensitive, so this matches "Active" and no other casing. A workspace emitting a
// different casing returns nothing here, and an empty result reads as a clean estate. The
// literal is used as Learn publishes it. MSD-007 cites the operator pages for == and !=.
| where LifecycleStatus == "Active"
| project
    Timestamp,
    AgentId,
    AgentName,
    Platform,
    PublishedStatus,
    LifecycleStatus,
    Availability,
    Owners,
    McpServers,
    DeclaredTools,
    Guardrails,
    EntraAgentId
| order by AgentName asc
```

`summarize arg_max(Timestamp, *) by AgentId` reduces the table to one current row per agent.
`AgentsInfo` is an inventory table: without that collapse you count snapshots, not agents.

**The window is a load-bearing choice, not a default.** Microsoft Learn does not document whether
`AgentsInfo` writes rows on a schedule or only on change. If rows are change-driven, an agent whose
configuration has been stable for longer than the window produces no row at all and **disappears
from the result** - and a stable, broadly deployed agent is exactly the one this pack most wants you
to see. Verification step 5 establishes which behaviour your workspace has. Until it does, treat any
window as a floor rather than a filter.

**Thirty days is also the ceiling, so the floor and the ceiling are the same number here.** The
advanced hunting overview states, verbatim: "Each query can look up native Defender XDR data from up
to the past 30 days", and extends that range only where a Microsoft Sentinel workspace is onboarded
and its analytics-tier retention is longer. If step 5 shows your cadence is slower than the window,
widening the window is not a remedy available to you on this target - re-run the query more often
than you consume its output, or read the result as a floor on the population and say so.

**On `Timestamp` and `ReportId`.** Microsoft recommends projecting both when a Defender query
becomes a custom detection rule, and MSD-001 records that recommendation and the enrichment it
names. `Timestamp` is projected here. **`ReportId` was not read on the `AgentsInfo` reference and
does not appear in the schema table above**, so this file does not project it; confirm against the
`getschema` diff in verification step 1 before building a rule from this query. The primary query is
a posture census meant to be read rather than scheduled; the change-detection variant is the one
this file says to schedule, so the recommendation and the enrichment it names bear on that variant.

**The custom-detection-rules page also says not to filter on `Timestamp` or `TimeGenerated`, and sets
the rule's lookback from its frequency rather than from the query.** MSD-001 records that in full,
including the page's own exception for narrowing inside the lookback. **It bears on the
change-detection variant below more than on anything else in this pack**: that variant's baseline leg
reaches back 30 days, and 30 days is the lookback only at the every-24-hours frequency. At a shorter
frequency the baseline is the leg that loses data, and a `leftanti` join against a truncated baseline
stops excluding, so the rule would alert on every agent with an MCP server attached rather than on a
change. **That is reasoning about the documented lookback figures, not a measurement**, and the page
does not describe what a prefiltered baseline leg does to a join. Settle it at conversion time, per
checklist Group 8.

### Change detection - the way to actually run this

```kusto
// Agents whose MCP configuration is new, or changed while remaining non-empty, in the last
// 24 hours. Fingerprinting the column rather than testing presence catches the case that
// matters: a sanctioned server swapped for another one on an agent that already had MCP
// attached. hash_sha256(tostring(...)) needs no knowledge of the column's internal shape -
// and pays for that with the stability limit stated below the query.
let baseline =
    AgentsInfo
    | where Timestamp between (ago(30d) .. ago(1d))
    | summarize arg_max(Timestamp, *) by AgentId
    | extend McpFingerprint = hash_sha256(tostring(McpServers))
    // arg_max(...) by AgentId returns one row per agent, so this distinct removes nothing as
    // the leg stands. It is kept because it states what the baseline is keyed on, and it
    // becomes load-bearing the moment the collapse above is removed.
    | distinct AgentId, McpFingerprint;
AgentsInfo
| where Timestamp > ago(1d)
| summarize arg_max(Timestamp, *) by AgentId
// == is case-sensitive here too. A workspace emitting a different casing returns nothing from
// this leg, which empties the variant rather than reporting no change. See the note on the
// primary query above.
| where LifecycleStatus == "Active"
| extend McpServersRaw = tostring(McpServers)
| where isnotempty(McpServersRaw) and McpServersRaw !in~ ("[]", "{}", "null")
| extend McpFingerprint = hash_sha256(McpServersRaw)
| join kind=leftanti baseline on AgentId, McpFingerprint
| project Timestamp, AgentId, AgentName, Platform, Availability, Owners, McpServers, DeclaredTools
```

Baselining on `AgentId` alone would only catch a *new agent* with MCP attached. Swapping a
sanctioned server for an attacker-controlled one on an existing agent would be invisible - and that
is precisely `AML.T0011.002` Poisoned AI Agent Tool, which this file's own header claims. The
fingerprint join catches both.

**The baseline leg applies neither the lifecycle filter nor the empty-forms predicate the current
side applies, and the lifecycle one is worth stating rather than leaving to be rediscovered.** With
no `LifecycleStatus == "Active"` filter, the baseline still carries a fingerprint for an agent that
was blocked, uninstalled or deleted at some point in the window, so reactivating an agent whose MCP
configuration is unchanged since its last snapshot does not alert as though it were new.

**What the baseline holds is each agent's most recent configuration in the window rather than every
configuration seen in it**, because `arg_max(Timestamp, *) by AgentId` returns one row per agent:
its reference page states that it "Returns a row in the table that maximizes the specified
expression". **So this leg is a transition test rather than a novelty test**, and the difference is
worth stating because it is not what a reader expects from the word baseline. A configuration that
appeared early in the window and was superseded before the window closed is not in the baseline,
and a revert to it alerts. **That bites hardest where the fingerprint churns**, which is the
stability limit set out below this query: an alternating serialisation differs from the last state
on every flip, where a baseline holding every configuration seen would stop alerting once both
forms had appeared. A reader comparing the two legs will notice the difference, and this is what it
does. **What the empty-forms omission does is set out below**, under "It cannot fire on removal".

> **Correction, 2026-09-20.** An earlier version of this passage said the missing lifecycle filter
> "runs in the safe direction" on the ground that "a configuration already seen stays recognised as
> already seen". **That ground is false of a baseline leg that collapses to one row per agent**,
> and the `arg_max()` reference read on the same date settles it. The conclusion about reactivated
> agents survives and is kept above; the general claim about any configuration already seen does
> not, and the transition-versus-novelty paragraph replaces it. `CHANGELOG.md` is the record.

A 30-day baseline is a starting point, not a recommendation. Set it to whatever exceeds your
agent-onboarding cycle, and re-check it after the first month. **On the current-day side**, the
`LifecycleStatus == "Active"` filter is carried from the primary query on purpose: without it a
`Deleted` or `Uninstalled` agent can surface as a new MCP attachment. The baseline leg is the one
that does without it, for the reason the paragraph above gives.

**What the two shipped queries return stays in your environment, and they do not return the same
things.** The primary query projects `Owners`, which this file's schema table describes as the
agent's primary owners, beside `McpServers` and the rest of the active-agent inventory, one row per
agent, and it projects `DeclaredTools`, `Guardrails` and `EntraAgentId` beside them. **Never paste
the contents of `McpServers`, `Owners`, `DeclaredTools`, `Guardrails` or `EntraAgentId` anywhere** -
not into an issue, not into a verification report, not into a pull request, and not privately
either. What may leave them is the field names and the empty forms your platforms emit, never a
value, which is the rule MSD-004 states for `McpServers`, `Owners` and `DeclaredTools` among the
columns its own list names. **`Guardrails` and `EntraAgentId` are on this file's list and not on
MSD-004's** because MSD-004's shipped queries select agents whose `Guardrails` is empty and so never
return a populated one, and MSD-004 does not project `EntraAgentId` at all. **The change-detection
variant carries `McpServers`, `Owners` and `DeclaredTools` over
a narrower population**: only agents whose MCP fingerprint is new or changed in the last day, and
without `PublishedStatus`, `LifecycleStatus`, `Guardrails` or `EntraAgentId`. It is also the one
this file tells you to schedule, so what it projects goes into alert storage rather than only in
front of you. What travels outward is a column name or a value name, never a result row, a row
count, or anything inside one. **That default holds where a column carries a closed set the platform
defines; where a column carries free text the value is itself estate data and the default does not
reach it**, which is why `McpServers` is named above. **Apply that reasoning to any other free-text
column these queries return rather than looking for it on a list**: `AgentName` is the clearest of
them, because Microsoft describes it as the agent's display name and publishes no value set for it,
so it is a name from your own estate rather than a class the platform assigns. Checklist Group 0
states the same rule for every step in the pack, and this file repeats it because a reader who
deploys one detection may never open the checklist.

> **The fingerprint hashes a serialisation, not a logical configuration, and two consequences follow
> from that.**
>
> **It can fire without a configuration change.** `hash_sha256(tostring(McpServers))` is sensitive
> to every byte of however the platform renders the column. A change in JSON property order between
> emissions, or any volatile field inside the value, produces a different hash with nothing
> security-relevant behind it. Microsoft describes this column as holding "server URLs and
> credential configuration", so a rotated secret reference or an expiry stamp is exactly the kind of
> inner field that can churn. Measure that churn against a baseline before you schedule this daily -
> verification step 7.
>
> **It cannot fire on removal.** The empty-forms predicate sits on the **current-day side** of this
> query, not on `baseline`, so the result keeps only agents whose `McpServers` is still non-empty
> and an agent whose MCP servers were all detached drops out of it rather than appearing in it.
> Detaching every server is a configuration change worth seeing; this variant does not see it.
> **Inverting that predicate on its own does not give you the removal query either.** That case
> needs the current side inverted *and* a baseline restricted to agents that previously had a
> server - which is the shape MSD-004's guardrail-removal variant uses, and which drops the
> fingerprint entirely, because a removal is a transition between two states rather than a change
> of hash.
>
> **`hash_sha256()` ran on this deployment target in a tenant on 2026-08-24.** That run called it
> in Defender XDR advanced hunting and it returned a 64-character hexadecimal digest. That
> function's reference page describes its return as a hex string and states no length, and no
> published value was matched against it, so
> what the run settles is the availability question for that tenant on
> that date. **It does not make this variant
> deployable**: the table did not resolve there, so the variant was blocked for a
> different reason, and this file's table is the blocker to watch rather than the function.
> **This is still the only query in the pack meant for deployment that depends on a hashing
> function, and every other use of one in this pack is a check run once rather than a rule
> scheduled.** It is called out for this one because the primary query above does not depend on it
> and this variant does not run at all without it. That answer came from one tenant on one date, so
> confirm the function in your own Defender portal before scheduling this -
> verification step 7, and Group 1b check 4 of the checklist.

## Status evidence

**Public Preview.**

- The `AgentsInfo` table reference page title reads "AgentsInfo (Preview)" and the page carries:
  "Some information relates to prereleased product that may be substantially modified before it's
  commercially released."
- The feature page that documents this hunting surface is titled "Detect and investigate threats to
  AI agents using Microsoft Defender **(Preview)**" and states, verbatim: "This feature is currently
  in public preview. The Microsoft Defender preview terms apply to features that are in public
  preview."
- That page lists `AgentsInfo` among the advanced hunting tables for AI agent investigation,
  described as: "Contains inventory and configuration details for AI agents, including agent
  identity, platform, ownership, and metadata."

### Prerequisite - what has to be true for this table to hold anything

Per the same feature page, verbatim: agent observability requires "the Microsoft 365 app connector
to collect Agent 365 observability data for AI agent actions", and "Agents built with Microsoft
Copilot Studio, Microsoft Foundry, and declarative agents built with the Microsoft Copilot Agent
Builder send observability data to Microsoft 365 by default." For the rest, verbatim from a re-read
of the same page on 2026-08-18: "For AI agents built on other platforms, enable observability using
the Microsoft Agent 365 SDK, as described in the Agent 365 development lifecycle documentation."

## What this detection cannot see

- **Any MCP server not attached to an agent that emits Agent 365 observability data.** This table
  reflects agents managed through Microsoft Agent 365, per the prerequisites above. It is not a
  tenant-wide MCP inventory, and a developer-configured MCP server on a workstation is outside it.
  Confirm your own coverage rather than assuming it.
- **What the MCP server actually does.** The column holds configuration, not behaviour. Tool
  invocation at runtime is a different question - see MSD-008.
- **Whether the server is allowlisted.** This pack does not read enterprise MCP allowlist state,
  and no query here can tell you whether a listed server is permitted. Compare the result against
  your own allowlist by hand.
- **Agents in a platform that reports no MCP configuration.** An empty `McpServers` value may mean
  the platform does not populate the column, not that no server is attached. Learn does not state
  how a non-reporting platform fills it, so the query cannot distinguish "no server" from
  "not reported".
- **Agents that have not changed inside the query window, if `AgentsInfo` writes rows only on
  change.** Learn does not document the emission cadence. This is a limit on the *population* the
  query sees rather than on what it can observe about any agent, which makes it the easiest one to
  miss. Verification step 5.

## False-positive guidance

- **This is an inventory query. Presence is not a finding.** Expect a standing population of
  legitimate agents on the first run. Anything that alerts on every row will alert every day.
- **First-party and sanctioned servers will usually dominate.** Build the allowlist comparison
  before the alerting, not after.
- **`arg_max` hides churn.** An agent whose MCP configuration changed twice inside the window shows
  only its latest state. Use the change-detection variant if you care about the transitions.
- **The change-detection variant's fingerprint is byte-sensitive, so expect hits with no
  configuration change behind them.** Property reordering between emissions, or a rotated credential
  reference inside the value, changes the hash. Baseline the daily hit rate before you alert on it,
  and treat a fingerprint change as a prompt to compare the two values rather than as a finding.
- **`Draft` agents are real exposure but a different conversation.** The primary query does not
  filter `PublishedStatus`, so drafts appear. Decide deliberately whether your response process
  covers them.
- **Preview schema drift is the live risk here.** Both the table and the feature carry preview
  qualifiers. Re-run verification after any Defender portal schema change, not on a calendar.

## Workspace verification before deployment

1. Confirm the table exists and its columns match this file:
   `AgentsInfo | getschema | project ColumnName, ColumnType`. **Diff that output against the schema
   table above.** Record as a **schema mismatch** any column the schema table lists which the output
   does not carry, or which the output types differently. **A column the output carries and the schema
   table does not is not a mismatch**, because the schema table is a deliberate subset. That outcome is
   worth more than a clean run. Then `AgentsInfo | take 10` to confirm it returns rows; if it does
   not, check the Microsoft 365 app connector prerequisite above before concluding you have no agents,
   and check it just the same if every row it returns has `Platform == "LocalAgents"`.
   Then record which names your workspace carries for the two columns marked Requires further
   validation: `AgentsInfo | getschema | where ColumnName in ("AgentName", "Name", "EntraAgentId",
   "EntraAgentID") | project ColumnName`. The case-sensitive `in` is deliberate, because two of the
   four names differ only by case.
2. Inspect the column shape before writing anything that indexes into it:
   `AgentsInfo | extend R = tostring(McpServers) | where R !in~ ("", "[]", "{}", "null") | take 5
   | project AgentId, McpServers`. Read the JSON. Field names are not documented and may differ from
   what you expect.
   **Exactly two things from this column are findings: its field names, and the empty forms your
   platforms emit for it.** This step gives you the field names; **step 3 below gives you the empty
   forms for this column**, and MSD-004 verification step 2 enumerates them for `Guardrails`,
   `DeclaredTools` and `Endpoints` as well. The workspace verification checklist and the
   verification-report template state the same two-item rule, so every place that states it says one
   thing. Learn's own description of this column, quoted in the schema table above, is that it holds
   "server URLs and credential configuration". A sample of its contents is environment data of the most
   sensitive kind this pack asks you to look at, so it does not belong in an issue, a verification
   report or a pull request - and not in a private message either. The field names and the empty
   forms are the finding; the values are not.
3. **Confirm what your platform emits for "no MCP servers"**, because the query's predicate depends
   on it: `AgentsInfo | where Timestamp > ago(30d) | summarize Rows = count() by McpServersRaw =
   tostring(McpServers) | order by Rows desc | take 20`. **The alias is `Rows` and not `Agents`
   because this step does not collapse to one row per agent**, and it is not meant to: a form that
   appeared in an older snapshot is a form the predicate has to allow for, which is the opposite of
   what the `arg_max` collapse in this file's primary query is for. Read the figure as snapshot
   rows, on the same rule the note under that query states. If a form appears that is not `""`,
   `[]`, `{}` or `null`, add it to the predicate. **This non-collapsing shape is the one the
   empty-forms enumeration takes wherever this pack runs it**, on the reason just given; MSD-004
   verification step 2 and checklist Group 3 print that shape at their own steps, over their own
   columns. **The ordering is a convenience rather than the answer.** `order by ... desc` with a
   `take` surfaces the common forms, and an unanticipated fifth form is by construction a rare one,
   so the truncation hides exactly the case this step exists to find. **Run this as well:**
   `AgentsInfo | where Timestamp > ago(30d) | summarize Rows = count() by McpServersRaw =
   tostring(McpServers) | where McpServersRaw !in~ ("", "[]", "{}", "null") | order by Rows asc |
   take 20`. It drops the four forms you already account for and brings the rarest of what is left
   to the top. **That is a candidate filter and not a definition of emptiness**: some of what it
   returns will be genuinely populated values rather than a fifth empty form, and telling those
   apart is the reading this step asks of you. **This groups by the whole value string, so it
   renders populated entries on your own screen.** That is unavoidable in your own portal and it
   changes nothing about what leaves it: read the empty forms out of the output and leave everything
   else there.
4. Confirm the `LifecycleStatus` and `PublishedStatus` values your workspace emits match the
   documented sets (`Active`/`Blocked`/`Uninstalled`/`Deleted` and `Draft`/`Published`), and record
   the `Availability` values, which Learn does not publish. Learn's description covers a
   specific-groups case and does not publish the serialisation, so a platform that serialises that
   case by naming the groups puts your own organisation's naming in the column. **Send the shape you
   found. A value travels only if you have looked at what it contains and it carries nothing of your
   own organisation's; if it carries anything of yours, the shape travels and the value does not.**
5. **Establish the emission cadence**, which decides whether the query window is safe:
   `AgentsInfo | where Timestamp > ago(30d) | summarize Rows = count() by AgentId | summarize
   percentiles(Rows, 50, 95)`. If most agents show one or two rows over 30 days, rows are
   change-driven and a short window drops stable agents. **This is the same query checklist Group 3
   ships**, which it has to be: a check stated in two places is one check. The two columns this step
   used to compute for first and last sighting were dropped by the outer aggregation before anything
   rendered, so removing them changes no result.
6. Confirm whether any query you hold still references `AIAgentsInfo`, and which of the two tables
   your workspace returns.
7. **Before scheduling the change-detection variant, settle its two open questions.** First, confirm
   `hash_sha256()` runs in Defender XDR advanced hunting at all: `print H = hash_sha256("test")`.
   Run it in the Defender portal specifically, because that is where this variant would be
   scheduled. **The 2026-08-24 run ran exactly that statement there and it returned a
   64-character hexadecimal digest**. That function's reference page describes its return as a hex
   string and states no length, and no published value was matched against it. So this question has
   an answer for one tenant on one date; run it in yours rather than inheriting that one. Second,
   measure how much
   the fingerprint moves on its own:
   `AgentsInfo | where Timestamp > ago(30d) | extend F = hash_sha256(tostring(McpServers))
   | summarize Fingerprints = dcount(F), Rows = count() by AgentId | order by Fingerprints desc`.
   An agent showing many distinct fingerprints over many rows is changing something inside
   `McpServers` often. **This query measures the rate and not the cause** - it cannot separate a
   serialisation difference from a real configuration change, and only reading two of that agent's
   raw values against each other will tell you which you have. The rate is still the useful number:
   it is what a day of alerts from this variant will look like.

## Sources

- [AgentsInfo table in the advanced hunting schema (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-agentsinfo-table) - last verified 2026-08-15, re-read 2026-09-27 for the `AgentName` and `EntraAgentId` rows, which are unchanged, at an unchanged rendered date of 2026-06-03
- [Azure Monitor Logs reference - AgentsInfo (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/agentsinfo) - read 2026-09-27, `ms.date` 2026-07-31, for the `Name` and `EntraAgentID` rows the Requires further validation note sets against this file's schema table
- [Discover local AI agents with Microsoft Defender for Endpoint (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-endpoint/discover-local-ai-agents) - read 2026-09-27, `ms.date` 2026-09-16, for its advanced-hunting queries on `AgentsInfo`, which use `Name` and read `name`, `type` and `endpoint` from `McpServers`, for the sentence listing the columns that carry local AI agent data, for where it reports a local agent's MCP servers, for its sentence on when the table adds a record, and for its prerequisites
- [Handle advanced hunting errors (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-errors) - read 2026-09-27, `ms.date` 2026-05-18, for the syntax-error row, whose cause includes "references to nonexistent operators, columns, functions, or tables"
- [Detect and investigate threats to AI agents using Microsoft Defender (Preview) (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/ai-agent-detection-protection) - last verified 2026-08-15, **re-read 2026-09-20, when the rendered "Last updated on" date read 2026-09-03**, and Microsoft's name for the product in the observability sentence quoted above read `Microsoft Copilot Agent Builder` rather than the form this file carried before that read. **The page's `ms.date` read 2026-08-07 on that same read**, so the two fields disagree here and the date above is the rendered one
- [Advanced hunting schema tables (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-schema-tables) - last verified 2026-08-15, for the `AIAgentsInfo` listing noted above
- [`isempty()` (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/isempty-function) - last verified 2026-08-15, for the empty-`dynamic` behaviour the query predicate works around, and re-read 2026-08-19 for the published example table quoted above. **Each date is kept rather than collapsed**, because each later read is what added the material recorded against it
- [`in` operator (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/in-cs-operator) - last verified 2026-08-15, for the case sensitivity of the `in` family
- [`!in~` operator (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/not-in-operator) - last verified 2026-08-15, **re-read 2026-09-20**, for its "List of scalars" section, which exemplifies the parenthesised scalar list this file's filters put behind `!in~` and publishes its own output. **MSD-004 cites this same page for a different section of it**: that file ships `!in~` over a `dynamic([...])` right-hand side, exemplified in the "Dynamic array" section, including the `let`-bound variant. **Both forms sit on this operator's own page**, so the two files cite one page for two sections rather than two different pages. An earlier version of this entry cited the case-sensitive `in` page for the `!in~` form and said that form was documented only in the comparison table shared across the `in` pages; **that was wrong**, and MSD-004's own entry for this page recorded the "List of scalars" section all along
- [`tostring()` (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/tostring-function) - last verified 2026-08-16, for the coercion step and the null case
- [`array_length()` (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/array-length-function) - last verified 2026-08-16, for the null-on-non-array behaviour that rules it out here
- [`hash_sha256()` (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/hash-sha256-function) - last verified 2026-08-16, for the function the change-detection variant uses
- [`arg_max()` (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/arg-max-aggregation-function) - read 2026-09-20, for the Returns statement that it "Returns a row in the table that maximizes the specified expression", which is what makes the baseline leg hold each agent's last state in the window rather than every state seen in it, and what makes the `distinct` after it a no-op as that leg stands. **This page was argued from in this file before it was cited**, which the entry records rather than leaves as a gap
- [Advanced hunting overview (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-overview) - last verified 2026-08-16, for the 30-day query date range and how a Microsoft Sentinel workspace extends it
- MITRE ATLAS technique IDs read from the distributed `atlas-data` dataset, `version: 5.6.0` (release tag `v2026.07`) - verified 2026-08-15
- OWASP Top 10 for LLM Applications item numbering carried from the companion capability-status matrix cross-walk, verified there by SHA-256 against OWASP's published download on 2026-08-09; the 2026 edition and its publication date re-confirmed 2026-08-15
