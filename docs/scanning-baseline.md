# Scanning across the boards: the baseline, the state, and the recorded reasons

Two readings stand in this page. The board list and the first state reading were
taken on 27 August 2026 against the roster in `iderex/operations` at
`origin/main` `bc2b8bd1f4238d8148f1d4e6fe765a1e8fb614e0`. Everything below the
baseline was re-read on 28 August 2026 at `origin/main`
`b07f8d05ec76b8e47213b863e14ec5ae9d0dd1f8`, after the settings sweep recorded on
`#32` had run, and the figures moved. Every figure carries the command that
produced it, and a settings change moves any of them without touching this file,
so re-run them rather than citing this page.

A THIRD READING WAS TAKEN ON 30 AUGUST 2026 against the roster at
`iderex/operations` `origin/main` `cb137493474faaf32439588f8d893f2e2b1dec87`,
and it is the one the sections below carry where they disagree with the two
above. It moved three things: the open question about default setup is answered
and closed, Dependabot separates into two readings that this page had as one, and
33 roster records turn out to declare a visibility the board does not have.

This is the home `#32` asks for: the baseline written down, the state measured,
and the register a board writes into when it deliberately does not scan. It
rolls nothing out. The settings themselves are changed in each repository's own
configuration and no change here reaches them.

SOMETHING READS THIS PAGE NOW, and that is what changed on 30 August. The
baseline leg of `.github/workflows/fleet-alert-sweep.yml` reads the rows below,
reads the register at the bottom, reads each roster board's live configuration,
and names every board short of its own row. What it does and where it stops is
`## What reads this page` below. Until then the page was a document nothing
executed, which is the half of `#32` that stayed open after the settings sweep.

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

**`zizmor` IS NOT IN EITHER ROW, AND THIS PAGE CARRIED THE QUESTION AS OPEN.**
Two decisions stood on `#32` and only one of them named the tool, so neither
reading was available as a fact and this page said so rather than picking the
one that made a done-condition easier. The contradiction is resolved on `#32` and
the answer is that `zizmor` is OUT.

The reason is not a judgement about the tool. What makes this set a baseline is
that it is turned on as a repository SETTING, so a board either carries it or
carries a recorded reason, and asking is one call per board with no reading of
any tree. `zizmor` is a workflow audit that runs as a job inside a board's own
checks, on that board's schedule, over that board's files, and is named among
that board's required contexts. That is a check a board OWES, not a setting a
board CARRIES. Folding it in would make the baseline mean two things at once and
the cost would land on the guard: a per-board query would have to read workflow
files to answer a question the settings API cannot, and a baseline that cannot be
verified in one call stops being verified.

So the public row is three controls and the private row is one, both as stated
above, and the leg below was already built to exactly that. What that agreement
does NOT buy is the claim that the leg is therefore right - it shows which
reading the leg was written to, and the reading is now decided beside it rather
than inferred from it.

WHAT IS UNCHANGED IS THE ABSENCE ITSELF. No board's `zizmor` state is read
anywhere in this repository, this board runs none, and a `Waives:` line naming it
refuses the run rather than being ignored. What has changed is what that absence
MEANS: nothing to read, rather than a shortfall on every public board.

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

WHICH OF THE TWO THOSE BOARDS ARE IN IS DECIDED NOW, AND THE PARAGRAPH THAT
STOOD HERE SAID IT COULD NOT BE. What it said was that both readings were
reproduced, that this page asserted neither over the other, and that no mechanism
was proposed for the disagreement. There is no disagreement. `not-configured` on
that endpoint is the correct answer for a board that configures CodeQL the OTHER
way - an advanced setup, which is a workflow in the board's own tree calling
`github/codeql-action` and which that endpoint does not describe. It was found by
asking each such board a third question instead of comparing the first two: the
analysis names the workflow that produced it, and whether that file is on the
board today is a reading of the present rather than of the past.

Over the 34 public boards of the roster on 30 August 2026, 6 report default setup
`configured` and 28 report `not-configured`. All 28 of the 28 carry the workflow
their newest analysis names, and every one of those files calls
`github/codeql-action`:

    for r in <the 28>; do
      key=$(gh api "repos/$r/code-scanning/analyses?per_page=30" \
              --jq '[.[] | .analysis_key | select(startswith(".github/workflows/"))] | first // ""')
      gh api "repos/$r/contents/${key%%:*}" --jq '.content' | base64 -d |
        grep -q 'github/codeql-action' && echo present || echo absent
    done | sort | uniq -c
    28 present

So every public board of the roster has CodeQL configured, by one route or the
other, and a rule that judged the default-setup endpoint alone would have named
28 boards that scan. THE HALF OF THE OLD PARAGRAPH THAT SURVIVES IS THE ONE THE
SECOND HALF OF `#32` RESTS ON: `analyses present` is a reading of the PAST and
does not say that scanning is configured NOW, so a sweep that asks only the
alerts endpoint cannot see a board whose setup has come back off. What replaces
the inference is two present-tense readings, either of which satisfies the row -
the default-setup endpoint, and the workflow file itself.

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

## The roster records disagree with the boards about which row they are in

Which row a board is in is decided by its LIVE visibility, because this page
says a private board that goes public crosses into the public row the day its
visibility changes. The roster record is the register's claim about the same
fact, and on 30 August 2026 the two disagree on 33 of the 73 boards, all in one
direction - the record declares `public` and the board is private:

    for f in $(git -C <operations> ls-tree --name-only origin/main store/repo/ | grep -v README); do
      git -C <operations> show "origin/main:$f" |
        awk '/^Owner: /{o=$2} /^Id: /{i=$2} /^Visibility: /{v=$2} END{if(o&&i) print o"/"i"\t"v}'
    done | sort > roster.tsv
    while read -r b _; do printf '%s\t%s\n' "$b" "$(gh api "repos/$b" --jq '.visibility')"; done < roster.tsv | sort > live.tsv
    join -t$'\t' roster.tsv live.tsv | awk -F'\t' '$2 != $3' | wc -l
    33
    join -t$'\t' roster.tsv live.tsv | awk -F'\t' '$2 != $3 { print $2 " -> " $3 }' | sort | uniq -c
    33 public -> private

Read against one board on both sides, so the join is not the only thing behind
the count:

    git -C <operations> show origin/main:store/repo/attrappe.md | grep -m1 '^Visibility: '
    Visibility: public
    gh api repos/iderex/attrappe --jq '[.visibility, (.private | tostring)] | @tsv'
    private    true

THIS IS NOT A SCANNING GAP AND IT IS NAMED HERE BECAUSE IT DECIDES ONE. A board
in the private row is at its baseline with both Advanced Security features off,
and a board in the public row with them off is behind it, so a register that puts
33 boards in the wrong row is a register that would answer the question this page
exists for backwards. The records are in another tree and nothing here changes
them; the leg below names the disagreement on every run rather than choosing a
side of it.

## The recorded reasons

### `iderex/jellyfin-web` - Dependabot alerts off on a fork

Waives: dependabot-alerts
Reason: the board is a fork of `jellyfin/jellyfin-web` and carried 62 open
Dependabot alerts, every one inherited from upstream's dependency tree. That
surface is watched upstream.
Decided: 27 August 2026, and recorded on `#32`.
Expires when: the fork grows security-relevant divergence of its own. The sweep
should say so when it does.

The reason is executed rather than only written, and the state it is a reason for
is the one the board reports:

    gh api repos/iderex/jellyfin-web/vulnerability-alerts -i | head -1
    HTTP/2.0 404 Not Found

This board is not in the roster the counts above are taken over, so the sweep
names the waiver as being for a board outside the population it swept rather than
holding anything out with it. That is the entry doing what it says and not a
defect. It is the shape the entries below follow.

### `iderex/.github` - no CodeQL-supported language

Waives: code-scanning
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

What they get instead is Dependabot and the local gate's own legs. Which of the
two things called Dependabot that means is separated below, and one of the two is
swept here now.

### `iderex/agent-operations` - archived, and judged against no row

The one archived board of the roster reports Dependabot alerts off:

    gh api repos/iderex/agent-operations/vulnerability-alerts -i | head -1
    HTTP/2.0 404 Not Found

It carries no `Waives:` line and is not an entry in this register, because no
setting on an archived repository can be changed and a recorded reason is for a
board that could sit inside its row and does not. The leg below names it as
archived and judges it against neither row. Reopening the board takes it out of
that state and back into whichever row its visibility puts it in.

### `iderex/wache`, `iderex/Typeset`, `iderex/iderex` - the entry that expired

This page recorded these three as public boards with no CodeQL analysis and no
detected language. All three now carry analyses and `iderex/Typeset` reports
`Swift`, so what is left of the entry is the correction: what was recorded was
true on 27 August, stopped being true on 28 August, and was found by re-running
the sweep rather than by re-reading the sentence.

WHAT THE EXPIRED ENTRY WAS RIGHT ABOUT IS ANSWERED NOW, AND ONE CLAUSE OF IT WAS
NEVER A FACT. It said the baseline puts `zizmor` over the workflow surface. The
baseline does not: `## The baseline` above carries the decision and the reason,
and `zizmor` is in neither row. THIS PARAGRAPH SAID THE QUESTION WAS UNDECIDED
AND POINTED AT `## What is not covered here` FOR IT; both halves of that have
moved and are corrected here rather than left pointing at a bullet that no longer
holds the question.

What holds regardless of that answer, and did before it: the workflow surface is
most of what these three boards are, a CodeQL analysis at `/language:actions` is
not a `zizmor` scan, this board runs no `zizmor` today, and no board's `zizmor`
state is read anywhere in this page. The roster record for this board already
says it carries none of the five universal guards yet, and that is the same
absence read from the other side. What the decision changes is not the absence
but what it means - a board without `zizmor` is not short of its baseline row.

## What reads this page

The baseline leg of `.github/workflows/fleet-alert-sweep.yml`. It runs in the
same scheduled job as the alert delta and is a different question: the delta asks
whether a count rose, this asks whether the configuration that produces counts is
still there. A board that loses code scanning stops answering the alerts
endpoint, and the delta is deliberately written to pass over a pair it cannot
read rather than report it as a fall to zero - so without this leg a board could
leave its row and the sweep would be silent by design.

**What it reads per board.** The live visibility, which decides the row. Whether
the board is archived. Secret scanning and push protection, from the repository
object. Code scanning, from the default-setup endpoint AND from whether the
workflow its newest analysis names is on the board today calling
`github/codeql-action` - either satisfies the row, for the reason the corrected
section above sets out. Dependabot alerts, from `/vulnerability-alerts`, which
answers 204 for enabled and 404 for disabled and so separates a feature that is
off from an endpoint that refused, which the alerts endpoint does not.

**How a recorded reason is declared.** An entry in the register above is a
heading naming its boards in backticks and carrying, at column zero, a line
beginning `Waives` and a colon, then the controls it holds out, comma separated.
The four control names are `code-scanning`, `secret-scanning`, `push-protection`
and `dependabot-alerts`. A heading with no such line waives nothing, which is what
keeps this page's ordinary headings and its expired entries out of the register,
and an entry naming several boards waives for all of them rather than for the
first.

**It fails closed in both directions.** A waiver naming a control the leg does
not judge refuses the run, because it would hold nothing out while reading as
though it did. So does a waiver under a heading that names no board. A waiver for
a control the board now satisfies is named as stale, and a waiver for a board
outside the swept population is named as such rather than passed over.

**IT REFUSES NOW, AND THIS PARAGRAPH SAID IT DID NOT.** What stood here was that
a board short of its row was named in the run summary, that the exit code stayed
the alert delta's, and that a red run would throw the day's alert reading away
because the artifact is kept on a successful run alone. The reason was real. It
is answered by ORDER rather than by trading the refusal away: the judging step
decides the verdict, the artifact is uploaded while the job is still green, and a
LAST step turns a shortfall into a red run after both. So the day's reading is
kept and the shortfall reddens the run.

**WHY A REFUSAL AND NOT A NAME.** "Staying covered is nobody's job" is what `#32`
is about. A board added to the roster tomorrow gets its scanning settings from
nothing, and a line in a run summary nobody is required to read is the same watch
as none. A red run is the only thing here that reaches a person.

**A FAILED RUN STILL KEEPS NOTHING, and this is not an exception to it.** That
rule exists because a run whose FILING failed halfway would hand the rises it did
not manage to file to the next run as the baseline they were measured against. A
configuration shortfall is not a filing failure: the alert leg ran to the end and
filed what it had, so its reading is exactly the one the next run needs.

**THREE ANSWERS AND NOT TWO.** A count that is absent, or that is not a number,
is a run whose judging step did not get that far - not a clean fleet. The last
step says so and exits green rather than inventing a verdict, because the job is
already red from whatever killed the step before it and a second error would only
bury the first. `baseline_refuses` in the workflow decides all three and has
fixtures for each.

**WHERE THAT NAMING LANDS BEYOND THE RUN, AND THIS PARAGRAPH SAID IT WAS AN OPEN
QUESTION.** It is `#48` and it is decided: every ordinary run writes the same
text it puts in its step summary into the body of one CLOSED issue on this board,
`iderex/wache#67`. Closed is the whole point of the choice. A permanently open
issue carrying a table is read as work by every count over this board, forever,
while a closed one still takes an edit, still has a stable number, is found by
search, and keeps every earlier reading in its edit history - so the register is
permanent and indexed without ever being counted as something left to do.

    git show origin/main:.github/workflows/fleet-alert-sweep.yml |
      grep -n 'REGISTER_ISSUE:'

**WHICH RUNS WRITE IT IS NARROWER THAN WHICH RUNS REPORT.** `may_register` in
that file answers it and has fixtures: a run writes the register when it is
allowed to keep a reading - not seeded, not a dry run - AND is on this board's
default branch. A dispatch on a feature branch is a rehearsal of a modified
sweep, and a rehearsal that overwrote the register would put a reading produced
by code that never landed where a reader takes it for the fleet's own history.

**IT WARNS RATHER THAN REFUSING WHERE THE WRITE FAILS, AND THAT IS A RESIDUAL
RATHER THAN AN OVERSIGHT.** A refusal there would throw the day's alert reading
away for the same reason the paragraph above gives, so a register that answered
anything but `200` leaves the PREVIOUS run's reading in place while the run
reports today's. The run says so on its own output in those words. Nothing reads
that line, which is what makes it a residual: a register that silently stopped
being written looks, from this page, exactly like one that is current.

**What it named on the first fleet-wide run of its judgement**, taken by hand
against the readings above on 30 August 2026 before it had run on a schedule: one
archived board, 33 boards whose roster record disagrees with their visibility,
and no board short of any of the four controls.

## What is not covered here

- No setting is changed by anything in this repository, and no route here reads
  one either. This page is a reading and a register; the state it records went
  stale the moment it was written, and the correction above is what that looks
  like when it happens within a day.
- DEPENDABOT IS TWO FEATURES AND THIS PAGE HAD THEM AS ONE. Dependabot ALERTS
  are the scanning half: the platform matches a board's dependency graph against
  its advisory database and reports what it finds. Dependabot VERSION UPDATES are
  the other half, configured by `.github/dependabot.yml` in each board's own
  tree, and they open pull requests rather than report anything. The bullet that
  stood here read the baseline's `Dependabot` as the second alone and called it
  the further from met of the two.

  The alert half is swept now, and it is on almost everywhere:

      while read -r b; do gh api "repos/$b/vulnerability-alerts" -i | head -1; done < boards.txt |
        awk '{print $2}' | sort | uniq -c
      72 204
       1 404

  The one board answering 404 is `iderex/agent-operations`, which is archived and
  has its own entry above. The update half is read in
  `docs/dependabot-across-the-boards.md`, returns 63 of the 73 roster boards
  without the file, and is swept nowhere here. WHICH OF THE TWO THE BASELINE ROW
  MEANS IS NOT DECIDED ON THIS PAGE. The decision it records says `dependabot`
  and nothing finer, and reading it either way changes what `#32` still owes, so
  it is left as the open question it is rather than settled by whichever reading
  makes a done-condition easier. The update half is `#24` either way.
- `zizmor` IS OUT OF BOTH ROWS AND THIS BULLET HELD THE QUESTION OPEN. It read
  that two decisions stood on `#32` and differed on exactly this, that neither
  reading was available as a fact, and that reading it either way changed what
  `#32` still owed. The contradiction is taken on `#32` and the answer is that
  `zizmor` is not part of the baseline; the reason is at `## The baseline`
  above, where the row is declared, rather than restated here. The row is still
  derived rather than pasted, so the figure cannot drift against the row it is
  about:

      git show origin/main:docs/scanning-baseline.md |
        sed -n '/^\*\*Public boards\./,/^$/p' | grep -c zizmor
      0

  THE TWO DECISIONS BOTH STILL STAND ON THE ISSUE and a reader who finds them
  will find them disagreeing. The first is superseded on this one point and the
  second stands as written, which is recorded there rather than tidied away:

      gh issue view 32 --repo iderex/wache --json comments \
        --jq '.comments[] | select(.body | test("(?i)^(decision|deciding)"))
              | [(.body|test("zizmor")), (.body|.[0:52])] | @tsv'
      true    Decision: the baseline is CodeQL plus secret scannin
      false   Deciding the baseline. The standing rule is that not

- WHAT DID NOT MOVE WITH THE ANSWER, and this is the half worth reading. No
  board's `zizmor` state is read anywhere in this repository, this board runs
  none, and the leg above does not judge it. A `Waives:` line naming it refuses
  the run rather than being ignored, which is the register failing closed on a
  control nothing reads. What the decision changes is only what that absence
  MEANS: nothing to read, rather than a shortfall on every public board. So a
  board that wants `zizmor` still gets nothing from this page, and that is now a
  scope statement instead of a gap.
- Whether a board that reports `analyses present` analyses the whole tree, or
  which languages its analysis covers. The endpoint says an analysis exists.
- THE FIGURES ABOVE STILL COME FROM COMMANDS SOMEBODY CHOSE TO RUN, and this
  bullet said nothing here runs on a schedule. The leg described above does, in
  the sweep `#31` landed. What it does not do is write this page: a scheduled run
  cannot land a document on this mainline, because the default branch takes pull
  requests only and the ruleset has no bypass actors. So the state recorded here
  goes stale exactly as it did before. WHAT HAS CHANGED IS WHERE THE CURRENT
  READING LIVES, and this bullet said it was the run's own summary: it is
  `iderex/wache#67`, permanently and by number, for the reasons at
  `## What reads this page` above. The figures written into THIS page are still a
  reading of the day somebody took them.
- The leg reads four controls and the row a board is in. It reads no alert, no
  finding and no severity; whether a board that scans scans WELL is not a
  question anything here asks.
