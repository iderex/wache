# What the boards hold more than once

Taken on 26 August 2026 against the roster in `iderex/operations`, at
`origin/main` `075b8ee5d1de42072452ea370f65b3caba5b45a9`. Every figure below is
the output of a command written beside it. The figures move every time a board
lands anything, so re-run the command rather than citing this page.

This is the survey and the shape proposal that `#8` asks for. It ends in a
decision request and not in an implementation.

## How the numbers were produced

The board list is derived from the roster rather than typed:

```
OPS=<path to a clone of iderex/operations>
boards() {
  git -C "$OPS" ls-tree --name-only origin/main store/repo/ | grep -v README |
  while read -r f; do
    git -C "$OPS" show "origin/main:$f" |
      awk '/^Id: /{i=$2} /^Owner: /{o=$2} END{print o"/"i}'
  done | sort
}
boards | wc -l
72
```

The inventory is one API call per board, at each board's default branch:

```
boards | while read -r r; do
  gh api "repos/$r/contents/.github/workflows" \
    --jq ".[] | \"$r\t\(.name)\t\(.sha)\"" 2>/dev/null
done > inventory.tsv
wc -l < inventory.tsv
867
```

867 workflow files across 72 boards, under 233 distinct file names. One board
carries no `.github/workflows` directory at all.

THE DRIFT TEST IS A BYTE COMPARISON AND NOT A JUDGEMENT. The `sha` the contents
API returns is the git blob id of the file, so two boards reporting one sha hold
byte-identical files and two boards reporting different shas do not. I checked
that rather than assuming it:

```
$ git hash-object dco-bremsweg.yml
5b53eab934abf424af0a207534d1864744c2395e
$ awk -F'\t' '$1=="iderex/bremsweg" && $2=="dco.yml"{print $3}' inventory.tsv
5b53eab934abf424af0a207534d1864744c2395e
$ cmp -s dco-bremsweg.yml dco-attrappe.yml && echo "bytes identical"
bytes identical
```

What it cannot tell me is whether two files that differ MEAN anything different.
A comment edited in one copy and a refusal deleted in another are the same
result here. Where that distinction decides a candidate below, I fetched the two
files and quoted the diff.

The counts per file name, copies against distinct contents:

```
awk -F'\t' '{c[$2]++; k=$2 SUBSEP $3; if(!(k in seen)){seen[k]=1; d[$2]++}}
     END {for(n in c) printf "%-26s %3d %3d\n", n, c[n], d[n]}' inventory.tsv |
  sort -k2,2nr | awk '$2>=9'

unicode-guard.yml           63  34
zizmor.yml                  62  44
scorecard.yml               61  32
dco.yml                     60  30
dependency-review.yml       60  29
pr-hygiene.yml              34  34
codeql.yml                  30  30
invariants.yml              23  23
build.yml                   20  20
shared-hygiene.yml          17  17
fuzz.yml                    15  15
coverage.yml                12  12
gate.yml                    12  12
build.yaml                  11  11
lint.yml                    11  11
publish.yaml                11  11
test.yml                    11  11
mutation.yml                10  10
test.yaml                   10  10
scan-codeql.yaml             9   9
```

A file name is a weak proxy for a function and I am not pretending otherwise: it
misses a board that renamed its copy, and it joins two boards that gave one name
to different things. It is the cheapest sweep that reaches all 72 boards, and
every candidate below was checked by reading files rather than by trusting the
name.

## The candidates

Each one gives its population, the size of its largest byte-identical cluster,
and what the tail looks like. The cluster sizes come from

```
awk -F'\t' -v F=<name> '$2==F{print $3}' inventory.tsv | sort | uniq -c | sort -rn
```

### `dependabot.yml` - the cheapest one, and the one with the most agreement

```
boards | while read -r r; do
  printf '%s\t%s\n' "$r" \
    "$(gh api "repos/$r/contents/.github/dependabot.yml" --jq '.sha' 2>/dev/null || echo absent)"
done > dependabot.tsv
```

72 boards, 72 copies, none absent, 10 distinct contents, and the largest cluster
is 63. Nine boards each hold a file nobody else holds.

This is the candidate to take first, and the reason is the shape of that
distribution rather than its size. 63 boards have already agreed on the answer
without anybody writing it down, so standardising costs a statement of what is
already true, and the nine outliers are a readable list rather than a research
project.

### `dco.yml` - the freshest copy, and the one #8 was opened about

60 copies, 30 distinct, clusters `29 3` and then 28 singletons.

The 29-cluster and the 3-cluster differ by one line, and it is a comment:

```
$ diff -u dco-bremsweg.yml dco-sternwarte.yml
-# Developer Certificate of Origin (DCO) sign-off gate (#746). A contributor
+# Developer Certificate of Origin (DCO) sign-off gate. A contributor
```

That is drift with no consequence, and a survey that reported the pair as
"drifted" and stopped would be overstating its case. The singletons are the
other half. `iderex/blende` carries a deviation that is load-bearing:

```
$ diff -u dco-bremsweg.yml dco-blende.yml
+# This repository tracks no DCO text and no contributor guide, so the trailer is an
+# assertion about a document that is not here yet, and the failure message in the
+# step below names two files this tree does not contain.
```

A board-local disclosure, written because that board's situation is genuinely
different. Any shape proposed below has to let that survive, and a sync that
overwrites it turns a stated residual risk into a silent one.

### `dependency-review.yml`, `scorecard.yml`, `zizmor.yml` - the analyser gates

```
dependency-review.yml   60 copies  29 distinct  clusters 32 then 28 singletons
scorecard.yml           61 copies  32 distinct  clusters 26  3  2  2 then singletons
zizmor.yml              62 copies  44 distinct  clusters 19 then 43 singletons
```

`zizmor.yml` is the worst of the three by a distance: fewer than a third of the
copies agree with each other. I have not read the 43 singletons and I am not
claiming to know why they differ.

### `unicode-guard.yml` - this board already holds one and nobody calls it

63 copies across the roster, 34 distinct, largest cluster 28. This board holds a
shared implementation of it and publishes a calling example for it in the
README.

Nobody calls it:

```
$ gh search code "iderex/wache/.github/workflows" --limit 100 \
    --json repository,path --jq '.[] | "\(.repository.nameWithOwner)\t\(.path)"' | sort -u
```

Seventeen boards appear, every one of them through
`.github/workflows/shared-hygiene.yml`, and every one of those calls
`pr-hygiene.yml`. Not one names `unicode-guard.yml`. So the shared unicode guard
is a workflow with no caller, and the 63 copies are exactly as unstandardised as
they were before it existed.

### `pr-hygiene.yml` - the migration that is half done

34 copies, 34 distinct: no two boards hold the same one. That is the count that
made this board necessary and it is unchanged for the boards that have not
moved.

For the boards that have moved, the reading is worse than I expected. Sixteen of
the seventeen callers still keep their own `pr-hygiene.yml` beside the call:

```
awk -F'\t' '$2=="shared-hygiene.yml"{s[$1]=1} $2=="pr-hygiene.yml"{p[$1]=1}
            END{for(b in s) if(b in p) print b}' inventory.tsv | sort
Flowfin/core
iderex/ausgleich
iderex/bremsweg
iderex/gutachten
iderex/hallraum
iderex/hoersaal
iderex/kanzlei
iderex/kontor
iderex/lichttisch
iderex/nachtwache
iderex/plattenschrank
iderex/rechenstrasse
iderex/schallweg
iderex/spurenarchiv
iderex/stammtisch
iderex/sternwarte
```

`iderex/lesesaal` is the one that removed its copy. The sixteen leftovers are
not dormant files: every one of them still declares its own `pull_request`
trigger, so it still runs. I fetched each and counted the trigger line:

```
for b in <the sixteen>; do
  gh api "repos/$b/contents/.github/workflows/pr-hygiene.yml" --jq '.content' |
    base64 -d | grep -c '^  pull_request:'
done
```

One for every board. So sixteen boards are judged twice on every pull request,
by two implementations that agree on nothing byte for byte, and the shared one
is the newer of the two.

The pins make it sharper. Twelve of the seventeen call `v1.0.0`:

```
Flowfin/core            v1.2.0      iderex/lichttisch       v1.0.0
iderex/ausgleich        v1.0.0      iderex/nachtwache       v1.0.0
iderex/bremsweg         v1.2.0      iderex/plattenschrank   v1.0.0
iderex/gutachten        v1.0.0      iderex/rechenstrasse    v1.0.0
iderex/hallraum         v1.0.0      iderex/schallweg        v1.2.0
iderex/hoersaal         v1.0.0      iderex/spurenarchiv     v1.2.0
iderex/kanzlei          v1.0.0      iderex/stammtisch       v1.0.0
iderex/kontor           v1.0.0      iderex/sternwarte       v1.2.0
iderex/lesesaal         v1.0.0
```

The README of this board says `v1.2.0` is the one release a caller cannot skip,
because below it the shared workflow's concurrency group cancels the calling
board's own gate. Twelve boards are below it, and sixteen boards have a local
gate for that cancellation to reach.

## The finding that is not a count

`iderex/sternwarte` had already solved, in its own copy, the defect this board
carried until tonight. Its local `pr-hygiene.yml` declares the activity types:

```
on:
  pull_request:
    # `edited` is here for the body leg. A body fixed after the first red run
    # has to be able to turn the check green without an empty commit pushed to
    # force it.
```

and `iderex/bremsweg` carries the same answer with the reason from the other
direction:

```
on:
  # `edited` matters as much as `opened`: a body emptied after the check passed
  # would otherwise keep the tick it earned when it was full.
  pull_request:
    types: [opened, edited, synchronize, reopened]
```

The shared caller here declared no types at all until `#22` landed on 26 August
2026, so a board that migrated from either of those two copies lost a working
answer and gained a gap, and nothing in the migration would have told it.

That is the argument for the survey being a prerequisite of the standardisation
rather than a report on it. Consolidating on the newest copy is not the same as
consolidating on the best one, and the difference is only visible if somebody
reads the copies before they are deleted.

## What I am not calling debt

The following recur across boards and I am NOT proposing them for
standardisation:

- `build.yml` and `build.yaml` (31 copies), `test.yml` and `test.yaml` (21),
  `fuzz.yml` (15), `coverage.yml` (12), `lint.yml` (11), `publish.yaml` (11),
  `mutation.yml` (10), `package.yml` (6), `format.yml` (6). Every one of these
  is wholly distinct: as many contents as copies. A file that no two boards
  wrote the same way is what a per-project file looks like, and there is no
  shared answer to lift out of it. I DID NOT READ THEM. That is a reading of the
  distribution and not of the files, and it is why these sit outside the tranche
  rather than a finding that they are fine.
- `codeql.yml` (30 copies, 30 distinct) and `scan-codeql.yaml` (9 and 9). Same
  distribution, same non-reading, and one extra reason to be careful: a code
  scanning configuration is language-shaped, so identical copies would be the
  surprising result rather than the expected one.
- `invariants.yml` (23 and 23) and `gate.yml` (12 and 12). Board-specific by
  name and unread by me.

A copy that never drifted and never will is not debt, and I cannot say which of
these that describes without reading them. What I can say is that none of them
shows the signature the seven candidates above show, which is a large agreeing
cluster with a tail hanging off it.

## The shape, as options with their costs

The decision between these is not mine and this section does not take it.

### A. A template collection here, copied with a pinned origin

Boards keep their own file and copy it from a template in this repository, with
a comment naming the template and the commit it came from.

- Cheapest to adopt: no caller change, no permissions question, no cross-account
  visibility requirement.
- A board-local deviation survives trivially, because the copy is the board's.
- It does not fix anything. The 63 copies stay 63 copies, they drift the same
  way, and the only new thing is a comment saying where they started. This is
  close to what the boards effectively do today.

### B. A shared workflow called by reference

What this board already is for `pr-hygiene.yml`.

- One implementation, one place to repair.
- The cost is a pin per board, and the measurement above says what that costs in
  practice: twelve of seventeen callers are two releases behind on a release the
  README says they cannot skip. A pin is not a subscription.
- A board-local deviation has to be expressible as an INPUT, or it cannot
  survive. `pr-hygiene.yml` already carries one such input,
  `subject_names_issue`, and `blende`'s DCO disclosure is an example of a
  deviation that no boolean expresses.
- It forces `Visibility: public` on this board, which the roster record already
  records as load-bearing for exactly this reason.
- The trigger cannot be shared. A `workflow_call` workflow declares no events,
  so every caller keeps its own `on:` block, and that block is where `#22` was.
  Whatever is shared, part of the answer stays in the caller.

### C. A sync mechanism with pins

A tool that pushes the canonical file into every board.

- The only option that closes the gap without each board acting.
- It is the option with a scar. `#8` names `iderex/operations#1604` and `#1588`
  as a synced tree that overwrote local repairs. I have not re-read those two
  and this document does not restate what they say; they are named so that the
  decision is taken with them open.
- The `blende` deviation and the `sternwarte` trigger comment above are both
  things a sync would delete, and neither is expressible as an input.

### What I would want the answer to carry, whichever it is

- A board-local deviation is DECLARED rather than tolerated: it says what it
  deviates from and why, so a later reader can tell it from drift. Today the two
  are indistinguishable, which is why the `dco.yml` tail needed a diff each.
- The migration removes the copy it replaces. Sixteen boards running both is
  worse than either alone, and it happened by nobody deciding it.

## The decision I am asking for

1. Which shape, per candidate. It does not have to be one answer for all seven:
   `dependabot.yml` at 63 agreeing copies and `zizmor.yml` at 19 are different
   problems.
2. Whether the sixteen duplicate `pr-hygiene.yml` files are removed now, ahead
   of any decision on the rest. I read that as the most expensive thing standing
   and the cheapest to end.
3. Whether `unicode-guard.yml` gets callers or the shared copy is retired. A
   shared workflow with no caller is a maintenance cost with no return.

## The first tranche

Filed on 26 August 2026, one per candidate, each standing alone with its own
counts, its own evidence and its own done-when. None of them needs this document
read first.

- `#24` `dependabot.yml`: 72 copies, 10 contents, 63 agreeing. The cheapest.
- `#25` `dco.yml`: 60 copies, 30 contents, and a tail that is half drift and
  half declared deviation.
- `#26` `zizmor.yml`: 62 copies, 44 contents. Filed as a reading rather than as
  a standardisation, because there is no majority to record.
- `#27` the sixteen boards running the shared hygiene check and a local copy at
  the same time.
- `#28` the shared unicode guard, which has no callers.

`dependency-review.yml` and `scorecard.yml` are the next tranche and are
deliberately not filed yet. Their counts are in the table above; `#26` is the
same family and is the one whose distribution says the family needs reading
before it needs a shape.

## What this survey did not cover

- Anything outside `.github/workflows/` and `.github/dependabot.yml`. Helper
  scripts, CI probes and refusal or logging shapes are named in `#8` and are not
  in this sweep. They need a tree walk per board rather than a directory
  listing, and I would rather file that as its own reading than pad this one.
- The 72 boards are those the roster holds. A repository outside the roster is
  invisible here.
- The contents of any file not quoted above. The byte comparison is the whole of
  the evidence for the counts, and where I say what a difference MEANS I fetched
  the files and pasted the diff.
