# What the callers actually ask this check for

Taken on 1 September 2026 against the roster in `iderex/operations` at
`origin/main` `913126ca2978025e262a7c391025d823b9fe40a7`. Every figure below is
the output of the command written beside it. The figures move every time a board
edits its caller, so re-run the command rather than citing this page.

THE FIGURES HAVE BEEN RE-TAKEN SINCE, AND `Re-read on 4 September 2026` NEAR
THE END IS WHERE THAT IS RECORDED. Every figure in the sections before it is 1
September's and is left as it was taken.

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
git show origin/main:.github/workflows/pr-hygiene.yml | grep '|| \[ "$ASCII_RULE" = "true" \]'
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
awk '/^          refusals\(\) \{/,/^          \}/' .github/workflows/pr-hygiene.yml |
  sed 's/^          //' > refusals.sh
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

## Re-read on 4 September 2026: the walk the index could not do, and what it found

Taken against the roster in `iderex/operations` at `origin/main`
`3dbef1117780a0cb202dd4362d5bb27b56778589`. This page closes with the bound that
the code search above is an index rather than a walk of every file, and with
`docs/local-hygiene-answers.md`'s older form of the same thing: "I did not fetch
all 902 workflow files to close it, so nineteen is a floor on the callers outside
this board and not a total." I fetched them. Nineteen is the total, and the index
is measurably not.

```
boards | while read -r r; do
  gh api "repos/$r/contents/.github/workflows" --jq '.[] | "\(.name)\t\(.sha)"' 2>/dev/null |
    while IFS="$(printf '\t')" read -r n s; do printf '%s\t%s\t%s\n' "$r" "$n" "$s"; done
done > wf.tsv

cut -f1 wf.tsv | sort -u | wc -l
73
wc -l < wf.tsv
924
```

73 of the 74 roster boards hold a `.github/workflows` directory and 924 files sit
in them. `iderex/lagetisch` holds no such directory and is outside this reading.
Every distinct blob was fetched and hash-checked against the id the listing gave
for it, so the bytes read below are the bytes tracked; `docs/dco-tail.md`'s 4
September section carries that check and its output.

## Every call site, over every file

```
while IFS="$(printf '\t')" read -r r n s; do
  grep -nE 'uses:.*(wache/\.github/workflows|\./\.github/workflows)' "blobs/$s.txt" |
    while read -r l; do printf '%s\t%s\t%s\n' "$r" "$n" "$l"; done
done < wf.tsv | sort -u | grep -c 'pr-hygiene\.yml'
20
```

Twenty call sites. Nineteen are boards other than this one, every one of them in
a file named `.github/workflows/shared-hygiene.yml`, and the twentieth is this
board's own `hygiene.yml` calling the file beside it by relative path. So the
nineteen that three readings carried as a floor is a total over the roster's
whole workflow corpus, and the file name was not hiding a caller after all.

THE INDEX MISSES TWO OF THE TWENTY, which is worth more than the confirmation is.
The same search this page ran on 1 September, run again tonight:

```
gh search code 'wache/.github/workflows/pr-hygiene.yml' --json repository,path \
  --limit 100 --jq '.[] | "\(.repository.nameWithOwner)\t\(.path)"' | sort -u |
  grep -c 'shared-hygiene.yml'
18
```

Eighteen where the walk finds nineteen. `iderex/stammtisch`'s caller is absent
from the result and present in its tree, and this board's own `hygiene.yml` is
absent too. The 1 September reading called the two methods agreeing on nineteen
"two methods agreeing and not a proof", and that was right in the direction it
mattered: they do not agree today, and the walk is the one to believe. A count
taken off that index is a floor, and the sentence above saying the filename bound
was gone rested on the weaker of the two methods.

## Nothing has been removed and no pin has moved

```
awk -F'\t' '$2=="pr-hygiene.yml"{print $1}' wf.tsv | sort > has-local
awk -F'\t' '$2=="shared-hygiene.yml"{print $1}' wf.tsv | sort > has-shared
wc -l < has-local ; wc -l < has-shared ; comm -12 has-local has-shared | wc -l
34
19
17
```

Seventeen boards still keep a `pr-hygiene.yml` beside the call,
`iderex/pruefstand` still keeps its gate under `hygiene.yml`, and
`iderex/lesesaal` is still the one caller with neither. Eighteen of nineteen,
exactly as the 31 August sweep found it. Not one copy has been removed.

THE THIRTY-FOUR INCLUDES THIS BOARD'S SOURCE FILE, and no reading on the hygiene
side has said so. `#27` opens on "the 34 local copies of `pr-hygiene.yml` across
the roster", and one of the 34 is the shared workflow itself:

```
awk -F'\t' '$1=="iderex/wache" && $2=="pr-hygiene.yml"{print $3}' wf.tsv
bacc56abe9b55e804ad2d6b1129eee3aa8b1fa2d
git rev-parse origin/main:.github/workflows/pr-hygiene.yml
bacc56abe9b55e804ad2d6b1129eee3aa8b1fa2d
```

That is the same confound `docs/dco-tail.md` names for `iderex/wache` on the DCO
side, arriving here unremarked. There are 33 copies and one source, and 33
distinct contents among the copies rather than 34.

The pins have not moved either, and no caller file has been touched since 27
August:

```
grep -oE '@[0-9a-f]{40}' <the twenty call sites> | sort | uniq -c
      6 @113085b269d3437a3f96ff9e7060b64b0af88ab1
     13 @9b311243c2d0d0ced7feb957a20bc178acce6a5d

while read -r r; do
  printf '%s\t%s\n' "$r" \
    "$(gh api "repos/$r/commits?path=.github/workflows/shared-hygiene.yml&per_page=100" \
         --jq '[.[].commit.committer.date] | first')"
done < has-shared | sort -t"$(printf '\t')" -k2,2r | head -2
iderex/messstube	2026-08-27T14:00:40Z
iderex/pruefstand	2026-08-27T06:38:21Z
```

Thirteen of the nineteen are below the `v1.2.0` floor `#27`'s second
done-condition line names, and the newest caller file on any board is eight days
old. The sequence landed on 27 August, gained its ruleset step on 31 August and
its subject-rule step on 1 September, and no board has executed a step of it.

## The three carried answers reach no board

`#35` carried `subject_exempt_authors`, `outside_contributions_exempt` and
`message_is_ascii` into the shared check on 27 August, so that a board deleting
its copy would not delete its answer. A caller can pass an input only if the ref
it pins declares it:

```
for ref in 9b311243 113085b2 4d91113f origin/main; do
  git show "$ref:.github/workflows/pr-hygiene.yml" |
    sed -n '/^ *inputs:/,/^jobs:/p' | grep -E '^      [a-z_]+:' | tr -d ' :' | paste -sd' ' -
done
subject_names_issue
subject_names_issue
subject_names_issue subject_exempt_authors outside_contributions_exempt message_is_ascii
subject_names_issue subject_exempt_authors outside_contributions_exempt message_is_ascii body_names_issue body_exempt_authors resolve_referenced_numbers issue_read_token
```

`9b311243` is `v1.0.0` and `113085b2` is `v1.2.0`, and every one of the nineteen
callers pins one or the other. So all three inputs are on the mainline, all three
are in `v1.3.0`, and not one of them is reachable from any calling board eight
days after they landed. The first done-condition line is met on this board's side
and its repair has arrived nowhere.

## One board met the cancellation and repaired it the other way

The second done-condition line asks that the pins reach `v1.2.0` before any copy
is removed, "so no board is left with the cancellation and no local gate". On 2
September `iderex/stammtisch` took the cancellation off its own board without
touching its pin, by namespacing the group its local copy declares:

```
gh api "repos/iderex/stammtisch/commits?path=.github/workflows/pr-hygiene.yml&per_page=3" \
  --jq '.[] | "\(.commit.committer.date)  \(.sha[0:7])  \(.commit.message | split("\n")[0])"'
2026-09-02T05:33:14Z  5abc17e  Stop the shared hygiene call from cancelling this board's own hygiene run
```

The diff replaces `pr-hygiene-${{ github.event.pull_request.number }}` with
`${{ github.workflow }}-${{ github.event.pull_request.number }}`, and the comment
it adds states the mechanism `#4` measured here, reached independently on that
board. Its pin is still `9b311243`, which is `v1.0.0`.

WHAT THAT DOES TO THE SEQUENCE IS NOT NOTHING. The floor in the second line is a
proxy: what it protects against is the shared run cancelling the local one, and
the pin is how the sequence here proposes to remove it. A board that has removed
the hazard by the other route satisfies the line's purpose and fails its wording,
and the sequence in `README.md` gives a reader no way to tell those two states
apart. Whether that wording should read the group rather than the tag is a
question this page does not take.

## What is unchanged

The removals are other boards' trees, one board at a time, pins at or above
`v1.2.0` first, `iderex/lesesaal` left alone. Nothing here lands any of them, and
nothing has landed anywhere.

The stranded required check is still on two boards and nowhere else, re-read
tonight across the whole removal set:

```
while read -r b; do
  gh api "repos/$b/rulesets" --jq '.[].id' | while read -r id; do
    gh api "repos/$b/rulesets/$id" --jq '[.rules[]? |
      select(.type=="required_status_checks") |
      .parameters.required_status_checks[]?.context] | join(",")'
  done
done < <(cat removal-set; echo iderex/pruefstand)
```

`iderex/hoersaal` requires `pr-hygiene` and `iderex/stammtisch` requires
`Deterministic PR-hygiene checks`. Three boards of the set require `local-gate`
and nothing a hygiene file produces, and the remaining thirteen require no status
check at all.

`iderex/hoersaal` is also still the only caller of the nineteen that leaves
`subject_names_issue` alone and takes the default of `true`. It is the board with
a required context only its own copy produces, and the board whose commits the
shared route actually reads, and it is one board.

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

The last of those five is answered by the 4 September section above and the
answer is not the reassuring one: the index does NOT see every caller, it missed
two of the twenty call sites on the night it was re-run, and the walk over all
924 workflow files is what nineteen now rests on.

Three bounds are that walk's own. Every listing and every blob comes from each
board's DEFAULT branch, so a caller living only on another branch is outside it.
The 924 files are what the 73 directories held at the moment they were listed.
And a call reaches this check only through `uses:`, which is what the grep looks
for; a board invoking the shared rules some other way would not appear, and I
know of none that does.

Whether `iderex/stammtisch`'s own claim about its runs holds. Its comment says
every run of its Shared hygiene workflow cancelled the local one. I read the two
workflows' run listings on that board rather than their jobs, and what those show
is seven Shared hygiene runs all concluding `success` and one local run
concluding `cancelled` in the same minute as one of them. That is one
coincidence, not a per-run audit, and it neither confirms nor refutes the claim.
