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
were run against a workspace for the first time**, in a Microsoft Sentinel workspace and in
Microsoft Defender XDR advanced hunting, and
[`checklists/workspace-verification-checklist.md`](checklists/workspace-verification-checklist.md)
was worked through against them. **Two further runs followed, on 2026-08-26 and on 2026-09-11.**
**No run completed every step.** **No detection carries the Verified
outcome, because no detection has been observed firing.** **Several other pack files carried the
statement that no query here had ever been run or parsed**, which those runs falsified, and **those
corrections were applied on 2026-08-25**. What the runs establish is stated under **Workspace
verification** below.
The date on this entry moves when the release gate is met, and no run has met it.

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

### Fixed

- **2026-09-21 - `README.md` described a failure mode that does not occur.** The section "Why that
  rule matters more than the queries" said a reader pastes a query naming a fictional column, gets
  zero rows, and cannot tell that from a clean environment. **A reference to a nonexistent column
  is not a zero-row result.** Microsoft classes it as a syntax error and the query never runs:
  "The query contained unrecognized names, including references to nonexistent operators, columns,
  functions, or tables", which is the cause the page gives for syntax errors, with
  `A recognition error occurred.` as that row's example message
  ([advanced hunting errors](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-errors),
  read 2026-09-21, re-read 2026-09-27). The 2026-09-27 entry below records what this sentence
  said before.

  **The corrected passage names the failure this pack is actually built against**: a real column
  filtered on a guessed value, or a placeholder left in place, which does return zero rows and does
  read as a clean environment. That is the case MSD-002's observed `DeliveryLocation` spelling and
  the MSD-006 and MSD-008 placeholders exist to guard, and the case the discovery queries exist to
  close.

  **The pack already stated the correct mechanism elsewhere and the README contradicted it.**
  [`SECURITY.md`](SECURITY.md) ranks a query built on a column that no longer exists as one that
  "fails loudly, which is survivable", against one built on a column that exists but means
  something else, which does not. The defect was an internal contradiction, not a gap.

  Recorded here because a citation is a claim in this pack, and a mechanism stated in the section
  that justifies the pack's central rule is a larger claim than a citation. **No status label, no
  schema element, and no query changed.**

- **2026-09-27 - The pack's own example of invented schema named a documented property.**
  `README.md` and section 1 of `docs/verification-methodology.md` used `XPIADetected` beside
  `AbnormalCopilotBehavior` as a name that reads like real schema, and the methodology said all
  three names a past review flagged "were not real schema". `XPIADetected` is not a
  `CloudAppEvents` column, but the Purview page on Copilot audit logs documents it as a property of
  `AccessedResources` in Copilot audit records
  ([Audit logs for Copilot and AI applications](https://learn.microsoft.com/en-us/purview/audit-copilot),
  `ms.date` 2026-08-26, read 2026-09-27). Both files drop it and keep `AbnormalCopilotBehavior`.
  The third name, `DataExfiltrationDetected`, was checked on the same date along with
  `AbnormalCopilotBehavior`: neither is a `CloudAppEvents` column, and a Learn site search returned
  no page for either. The methodology sentence now states those two findings and carries a dated
  marker. On this pack's reading, which no page read states and no run has tested, a property
  nested in a `dynamic` payload is also the silent case rather than the loud one the README paired
  it with. No status label, no schema element and no query changed.
- **2026-09-27 - MSD-007 overstated what Learn leaves undocumented about Copilot interaction
  records.** Its "cannot see" bullet said Learn documents neither the `CopilotInteraction` record's
  contents nor the shape of `LLMEventData`. The Purview page cited above documents what the records
  hold, and its table of "some of the common properties" includes a per-message `JailbreakDetected`
  flag and no property holding prompt or response text; the `CopilotActivity` table reference,
  re-read 2026-09-27, still publishes no schema for `LLMEventData`. The bullet now says so, names
  Microsoft's Azure-Sentinel sample row and the Azure Monitor example-queries page as where the
  shape shows, and carries a dated marker. The sample row carries `Messages[].JailbreakDetected` and
  an empty `AccessedResources` array, so it is cited for the first path only. The opening warning
  is narrowed from prompt-injection detection to prompt-text detection, because Microsoft's own
  Sentinel analytic rule reads `JailbreakDetected` from `LLMEventData`, and the bullet on prompt
  content and the matching row of the methodology's section 3.3 table are narrowed the same way.
  The sentence after the warning now describes what this file uses the table for rather than what
  the table supports. Verification step 3 said nothing tells a reader in advance what the column
  carries; it now scopes that to the table reference and carries a dated marker. No status label
  and no query changed.
- **2026-09-27 - The headline negative claim was broader than its evidence on chat interactions.**
  `README.md`, section 3.1 of the methodology and the scope document said no table or column in the
  pages read documents Microsoft 365 Copilot user prompts or chat interactions. The `CopilotActivity`
  table carries `CopilotInteraction` audit records, and advanced hunting in the Defender portal can
  query a connected Microsoft Sentinel workspace's tables, per
  [advanced hunting with Microsoft Sentinel data](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-microsoft-defender)
  (`ms.date` 2026-06-09, read 2026-09-27), which says "many of that workspace's tables" and does not
  name `CopilotActivity`. The claim now covers user prompts at all three sites. The README and the
  methodology also state it as one about the native Defender XDR schema, call the schema table list
  an enumeration of every native table, and record a further bound: a `CloudAppEvents` filter on
  `ActionType == "CopilotInteraction"`, used in queries published outside Microsoft, uses a value no
  Microsoft Learn page read for this pack documents for that column. The methodology adds that the
  Purview page documents `CopilotInteraction` as the `Operation` of an audit record, which is a
  different field. MSD-007's opening warning now calls its table not a native Defender XDR
  advanced-hunting table. The headline claim about chat prompts is unchanged, and its two
  load-bearing figures were re-derived on 2026-09-27 and still reproduce.
- **2026-09-27 - Two `AgentsInfo` column names are Requires further validation in MSD-003 and
  MSD-004.** The Defender XDR `AgentsInfo` reference both files quote lists `AgentName` and
  `EntraAgentId`, re-read 2026-09-27 at its unchanged 2026-06-03 stamp. The Azure Monitor Logs
  reference for the same table lists `Name` and `EntraAgentID` (`ms.date` 2026-07-31), and the
  Defender for Endpoint page on discovering local AI agents queries the table in advanced hunting
  with `Name` (`ms.date` 2026-09-16). Microsoft's own sources conflict, which is that label's
  definition in the canonical legend. The label is scoped to the column names: MSD-003 carries it
  for both and MSD-004 for `AgentName`, the only one of the two it uses, and neither detection's
  status moves. The queries keep the names the Defender XDR reference publishes, so a wrong name
  fails loudly as a syntax error; `column_ifexists()` was not used because it would make that
  failure silent. MSD-003's verification step 1 and checklist Group 3 now carry a `getschema` check
  that settles which names a workspace carries. The free-text column lists in the scope document
  and the checklist note the contested name, and MSD-008 now calls MSD-007 the one detection whose
  status is Requires further validation rather than the one row carrying that label, since the
  label now also appears scoped to columns. No query changed.
- **2026-09-27 - `CloudAppEvents` agent `ActionType` values are documented.** The scope document
  and section 6 of the methodology said the `ActionType` values for agent activity in
  `CloudAppEvents` are not documented. The
  [Agent 365 observability concepts page](https://learn.microsoft.com/en-us/microsoft-agent-365/developer/observability-concepts)
  (`ms.date` 2026-09-02, read 2026-09-27) documents five, `InvokeAgent`, `InferenceCall`,
  `ExecuteToolBySDK`, `ExecuteToolByGateway` and `ExecuteToolByMCPServer`, without saying which
  span type produces which. Both sites now say so and carry dated markers, and the scope document's
  trigger table records that the first half of its `CloudAppEvents` trigger was met. The v0.2
  deferral stands on its stated reason, the cost of moving every inline count. The scope
  document's note that the table reference does not settle `ActionType` against `ActivityType` now
  adds that this page places the operation in `ActionType`, and section 1 of the methodology names
  `BehaviorInfo.ActionType` as the column its invented illustration stands in for, where "that
  column" could now be read as the documented one. No query changed.
- **2026-09-27 - The Defender for Endpoint page on local AI agents reaches several statements about
  `AgentsInfo`.** That page reports local AI agents discovered on onboarded devices in `AgentsInfo`
  ([Discover local AI agents with Microsoft Defender for Endpoint](https://learn.microsoft.com/en-us/defender-endpoint/discover-local-ai-agents),
  `ms.date` 2026-09-16, read 2026-09-27). The sites it reaches:
  - MSD-003 said a developer-configured MCP server on a workstation is outside its table. On a
    device where local agents are discovered, a local MCP server is inside it, in `RawAgentInfo`,
    and a remote one is in `McpServers`. Corrected, with a dated marker.
  - MSD-004 said agents outside Microsoft Agent 365 management are outside it. Local agents are
    reported in the table, and on this pack's reading a published, active one with declared tools
    reaches the primary query, because the page lists the columns carrying local-agent data without
    `Guardrails`. Corrected, with a dated marker.
  - Checklist Group 3 said the table is empty without the Microsoft 365 app connector, and MSD-003's
    prerequisite heading said what has to be true for the table to hold anything. The page names no
    connector among its prerequisites and says "You don't need additional deployment, configuration,
    or scripts beyond the device onboarding requirements". Both now limit that to Agent 365 agents
    and carry dated markers, and MSD-007's reference to the prerequisite is
    narrowed the same way.
  - Section 3.3 of the methodology said the table's emission cadence is not documented at all, and
    MSD-003 and MSD-004 said Learn does not document it. The page states that "AgentsInfo adds a
    record each time an agent profile is updated", which names one trigger and no schedule. The
    methodology's table cell said "Not documented at all", which the page falsifies; a table cell
    cannot carry a marker, so this entry is the record. MSD-003's and MSD-004's statements are
    narrowed rather than falsified, because the page does not say whether rows are also written on
    a schedule: each window paragraph now scopes its statement to the table reference and quotes the
    page, and each cannot-see bullet names the trigger. The 30-day windows, the cadence step and the
    query comments are unchanged.
  - MSD-003, MSD-004, the checklist and section 3.3 said no field names are published inside
    `McpServers` or `DeclaredTools`. The page's queries read `name`, `type` and `endpoint` from
    both, scoped to `Platform == "LocalAgents"`. Each site now scopes its statement to the table
    reference. The Provisional labels stand, because the table reference still publishes nothing and
    the page speaks for one platform.
  - A new section of the scope document records local-agent posture, nested in `RawAgentInfo`, as a
    gap neither detection covers, and the lead of that list now says its last entry is a gap found
    after the scope was set.

  No query changed.
- **2026-09-27 - This file paired the syntax-error cause with the semantic-error example.** The
  2026-09-21 entry above quoted the cause the advanced hunting errors page gives for syntax errors
  and then gave `'project' operator: Failed to resolve scalar expression named 'x'` as its example.
  On that page, re-read 2026-09-27 (`ms.date` 2026-05-18), that message is the example for semantic
  errors, and the syntax-error row's example is `A recognition error occurred.`. The entry now gives
  the matching example. Its conclusion is unaffected, because both kinds of error stop the query.
- **2026-09-27 - MSD-006 cites a narrower gap on `ConditionalAccessStatus` values.** The Log
  Analytics reference for the sibling `SigninLogs` table publishes three of the four strings
  Microsoft Graph documents, "Possible values: success, failure, or notApplied", for its own
  `ConditionalAccessStatus` column (`ms.date` 2026-08-27, read 2026-09-27), while the
  `AADServicePrincipalSignInLogs` reference, re-read the same day, still publishes none. MSD-006's
  note now says so. The Provisional label, the placeholder and the shipped
  query are unchanged, because the published strings belong to a different table.
- **2026-09-27 - MSD-008 records a published-agents limit that Microsoft states for threat
  detection.** The AI-agent detection page states that "Threat detection is supported only for
  published Microsoft Foundry agents" (re-read 2026-09-27, `ms.date` 2026-08-07). The real-time
  protection page, which describes Prompt Shields block events being recorded as behaviours, states
  no such limit (`ms.date` 2026-07-01, read 2026-09-27). MSD-008 now quotes the sentence with its
  own scope, records that whether block events from unpublished agents reach `BehaviorInfo` is not
  established, and suggests a published agent for its positive control. No status label and no
  query changed.
- **2026-09-27 - MSD-002 cites Microsoft-authored queries that use its observed spelling.** Queries
  in the Microsoft Defender XDR solution of Microsoft's Azure-Sentinel repository compare
  `DeliveryLocation` with `"Junk folder"` and `"Inbox/folder"`, read 2026-09-27 at pinned commits.
  That corroborates the spelling MSD-002 carries as observed. It is not a Microsoft Learn citation,
  so the value stays in the observed class and both of its point-of-use labels are unchanged.
- **2026-09-27 - The README's licence note no longer implies it covers every quoted page.** It said
  four documentation repositories are in play and listed their licence files. This version quotes
  pages from the Purview and Agent 365 documentation, and MSD-006 has quoted the Microsoft Graph
  documentation since 2026-09-19; none of those is among the four. The note now says four were
  checked and that pages from other repositories are not covered by it. No licence file was read
  for this change.
- **2026-09-27 - The methodology states the failure its "never indexed into" handling guards
  against.** On this pack's reading, reading a key that a `dynamic` value does not carry returns
  null rather than an error. Section 3.3 now says so, records that no Kusto reference page read on
  2026-09-27 states it and that no run of this pack has tested it, and names `bag_has_key()` for
  where a key's absence
  matters, noting that its reference settles nothing about availability in Defender XDR advanced
  hunting.

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
    `BehaviorInfo` table reference, re-read for its column table after one tenant's schema did
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
    earlier read. **MSD-008's step 1 cites it**, and MSD-008 records that step 2 and the self-check
    wrapper name it nowhere.
  - **A page re-read to confirm a quotation rather than to add a citation:** the `BehaviorInfo` table
    reference, re-read for the deployment sentence MSD-008 had paraphrased and now quotes in full.
    **The citation keeps the date it already carries**, because that read changed the form of a
    quotation and not a claim.
- **2026-09-20 for a staleness re-read of six cited pages, which is the checklist's Group 8 item
  performed rather than deferred.** Five of the six had been updated since this pack read them, and
  two of those updates had already falsified copy this pack marks verbatim. **The sweep is the entry
  here, not only the two corrections**, because a fix confined to the mismatches would leave the
  other three changed pages unchecked.
  - **Pages whose rendered date moved, with that date read on 2026-09-20:** the `EmailEvents` table
    reference, **2026-09-02**; the Defender for Office 365 prompt injection guide, **2026-09-08**;
    the custom detection rules page, **2026-09-02**; the AI agent detection and protection page,
    **2026-09-03**; and the `AADServicePrincipalSignInLogs` table reference, **2026-08-28**.
  - **Which field a page stamp names, stated here because this pack uses the word for more than
    one.** In the five figures above, and in every entry this pack dates to this sweep, a page stamp
    is **the date Learn renders on the page itself, under the words "Last updated on"**. It is
    neither of the two fields the page publishes in its metadata: re-derived over these same five
    pages on 2026-09-20, the rendered date reproduces all five figures above, `updated_at`
    reproduces three of them and `ms.date` reproduces one, the custom detection rules page being the
    only one where all three agree. **Where an entry names `ms.date` or `updated_at` explicitly,
    that named field is what it means**, and those entries are the reason the unqualified form
    needed defining rather than an inconsistency with it.
  - **The limit on that definition, which is the reason it is scoped rather than general.** The two
    earlier figures in section 3.1 of the verification methodology were re-derived against the live
    pages on this same date and **both still reproduce**, and that file now names the field at each;
    on those two pages the rendered date and `ms.date` agree, so neither figure distinguishes them.
    **One earlier figure was not re-derivable and is left as recorded**: the prompt injection
    guide's stamp in MSD-001, where the page has since moved on both fields and neither now matches
    the figure, so which one it came from cannot be established from the page. **It is not restated
    as a rendered date**, and closing it needs a fresh read against a named field rather than an
    inference from this entry.
  - **A product Microsoft renamed, corrected at three sites.** The AI agent detection and protection
    page now names the declarative-agent builder `Microsoft Copilot Agent Builder`, where this pack
    had quoted it with `365` in the name. Corrected in MSD-003, in MSD-008 and in the workspace
    verification checklist, all three of which introduce the sentence as verbatim. **The Copilot
    Studio half of MSD-008's coverage note was confirmed unchanged on the same read**, because that
    half is the operationally important one.
  - **A schema description Microsoft reworded, corrected in one cell.** `AppId` on the
    `AADServicePrincipalSignInLogs` reference now reads "Microsoft Entra ID" where it read "Azure
    Active Directory". MSD-006 tells a reader to diff that table against the page and record any
    difference as a schema mismatch, so the stale cell was manufacturing a false positive against
    this pack's own Group 1 step. The other rows quoted there were unchanged, including the
    "Th identifier" typo in `FederatedCredentialId`, which is Microsoft's.
  - **A page title claim made precise rather than withdrawn.** MSD-002 attributed the title "The
    case-insensitive in~ string operator" to the `in~` reference page. That string is the page's
    document title and does not appear in the article body, whose heading reads `in~ operator`. The
    file now cites the heading and the lead sentence, which are both checkable on the rendered page.
  - **What did not change.** The quotations this pack takes from the `EmailEvents` page, the prompt
    injection guide and the custom detection rules page were re-checked on the same read and still
    match. **No status label moved on any of the five pages**, and no query changed.
  - **An operator citation corrected, and a page added to MSD-003's Sources.** MSD-003 cited the
    case-sensitive `in` page for the `!in~` form its filters ship, on the ground that the form was
    documented only in the comparison table shared across the `in` pages. **That ground was wrong.**
    The `!in~` reference page carries both a "List of scalars" section, which exemplifies the
    parenthesised scalar list MSD-003 ships, and a "Dynamic array" section, which exemplifies the
    form MSD-004 ships; MSD-004's own Sources entry had recorded the first of those all along. That
    page is now cited in MSD-003 for the section that exemplifies its form, re-read 2026-09-20, and
    the case-sensitive page is kept for the `in`-family case-sensitivity note only. **No query
    changed**, and the operators the two files ship are unchanged.
  - **The `arg_max()` reference page added to MSD-003's and MSD-004's Sources, read 2026-09-20.**
    Both files argued from that function's semantics without citing it: that it reduces the table
    to one current row per agent, that it shows current state only, and that it hides churn. It is
    the construct both change-detection baselines turn on. Its Returns statement, "Returns a row in
    the table that maximizes the specified expression", is now quoted at the point of use in both
    files. **What that closed and what it left open, counted rather than characterised:** this pack
    cites 16 Kusto reference pages and `arg_max()` is now one of them, and **no join-family
    reference page is among them**. The anti-join is argued from without one: across the detection
    files and the checklist, the literal `leftanti` appears at 6 sites in 5 of them, 3 being shipped
    query legs written `join kind=leftanti` and 3 being prose. **That denominator excludes this
    file**, because a count written into the artefact it counts moves as it is written. **That
    citation is still owed**, and this entry records it as open rather than as closed. **No query
    changed.** What did change is the prose each file carries about what its own baseline holds,
    corrected in the same pass and described in the entry below.
  - **What each change-detection baseline actually holds, corrected in MSD-003 and MSD-004.**
    MSD-003 said its baseline leg's missing lifecycle filter "runs in the safe direction" because
    "a configuration already seen stays recognised as already seen". **That is false of a leg that
    collapses to one row per agent**: a configuration superseded before the window closed is not in
    the baseline, and a revert to it alerts. MSD-004 attributed the shrinking of its inclusion list
    entirely to lookback truncation, which arises only where the frequency is shorter than the
    baseline reaches back, **where the collapse shrinks it at every frequency with no truncation
    involved**. Both files now state what their baseline holds and what it costs; the checklist
    step that restated MSD-004's attribution now says it checks truncation only; and MSD-004
    carries a new verification step for re-running the variant after a scheduling gap. **Each
    corrected site carries a dated marker.** **No query changed**, and the `distinct` in each
    baseline leg is kept with a comment recording that it is a no-op as the leg stands and becomes
    load-bearing the moment the collapse above it is removed.
- **2026-09-27 for the reads the twelve corrections dated 2026-09-27 under Fixed rest on**, in the
  senses the sub-entries below name. Dates given as `ms.date` name that field.
  - **Pages new to the pack:** the Purview page on Copilot audit logs; the Azure Monitor
    example-queries page for `CopilotActivity`; the page on advanced hunting with Microsoft Sentinel
    data; the Azure Monitor Logs reference for `AgentsInfo`; the Defender for Endpoint page on
    discovering local AI agents; the Agent 365 observability concepts page; the `SigninLogs` table
    reference; the page on protecting AI agents in real time; and two Kusto references, the
    `dynamic` data type page and `bag_has_key()`.
  - **Not Microsoft Learn, and cited as corroboration only:** four files in Microsoft's
    Azure-Sentinel repository, each pinned to a commit - the Copilot sample row and the Copilot
    jailbreak analytic rule cited by MSD-007, and two Microsoft Defender XDR solution hunting
    queries cited by MSD-002. None of them sets a status or moves a value's class.
  - **A new reason on a page already cited:** the advanced hunting errors page, now in MSD-003's
    and MSD-004's Sources for why a wrong column name fails loudly, and re-read for the example
    message corrected above; the AI agent detection and protection page, for the published-agents
    sentence MSD-008 now quotes; and the advanced-hunting schema tables page, for the sentence
    section 3.1 of the methodology now quotes about where `ActionType` values are listed.
  - **Pages re-read to settle a question rather than to add a citation:** the Defender XDR
    `AgentsInfo` reference, whose rendered date is unchanged at 2026-06-03; the `CopilotActivity`
    reference, whose `LLMEventData` row is unchanged; the `CloudAppEvents`
    reference, which lists no `ActionType` values and none of the three names section 1 of the
    methodology discusses; the `AADServicePrincipalSignInLogs` reference, which still publishes no
    `ConditionalAccessStatus` values; and the advanced hunting overview and the schema tables list,
    whose section 3.1 figures reproduce at unchanged rendered dates. None of these moves a
    citation date, per the rule stated under 2026-08-18.
  - **A Microsoft Learn site search, recorded because two absence claims rest on it.** It returned
    no page for `DataExfiltrationDetected` or `AbnormalCopilotBehavior` and one, the Purview page,
    for `XPIADetected`, which is the control. And no page it returned for `CopilotInteraction` and
    `CloudAppEvents` carries both terms.

### Workspace verification

**The queries in this pack were run against a workspace on three dates: 2026-08-24, 2026-08-26
and 2026-09-11.** **No run completed every step of the verification checklist**, and the steps
not reached are recorded in the checklist itself rather than counted here.

**No detection in this pack has been observed firing.** Every query here therefore remains a
construction validated against Microsoft Learn schema documentation rather than an observed result.

**Nothing these runs showed reaches past those three dates**, or past the workspace they were
run against. What each detection cannot see, and what its own verification steps ask you to
establish before you deploy it, is stated in the detection file.

### Known gaps carried into this version

- No positive control has been **observed firing** for any detection, so none of the eight can
  distinguish a clean environment from a broken filter without one. The gap is widest for five of
  them when the result comes back empty, which is a judgement rather than a category derived from
  a rule;
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
