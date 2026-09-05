# The three answers of `#59`, counted across the roster

Taken on 31 August 2026 against the roster in `iderex/operations`, at
`origin/main` `a320756a539d11ffc078e66c35c1748c91b926d3`. Every figure below is
the output of a command written beside it. The figures move every time a board
lands anything, so re-run the command rather than citing this page.

`#59` names three answers `iderex/rechenstrasse` holds that this board's shared
check has no form of, and closes with the count it did not take: "Whether any
other of the seventeen callers wants any of the three. I read one board's file
and placed what it answers; I did not ask the other sixteen, and this names no
board that asked." This is that count.

IT IS THE COUNT A RECORDED RULING RESTS ON, which is why it is worth the sweep
rather than being a tidier version of what `#59` already says. The ruling on
`#36` of 28 August 2026 refused the `Scope:` comparison a shared route in these
words: "One board carrying it is not seventeen boards wanting it, and a shared
answer nobody asked for is inventory ... if a shared route is ever justified,
that arrives as its own issue naming at least three boards that asked." So the
number of boards holding an answer is the quantity that ruling turns on, and
`#59`'s three answers had never been counted against it.

This decides nothing. Two of the three come out below the bar that ruling names
and one comes out at it, and which way that resolves is not settled here.

## The population, and why it is wider than every earlier reading on this board

The board list comes from the roster:

```
OPS=<path to a clone of iderex/operations>
git -C "$OPS" grep -n '^Owner:' origin/main -- store/repo/ |
  sed 's|^origin/main:store/repo/||' | sed 's|\.md:[0-9]*:Owner: *|\t|' |
  awk -F'\t' '{print $2"/"$1}' > roster.txt
wc -l < roster.txt
73
```

EVERY EARLIER READING OF THESE COPIES ASKED FOR ONE FILENAME, and each declared
that as its own bound. The reading on `#27` of 30 August states it exactly: "I
asked every board for `.github/workflows/shared-hygiene.yml` and for nothing
else, so a caller that named its file something else is not in the count and
would not be. That is the bound rather than a claim of completeness." The same
bound was in force on the local copies, which were fetched as
`.github/workflows/pr-hygiene.yml` and as nothing else.

I asked each board for the whole directory instead:

```
while read -r r; do
  gh api "repos/$r/contents/.github/workflows" --jq ".[].name" | sed "s|^|$r\t|"
done < roster.txt > wf.tsv
wc -l < wf.tsv
902
grep -iE 'hygiene|pull|pr[-_.]' wf.tsv | awk -F'\t' '{print $2}' | sort | uniq -c | sort -rn
     34 pr-hygiene.yml
     19 shared-hygiene.yml
      3 pr-hygiene.yaml
      3 hygiene.yml
      2 pull-request.yml
      1 text-hygiene.yml
      1 pull-request-check.yml
      1 pr-hygiene.py
      1 doc-hygiene.yml
```

THE FILENAME BOUND WAS HIDING BOARDS AND NOT ONLY FILES. Four of those names
are not a local gate. `iderex/lesesaal`'s `text-hygiene.yml` refuses line
endings and encoding and its `doc-hygiene.yml` is a documentation lint;
`Flowfin/jellyfin-plugin-stats`'s `pr-hygiene.py` is the rule file its own
`pr-hygiene.yml` runs; and `iderex/wache`'s `hygiene.yml` is THIS BOARD CALLING
ITS OWN CHECK by local path, which is a twentieth caller the
`shared-hygiene.yml` sweep could not see and not a copy of anything. The eight
that are left were read one header at a time rather than placed by name, and
seven of them are a pull-request hygiene gate:

```
Flowfin/jellyfin-plugin-invites       pr-hygiene.yaml          hygiene gate
Flowfin/jellyfin-plugin-requests      pr-hygiene.yaml          hygiene gate
Flowfin/jellyfin-plugin-watchlist     pr-hygiene.yaml          hygiene gate
Flowfin/jellyfin-plugin-watch-sync    pull-request-check.yml   hygiene gate
Flowfin/lab                           pull-request.yml         hygiene gate
iderex/linienbuch                     hygiene.yml              hygiene gate
iderex/pruefstand                     hygiene.yml              hygiene gate
iderex/retusche                       pull-request.yml         NOT one - the
                                                               type, format and
                                                               test gates
```

So 41 boards hold a pull-request hygiene gate rather than 34. One of the 34 is
this board's own `pr-hygiene.yml`, which is the shared implementation itself and
not a copy of it - the same thing a byte comparison could not tell apart on
`dco.yml`, recorded on `#25`. The rules of eighteen files live outside the
workflow that runs them:

```
grep -inE '(bash|python3?|sh) +[^ ]*\.(sh|py)' h/*.yml h2/*
ls rules/ | wc -l
18
cat rules/* | wc -l
5529
```

`iderex/messstube` looks like a delegating board on that grep and is not: its
only match is a comment telling a reader how to extract its inline script to a
file. I read it rather than counting it.

## The three answers, counted

Each was searched for by BEHAVIOUR rather than by `iderex/rechenstrasse`'s name
for it, because an arm name is that board's vocabulary and a board answering the
same question in its own words would not match it. Every hit below was then
opened and read at the refusal itself.

```
grep -linE 'default_branch|defaultBranch|default branch' h/*.yml h2/* rules/*
grep -linE 'generated|DO NOT EDIT|autogen|lock ?file'    h/*.yml h2/* rules/*
grep -linE "commit-has-no-body|no body|subject line and nothing|format=%B" h/*.yml h2/* rules/*
```

**`head-is-default-branch`: three boards.**

```
iderex/rechenstrasse   .github/pr-hygiene/hygiene.py:104     head_failures(), head == default
iderex/findbuch        .github/scripts/pr-hygiene.sh:83      refuse head-is-the-default-branch
iderex/plattenschrank  .github/workflows/pr-hygiene.yml:188  headIsTheDefaultBranch(pr.head.ref, defaultBranch)
```

`iderex/findbuch` and `iderex/plattenschrank` are outside everything `#59`
reads, and `iderex/plattenschrank` is one of the boards whose copy is to be
removed. Each of the three refuses for its own stated reason, and the reasons
are not the same: `iderex/findbuch` names a fork's default branch being
force-moved under a review that is reading it, and the other two name the
history the ruleset protects.

`iderex/kanzlei` matches that grep and is not a fourth. Its match is a comment
saying the opposite - "a direct push to the default branch is refused by the
ruleset rather than by a check" - which is the same shape of false positive the
reading on `#27` already placed for `iderex/plattenschrank` on `Scope:`.

**`generated-file-edited`: one board.** `iderex/rechenstrasse`,
`.github/pr-hygiene/hygiene.py:116`, against a `Regenerated: <path>` line in the
body.

FIVE BOARDS MATCH THAT GREP FOR THE OPPOSITE REASON, and reading them is what
separates them. `Flowfin/jellyfin-plugin-metadata-sync`, `-smart-collections`,
`-sso`, `iderex/Easy-Compliance-Manager` and `iderex/swarm.asm` all treat
generated files as a category to EXCLUDE from a churn count, so a change to one
counts for less rather than being refused. Nothing in the five refuses an
undeclared change to a generated file.

**`commit-has-no-body`: one board.** `iderex/rechenstrasse`,
`.github/pr-hygiene/hygiene.py:144`, `len(lines) < 2` with trailers stripped
first.

Eight boards read the full message with `git log --format=%B` and none of them
asks this question of it: `iderex/ausgleich`, `iderex/messbuch`,
`iderex/spurenarchiv`, `iderex/stammtisch`, `iderex/sternwarte`,
`Flowfin/core`, `iderex/gutachten` and `iderex/lehrkanzel` read the whole
message to scan its bytes, to find a closing keyword or to find an issue
reference. `iderex/gutachten`'s seven rules and `iderex/lehrkanzel`'s four are
listed in their own headers and neither list holds it.

## What the count says against the bar, and what it does not

Two of the three are held by one board. Against the `#36` ruling's own words
they are the case it decided: a shared answer nobody else asked for is
inventory.

One of the three is held by three boards, which is the number that ruling names
as what a re-opening would have to show. WHETHER THAT MAKES IT A DIFFERENT
ANSWER IS NOT SETTLED HERE, and there are two readings I cannot choose between
from a count. The ruling's bar is "three boards that asked", and three boards
holding an answer is not three boards asking for a shared one - none of the
three has said anything about a shared route. Against that, the bar was written
to be evaluable, and a count of boards holding the answer is the only thing it
could be evaluated on.

What the count does settle is that the premise `#59` reasons from is not the
state of the roster. It reads as three answers of one board, and one of them is
three boards' answer.

## Two corrections to the caller reading, from the same sweep

These belong to `#27` and are recorded here because one sweep produced them.

**There are nineteen callers and `iderex/messstube` is the nineteenth.**

```
gh api 'repos/iderex/messstube/commits?path=.github/workflows/shared-hygiene.yml' \
  --jq '.[] | [.sha[0:9], .commit.committer.date] | @tsv'
e9d3f8a44       2026-08-27T14:00:40Z
```

It has called since 27 August, before the reading that counted eighteen, so it
is a board that sweep missed rather than one that arrived after it. It pins
`113085b269d3437a3f96ff9e7060b64b0af88ab1`, which is `v1.2.0` and at the floor
`#27`'s second done-condition names.

IT KEEPS ITS OWN COPY, so it is in the removal set and not in
`iderex/lesesaal`'s position. Its local `pr-hygiene.yml` declares its own
`pull_request` trigger and still runs. Its concurrency group is the bare
`pr-hygiene-${{ github.event.pull_request.number }}`, which is the string the
shared file claimed below `v1.2.0` - so it is the second board found spelling
it, after `iderex/stammtisch`. It cannot collide, because the exposure needs the
board to be BELOW the floor and this one is at it. That is a fact about the
board rather than a risk to it.

**`iderex/pruefstand` is in the removal set too, and the reading that put it
beside `iderex/lesesaal` was defeated by the filename bound.** The 30 August
reading concluded "It holds no local copy, so it is in `iderex/lesesaal`'s
position rather than among the removals", from a 404 on
`.github/workflows/pr-hygiene.yml`. Its gate is at
`.github/workflows/hygiene.yml`:

```
gh api repos/iderex/pruefstand/contents/.github/workflows/hygiene.yml --jq '.content' |
  base64 -d | sed -n '1,2p;14,18p'
# The deterministic pull request hygiene check (#93). This workflow is a wrapper
# around `python -m hygiene` and carries no logic of its own, for the same
name: hygiene

on:
  pull_request:
    types: [opened, synchronize, reopened, edited]
```

Its own `pull_request` trigger, so it still runs, and 642 lines of rules in
`hygiene.py` beside it. So eighteen of the nineteen callers keep a gate of their
own, and `iderex/lesesaal` is the only one that does not.

THAT FILE ALSO CARRIES THE `Scope:` COMPARISON, as `changed-paths-inside-scope`,
and so does `iderex/lehrkanzel` as `paths-inside-scope`. The reading on `#27`
of 31 August placed three boards holding it and met the `#36` bar on that
count; there are five, and `iderex/pruefstand` is one of the four that call this
check. The ruling stands as it was taken and nothing here re-opens it.

## A trap in the removal sequence that nothing has recorded

`#27`'s sequence is pin, then read, then delete. On two boards of the removal
set the delete strands a REQUIRED status check, and a required check that never
reports leaves a pull request permanently pending rather than red - so nothing
merges and nothing says why.

```
for b in <the removal set>; do
  gh api "repos/$b/rulesets" --jq '.[].id' | while read -r id; do
    gh api "repos/$b/rulesets/$id" --jq '[.rules[]? |
      select(.type=="required_status_checks") |
      .parameters.required_status_checks[]?.context] | join(",")'
  done
done
```

```
iderex/hoersaal     ... ,pr-hygiene, ...
iderex/stammtisch   ... ,Deterministic PR-hygiene checks, ...
```

Both names are produced by the local file and by nothing else.
`iderex/hoersaal`'s job is `pr-hygiene:` with no `name:`, so its check run
carries the job id; `iderex/stammtisch`'s job declares
`name: Deterministic PR-hygiene checks`.

THE SHARED CHECK PRODUCES NEITHER NAME, and the near miss is the dangerous part:

```
git show origin/main:.github/workflows/pr-hygiene.yml | grep 'name: Deterministic PR hygiene'
    name: Deterministic PR hygiene
```

`Deterministic PR hygiene` against a required `Deterministic PR-hygiene checks`
is one hyphen and one word apart, and a required context is matched by its
literal name. A board reading the two side by side can take them for the same
check. On top of that a called workflow's check run is named
`<caller job id> / <called job name>`, so even the exact string would not arrive
unprefixed.

What a board in this position has to do is a ruleset edit in the same change as
the delete, and `#27`'s sequence has no step for it. `iderex/messstube` states
the same hazard about its own file in its own words - that renaming the job
"silently detaches it from the ruleset and takes a required check off the gate
with it" - so the shape is known on the boards and not on the sequence.

## Not evaluated

Whether any of these gates is failing today. I read files, trees and rulesets,
and no run.

What `iderex/linienbuch` answers. Its `hygiene.yml` is a wrapper around a gate
command compiled from Rust in that tree, which I did not fetch. It calls nothing
here, so it is outside the removal set, but its answers are unread and this
names none of them.

Whether a board calls this check from a file that is not in
`.github/workflows/`. The directory listing above is that directory and nothing
else.

How many boards call this check from a file not named `shared-hygiene.yml`.
This board does, from its own `hygiene.yml`, which is how that bound was found;
I did not fetch all 902 workflow files to close it, so nineteen is a floor on
the callers outside this board and not a total. THAT BOUND IS CLOSED, on 4
September 2026, by a walk over all 924 workflow files of the 73 roster boards
that hold a workflow directory: nineteen is the total, and every one of the
nineteen calls from `shared-hygiene.yml`.
`docs/caller-inputs-and-the-two-late-copies.md` carries that walk, its commands,
and the two call sites the code search missed.

Whether the other rules of `iderex/pruefstand` and of the seven gates found
under another filename hold anything else the shared check has no form of. I
searched those 5529 lines for the three answers `#59` names and placed what they
say about those three. I did not place everything else in them, and `#27`'s
first done-condition line asks for exactly that reading over the boards in its
removal set.
