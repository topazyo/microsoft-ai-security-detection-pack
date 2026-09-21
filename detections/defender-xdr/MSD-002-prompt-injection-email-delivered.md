# MSD-002 - Prompt-injection email that still reached a mailbox

| Field | Value |
|---|---|
| **ID** | MSD-002 |
| **Deployment target** | Microsoft Defender XDR advanced hunting |
| **Primary table** | `EmailEvents` |
| **Status** | **GA (no preview qualifier)** - see the status legend on the distinction. Inherits MSD-001's Provisional `DetectionMethods` serialisation. |
| **Last verified** | 2026-08-15 |
| **MITRE ATLAS** | `AML.T0051` LLM Prompt Injection · `AML.T0051.001` LLM Prompt Injection: Indirect |
| **OWASP LLM 2026** | LLM01:2026 Prompt Injection |

## Purpose

Narrow [MSD-001](MSD-001-prompt-injection-email-detected.md) to the subset that matters for
response: messages Defender flagged as prompt injection that were **not** blocked or quarantined,
and therefore sat somewhere an AI assistant could read them.

MSD-001 answers "is this happening." MSD-002 answers "did anything land." They are separate
detections because they carry different response paths: MSD-001 is a trend signal, MSD-002 is a
per-message action item.

## Schema this depends on

**This is a delta table about documented values, not a complete column list.** The queries below
also read `Timestamp`, `ReportId`, `DetectionMethods`, `NetworkMessageId`, `SenderFromAddress`,
`SenderFromDomain`, `RecipientEmailAddress`, `Subject`, `ThreatTypes`, `ConfidenceLevel` and
`EmailClusterId`; every one of those is quoted with its data type and its Learn description in
[MSD-001's schema table](MSD-001-prompt-injection-email-detected.md), verbatim with any elision
marked, from the same page and the same read. Nothing this file queries is undeclared across the pair.

Columns and their documented values, quoted from the `EmailEvents` table reference on Microsoft
Learn, read 2026-08-15, when the page's `ms.date` read 2026-08-03. **Re-read 2026-09-20, when the
rendered "Last updated on" date read 2026-09-02**, which is a second field rather than a later value
of the first: `ms.date` still read 2026-08-03 on that later read. The three value lists and the
Streaming API note were unchanged on that read.

| Column | Data type | Documented values (verbatim from Learn) |
|---|---|---|
| `EmailDirection` | `string` | Inbound, Outbound, Intra-org |
| `DeliveryAction` | `string` | Delivered, Junked, Blocked, or Replaced |
| `DeliveryLocation` | `string` | Inbox/Folder, On-premises/External, Junk, Quarantine, Failed, Dropped, Deleted items |
| `LatestDeliveryAction` | `string` | *(no value list in the `EmailEvents` table reference; see note)* |
| `LatestDeliveryLocation` | `string` | *(no value list in the `EmailEvents` table reference; see note)* |

> **Delivery status has two readings, and confusing them is the likeliest way to get this wrong.**
> `DeliveryLocation` records where the message went at delivery time. Learn documents
> `LatestDeliveryLocation` and `LatestDeliveryAction` as separate columns and appends the note:
> "The LatestDeliveryLocation and LatestDeliveryAction columns are not available in the Streaming
> API." A message can therefore show `Inbox/Folder` in one column and something else in the other.
> Compare both before concluding an item is still in a mailbox. Learn does not publish value lists
> for the two `Latest*` columns on that page, so this pack does not filter on them - it projects
> them for the analyst to read.

> **What one tenant emitted on 2026-08-24, beside what Learn publishes.** The value lists in the
> table above are Microsoft's, re-read 2026-08-24 and unchanged by that read. A tenant emitted
> three `DeliveryLocation` values differing from the published spellings, two of them by case alone
> and one by a longer token, plus a fourth the published list does not carry, `Forwarded`. It also
> emitted an `EmailDirection` value the published list does not carry, `Unknown`. **The page frames
> neither list as exhaustive**: it introduces both with a bare colon and no "possible values" or
> "one of" qualifier, so an unlisted emitted value is an undocumented platform value rather than a
> contradiction of the documentation. **Both readings are kept.** Learn says what is documented, the
> run says what one workspace produced on one date, and neither replaces the other. What that
> changes below is the filter operator and one extra spelling. It changes nothing in the table above.

## Query

```kusto
// MSD-002 - prompt-injection email that was not blocked or quarantined at delivery.
// Deployment target: Microsoft Defender XDR advanced hunting.
// Columns, and the value strings below except one, verified against Microsoft Learn on
// 2026-08-15. The exception is "Junk folder", which Learn does not publish for this
// column: a tenant emitted that spelling on 2026-08-24, and it is kept beside the
// published "Junk" so the filter reaches both. The note on in~ below the first query says why.
EmailEvents
| where Timestamp > ago(7d)
| where DetectionMethods has "Prompt injection protection"
| where EmailDirection == "Inbound"
| where DeliveryLocation in~ ("Inbox/Folder", "Junk", "Junk folder", "On-premises/External")
| project
    Timestamp,
    ReportId,
    NetworkMessageId,
    EmailClusterId,
    SenderFromAddress,
    SenderFromDomain,
    RecipientEmailAddress,
    Subject,
    DeliveryAction,
    DeliveryLocation,
    LatestDeliveryAction,
    LatestDeliveryLocation,
    ThreatTypes,
    ConfidenceLevel
| order by Timestamp desc
```

**`Timestamp` and `ReportId` are projected because this is the file the README designates as a
custom detection rule.** Microsoft's custom-detection-rules page says, for "all other Defender
tables", project both "from the same event to ensure Defender identifies the original event that
triggered the alert", and names the cost of omitting them as alerts not tagged with the correct
entity scope and a less enriched alert timeline. It states this as a recommendation, not as a
condition the wizard enforces. The same page separately states that `NetworkMessageId` and
`RecipientEmailAddress` "must be present in the output results of the query to apply actions to
email messages" - both are already projected here, so the query meets that condition. **Meeting it
is necessary and not sufficient**: Microsoft states it as a requirement rather than as a guarantee,
and the page does not say the two columns are the only thing the actions depend on. Read this as one
prerequisite already satisfied, not as confirmation that the email response actions will be
available to you.

**The same page also says not to filter on `Timestamp` or `TimeGenerated`, and sets the rule's
lookback from its frequency rather than from the query**, with an exception for narrowing inside
the lookback. **The rule query above uses a seven-day window and the exposure rollup below uses
thirty days; on the page's own figures the only frequency whose lookback reaches either is every 24
hours.** MSD-001 records that in full, and this file inherits it along with the rest of MSD-001's
verification steps. **This file has no baseline leg**, so the truncation case MSD-003 and MSD-004
name does not arise here.

`Junk`, the `Junk folder` spelling beside it, and `On-premises/External` are included deliberately. A
message in the Junk folder is still
in the mailbox and can still be read by an assistant or an automation that enumerates folders; a
message routed on-premises or externally has left Defender's view without having been blocked.
Drop any of those values if your response model treats the location as contained, and **dropping the
Junk case means dropping both spellings**: removing one of the two leaves the other matching. **A
value you drop from the filter comes out of the rollup's bucket expression too**, which is the second
place each of these values is named.

**Why this filter uses `in~` rather than `in`, and why `Junk folder` sits beside `Junk`.** Both
queries in this file shipped with the case-sensitive `in` until this change was made on 2026-08-25.
**The two dates do different jobs and are kept apart deliberately: 2026-08-24 is when the defect was
observed, and 2026-08-25 is when the operator changed.** Anyone holding a copy taken between them has
the `in` version. **On 2026-08-24 a tenant emitted none of the three literals these queries then
filtered on**: two came back differing from the
published spelling only in case, and one by a longer token, so the filter could match nothing there.
**An empty result of that kind is indistinguishable from a clean estate**, which is the failure mode
verification step 2 exists to catch. Step 2 named it in advance for the `EmailDirection` `==` filter,
and it arrived on the `DeliveryLocation` filter beside it. **Why this filter changed operator and
that one did not**: the deviation observed on `EmailDirection` was an unpublished value, `Unknown`,
and case-insensitivity does not reach a different token any more than `in~` reaches `Junk folder`.
No casing variant on that column has been observed in this pack's runs, and changing its operator
would widen the population in a way no run here has measured. **Verification step 2 is what catches
either**, which is why it names that filter first. **The run did not demonstrate a concealed
detection**, because it generated no event: what it established is that the literals could not match
the spellings that tenant emitted. Microsoft's reference for the case-sensitive `in` publishes a comparison
table giving `in` as case-sensitive and `in~` as its case-insensitive counterpart, and `in~` carries
its own reference page, headed `in~ operator`, whose lead sentence reads "Filters a record set for
data with a case-insensitive string"; both read 2026-08-24, which
are this file's first reads of either, and the lead sentence re-read 2026-09-20.
`in~` removes the case dependency. **It does not remove the spelling dependency**, which is why
`Junk folder` is listed beside `Junk`: the same tenant emitted the longer form, and
case-insensitivity cannot reach a different token. Both are kept because Microsoft Learn publishes
the short form and a workspace emitted the long one, and neither observation cancels the other. The
`in~` page carries a performance note preferring the case-sensitive `in` "when possible", and this
file accepts that cost for a filter that fires. The same page also states that case-insensitive
operators "are currently supported only for ASCII-text", which these value strings are.

**One value that run emitted and this filter does not carry: `Forwarded`.** Microsoft Learn does not
publish it for this column, re-read 2026-08-24, so whether a forwarded message counts as delivered
for your response model is a decision rather than a documented fact. Add it if it is.

### Exposure rollup

```kusto
// Which mailboxes accumulated delivered-but-not-blocked injection attempts.
// Columns, and the value strings below except one, verified against Microsoft Learn on
// 2026-08-15. The exception is "Junk folder", which Learn does not publish for this
// column: a tenant emitted that spelling on 2026-08-24, and it is kept beside the
// published "Junk" so the filter reaches both. The note on in~ below the first query says why.
EmailEvents
| where Timestamp > ago(30d)
| where DetectionMethods has "Prompt injection protection"
| where EmailDirection == "Inbound"
| where DeliveryLocation in~ ("Inbox/Folder", "Junk", "Junk folder", "On-premises/External")
// Fold every spelling and casing the filter admits back onto one published spelling before
// grouping. Without this, a tenant emitting two spellings of one location splits a mailbox
// across two rows and the ranking below reports it as less exposed than it is.
| extend DeliveryBucket = case(
    DeliveryLocation in~ ("Junk", "Junk folder"), "Junk",
    DeliveryLocation in~ ("Inbox/Folder"), "Inbox/Folder",
    DeliveryLocation in~ ("On-premises/External"), "On-premises/External",
    DeliveryLocation)
| summarize
    Messages = dcount(NetworkMessageId),
    Payloads = dcount(EmailClusterId),
    SenderDomains = dcount(SenderFromDomain),
    LastSeen = max(Timestamp)
    by RecipientEmailAddress, DeliveryBucket
| order by Messages desc
```

**Why the rollup groups on a normalised bucket rather than on `DeliveryLocation` itself.** The filter
above admits two spellings of one delivery location, and a grouping key is a string. `Junk` and
`Junk folder` are different keys, so a tenant emitting both splits one mailbox's Junk exposure across
two rows, each carrying part of the total, and `order by Messages desc` then ranks that mailbox below
one that is less exposed. **Lower-casing the column would not reach this**, because the two spellings
differ by a token rather than by case, and it is the token that does the damage. The `case()`
expression maps every spelling and casing the filter admits back onto the spelling Microsoft Learn
publishes, so one location is one row per recipient however your platform spells it. **Widening the
filter means widening the bucket in the same edit**, or the split returns for whatever you added. The
false-positive guidance below still says to rank on `Inbox/Folder` first, and that is now the bucket
name rather than the raw column value.

**What these queries return stays in your environment, and they do not return the same things.** The
rule query projects sender and recipient identifiers, the sending domain and the subject line, which
is the most directly identifying output this pack produces. **The exposure rollup groups by the
recipient and the normalised delivery location and ranks named mailboxes**, and it reduces the sending domain to
a count and carries no subject line and no sender identifier. **It does carry a last-seen timestamp,
and that is estate data as much as the addresses are**, which is the rule MSD-008's verification
step 2 states for first-seen and last-seen. What travels outward is a column name or a value name,
never a result row, a row count, or anything inside one. **That default holds where a column carries
a closed set the platform defines, and `Subject` does not**: it is free text, so the value is itself
estate data and the default does not reach it, which is why the checklist carries
`EmailEvents.Subject` on its free-text list and why a subject line does not travel even as an
example. **Apply that reasoning to any other free-text column this query returns rather than
looking for it on a list**: the recipient and sending-domain identifiers named above carry a
description and no published value set, exactly as `Subject` does.
Checklist Group 0 states the same rule for every step in the pack, and this file repeats it
because a reader who deploys one detection may never open the checklist.

## Status evidence

**GA (no preview qualifier)**, on the same basis as MSD-001: the prompt-injection guide carries no
preview qualifier in the article body, which is the scope the search covered and the only scope it
speaks to, as read on 2026-08-15 and re-read on 2026-08-17. The `EmailEvents` table
reference carries the same standing prerelease disclaimer noted in MSD-001.

**The plan scope is Plan 2.** The same page's "Applies to" line, read 2026-08-16, is "Applies to:
Microsoft Defender for Office 365 Plan 2, Microsoft Defender XDR", and its opening section states,
re-read 2026-08-17: "Microsoft Defender for Office 365 Plan 2 detects prompt injection content in
inbound email before that content reaches a user or an AI assistant." **The string "Plan 1" appears
nowhere on the page** on either read. This detection narrows MSD-001 and inherits that boundary:
without the Plan 2 signal there is nothing here for it to narrow.

The `DeliveryAction` and `DeliveryLocation` value lists above are quoted from Learn and are not
inferred.

## What this detection cannot see

- Everything MSD-001 cannot see. Read that section first; this detection inherits all of it.
- **Whether the message was later removed.** Post-delivery remediation is reflected in the
  `Latest*` columns, which this query projects rather than filters on, because Learn does not
  publish their value lists on the table reference page.
- **Whether the recipient or an assistant opened the item.** Delivery is not consumption.
- **Anything on the outbound or intra-org path.** The `EmailDirection == "Inbound"` filter is
  deliberate; remove it only if you intend to hunt internally-relayed injection, and expect
  forwarded copies of already-detected messages to dominate.

## False-positive guidance

- **Junk-folder accumulation is normal and not by itself an incident.** Most flagged messages
  should land in Junk or Quarantine. A steady Junk population is the control working. Rank on
  `Inbox/Folder` first.
- **Phishing-simulation traffic is commonly allowlisted into the inbox**, which puts it in exactly
  the bucket this detection ranks highest. Exclude your simulation platform's sender domain before
  measuring anything.
- **Mail-flow rules that bypass filtering** (for example a transport rule that skips filtering for
  a partner domain) will produce genuine `Inbox/Folder` hits that reflect a policy decision rather
  than a detection failure. Confirm the rule before escalating.
- **Distribution list expansion** inflates recipient counts. `RecipientEmailAddress` is documented
  as "Email address of the recipient, or email address of the recipient after distribution list
  expansion" - one message can produce many rows.

## Workspace verification before deployment

1. Complete the MSD-001 verification steps first, including the `getschema` diff **and its
   operator-selection step, which carries the multi-term `has` question this query shares.** **The
   2026-08-24 run answered that question in part**: a multi-term right-hand side matched, on both
   surfaces, with all three of the test's positive controls true. **What it did not do is exclude the
   reading on which the terms match while separated**, which is the residual MSD-001 records and this
   file inherits along with the rest of its steps.
   MSD-002 cannot be trusted if the `DetectionMethods` serialisation is unconfirmed, and the two files
   share a table and the same three-term literal.
   **That diff is against MSD-001's schema table, which carries neither `LatestDeliveryAction` nor
   `LatestDeliveryLocation`. Diff the same `getschema` output against the delta table above as
   well**, because both columns are declared only in this file, and MSD-001's step treats a column
   its own schema table omits as not a mismatch rather than as a finding. Without that second diff a
   renamed or withdrawn `Latest*` column reaches the query below unchecked.
2. Run `EmailEvents | where Timestamp > ago(30d) | summarize count() by DeliveryLocation,
   DeliveryAction, EmailDirection` and confirm which value strings your workspace emits. **Those
   counts stay in your environment**, as Group 0 of the checklist says for every step in the pack:
   what travels from this step is a value name, never a count of rows. **A value the filter does not
   match is a row the detection silently drops**, and under the case-sensitive
   `in` these queries shipped with until the 2026-08-25 correction a difference of case alone was
   enough to do it.
   **This step found exactly that defect.** On 2026-08-24 it showed a tenant emitting
   `DeliveryLocation` values the shipped `in ()` filter could not match, which is why the queries
   above now use `in~` and carry a second `Junk` spelling. Spelling still has to match even under
   `in~`, so read your own output rather than assuming the published list.
   **`EmailDirection` is on that list for the same reason and it is the one most easily missed**: the
   queries above filter it with `==`, which MSD-007 records as the case-sensitive comparison and
   cites the operator's own reference page for, so a workspace emitting `inbound` returns nothing
   from either query and the empty result reads as a clean estate. **The same run emitted an
   `EmailDirection` value Microsoft Learn does not publish, `Unknown`.** It does not affect the
   `== "Inbound"` filter, and it does tell you the published list is not the whole set a tenant can
   produce.
3. Compare `DeliveryLocation` against `LatestDeliveryLocation` on a sample of rows to understand
   how far apart the two run in your environment.

## Sources

- [EmailEvents table in the advanced hunting schema (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-emailevents-table) - last verified 2026-08-15, **re-read 2026-08-24** for the `DeliveryLocation`, `DeliveryAction` and `EmailDirection` rows after a tenant emitted values the published lists do not carry. That read confirmed all three published lists unchanged, and confirmed the page attaches no completeness qualifier to any of them. Page stamp on that read: `ms.date` 2026-08-03, `updated_at` 2026-08-07
- [`in` operator, case-sensitive (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/in-cs-operator) - read 2026-08-24, for the comparison table giving `in` as case-sensitive, which is the operator these queries shipped with until the 2026-08-25 correction
- [`in~` operator, case-insensitive (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/in-operator) - read 2026-08-24, re-read 2026-09-20, for the lead sentence naming it case-insensitive, for the performance note preferring `in` "when possible", and for the ASCII-text limit on case-insensitive operators
- [`case()` function (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/case-function) - read 2026-08-25, for the syntax `case(predicate_1, then_1, [predicate_2, then_2, ...] else)`, for the `else` argument being required, and for the published example applying it as a bucketing expression under `extend`. Page stamp on that read: `ms.date` 2024-08-11, `updated_at` 2025-05-25. **Its "Applies to" line names the same four products every Kusto reference page names**, so on this pack's own canonical reading it says nothing either way about Defender XDR advanced hunting; `docs/verification-methodology.md` section 6 records that reading and names the other functions this pack relies on under it
- [Prompt injection protection in Microsoft Defender for Office 365 (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-office-365/step-by-step-guides/prompt-injection-protection-defender-for-office-365) - **two dated reads, and both are load-bearing.** Last verified 2026-08-17, for the Plan 2 scope sentence and the absence of any preview qualifier. **The "Applies to" line quoted above is from the 2026-08-16 read**, which is the read that recorded it; the 2026-08-17 conversion did not surface that element, and a conversion is evidence of presence rather than of absence for a page element
- [Understand detection technology in the email entity page (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-office-365/step-by-step-guides/understand-detection-technology-in-email-entity) - last verified 2026-08-16
- [Create custom detection rules in Microsoft Defender XDR (Microsoft Learn)](https://learn.microsoft.com/en-us/defender-xdr/custom-detection-rules) - last verified 2026-08-16, for the recommended column projection and the email-action requirement
- MITRE ATLAS technique IDs read from the distributed `atlas-data` dataset, `version: 5.6.0` (release tag `v2026.07`) - verified 2026-08-15
