# What the boards actually configure for Dependabot

Taken on 27 August 2026 against the roster in `iderex/operations`, at
`origin/main` `bc2b8bd1f4238d8148f1d4e6fe765a1e8fb614e0`. The board list is
derived by the `boards` function in `docs/standardisation-survey.md` and returns
73 today. Every figure below carries the command that produced it, and the
figures move whenever a board lands anything, so re-run them rather than citing
this page.

This is the reading `#24`'s first done-when asks for. It does not standardise
anything, and it says at the end why it cannot yet.

## Nine boards carry the file and 64 do not

```
out=$(gh api "repos/$r/contents/.github/dependabot.yml" --jq '.sha' 2>/dev/null)
case "$out" in [0-9a-f][0-9a-f]*) printf '%s\t%s\n' "$r" "$out" ;;
                              *) printf '%s\tabsent\n' "$r" ;; esac
```

    64 absent
     9 present, and 9 distinct blob ids

The nine, with the ecosystems each one declares:

    Flowfin/jellyfin-plugin-invites   github-actions nuget
    Flowfin/jellyfin-plugin-sso       github-actions nuget
    Flowfin/lab                       github-actions gomod
    Flowfin/site                      github-actions gomod
    iderex/Easy-Compliance-Manager    cargo github-actions
    iderex/cudec                      github-actions
    iderex/lichttisch                 github-actions
    iderex/retusche                   github-actions uv
    iderex/swarm.asm                  github-actions nuget

    grep -oE 'package-ecosystem: *"?[a-z-]+' <file> | sed 's/.*: *"\?//' | sort -u

WHY THIS IS NOT THE DISTRIBUTION `#24` WAS FILED ON is at the corrected
`### dependabot.yml` section of `docs/standardisation-survey.md`: the 63-board
identical cluster was `gh api`'s 404 body, captured as a sha by the command that
counted it, and it is the same constant string for every board without the file.

## What the nine agree on, and it is one line

    grep -oE 'default-days: *[0-9]+' <file>
    7   on all nine

Seven days of cooldown before a brand-new release may be proposed, on every
board that configures anything at all. Two of the nine write down the same
reason for it - the window in which a compromised publish is most dangerous is
the hours after it appears - and the rest carry the number without the argument.

Nothing else is unanimous:

    board                            interval  pr-limit  groups  labels
    Flowfin/jellyfin-plugin-invites  monthly   3         yes     yes
    Flowfin/jellyfin-plugin-sso      weekly    10        yes     no
    Flowfin/lab                      weekly    5         no      yes
    Flowfin/site                     weekly    -         yes     yes
    iderex/Easy-Compliance-Manager   weekly    10        yes     no
    iderex/cudec                     weekly    5         yes     no
    iderex/lichttisch                weekly    5         yes     yes
    iderex/retusche                  weekly    3         yes     no
    iderex/swarm.asm                 weekly    5         yes     no

Weekly on eight of nine, a pull-request ceiling of 3, 5 or 10, and grouping on
eight of nine. Those are variations on one answer rather than nine answers.

## The part that could be canonical, and the part that cannot

`dependabot.yml` is language-shaped. Every one of the nine declares
`github-actions` and then whatever the board is written in - `nuget`, `gomod`,
`cargo`, `uv` - so identical files across boards would be the surprising result
rather than the expected one. That is the same reason
`docs/standardisation-survey.md` gives for leaving `codeql.yml` out of its
tranche.

What is NOT language-shaped is the `github-actions` block. Every board here pins
its actions by commit sha, a pinned sha never moves on its own, and that block
is the only thing keeping those pins current. It is the same on all nine boards
up to wording, and it is the part a canonical file could carry.

## What this leaves for the shape decision

The decision recorded on `#24` is a template with a pinned origin: this board
holds the canonical file, each board copies it carrying origin and version in a
comment, and drift is reported rather than pushed. That decision was taken while
the count said 63 boards had already converged on one file.

The count says something else. There is no majority to record, because 64 boards
have no dependency updater configured at all. A template still fits what is
here - it is the only shape that survives a file this language-shaped - but what
it costs and what it buys have both changed: introducing a file to 64 boards
that do not have one is a rollout, not the writing-down of an agreement, and the
nine that do have one are not outliers to be corrected but the only boards with
any answer at all.

WHAT THIS PAGE DOES NOT EVALUATE. Whether the 64 boards are deliberately without
an updater. I read the distribution and the nine files, and nothing else. On
`iderex/lichttisch` the file's own comment argues that no updater is the larger
risk where every action is pinned by sha, which is an argument that reaches the
other 64 boards, but it is that board's sentence and not a reading of theirs.

## The canonical file, and how a copy declares its origin

Decided on `#24`: a template with a pinned origin, not a sync that pushes.
GitHub reads `.github/dependabot.yml` out of each repository, so every copy is
sovereign whatever this board does; a pusher would be a second writer racing the
board's own workers. This board holds the content and measures what has drifted
away from it, and the board's own workers move their copy.

The named place is [`templates/dependabot.yml`](../templates/dependabot.yml).
What it carries is the `github-actions` block and nothing else, for the reason
the section above gives: the rest of the file is language-shaped and a board's
own ecosystems belong below the block rather than in it.

A copy replaces two lines in the template's leading comment:

    #   origin: iderex/wache templates/dependabot.yml
    #   taken-at: <the 40-character iderex/wache commit this copy was taken from>

Those two lines are the whole of the contract. A copy carrying neither is not
read as up to date and not read as drifted - it is not read at all, and that is
the state the drift test reports separately from a difference.

## The drift test

Two files and one comparison. `block` prints the `github-actions` entry of a
`dependabot.yml`, comment lines and blank lines dropped, and stops at the next
ecosystem:

```
block() {
  awk '
    /^  - package-ecosystem:/ { keep = ($0 ~ /github-actions/) }
    keep && $0 !~ /^[[:space:]]*#/ && NF { print }
  '
}

gh api "repos/$r/contents/.github/dependabot.yml" --jq '.content' | base64 -d > copy.yml
sed -n 's/^#   taken-at: *//p' copy.yml          # the commit the copy names
git show "$taken_at:templates/dependabot.yml" | block | diff - <(block < copy.yml)
```

Run against the nine boards that carry the file, with the canonical side taken
from `templates/dependabot.yml` by hand, because not one of the nine names an
origin and there is nothing to resolve `$taken_at` to on any of them:

    Flowfin/jellyfin-plugin-invites            19 differing line(s)
    Flowfin/jellyfin-plugin-sso                14
    Flowfin/lab                                11
    Flowfin/site                                5
    iderex/cudec                               12
    iderex/Easy-Compliance-Manager             10
    iderex/lichttisch                          10
    iderex/retusche                             8
    iderex/swarm.asm                           10

Nine of nine differ, which is what the section above predicts and not a
surprise: none of these boards has ever been shown this file. The presence
reading was re-run at the same time and still returns 64 absent, nine present,
nine distinct blob ids.

THAT PARAGRAPH ONCE SAID NO COPY ANYWHERE NAMED AN ORIGIN, AND IT SAID IT OF THE
WHOLE ROSTER RATHER THAN OF THE NINE. One does now, and it is the section below.
The nine are still nine, so the sentence above is narrowed rather than deleted:
the hand-supplied canonical side is what a board that names no origin costs, and
it is not what the test does when a board names one.

WHAT THIS TEST CANNOT DO, and it is a floor rather than a measurement of
meaning. It compares BYTES. A copy that differs only in quoting, in key order or
in a `day:` and `time:` beside the interval is reported as drifted, and the
figures above include exactly that. Four of the nine write the schedule out in
more detail than the canonical block does, and every one of those lines counts:

    block < copy.yml | grep -cE '^      (day|time|timezone):'
    iderex/cudec 3, iderex/Easy-Compliance-Manager 3, iderex/swarm.asm 3,
    iderex/retusche 1, and 0 on the other five

A comparison that judged meaning would need a YAML parser, which is a means this
board does not carry today and is not added here for a reading.

## The first copy, and the first run in which `taken-at` resolved

This board carries `.github/dependabot.yml` from this change. It is the first
copy of the template anywhere, and until it existed the drift test had never
been run the way it is written: every run above supplied the canonical side by
hand, because no board named a commit to resolve.

The two contract lines it carries:

    sed -n 's/^#   \(origin\|taken-at\): *//p' .github/dependabot.yml
    iderex/wache templates/dependabot.yml
    a637780d22d9472988fc5c330643de63b4e5e68a

`a637780` is the mainline this copy was taken from, and it carries
`templates/dependabot.yml`, so the test resolves it rather than being handed a
file:

    taken_at=$(sed -n 's/^#   taken-at: *//p' .github/dependabot.yml)
    git show "$taken_at:templates/dependabot.yml" | block | diff - <(block < .github/dependabot.yml)
    diff exit=0

No differing line. That is the whole verdict for a copy that is up to date, and
it is the reading this page could not produce before.

THE TEST IS SHOWN BITING RATHER THAN PASSING, because a comparison that has only
ever returned agreement proves nothing about what it would refuse. Three
near-misses against the same copy, each one a change somebody would actually
make:

    # one value moved
    sed 's/interval: weekly/interval: daily/' .github/dependabot.yml > drifted.yml
    git show "$taken_at:templates/dependabot.yml" | block | diff - <(block < drifted.yml)
    4c4
    <       interval: weekly
    ---
    >       interval: daily
    diff exit=1

    # the taken-at line removed
    grep -v '^#   taken-at:' .github/dependabot.yml > unmarked.yml
    sed -n 's/^#   taken-at: *//p' unmarked.yml
    (empty - there is nothing to resolve, so this copy is not judged at all)

    # the leading comment rewritten and the block untouched
    sed 's/^# THIS BOARD.S COPY.*/# A DIFFERENT HEADING ENTIRELY./' .github/dependabot.yml > recommented.yml
    git show "$taken_at:templates/dependabot.yml" | block | diff - <(block < recommented.yml)
    diff exit=0

So a moved value is drift, a rewritten comment is not, and a copy that names no
commit is the third state this page has been claiming since the contract was
written: not up to date, not drifted, unjudged.

WHAT THIS COPY IS NOT IS THE ROLLOUT. The presence reading was re-run today, 29
August 2026, over the 73 boards the roster holds at `iderex/operations`
`origin/main` `f544394ab14a48f307602e768c15f5084d0e0999`, with the same command
the first section uses:

    64 absent
     9 present, and 9 distinct blob ids

`iderex/wache` was one of the 64 until this change and is the tenth board with
the file after it. The other 63 are other boards' trees, this board writes into
none of them, and `#24`'s second done-when asks for a copy on each - so one
board taking the copy moves that line by one board and does not meet it.

WHAT OPENS AN ISSUE ON A DRIFTED BOARD IS THE SWEEP AND NOT THIS PAGE. `#31` is
where that sweep lives, this board writes into no other tree, and nothing in
this repository runs the comparison above on a schedule today.
