# How each detection in this pack was verified

This document exists so a reader can audit the pack rather than trust it. It records what was
checked, how, on what date, and **what the checks do not establish.**

**First source pass for every item below: 2026-08-15.** Several statements in this pack now rest
on later reads instead; each is recorded against its own Sources entry and in the CHANGELOG, and
those dates rather than this one are what the rule below points at.

---

## 1. The rule

Every table, column, value string and status label **that a detection file or a shipped query relies
on**, as opposed to one it merely reports, is either:

- **quoted from a Microsoft Learn page read on a stated date**, with the URL and that date recorded
  where the file that establishes the quotation records its sources, whether that is a Sources
  section or a source line beside the quotation itself; or
- **marked Provisional or Requires further validation**, with the reason stated and a workspace step
  that resolves it; or
- **labelled at the point of use as observed in a tenant on a stated date rather than as
  documented.** **This third class is scoped more narrowly than the sentence above it**, and
  deliberately: it is available to a value **a shipped query acts on**, because that is the value
  that can change what a detection returns without anyone noticing. **Its members are named here
  rather than counted, and there is one**: the `Junk folder`
  spelling in MSD-002's `DeliveryLocation` filter, and in the bucket expression of the rollup below
  that filter, which a tenant emitted on 2026-08-24 and which
  Microsoft Learn does not publish. It is kept beside the published spelling so the filter matches
  either, and MSD-002 labels it inside both of its query blocks. **A member of this class may widen a
  filter and may never narrow one**, because an observed value that removed rows would be the silent
  failure this pack argues against, introduced from a single tenant. **That bound reaches an
  aggregation on the same column as well as a predicate on it**, and it has to: a filter widened to
  admit two spellings of one value splits any group keyed on that column, which understates whatever
  the grouping ranks. MSD-002's rollup normalises its grouping key for that reason.

There is no fourth category for anything a shipped query acts on. A column name or a value string
that appears in a query and is none of the three is a defect - report it.

**Prose is a different matter, and the scope sentence above says so with "relies on rather than
merely reports".** A detection file may report what a tenant emitted, with the tenant-and-date
bound beside it, without that value joining any of the three classes, because nothing in the pack
acts on it. `Forwarded` and `Unknown` in MSD-002 and `notApplied` in MSD-006 are reported that way.
**They are named here so the enumeration above is not read as short by three**, and any one of them
would owe the point-of-use label the moment a shipped query acted on it.

**This section is the canonical statement of that rule, and the scope above is part of it.** The
README and the pull-request template each restate the rule more briefly; if either ever disagrees
with this section, this section governs, and the difference is a defect to report rather than a
nuance to interpret. The scope is what decides marginal cases, including whether the `CloudAppEvents`
candidate query in [`scope-and-out-of-scope.md`](scope-and-out-of-scope.md) is inside the rule. It
is: it is a shipped query.

**The classes of example value this pack ships are enumerated in
[`scope-and-out-of-scope.md`](scope-and-out-of-scope.md), which marks for each whether it sits
inside the rule above, and that enumeration is the canonical one.** Its first class sits inside the
rule rather than outside it, so read the shipped classes as a taxonomy of what the pack ships rather
than as a list of exceptions. **Its last class is not one this pack ships**: it is the value an
inbound contributor originates, and it is in that list because the same list answers the templates'
question about what a contributor may send. Where this section and that list disagree about which
classes exist, the list governs; what the two categories above require of each shipped class is
unchanged. **One of the classes
that sits outside the rule is named here rather than left to be discovered**, because a reader of
this section meets it first. The
workspace verification report template and [`SECURITY.md`](../SECURITY.md) carry an invented
`ActionType` illustration, because no documented value exists for that column in this scope and a
template has to show a contributor the shape to send. **Those two files and no others**: the other
three contribution templates point at the illustration to tell a contributor not to send it back,
and do not carry it. **It is marked as invented at every point of use.** That is the
rule above applied to a different problem rather than an exception to it: what the rule protects is
that nothing unsourced ships unmarked, and an illustration that says it is one does not breach it.

### Why the rule is stated this strictly

The companion repository, `microsoft-ai-security-control-plane`, had a technical review flag
column names such as `DataExfiltrationDetected` and `AbnormalCopilotBehavior` as
plausible-but-fabricated `CloudAppEvents` fields. They read exactly like real schema. Neither is a
column of that table, whose reference was re-read on 2026-09-27, and a Microsoft Learn site search
for each name returned no page on the same date. A detection engineer running a query built on
invented columns learns nothing except that the author did not check.

> **Correction, 2026-09-27.** An earlier version of this passage named a third column,
> `XPIADetected`, and said all three were not real schema. That was wrong for `XPIADetected`. It is
> not a `CloudAppEvents` column, but Microsoft documents it as a property of `AccessedResources` in
> Copilot audit records:
> "XPIADetected is a boolean that denotes whether there was an XPIA (Cross Prompt Injection Attack)
> detected from a particular resource which Copilot accessed"
> ([Audit logs for Copilot and AI applications](https://learn.microsoft.com/en-us/purview/audit-copilot),
> `ms.date` 2026-08-26, read 2026-09-27). The same site search that returned no page for the other
> two names returned that page for this one. A property nested inside a `dynamic` payload is also,
> on this pack's reading, the silent case rather than the loud one, because a wrong key path there
> does not fail the query; no Kusto reference page read on 2026-09-27 states that, and no run of
> this pack has tested it. `CHANGELOG.md` is the record.

That is the failure this pack is built to avoid, and it is why several queries here are less
convenient than they could be: the shipped query does not filter on a value Microsoft has not
published, even when a plausible guess would produce a tidier rule.

**Two further standards this pack holds itself to, both about the ground a conclusion rests on
rather than about schema.**

**An outcome that is accidentally correct is not evidence.** A conclusion can be right for a reason
that does not license it, and a reading taken at the wrong scope is the ordinary way that happens.
Where such a reading is redone at the scope the claim actually needs and the answer does not move,
what changed is the evidence and not the answer, and the record says so rather than quietly keeping
the original wording. A conclusion here carries the ground it rests on.

**An estimate this pack disowns in one place cannot be a threshold it relies on in another.** Where
an interval or a latency is recorded as this pack's own estimate rather than as a figure Microsoft
publishes, no later claim may lean on it as though it were documented. Elapsed time can make a late
arrival less likely; it cannot license a claim that something is absent. What licenses an absence
claim is a search whose scope is stated and whose control fires.

---

## 2. How status labels were assigned

| Label | Rule |
|---|---|
| **GA (stated)** | Microsoft Learn asserts general availability outright - a release-state row, or a release-note sentence naming the GA date. |
| **GA (no preview qualifier)** | The documenting page carries no preview qualifier and no release-state sentence of any kind. **This is inference from absence and it is a weaker class of evidence.** |
| **Public Preview** | Learn carries a `(Preview)` qualifier in the page or table title, or an explicit public-preview sentence. |
| **Provisional** | The table or capability has a status, but a specific column, value set, or serialisation used by the query is undocumented. Scoped to the element, not the detection. |
| **Requires further validation** | Microsoft's own sources conflict, or the status cannot be established from primary sources. Never guessed, never averaged. |

**This is the canonical legend for the pack.** The README reproduces it and every detection file
refers to it; if the two ever disagree, this one governs. Two surfaces in one repository shipping
two spellings of one idea would make a reader wonder whether they mean different things.

**The label vocabulary matches the published companion repository**, which uses "Requires further
validation" and never the shorter form. The *definitions* here are this pack's own and are narrower
than the companion's, because a detection pack has to distinguish a stated GA from an inferred one
and a capability matrix does not.

**Why GA is split, and why the split is the argument rather than a technicality.** Section 3.2 below
argues, correctly, that a scope boundary evidenced by positive enumeration is not the same as a
stated exclusion. A single GA label folding both evidence classes together would apply the opposite
standard one page later: it would let inference from absence and an explicit Microsoft assertion
print identically, and a reader could not tell which they were holding. **In this pack exactly one
detection carries GA (stated)** - MSD-005, where Learn publishes a Release state row. MSD-001,
MSD-002 and MSD-006 carry GA (no preview qualifier). The distinction costs two words and it is what
the pack is for.

**A status is never inferred from a product blog, a launch announcement, or a Tech Community post.**
Those are official-adjacent context and can never set a status here.

---

## 3. The negative claims, and exactly how far each one reaches

Every "Microsoft does not document X" statement in this pack is scoped to named pages. An unscoped
absence claim is not verifiable and is not made.

### 3.1 "There is no advanced-hunting surface for Microsoft 365 Copilot chat prompts"

This is the load-bearing negative claim of the whole pack, so its scope is stated in full.

**Pages read** - seven, all on 2026-08-15: the Defender XDR advanced hunting overview, the
advanced-hunting schema table list, the `CloudAppEvents` table reference, the `EmailEvents` table
reference, the `AgentsInfo` table reference, the `BehaviorInfo` table reference, and the AI-agent
detection and protection page.

**What was found:** no table, column, or page in that set documents Microsoft 365 Copilot user
prompts or Copilot chat interactions. Two of the seven carry the weight, and both were checked by
full-text search rather than by reading:

- The **advanced hunting overview** contains **zero occurrences of "Copilot"** and **zero of
  "prompt"**.
- The **advanced-hunting schema table list** - the page that enumerates every advanced hunting
  table - contains exactly **one** occurrence of "Copilot", and it refers to Copilot **Studio**
  agents: the `AIAgentsInfo` entry, described there as "Information about AI agents created with
  Microsoft Copilot Studio, including agent configuration and ownership details". **No entry in
  that enumeration is a Copilot chat-prompt table.**

**Both of those two were re-read on 2026-08-17 and both results are unchanged.** The advanced hunting
overview still contains zero occurrences of "Copilot" and zero of "prompt", at a rendered "Last
updated on" date of 2026-08-07. The schema table list still contains exactly one occurrence of
"Copilot", still the `AIAgentsInfo` entry and still describing Copilot **Studio** agents in the
words quoted above, at a rendered "Last updated on" date of 2026-07-27. **Both figures were
re-derived against the live pages on 2026-09-20 and both still reproduce**, and on each of these
two pages the rendered date and the `ms.date` field agree, so neither figure distinguishes them.
No entry in that enumeration is described as carrying Microsoft 365 Copilot user prompts or chat
interactions. The re-read is recorded because this is the claim the most rests
on, and because a second dated read of the two pages that carry the weight costs nothing. Both reads
are searches of the page **text**, which is the same instrument as the 2026-08-15 read and is what an
absence claim of this kind can rest on.

That is a stronger form of the evidence than "we looked and did not see it": the schema list is an
enumeration of the full table set, so an absence from it carries the same weight as the positive
enumeration argued for in 3.2 below. What exists is two adjacent surfaces on different release
states, and this pack builds on both:

- **Prompt injection carried in email**, through Microsoft Defender for Office 365. It reaches
  advanced hunting as a detection-technology value, and its page carries no preview qualifier -
  [MSD-001](../detections/defender-xdr/MSD-001-prompt-injection-email-detected.md) and
  [MSD-002](../detections/defender-xdr/MSD-002-prompt-injection-email-delivered.md).
- **AI-agent telemetry**, explicitly public preview -
  [MSD-003](../detections/defender-xdr/MSD-003-agents-with-mcp-servers.md),
  [MSD-004](../detections/defender-xdr/MSD-004-broad-agents-without-guardrails.md) and
  [MSD-008](../detections/defender-xdr/MSD-008-agent-runtime-protection-behaviours.md).

**How far the claim reaches:** it is bounded to those pages, read on the verification date. It is
strong evidence, **not proof of absence across Microsoft's whole documentation set**, and it is not
a claim about what any product can do internally. If you find a page that documents such a surface,
that is a correction to file, not an argument.

**One adjacent surface that is easy to mistake for a refutation.** The Sentinel `CopilotActivity`
table does carry Copilot audit activity, including a `CopilotInteraction` record type. It is a
Log Analytics audit table populated through the Office Management API - not a Defender advanced
hunting table - and Microsoft does not document that its `LLMEventData` column carries prompt text.
[MSD-007](../detections/sentinel/MSD-007-copilot-settings-change.md) states this in its opening
lines for that reason.

### 3.2 "Defender for Cloud AI threat protection does not cover Microsoft 365 Copilot or GitHub Copilot"

**How it was checked:** the full text of the AI threat protection availability page was searched for
the strings "Microsoft 365 Copilot" and "GitHub Copilot". **Neither appears.** The page carries an
availability table row labelled **"Supported AI services"** whose service entries are Azure OpenAI
supported models and Azure AI Model Inference service supported models, followed in the same cell by
"Defender for Cloud currently supports text tokens only. Image and audio tokens aren't scanned."

**The enumeration is complete and scoped to that label**, which is what makes the argument work. This
is not "we searched and found nothing"; it is "Microsoft published a list of what it supports, under a
label that says so, and neither Copilot is on it."

**What that establishes:** the boundary is evidenced by **positive enumeration**, not by a stated
exclusion. Microsoft does not deny coverage on that page; it lists what it covers and neither
Copilot is on the list. [MSD-005](../detections/sentinel/MSD-005-defender-for-cloud-ai-alerts.md)
phrases it that way deliberately.

### 3.3 Undocumented elements, by detection

**Every detection in this pack depends on at least one element Microsoft does not publish.** That is
the headline, and it is why this table is derived from the detection files rather than summarised
into a number. Each element is handled by shipping a discovery query, a shape-independent
predicate, or a projected-but-never-filtered column - never by a guess.

| Element | Detection | Learn says | How the pack handles it |
|---|---|---|---|
| `DetectionMethods` serialisation | MSD-001, MSD-002 | Types the column `string`; publishes no value format | `has`, with a documented operator-selection step |
| `LatestDeliveryAction` / `LatestDeliveryLocation` value lists | MSD-002 | Columns and descriptions published; no value lists | Projected, never filtered on |
| `McpServers` internal shape | MSD-003, MSD-004 | `dynamic`; no field names published | String-form test, never indexed into |
| `Guardrails` internal shape | MSD-004 | `dynamic`; no field names published | String-form test against explicit empty forms |
| `DeclaredTools` internal shape | MSD-004 | `dynamic`; no field names published | String-form test |
| `Endpoints` internal shape | MSD-004 | `dynamic`; describes what it holds, publishes no field names | String-form test in the posture rollup, never indexed into |
| `Availability` value list | MSD-003, MSD-004 | Describes the column; enumerates no values | Projected, never filtered on |
| `AgentsInfo` emission cadence | MSD-003, MSD-004 | Not documented at all | 30-day window plus a cadence-measuring verification step |
| Which `SecurityAlert` column carries the alert identifier | MSD-005 | Column names published with **empty description cells** | Matches on both `AlertName` and `AlertType` |
| `ConditionalAccessStatus` values | MSD-006 | "Status of all the conditionalAccess policies related to the sign-in" - no value list | Grouped by, never filtered on; operator completes the filter. The page still publishes no value list, so the handling stands |
| `ResultType` stored values | MSD-006 | Describes semantics ("Success or Failure"), not stored strings | Not filtered on. A tenant stored numeric codes rather than the described words on 2026-08-24, which is the one of these three worth reading before you write an equality filter on the column |
| `Agent` column shape | MSD-006 | One sentence: "Details of agentic sign-in." | Projected, never filtered on. A tenant returned a JSON object on 2026-08-24; the page still publishes no shape |
| `ConditionalAccessPolicies` / `LocationDetails` serialisation | MSD-006 | Typed `string`, with composite descriptions and no published value format | Flagged Provisional; inspect before filtering. A tenant returned a JSON array and a JSON object respectively on 2026-08-24; the page still publishes no format, so the label stands |
| `CopilotActivity.RecordType` full value set | MSD-007 | Two examples behind an "e.g." | Step 2 is an **exclusion**, so a new record type appears without a rule change |
| `LLMEventData` contents | MSD-007 | "Parsed LLM event data" - no schema | Not read; the file forbids building prompt detection on it |
| `BehaviorInfo.ActionType` values for AI-agent protection | MSD-008 | "Type of behavior" - no value list | Discovery query first; operator completes the filter |
| `ServiceSource` / `DetectionSource` values | MSD-008 | Described, not enumerated | Grouped by in discovery, never filtered on |
| `BehaviorInfo.Categories` serialisation | MSD-008 | Types the column, publishes no value format | Grouped by in discovery, never filtered on; one environment returned a serialised array string, so the rollup keys on combinations |

**Eighteen rows, across eight detections.** The unit is deliberate: this table has one row per
element-and-handling, so a row covering two columns is one row here and two elements there. Three
rows do that, which is why the row count and any element count differ.

**How the per-file Provisional counts in the detection headers are derived**, because they are
counted on a different unit again and will not add up to the row count above. A header count is the
number of distinct **elements** that file marks Provisional - one per column, value set or
serialisation - not the number of blockquotes carrying them. MSD-006's two serialisation columns
share one blockquote and count as two. **The converse case occurs too, and it is MSD-008's: where a
single undocumented value set spans several columns, it counts once rather than once per column.**
That file's first Provisional names `ActionType`, `ServiceSource` and `DetectionSource` together,
because what is undocumented is one value set rather than three, so its header reads two elements
and not four. **Read "one per column" as the common case rather than as the rule**; the unit is the
documentation gap, and a column is only the usual shape of one. An element the file names as
unestablished without applying
the Provisional label, such as whether a given function runs on a given deployment target, is not
in the count: the label scopes a documentation gap in a column, and that is a different thing.

---

## 4. The MITRE ATLAS mappings

ATLAS technique IDs were **read from MITRE's distributed `atlas-data` dataset, not recalled**.

- **Artefact:** `dist/ATLAS.yaml` at release tag `v2026.07`
- **Dataset version field:** `version: 5.6.0`
- **Parsed:** 170 technique id/name pairs
- **Verified:** 2026-08-15

Every ID this pack cites was matched against that parse. **This uses the same reading of *cites* as
the v6 check below**, being the identifiers this pack carries as citations rather than every ATLAS
identifier that appears anywhere in these files: **`AML.T0115` is named below to illustrate the
restructure rather than cited**, and its absence from this parse is what makes it an illustration.

**What that artefact says about itself, disclosed because it bears on what this pack uses it for.**
`dist/ATLAS.yaml` at that tag opens with the comment `# This version of the ATLAS data is deprecated
and is no longer being updated with new content.`, and the same release tag ships a `dist/v6`
directory whose emission carries `format-version: 6.0.0` and a collection version of `2026.07`.
**The restructure is demonstrated rather than hypothetical:** `AML.T0115` is declared in
`dist/v6/ATLAS-2026.07.yaml` and appears nowhere in `dist/ATLAS.yaml`. **No cell in this pack was
re-derived against `dist/v6`, and no cited identifier or title was changed on it.** What was
checked, on 2026-08-23, is narrower: **each of the 17 ATLAS identifiers this pack cites anywhere is
also declared as an `id:` entry in `dist/v6/ATLAS-2026.07.yaml`, 17 of 17**, among 299
identifiers declared there. **That set is what the pack cites rather than what it maps**: it
includes the identifiers the cross-walk's not-mapped table records and no detection cell carries,
which is the point of naming them there. **`AML.T0115` is named just above as an illustration of the restructure
and is not one of those 17.** Re-deriving the mappings against the v6 emission is work this pack has
not done, and a reader who does it should expect the deprecated artefact and the successor to
disagree about techniques this pack does not cite.

**The mapping itself is this pack's synthesis.** MITRE did not produce it and does not endorse it.
Which technique a Microsoft detection surface corresponds to is a judgement, and it is the part of
this pack most worth disagreeing with. The cross-walk names its own weakest cell.

## 5. The OWASP mappings

OWASP items use the **2026 edition** numbering (LLM01:2026 - LLM10:2026).

**Provenance, stated because it is a carry rather than a direct read.** The edition's existence and
publication date were confirmed against the OWASP GenAI Security Project resource page on
2026-08-15: "OWASP GenAI LLM Top 10 2026", dated August 3, 2026. **That page was re-read on
2026-08-23 and carries the same title and the same date.** **The artefact itself is dated
differently, and both are recorded rather than reconciled**: its cover carries
`[Publication date to be set]` above `August 4th, 2026`, and its revision history leaves the 2026
release date as an unfilled placeholder. **So August 3 is the resource page's date and not the
document's**, and the version parenthetical in the crosswalk carries the label OWASP distributes the
file under rather than the label the document gives itself, which is `Version 2026`.
**The per-item numbering** is
carried from the published companion capability-status matrix repository's cross-walk, which
verified the artefact by SHA-256 against OWASP's published download in a pass dated 2026-08-09. Each
detection file's Sources section records the same chain.

**One thing to expect if you check this yourself:** the `genai.owasp.org/llm-top-10` landing page
still presented the **2025** edition when re-read on 2026-08-23, listing its ten items as LLM01:2025
through LLM10:2025.
The 2026 edition is on the project's resource page. That discrepancy is why the numbering here is
attributed to a checksum-verified artefact rather than to a web page.

Only LLM01 Prompt Injection and LLM02 Sensitive Information Disclosure kept the numbers they held in
2025, and **the companion cross-walk's renumbering table records that every other slot changed
occupant** - that universal is carried from it, not independently derived here. A 2025-era `LLM0x`
ID must never be carried over unchanged.

---

## 6. What the runs established, and what verification still does not include

Stated plainly, because the gap is the reason the release gate exists.

- **This author has run the queries in this pack three times, on 2026-08-24, 2026-08-26 and
  2026-09-11** - against a Microsoft Sentinel workspace and Microsoft Defender XDR advanced hunting.
  **This pack does not identify any environment it ran against.** **No detection
  was observed firing**, so every query remains a schema-verified construction rather than an
  observed result.
- **The KQL has been run against an engine on 2026-08-24, 2026-08-26 and 2026-09-11.** **Each
  shipped query was submitted on the surface its own file names as its deployment target**, rather
  than on both, and **submitted is not the same as ran**. **Two schema failures are on the record
  and they are not on the same footing**: one table did not resolve on any run this pack records,
  while one column failed on the first run only and its three deployment queries parsed and ran on
  a later one. Each detection file records what its own queries did, because that record is what
  its own hedges rest on.
  **The construct checks listed below did
  not all reach both surfaces either.** Of the five listed, three were checked on both,
  `hash_sha256()` in Defender XDR advanced hunting only, and fingerprint stability was not reached
  at all. **Read the five entries below for what each one actually got rather than this sentence for
  a coverage figure.** Column names and
  value strings are still verified against Microsoft Learn rather than against that tenant, and
  parsing is a separate question from being correct. Several constructs were explicitly unverified
  before that run and each is flagged where it is used rather than counted here, because a count is
  one more derived number to keep true. Named in place, with what the run did to each:
  - whether a `dynamic([...])` literal whose only
    element is a comment on its own line parses at all, which MSD-008's step-2 note and checklist
    Group 1b both print as a runnable block rather than inline, because written on one line the `//`
    would run to the end of it and swallow the closing `])` - **this pack's reading of line-comment
    behaviour rather than something a cited page states** (**MSD-006**, MSD-008 and checklist Group 1b:
    MSD-006's step-2 `NoPolicyApplied` list is the same comment-only form, and MSD-006's own prose
    names both outcomes in the paragraph under that query, while the comment inside the block names
    neither). **Answered on both surfaces: it parses**, and `array_length()` over it returned zero
    rather than null, with the one-element control passing first on each.
  - whether a
    `print` statement works as a leg of a `union`, which MSD-008's optional self-check wrapper depends
    on (MSD-008 and Group 1b). **Answered on both surfaces: it works**, for the same-shape pair that
    check ships. **The wrapper's own two-shape pair was answered later, on 2026-08-26 and on the
    Defender XDR surface only: the wrapper parsed and returned its unconfigured row**, which is also
    the observable that its length guard compared against a number rather than against a null.
  - whether `hash_sha256()` runs in Defender XDR advanced hunting, which
    none of the pages this pack read settles either way (MSD-003 and
    Group 1b). **Answered for one tenant on one date, and on the Defender XDR surface only: it ran
    without error and returned a 64-character hexadecimal digest**. That function's reference page
    describes its return as a hex string and states no length; the length is the one every digest
    in that page's own worked examples carries. **No comparand is published for that check and none
    is asserted here**: the cited page carries worked digests for other inputs, not for the string
    this pack's
    statement hashes, so what the run establishes is that the function resolved and returned a digest
    of the documented shape rather than that any particular value came back. The pages
    still settle nothing, which is why the entry stays here rather than moving to a cited claim.
  - how stable a `hash_sha256` fingerprint over a serialised column is between emissions
    (MSD-003). **Still open**, and the run could not touch it: measuring it needs a baseline over
    time, and `AgentsInfo` did not resolve in that tenant.
  - whether
    `has` matches a right-hand side carrying more than one term, where every example of `has` on the
    string-operators page is single-term and the literal these two files ship is three terms (MSD-001,
    and MSD-002 which follows it; the test is checklist Group 2). The scoping to `has` is deliberate:
    the same page's IPv4 operators do take arguments that are multi-term under its own definition of a
    term, but they are a separate operator group taking an argument rather than a right-hand side, and
    they establish nothing about `has`. **Answered in part: a multi-term right-hand side matched**,
    on both surfaces, with all three of Group 2's positive controls true. **The run did not exclude
    the reading on which the terms match while separated**, which is the residual that test's own
    table names.

  **Every answer above came from one tenant, and each from the one or two dates named against it**,
  which is a weaker thing than a documented guarantee and a stronger thing than the reasoning it
  replaced.

  **One inference this pack does not draw, stated because it is available and wrong.** Kusto
  reference pages carry an "Applies to" line naming Microsoft Fabric, Azure Data Explorer, Azure
  Monitor and Microsoft Sentinel. That line is the docset's own publication scope rather than a
  per-function availability statement: `tostring()`, `array_length()`, `case()`, `column_ifexists()`
  and the string-operators page that documents `has` all carry it identically, and this pack relies
  on all five inside Defender XDR queries. **Its silence about a product therefore carries no information about whether a
  function is available there.** Reading it as availability would be an absence argument of exactly
  the kind section 3.2 above holds to a higher bar: an absence carries weight only where the source
  enumerates the class being inferred about, under a heading that says so. **The 2026-08-24 run
  bears this out for one function.** `hash_sha256()` ran in Defender XDR advanced hunting, and its
  own reference page's "Applies to" line names Microsoft Fabric, Azure Data Explorer, Azure
  Monitor and Microsoft Sentinel, and does not name Defender XDR at all - read 2026-08-25, and that
  read did carry the element rather than dropping it. **The products are listed rather than quoted**,
  because the page prints them as a tick-marked row rather than as a comma-separated sentence. **The
  2026-08-26 run bears it out for a second function**: `column_ifexists()` ran on that same surface,
  and its own reference page's line names those same four products and does not name Defender XDR.
  Two functions on two dates do not prove the
  general reading. It is the direction the reading predicted, and the opposite of what treating that
  line as an availability statement would have told an operator to expect.

- **One construct these files ship is exemplified on the reference pages of the two operators that
  use it**: `in~` and `!in~` over a `dynamic([...])` right-hand side. MSD-004 ships both; MSD-006 and
  MSD-008 ship `in~` alone. The comparison table shared across the `in` operators gives scalar
  examples only for the case-insensitive pair, `"Abc" in~ ("123", "345", "abc")` and
  `"bCa" !in~ ("123", "345", "ABC")`, so it settles the dynamic-array form for neither of them.
  **`in~`'s dynamic-array form sits in the `in~` page's own Examples section**, under a heading
  reading "Dynamic array", whose example passes a `dynamic([...])` literal directly to `in~` and
  publishes the count it returns, with one of the literal's three elements lower-cased where the
  other two are not. Read 2026-08-18. **`!in~`'s sits on `!in~`'s own reference page**, under a
  heading of the same name, whose example publishes its output and which also carries a variant
  binding the array to a `let` first, the shape MSD-004 ships. Read 2026-08-18. **The failure
  direction that would otherwise apply here is excluded by example rather than merely unmentioned,
  and the two operators are not excluded to the same depth.** For `in~`, always false with no error
  is ruled out, because the published example returns a non-zero count. For `!in~`, the example
  likewise runs and publishes an output, which rules out a silent failure; ruling out always true
  would need the table's total row count, and neither page publishes that. So what the `!in~` example
  excludes is the error case rather than the over-match. This construct is not on the unverified list
  above.

  **One thing the `in~` example does not cover.** It passes its
  literal directly, while all three files bind the array with a `let` first, and the variant the
  `in~` page puts beside it uses a different operator. Of the case-insensitive pair, the `let`-bound
  **dynamic-array** form is exemplified for `!in~` and not for `in~`. **The `in~` page does carry a
  `let`-bound variant elsewhere**, in its Tabular expression section, but that one binds a different
  kind of right-hand side and settles nothing about the dynamic-array form. **Two restrictions the
  `in~` page carries survive that.** It states that "Case-insensitive operators are currently supported only for
  ASCII-text. For non-ASCII comparison, use the `tolower()` function", which reaches MSD-006 and
  MSD-008 among the three named above, where the values are ones you paste by hand rather than the
  empty forms this pack enumerates, **and reaches MSD-002 as well**, whose `DeliveryLocation` filter
  uses `in~` over hand-transcribed value names. And its performance guidance is "When possible, use
  the case-sensitive `in`", which
  the string-operators page gives as "Use `in`, not `in~`" - **a cost this pack trades away
  deliberately**, because a casing mismatch returns nothing and says nothing, while the cost of the
  case-insensitive operator is speed.
- **No detection here has a measured false-positive rate.** The false-positive guidance in each file
  is reasoning about the documented semantics of the columns, not measurement. Several passages do
  predict what a reader's environment will contain, and some of those carry a frequency word -
  "usually", "commonly", "often", "probably". **Read every one of them as a hedged design
  expectation reasoned from what Microsoft documents about the column, and none of them as an
  observed rate**. **None of the three runs changes that.** No run measured a frequency or recorded
  a count beyond whether a query returned anything at all, so nothing in any
  of them turns any of these words into a measurement. Where a frequency word appears
  inside quoted Microsoft text it is Microsoft's, not this pack's.
- **No positive control has been observed firing, for any detection.** Strictly, none of the eight
  can separate "your environment is clean" from "your filter is wrong" without one that fires,
  **so the absence of one is not what picks out a subset.** The five where the gap is widest -
  **MSD-003, MSD-004, MSD-005, MSD-007 and MSD-008** - are named as a judgement about how much else
  a reader has to fall
  back on when the result is empty, and not derived from a rule. **This paragraph is the canonical
  statement of that judgement.** The README, the checklist, `disclaimer.md` and the CHANGELOG each
  restate it; if any of them ever disagrees with this one, this one governs, and the difference is a
  defect to report rather than a nuance to interpret. Their handling differs and the
  difference matters. **MSD-007 and MSD-008 ship a discovery query first** and ask for a
  lab-generated event. **MSD-005 ships a complete, populated query** built from Microsoft's own
  published alert list, so its discovery work sits in verification step 2 rather than in the shipped
  query, and an empty result there is more likely to mean the connector or the plan than a wrong
  filter. **MSD-003 and MSD-004 are inventory queries whose emptiness is ambiguous** rather than
  discovery-first, which is the weakest of the three positions and is stated here rather than
  smoothed over. MSD-001, MSD-002 and MSD-006 sit on GA surfaces where an empty result is more
  readily interpretable, **and the 2026-08-24 run showed the limit of that**: MSD-002's filter as
  shipped matched nothing in one tenant, and an empty result of that kind reads as a clean estate
  on a GA surface exactly as it would on any other. None of the eight has been observed firing.

  **Three detections ship a discovery query first: MSD-006, MSD-007 and MSD-008**, and only MSD-006
  and MSD-008 additionally leave a placeholder value set for the operator to complete. That trio is
  about query *shape*, not about which detections need a positive control, and the two lists do not
  have the same members.
- **`BehaviorEntities` is cited but not relied on, and the distinction matters.** Its reference page
  was read for this pack on 2026-08-19, and MSD-008 quotes that page for the table's description and
  for its table-level preview and GCC status. **What was not done is a workspace pass:** no query
  here joins to that table. **Its published column set has since been checked against a workspace
  once, in one environment, and three of its columns were read**; what those columns carry for an
  agent-initiated behaviour has not been checked and needs a generated behaviour.
  **The reason MSD-008 cannot attribute a behaviour to an agent is separate from that**, and it is
  not a gap in this pack's reading: neither `BehaviorInfo` nor the `BehaviorEntities` reference
  **documents a column as carrying an agent identifier.** That is a statement about what the two
  pages document and not about what a join returns at runtime, which nothing here has observed.
- **`CloudAppEvents` was read but is not used as a detection surface in v0.1, and that is this
  pack's largest structural gap.** Microsoft documents it as carrying Agent 365 observability data
  for AI agent activity - **the only documented surface in this evidence set that records what an
  agent did.** `BehaviorInfo`, which MSD-008 reads, also carries runtime records, but they are
  protection audit and block events rather than the agent's own actions, so it answers "what did a
  control do" and not "what did the agent do". `CloudAppEvents` `ActionType` values for agent
  activity are not documented, so a narrowing filter would be a guess.

  Two detections here tell you what agents are **configured** to do - MSD-003 and MSD-004, both on
  `AgentsInfo` - and none tells you what any agent **did**. That is a gap in coverage rather than
  only in the backlog. **The full argument for and against the deferral, and the drafted query, are
  in [`scope-and-out-of-scope.md`](scope-and-out-of-scope.md)**, which is where the decision lives.

**This is why [`checklists/workspace-verification-checklist.md`](../checklists/workspace-verification-checklist.md)
is a release gate and not a suggestion.** The pack cannot be released until a human has run it.

---

## 7. Re-verification cadence

Four of the eight detections carry a preview or contested **label** - MSD-003, MSD-004 and MSD-008
are Public Preview, MSD-007 is Requires further validation. **MSD-005 belongs in the same refresh
lane without carrying either label**: its plan is GA (stated), and two of the seventeen alerts it
matches on are preview-tagged, so a change to those two moves what the detection sees without
moving its status. Calendar-based refresh is therefore not enough on its own.

| Trigger | Action |
|---|---|
| Monthly | Re-read every cited Learn URL; update last-verified dates; record every status change in [`CHANGELOG.md`](../CHANGELOG.md) |
| An alert MSD-005 matches on gains or loses a `(Preview)` tag | Update MSD-005's alert table and its preview count; the detection's own status label does not move on that alone |
| A cited page gains or loses a preview qualifier | Re-label the affected detection, do not adjust prose around it |
| Microsoft resolves the Sentinel connector labelling conflict | MSD-007 leaves **Requires further validation** - and only then |
| An advanced hunting schema change | Re-run the workspace verification checklist for the affected table, not only the citation check |
| A new ATLAS dataset release | Re-parse and re-match every cited technique ID; record the new version |
| A new OWASP edition | Re-base every OWASP cell and publish a renumbering table; never carry an old number forward |

A last-verified date older than the current monthly window is a staleness signal, and a detection
whose status rests on a contested source never leaves that label automatically.

**What the per-file `Last verified` header means, stated here because nothing else states it.** It is
the date that detection file's **primary table reference** was read - the Microsoft Learn page for
the table the detection queries, which is the citation the file's schema table rests on. It is **not**
a statement that every claim in the file was checked on that date. Each file's other reads carry
their own dates, in its Sources section and in the dated clause beside the claim they support, and
several of those dates are later than the header. Read the header as the anchor date for the schema
and the dated clause beside a claim as the date for that claim. All eight detection files carry the
header, and in all eight it agrees with that file's own primary-table Sources entry.
