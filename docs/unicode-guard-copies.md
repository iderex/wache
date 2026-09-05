# The 63 unicode guards, read rather than counted

Taken on 27 August 2026 against the roster in `iderex/operations`, at
`origin/main` `bc2b8bd1f4238d8148f1d4e6fe765a1e8fb614e0`, over the 73 boards the
`boards` function in `docs/standardisation-survey.md` returns. The figures move
whenever a board lands anything; re-run the commands rather than citing this
page.

A SECOND READING WAS TAKEN ON 31 AUGUST 2026 against the roster at
`iderex/operations` `origin/main`
`66f3e6a3fe24f165f16f75b196431b170405d2ab`, and `## Who calls what today` below
is the one section it replaces. It was taken because the command that section
printed had stopped producing the count printed beside it, which that section
now says in its own words. Every other section here is the 27 August reading and
is untouched: the copy counts, the refusal patterns and the three differences
were not re-read, so nothing above that section is asserted as today's state.

`#28` says of its own counts that they are a byte comparison, and lists as not
evaluated whether the 34 distinct contents differ in what they refuse or only in
how they say it. This is that reading. It is the question a board has to answer
before it deletes its own copy, because a migration that quietly narrows what is
refused is worse than the duplication it removes.

## The byte counts reproduce

    awk -F'\t' '$2=="unicode-guard.yml"{print $3}' inventory.tsv |
      sort | uniq -c | sort -rn | awk '{printf "%s ", $1} END{print ""}'
    28 3 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1

63 copies, 34 distinct contents, largest identical cluster 28, unchanged from
the survey. `inventory.tsv` is rebuilt here by the same shape the survey
declares, with the per-board response parsed only when the call succeeded.

## What they refuse is one string, written 33 times

Each distinct copy was fetched by blob id and its refusal pattern extracted:

    gh api "repos/$r/git/blobs/$sha" --jq '.content' | tr -d '\n' | base64 -d
    grep -hoE "\(\*UTF\)\[[^]]*\]" <file>

    33  (*UTF)[\x{202A}-\x{202E}\x{2066}-\x{2069}\x{200E}\x{200F}\x{061C}\x{200B}-\x{200D}\x{2060}]
     1  no PCRE class in the file          iderex/lesesaal

So 33 of the 34 distinct contents refuse the identical codepoint set, character
for character, and it is the set this board's `unicode-guard.yml` carries. The
34 versions differ in comments, in action pins, in job names and in triggers.
They do not differ in what they refuse. THE ANSWER TO `#28`'S OPEN QUESTION IS
THEREFORE THE CHEAP ONE: for 33 of 34, adopting the shared guard changes nothing
about which characters are caught.

## Three differences that are not wording

**Only one copy proves its pattern before applying it.** Counting the fixture
lines in each distinct copy:

    grep -cE 'must_catch|must_pass|fixture' <file>
    33 lines   iderex/wache
     0 lines   the other 33

Every other copy on the fleet is a grep that has never been shown to refuse
anything. That is the argument this board's own guard opens with, and it turns
out to describe the whole population rather than a hypothetical: a guard that
refuses this rarely can go silent without anybody noticing. It is also the
largest single thing a caller gains, and it is invisible in a byte comparison.

**Three boards scan `main` only.**

    grep -oE 'branches: *\[[^]]*\]' <file>
    branches: [main]    iderex/kartei, iderex/kontor, iderex/lichttisch
    branches: ["**"]    every other copy

The shared guard's own comment names `kontor` for this and no one else. Two more
boards do it, so the comment understates the case rather than misstating it.
The trigger belongs to the caller either way: `unicode-guard.yml` runs on
`workflow_call` and declares no events, so a board that migrates keeps whatever
`branches:` it had unless it changes that line too.

**Two deviations survive no migration as written.**

`iderex/swarm.asm` narrows the scan to a path list instead of the whole tree:

    git grep -nIP "$pattern" -- src tools '*.ps1' .github tests

Its own comment argues the narrowing - `docs/**` is left out on purpose because
that README carries intentional emoji and em dashes - and names a test in its C#
harness that refuses a further narrowing of the list. The shared guard scans
`-- .` and takes no input for a path list, so this board cannot call it without
either widening its scan or the shared guard gaining that input.

`iderex/lesesaal` has no workflow-level implementation at all. Its step is

    go run . ci unicode

which is the same procedure a contributor runs before pushing, so that tree
holds one gate rather than two. Calling the shared guard would give it two, and
the shared file's own comment already records why the Go version did not move
here: a guard needing a toolchain is a guard some boards cannot call.

Both are the case `docs/standardisation-survey.md` states in the abstract - a
board-local deviation has to be expressible as an input or it cannot survive -
now measured on this file. Neither is drift.

## Who calls what today

THE COMMAND THIS SECTION PRINTED NO LONGER PRODUCES THE COUNT PRINTED BESIDE IT,
AND THE COUNT WAS RIGHT ON THE DAY IT WAS WRITTEN. Those are two statements and
this section keeps them apart. What stood here was a `gh search code` query
answered by "seventeen boards, every one through
`.github/workflows/shared-hygiene.yml`". Run on 31 August 2026 the same query
returns three repositories, two of which are boards:

    gh search code "iderex/wache/.github/workflows" --limit 100 \
      --json repository,path --jq '.[] | "\(.repository.nameWithOwner)\t\(.path)"' |
      sort -u
    iderex/messstube    .github/workflows/shared-hygiene.yml
    iderex/pruefstand   .github/workflows/shared-hygiene.yml
    iderex/wache        .github/workflows/calling-examples.yml
    iderex/wache        docs/unicode-guard-copies.md
    iderex/wache        README.md

So anybody who re-ran the command beside the old count read a fleet that had
almost stopped calling this board. What the index does with a file that has not
changed recently is not read here and no mechanism for it is asserted. The
repair is to stop asking the index, because a reading nobody can reproduce is
the defect whatever the answer turns out to be.

READ OFF THE TREES INSTEAD. Every file in every roster board's
`.github/workflows/` fetched and searched for a reference to this board:

    gh api "repos/$b/contents/.github/workflows" --jq '.[]|select(.type=="file")|.path' |
    while IFS= read -r p; do
      gh api "repos/$b/contents/$p" --jq '.content' | base64 -d |
        grep -nE 'iderex/wache/\.github/workflows/[A-Za-z0-9._-]+\.yml'
    done

    73 boards, of which 1 has no .github/workflows directory
    902 files, 902 read
    20 references to iderex/wache/.github/workflows/pr-hygiene.yml, all on uses: lines
     1 reference to iderex/wache/.github/workflows/unicode-guard.yml, on a uses: line

THE ONE `unicode-guard.yml` REFERENCE IS THIS BOARD'S OWN
`.github/workflows/calling-examples.yml`, so no board outside this one calls the
shared guard. One of the twenty `pr-hygiene.yml` references is that same file,
so nineteen other boards call the hygiene check, every one of them through
`.github/workflows/shared-hygiene.yml`. The count this section used to carry was
seventeen; which two boards account for the move is not read here, because the
seventeen were never named in this page and there is nothing to subtract them
from. What is read is that `iderex/messstube` and `iderex/pruefstand` are among
the nineteen and are the only two boards the stale index still returns.

WHERE THIS READING STOPS, and these are different bounds from the ones it
replaces. It covers the whole roster and every workflow file on it, which the
search did not. Six files answered nothing on the first pass and were re-read
rather than counted as carrying no reference. `iderex/lagetisch` has no
`.github/workflows` directory at all: the contents API answers 404, `gh` writes
that body to stdout, and the board is named here rather than counted as one with
no call - the same trap `docs/dependabot-across-the-boards.md` records against a
different endpoint. And the subject is the full
`iderex/wache/.github/workflows/<file>.yml` form, so a bare mention of a file
name in a comment is outside it, which is what the sentence this section used to
carry about one board's own comment was about.

Local copies beside the call, counted off the same listing rather than fetched
again:

    63 boards hold a local .github/workflows/unicode-guard.yml
    34 boards hold a local .github/workflows/pr-hygiene.yml
    19 of the 19 callers keep a local unicode-guard.yml
    17 of the 19 callers keep a local pr-hygiene.yml

It was seventeen of seventeen and sixteen of seventeen on 27 August, so both
moved with the two new callers and neither changed shape: a board that calls the
hygiene check still keeps its own unicode guard, without exception.

And the pins those nineteen carry:

    gh api "repos/$b/contents/.github/workflows/shared-hygiene.yml" --jq '.content' |
      base64 -d |
      grep -oE 'iderex/wache/\.github/workflows/pr-hygiene\.yml@[0-9a-f]+[[:space:]]*#[[:space:]]*v[0-9.]+'

    13 v1.0.0    @9b311243c2d0d0ced7feb957a20bc178acce6a5d
     6 v1.2.0    @113085b269d3437a3f96ff9e7060b64b0af88ab1

Nineteen of nineteen carry a version comment beside the sha, and each tag name
appears against exactly one commit.

## What a first caller has to carry

`#28` records the decision that the guard gets callers, and its first done-when
is a board that calls it with its local copy removed in the same change. This
reading says what that change costs on each kind of board:

- On any of the 33 boards holding a grep copy other than `swarm.asm`, the
  refusal set is unchanged, the fixtures are new, and the only line to think
  about is `branches:` - unchanged for 30 of them, a widening from `main` for
  `kartei`, `kontor` and `lichttisch`.
- On `iderex/swarm.asm` it is a widening of the scanned tree, which that board's
  own tests refuse in the other direction. It is the wrong board to go first
  with.
- On `iderex/lesesaal` it is a second gate over a tree that deliberately holds
  one. Also the wrong board to go first with.

WHAT THIS PAGE DOES NOT EVALUATE. Whether each copy's `on:` block, permissions
and concurrency are equivalent to the shared caller's - only `branches:` was
extracted. And whether the shared guard's pattern is the RIGHT set: every copy
agreeing on it is evidence that nobody has revisited it, not evidence that it is
complete.

## The reading on 4 September, and every figure of 31 August returns

Taken 4 September 2026 against the roster at `iderex/operations` `origin/main`
`e4ce672f773cc675bfe5b88a6e7899c7bf6f66d8`, which holds 74 boards where the 31
August reading had 73. The board added since is `erawright/steinbruch`, whose
owner is not `iderex` - the roster record carried `Owner: iderex` earlier and
the field now reads `erawright`, so a sweep that derived the board list once and
kept it would have asked the wrong account:

    git show e4ce672f773cc675bfe5b88a6e7899c7bf6f66d8:store/repo/steinbruch.md |
      awk '/^Id: /{i=$2} /^Owner: /{o=$2} END{print o"/"i}'
    erawright/steinbruch

It holds one workflow file, `ci.yml`, and no reference to this board.

THE SWEEP ASKS FOR TREES AND NOT FOR FILES, and that is what changed about the
route rather than about the answer. The section above costs one `contents` call
per board and one per file, which over the roster read below is 74 + 923 = 997
of them. The same bytes come back from four calls, when each board's
`.github/workflows` tree is asked for as a GraphQL object with every blob's text
inline, nineteen boards to a query:

    fragment B on Repository {
      nameWithOwner defaultBranchRef{ name target{ oid } }
      wf: object(expression:"HEAD:.github/workflows"){
        ... on Tree{ entries{ name type object{ ... on Blob{ oid isBinary text } } } } }
      dep: object(expression:"HEAD:.github/dependabot.yml"){ ... on Blob{ oid text } } }
    query {
      r1: repository(owner:"Flowfin", name:".github"){ ...B }
      r2: repository(owner:"Flowfin", name:"core"){ ...B }
      ...                                    # one alias per board, in roster order
    }

The board list is the roster derivation `docs/standardisation-survey.md`
declares, written to `roster.txt`, and the aliases above are the first two lines
the generator emits for the first part:

    split -l 19 -d roster.txt roster.part.
    gh api graphql -f query="$q" > batch.$i.json      # once per part

The route also answers the trap the section above records, rather than guarding
against it: a blob's `oid` is a field of a typed object, so an error body cannot
reach the id column at all and there is nothing to validate as hexadecimal. What
it can still do is answer `null`, which is why the count of blobs whose `text`
came back null is printed rather than assumed, and why every reply was checked
for an `errors` key before anything was counted:

    jq -r 'if .errors then (.errors|length|tostring)+" errors" else "no errors key" end' batch.*.json
    no errors key
    no errors key
    no errors key
    no errors key

    74 boards, 74 with a default branch, of which 1 has no .github/workflows
    923 files, 923 blobs, 923 with text, 0 binary
     20 references to iderex/wache/.github/workflows/pr-hygiene.yml, all on uses: lines
      1 reference to iderex/wache/.github/workflows/unicode-guard.yml, on a uses: line

The board without the directory is `iderex/lagetisch`, as on 31 August. The one
`unicode-guard.yml` reference is this board's own `calling-examples.yml`, as on
31 August. Nineteen boards outside this one call the hygiene check, every one of
them through `shared-hygiene.yml`, and no board outside this one calls the
guard.

The file count is the one number that moved: 902 on 31 August against 923 now.
It is a count of every workflow file on the fleet and moves whenever any board
adds one, so it says nothing about this guard; it is recorded because a reader
comparing the two sections will see it move and should not have to wonder.

The copies and the pins, counted off the same trees:

    63 boards hold a local .github/workflows/unicode-guard.yml, in 34 distinct blob ids
    28 boards share the largest of the 34
    34 boards hold a local .github/workflows/pr-hygiene.yml
    19 of the 19 callers keep a local unicode-guard.yml
    17 of the 19 callers keep a local pr-hygiene.yml
    13 v1.0.0    @9b311243c2d0d0ced7feb957a20bc178acce6a5d
     6 v1.2.0    @113085b269d3437a3f96ff9e7060b64b0af88ab1

Every one of those is the figure the section above records. A week after the
ruling on `#28` that every board calls the guard by pinned commit and retires
its copy, no board has done either.

## The 63 copies by blob id, so the next reading can compare

No section above records those ids. Each of them counts the copies and the shape of
their distribution, and the distinct ids themselves appear on this page only as
a number:

    git grep -c -E '^    [A-Za-z0-9._/-]+ +[0-9a-f]{40}$' -- docs/unicode-guard-copies.md
    docs/unicode-guard-copies.md:63

with the 63 that command now finds being the table below and nothing earlier. So
a reading here could say the distribution had not changed shape and could not
say that no single copy had moved. Sorted by id, so the 28-board cluster and
the 3-board cluster group:

    iderex/entwurf                         0032f2700c1d3d9b556d3b0685b2f1dfced2bbe1
    iderex/plattenschrank                  16206c0862aae277f2e5db1a51c352c3e67b629b
    iderex/drehbank                        182539816a0a8d14f9b00b9878e53b6bf6d23ff6
    iderex/kartei                          19fae0337a761854e46db92e68cc19e7c9ffce13
    Flowfin/lab                            1cf5abdd189e83e32939b862e5d4cb62e868d294
    iderex/messlatte                       226d08794e9a63ad45ad5f486ff34f81c278b7eb
    iderex/schallweg                       254e4fe44af6a06ae71ef5f9d70cd9e731107496
    Flowfin/jellyfin-plugin-server-pairing 3a7415a4b8ddb8984147ba1ee91617a384e4004e
    iderex/scheinbild                      3a7d1627a54cae9cc3f4c03921d002870e933008
    iderex/reissbrett                      3fa24fdf4fbe468f8ab769ddf94cdfa4c307b020
    iderex/wache                           46db0f6c0fa9b9efd1ccdef705479fc0e5e54607
    iderex/swarm.asm                       48ddd71bfaad618a9e7c824a9f00225439ea8907
    iderex/kanzlei                         496aff96484b744979625dd373168948b97a5e0a
    iderex/relais                          496aff96484b744979625dd373168948b97a5e0a
    iderex/sternwarte                      496aff96484b744979625dd373168948b97a5e0a
    iderex/niedergang                      4b0fc0beded769b8157b44f44270af17dad4a514
    Flowfin/core                           516dcf73b207284686eaefc7e6e7a959a3545bf7
    iderex/stammtisch                      5381a941d1dab132f98d91f8c58cacfdafac0f77
    Flowfin/jellyfin-plugin-smart-collections 544e41e627edd03d4a06259b9670750236ce353d
    Flowfin/.github                        60083b70eb2762026be377a0baa2d540f33719f7
    Flowfin/jellyfin-plugin-discover       60083b70eb2762026be377a0baa2d540f33719f7
    Flowfin/jellyfin-plugin-invites        60083b70eb2762026be377a0baa2d540f33719f7
    Flowfin/jellyfin-plugin-requests       60083b70eb2762026be377a0baa2d540f33719f7
    Flowfin/jellyfin-plugin-share-links    60083b70eb2762026be377a0baa2d540f33719f7
    Flowfin/jellyfin-plugin-stats          60083b70eb2762026be377a0baa2d540f33719f7
    Flowfin/jellyfin-plugin-watch-sync     60083b70eb2762026be377a0baa2d540f33719f7
    Flowfin/site                           60083b70eb2762026be377a0baa2d540f33719f7
    iderex/attrappe                        60083b70eb2762026be377a0baa2d540f33719f7
    iderex/bremsweg                        60083b70eb2762026be377a0baa2d540f33719f7
    iderex/eichstelle                      60083b70eb2762026be377a0baa2d540f33719f7
    iderex/einschlag                       60083b70eb2762026be377a0baa2d540f33719f7
    iderex/findbuch                        60083b70eb2762026be377a0baa2d540f33719f7
    iderex/gegenprobe                      60083b70eb2762026be377a0baa2d540f33719f7
    iderex/gutachten                       60083b70eb2762026be377a0baa2d540f33719f7
    iderex/hallraum                        60083b70eb2762026be377a0baa2d540f33719f7
    iderex/indexwerk                       60083b70eb2762026be377a0baa2d540f33719f7
    iderex/lehrkanzel                      60083b70eb2762026be377a0baa2d540f33719f7
    iderex/linienbuch                      60083b70eb2762026be377a0baa2d540f33719f7
    iderex/linienschluessel                60083b70eb2762026be377a0baa2d540f33719f7
    iderex/messbuch                        60083b70eb2762026be377a0baa2d540f33719f7
    iderex/messstube                       60083b70eb2762026be377a0baa2d540f33719f7
    iderex/pruefstand                      60083b70eb2762026be377a0baa2d540f33719f7
    iderex/raumbuch                        60083b70eb2762026be377a0baa2d540f33719f7
    iderex/rechenblatt                     60083b70eb2762026be377a0baa2d540f33719f7
    iderex/rechenstrasse                   60083b70eb2762026be377a0baa2d540f33719f7
    iderex/spurenarchiv                    60083b70eb2762026be377a0baa2d540f33719f7
    iderex/stoffbuch                       60083b70eb2762026be377a0baa2d540f33719f7
    iderex/nachtwache                      696426bbf229e91b4113514f3689ae9bb7839fc4
    iderex/lichttisch                      77259cf95330515e783659e8e4cb50c27debbf3b
    iderex/ausgleich                       785eece4d13f4f305f194ea766f287b8ad4ed844
    iderex/hoersaal                        908af47c3fd9a812e045202d58bc666eb6c294a5
    Flowfin/hub                            91f87e50fe31610cd0efce9233ae5f468aaffbc8
    Flowfin/jellyfin-plugin-metadata-sync  92bdb6afd936a573363c3453f47821563950dfbe
    iderex/Easy-Compliance-Manager         99b027f7471ea43f54455c9429cd8fe56064fe3c
    Flowfin/jellyfin-plugin-sso            9a1bb93cfc976c3c535d20eca5898deb1537608d
    iderex/kontor                          a176522056cf7110b0c1c29ed579cef63226f4ff
    iderex/blende                          a6744232a7a929bcb8c2601b524e2e588f5b1e5e
    iderex/lesesaal                        a6e4d388c7f6776a4e4c001796b3b18eda7dabf0
    iderex/beiblatt                        b2beba9e6dca30075fd323d31353f3be8f6e651a
    iderex/schichtwerk                     b4fb7b6a8fa2f7396a082bffa7ed29476fe250b7
    iderex/retusche                        e8782443f6702587bf8c487b04a0401481170073
    Flowfin/jellyfin-plugin-whisper-subtitles f671a83da2a5b1de3fa915c329fd2049b4ea7255
    Flowfin/jellyfin-plugin-watchlist      fced059bedfdbd6c9119f916afa9194c97826e39

This board's own canonical file is one of the 63 and shares its id with nothing:

    git ls-tree 0486b33 .github/workflows/unicode-guard.yml
    100644 blob 46db0f6c0fa9b9efd1ccdef705479fc0e5e54607	.github/workflows/unicode-guard.yml

So 62 copies stand on other boards, in 33 distinct contents.

THE REF IS THE COMMIT THIS SECTION LANDED ON AND NOT `origin/main`, which is
what it named until this line was repaired. The reading is of 4 September and
this board does not hold the file any more, so the command against a moving
reference prints nothing at all and exits 0 - a silence a reader cannot tell
from a board that never carried a copy:

    git ls-tree origin/main .github/workflows/unicode-guard.yml
    echo "exit=$?"
    exit=0

The entry point was removed the next day, in the change `### What the removal
takes and what it leaves` below records:

    git log --format='%h %ad %s' --date=short --diff-filter=D \
      -- .github/workflows/unicode-guard.yml
    f1a09c6 2026-09-05 Retire the shared unicode guard and keep the check local [#28]

WHERE THIS READING STOPS. The subject is still the full
`iderex/wache/.github/workflows/<file>.yml` form, so a call written any other
way is outside it. The ids above are this reading's own and are compared against
no earlier list, because no earlier list exists; the first sentence a later
reading can write about movement is one written against this table. And that a
copy's id is unchanged says its bytes are unchanged, never that its board
considered the migration and declined - why no board has taken the guard is
decided on those boards and written nowhere this page can read.

## The retirement on 5 September, and where the 63 copies stand

`#28` was decided on 4 September 2026 in the direction this page had not been
written for: the shared guard is retired rather than given callers. Its
retirement done-when asks that the workflow, its calling example and its row in
the README table go together, and that the removal say where the 63 copies stand
instead. This section is that statement, taken before the removal rather than
justified after it.

Read 5 September 2026 against the roster at `iderex/operations` `origin/main`
`e1903807dc380addc3de69d3c4996bf5e7b89a77`, by the tree route the 4 September
section sets out - four GraphQL calls, nineteen boards to a query, every reply
checked for an `errors` key before anything was counted:

    jq -r 'if .errors then (.errors|length|tostring)+" errors" else "no errors key" end' batch.*.json
    no errors key
    no errors key
    no errors key
    no errors key

    74 boards, 74 with a default branch, of which 1 has no .github/workflows
    924 files, 924 blobs, 924 with text, 0 binary
     20 references to iderex/wache/.github/workflows/pr-hygiene.yml, all on uses: lines
      1 reference to iderex/wache/.github/workflows/unicode-guard.yml, on a uses: line

    63 boards hold a local .github/workflows/unicode-guard.yml, in 34 distinct blob ids
    28 boards share the largest of the 34
    34 boards hold a local .github/workflows/pr-hygiene.yml
    19 of the 19 callers keep a local unicode-guard.yml
    17 of the 19 callers keep a local pr-hygiene.yml
    13 v1.0.0    @9b311243c2d0d0ced7feb957a20bc178acce6a5d
     6 v1.2.0    @113085b269d3437a3f96ff9e7060b64b0af88ab1

The board without the directory is `iderex/lagetisch`, as on 31 August and 4
September. The one `unicode-guard.yml` reference is this board's own
`calling-examples.yml`, where it is fixture text rather than a call. So on the
day the shared guard is deleted, no board outside this one calls it, exactly as
on the day it was written.

The file count is again the only number that moved, 923 to 924. It counts every
workflow file on the fleet and says nothing about this guard.

### The first reading that can say a copy did not move

Every reading before 4 September closed by admitting it could not tell a
standing copy from a changed one, because it kept counts and no ids. The table
below `## The 63 copies by blob id` closed that going forward, and this is the
first reading on the far side of it. The 63 pairs on that page, against the 63
this sweep returned:

    diff <(sort table-0904.tsv) <(sort uguard.tsv) && echo IDENTICAL
    IDENTICAL

Same 63 boards, same 63 blob ids. Not one copy moved in the day between the two
readings, which is a much smaller claim than the shape comparisons that preceded
it and is the first one on this page that is actually about movement.

### What the removal takes and what it leaves

WHAT GOES IS THE ENTRY POINT AND NOT THE CHECK. `.github/workflows/unicode-guard.yml`
declared `on: workflow_call:` and nothing else, so the file was reachable two
ways: by another board naming it in a `uses:` line, and by this board's
`guards.yml` naming it in the local path form. The first has no users. The
second is why the job moves into `guards.yml` rather than being deleted with the
file. The job is byte-identical there - same `name: Reject Trojan Source
Unicode`, same pattern, same twenty fixtures:

    git show 2eef8773:.github/workflows/unicode-guard.yml | sed -n '46,154p' | sha256sum
    7389df9e58a48dbf32db5f4a760e51eaafc0bdd3707bc35bba1e02f2436037d6 *-
    sed -n '40,$p' .github/workflows/guards.yml | sha256sum
    7389df9e58a48dbf32db5f4a760e51eaafc0bdd3707bc35bba1e02f2436037d6 *-

THE LEFT REF IS `f1a09c6^` AND NOT `origin/main`, and this pair is the one place
on the page where the two cannot be the same commit. Every other section here is
addressed at the tree it was read against; this equality is between a file the
change DELETED and the file it moved the job into, so the change that wrote the
pair is the change that took its left-hand side away. It named `origin/main`,
and the pipeline as it was pasted does not stop there: `git show` fails, `sha256sum`
reads the empty stream it is handed, and a hash that belongs to no file is printed
under the equality:

    git show origin/main:.github/workflows/unicode-guard.yml | sed -n '46,154p' | sha256sum
    fatal: path '.github/workflows/unicode-guard.yml' does not exist in 'origin/main'
    e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855 *-

`e3b0c442...` is the hash of no bytes at all. So a reader who kept stdout and let
the fatal go to stderr saw the two sides disagree, with nothing on the page saying
the left one had ceased to exist.

`2eef8773` is that change's parent, which is reachable from `main` and is the
last commit holding the file:

    git merge-base --is-ancestor 2eef8773 origin/main && echo reachable
    reachable

The right-hand side stays at the working tree, because `guards.yml` is where the
job runs from now and today's bytes are what a reader wants on that side.
Neither value moved: the hash is the one this section printed when it landed.

So this board is not a board that stopped scanning its own tree. It is a board
that stopped publishing, which is the thing `#28` measured as costing
maintenance and returning nothing.

NO RULESET LOSES A REQUIRED CHECK, and that is read rather than assumed, because
the neighbouring migration on `#25` has exactly this trap on eleven boards. This
board's own gate requires no status check at all:

    gh api repos/iderex/wache/rulesets --jq '.[] | "\(.id)\t\(.name)\t\(.enforcement)"'
    21215911	gate	active
    gh api repos/iderex/wache/rulesets/21215911 --jq '[.rules[].type]'
    ["deletion","non_fast_forward","pull_request","required_signatures"]

and no other board's gate can be affected, because a required check is the name
of a check run that some workflow produces, and no board outside this one runs
this file at all - which is the `1 reference` line above.

THE TAGS ARE NOT REWRITTEN, so the file stays reachable at every release that
carried it:

    for t in v1.0.0 v1.1.0 v1.2.0 v1.3.0; do
      printf '%s\t' "$t"
      git ls-tree "$t" -- .github/workflows/unicode-guard.yml | grep -q . &&
        echo "holds unicode-guard.yml" || echo "no unicode-guard.yml"
    done
    v1.0.0	no unicode-guard.yml
    v1.1.0	holds unicode-guard.yml
    v1.2.0	holds unicode-guard.yml
    v1.3.0	holds unicode-guard.yml

A board pinned at `v1.1.0`, `v1.2.0` or `v1.3.0` therefore keeps running what it
ran. What ends is the tag after `v1.3.0`, which will not carry the file, so a
pin bumped past it would reach `startup_failure` with no job and no check run.
No board is in that position today and the count that says so is the same `1
reference` line.

WHAT THE 63 BOARDS ARE ASKED TO DO IS NOTHING. The direction this page was
written under - each board calls the shared guard by pinned commit and deletes
its copy - is withdrawn by the decision. `## What a first caller has to carry`
above is the cost of a migration that is no longer asked for, and it is left
standing because it is also the reading of what the copies differ in, which the
retirement does not change: 33 of the 34 distinct contents refuse the identical
string, and `iderex/lesesaal` holds the one that does not.

### What this section does not evaluate

Whether `iderex/lesesaal`'s divergent copy still refuses less than the other 33.
Its blob id is unchanged since 4 September and its content was read on 27 August;
that its bytes have not moved is not the same statement as its behaviour having
been re-read, and it has not been.

Whether any of the 63 boards ought to keep a Trojan Source check at all is a
question on those boards. This page reads what they hold and this repository
writes into none of them.
