# Scanning across the boards: the baseline, the state, and the recorded reasons

Two readings stand in this page. The board list and the first state reading were
taken on 27 August 2026 against the roster in `iderex/operations` at
`origin/main` `bc2b8bd1f4238d8148f1d4e6fe765a1e8fb614e0`. Everything below the
baseline was re-read on 28 August 2026 at `origin/main`
`b07f8d05ec76b8e47213b863e14ec5ae9d0dd1f8`, after the settings sweep recorded on
`#32` had run, and the figures moved. Every figure carries the command that
produced it, and a settings change moves any of them without touching this file,
so re-run them rather than citing this page.

This is the home `#32` asks for: the baseline written down, the state measured,
and the register a board writes into when it deliberately does not scan. It
rolls nothing out. The settings themselves are changed in each repository's own
configuration and no change here reaches them.

## The baseline

Decided on `#32`, and it has two rows because the deciding fact is the price.
The standing rule is that nothing at GitHub costs money, and that cuts the
roster exactly where visibility cuts it.

**Public boards.** CodeQL default setup for every supported language, secret
scanning with push protection, and Dependabot. All three are free on a public
repository, so the baseline is what the platform already gives and the only
thing between a board and it is a setting.

**Private boards.** Dependabot, which is free everywhere, plus the local gate's
own legs. NOT CodeQL and NOT secret scanning, because on a private repository
both are GitHub Advanced Security, which is the paid product. THIS IS A BASELINE
CHOSEN AND NOT A GAP LEFT, and the difference matters to whoever reads the state
next: a private board reporting both features off is at its baseline, not behind
it, and a sweep must not open a finding on it.

A private board that later goes public crosses into the public row on the day
its visibility changes, and the sweep is what should say so.

A board that deliberately sits outside its own row records the reason in the
register below, in the shape the first entry uses - reason, decider, date, and
the condition under which the reason expires.

## Reading the state

`security_and_analysis` on the repository object answers for a public board and
does not answer for a private one. It is `null` on every private board here, so
it cannot distinguish a board with scanning off from a board whose settings the
token cannot see:

    gh api repos/iderex/bremsweg --jq '.security_and_analysis // "absent"'
    absent

The alerts endpoints answer directly, and their refusals name the state:

    gh api "repos/$r/code-scanning/alerts?per_page=1"
    gh api "repos/$r/secret-scanning/alerts?per_page=1"

    "Code scanning is not enabled for this repository."   -> off
    "no analysis found"                                   -> on, nothing analysed yet
    a JSON array                                          -> on, with analyses
    "Secret scanning is disabled on this repository."     -> off
    a JSON array                                          -> on

## What the 73 roster boards report

Re-read 28 August 2026. `scanning.tsv` is one tab-separated line per board -
the board, its visibility, whether it is archived, its detected language, and
the two verdicts the endpoints above return - and the counts are read off it:

    cut -f2,3,5,6 scanning.tsv | sort | uniq -c
     37 private  false  off               off
      1 private  true   off               off
     35 public   false  analyses-present  on

The one archived board is `iderex/agent-operations`, and it is private and off
on both like the other 37.

THE THREE PUBLIC BOARDS THAT REPORTED `no analysis found` ON 27 AUGUST NO LONGER
DO, AND THIS PAGE SAID THEY DID. `iderex/wache`, `iderex/Typeset` and
`iderex/iderex` all carry CodeQL analyses now, created on 28 August at 13:06 and
13:07 UTC. It was found by re-running the sweep rather than by reading the
sentence again:

    gh api "repos/$r/code-scanning/analyses?per_page=5" --jq '[.[] | .category] | join(" ")'
    iderex/wache      /language:actions
    iderex/Typeset    /language:swift /language:actions
    iderex/iderex     /language:actions

`iderex/Typeset` also reports `Swift` where the entry below recorded no detected
language, which is the same reading going stale in the other direction.

## The public side, across both owners rather than across the roster

The settings sweep recorded on `#32` was run over every non-archived public
board of both owners, which is a different population from the roster and a
larger one - 39 boards against the roster's 35, the four extra being
`iderex/.github`, `iderex/linux`, `iderex/myCat` and `iderex/seerr`:

    for o in iderex Flowfin; do
      gh api "users/$o/repos?per_page=100&type=owner" --paginate \
        --jq '.[] | select(.archived==false and .visibility=="public") | .full_name'
    done | sort | wc -l
    39

Secret scanning with push protection is on for all 39, with no exception:

    gh api "repos/$r" --jq '[.security_and_analysis.secret_scanning.status,
                             .security_and_analysis.secret_scanning_push_protection.status] | @tsv'
    39  enabled  enabled

Code scanning analyses are present on 38 of the 39. The one that has none is
`iderex/.github`, and it is the refusal the sweep predicted rather than a board
that was missed - see the register entry below.

CODEQL DEFAULT SETUP AND CODEQL ANALYSES ARE TWO DIFFERENT READINGS AND THEY DO
NOT AGREE TODAY. Read at 17:02 UTC on 28 August:

    gh api "repos/$r/code-scanning/default-setup" --jq '[.state, (.updated_at // "null")] | @tsv'
     7  configured
    32  not-configured

Seven of the 39 report default setup configured. On several of the 32 that
report it not configured, the analysis created today carries default setup's own
analysis key:

    gh api "repos/iderex/kanzlei/code-scanning/analyses?per_page=1" \
      --jq '.[0] | [.created_at, .analysis_key] | @tsv'
    2026-08-28T13:07:46Z    dynamic/github-code-scanning/codeql:analyze

    gh api "repos/iderex/kanzlei/code-scanning/default-setup" --jq '[.state, (.updated_at // "null")] | @tsv'
    not-configured    null

WHICH OF THE TWO THOSE 32 BOARDS ARE ACTUALLY IN IS NOT DECIDED HERE. Both
readings are reproduced above, this page asserts neither over the other, and no
mechanism is proposed for the disagreement. What it does establish is the thing
the second half of `#32` is about: `analyses present` is a reading of the PAST
and does not say that scanning is configured NOW, so a sweep that asks only the
alerts endpoint cannot see a board whose setup has come back off. The sweep in
`#31` has to read the setup state rather than infer it.

## The line is still visibility, and both sides of it have changed meaning

On 27 August the split was exactly visibility, 73 boards out of 73, with every
public board on and every private board off. It still is, and each half means
something different now. The public half is on because the settings sweep put it
there. The private half is off because the baseline above puts it there.

Every private board is owned by a personal account:

    cut -f1,2 scanning.tsv | grep -c 'private'
    38

    gh api users/iderex --jq '.type'      User
    gh api users/Flowfin --jq '.type'     Organization

THIS PAGE CARRIED AN OPEN QUESTION HERE AND IT IS ANSWERED. What stood here was
a claim, marked as unmeasured, that code scanning and secret scanning are free
on public repositories and are part of GitHub Advanced Security on private ones,
that Advanced Security is sold to organizations rather than to personal
accounts, and that whether it held decided who the rollout was for. The decision
on `#32` settles it from the other end and does not need the billing page: paid
is out, so the private boards get the free baseline, and whether Advanced
Security could be bought for them stops being a question this page has to
answer.

## The recorded reasons

### `iderex/jellyfin-web` - Dependabot alerts off on a fork

Reason: the board is a fork of `jellyfin/jellyfin-web` and carried 62 open
Dependabot alerts, every one inherited from upstream's dependency tree. That
surface is watched upstream.
Decided: 27 August 2026, and recorded on `#32`.
Expires when: the fork grows security-relevant divergence of its own. The sweep
should say so when it does.

This board is not in the roster the counts above are taken over, and it is the
shape the entries below follow.

### `iderex/.github` - no CodeQL-supported language

Reason: there is nothing on the board for CodeQL to analyse. It is the one
public board of the 39 with no code-scanning analysis, and GitHub's own
enumeration of what it could analyse there is empty:

    gh api repos/iderex/.github/code-scanning/default-setup
    {"state":"not-configured","languages":[], ...}

    gh api "repos/iderex/.github/code-scanning/alerts?per_page=1"
    {"message":"no analysis found", ...}

The languages list is empty, and that is the whole reason: default setup has no
language to enable there. Secret scanning with push protection is on as it is
everywhere else on the public side.
Decided: 28 August 2026. The settings sweep recorded on `#32` said its one
refusal would be a board without a CodeQL-supported language and left it to be
confirmed; the reading above is that confirmation.
Expires when: the board grows code in a CodeQL-supported language.

### The private boards - the baseline, not an exception

The 38 private boards report code scanning and secret scanning off, and that is
the private row of the baseline rather than a reason anyone owes. Nothing is
recorded here for them, because a register entry is for a board sitting outside
its own row and none of them is.

What they get instead is Dependabot and the local gate's own legs. Whether they
actually carry Dependabot is a different axis, and this page does not sweep it -
see below.

### `iderex/wache`, `iderex/Typeset`, `iderex/iderex` - the entry that expired

This page recorded these three as public boards with no CodeQL analysis and no
detected language. All three now carry analyses and `iderex/Typeset` reports
`Swift`, so what is left of the entry is the correction: what was recorded was
true on 27 August, stopped being true on 28 August, and was found by re-running
the sweep rather than by re-reading the sentence.

WHAT THE EXPIRED ENTRY WAS RIGHT ABOUT IS STILL OPEN. The baseline puts `zizmor`
over the workflow surface, and the workflow surface is most of what these three
boards are. A CodeQL analysis at `/language:actions` is not that scan, this
board runs no `zizmor` today, and no board's `zizmor` state is read anywhere in
this page. The roster record for this board already says it carries none of the
five universal guards yet, and that is the same absence read from the other
side.

## What is not covered here

- No setting is changed by anything in this repository, and no route here reads
  one either. This page is a reading and a register; the state it records went
  stale the moment it was written, and the correction above is what that looks
  like when it happens within a day.
- Dependabot is in both rows of the baseline and is swept nowhere here. Whether
  a board carries `.github/dependabot.yml` at all is read in
  `docs/dependabot-across-the-boards.md`, and it returns 64 of the 73 roster
  boards without one, so the Dependabot half of the baseline is the further from
  met of the two.
- `zizmor` is in the public row of the baseline and no board's state for it is
  read anywhere in this repository.
- Whether a board that reports `analyses present` analyses the whole tree, or
  which languages its analysis covers. The endpoint says an analysis exists.
- Nothing here runs on a schedule. Every figure above was produced by a command
  somebody chose to run, and `#31` is where the sweep that would run them lives.
