# Changelog

Every status label change, every citation change, and every schema correction in this pack is
recorded here with the date it was made. This is the release history that
[`SECURITY.md`](SECURITY.md) and
[`docs/verification-methodology.md`](docs/verification-methodology.md) refer to.

Versions are `MAJOR.MINOR.PATCH`, and **a change to any status label or any cited page is at least a
patch** - a citation is a claim in this pack, so correcting one is a release-worthy change rather
than housekeeping.

**This file borrows from the Keep a Changelog convention rather than following it**, and both
departures are deliberate. An unreleased version is headed `[0.1.0] - unreleased` rather than
`[Unreleased]`, because here the version number is settled and only the date is not. And there are
sections that convention does not define, one for verification dates, one for the workspace
verification run, and one for known gaps, because in this pack a date is part of the claim. No page
is cited for the convention, because nothing here rests on it.

**Read a dated entry as a statement about what the sources said on that date**, not about what they
say now. A last-verified date older than the current monthly refresh window is a staleness signal
whatever the entry around it says.

## [0.1.0] - unreleased

The first version. It is written and it is not released. **On 2026-08-24 the queries in this pack
were run against a workspace for the first time**, in a lab Microsoft Sentinel workspace and in
Microsoft Defender XDR advanced hunting, and
[`checklists/workspace-verification-checklist.md`](checklists/workspace-verification-checklist.md)
was worked through against them. **That run did not complete every step.** No detection carries the
Verified outcome, because the control that would earn a detection that outcome is an event generated
deliberately in a lab tenant, and none was generated. **Several other pack files carried the
statement that no query here had ever been run or parsed**, which this entry falsified. **Those
corrections were applied on 2026-08-25** and are recorded under **Corrected after the workspace
verification run** below. What the run itself settled is recorded under **Workspace verification**.
The date on this entry moves when the release gate is met, and this run did not meet it.

### Added

- Eight detections, MSD-001 to MSD-008, across six documented tables and two deployment targets.
  Each carries its table and columns with documented data types, a Microsoft Learn citation, a
  last-verified date, a release-status label, framework mappings, false-positive guidance, an
  explicit statement of what it cannot see, and workspace verification steps.
- A framework cross-walk at item level, with MITRE ATLAS technique IDs read from MITRE's
  distributed `atlas-data` dataset (`version: 5.6.0`, release tag `v2026.07`) rather than recalled,
  and OWASP Top 10 for LLM Applications 2026 numbering.
- A verification methodology recording what was checked, on what date, and what the checks do not
  establish, including the scope of every negative claim the pack makes.
- A scope statement naming what was excluded and why, including the exclusions whose absence would
  otherwise be silent.
- The workspace verification checklist that gates the release.
- The inbound governance layer: `SECURITY.md` with a private reporting route and a data-handling
  section, three issue templates, an issue-template configuration that disables blank issues, and a
  pull request template carrying the no-invented-schema check.
- `LICENSE`, this file, and `.gitignore`.

### Verification dates in this version

- **2026-08-15** for the first source pass: every table reference, every status page, the MITRE
  ATLAS dataset parse, and the OWASP edition confirmation.
- **2026-08-16** for the Kusto function references, named rather than counted - `tostring()`,
  `array_length()` and `hash_sha256()` - and for the advanced hunting overview, the custom detection
  rules page, and the email-entity detection-technology page.
- **2026-08-17** for the Defender for Office 365 prompt injection guide, the Defender for Cloud
  AI-services alerts page, the Kusto string operators reference, the Kusto `union` and `print`
  operator references, a re-read of the advanced hunting overview and the advanced-hunting schema
  table list for the negative claim in
  [`docs/verification-methodology.md`](docs/verification-methodology.md) section 3.1, the licence
  files of the four Microsoft documentation repositories named in
  [`README.md`](README.md), a re-read of the custom-detection-rules page for the prefilter
  statement and the frequency-to-lookback mapping MSD-001 now records, and a re-read of the
  Conditional Access for workload identities article for the group-assignment limit MSD-006 now
  records. **Each of those two pages therefore carries more than one dated read**, as the advanced
  hunting overview does, and in the Sources entries citing those pages every date is kept rather than
  collapsed. No read count is stated here, because a later read moves it: the Sources entry in each
  file is what carries the reads.
- Two Kusto references, `isempty()` and the `in` operator, carry **2026-08-15** with the first
  source pass rather than either bucket above.
- **2026-08-18** for what this version cites at that date. Two senses of "adds" are separated below
  rather than run together.
  - **Pages new to the pack:** the `in~` operator reference, whose "Dynamic array" section settles
    that operator's dynamic-array form, and the `!in~` operator reference, whose section of the same
    name settles `!in~`'s and carries the `let`-bound variant MSD-004 ships.
  - **A new reason on a page already cited:** the advanced hunting schema table list, which MSD-003
    already cites at 2026-08-15 for the `AIAgentsInfo` listing, read again here for the
    `BehaviorEntities` preview tag and for where the GCC qualifier sits on that row.
  - **Two further pages supplied text this version had not carried before**, so they are additions
    rather than confirmations: the custom detection rules page, for the deduplication and alert-cap
    sentences the workspace verification checklist now quotes, and the AI agent detection and
    protection page, for the prerequisite clause MSD-003 now quotes in full.
  - **The remaining pages read that day confirmed quotations this version already carried:** the
    Kusto string operators reference and the comparison shared across the `in` operator variants,
    the Kusto `union` reference, the Defender for Cloud AI threat protection page, the
    `AADServicePrincipalSignInLogs` table reference, and the Conditional Access for workload
    identities article.
  - **A confirming re-read does not by itself move a citation's date**, so those Sources entries
    keep the dates they already carry. Where a file records a read inline beside the sentence it
    supports, that inline date is a record of the read rather than a change to the entry. MSD-006
    carries one for the workload-identity article, and MSD-003 carries one for the AI agent
    detection and protection page.
- **2026-08-19** for what this version cites at that date, in the senses the sub-entries below name.
  - **Pages new to the pack:** the `BehaviorEntities` table reference, which MSD-008 cites for the
    table's description and for its preview and GCC status, and the `CloudAppEvents` table
    reference, which [`docs/scope-and-out-of-scope.md`](docs/scope-and-out-of-scope.md) now cites
    for the six columns its v0.2 candidate query uses.
  - **A new reason on a page already cited:** the AI agent detection and protection page, read again
    for the `BehaviorEntities` row MSD-008 quotes beside its attribution limit.
  - **Pages re-read to settle a question rather than to add a citation:** the `BehaviorInfo` table
    reference, for the separate `StartTime` and `EndTime` descriptions MSD-008 now carries as two
    rows, and for the further identity column, `DeviceId`, that MSD-008 records the published table
    as carrying beyond the subset it reproduces; the `AADServicePrincipalSignInLogs` reference,
    which settled that `FederatedCredentialId`'s "Th identifier" opening is Microsoft's rather than
    this pack's; the custom detection rules page, for the "Required columns in the query results"
    heading MSD-001 now records beside the recommendation sentence it already quoted; the
    `isempty()` reference, for the published example rows MSD-003 and MSD-004 now lead with; and the
    Conditional Access for workload identities article, for the positive control MSD-006's GA
    evidence now names.
- **2026-08-20** for one page, read to settle a question rather than to add a citation: the
  `EmailEvents` table reference, re-read for the `SenderFromDomain` description in full and for the
  pairing of the four sender rows by header, each pair sharing its own trailing clause. MSD-001's
  Sources entry records the same read.
- **2026-08-22** for what this version cites at that date, in the senses the sub-entries below name.
  - **Pages new to the pack:** the `==` operator reference and the `!=` operator reference, each
    cited by MSD-007 for the case sensitivity of the operator the step beside it uses, and each
    carrying the comparison table that marks both operators case-sensitive.
  - **A new reason on a page already cited:** the case-sensitive `in` operator reference, which
    MSD-004 already cites at 2026-08-15, read again for its "Tabular expression" worked example and
    for the parameter note stating that "The search considers up to 1,000,000 distinct values".
  - **Pages re-read to settle a question rather than to add a citation:** the Kusto string operators
    reference, for the `has` and `has_all` rows, the worked example MSD-001 quotes, the definition
    of a term and the term-index sentences; the email-entity detection-technology reference, for the
    `Prompt injection protection` row MSD-001 quotes and the `Antimalware protection` row the
    checklist's Group 2 operands are built from; the licence files of the four Microsoft
    documentation repositories named in [`README.md`](README.md); and MITRE's distributed
    `atlas-data` dataset at the release tag the cross-walk records, for its dataset version field.
    **None of these moves a citation date**, per the rule stated under 2026-08-18.
- **2026-08-23** for what this version cites at that date, in the senses the sub-entries below name.
  - **Pages new to the pack:** the `getschema` operator reference, which
    [`checklists/workspace-verification-checklist.md`](checklists/workspace-verification-checklist.md)
    now cites in a source line beside the quotation it establishes, for the four column names the
    operator returns and for the example output whose `DataType` and `ColumnType` differ on every
    published row. That reading is what the Group 1 diff steps' `ColumnType` projection rests on.
  - **A new reason on a page already cited:** the AI agent detection and protection page, read again
    for the `CloudAppEvents` row of its advanced-hunting table list, which
    [`docs/scope-and-out-of-scope.md`](docs/scope-and-out-of-scope.md) cites inline and which
    MSD-008's Sources entry records against the same read.
  - **Pages re-read to settle a question rather than to add a citation:** the Microsoft Sentinel data
    connectors reference, for the `(Preview)` occurrences MSD-007 counts and the instrument it names
    beside them; the OWASP GenAI Security Project resource page, for the 2026 edition title and date
    [`docs/verification-methodology.md`](docs/verification-methodology.md) records; the
    `genai.owasp.org/llm-top-10` landing page, for the 2025 numbering the cross-walk and the
    methodology both record as a caveat; and MITRE's distributed `atlas-data` dataset at the release
    tag the cross-walk records, for the `dist/v6` emission the methodology's identifier check ran
    against. **None of these moves a citation date**, per the rule stated under 2026-08-18.
  - **A read already recorded here now carries a source where it is quoted:** the term-index
    sentences from the Kusto string operators reference, read 2026-08-22 and recorded under that date
    above. MSD-001's Sources entry now carries that read as its own dated clause, and the checklist
    carries a source line beside the quotation because that file has no Sources section. **Neither
    moves a citation date.** The 2026-08-22 entry above records why the read was made, which is not
    changed by what now cites it.
  - **Why this entry exists at all**, stated because the gap it closes is the kind this file is for:
    dated reads written inline beside the sentences they support are still reads, and until this
    entry the latest date here was older than the latest date in the copy.
- **2026-08-24** for what this version cites at that date, in the senses the sub-entries below name.
  - **Pages new to the pack:** the Microsoft Sentinel page on creating incidents from alerts, which
    MSD-005 now cites for what its duplicate-incident bullet describes. **That bullet previously
    presented a setting name in quotation marks, and no page read for this pack publishes that
    wording**, so the bullet now describes the behaviour instead and the checklist's Group 5 step
    does the same.
  - **A read that now carries a date on an entry which carried none:** the Defender for Cloud release
    notes, read for the AI-agent preview entry MSD-005's status evidence rests on. That Sources entry
    carried a URL and no read date, which is what this closes.
  - **A read recorded here because the entry above did not name it:** the Defender for Cloud
    release-notes archive, read 2026-08-23 to establish that the "General Availability for Defender
    for AI Services" heading and the date beneath it are separate page elements. **Its citation keeps
    2026-08-23** rather than today's date. It was re-read on 2026-08-24 and is unchanged, and **a
    confirming re-read does not move a citation date**, per the rule stated under 2026-08-18.
  - **Corrections made later on the same date, which revise the sub-entries above rather than
    replacing them.** The duplicate-incident bullet in MSD-005 and the checklist's Group 5 step were
    corrected twice on this date. The first correction removed a setting name in quotation marks that
    no page read for this pack publishes, which is the sub-entry above and is right as far as it
    goes. **What replaced it was wrong in the other direction**: it asserted that no page publishes a
    label for the control at all. The page MSD-005 cites publishes two, a **Create incidents -
    Recommended** section carrying an **Enable** control, which it names twice, and
    **Create > Microsoft incident creation rule** on the Analytics page. Both sites now name those
    labels, and the checklist's Group 5 step names where to look rather than declining to on the
    strength of the earlier claim. **The page prints the separator inside that section name as an en
    dash and this pack reproduces it as a hyphen**, which is a transcription difference rather than a
    different name.
  - **An attribution narrowed to what the cited sentence supports.** The checklist's Group 6 step ran
    a consequence about the sign-in table under the same "per Microsoft Learn" attribution as the
    Microsoft Entra ID bypass it follows from. **The cited sentence supports the bypass and the
    non-application of Conditional Access policy and says nothing about any log table**, so the
    consequence is now marked as this pack's reading. That is how MSD-006 and the cross-walk already
    state it. `disclaimer.md` carries the same narrowing.
  - **Two absence claims scoped to the instrument that established them.** MSD-001's and MSD-002's
    "no occurrence of the word preview" now reads over the **article body**, which is the scope the
    search covered and the only scope it speaks to. **No status label moved**: both files' own
    Sources entries already record that a page conversion is evidence of presence rather than of
    absence for a page element, and three other sites in this pack already carried the scoping
    sentence.
  - **A read that added a second side to material already cited:** the Defender XDR
    custom-detection-rules page, re-read 2026-08-24 for the sentence under its **Rule frequency**
    heading, "The rule frequency is based on the event timestamp and not the ingestion time."
    MSD-001 already quoted that page's `ingestion_time()` sentence and now records both, in the
    two-sided form it already uses one paragraph earlier. **No claim in that paragraph changed** and
    its hedge is unchanged. MSD-001's Sources entry carries the new read as its own dated clause.
  - **A read that confirmed a citation and did not move its date:** the Defender for Cloud release
    notes, re-read 2026-08-24 over the whole page. The "Threat protection for AI agents (Preview)"
    entry, dated February 2, 2026, is present and states that the capability is available in preview
    as part of the Defender for AI Services plan, which is what MSD-005's status evidence rests on.
    **The citation keeps 2026-08-24**, which it already carried.
  - **Corrections in this pack's copy now carry a dated marker at the site.** It is a short
    blockquote naming what was wrong, what is right, and this file as the record. **A marker is added
    when a later read falsifies a claim**, not when wording is tightened or a scope is narrowed, and
    the sub-entries above say which of the two each change was. **The sites carrying one are not
    counted here**, because a count of markers written outside the files holding them goes stale
    without anything in this file changing.
- **2026-08-24 and 2026-08-25, for the reads the correction pass rested on.** They are recorded
  apart from the workspace verification run below, because a run and a source read are different
  instruments and this file carries every dated read.
  - **Pages re-read to settle a question rather than to add a citation, both read 2026-08-24:** the
    `BehaviorInfo` table reference, re-read for its column table after one lab tenant's schema did
    not carry `Title`, which confirmed that Microsoft Learn publishes the column and left MSD-008's
    schema table unchanged; and the `EmailEvents` table reference, re-read for the
    `DeliveryLocation`, `DeliveryAction` and `EmailDirection` rows after that tenant emitted values
    the published lists do not carry, which confirmed all three lists unchanged and confirmed the
    page attaches no completeness qualifier to any of them. **Neither read moved a last-verified
    date**, because neither changed a claim.
  - **Two pages already cited by MSD-008, now cited by MSD-002 as well, both read 2026-08-24:** the
    Kusto reference for the case-sensitive `in` and the separate reference for the case-insensitive
    `in~`. MSD-002 cites them one per operator, for the comparison table giving `in` as
    case-sensitive and `in~` as its case-insensitive counterpart, for the performance note preferring
    `in` "when possible", and for the statement that case-insensitive operators are supported only
    for ASCII text. **These are the pages the MSD-002 operator change rests on.**
  - **A page re-read for a different reason than the one it was first cited for, 2026-08-25:** the
    `hash_sha256()` reference, read for its "Applies to" line, which names Microsoft Fabric, Azure
    Data Explorer, Azure Monitor and Microsoft Sentinel and does not name Defender XDR. Section 6 of
    the methodology records that the function nonetheless ran there, which is the direction that
    section's reading of those lines predicted. **The citation keeps its 2026-08-16 date**, which is
    the read the function claim itself rests on.
- **2026-08-25, for the reads a later correction pass on the same date rested on.** They are recorded
  apart from the entry above because they belong to a separate pass, and this file carries every
  dated read rather than the last one.
  - **A page new to the pack:** the Kusto `case()` reference, cited by MSD-002 for the syntax
    `case(predicate_1, then_1, [predicate_2, then_2, ...] else)`, for the `else` argument being
    required, and for the published example applying the function as a bucketing expression under
    `extend`. **This is the page MSD-002's rollup normalisation rests on.**
  - **A page read to settle an open question, and cited by a query once that question was settled:**
    the Kusto `column_ifexists()` reference, read for the syntax
    `column_ifexists(columnName, defaultValue)`, for both arguments being required, for the default
    column being returned where the named one does not exist, and for the deprecated alias it names.
    **Re-read 2026-08-25 before the query that uses it was written**, rather than carried from the
    earlier read. **MSD-008's step 1 cites it**, and the decision below records why step 2 and the
    self-check wrapper do not.
  - **A page re-read to confirm a quotation rather than to add a citation:** the `BehaviorInfo` table
    reference, re-read for the deployment sentence MSD-008 had paraphrased and now quotes in full.
    **The citation keeps the date it already carries**, because that read changed the form of a
    quotation and not a claim.

### Workspace verification

**2026-08-24, in a lab tenant.** The queries were run in a lab Microsoft Sentinel workspace and in
Microsoft Defender XDR advanced hunting, each on the surface its own file names as its deployment
target. **Nothing from that environment is recorded here beyond what this list carries**: an
outcome per detection, the run date, a schema correction stated as a column or a value name, a
description of how an emitted value differed from the spelling this pack publishes, a holdback, the
answers to the KQL behaviour checks, and the value names and serialisation shapes of columns for
which this pack reproduces no value set. No rows, no contents of a free-text column, no identifier of
any kind, and no count beyond whether a query returned anything at all. **This list is closed**, and
it is the same list wherever this file restates it.

**No detection is recorded Verified**, for the same reason in all eight cases. The control that
earns a detection that outcome is an event generated deliberately in a lab tenant, and none was
generated in this run. **The checklist's other controls are a separate thing and they did run**:
the one-element control on the literal check, and the three positive controls carried by the
multi-term operator test, all of which passed and are reported below.

| Detection | Surface | Outcome |
|---|---|---|
| MSD-001 | Defender XDR | Schema mismatch |
| MSD-002 | Defender XDR | Schema mismatch |
| MSD-003 | Defender XDR | Blocked |
| MSD-004 | Defender XDR | Blocked |
| MSD-005 | Microsoft Sentinel | Runs, unconfirmed |
| MSD-006 | Microsoft Sentinel | Runs, unconfirmed |
| MSD-007 | Microsoft Sentinel | Runs, unconfirmed |
| MSD-008 | Defender XDR | Schema mismatch |

> **Forward pointer, added 2026-09-12.** The table above is a dated statement about the 2026-08-24
> lab run and it stands as recorded. **Three of these outcomes were later superseded**, and a reader
> acting on this table alone would act on a stale label. **MSD-005** moved from Runs, unconfirmed to
> Blocked on what the 2026-09-11 production environment returned. **MSD-001 and MSD-002** moved from
> Schema mismatch to Runs, unconfirmed on the owner's reading of the pack's own value tables, which
> is a re-classification rather than a claim about any environment. The 2026-09-11 entry further down
> this file governs.

**Held back from the release**, on the rule that a detection whose table is not present, or whose
query returns a schema error, is held back rather than shipped with a caveat:

- **MSD-003 and MSD-004.** `AgentsInfo` did not resolve. The retired `AIAgentsInfo` name did not
  resolve either. A table name known not to exist returned an identically shaped error, and
  `BehaviorInfo` returned a schema through the same call, so the absence is the tenant's rather than
  the query's.
- **MSD-008.** Each of the queries it ships for deployment failed to resolve `Title`: in `summarize`
  for step 1, and in `project` for step 2 and for the self-check wrapper. **The two literal probe
  blocks that file also ships ran without error**, which is how the behaviours below were settled on
  that surface.

**MSD-001 and MSD-002 are not held back by that rule**, because their table is present and none of
the queries either file ships returned an error. They carry corrections instead. Whether MSD-002
should ship while the filter named below matches nothing is recorded below rather than in this
entry.

#### Schema corrections this run owes

- **`Title`, on `BehaviorInfo`, for MSD-008.** The file's column table records it; the observed
  schema does not carry it.
- **`DeliveryLocation` values, for MSD-001 and MSD-002.** Both files reproduce Microsoft's published
  value list for this column. Three entries in it came back differently: `Inbox/Folder` as
  `Inbox/folder`, `On-premises/External` as `On-premises/external`, and `Junk` as `Junk folder`. A
  value the reproduced list does not carry, `Forwarded`, was emitted as well, and the remaining
  entries came back unchanged. **MSD-002 filters this column with a case-sensitive `in` over those
  same three names as its file spells them**, so as shipped that filter matched nothing.
- **`EmailDirection`, for MSD-001 and MSD-002.** `Unknown` was emitted, beyond the `Inbound`,
  `Outbound` and `Intra-org` both files reproduce.
- **`DeliveryAction` needs no correction.** Every observed value sat inside the reproduced set.

#### The four Group 1b behaviours, all settled

- **A bare `isempty()` does not see an empty JSON array.** It returned false, and `tostring()` over
  the same literal returned the two-character string form. This is the behaviour MSD-003 and MSD-004
  assume when they test the string form instead.
- **The comment-only `dynamic([...])` literal parses**, on both surfaces, with the one-element
  control passing first on each. `array_length()` over it returned zero rather than null, so
  MSD-008's unconfigured branch can fire, and `in` and `in~` over it both returned false.
  **MSD-006 step 2 left unfilled therefore gives an empty match rather than a parse error**, which
  running that step confirmed directly.
- **A `print` works as an inline leg of a `union`**, on both surfaces, returning the single expected
  row. The construct MSD-008's self-check wrapper needs is available, and the missing column is what
  stops the wrapper.
- **`hash_sha256()` runs in Defender XDR advanced hunting**, returning without error a 64-character
  hexadecimal digest, which is the shape the function's own reference page describes. **The cited
  page publishes worked digests for other inputs and none for the string this check hashes**, so what
  the check settles is that the function resolved and returned a digest of the documented shape,
  rather than that it matched a published value. MSD-003's change-detection variant remains
  undeployable here, for the separate reason that its table is absent.

#### The multi-term `has` question

The checklist's six-column test ran unchanged on both surfaces and gave the same answer on each: a
multi-term right-hand side matched, a left-hand side sharing one term did not, and a left-hand side
carrying all three terms out of order did not. All three positive controls were true, so each
reading is live. That is the row of the checklist's table where the operator matched the terms in
order, and its prescription is that nothing changes for the published value set. **The residual that
row names still stands**: this run did not exclude the reading on which the terms match while
separated.

#### Undocumented elements this run resolved

Discoveries rather than mismatches, because the pack reproduces no value set for any of them.

- `EmailEvents.DetectionMethods` is serialised as a JSON object, with an empty form emitted as an
  empty string. It is neither a bare string nor a comma-delimited list.
- `AADServicePrincipalSignInLogs.ConditionalAccessStatus` emitted `notApplied`.
- `AADServicePrincipalSignInLogs.ResultType` stored numeric codes rather than the semantic strings
  Learn describes.
- **The two serialisations MSD-006 marks Provisional are settled.**
  `ConditionalAccessPolicies` is a JSON array, and `LocationDetails` is a JSON object.
- `AADServicePrincipalSignInLogs.Agent` is a JSON object.
- `CopilotActivity.RecordType` emitted `CopilotInteraction`, `Microsoft365CopilotScheduledPrompt`
  and `TeamCopilotInteraction`, and `CopilotActivity.Workload` emitted `Copilot`. **No
  settings-change record type was among them**, which is why MSD-007's step 3 returned nothing.

#### Steps recorded as not completed

- **Every step that asks for an event to be generated in a lab tenant**, which is every control that
  could earn a detection the Verified outcome. The checklist's literal controls are not in this
  class and did run; they are reported above.
- **The `BehaviorEntities` join** against a generated behaviour, which MSD-008 names as the one check
  that would close its largest limit.
- **The steps needing a management-plane token** this run did not hold: the Defender and Sentinel
  plans the workspace carries, the `CopilotActivity` table plan, whether incidents are already being
  created from Defender for Cloud alerts, and the Workload Identities Premium check.
- **The `AgentsInfo` steps of the agent-inventory group**, unreachable while that table is absent.
- **Which `SecurityAlert` column carries the AI identifier.** The discovery query ran and returned
  nothing, so the two columns could not be told apart. A zero from it is uninterpretable rather than
  clean, which is what MSD-005 and the checklist both say in advance.

#### Corrections this run creates and does not make

**Until this run, every file in this pack could say that nothing here had been executed against a
workspace or parsed by an engine, and several of them do say it.** That wording is now false, and it
is load-bearing where it appears: it is the ground for calling a construct unestablished, and this
run established the ones Group 1b names. It also reaches the residual notes in the detection files,
which name questions this run answered. **Those corrections are owed and this entry does not make them**, since
recording the run and revising the pack's copy are separate passes. A search for the never-run and
never-parsed wording finds the sites; no list of them is given here, because a list written outside
the files holding it goes stale without anything in this file changing.

#### 2026-08-26, the same checklist re-run after the corrections

**A second run, in the same lab tenant, on both surfaces.** It exists because the corrections made on
2026-08-25 changed queries the first run could not execute, and a correction that has not been run is
a reasoned fix rather than a demonstrated one. **The same closed list governs this entry as governs
the one above**, and nothing from that environment is recorded beyond it.

**What changed for MSD-008, which is why this run mattered.** The first run held it back because all
three of its deployment queries failed to resolve `Title`. **On this run all three parsed and ran on
Defender XDR advanced hunting**: the discovery rollup returned rows, the narrowed query returned none,
and the self-check wrapper returned its unconfigured row. **The schema error that triggered the
holdback is gone**, and the column is still absent on that surface, which is what the guard added on
2026-08-25 exists to absorb.

**What did not change for MSD-003 and MSD-004.** `AgentsInfo` did not resolve on this run either.
**The first run's holdback stands, reproduced on a second date.** A `getschema` against that table
alone failed the same way, which is the check that separates an absent table from a bad column
reference inside a query.

**What this run settles that the pack recorded as open.**

- **The self-check wrapper's two legs parse together** on that surface. The first run settled the
  construct and not the combination, because the wrapper never ran there.
- **The unconfigured branch fires.** Its guard compares a length against zero, so the branch emitting
  is the observable that the comment-only literal parsed to an array rather than to a null.
- **The guarded discovery rollup parses inside an aggregation**, a position its reference page does
  not exemplify.

**MSD-002's two queries both parsed on that surface and the rollup returned its bucket column.** No
rows matched, so the ranking the normalisation exists to fix is still unexercised.

**No detection is recorded Verified by this run either**, and for the same reason as the first: no
event was generated deliberately in a lab tenant, so no positive control exists for any of the eight.

**Release-gate item 3 is not cleared by this run.** It asks that a human has run the checklist end to
end, and a session executing queries is not that.

#### 2026-09-11, the third run, and the first under the owner's own interactive authentication

**Run date: 2026-09-11. Workspace type: production.** A third run, on both surfaces, executed in a
session the owner authenticated interactively, under the owner's own identity and permissions, with
the owner present and accepting the result. The owner remains accountable for the run and for the
outcome recorded against each detection. **Every group was attempted**, and every step not completed
is recorded below with its reason rather than omitted. The same closed list of what may be recorded
governs this entry as governs the two above, and nothing from the environment is recorded beyond it.

**For this run, the lab tenant role the checklist assumes is filled by the owner's production
tenant.** That is why the workspace type is recorded as production, and it is why **Group 0's first
item is recorded NOT COMPLETED**: production must not be used to generate test attack traffic. It is
not recorded as passed with a note. The published Group 0 guidance is correct for a reader who has a
lab tenant and was not changed.

**This entry claims nothing about which tenant the earlier two runs used, and the heading above is
scoped accordingly.** Those entries describe a lab tenant, and the amended gate item describes them
as not run in the owner's own environment; this run did not re-establish either statement and does
not rest on one. **What is first here is the authentication, not the environment**: this is the first
run executed in a session the owner authenticated interactively under their own identity, which is
the condition the amended item states. Whether a separate lab tenant exists is left where the earlier
records leave it.

| Detection | Surface | Outcome this run |
|---|---|---|
| MSD-001 | Defender XDR | Runs, unconfirmed |
| MSD-002 | Defender XDR | Runs, unconfirmed |
| MSD-003 | Defender XDR | Blocked |
| MSD-004 | Defender XDR | Blocked |
| MSD-005 | Microsoft Sentinel | Blocked |
| MSD-006 | Microsoft Sentinel | Runs, unconfirmed |
| MSD-007 | Microsoft Sentinel | Runs, unconfirmed |
| MSD-008 | Defender XDR | Schema mismatch |

**No detection is recorded Verified by this run.** One production event was generated, under the
separate approval recorded below, and its observation window had not closed when this entry was
written.

**Three outcomes moved.** One of the three, the alert detection, moves on what this environment
returned. **The other two move on a reading of the pack's own value tables rather than on anything
this environment showed**, so they are a re-classification and not an environment claim, and the
earlier tables are left standing because they are dated statements about their own dates rather than
because a re-classification could not reach them.

- **MSD-005 moves from Runs, unconfirmed to Blocked**, on two independent grounds, each paired with
  a control that fires. No Defender for Cloud product name appears among the alert products reaching
  this workspace, while alerts from several other products do reach it, so the table and the query
  are both working. Separately, the Defender for AI Services plan reads as not enabled on the
  subscription holding the workspace, while two other plans on the same subscription read as enabled,
  so that reading is genuine rather than a default returned on error. The checklist routes a plan that
  is not enabled to Blocked, so the outcome rests on the plan reading alone.
  **Which of the checklist's three explanations applies is NOT settled here, and an earlier draft of
  this entry wrongly said it was.** That draft concluded the connector is not configured. **No
  connector state was read at any point in this run**, and the plan reading is by itself a sufficient
  explanation of an empty alert table, so nothing observed discriminates a missing connector from a
  plan that is off. The eliminator that draft offered was also scoped wrongly: the behaviour table
  shows Defender for Cloud active **tenant wide**, while the absence being explained is scoped to one
  subscription and one workspace, and the same bullet insists on exactly that scope distinction.
  **Recorded as cannot tell**, which the checklist names as itself the finding. Tenant wide activity
  alongside a subscription scoped absence is still not a contradiction, and a later reader could
  easily misread it as one.
  One further scope note, unrecorded until now: the checklist asks for the plan on the subscriptions
  hosting the AI resources, and what was read is the subscription holding the workspace. Whether those
  coincide was not established.
- **MSD-001 and MSD-002 move from Schema mismatch to Runs, unconfirmed**, on the owner's reading,
  recorded as the owner's and not the agent's. The reading: the documented value tables reproduce what
  Microsoft publishes and do so accurately, so a tenant emitting additional values is a discovery
  rather than a stale or wrong citation, which is what the mismatch route exists to catch. Both files
  already say in terms that the published list is not the whole set a tenant can produce.
  **The two were not put to the owner in the same way, and the difference is recorded rather than
  smoothed over.** MSD-002 was put to the owner explicitly and answered. **MSD-001 follows by
  parity**, because the two share a table and rest on the same value evidence, and the parity is
  flagged here so the owner may split them rather than being left to look like a second explicit
  decision.

**What this run settled that the earlier two did not.**

- **The two serialisation questions, and one deployment question, that the earlier runs left open are
  not among these.** Most of what this run observed reproduces the 2026-08-26 run, including all four
  Group 1b behaviours on both surfaces, the multi term operator result and its unexcluded residual,
  and the undocumented elements that run resolved. **Reproduction on a third date is evidence and is
  recorded as reproduction, not as new.**
- **Whether the shipped filter fires against the observed serialisation**, which knowing the
  serialisation does not answer. Tested with synthetic literals in the observed shape, so no estate
  data was involved: the shipped operator does match inside that shape, the documented split form also
  matches, the alternative that matches on any one element over matches a different published value,
  and the phrase operator matches. **No operator change is required for either email detection**, a
  conclusion the earlier runs could not reach.
- **The Workload Identities Premium check**, which the earlier runs recorded as not completed. The
  tenant does not hold it, with a control confirming the negative is a real reading rather than an
  empty one. The consequence the checklist names applies: existing policies keep working and cannot be
  created or modified.
- **The duplicate incident question**, also recorded as not completed before. No Defender for Cloud
  alert reaches this workspace, so nothing is present for a second rule to duplicate. **That rests on
  the observed absence of those alerts and on the plan reading, not on any reading of connector
  state**, which was never taken.
- **The Copilot table's plan**, which is the tier on which a scheduled analytics rule can run.
- **`CopilotActivity.LLMEventData` is a JSON object**, and its field names are recorded in the
  owner's release record. That column was absent from the earlier runs' resolved elements entirely.
  Its contents are free text and did not travel.
- **`AADServicePrincipalSignInLogs` carries a second conditional access policy column** that no file
  in this pack mentions. It was null in every sampled row.
- **`BehaviorEntities` resolves on this surface**, and its column set includes an entity type column,
  an entity role column and a third, more detailed role column. MSD-008 states that nothing it
  publishes about that table has been checked against a workspace, so this is the first such check.
  What those columns carry for an agent initiated behaviour is still unsettled, because that needs a
  generated behaviour of a kind this tenant cannot produce.
- **`BehaviorInfo.Categories` is a JSON array string** rather than a bare string.
- **MSD-006's step 2 placeholder can now be filled from observed data**, from the single value the
  tenant emits for that column. It was not filled, because filling it is a copy change rather than a
  record of a run.
  **WITHDRAWN 2026-09-12, and listed here as a discovery in error.** The value this run observed is
  the one MSD-006 had already recorded under its 2026-08-25 settlement, which states that the
  placeholder stays empty and why. **This run reproduced that value rather than discovering a new
  one**, so it is corroboration and not grounds to fill. See the 2026-09-12 entry.
- **The alert identifier set, its severities and both preview tags reconcile exactly** against the
  live published page on this date, in the same order, with no drift and no date change on that page.
  Two of the seventeen names differ from the published heading only by a leading preview marker that
  the page carries inline and this pack relocates into its own column, which the file already
  describes at two separate sites. **That is recorded as a question rather than a correction**, because
  whether the platform emits the marker inside the alert name cannot be settled in a tenant that
  receives none of these alerts, and adding it to a case sensitive filter unverified would risk
  breaking a filter that currently matches.
  **The page read was [Alerts for AI workloads (Microsoft Defender for Cloud, Microsoft
  Learn)](https://learn.microsoft.com/en-us/azure/defender-for-cloud/alerts-ai-workloads), read
  2026-09-11**, which is the page MSD-005 already cites for this set. The whole page was read rather
  than a window, and the reconciliation was paired with a control that matched known text and a probe
  that correctly matched nothing. **The read confirmed the citation and does not move MSD-005's
  last-verified date**, on this file's own precedent that a confirming read does not.
- **The Sentinel connector conflict still stands**, re-read whole on this date. Both the page level
  notice and per entry tagging are present simultaneously, the Copilot entry carries no per entry tag,
  its sole stated prerequisite matches this pack's quotation character for character, and no licensing
  or requirement of that kind is stated for the connector, tested as an absence with a control that
  fires. **MSD-007's status label therefore does not change**, which is the only condition that file
  gives for changing it.
  **The page read was [Find your Microsoft Sentinel data connector (Microsoft
  Learn)](https://learn.microsoft.com/en-us/azure/sentinel/data-connectors-reference), read
  2026-09-11**, which is the page MSD-007 already cites for this conflict. **The read confirmed the
  citation and does not move MSD-007's last-verified date**, on the same precedent. One caution
  belongs with it: the quoted notice contains inline emphasis, so a literal match against page source
  returns a false zero, and every figure above was taken on extracted text rather than on source.

**One production event was generated, under a separate approval given at the time of action.**

The owner approved, in the session and as a distinct decision rather than by way of the run's
instructions, the narrowest available event for the Group 7 positive control: a single tenant level
Copilot administrative setting switched off and immediately back on, restoring the documented default.
**The owner performed the change**, because it is a configuration action; the agent only observed.
Blast radius is administrator facing, with no documented effect on any end user. A baseline taken
before the change established that the record type the relevant query filters on had never appeared in
the retention window, while a control leg over the same table returned rows, so a later appearance
cannot be confused with pre existing data and the pre change absence is not an artefact of a dead
query. **The observation window had not closed when this entry was written.** The availability figure
usually quoted for audit data was NOT re-read in this run and is carried from the run brief rather
than from a page this entry names, and on that second hand reading it is scoped to workloads that do
not include this one. **The roughly three day interval is therefore this run's own estimate, not a
published latency**, and it is recorded as an estimate so a later reader does not treat it as
Microsoft's figure. **MSD-007 therefore remains Runs, unconfirmed.**

**A finding from that work which holds whatever the event produces, and which bears on deployment
rather than on this run.** The audit operation documented for a Copilot settings change and the record
type this pack's step 3 filters on are named on different pages, and **neither page names the other**,
so the mapping between them is an inference from two descriptions agreeing rather than anything
published. Separately, the workspace table's primary timestamp is the audit event time, so a scheduled
rule whose frequency and lookback are both short filters such an event back out even when the row is
present and correct. That is the same class as the lookback against frequency item the checklist's
Group 8 already raises, at a site the pack does not currently name.

**Steps recorded as not completed, with reasons.**

- **Group 0's first item**, because the environment is production.
- **Every step of Group 3**, and both agent inventory detections, because the table does not resolve.
  The retired predecessor name does not resolve either, and five other tables resolved through the
  same drivers in the same sessions, so the absence is the tenant's and not the instrument's.
- **The event generating steps other than the one approved above**, and the entity join that depends
  on a generated behaviour. For the alert surface and the behaviour surface these are not declined but
  impossible here: the plan that would produce the alerts is not enabled, and nothing of the relevant
  kind populates the behaviour table. **Approval was therefore not sought for events that provably
  could not occur.**
- **The lab subscription positive control**, because the plan it requires reads as not enabled on that
  subscription as well. Enabling it is an owner configuration action.
- **The unified audit log preflight**, because the module it needs is absent from the machine the run
  was executed on. This is a limit of the apparatus and not a statement about the estate.
- **The application to agent mapping, the authentication method inventory, and the phishing simulation
  sender domains**, which are owner actions whose artefacts stay in the owner's environment by this
  pack's own rule, and which were deliberately not attempted.
- **The lookback against frequency comparisons in Group 8**, because they require creating a scheduled
  rule, and no rule was created.

**Release-gate item 3 is not cleared by this entry.** This entry records a run and its date, which is
what the item asks be recorded. **Whether the item is satisfied is the owner's determination**, and the
owner had not recorded that determination when this entry was written. The amendment of 2026-09-11 is
prospective and clears no earlier run.

#### 2026-09-12, the owner decisions the third run surfaced, and one claim withdrawn

**No workspace run, and no new evidence about any environment.** No query was executed, no
configuration was read or changed. This entry records decisions taken in words and the edits applying
them. **The three corrections the 2026-09-11 run owed to public copy are applied**, each at one site,
under the owner's authorisation.

- **The workspace verification checklist's opening no longer says it was worked through once.** Three
  runs have happened: 2026-08-24 and 2026-08-26 in one lab tenant, and 2026-09-11 in a production
  tenant. The opening now scopes the MSD-008 `Title` failure to the first run, records the 2026-09-11
  event and its unclosed observation window in place of the claim that no event was generated, and
  states that this file governs where the two disagree.
- **Its Group 4 sentence no longer calls the step the first run of the `column_ifexists()` guard.**
  The 2026-08-26 run parsed the guarded rollup inside the aggregation and the 2026-09-11 run
  reproduced it. **The settlement is not reopened** and the surrounding instruction is unchanged.
- **The 2026-08-24 outcome table now carries a forward pointer** naming the three outcomes later
  superseded. **The table itself is not rewritten**, because it is a dated statement about its own
  date, and overwriting it would destroy evidence while leaving it unmarked would let a reader act on
  a stale label.

**One claim from the 2026-09-11 entry is withdrawn.** That entry recorded MSD-006's step 2
placeholder as now fillable from an observed value. **It was not a new value.** MSD-006 already
carried a dated settlement, 2026-08-25, recording that the 2026-08-24 lab run emitted `notApplied`
and that filling the literal with it would generalise a single observation into shipped copy.
**The 2026-09-11 run observed the same value.** What it actually established is corroboration, a
second and different estate agreeing with the first, which rules out the reading that the earlier
result was an artefact of one lab tenant. **The placeholder stays empty and the shipped query is
unchanged**, and the corroboration is recorded beside the settlement in MSD-006.

**MSD-001 keeps parity with MSD-002** at Runs, unconfirmed. The 2026-09-11 entry flagged the parity
rather than assuming it so the owner could split the two; **the owner declined to split them**.

**A scope gap in Group 5 is recorded as open.** That group asks for the Defender for AI Services plan
state on the subscriptions hosting the AI resources, and what was read was the plan state on the
subscription holding the workspace. **Whether those coincide is unestablished.** MSD-005 stays
Blocked: the plan ground is narrowed to the subscription that was read, but the independent ground is
untouched, since no Defender for Cloud product name appears among the alert products reaching this
workspace while several other products' alerts do. **The Group 5 step is recorded as not completed**
rather than passed with a note.

**The Track C observation is still outstanding and no absence is claimed.** The event was performed
by the owner on 2026-09-11, absence is not claimable before roughly 2026-09-14 on this run's own
estimate rather than a published latency, and the observation query did not run because the
credential expired under a conditional access sign in frequency check. **Re-authentication is an
owner action.**

**Release-gate item 3 is not cleared by this entry**, and Gate 5 was neither sought nor granted.

#### 2026-09-19, the Track C observation completed, and a recorded negative for MSD-007

**The observation left outstanding on 2026-09-11 is closed on the workspace side.** One query, run
once, read only, against the same production workspace. **Eight days elapsed** against the roughly
three day threshold the earlier entry recorded as an estimate rather than a published latency, so the
wait window is closed and an absence claim is permissible.

**The deliberate Copilot administrative settings change of 2026-09-11 produced no record of the type
MSD-007 step 3 filters on, within eight days.** The query returned two record types, both already in
the 2026-09-11 baseline set and both belonging to the Copilot workload. Neither falls outside that
baseline and neither equals the documented settings type, on the two flags the query carries for
exactly this purpose.

**The control fires, which is what makes this an absence rather than an empty query.** The query
returns a pipeline freshness bucket so a zero can be told apart from a stalled pipeline, and one of
the two record types is arriving in the one to three hour bucket. **The connector, the table and the
workload are live at the moment of the read.** The baseline had separately proved the target record
type absent across the full retention window with a control leg returning rows over the same table in
the same statement, so the pre change zero was real and no pre existing row could be mistaken for the
generated event.

**Therefore MSD-007 step 3 is not detectable in this tenant through this table.** This is the outcome
the 2026-09-11 entry named as the one to expect, on the grounds that Microsoft's own shipped Copilot
analytics rules key on an interaction type and three plugin types, none of which uses the settings
record type. **It is a finding about what this tenant's pipeline delivers, not a defect in the query**,
which parses, runs and matched its schema in full.

**What is still unknown, and it is a real limit.** Whether the change was written to the unified audit
log at all is not established. **A connector gap and an operation that was never audited produce the
identical workspace side zero**, and only an audit log search discriminates them. This machine cannot
perform one, because the Exchange Online module is absent, so that remains an owner action and an
apparatus limit rather than a finding about the estate. A stalled pipeline is ruled out by the
control. **Any such audit search must run under rights that include audit log read**, since the
directory role required to make the change does not confer them and an empty search under the wrong
rights is indistinguishable from the event never being recorded.

**MSD-007 stays Runs, unconfirmed.** It does not reach Verified, because the positive control was
performed and did not fire. **It is not moved to Blocked**, because the routes to that label are a
table that does not resolve or a plan that is not enabled, and neither applies here: the table
resolves, it is populated, the schema matched in full and all four queries run. **What the detection
now carries is a recorded negative**, a state the outcome vocabulary has no name for. Whether it
should gain one, and whether MSD-007's own copy should carry this finding, are owner decisions and
no detection file was edited.

**Release-gate item 3 is not cleared by this entry**, and Gate 5 was neither sought nor granted.

#### 2026-09-19, the Group 5 scope gap closed, and MSD-005's basis corrected under an unchanged outcome

**Read only, management plane only, run while the same interactive session was still authenticated.**

**The gap recorded on 2026-09-12 was real.** Group 5 asks for the Defender for AI Services plan state
on the subscriptions hosting the AI resources. The 2026-09-11 run read the plan state on the
subscription holding the Sentinel workspace, and whether those coincide was left unestablished.
**They do not coincide.** The subscription holding the workspace **hosts no AI resources at all**, and
the AI resources in this estate sit on other subscriptions the earlier read never reached.

**The correctly scoped read returns the same answer.** On every subscription that hosts AI resources,
the plan reads as not enabled.

**MSD-005 stays Blocked, and this changes its basis rather than its outcome.** Before this check the
outcome was right for a reason that did not license it, because the reading came from a subscription
the question was not about. **An outcome that is accidentally correct is not evidence**, and the
standard this pack holds itself to is that a conclusion carries the ground it actually rests on. **The
independent ground is untouched**: no Defender for Cloud product name appears among the alert products
reaching this workspace while alerts from several other products do, which is a statement about the
workspace and owes nothing to any subscription's plan state.

**Controls, named rather than asserted.** Plans in a non Free tier were observed elsewhere in the same
estate during the same pass, so a reading of not enabled is a real reading and not a default returned
on error. **Coverage was complete**: every subscription in scope was readable for both the plan check
and the AI resource check, so the conclusion is about the estate rather than about what happened to be
reachable. Subscriptions were classified as hosting AI resources by listing the resource type, **not
by reading anything into what a subscription is called**.

**The transferable lesson, and it is about scoping rather than about this pack's copy.** An operator
reproducing Group 5 should read the plan on the subscriptions where their AI resources actually live,
and should not assume the subscription carrying their Sentinel workspace is one of them. The checklist
already said subscriptions hosting the AI resources; the earlier run scoped its read wrongly.

**No inventory travelled.** Per subscription detail, identifiers, names and every count stayed outside
this record. **Release-gate item 3 is not cleared by this entry**, and Gate 5 was neither sought nor
granted.

#### 2026-09-19, the audit log discrimination settled, and a predicted outcome falsified

**Read only. Audit searches only, through Microsoft Graph, under a permission the owner granted for
this purpose.** No configuration read or changed, no query altered, no detection file edited.

**The question the 2026-09-19 workspace entry left open is answered, and not in the direction this
arc predicted.** That entry recorded that a connector gap and an operation that was never audited
produce the identical workspace side zero, and that only an audit log search separates them. **It is
not a connector gap.** The deliberate settings change produced no audit record at all, so there was
nothing for any connector to forward. **The expectation recorded on 2026-09-11, that a connector gap
was the outcome to plan for, is falsified.**

**The blocker was one route, not the question.** The earlier entry recorded the discrimination as
impossible because the Exchange Online module is absent from the machine. The unified audit log is
also reachable through Microsoft Graph, which this arc already used for advanced hunting.

**Three searches, each with its control.** A targeted search for the operation MSD-007 step 3 filters
on returned no records, complete rather than truncated. **A control search over the same window, the
same store and the same identity, for the Copilot interaction operation, returned records in volume**,
so the store is readable and populated and the targeted zero is neither a permissions artefact nor an
empty instrument. **An exhaustive enumeration of every operation name on the event day then closed
the remaining alternative**, that the change is audited under some other name: the window was
exhausted with no unread pages, exactly one operation name matches Copilot and it is the interaction
operation, **no operation name matches a setting at all**, and the matcher's own control fires on a
term known present in the same set.

**Scoped precisely.** This establishes that **for this specific toggle, in this tenant, on the event
day**, no audit record was produced under any Copilot named or setting named operation. **It does not
establish that Copilot settings changes are unaudited in general.** One toggle was exercised, switched
off and immediately back on, and whether a net zero change is recorded differently from a persisted
one is untested. **It also cannot exclude an operation named with neither word.**

**Why it matters to the pack.** MSD-007 step 3 filters on a record type Microsoft publishes only as an
example behind an "e.g.", which that file already says. **This run adds that a real settings change in
a real tenant produced no such record anywhere in the audit log**, not merely none in the workspace
table. **Step 3's emptiness is therefore upstream of Sentinel entirely**, and an operator seeing
nothing from step 3 should not conclude that their connector or their workspace is at fault.

**MSD-007's outcome is unchanged and stays Runs, unconfirmed.** Nothing here moves it to Verified or
to Blocked, and no detection file was edited.

**An apparatus defect of this run's own is recorded rather than quietly fixed.** The first targeted
search printed a verdict line stating the opposite of its own data, because a function returned its
progress output alongside its row count and the numeric test that decided the verdict silently became
a truthy array filter. **It was caught by reading the rows instead of the verdict**, which is the same
guard this pack applies to estate results, turned on the instrument. Every conclusion above rests on
the row level facts and the raw result files.

**Release-gate item 3 is not cleared by this entry**, and Gate 5 was neither sought nor granted.

#### 2026-09-19, release-gate item 3 determined satisfied by the owner

**This is the owner's determination, recorded here rather than reached here.** Every entry above
states that the agent does not clear this item and has not cleared it, and that remains true of all of
them. The determination was taken by the owner in words on 2026-09-19.

**The item asks that the workspace verification checklist be written and run end to end with the date
recorded, and that it be cleared by the run rather than by any repair to the file.** The checklist has
been run three times, on 2026-08-24 and 2026-08-26 in one lab tenant and on 2026-09-11 in a production
tenant, with the date recorded per run above. **The third run's outstanding observation was completed
on 2026-09-19**, and the question it left open was settled the same day, against this arc's own
prediction rather than in its favour. **Controls fire throughout**, including on the instruments.

**What the determination does not do.** It does not convene or clear Gate 5, which was neither sought
nor granted. **It releases, tags, publishes and announces nothing.** It marks no detection Verified,
and none is. **It does not convert any step recorded as not completed into a completed one.**

**A limit worth carrying forward.** MSD-007's positive control is structurally unreachable in this
tenant rather than merely unobserved, because the settings change that was exercised produced no audit
record at all. **A future run of that step here cannot reach Verified without a different toggle or a
different tenant**, which is a property of the environment and not a defect in the detection.

### Corrected after the workspace verification run

**2026-08-25.** These are the corrections the run above created and deliberately did not make.
Recording an observed outcome and revising the pack's copy are different kinds of work, so they were
done in separate passes. **No estate data crossed into this repository in either pass.** What
travelled is the list stated under Workspace verification above, and nothing beyond it. **The two
statements are one list rather than two**, and a difference between them is a defect to report rather
than a narrowing to interpret.

#### The three schema corrections, and what each turned out to be

- **`Title`, on `BehaviorInfo`, for MSD-008. A tenant availability difference rather than a
  transcription defect.** The reference page was re-read on 2026-08-24 and does publish the column,
  which MSD-008 had quoted correctly. **The pack's transcription was right and the lab tenant's
  schema did not carry it.** The schema table therefore keeps the column, and the three queries
  MSD-008 shipped for deployment no longer depend on it: step 1 rolls up `Description` in place of
  `Title`, and step 2 and the self-check wrapper project `Description` without it. The wrapper's
  inline column counts were re-derived from the query blocks rather than adjusted by hand.
- **`DeliveryLocation` values, for MSD-001 and MSD-002. The published list is current, and one lab
  tenant emitted values on 2026-08-24 that it does not carry.** The `EmailEvents` reference was re-read on 2026-08-24
  and publishes the same seven values this pack already reproduced, with the same capitalisation.
  **The reproduction was not stale, so it stands unchanged.** What changed is a note in both files
  recording that a lab tenant emitted three of those values spelled differently, two by case alone
  and one by a longer token, plus `Forwarded`, which the published list does not carry.
- **`EmailDirection`, for MSD-001 and MSD-002. A value one lab tenant emitted on 2026-08-24 that the
  documentation does not list, rather than a
  documentation mismatch.** The same re-read confirms the published three unchanged and confirms the
  page attaches no completeness qualifier to the list. `Unknown` is therefore a value the
  documentation neither lists nor excludes, and both files now say so in those terms.

#### MSD-002, the filter that matched nothing

**The corrected query had not been run anywhere when this entry was written**, because that pass ran
nothing. **It was run on 2026-08-26 and the outcome is recorded under the workspace verification
heading above.** What follows is a
defect that was observed and a correction that was reasoned from the source, not a demonstrated fix.

**The defect.** Both of MSD-002's queries filtered `DeliveryLocation` with the case-sensitive `in`
over three literals, and the lab tenant matched none of them, so the filter returned nothing and the
empty result read as a clean estate. **That is the silent failure this pack argues against, shipped
inside it.**

**The fix, and why this one.** Microsoft Learn publishes the spellings the pack already carried, so
correcting the literals to the platform's spellings was not the available answer. Both queries now
use `in~`, which survives either answer to the source question because it matches the published
spelling and the observed one alike. **Case-insensitivity cannot reach a different token**, so
`Junk folder` is listed beside `Junk`. Both operator reference pages were read on 2026-08-24 and are
cited per operator in the file, with Microsoft's own note preferring the case-sensitive form.

**MSD-001 was evaluated separately and needed no query change.** It projects these columns and groups
by one of them rather than filtering on either, so an unlisted value widens its output instead of
emptying it. It carries a note recording that difference.

**That fix put a value string into a shipped query that Microsoft Learn does not publish, so the
pack's own two-class rule now has a third class.** The rule said every table, column, value string
and status label in a detection file or a shipped query is either quoted from Learn on a stated date
or marked Provisional, with no third category. `Junk folder` is neither. **Rather than leave the rule
with an unstated exception, the third class is written into it**: a value labelled at the point of
use as observed in a lab tenant on a stated date rather than as documented, which **may widen a
filter and may never narrow one**. `docs/verification-methodology.md` section 1 is the canonical
statement and names the members rather than counting them; `README.md` and the pull-request template
restate it and were corrected to match.

#### Wording retired, and what replaced it

**Every shipped pack surface asserting that nothing here had been run or parsed now carries a
tenant-bounded and date-bounded statement instead**, in `README.md`, `disclaimer.md`,
`docs/verification-methodology.md` section 6, the workspace verification checklist, the
verification-report issue template, and residual notes in MSD-001, MSD-002, MSD-003, MSD-006 and
MSD-008. **MSD-002 was added to that list on 2026-08-25**: its verification step 1 still called the
multi-term `has` question unverified after the run had answered it in part, which put it at odds
with MSD-001, the file it defers to for that step. **Section 6 is the canonical statement and the
others defer to it**, so it carries the per-construct detail of what the run answered and what it
left.

**What was deliberately not retired.** The positive-control statement stands untouched wherever it
appears, because the run generated no event and no detection has been observed firing. So does the
fingerprint-stability entry, which needs a baseline over time. So does the residual on the multi-term
`has` reading: the run matched a multi-term right-hand side without excluding the reading on which
the terms match while separated.

#### Decisions settled 2026-08-25, and what each settled to

- **Whether MSD-002 ships: it ships, on a condition that is not yet met.** Its table is present and
  neither query returned an error, so the holdback rule stated under the workspace verification
  heading above, which reaches an absent table or a schema error, does not reach it. As shipped on
  the day of the run it matched nothing, and it now carries an operator that would have matched.
  **The condition is a workspace
  run: both of its queries parse there, and its rollup returns the bucket column it groups on.** If
  either fails, it is held back rather than shipped with a caveat. **Both were run on 2026-08-26, in
  one lab tenant, on Defender XDR advanced hunting: both queries parsed and the rollup returned its
  bucket column.** **No rows matched**, so the ranking the normalisation exists to fix is still
  unexercised, and whether the condition required matching rows is a reading the owner takes rather
  than this entry. **Nothing here marks this detection released.**
- **Whether MSD-006's step-2 placeholder is filled with `notApplied`: it stays empty.** The run
  emitted that `ConditionalAccessStatus` value, and one tenant on one date is not a documented value
  set. **Filling it would narrow rather than widen**, which is the half of the third-class rule that
  does the work: a lab-observed value may widen an inclusion list and may never define the predicate
  a rule fires on. The value stays recorded in the file and the literal stays empty.
- **Whether MSD-008's step 1 should guard `Title` rather than drop it: it guards, in step 1 only.**
  Learn publishes the column and one lab tenant did not carry it, which is the case a schema-tolerant
  reference exists for: `column_ifexists(columnName, defaultValue)` is documented, both arguments
  required, read 2026-08-25, and returns the default column where the named one does not exist. Step
  1 now rolls up `column_ifexists("Title", Description)`. **Step 2 and the self-check wrapper are
  unchanged and name `Description` alone**, because those are per-row output where one documented
  column is the right answer rather than a discovery rollup. **The reference page settles syntax and
  not availability**: its "Applies to" line names the same four products every Kusto reference page
  names, so this pack claims nothing either way about Defender XDR advanced hunting. **It also does
  not exemplify the shape this query ships**, applying the function under `project` rather than inside
  an aggregation, so whether step 1 parses is a question for a workspace run.

**Release-gate item 3 is not cleared by this pass**, and the run above did not clear it either. A
human clears it.

### Known gaps carried into this version

- No positive control has been demonstrated for any detection, so none of the eight can distinguish a
  clean environment from a broken filter without one. The gap is widest for five of them when the
  result comes back empty, which is a judgement rather than a category derived from a rule;
  [`docs/verification-methodology.md`](docs/verification-methodology.md) section 6 names the five,
  says on what ground, and governs this wording.
- The constructs that remain unsettled are named where they are used rather than
  counted, in [`docs/verification-methodology.md`](docs/verification-methodology.md) section 6,
  which now records for each one what the 2026-08-24 run answered and what it left open.
  Checklist Group 1b carries the workspace checks for the ones a single query can settle, and **it is
  not the same set**. One of its checks confirms documented `isempty()` behaviour rather than an
  unparsed construct. How stable a fingerprint is between emissions is not in it at all, because
  measuring that needs a baseline over time rather than one query, and it is the one section-6 entry
  that run could not touch. And the multi-term `has` question
  is checked in Group 2, where the operator-selection decision is actually made, rather than in
  Group 1b.
- `CloudAppEvents` for AI-agent activity is the largest structural gap and is argued in full in
  [`docs/scope-and-out-of-scope.md`](docs/scope-and-out-of-scope.md), with the candidate query
  drafted there.
