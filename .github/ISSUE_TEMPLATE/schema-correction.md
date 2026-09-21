---
name: Schema correction
about: A table, column, value, or citation in this pack does not match Microsoft's documentation or your workspace
title: "[schema] MSD-0XX - "
labels: correction, schema
---

> **Before you fill this in: nothing from a real environment goes in a public issue.** No
> organisation or project names, non-public domains, IP addresses, hostnames, tenant / workspace /
> subscription GUIDs, user names, email addresses, screenshots, log lines, row counts, or query
> output. **Value names are useful; the data in them is not.** Anything containing the above will be
> deleted rather than edited. **Deletion is not retraction**, and this issue is public the moment it
> exists: deleting it reduces exposure rather than undoing it, which is why the check belongs before
> you post rather than after. If explaining your finding needs environment detail, use the private
> route in [`SECURITY.md`](../../SECURITY.md) instead - the file is at the root of this repository if
> that link does not resolve from here.
>
> **If you are here because the private route is not working, this form has a flag-only mode.** Give
> the detection ID and one line saying a private channel is needed, and leave **every** other field
> blank - no page quotes, no workspace type, no date, no observed values, no query, and nothing about
> what you saw. **That includes the confidentiality checklist at the end of this form, and leaving it
> unticked is correct here rather than an oversight**: a flag carrying only a detection ID and one
> line has nothing to attest about, and this is the one public issue this repository accepts with
> the confidentiality attestation left blank in its entirety. That is a flag rather than a report.
> The reply to it will be public, because GitHub has no private reply path out of a public issue, so
> it will carry a route and nothing else. `SECURITY.md` describes that path in the same terms.

**This is the most valuable issue type in this repository.** A stale or wrong schema citation
defeats the pack's entire premise, and a mismatch found in a real workspace is worth more than a new
detection.

**One correction this repository cannot accept.** If the only place you have seen a column, a value
or a behaviour is a Microsoft private preview or anything else under NDA, do not report it here or
privately. This form asks precisely the question such a programme answers, so the boundary is easy
to cross without meaning to. Wait until Microsoft documents it on a public page, then send that page.

## Which detection

- Detection ID (MSD-0XX):
- Table:
- Column or value affected:

## What this pack currently says

Quote the line from the detection file.

## What is actually the case

Choose one and complete it.

- [ ] **Microsoft Learn now says something different.**

  - Page URL:
  - Date you read it:
  - Quote the current wording:

- [ ] **My workspace emits something different from what Learn documents.**

  - Workspace type *(optional)*: lab / production
  - Date observed:
  - What you observed, as the value string or the column name and not the rows it matched - **read
    the free-text rule below before you fill this in, because for some columns the value is itself
    the content**:
  - Query you ran, with every domain, `AppId`, identity or resource name, and every other
    environment-specific literal replaced by `<redacted>`, wherever in the query it sits:

  **Send the value string or the column name, not the rows it matched.** A value your workspace
  emits describes the schema and is the whole point of this form; the rows behind it, how many there
  were, and anything identifying inside them describe your estate and are not wanted here.

  > **That holds where a column carries a closed set the platform defines. Where a column carries
  > free text, the value is itself estate data and the rule above does not reach it**, because there
  > is no value name to send that is not also the content: a subject line, a behaviour title or the
  > contents of an event-data column is content whatever it is called. **For those, what travels is
  > the column name and the shape you found, never the contents.**
  > [`SECURITY.md`](../../SECURITY.md) and the workspace verification checklist state the rule in
  > full and name the free-text columns this pack touches. **Neither naming is a closed list, and
  > neither is a substitute for the detection file you are working from**, so apply the rule to any
  > free-text column not named there on the same reasoning rather than by looking for it in a list,
  > **and read the never-paste list in the detection file itself, which bars columns the two shared
  > surfaces do not repeat.** MSD-003 and MSD-004 each carry such a list, and between them they bar
  > columns neither shared surface names.
  >
  > **`AgentsInfo.McpServers` is barred outright**, because Microsoft describes it as holding "server
  > URLs and credential configuration". **From that column, send the field names inside it and the
  > empty forms your platforms emit, and nothing else - never a sample of its contents, in any field
  > of this form.**

  **Strip your own filter values before pasting.** A domain, an `AppId`, an identity name, a resource
  name or any other environment-specific literal is environment data even though it looks like code,
  and a `let` binding carries it exactly as a `where` clause does. Replace each with `<redacted>`,
  wherever in the query it sits. **Several of this pack's own queries put their value sets in a
  `let`**, so a completed pack query is one of the cases this covers; **others write the set inline
  in the `where` clause instead**, which carries exactly the same risk and is easier to walk past.
  A real mismatch-proving query usually carries several. **"Wherever in the query it sits" widens
  where to look rather than what to look for, and some carriers are not literals at all**: a `//`
  comment left in the query, an identifier you named after an internal system or team in a `let` or a
  `summarize` alias, and the saved query's own title if you paste it with the query. Redact those on
  the same rule.

  **The workspace-type field is optional, and here is why you might leave it blank.** A report from
  an identifiable account saying a named detection is wrong **in production** is a public statement
  about your organisation's monitoring. This repository does not identify the environments its own
  author ran against, and it will not ask you to identify yours. **It does publish, of its own runs,
  which detections could not resolve their table or a column, what shape a function returned, which
  serialisation a column emitted, and a value a tenant emitted where no page this pack cites
  publishes one**, so take the reassurance as being about identification rather than about outcomes.
  [`docs/scope-and-out-of-scope.md`](../../docs/scope-and-out-of-scope.md) sets out that class, and
  this sentence restates it. Report from a personal account, leave it blank, or use the private
  route in [`SECURITY.md`](../../SECURITY.md) - a correction with it blank is still the most
  valuable issue this repository receives.

- [ ] **The citation URL is dead or redirects.**

  - URL:
  - What it returns *(the status code, or what the page says; if something on your own network
    redirected you, describe the page you landed on rather than pasting its address - "a sign-in
    portal" is enough - without naming the product that served it or your organisation)*:

## Impact

- [ ] The query returns nothing when it should return rows
- [ ] The query returns rows it should not
- [ ] The status label is wrong
- [ ] Documentation only, the query still works

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
placeholder, **leave the box unticked and say why in the body** rather than ticking it. **For this
form's option 2, a value your workspace emits that differs from what Learn documents, that is what
happens every time**: such a value is by construction neither a documented Microsoft value string
nor a placeholder, it is the last class in the list that link points at, and it arrives on the form
this repository calls its most valuable issue type. Leaving the box unticked there is correct rather
than
an oversight, and the workspace verification report form carries a box written for exactly that
case. An unticked box with a reason is useful; a ticked one that is not true costs the form its
meaning.
