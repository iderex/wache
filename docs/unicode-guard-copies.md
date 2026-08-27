# The 63 unicode guards, read rather than counted

Taken on 27 August 2026 against the roster in `iderex/operations`, at
`origin/main` `bc2b8bd1f4238d8148f1d4e6fe765a1e8fb614e0`, over the 73 boards the
`boards` function in `docs/standardisation-survey.md` returns. The figures move
whenever a board lands anything; re-run the commands rather than citing this
page.

`#28` says of its own counts that they are a byte comparison, and lists as not
evaluated whether the 34 distinct contents differ in what they refuse or only in
how they say it. This is that reading. It is the question a board has to answer
before it deletes its own copy, because a migration that quietly narrows what is
refused is worse than the duplication it removes.

## The byte counts reproduce

    awk -F'\t' '$2=="unicode-guard.yml"{print $3}' inventory.tsv |
      sort | uniq -c | sort -rn | awk '{printf "%s ", $1} END{print ""}'
    28 3 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1

63 copies, 34 distinct contents, largest identical cluster 28, unchanged from
the survey. `inventory.tsv` is rebuilt here by the same shape the survey
declares, with the per-board response parsed only when the call succeeded.

## What they refuse is one string, written 33 times

Each distinct copy was fetched by blob id and its refusal pattern extracted:

    gh api "repos/$r/git/blobs/$sha" --jq '.content' | tr -d '\n' | base64 -d
    grep -hoE "\(\*UTF\)\[[^]]*\]" <file>

    33  (*UTF)[\x{202A}-\x{202E}\x{2066}-\x{2069}\x{200E}\x{200F}\x{061C}\x{200B}-\x{200D}\x{2060}]
     1  no PCRE class in the file          iderex/lesesaal

So 33 of the 34 distinct contents refuse the identical codepoint set, character
for character, and it is the set this board's `unicode-guard.yml` carries. The
34 versions differ in comments, in action pins, in job names and in triggers.
They do not differ in what they refuse. THE ANSWER TO `#28`'S OPEN QUESTION IS
THEREFORE THE CHEAP ONE: for 33 of 34, adopting the shared guard changes nothing
about which characters are caught.

## Three differences that are not wording

**Only one copy proves its pattern before applying it.** Counting the fixture
lines in each distinct copy:

    grep -cE 'must_catch|must_pass|fixture' <file>
    33 lines   iderex/wache
     0 lines   the other 33

Every other copy on the fleet is a grep that has never been shown to refuse
anything. That is the argument this board's own guard opens with, and it turns
out to describe the whole population rather than a hypothetical: a guard that
refuses this rarely can go silent without anybody noticing. It is also the
largest single thing a caller gains, and it is invisible in a byte comparison.

**Three boards scan `main` only.**

    grep -oE 'branches: *\[[^]]*\]' <file>
    branches: [main]    iderex/kartei, iderex/kontor, iderex/lichttisch
    branches: ["**"]    every other copy

The shared guard's own comment names `kontor` for this and no one else. Two more
boards do it, so the comment understates the case rather than misstating it.
The trigger belongs to the caller either way: `unicode-guard.yml` runs on
`workflow_call` and declares no events, so a board that migrates keeps whatever
`branches:` it had unless it changes that line too.

**Two deviations survive no migration as written.**

`iderex/swarm.asm` narrows the scan to a path list instead of the whole tree:

    git grep -nIP "$pattern" -- src tools '*.ps1' .github tests

Its own comment argues the narrowing - `docs/**` is left out on purpose because
that README carries intentional emoji and em dashes - and names a test in its C#
harness that refuses a further narrowing of the list. The shared guard scans
`-- .` and takes no input for a path list, so this board cannot call it without
either widening its scan or the shared guard gaining that input.

`iderex/lesesaal` has no workflow-level implementation at all. Its step is

    go run . ci unicode

which is the same procedure a contributor runs before pushing, so that tree
holds one gate rather than two. Calling the shared guard would give it two, and
the shared file's own comment already records why the Go version did not move
here: a guard needing a toolchain is a guard some boards cannot call.

Both are the case `docs/standardisation-survey.md` states in the abstract - a
board-local deviation has to be expressible as an input or it cannot survive -
now measured on this file. Neither is drift.

## Who calls what today

    gh search code "iderex/wache/.github/workflows" --limit 100 \
      --json repository,path --jq '.[] | "\(.repository.nameWithOwner)\t\(.path)"' |
      sort -u

Seventeen boards, every one through `.github/workflows/shared-hygiene.yml`. No
`uses:` line in any of the seventeen names `unicode-guard.yml`:

    grep -c 'uses:.*unicode-guard' <the seventeen>
    0

One of the seventeen mentions the file name in a comment about its own local
copy, which is why a plain grep for the string returns one hit and a grep for a
call returns none.

All seventeen keep a local `unicode-guard.yml` beside the call, against sixteen
of seventeen for `pr-hygiene.yml`:

    awk -F'\t' '$2=="shared-hygiene.yml"{s[$1]=1} $2=="unicode-guard.yml"{u[$1]=1}
                END{n=0; for(b in s) if(b in u) n++; print n}' inventory.tsv
    17

And the pins those seventeen carry:

    grep -hoE 'iderex/wache/\.github/workflows/pr-hygiene\.yml@[0-9a-f]+ # v[0-9.]+' |
      grep -oE 'v[0-9.]+$' | sort | uniq -c
    12 v1.0.0
     5 v1.2.0

## What a first caller has to carry

`#28` records the decision that the guard gets callers, and its first done-when
is a board that calls it with its local copy removed in the same change. This
reading says what that change costs on each kind of board:

- On any of the 33 boards holding a grep copy other than `swarm.asm`, the
  refusal set is unchanged, the fixtures are new, and the only line to think
  about is `branches:` - unchanged for 30 of them, a widening from `main` for
  `kartei`, `kontor` and `lichttisch`.
- On `iderex/swarm.asm` it is a widening of the scanned tree, which that board's
  own tests refuse in the other direction. It is the wrong board to go first
  with.
- On `iderex/lesesaal` it is a second gate over a tree that deliberately holds
  one. Also the wrong board to go first with.

WHAT THIS PAGE DOES NOT EVALUATE. Whether each copy's `on:` block, permissions
and concurrency are equivalent to the shared caller's - only `branches:` was
extracted. And whether the shared guard's pattern is the RIGHT set: every copy
agreeing on it is evidence that nobody has revisited it, not evidence that it is
complete.
