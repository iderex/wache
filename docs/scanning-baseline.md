# Scanning across the boards: the state, the baseline, and the recorded reasons

Taken on 27 August 2026 against the roster in `iderex/operations`, at
`origin/main` `bc2b8bd1f4238d8148f1d4e6fe765a1e8fb614e0`, over the 73 boards the
`boards` function in `docs/standardisation-survey.md` returns. Every figure
carries the command that produced it, and the figures move whenever a setting is
changed, so re-run them rather than citing this page.

This is the home `#32` asks for: the baseline written down, the current state
measured, and the register a board writes into when it deliberately does not
scan. It rolls nothing out. The settings themselves are changed in each
repository's own configuration and no change here reaches them.

## The baseline

Recorded on `#32`. CodeQL plus secret scanning: secret scanning wherever GitHub
allows it, CodeQL on every board with a supported language, `zizmor` over the
workflow surface. A board that deliberately does not scan records its reason
here, in the shape the first entry below uses - reason, decider, date, and the
condition under which the reason expires.

## Reading the state

`security_and_analysis` on the repository object is not the reading to use. It
is `null` on every private board here, so it cannot distinguish a board with
scanning off from a board whose settings the token cannot see:

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

## What the 73 boards report

    code scanning                 secret scanning
    38 not enabled                38 disabled
    32 analyses present           35 enabled
     3 enabled, no analysis

Crossed against visibility, the table has three rows and no exceptions:

    38  private  code scanning off        secret scanning off
    32  public   analyses present         secret scanning on
     3  public   enabled, no analysis     secret scanning on

    awk -F'\t' '{print $2"\t"$4"\t"$5}' scanning.tsv | sort | uniq -c | sort -rn

## The gap is not a per-board default, it is one line

THE SPLIT IS EXACTLY VISIBILITY, 73 BOARDS OUT OF 73. Every private board is
off on both. Every public board has secret scanning on. No board sits on the
wrong side of that line, which is what a gap made of forgotten per-board
defaults would look like.

Every private board is owned by a personal account:

    awk -F'\t' '$2=="private"{split($1,a,"/"); print a[1]}' scanning.tsv |
      sort | uniq -c
    38 iderex

    gh api users/iderex --jq '.type'      User
    gh api users/Flowfin --jq '.type'     Organization

CLAIM, NOT MEASURED FROM THIS TREE: code scanning and secret scanning are
included for public repositories and are part of GitHub Advanced Security for
private ones, and GitHub Advanced Security is sold to organizations and
enterprises rather than to personal accounts. The measurement above is
consistent with it and does not establish it. What would settle it is the
billing page for the account, which is not readable from here. It matters
because it decides who the rollout is for: if it holds, the 38 private boards
are not 38 settings clicks but one purchasing decision, or a move of those
repositories under an organization, and neither is a checklist item.

The public side is where a checklist applies, and it is three boards long.

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

### `iderex/wache`, `iderex/Typeset`, `iderex/iderex` - no CodeQL-supported language

Reason: GitHub detects no language on any of the three.

    gh api "repos/$r" --jq '.language // "no language detected"'
    iderex/wache       no language detected
    iderex/Typeset     no language detected
    iderex/iderex      no language detected

These are the three public boards reporting `no analysis found` above. Secret
scanning is on for all three; what is absent is a CodeQL analysis, and CodeQL
has nothing to analyse where no supported language is present.
Decided: not yet. This entry records the measurement and the candidate reason,
not an accepted one.
Expires when: any of the three grows code in a CodeQL-supported language.

WHAT THIS ENTRY DOES NOT COVER, and it is the half that is not automatic:
the baseline puts `zizmor` over the workflow surface, and the workflow surface is
most of what these three boards are. This board runs no `zizmor` today - its
`.github/workflows` holds `guards.yml`, `hygiene.yml`, `pr-hygiene.yml` and
`unicode-guard.yml` and nothing else - so "no supported language" is a reason
for the absent CodeQL and NOT a reason for the absent workflow scan. The roster
record for this board already says it carries none of the five universal guards
yet, and that is the same absence read from the other side.

### The 38 private boards

No reason is recorded, and none should be written until the claim above is
settled. A reason saying "GitHub Advanced Security is not available here" is
either the correct entry for all 38 at once or it is wrong for all 38, and which
one it is depends on a fact this page could not read.

## What is not covered here

- No setting is changed by anything in this repository, and no route here reads
  one either. This page is a reading and a register; the state it records went
  stale the moment it was written.
- Dependabot alerts and Dependabot security updates are a third axis and are not
  swept above. The one recorded reason so far is about exactly that axis, which
  is a reason to sweep it and not evidence that it is fine.
- Whether a board that reports `analyses present` analyses the whole tree, or
  which languages its analysis covers. The endpoint says an analysis exists.
