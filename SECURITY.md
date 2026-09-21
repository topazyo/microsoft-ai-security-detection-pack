# Security and private reporting

This repository publishes detection logic. It holds no service, no credentials and no user data, so
"security issue" here usually means one of two things: **a detection that is wrong in a way that
could cause a bad security decision**, or **a report you cannot explain without environment detail
that should not be public.**

Both have a private route. Use it.

---

## Do not paste environment detail into a public issue

This is the rule that matters most, and it is the reason this file exists.

Public issues, pull requests, commit messages and branch names are **public the moment they exist**.
Third-party services index public repositories continuously and can do so before anyone notices, a
pull-request diff lives under refs that cannot be rewritten, and forks and clones keep everything.
**Deletion is not retraction.** A maintainer removing a comment reduces exposure; it does not undo
it.

So none of the following belongs in a public issue, comment, pull request, commit message or branch
name:

- query results, row counts, or any output from a real environment
- organisation, project or system names
- non-public domain names, hostnames or IP addresses
- tenant, workspace or subscription GUIDs
- user names or email addresses
- screenshots of any kind
- log lines or captured telemetry
- real incident details
- **anything under NDA, and anything from a Microsoft preview programme that is not publicly
  documented.** This one is easy to trip over here, because the schema-correction template asks
  exactly the question a private-preview participant can answer: what does your workspace actually
  emit. If the only place you have seen a column, a value or a behaviour is a private preview, that
  is not a correction this repository can accept - and posting it would breach your agreement rather
  than this repository's. Wait until Microsoft documents it publicly, then send the public page.

**Value *names* are useful and welcome. The data in them is not.** "My workspace emits an
`ActionType` called `AgentToolInvocationBlocked`" is exactly the contribution this pack needs.
"Here are the 40 rows it returned" is not, and will be removed. **`AgentToolInvocationBlocked` is an
invented illustration rather than a documented value**; it follows the spelling convention of real
Defender values because that is the shape to send, and the verification-report template marks the
same string the same way.

**That welcome holds where a column carries a closed set the platform defines. Where a column
carries free text, the value is itself estate data and this rule does not reach it**, because there
is no value name to send that is not also the content: a subject line, a behaviour title or the
contents of an event-data column is content whatever it is called. **`AgentsInfo.McpServers` is
barred outright**, because Microsoft describes it as holding "server URLs and credential
configuration". The workspace verification checklist and
[`docs/scope-and-out-of-scope.md`](docs/scope-and-out-of-scope.md) state the rule in full and name
free-text columns this pack touches. **Neither naming is a closed list, and neither is a substitute
for the detection file you are working from**: apply the rule to any free-text column not named
there on the same reasoning rather than by looking for it in a list, and read the never-paste list in
the detection file itself, which bars columns the two shared surfaces do not repeat.

**Any issue or comment containing the above will be deleted rather than edited.** That is a
commitment, not a warning: editing leaves the content in the edit history.

---

## The private route

If explaining your finding requires environment detail, do not open a public issue.

The private route is
[GitHub private vulnerability reporting](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability)
on this repository, and the **Report a security finding privately** link on the new-issue page goes
straight to it.

**That link is written into this repository's issue configuration, so it appears whether or not the
feature behind it is switched on.** Its presence is not confirmation that the route works. The
state to watch for is a link that opens an error page, or a page saying private reporting is not
enabled for this repository.

**If that happens, stop there rather than falling back to a public issue with the detail in it.**
Blank issues are disabled here on purpose, so there is no path for a bare "please contact me".
Open a **Schema correction** issue carrying the detection ID and one line saying a private channel
is needed, and **leave every other field in it blank** - no page quotes, no workspace type, no
dates, no observed values, nothing about what you saw. That issue is a flag, not a report.

**The reply to it will be public**, because GitHub has no private reply path out of a public issue.
It will say only that private reporting has been re-enabled, or name another way to reach the
maintainer. The detail moves once you are on a private channel, never in the public thread.

**One thing the private route does not accept either.** If the only place you have seen a column, a
value or a behaviour is a Microsoft private preview, or anything else under NDA, it does not belong
here privately any more than publicly - sending it privately would still breach your own agreement
rather than this repository's. That exclusion is absolute and is the one item on the list above that
the private route does not relax.

Please include, as far as you can without exposing your environment:

- which detection (MSD-0XX)
- what is wrong, and what you expected instead
- whether it is a schema difference, a wrong status label, or a query that behaves differently from
  what its file says
- redact identifiers before sending, even privately

### What happens to what you send

Environment detail sent privately is used to correct the pack and for nothing else. **No part of it
is republished.** A correction that ships carries the column name, the value name, or the corrected
citation - never the data you sent to establish it, and never the name of the environment it came
from. You are not credited unless you ask to be, because a credit line naming an employer is itself
a disclosure.

**Who reads it, and for how long.** This repository has one maintainer, and a private report is read
by that person and by nobody else; it is not forwarded, and no third party is given access to it.
GitHub holds the advisory itself under its own terms, which are not this repository's to set. The
report is kept while the correction it drives is being made, and closed afterwards. **If you want it
deleted once the correction ships, say so and it will be** - and you do not have to give a reason.

**What that promise reaches, and what it does not, because the referent is the object named two
sentences above as not this repository's to set.** It reaches what the maintainer controls: your
request is acted on with whatever controls the platform provides, and no separate copy of your report
is kept anywhere else. **It does not reach what GitHub retains on its own systems afterwards**, which
falls under those same terms, and **no page this file cites settles that either way** - this file
does not claim it is retained and does not claim it is not. **So weigh what you send on the
assumption that sending it is not reversible**, and send the least environment detail that
establishes the defect.

---

## What counts as a security-relevant defect here

Ranked by how much damage it does if left alone.

1. **A detection that silently returns nothing when it should return rows.** The worst class,
   because a zero-row result reads as a clean environment. MSD-004 carries a worked example of it:
   the note explaining why the query does not call `isempty()` on a `dynamic` column, because the
   `isempty()` reference publishes `isempty(parsejson("[]"))` as false, so a bare `isempty()` would
   have matched nothing and made an unguarded estate read as clean.
2. **A wrong status label** that would lead someone to rely on a preview or contested surface as
   though it were generally available.
3. **A wrong or stale schema citation.** A query built on a column that no longer exists fails
   loudly, which is survivable. One built on a column that exists but means something else does not.
4. **A "what this detection cannot see" section that understates a blind spot.** Those sections are
   the more important half of every detection file; an incomplete one produces false confidence.

**A schema mismatch you found in your own workspace is the most valuable report this repository can
receive**, and it does not need the private route as long as you send the column name and not the
data. Use the schema-correction issue template.

---

## What this repository does not do

- It does not accept vulnerability reports about Microsoft products. Report those to Microsoft.
- It does not run any service, so it has no infrastructure to disclose against.
- It does not ship executable code in v0.1. The KQL is documentation; it is not run by this
  repository against anything.

---

## Response expectations

Corrections that affect whether a published detection is safe to rely on are handled first. This is
an independently maintained repository rather than a vendor product, so there is no service-level
commitment behind that ordering. Every status and citation change is recorded in
[`CHANGELOG.md`](CHANGELOG.md) with its date.
