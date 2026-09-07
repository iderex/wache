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

`wf-list.tsv`, WHICH THE 7 SEPTEMBER SECTION BELOW READS TWICE, IS DERIVED FROM
`wf.tsv` AND WAS WRITTEN DOWN NOWHERE. It is the same rows with one line per
board instead of one line per file, and giving it here keeps both sections on one
population rather than on two fetches taken at different moments:

```
awk -F'\t' '{ if($1!=b){ if(b!="") print b"\t"n; b=$1; n=$2 } else n=n","$2 }
     END{ if(b!="") print b"\t"n }' wf.tsv > wf-list.tsv
wc -l < wf-list.tsv
74
```

Re-derived on 7 September 2026 against the roster at `iderex/operations`
`origin/main` `a2d6dfe2472b09981b2bb5345427f7cd1bdcfd70`, this returns the 19,
34 and 17 the section below reads off it, with that section's two `awk` lines
unchanged.

ONE ROW OF `wf.tsv` IS AN ERROR BODY AND NOT A FILENAME, and it takes the next
board's first file with it. `gh api` writes its error body to STDOUT rather than
to stderr, does not apply `--jq` to it and ends it with no newline, so the board
holding no `.github/workflows` directory lands the 404 JSON in the name column
AND the first line of the NEXT board's listing is glued onto the same line, where
the `sed` prefixes the pair once:

```
grep -c 'Not Found' wf.tsv
1
grep -P '^iderex/learn-rust\t' wf.tsv
iderex/learn-rust       prueflauf.yml
gh api "repos/iderex/learn-rust/contents/.github/workflows" --jq '.[].name'
codeql.yml
prueflauf.yml
```

`iderex/lagetisch` tracks no such directory, which `docs/dco-tail.md` records
against the same roster, so it is the board that produces the body;
`iderex/learn-rust` is the board after it, and its `codeql.yml` has no row of its
own on this page's evidence while the API lists it.

THE COUNTS BELOW SURVIVE IT BY LUCK AND NOT BY CONSTRUCTION, which is the part
worth more than the repair. The swallowed name is `codeql.yml`, so the malformed
row matches neither `shared-hygiene.yml` nor `pr-hygiene.yml` and the 19, 34 and
17 are what they would be without it. Had the board after the gap declared its
own `pr-hygiene.yml` first, that board would have dropped out of `has-local`, out
of `has-both` and out of the removal set, and nothing on this page would have
said so. The row is left in `wf.tsv` and named here rather than filtered out,
because a filter would take the disclosure with it and the next reader would meet
the same trap with nothing to read.

`docs/standardisation-survey.md` names this trap for `dependabot.tsv` and says of
the workflow inventory that a 404 there "lands one malformed line instead of a
false cluster". That is the half of it. A malformed line is also a LOST line, and
which line is lost depends on the order the roster is walked in - so the sentence
understates the same defect it was written to bound. That page is `#8`'s and is
not edited from here.

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

## Read on 7 September 2026: both are in force, and one job per file was too few

The reading above intersected a required context with the job each local copy
declares, and stopped there. Three things it did not ask decide whether the
strand it describes is real, and the section did not name any of them as unread
either - which is worse than naming them, because a bound nobody declared is a
bound nobody can go and close.

A ruleset in `evaluate` mode blocks no merge. A ruleset targeting branches a
board's pull requests never go to blocks none of them. Classic branch protection
is a second mechanism with a required set of its own, and nothing here had looked
at it. And the match itself read ONE job out of each copy, where two boards in
the set declare more than one.

All four are read now, against the roster at `iderex/operations` `origin/main`
`2d5c6d9eb0fe08b32bd6a8997542e22b3ddbfb9d`, over the removal set this page
already derives - the boards holding a copy beside the call, plus
`iderex/pruefstand`, whose gate is `hygiene.yml`:

```
$ awk -F'	' '$2 ~ /shared-hygiene\.yml/{print $1}' wf-list.tsv | sort > has-shared
$ awk -F'	' '$2 ~ /(^|,)pr-hygiene\.yml(,|$)/{print $1}' wf-list.tsv | sort > has-local
$ comm -12 has-shared has-local > has-both
$ wc -l < has-shared ; wc -l < has-local ; wc -l < has-both
19
34
17
$ cp has-both removal-set; echo iderex/pruefstand >> removal-set; sort -o removal-set removal-set
$ wc -l < removal-set
18
```

Nineteen callers, thirty-four files named `pr-hygiene.yml` with this board's
source among them, seventeen boards holding both, eighteen in the removal set:
where 31 August left it and where 4 and 5 September found it again.

EVERY JOB OF EVERY COPY IS MATCHED NOW, NOT THE FIRST. Reading one job per file
was an assumption rather than a rule about these files, and two boards break it.
The copies are fetched into `copies/` and every job in each one is taken with its
id and, where it declares one, its name:

```
$ for f in copies/*.yml; do
    b=$(basename "$f" .yml | sed 's|_|/|')
    awk -v b="$b" '
      /^jobs:/{inj=1; next}
      inj && /^[^ ]/{inj=0}
      inj && /^  [A-Za-z0-9_-]+:/{ jid=$1; sub(/:$/,"",jid);
        if(cur!="") print b"	"cur"	"jname; cur=jid; jname="" }
      inj && /^    name:/{ line=$0; sub(/^    name: */,"",line); jname=line }
      END{ if(cur!="") print b"	"cur"	"jname }' "$f"
  done > hyg-jobs.tsv
$ wc -l < hyg-jobs.tsv ; cut -f1 hyg-jobs.tsv | sort -u | wc -l
22
18
$ cut -f1 hyg-jobs.tsv | sort | uniq -c | awk '$1>1{print $1" "$2}'
4 iderex/kontor
2 iderex/schallweg
```

`iderex/kontor` splits its gate into four jobs and `iderex/schallweg` carries a
self-test beside its check, so four of the twenty-two rows are jobs a
first-job-only match never reached. `iderex/hoersaal` is the other shape worth
naming: its job declares no `name:` at all, so what its ruleset requires is the
job ID. Both halves of every row are matched below for that reason.

Widening the match leaves the answer where it was:

```
$ : > hyg-rulesets.tsv
$ while read -r b; do
    ids=$(gh api "repos/$b/rulesets" --jq '.[].id' 2>/dev/null)
    for id in $ids; do
      gh api "repos/$b/rulesets/$id" --jq '"\(.id)	\(.name)	\(.enforcement)	\([.conditions.ref_name.include[]?]|join(","))	\([.rules[]? |
        select(.type=="required_status_checks") |
        .parameters.required_status_checks[]?.context] | join("|"))"'         | sed "s|^|$b	|" >> hyg-rulesets.tsv
    done
  done < removal-set
$ awk -F'	' '$6!=""{print $1}' hyg-rulesets.tsv | sort -u | wc -l
5
$ awk -F'	' 'NR==FNR{ k[$1"	"$2]=1; if($3!="") k[$1"	"$3]=1; next }
    $6!="" { n=split($6,a,"|"); for(i=1;i<=n;i++) if(($1"	"a[i]) in k)
      printf "%s	%s	%s	%s
", $1, a[i], $4, $5 }' hyg-jobs.tsv hyg-rulesets.tsv | sort -u
iderex/hoersaal      pr-hygiene                       active  ~DEFAULT_BRANCH
iderex/stammtisch    Deterministic PR-hygiene checks  active  ~DEFAULT_BRANCH
```

Two boards, five of the eighteen carrying a `required_status_checks` rule at
all, both rulesets `active`, and both reaching the branch their own pull requests
target. So the strand is real on both rather than discounted by enforcement or by
a branch pattern, and the count of two survives a match wider than the one that
produced it. The three boards that require something and are not in the pair
require `local-gate` and nothing else, which no copy here declares.

NOTHING OUTSIDE A RULESET REQUIRES ANY OF THOSE NAMES, which is the mechanism
this page had never looked at:

```
$ while read -r b; do
    def=$(gh api "repos/$b" --jq '.default_branch' 2>/dev/null)
    out=$(gh api "repos/$b/branches/$def/protection"             --jq '[.required_status_checks.contexts[]?]|join("|")' 2>&1)
    rc=$?
    if [ $rc -ne 0 ]; then
      echo "$out" | grep -q 'Branch not protected' && st=unprotected || st="ERR:$out"
    else st=protected; fi
    printf '%s	%s	%s
' "$b" "$def" "$st"
  done < removal-set > hyg-classic.tsv
$ cut -f2 hyg-classic.tsv | sort | uniq -c
     18 main
$ cut -f3 hyg-classic.tsv | sed 's/ERR:.*/ERR/' | sort | uniq -c
     18 unprotected
```

No line came back `ERR`, so no board in that tally is an access failure being
read as an absence, and every one of the eighteen defaults to `main` - which is
NOT the case on the DCO side of the same question, where
`docs/dco-rulesets-and-the-delete.md` records seven of thirteen defaulting to
`master` and one to `4.4`. A reader who carries `main` across from this page to
that one gets eight boards wrong.

`iderex/pruefstand` stays outside the trap for the reason the 1 September reading
gave, re-read here with the enforcement beside it:

```
$ for id in $(gh api repos/iderex/pruefstand/rulesets --jq '.[].id'); do
    gh api "repos/iderex/pruefstand/rulesets/$id" --jq '"\(.id)	\(.name)	\(.enforcement)	\([.conditions.ref_name.include[]?]|join(","))	\([.rules[]? |
      select(.type=="required_status_checks") |
      .parameters.required_status_checks[]?.context]|join("|"))"'
  done
20521019        gate    active  ~DEFAULT_BRANCH
```

An active ruleset that requires no status check at all.

WHAT MOVES IS THE COMMAND AND NOT THE COUNT. Every one of the four readings
agrees with what this page assumed, so the trap is still the same two boards.
What was assumed rather than read is exactly what a migrating board would assume,
and the sequence in `README.md` handed it a command printing the contexts alone,
under two lines of output that were a summary of this page rather than anything
that command prints. So a board whose ruleset is in evaluate mode, or whose
ruleset targets branches its pull requests never touch, would have taken a
ruleset edit it does not owe, and a board comparing its own run against that
paste would have found nothing to compare. That step now prints the enforcement
and the branches beside the contexts, as the DCO sequence's does, and shows one
real board's line.

## Not evaluated

Whether any of these gates is failing today. I read files, trees and rulesets,
and no run.

Whether either of the two rulesets is `active` at any later moment, and whether a
board outside the two enters the set. Both were read on 7 September 2026 and both
were; a ruleset moves by an edit on a board that is not this one, and
`docs/dco-rulesets-and-the-delete.md` measures such an edit landing two and a
half minutes after a reading taken here.

Whether classic branch protection requires one of these names on a branch other
than the default of one of the eighteen, or on a board outside the removal set. I
read one branch per board, over that set and no wider.

Whether `local-gate` is reported on the three boards that require it. Their
rulesets were read and none of the three names anything a copy in this set
produces, so they are outside the strand - but what does produce `local-gate`
there, and whether it reports at all, is a question about those boards and not
this one, and nothing here asked it.

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
