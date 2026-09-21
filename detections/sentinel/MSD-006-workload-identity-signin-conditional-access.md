# MSD-006 - Workload-identity sign-in where Conditional Access did not apply

| Field | Value |
|---|---|
| **ID** | MSD-006 |
| **Deployment target** | Microsoft Sentinel (Log Analytics) |
| **Primary table** | `AADServicePrincipalSignInLogs` |
| **Status** | **GA (no preview qualifier)** for the table - see the status legend on the distinction. **Five Provisional elements** below: the `ConditionalAccessStatus` value set, the `ResultType` stored values, the `Agent` column, and the serialisation of `ConditionalAccessPolicies` and of `LocationDetails`. |
| **Last verified** | 2026-08-15 |
| **MITRE ATLAS** | `AML.T0012` Valid Accounts |
| **OWASP LLM 2026** | LLM03:2026 Excessive Agency |

> **`AML.T0083` Credentials from AI Agent Configuration is deliberately not in the ATLAS cell**,
> though it is the technique a reader might expect on a detection about agent identity. The
> condition it describes sits **outside** what this query can see: an agent authenticating with a
> key held in its configuration bypasses Microsoft Entra ID and produces no row in this table at
> all. A technique ID in a column headed MITRE ATLAS reads as coverage, so putting it there would
> claim the opposite of what this detection can do. It is recorded in the cross-walk's not-mapped
> table instead, and the blind spot it names is the first section of this file.

## Purpose

Show which service principals - including the ones behind AI applications and agents - are signing
in without a Conditional Access policy being applied, and what they are reaching.

Conditional Access for workload identities is the one **preventive** control in this pack's
coverage. This detection measures whether it is actually in the path.

## The limit that decides how you read the results

From [the Conditional Access for agents article](https://learn.microsoft.com/en-us/entra/identity/conditional-access/agent-id),
verbatim: "Conditional Access only protects resources secured by Microsoft Entra ID. For example, if
an agent accesses resources using an API key, it bypasses the Microsoft Entra ID authentication and
token issuance pipeline entirely and Conditional Access policies won't apply to them."

*Attribution note, because this pack's own rule is that every quote names its own page:* that
sentence is **not** on the workload-identity article, which is where a reader would look for it.
It was confirmed on the agents article on 2026-08-15, and on no other page in this file's source
list.

**An agent authenticating with a key produces no row in this table at all.** So a clean result here
is not evidence of coverage - it is evidence about the identities that did authenticate through
Entra ID. This detection cannot see the population it most needs you to worry about, and no query
can fix that. Pair it with an inventory of how your AI applications authenticate.

Further scope limits, each attributed to the page it was read on and the date it was read:

- From [the workload-identity article](https://learn.microsoft.com/en-us/entra/identity/conditional-access/workload-identity),
  2026-08-15: "Policy can be applied to single tenant service principals that are registered in your
  tenant. Microsoft and third-party SaaS applications, including multitenant apps, are not covered by
  these policies."
- "Managed identities aren't covered by policy." - 2026-08-15, present on **both** the
  workload-identity article and [the users, groups, agents and workload identities article](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-users-groups).
- From the workload-identity article, read 2026-08-17: "While service principals can be added to
  groups, Conditional Access policies assigned to a group that contains a service principal are not
  enforced for that service principal. To enforce a Conditional Access policy for a service principal,
  it must be assigned directly to the policy as a workload identity." **This one changes how you read
  a no-policy-applied row.** A tenant that scoped its service principals by group has no
  workload-identity coverage while believing it has some, and those identities land in this
  detection's ordinary no-policy-applied population with nothing in the row to distinguish them from
  an identity nobody ever tried to cover. Check how your policies are assigned before you read that
  population as a backlog.

Creating or modifying such policies requires a plan, from the workload-identity article: "Workload
Identities Premium licenses are required to create or modify Conditional Access policies scoped to
service principals."

## Schema this depends on

Columns quoted from the `AADServicePrincipalSignInLogs` table reference on Microsoft Learn, read
2026-08-15, **re-read 2026-09-20, when the rendered "Last updated on" date read 2026-08-28** and the
page's `ms.date` read 2026-08-27, which is the one-day split these generated reference pages carry.
That re-read changed one cell:
`AppId`'s description now reads "Microsoft Entra ID" where it read "Azure Active Directory" before,
and the table below carries the current wording. The other rows quoted here were unchanged on that
read, including the "Th identifier" typo in `FederatedCredentialId`, which is Microsoft's.

| Column | Data type | Learn description (verbatim) |
|---|---|---|
| `TimeGenerated` | `datetime` | The date and time of the event in UTC |
| `ServicePrincipalId` | `string` | ID of the service principal who initiated the sign-in |
| `ServicePrincipalName` | `string` | Service Principal Name of the service principal who initiated the sign-in |
| `AppId` | `string` | Unique GUID representing the app ID in the Microsoft Entra ID |
| `ConditionalAccessStatus` | `string` | Status of all the conditionalAccess policies related to the sign-in |
| `ConditionalAccessPolicies` | `string` | Details of the conditional access policies being applied for the sign-in |
| `ClientCredentialType` | `string` | The type of client credential used. Examples include client assertion, client secret, etc. |
| `FederatedCredentialId` | `string` | Th identifier of an application's federated identity credential if a federated identity credential was used to sign in. |
| `ResourceDisplayName` | `string` | Name of the resource that the service principal signed into |
| `ResourceIdentity` | `string` | ID of the resource that the service principal signed into |
| `IPAddress` | `string` | IP address of the client used to sign in |
| `LocationDetails` | `string` | Details of the sign-in location |
| `ResultType` | `string` | The result of the sign-in operation can be Success or Failure |
| `ResultDescription` | `string` | Provides the error description for the sign-in operation |
| `Agent` | `string` | Details of agentic sign-in. |
| `CorrelationId` | `string` | ID to provide sign-in trail |

**`FederatedCredentialId`'s description opens "Th identifier", and the typo is Microsoft's.** It is
reproduced as published rather than silently corrected, confirmed against the live reference page on
2026-08-19. MSD-005 marks a Microsoft grammatical error on its own table description in the same way
and for the same reason: a reader diffing this table against the page should find the two identical.

> **Provisional: the `ConditionalAccessStatus` value set. Corrected 2026-09-19.** **An earlier
> version of this note said the column "has no documented value list". That is wrong as written**,
> and the correction matters because this note is the stated reason for the shipped query's shape.
> **Two different surfaces are involved.** The Azure Monitor Logs reference for this table
> documents what the column means and publishes no value list for it. **Microsoft Graph does
> publish an enumeration for the analogous `signIn` property**, `conditionalAccessStatus`: "The
> possible values are: `success`, `failure`, `notApplied`, and `unknownFutureValue`."
>
> **What is genuinely undocumented is the link between the two.** No page this pack has found
> states that the Log Analytics column stores exactly those strings, in that casing. **The shipped
> query therefore still does not filter on a status value** - it groups by the column so you can
> read what your own environment stores, and the narrowing filter is one you complete afterwards.
> **The reason is no longer that guessing would invent schema; it is that the documented
> enumeration belongs to a different surface and this column's own values are unconfirmed.**
>
> Sources: the [`AADServicePrincipalSignInLogs` table
> reference](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/aadserviceprincipalsigninlogs)
> and the [Microsoft Graph `signIn` resource
> type](https://learn.microsoft.com/en-us/graph/api/resources/signin?view=graph-rest-1.0), both read
> 2026-09-19.

> **Provisional: `ResultType`.** Learn describes it as "The result of the sign-in operation can be
> Success or Failure", which describes the semantics rather than the stored strings. The shipped
> query does not filter on it. Confirm your own values before adding one. **A tenant stored
> numeric codes there on 2026-08-24 rather than either of the two words Learn names**, which is the
> reason this note exists and is worth reading before anyone writes `ResultType == "Success"`.

> **Provisional: `Agent`.** Learn's entire description is "Details of agentic sign-in." No shape, no
> values. It is projected for review and never filtered on. Treat any read of it as unverified until
> you have inspected it. **A tenant returned a JSON object on 2026-08-24**, which is a shape
> rather than a value set, and one tenant rather than a documented guarantee.

> **Provisional: the serialisation of `ConditionalAccessPolicies` and `LocationDetails` is not
> documented.** Learn's Azure Monitor reference types both columns `string`, and describes their
> contents as "Details of the conditional access policies being applied for the sign-in" and
> "Details of the sign-in location" - composite descriptions with no published value format. Whether
> your workspace stores a flat string or a serialised structure in either one is therefore not
> established by the page. **A tenant answered it on 2026-08-24**: `ConditionalAccessPolicies`
> came back a JSON array and `LocationDetails` a JSON object, so an operator there needs
> `parse_json()` rather than a string comparison. **The label stays Provisional**, because it records
> what the cited page confirms, and the page still publishes no format for either column. Inspect
> both before writing any filter on them - verification step 6 - because the answer decides whether
> an operator reaches for `parse_json()`, and one tenant's answer on one date is not yours.

> **The `in` operator is case-sensitive**, which matters more here than anywhere else this pack asks
> an operator to paste a value they observed: every narrowing filter below is one an operator pastes
> from their own observed output. A casing difference returns zero rows and looks like a clean
> environment. Microsoft documents `in~` as the case-insensitive form; prefer it when pasting
> observed values.

## Query

```kusto
// MSD-006 step 1 - the Conditional Access posture of every workload identity, by status value.
// Deployment target: Microsoft Sentinel analytics rule or hunting query (Log Analytics).
// Schema verified against Microsoft Learn on 2026-08-15.
// No filter on ConditionalAccessStatus: this column's reference publishes no value list, and the
// enum Graph documents for the analogous property is unconfirmed here. Read yours from this output.
AADServicePrincipalSignInLogs
| where TimeGenerated > ago(7d)
| summarize
    SignIns = count(),
    Resources = make_set(ResourceDisplayName, 20),
    DistinctSourceIPs = dcount(IPAddress),
    CredentialTypes = make_set(ClientCredentialType, 10),
    FirstSeen = min(TimeGenerated),
    LastSeen = max(TimeGenerated)
    by ServicePrincipalName, ServicePrincipalId, AppId, ConditionalAccessStatus
| order by SignIns desc
```

Read the `ConditionalAccessStatus` values that appear. Then complete the narrowing query:

```kusto
// MSD-006 step 2 - narrowed to the not-evaluated case.
// Fill the value from your own step-1 output. Do not deploy with the placeholder in place.
// in~ rather than in: the values are ones you paste by hand, and in is case-sensitive.
let NoPolicyApplied = dynamic([
    // "<ConditionalAccessStatus value(s) observed in your tenant that mean no policy was applied>"
]);
AADServicePrincipalSignInLogs
| where TimeGenerated > ago(1d)
| where ConditionalAccessStatus in~ (NoPolicyApplied)
| project
    TimeGenerated,
    ServicePrincipalName,
    ServicePrincipalId,
    AppId,
    ConditionalAccessStatus,
    ConditionalAccessPolicies,
    ClientCredentialType,
    FederatedCredentialId,
    ResourceDisplayName,
    ResourceIdentity,
    IPAddress,
    LocationDetails,
    ResultType,
    ResultDescription,
    Agent,
    CorrelationId
| order by TimeGenerated desc
```

**The `NoPolicyApplied` literal above parsed in a tenant on 2026-08-24.** It is the comment-only
`dynamic([...])` form, MSD-008 ships the same construct, and checklist Group 1b check 2 prints the
runnable test for it. **That run ran the Group 1b test on both surfaces, and ran this step directly
in the Sentinel workspace**, which is the deployment target this file declares: the literal parsed,
and step 2 left unfilled returned an empty match rather than a parse error. **Two outcomes need
different responses and the comment in the block covers only one**, which is why this paragraph
exists. A literal that parses to an empty list matches nothing, which is why you
fill it before deploying. A literal that does not parse fails the statement instead, so step 2 errors
rather than returning nothing. **The run answered which of the two you get, in one tenant on one
date.** Run Group 1b check 2 before you rely on this block in your own.

> **The placeholder stays empty and the shipped query is unchanged. Read your own values first**,
> and the shipped query groups by this column so that you can. That instruction is unchanged by
> everything below, which is the record of how it was reached.
>
> **Settled 2026-08-25: what not to fill it with, and why.**
> The 2026-08-24 run emitted `notApplied` for this column. **That is one environment on one
> date**, so filling the literal with it would generalise a single observation into shipped copy,
> which is the defect this pack exists to avoid. **The basis for this settlement was corrected on
> 2026-09-19.** It originally rested on the claim that the value was not documented anywhere, and
> Microsoft Graph does document `notApplied` for the analogous property. **The settlement stands on
> the narrower ground**: a value read in one environment is not a value set to ship, and filtering
> to a single status narrows the detection rather than widening it. The value is recorded here so a
> reader completing step 2 knows what one workspace produced.
>
> **Re-observed 2026-09-11.** A third run, on a later date, read the same value for this column.
> **That agreement is not corroboration and is not treated as any here**, for the reason set out
> immediately below.
>
> **Agreement between two readings is only informative if at least one of them could have returned a
> different value, and this pack never established that for either reading.** Microsoft states that
> Conditional Access policy
> for workload identities "can be applied to single tenant service principals that are registered in
> your tenant. Microsoft and third-party SaaS applications, including multitenant apps, are not
> covered by these policies." That is the workload-identity article quoted in the scope limits above,
> re-read on 2026-09-19. **So for sign-ins of the excluded kind no policy could apply and the status
> follows by design**, whatever the environment. **This pack publishes no policy
> inventory**, so it cannot separate that mechanism from a genuine finding. **The control that would
> be informative is an environment known to hold at least one policy in scope targeting workload
> identities, which is a new run rather than a re-reading.**
>
> **The 2026-08-25 settlement stands on its own reason**, which is the one given above and is
> sufficient without any appeal to agreement between readings.

### New-identity and new-resource change detection

```kusto
// A workload identity reaching a resource it has not reached in the previous 30 days.
let baseline =
    AADServicePrincipalSignInLogs
    | where TimeGenerated between (ago(31d) .. ago(1d))
    | distinct ServicePrincipalId, ResourceIdentity;
AADServicePrincipalSignInLogs
| where TimeGenerated > ago(1d)
| join kind=leftanti baseline on ServicePrincipalId, ResourceIdentity
| summarize
    SignIns = count(),
    SourceIPs = make_set(IPAddress, 10),
    LastSeen = max(TimeGenerated)
    by ServicePrincipalName, ServicePrincipalId, ResourceDisplayName, ConditionalAccessStatus, ClientCredentialType
| order by LastSeen desc
```

This variant is usually the more useful of the two. Step 1 is a census; this one alerts on change,
which is what a scheduled rule should do. The reason is in the false-positive guidance below.

**What the four shipped queries in this file return stays in your environment, and they do not
return the same things.** Step 1 groups by named service principals and returns a **count** of
distinct source network addresses, `DistinctSourceIPs`, beside the resources those principals
reached, the credential types they used, and first-seen and last-seen timestamps. **The
change-detection variant returns a set** of source addresses, `SourceIPs`, beside the same
principals and resources. **Step 2 carries most of all**: it is a per-row projection, one row per
sign-in, naming the principal, the application identity, the Conditional Access policies, the
federated credential identifier, the resource, the source address, the location details, `Agent` and
the correlation identifier. The credential-type view groups named principals by credential type.
**The timestamps are estate data as much as the addresses are**, which is the rule MSD-008's
verification step 2 states for first-seen and last-seen. **`Agent` is a free-text column**: its
one-sentence description is of per-event content rather than of a class this platform assigns, which
is the test verification step 4 below states and the test the checklist applies. **Learn publishing
no value set for a column is not on its own what makes it free text**, and step 4 names the
counterexample this file already ships. So for `Agent` what travels is the column name and the shape
you found and never the contents. **Apply that same test to any other free-text column these queries
return rather than looking for it on a list.** **This file's own queries return these**:
`ServicePrincipalName` and `ResourceDisplayName` carry a description of per-event content and no
published value set, exactly as `Agent` does, so what travels from either is the column name and the
shape you found and never the contents. What travels outward is a column name or a value name, never
a result row, a row count, or anything inside one. **That default holds where a column carries a
closed set the platform defines; where a column carries free text the value is itself estate data
and the default does not reach it.** Checklist Group 0 states the same rule for every step in the
pack, and this file repeats it because a reader who deploys one detection may never open the
checklist.

### Credential-type view

```kusto
// Which workload identities authenticate with a secret rather than a certificate or
// federated credential. This is context for the blind spot at the top of the file rather
// than a technique mapping: a secret in agent configuration is a credential an attacker can
// lift, and Conditional Access does not change that.
AADServicePrincipalSignInLogs
| where TimeGenerated > ago(30d)
| summarize SignIns = count(), LastSeen = max(TimeGenerated)
    by ServicePrincipalName, ClientCredentialType, ConditionalAccessStatus
| order by SignIns desc
```

## Status evidence

- **The table is a documented Azure Monitor Logs table** and its reference page carries no preview
  qualifier as read on 2026-08-15. Under this pack's status rule that is the GA signal for the
  telemetry surface.
- **Conditional Access for workload identities is GA (no preview qualifier), and here is the
  evidence rather than the assertion:** a full-text search of the workload-identity article on
  2026-08-15 returned **zero occurrences of the word "preview"**, re-run on 2026-08-19 with the same
  result. **The positive control for that zero, on the same search of the same article, is the
  literal "Conditional Access" at 24 occurrences**, which is what shows the search reached the
  article body rather than returning nothing. **Both figures are over the article body rather than
  the whole page**, which is the scope the search covered and the only scope they speak to. That is
  this pack's GA test, and
  it is the same class of evidence as MSD-001 and MSD-002 rather than the stronger stated-GA
  evidence MSD-005 carries.
- **Agent-user targeting is Public Preview.** Per
  [the target-agent-identities how-to](https://learn.microsoft.com/en-us/entra/identity/conditional-access/howto-target-agent-identities),
  verbatim: "The **agent users** option targets agents' user accounts. This option is currently in
  Preview."
- **One sub-question is unsettled and is not asserted here.** The users-and-groups article labels
  agent scoping "Agents (Preview)" as a class, while the how-to tags the *agent users* option as
  Preview in the sentence quoted above. What those pages do not settle is whether targeting an
  *agent identity*, as distinct from an *agent user*, carries its own preview state. It does not affect the service-principal capability this detection
  measures.

## What this detection cannot see

- **Any agent or application authenticating with an API key.** Quoted in full above. This is the
  largest blind spot in the detection and it is structural, not a tuning problem.
- **Managed identities**, which Learn states are not covered by policy.
- **Microsoft and third-party SaaS applications, including multitenant apps**, which Learn states
  are not covered by these policies.
- **Whether an applied policy was the *right* policy.** `ConditionalAccessStatus` records that
  evaluation happened, not that the control was adequate. Read `ConditionalAccessPolicies` to see
  which policies were involved.
- **Whether the service principal belongs to an AI workload.** Nothing in this table labels that.
  Correlate `AppId` against your own application inventory, or against
  [MSD-003](../defender-xdr/MSD-003-agents-with-mcp-servers.md)'s `EntraAgentId` where the agent is
  Agent 365-managed.

## False-positive guidance

- **Expect a large standing population, and expect most of it to be legitimate.** Conditional Access
  policies for workload identities are commonly scoped to a subset of service principals rather than
  to all of them, so a sign-in with no policy applied is the ordinary case rather than the finding.
  Deploying step 2 as a per-event alert will bury you. Use the change-detection variant as the
  alerting rule and step 1 as the periodic posture review.
- **High-volume automation dominates the counts.** Ranking by `SignIns` surfaces your busiest
  integrations, not your riskiest ones. Rank by `DistinctSourceIPs` or by resource sensitivity
  instead.
- **A new resource is often a deployment, not an intrusion.** Correlate the change-detection variant
  with change records before escalating.
- **`ServicePrincipalName` is not stable across all sign-in types.** Join on `ServicePrincipalId` or
  `AppId` when you need identity continuity.
- **Empty `ConditionalAccessStatus` is not the same as "no policy applied".** It may mean the field
  was not populated for that sign-in type. Confirm before treating it as a finding.

## Workspace verification before deployment

1. Confirm the table exists and its columns match this file:
   `AADServicePrincipalSignInLogs | getschema | project ColumnName, ColumnType`. **Diff that output
   against the schema table above.** Record as a **schema mismatch** any column the schema table
   lists which the output does not carry, or which the output types differently. **A column the output
   carries and the schema table does not is not a mismatch**, because the schema table is a deliberate
   subset. That includes the two columns whose serialisation is flagged Provisional - `getschema`
   settles their name and data type and cannot settle their serialisation, which is step 6. Then run
   `AADServicePrincipalSignInLogs | take 10`. If it comes back empty, confirm that service principal
   sign-in logs are being exported to the workspace: this is a separate diagnostic-setting category
   from user sign-ins and is easy to miss.
2. Run step 1 and record every `ConditionalAccessStatus` value your tenant emits, with counts. This
   is the value set the shipped query deliberately does not guess.
3. Inspect `ResultType`: `AADServicePrincipalSignInLogs | summarize count() by ResultType | take
   20`. Confirm whether your workspace stores the semantic strings Learn describes or something
   else, before adding any success/failure filter.
4. Inspect the `Agent` column on a sample of rows and record what it contains. Its one-sentence
   description is of per-event content rather than of a class this platform assigns, which is what
   makes it a free-text column: there is no value name to send that is not also the content.
   **The shape, not the values**, the same rule step 6 applies two columns over. What a public issue
   or a verification report wants from here is the column name and the shape you found, and the
   contents **stay in your environment**. **Learn publishing no value set for a column is not on its
   own what makes it free text**: step 2 above asks you to send every `ConditionalAccessStatus`
   value string, and Learn publishes no value set for that column either.
5. Identify which `AppId` values correspond to AI applications and agents in your environment, and
   record that mapping next to the rule. Nothing in the telemetry does it for you.
   **That mapping stays in your environment.** It is among the most sensitive artefacts this pack
   asks you to produce, and it does not belong in a public issue or a verification report.
   **A Conditional Access policy takes a different identifier than the one App registrations shows.**
   The workload-identity article, read 2026-08-18, says "You can get the objectID of the service
   principal from Microsoft Entra Enterprise Applications", and states that the Object ID shown under
   App registrations cannot be used.
6. Inspect the two columns whose serialisation is Provisional, before writing any filter on them:
   `AADServicePrincipalSignInLogs | take 5 | project ConditionalAccessPolicies, LocationDetails`.
   Both are typed `string` on Learn, which does not tell you whether the stored value is flat or
   structured. Record which it is. **The shape, not the values.** These two columns carry
   your organisation's own policy naming and the geography its sign-ins come from, so the output of
   this step **stays in your environment** for the same reason the `AppId` mapping in step 5 does.
   What a public issue or a verification report wants from here is "flat string" or "JSON", nothing
   more.
7. Confirm whether your tenant holds Workload Identities Premium. Without it, existing policies keep
   working but cannot be created or modified - which changes what a finding here means you can do
   about it.
8. **Record how your AI applications and agents authenticate.** Any that use an API key produce no
   row in this table at all. That list is the detection's blind spot made explicit, and like the
   `AppId` mapping in step 5 it **stays in your environment**.

## Sources

- [AADServicePrincipalSignInLogs table (Azure Monitor Logs reference, Microsoft Learn)](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/aadserviceprincipalsigninlogs) - last verified 2026-08-15, re-read 2026-08-19 to confirm that the "Th identifier" opening on `FederatedCredentialId` is Microsoft's own. **Each date is kept rather than collapsed**, because each later read is what added the material recorded against it
- [Conditional Access for workload identities (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/conditional-access/workload-identity) - last verified 2026-08-15, re-read 2026-08-17 for the group-assignment limit quoted above, re-read 2026-08-18 for the service-principal object-ID instruction verification step 5 carries, and re-read 2026-08-19 to re-run the preview search recorded above. **Each date is kept rather than collapsed**, because each later read is what added the material recorded against it
- [Conditional Access for agents (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/conditional-access/agent-id) - last verified 2026-08-15; **this is the page carrying the API-key bypass sentence**
- [Conditional Access: Users, groups, agents, and workload identities (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-users-groups) - last verified 2026-08-15
- [Target agent identities (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/conditional-access/howto-target-agent-identities) - last verified 2026-08-15
- [`in` operator (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/in-cs-operator) - last verified 2026-08-15, for the case-sensitivity note
- [`in~` operator (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/in-operator) - last verified 2026-08-18, for the "Dynamic array" section, whose example passes a `dynamic([...])` literal directly to `in~` and publishes its output. It does not exemplify the `let`-bound form this file's step-2 query ships, and the `let` variant beside it substitutes another operator; [`docs/verification-methodology.md`](../../docs/verification-methodology.md) section 6 states what that leaves open
- MITRE ATLAS technique IDs read from the distributed `atlas-data` dataset, `version: 5.6.0` (release tag `v2026.07`) - verified 2026-08-15
- OWASP Top 10 for LLM Applications item numbering carried from the companion capability-status matrix cross-walk, verified there by SHA-256 against OWASP's published download on 2026-08-09; the 2026 edition and its publication date re-confirmed 2026-08-15

> **Provenance note.** The four Conditional Access citations above were **re-read live on 2026-08-15
> for this pack**, which is how the API-key sentence was traced to the agents article rather than the
> workload-identity article it had been attributed to, **and the workload-identity article was read
> again on 2026-08-17** for the group-assignment limit, **and again on 2026-08-18** for the
> service-principal object-ID instruction that verification step 5 now carries. The underlying quotes were
> first collected in the companion capability-status matrix repository on 2026-08-10.
