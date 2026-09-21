---
name: New detection proposal
about: Propose a detection for this pack
title: "[detection] "
labels: detection, proposal
---

> **Before you fill this in: nothing from a real environment goes in a public issue.** No
> organisation or project names, non-public domains, IP addresses, hostnames, tenant / workspace /
> subscription GUIDs, user names, email addresses, screenshots, log lines, **exact** row counts, or
> query output - **including anything inside the query block below.** A rate rounded to the nearest
> order of magnitude is not an exact count, and **that is the level this banner allows**: the
> False-positive guidance section below asks for one in those terms. Value names are useful; the data
> in them is not. **Where a column carries free text the value is itself estate data**, so what
> travels from those is the column name and the shape you found, never the contents, and
> [`SECURITY.md`](../../SECURITY.md) states that rule in full. Anything containing the above will be
> deleted rather than edited. **Deletion is not
> retraction**, and this issue is public the moment it exists: deleting it reduces exposure rather
> than undoing it, which is why the check belongs before you post rather than after. If your proposal
> needs environment detail to explain, use the private route in
> [`SECURITY.md`](../../SECURITY.md) - the file is at the root of this repository if that link does
> not resolve from here.

A proposal arrives on the same terms as the detections already here. **A proposal that cannot
name the Microsoft Learn page for every column it uses will be closed with thanks and a pointer
back to this line.**

## What it detects

One paragraph. What an analyst learns from a hit that they did not know before.

## Deployment target

- [ ] Microsoft Defender XDR advanced hunting (deploys as a Defender custom detection rule)
- [ ] Microsoft Sentinel Log Analytics (deploys as a Sentinel analytics rule)

## Schema

| Column | Data type per Learn | Learn description (verbatim) |
|---|---|---|
| | | |

- Primary source URL(s):
- Date you read them:

**Every column above must appear on a page you read.** If a column is real but undocumented, say so
here rather than omitting it - undocumented elements are acceptable when they are labelled, and only
then.

## Query

**Strip your own filter values before pasting.** A domain, an `AppId`, an identity name, a resource
name or any other environment-specific literal is environment data even though it looks like code,
and a `let` binding carries it exactly as a `where` clause does. Replace each with `<redacted>`,
wherever in the query it sits. **That widens where to look rather than what to look for, and some
carriers are not literals at all**: a `//` comment left in the query, an identifier you named after
an internal system or team in a `let` or a `summarize` alias, and the saved query's own title if you
paste it with the query. Redact those on the same rule.

```kusto

```

If the query depends on any value string, state where that value is documented. If it is not
documented, the query must ship **discovery-first** - enumerating what a tenant actually emits
rather than filtering on a guess. MSD-006, MSD-007 and MSD-008 all use that shape; MSD-006 and
MSD-008 additionally leave the narrowing filter as a labelled placeholder for the operator to
complete.

## Status label

The five definitions below are reproduced verbatim from the canonical legend in
`docs/verification-methodology.md` section 2, so that a proposal and the pack use one vocabulary.
The first two boxes are **not** interchangeable, and picking between them is the single judgement
this section is asking you to make.

- [ ] **GA (stated)** - Microsoft Learn asserts general availability outright - a release-state row,
      or a release-note sentence naming the GA date.
- [ ] **GA (no preview qualifier)** - The documenting page carries no preview qualifier and no
      release-state sentence of any kind. **This is inference from absence and it is a weaker class
      of evidence.**
- [ ] **Public Preview** - Learn carries a `(Preview)` qualifier in the page or table title, or an
      explicit public-preview sentence.
- [ ] **Provisional** - The table or capability has a status, but a specific column, value set, or
      serialisation used by the query is undocumented. Scoped to the element, not the detection.
- [ ] **Requires further validation** - Microsoft's own sources conflict, or the status cannot be
      established from primary sources. Never guessed, never averaged.

Evidence for the label (quote it):

## Framework mapping

- MITRE ATLAS technique ID(s), read from the distributed `atlas-data` dataset rather than recalled -
  state the dataset version:
- OWASP LLM 2026 item (2026 numbering only):

## What it cannot see

At least two limits, each scoped to a named page or a stated mechanism. "It has some limitations" is
not one.

## False-positive guidance

What produces a benign hit, and how an analyst tells the difference. If you give a rate, round it to
the nearest order of magnitude - "a few a week", "tens a day". **That is the level the banner above
allows**, and it is the level the workspace verification report asks for as well: an exact count is
environment-revealing and a reader cannot act on the extra precision anyway. **If what you would
write here is an illustrative benign hit rather than a rate, the free-text rule in the banner reaches
it**: a subject line or a behaviour title is content, so describe its shape rather than pasting one.

## Workspace verification

Have you run this in a workspace?

- [ ] Verified, with a positive control observed
- [ ] Runs, but no positive control observed
- [ ] Not run

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

**That last box is deliberately stricter than the rule this pack applies to its own copy, and it
stays that way.** The pack also ships literals it wrote for a check, and one illustration it
invented and marks as invented, **and those are examples of the difference rather than the whole of
it**; [`docs/scope-and-out-of-scope.md`](../../docs/scope-and-out-of-scope.md) sets out every class
and is the list to work from. **Inbound content is held to the narrower line on purpose**, because a
value arriving from someone else's workspace cannot be checked against a published page the way this
pack's own values can. Two things follow for you. **Do not send back the `ActionType`
illustration**: it is this pack's own invention rather than a value to report, and the files that
display it are [`SECURITY.md`](../../SECURITY.md) and the workspace verification report template.
And if something you need to send is neither a documented Microsoft value string nor an obvious
placeholder, **leave the box unticked and say why in the body** rather than ticking it. **For a
value your own workspace emits that no page documents, that is the ordinary case and not an edge
one**: it is the last class in the list that link points at, it is the contribution this repository
most wants, and the box cannot truthfully be ticked for it. Leaving it unticked there is correct
rather
than an oversight, and the workspace verification report form carries a box written for exactly that
case. An unticked box with a reason is useful; a ticked one that is not true costs the form its
meaning.
