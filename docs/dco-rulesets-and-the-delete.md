# What a board loses when it deletes its own DCO gate

`#25` asks that sixty-odd hand-written copies of `.github/workflows/dco.yml` be
replaced by calls to the one on this board. Every one of those migrations
deletes a workflow file. This is a reading of what else goes with it, taken on 4
September 2026 against the roster at `iderex/operations` `origin/main`
`e95252b6ed290e2e4aaf37a0f4879a201cc613ac`.

The answer is that on eleven boards the delete takes a REQUIRED status check off
the gate and puts nothing back, so every pull request there is left permanently
pending rather than red - nothing merges and nothing says why. The shared gate's
job carries the same name those eleven rulesets require, which is what makes the
trap invisible to a board that checks the name.

## The population

```
$ git -C <operations> ls-tree --name-only e95252b6 store/repo/ | grep -v README | wc -l
74
$ while read -r r; do
    gh api "repos/$r/contents/.github/workflows/dco.yml" --jq '.content' 2>/dev/null |
      base64 -d > "f-$(echo "$r" | tr '/' '_').yml"
  done < roster.txt
$ ls f-*.yml | wc -l
63
```

Sixty-three, which is where `docs/dco-tail.md` left the count on 30 August and
where the corpus walk of 4 September confirmed it. ONE OF THE SIXTY-THREE IS
THIS BOARD'S SOURCE FILE and not a copy, which is the confound the walk already
named on this side; sixty-two boards hold a copy.

## What each copy is called on the gate

A ruleset requires a check by the literal name of the check run, and for a job
that is the job's `name:` where it declares one and the job id where it does
not:

```
$ cut -f3 jobs.tsv | sort | uniq -c | sort -rn
     61 DCO sign-off
      1 dco
      1 DCO sign-off on every commit
$ awk -F'\t' '$3!="DCO sign-off"' jobs.tsv
iderex/Easy-Compliance-Manager  dco     DCO sign-off on every commit
iderex/swarm.asm                dco     dco
```

## Eleven rulesets require the name their own copy produces

Every ruleset on all sixty-three boards, read one at a time, and the required
contexts intersected with the name above:

```
$ while IFS=$'\t' read -r b jid nm; do
    for id in $(gh api "repos/$b/rulesets" --jq '.[].id' 2>/dev/null); do
      gh api "repos/$b/rulesets/$id" --jq '[.rules[]? |
        select(.type=="required_status_checks") |
        .parameters.required_status_checks[]?.context] | .[]'
    done
  done < jobs.tsv
$ awk -F'\t' '{n=split($4,a,"|"); for(i=1;i<=n;i++) if(a[i]==$2 || a[i]==$3) print $1"\t"a[i]}' required.tsv
Flowfin/hub                              DCO sign-off
Flowfin/jellyfin-plugin-requests         DCO sign-off
Flowfin/jellyfin-plugin-server-pairing   DCO sign-off
Flowfin/jellyfin-plugin-share-links      DCO sign-off
Flowfin/jellyfin-plugin-sso              DCO sign-off
Flowfin/jellyfin-plugin-stats            DCO sign-off
Flowfin/jellyfin-plugin-watchlist        DCO sign-off
iderex/hoersaal                          DCO sign-off
iderex/reissbrett                        DCO sign-off
iderex/relais                            DCO sign-off
iderex/stammtisch                        DCO sign-off
```

Thirty-five of the sixty-three carry a `required_status_checks` rule at all and
twenty-eight carry none, so the eleven are eleven of thirty-five rather than of
sixty-three.

NOTHING ELSE ON THOSE ELEVEN BOARDS PRODUCES THAT NAME, which is what makes the
delete the whole of the loss rather than a rename:

```
$ while read -r b; do
    for f in $(gh api "repos/$b/contents/.github/workflows" --jq '.[].name'); do
      [ "$f" = "dco.yml" ] && continue
      gh api "repos/$b/contents/.github/workflows/$f" --jq '.content' | base64 -d |
        grep -qE '^ +name: *"?DCO sign-off"? *$' && echo "$b $f"
    done
  done < eleven.txt
```

No output. On each of the eleven, `dco.yml` is the only file declaring it.

## The name matches exactly and that is the trap

The shared gate's job carries the same string those rulesets require:

```
$ git show origin/main:.github/workflows/dco.yml | sed -n '/^jobs:/,+3p'
jobs:
  dco:
    name: DCO sign-off
    runs-on: ubuntu-latest
```

So a board that compares the two NAMES concludes they line up. They do not,
because a called workflow's check run arrives prefixed with the calling job's
id. Measured on this board's own pull request rather than argued:

```
$ gh pr checks 75 --repo iderex/wache --json name,state --jq '.[] | .name' | sort -u
A calling example agrees with its pin
Analyze (actions)
CodeQL
hygiene / Deterministic PR hygiene
hygiene / Every referenced number is an issue that exists
unicode / Reject Trojan Source Unicode
```

`hygiene` and `unicode` were the job ids in this board's `hygiene.yml` and
`guards.yml` at that pull request, and `Deterministic PR hygiene` and
`Reject Trojan Source Unicode` are the job names inside the workflows those two
jobs called. A board following this board's own calling example names its job
`dco`, so what would report there is `dco / DCO sign-off` and the required
`DCO sign-off` would never be reported again.

THE SECOND HALF OF THAT PASTE STOPPED SHOWING A CALL SIX HOURS AFTER THIS PAGE
LANDED, and the prefix went with it. `#28` retired the shared unicode guard and
kept the check local, so `guards.yml` calls nothing now and its job is `bidi`:

```
$ git log --format='%h %ad %s' --date=iso-strict --diff-filter=A -1 origin/main -- docs/dco-rulesets-and-the-delete.md
0c64f07 2026-09-04T19:42:52+02:00 Write down what deleting a local DCO gate costs a board [#25]
$ git log --format='%h %ad %s' --date=iso-strict --diff-filter=D -1 origin/main -- .github/workflows/unicode-guard.yml
f1a09c6 2026-09-05T01:36:27+02:00 Retire the shared unicode guard and keep the check local [#28]
$ git show origin/main:.github/workflows/guards.yml | sed -n '/^jobs:/,+2p'
jobs:
  bidi:
    name: Reject Trojan Source Unicode
$ gh pr checks 83 --repo iderex/wache --json name --jq '.[].name' | sort -u | grep Trojan
Reject Trojan Source Unicode
```

An unprefixed name is what a job that calls nothing reports, so this is the rule
above read from its other side rather than an exception to it. `hygiene` still
calls `pr-hygiene.yml` and still reports prefixed, so one pull request on this
board now carries both shapes at once, and the gate this page is about is the
called one. The transcript above is left addressed at `#75`, where it was taken
and where it still reproduces; what had drifted is the sentence reading it as a
statement about the tree today. The two log commands ask for the commit that
ADDED this page and the commit that DELETED that file rather than for the last
one to touch either, so the next edit here does not move what they print.

THIS IS SHARPER THAN THE SAME TRAP ON THE HYGIENE SIDE. There the shared job is
called `Deterministic PR hygiene` and the two rulesets require `pr-hygiene` and
`Deterministic PR-hygiene checks`, so a board comparing the names sees a
mismatch and looks further. Here the names are byte-identical and only the
prefix separates them.

## There is no concurrency hazard to go with it

The hygiene side carries one - below `v1.2.0` the shared file claimed the bare
`pr-hygiene-<number>` and cancelled the calling board's own run. This gate was
namespaced from its first commit and no copy spells that string:

```
$ git show origin/main:.github/workflows/dco.yml | sed -n '/^concurrency:/,+2p'
concurrency:
  group: wache-dco-${{ github.event.pull_request.number }}
  cancel-in-progress: true
$ for f in f-*.yml; do  # the sixty-two copies, this board's source excluded
    sed -n '/^concurrency:/,+2p' "$f" | sed -n 's/^ *group: *//p'
  done | sort | uniq -c | sort -rn
     60 dco-${{ github.event.pull_request.number }}
      1 ${{ github.workflow }}-${{ github.ref }}
      1 ${{ github.workflow }}-${{ github.event.pull_request.number }}
```

The one file that does spell `wache-dco-` is this board's own, counted among the
sixty-three by a comparison of bytes that cannot tell a source from a copy. So
the migration sequence for this gate needs a ruleset step and does NOT need the
pin-first step the hygiene sequence carries.

## Not evaluated

Whether any of the eleven rulesets is enforced. I read the required contexts and
not `enforcement`, so a ruleset in evaluate mode counts here the same as an
active one.

Whether classic branch protection requires the same context anywhere. I read
rulesets only, which is the same bound the hygiene reading declared for itself.

Whether any of the sixty-two copies is failing today. This reads files, listings
and rulesets, and no run's verdict.

Every listing comes from each board's default branch, so a ruleset targeting
another branch pattern is counted here by its contexts alone and a gate living
only on another branch is outside the reading.
