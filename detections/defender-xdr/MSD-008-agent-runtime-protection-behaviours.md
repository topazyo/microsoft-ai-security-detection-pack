# MSD-008 - AI-agent real-time protection block and audit behaviours

| Field | Value |
|---|---|
| **ID** | MSD-008 |
| **Deployment target** | Microsoft Defender XDR advanced hunting |
| **Primary table** | `BehaviorInfo` |
| **Status** | **Public Preview**, and **not available for GCC**. Two elements are **Provisional**: the AI-agent value set, and the `Categories` serialisation. |
| **Last verified** | 2026-08-15 |
| **MITRE ATLAS** | `AML.T0051` LLM Prompt Injection · `AML.T0054` LLM Jailbreak |
| **OWASP LLM 2026** | LLM01:2026 Prompt Injection |

## Purpose

Reach the audit and block events produced by AI-agent real-time protection, which Microsoft
documents as landing in `BehaviorInfo` rather than in an AI-specific table.

**This detection ships as discovery-first on purpose.** The `ActionType` values that correspond to
AI-agent real-time protection are not published on Microsoft Learn. A pack that guessed them would
be inventing schema, which is the defect this pack exists to avoid. So the shipped query enumerates
what your tenant actually emits, and the narrowing filter is one you complete after reading it.

## Schema this depends on

Columns quoted from the `BehaviorInfo` table reference on Microsoft Learn, read 2026-08-15 (page
stamp 2026-01-12). Page title: **`BehaviorInfo (Preview)`**.

| Column | Data type | Learn description (verbatim) |
|---|---|---|
| `Timestamp` | `datetime` | Date and time when the record was generated |
| `BehaviorId` | `string` | Unique identifier for the behavior |
| `Title` | `string` | Title of the behavior |
| `Description` | `string` | Description of the behavior |
| `Categories` | `string` | Type of threat indicator or breach activity identified by the behavior, as defined by the MITRE ATT&CK framework |
| `AttackTechniques` | `string` | MITRE ATT&CK techniques associated with the activity that triggered the behavior |
| `ServiceSource` | `string` | Product or service that identified the behavior |
| `DetectionSource` | `string` | Detection technology or sensor that identified the notable component or activity |
| `DataSources` | `string` | Products or services that provided information for the behavior |
| `AccountUpn` | `string` | User principal name (UPN) of the account |
| `AccountObjectId` | `string` | Unique identifier for the account in Microsoft Entra ID |
| `StartTime` | `datetime` | Date and time of the first activity related to the behavior |
| `EndTime` | `datetime` | Date and time of the last activity related to the behavior |
| `ActionType` | `string` | Type of behavior |

> **Provisional: no value in this table is documented as belonging to AI-agent protection.** Learn
> describes `ActionType` as "Type of behavior" and publishes no value list, and the same is true of
> `ServiceSource` and `DetectionSource` on this page. The link between this table and AI agents comes
> from a different page, quoted below - not from any value the schema reference names. The label is
> **Provisional** rather than Requires further validation because no Microsoft source contradicts
> another here; the value set is simply undocumented, which is what Provisional means in the
> canonical legend. MSD-007 is the pack's one detection whose status is Requires further
> validation, and keeping that label for the conflicting-sources case is what makes it worth
> anything.

> **Provisional: `Categories` has no published value format, and one environment returned a
> serialised array string rather than a bare name.** Microsoft types the column and publishes no
> format for it. Step 1 groups by this column, so the inventory it returns may be a set of
> combinations rather than of category names, and an equality filter built from one of them matches
> only the rows carrying that exact combination. **One environment's answer on one date is not
> yours**; record the shape you find.

> **Outcome, 2026-08-24: one tenant's `BehaviorInfo` schema did not carry `Title`.** Microsoft
> Learn publishes the column, re-read 2026-08-24 and quoted verbatim in the table above, so this is
> a difference between what Learn documents and what one workspace returned rather than a
> transcription error here. **All three queries this file shipped for deployment referenced `Title`
> and failed to resolve it there.** **Step 1 names it again behind `column_ifexists()`, which returns
> `Description` where the column is absent; step 2 and the self-check wrapper name it nowhere.** The
> column stays in the table because Learn publishes it, and verification step 1 is what tells you
> whether your own tenant carries it. **Re-checked 2026-08-26 in the same tenant, on Defender XDR
> advanced hunting: the column was still absent, and the guarded step-1 query ran and returned rows
> there.** **What that run does not tell you is why.** The table is in preview, and Learn
> states: "If your organization doesn't deploy these services in Microsoft Defender, queries that use
> the table won't work or return any results". **That speaks to results rather than to a column
> list**, and the page does not say what happens where one of the two services is deployed and the
> other is not. That is
> the nearest documented thing to an explanation and it is not one.

## Query

```kusto
// MSD-008 step 1 - discovery. Enumerate what this table actually carries in your tenant.
// Deployment target: Microsoft Defender XDR advanced hunting.
// BehaviorInfo is in Public Preview and is not available for GCC.
BehaviorInfo
| where Timestamp > ago(30d)
| summarize
    Behaviors = count(),
    SampleLabels = make_set(column_ifexists("Title", Description), 25),
    FirstSeen = min(Timestamp),
    LastSeen = max(Timestamp)
    by ActionType, ServiceSource, DetectionSource, Categories
| order by Behaviors desc
```

Read that output and identify which `ActionType` and `ServiceSource` combinations correspond to
AI-agent real-time protection in your tenant. Then complete the narrowing query:

**What `SampleLabels` carries, and what it costs you.** `column_ifexists("Title", Description)`
returns `Title` where your tenant carries that column and `Description` where it does not, so which
of the two you get is a property of your tenant rather than of this file. Learn describes `Title` as
"Title of the behavior" and `Description` as "Description of the behavior", and the two behave
differently under `make_set`: a title is a short label that repeats across behaviours sharing an
`ActionType`, so the set collapses to a handful of distinct labels, while a description is long free
text and is likely to be closer to unique, so the same cap returns a longer and less repetitive
sample for each row. **Treat the column as free text either way**, which is what the egress rule in
verification step 2 does. **Neither effect has been measured**: the reasoning above is drawn from the
two documented column descriptions rather than from a count, because the 2026-08-24 run recorded
none. **The guard settles syntax and not availability.** Its reference page names the same four
products every Kusto reference page names, so this file makes no claim either way about whether the
function is available in Defender XDR advanced hunting, which is the ground this pack applies to
every other Kusto function it uses there. **The residual this shape carried is closed by a run.**
That page exemplifies the function under `project` and not inside an aggregation, so the shape this
query ships is unexemplified there. **On 2026-08-26, in one tenant, this query ran on Defender XDR
advanced hunting and returned rows, against a table whose schema did not carry `Title` on that date.**
So it parses in that position and the fallback is what you get. **That settles the shape and not
availability**, and it is one tenant on one date.

```kusto
// MSD-008 step 2 - narrowed. Fill the value set from your own step-1 output.
// in~ rather than in: the values are ones you paste by hand, and in is case-sensitive,
// which returns zero rows on a casing difference and looks like a clean environment.
// Read the three-failure-modes note below this query before relying on either shape.
let AgentProtectionActionTypes = dynamic([
    // "<ActionType values observed in your tenant - see step 1>"
]);
BehaviorInfo
| where Timestamp > ago(7d)
| where ActionType in~ (AgentProtectionActionTypes)
| project
    Timestamp,
    BehaviorId,
    Description,
    ActionType,
    Categories,
    AttackTechniques,
    ServiceSource,
    DetectionSource,
    DataSources,
    AccountUpn,
    AccountObjectId,
    StartTime,
    EndTime
| order by Timestamp desc
```

**A comment is not a control, so add one.** Deployed with the list still empty, the query above
returns zero rows and looks healthy. If your platform supports it, wrap the rule so an unconfigured
state announces itself instead of failing silently:

```kusto
// Optional self-check wrapper. Emits one explanatory row while the list is empty, and
// returns step 2's thirteen columns plus RuleState once it is filled. The two legs are
// NOT the same shape - branch 1 projects fourteen columns, branch 2 emits RuleState alone.
// The union reference documents kind=outer as the default, so the result is expected to
// carry every column from either leg, with nulls where a row does not define one.
// Verify it parses in your own workspace first - see the note below.
let AgentProtectionActionTypes = dynamic([
    // "<ActionType values observed in your tenant - see step 1>"
]);
union
  (
    BehaviorInfo
    | where Timestamp > ago(7d)
    | where ActionType in~ (AgentProtectionActionTypes)
    | extend RuleState = "configured"
    | project
        Timestamp,
        BehaviorId,
        Description,
        ActionType,
        Categories,
        AttackTechniques,
        ServiceSource,
        DetectionSource,
        DataSources,
        AccountUpn,
        AccountObjectId,
        StartTime,
        EndTime,
        RuleState
  ),
  (
    print RuleState = "RULE UNCONFIGURED - the ActionType list is empty, so this rule cannot match anything"
    | where array_length(AgentProtectionActionTypes) == 0
  )
| order by Timestamp desc
```

**What the step-2 query and the wrapper return stays in your environment.** Both project `AccountUpn`
and `AccountObjectId`, which name a person, and `Description`, which is free text out of
your own tenant: `Description` is documented only as "Description of the behavior", so nothing tells
you in advance what yours will carry. **What travels from this file is a closed list: the `ActionType`,
`ServiceSource`, `DetectionSource` and `Categories` value strings, and nothing else.** That is
verification step 2's rule in its own words, and it governs the step-2 query and the wrapper exactly
as it governs step 1. What travels outward is a column name or a value name, never a
result row, a row count, or anything inside one. Checklist Group 0 states the same rule for every
step in the pack, and this file repeats it because a reader who deploys one detection may never open
the checklist.

> **Three failure modes, and the warnings above cover only one of them.** A placeholder value set
> can fail as (1) a **parse error**, which is the safe case because you see it immediately;
> (2) an **empty list**, which returns zero rows and looks clean - the case the comments warn about;
> or (3) an operator that does not do what you assumed. **Mode 3 is ruled out.** Microsoft's `in`
> operator reference documents a dynamic array as a supported right-hand form for `in`, with a worked
> `let ... dynamic([...])` example, and states "Nested arrays are flattened into a single list of
> values." **The case-insensitive variant these queries ship is documented on its own reference page**,
> which carries a section headed "Dynamic array". Its example passes a `dynamic([...])` literal
> directly to `in~` and publishes the count it returns, and one of the literal's three elements is
> lower-cased where the other two are not. Read 2026-08-18. So the failure direction this pack names
> for a placeholder set, `in~` always false with no error, is excluded by example for the operator
> these queries ship. **What that excludes is silent non-expansion by the operator itself**, and not
> the combination this query ships. A `let`-bound array passed to `in~` is exemplified on neither
> page: the `in` page's worked example binds a `let` for the case-sensitive operator, and the `in~`
> page passes its literal directly. Section 6 of
> [`docs/verification-methodology.md`](../../docs/verification-methodology.md) states that residual.
> **Mode 1 was settled on 2026-08-24, in one tenant, on both surfaces.** A `dynamic([...])`
> literal whose only element is a comment on its own line parsed, `array_length()` over it returned
> zero rather than null, and `in` and `in~` over it both returned false. The one-element control
> passed first on each surface, so the reading is live. **That answer came from one tenant on one
> date.** Run this once in your own workspace before
> relying on either query shape, and run the probe exactly as printed rather than on one line.
> **The control block comes first, and it is printed first.** It is the same statement over a literal
> carrying one element rather than only a comment, so the machinery is exercised independently of the
> question the probe asks: the `let`, `array_length()`, and `in` and `in~` used as scalar expressions
> inside `print`, which is a syntactic position no page this pack cites exemplifies. **If the control
> errors too, the comment-only literal is not what the engine rejected**: record both error texts and
> treat this residual as still open rather than answered.
>
> **Expect the control block to return one row, with `ArrayLength` reading `1`.** The 2026-08-24 run
> records this control passing first on each surface, so a block that errors rather than returning a
> row leaves the probe below settling nothing.
>
> ```kusto
> let Y = dynamic(["a"]);
> print ArrayLength = array_length(Y), Matches = ("a" in (Y)), MatchesCI = ("a" in~ (Y))
> ```
>
> ```kusto
> let X = dynamic([
>     // placeholder
> ]);
> print ArrayLength = array_length(X), Matches = ("a" in (X)), MatchesCI = ("a" in~ (X))
> ```
>
> **`MatchesCI` is the column that speaks to the residual above and `Matches` is the case-sensitive
> control beside it.** This file's queries ship `in~`, so a probe carrying only the case-sensitive
> `in` settles nothing about the operator they use. Checklist Group 1b check 2 carries the identical
> pair and says the same thing there.
>
> **A fourth thing about the wrapper, separate from the three modes.** Its unconfigured branch is a
> `print` statement, so the row it emits carries no `Timestamp` and no table columns at all.
> Microsoft's custom-detection-rules page recommends projecting `Timestamp` for tables outside
> Defender for Endpoint, so that branch does not meet the recommendation and the wrapper is written
> as a hunting-query aid rather than as a rule shape. Checklist Group 1b check 3 tests whether a
> `print` works as a `union` leg at all - and it tests only that, because both of its legs project
> one identically-named, identically-typed column. **How a `union` of two different shapes renders is
> a separate question and it is documented**: the `union` reference states that `kind=outer` "causes
> the result to have all the columns that occur in any of the inputs", that "Cells that aren't
> defined by an input row are set to `null`", and that "The default is `outer`". So the shape
> mismatch above is expected to widen the result rather than to fail, on the page rather than on
> assumption. **Whether these two legs parse together was settled on 2026-08-26**, in one tenant:
> the wrapper ran on Defender XDR advanced hunting and returned a row carrying `RuleState`. **The
> 2026-08-24 run had settled the construct and not the combination**, because the wrapper did not
> run there at all, having referenced `Title`. Removing that reference is what let the pair be
> tested.
>
> **A documented shape exists if you want one before running that check.** The `union` page reaches a
> `print` leg through a named view rather than inline, and its notes state that "The `union` scope
> can include let statements if attributed with the `view` keyword". Defining the unconfigured
> branch as `let Unconfigured = view () { print RuleState = "RULE UNCONFIGURED ..." };` and then using
> `(Unconfigured | where array_length(AgentProtectionActionTypes) == 0)` as the second leg puts the
> same behaviour on a shape that page shows. **That form has not been parsed here either**, and the
> 2026-08-24 run did not test it, and it
> does not change the `array_length()` problem below, which is what decides whether the branch fires
> at all.
>
> **And the unconfigured branch's own guard uses the function this pack rejects everywhere else.**
> That guard is `array_length(AgentProtectionActionTypes) == 0`, evaluated over the same comment-only
> literal whose parse mode 1 settled above. The `array_length()` reference states it
> "Returns the number of elements in array, or `null` if array isn't an array", which is exactly why
> MSD-003 and MSD-004 both decline to use it - and `null == 0` does not evaluate true. **So if that
> literal parses to anything that is not an array, branch 2 does not fire. On 2026-08-26, in one
> tenant, it did fire**: the wrapper returned its unconfigured row on that surface, and that row is
> the observable saying the literal parsed to an array and the guard compared a length rather than a
> null. **One tenant, one date, and it does not replace the check in yours.** What the union returns
> then depends on branch 1, which filters `in~` over the same literal, and **this file does not
> establish how `in~` behaves against a right-hand side that is not an array**, which is the residual
> the operator note above leaves open. If branch 1 also returns nothing, **the wrapper produces
> exactly the silent-empty state it was added to prevent**, and that is the case to rule out rather
> than to assume either way. Group 1b check 2
> already answers this and nothing new is needed: its `ArrayLength` column is the result to read.
> **It came back `0` rather than `null` in the 2026-08-24 run, on both surfaces**, so branch 2
> could fire there. If
> it comes back `null` rather than `0` in your own workspace, do not deploy this wrapper in this
> shape - use the plain step-2 query and the comment, and rely on the value set being filled.

**On `ReportId`.** The same Microsoft page recommends projecting `Timestamp` and `ReportId` together
for tables outside Defender for Endpoint, and MSD-001 records the enrichment it names. **The
`BehaviorInfo` table reference publishes no `ReportId` column**, so that projection cannot be made
from this table and its absence here is a property of the schema rather than an omission. `Timestamp`
is projected by the step-2 query and by the wrapper's configured branch. The step-1 `summarize`
carries `FirstSeen` and `LastSeen` rather than `Timestamp`, and the wrapper's unconfigured branch
carries `RuleState` alone.

**That page also says not to filter on `Timestamp` or `TimeGenerated`, and sets the rule's lookback
from its frequency rather than from the query**, with an exception for narrowing inside the lookback
that the step-2 query's seven-day window relies on. MSD-001 records that in full. **No query block in
this file carries a baseline leg**, so the truncation case MSD-003 and MSD-004 name does not arise
here, and the self-check wrapper is written as a hunting-query aid rather than as a rule shape in any
case.

To pull the entities attached to a behaviour, Microsoft documents a companion table,
`BehaviorEntities`. **Its own reference page was read for this pack on 2026-08-19**, and describes the
table as holding "information about entities (file, process, device, user, and others) that are
involved in a behavior in Microsoft Defender for Cloud Apps and User and Entity Behavior Analytics
(UEBA)". The advanced hunting schema-tables list carries **substantially the same description, with
the GCC qualifier added inline**, which is the difference the next paragraph turns on.

**The AI-agent page describes the same table in different words, and those are the words the
attribution limit below has to be read against.** Its advanced-hunting table list gives the
`BehaviorEntities` row as "Contains
the entities and artifacts associated with behaviors, such as agents, users, tools, and resources",
read 2026-08-19. That row is easy to confuse with the `AlertEvidence` row of the same list, which
uses almost the same words about alerts. **The row is named rather than placed**, because the list's
order is the page's to change and the confusion is about the wording rather than the position. **Read the agent-facing wording beside the attribution limit stated
below**: Microsoft names agents among the entity kinds this table holds, and the table's own
reference page publishes no column that identifies one, so both statements are true and a join still
does not attribute a behaviour to an agent. **This pack ships no join against it**, and a reader who
adds one is working from those two pages rather than from anything verified in a workspace here.

**That read settles the GCC question this file previously left open, and settles it the other way.**
The reference states, verbatim: "The `BehaviorEntities` table is in preview and is not available for
GCC." The exclusion is therefore table-level for `BehaviorEntities` exactly as it is for
`BehaviorInfo`, whose own reference page is quoted under Status evidence below. **The earlier reading
of the schema-tables list was over-cautious rather than wrong:** on that list "(not available for
GCC)" does sit inside the description and does qualify **Microsoft Defender for Cloud Apps**, the data
source, on the `BehaviorEntities` and `BehaviorInfo` rows alike, so the list on its own never settled
the table's own status. The reference does. **So the answer you recorded in step 1 carries over to
this table**, and a GCC environment has neither.

## Status evidence

**Public Preview, with a platform exclusion.**

- The `BehaviorInfo` table reference page title reads "BehaviorInfo (Preview)" and states, verbatim:
  "The `BehaviorInfo` table is in preview and **is not available for GCC**. The information here may
  be substantially modified before it's commercially released."
- The link to AI agents comes from the AI-agent feature page, verbatim: "Real-time protection audit
  and block events are recorded as behaviors in the `BehaviorInfo` table, which you can correlate
  with alerts during investigation." That page is titled "… (Preview)" and states "This feature is
  currently in public preview."
- The same page carries a coverage note, verbatim: "Block events from Microsoft Prompt Shields for
  Foundry and Microsoft Copilot Agent Builder are also recorded as behaviors. This isn't yet
  supported for agents built with Microsoft Copilot Studio."

That last quote is the operationally important one: **Copilot Studio agents are named as not yet
supported for behaviour recording.** If your agent estate is mostly Copilot Studio, this detection
covers less of it than the table's presence suggests.

## What this detection cannot see

- **Copilot Studio agent block events**, per the Microsoft quote above, at the verification date.
- Whether block events from unpublished Foundry agents reach this table. The AI-agent detection
  page states, under its prerequisites, that "Threat detection is supported only for published
  Microsoft Foundry agents. Unpublished agents, including agents used only in a playground
  environment, aren't supported." That sentence scopes threat detection rather than the behaviour
  route this file reads, and the real-time protection page, which describes Prompt Shields block
  events being recorded as behaviours, states no such limit. Both read 2026-09-27. So whether a
  playground-only agent's block events reach `BehaviorInfo` is not established here, and a positive
  control is better generated from a published agent.
- **Which behaviours are AI-related, without your own step-1 work.** No documented value marks them.
- **Microsoft 365 Copilot chat prompts.** No documented advanced-hunting surface exists for them;
  see [`docs/verification-methodology.md`](../../docs/verification-methodology.md) for the scope of
  that check.
- **Anything in a GCC environment.** Learn states the table is not available there.
- **Agent activity where real-time protection is not enabled.** Behaviours record protection
  events. Absence of a behaviour is not evidence of absence of activity. **Agent runtime activity is
  a different surface**: Microsoft documents `CloudAppEvents` as carrying "Agent 365 observability
  data for AI agent activity, including agent actions, tool invocations, and data access events",
  in the advanced-hunting table list on the
  [AI agent detection and protection page](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/ai-agent-detection-protection)
  and not on the `CloudAppEvents` table reference,
  and **this pack does not build on that table in v0.1** - see
  [`docs/scope-and-out-of-scope.md`](../../docs/scope-and-out-of-scope.md). Do not reach for
  MSD-003 or MSD-004 here; both read a configuration table and neither can answer what an agent did.
- **Which agent produced a behaviour.** `BehaviorInfo` carries no agent identifier. **The identity
  columns this file reproduces** are `AccountUpn` and `AccountObjectId`, and Learn does not document
  what populates them for an agent-initiated behaviour. The published table carries one further
  identity column that the deliberate subset above omits, `DeviceId`, which at best narrows an
  agent running on an endpoint to the device it ran on and does not name the agent.
  **`BehaviorEntities` does not close this either.** A full-text search of that table's reference
  article body on 2026-08-19 returned **zero occurrences of "agent"**, and the same search over the
  same article body returned **25 occurrences of "behavior"**, which is the positive control that
  shows the search ran. **Both figures are over the article body rather than the whole page**, which
  is the scope the search covered and the only scope they speak to. **Both count case-insensitive
  substring matches rather than whole words**, so a match inside a longer identifier counts, and
  both are over the rendered article rather than its source. The conventions are named because a
  figure stated without them is not reproducible, and reproducibility is the standard this file sets
  for itself; **a count made a different way is a different instrument rather than a correction.**
  Neither reference **documents a
  column as carrying an agent identifier**, so nothing either page publishes lets a join attribute a
  behaviour to the agent that produced it. Its entity-type and entity-role columns are the place a
  reader would look, and what they carry for an agent-initiated behaviour is undocumented and
  unverified here. **This is arguably the most consequential limit in the file:** a behaviour you
  cannot attribute is hard to action, and neither reference page resolves it.

## False-positive guidance

- **This table is not an AI table.** Learn states it "contains information about behaviors from
  Microsoft Defender for Cloud Apps and User and Entity Behavior Analytics (UEBA)". Without the
  step-2 narrowing, the query returns your entire behaviour stream and nothing about it is
  AI-specific.
- **Audit events are not block events.** The feature page names both. An audit-mode behaviour
  records that a rule matched and did not act. Treating the two the same will overstate
  enforcement.
- **A behaviour is explicitly not a verdict.** Learn: "Behaviors provide contextual insight into
  events and can, but not necessarily, indicate malicious activity."
- **`AttackTechniques` here is MITRE ATT&CK, not ATLAS.** The ATLAS mapping in this file is this
  pack's synthesis. Do not read the column as agreeing with it.

## Workspace verification before deployment

1. Confirm the table exists and its columns match this file:
   `BehaviorInfo | getschema | project ColumnName, ColumnType`. **Diff that output against the schema
   table above.** Record as a **schema mismatch** any column the schema table lists which the output
   does not carry, or which the output types differently. **A column the output carries and the schema
   table does not is not a mismatch**, because the schema table is a deliberate subset. Then
   `BehaviorInfo | take 10`. Confirm your environment is not GCC.
2. Run the step-1 discovery query over the **full retention window** and record the complete
   `ActionType` / `ServiceSource` / `DetectionSource` / `Categories` inventory. **`Categories` came
   back as a serialised array string in one environment**, so the values this rollup returns for
   that column may be combinations rather than category names. **Record them as the shape you
   found, and do not build an equality filter from one**: such a filter matches only the rows
   carrying that exact combination and silently misses every other combination containing the same
   category. Thirty days is a
   ceiling here rather than a floor: the advanced hunting overview states "Each query can look up
   native Defender XDR data from up to the past 30 days", and extends that range only where a
   Microsoft Sentinel workspace is onboarded and its analytics-tier retention is longer. Confirm which
   case yours is before treating the inventory as complete. **The output of this step stays in your
   environment.** The step-1 query returns up to twenty-five free-text behaviour labels from
   your own tenant in `SampleLabels` **for each row it returns**, and it returns one row per
   `ActionType` / `ServiceSource` / `DetectionSource` / `Categories` combination, so twenty-five is a
   per-row cap and not a ceiling on the total. `Title` and `Description` are documented only as
   "Title of the behavior" and "Description of the behavior", so nothing tells you in advance what
   yours will carry. **What travels from this
   step is a closed list: the `ActionType`, `ServiceSource`, `DetectionSource` and `Categories` value
   strings, and nothing else.** Not `SampleLabels`, not the behaviour count, and not the
   first-seen and last-seen timestamps - the count and the dates describe your estate rather than the
   schema.
3. Run the comment-only `dynamic([...])` parse test in the step-2 note above, once, exactly as it is
   printed there, and record the result. It decides which of the two query shapes is safe to deploy.
   **If you intend to use the self-check wrapper, run checklist Group 1b check 3 as well** - it
   tests whether a `print` statement works as a leg of a `union`, which is the wrapper's other
   construct. **The 2026-08-24 run parsed it on both surfaces**, in the same-shape form that
   check ships, which is not the wrapper's two-shape pair.
4. Generate a known AI-agent protection event in a lab tenant and locate it in the step-1 output.
   Without a positive control you cannot distinguish "no events" from "wrong filter".
5. Record the value set you identified, with the date, next to your deployed rule. It is not
   documented, so it can change without notice.
6. Read the `BehaviorEntities` table reference before adding any join, for its column list rather
   than for its status. **Both status questions are already settled against that reference**, quoted
   in the entity-join note above: the table is in preview, and it is not available for GCC at the
   table level, so the answer you recorded in step 1 carries over to it. What the reference is for
   here is the columns a join would return. **One run resolved the table and recorded three of its
   columns**; nothing further about it has been checked against a workspace.
   **The concrete question to take to it is attribution.** Run the join in a lab tenant against a
   behaviour you generated in step 4, and record what its entity-type and entity-role columns carry
   for an agent-initiated behaviour. That is the one thing that would close the limit stated above,
   and no page this pack read settles it.

## Sources

- [BehaviorInfo table in the advanced hunting schema (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-behaviorinfo-table) - last verified 2026-08-15, **re-read 2026-08-24** after one tenant's schema did not carry `Title`. That read confirmed the column is still published, quoted verbatim in the schema table above, and is also the source of the sentence that the table is populated from Defender for Cloud Apps and UEBA, and of the deployment sentence beginning "If your organization doesn't deploy these services in Microsoft Defender", which is quoted in full where it is used rather than paraphrased into a condition the page does not state. Page stamp on that read: `ms.date` 2026-01-12, `updated_at` 2026-06-14
- [Advanced hunting schema tables (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-schema-tables) - last verified 2026-08-18, for the `BehaviorEntities` preview tag, and for where the GCC qualifier sits on that row
- [BehaviorEntities table in the advanced hunting schema (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-behaviorentities-table) - last verified 2026-08-19, for the table description quoted in the entity-join note above, for its verbatim table-level preview and GCC statement, and for the absence of any agent identifier in its published column list, which is recorded under what this detection cannot see. **This pack ships no join against this table**, so the entry records what was read rather than a column this pack relies on
- [Detect and investigate threats to AI agents using Microsoft Defender (Preview) (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/ai-agent-detection-protection) - last verified 2026-08-15 for the real-time-protection sentence and the Copilot Studio coverage note quoted under Status evidence, **re-read 2026-09-20, when the rendered "Last updated on" date read 2026-09-03 and the page's `ms.date` read 2026-08-07**, and the product name in that coverage note read `Microsoft Copilot Agent Builder` rather than the form this file carried before that read. The Copilot Studio half of the note was unchanged on that read, and re-read 2026-08-19 for the `BehaviorEntities` row of its advanced-hunting table list, quoted in the entity-join note above, and re-read again 2026-08-23 for the `CloudAppEvents` row of that same table list, which is where the quotation in the scope note above sits. **Each date is kept rather than collapsed**, because the later read is what added that row. Re-read 2026-09-27, `ms.date` 2026-08-07, for the published-agents sentence under its prerequisites, quoted under what this detection cannot see
- [Protect AI agents in real time using Microsoft Defender (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/ai-agent-real-time-protection) - read 2026-09-27, `ms.date` 2026-07-01, for its "How real-time protection works" section, which carries the same Prompt Shields note as the detection page and states no published-agents limit
- [`in` operator (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/in-cs-operator) - last verified 2026-08-15, which documents dynamic-array expansion and the `in` / `in~` case-sensitivity pair
- [`in~` operator (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/in-operator) - last verified 2026-08-18, for the "Dynamic array" section, whose example passes a `dynamic([...])` literal directly to `in~` and publishes its output. It does not exemplify the `let`-bound form this file's step-2 query and self-check wrapper ship, and the `let` variant beside it substitutes another operator; [`docs/verification-methodology.md`](../../docs/verification-methodology.md) section 6 states what that leaves open
- [`column_ifexists()` (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/column-ifexists-function) - last verified 2026-08-25, for the syntax `column_ifexists(columnName, defaultValue)`, for both arguments being required, for the default column being returned where the named column does not exist, and for the deprecated alias `columnifexists()` this file does not use. **This is the page step 1's `Title` guard rests on.** Its "Applies to" line names the same four products every Kusto reference page names, so it settles the syntax and not whether the function is available in Defender XDR advanced hunting, and its one worked example applies the function under `project` rather than inside an aggregation
- [`array_length()` (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/array-length-function) - last verified 2026-08-16, for the null-on-non-array behaviour that the wrapper's unconfigured guard depends on
- [`union` operator (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/union-operator) - last verified 2026-08-17, for `kind=outer` being the default, for what the result carries when the legs do not share a schema, and for the `view`-form examples described below
- [`print` operator (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/print-operator) - last verified 2026-08-17, for `print` returning "A table with one or more columns and a single row". **The `union` page does show a `print` as a `union` leg, in the `view` form**: its `isfuzzy` example defines `let View_1 = view () { print x=1 };` and then uses `(View_1 | where x > 0)` as a leg, and its type-mismatch example unions two `print`-derived views that do not share a schema. **What that page does not show is the wrapper's inline form**, a parenthesised `print` used directly as a leg, and the same page notes that "The `union` scope can include let statements if attributed with the `view` keyword". So whether the inline form parses is still open - Group 1b check 3 is what settles it, and the note above gives the `view`-based shape for an operator who wants a documented one first
- [Advanced hunting overview (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-overview) - last verified 2026-08-16, for the 30-day query date range and how a Microsoft Sentinel workspace extends it
- [Create custom detection rules in Microsoft Defender XDR (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/custom-detection-rules) - last verified 2026-08-16, for the recommended `Timestamp` and `ReportId` projection this table cannot fully satisfy
- MITRE ATLAS technique IDs read from the distributed `atlas-data` dataset, `version: 5.6.0` (release tag `v2026.07`) - verified 2026-08-15
- OWASP Top 10 for LLM Applications item numbering carried from the companion capability-status matrix cross-walk, verified there by SHA-256 against OWASP's published download on 2026-08-09; the 2026 edition and its publication date re-confirmed 2026-08-15
