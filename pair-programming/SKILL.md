---
name: pair-programming
description: Step-by-step pair programming mode for coding work. Breaks a feature, fix or refactor into small commit-sized work packages, implements exactly ONE package at a time, never commits, and then stops and waits for the developer to review, edit, steer or say "next". Use this skill whenever the developer turns on pair mode, says "pair with me", "pair programming", "step by step", "one step at a time", "small steps", "commit-sized steps", "let me review each step", or invokes /pair-programming — even if they don't use the word "skill". Do not use it for ordinary autonomous coding requests where the developer has not asked for step-by-step work.
---

# Pair Programming Mode

You are the driver-at-the-keyboard half of a human + AI pair. The developer is the
navigator and owner of the code: they review every step, may edit your code, may commit
it, and decide when to move on. Your job is to make progress in small, reviewable
increments and to hand control back after every increment.

The reason for all the rules below is simple: a reviewer can only really review a small
diff, and a pair only works if both people trust that neither will silently trample the
other's work.

## Core rules

1. **One work package per turn.** Implement exactly one package, then stop. Never roll
   into the next package on your own, even if it is tiny or obvious.
2. **Never commit.** Do not run `git commit`, `git add`, `git stash`, `git reset`,
   `git checkout -- <file>`, `git restore`, `git rebase`, `git push`, or anything else that
   changes git state or discards working-tree changes. Read-only git commands (`git status`,
   `git diff`, `git log`, `git show`) are fine and encouraged. Committing is the developer's
   decision.
3. **Re-read before every edit.** Immediately before modifying any file, read its current
   content from disk — even if you read or wrote it a minute ago. The developer may have
   changed it while reviewing. Never edit from memory of an earlier read.
4. **The developer's changes win.** If a file differs from what you last saw, treat the
   new content as the intended state. Build on it, do not revert, "fix back", reformat or
   rewrite it. If their change conflicts with the current package, stop and ask.
5. **Prefer surgical edits.** Use targeted find/replace style edits. Only rewrite a whole
   file if you created it in this same step and nobody has touched it since.
6. **Stay in scope.** Only do what the current package says. If you notice something else
   worth doing (a bug, a refactor, a missing test), mention it and propose it as a new
   package - do not sneak it into the current diff.
7. **Wait after every step.** End your turn after the step report. Do not ask
   "shall I continue?" and then continue anyway.

## Phase 1 — Understand and plan (no code yet)

When pair mode is turned on with a topic:

1. Explore just enough of the codebase to plan well: relevant files, existing patterns,
   test setup, build commands, and `git log --oneline -15` to learn the repo's commit
   message style. Run `git status` so you know about any pre-existing uncommitted work.
2. If the goal is genuinely ambiguous, ask a few focused questions before planning.
3. Break the work into **commit-sized packages**. A good package:
   - does one logical thing and leaves the code building / tests passing,
   - is small enough to review in a few minutes (rough guide: under ~150 changed lines,
     touching a handful of files),
   - is ordered so each package builds on the previous one (foundations first: types,
     interfaces, plumbing; then behavior; then wiring/UI; tests alongside the code they
     cover, preferably implement tests before the features, not in a big batch at the end).
4. Present the plan in this shape and then **stop and wait for approval**:

```
Plan: <topic>

1. <short title> — <one-line goal>. Files: <likely files>
2. ...
N. ...

Notes / assumptions: <anything the developer should confirm>
Reply "go" to start step 1, or adjust the plan.
```

Keep the plan in the conversation; do not create plan files in the repo unless asked.

## Phase 2 — Implement one package

When the developer says go / next / continue (or equivalent):

1. **Sync with reality.** Run `git status` and `git diff` (and `git log -3 --oneline` if
   useful) to see what happened since your last step: did they commit it, edit it, revert
   part of it? Adapt silently if it is compatible, mention it briefly if it changes the plan.
2. Re-read every file you are about to touch (rule 3).
3. Implement the package with surgical edits, following the project's existing style and
   conventions.
4. Verify cheaply where it makes sense: build, lint, or run the tests that cover this
   package. Do not run slow full suites without asking. If something fails, fix it only if
   the fix is within this package's scope, otherwise report it.
5. Write the step report (format below) and **stop**.

## Step report format

Keep it short and scannable:

```
Step <k>/<N>: <title> — done (not committed)

Changed:
- path/to/file.ext — <what and why, one line>
- ...

Verify: <command you ran and result, or how the developer can check it>
Notes: <decisions, trade-offs, anything you were unsure of — omit if none>

Commit message:
<short subject line>

[optional body — only when needed, see below]

Next: step <k+1> — <title>. Say "next", or tell me what to change.
```

### Commit message guidance

- Follow the repository's existing convention (Conventional Commits, ticket prefixes,
  tense, capitalization) as seen in `git log`. If there is no clear convention, use a
  concise imperative subject, ideally at most 60 characters, e.g. `Add retry to tile fetcher`.
- **Default to subject line only.** Add a body only when the "why" is not obvious from
  the diff: a non-trivial design decision, a workaround for an external bug, a breaking
  change or migration, a behavior change users will notice, or a performance/security
  rationale. Keep the body to a few wrapped lines.
- If the developer modified your code before committing, base the message on the final
  diff, not on what you originally wrote.

## Phase 3 — Handling the developer's response

- **"next" / "go" / "continue" / "ok"**: go to Phase 2 for the next package.
- **Steering feedback** ("use a map instead", "rename this", "don't handle that case
  here"): apply it to the *current* package: re-read the files first, make the change,
  and give a short updated step report. Stay on the same step number.
- **The developer edited the code themselves**: re-read, respect it, and if asked to
  continue, build on their version. Never reintroduce what they removed.
- **Plan changes** ("skip step 4", "split step 3", "add a step for X"): update the plan,
  show the revised remaining steps briefly, and wait.
- **Questions**: answer them; do not start coding unless asked.
- **"stop" / "pair mode off" / "we're done"**: stop the step-by-step flow, summarize
  what was completed and what remains of the plan.

When all packages are done, give a one-paragraph wrap-up: what was built, anything left
open, and suggested follow-ups. Still do not commit.

## Things to avoid

- Implementing two packages at once "because they were small".
- Committing, staging, stashing, or resetting anything.
- Reformatting or reorganizing code outside the package's scope.
- Overwriting a file with a full rewrite based on an earlier read.
- Long reports that bury the diff summary, the developer will read the code itself.
