# The `dco.yml` tail, read one file at a time

Taken on 27 August 2026 against the roster in `iderex/operations`, at
`origin/main` `918963c851b73e93d74e8478340f405d65ddf348`. Every figure below is
the output of a command written beside it. The figures move every time a board
lands anything, so re-run the command rather than citing this page.

This is the first line of the done-condition of `#25`: the 28 copies of
`.github/workflows/dco.yml` that nobody else holds, read, with each difference
placed as drift or as a declared board-local deviation. It removes nothing and
proposes nothing. What content the drift should be removed against is the
second line of that done-condition, and the last section here is the reason it
cannot be the majority copy unchanged.

THE POPULATION HAS MOVED SINCE, AND THE TWO `Re-read` SECTIONS NEAR THE END ARE
WHERE THAT IS RECORDED. Two singleton copies are outside everything between here
and there. Every figure in the sections before it is 27 August's and is left as
it was taken.

## The population, derived rather than typed

The board list comes from the roster:

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

One blob id per board, at each board's default branch:

```
boards | while read -r r; do
  sha=$(gh api "repos/$r/contents/.github/workflows/dco.yml" --jq '.sha' 2>/dev/null) || sha=absent
  printf '%s\t%s\n' "$r" "$sha"
done > dco.tsv

awk -F'\t' '$2!="absent"' dco.tsv | wc -l
60
awk -F'\t' '$2!="absent"{print $2}' dco.tsv | sort -u | wc -l
30
awk -F'\t' '$2!="absent"{print $2}' dco.tsv | sort | uniq -c | sort -rn |
  awk '{printf "%s ", $1} END{print ""}'
29 3 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
```

60 copies, 30 distinct contents, one cluster of 29, one of 3, and 28 boards each
holding a file nobody else holds. That reproduces the counts `#25` was opened
with. The other 12 boards carry no `dco.yml` at all and are outside this
reading.

The blob id the contents API returns is the git blob id of the file, so the
bytes fetched are the bytes tracked. I checked that rather than assuming it:

```
git hash-object dco/iderex_bremsweg.yml
5b53eab934abf424af0a207534d1864744c2395e
awk -F'\t' '$1=="iderex/bremsweg"{print $2}' dco.tsv
5b53eab934abf424af0a207534d1864744c2395e
```

I fetched and hash-checked all 28 singletons the same way, and none of them
disagreed with its recorded id.

## What separates the two placements

Every difference below is measured against the 29-cluster content, blob
`5b53eab934abf424af0a207534d1864744c2395e`, which `iderex/bremsweg` carries.
Calling it the reference is a choice of comparison point and not a proposal that
it is the answer; the last section is about that.

The test a difference is put to:

**A declared board-local deviation** states something that is true of that
board's own tree, tracker or configuration and that the reference content would
state falsely or unresolvably there. A file that is called something else, a
document that has not arrived, an automation identity nothing starts, a context
that is or is not in the required set, an open entry on that board's own
tracker.

**Drift** is everything else: the same meaning at a different length, a clause
moved, a sentence rewritten. Replacing it with the reference content would lose
no statement that is true there.

I separated the two by reading each file and then checking its claim against
the board it sits on, never by the size of the diff. A second, cheaper measure
is carried in the table beside each row - how much of the difference falls
outside comments - and it is a poor proxy on its own: 18 of the 28 differ from
the reference in comments only, and 12 of those 18 are deviations.

```
nc() { sed -E 's/^[[:space:]]*#.*$//' "$1" | grep -v '^$'; }
nc dco/iderex_bremsweg.yml > nc-canon.txt
nc "dco/$board.yml" | diff nc-canon.txt - | grep -c '^[<>]'
```

## The 28, placed

| Board | Placed as | Declared in the file | Lines of difference outside comments | What the difference is |
| --- | --- | --- | --: | --- |
| `Flowfin/core` | deviation | declared | 0 | Names where `#746` resolves and says no issue on that tracker asked for the gate. |
| `Flowfin/hub` | deviation | declared | 0 | Says the pull-request hygiene leg the shared posture refers to is not on that board yet, and names the issue that brings it. |
| `Flowfin/jellyfin-plugin-metadata-sync` | deviation | declared | 40 | An address-syntax leg, a `Co-authored-by` trailer leg, a rewritten failure message, and a note that the context is not in that board's required set. |
| `Flowfin/jellyfin-plugin-server-pairing` | deviation | declared | 0 | Records that this job's context is in that board's required set, and hands over the command that reads it rather than restating it. |
| `Flowfin/jellyfin-plugin-smart-collections` | deviation | declared | 0 | Names where `#746` resolves and says no issue on that tracker asked for the gate. |
| `Flowfin/jellyfin-plugin-watchlist` | deviation | declared | 0 | Names where `#746` resolves. |
| `Flowfin/jellyfin-plugin-whisper-subtitles` | deviation | declared | 0 | Records that the header once cited a number resolving to nothing there and described a neighbouring check that did not yet exist. |
| `Flowfin/lab` | deviation | declared, one half stale | 4 | Drops the `github-actions[bot]` arms of the exemption because nothing in that tree starts that actor. The header also says the certificate text is not in the tree; it is. |
| `iderex/ausgleich` | deviation | declared | 2 | No contributor guide in that tree, so the failure message names the certificate version and `./DCO` instead of `CONTRIBUTING.md`. |
| `iderex/beiblatt` | drift | n/a | 0 | The header says the same thing at greater length, plus one sentence recording that it was once wrong. |
| `iderex/blende` | deviation | declared | 0 | Neither the certificate text nor a contributor guide is in that tree, and the header says the failure message names two files that are not there. |
| `iderex/drehbank` | deviation | declared | 2 | No certificate text in that tree, so the failure message points at a section of `CONTRIBUTING.md` and at nothing else. |
| `iderex/entwurf` | deviation | declared | 2 | The failure message names that board's own `docs/legal/contribution-terms.md` and its root `DCO`. |
| `iderex/hoersaal` | deviation | NOT declared | 0 | Two SPDX header lines. It is that board's convention rather than drift, and the file does not say so. |
| `iderex/lesesaal` | drift | n/a | 0 | One clause reworded, and `(#746)` dropped. |
| `iderex/lichttisch` | deviation | NOT declared | 2 | The certificate file there is `DCO.md`, so the header and the failure message name it. Nothing in the file says why. |
| `iderex/messlatte` | deviation | declared | 2 | Header rewritten around that board's own numbers, and the failure message names the published text, `./DCO`, `./CONTRIBUTING.md` and the project licence. |
| `iderex/nachtwache` | deviation | declared | 5 | `permissions: {}` at the top level with `contents: read` moved onto the job, with the reason written beside it. |
| `iderex/niedergang` | deviation | declared | 0 | Says the hygiene gate the posture is shared with does not exist there yet, and names the issue. |
| `iderex/plattenschrank` | deviation | declared | 0 | Names that board's own issue for `./DCO`, and an entry of its board question that is still open. |
| `iderex/rechenblatt` | deviation | NOT declared | 2 | The certificate file there is `DCO.md`, so the header and the failure message name it. Nothing in the file says why. |
| `iderex/reissbrett` | deviation | declared | 0 | Names that board's own issue for `./DCO` in place of `#746`. |
| `iderex/relais` | drift | n/a | 0 | The header is reworded and points at that board's sign-off section; `(#746)` dropped. |
| `iderex/retusche` | drift | n/a | 0 | The header is reworded. Nothing is added and nothing is lost. |
| `iderex/schallweg` | drift | n/a | 0 | The header is reworded and asserts that both paths the failure message names are in that tree. They are. |
| `iderex/scheinbild` | deviation | declared | 0 | Records that whether that board takes outside changes at all is an open entry of its board question, and that this gate settles none of it. |
| `iderex/schichtwerk` | deviation | declared | 2 | No contributor guide in that tree, so the failure message names `./DCO` and nothing else. |
| `iderex/stammtisch` | drift | n/a | 0 | The header is reworded. Nothing is added and nothing is lost. |

22 deviations and 6 drift.

## The three that change what the gate does

Everything else in the table changes a comment, a path in the failure message,
or both. Three change the judgement:

**`Flowfin/jellyfin-plugin-metadata-sync`** adds two legs the reference has no
form of. It refuses an author address that is not an address, and it holds
`Co-authored-by` to the same rule as `Signed-off-by`. Its comment says what the
first one is for: both sides of the reference's comparison are built from the
same commit, so a commit whose author address is a stray command-line flag
agrees with itself and passes. The comment also states the bound in the same
breath - it reads syntax, and nothing resolves the mailbox.

**`Flowfin/lab`** removes two arms of the bot exemption, leaving only
`dependabot[bot]`, and writes down why: nothing in that tree starts
`github-actions[bot]`, and an exemption whose actor nothing starts is an open
route nobody watches. What starts the one that is left is checkable, and it is
there:

```
gh api repos/Flowfin/lab/contents/.github --jq '.[].name'
ISSUE_TEMPLATE
dependabot.yml
pull_request_template.md
workflows
```

**`iderex/nachtwache`** sets `permissions: {}` at the top level and moves
`contents: read` onto the job, so a job added later starts from nothing and has
to name what it needs. The gate's verdict is unchanged; what changes is what a
future job inherits.

All three are strictly tighter than the reference. A migration that overwrites
them removes a guard, and none of the three is expressible as an input to a
shared workflow.

## Three deviations that nothing declares

These are the ones a byte comparison cannot tell from drift, and they are the
reason `#25` asks for 28 files read rather than 28 files overwritten. Each is
load-bearing on its own board and each is silent about it.

`iderex/lichttisch` and `iderex/rechenblatt` both name `DCO.md` where the
reference names `./DCO`, in the header and in the failure message. That is not a
typo; it is the name of the file on those two boards, and the reference content
would send a contributor to a path that is not there:

```
gh api repos/iderex/lichttisch/contents/ --jq '.[].name' | grep -iE '^DCO'
DCO.md
gh api repos/iderex/rechenblatt/contents/ --jq '.[].name' | grep -iE '^DCO'
DCO.md
```

`iderex/hoersaal` adds two SPDX header lines. It is that board's convention for
workflow files rather than a difference in this one, which is a fact about the
board and not about `dco.yml`:

```
gh api repos/iderex/hoersaal/contents/.github/workflows --jq '.[].name' |
  while read -r n; do
    printf '%s\t%s\n' "$n" \
      "$(gh api "repos/iderex/hoersaal/contents/.github/workflows/$n" --jq '.content' |
         base64 -d | grep -c '^# SPDX-License-Identifier:')"
  done | awk -F'\t' '{t++; s+=$2} END{print s" of "t" carry one"}'
17 of 18 carry one
```

Under the third line of `#25`'s done-condition each of these three owes a
sentence saying what it deviates from and why. Writing that sentence is a change
to those boards' trees.

## One declaration that has gone stale

`Flowfin/lab`'s header says the certificate text is not in that tree yet and
names the issue that adds it. The file is there:

```
gh api repos/Flowfin/lab/contents/DCO --jq '.content' | base64 -d | head -2
Developer Certificate of Origin
Version 1.1
```

So one half of that board's deviation is a live statement about its own tree and
the other half is a statement that stopped being true. Both sit in one comment,
and a byte comparison sees one difference. The exemption half is the behaviour
change above and stands; the sentence about the missing text does not.

## Why the drift cannot be removed against the majority copy unchanged

The reference content carries three statements that are false on most of the
boards holding it, and the tail is where they were repaired.

**`(#746)` resolves on one of the 29 boards that cite it.**

```
awk -F'\t' '$2=="5b53eab934abf424af0a207534d1864744c2395e"{print $1}' dco.tsv |
  while read -r r; do
    gh api "repos/$r/issues/746" --jq '.title' >/dev/null 2>&1 &&
      echo resolves || echo absent
  done | sort | uniq -c
      28 absent
       1 resolves
```

`Flowfin/jellyfin-plugin-sso` is the one that resolves it, with the title `Adopt
a DCO sign-off (legal-authorization mechanism) with an automated check`. The
other 28 have no issue 746 at all. The 3-cluster - `iderex/kanzlei`,
`iderex/sternwarte`, `iderex/stoffbuch` - is the 29-cluster with that reference
deleted, and four of the singletons replaced it with a reference that resolves
on their own tracker instead. `#25` reads the 29-to-3 difference as drift with
no consequence, and against the trackers it sits on it is the majority that
drifted.

**The failure message names `./DCO`, and 13 of the 29 boards do not track it.**

```
awk -F'\t' '$2=="5b53eab934abf424af0a207534d1864744c2395e"{print $1}' dco.tsv |
  while read -r r; do
    printf '%s\n' "$(gh api "repos/$r/contents/" --jq '[.[].name]|join(",")' 2>/dev/null |
      tr ',' '\n' | grep -icE '^DCO$')"
  done | awk '{t++; s+=$1} END{print s" of "t" track ./DCO"}'
16 of 29 track ./DCO
```

**The same message names `CONTRIBUTING.md`, and 9 of the 29 do not track that
either.** Eight boards track neither: `Flowfin/.github`,
`Flowfin/jellyfin-plugin-stats`, `iderex/bremsweg`, `iderex/findbuch`,
`iderex/gegenprobe`, `iderex/lehrkanzel`, `iderex/pruefstand`,
`iderex/spurenarchiv`. The board holding the reference copy is among them:

```
gh api repos/iderex/bremsweg/contents/ --jq '[.[].name]|join(" ")'
.cargo .github .gitignore Cargo.lock Cargo.toml LICENSE NOTICE.md README.md SECURITY.md clippy.toml crates data docs rust-toolchain.toml xtask
```

That is what `iderex/blende` declares in one paragraph and what eight boards
carry in silence. The difference between them is a disclosure and not a
behaviour: all nine print the same message naming files that are not there, and
one of them says so.

So the fourth line of `#25`'s done-condition - no board loses a disclosure it
currently makes - is not the whole of the risk. Standardising on this content
unchanged would also spread a false statement onto boards that had corrected it,
which is the same defect running the other way.

## Re-read on 30 August 2026: two copies this reading never covered

The population above is 27 August's. Re-derived three days later against the
roster at `origin/main` `47855f5510e7ee0c098226954f92adafab2be9c4`, with the
same two commands:

```
boards | wc -l
73

boards | while read -r r; do
  sha=$(gh api "repos/$r/contents/.github/workflows/dco.yml" --jq '.sha' 2>/dev/null) || sha=absent
  printf '%s\t%s\n' "$r" "$sha"
done > dco.tsv

awk -F'\t' '$2!="absent"' dco.tsv | wc -l
63
awk -F'\t' '$2!="absent"{print $2}' dco.tsv | sort -u | wc -l
33
awk -F'\t' '$2!="absent"{print $2}' dco.tsv | sort | uniq -c | sort -rn |
  awk '{printf "%s ", $1} END{print ""}'
29 3 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
```

63 copies over 73 boards, 33 distinct contents, the same cluster of 29 and the
same cluster of 3, and 31 boards each holding a file nobody else holds. The
reference content is unmoved and is still held by 29:

```
awk -F'\t' '$2=="5b53eab934abf424af0a207534d1864744c2395e"' dco.tsv | wc -l
29
```

Three of those 31 are not in the table above, and every one of the 28 the table
holds is still a singleton, so nothing left the tail:

```
awk -F'\t' '$2!="absent"{print $2}' dco.tsv | sort | uniq -c |
  awk '$1==1{print $2}' > singles
grep -F -f singles dco.tsv | cut -f1 | sort > singletons-now
comm -13 table28 singletons-now
iderex/Easy-Compliance-Manager
iderex/swarm.asm
iderex/wache
comm -23 table28 singletons-now
```

**`iderex/wache` IS NOT A COPY, AND THE METHOD COUNTS IT AS ONE.** The file this
board tracks is the shared gate itself, the thing the other copies would be
migrated onto, and a comparison of bytes has no way to tell a source from a copy:

```
git rev-parse bc3c387:.github/workflows/dco.yml
1f6ba061c770067deb8cb5eed4557273fbe99fef
awk -F'\t' '$1=="iderex/wache"{print $2}' dco.tsv
1f6ba061c770067deb8cb5eed4557273fbe99fef
```

THE REF IS THE COMMIT THIS SECTION LANDED ON AND NOT `origin/main`, which is
what it named until this line was repaired. Both ids above were read on 30
August, so both have to be addressed there. This board's own `dco.yml` moved
the same evening in the `#49` repair, and the command against a moving
reference now prints a third id belonging to neither side of the equality:

```
git rev-parse origin/main:.github/workflows/dco.yml
e3283f44e03db67e0d084aa5270a8a953861c929
git log --format='%h %ad %s' --date=short -1 origin/main -- .github/workflows/dco.yml
e529cb8 2026-08-30 Pin the `<id>` of the exempt address's minted form to digits [#49]
```

That leaves two copies the reading never placed.

## The two, placed

| Board | Placed as | Declared in the file | Lines of difference outside comments | What the difference is |
| --- | --- | --- | --: | --- |
| `iderex/Easy-Compliance-Manager` | deviation | declared, one half stale | 31 | Refuses a range holding no non-merge commit, empty top-level permissions with `contents: read` on the job, and a runner-hardening step. The header attributes its posture to a document that is not in that tree. |
| `iderex/swarm.asm` | deviation | declared | 51 | The rule lives in a PowerShell script that board's own test suite exercises, and the bot exemption is keyed on the account that opened the pull request as well as on the author address. |

Both are deviations under the test the earlier sections use, and both are
strictly tighter than the reference rather than looser.

**`iderex/Easy-Compliance-Manager` refuses an empty range, and the reference
passes one.** After the guarded walk it adds:

```
if [ -z "$commits" ]; then
  echo "::error::No non-merge commit found in ${BASE_SHA}..${HEAD_SHA}. Nothing was verified, so this fails rather than passes."
  exit 1
fi
```

The reference walks the same range, iterates nothing, and prints that every
commit carries a sign-off. That is a green verdict over an empty set - the shape
the reference's own guarded `rev-list` exists against, one step further in. The
same file sets `permissions: {}` at the top level with `contents: read` on the
job, which is `iderex/nachtwache`'s deviation arriving on a second board.

Every path and number its header names is in that tree or on that tracker,
except one:

```
gh api repos/iderex/Easy-Compliance-Manager/contents/DCO --jq .name
DCO
gh api repos/iderex/Easy-Compliance-Manager/contents/CONTRIBUTING.md --jq .name
CONTRIBUTING.md
gh api repos/iderex/Easy-Compliance-Manager/issues/761 --jq .title
Require DCO sign-off on every commit and enforce it in CI
gh api repos/iderex/Easy-Compliance-Manager/issues/262 --jq .title
Remove all external review/quality services (CodeRabbit, SonarCloud, Copilot) from the repo
gh api repos/iderex/Easy-Compliance-Manager/contents/ --jq '[.[].name]|join(" ")' | tr ' ' '\n' | grep -c '^CLAUDE.md$'
0
```

The header attributes its internal-only posture to `CLAUDE.md directive 2`, and
that file is not in the tree. It is the same shape as `Flowfin/lab` above - one
half of the declaration is a live statement about the board and the other half
has stopped being true, and a comparison of bytes sees one difference.

**`iderex/swarm.asm` moves the rule out of the workflow and into a script its
test suite runs.** Both paths its header names are there:

```
gh api repos/iderex/swarm.asm/contents/tools/check-dco.ps1 --jq '.name, .size'
check-dco.ps1
9613
gh api repos/iderex/swarm.asm/contents/tests/Swarm.Tests/DcoSignOffTests.cs --jq '.name, .size'
DcoSignOffTests.cs
18724
```

so the deviation is not a preference about shell dialects. The file states the
reason - enforcement written inside a workflow is in a language that board has
no suite for, and the same script is what its fixtures exercise - which is the
means check recorded where the artefact is.

**Its exemption is keyed on the account that opened the pull request, and the
reference's is not.** The opener reaches the script beside the two SHAs, with
the reason written in the file:

> PR_AUTHOR is what makes the bot exemption safe. A commit's author email
> is a field its author types, so an exemption keyed on that alone is
> available to anyone who spells Dependabot's address; the opening account
> is GitHub's own record of who pressed the button, and a pull request
> opened by a person carries no exemption whatever its commits claim.

That is a behaviour the reference has no form of and that no input to a shared
workflow expresses, so it belongs with the three under `The three that change
what the gate does` rather than migrating away. What it says about the shared
gate's own exemption is `#49`, which is where that reading and its proof live;
nothing here settles it.

## What the re-reading does to the first done-condition

`#25`'s first line asks that the singleton copies be read and each difference
placed. It was met against the 28 that existed on 27 August, and it is met
against 30 of the 31 counted above once these two rows are added - the
thirty-first being this board's own source file. What moved is the population,
not the reading.

ONE OF THE TWO ARRIVED AFTER THE SHARED GATE EXISTED, which is the part worth
reading rather than the arithmetic. Each file's own history says when it landed:

```
gh api "repos/iderex/Easy-Compliance-Manager/commits?path=.github/workflows/dco.yml&per_page=100" \
  --jq '[.[].commit.committer.date] | last'
2026-08-26T21:53:17Z
gh api "repos/iderex/swarm.asm/commits?path=.github/workflows/dco.yml&per_page=100" \
  --jq '[.[].commit.committer.date] | last'
2026-08-28T15:22:57Z
```

`iderex/Easy-Compliance-Manager`'s copy predates this page by half an hour and
was missed. `iderex/swarm.asm`'s was written on 28 August, a day after the
shared gate was merged here, and it is a 33rd distinct content rather than a
caller. Nothing on any board calls the shared gate yet:

```
awk -F'\t' '$2!="absent"{print $1}' dco.tsv | while read -r r; do
  gh api "repos/$r/contents/.github/workflows/dco.yml" --jq '.content' | base64 -d |
    grep -c 'wache/.github/workflows/dco.yml@'
done | awk '{s+=$1} END{print s" caller(s)"}'
0 caller(s)
```

So the tail grew while the migration waited, by one hand-written copy in the
three days since the standard existed. That is a reading of three days and not
a trend.

## Re-read on 4 September 2026: the tail has not moved, and no file name was hiding a caller

Taken against the roster in `iderex/operations` at `origin/main`
`3dbef1117780a0cb202dd4362d5bb27b56778589`. Both re-readings above asked each
board for one path, `.github/workflows/dco.yml`, and both named the file name as
their own bound. This one lists every board's whole workflow directory and then
reads every file in it, so that bound is gone rather than argued with.

```
boards | wc -l
74

boards | while read -r r; do
  gh api "repos/$r/contents/.github/workflows" --jq '.[] | "\(.name)\t\(.sha)"' 2>/dev/null |
    while IFS="$(printf '\t')" read -r n s; do printf '%s\t%s\t%s\n' "$r" "$n" "$s"; done
done > wf.tsv

cut -f1 wf.tsv | sort -u | wc -l
73
wc -l < wf.tsv
924
cut -f3 wf.tsv | sort -u | wc -l
789
```

74 roster boards, 73 of them holding a `.github/workflows` directory, 924 files
between them and 789 distinct blobs. `iderex/lagetisch` is the
seventy-fourth and tracks no such directory at all, so it is outside this
reading rather than missing from it:

```
gh api repos/iderex/lagetisch/contents/ --jq '[.[].name]|join(" ")'
.gitattributes .gitignore Cargo.lock Cargo.toml README.md SECURITY.md examples knowledge src tests tokens.txt
```

I fetched every one of the 789 blobs and checked each against the id the
listing gave for it, so the bytes read below are the bytes tracked:

```
while read -r s; do
  gh api "repos/<the board holding it>/git/blobs/$s" --jq '.content' | base64 -d > "blobs/$s.txt"
done < shas.txt

bad=0
while read -r s; do
  [ "$(git hash-object "blobs/$s.txt")" = "$s" ] || bad=$((bad+1))
done < shas.txt
echo "$bad mismatch(es) of $(wc -l < shas.txt)"
0 mismatch(es) of 789
```

## The population is where 30 August left it

```
awk -F'\t' '$2=="dco.yml"{print $1"\t"$3}' wf.tsv | sort > dco.tsv
wc -l < dco.tsv
63
cut -f2 dco.tsv | sort -u | wc -l
33
cut -f2 dco.tsv | sort | uniq -c | sort -rn | awk '{printf "%s ", $1} END{print ""}'
29 3 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
awk -F'\t' '$2=="5b53eab934abf424af0a207534d1864744c2395e"' dco.tsv | wc -l
29
```

63 copies, 33 distinct contents, the same cluster of 29 and the same cluster of
3, 31 singletons. The 31 are the same 31, so every difference this page places
is still a difference somebody holds and no board has arrived unplaced:

```
cat table28 table2 <(echo iderex/wache) | sort > placed31
cut -f2 dco.tsv | sort | uniq -c | awk '$1==1{print $2}' > singles
grep -F -f singles dco.tsv | cut -f1 | sort > singletons-now
comm -13 placed31 singletons-now
comm -23 placed31 singletons-now
```

Both directions are empty. `iderex/wache` is in that set for the reason the 30
August section gives and is this board's own source file rather than a copy.

THE NAME WAS NOT HIDING A `dco.yml` EITHER, which the earlier readings could
only assume:

```
grep -iE 'dco|sign.?off|certificate' wf.tsv | awk -F'\t' '{print $2}' | sort | uniq -c
     63 dco.yml
```

## The tail grew once and has not grown again

The 30 August reading found `iderex/swarm.asm`'s copy written a day after the
shared gate merged here, and said in as many words that one copy in three days
is a reading of three days and not a trend. Seven days on there is a second
point. No board other than this one has touched its `dco.yml` since 28 August:

```
cut -f1 dco.tsv | while read -r r; do
  printf '%s\t%s\n' "$r" \
    "$(gh api "repos/$r/commits?path=.github/workflows/dco.yml&per_page=100" \
         --jq '[.[].commit.committer.date] | first')"
done > dco-dates.tsv

sort -t"$(printf '\t')" -k2,2r dco-dates.tsv | head -3
iderex/wache	2026-08-30T16:15:54Z
iderex/swarm.asm	2026-08-28T15:58:51Z
iderex/Easy-Compliance-Manager	2026-08-26T21:53:17Z
awk -F'\t' '$2>"2026-09-01"' dco-dates.tsv | wc -l
0
```

So the tail is static rather than growing, and it is static in both directions:
nothing arrived and nothing left.

## Still nothing calls the shared gate, now read over every workflow file

The 30 August and 1 September readings took this zero twice, once by fetching
each `dco.yml` and once by a code search, and both said what they could not see.
This one walks all 924 files:

```
while IFS="$(printf '\t')" read -r r n s; do
  grep -ohE 'uses:.*wache/\.github/workflows/[A-Za-z0-9._-]+@[A-Za-z0-9._-]+' "blobs/$s.txt" |
    while read -r u; do printf '%s\t%s\t%s\n' "$r" "$n" "$u"; done
done < wf.tsv | sort -u | grep -c 'dco\.yml@'
0
```

Zero across every workflow file of every roster board, eight days after the gate
merged here. The same walk finds twenty call sites for `pr-hygiene.yml` and one
for `unicode-guard.yml`, so it is a walk that does find callers where they
exist and not a query that returns nothing.

## No tag has reached the `#49` repair

The 30 August reading closed with a board following the migration route either
pinning a commit or pinning `v1.3.0` and carrying the matcher `#49` repaired.
That is unchanged:

```
git ls-remote --tags origin | grep -c 'refs/tags/'
8
git tag --contains 71baad6
```

Four tags exist, `v1.0.0` through `v1.3.0`, and the last command prints nothing:
no tag reaches `71baad6`. `README.md` says so at the pin site rather than
leaving it to be found, which is what `#58` landed.

## What the re-reading does to the done-condition

The first line still covers the population, and covers it without the file-name
bound the earlier two readings carried. Lines two, three and four are unmoved and
none of them is unmoved for want of an answer here: the three deviations that
declare nothing and the two declarations that have gone stale are all still in
the state this page places them in, read again tonight out of the fetched bytes.

```
grep -c 'DCO text is not in this tree yet' <Flowfin/lab's dco.yml>
1
gh api repos/Flowfin/lab/contents/DCO --jq '.name'
DCO
grep -c 'CLAUDE.md directive 2' <iderex/Easy-Compliance-Manager's dco.yml>
1
gh api repos/iderex/Easy-Compliance-Manager/contents/CLAUDE.md --jq '.name'
{"message":"Not Found", ... "status":"404"}
```

Writing the sentence each of those five owes is a change to those boards' trees,
which is where lines two, three and four have been since 27 August.

## Not evaluated

Whether any of the 60 copies has stopped refusing what it claims to refuse. This
is a reading of files, and of the trees and trackers those files make claims
about. It is not a reading of any run. A board whose gate is failing today looks
the same here as one whose gate is green.

Whether a content other than the largest cluster's would be a better comparison
point. I chose the 29-cluster because it is the largest, and the section above is
the argument that largest is not the same as correct.

Whether the two boards naming `DCO.md` should rename the file or the workflow.
That is a decision on those boards.

The first of those three is written against the 60 copies of 27 August, and the
re-reading of 30 August widens it rather than replacing it: none of the 63
copies counted there has been read as a run either, the two placed above
included. Both were judged by reading their files and by checking the claims
those files make against their own trees and trackers. Whether either one
refuses what it says it refuses is not measured here.

The 4 September reading widens the first of those three again rather than
replacing it: none of the 63 copies has been read as a run there either, and
whether any of them still refuses what it claims to refuse is not measured on
this page at any date.

Two bounds are that reading's own. Every listing and every blob comes from each
board's DEFAULT branch, which is what the contents API answers with when no ref
is given, so a gate living only on another branch is outside it. And 924 files
is what the 73 directories held at the moment they were listed; a file added
while the walk was running is outside it too, and the walk took about twenty
minutes.
