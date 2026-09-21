# Disclaimer

## Validate every query in your own environment

**Its author worked through the verification checklist once, on 2026-08-24, in one lab tenant** - a
Microsoft Sentinel workspace and Microsoft Defender XDR advanced hunting. **Not every query ran, and
the checklist was not completed**: two detections could not resolve their table there, one could not
resolve a column, and several steps are recorded as not completed.
[`CHANGELOG.md`](CHANGELOG.md) records the outcome for each. **That run generated no event, so no
detection in this pack has been observed firing**, and every query here remains a construction
validated against Microsoft Learn schema documentation rather than an observed result. Nothing the
run established generalises past a single lab tenant on a single date.

Every detection in this pack depends on at least one element Microsoft does not document - a value
format, a value set, an emission cadence, or the internal shape of a `dynamic` column. Three of them
ship a discovery query first, because a narrowing filter built on a guess would return nothing while
looking healthy. **None of the eight can distinguish a clean environment from a broken filter** until
you have generated a known event and seen it appear, and for five of them that gap is widest - a
judgement about how much else there is to fall back on **when the result comes back empty** rather
than a category derived from a rule.
[`docs/verification-methodology.md`](docs/verification-methodology.md) section 6 names the five and
is the canonical statement of that judgement; if this file and that one ever disagree, that one
governs.

[`checklists/workspace-verification-checklist.md`](checklists/workspace-verification-checklist.md)
is the release gate for this repository, not a suggestion.

## Statuses change, and preview surfaces change fastest

Every status label carries a last-verified date. Four of the eight detections carry a public preview
or contested label - MSD-003, MSD-004 and MSD-008 are Public Preview, and MSD-007 is Requires
further validation - where schema can change without a documentation update reaching you first.
MSD-005 needs the same watching without carrying either label: two of the seventeen alerts it
matches on are preview-tagged inside a plan that is GA. A label older than the current monthly
refresh window is stale, whatever it says.

Preview terms apply to the preview capabilities named here. Microsoft's preview terms are the
authority on what that means, not this repository.

## This is not Microsoft's content

This repository is independent work. It is not published, endorsed, reviewed, or supported by
Microsoft. Microsoft product names, table names, column names and quoted documentation text are
Microsoft's, cited with their source URLs and reproduced only as far as the technical claims require.

The MITRE ATLAS and OWASP mappings are the author's synthesis. MITRE and the OWASP GenAI Security
Project did not produce them and do not endorse them. Framework item IDs and titles are cited with
attribution to their publishers.

## A detection is not coverage

Each detection states what it cannot see, and those sections are the more important half of the
file. The **primary** queries of MSD-003, MSD-004 and MSD-006 detect nothing at all - they are
inventory and posture queries that rank candidates for human review. Each of the three also ships a
change-detection variant that alerts on a transition, which is a different job from detection.

Two limits recur and are worth repeating here:

- **There is no documented advanced-hunting surface for Microsoft 365 Copilot chat prompts** - a
  claim scoped to the seven Microsoft Learn pages listed in
  `docs/verification-methodology.md` section 3.1, read 2026-08-15 - the two that carry the weight
  re-read 2026-08-17 with the same result - and bounded there as strong evidence rather than proof of
  absence. Nothing in this repository provides such a surface, and no
  combination of these detections adds up to one.
- **An agent that authenticates with an API key bypasses Microsoft Entra ID entirely**, per Microsoft
  Learn, and on this pack's reading therefore produces no row in the sign-in telemetry MSD-006
  reads. A clean result there is not evidence of coverage.

## No warranty

The content is provided as is, without warranty of any kind, express or implied. The author accepts
no liability for any consequence of using it. You are responsible for what you deploy in your own
environment, including the query cost of anything you schedule.

## Corrections are welcome and rank above additions

If a citation here is stale, a column has been renamed, or a value set differs from what your
workspace emits, that is the most valuable thing you can report. Open an issue with the page URL and
the date you read it.

**Nothing from a real environment belongs in that issue.** Send the value name or the column name,
never the rows it matched, how many there were, or anything identifying inside them. **That default
does not reach a free-text column**, where the value is itself estate data and there is no value
name to send that is not also the content: from those, send the column name and the shape you found
and nothing else. The exclusion
list is in [`docs/scope-and-out-of-scope.md`](docs/scope-and-out-of-scope.md) and the issue templates
repeat it as a checklist. If explaining the finding needs environment detail, use the private route in
[`SECURITY.md`](SECURITY.md) rather than a public issue.
