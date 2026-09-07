# What a board loses when it deletes its own DCO gate

`#25` asks that sixty-odd hand-written copies of `.github/workflows/dco.yml` be
replaced by calls to the one on this board. Every one of those migrations
deletes a workflow file. This is a reading of what else goes with it, taken on 4
September 2026 against the roster at `iderex/operations` `origin/main`
`e95252b6ed290e2e4aaf37a0f4879a201cc613ac`.

The answer is that on eleven boards the delete takes a REQUIRED status check off
the gate and puts nothing back, so every pull request there is left permanently
pending rather than red - nothing merges and nothing says why. The shared gate's
job carries the same name those rulesets require, which is what makes the trap
invisible to a board that checks the name.

THE SET IS THIRTEEN NOW AND THE `Re-read` SECTION AT THE END IS WHERE THAT IS
RECORDED. Every figure between here and there is 4 September's and is left as it
was taken. Two of the thirteen joined the set later the same evening, one of them
two and a half minutes after this page was committed, so the count above is a
reading with a date rather than a state of the fleet.

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

## Re-read on 7 September 2026: the set is thirteen

Taken against the roster at `iderex/operations` `origin/main`
`c71f67ecfaa2fccdc78aacef1a1464017ba75dc2`, by the same two steps as above - the
job each copy declares, then every ruleset on each of those boards:

```
$ wc -l < jobs.tsv
63
$ cut -f3 jobs.tsv | sort | uniq -c | sort -rn
     61 DCO sign-off
      1 dco
      1 DCO sign-off on every commit
$ awk -F'\t' '$4!=""' required.tsv | wc -l
35
$ awk -F'\t' '{n=split($4,a,"|"); for(i=1;i<=n;i++) if(a[i]==$2 || a[i]==$3) print $1"\t"a[i]}' required.tsv
Flowfin/hub                                 DCO sign-off
Flowfin/jellyfin-plugin-requests            DCO sign-off
Flowfin/jellyfin-plugin-server-pairing      DCO sign-off
Flowfin/jellyfin-plugin-share-links         DCO sign-off
Flowfin/jellyfin-plugin-smart-collections   DCO sign-off
Flowfin/jellyfin-plugin-sso                 DCO sign-off
Flowfin/jellyfin-plugin-stats               DCO sign-off
Flowfin/jellyfin-plugin-watchlist           DCO sign-off
Flowfin/jellyfin-plugin-whisper-subtitles   DCO sign-off
iderex/hoersaal                             DCO sign-off
iderex/reissbrett                           DCO sign-off
iderex/relais                               DCO sign-off
iderex/stammtisch                           DCO sign-off
```

The population of copies has not moved: 63 files, the same three job names, 35
of the 63 carrying a `required_status_checks` rule at all. What moved is two
rulesets. `Flowfin/jellyfin-plugin-smart-collections` and
`Flowfin/jellyfin-plugin-whisper-subtitles` require `DCO sign-off` now and did
not on 4 September.

**THE READING ABOVE WAS RIGHT WHEN IT WAS TAKEN AND STOPPED BEING RIGHT THE SAME
EVENING.** A ruleset carries a version history, so this is dated rather than
inferred:

```
$ git log --format='%h %ad %s' --date=iso-strict --diff-filter=A -1 origin/main -- docs/dco-rulesets-and-the-delete.md
0c64f07 2026-09-04T19:42:52+02:00 Write down what deleting a local DCO gate costs a board [#25]
$ gh api repos/Flowfin/jellyfin-plugin-smart-collections/rulesets/20465770/history --jq '.[0,1] | "\(.version_id)\t\(.updated_at)"'
48712320        2026-09-04T19:45:22.543+02:00
47958657        2026-08-28T14:22:44.172+02:00
$ gh api repos/Flowfin/jellyfin-plugin-smart-collections/rulesets/20465770/history/47958657 --jq '[.state.rules[]? | select(.type=="required_status_checks") | .parameters.required_status_checks[]?.context] | join(",")'
call / build,call / test,Reject Trojan Source Unicode,Audit workflows (zizmor)
```

Two minutes and thirty seconds. The version standing when this page was written
required four contexts and none of them was `DCO sign-off`; the version that
replaced it requires seventeen and one of them is. The second board is the same
shape two hours and thirteen minutes later:

```
$ gh api repos/Flowfin/jellyfin-plugin-whisper-subtitles/rulesets/20467991/history --jq '.[0,1] | "\(.version_id)\t\(.updated_at)"'
48723071        2026-09-04T21:55:53.227+02:00
47958670        2026-08-28T14:22:52.191+02:00
$ gh api repos/Flowfin/jellyfin-plugin-whisper-subtitles/rulesets/20467991/history/47958670 --jq '[.state.rules[]? | select(.type=="required_status_checks") | .parameters.required_status_checks[]?.context] | join(",")'
call / build,call / test,Reject Trojan Source Unicode,Audit workflows (zizmor)
```

WHAT THAT SAYS IS NOT THAT THE READING WAS SLOPPY. This figure moves on a board
that is not this one, by an edit that touches no file in any tree here, and
nothing in this repository reads a ruleset outside a run somebody starts by
hand. There is no interval on which a count of it stays true. That is why the
sequence in `README.md` asks the migrating board to run the command on ITSELF,
and why every number on this page carries the command beside it.

Both new boards pass the two tests the eleven were held to. Their `dco.yml`
declares the job name the ruleset requires, and nothing else in either workflow
directory declares it:

```
$ awk -F'\t' '$1 ~ /smart-collections|whisper-subtitles/' jobs.tsv
Flowfin/jellyfin-plugin-smart-collections       dco     DCO sign-off
Flowfin/jellyfin-plugin-whisper-subtitles       dco     DCO sign-off
$ for b in Flowfin/jellyfin-plugin-smart-collections Flowfin/jellyfin-plugin-whisper-subtitles; do
    for f in $(gh api "repos/$b/contents/.github/workflows" --jq '.[].name'); do
      [ "$f" = "dco.yml" ] && continue
      gh api "repos/$b/contents/.github/workflows/$f" --jq '.content' | base64 -d |
        grep -qE '^ +name: *"?DCO sign-off"? *$' && echo "$b $f"
    done
  done
```

No output, so on each of the thirteen `dco.yml` is still the only file declaring
the required name, and the delete is the whole of the loss rather than a rename.

BOTH OF THE TWO ARE ENFORCED, which the 4 September section declares it did not
read for its eleven:

```
$ for r in Flowfin/jellyfin-plugin-smart-collections/rulesets/20465770 Flowfin/jellyfin-plugin-whisper-subtitles/rulesets/20467991; do
    gh api "repos/$r" --jq '"\(.name)\t\(.enforcement)"'
  done
gate    active
gate    active
```

That answers it for two of the thirteen and leaves the other eleven where the
`## Not evaluated` line below puts them.

## The same night: all thirteen are active, and rulesets are the whole of it

A required context strands a pull request only where the rule requiring it is in
force and reaches the branch that pull request targets. The reading above
establishes neither, and the `## Not evaluated` section below said so on both
counts. Both are read now, against the roster at `iderex/operations`
`origin/main` `8751e8a57e5898402277c939834130ad07346098`, over the same sixty-three
copies - with `enforcement` and the target carried beside the contexts rather
than the contexts alone.

`boards` is the roster, `jobs.tsv` is the board, job id and job name of each
copy, and `thirteen.tsv` is the board and ruleset id of each pair the
intersection returns. THIS SENTENCE SAID THE SECTION ABOVE BUILDS `jobs.tsv` AND
NO SECTION ON THIS PAGE DID; the command that produces it is in
`## The extraction every count here rests on was never written down` below, and
until that section landed the promise this sentence ends on did not hold for the
one name every count here depends on:

```
$ ops=<operations>
$ git -C $ops ls-tree --name-only origin/main store/repo/ | grep -v README |
    sed 's|store/repo/||;s|[.]md$||' > ids
$ while read -r id; do
    printf '%s/%s
' "$(git -C $ops show "origin/main:store/repo/$id.md" |
      awk -F': ' '/^Owner:/{print $2}')" "$id"
  done < ids > boards
$ wc -l < boards
74
$ wc -l < jobs.tsv
63
$ while IFS=$'\t' read -r b jid nm; do
    for id in $(gh api "repos/$b/rulesets" --jq '.[].id' 2>/dev/null); do
      gh api "repos/$b/rulesets/$id" --jq "\"$b\t\(.id)\t\(.name)\t\(.enforcement)\t\(.target)\t\"+([.rules[]? |
        select(.type==\"required_status_checks\") |
        .parameters.required_status_checks[]?.context] | join(\"|\"))"
    done
  done < jobs.tsv > rulesets.tsv
$ awk -F'\t' 'NR==FNR{jid[$1]=$2; nm[$1]=$3; next}
    $6!="" { n=split($6,a,"|"); for(i=1;i<=n;i++) if(a[i]==jid[$1] || a[i]==nm[$1]) print $4 }' \
    jobs.tsv rulesets.tsv | sort | uniq -c
     13 active
$ awk -F'\t' 'NR==FNR{jid[$1]=$2; nm[$1]=$3; next}
    $6!="" { n=split($6,a,"|"); for(i=1;i<=n;i++) if(a[i]==jid[$1] || a[i]==nm[$1]) print $1"\t"$2 }' \
    jobs.tsv rulesets.tsv | sort -u > thirteen.tsv
$ wc -l < thirteen.tsv
13
```

Thirteen boards and thirteen `active` rulesets. Not one of the thirteen is in
evaluate mode, so the strand this page describes is what every one of them meets
rather than what some subset of them meets, and the count of thirteen needs no
discount.

THE RULE HAS TO REACH THE BRANCH AS WELL, and twelve of the thirteen say so with
one condition rather than a branch name. `Flowfin/jellyfin-plugin-sso` is the
exception in both directions: it names three branches literally, its default is
not `main`, and the strand there would reach two branches beyond the default:

```
$ while IFS=$'\t' read -r b rid; do
    def=$(gh api "repos/$b" --jq '.default_branch')
    inc=$(gh api "repos/$b/rulesets/$rid" --jq '[.conditions.ref_name.include[]?]|join(",")')
    printf '%-42s %-7s %s\n' "$b" "$def" "$inc"
  done < thirteen.tsv
Flowfin/hub                                main    ~DEFAULT_BRANCH
Flowfin/jellyfin-plugin-requests           master  ~DEFAULT_BRANCH
Flowfin/jellyfin-plugin-server-pairing     master  ~DEFAULT_BRANCH
Flowfin/jellyfin-plugin-share-links        master  ~DEFAULT_BRANCH
Flowfin/jellyfin-plugin-smart-collections  master  ~DEFAULT_BRANCH
Flowfin/jellyfin-plugin-sso                4.4     refs/heads/main,refs/heads/5.0,refs/heads/4.4
Flowfin/jellyfin-plugin-stats              master  ~DEFAULT_BRANCH
Flowfin/jellyfin-plugin-watchlist          master  ~DEFAULT_BRANCH
Flowfin/jellyfin-plugin-whisper-subtitles  master  ~DEFAULT_BRANCH
iderex/hoersaal                            main    ~DEFAULT_BRANCH
iderex/reissbrett                          main    ~DEFAULT_BRANCH
iderex/relais                              main    ~DEFAULT_BRANCH
iderex/stammtisch                          main    ~DEFAULT_BRANCH
```

`refs/heads/4.4` is in that list, so every one of the thirteen reaches the branch
its own pull requests are opened against. Eight of the thirteen would have been
read wrong by a reader who assumed `main`: seven default to `master` and one to
`4.4`.

```
$ while IFS=$'\t' read -r b rid; do gh api "repos/$b" --jq '.default_branch'; done < thirteen.tsv |
    sort | uniq -c
      1 4.4
      5 main
      7 master
```

NOTHING OUTSIDE A RULESET REQUIRES THE NAME, which is the other bound the section
below declared. Classic branch protection is a separate mechanism with its own
required set, and none of the sixty-three carries it on the branch these gates
run against:

```
$ while IFS=$'\t' read -r b jid nm; do
    def=$(gh api "repos/$b" --jq '.default_branch' 2>/dev/null)
    out=$(gh api "repos/$b/branches/$def/protection" \
            --jq '[.required_status_checks.contexts[]?]|join("|")' 2>&1)
    rc=$?
    if [ $rc -ne 0 ]; then
      echo "$out" | grep -q 'Branch not protected' && st=unprotected || st="ERR:$out"
    else st=protected; fi
    printf '%s\t%s\t%s\n' "$b" "$def" "$st"
  done < jobs.tsv > classic.tsv
$ cut -f3 classic.tsv | sed 's/ERR:.*/ERR/' | sort | uniq -c
     63 unprotected
```

No line came back `ERR`, so no board in that tally is an access failure being
read as an absence: each of the sixty-three answered with the API's own `Branch
not protected`.

WHAT THIS CHANGES IS THE COMMAND AND NOT THE COUNT. Every one of the three
readings agrees with what the sections above assumed, so the set is thirteen
either way. What was assumed rather than read is exactly what a migrating board
would also assume, and the sequence in `README.md` handed it a command printing
the contexts alone - so a board whose ruleset is in evaluate mode, or whose
ruleset targets branches its pull requests never touch, would have taken a
ruleset edit it does not owe and read a trap that is not there. That command now
prints the enforcement and the branches beside the contexts.

## The extraction every count here rests on was never written down

`jobs.tsv` is read by four commands on this page and produced by none of them.
The sentence above says the section above it builds the file, and that sentence
is what I am correcting: nothing on this page turned sixty-three fetched copies
into a board, a job id and a job name. So the tally of job names, the
intersection that returns the thirteen, the ruleset walk and the classic-
protection walk all read a file a reader cannot rebuild, and the line promising
that no name below is one this page has not given did not hold for the one name
every count depends on.

The hygiene side of the same question does give it, in
`docs/local-hygiene-answers.md`, which is how the absence here became visible:
one page builds `hyg-jobs.tsv` in front of the reader and the other does not.

Here it is, against the roster at `iderex/operations` `origin/main`
`a2d6dfe2472b09981b2bb5345427f7cd1bdcfd70` and its seventy-four boards. The
copies are fetched into `copies/` and EVERY job in each one is taken with its id
and, where it declares one, its name - rather than the first job of each file:

```
$ wc -l < boards
74
$ while read -r r; do
    gh api "repos/$r/contents/.github/workflows/dco.yml" --jq '.content' 2>/dev/null |
      base64 -d > "copies/$(echo "$r" | tr '/' '_').yml"
  done < boards
$ find copies -name '*.yml' -size 0 -delete; ls copies/*.yml | wc -l
63
$ for f in copies/*.yml; do
    b=$(basename "$f" .yml | sed 's|_|/|')
    awk -v b="$b" '
      /^jobs:/{inj=1; next}
      inj && /^[^ ]/{inj=0}
      inj && /^  [A-Za-z0-9_-]+:/{ jid=$1; sub(/:$/,"",jid);
        if(cur!="") print b"\t"cur"\t"jname; cur=jid; jname="" }
      inj && /^    name:/{ line=$0; sub(/^    name: */,"",line); gsub(/^"|"$/,"",line); jname=line }
      END{ if(cur!="") print b"\t"cur"\t"jname }' "$f"
  done > jobs.tsv
$ wc -l < jobs.tsv ; cut -f1 jobs.tsv | sort -u | wc -l
63
63
```

SIXTY-THREE ROWS OVER SIXTY-THREE COPIES, so no copy on this population declares
a second job and the first-job-only reading lost nothing here. That is the
opposite answer to the hygiene side, where four of twenty-two rows were jobs such
a reading never reached, and it is an answer rather than an assumption only
because the extraction above was run. A reader migrating a board still matches
every job of their own file: what is measured here is this population today, not
a rule about the shape of a `dco.yml`.

The extractor is the load-bearing part, so its own bounds are read rather than
trusted - every copy carries a `jobs:` block at column zero, none of the
sixty-three indents with tabs, and an independent count of the keys one level
inside that block agrees with it file by file:

```
$ grep -L '^jobs:' copies/*.yml ; grep -l $'\t' copies/*.yml
$ for f in copies/*.yml; do
    awk '/^jobs:/{i=1;next} i&&/^[^ #]/{i=0} i&&/^  [^ #]/{c++} END{print c+0}' "$f"
  done | sort | uniq -c
     63 1
```

Both greps print nothing. The tally the section above reads off this file
reproduces exactly, both exception rows included:

```
$ cut -f3 jobs.tsv | sort | uniq -c | sort -rn
     61 DCO sign-off
      1 dco
      1 DCO sign-off on every commit
$ awk -F'\t' '$3!="DCO sign-off"' jobs.tsv
iderex/Easy-Compliance-Manager  dco     DCO sign-off on every commit
iderex/swarm.asm                dco     dco
$ cut -f2 jobs.tsv | sort | uniq -c
     63 dco
$ awk -F'\t' '$3==""' jobs.tsv
```

EVERY JOB ID IS `dco` AND EVERY COPY DECLARES A `name:`, which decides the
direction the id half of the match can fail in. Nothing here is the
`iderex/hoersaal` shape the hygiene reading names, where a job declaring no
`name:` makes the job ID the thing a ruleset requires; on this population the id
half can only ADD a board, by matching a ruleset that happens to require the bare
string `dco`. It adds none. Re-run over every row of the file above, with the
matched half printed beside each line:

```
$ cut -f1 jobs.tsv | sort -u | while read -r b; do
    for id in $(gh api "repos/$b/rulesets" --jq '.[].id' 2>/dev/null); do
      gh api "repos/$b/rulesets/$id" --jq "\"$b\t\(.id)\t\(.name)\t\(.enforcement)\t\([.conditions.ref_name.include[]?]|join(\",\"))\t\"+([.rules[]? |
        select(.type==\"required_status_checks\") |
        .parameters.required_status_checks[]?.context] | join(\"|\"))"
    done
  done > rulesets.tsv
$ awk -F'\t' '$6!=""{print $1}' rulesets.tsv | sort -u | wc -l
35
$ awk -F'\t' 'NR==FNR{ k[$1"\t"$2]="id"; if($3!="") k[$1"\t"$3]="name"; next }
    $6!="" { n=split($6,a,"|"); for(i=1;i<=n;i++) if(($1"\t"a[i]) in k)
      printf "%-42s %-16s %-8s %-9s %s\n", $1, a[i], k[$1"\t"a[i]], $4, $5 }' \
    jobs.tsv rulesets.tsv | sort -u
Flowfin/hub                                DCO sign-off     name     active    ~DEFAULT_BRANCH
Flowfin/jellyfin-plugin-requests           DCO sign-off     name     active    ~DEFAULT_BRANCH
Flowfin/jellyfin-plugin-server-pairing     DCO sign-off     name     active    ~DEFAULT_BRANCH
Flowfin/jellyfin-plugin-share-links        DCO sign-off     name     active    ~DEFAULT_BRANCH
Flowfin/jellyfin-plugin-smart-collections  DCO sign-off     name     active    ~DEFAULT_BRANCH
Flowfin/jellyfin-plugin-sso                DCO sign-off     name     active    refs/heads/main,refs/heads/5.0,refs/heads/4.4
Flowfin/jellyfin-plugin-stats              DCO sign-off     name     active    ~DEFAULT_BRANCH
Flowfin/jellyfin-plugin-watchlist          DCO sign-off     name     active    ~DEFAULT_BRANCH
Flowfin/jellyfin-plugin-whisper-subtitles  DCO sign-off     name     active    ~DEFAULT_BRANCH
iderex/hoersaal                            DCO sign-off     name     active    ~DEFAULT_BRANCH
iderex/reissbrett                          DCO sign-off     name     active    ~DEFAULT_BRANCH
iderex/relais                              DCO sign-off     name     active    ~DEFAULT_BRANCH
iderex/stammtisch                          DCO sign-off     name     active    ~DEFAULT_BRANCH
```

Thirteen boards, thirty-five carrying a `required_status_checks` rule at all,
every enforcement `active`, and the ref conditions the section above prints -
`Flowfin/jellyfin-plugin-sso` naming its three branches literally and the other
twelve resting on one condition. Every match is on the NAME half and not one is
on the id, so the set is the same thirteen read against the same shape of
evidence, and the two extra fields still change no answer.

WHAT MOVES IS WHAT A READER CAN REBUILD, NOT THE COUNT. Every figure this page
carried survives the extraction being written down, which is the outcome to
expect and not the reason to write it: a count nobody can reproduce is a count
whose next re-reading starts from the beginning, and the bound it hid - one job
per file - was one the hygiene side had already found to bite. It was declared as
unread on neither page until it was closed on that one.

## Not evaluated

Whether the thirteen rulesets are enforced was this section's first line and is
read above: all thirteen are `active` on 7 September 2026. What is not evaluated
is that fact at any later moment. Enforcement moves by an edit on a board that is
not this one, and the section above measures such an edit landing two and a half
minutes after a reading here.

Whether classic branch protection requires the same context anywhere was this
section's second line and is read above for the default branch of each of the
sixty-three: none of them is protected by that mechanism at all. A classic rule
on any other branch of any of them is outside the reading, and so is any board
this roster does not carry. The hygiene reading still declares the narrower bound
for itself.

Whether any of the sixty-two copies is failing today. This reads files, listings
and rulesets, and no run's verdict.

Every listing comes from each board's default branch, so a gate living only on
another branch is outside the reading. Which branches a ruleset reaches is no
longer taken on its contexts alone for the thirteen, which are read above; the
other twenty-two boards carrying a `required_status_checks` rule were not read
for it, because no context any of them requires is the name its own gate
produces.

Whether a copy declares more than one job was read by nothing until the section
above, and was declared as unread by nothing either. It is read now over all
sixty-three: none of them does. What is not evaluated is that at any later
moment, and on any board outside this roster.

Whether the extractor above reads a `dco.yml` written in a shape none of these
sixty-three uses. It requires a `jobs:` key at column zero and two-space
indentation under it, both of which are checked against this population and
neither of which YAML requires; a copy using four spaces, a quoted key or a flow
mapping would be read as carrying no job at all and would drop out of every count
silently rather than loudly.
