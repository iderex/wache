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

## The reading on 31 August, and the population grew outside the contract

Re-read against the roster at `iderex/operations` `origin/main`
`66f3e6a3fe24f165f16f75b196431b170405d2ab`, with the same sha-validated command
the first section uses:

    61 absent
    12 present, and 12 distinct blob ids

The section above returns 64 absent and nine present on 29 August, and this
board made the tenth. Two more boards carry the file now, `Flowfin/core` and
`Flowfin/jellyfin-plugin-watchlist`, and the contract lines were read off every
one of the twelve rather than off the two:

    sed -n 's/^#   \(origin\|taken-at\): *//p' <each copy>

    iderex/wache      iderex/wache templates/dependabot.yml
                      a637780d22d9472988fc5c330643de63b4e5e68a
    the other eleven  neither line

One copy of twelve names an origin and a commit, and it is this board's. So the
third state this page describes - not up to date, not drifted, unjudged - covers
eleven boards where it covered nine, and the two files that arrived after the
template landed arrived outside it.

What the two new ones differ by, with the canonical side supplied by hand
because neither names a commit to resolve:

    Flowfin/core                        11 differing line(s)  cargo, github-actions
    Flowfin/jellyfin-plugin-watchlist   14 differing line(s)  github-actions, nuget

THE DIRECTION IS WHAT TO READ HERE AND IT IS NOT THE COUNT. Over the window in
which this board held a canonical file and a contract for copying it, no board
outside this one took either, and the population of files that contract is meant
to govern grew by two. That is a reading of the state and not a claim about why:
what would put the template in front of a board writing its own file is a leg
that measures drift and opens an issue on the board, and nothing in this
repository does that today.

    grep -c -i 'templates/dependabot\|dependabot-across-the-boards\|taken-at' \
      .github/workflows/fleet-alert-sweep.yml
    0

The scheduled sweep this board runs reads alert counts and the scanning
baseline. It does not read this page, the template, or a copy's contract lines,
which is `#31`'s sweep having a different subject rather than a defect in it.

WHAT THIS READING DOES NOT COVER. Whether any of the eleven files without the
contract has changed content since 29 August; blob ids were compared within this
reading and not against the earlier one. And why the two new boards configured an
updater, which is their own decision and is written nowhere this page can read.

## The reading on 4 September, and the contract count is still one

Re-read against the roster at `iderex/operations` `origin/main`
`e4ce672f773cc675bfe5b88a6e7899c7bf6f66d8`, 74 boards where the section above
had 73. The board added since is `erawright/steinbruch`, and it carries no
`dependabot.yml`.

THE FILE CAME BACK ON THE SWEEP THAT READS TREES, not on a `contents` call per
board. `docs/unicode-guard-copies.md` sets that route out under its own 4
September reading; the same four queries carry
`dep: object(expression:"HEAD:.github/dependabot.yml")` beside the workflows
tree, so this page and that one are two readings of one fetch rather than two
fetches that have to be argued into agreement. What it changes here is the trap
this page opens on: the sha every section above validates as hexadecimal before
counting it is now a blob object's own `oid` field, and an error body has no
route into that column to be validated out of.

    62 absent
    12 present, and 12 distinct blob ids

The count is what the section above records. Whether it is the same twelve
BOARDS is not a comparison this page could make until now, because that reading
kept no list, so the list is here:

    Flowfin/core                           8d318eacc2fb0a6967779f5410e6e05e97a97ba3
    Flowfin/jellyfin-plugin-invites        549a1abd7b4daa980f3c4e7e622bca77f5afd5a7
    Flowfin/jellyfin-plugin-sso            c50ea5247148f0a3011bb2f87875c7e8edad13c9
    Flowfin/jellyfin-plugin-watchlist      a0f8498b8a4c6f5fc0ae1f4ecbae2047e7666d8b
    Flowfin/lab                            87affb3ca20e7d373c049faf93682dac660bf27d
    Flowfin/site                           f34392a1aea9a5685b9ced0cc52db94686020cc2
    iderex/Easy-Compliance-Manager         46a0f5e6baf5e667b0f314ca5e9ed67708c341a3
    iderex/cudec                           880e0e3d6cb4a6e0bf3016f756bb6ba0cf512ba9
    iderex/lichttisch                      92f0ad415f82f6233cc6c24532cae5ffa10c915b
    iderex/retusche                        f9a5231933406b319fd657e8b1eabe6443f39437
    iderex/swarm.asm                       ad0be0bc08b68146d3e1d4bd7385ed5effee0c5c
    iderex/wache                           7b0444a82e1268a4a0dd1fc865ef0fed6c98f845

The two boards the section above names as having arrived after the template,
`Flowfin/core` and `Flowfin/jellyfin-plugin-watchlist`, are both in that list.
So is the one id an earlier reading wrote down, in the reproduction
`docs/standardisation-survey.md` carries of the 404 trap:

    git grep -n 'iderex/retusche f9a52' -- docs/standardisation-survey.md
    docs/standardisation-survey.md:132:    iderex/retusche f9a5231933406b319fd657e8b1eabe6443f39437

which is the id above, so that copy has not moved since 27 August. Beyond those
three the set is uncompared, for the reason above.

The contract lines, read off all twelve rather than off a sample, with the
comment prefix stripped so the two values print as they would be resolved:

    jq -r '.data | to_entries[] | .value | select(.dep != null)
           | [ .nameWithOwner,
               (.dep.text | split("\n")
                | map(select(test("^#   (origin|taken-at): ")))
                | map(sub("^#   [a-z-]+: +";""))
                | join(" | ")) ] | @tsv' batch.*.json | sort |
      awk -F'\t' '{printf "%-38s %s\n", $1, ($2=="" ? "(neither line)" : $2)}'

    Flowfin/core                           (neither line)
    Flowfin/jellyfin-plugin-invites        (neither line)
    Flowfin/jellyfin-plugin-sso            (neither line)
    Flowfin/jellyfin-plugin-watchlist      (neither line)
    Flowfin/lab                            (neither line)
    Flowfin/site                           (neither line)
    iderex/Easy-Compliance-Manager         (neither line)
    iderex/cudec                           (neither line)
    iderex/lichttisch                      (neither line)
    iderex/retusche                        (neither line)
    iderex/swarm.asm                       (neither line)
    iderex/wache                           iderex/wache templates/dependabot.yml | a637780d22d9472988fc5c330643de63b4e5e68a

One copy of twelve names an origin and a commit, as on 29 and 31 August, and it
is this board's. The template has stood unchanged since it landed:

    git log --format='%h %ad %s' --date=short -- templates/dependabot.yml
    f211804 2026-08-28 Hold the canonical dependabot block in one place [#24]

so a week has passed in which a board could have taken it and none has.

WHAT THIS READING DOES NOT COVER. Whether any of the eleven files without the
contract changed content between 31 August and now: that reading kept no ids, so
the comparison starts at the table above rather than reaching back. And why no
board has taken the template is a decision taken on those boards and written
nowhere this page can read - the leg that would put it in front of them is still
`#31`'s, and the sentence the section above proves about the scheduled sweep is
unchanged.

## The reading on 5 September, and the step the contract was missing

Every reading above counts copies. This one reads what a board has to do to
BECOME one, because the second done-when of `#24` asks that each board's copy
name the place and the commit, and one copy of 74 does after eight days in which
the template stood unchanged. What was written down was the contract - the two
comment lines and the byte comparison over the `github-actions` block - and not
the sequence a board executes to satisfy it. The sequence is in the README now,
under `### If you are taking this template`, and this section is the reading
behind its third step.

The counts first, read 5 September 2026 against the roster at
`iderex/operations` `origin/main` `e1903807dc380addc3de69d3c4996bf5e7b89a77`,
over 74 boards, by the same four-call route the 4 September section sets out:

    62 absent
    12 present, and 12 distinct blob ids
     1 copy carries both contract lines, and it is this board's

Every one of those is the figure of 4 September. Nine days after the template
landed, no board outside this one has taken it.

### The step nobody would have looked for

A board that adds `.github/dependabot.yml` starts receiving pull requests whose
commit subjects name no issue and cannot. `subject_names_issue` in the shared
hygiene check defaults to `true`, so on a board that calls the check and does not
switch that rule off, every one of those pull requests is refused from the day
the file lands. This board carries the exemption for exactly that reason, and its
own `hygiene.yml` says so, five hundred lines from anything a copying board
reads.

Read off the same fetch as the counts above - each board's `.github/workflows`
tree with every blob's text inline, so the `with:` block beside each call is read
rather than sampled:

    19 boards outside this one call pr-hygiene.yml
    18 of the 19 pass subject_names_issue: false
     1 does not: iderex/hoersaal

`iderex/hoersaal` is deliberate rather than an oversight, and its own file says
so:

    # THE SUBJECT RULE IS ON HERE AND OFF ALMOST EVERYWHERE ELSE. This board's
    # `internal/prhygiene` already refuses a subject without [#N], and all twenty of
    # the last authored subjects carry one. It is the only board of the twenty-one
    # that does not have to switch the rule off to call the check.

It carries no `dependabot.yml` today, so nothing is red there now. It is the one
board on which taking this template would turn a green gate red, and the step
exists so that whoever does it there does it in one change rather than two.

### The exemption is not available at the ref most boards pin

This is the half that makes the step cost more than a line. `subject_exempt_authors`
arrived in `v1.3.0` and is in no earlier release:

    git show 9b311243c2d0d0ced7feb957a20bc178acce6a5d:.github/workflows/pr-hygiene.yml |
      sed -n '/^on:/,/^permissions:/p'
    on:
      workflow_call:
        inputs:
          subject_names_issue:
            description: 'Every non-merge commit names an issue in its subject'
            type: boolean
            default: true
            required: false

    permissions: {}

That is `v1.0.0`, which thirteen of the nineteen callers pin; six pin `v1.2.0`,
which declares the same single input. So no caller today can exempt an author
without bumping to `v1.3.0` first, and that bump is itself a behaviour change -
the line-by-line closing-keyword refusal the README's `## Versions` section
describes. For `iderex/hoersaal` the template therefore costs three things in one
change and not one: the file, the exemption, and a pin bump past a refusal it has
not met.

### What this section does not evaluate

The thirty-four boards holding a LOCAL `pr-hygiene.yml` rather than calling this
one. Their subject rules are their own files and were not read, so the count of
one board above is a count over CALLERS and not over the fleet. A local
implementation with the same rule would meet the same red gate and nothing here
says whether any does.

Why no board has taken the template in nine days. The missing step above is a
candidate and is not evidence: it explains a cost on one board of the seventy-four
and says nothing about the other seventy-three, whose reasons are decided on
those boards and written nowhere this page can read.
