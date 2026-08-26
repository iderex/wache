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
