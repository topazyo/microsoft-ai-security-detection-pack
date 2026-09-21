# Workspace verification checklist

**This pack cannot be released until this checklist has been completed in a real Microsoft Sentinel
and Microsoft Defender XDR workspace.**

**The author has worked through this checklist three times, on 2026-08-24, 2026-08-26 and
2026-09-11**, against a Microsoft Sentinel workspace and Microsoft Defender XDR advanced hunting.
**This pack does not identify any environment it ran against.**

**No run has completed every step below**, and the release condition above is therefore not met: the
steps asking for a generated event, the `AgentsInfo` steps, the `BehaviorEntities` join, the plan
checks and the `SecurityAlert` identifier question are all
recorded as not completed on at least one run.

**Queries were submitted on both surfaces, and some of them did not resolve there.** **Two schema
failures are on the record and they are not on the same footing**: one table did not resolve on any
run this pack records, while one column failed on the first run only and its three deployment
queries parsed and ran on a later one. Each detection file records what its own queries did.

**No detection here has yet been observed firing.**

**Every query here remains a schema-verified construction validated against Microsoft Learn rather
than an observed result**, and that phrase means what it says and no more. **Validation is against
documented schema, not against availability in any particular workspace.** A query can be correctly
built against a published table that does not resolve on the surface you are using, which is what
happened on the runs this pack records. **Why it did not resolve is a separate question**, and this
pack has not settled it.

Nothing below generalises past three runs.
Every detection in this pack carries at least one element
Microsoft does not document, and **none of the eight can distinguish a clean environment from a
broken filter without a lab-generated positive control.** For **five** of them - MSD-003, MSD-004,
MSD-005, MSD-007 and MSD-008 - that gap is widest, **a judgement about how much else a reader has to
fall back on when the result comes back empty rather than a category derived from a rule.**
`docs/verification-methodology.md` section 6 is the canonical statement of that judgement; if this
file and that one ever disagree, that one governs.

Work through this in order. Record the date, the workspace type (lab or production), and the
outcome of every step. **A step that cannot be completed is recorded as not completed** - never as
passed with a note.

**How the quoted notes below are placed, because the indentation carries meaning.** A note at the
**left margin** states a rule for the group it sits in or for the whole checklist, and applies
whether or not you ran the step above it. A note **indented to sit under a step** belongs to that
step alone; every one of those opens `Correction,` with a date and records that an earlier version
of the step said something this pack has since corrected. **The two are different things rather
than two house styles**, so read the indentation before deciding what a note governs.

---

## Group 0 - Before you start

- [ ] Use a **lab or synthetic tenant** wherever a step asks you to generate an event. Do not
      generate test attack traffic in production.
- [ ] Record which Defender and Sentinel plans the workspace holds. Several detections return
      nothing when a plan is absent, and that is indistinguishable from a clean environment.
- [ ] Confirm the environment is **not GCC** before testing MSD-008 - Microsoft Learn states
      `BehaviorInfo` is not available there.
- [ ] Note the date. Every result below is only valid as of that date.

> **What travels outward, and it applies to every group below rather than to any one of them.**
> Everything this checklist asks you to record is about your own environment: which plans you hold,
> which connectors are configured, which serialisations your platforms emit, what your `McpServers`
> values contain. **The default for all of it is that it stays with you.** What travels outward is a
> **value name** or a **column name**, never what it matched and never how much of it there was.
> **That default carries qualifications, and each is stated where it bites.** It does not reach a
> column whose values are themselves the content, and **the results section at the end of this file
> states that one in full**. On the column-name side it rests on the schema being Microsoft's, and a
> Log Analytics workspace can be configured to add columns that are not: **a column name your own
> output carries and the detection file's schema table does not is one to look at before you send
> it**, rather than one the default clears. Group 1 is the step that surfaces it, because Group 1 is
> the diff.
> This is stated here, before the first step, because a reader who works through the groups and
> files a report should have met it before they start rather than after.

---

## Group 1 - Schema comparison, then table availability

**Do the schema diff first.** This checklist's results table treats "Schema mismatch" as the most
valuable outcome it can produce. A renamed or missing column produces a semantic error that makes
the whole query fail, and
the "Runs, unconfirmed" outcome has no path to that result.

For each table, run `getschema` and **diff the output against the schema table in the detection
file**. Record as **Schema mismatch** any column the schema table lists which the output does not
carry, or which the output types differently. **A column the output carries and the schema table
does not is not a mismatch**: the schema tables in this pack are deliberate subsets of the published
tables rather than reproductions of them, so the output is expected to be longer. **That asymmetry is
about columns.** The results table at the end of this checklist keeps it, and separately gives the
same outcome label to the value differences the later groups look for, in the narrower set of cases
that table sets out.

**Project `ColumnType`, not `DataType`.** `getschema` returns four columns, `ColumnName`,
`ColumnOrdinal`, `DataType` and `ColumnType`, and the `getschema` operator reference publishes an
example output in which the last two differ on every row: `DataType` carries CLR type names and
`ColumnType` carries the Kusto scalar type names, `System.String` beside `string` and
`System.Object` beside `dynamic`. **The schema tables in this pack record Kusto type names.** A diff
taken against `DataType` therefore types every column differently from the table it is being diffed
against, and sends every row to the mismatch outcome. That reading of the reference is dated
2026-08-23. **The `getschema` form below did run in the 2026-08-24 run**, on both surfaces, and
it is what surfaced the `Title` mismatch that run recorded on `BehaviorInfo`. Running it is not a
check of the reading above, which stands on the reference page rather than on the run.

**Source for that reading**, recorded here beside the quotation because this file has no Sources
section: [`getschema` operator (Kusto Query Language reference, Microsoft
Learn)](https://learn.microsoft.com/en-us/kusto/query/getschema-operator), read 2026-08-23, for the
four column names and for the example output whose `DataType` and `ColumnType` differ on every
published row. It is a Kusto reference page, so `docs/verification-methodology.md` section 6 governs
what its "Applies to" line does and does not settle.

- [ ] `EmailEvents | getschema | project ColumnName, ColumnType` - diff against MSD-001 *(and MSD-002, which shares the table)*
- [ ] `AgentsInfo | getschema | project ColumnName, ColumnType` - diff against MSD-003 and MSD-004
- [ ] `BehaviorInfo | getschema | project ColumnName, ColumnType` - diff against MSD-008
- [ ] `SecurityAlert | getschema | project ColumnName, ColumnType` - diff against MSD-005
- [ ] `AADServicePrincipalSignInLogs | getschema | project ColumnName, ColumnType` - diff against MSD-006, **including the two columns whose serialisation it flags Provisional.** `getschema` settles their name and data type and cannot settle their serialisation; that is the `ConditionalAccessPolicies` and `LocationDetails` inspection step in Group 6 below
- [ ] `CopilotActivity | getschema | project ColumnName, ColumnType` - diff against MSD-007

Then confirm each table returns rows. An empty table is a finding, not a pass.

- [ ] `EmailEvents | take 10` - populated? *(MSD-001, MSD-002)*
- [ ] `AgentsInfo | take 10` - populated? *(MSD-003, MSD-004)*
- [ ] `BehaviorInfo | take 10` - populated? *(MSD-008)*
- [ ] `SecurityAlert | take 10` - populated? *(MSD-005)*
- [ ] `AADServicePrincipalSignInLogs | take 10` - populated? *(MSD-006)*
- [ ] `CopilotActivity | take 10` - populated? *(MSD-007)*

For every table that returned nothing, record **which** of the three explanations applies: the
service is not deployed, the connector or diagnostic setting is not configured, or the table is
deployed and genuinely empty. If you cannot tell, that is the finding.

### Group 1b - Four KQL behaviours the pack's own queries rest on

None of these needs any of the tables above, so these four are the cheapest thing to settle before
anything else. **The 2026-08-24 run answered all four** - checks 1 to 3 on both surfaces, and
check 4 in Defender XDR advanced hunting, which is where that question lives - and each check below
records what it returned there. Run them in your own tenant regardless: that answer came from one
tenant on one date, and these checks decide which query shapes are safe for **you** to deploy.
Each block below sits
at the left margin rather than inside its checkbox, so that what you copy is the query and not a row
of fence markers.

- [ ] **1. Does a bare `isempty()` see an empty JSON array?** Expect `false` and `"[]"`. This is why
      MSD-003 and MSD-004 test the string form instead of calling `isempty()` on a `dynamic` column.
      **The 2026-08-24 run returned exactly that, on both surfaces**: `isempty()` returned false,
      and `tostring()` over the same literal returned the two-character string form. **This check
      confirms documented behaviour rather than settling an unparsed construct**, which is why
      `docs/verification-methodology.md` section 6 does not carry it.

```kusto
print BareIsEmpty = isempty(dynamic([])), StringForm = tostring(dynamic([]))
```

- [ ] **2. Does the comment-only literal parse at all?** **The 2026-08-24 run settled this on
      both surfaces: the block parsed, `ArrayLength` returned `0` rather than null, and `Matches`
      and `MatchesCI` both returned false.** The one-element control passed first on each surface,
      so the reading is live. Run the
      probe block exactly as printed - the comment has to sit on its own line, and MSD-008 carries
      the identical pair. **Two detection files ship this construct rather than one:** MSD-008's step-2
      placeholder and MSD-006's step-2 `NoPolicyApplied` list are the same comment-only
      `dynamic([...])` form, so this answer governs both. **The comment inside MSD-006's step-2
      block tells you to fill the placeholder without naming either outcome**, which is why the
      file rather than the comment is what settles it here: if the
      literal does not parse, MSD-006's step 2 errors rather than returning nothing, and the two
      failures need different responses. **MSD-006's own prose names both outcomes**, in the
      paragraph directly under that query. Record the answer; it decides which MSD-008 query shape
      is safe to deploy.
      **Read the `ArrayLength` column as well as whether the block parses.** MSD-008's self-check
      wrapper guards its unconfigured branch with `array_length(...) == 0`, and `array_length()`
      returns null rather than zero for a value that is not an array. If `ArrayLength` comes back
      null, that branch cannot fire, and whether the wrapper returns anything at all then rests on
      its other branch, whose behaviour MSD-008 says that file does not establish. **It came back
      `0` in the run**, so that branch could fire there.
      **Read `MatchesCI` and not `Matches` for the question MSD-008 leaves open.** Both files this
      check governs ship `in~`, while `Matches` uses the case-sensitive `in` and therefore settles
      nothing about the operator those files actually use. **`Matches` is kept beside it as the
      case-sensitive control**, so the pair tells you whether the two operators differ here at all.
      MSD-008 names its residual as how `in~` behaves against a right-hand side that is not an array,
      and `MatchesCI` is the column that speaks to it.
      **Run the control block first, and it is printed first.** It is the same statement over a
      literal carrying one element rather than only a comment, so every part of the machinery is
      exercised independently of the question this check asks: the `let`, `array_length()`, and `in`
      and `in~` used as scalar expressions inside `print`. **That last one is a syntactic position no
      page this pack cites exemplifies**, both `in`-family pages documenting the `where` form only, so
      an error on the probe alone does not establish that the comment-only literal is what the engine
      rejected. **If the control errors too, it is not**: record this step as not completed with both
      error texts, and treat the deployment decisions Group 8 keys to this check as unmade rather than
      answered. Group 2 below states the same rule for its own operator test.
      **Expect the control block to return one row, with `ArrayLength` reading `1`.** The
      2026-08-24 run records this control passing on both surfaces before the probe beside it, so a
      block that errors rather than returning a row leaves the probe settling nothing.

```kusto
let Y = dynamic(["a"]);
print ArrayLength = array_length(Y), Matches = ("a" in (Y)), MatchesCI = ("a" in~ (Y))
```

```kusto
let X = dynamic([
    // placeholder
]);
print ArrayLength = array_length(X), Matches = ("a" in (X)), MatchesCI = ("a" in~ (X))
```

- [ ] **3. Does a `print` statement work as a leg of a `union`?** MSD-008's optional self-check
      wrapper depends on it. **The 2026-08-24 run exercised it on both surfaces and it returned
      the single expected row**, so the construct is available there. Expect one row, reading
      `unconfigured`. If it errors, the wrapper is not available to you and step 2 of MSD-008 ships
      without it. **This settles the construct and not the shape.** Both legs below project one
      identically-named, identically-typed column, while the wrapper's two legs do not share a
      schema at all. That second question is documented rather than open: the `union` reference
      gives `kind=outer` as the default and says the result carries every column occurring in any
      input, with undefined cells set to `null`. So what is unestablished here, and what this check
      is for, is whether the construct parses on this target. The `union` reference does show a
      `print` reached through a `let ... = view () { ... }` and used as a leg; **what it does not show
      is the inline form**, which is what both this check and the wrapper use.

```kusto
let Empty = dynamic([]);
union
  (print Leg = "configured" | where array_length(Empty) > 0),
  (print Leg = "unconfigured" | where array_length(Empty) == 0)
```

- [ ] **4. Does `hash_sha256()` run in Defender XDR advanced hunting?** MSD-003's change-detection
      variant is the only query in this pack meant for deployment that depends on a hashing function.
      **The 2026-08-24 run ran it there and it returned a 64-character hexadecimal digest**.
      That function's reference page describes its return as a hex string and states no length,
      and no published value was matched against it. The function was available on that target,
      in that tenant, on that
      date. That removes the availability question
      and not the variant's other blocker: the table did not resolve there, so the
      variant stayed undeployable for a different reason. **Every other use of one in this pack is
      a check run once rather than a rule scheduled**, this one included. Run it in the Defender
      portal specifically, not in Sentinel, because that is
      where the variant would be scheduled. If it errors, MSD-003's primary query is unaffected and
      the variant is not deployable as written.

```kusto
print H = hash_sha256("test")
```

---

## Group 2 - The email prompt-injection surface *(MSD-001, MSD-002)*

- [ ] `EmailEvents | where Timestamp > ago(30d) | summarize count() by DetectionMethods` -
      **record the actual serialisation.** Learn types this column `string` and publishes no value
      format. Is it a bare string, a delimited list, or JSON?
- [ ] Confirm at least one row carries `Prompt injection protection`. If none does, record that -
      and note you cannot conclude from this alone whether the capability is inactive.
- [ ] **Settle whether `has` matches a right-hand side carrying more than one term.** This needs no
      table and it is the pack's largest declared evidence gap: the string-operators reference defines
      a term as a maximal sequence of alphanumeric characters and every example of `has` on it puts a
      single term on the right, while MSD-001 and MSD-002 both ship the three-term literal
      `Prompt injection protection`. Run this:
      `print MultiTerm = ("Advanced filter, Prompt injection protection" has "Prompt injection protection"), Control = ("Advanced filter, Prompt injection protection" has "injection"), Negative = ("Advanced filter, Antimalware protection" has "Prompt injection protection"), Scramble = ("protection injection Prompt" has "Prompt injection protection"), ControlNeg = ("Advanced filter, Antimalware protection" has "protection"), ControlScr = ("protection injection Prompt" has "injection")`.
      **Read the columns together rather than one at a time**, against the decision table below
      this list. **Three of them are positive controls, one for each left-hand string.** `Control`
      covers the string `MultiTerm` uses, `ControlNeg` covers `Negative`'s and `ControlScr` covers
      `Scramble`'s, each putting a single term on the right, and all three should be true whatever
      the operator does with a multi-term right-hand side. A false one means that left-hand string is
      not tokenising as this test assumes, and no reading resting on it can be read.
      `Scramble`'s left-hand side carries all three right-hand terms and does not carry them in
      the phrase's order, so it distinguishes a phrase match from a decomposition; `Negative`'s shares
      exactly one term, so it is what narrows **which** decomposition. **A statement that returns an
      error rather than a row of values is a result as well**, and it is not one of the eight
      combinations the table is built from. **It does not on its own say which expression caused it**,
      because a compile error fails the whole statement, so isolate it in two steps rather than one.
- [ ] First re-run
      `print Control = ("Advanced filter, Prompt injection protection" has "injection")` by itself.
      That re-run drops five of the six columns and two of the three left-hand strings as well as the
      multi-term right-hand side, so read it as the check that the machinery and the literal are
      sound rather than as the answer: if it errors too, the multi-term right-hand side is not what
      the engine rejected and the error text is about something else.
      **If it errors too, record the step as not completed with both error texts and stop the split
      here.** That re-run is a single-term `has` between two string literals inside `print`, so an
      error on it means your target rejected the test machinery rather than the multi-term
      right-hand side, and **the four Group 1b checks rest on the same machinery and are
      unestablished on this target as well**. **No row of the table below is in evidence in that
      state**, because none of `MultiTerm`, `Negative` or `Scramble` has a value. Decide what to
      deploy on what your workspace actually emits rather than on this test, and record in the
      verification report that the operator test could not be run here: that record is what tells
      this pack its own test does not travel. If it succeeds, run
      `print MultiTerm = ("Advanced filter, Prompt injection protection" has "Prompt injection protection")`,
      which differs from the line above in the right-hand side and in the column alias; the alias
      only names the column the statement returns, and it is there because the value you need next
      is the `MultiTerm` one. If that errors where the line above did not, the multi-term right-hand
      side is what the engine rejected: record the error text and take the `has_all` split.
      **If both succeed, the error came from elsewhere in the six-column statement, and the
      `MultiTerm` re-run has still returned a value: read it before you go looking.** The columns
      you are still missing are `Negative` and `Scramble`, and the statement that carried them is
      the one that errored, so get them the way you got the first two, one expression per statement,
      taking each expression unchanged from the six-column statement above. Run
      `print Negative = ("Advanced filter, Antimalware protection" has "Prompt injection protection")`,
      then `print ControlNeg = ("Advanced filter, Antimalware protection" has "protection")`,
      then `print Scramble = ("protection injection Prompt" has "Prompt injection protection")`,
      then `print ControlScr = ("protection injection Prompt" has "injection")`.
      **Whichever of the four errors is the expression the engine rejected**: record the error text.
- [ ] Read the controls before the readings, on the rule stated above: a false `Control`,
      `ControlNeg` or `ControlScr` means that left-hand string is not tokenising as this test
      assumes, and no reading resting on it can be read. **A false `MultiTerm` puts you in row 1
      only where `Control` is true and `Scramble` is false**; a false `MultiTerm` with a true
      `Scramble` is one of the two combinations the table omits, and the paragraph under the table
      tells you to record that as a broken test rather than as a result. **Row 1's prescription and
      the repair those broken-test cases call for are the same `has_all` split**, so you can act on
      a false `MultiTerm` before you have `Scramble`; what `Scramble` buys you is the record of
      which of the two you were in. A true `MultiTerm` places you nowhere on its own, because rows
      2, 3 and 4 all carry one. **So `Scramble` is not optional once `MultiTerm` is true**, and
      there is no equivalent bridge on that side. With `MultiTerm` true and `Negative` false, rows 2
      and 3 are both live and they do not prescribe the same thing. **The two prescriptions differ
      in effect and neither is a superset of the other**: shipping row 3's `has_all` split while you
      are in fact in row 2 widens the filter, because `has_all` does not care about order and row 2
      is the row where order is what you observed, and leaving the filter alone while you are in
      fact in row 3 keeps the over-match that row exists to remove. Get `Scramble` before you
      decide; its expression is the same form as `MultiTerm`'s, so a target that ran one runs the
      other. If you cannot get it, record that you are between rows 2 and 3 rather than choosing
      one.
      **All the left-hand sides are arrangements built for this test rather than values any page
      publishes, and there are three distinct ones.** Two of the three join two published
      detection-technology values with a comma. The third, `Scramble`'s, re-orders the terms inside
      one published value, which it has to: on the
      detection-technology page this pack cites, no published value other than the target itself
      carries `Prompt` or `injection`, so an arrangement that scrambles those terms cannot be
      assembled from published values at all. Each of the three appears in more than one column of
      the statement, which is what lets one control cover each. The test reads no table.

### Reading the multi-term result

**Everything from here to "The remaining Group 2 steps" below is exposition rather than a step**, and
the tick list resumes under that heading. It is here because the step above cannot be acted on
without it.

The decision table for that step sits at the left margin, for the same reason the Group 1b query
blocks do: it is a thing to read across rather than a thing to tick. **Every reading in it is this
pack's inference from the operator's documented definition rather than anything Microsoft states
about a multi-term right-hand side**, which is the same register the paragraph below the table uses:
the string-operators reference defines the operator for a single-term right-hand side and settles
none of these rows.

| `MultiTerm` | `Negative` | `Scramble` | What `has` did | What to ship |
|---|---|---|---|---|
| false | false | false | Did not match a multi-term right-hand side at all, even with the phrase present in the left | The `has_all` split. The shipped filter in both files matches nothing until you make it |
| true | false | false | Matched the terms in order. **This row does not say whether it also required them to be adjacent.** A phrase match and an in-order-but-separated match both produce it, and nothing in the statement separates them | Nothing changes for the published value set, where no value other than the target carries `Prompt` or `injection`. Record that you could not exclude the in-order-but-separated reading, because it bears on a value your own workspace emits and no page publishes |
| true | false | true | Decomposed the right-hand side. **This row does not say which decomposition.** Requiring every term and keying on one leading term both produce it, and `Negative` cannot tell them apart, because the only term its left-hand side shares with the right-hand side is the last one | **The `has_all` split**, which is never wider than any **term-presence** decomposition consistent with this row: it changes nothing if the operator required every term, and it removes an over-match if the operator keyed on one. It is not narrower than a decomposition that also required the terms to sit together, which this row does not exclude. Ship `contains` only if you need a match on the phrase rather than on the terms, and record that choice |
| true | true | true | Matched on at least one term of the right-hand side, so the shipped filter over-matches | The `has_all` split, which requires all three terms and is **strictly narrower** than what you observed. **The guarantee is quantified over the same term-presence readings row 3 names**, and this row's own operands witness it: a left-hand side sharing exactly one term with the right-hand side matched, so requiring all three cannot match more |

**A true `Negative` forces a true `Scramble`**, because `Scramble`'s left-hand side carries the same
shared term, so there is no row where `Negative` is true and `Scramble` is false. **That follows from
the shared term only under two assumptions this test cannot check: that matching depends on which
right-hand terms are present rather than on where they sit, and that adding terms to a left-hand side
never removes a match.** The second is easy to leave out and the entailment needs it: an operator
matching only when exactly one right-hand term is present would give a true `Negative` and a false
`Scramble`, which is the combination the entailment says cannot occur. No plausible operator behaves
that way, and the paragraph states the assumption rather than relying on that. Row 3 above does not
rest on either.

**Four of the eight combinations are absent from the table, and here is why.** Two are the ones the
entailment just stated rules out. The other two pair a false `MultiTerm` with a true `Scramble`, and
both are incoherent under the same assumption: each of those two left-hand sides carries all three
right-hand terms, so a statement that matches the scrambled arrangement and not the other one is not
reading the terms at all. **Record any of the four as a broken test rather than as a result**, and
the ground differs across the two pairs. For the two pairing a false `MultiTerm` with a true
`Scramble` it is **that same presence-rather-than-position assumption, which row 2 above declines to
settle.** For the two the entailment rules out, observing one means an assumption the entailment
rests on does not hold on your target, which is a statement about the test rather than a reading of
the operator.

**How far the positive controls reach.** `Control` shares its left-hand side with `MultiTerm` and
with neither of the other two, so on its own it confirms that one string tokenises as expected and
says nothing about how the other two do. **`ControlNeg` and `ControlScr` close that gap**, one for
each remaining left-hand string, which is why the statement above carries six columns rather than
four: naming a limit and leaving it open costs the same two expressions as closing it. A false
control of any kind is decisive for every reading resting on that string. A true one still says
nothing about how the operator treats a multi-term right-hand side, which is the whole question.

**What to do when a control comes back false, because saying a reading is lost is not the same as
saying what to do without it.** A false `Control` is the decisive case: `MultiTerm` rests on that
left-hand string, and every row of the table above keys on `MultiTerm`, so no row of the table is
live. Record the step as not completed with the result you got, decide what to deploy on what your
workspace actually emits rather than on this test, and record in the verification report that the
operator test could not be run here. **That is the disposition the error branch in the step above
takes, and it stops one clause short of it**: that branch also reports the four Group 1b checks as
unestablished on the target, and a false `Control` does not reach them, because the statement ran
and the machinery is not what failed. A false `ControlNeg` or `ControlScr` is narrower, costing you
`Negative` or `Scramble` rather than `MultiTerm`. A lost `Scramble` is the case the step above
already describes twice, once where `MultiTerm` is false and once where it is true. **A lost
`Negative` does not change what you ship**, because the only two printed rows it separates, rows 3
and 4, prescribe the same `has_all` split; record that you could not separate them rather than
choosing one.

**The split form** is documented on the string-operators page as "Same as `has` but works on all of
the elements", with the worked example `"North and South America" has_all("south", "north")`.
**The shape to run is `where DetectionMethods has_all ("Prompt", "injection", "protection")`**,
which the 2026-08-24 run did not exercise: the operator test it ran covers `has` with a
multi-term right-hand side, not this split form. **`has_all` is order-independent and position-independent, so
it is weaker than a phrase match**: it also matches a value carrying the three terms in any
arrangement. **Against the published value set that weakness cannot bite**, because on the
detection-technology page this pack cites, no published value other than the target carries `Prompt`
or `injection`, so the only published value satisfying all three terms is the target itself. What the
weakness is about is a value your own workspace emits that no page publishes. **`has_any` is not the
answer here** - it matches on any one of the elements, so `has_any ("Prompt", "injection",
"protection")` would also match the published value `Antimalware protection`, a real row on the
detection-technology page this pack cites.

**For a match on the phrase rather than on the terms, this pack's reading is that `contains` is the
operator**, which the same page defines as matching where the right-hand side "occurs as a
subsequence of" the left. **"Subsequence" and "phrase" are not the same idea, and the step from one to
the other is this pack's inference rather than anything Microsoft states**, which is a further reason
to run the test rather than to trust the reasoning. The cost is one that page names in the same place:
a query that "uses a `contains` operator" will "revert to scanning the values in the column", and
"Scanning is much slower than looking up the term in the term index". MSD-001 already offers that swap
for a different case with the same cost stated, **and it puts the serialisation question first**:
replace the literal with the form your workspace actually emits before you change the operator,
because the two are separate decisions and the observed form can remove the need for the swap. No
amount of further page reading settles any of this; only the test does.

### The remaining Group 2 steps

- [ ] Adjust the `has` operator in MSD-001 and MSD-002 if the observed serialisation requires it,
      **and read the result of the multi-term check above before you choose**. The operator choice and
      the serialisation are two questions and the pack's own guidance is to fix the literal to the
      observed form first, then pick the operator. MSD-001's step 2 carries four branches. The first
      three are keyed on the serialisation you observed; **the fourth is keyed on this test** and
      ships the same `has_all` split prescribed here, so the two files agree and you can make the
      change from either.
- [ ] `EmailEvents | where Timestamp > ago(30d) | summarize count() by DeliveryAction,
      DeliveryLocation, EmailDirection` - confirm your workspace emits the documented value strings
      **exactly**. **Those counts stay in your environment**, as Group 0 states for every step in
      this pack: what travels from this step is a value name and never a count. **A spelling
      difference on one of MSD-002's `DeliveryLocation` literals drops the rows carrying that value
      rather than emptying the result**, and a partly populated result hides the fault better than an
      empty one does. Casing is covered since that filter moved to `in~` in the correction of
      2026-08-25, and a
      different token is not. **MSD-002's `EmailDirection == "Inbound"` filter still breaks on casing
      alone**, and that one does empty the result: `==` is the case-sensitive comparison MSD-007
      records against the operator's own
      reference page, and both MSD-002 queries ship it. **This is the step that found the
      `DeliveryLocation` defect** in the 2026-08-24 run.
- [ ] Compare `DeliveryLocation` against `LatestDeliveryLocation` on a sample of rows. Record how
      far apart they run.
- [ ] Identify and record your phishing-simulation sender domains. These are the largest expected
      false-positive source for both detections. **That list stays in your environment**: it names a
      service relationship and the infrastructure that sends on your behalf, which is exactly what
      Group 0 says does not travel outward.
- [ ] **Confirm `Timestamp` and `ReportId` are both in the output before you create a custom
      detection rule from either query.** Microsoft's custom-detection-rules page recommends
      projecting both for tables outside Defender for Endpoint, and names the cost of omitting them
      as alerts not tagged with the correct entity scope and a less enriched alert timeline. Both are
      projected in MSD-001 and MSD-002 as shipped; this step is to catch the case where you have
      edited the projection. Microsoft states it as a recommendation rather than a requirement, and
      **the page does not say the rule wizard rejects a query without them** - so read a dropped
      projection as enrichment you gave up rather than as a rule that will fail to save. MSD-001
      carries the same wording.
- [ ] For MSD-002 specifically, confirm `NetworkMessageId` and `RecipientEmailAddress` survive any
      edit you make to the projection. Microsoft states both must be present in the query results to
      apply actions to email messages, so removing either takes the response actions off the rule.

---

## Group 3 - The AI agent inventory surface *(MSD-003, MSD-004)*

- [ ] Confirm the **Microsoft 365 app connector** is configured to collect Agent 365 observability
      data. **Without it, expect `AgentsInfo` to be empty however many agents exist**, and check this
      prerequisite before concluding from an empty result that you have no agents. **What the
      connector gates is not settled here**: the same feature page also states, verbatim, that
      "Agents built with Microsoft Copilot Studio, Microsoft Foundry, and declarative agents built
      with the Microsoft Copilot Agent Builder send observability data to Microsoft 365 by
      default", and how the two statements interact was not
      established for this pack. MSD-003 states the prerequisite in the same terms.
- [ ] `AgentsInfo | extend R = tostring(McpServers) | where R !in~ ("", "[]", "{}", "null") | take 5
      | project AgentId, McpServers` - **read the JSON.** Record the field names. They are not
      documented, and no query should index into this column before you have. This is MSD-003
      verification step 2; a bare `isnotempty(McpServers)` returns every agent whose platform emits
      an empty array, so a `take 5` after it hands you five arbitrary agents rather than five with a
      server attached, and the step teaches you nothing.

> **Exactly two things from `McpServers` are findings: its field names, and the empty forms your
> platforms emit for it.** Microsoft's own description of this column, quoted in MSD-003 and
> MSD-004, is "The Model Context Protocol (MCP) servers connected to the agent, **including server
> URLs and credential configuration**", so a populated value is environment data of the most
> sensitive kind this pack asks you to look at. **Never paste this column's contents anywhere - not
> into an issue, not into a verification report, not into a pull request, and not privately either.**
>
> The step above gives you field names. **The step below groups by the whole value string, so for
> this column it will render populated entries on your screen.** That is unavoidable in your own
> portal and it changes nothing about what leaves it: read the empty forms out of that output and
> leave everything else there. MSD-004 verification step 2 states the same rule for the same query.
- [ ] **Enumerate what your platforms emit for an empty column**, which is what every `AgentsInfo`
      predicate depends on: `AgentsInfo | where Timestamp > ago(30d) | summarize Rows = count() by
      Platform, GuardrailsRaw = tostring(Guardrails) | order by Rows desc
      | take 30`. Repeat for `DeclaredTools`, `Endpoints` and `McpServers` - `Endpoints` because
      MSD-004's posture rollup counts agents by whether it is empty, and `McpServers` under the rule
      in the note above. **If a form appears that is not `""`, `[]`, `{}` or `null`, add it to the
      empty-forms predicate in every MSD-003 and MSD-004 query.** **The two files build that
      predicate differently, and only one of them has a named list to edit.** MSD-004 binds
      `let EmptyForms = dynamic([...])` and tests against that identifier, so a fifth form goes into
      the list once per query. **MSD-003 carries no `EmptyForms` identifier at all**: it writes its
      forms inline as `!in~ ("[]", "{}", "null")` and handles the empty-string case separately
      through `isnotempty()`, so a fifth form goes into each inline list, or into the `isnotempty()`
      leg if it is empty in that sense. **Searching MSD-003 for `EmptyForms` returns nothing, and
      that is not evidence MSD-003 needs no change.**
      **The query above does not collapse to one row per agent, and that is deliberate.** MSD-003's
      and MSD-004's primary queries open with `summarize arg_max(Timestamp, *) by AgentId` to count
      agents; for this question that collapse hides a form, because an empty form a platform emitted
      in an earlier snapshot inside the window is not in the collapsed set and is still a form every
      predicate has to allow for. **The alias is `Rows` and not `Agents` for the same reason**, so the
      figure reads as snapshot rows rather than as agents. MSD-003 verification step 3 ships that
      shape over its own column and states the reason there.
      **The ordering is a convenience rather than the answer.** `order by ... desc` with a `take`
      surfaces the common forms, and an unanticipated fifth form is by construction a rare one, so the
      truncation hides exactly the case this step exists to find. **Run this as well:**
      `AgentsInfo | where Timestamp > ago(30d) | summarize Rows = count() by GuardrailsRaw =
      tostring(Guardrails) | where GuardrailsRaw !in~ ("", "[]", "{}", "null") | order by Rows asc |
      take 20`. It drops the four forms you already account for and brings the rarest of what is
      left to the top. **It also groups by the value alone rather than by platform and value**, so
      its `Rows` figures are across platforms and are not comparable with the block above, where the
      same alias counts rows per platform and value. A form several platforms emit shows as one row
      here, which is what keeps it inside `take 20`. **That is a
      candidate filter and not a definition of emptiness**: some of what it returns will be genuinely
      populated values rather than a fifth empty form, and telling those apart is the reading this
      step asks of you. For `McpServers` the note above governs what you do with the contents.
      *(Group 1b already established that `isempty()` alone is false for `[]`; this step establishes
      which forms your own platforms actually produce.)*
- [ ] **Measure the emission cadence**, which decides whether the queries' 30-day window is safe:
      `AgentsInfo | where Timestamp > ago(30d) | summarize Rows = count() by AgentId
      | summarize percentiles(Rows, 50, 95)`. If most agents show one or two rows over 30 days,
      rows are change-driven and a shorter window silently drops stable agents.
- [ ] Confirm the `LifecycleStatus` and `PublishedStatus` values match the documented sets
      (`Active`/`Blocked`/`Uninstalled`/`Deleted`, `Draft`/`Published`).
- [ ] Record the `Availability` values your workspace emits. Learn publishes no value list, and
      MSD-004 deliberately does not filter on it until you do. **Record the shape before the
      values.** Learn's description covers a specific-groups case and does not publish the
      serialisation, so a platform that serialises that case by naming the groups puts your own
      organisation's naming in the column. **Send the shape you found. A value travels only if you
      have looked at what it contains and it carries nothing of your own organisation's; if it
      carries anything of yours, the shape travels and the value does not.**
- [ ] Run the MSD-004 posture rollup and **record the per-platform baseline.** Without it you cannot
      distinguish a posture finding from a platform that does not report guardrails.
- [ ] Search your saved and scheduled queries for any remaining reference to `AIAgentsInfo`. Learn
      states it remained accessible until July 1, 2026, a date now past.
- [ ] **Before scheduling MSD-003's change-detection variant, measure how much its fingerprint moves
      on its own**, having first confirmed the function runs at all in Group 1b check 4:
      `AgentsInfo | where Timestamp > ago(30d) | extend F = hash_sha256(tostring(McpServers))
      | summarize Fingerprints = dcount(F), Rows = count() by AgentId | order by Fingerprints desc`.
      An agent showing many distinct fingerprints over many rows is changing something inside the
      column often. This measures the rate and not the cause, which is MSD-003 verification step 7.
- [ ] Note that **thirty days is a ceiling on this table as well as a starting point.** The advanced
      hunting overview states, verbatim: "Each query can look up native Defender XDR data from up to
      the past 30 days", extending only where a Microsoft Sentinel workspace is onboarded with longer
      analytics-tier retention. MSD-003's and MSD-004's baselines already sit at that ceiling, so if
      the cadence you measured above is slower than 30 days, widening the window is not the fix
      available to you.

---

## Group 4 - The behaviour surface *(MSD-008)*

**Group 1's `BehaviorInfo | getschema` diff is the prerequisite for this group, and it is not
repeated here as a second checkbox.** On 2026-08-24 a tenant did not carry `Title`, which
Microsoft Learn publishes and MSD-008's schema table reproduces, and every query that file shipped
for deployment referenced it and failed to resolve it. **Step 1 now names it behind
`column_ifexists()`, which returns `Description` where the column is absent; step 2 and the
self-check wrapper name it nowhere.**
**Record a column your own output does not carry as a schema mismatch there** where Microsoft's
page no longer carries it, and as a discovery where the page still publishes it, rather than editing
a query until it runs. `Title` on `BehaviorInfo` is the worked example: the page publishes it, one
tenant's schema did not carry it, and no citation was stale.

- [ ] Run the MSD-008 step-1 discovery query over the **full retention window** and record the
      complete `ActionType` / `ServiceSource` / `DetectionSource` / `Categories` inventory.
      **`Categories` came back as a serialised array string in one environment**, so the values
      this rollup returns for that column may be combinations rather than category names. **Record
      them as the shape you found, and do not build an equality filter from one**: such a filter
      matches only the rows carrying that exact combination and silently misses every other
      combination containing the same category.
      **Thirty days is a ceiling here rather than a floor.** The advanced hunting overview states,
      verbatim: "Each query can look up native Defender XDR data from up to the past 30 days", and
      records that the range extends only where a Microsoft Sentinel workspace is onboarded and its
      analytics-tier retention is longer. Confirm which case yours is before treating the inventory
      as complete. **Step 1's `column_ifexists()` guard has already been exercised, and this is not
      its first run**: the 2026-08-26 run parsed the guarded rollup inside the aggregation, and the
      2026-09-11 run reproduced that result. **Name the question that is settled and the one that is
      not.** What is settled is narrow: the guarded expression **parses and runs inside an
      aggregation**, which is exactly what that function's reference page leaves open by applying it
      under `project` instead. **What is not settled is whether the fallback returns the intended
      value**, because establishing that needs a workspace whose schema lacks the named column and
      which also returns rows, and no run has had both. Record a
      parse error here as a defect in the query rather than as a schema mismatch.
      **The output stays in your environment**: that query returns up to twenty-five
      free-text behaviour labels out of your own tenant in `SampleLabels` **for each row
      it returns**, and it returns one row per `ActionType` / `ServiceSource` / `DetectionSource` /
      `Categories` combination, so twenty-five is a per-row cap and not a ceiling on the total.
      `Title` and `Description` are documented only as "Title of the behavior" and "Description of
      the behavior", so nothing tells you in
      advance what yours will carry. **What travels from this step is a closed list: the `ActionType`,
      `ServiceSource`, `DetectionSource` and `Categories` value strings, and nothing else.** Not
      `SampleLabels`, not the behaviour count, and not the first-seen and last-seen timestamps -
      the count and the dates describe your estate rather than the schema. MSD-008 verification step 2
      states the same rule in the same words.
- [ ] **Generate a known AI-agent protection event in a lab tenant** and locate it in that output.
      This is the positive control. Without it you cannot distinguish "no events" from "wrong
      filter", and MSD-008 stays unverified.
- [ ] Record which platform your agents run on. Microsoft states block-event recording "isn't yet
      supported for agents built with Microsoft Copilot Studio" - if your estate is mostly Copilot
      Studio, record how much of it this detection covers.
- [ ] Write the identified value set, **with the date**, next to the deployed rule. It is
      undocumented, so it can change without notice.
- [ ] Read the `BehaviorEntities` table reference before adding any join, **for the columns a join
      would return.** MSD-008 quotes that reference for the table's description and its status, so
      those two questions are settled there. **What is not settled is anything about your own
      workspace**: no query in this pack joins to that table, and beyond a single column-set
      reading, nothing it publishes has been checked against a
      workspace here.
- [ ] **Run that join in a lab tenant against the behaviour you generated above, and record what its
      entity-type and entity-role columns carry for an agent-initiated behaviour.** MSD-008 calls
      this the one thing that would close the most consequential limit in that file, which is that a
      behaviour cannot be attributed to the agent that produced it, and no page this pack read
      settles it. MSD-008 verification step 6 states the same action.

---

## Group 5 - The Defender for Cloud alert surface *(MSD-005)*

- [ ] `SecurityAlert | where TimeGenerated > ago(30d) | summarize count() by ProductName,
      ProviderName` - confirm Defender for Cloud alerts reach the workspace.
- [ ] **Reconcile MSD-005's 17 alert names against the published list character by character, and
      treat casing as part of the check.** MSD-005 filters with case-sensitive `in` over long
      literals transcribed from Microsoft's alerts page, so any drift between the page and the file,
      including a change of case, makes that leg match nothing and say nothing. **The check reaches
      both of MSD-005's queries**: its prompt-injection subset matches on three alert-type
      identifiers from the same page, also with case-sensitive `in`, and that file says in terms
      that both its queries filter that way. **This is the same hazard Group 2 above raises for
      MSD-002**, at seventeen alert names rather than a handful of short values. Record any difference as a
      schema mismatch against MSD-005 rather than fixing it locally.
- [ ] `SecurityAlert | where TimeGenerated > ago(90d) | where AlertType startswith "AI." or
      AlertName has "AI model" | distinct AlertName, AlertType | take 50` -
      **establish which column carries the identifier.** Learn's `SecurityAlert` reference publishes
      these column names with empty descriptions. Narrow the shipped query to whichever your
      workspace fills. **`has "AI model"` puts two terms on the right-hand side**, which is what
      Group 2 above exists to settle and which none of the pages this pack read settles. If that test
      returns a false `MultiTerm`, this leg matches nothing: **split it across `has_all`**, which is
      the option that survives either answer to the question this step is asking. **Dropping the leg
      and relying on the `AlertType` prefix alone assumes that answer**: in a workspace that fills
      `AlertName` and leaves `AlertType` empty, which is one of the two states this step exists to
      distinguish, the reduced query returns zero rows, and **a zero from it is uninterpretable
      rather than clean**. **How far the `AlertName` leg reaches**: of the seventeen alert names
      MSD-005 reproduces, **four carry the literal `AI model`**, so that leg alone finds those four
      and the `AlertType` prefix is what reaches the rest. MSD-005 verification step 2 carries the
      same warning.
      **One difference from Group 2's own operands, and record whether it changes the answer.** The
      string-operators page states that the term index carries "all terms that are three characters
      or more" and that where a term is shorter the query "will revert to scanning the values in the
      column". `AI` is two characters, so this right-hand side mixes an indexed term with one that is not, where every
      Group 2 operand is three characters or more. The page frames that as an index-versus-scan path
      rather than as a change in what matches, so carrying Group 2's result across is probably sound;
      it is not something the page says.
      **Source for those two quotations**, recorded here beside them because this file has no Sources
      section: [String operators (Kusto Query Language reference, Microsoft
      Learn)](https://learn.microsoft.com/en-us/kusto/query/datatypes-string-operators), read
      2026-08-22, for the term-index threshold and the scan a shorter term falls back to. MSD-001's
      Sources entry records the same read against the same page.
- [ ] Confirm the Defender for AI Services plan is enabled on the subscriptions hosting your Azure
      AI resources.
- [ ] **Trigger one alert deliberately in a lab subscription** and confirm it arrives with its
      identifier intact. This is the positive control for the previous step.
- [ ] Re-read the alerts page and reconcile against the 17-item table in MSD-005. Two entries
      carried a `(Preview)` tag at the pack's verification date; record whether that changed. **The
      page carries one further entry with no published `AI.` identifier** - a Kubernetes exposure
      alert - so a heading count and an identifier count do not agree, and MSD-005's table is the
      identifier set. That is not a gap in the table.
- [ ] Confirm you are not double-counting the same alert through two connectors.
- [ ] **Check whether Microsoft Sentinel is already creating incidents automatically from Microsoft
      Defender for Cloud alerts.** **Learn names where to look, so this is a lookup rather than a
      hunt.** On the Defender for Cloud data connector, look for the **Create incidents -
      Recommended** section and whether its **Enable** control is on; from the Analytics page the
      same page gives **Create > Microsoft incident creation rule** as the from-scratch route. What
      either produces is a Microsoft security analytics rule created from a rule template, with one
      template per Microsoft source solution; MSD-005 carries that page in its Sources. **The page
      prints the separator inside that section name as an en dash and this pack reproduces it as a
      hyphen**, so search the portal for the words rather than for the punctuation. If incidents are
      already being created that way, deploying MSD-005 as an analytics rule creates a second
      incident for every alert that already has one. Either run it as a hunting query, or **deploy
      it as an analytics rule with incident creation turned off and triage from the alerts.** **A
      rule with incident creation off puts nothing in the incident queue**, so triage means the
      alert queue or a `SecurityAlert` query rather than the incident workflow you may be used to.

    > **Correction, 2026-09-19.** An earlier version of this step offered alert grouping against
    > `SystemAlertId` instead, which does not solve the problem stated, because alert grouping is
    > configured per analytics rule and cannot merge its incident with one another rule created.
    > **The replacement is reasoned from the product's per-rule incident setting and has not been
    > exercised here**, and the duplicate-incident question stays recorded as not completed.

    > **Correction, 2026-08-24.** An earlier version of this step said that no page this pack has
    > read publishes a label for the control, and declined to name where to look on that ground.
    > The page MSD-005 cites publishes two labels, and both are named above.

---

## Group 6 - The workload-identity surface *(MSD-006)*

- [ ] Confirm service principal sign-in logs are exported to the workspace. This is a **separate
      diagnostic-setting category** from user sign-ins, so an empty table here does not mean an
      empty tenant.
- [ ] Run the MSD-006 step-1 query and **record every `ConditionalAccessStatus` value your tenant
      emits, with counts.** This is the value set the pack deliberately does not guess. **Note the
      asymmetry, corrected 2026-09-19**: Microsoft Graph publishes an enumeration for the analogous
      `signIn` property, while this table's own column reference publishes none, so what is
      unconfirmed is whether the Log Analytics column stores those same strings. Your reading is
      what settles it. If you report it back, send the value strings and leave the counts behind -
      the value names describe the schema, the counts describe your estate.
- [ ] `AADServicePrincipalSignInLogs | summarize count() by ResultType | take 20` - record whether
      your workspace stores the semantic strings Learn describes, or something else.
- [ ] Inspect the `Agent` column on a sample of rows and record what it contains. Learn documents it
      in one sentence, "Details of agentic sign-in", and publishes no value set. **That sentence
      describes per-event content rather than a class the platform assigns, and that is what makes
      it a free-text column: there is no value name to send that is not also the content, so what
      travels is the column name and the shape you found, never the contents** - the same rule the
      results section states for every free-text column it names, `Agent` among them. **Having no
      documented value set is what sends a column to the routing test in that section; it is not on
      its own what makes a column free text**, which is why this group's `ConditionalAccessStatus`
      step asks you for that column's value strings. **MSD-006 marks `Agent` Provisional as a
      column, not as a serialisation**, and an undocumented serialisation is not on its own a reason
      to treat a column as free text. MSD-006's step 2 projects this column, so the rule bears on a
      query you deploy and not only on a report.
- [ ] **Inspect the two columns whose serialisation MSD-006 flags Provisional, before writing any
      filter on either:** `AADServicePrincipalSignInLogs | take 5 | project ConditionalAccessPolicies,
      LocationDetails`. Learn types both `string` and gives each a composite description, which does
      not tell you whether the stored value is flat or structured. Record which it is. **This is the
      step the Group 1 `getschema` diff sends you to**, and it is the only step in this checklist that
      settles those two serialisations; MSD-006 verification step 6 is the same check in its own file.
- [ ] **Build the AI application inventory.** Map which `AppId` values correspond to AI applications
      and agents. Nothing in the telemetry does this for you, and without it MSD-006 is a general
      workload-identity query rather than an AI one.

> **Both artefacts this group asks you to produce stay in your environment.** The `AppId`-to-
> application map and the list of which agents authenticate outside Entra ID are named here because
> the first ties your own application estate to the identities behind it, and the second is a written
> inventory of your own identity-monitoring blind spots, which is as
> sensitive as anything in this file. **Neither belongs in a public issue, a verification report, or
> a pull request.** If you need help interpreting them, use the private route in `SECURITY.md`, at
> the root of this repository.
>
> **Naming these two does not clear the rest.** They are singled out because they are among the most
> sensitive things this file asks you to write down, not because the default in Group 0 stops
> applying to everything else.
- [ ] Confirm whether the tenant holds **Workload Identities Premium**. Without it, existing
      policies keep working but cannot be created or modified - which changes what you can do about
      a finding.
- [ ] **Record how your AI applications and agents authenticate.** Any that use an API key bypass
      Microsoft Entra ID entirely, per Microsoft Learn, and on this pack's reading therefore produce
      no row in this table at all. Write down that list; it is the detection's blind spot made
      explicit.

    > **Correction, 2026-08-24.** An earlier version of this step ran the no-row consequence under
    > the same attribution as the bypass. The cited sentence supports the bypass and the
    > non-application of policy and says nothing about this table, so the consequence is now
    > marked as this pack's reading, which is how MSD-006 and the cross-walk already state it.

---

## Group 7 - The Copilot audit surface *(MSD-007)*

- [ ] Confirm the **Microsoft Copilot logs data connector** is deployed. Per Learn the prerequisite
      is a tenant role - "'Security Administrator' or 'Global Administrator' on the workspace's
      tenant" - and no licensing or SKU requirement is stated for the connector itself.
- [ ] Run the MSD-007 step-1 query and record the full `RecordType`, `Workload` and `AppHost`
      inventory. Compare against the examples Learn gives behind an "e.g." for `RecordType` and for
      `Workload`, note that it gives `AppHost` neither an example nor a value set, and **note the
      difference**. **That difference is what routes `AppHost`**: it is named in the no-value-list
      class under **Recording the result** below, so read what its values contain before any of them
      travels and send the shape rather than the value where they name anything specific to your
      estate, whoever chose the name. **The inventory you record here is governed there too, by a
      separate rule from the per-value one**: it states the test for what makes a value specific, and
      it says what happens to the set.
- [ ] Inspect `LLMEventData` on a sample of rows, per `RecordType`, and record what it contains.
      **Re-read MSD-007's opening warning before drawing any conclusion from what you find**, and
      keep what you write down in your environment. MSD-007's step 3 states what a public issue or a
      verification report can take from it, and why.
- [ ] Confirm the table's plan in your workspace (Analytics, Basic, or Auxiliary) and confirm a
      scheduled analytics rule can run against it on that plan.
- [ ] **Make one Copilot settings change deliberately in a lab tenant** and confirm it appears with
      the expected `RecordType`. Positive control.
- [ ] **If that change does not appear, search the unified audit log directly before concluding
      anything about your connector, your workspace or the filter value.** MSD-007 verification
      step 5 sets out the route and the three bounds on it: the audit log is reachable through
      Microsoft Graph on the v1.0 endpoint where the Exchange Online module is not available to you;
      **this pack reads that query as needing its own permission, with an advanced-hunting grant not
      carrying across to it**, neither cited page stating either way; audit timestamps are UTC, so
      pad a single-day window by a day on each side and state the
      timezone you mean; and **enumerate the operation names in the window rather than matching on
      terms**, because a term search excludes only what its terms match. Record which of those you
      did, since an unpadded or term-matched search does not support an absence claim.
- [ ] Re-read the Sentinel data-connectors reference and record whether the page-level-notice versus
      per-entry-tag conflict still stands. **If Microsoft resolves it, MSD-007's status label
      changes** - and only then.
- [ ] Record which `Workload` values separate Microsoft 365 Copilot from Security Copilot in your
      tenant. The connector spans both.

---

## Group 8 - Before you deploy anything as a rule

- [ ] Every query above returns a **result you can explain**, including the empty ones.
- [ ] Every placeholder value set in **MSD-006 and MSD-008** has been **filled from observed data**.
      A rule deployed with a placeholder in place returns nothing and looks healthy. *(MSD-007 ships
      no placeholder - its step 2 and step 3 are complete queries. Three detections ship a discovery
      query first, MSD-006, MSD-007 and MSD-008; only two of those carry a placeholder list.)*
- [ ] **Group 1b's results are recorded, and the deployment decisions they gate are made**: which
      MSD-008 query shape is safe to deploy, whether MSD-008's self-check wrapper is usable at all,
      **which failure mode MSD-006's step 2 gives if its placeholder is left unfilled** (an empty
      match or a parse error, MSD-006 having ruled out deploying it unfilled under either branch),
      and whether MSD-003's change-detection variant is deployable. **Check 2 gates two files, not
      one**: MSD-006's step-2 `NoPolicyApplied` list uses the same comment-only `dynamic([...])`
      form MSD-008 does, and while MSD-006's own file names both outcomes in the paragraph under
      that query, the comment inside the block names neither. **Check 1 gates no
      deployment decision of its own** - it confirms documented `isempty()` behaviour, which is why
      MSD-003 and MSD-004 test the string form instead.
- [ ] **For every Defender XDR query you convert into a custom detection rule, check the query's
      window against the lookback the rule frequency gives it.** Microsoft's custom-detection-rules
      page says not to filter on `Timestamp` or `TimeGenerated`, states that the service prefilters on
      the detection lookback, permits filtering for narrowing inside that window, fixes the lookback
      from the frequency for Defender data, and states that results outside the lookback are ignored.
      MSD-001 records the wording. **The case to catch is a query whose baseline leg is wider than the
      lookback**: MSD-003's and MSD-004's change-detection variants both baseline reaching back 30
      days, and 30 days is the lookback only at the every-24-hours frequency. **The two variants fail
      in opposite directions, and only one of the two failures is visible.** MSD-003 excludes its
      baseline with `join kind=leftanti`, so a truncated baseline stops excluding and the rule
      over-reports, turning a change rule into a census. MSD-004 includes its baseline instead, with
      `where AgentId in (HadGuardrails)`, so a truncated baseline **shrinks the inclusion list**: an
      agent whose guardrail evidence fell outside the surviving slice drops out of the list, a
      genuine guardrail removal on that agent is never reported, and the rule goes quiet and looks
      healthy. **Check MSD-004's direction even though its symptom is silence rather than noise**,
      because a rule that reports nothing is the failure this pack exists to prevent.
      **Truncation is not the only thing that shrinks that inclusion list, and this step checks
      only truncation.** MSD-004's baseline leg collapses to one row per agent before it filters,
      so `HadGuardrails` holds each agent's last pre-window state rather than every state it held,
      and the list shrinks that way at every frequency with no truncation involved. MSD-004 sets
      that out under its change-detection query and carries a verification step for it. **Passing
      this step is not evidence against that one.** The page
      also says custom detections evaluate `ingestion_time()` rather than event timestamps, and
      **under its own Rule frequency heading it says the opposite-looking thing about a different
      object**: "The rule frequency is based on the event timestamp and not the ingestion time." Both
      sentences are recorded in MSD-001 rather than reconciled here, and **the interaction is not
      settled on the page and nothing here has measured it** - run the variant at
      the frequency you intend and compare its row count against the same query run interactively.
      **Compare on the second scheduled run or later.** The same page states that when you save a new
      rule it runs and checks for matches from the past 30 days, and that the rule then runs again at
      fixed intervals applying the lookback its frequency gives it. So the save-run covers 30 days
      whatever frequency you chose, the mapped lookback applies only from the second run, and a
      first-run comparison agrees with the interactive query and shows no truncation even when the
      rule will truncate every run afterwards.
      **A first scheduled run that generates exactly 150 alerts is a ceiling rather than a count**:
      the same page, read 2026-08-18, states that "each rule can generate only 150 alerts each time
      it runs". **That cap is on alerts and the comparison above is on rows**, and the page puts a
      deduplication step between the two, from that same read: "Custom detections group and
      deduplicate events into a single alert", and "Duplicates can occur when the lookback period is
      longer than the frequency." So a rule can return far more than 150 rows and still generate 150
      alerts, and an alert count sitting at the cap says nothing about the row count. Compare the
      rows, and read a saturated alert count as a reason to look at the baseline rather than as the
      truncated-baseline result itself.
- [ ] **If you build a Sentinel-only variant as a Defender-portal custom detection rather than as a
      Sentinel analytics rule, check it against the other lookback table.** The same page publishes a
      second, customisable mapping for detections that "target Microsoft Sentinel data only": "less
      than 48 hours" at frequencies "higher (more frequent) than one hour", "up to 14 days" at
      frequencies "higher than one day", and "up to 30 days" at "one day or less". **Two baselines in
      this pack reach 31 days and so exceed every value in that mapping**: MSD-006's change-detection
      baseline and MSD-007's access-pattern view, which ship the identical baseline construct
      `between (ago(31d) .. ago(1d))`. **Narrowing to 30 days is enough only at "one day or less"**;
      at a frequency higher than one day the ceiling is 14 days, and above hourly it is under 48
      hours, so the fix depends on the frequency you pick. **Match the baseline to the frequency you
      intend, not to the widest figure on the page.** Both files state a Microsoft Sentinel
      deployment target, where this does not arise.
- [ ] Every detection has a documented owner and a triage path.
- [ ] Baselines are recorded for the three posture and inventory queries (MSD-003, MSD-004,
      MSD-006), so the first alert is a change rather than a census.
- [ ] The last-verified dates in the detection files have been re-checked against the live Learn
      pages, and any drift is recorded.

---

## Recording the result

For each detection, record one of:

| Outcome | Meaning |
|---|---|
| **Verified** | Query ran, schema matched, a positive control was observed |
| **Runs, unconfirmed** | Query ran without error but no positive control was observed |
| **Blocked** | Table absent, connector absent, or plan not enabled |
| **Schema mismatch** | The output is missing a column the detection file records, types one differently, or emits a value outside a documented value set the file reproduces, **in either case where the published page no longer carries what the file records** - **file a correction**. Where the page still carries it and your output does not, that is a **tenant availability difference** and a **discovery**, not a mismatch: see the discriminator below |

**A "Schema mismatch" outcome is the most valuable result this checklist can produce**, because it
means a citation in this pack is stale or wrong, and correcting it is worth more than any of the
queries. **Not every divergence is one**: a column or value the page still publishes and your
tenant does not carry is a difference between tenants, not a defect in a citation.

### Correction or discovery - which route a divergence takes

**A value from a column this pack records no value set for is a discovery rather than a mismatch.**

**And so is a value beyond a set this pack reproduces accurately. Record it as a discovery, report
it, and widen your own filter.** Added 2026-09-19, because the row above over-reached as written. A
filter built from the published list under-matches an environment emitting values outside it, and
that failure is a silent false negative.

**The discriminator, stated once so the two routes cannot be confused.** The Schema mismatch route
exists to catch a citation that is stale or wrong. **A value outside a reproduction is a correction
where the published page has since changed, and a discovery where the page still publishes exactly
what the file reproduces.** **You establish which by re-reading the page, not by assuming either.**
One worked case each way is already in this pack: a tenant emitted a `DeliveryLocation` value
the reproduced list does not carry while a re-read confirmed the page unchanged, so that was a
discovery; and `Title` on `BehaviorInfo` was absent from one tenant's schema while the page still
published it, which was a tenant availability difference and not a stale citation either.

**The alert identifiers are a separate case with their own ground, and it is not derived from the
discriminator.** The 17 `AI.` identifiers are a closed set published under a heading that
enumerates them, so an eighteenth is a page-versus-file divergence to check rather than a value your
environment invented. **Re-read the alerts page before filing**: if the page now carries it, the
reproduction is stale and a correction is owed; if it does not, record it as a discovery like any
other.

**Read the rest of this as a test rather than as a list**, because a list here goes stale the moment a
detection file gains or drops a reproduction. The value clause above reaches a column exactly when a
detection file in this pack reproduces a value set Microsoft publishes for it **in full**. Today
that is `DeliveryAction`,
`DeliveryLocation` and `EmailDirection` for the email pair, `LifecycleStatus` and
`PublishedStatus` for the agent inventory pair, and **the 17 `AI.` alert identifiers MSD-005
reproduces from Microsoft's published alerts page** - the largest such set in this pack, and the one
Group 5 above already sends you to reconcile. **The 17 names beside them are not the same case**:
the same page carries one further entry that publishes a name and no `AI.` identifier, so the name
side reproduces every name that comes with an identifier rather than every name the page publishes.

**`ConfidenceLevel` is deliberately not on that list, and the reason is worth stating.** MSD-001
reproduces that column's description with the pack's own ellipsis and keeps only the phishing half,
`High` and `Low`. Microsoft publishes a second value set on the same column, keyed to the verdict
type, and this pack does not reproduce it. **So a `ConfidenceLevel` value your workspace emits that
is not `High` or `Low` may well be a documented Microsoft value**, and routing it as a schema
mismatch would send you to the correction lane, which asks for more of your own environment than the
discovery lane does. Record it as a discovery. If you want the elision itself closed, a correction
against MSD-001's own reproduction is the useful contribution, and it is a different issue from a
value your workspace emitted.

**An alert identifier beginning `AI.` that your workspace emits and that is not among those 17 is
the page-versus-file case described above: re-read the alerts page, file a correction if the page
now carries it, and record a discovery if it does not.** **On the name side
there is no prefix to key on, so the criterion is the one Group 5's discovery step already applies**:
an alert name that step surfaces and that is not among the 17 is the same case and takes the same
re-read. **Learn
describes `SecurityAlert` as carrying alerts generated by security products, which MSD-005
reproduces verbatim**, so a value produced by a product other than the one this rule targets is
neither a mismatch nor a gap, **and it is not a value to send**: a name carrying another product
discloses which security products your estate runs, which is your estate's posture rather than a
schema fact. **That carve-out reaches the name side as much as the identifier
side**. **A custom analytics-rule name your own workspace emits is your estate's naming rather than
another plan's**, and the carve-out phrased in terms of plans does not by itself reach it: it is
neither a mismatch nor a gap, and it is not a value to send. `docs/scope-and-out-of-scope.md` states
that rule, and what MSD-005's verification step 2 returns is a shape to describe rather than a list
to send. That is what the quoted description supports; no page this pack read states the wider form,
that the table carries every Defender plan's alerts. It has to reach the name side: Group
5's discovery step keys its second leg on a name fragment rather than on a plan, so a name another
plan uses will surface there, and MSD-005 records four of the seventeen names as phrasings it reads
as generic across Defender plans, marking that as its own reading rather than as a cited statement. **Confirm the plan before routing a name as a mismatch.** Group 5 above and
MSD-005 both record one further entry on the same Microsoft page with no published `AI.` identifier
and say in terms that it is not a gap in the table. **The 17 names and
the 17 identifiers are one set covering a pair of columns**, because Learn does not state which
`SecurityAlert` column receives these values and MSD-005 matches on both `AlertName` and `AlertType`
for that reason. **This routing applies once Group 5's step has settled which of the two your own
workspace fills**, and until then a value outside the set is a discovery.

### What may travel outward, and what stays in your environment

Where the pack records no value list at all, the opposite applies: that absence is by design and
marked as such, so no value your workspace emits for those columns can contradict a citation.
`Availability`, `ConditionalAccessStatus`, `ResultType`, `BehaviorInfo.ActionType`,
`CopilotActivity.RecordType`, `CopilotActivity.Workload`, `CopilotActivity.AppHost`, `McpServers`
and the `DetectionMethods` serialisation are in that class. **Apply the test above to any column not
named here rather than reading either list as closed.** These belong in the verification report's
**Undocumented elements you resolved** table, which is the contribution the pack most wants, and not
in a correction issue. **Having no documented value set does not relax a column's handling rule.**
`McpServers` is in this class and is still the column Group 3 above tells you never to paste the
contents of: what travels from it is the field names and the empty forms your platforms emit, never
a value. **`Availability` is in this class too, and the same caution reaches it before any value
travels.** Learn's description covers a specific-groups case and does not publish the serialisation,
so a platform that serialises that case by naming the groups puts your own organisation's naming in
the column. **Send the shape you found. A value travels only if you have looked at what it contains
and it carries nothing of your own organisation's; if it carries anything of yours, the shape
travels and the value does not.** **`CopilotActivity.AppHost` takes the same condition, stated once
above rather than repeated here.** Learn describes it as the application that hosts copilot and
gives it neither an example nor a value set, so whose applications it names is not something the
page settles either way, and the condition is what this pack applies to a column in that state.
**Read that condition's trigger widely here, because this column's exposure has a different shape.**
A value here names an application your estate runs, whoever chose the name, so it can carry none of
your own organisation's naming and still say something about your estate; a set of such values
states part of what your estate runs. **So a value from this column goes out only where it names
nothing specific to your estate, and where it does, the shape goes out and the value stays with
you.**

**What makes a value specific, put as a test rather than left to judgement.** A value is **not**
specific where it names a product Microsoft ships that anyone could be running: the name tells a
reader nothing about your estate that the product's existence does not already tell them. A value
**is** specific where the name was chosen inside your organisation or by a supplier for you, or where
it carries an internal product, team or project word - in short, where a reader who did not already
know your estate could learn something about it from the value alone. **Where you cannot decide, the
shape goes and the value stays**, because that direction costs a contribution and the other direction
costs a disclosure.

**The set takes its own rule, because what it carries is not the sum of its members.** A complete
inventory of the applications hosting Copilot in your tenant states part of what your estate runs
even where every member of it passes the test above, and Group 7 below and MSD-007 verification step
2 both produce exactly that inventory. **The inventory stays in your environment.** What may travel
is an individual value that passes the per-value test, sent because the schema emits it and the next
reader needs to know the value exists, rather than the list of everything your tenant emits. This
pack routes comparable inventories the same way: MSD-006's `AppId` mapping and its authentication
list stay in your environment, and Group 6 above gives the reason in the same terms.

**The rule bars the set and permits a member, and the middle is where it is easiest to walk past.**
**Sending the inventory a value at a time is sending the inventory**, whatever each send looks like
on its own, so the rule reaches a subset as much as the whole: send an exemplar because the value
exists, not a run of them that adds up to the list. **What a maintainer may ask for instead** is the
count-free shape of what you found, which value strings a given `RecordType` puts there, or a single
value that is in question, and anything wider goes through the private route in `SECURITY.md` rather
than through a public issue. The purpose clause above is the reason for that and this sentence is the
bound.

### Free-text columns, where the default does not reach

**The rule behind that, stated once rather than applied column by column.** Group 0's default, that
what travels outward is a value name or a column name and never what it matched, holds where a column
carries a closed set the platform defines. **Where a column carries free text, the value is itself
estate data and the default does not reach it**, because there is no value name to send that is not
also the content. The free-text columns this pack touches include `BehaviorInfo.Description`,
`BehaviorInfo.Title` where a tenant carries it, the `SampleLabels` rollup built from whichever of
those two that tenant has, `CopilotActivity.LLMEventData`, `AgentsInfo.McpServers`,
`EmailEvents.Subject`, `AADServicePrincipalSignInLogs.Agent`, `AgentsInfo.AgentName` and
`CopilotActivity.AgentName`. **The two agent-name columns are listed separately because they are
different columns that share a name**: MSD-003 and MSD-004 return the one in `AgentsInfo`, MSD-007
returns the one in `CopilotActivity`, and all three reach the same classification by the test above,
Microsoft publishing no value set for either. **`Title` is on this list because
MSD-008's step 1 names it behind `column_ifexists()`**, so an operator whose tenant carries the
column gets that free text in `SampleLabels` rather than nothing. **For those, what travels is the column name and the shape you
found, never the contents.** Having no documented value set is what puts a column into the routing
test above; it is not what exempts it from this rule. **Apply this to any free-text column not named
here**, on the same reasoning rather than by looking for it in this list.
