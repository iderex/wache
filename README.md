# wache

One implementation of the checks every board shares, called rather than copied.

On 22 August 2026 I counted the pull request hygiene check across my boards.
Nineteen boards carried it. All nineteen were separate implementations of the
same idea, written in four languages, and no two of them agreed. What they
enforce is three or four rules. What differs is the wording: fourteen boards
want a pull request to name an issue, and they say so twenty-eight different
ways. On top of that sat four questions nobody had ever answered once, so each
board had answered them separately and differently.

The cost showed up when I tried to change two lines across thirty-seven boards
and needed six attempts, because every board refused for a different reason.

This repository holds the checks once. A board calls them instead of keeping a
copy, so a fix lands in one place and reaches every caller.

## What is here

| Check | What it refuses |
|---|---|
| `pr-hygiene.yml` | an empty body, a placeholder title, a closing keyword inside a paragraph, a commit subject naming no issue |
| `unicode-guard.yml` | Trojan Source: bidirectional overrides, isolates and marks, and zero-width characters in the tracked tree |
| `dco.yml` | a commit with no Developer Certificate of Origin sign-off matching its author, an author or trailer address that is not an address, and a board pointing a contributor at a document it does not hold |

Each of them runs fixtures before it judges anything. If a fixture disagrees
with the rule it is meant to prove, the run fails and judges nothing, rather
than passing your change on a check that had quietly stopped working.

ONE THING HERE IS COPIED RATHER THAN CALLED, and it is the only file on this
board that cannot be a reusable workflow. GitHub reads `.github/dependabot.yml`
out of each repository, so `templates/dependabot.yml` is a template: it holds the
`github-actions` block every board's copy can share, a board's own ecosystems go
below it, and the copy names the origin and the commit it was taken at in a
leading comment. This board measures drift and writes into no other tree. The
reading, the copy contract and the drift test are
[`docs/dependabot-across-the-boards.md`](docs/dependabot-across-the-boards.md).

THIS BOARD IS ALSO THE FIRST COPY, for the same reason it is the first caller of
the check it holds: a contract nobody has executed is a paragraph.
`.github/dependabot.yml` here carries the two contract lines and is the run in
which `taken-at` first resolved to a commit instead of the canonical side being
supplied by hand.

`docs/standardisation-survey.md` is the reading behind what comes here next: what
the boards hold more than once, how far the copies have drifted, and the shapes
standardisation could take. It ends in a decision request rather than in an
implementation.

## Calling a check

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened, edited]

jobs:
  hygiene:
    uses: iderex/wache/.github/workflows/pr-hygiene.yml@4d91113f744527d4dd6397bb09fb276ef18b09fc # v1.3.0
```

DECLARE THE TYPES. The default set for `pull_request` is `opened`,
`synchronize` and `reopened`, and this check judges the body and the title -
both of which are editable after the run that judged them. Without `edited` a
body is read once and never again, so a green status can stand over bytes that
have since been replaced. It is not a theoretical case: it happened on this
board's own pull request #21 before the trigger was fixed.

The trigger has to be declared by the caller. `pr-hygiene.yml` runs on
`workflow_call` and carries no events of its own, so no change here can close
that gap for a board that leaves the types out.

A board whose commit subjects do not carry an issue number yet can switch that
one rule off and keep the rest:

```yaml
jobs:
  hygiene:
    uses: iderex/wache/.github/workflows/pr-hygiene.yml@4d91113f744527d4dd6397bb09fb276ef18b09fc # v1.3.0
    with:
      subject_names_issue: false
```

Switching it off is a statement that the board does not meet the rule yet. It
is better than not calling the check at all, which is a statement about
nothing.

### If you are deleting a local copy of this check

Eighteen of the nineteen boards that call this check keep a gate of their own
beside it, and `iderex/lesesaal` is the only one that does not. Seventeen of the
eighteen keep it as `pr-hygiene.yml`; `iderex/pruefstand` keeps it as
`hygiene.yml`, which is why a board answering "we have no local copy" should
check the file rather than the filename. Reading the copies found five answers
this check had no form of, and three of them would turn work that passes on
those boards today into work that is refused the day the local file goes. What this check has a form of now is the
list of inputs below rather than a count written here; `ready_for_review` is a
trigger you declare and not an input; and one answer is decided NOT to come here
at all, which is the `Scope:` path comparison at the end of this section. The
reading is `#27` and the two answers that need the caller's permission are
`#36`.

READ YOUR OWN COPY BEFORE YOU DELETE IT and carry its answer across in the same
change, where a reader can see it. Every default below is what this check
already did, so a board that adds nothing moves nowhere.

READ YOUR RULESET TOO, BECAUSE THE DELETE CAN STRAND A REQUIRED CHECK. If your
branch ruleset requires the status check your local file produces, deleting the
file leaves a required context that will never be reported again, and a required
check that never reports leaves every pull request PERMANENTLY PENDING rather
than red - nothing merges and nothing says why. Two boards are in that position
today:

```
$ gh api "repos/$BOARD/rulesets" --jq '.[].id' | while read -r id; do
    gh api "repos/$BOARD/rulesets/$id" --jq '[.rules[]? |
      select(.type=="required_status_checks") |
      .parameters.required_status_checks[]?.context] | join(",")'
  done
iderex/hoersaal     ... ,pr-hygiene, ...
iderex/stammtisch   ... ,Deterministic PR-hygiene checks, ...
```

THIS CHECK PRODUCES NEITHER OF THOSE NAMES, and the near miss is the part to
read slowly:

```
$ git show origin/main:.github/workflows/pr-hygiene.yml | sed -n '142p'
    name: Deterministic PR hygiene
```

`Deterministic PR hygiene` is one hyphen and one word away from a required
`Deterministic PR-hygiene checks`, and a required context is matched by its
literal name. A called workflow's check run also arrives as
`<your caller's job id> / <the job name above>`, so even an exact match would
not land unprefixed. Do not assume the names line up; run the command.

So on a board whose ruleset names its local check, the ruleset edit belongs in
the same change as the delete: take the old context out and put the one your
caller actually produces in, read off a run rather than typed. The reading
behind this is `docs/local-hygiene-answers.md` and the issue is `#27`.

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened, edited, ready_for_review]

jobs:
  hygiene:
    uses: iderex/wache/.github/workflows/pr-hygiene.yml@4d91113f744527d4dd6397bb09fb276ef18b09fc # v1.3.0
    with:
      subject_exempt_authors: |
        dependabot[bot]@users.noreply.github.com
      outside_contributions_exempt: true
      message_is_ascii: true
```

`v1.3.0` IS THE FIRST RELEASE THAT DECLARES THESE THREE INPUTS, so the pin above
names a version in its comment again rather than standing as a bare commit.
`v1.2.0` declares `subject_names_issue` and nothing else, which is above the
floor `#27` asks for and below what this example passes, and a caller in that
state does not quietly fall back to the defaults - the run reaches
`startup_failure` with no job and no check run, so a board that deleted its own
copy in the same change is left with neither gate. That was read on this board's
pull request `#37` rather than assumed, and it is why the version beside the
hash matters here more than anywhere else in this file.

`subject_exempt_authors` is the automation exemption three boards hold. Without
it a board that deletes its copy starts refusing its own dependency-update pull
requests, because a bot cannot name an issue it does not know about. It is an
allowlist of addresses and not a `*[bot]@...` glob.

THE ALLOWLIST IS LOOSER AT THE COMMIT THIS EXAMPLE PINS THAN IT IS ON `main`,
and the sentence that stood here said flatly that a human could not exempt
themselves by choosing an author address. At `v1.3.0` an entry matches the whole
`<id>+<entry>` form GitHub mints without asking what the `<id>` is, so any local
part in front of a listed address is exempt; on `main` the `<id>` has to be
digits. That is `#55`, and `## Versions` below carries the reading, the two
commands behind it and what it costs a board today.

`outside_contributions_exempt` waives the subject rule for a fork or for an
author who is not an owner, member or collaborator. On a board that takes
outside changes, the alternative is refusing a new contributor's first commit
for a convention nobody has told them. `subject_names_issue: false` is the only
other lever and it turns the rule off for everybody rather than for the case it
was waived for.

`message_is_ascii` refuses a commit message carrying a byte outside printable
US-ASCII, tab and newline, and refuses a scanner error rather than reading one
as a clean message. This is the commit MESSAGE. The tracked tree is the unicode
guard's subject and a different question.

`ready_for_review` IS A TRIGGER AND NOT AN INPUT, so no change here can add it
for you. Two boards declare it. On a board that opens drafts, a pull request
taken out of draft is not re-judged without it, so a check that was red on the
draft is not re-run at the moment the change becomes reviewable.

### Resolving the number instead of matching its shape

`resolve_referenced_numbers` asks the tracker whether the number a commit
subject names is an issue at all. Without it this check matches `[#181]` and
`owner/repo#N` as SHAPES and looks nothing up, so a subject naming a pull
request, or a number nobody ever opened, passes here. `iderex/stammtisch`,
`iderex/nachtwache` and `iderex/hallraum` hold that answer in their own copies
today, AND THEY ARE THE THREE BOARDS THAT MUST SET THIS INPUT IN THE SAME
CHANGE THAT DELETES THEIR FILE. Every other board loses nothing by ignoring it:
it is off by default.

IT COSTS TWO LINES IN YOUR CALLER AND NOT ONE, and the second one is the part
that surprises people:

```yaml
permissions:
  contents: read
  issues: read
```

```yaml
    with:
      resolve_referenced_numbers: true
    secrets:
      issue_read_token: ${{ secrets.GITHUB_TOKEN }}
```

The permission alone is not enough and the secret alone is not enough. A called
workflow cannot hold more permission than its caller, so this file cannot ask
for `issues: read` on your behalf - and it may not ask for it unconditionally
either. That was read rather than reasoned about: a version of this job that
declared `issues: read` in its own `permissions:` block, gated behind the input
so it would be SKIPPED for every caller that had not asked for it, still put
this board's own caller into `startup_failure` with no job and no check run. The
comparison happens before any job exists, so the gate never runs. Every board
calling this file today grants `contents: read` alone, so that shape would have
taken the hygiene check off all seventeen in one merge. The token therefore
arrives as an optional secret, which costs a caller that wants nothing exactly
nothing.

WHAT IT REFUSES, and the fourth case is the one worth reading. A number that
resolves to an issue passes. A number that resolves to a pull request is
refused. A number that does not exist is refused. AND A LOOKUP THAT DID NOT
ANSWER IS REFUSED TOO - a 403 because the permission is missing, a 500, a
request that never connected - because a reading nothing could make is not a
reading that found nothing wrong. A caller that turns the input on and passes no
token is refused before the first lookup, with the two lines above named in the
error.

WHAT IT DOES NOT JUDGE is whether the issue a subject names is the issue the
change belongs to. It resolves a number. The run prints that line itself.

NO RELEASED TAG DECLARES THIS INPUT YET, so the pinned examples above do not
pass it and cannot: this board's `calling-examples.yml` resolves every pin in
every tracked document and refuses a key the pinned commit does not declare,
which is exactly the trap it exists for. Read the tags rather than this
sentence, because this sentence goes stale the day a release is cut and the
command does not:

```sh
for t in $(git ls-remote --tags https://github.com/iderex/wache | grep -v '\^{}' |
             sed 's#.*refs/tags/##'); do
  printf '%-8s %s\n' "$t" \
    "$(gh api "repos/iderex/wache/contents/.github/workflows/pr-hygiene.yml?ref=$t" \
         --jq '.content' | base64 -d | grep -c '^      resolve_referenced_numbers:')"
done
v1.0.0   0
v1.1.0   0
v1.2.0   0
v1.3.0   0
```

### The `Scope:` path comparison is decided not to come here

`iderex/stammtisch` compares the changed paths against the `Scope:` line of each
referenced issue, and nothing on this route reads an issue body. It stays that
way. One board carrying an answer is not seventeen boards asking for one, and a
shared answer nobody asked for is inventory. That board keeps its local answer;
if a shared route is ever justified it arrives as its own issue naming at least
three boards that asked. The decision is on `#36`.

THE DECISION IS UNCHANGED AND THE COUNT IT RECITES IS NOT THE COUNT TODAY, and a
reader who takes "one board" for the present state is reading a sentence written
on 28 August. Five boards hold the comparison: `iderex/stammtisch`,
`Flowfin/core`, `iderex/hallraum`, `iderex/lehrkanzel` and `iderex/pruefstand`,
of which four call this check. Three were found by the reading on `#27` and two
more by `docs/local-hygiene-answers.md`, which also says why the earlier sweeps
could not see them. Nothing here re-opens `#36`: a board holding an answer and a
board asking for a shared one are different statements, and this corrects the
first without asserting the second.

The unicode guard is called the same way, and needs its own triggers because it
reads the tree rather than the pull request:

```yaml
on:
  push:
    branches: ["**"]
  pull_request:
    branches: ["**"]

permissions:
  contents: read

jobs:
  unicode:
    uses: iderex/wache/.github/workflows/unicode-guard.yml@4d91113f744527d4dd6397bb09fb276ef18b09fc # v1.3.0
```

Every branch, not only `main`. A release branch is a code line too, and the
scan is a cheap read-only grep.

## Calling the DCO gate

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read

jobs:
  dco:
    uses: iderex/wache/.github/workflows/dco.yml@4d91113f744527d4dd6397bb09fb276ef18b09fc # v1.3.0
    with:
      references: |
        DCO
        CONTRIBUTING.md
```

`v1.3.0` IS THE FIRST RELEASE THAT CARRIES THIS GATE. `v1.2.0` was cut before
the file existed, so a pin at any earlier tag names a commit that holds no such
workflow. What a caller in that state gets is not read here: the
`startup_failure` above was measured for a pin declaring too few inputs, on this
board's pull request `#37`, and a missing file is a different case nobody has
run.

`references` IS WHAT YOUR BOARD HOLDS AND NOTHING ELSE. The gate names no
document of its own; what a contributor is pointed at is the list you declare,
and the run checks every path in it against your tree before it reads a single
commit. A path that is not there reds the run. That is deliberate: sixty boards
carried this gate, the majority content named a certificate file thirteen of
them do not track and a contributor guide nine of them do not track, and each of
those boards printed a message sending a contributor to a path that is not
there. Declare nothing and the message names nothing, which is honest; declare a
path you do not hold and you find out on your own pull request instead of on
somebody else's.

`exempt_authors` defaults to `dependabot[bot]` alone. Widen it only for an
identity you can point at a configuration for. An exemption whose actor nothing
starts is an open route nobody is watching, and one board had already narrowed
its own copy for exactly that reason. At the commit this example pins it carries
the same looseness as `subject_exempt_authors` above, for the same reason and
with the same repair unreleased, which is `#49` and `## Versions` below.

THE GATE HAS EXECUTED ON A RUNNER. Until it did, everything anybody knew about
it came from running the script extracted from the file on a workstation, which
cannot say whether the workflow parses on the platform or whether the fixtures
pass in the runner's shell. A caller was added to this board for the length of
one pull request and removed again, and on `ubuntu-24.04` all eighteen fixtures
pass and a range whose commits all carry a matching sign-off is reported as
read. The second push to that pull request carried a commit with no trailer, and
the gate refused that commit by hash, named the exact trailer it wanted, and
pointed at the one path the caller had declared - `README.md` and nothing the
board does not hold, which is the whole reason `references` exists. Both runs
and their commands are in `#39`.

NOTHING ON THIS BOARD CALLS THE GATE, so that pull request is the whole of what
has executed. A board adopting it is still the first to run it against its own
history and its own `references`.

The reading behind those figures is
[`docs/dco-tail.md`](docs/dco-tail.md): twenty-eight singleton copies read one at
a time, each difference placed as drift or as a board-local deviation, with the
command beside every number.

## Pin by hash, not by branch

Callers pin by commit hash with the version in a comment beside it:

```yaml
uses: iderex/wache/.github/workflows/pr-hygiene.yml@4d91113f744527d4dd6397bb09fb276ef18b09fc # v1.3.0
```

EVERY CALLING EXAMPLE IN THIS BOARD'S DOCUMENTS IS JUDGED AGAINST ITS OWN PIN.
A pin and the inputs written under it are two independent halves that have to
agree, and they came apart inside one merge (`#38`). `calling-examples.yml`
resolves each ref, reads what that commit declares under
`on.workflow_call.inputs`, and refuses a key passed to it that is not there. It
does not judge whether the version comment beside a pin names the tag that ref
resolves to, and it does not judge whether the ref is reachable from `main`.

`@main` would let this repository change what executes on every board that calls
it, without anybody reviewing the change. Several boards run an action-pin guard
that refuses a moving reference outright, and they are right to.

The cost is that a fix here reaches a board only when that board bumps one line.
That is the trade: one reviewable line per board, instead of one implementation
per board that nobody keeps in step. The measurement behind that claim is on
`iderex/operations#1556`: thirty-seven boards carried the unicode guard as
twenty-one distinct files, and thirty-two of them sat four patch versions behind
on `codeql-action` while one board was current.

## Versions

`v1.0.0` was the hygiene check alone. `v1.1.0` added the unicode guard and
changed nothing in the hygiene check. `v1.2.0` changed only the hygiene check's
concurrency group, and it is the one release a caller cannot skip. `v1.3.0`
changes what the hygiene check REFUSES, and adds the DCO gate and three inputs
beside it.

Below `v1.2.0` the shared workflow's group is the bare `pr-hygiene-<PR number>`,
and a called workflow's concurrency applies to the CALLING board's run. With
`cancel-in-progress` the shared run therefore cancelled that board's own
pr-hygiene run before it executed a step, on every push. A cancelled guard
blocks nothing where no status check is required, so the board's own rules stop
being judged while still looking present.

The tags decide this and the paragraph above does not:

```sh
git ls-remote --tags https://github.com/iderex/wache | grep -v '\^{}'
git rev-list -n1 v1.3.0
```

REPINNING PAST `v1.2.0` MEETS A REFUSAL THAT WAS NOT THERE BEFORE, and it is the
whole of what `v1.3.0` changes for a board that already calls this check. The
hygiene check judges every line of a body that carries a closing keyword, rather
than asking whether one well-formed line stands somewhere in it, so a body that
was green the day before is refused when `closes #8` sits inside a paragraph
above a proper `Closes #7`. Everything else in the release is additive and every
new input defaults to what the check already did.

`v1.3.0` DECLARES THE HYGIENE CHECK'S THREE MIGRATION INPUTS AND `v1.2.0`
DECLARES NONE OF THEM, so a board carrying an answer across from its own copy
pins `v1.3.0` or later rather than the commit that added them:

```sh
gh api "repos/iderex/wache/contents/.github/workflows/pr-hygiene.yml?ref=v1.3.0" --jq '.content' |
  base64 -d | grep -cE '^      (subject_exempt_authors|outside_contributions_exempt|message_is_ascii):'
3
gh api "repos/iderex/wache/contents/.github/workflows/pr-hygiene.yml?ref=v1.2.0" --jq '.content' |
  base64 -d | grep -cE '^      (subject_exempt_authors|outside_contributions_exempt|message_is_ascii):'
0
```

`v1.3.0` CARRIES THE DCO GATE AND NO EARLIER TAG DOES. `v1.2.0` was cut before
that file existed:

```sh
gh api "repos/iderex/wache/contents/.github/workflows/dco.yml?ref=v1.3.0" --jq '.name'
dco.yml
gh api "repos/iderex/wache/contents/.github/workflows/dco.yml?ref=v1.2.0"
{"message":"Not Found","documentation_url":"https://docs.github.com/rest/repos/contents#get-repository-content","status":"404"}
```

NO TAG REACHES THE EXEMPTION REPAIR, so `v1.3.0` is behind `main` on what both
gates let past. At `v1.3.0` an entry in `subject_exempt_authors` or in
`exempt_authors` matches the whole `<id>+<entry>` form GitHub mints without
asking what the `<id>` is, so on an allowlist naming only
`dependabot[bot]@users.noreply.github.com` the address
`attacker+dependabot[bot]@users.noreply.github.com` is exempt. On `main` the
`<id>` has to be digits and that address is refused. The shape is readable
without running anything, and the tags decide the rest:

```sh
git show v1.3.0:.github/workflows/pr-hygiene.yml | grep -n '"$entry" | \*"+$entry"'
git show origin/main:.github/workflows/pr-hygiene.yml | grep -n '\[!0-9\]'
git tag --contains 589c8d2
git tag --contains 71baad6
```

Both `--contains` runs print nothing. `#55` and `#49` are the two repairs,
`589c8d2` and `71baad6` are where they landed, and `#57` is where the matcher
was extracted out of each ref and run rather than read.

WHAT IT COSTS A BOARD TODAY IS NOTHING, and the reason is worth reading before
the paragraph is dismissed. Nothing calls the DCO gate, and on 31 August 2026
every caller of the hygiene check was pinned at `v1.0.0` or `v1.2.0`, neither of
which declares `subject_exempt_authors` at all:

```sh
for t in v1.0.0 v1.2.0; do
  echo "$t $(git show $t:.github/workflows/pr-hygiene.yml |
             grep -c '^      subject_exempt_authors:')"
done
v1.0.0 0
v1.2.0 0
```

The tags are in this clone and that half reproduces; the pins are on other
boards and move, so re-read them rather than citing this paragraph. The cost
arrives with the first board that repins and turns the exemption on, which is
the sequence `#27` writes down. Until a tag carries the repair the tighter
matcher is reachable only by pinning a commit, and whether a tag is cut is
`#25`'s open question rather than this section's answer.

Read them before pinning. A version list kept by hand drifts against the tags,
and this section did: it stopped at `v1.1.0` while the example above it named
the hash of `v1.0.0`, which is the release the concurrency repair is missing
from.

## What the hygiene check asks for

The four points of contention are decided once and applied here rather than
argued again in each board:

- the issue number goes in the commit subject line
- it goes in square brackets, `[#181]`
- a closing keyword such as `Closes #7` goes on a line of its own
- a mention on another board counts, written as `owner/repo#N`

Beyond that it refuses an empty pull request body and a placeholder title.

## The fixtures run before the judgement

A check that has never refused anything is a green line with nothing behind
it. Before this workflow judges your pull request it judges its fixtures, and
every rule has one line that must be refused and one that must pass. If a
fixture disagrees with the rule, the run fails and judges nothing, rather than
passing your change on a check that had quietly stopped working. How many
there are is printed by the run, one line per fixture; a count written here
would drift against the file that decides it, the way the version list above
already did.

The near misses are the interesting half. `#181818` is a colour value and not
an issue mention. `(#181)` is a round bracket and does not count. A bare `#181`
does not count either, because the decision was square brackets. A closing
keyword is judged line by line, so `Closes #7` standing further down the body
does not excuse `closes #8` written into a paragraph above it.

I took that shape from the version [bremsweg](https://github.com/iderex/bremsweg)
had already found for itself.

## Licence

AGPL-3.0-or-later. See [LICENSE](LICENSE) and [NOTICE.md](NOTICE.md).
