---
name: coordinate-issues
description: Act as coordinator for a batch of GitHub issues in the current repo. Triage them, run each ticket in its own fresh session in dependency-ordered waves, check each PR, and sequence merges. Use when the user says /coordinate-issues, or asks to work through the issues, crank through the backlog, get all the tickets done, fan out across issues, or run several tickets at once with you coordinating, even if they don't name the skill. Not for one ticket that can be done in this session.
---

# /coordinate-issues

Be the coordinator. You plan, start sessions, check their PRs and decide what happens next.
The sessions write the code.

Arguments are optional issue numbers (`/coordinate-issues 58 61 62`). With none, work from all
open issues.

**Why this shape.** Each ticket gets a fresh session because a well-written issue is already a
design doc, so a short brief is enough, and because your context stays small: you only get a short
note when a session stops. Don't use forks, which copy this whole conversation, or `/loop`, which
repeats one task on a timer.

Keep your own context small throughout. Read issue summaries, changed-file lists and check
results, not full diffs. Hand a session the issue number rather than retyping the issue.

**Don't stop to ask.** These are settled for every run, unless the repo's own instructions
(CLAUDE.md, CONTRIBUTING) or the user say otherwise:

- **Merging:** you merge every PR this run's sessions open, once its required checks pass. This
  overrides any shipping skill's default that a human merges. If branch protection requires a
  human review, don't work around it: leave the PR green and list it in the final report.
- **PRs that were already open:** leave them alone. Don't merge, close or rebase them, even when
  they look ready. List them in the final report.
- **Issues already done or outdated:** if merged work already covers an issue, or it no longer
  applies, comment with the PR that handled it and close it.
- **Issues that can't be taken on yet:** skip them. Examples are one waiting on a product decision
  the issue doesn't settle, a secret only the user can rotate, or a paid account. You may comment
  on the issue saying what blocks it. List these issues in the final report.
- **Finished sessions:** archive a ticket's session with
  `mcp__ccd_session_mgmt__archive_session` as soon as its PR is merged and its issue is closed.
  Don't wait for the end of the run, and don't ask. Leave a session up only while it still has
  work: an open PR, CI to fix, or a blocker the user needs to see.

## 1. Survey

Learn the repo first. You need these facts for every later step, so write them down once in a
scratchpad file:

- **Repo and default branch:** `gh repo view --json nameWithOwner,defaultBranchRef`. Below,
  `<owner>/<repo>` and `<default>` mean these.
- **Baseline commands:** how this repo typechecks, lints (check only, no auto-fix), tests and
  builds. Find them in CLAUDE.md, the README, the package manifest or Makefile, and the CI
  workflows. Use what CI runs when they differ.
- **Required checks:** which CI checks gate a merge. Read branch protection
  (`gh api repos/<owner>/<repo>/branches/<default>/protection`) or, if there is none, the checks
  on recently merged PRs. Note any check that is advisory or known to be broken, so you don't
  wait on it.
- **Merge method:** what recent merged PRs used (merge commit, squash or rebase) and what the repo
  allows. Use the same.
- **What a merge triggers:** whether landing on `<default>` deploys or releases anything, and how
  soon.
- **Environment quirks:** what a fresh session on this machine has to know to work: toolchain
  activation, dependency install per worktree, git identity, missing display, missing API keys or
  services, how to run the app headless. Take these from memory and the repo docs, then check each
  against the current machine by running it.

Then the issues:

- List the issues with their bodies. Save them to a scratchpad file and print only a summary:
  title, labels, size, and which `#N` each one references.
- List open PRs. An issue may depend on unmerged work, and a stacked PR must land bottom-first.
- Check each issue against what merged since it was written: `git log` and `git diff --stat` from
  the issue's creation date, and the code it names. Close what's done or outdated, with a comment
  naming the PR.
- Mark the issues that can't be taken on yet and skip them. If one has a code part that stands on
  its own, you can still run that part and leave the issue open.
- Run the baseline on `<default>`. If any of it fails, no session's CI can go green, so fixing it
  is wave 0.

## 2. Plan the waves

Order by two things:

- **Dependencies:** what each issue needs merged first. Infrastructure goes early: the baseline,
  then CI, then shared plumbing that several tickets build on. Later PRs then get checked by CI
  and build on the plumbing.
- **File overlap:** sessions in the same wave shouldn't edit the same files. Find this repo's hot
  files before planning: the dependency manifest and lockfile, CI config, entry points, shared
  type or schema definitions, route or command registries, and whatever `git log --name-only`
  shows changing in most recent PRs.

When a hot file can't be avoided, pair tickets that touch different parts of it. Whichever PR
merges second absorbs the conflict. To save time, a dependent ticket can start early if most of it
is new files. Tell it to build the standalone parts first and merge `<default>` before wiring in.

Show the user the waves, along with what you closed and what you're skipping, and start wave 1 in
the same turn. Don't wait for a go-ahead. If a merge deploys or releases, say so in that message,
so the user knows what ships and when.

Where a choice comes up, take the issue's own recommendation, or the safer option if it has
none, and list every judgment call at the end.

## 3. Start a wave

Sessions are cut from a local branch (`source_branch`), so keep one that tracks the remote default
branch. If you are in a worktree, local `<default>` is usually held by the root checkout and can't
be updated from here, so use your own branch. Before each wave:

```bash
git fetch origin && git merge --ff-only origin/<default>
```

Skip this and sessions start from stale code.

Start each session with `mcp__ccd_session__start_session`, using `initiation: "user_asked"`,
`context: "fresh"`, `use_worktree: true`, `source_branch` set to your branch, a short `name` and
`branch_name`, and `wake_when_done: true`. When starting two or more, call `mark_chapter` first.

Tickets that belong together can share a session when they edit the same config, for example two
packaging issues. One session owns the CI config at a time.

### Brief template

The `prompt`:

```
Implement GitHub issue <owner>/<repo>#N, "<title>". Read it in full with `gh issue view N`,
follow its design and acceptance criteria, and take the recommendation on each open question.

Changes since the issue was written: <what merged, where code moved, what to reuse>.

In parallel: <other sessions and their files>. Stay out of <files>. If <PR> merges first, merge
origin/<default> and resolve the conflicts, keeping both behaviors.

After you: <what builds on this, so the API stays easy to extend>.

Verification:
- <the baseline commands> pass.
- <how to see the change working here: run the app or service the way this machine allows, and
  capture evidence such as screenshots or output. Clean up anything you start.>
- <what can't be exercised for real here, and the stand-in to use: mocks, fixtures, a local
  database.>
- Open a PR against `<default>` that says "Closes #N". Wait for CI with
  `gh run watch <id> --exit-status`, in the background if needed, and fix failures until
  <the required checks> pass.
- End your turn only when CI is green or you are truly blocked. Do not merge.

Report the PR URL, the CI run URL, and what you could not verify.
```

Fill every `<...>` from the survey. A session knows nothing you don't tell it.

The `background` needs: what the project is, that a coordinator runs the waves, what's on
`<default>` now, and the environment quirks from the survey, as exact commands. Typical ones:

- How to activate the toolchain, if the shell doesn't do it on its own.
- The dependency install to run first, because installed dependencies are per worktree.
- How to commit when git has no identity configured: pass it with `git -c`, don't change git
  config.
- Where code style is defined (CLAUDE.md, formatter and linter config).

## 4. When a session reports

A note is that session's report, not proof. Short notes like "waiting on CI" mean it isn't done.

Check the checks, mergeability and changed files yourself. Go by the required checks. Ignore
advisory ones you noted in the survey as broken.

- **CI still running, session watching it:** end your turn. It will report.
- **Session stopped before CI finished:** message it to wait for the run and fix failures.
- **Green:** go to step 5.
- **Blocked or failing:** help it with `SendMessage` to its `local_…` id. It already has the
  context, so don't start a new session. Answer what you can. For a product decision, give it the
  issue's recommendation, or the safer option. If that can't unblock it, stop the ticket, leave
  any PR open, and report it as skipped.

Then start whatever just became unblocked.

## 5. Merge

Merge only this run's PRs, never ones that were open before it started. Merge one PR at a time
with `gh pr merge N`, using the repo's merge method. `mergeable` reads `UNKNOWN` for a few seconds
after a merge, so retry before deciding. Then check every other open PR:

- **`CONFLICTING`:** message its session with what merged and which files changed. It merges
  `<default>` in, reruns everything and watches CI.
- **`MERGEABLE`, but CI ran before that merge:**
  - If the merged work touched the dependency manifest, the lockfile or CI config, or either PR
    changes the toolchain, send it back to merge `<default>` and rerun CI. A clean merge doesn't
    prove the two work together.
  - Otherwise merge it, then run the baseline on `<default>` yourself.

For a stacked PR, merge the bottom one, retarget the next to `<default>`, and have its session
merge `<default>` in so it gets real CI.

After merging, confirm the issue actually closed. Tickets with a later phase stay open on purpose.
Then archive that ticket's session right away.

## 6. Keep the user posted and finish

Each update says what merged, what's running, what's next and what needs them. Link sessions and
PRs. If the user asks for status, check live rather than repeating your last note: after a long
stretch of automated wake-ups, notes can sit until your next turn.

When the queue is empty:

1. Run the full baseline on `<default>`. If merges trigger a deploy or release, check its latest
   run.
2. Report:
   - **Merged:** what merged and closed.
   - **Closed as done or outdated:** each issue, with the PR that covered it.
   - **Skipped:** each issue you skipped, and what blocks it.
   - **Left open:** PRs that were already open when the run started, and any of this run's PRs
     waiting on a human review.
   - **Judgment calls:** every one you made.
   - **Not verified:** be specific, for example no display, no live API key, no production data,
     no target hardware. A headless screenshot is not hands-on use.
3. Check with `mcp__ccd_session_mgmt__list_sessions` (`linked: true`) that every finished session
   is archived, and archive any that aren't. The only sessions still up should be the ones the
   report names as still having work.

Never push a session's branch for it, and never merge a PR whose CI you haven't seen pass.
