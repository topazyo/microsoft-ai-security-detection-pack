---
name: Workspace verification report
about: Report the outcome of running the verification checklist against a real workspace
title: "[verification] MSD-0XX - "
labels: verification
---

> **Before you fill this in: no query results, no exact counts, no screenshots, and no environment
> identifiers.** Any issue containing them will be **deleted, not edited** - editing leaves the
> content in the edit history. **Deletion is not retraction**, and this issue is public the moment it
> exists: deleting it reduces exposure rather than undoing it, which is why the check belongs before
> you post rather than after.
>
> **The line runs between a value string and the data it returned.** A value string your workspace
> emits - a value of `ActionType`, `ConditionalAccessStatus` or `RecordType` - is the contribution
> this section exists for, and it describes the schema rather than your estate.
> The rows it matched, how many there were, and who or what appears in them are not. If your report
> needs environment detail to make sense, use the private route in
> [`SECURITY.md`](../../SECURITY.md) - the file is at the root of this repository if that link does
> not resolve from here.
>
> **That holds where a column carries a closed set the platform defines. Where a column carries free
> text, the value is itself estate data and the rule above does not reach it**, because there is no
> value name to send that is not also the content: a subject line, a behaviour title or the contents
> of an event-data column is content whatever it is called. **For those, what travels is the column
> name and the shape you found, never the contents** - in any field of this form. The workspace
> verification checklist and [`docs/scope-and-out-of-scope.md`](../../docs/scope-and-out-of-scope.md)
> state the rule in full and name free-text columns this pack touches. **Neither naming is a closed
> list, and neither is a substitute for the detection file you are working from**: apply the rule to
> any free-text column not named there on the same reasoning rather than by looking for it in a list,
> and read the never-paste list in the detection file itself, which bars columns the two shared
> surfaces do not repeat.

This repository ships queries its author has run **three** times, on 2026-08-24, 2026-08-26 and
2026-09-11. **Not every query ran**, and **no detection here has been observed firing**. Every
verification report closes part of that gap, and a **negative** result is as useful as a positive
one. **Reports from tenants other than the
author's are what this pack is missing**, because breadth is worth more here than another reading of
ground already covered.

## Detection

- Detection ID (MSD-0XX):
- Table:
- Date run:

**The next two fields are optional, and here is why you might leave them blank.** A report from an
identifiable account saying a named detection is **Blocked in production** is a public statement
that your organisation does not have that coverage. This repository does not identify the
environments its own author ran against, and it will not ask you to identify yours. **It does
publish, of its own runs, which detections could not resolve their table or a column, what shape a
function returned, which serialisation a column emitted, and a value a tenant emitted where no page
this pack cites publishes one**, so take the reassurance as being about identification rather than
about outcomes. [`docs/scope-and-out-of-scope.md`](../../docs/scope-and-out-of-scope.md) sets out
that class, and this sentence restates it. Report from a personal account, omit these two fields, or
use the private route in [`SECURITY.md`](../../SECURITY.md) - a report with them blank is still
useful.

- Workspace type *(optional)*: lab / production
- Environment note *(optional, for example: GCC, sovereign cloud, commercial)*:

## Outcome

- [ ] **Verified** - query ran, schema matched, a positive control was observed
- [ ] **Runs, unconfirmed** - query ran without error, no positive control observed
- [ ] **Blocked** - table absent, connector absent, or plan not enabled
- [ ] **Schema mismatch** - a column the detection file records is missing or typed differently, or
      a value falls outside a documented value set that file reproduces **and that reproduction no
      longer matches the page it came from**. Two things are **not** a mismatch: a value from a
      column the pack records no value set for, and a value beyond a reproduction that still
      matches its page. Record either under **Undocumented elements you resolved** below

If **Schema mismatch**, please also open a Schema correction issue - that is the one that gets acted
on first.

## Undocumented elements you resolved

Several detections carry elements Microsoft does not publish. If you established one, record it -
this is the part of the pack that cannot be improved from documentation alone.

| Element | What you observed |
|---|---|
| e.g. `DetectionMethods` serialisation | |
| e.g. `ConditionalAccessStatus` values | |
| e.g. `BehaviorInfo.ActionType` values for AI-agent protection | |
| e.g. `CopilotActivity.RecordType` values | |
| e.g. `McpServers` field names - **field names and empty forms only, see below** | |

**Send the value strings, not what they matched.** A line of the shape
`ActionType = "AgentToolInvocationBlocked"` is exactly the contribution this table is for: it is a
value name your workspace emits, and it tells the next reader what to filter on. **That string is an
invented illustration and not a documented value** - it follows the spelling convention of real
Defender values precisely because that is the shape to send, and this pack marks it as invented
wherever it uses it. **So do not send this particular string back**: it is this pack's own invention
rather than anything your workspace emits, and a report carrying it tells the next reader nothing.
The rows your own value matched, how many there were, and any identifier inside them are not wanted,
and an issue carrying them will be deleted rather than edited.

> **One column is a narrower case than the rest, and it is the one this table most wants resolved.**
> Microsoft describes `AgentsInfo.McpServers` as holding "server URLs and credential configuration".
> **From that column, send the field names inside it and the empty forms your platforms emit, and
> nothing else - never a sample of its contents, in any field of this form.** A populated value
> there is environment data of the most sensitive kind this pack asks anyone to look at. The
> workspace verification checklist states the same rule at the step that produces it.

> **`AgentsInfo.Availability` is not that case and it is not the ordinary one either.** It is a
> value string this table wants, and the checklist attaches a condition to it at the step that
> produces it: look at what a value contains before you send it, and send the shape rather than the
> value where it carries anything of your own organisation's, because a platform that serialises the
> specific-groups case by naming the groups puts your own naming in the column. **The same condition
> reaches `CopilotActivity.AppHost`**, for the reason the checklist gives there. Neither is barred;
> both are looked at first.
>
> **`AppHost` carries a second rule that the per-value one does not cover, and the checklist states
> it in full.** A complete inventory of the applications hosting Copilot in your tenant states part
> of what your estate runs even where every member of it passes the test above, so **the inventory
> stays in your environment**. What may travel is an individual value that passes that test, rather
> than the list. **Sending the inventory a value at a time is sending the inventory.** If a
> maintainer needs more than an exemplar, the private route in
> [`SECURITY.md`](../../SECURITY.md) is where to ask.

## Positive control

Did you generate a known event and locate it?

- [ ] Yes - describe how, without environment-specific detail
- [ ] No - state why

Without a positive control, an empty result cannot be distinguished from a wrong filter. Saying so
plainly is more useful than a hedged pass.

## False positives observed

Cause, and rate to the nearest order of magnitude if you ran it long enough to have one - "a few a
week", "tens a day". That is the level the banner above allows and the level a reader can act on.
An exact count is both unnecessary and environment-revealing.

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
- [ ] Every value string I have given is one the schema emits, not data from a matched row, **and
      none of them comes from a free-text column** - for those I have sent the column name and the
      shape and not the contents
