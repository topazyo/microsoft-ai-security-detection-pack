# Pull request

> **Before you fill this in: nothing from a real environment goes into this repository**, and that
> includes your commit messages and your branch name, not only the files. Public pull-request diffs
> live under refs that cannot be rewritten. The confidentiality section at the bottom is the
> attestation; this line is the reminder that comes before you write anything.

## What this changes

## Type

- [ ] Schema or citation correction
- [ ] New detection
- [ ] Status label change
- [ ] Framework mapping change
- [ ] Documentation

---

## The no-invented-schema check

**This is the check that decides whether the change can merge.** Every table, column, value string
and status label **that a detection file or a shipped query relies on**, as opposed to one it merely
reports, must be either quoted from a Microsoft
Learn page read on a stated date, or marked Provisional or Requires further validation, with the
reason stated and a workspace step that resolves it, or labelled at the point of use as observed in a
lab tenant on a stated date rather than as documented. **The third of those is available only to a
value a shipped query acts on**, and section 1 names its members rather than counting them. There is
no fourth category for anything a shipped query acts on.
`docs/verification-methodology.md` section 1 is the canonical statement of the rule and names one of
the classes that sit outside it, which
[`docs/scope-and-out-of-scope.md`](../docs/scope-and-out-of-scope.md) enumerates in full; if this
template and that section ever disagree, that section governs.

- [ ] Every **table** touched by this change appears on a Microsoft Learn page I read, and the URL is
      in the detection file.
- [ ] Every **column** touched by this change appears on that page, with its documented data type
      recorded.
- [ ] Every **value string** the query filters on **or groups on** is either quoted from Learn, is a
      labelled placeholder the operator fills from their own workspace, or is labelled at the point
      of use as observed in a lab tenant on a stated date.
- [ ] A lab-observed value **widens a filter and never narrows one**, and **where the same column is
      also grouped on, the grouping key is normalised** so that admitting a second spelling of one
      value cannot split it into two rows and understate whatever the grouping ranks.
- [ ] I did **not** infer a column name from a product name, a portal label, a blog post, or another
      detection pack.
- [ ] Any element Microsoft does not document is marked **Provisional** or **Requires further
      validation**, with the reason stated in the file.

## Citation discipline

- [ ] Every claim carries a primary-source URL.
- [ ] Every citation carries the date the page was read.
- [ ] No status is sourced from a blog, launch announcement, or community post. Tech Community is
      official-adjacent context and can never set a status.
- [ ] Every **negative claim** ("Microsoft does not document X") names the pages it is scoped to, and
      states how far the claim reaches.

## Status label

- [ ] The label follows the canonical legend in `docs/verification-methodology.md` section 2, which
      the README reproduces. Where the two ever differ, the methodology governs.
- [ ] The evidence for the label is quoted in the file, not summarised.
- [ ] Nothing moved out of **Requires further validation** except on a specific, dated, cited
      finding. A label does not change because a source stopped being checked.

## Framework mapping

- [ ] MITRE ATLAS IDs were read from the distributed `atlas-data` dataset, not recalled, and the
      dataset version is recorded.
- [ ] OWASP items use **2026** numbering. No 2025-era `LLM0x` ID was carried over unchanged.
- [ ] The mapping is labelled as this repository's synthesis, not as an official mapping.

## Detection file shape

For a new or changed detection, confirm every required field is present:

- [ ] ID, title, deployment target, table, status label, last-verified date
- [ ] Schema table with documented data types and verbatim Learn descriptions
- [ ] Query, with any undocumented dependency called out in a comment
- [ ] Status evidence, quoted
- [ ] **What this detection cannot see** - at least two limits, **each scoped to a named page or a
      stated mechanism**, which is the wording the new-detection-proposal form uses for the same
      requirement. A limit derived from reasoning about a column's documented semantics is a stated
      mechanism and counts; do not drop one to satisfy this box.
- [ ] False-positive guidance
- [ ] Workspace verification steps
- [ ] Sources, with dates

## Confidentiality

- [ ] No organisation names, or project and system names used privately
- [ ] No non-public domain names or URLs
- [ ] No IP addresses or hostnames
- [ ] No tenant, workspace, or subscription GUIDs
- [ ] No user names or email addresses
- [ ] No captured logs, telemetry, query results, or screenshots
- [ ] No real incident details, vendor evaluation findings, licence counts, or commercial details
- [ ] Nothing under NDA, and nothing from a Microsoft preview programme that is not publicly
      documented
- [ ] Every example value is a documented Microsoft value string or an obvious placeholder
- [ ] **My commit messages carry none of the above.** A commit message is a published surface that
      survives a history rewrite, and correcting one that is already pushed needs a force-push.
- [ ] **My branch name carries no organisation, project, or environment name.** A branch name is
      public the moment the pull request opens, and renaming does not retract it - the diff lives
      under refs nobody can rewrite.

**Value names are useful and the data in them is not, and one class of column collapses that
distinction.** Where a column carries a closed set the platform defines, the value name is safe to
send. **Where a column carries free text, the value is itself estate data and that rule does not
reach it**, because there is no value name to send that is not also the content: a subject line, a
behaviour title or the contents of an event-data column is content whatever it is called. **For
those, what travels is the column name and the shape you found, never the contents.**
[`SECURITY.md`](../SECURITY.md) states that rule, and the workspace verification checklist and
[`docs/scope-and-out-of-scope.md`](../docs/scope-and-out-of-scope.md) are where free-text columns
this pack touches are named. **Neither naming is a closed list, and neither is a substitute
for the detection file you are working from**: apply the rule to any free-text column not named there
on the same reasoning, and read the never-paste list in the detection file itself, which bars columns
the shared surfaces do not repeat. **`AgentsInfo.McpServers` is barred outright**, because Microsoft
describes it as holding "server URLs and credential configuration".

**Strip your own filter values before pushing.** A domain, an `AppId`, an identity name, a resource
name or any other environment-specific literal is environment data even though it looks like code,
and a `let` binding carries it exactly as a `where` clause does. Replace each with `<redacted>`,
wherever in the query it sits. **"Wherever in the query it sits" widens where to look rather than
what to look for, and some carriers are not literals at all**: a `//` comment left in the query, an
identifier you named after an internal system or team in a `let` or a `summarize` alias, and the
saved query's own title if you paste it with the query. Redact those on the same rule. **This matters
more here than on an issue, not less**: a pull request carries whole query files, and the diff lives
under refs nobody can rewrite.

**The example-value box is deliberately stricter than the rule this pack applies to its own copy,
and it stays that way.** The pack also ships literals it wrote for a check, and one illustration it
invented and marks as invented, **and those are examples of the difference rather than the whole of
it**; [`docs/scope-and-out-of-scope.md`](../docs/scope-and-out-of-scope.md) sets out every class and
is the list to work from. **Inbound content is held to the narrower line on purpose**, because a
value arriving from someone else's workspace cannot be checked against a published page the way this
pack's own values can. Two things follow for you. **Do not send back the `ActionType`
illustration**: it is this pack's own invention rather than a value to report, and the files that
display it are [`SECURITY.md`](../SECURITY.md) and the workspace verification report template. And
if something you need to send is neither a documented Microsoft value string nor an obvious
placeholder, **leave the box unticked and say why in the body** rather than ticking it. **For a
value your own workspace emits that no page documents, that is the ordinary case and not an edge
one**: it is the last class in the list that link points at, it is the contribution this repository
most wants, and the box cannot truthfully be ticked for it. Leaving it unticked there is correct
rather than an oversight, and the workspace verification report form carries a box written for
exactly that case. An unticked box with a reason is useful; a ticked one that is not true costs the
form its meaning.

## Workspace verification

- [ ] I ran the affected queries in a workspace - **state which queries**, and whether a positive
      control was observed. Do not name or describe the workspace; whether it was a lab is the only
      detail asked for, and even that is optional
- [ ] I did not run them, and the change is documentation or citation only
