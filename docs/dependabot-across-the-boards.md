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

## The reading on 6 September, and the first backward comparison of the copies

Every section above counts the copies. This one compares them to the list before
it, which is a thing no earlier section could do: the reading of 31 August kept
no ids, so 4 September had nothing to reach back to and said so. That section
kept twelve, and this is the run in which the question it left open is answered
rather than restated.

Against the roster at `iderex/operations` `origin/main`
`77d2c1f5aada75b1a40796fb3d6afeca630388cc`, 74 boards, derived with the
`boards()` function `docs/standardisation-survey.md` declares rather than with a
list kept anywhere.

THE QUERY IS NARROWER THAN THE ONE THE SECTION ABOVE SHARES, and the difference
is worth naming because that section's argument is that this page and
`docs/unicode-guard-copies.md` are two readings of ONE fetch. This reading needs
no workflow bytes, so it asks each board only for its `.github/dependabot.yml`
as a typed blob with its `oid` and `text`, nineteen boards to a query, four
queries:

    fragment B on Repository {
      nameWithOwner defaultBranchRef{ name target{ oid } }
      dep: object(expression:"HEAD:.github/dependabot.yml"){ ... on Blob{ oid text } } }

So it is one fetch with the other page's half dropped, not a second route. The
trap the sections above validate against is closed the same way it is there - an
`oid` is a field of a typed blob and an error body has no way into that column -
and every reply was checked for an `errors` key before anything was counted:

    jq -r 'if .errors then (.errors|length|tostring)+" errors" else "no errors key" end' batch.*.json
    no errors key
    no errors key
    no errors key
    no errors key

    74 boards, 74 with a default branch
    61 absent
    13 present, 13 distinct blob ids, and 0 whose text came back null

    erawright/steinbruch                   4c1f97011aba8487d4989b27ede9ea3907361985
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

### The twelve of 4 September have not moved

Joined mechanically against the table the section above landed, so the answer
comes from the ids rather than from reading two lists side by side. The
left-hand file is that table extracted out of this page at `origin/main`,
scoped to the section that carries it:

    git show origin/main:docs/dependabot-across-the-boards.md |
      sed -n '/^## The reading on 4 September/,/^## /p' |
      grep -E '^    [A-Za-z0-9._/-]+ +[0-9a-f]{40}$' | awk '{print $1"\t"$2}' | sort > sep04.tsv
    jq -r '.data | to_entries[] | .value | select(.dep != null)
           | [.nameWithOwner, .dep.oid] | @tsv' batch.*.json | sort > sep06.tsv
    join -t$'\t' sep04.tsv sep06.tsv -o 0,1.2,2.2 |
      awk -F'\t' '{print ($2==$3 ? "SAME  " : "MOVED ") $1}'

    SAME on all twelve
    comm -23: nothing only in the 4 September list
    comm -13: erawright/steinbruch, only in this one

THE `sed` LINE WAS NOT THERE WHEN THIS SECTION LANDED, AND WITHOUT IT THE COMMAND
STOPPED REPRODUCING AT THE MOMENT ITS OWN OUTPUT WAS PASTED BELOW IT. The pattern
matches an indented `board  <40 hex>` line anywhere in the file, and the table
this section landed is such a table, so the page has carried two of them since
this change merged. Counted at `origin/main`
`dad1729adfc2f0335f1a7ffe7b70da511c925664`:

    git show origin/main:docs/dependabot-across-the-boards.md |
      grep -cE '^    [A-Za-z0-9._/-]+ +[0-9a-f]{40}$'
    25

Twelve boards appear in both tables, `join` emits one row per matching pair, and
the unscoped run therefore returns 25 rows where the block above records twelve.
With the `sed` line it returns the block above again, which is what makes this a
repair of the reading rather than a second reading:

    join -t$'\t' sep04.tsv sep06.tsv -o 0,1.2,2.2 | wc -l
    12

WHAT MAKES IT WORTH REPAIRING RATHER THAN NOTING IS THAT THE VERDICTS DO NOT
MOVE. Both copies of each duplicated board carry the same id today, so all 25
rows of the unscoped run read `SAME` and the fault is invisible in the output - a
green answer over a denominator nobody counted. It becomes visible only once a
board has actually moved, and that is a one-line near miss rather than a
supposition. `unscoped.tsv` is the left-hand file the command produced before the
`sed` line, with one of the two `iderex/cudec` ids replaced by zeroes:

    awk -F'\t' 'BEGIN{OFS="\t"} {if($1=="iderex/cudec" && !seen++){$2="000...0"} print}' \
      unscoped.tsv | sort | join -t$'\t' - sep06.tsv -o 0,1.2,2.2 |
      awk -F'\t' '{print ($2==$3 ? "SAME  " : "MOVED ") $1}' | grep cudec
    MOVED iderex/cudec
    SAME  iderex/cudec

One board, two rows, opposite verdicts, and nothing in the output saying which of
them is the answer.

So no copy changed content between 4 and 6 September, this board's included, and
the eleven that carry no contract line are byte-for-byte what they were. That is
a reading of a two-day window and not a statement about how stable the
population is; what it retires is the sentence the section above closes with,
which is that the comparison could not be made at all.

### The thirteenth board, and it took the file rather than the template

`erawright/steinbruch` is the board the section above names as having joined the
roster since 31 August and as carrying no `dependabot.yml`. It carries one now,
so the file arrived there inside the two days between these two readings. The
contract lines were read off all thirteen rather than off the new one:

    jq -r '.data | to_entries[] | .value | select(.dep != null)
           | [ .nameWithOwner,
               (.dep.text | split("\n")
                | map(select(test("^#   (origin|taken-at): ")))
                | map(sub("^#   [a-z-]+: +";""))
                | join(" | ")) ] | @tsv' batch.*.json

    erawright/steinbruch   (neither line)
    the other eleven       (neither line)
    iderex/wache           iderex/wache templates/dependabot.yml | a637780d22d9472988fc5c330643de63b4e5e68a

One copy of thirteen names an origin and a commit, and it is this board's. The
third state this page describes - not up to date, not drifted, unjudged - now
covers twelve boards where it covered eleven.

### The drift test against it, canonical side supplied by hand

There is no `taken-at` to resolve, so the canonical side is
`templates/dependabot.yml` at `origin/main`, as in every earlier hand run:

    git show origin/main:templates/dependabot.yml | block | diff - <(block < steinbruch.yml)
    2c2
    <     directory: "/"
    ---
    >     directory: /
    5c5,7
    <     open-pull-requests-limit: 5
    ---
    >       day: monday
    >       time: "04:00"
    >       timezone: Europe/Berlin
    10,11c12
    <         patterns:
    <           - "*"
    ---
    >         patterns: [ "*" ]

    9 differing line(s)  cargo, github-actions

Eleven boards have been measured this way before - nine in the drift-test
section and two on 31 August - with figures from 5 to 19, and nine sits inside
that range rather than at either end.

WHAT THE NINE DECOMPOSE INTO IS THE TEST'S OWN FLOOR AND NOT A NEAR-TAKE. Three
of them are the extra schedule detail the drift-test section already names as
counted:

    block < steinbruch.yml | grep -cE '^      (day|time|timezone):'
    3

Five more are quoting and sequence style: `directory: /` against `directory:
"/"`, and a flow sequence `patterns: [ "*" ]` against the canonical block
sequence, which is two lines on one side and one on the other. That leaves one
line in which the two blocks ask for different things - the copy declares no
`open-pull-requests-limit`, which the canonical block sets to 5.

THAT DECOMPOSITION IS A JUDGEMENT AND THE NINE IS THE MEASUREMENT. Reading those
five lines as saying the same thing is a reading of YAML, and the drift-test
section says plainly that a comparison judging meaning would need a parser this
board does not carry. None is added here for a reading. The test reports nine, a
board acting on it would have to close all nine, and the paragraph above is what
a person sees in the diff rather than what any route here decides.

### What this section does not evaluate

Whether `erawright/steinbruch` meets the red gate step 3 of the README's
sequence warns about. It calls none of this board's checks:

    gh api graphql -f query='query { repository(owner:"erawright", name:"steinbruch"){
      object(expression:"HEAD:.github/workflows"){ ... on Tree { entries { name
      object { ... on Blob { text } } } } } } }'

    ci.yml  codeql.yml  coverage.yml  zizmor.yml
    no line in any of the four naming iderex/wache

so the step costs it nothing today. Whether those four files carry a
pull-request subject rule of their own is not read here, and that is the same
gap the section above leaves open over the boards with a local implementation.

Why the twelve have not moved, and why the thirteenth wrote its own file two
days after the roster gained it. Both are decided on those boards and written
nowhere this page can read, unchanged from the section above.

## The second reading on 6 September, and the red gate is live on one board

The section above is the first reading of the day and this is the second. What
it reads is not the copies. It is the gap the 5 September section declares and
the README repeats: the boards that run their OWN pull-request hygiene rather
than calling this one were never read, so the count of one board at risk from
step 3 was a count over CALLERS. That gap is closed here, and the answer is
larger and differently shaped than the sentence it replaces.

Against the roster at `iderex/operations` `origin/main`
`23205d6cd8d3f59d9d26f3b7bcf67e86e8547c96`, 74 boards, derived with the
`boards()` function `docs/standardisation-survey.md` declares. The fetch is the
four-call route the sections above use, widened back to carry the workflows tree
with every blob's text inline beside the `dependabot.yml`, because the rules
being read here live in those blobs:

    fragment B on Repository {
      nameWithOwner defaultBranchRef{ name target{ oid } }
      wf: object(expression:"HEAD:.github/workflows"){ ... on Tree { entries { name object { ... on Blob { oid text } } } } }
      dep: object(expression:"HEAD:.github/dependabot.yml"){ ... on Blob { oid text } } }

    jq -r 'if .errors then (.errors|length|tostring)+" errors" else "no errors key" end' batch.*.json
    no errors key
    no errors key
    no errors key
    no errors key

    74 boards, 74 with a default branch, 73 with a .github/workflows tree
    61 absent
    13 present, and 13 distinct blob ids

Joined against the table the section above landed, by the same route it used:

    SAME on all thirteen
    comm -23: nothing only in the earlier list
    comm -13: nothing only in this one

and the contract lines read off all thirteen still return one copy naming an
origin and a commit, which is this board's. Both are unmoved from this morning
and are here so the reading below rests on a state that was read rather than
assumed.

### Dependabot cannot satisfy an issue-reference rule, and its body can

Everything below turns on what text a Dependabot pull request actually carries,
so that is measured first rather than supposed. Over every Dependabot pull
request on the `iderex` boards:

    gh api -X GET search/issues -f q='is:pr author:app/dependabot org:iderex' \
      -f per_page=100 --jq '[.items[].title] | length'
    77
    gh api -X GET search/issues -f q='is:pr author:app/dependabot org:iderex' \
      -f per_page=100 --jq '[.items[].title | select(test("(^|[^A-Za-z0-9_])#[0-9]+"))] | length'
    0

None of the 77 titles carries an issue reference, and the commit subjects are
the same shape, read off four of them rather than inferred from the titles:

    gh api "repos/$r/pulls/$n/commits" --jq '.[] | .commit.message | split("\n")[0]'
    iderex/swarm.asm #322                Bump xunit.v3 from 3.2.2 to 4.0.0
    iderex/retusche #163                 Bump the actions group with 3 updates
    iderex/Easy-Compliance-Manager #1139 Bump the cargo-minor-and-patch group with 2 updates
    iderex/lichttisch #203               actions: bump the actions group across 1 directory with 5 updates

THE BODY IS THE OPPOSITE CASE AND IT IS THE ONE NOBODY WOULD PREDICT. The same
77, asked whether the body carries something a hash-and-digits test matches:

    gh api -X GET search/issues -f q='is:pr author:app/dependabot org:iderex' \
      -f per_page=100 --jq '[.items[] | (.body // "") | test("(^|[^A-Za-z0-9_])#[0-9]+")]
            | [(map(select(.))|length), length] | @tsv'
    65	77

Sixty-five of seventy-seven. Those numbers are the upstream project's issue
numbers, quoted into the body out of a release note, and nothing in the text
separates them from a reference to an issue on the board being judged. So a rule
keyed to the subject refuses a Dependabot pull request every time, and a rule
keyed to the body passes it most of the time, for a reason that is not the rule's
reason.

### The population, read with two keys because neither one is complete

By filename, over the same fetch:

    34 files named .github/workflows/pr-hygiene.yml
     1 of them is this board's shared definition
    33 local implementations on other boards

Thirty-four is what the README's closing paragraph says, and the difference is
whether this board's own file is counted rather than a movement in the
population.

THAT KEY HAS A BOUND THIS TREE ALREADY RECORDED. `docs/local-hygiene-answers.md`
says of `iderex/pruefstand` that a 404 on `.github/workflows/pr-hygiene.yml`
concluded it held no local copy, and that its gate is at
`.github/workflows/hygiene.yml` - so a filename is not what a gate is. Read by
CONTENT instead, meaning any workflow blob on any board whose text carries an
issue-reference rule, the population is 30 boards over 36 files, and five of them
are boards the filename key does not reach:

    Flowfin/jellyfin-plugin-invites    Flowfin/jellyfin-plugin-requests
    Flowfin/jellyfin-plugin-watchlist  Flowfin/jellyfin-plugin-watch-sync
    iderex/reissbrett

The content key misses boards the filename key finds, for the opposite reason:
their workflow gathers inputs and the deciding is in a script or a compiled
program elsewhere in that tree. NEITHER COUNT IS THE FLEET TOTAL and this page
claims no total. What follows is read over the 33 the filename key returns,
because those are the ones whose judging text this reading fetched.

### What the 33 do about a bot

Nine of the 33 decide in a file outside `.github/workflows`, and those files were
fetched by path and read beside the workflow that names them, so the corpus per
board is the workflow plus its script where one exists.

    13 exempt a bot author from the issue-reference rule
    12 refuse a pull request naming no issue, keyed to the BODY or to the whole
       commit MESSAGE, with no bot exemption anywhere in the corpus
     2 refuse keyed to the commit SUBJECT, with no bot exemption
     6 decide in source this reading did not fetch - Go, C#, or a tool in the
       board's own tree - and are reported unread rather than clean

THE SIX WERE FETCHED AND READ THE SAME DAY, AND THE FOUR NUMBERS ABOVE MOVE.
They are kept as they stood because the section below is the correction and
because two of the six landed in the column nobody would have guessed. Read the
tally at `### The six unread gates, read` before quoting any figure from this
block.

The two keyed to the subject are `iderex/messlatte`, whose leg walks
`git log --no-merges --format=%s` and greps each subject for a hash and digits,
and `iderex/spurenarchiv`, whose comment says it plainly:

    # The subject is the first line and nothing else. A reference in the
    # body is not what this rule is about.

`iderex/spurenarchiv` waives the rule for an outside contribution, which it
defines as a fork whose author has no write access. Dependabot pushes a branch
inside the repository, so that waiver does not reach it.

Two of the six unread declare a subject rule in prose without this reading having
executed it: `iderex/kanzlei`'s workflow says the deciding is in
`internal/prhygiene` and that the fail tier is that the body names an issue and
every commit subject names one, and `iderex/hoersaal` is the board the 5
September section already quotes for the same shape. Reading either one means
reading that board's Go, which this page did not do.

### The red gate is live today, and it came through the body

Six open Dependabot pull requests were examined over the five boards that carry
both a `dependabot.yml` and a pull-request hygiene gate. One is red:

    sha=$(gh api repos/$r/pulls/$n --jq '.head.sha')
    gh api "repos/$r/commits/$sha/check-runs" --jq '.check_runs[] | [.name, .conclusion] | @tsv'

    Flowfin/core #256                       Deterministic PR-hygiene checks = failure
    Flowfin/site #179                       Deterministic PR-hygiene checks = success
    Flowfin/jellyfin-plugin-invites #405    Deterministic pull-request hygiene = success
    Flowfin/jellyfin-plugin-watchlist #275  Deterministic PR-hygiene checks = success
    Flowfin/lab #238, #237                  no hygiene check on the head at all

THE BOARD THAT IS RED EXEMPTS DEPENDABOT BY NAME, and that is the finding rather
than an irony. `Flowfin/core` holds an explicit login list, and its own comment
says what the exemption covers:

    # It exempts that author from `names-an-issue` and from nothing else.

The exemption is never reached. Its guard requires the reference set to be empty,
and that set is what the body yields, so a body quoting twenty-five upstream
numbers is not empty and the rule reports agreement instead:

    -- names-an-issue
    ok    names: 994 995 996 1004 1005 1007 1013 1014 1016 1017 3956 3995 4007
          4019 4023 4037 4051 4061 4070 4080 4081 4085 4098 4102 4106

Those numbers then flow into the next rule, which reads each one as an issue on
the board being judged:

    -- changed-paths-inside-scope
    gh: Not Found (HTTP 404)
    ##[error]Could not read issue #994. The scope comparison cannot be made, and
    this run will not pass in place of it.

That refusal is `Flowfin/core`'s gate working as written: it will not pass
vacuously when it cannot read what it needs. Its own code already anticipates the
case one verdict over, printing that a body quoting another repository's release
notes carries bare references to issues over there, and this run took the
unreadable branch rather than the absent one. What produced the red is the
combination and not a defect in either half: an identity exemption placed on one
rule, and a second rule downstream consuming what the first rule harvested.

THE RUN BEHIND THAT RED WAS WALKED ON 7 SEPTEMBER AND IT IS NOT THE STATE OF
THAT BOARD. The paragraph is kept as it was read, because that is what the
check-runs on that head said at that hour. What it could not see is that the red
is a re-run executing a script `Flowfin/core` had already replaced, and that the
sentence about which branch the run took describes a branch the version that ran
does not have. Read `## The second reading on 7 September` below before quoting
this board as a live example of anything.

### The six unread gates, read

The block above reports six boards as deciding in source it did not fetch. The
source was fetched: one GraphQL call for four directory trees and two files, at
`checks/pr-hygiene`, `scripts/check-pr-hygiene.sh`, `test/prhygiene`,
`internal/prhygiene` twice, and `internal/hygiene`. None of the six is unread
now, and the tally over the 33 becomes

    15 exempt a bot author
    13 refuse keyed to the BODY or the whole commit MESSAGE, no exemption
     4 refuse keyed to the commit SUBJECT, no exemption
     1 refuses keyed to the CLOSING REFERENCES GitHub resolved, no exemption
     0 unread

TWO OF THE SIX EXEMPT A BOT AND ONE OF THOSE TWO IS `iderex/hoersaal`, which is
the board the 5 September section names as the one where taking this template
turns a green gate red. Its own gate skips a bot before it reads anything:

    if in.AuthorIsBot {
      v.Skipped = true
      v.SkipReason = "the author is a bot, and a bot cannot know the convention these rules hold a person to"
      return v
    }

THAT DOES NOT RETIRE THE 5 SEPTEMBER FINDING AND IT SHARPENS IT. What that
section is about is the SHARED check `iderex/hoersaal` calls with
`subject_names_issue` on and no exempt author, at a pin below the release that
carries `subject_exempt_authors`. That call is unchanged by anything here. What
moves is only the reason a reader would give: the board's own Go gate is not
where the red would come from, so a repair aimed at `internal/prhygiene` would
change nothing.

`Flowfin/site` is the other, and it is the one board carrying both a
`dependabot.yml` and a local gate, so its live pull request is the check on this
whole classification. Its rule is keyed to the subject and would refuse - and its
bot list is read from the author's address rather than a login shape:

    var botAuthors = []string{
      "dependabot[bot]@users.noreply.github.com",
      "github-actions[bot]@users.noreply.github.com",

which is why `Flowfin/site` #179 is green in the table above. The reading and the
run agree, which is the only place in this page where they could be compared.

THE OTHER FOUR CARRY NO BOT HANDLING OF ANY KIND, checked as a word rather than
as a substring, because `both` matches a careless grep for `bot` and three of
these files are full of it:

    grep -n -i -E 'bot\]|isbot|is_bot|"Bot"|dependabot|github-actions' <the four>
    exit=1

- `iderex/kanzlei` refuses on the BODY and on every commit SUBJECT, in
  `Judge()`, with no author read at any point.
- `Flowfin/jellyfin-plugin-whisper-subtitles` refuses on the commit subjects, and
  its deciding tier is skipped for a FORK head and for nothing else. Dependabot
  pushes a branch inside the repository.
- `iderex/lehrkanzel` gathers `#<number>` out of the commit messages and the body
  together, which is the harvesting shape rather than the subject one.
- `iderex/relais` is the fourth column and the reason it exists. Its rule is
  `body-names-a-closing-issue`, and it reads what GitHub RESOLVED as closed by
  the pull request rather than matching text:

      "This is read from the resolved references rather than by matching text in
      the body, which is the difference the pull request template already
      explains: a closing keyword inside a code block reads as a link to a
      person and does nothing on merge"

  A Dependabot pull request resolves no issue on that board, so this refuses
  every time - and it is the one shape in this page that an upstream release note
  cannot accidentally satisfy. The rule was written against a different failure
  and it is stricter here for that reason rather than by design.

SO THE SUBJECT COLUMN DOUBLED AND A FOURTH COLUMN APPEARED. Four boards refuse a
Dependabot pull request every time on the subject, one refuses every time on the
resolved links, and thirteen decide on text an upstream project wrote. What did
not change is the direction: no board outside this one carries the contract, and
none of these five would be found by reading a `with:` block.

### The shared check does not carry that shape, and the reason is one word

This board turns the same two things on that `Flowfin/core` runs -
`body_names_issue: true` and `resolve_referenced_numbers: true` - so the
comparison is worth making rather than assuming, and it comes out the other way.
The body rule takes its exemption before it reads anything:

    if is_exempt_login "${PR_AUTHOR:-}" "${BODY_EXEMPT_AUTHORS:-}"; then
      echo "skip  the body rule (automation: ${PR_AUTHOR:-none})"

and the job that resolves numbers reads the COMMIT SUBJECT rather than the body:

    reference_targets() {
      local subject=$1 repo=$2 n
      printf '%s' "$subject" | grep -oE '\[#[0-9]+\]' | tr -d '[]#' |
      ...

A Dependabot subject carries neither `[#N]` nor `owner/repo#N`, as the four
subjects above show, so the target set is empty and there is nothing to resolve
against a tracker. The harvest that reddens `Flowfin/core` needs a resolver fed
from the BODY, and this one is not.

THAT IS A READING OF THIS FILE AND NOT A GUARANTEE ABOUT THE FLEET. A caller
turning `resolve_referenced_numbers` on gets the subject-fed version of it
whatever its board's own gate does beside it, and the board's own gate is the
thing this section is about.

### What this does to step 3 of the README sequence

The step says to check whether your board calls `pr-hygiene.yml` without passing
`subject_names_issue: false`. That is one route of at least three, and it is not
the one that is red today:

- a rule over the commit SUBJECT refuses every Dependabot pull request, whether
  it is the shared check's `subject_names_issue` or a board's own leg;
- a rule over the BODY passes 65 times in 77 on numbers that belong to another
  project, which is worse than a refusal because it is intermittent and its green
  means nothing;
- a rule that READS the issues a body names meets a 404 for a number that was
  never an issue on that board, and a gate that refuses rather than passing
  vacuously goes red for a reason nobody wrote a rule about.

So the check to run before taking this template is over your own gate's issue
rules and not only over the `with:` block of a shared call. An exemption keyed to
the author has to cover every rule downstream of the one that collects the
references, or it covers nothing.

### What this section does not evaluate

Whether the 13 body-keyed boards would actually go red. Each one's verdict
depends on the upstream release note in the pull request in front of it, which is
text no reading here can hold, and the 65 in 77 above is a rate over a different
population than any one board's next update. THIS SAID TWELVE UNTIL NOW, AND
THE CORRECTION THAT MOVED IT LANDED IN THE SAME MERGE. `### The six unread
gates, read` above replaced that column with 13, and this sentence went on
carrying the figure out of the block that section tells a reader to stop
quoting.

The boards the filename key does not reach at all. The six whose rules are in
their own source were the other half of this sentence and are read above; the
five the content key finds and the filename key misses are named rather than
judged, and nothing here claims a fleet total.

Whether the red at `Flowfin/core` is new. The check-run on that head is the state
today; no earlier Dependabot pull request on that board was walked, so how long
this has stood is not read here.

## The reading on 7 September, and the template's own values are derived again

Every section above counts the copies, compares them, or reads what a board would
have to do to become one. None of them asks whether the CONTENT this board holds
is still the answer the boards hold. That is the half of `#24`'s second done-when
about the named place rather than about who has copied out of it, and it has an
expiry the copy count does not: `templates/dependabot.yml` states in its own
header that each of its four values is what the nine boards carrying an updater
already hold. Nine was the population on 28 August. Four more boards have gained
the file since, and no section above runs the derivation again over them - each
one counts the copies or compares them, and the header has gone on stating a
population that stopped being the population on 29 August.

Against the roster at `iderex/operations` `origin/main`
`8fb4684c49d1418c51679c52149c240af81e7dcd`, 74 boards, derived with the
`boards()` function `docs/standardisation-survey.md` declares rather than from a
list kept anywhere. The same route as the two sections above - each board's
`.github/dependabot.yml` asked for as a typed blob with its `oid` and `text`,
nineteen boards to a query, four queries - and every reply checked for an
`errors` key before anything was counted:

    jq -r 'if .errors then (.errors|length|tostring)+" errors" else "no errors key" end' batch.*.json
    no errors key
    no errors key
    no errors key
    no errors key

    74 boards, 74 with a default branch
    60 absent
    14 present, 14 distinct blob ids, and 0 whose text came back null

    Flowfin/core                           8d318eacc2fb0a6967779f5410e6e05e97a97ba3
    Flowfin/hub                            30b8f8e2d1727b46c47db4a140965f9d9728d54d
    Flowfin/jellyfin-plugin-invites        549a1abd7b4daa980f3c4e7e622bca77f5afd5a7
    Flowfin/jellyfin-plugin-sso            c50ea5247148f0a3011bb2f87875c7e8edad13c9
    Flowfin/jellyfin-plugin-watchlist      a0f8498b8a4c6f5fc0ae1f4ecbae2047e7666d8b
    Flowfin/lab                            87affb3ca20e7d373c049faf93682dac660bf27d
    Flowfin/site                           f34392a1aea9a5685b9ced0cc52db94686020cc2
    erawright/steinbruch                   4c1f97011aba8487d4989b27ede9ea3907361985
    iderex/Easy-Compliance-Manager         46a0f5e6baf5e667b0f314ca5e9ed67708c341a3
    iderex/cudec                           880e0e3d6cb4a6e0bf3016f756bb6ba0cf512ba9
    iderex/lichttisch                      92f0ad415f82f6233cc6c24532cae5ffa10c915b
    iderex/retusche                        f9a5231933406b319fd657e8b1eabe6443f39437
    iderex/swarm.asm                       ad0be0bc08b68146d3e1d4bd7385ed5effee0c5c
    iderex/wache                           7b0444a82e1268a4a0dd1fc865ef0fed6c98f845

THE DERIVATION READS VALUES OUT OF EVERY FILE, WHICH IS WHERE A READING CAN
QUIETLY STOP BEING ABOUT THE BYTES THE ID COLUMN COUNTS. The comparisons above
need an `oid` and the two contract lines; the table below needs the whole block
parsed, so a decoder that mangled anything on the way would move a value rather
than announce itself. Each copy was written out and hashed:

    for f in copies/*.yml; do printf '%s %s\n' "$(git hash-object "$f")" "$f"; done
    14 of 14 equal the oid the query returned beside that board

`git hash-object` computes the same blob id git would, so an equal hash is the
whole file matching and not a sample of it.

### The thirteen of 6 September have not moved, and the fourteenth is Flowfin/hub

Joined against the table the section above landed, extracted with the scoped form
that section now carries:

    git show origin/main:docs/dependabot-across-the-boards.md |
      sed -n '/^## The reading on 6 September/,/^## /p' |
      grep -E '^    [A-Za-z0-9._/-]+ +[0-9a-f]{40}$' | awk '{print $1"\t"$2}' | sort > sep06.tsv
    join -t$'\t' sep06.tsv sep07.tsv -o 0,1.2,2.2 |
      awk -F'\t' '{print ($2==$3 ? "SAME  " : "MOVED ") $1}'

    SAME on all thirteen
    comm -23: nothing only in the 6 September list
    comm -13: Flowfin/hub, only in this one

`Flowfin/hub` is the fourteenth board, and the file arrived there between 6 and
7 September. It names neither contract line, as the twelve other boards without
the contract do not:

    jq -r '.data | to_entries[] | .value | select(.dep != null)
           | [ .nameWithOwner,
               (.dep.text | split("\n")
                | map(select(test("^#   (origin|taken-at): ")))
                | map(sub("^#   [a-z-]+: +";""))
                | join(" | ")) ] | @tsv' batch.*.json

    Flowfin/hub            (neither line)
    the other twelve       (neither line)
    iderex/wache           iderex/wache templates/dependabot.yml | a637780d22d9472988fc5c330643de63b4e5e68a

One copy of fourteen names an origin and a commit, and it is this board's, as on
29 and 31 August and 4 and 6 September. Ten days after the template landed no
board outside this one has taken it, and the third state this page describes -
not up to date, not drifted, unjudged - now covers thirteen boards where it
covered twelve.

### The four values, re-derived over the thirteen

The population is the thirteen boards OTHER than this one. This board's copy IS
the template's block, so counting it would be the template voting for itself, and
the derivation the header records was taken over a population this board was not
in. The values are read out of the `github-actions` block the drift test extracts
and nowhere else, so a board's own ecosystems below it cannot reach this table:

    board                              days  interval  limit  groups  labels
    Flowfin/core                       7     weekly    1      yes     yes
    Flowfin/hub                        7     weekly    10     yes     no
    Flowfin/jellyfin-plugin-invites    7     monthly   3      yes     yes
    Flowfin/jellyfin-plugin-sso        7     weekly    10     yes     no
    Flowfin/jellyfin-plugin-watchlist  7     weekly    10     yes     no
    Flowfin/lab                        7     weekly    5      no      yes
    Flowfin/site                       7     weekly    -      yes     yes
    erawright/steinbruch               7     weekly    -      yes     no
    iderex/Easy-Compliance-Manager     7     weekly    5      yes     no
    iderex/cudec                       7     weekly    5      yes     no
    iderex/lichttisch                  7     weekly    5      yes     yes
    iderex/retusche                    7     weekly    3      yes     no
    iderex/swarm.asm                   7     weekly    5      yes     no

    default-days               7 on 13 of 13
    interval                   weekly 12, monthly 1
    open-pull-requests-limit   5 on five, 10 on three, 3 on two, 1 on one, absent on two
    groups                     present on 12 of 13
    labels                     set on 5 of 13, absent on 8

EVERY VALUE THE TEMPLATE HOLDS IS STILL THE ONE WITH THE MOST BOARDS BEHIND IT,
and that is the answer rather than a formality: the file could have become a
preference nobody holds while the copy count went on being the thing anybody
measured. `default-days: 7` is unanimous on thirteen as it was on nine.
`interval: weekly` is 12 of 13. `groups` is 12 of 13. Labels are still set by a
minority, so leaving them out is still not a decision taken on anybody's behalf.

`open-pull-requests-limit` IS THE ONE THAT MOVED AND IT IS THE WEAKEST OF THE
FOUR. Five of thirteen hold the canonical `5`, which is a plurality of 38 per
cent where the nine gave it 44, and three boards now hold `10` against two
before. It is still ahead and it is ahead of a spread rather than of one rival.
This is the value to re-derive first the next time a board lands a file, and the
only one where the template could stop being derived without any board disagreeing
with it directly.

The quoting is derived the same way and it holds:

    package-ecosystem unquoted    8 of 13
    interval unquoted             8 of 13
    directory quoted             10 of 13

### The header was carrying a live count of nine, and it is dated now

`templates/dependabot.yml` said its values are what "the nine boards that already
configure an updater actually hold", in the present tense, with `all nine` and
`eight of nine` beside each one. Thirteen boards hold one. The header carries the
figures above with the date each derivation was taken instead, so a reader can
see when to run it again rather than reading a population that stopped being the
population on 31 August, which is the section above that first counted more than
nine.

THE BLOCK IS UNTOUCHED AND THIS BOARD'S COPY IS STILL UP TO DATE UNDER THE
CONTRACT, which is a property of the contract rather than luck. The drift test
compares the `github-actions` block, comments dropped, and the near-misses under
`## The first copy, and the first run in which taken-at resolved` already close
that: "a moved value is drift, a rewritten comment is not". Run rather than
cited, against the commit this board's copy names:

    git show a637780d22d9472988fc5c330643de63b4e5e68a:templates/dependabot.yml |
      block | diff - <(block < .github/dependabot.yml)
    diff exit=0

So the copy's `taken-at` still resolves to a template whose block is
byte-identical to it, and nothing here asks any board that had taken the template
to take it again.

### What this section does not evaluate

Whether the four boards other than this one that gained the file since the
template landed were shown it. None of them names it, which is what the contract
lines say; why they did not take it is decided on those boards and written
nowhere this page can read, and the leg that would put the template in front of
them is still `#31`'s.

Whether `Flowfin/hub` meets the red gate step 3 of the README sequence warns
about. It already carries the file, so the sequence is behind it rather than in
front of it, and its own pull-request rules were not read here.

Whether the values would still be derived if the population were the boards a
template SHOULD reach rather than the boards that already configure an updater.
Sixty boards configure none, and a majority taken over the thirteen says nothing
about what the sixty would choose. That is the same bound the section on the
shape decision opens with and this reading does not narrow it.

## The second reading on 7 September, and the red gate was a re-run of a replaced script

The section above closes by naming what it does not evaluate, and one of the
three is whether the red at `Flowfin/core` is new: "the check-run on that head is
the state today; no earlier Dependabot pull request on that board was walked."
It is answered here, and the answer moves the finding rather than dating it.

`Flowfin/core` carries two Dependabot pull requests, one closed and one open:

    gh api -X GET repos/Flowfin/core/pulls -f state=all -f per_page=100 \
      --jq '.[] | select(.user.login=="dependabot[bot]")
            | [.number, .state, .created_at, .head.sha] | @tsv'

    315  open    2026-09-07T05:20:06Z  9ba595887018f659b43e00b8746bec87d0407856
    256  closed  2026-08-31T05:21:16Z  fe2fcb98bac0710820a40a247f2b47bb4aa19ec1

### The red check run is five days younger than the pull request it sits on

Every other check on #256's head ran when the pull request opened. The one that
is red did not:

    gh api repos/Flowfin/core/commits/fe2fcb98/check-runs \
      --jq '.check_runs | sort_by(.started_at) | .[] | [.started_at, .name, .conclusion] | @tsv'

    2026-08-31T05:21:23Z  hygiene / Deterministic PR hygiene   success
    ... nineteen more, all started 2026-08-31T05:21, all success
    2026-09-05T20:57:29Z  Deterministic PR-hygiene checks      failure

and the run it belongs to is the one created with the pull request, on its third
attempt:

    gh run view 33360249735 --repo Flowfin/core --json attempt,createdAt,updatedAt,conclusion
    attempt=3 created=2026-08-31T05:21:20Z updated=2026-09-05T20:57:37Z conclusion=failure

So the local gate produced no verdict at all when #256 opened, and the red is a
re-run started five days later.

### The script that produced the red had already been replaced

`Flowfin/core` decides this in `.github/pr-hygiene/hygiene.sh`, which the
workflow calls, and that file was changed 43 minutes after #256 opened:

    gh api -X GET repos/Flowfin/core/commits -f path=.github/pr-hygiene/hygiene.sh \
      --jq '.[] | [.commit.committer.date, .sha[0:8], (.commit.message|split("\n")[0])] | @tsv'

    2026-08-31T06:04:13Z  28b948fb  Tell a number that names no issue here from a lookup that could not be made (#259)
    2026-08-31T05:16:36Z  630660bd  Propose dependency updates weekly, and let the gate judge a proposal it cannot ask for an issue from (#254)

WHICH OF THE TWO RAN IS DECIDED BY THE LOG AND NOT BY THE DATES, because a
re-run's checkout is not something this page can read. The two versions print
different error lines for the same failure, and the line in the log is the
earlier one, word for word:

    630660bd:413  Could not read issue #${n}. The scope comparison cannot be made, and this run will not pass in place of it.
    28b948fb:482  Could not read issue #${n}, and this repository did not answer either, so nothing was learned about that number. The scope comparison cannot be made, and this run will not pass in place of it.

The failing job printed the first of those. So the red was produced by the
version `Flowfin/core` replaced on 31 August, and the paragraph above that says
this run "took the unreadable branch rather than the absent one" is describing a
choice the version that ran could not make: `630660bd` has no absent branch.
`28b948fb` is the change that added one, and its diff is where the three verdicts
`found`, `absent` and `unreadable` first exist.

### The repaired gate is green on the next proposal, and its green says nothing

`Flowfin/core` #315 opened this morning and every check on its head is green,
including the one that is red above:

    gh api repos/Flowfin/core/commits/9ba59588/check-runs \
      --jq '[.check_runs[] | select(.conclusion != "success")] | length'
    0

Its `changed-paths-inside-scope` leg meets the same shape of body - twenty-two
bare numbers quoted out of upstream release notes - and takes the branch the
repair added:

    ok    names: 25 27 34 35 37 39 3956 3995 4007 4019 4023 4037 4051 4061 4070
          4072 4085 4099 4101 4103 4106 4107
    ...
          issue #25 declares no Scope: line at column zero
          #3956 names no issue on this repository, so it declares no scope. ...

and it ends by saying, on its own line, what its green is worth:

    NOT MADE: no issue this pull request names declares a 'Scope:' line at column
    zero, so the changed paths were compared against nothing. This is a pass with
    no comparison behind it, and it is not a pass with one.

THAT IS THE SECOND OF THE THREE ROUTES THE SECTION ABOVE NAMES, ARRIVING ON THE
BOARD IT USED AS THE EXAMPLE OF THE THIRD. The section says a rule over the body
"passes 65 times in 77 on numbers that belong to another project, which is worse
than a refusal because it is intermittent and its green means nothing". On this
board the green now says so itself, which is better than the boards that pass
silently and is still not a comparison.

### What moves and what does not

The three routes stand. The count of 33 local gates and their columns stand: they
were read out of the boards' own source, not off a run.

What moves is the one live example. `Flowfin/core` is not a board whose gate
reddens a Dependabot pull request today; it is a board that met that failure on
31 August, repaired it inside the hour, and carries one stale red on a re-run of
the pull request that found it. A reader taking that row as the state of the
board would be reading a job that ran a script the board no longer has.

The red also gated nothing. #256 merged with it standing:

    gh api repos/Flowfin/core/pulls/256 --jq '[.merged, .closed_at] | @tsv'
    true  2026-09-06T15:41:53Z

### What this section does not evaluate

Why the run was re-attempted on 5 September. The attempt count is readable and
what asked for it is not, and nothing here supposes a cause.

Whether the other four boards in the table above are reading live gates or stale
ones. Only the red row was walked, because only a red row was ever used as
evidence of a shape; the four green rows say what they said.

Whether any of the 13 body-keyed boards has repaired the same collapse. That
would be a reading of thirteen trees rather than of one, and the tally above is
over source as it stood when it was read.
