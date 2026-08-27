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
    uses: iderex/wache/.github/workflows/pr-hygiene.yml@113085b269d3437a3f96ff9e7060b64b0af88ab1 # v1.2.0
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
    uses: iderex/wache/.github/workflows/pr-hygiene.yml@113085b269d3437a3f96ff9e7060b64b0af88ab1 # v1.2.0
    with:
      subject_names_issue: false
```

Switching it off is a statement that the board does not meet the rule yet. It
is better than not calling the check at all, which is a statement about
nothing.

### If you are deleting a local copy of this check

Sixteen boards call this check and keep their own `pr-hygiene.yml` beside it.
Reading those sixteen files found five answers this check had no form of, and
three of them would turn work that passes on those boards today into work that
is refused the day the local file goes. Three of the five are inputs here now
and one is a trigger you declare; the reading and what is left of it are
`#27`.

READ YOUR OWN COPY BEFORE YOU DELETE IT and carry its answer across in the same
change, where a reader can see it. Every default below is what this check
already did, so a board that adds nothing moves nowhere.

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened, edited, ready_for_review]

jobs:
  hygiene:
    uses: iderex/wache/.github/workflows/pr-hygiene.yml@c96bdee7eefddee8120ae4be58e6f3d4eaa1f157
    with:
      subject_exempt_authors: |
        dependabot[bot]@users.noreply.github.com
      outside_contributions_exempt: true
      message_is_ascii: true
```

NO RELEASE CARRIES THESE THREE INPUTS YET, so the pin above is the commit that
added them rather than a tag, and no version comment is invented beside it.
`v1.2.0` is the newest tag; it declares `subject_names_issue` and nothing else,
which is above the floor `#27` asks for and below what this example passes. A
caller in that state does not quietly fall back to the defaults - the run
reaches `startup_failure` with no job and no check run, so a board that deleted
its own copy in the same change is left with neither gate. That was read on this
board's pull request `#37` rather than assumed.

`subject_exempt_authors` is the automation exemption three boards hold. Without
it a board that deletes its copy starts refusing its own dependency-update pull
requests, because a bot cannot name an issue it does not know about. It is an
allowlist of addresses and not a `*[bot]@...` glob, so a human cannot exempt
themselves by choosing an author address.

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
    uses: iderex/wache/.github/workflows/unicode-guard.yml@113085b269d3437a3f96ff9e7060b64b0af88ab1 # v1.2.0
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
    uses: iderex/wache/.github/workflows/dco.yml@30fee603352e802464f87d65a5fa1e44a351067e
    with:
      references: |
        DCO
        CONTRIBUTING.md
```

NO RELEASE CARRIES THIS GATE YET, so there is no version for the comment beside
the hash and none is invented here. The hash above is the commit that added the
gate, which is what a caller pins until a tag carries it.

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
its own copy for exactly that reason.

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
uses: iderex/wache/.github/workflows/pr-hygiene.yml@113085b269d3437a3f96ff9e7060b64b0af88ab1 # v1.2.0
```

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
concurrency group, and it is the one release a caller cannot skip.

Below `v1.2.0` the shared workflow's group is the bare `pr-hygiene-<PR number>`,
and a called workflow's concurrency applies to the CALLING board's run. With
`cancel-in-progress` the shared run therefore cancelled that board's own
pr-hygiene run before it executed a step, on every push. A cancelled guard
blocks nothing where no status check is required, so the board's own rules stop
being judged while still looking present.

The tags decide this and the paragraph above does not:

```sh
git ls-remote --tags https://github.com/iderex/wache | grep -v '\^{}'
git rev-list -n1 v1.2.0
```

NO TAG CARRIES THE HYGIENE CHECK'S THREE MIGRATION INPUTS EITHER.
`subject_exempt_authors`, `outside_contributions_exempt` and `message_is_ascii`
landed after `v1.2.0` was cut, so a board carrying an answer across from its own
copy pins the commit that added them rather than a version:

```sh
gh api "repos/iderex/wache/contents/.github/workflows/pr-hygiene.yml?ref=v1.2.0" --jq '.content' |
  base64 -d | grep -cE '^      (subject_exempt_authors|outside_contributions_exempt|message_is_ascii):'
0
```

NO TAG CARRIES THE DCO GATE. `v1.2.0` is the newest tag and was cut before that
file existed, so a board calling the gate pins the commit that added it rather
than a version:

```sh
gh api "repos/iderex/wache/contents/.github/workflows/dco.yml?ref=v1.2.0"
{"message":"Not Found","status":"404"}
```

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
