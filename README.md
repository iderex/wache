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

## Calling a check

```yaml
jobs:
  hygiene:
    uses: iderex/wache/.github/workflows/pr-hygiene.yml@main
```

A board whose commit subjects do not carry an issue number yet can switch that
one rule off and keep the rest:

```yaml
jobs:
  hygiene:
    uses: iderex/wache/.github/workflows/pr-hygiene.yml@main
    with:
      subject_names_issue: false
```

Switching it off is a statement that the board does not meet the rule yet. It
is better than not calling the check at all, which is a statement about
nothing.

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
it. Before this workflow judges your pull request it judges nine fixtures, and
every rule has one line that must be refused and one that must pass. If a
fixture disagrees with the rule, the run fails and judges nothing, rather than
passing your change on a check that had quietly stopped working.

The near misses are the interesting half. `#181818` is a colour value and not
an issue mention. `(#181)` is a round bracket and does not count. A bare `#181`
does not count either, because the decision was square brackets.

I took that shape from the version [bremsweg](https://github.com/iderex/bremsweg)
had already found for itself.

## Licence

AGPL-3.0-or-later. See [LICENSE](LICENSE) and [NOTICE.md](NOTICE.md).
