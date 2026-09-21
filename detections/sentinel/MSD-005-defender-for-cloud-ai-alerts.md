# MSD-005 - Defender for Cloud AI-workload alerts arriving in Sentinel

| Field | Value |
|---|---|
| **ID** | MSD-005 |
| **Deployment target** | Microsoft Sentinel (Log Analytics) |
| **Primary table** | `SecurityAlert` |
| **Status** | **GA (stated)** for the plan - Learn asserts general availability outright. Separately, **2 of 17 alerts carry a `(Preview)` tag on the alerts page**; the note under "The alert set" below says how that tag is read. The alert-identifier column is **Provisional**. |
| **Last verified** | 2026-08-15 |
| **MITRE ATLAS** | `AML.T0054` LLM Jailbreak · `AML.T0051.001` LLM Prompt Injection: Indirect · `AML.T0057` LLM Data Leakage · `AML.T0034` Cost Harvesting · `AML.T0053` AI Agent Tool Invocation · `AML.T0010` AI Supply Chain Compromise |
| **OWASP LLM 2026** | LLM01:2026 Prompt Injection · LLM02:2026 Sensitive Information Disclosure · LLM06:2026 Unbounded Consumption · LLM04:2026 Supply Chain · LLM05:2026 Data and Model Poisoning |

> **Six techniques because the alert set genuinely spans six, and each one names the alerts it
> rests on.** Alert descriptions quoted below were re-read on 2026-08-17.
>
> - **`AML.T0054` LLM Jailbreak** - the two Jailbreak alerts. They map here rather than to
>   `AML.T0051` LLM Prompt Injection, which is the ID a reader may expect: ATLAS carries the two as
>   **distinct** techniques, both are in the verified 5.6.0 parse, and MSD-008 uses `AML.T0054` for
>   the same concept, so mapping the jailbreak alerts to `AML.T0051` would put two different
>   techniques under one ID across two files in this pack.
> - **`AML.T0051.001` LLM Prompt Injection: Indirect** - `AI.Azure_ASCIISmuggling`, on Learn's own
>   wording: "These attacks are commonly attributed to indirect prompt injections, where the
>   malicious threat actor is passing hidden instructions to bypass the application and model
>   guardrails."
> - **`AML.T0057` LLM Data Leakage** - `AI.Azure_CredentialTheftAttempt`, which Learn describes as
>   notifying the SOC "when credentials are detected within GenAI model responses to a user prompt,
>   indicating a potential breach".
>   The leak is out of the model's own response, which is what makes it this technique rather than a
>   generic credential-access one.
> - **`AML.T0034` Cost Harvesting** - the two wallet-attack alerts,
>   `AI.Azure_DOWDuplicateRequests` and `AI.Azure_DOWVolumeAnomaly`. Learn: "Wallet attacks are a
>   family of attacks common for AI resources that consist of threat actors excessively engage with
>   an AI resource directly or through an application in hopes of causing the organization large
>   financial damages."
> - **`AML.T0053` AI Agent Tool Invocation** - `AI.Azure_AnomalousToolInvocation`.
> - **`AML.T0010` AI Supply Chain Compromise**, with LLM04 - `AI.AIModelScan_MalwareDetected`,
>   which is a poisoned model artefact rather than a runtime attack. **LLM05 Data and Model
>   Poisoning is the stretch in that last cell** and is recorded as one: the alert reports malicious
>   content found in an uploaded model, which is the supply-chain event LLM04 names directly, while
>   LLM05 describes poisoning of training or model data that this alert does not itself establish.
>
> **One identifier a reader would expect to be mapped is deliberately not.** The bullets above
> attach a technique to the alerts that carry one; this is the one that looks as though it should and
> does not, so it is called out rather than left to be noticed as an omission.
> `AI.Azure_LLMReconnaissance` is the closest
> thing here to `AML.T0056` Extract LLM System Prompt, and the cross-walk's not-mapped table states
> why it is not claimed: system-prompt extraction is one of three behaviours that alert bundles, and
> its own description says the activity *resembles* reconnaissance.

## Purpose

Bring Microsoft Defender for Cloud's AI-workload alerts into Sentinel using the alert names and
identifiers Microsoft publishes, rather than a substring guess. **Run it as a hunting query**, for
the duplicate-incident reason in the false-positive guidance below.

Of everything in this pack, this is the surface with the most detection content already built by
Microsoft. The work here is not writing the detection - it is knowing exactly what the alerts cover
and what they do not.

## The scope boundary - read this before deploying

Microsoft Defender for Cloud's AI threat protection covers **Azure-hosted AI workloads**. Per the
Learn availability page, the supported services are **Azure OpenAI supported models** and
**Azure AI Model Inference service supported models**.

**Neither "Microsoft 365 Copilot" nor "GitHub Copilot" appears anywhere on that page** - a check run
on the full page text on 2026-08-15. Note precisely what that is and is not: the page **enumerates
what it supports and does not name either Copilot**. It does not state an exclusion. Treat the
boundary as evidenced by positive enumeration, not by a Microsoft denial.

An alert from this detection is evidence about an Azure AI workload. It is not evidence of a
Microsoft 365 Copilot incident, and mapping it to one on a coverage map is the error this row
exists to prevent.

## Schema this depends on

Columns from the `SecurityAlert` table reference on Microsoft Learn, read 2026-08-15 (page stamp
2026-07-28, which is the rendered date; the source file's own `ms.date` reads 2026-07-27, and these
generated reference pages carry both). Table description, verbatim: "Alerts that been generated by
security products." **The grammar there is Microsoft's** and is reproduced as published.

| Column | Data type | Learn description |
|---|---|---|
| `TimeGenerated` | `datetime` | *(no description published)* |
| `AlertName` | `string` | *(no description published)* |
| `AlertType` | `string` | *(no description published)* |
| `AlertSeverity` | `string` | *(no description published)* |
| `DisplayName` | `string` | *(no description published)* |
| `Description` | `string` | *(no description published)* |
| `ProductName` | `string` | *(no description published)* |
| `ProductComponentName` | `string` | *(no description published)* |
| `ProviderName` | `string` | *(no description published)* |
| `VendorName` | `string` | *(no description published)* |
| `CompromisedEntity` | `string` | *(no description published)* |
| `Entities` | `string` | *(no description published)* |
| `ExtendedProperties` | `string` | *(no description published)* |
| `Tactics` | `string` | *(no description published)* |
| `Techniques` | `string` | *(no description published)* |
| `SubTechniques` | `string` | *(no description published)* |
| `Status` | `string` | *(no description published)* |
| `SystemAlertId` | `string` | *(no description published)* |
| `ResourceId` | `string` | *(no description published)* |
| `StartTime` | `datetime` | *(no description published)* |
| `EndTime` | `datetime` | *(no description published)* |

> **The empty description column is the finding, not an omission in this file.** Learn's
> `SecurityAlert` reference publishes these column **names and data types with empty description
> cells** - only `_BilledSize`, `_IsBillable` and `Type` carry descriptions. That is why this
> detection cannot tell you which column receives an alert identifier, and why the queries match on
> two columns instead of one. Recorded rather than smoothed over, because a reader who checks the
> page should find exactly what this file says they will find.

> **Provisional: which column carries the alert identifier is not documented.** Microsoft publishes
> the identifiers (for example `AI.Azure_Jailbreak.ContentFiltering.BlockedAttempt`) on the alerts
> reference page, but **neither the `SecurityAlert` table reference nor the alerts page states which
> `SecurityAlert` column receives them.** The queries below match on both `AlertName` and `AlertType`
> for that reason. Verification step 2 tells you how to find out which your workspace populates,
> after which you should narrow to it.

## The alert set

All 17 AI alert identifiers published on the "Alerts for AI services" page, read 2026-08-15 (page
stamp 2026-07-06). Two carry a `(Preview)` tag in their entry title; the other 15 do not.

**Re-read 2026-08-17: the identifier set, the names, the severities and the two `(Preview)` tags are
unchanged, and the page's own stamp still reads 2026-07-06.** Two things about that page are worth
knowing before you reconcile it against this table:

- **The `(Preview)` tag reads as a prefix on the entry heading rather than as part of the alert
  name.** The two headings read "(Preview) LLM Reconnaissance Attempt Detected" and "(Preview)
  Malicious content detected in uploaded AI model". The names in the table below are the names
  without that prefix, which is what the query matches on. **That reading is an inference from the
  page's own formatting and not something the page states**: the prefix sits in the heading and not in
  the identifier, and no sentence on the page says whether the alert's name carries it. Confirm
  against what your own workspace emits in verification step 2 before narrowing a filter, and treat
  the prefix as a candidate form if those two entries return nothing.
- **The page carries one further entry with no published `AI.` identifier**, a Kubernetes exposure
  alert. Per the page: "Some AI workload risk signals can also come from infrastructure protection
  plans, such as Defender for Containers, when AI applications run on Kubernetes." It is outside
  this table because this table is the identifier set. **It is outside the query for a narrower
  reason than that**: the page does publish a name for the entry, but the query's name list is the
  names that come with those 17 identifiers, and this entry's name is not among them, so neither
  the `AI.` prefix leg nor the name leg reaches it.

| Alert name (verbatim) | Identifier | Severity | Preview? |
|---|---|---|---|
| Detected credential theft attempts on an Azure AI model deployment | `AI.Azure_CredentialTheftAttempt` | Medium | |
| A Jailbreak attempt on an Azure AI model deployment was blocked by Azure AI Content Safety Prompt Shields | `AI.Azure_Jailbreak.ContentFiltering.BlockedAttempt` | Medium | |
| A Jailbreak attempt on an Azure AI model deployment was detected by Azure AI Content Safety Prompt Shields | `AI.Azure_Jailbreak.ContentFiltering.DetectedAttempt` | Medium | |
| Corrupted AI application\model\data directed a phishing attempt at a user | `AI.Azure_MaliciousUrl.ModelResponse` | High | |
| Phishing URL shared in an AI application | `AI.Azure_MaliciousUrl.UnknownSource` | High | |
| Phishing attempt detected in an AI application | `AI.Azure_MaliciousUrl.UserPrompt` | High | |
| Suspicious user agent detected | `AI.Azure_AccessFromSuspiciousUserAgent` | Medium | |
| ASCII Smuggling prompt injection detected | `AI.Azure_ASCIISmuggling` | High | |
| Access from a Tor IP | `AI.Azure_AccessFromAnonymizedIP` | High | |
| Access from suspicious IP | `AI.Azure_AccessFromSuspiciousIP` | High | |
| Suspected wallet attack - recurring requests | `AI.Azure_DOWDuplicateRequests` | Medium | |
| Suspected wallet attack - volume anomaly | `AI.Azure_DOWVolumeAnomaly` | Medium | |
| Access anomaly in AI resource | `AI.Azure_AccessAnomaly` | Medium | |
| Suspicious invocation of a high-risk 'Initial Access' operation by a service principal detected (AI resources) | `AI.Azure_AnomalousOperation.InitialAccess` | Medium | |
| Anomalous tool invocation | `AI.Azure_AnomalousToolInvocation` | Low | |
| LLM Reconnaissance Attempt Detected | `AI.Azure_LLMReconnaissance` | Low | **(Preview)** |
| Malicious content detected in uploaded AI model | `AI.AIModelScan_MalwareDetected` | High | **(Preview)** |

Severity values are Microsoft's, quoted from the same page. Do not re-score them in the rule; carry
them and let your own triage weighting sit above.

## Query

```kusto
// MSD-005 - Defender for Cloud AI-workload alerts in Sentinel.
// Deployment target: Microsoft Sentinel hunting query (Log Analytics). Deploying this as an
// analytics rule can create a second incident for every alert that already has one - see
// the duplicate-incident bullet in the false-positive guidance.
// Alert names and identifiers quoted from Microsoft Learn on 2026-08-15.
// Matches on both AlertName and AlertType because Learn does not document which
// SecurityAlert column carries the identifier - narrow after verification step 2.
// One name below doubles its backslashes. Kusto reads a backslash in a quoted string as an
// escape character, so the doubled pair is how the single backslash Microsoft publishes is
// written here. The alert-set table above reproduces that name with single backslashes, and both
// are correct for where they sit. Do not file that difference as a schema mismatch.
let AIAlertNames = dynamic([
    "Detected credential theft attempts on an Azure AI model deployment",
    "A Jailbreak attempt on an Azure AI model deployment was blocked by Azure AI Content Safety Prompt Shields",
    "A Jailbreak attempt on an Azure AI model deployment was detected by Azure AI Content Safety Prompt Shields",
    "Corrupted AI application\\model\\data directed a phishing attempt at a user",
    "Phishing URL shared in an AI application",
    "Phishing attempt detected in an AI application",
    "Suspicious user agent detected",
    "ASCII Smuggling prompt injection detected",
    "Access from a Tor IP",
    "Access from suspicious IP",
    "Suspected wallet attack - recurring requests",
    "Suspected wallet attack - volume anomaly",
    "Access anomaly in AI resource",
    "Suspicious invocation of a high-risk 'Initial Access' operation by a service principal detected (AI resources)",
    "Anomalous tool invocation",
    "LLM Reconnaissance Attempt Detected",
    "Malicious content detected in uploaded AI model"
]);
SecurityAlert
| where TimeGenerated > ago(1d)
| where AlertType startswith "AI." or AlertName in (AIAlertNames)
| project
    TimeGenerated,
    AlertName,
    AlertType,
    AlertSeverity,
    ProductName,
    ProviderName,
    VendorName,
    CompromisedEntity,
    ResourceId,
    Tactics,
    Techniques,
    Entities,
    ExtendedProperties,
    Status,
    SystemAlertId
| order by TimeGenerated desc
```

### Prompt-injection subset

```kusto
// The three alerts that are prompt-injection or jailbreak in substance.
// Matches on BOTH columns for the same reason the main query does - which column carries the
// identifier is undocumented, and filtering on AlertType alone would silently return nothing
// in a workspace that populates AlertName instead. Narrow to one column only after
// verification step 2 tells you which your workspace fills.
let InjectionAlertTypes = dynamic([
    "AI.Azure_Jailbreak.ContentFiltering.BlockedAttempt",
    "AI.Azure_Jailbreak.ContentFiltering.DetectedAttempt",
    "AI.Azure_ASCIISmuggling"]);
let InjectionAlertNames = dynamic([
    "A Jailbreak attempt on an Azure AI model deployment was blocked by Azure AI Content Safety Prompt Shields",
    "A Jailbreak attempt on an Azure AI model deployment was detected by Azure AI Content Safety Prompt Shields",
    "ASCII Smuggling prompt injection detected"]);
SecurityAlert
| where TimeGenerated > ago(7d)
| where AlertType in (InjectionAlertTypes) or AlertName in (InjectionAlertNames)
| summarize Alerts = count(), LastSeen = max(TimeGenerated)
    by AlertName, AlertType, AlertSeverity, CompromisedEntity, ResourceId
| order by Alerts desc
```

The distinction between `BlockedAttempt` and `DetectedAttempt` is the one worth wiring into
triage. Per Learn, `DetectedAttempt` fires when attempts "were detected … but weren't blocked due
to content filtering settings or due to low confidence." A run of `DetectedAttempt` without
`BlockedAttempt` is a content-filter configuration question, not only a threat question.

**Both queries here filter with case-sensitive `in`, and the choice is deliberate.** The 17 alert
names and the three injection names are long literals reproduced from Microsoft's published alert
list rather than typed by a reader, so an exact comparison is the one that fails loudly when the
reproduction drifts: a name that no longer matches returns nothing, and Group 5 of the checklist is
where you reconcile the list against the page. **`in~` would mask a casing change on Microsoft's
side and would not help with a transcription error**, which is the larger risk here. That is the
opposite of MSD-006's and MSD-008's case, where the values are ones you paste in by hand and those
files ship `in~` for exactly that reason. **MSD-003 and MSD-004 are a third case rather than either
of these two**: they ship `!in~` against an empty-forms list, where the operand is a shape rather
than a name. **`in` is case-sensitive and `in~` is its case-insensitive
counterpart**, per the `in` operator reference now cited in Sources below.

**What these queries return stays in your environment, and they do not return the same things.** The
main query projects the compromised entity, the resource identifier, the entity list and the extended
properties Microsoft attaches to an alert, all of which name things in your own subscriptions. **The
prompt-injection subset returns less**: it groups by alert name, alert type, severity, the compromised
entity and the resource identifier, and carries neither the entity list nor the extended properties.
What travels outward is a column name or a published alert name or identifier, never a result row,
a row count, or anything inside one. Checklist Group 0 states the same rule for every step in the
pack, and this file repeats it because a reader who deploys one detection may never open the
checklist.

## Status evidence

- **The plan is GA, and this is the pack's only GA (stated) label.** The Defender for Cloud AI
  threat protection availability page carries a **Release state** row reading
  "Generally available (GA)" - both strings confirmed on the live page 2026-08-15. The Defender for
  Cloud release-notes archive carries a "General Availability for Defender for AI Services"
  entry, dated May 1, 2025 on the page. **The heading and the date are separate elements there**, so
  they are cited separately rather than joined into one quoted string. Every other GA in this pack rests on the *absence* of a preview qualifier, which is
  a weaker class of evidence; see the status legend.
- **AI-agent (Foundry) protection within the same plan is Public Preview**, per the Defender for
  Cloud release notes, recorded as available in preview as part of the Defender for AI Services
  plan.
- **Two alerts carry `(Preview)` in their entry title** on the alerts page as read 2026-08-15:
  `AI.Azure_LLMReconnaissance` and `AI.AIModelScan_MalwareDetected`. The page also carries a blanket
  note: "For alerts that are in preview: The Azure Preview Supplemental Terms include additional
  legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released
  into general availability."
- **Learn caveats on the plan** (availability page, read 2026-08-15): text tokens only, so image
  and audio tokens are not scanned; the availability table records **Azure Government: No**,
  **21Vianet: No**, and **Connected AWS accounts: No**.

## What this detection cannot see

- **Microsoft 365 Copilot and GitHub Copilot.** See the scope boundary above for exactly how that
  is evidenced.
- **Non-text modalities.** Per Learn the plan scans text tokens only.
- **Sovereign and connected-cloud workloads**, per the availability table values quoted above.
- **Anything, if the connector is not streaming.** This detection reads alerts another product
  produced. It adds correlation, not detection. If Defender for Cloud alerts are not reaching the
  workspace, this rule is silently empty.
- **The one entry on the same Microsoft page that publishes no `AI.` identifier.** Neither leg of
  the main query reaches it, as the note under the schema table sets out: it has no `AI.` prefix for
  the first leg, and its published name is not in the name list the second leg matches on. It is a
  Kubernetes exposure alert, and **that an infrastructure protection plan is what raises it is this
  pack's reading of where the page places it** rather than something the page states of that entry.
  The page does say that AI workload risk signals can come from such plans when AI applications run
  on Kubernetes, quoted in full in the note above, so an environment running AI applications on
  Kubernetes can raise a signal this rule does not reach.
- **Alerts Microsoft adds after the verification date.** The 17-item list is a snapshot of a page
  that changes. The `startswith "AI."` clause is there so a new identifier still matches even though
  its name is not in the list. The prefix is `"AI."` rather than `"AI.Azure_"` because
  `AI.AIModelScan_MalwareDetected` does not carry the `Azure_` segment, and a narrower prefix would
  drop it.

## False-positive guidance

- **Four alert names are generic and can collide.** "Suspicious user agent detected", "Access from
  a Tor IP", "Access from suspicious IP" and "Access anomaly in AI resource" are phrasings this pack
  reads as generic across Defender plans. **That is a reading rather than something any page this
  file cites states**, and no page read for this file enumerates what another plan names its alerts.
  **Prefer the `AlertType` prefix match over the name list** once you have
  confirmed which column your workspace populates. Matching on name alone can pull in non-AI
  alerts.
- **`DetectedAttempt` is expected volume in a permissive content-filter configuration.** Baseline it
  before alerting per-event.
- **Wallet-attack alerts fire on volume** and follow legitimate load. A batch job, a load test, or a
  new production rollout will produce them. Correlate with change records before treating one as an
  attack.
- **`AI.Azure_AnomalousToolInvocation` is Low severity for a reason.** Learn describes it, verbatim:
  "The application attempted to invoke a tool in a manner that deviates from expected behavior."
  Any new feature release also deviates from expected behaviour.
- **Deploying this as an analytics rule can double your incidents.** If Microsoft Sentinel is
  already creating incidents automatically from Defender for Cloud alerts, every alert this
  rule matches already has an incident, and the rule creates a second. **Learn publishes a name for
  where that is switched on.** On a Microsoft security solution's data connector the page names a
  **Create incidents - Recommended** section carrying an **Enable** control, and from the Analytics
  page it gives **Create > Microsoft incident creation rule** as the from-scratch route; it describes
  what either produces as a Microsoft security analytics rule made from a rule template, with one
  template per Microsoft source solution. **The page prints the separator inside that section name as
  an en dash and this file reproduces it as a hyphen**, which is a transcription difference rather
  than a different name. Either run this as a **hunting query** rather than an analytics rule, or
  **deploy it as an analytics rule with incident creation turned off and triage from the alerts.**
  This detection adds correlation, not detection.

  > **Correction, 2026-09-19.** An earlier version of this bullet offered alert grouping against
  > `SystemAlertId` as the alternative. **That does not solve the problem stated.** The duplication
  > described is across two rules, and alert grouping is configured per analytics rule and groups the
  > alerts that rule produces, so it cannot merge its incident with one created by a Microsoft
  > incident creation rule. `SystemAlertId` is also unique per source alert, so grouping on it is the
  > setting that most reliably produces one incident per alert rather than suppressing one.
  > **The replacement is reasoned from the product's per-rule incident setting and has not been
  > exercised here**, and the duplicate-incident question stays recorded as not completed.
  > **A rule with incident creation off puts nothing in the incident queue**, so
  > triage means the alert queue or a `SecurityAlert` query.

  > **Correction, 2026-08-24.** An earlier version of this bullet said that no page this file cites
  > publishes a label for the control. The page cited for it publishes two, and both are named
  > above.
- **Duplicate ingestion.** If both a Defender for Cloud connector and a tenant-based Defender
  connector are enabled, confirm you are not counting the same alert twice.

## Workspace verification before deployment

1. Confirm the table exists and its columns match this file:
   `SecurityAlert | getschema | project ColumnName, ColumnType`. **Diff that output against the schema
   table above.** Record as a **schema mismatch** any column the schema table lists which the output
   does not carry, or which the output types differently. **A column the output carries and the schema
   table does not is not a mismatch**, because the schema table is a deliberate subset. Then confirm
   the table is populated and that Defender for Cloud alerts reach the workspace:
   `SecurityAlert | where TimeGenerated > ago(30d) | summarize count() by ProductName, ProviderName`.
2. Establish which column carries the identifier:
   `SecurityAlert | where TimeGenerated > ago(90d) | where AlertType startswith "AI." or AlertName
   has "AI model" | distinct AlertName, AlertType | take 50`. Narrow the shipped query to whichever
   column your workspace actually fills. **`has "AI model"` puts two terms on the right-hand side**,
   which is the construct checklist Group 2 exists to settle and which none of the pages this pack
   read settles. If that test returns a false `MultiTerm`, this leg matches nothing: **split it across
   `has_all`**, which is the option that survives either answer to the question this step is asking.
   **Dropping the leg and relying on the `AlertType` prefix alone assumes that answer**: in a
   workspace that fills `AlertName` and leaves `AlertType` empty, which is one of the two states this
   step exists to distinguish, the reduced query returns zero rows, and **a zero from it is
   uninterpretable rather than clean**. **How far the `AlertName` leg reaches, derived from the
   table above**: of the seventeen alert names this file reproduces, **four carry the literal
   `AI model`**, so that leg alone finds those four and the `AlertType` prefix is what reaches the
   rest. A fifth name carries `model` inside a longer token and the two-term operand does not match
   it. **One difference from Group 2's own
   operands, worth recording alongside the result.** The string-operators page states that the term
   index carries "all terms that are three characters or more" and that where a term is shorter the
   query "will revert to scanning the values in the column". `AI` is two characters, where every Group 2 operand is
   three or more, so this right-hand side mixes an indexed term with one that is not. The page frames
   that as an index-versus-scan path rather than a change in what matches, so carrying Group 2's
   result across is probably sound; it is not something the page says.
   **What leaves this step is a shape, not the list it returns.** The query takes up to fifty
   distinct names, and in your workspace those can include analytics rules you named yourself. **A
   rule name your own workspace emits is your estate's naming rather than a value set Microsoft
   publishes**, so describe what you found and send no name that is not among the seventeen this
   file reproduces. The checklist and `docs/scope-and-out-of-scope.md` state the same rule; it is
   restated here because this is the step that returns them, and a rule a reader meets only in
   another file is a rule they may not meet at all.
3. Confirm the AI plan is enabled on the subscriptions that host your Azure AI resources. A rule
   that never fires because the plan is off looks identical to a clean environment.
4. Trigger one alert deliberately in a lab subscription and confirm it arrives, with its identifier
   intact. Without a positive control, step 3 is unverifiable.
5. Re-read the alerts page and reconcile the 17-item list. Two entries were preview-tagged at the
   verification date; check whether that changed.

## Sources

- [Alerts for AI services (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/defender-for-cloud/alerts-ai-workloads) - last verified 2026-08-17; the alert-set table, the per-alert descriptions quoted in the technique note, and the two `(Preview)` tags were all re-read on that date and are unchanged from the 2026-08-15 read
- [AI threat protection in Microsoft Defender for Cloud (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/defender-for-cloud/ai-threat-protection) - last verified 2026-08-15
- [Microsoft Defender for Cloud release notes](https://learn.microsoft.com/en-us/azure/defender-for-cloud/release-notes) - last verified 2026-08-24, for the AI-agent (Foundry) preview entry the status evidence rests on · [release-notes archive](https://learn.microsoft.com/en-us/azure/defender-for-cloud/release-notes-archive) - last verified 2026-08-23, for the "General Availability for Defender for AI Services" heading and the date beneath it, re-read 2026-08-24 and unchanged. **The two pages keep separate dates rather than collapsing to one**, because they were read on different days and the later read of the archive confirmed it rather than adding to it
- [Create incidents from alerts in Microsoft Sentinel (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/sentinel/create-incidents-from-alerts) - last verified 2026-08-24, for the two labels the duplicate-incident bullet in the false-positive guidance now names: the connector-side **Create incidents - Recommended** section with its **Enable** control, which the page names twice, and **Create > Microsoft incident creation rule** on the Analytics page. The page also states that incident creation from Microsoft security alerts is a Microsoft security analytics rule made from a rule template, with one such template per Microsoft source solution. **The page prints the separator inside the section name as an en dash and this file reproduces it as a hyphen**, which is a transcription difference rather than a different name
- [SecurityAlert table (Azure Monitor Logs reference, Microsoft Learn)](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/securityalert) - last verified 2026-08-15
- [`in` operator (Kusto Query Language reference, Microsoft Learn)](https://learn.microsoft.com/en-us/kusto/query/in-cs-operator) - last verified 2026-08-15, for the case sensitivity of `in` against `in~` and for the `let`-bound `dynamic([...])` right-hand side both queries in this file ship. **This is the case-sensitive operator's own page**, which is the one this file needs because it ships `in` rather than `in~`; the sibling files that ship `in~` and `!in~` cite those operators' own pages instead
- MITRE ATLAS technique IDs read from the distributed `atlas-data` dataset, `version: 5.6.0` (release tag `v2026.07`) - verified 2026-08-15
