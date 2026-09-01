# What the callers actually ask this check for

Taken on 1 September 2026 against the roster in `iderex/operations` at
`origin/main` `913126ca2978025e262a7c391025d823b9fe40a7`. Every figure below is
the output of the command written beside it. The figures move every time a board
edits its caller, so re-run the command rather than citing this page.

`#27` reads the boards that call this check and keep a gate of their own, and
every reading it holds so far has read the LOCAL copies. None of them read the
CALLER. This is that reading, and it changes what the duplication on those
boards is.

Beside it, `#27`'s own last not-evaluated line asks for the local rules of the
two boards its 31 August sweep added to the removal set, `iderex/pruefstand` and
`iderex/messstube`, which no reading here has ever had. That is the second half
of this page.

## The callers, derived by content rather than by filename

Every earlier sweep here asked each board for `.github/workflows/shared-hygiene.yml`
and declared the filename as its own bound. `docs/local-hygiene-answers.md`
states it: "How many boards call this check from a file not named
`shared-hygiene.yml`. This board does, from its own `hygiene.yml` ... so nineteen
is a floor on the callers outside this board and not a total." A search over
content rather than over names does not carry that bound:

```
gh search code 'wache/.github/workflows/pr-hygiene.yml' --json repository,path \
  --limit 100 --jq '.[] | "\(.repository.nameWithOwner)\t\(.path)"' | sort -u
```

Nineteen boards outside this one answer, every one of them from
`.github/workflows/shared-hygiene.yml`, and the only other hits are this board's
own `README.md`, `docs/unicode-guard-copies.md` and `calling-examples.yml`. So
the count that was a floor is the same number read a second way, and the
filename is no longer what the reading rests on.

WHAT BOUNDS THIS ONE INSTEAD is the index. A code search reads what the platform
has indexed on a default branch, and neither the lag nor the completeness of that
index is measured here. It does reach private boards: 40 of the 74 roster records
declare `Visibility: private` and two of the nineteen answers,
`iderex/messstube` and `iderex/pruefstand`, are among them.

```
git -C <operations> grep -h '^Visibility:' origin/main -- store/repo/ | sort | uniq -c
     40 Visibility: private
     34 Visibility: public
```

## Eighteen of the nineteen turn off the rule this check is mostly about

```
for b in <the nineteen>; do
  gh api "repos/$b/contents/.github/workflows/shared-hygiene.yml" --jq '.content' | base64 -d |
    awk -v b="$b" '/pr-hygiene\.yml@/{if(!p){match($0,/@[0-9a-f]+/); p=substr($0,RSTART+1,8)}}
                   /^ *subject_names_issue:/{s=$2}
                   END{printf "%-24s %s  subject_names_issue=%s\n", b, p, (s==""?"<not passed>":s)}'
done
```

```
Flowfin/core             113085b2  subject_names_issue=false
iderex/ausgleich         9b311243  subject_names_issue=false
iderex/bremsweg          113085b2  subject_names_issue=false
iderex/gutachten         9b311243  subject_names_issue=false
iderex/hallraum          9b311243  subject_names_issue=false
iderex/hoersaal          9b311243  subject_names_issue=<not passed>
iderex/kanzlei           9b311243  subject_names_issue=false
iderex/kontor            9b311243  subject_names_issue=false
iderex/lesesaal          9b311243  subject_names_issue=false
iderex/lichttisch        9b311243  subject_names_issue=false
iderex/messstube         113085b2  subject_names_issue=false
iderex/nachtwache        9b311243  subject_names_issue=false
iderex/plattenschrank    9b311243  subject_names_issue=false
iderex/pruefstand        9b311243  subject_names_issue=false
iderex/rechenstrasse     9b311243  subject_names_issue=false
iderex/schallweg         113085b2  subject_names_issue=false
iderex/spurenarchiv      113085b2  subject_names_issue=false
iderex/stammtisch        9b311243  subject_names_issue=false
iderex/sternwarte        113085b2  subject_names_issue=false
```

`iderex/hoersaal` is the only board of the nineteen that leaves the input alone
and so takes the default, which is `true`. This board is the twentieth caller,
from its own `.github/workflows/hygiene.yml`, and it leaves the input alone too.

NO CALLER PASSES ANY OTHER INPUT. The same files were read for the four
remaining names and none of them appears in any of the nineteen:

```
grep -l 'subject_exempt_authors\|outside_contributions_exempt\|message_is_ascii\|resolve_referenced_numbers' <the nineteen caller files>
```

That is empty. The three inputs `#35` carried in so that a board could keep its
own answer while deleting its copy are set by nobody, which is expected while no
copy has been deleted; the point of listing them here is that the fourth,
`resolve_referenced_numbers`, is off everywhere too, so the second job of this
workflow has never been created on any board but this one.

## What that leaves the shared route judging, read on a runner

With `subject_names_issue` false and `message_is_ascii` unset, the commit range
is never walked. The guard is one line:

```
git show origin/main:.github/workflows/pr-hygiene.yml | sed -n '395p'
          if [ "$subject_rule" = "true" ] || [ "$ASCII_RULE" = "true" ]; then
```

That is the file. The run says the same thing on a real runner, on `Flowfin/core`
on 31 August 2026:

```
gh run view 33433087710 --repo Flowfin/core --log | grep 'the subject rule is off' | tail -1
hygiene / Deterministic PR hygiene	UNKNOWN STEP	2026-08-31T19:54:21.4846607Z the subject rule is off for this repository, so no commit subject was read
```

That run pins `113085b2`, so the text it prints is that tag's and not
`origin/main`'s. The line quoted is the same on both; the run's closing line is
worded differently there and is not quoted for that reason.

So on eighteen of the nineteen boards this check judges three things and no
more: an empty body, a closing keyword that is not the whole of its own line,
and a placeholder title. Nothing it does reads a commit, an author address, a
changed path or a branch name.

I extracted the body judgement out of the file and ran it rather than reading it,
because "refuses nothing" is the claim that is easiest to get wrong by reading:

```
sed -n '238,272p' .github/workflows/pr-hygiene.yml | sed 's/^          //' > refusals.sh
refusals "Read what the boards hold [#24]" "I changed the reader so it stops on the first failure."
```

It prints nothing. A body that names no issue at all is not refused; what is
refused is an EMPTY body and a MALFORMED closing line.

## What that costs the removal sequence, which is `#27`'s third done-condition

`#27` opens on a board where "a pull request is judged twice, by two
implementations that agree on nothing, and a contributor who satisfies one can be
refused by the other for a rule the first does not have". On eighteen of the
nineteen boards the second implementation is three rules about a title and a
body. The double judgement is real and it is much smaller than the sentence
describes.

The direction that matters is the other one. On those eighteen boards the LOCAL
copy is the only thing that reads a commit at all, so the sequence written in
`README.md` - pin, read the copy, set whichever input it answers, delete - leaves
a board with no commit judgement whatsoever unless it also turns
`subject_names_issue` back on, and the sequence has no step that says so. A board
following it exactly ends with three rules about its body and its title where it
had a gate over its commits.

THIS IS NOT AN ARGUMENT FOR FLIPPING THE INPUT ON THOSE BOARDS. `README.md`
already says what switching it off means - "a statement that the board does not
meet the rule yet" - and eighteen boards have made that statement about
themselves. What is missing is that the statement was made while a local gate was
still standing, and the delete is the moment it stops being harmless. Whether any
given board should flip it is that board's, and this names none.

The step is added to `README.md` in the same change as this page.

## The two boards the sweep added late, read one rule at a time

`#27`'s first done-condition asks that every local copy be read "and anything it
answers that the shared check does not is either raised here as an issue or
carried into the shared check first". The 31 August sweep put two boards into the
removal set after every reading that met that line had already run. Both are read
here.

### `iderex/pruefstand`

Its workflow carries no rules; it is 57 lines around one command, and the rules
are 642 lines of `hygiene.py` at the root of that tree.

```
gh api repos/iderex/pruefstand/contents/hygiene.py --jq '.content' | base64 -d | wc -l
642
```

Five conditions, named by that file's own constants:

- **`names-an-issue`.** Refused when neither any commit message nor any line of
  the body names `#N`. This is WEAKER than the subject rule here, which asks it
  of every non-merge commit subject separately - but that rule is off at this
  board's caller, so on that board this is the only form of the question that
  runs at all.
- **`changed-paths-inside-scope`.** The `Scope:` path comparison, already placed:
  `docs/local-hygiene-answers.md` counts five boards holding it and names this
  one, and the ruling that keeps it off a shared route is `#36`. Nothing here
  reopens it.
- **`means-recorded`.** REFUSED when a change ADDS a dependency manifest, a file
  whose suffix the tree did not already carry, or a workflow, and no line of the
  body of at least forty characters carries the word "means". This board's check
  has no form of it: nothing here reads a changed path, and the only thing it
  reads out of a body is emptiness, a closing line and a placeholder title.
- **`record-added-not-modified`** and **`published-number-carries-its-record`.**
  Both are keyed to that tree's own `results/` and `report/` directories: a file
  under `results/` arriving with any status other than `A` is refused, and a
  changed artefact under `report/` with no newly added record beside it is
  refused. Those are that board's domain rather than an answer about pull
  requests in general.

### `iderex/messstube`

185 lines, inline, with its rules in two tiers.

```
gh api repos/iderex/messstube/contents/.github/workflows/pr-hygiene.yml --jq '.content' |
  base64 -d | wc -l
185
```

- **The body names the issue it closes.** FAIL tier, matched case-insensitively
  anywhere in the body as `(close[sd]?|fix(e[sd])?|resolve[sd]?)\s+#N`. This
  check refuses an EMPTY body and a MALFORMED closing line and has no form of
  "there is no closing reference at all", which is the run above rather than a
  reading. With `subject_names_issue: false` at this board's caller, deleting
  this file takes away the last thing on that board asking a change to name an
  issue anywhere.
- **A changed reader crate moves its tests, or the body says why not.** FAIL
  tier, over `crates/readers/<name>/`, satisfied by a file under that crate's
  `tests/`, by an added `#[test]`, or by a body line reading
  `No test change for <crate>: <reason>`. That board's own domain.
- **A version bump carries a changelog entry.** FAIL tier, on an added
  `version = "` line inside a `Cargo.toml` only, with the exclusion of
  `Cargo.lock` written beside it and the reason given. It is the same SHAPE as
  `iderex/pruefstand`'s `published-number-carries-its-record` - a number moving
  owes the record that explains it - in a second vocabulary. Two boards hold that
  shape; nothing here proposes a form for it.
- **A 400-line reading cap that warns and never fails.** WARN tier.
  `iderex/gutachten`'s warn-without-refusing tier was placed on `#27` as
  something "no board has asked for"; this is a SECOND board holding one, so that
  count is two rather than one. It remains a change to what a verdict is here
  rather than an answer beside the ones already carried, and nothing here asks
  for it.

### Neither of the two adds to the stranded-check trap

`README.md` names `iderex/hoersaal` and `iderex/stammtisch` as the two boards
whose ruleset requires the context their local file produces. Both new boards
were read the same way and neither joins them:

```
gh api repos/iderex/pruefstand/rulesets --jq '.[] | "\(.id) \(.name) \(.enforcement)"'
20521019 gate active
gh api repos/iderex/pruefstand/rulesets/20521019 --jq '[.rules[]? |
  select(.type=="required_status_checks") | .parameters.required_status_checks[]?.context] | join(",")'

gh api repos/iderex/messstube/rulesets/20523268 --jq '[.rules[]? |
  select(.type=="required_status_checks") | .parameters.required_status_checks[]?.context] | join(",")'
local-gate
```

`iderex/pruefstand` carries no `required_status_checks` rule at all, and
`iderex/messstube` requires `local-gate` and nothing else. Neither board is
protected by classic branch protection either, so the ruleset is the whole
answer:

```
gh api repos/iderex/messstube/branches/main/protection/required_status_checks
{"message":"Branch not protected", ... "status":"404"}
```

THE NEAR MISS IS WORTH THE LINE ANYWAY, because `iderex/messstube`'s job is
named `Deterministic PR-hygiene checks` - the exact string `iderex/stammtisch`'s
ruleset requires - and the comment above that job says the name "is the literal
check-run name the branch ruleset's required set references". Its ruleset does
not reference it. `iderex/pruefstand`'s job carries the same name. So the string
that is required on one board is produced on three, and a reader who checks the
NAME instead of the RULESET gets the wrong answer in both directions. The command
in `README.md` is the one to run, per board.

## The `#59` count, re-derived against a roster that has grown

`docs/local-hygiene-answers.md` counts the three answers `#59` holds over 73
boards at `iderex/operations` `a320756a`. The roster is 74 now, and one board
arrived:

```
comm -13 <roster at a320756a> <roster at 913126ca>
iderex/steinbruch
```

It holds one workflow file, 20 lines, and no pull-request hygiene gate:

```
gh api repos/iderex/steinbruch/contents/.github/workflows --jq '.[].name'
ci.yml
```

Neither `iderex/pruefstand` nor `iderex/messstube` holds any of the three
either, searched by behaviour as that page searched:

```
grep -inE 'default_branch|defaultBranch|default branch' <both files>
grep -inE 'generated|DO NOT EDIT|autogen' <both files>
grep -inE 'no body|subject line and nothing|format=%B' <both files>
```

The default-branch search is empty in both. `iderex/messstube` matches
"generated" once, in a comment listing what legitimately exceeds the churn cap,
which is not a refusal. `iderex/pruefstand` matches `format=%B` once, where it
reads whole commit messages to find issue references and not to ask whether a
body exists. So the counts that ruling rests on are unmoved: one board, one
board, three boards.

## Not evaluated

Whether the eighteen boards passing `subject_names_issue: false` intend it as a
permanent statement or as a step. I read the value and the comment beside it
where there was one, and asked no board.

Whether any of these gates is failing today. One run was read, on `Flowfin/core`,
and it was read for what the shared route SKIPS rather than for a verdict. Every
other statement here is off files, trees and rulesets.

Whether the eleven boards whose rules live in their own workflow file hold
anything further. That half still rests on the reading of 26 August and I did not
re-do it.

What the seven hygiene gates found under another filename on boards that do NOT
call this check hold. They are outside the removal set and `#27`'s first
done-condition does not reach them.

Whether the code search above sees every caller. It is an index rather than a
walk of every file, and its lag is not measured here. It agrees with the
filename sweep on nineteen, which is two methods agreeing and not a proof.
