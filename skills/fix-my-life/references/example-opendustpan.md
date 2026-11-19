# Worked example: OpenDustpan/OpenDustpan

This file shows the report that the skill gives for one real repository. It is a record of
19 November 2026. The numbers go stale: run the commands again before you use them for a
decision.

## How the steps went, and what went wrong first

- **Step 1.** The session clone had 50 commits. `git fetch --deepen=3000 origin main` gave
  2,310 commits in 41 days. With the shallow clone, no pattern below was visible. Two
  ranges were read: the last two weeks (825 commits) for the findings, and the full
  history for what the project repaired.
- **Step 2.** `scripts/churn.py --ref origin/main --since "2 weeks ago"`, then the same
  without `--since`. The repository lists its generated files in `.gitattributes`, and the
  script reads that list. From GitHub: the last 100 runs of `ci.yml` and the open pull
  requests.
- **Step 3 and 4, first draft.** The first draft ranked by commit count. The two
  `queries.sql` files had the most commits (224 and 211), so finding 1 proposed to split
  them into one file for each area. **That was wrong two times.** The conflict simulation
  gives these files a share of 8%: they are busy, not a conflict point. And the project
  had measured the same question two weeks before (RFC 120, ADR 326), chose a placement
  rule, and refused the split until new data shows the need. The draft had searched the
  decision records for "generated" and "conflict", but not for the file name.
- **What changed.** The ranking below uses the simulation share. Each finding names the
  decision that covers it. The `queries.sql` finding became the count that ADR 326
  decision 4 asks for.

---

# Toil report: OpenDustpan/OpenDustpan

Read: 825 commits on `origin/main`, 5 to 19 November 2026 (15 active days, median 32
commits a day, 196 on the busiest day), and the full history of 2,310 commits for the
"Resolved" section. CI: the last 100 completed runs of `ci.yml`. Pull requests: the 62
that were open.
Not read: review comments, CI logs, the `OpenDustpan/console` repository.
Limits: the history is linear (0 merge commits), so each conflict number is from the
simulation of `churn.py` and is an estimate.

## Summary

The repository takes work from many coding agents in parallel. Three costs lead. 30% of
the commits change only plans. A small set of lines that each feature must edit (one sum
in a test, the last row of an index table, one `.PHONY` line) makes parallel changes
conflict. And one third of the CI runs give no merge. The project knows the pattern: it
repaired four such points in six weeks, each time with a decision record. Do finding 2
first: it is one expression in one test file, and no decision covers it.

## Findings, most expensive first

### 1. Open plans name details of a main branch that moves

- **Pattern:** the plan that goes stale.
- **Evidence:** 354 of 825 commits (43%) touch `openspec/changes`, and 247 (30%) touch
  nothing else. Subjects: "rewrite nightly-load-soak-fault-job against main" (#3636),
  "open changes name the moved caller routes" (#3357), "hourly spec fixes 4" to "11",
  "passes the reground check". The `tasks.md` of one change has a simulation share of 93%
  (14 of 15). `openspec/changes` holds 236 open changes and 41 archived ones. 6 of the 62
  open pull requests rewrite a change "against main".
- **Cost:** 247 pull requests in two weeks, each with a CI run. Some of them add a new
  plan, which is useful work. This report did not count the split.
- **Cause:** tasks in a change name file paths, route paths and error codes. `main` moves
  32 times a day, so an open change goes stale in days. 236 open changes is more than the
  agents start before the changes go stale.
- **Status in the project:** partly repaired. A "reground" check finds a stale change at
  the time of use. No decision limits the number of open changes.
- **Repair (rank 1, remove the need):** keep a proposal (the intent and the decisions)
  for work that is not scheduled, and write `tasks.md` with its paths when an agent starts
  the change. The reground check then runs one time, on a new plan.
- **First step:** count the open changes that no pull request implements, and stop the
  periodic correction runs for those. This needs a decision of the owner, so write it as
  an RFC.
- **Shows that it worked:** the "only" count of `openspec/changes` goes below 15% of the
  commits.

### 2. Each feature edits one sum in the round-trip test

- **Pattern:** the shared total.
- **Evidence:** `internal/servertest/roundtrips_test.go`: 39 commits, simulation share 72%
  (26 of 36), the highest of all code files. `git show -U0` shows the line:
  `droppedRunnerCredentialTrips + proxyRuleSetTrips + trustBundleTrips,` gets one more
  term with each feature that adds a table or a read (#3512, #3498).
- **Cost:** about 3 changes a day, and 7 of 10 could not be open at the same time as the
  3 changes before them.
- **Cause:** the test holds the exact number of database round trips of each flow as one
  expression on one line.
- **Status in the project:** not decided. A search of `docs/adr` and `docs/rfc` for the
  file name found no record.
- **Repair (rank 3, change the shape):** write the terms as a list with one named term on
  each line, in name order, and add them in a loop. Two changes that add a term then
  merge. Rank 2 is possible later: each area declares its trips next to its store code.
- **First step:** the one-term-on-each-line form. It changes no behavior.
- **Shows that it worked:** the simulation share of the file goes below 15%.

### 3. Decision records share an index table, a counter and one long line

- **Pattern:** the conflict magnet, and the committed copy.
- **Evidence:** `docs/rfc/README.md` has a simulation share of 68% (25 of 37, full
  history): each new RFC adds the last row of one table, and each accepted RFC edits its
  row. `docs/adr/INDEX.md` (36 commits) and `.repowise/decisions.yaml` (39 commits) are
  generated and committed, and change together (share 0.85). `decisions.yaml` has one line
  of 41,411 bytes. ADR numbers collide: "renumber ADR 312 to 314 as ADR 313 to 315".
- **Cost:** 58 ADRs in two weeks. Each one edits both generated files, and can take a
  number that a parallel pull request also took.
- **Cause:** a sequence number, a hand-written table that grows at the end, and two
  generated files in Git.
- **Status in the project:** ADR 383 decided the same problem for the TypeScript client:
  a generated file must have no line that each change edits. It does not name these files.
- **Repair (rank 2, derive):** generate the RFC table from the RFC files, as
  `make adr-index` does for ADRs. Write `decisions.yaml` with one short line for each
  field. Give an ADR its number when it merges, and use a slug before that time.
- **First step:** change the writer of `decisions.yaml` to many lines.
- **Shows that it worked:** `churn.py` prints no line above 10,000 bytes, and the share of
  `docs/rfc/README.md` goes below 15%.

### 4. One `.PHONY` line and one `check:` line in the Makefile

- **Pattern:** the conflict magnet.
- **Evidence:** `Makefile`: 29 commits, simulation share 58% (15 of 26). The `.PHONY`
  line is 655 bytes and names all 67 targets. `.github/workflows/ci.yml` has 43% (10 of
  23) and grew to 846 lines.
- **Cost:** about 2 changes a day to the Makefile, more than half in conflict with the
  ones before.
- **Cause:** each new target edits the same two lines.
- **Status in the project:** not decided.
- **Repair (rank 3):** write `.PHONY: <target>` on the line above each target, and one
  prerequisite of `check` on each line.
- **First step:** the `.PHONY` change. It is mechanical.
- **Shows that it worked:** the share of `Makefile` goes below 15%.

### 5. Central lists that each feature edits

- **Pattern:** the shared list.
- **Evidence and status, file by file:**
  - `pkg/store/*/queries.sql` (46 commits each): **decided, follow-up open.** ADR 326
    decision 1 put a placement rule in the agent instructions. Decision 4 says to count
    the conflicts again after one week and to split only if the count shows the need. This
    report found no record of that count. The simulation gives 3 of 37 (8%) for each
    dialect. The rule works, and the data does not show a need for the split.
  - `pkg/dustpan/dustpan.go` (38 commits, share 20%): **decided, follow-up open.** ADR 326
    decision 5 moves each area to its own file. The file still has 4,026 lines.
  - `e2e/coverage.go`: **decided and done.** ADR 326 decision 2 split it into 25 area
    files. It had 142 commits in the full history and 20 in the newer half.
  - `pkg/backup/portable_tables.go` (share 29%), `pkg/store/sqlite/backup_columns.go`, and
    the `domain` rule of depguard in `.golangci.yml` with 72 package paths: **not
    decided.**
- **Cost:** a feature with a table and a route edits 5 to 7 of these files.
- **Repair for the lists with no decision (rank 2, derive):** a test already compares the
  backup column lists with the live schema. A list that a test can check against a source
  can come from that source: generate the column lists from the baseline migration, and
  keep by hand only the choice that a person makes (which tables are portable). For
  depguard, a glob for `**/pkg/**` with a short list of exceptions could replace 72
  entries. The comments in `.golangci.yml` say that a broad glob matched wrong paths
  before, so test this first.
- **First step:** record the `queries.sql` count in the place that ADR 326 names, so that
  decision 4 is closed.
- **Shows that it worked:** the share of `portable_tables.go` goes below 15%, and
  `.golangci.yml` leaves the first 25 configuration files by commits.

### 6. One third of the CI runs give no merge

- **Pattern:** more open work than the pipeline can merge.
- **Evidence:** 62 open pull requests, 57 of them opened in a window of 3.5 hours on
  19 November. 100 CI runs completed in 2 hours 44 minutes: 17 failed and CI canceled 16.
  A successful run takes 13 minutes (median) and 21.6 (p90).
- **Cost:** one third of the CI minutes. This report did not read the logs, so the split
  between real defects and stale branches is not known.
- **Cause (could be):** findings 1 to 5. The number of pairs of branches that can conflict
  grows with the square of the open branches that touch the same lines.
- **Status in the project:** partly repaired: a merge queue, and path filters that skip
  the Go jobs for changes of documents only (#3211, #3213).
- **Repair:** do findings 2, 3 and 4 first. Then measure again.
- **First step:** for one day, record for each failed run if the cause was the change or
  the base.
- **Shows that it worked:** the share of failed and canceled runs goes below 15%.

### 7. Six lint rules have more than 100 suppressions each

- **Pattern:** the suppressed check.
- **Evidence:** `//nolint:gosec` 291, `errcheck` 275, `gochecknoglobals` 169,
  `tagliatelle` 165, `exhaustruct` 127, `ireturn` 120, `paralleltest` 108. A CI job
  (`nolint-check`, #2725) checks each new suppression.
- **Cost:** about 1,250 lines, each with a reason that a reviewer must read.
- **Cause (could be):** `tagliatelle` checks JSON names, and the tool-compatible APIs must
  keep the names that the tools expect. `ireturn` flags constructors that return a port.
  This report did not read the suppressions one by one.
- **Status in the project:** not decided.
- **Repair (rank 1):** exclude the wire packages from `tagliatelle` in `.golangci.yml`,
  and give `ireturn` an allowlist of the port interfaces. Sample 20 suppressions of
  `gosec` and `errcheck` before you propose something for them.
- **First step:** `tagliatelle`.
- **Shows that it worked:** the count of each marker in `churn.py`.

### 8. Tests that depend on the load of the machine

- **Pattern:** the check that depends on the machine.
- **Evidence:** 42 subjects in the full history (2%) name a race, a timeout, a slow store
  or a deadlock: "Make the store conformance cases hold on a slow store" (#1486), "Fix two
  load-sensitive test flakes" (#1517), "order the deadlock timers" (#3309), and an open
  pull request "Make the postgres swap deadlock test independent of machine load" (#3595).
- **Cost:** not measured. A run that fails for this cause costs 13 to 22 minutes.
- **Cause:** tests of locks and leases use real time. Test code has only 2 `time.Sleep`
  calls, so the cause is deadlines and budgets, not sleeps.
- **Status in the project:** repaired case by case. A test clock exists (#3602).
- **Repair (rank 3):** make the test clock the only way to get a deadline in the store
  conformance suites.
- **First step:** list the tests that failed and then passed with no code change in the
  last week of CI runs.
- **Shows that it worked:** that list gets shorter.

## Resolved (seen in the history, repaired since)

- Numbered migrations: the history added and later removed 400 migration files, with
  subjects like "number the lock record migration 0163 again". ADR 322 replaced them with
  one baseline for each dialect.
- The committed TypeScript client: `description.ts` held the API description on one line
  and changed with `openapi.json` in 53 commits. ADR 383 and commit `49739d4d` took the
  client out of Git.
- `e2e/coverage.go`: split by area (ADR 326 decision 2). See finding 5.
- CI hosting changed four times in October (own machines, Blacksmith, own machines "again,
  for now", GitHub-hosted on 6 November). No flip since.

## Not toil

- `pkg/store/*/queries.sql`: the most commits of all hand-written files, and a simulation
  share of 8%. Busy, not a conflict point.
- `GLOSSARY.md`: 138 commits in the full history, share 19%. Each new term adds a section
  in name order.
- `AGENTS.md` grew from 155 to 436 lines in six weeks, but from 402 to 436 in the last
  two. The growth slowed. Look again if it passes 500 lines.
- `e2e/e2e_test.go` and `e2e/harness.go`: 49 and 36 commits on 11 October. That was the
  first build of the suite.
- One revert in 2,310 commits.

## Order of work

1. Finding 2 (the sum on one line): one mechanical change, the highest share of a code
   file, no decision needed.
2. Finding 4 (`.PHONY`) and the first step of finding 3 (`decisions.yaml`): two more
   mechanical changes.
3. Finding 5, first step: record the `queries.sql` count and close ADR 326 decision 4.
4. Finding 1: needs a decision of the owner. Write the RFC while 1 to 3 merge. It has the
   largest cost, but its repair changes how the agents work.
5. Measure findings 6 and 8 again after the others. Their numbers could fall without a
   direct repair.
6. Finding 7 at any time. It conflicts with nothing.
