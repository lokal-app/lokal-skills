---
name: basic-coding-standards
description: Audit a repository — and, app by app, every subproject/monorepo package it contains — against baseline hygiene standards — README specificity, CLAUDE.md presence/coding-standards content/tech-stack documentation written as discrete headed rule sections (not one blob), GitHub Actions CI coverage with named checks and branch triggers covering both the real default branch and main/master, committed secrets/.env files, .gitignore, .env.example — plus, per app, surfacing every significant codebase-structure gap (lengthy-file pattern, everything mixed in one place, missing pieces, etc.), grouped by pattern with every affected file clubbed together rather than one finding per file, and asking whether to include a refactoring plan built around the user's stated priorities. Proposes a fix plan and only applies changes after explicit user confirmation. Use when the user asks to "audit this repo", "standardize this repo", "check repo hygiene/standards", "onboard this repo", or similar. Never modifies files during inspection; strict Plan → Confirm → Execute gate before any write.
---

# Basic Coding Standards Audit

This checklist is **technology-agnostic by design** — the same standard applies
whether the repo is Python, Go, a JS/TS monorepo, a mobile app, or anything else.
Never let a check collapse into "does it use this stack's specific tool/framework
convention"; express every finding and every proposed fix in terms that would make
sense to someone who has never seen this language, then translate to the repo's
actual tooling only when writing the concrete plan.

Audits a repository against a fixed checklist, then proposes changes. **No file in the
target repository is ever created, edited, or deleted until the user gives explicit,
unambiguous approval of a written plan.** This gate is the core rule of this skill and
overrides any instinct to "just fix it while I'm here."

The workflow has exactly two phases, in order, with a hard stop between them:

```
INSPECT (read-only) → AUDIT FINDINGS → PLAN → CONFIRM (explicit) → EXECUTE → VALIDATE → REPORT
```

## Phase 1 — Inspect (read-only, no exceptions)

Do not run any command or tool call that writes, moves, deletes, stages, or commits
anything in the target repo during this phase. Read-only shell (`ls`, `find`, `grep`,
`git status`, `git ls-files`, `cat`/Read), and nothing else.

### 0. Map the repo shape first — count and name every subproject/app

Before auditing individual files, determine whether this is a single project or a
monorepo. Don't stop at "yes, this is a monorepo" — produce a concrete, named,
counted inventory: exactly how many apps/subprojects exist and what each is called
(e.g. "3 apps: `apps/web`, `apps/admin`, `apps/api`"). This inventory is a
first-class fact of the audit, not internal bookkeeping — state it up front in the
findings, since every check and the final report are organized around it, and
every later check must be repeated for each entry in it.

Signals to check:
- Workspace manifests: `pnpm-workspace.yaml`, `lerna.json`, `nx.json`, `turbo.json`,
  a `"workspaces"` field in root `package.json`, `go.work`, a multi-module
  `settings.gradle` — these often enumerate the subprojects explicitly; read the
  list out of the manifest rather than inferring it only from directory names.
- Multiple independent manifests below root: several `package.json` / `pyproject.toml`
  / `go.mod` / `pom.xml` / `Cargo.toml` files in different top-level directories that
  each look like an installable/buildable unit (has its own lockfile or build config).
- A top-level layout like `apps/*`, `packages/*`, `services/*`, each with its own
  README/CI/config.

If the workspace manifest and the directory listing disagree (e.g. a folder under
`apps/` isn't in the workspace's package list, or vice versa), that mismatch is
itself worth a note — don't silently pick one source over the other.

If subprojects exist, record the list and note that each one needs its own pass
through **every** check below, 1 through 5 — nothing in this skill is a
repo-once check when the repo is a monorepo. That includes the parts that are
easy to do only at the root by accident: each app/package gets judged on its own
README, its own `CLAUDE.md` (goal-statement preamble, tech stack, standards
rubric), its own CI coverage, its own secrets/gitignore posture, and its own
structural-gaps read — an app can be messy and gap-ridden while its
neighbor in the same repo is small and clean, and a monorepo-root `CLAUDE.md`
does not excuse an individual app from needing its own if its stack/conventions
differ. A repo-level finding like "root README is generic" is distinct from
"packages/api has no README at all." Don't silently audit only the root when
subprojects exist.

### 1. README

- Does `README.md` (or `README`) exist at the root, and at each subproject root?
- If it exists, is it **specific** or **generic**? Generic = boilerplate scaffold
  text (framework default like CRA/Vite/`cargo new` placeholder, "# Project Name",
  a stock license/description with no real content), doesn't name the actual
  project, doesn't match the actual tech stack/scripts in the manifest, or is
  missing basic setup/run instructions a new contributor would need. Specific =
  names the project, explains what it does, and setup/run instructions match what
  actually exists (compare its documented commands against `package.json` scripts /
  Makefile targets / actual CI steps).

**What counts as a real README.** Same bar for every repo regardless of stack —
enough that someone with zero context could clone the repo and get it running
without asking a teammate. When judging an existing README, or drafting one, cover
(translate to the repo's actual stack/tooling, never paste a template verbatim):

- **What it is** — one or two sentences naming the project and what problem it
  solves; not just a name/logo with no description.
- **Prerequisites** — runtime/language version, required external services or
  accounts, anything a fresh machine wouldn't already have.
- **Setup** — the exact commands to install dependencies and configure local
  environment (e.g. copying `.env.example`), verified against what the manifest/
  lockfile actually requires, not a guess.
- **Run / build / test** — the exact commands to run the project locally, run the
  test suite, and produce a build, each matching a real script/target that exists
  (`package.json` scripts, `Makefile`, `justfile`, CI steps) — a documented command
  that doesn't exist in the repo is itself a finding.
- **Project structure** (for anything beyond a trivial single-file repo) — a short
  map of top-level directories and what lives where, especially in a monorepo.
- **Where to look next** — pointers to `CLAUDE.md`/contributing docs, architecture
  docs, or where CI/deploy is configured, if those exist.

A README satisfying only some of the above is a partial finding — name which parts
are missing rather than treating it as pass/fail. A README that is accurate but
stale (documents a command/script that was renamed or removed) is a finding too.

### 2. CLAUDE.md

- Does `CLAUDE.md` exist at the root, **and at each app/subproject root**? Default
  to expecting one per app — each app is audited as its own unit (check 0), so it
  needs its own standards file. The only exception is when a root `CLAUDE.md`
  explicitly and completely covers that app already (states its stack, and its
  standards apply to it without gaps) — don't assume this, verify it by actually
  checking whether the root file mentions the app/its stack; a root `CLAUDE.md`
  written with only one app's stack in mind does not cover a sibling app on a
  different stack just because they share a repo.
- If it exists, does it contain actual **coding standards**, or is it
  empty/near-empty/just a copy of README content with no standards?

**What counts as real coding standards.** The bar is the same across every repo
regardless of language, framework, or platform — the goal is code that is clean,
reusable, and structured, not compliance with any single stack's idioms.

**Required opening.** Every `CLAUDE.md` this skill drafts must open with this goal
statement, verbatim, before anything else (tech stack, standards, or otherwise).
When judging an existing `CLAUDE.md`, its absence is itself a finding — add it
rather than treating the file as already sufficient:

> The goal is to produce code that is:
>
> - Maintainable
> - Readable
> - Consistent
> - Reusable where appropriate
> - Easy to extend
> - Safe to modify
>
> Follow these guidelines when creating or modifying code.

When judging an existing `CLAUDE.md`, or drafting the rest of one, cover (in
language-agnostic terms — translate to the repo's actual stack, never paste a
template verbatim):

- **Naming** — names say what something is/does; no abbreviations that need
  decoding, no misleading names.
- **Single responsibility & size** — functions/files/modules do one thing; large
  files or functions are a signal to split, not a target to hit.
- **DRY / reuse** — shared logic gets extracted once and reused, instead of copied
  across call sites; the standard should say *where* shared code lives (e.g. a
  `utils`/`shared`/`common` layer) so reuse has an obvious home.
- **Structure & boundaries** — a stated directory/module layout and a rule for what
  goes where, so a new file's location is obvious rather than improvised each time.
- **Readability over cleverness** — straightforward code preferred to dense or
  clever code; comments explain *why*, not *what*, and only where the reason isn't
  obvious from the code itself.
- **Error handling** — a stated convention (e.g. fail fast vs. recover, how errors
  surface to the caller) rather than ad hoc handling per file.
- **Testing expectations** — what must be tested and to what extent, stated
  concretely enough to be checked (not just "write tests").
- **Commit/PR conventions** — message format, what a PR description must contain.

Also required, but factual rather than a principle — **Tech stack**: `CLAUDE.md`
must name the actual language(s), framework(s), package manager, and other major
tools/runtimes in use (verified against the real manifest/lockfile, not guessed).
This is the one place stack-specific detail belongs; the areas above stay written
as portable principles, and may reference the stated stack for how a principle is
applied locally (e.g. "naming: files use kebab-case per this repo's Next.js
convention").

Naming the tech stack is necessary but never sufficient on its own. A `CLAUDE.md`
that only restates the tech stack, lists dependencies, or repeats the README does
**not** satisfy this check — flag it as missing standards even though the file
exists. A `CLAUDE.md` that covers the above only partially is a finding too: name
which of the areas above are missing.

**Required format — discrete rules, not one blob.** The document itself must model
the structure it preaches (check 2's own "Structure & boundaries" item applies to
`CLAUDE.md` as a file, too). Each topic above gets its **own heading** and a short,
direct rule statement under it — never all of them merged into one undifferentiated
paragraph or one flat bullet list with no separation. A `CLAUDE.md` with genuinely
good content that's still crammed into a single wall of text is a **formatting
finding**, separate from a content finding — call it out even when the content
itself is otherwise solid. Skeleton to draft against (headings are illustrative;
name them for what's actually in the repo):

```markdown
> [goal statement — verbatim, from above]

## Tech Stack
[languages/frameworks/tools, factual]

## Naming
[rule, 1-2 sentences]

## Structure
[rule: directory/module layout, what goes where]

## Reuse (DRY)
[rule: where shared logic lives]

## Readability
[rule: clarity over cleverness, comment policy]

## Error Handling
[rule: convention for this repo]

## Testing
[rule: what must be tested, to what extent]

## Commits & PRs
[rule: message format, PR description requirements]

## AI-Assisted Review Tooling (optional)
[optional: recommend a codebase-graph/blast-radius tool for AI code review, see below]
```

Each section should be short — a rule, not an essay — and every rule stated
independently checkable on its own, so a reader (or reviewer) can jump straight to
the one section relevant to what they're doing instead of reading the whole file.

**Optional: AI-assisted review tooling.** Unlike the sections above, this one is a
suggestion, not a required rubric item — present it in the plan as an optional
addition the user can accept or skip, never fold it in silently. When it fits
(especially larger codebases, or ones where check 5 found real structural gaps),
the `CLAUDE.md` may recommend installing
[code-review-graph](https://github.com/tirth8205/code-review-graph) — a CLI/MCP
tool (`pip install code-review-graph`) that parses the codebase into a dependency
graph so AI coding assistants can scope a review to the actual "blast radius" of a
change instead of reading the whole repo. It works across languages (tree-sitter
based), so it fits regardless of the repo's stack. Note it needs Python available
to run even in a non-Python repo, since the tool itself is a Python package.

### 3. GitHub Actions / CI

- Does `.github/workflows/` contain any workflow files?
- If yes, is coverage **sufficient for what the repo actually has**? Don't apply a
  generic "must have lint+test+build" template — derive expectations from the repo:
  if there's a test script/test directory, CI should run tests; if there's a build
  step (compiled language, bundler config), CI should build; if there's a linter
  config, CI should lint. Flag gaps where the repo clearly has the capability but CI
  doesn't exercise it, not stylistic preferences.
- **In a monorepo, judge coverage per app, not just "does CI exist."** A single
  shared `.github/workflows/` at root is normal and fine — a monorepo doesn't need
  one workflow file per app — but check that CI's actual jobs/steps (path filters,
  a build matrix, or separate jobs per app) exercise **every** app's own
  lint/test/build capability. It's a common and easy-to-miss gap: root CI runs
  `apps/web`'s tests but silently never touches `apps/api`'s, because a workflow
  was written for the first app added and never extended. Name which specific
  app(s) are missing coverage, not just "CI could be broader."
- If workflows reference environment variables/secrets, check whether they're pulled
  from `secrets.*` / `vars.*` properly or hardcoded inline.
- **Named checks.** Every job (and the workflow itself) should have an explicit,
  descriptive `name:` — not just a filename-derived or default identifier. A status
  check with a clear name (e.g. `name: lint`, `name: test`, workflow
  `name: CI`) is what shows up in the PR checks list and in branch-protection
  "required status checks," so an unnamed or vaguely named job (`build`, `job1`,
  or nothing at all) is a finding: it makes required checks hard to configure
  correctly and hard for a reviewer to tell what actually ran.
- **Confirm the branch name actually matches the repo, and trigger on both.** Find
  the repo's real default branch (`git symbolic-ref refs/remotes/origin/HEAD` or
  `git remote show origin`, or ask the user if that's not conclusive — don't just
  assume `main`). Check every `on: push`/`on: pull_request` (and any other
  branch-filtered trigger, protection rule references, deploy-target branch, etc.)
  `branches:` list in every workflow file against that real name — a workflow
  hardcoded to a branch that isn't the actual default (most commonly `master` in a
  repo whose default is now `main`, or vice versa) is a **silent CI failure mode**:
  the file is present and looks correct, but never triggers, because it filters on
  a branch nobody pushes to. The fix to propose is not just "rename to the
  default" — recommend the trigger list include **both** the actual default branch
  **and** the other of `main`/`master` (i.e. `branches: [main, master]`), so CI
  keeps working through a branch rename, a fork, or any repo where the two are
  used inconsistently, instead of being one rename away from silently going dark
  again.

### 4. Secrets, `.gitignore`, `.env.example`

- `git ls-files` (not just `ls`) for anything matching `.env`, `.env.*`,
  `*.pem`, `*.key`, `credentials*`, `*secret*` etc. — tracked env/secret-shaped
  files are a finding regardless of content.
- For any tracked env-shaped file, skim it for what looks like real
  values (non-placeholder-looking API keys, DB URLs with real hosts/passwords,
  tokens) vs. genuinely empty/placeholder content — note the distinction in the
  finding since remediation urgency differs (rotate + purge from history vs. just
  stop tracking it).
- Does `.gitignore` exist and does it cover env/secret file patterns?
- Does `.env.example` (or equivalent) exist to document required variables when a
  tracked or gitignored `.env` pattern is in use?

### 5. Codebase structure — gaps per app, grouped by pattern

This check is about the *code itself*, not repo hygiene files, so it's additive to
1–4, not a replacement. Run it for **every** app/subproject from check 0 — this is
not size-gated; even a small app can have real structural gaps worth naming (and a
small, clean app may genuinely have none — that's a fine outcome, don't invent
findings to fill space).

- Get a quick read-only signal, e.g.: `git ls-files | xargs wc -l 2>/dev/null | sort
  -rn | head -20` for the biggest files, a rough file count per top-level
  directory, and a skim for structural smells.
- Look across three angles:
  - **Messy** — readability/maintainability problems. Concretely, look for: a
    **lengthy-file pattern** (one or more files far larger than the rest, that
    keep growing because there's no natural place to split them); **everything
    mixed in one place** — unrelated responsibilities (e.g. request handling,
    business logic, and data access all in the same file/function) with no
    separation of concerns; duplicated logic copy-pasted across files instead of
    shared; deep nesting; unclear/misleading naming.
  - **Scalable** — whether the current structure holds up as the app grows: tight
    coupling between unrelated parts, no module boundaries, files that will keep
    growing without a natural place to split, missing abstraction where reuse is
    clearly needed.
  - **Gaps** — pieces missing relative to what a codebase this size/kind should
    have: no tests, no stated error-handling convention, no clear layering between
    e.g. data/business-logic/presentation.
- **No cap on how many findings — but club by pattern, don't list one finding per
  file.** Report every genuinely significant gap category found; there's no fixed
  number (not 3, not any other target) to hit or stay under. What keeps this from
  turning into a wall of noise is grouping: if several files across the app share
  the same violation (e.g. all missing the same validation, or all copy-pasted
  from the same original), that is **one** finding with every affected file listed
  under it — not one finding per file. Only split into separate findings when the
  pattern or root cause is genuinely different.
- State each finding as **[named pattern/principle violated] — affected
  file(s)**, not a narrative paragraph: lead with the concrete label (e.g.
  "Duplicate/near-duplicate code," "Single Responsibility Principle violation,"
  "Lengthy-file / god-file pattern," "No test coverage"), then *every* file that
  shares it and a short evidence note (line counts, what's duplicated, why it
  violates the label). e.g.:
  - "**Duplicate/messy code** — `src/features/chat-legacy/QuizCard.jsx` and
    `chat-redesign/QuizCard.jsx` (426 lines each) are near-duplicate components
    instead of one shared one."
  - "**Single Responsibility Principle violation** — `MessageContent.jsx`,
    `OrderSummary.jsx`, `UserProfileCard.jsx` (each 300+ lines) all mix rendering,
    data-fetching, and formatting in one component."
  - "**No test coverage** — no test framework or test files exist at all; only
    `dev`/`build`/`preview` scripts."
  This makes each finding scannable as "pattern → every file affected," not prose
  that has to be read end to end to find out what's wrong or where, and not a
  flood of near-duplicate one-file findings that are really the same problem.

Do **not** draft a refactor plan at this point — report the findings and ask
whether to include one; see Phase 2, which handles this per app.

Repeat checks 1–5 for every subproject identified in step 0.

## Phase 2 — Audit findings → Plan → Confirm → Execute

### Codebase structure (check 5) gets a separate track — ask before planning, per app

Hygiene findings (checks 1–4) go straight into the itemized plan below. **Check 5
does not** — it's a bigger, more subjective decision than "add a missing
`.env.example`," so never draft a refactoring plan unprompted. Run this entire
track **independently per app** — one app's gaps and yes/no answer are unrelated
to another's; the user may want a refactor plan for `apps/web` while declining one
entirely for `apps/admin`. Never merge multiple apps' gaps into one combined ask.
Instead, per app:

1. Report that app's **structural gaps** from check 5 — every pattern found, each
   labeled by pattern/principle with *every* affected file clubbed under it (the
   format check 5 requires — not a narrative paragraph, and not one finding per
   file), then ask directly whether they want a refactoring plan for that app.
   e.g.:

   > For `apps/web`, the structural gaps found:
   > 1. **Lengthy-file / god-file pattern** — `OrderProcessor.jsx` (1,340 lines),
   >    `CheckoutFlow.jsx` (1,180 lines) mix unrelated responsibilities.
   > 2. **Duplicate/messy code** — validation logic duplicated across
   >    `SignupForm.jsx`, `ProfileForm.jsx`, `BillingForm.jsx`, already diverged
   >    (one copy is missing a check the others have).
   > 3. **No test coverage** — the payments flow has no tests.
   >
   > Want me to put together a refactoring plan for `apps/web`?

   There's no fixed count to report — list every finding that clears the bar in
   check 5, no more and no fewer.
2. If they say yes, ask for their priorities/constraints before drafting anything —
   don't assume scope. Useful questions: which parts of the codebase matter most
   right now, incremental (small PRs alongside ongoing feature work) vs. a
   dedicated refactor push, any areas that are off-limits (e.g. "don't touch the
   payments module, it's being rewritten separately"), and any deadline pressure
   that should shape how aggressive to be.
3. Only then draft the refactor plan for that app, and present it as its **own**
   itemized list with its own confirmation question — never bundled into the same
   yes/no as the hygiene fixes, or with another app's refactor plan. Someone may
   want the quick hygiene fixes applied immediately while still thinking over a
   multi-week refactor for one specific app.
4. If they say no (or don't want a refactor plan right now) for that app, just note
   its structural gaps in the report as an open item and move on — don't push.
   Repeat steps 1–4 for the next app.

### Create the plan

Turn findings into a concrete, itemized plan. The checklist below is what you must
have *worked out* for each item before presenting anything — it is not a template
for the report itself. What you actually show the user must stay **short and
simple**: one line per issue, one line per proposed change, in the same compressed
form as the example below. Expand beyond one line only if a single item is
genuinely destructive or ambiguous enough to need a caveat.

For each item, work out (but don't necessarily print in full):
- **What** needs to change (exact file(s), created vs modified vs removed-from-git).
- **Why** (which check it addresses).
- **Content summary** of what's being added/changed — not the full file necessarily,
  but enough that the user knows what they're approving (e.g. "CLAUDE.md will
  document: naming conventions [x], test requirements [y], directory layout [z]" or
  "CI workflow gains a `test` job running `npm test` on push/PR"). When drafting new
  `CLAUDE.md` content, base it on the rubric in check 2 (naming, single
  responsibility, DRY/reuse, structure, readability, error handling, testing,
  commit conventions), written in this repo's actual language/tooling terms and
  laid out as the discrete headed rule sections check 2 requires — not one blob —
  so the goal stated to the user should be recognizable as "standards for clean,
  reusable, structured code," not a generic template.
- **Assumptions or risks**, especially for anything destructive or security-sensitive
  (e.g. "removing `.env.staging` from git tracking does not remove it from history —
  if it contains real secrets, they should be rotated; git history purge is out of
  scope unless you ask for it explicitly").

If the repo is a monorepo, open the findings with the app inventory from check 0
(count and names), and label each numbered issue with which app it belongs to —
never present a flattened list that hides that. Present the plan as: numbered
issues found, then a bulleted list of proposed changes, ending with an explicit
question asking whether to proceed — always shown to the user before any file is
touched, no exceptions. Example (single-project repo; a monorepo's issue list
would prefix each line with its app, e.g. "1. [apps/web] ..."):

> I found the following issues:
>
> 1. `README.md` is generic and does not document this repository.
> 2. `CLAUDE.md` is missing.
> 3. GitHub Actions only runs lint; tests and build are not covered.
> 4. CI workflows trigger on `master`, but the repo's default branch is `main` —
>    CI has not run on pushes/PRs to `main`. Jobs are also unnamed.
> 5. `.env.staging` is committed.
>
> Proposed changes:
> - Update `README.md` with repository-specific setup instructions.
> - Add `CLAUDE.md` with the required repository standards.
> - Update GitHub Actions to run lint, tests, and build.
> - Update workflow triggers to `branches: [main, master]` and add named checks
>   (`name: lint`, `name: test`, `name: build`).
> - Remove `.env.staging` from Git tracking and update `.gitignore`.
> - Add `.env.example`.
>
> Should I proceed with these changes?

### Confirmation gate — do not skip, do not infer

**Only an explicit, unambiguous go-ahead authorizes execution.** Accept: "yes",
"proceed", "go ahead", "approve", "do it", or equally unambiguous equivalents.

**None of the following count as approval, even if they sound positive:**
- The user asking for the audit itself.
- The user asking what should be changed / discussing the findings.
- The user saying "looks good" or reacting positively without clearly authorizing
  execution.
- Silence, or moving on to a different topic.
- This skill's own judgment that a fix is "obviously" needed.

If approval is ambiguous, ask a direct yes/no confirmation question before touching
any file — don't guess, don't proceed "to save time."

If the user asks for changes to the plan instead of approving it: revise the plan,
present the revision, and ask for confirmation again. Repeat until an explicit
approval or an explicit decline is received.

### Execute — only the approved plan

- Make exactly the changes in the approved plan. No unrelated refactors, renames, or
  "while I'm here" cleanup, even if clearly beneficial.
- If, during execution, you discover an additional necessary change not in the
  approved plan (e.g. fixing the README also requires updating a script that no
  longer exists): **stop**, explain what was found and why it needs a change, update
  the plan, and get confirmation on that addition specifically before making it. Do
  not fold it in silently.
- Prefer safe, reversible operations for anything git-tracked: use `git rm --cached`
  to untrack a committed env file (keeps the local file, stops tracking it) rather
  than deleting it outright, unless the user asked for outright deletion.
- Never rewrite git history (e.g. to purge secrets from past commits) without a
  separate, explicit ask — that's a distinct, higher-risk operation from what this
  skill's default plan covers. If tracked secrets look real, call this out clearly
  in the plan/report as something the user should rotate and decide on separately.

### Validate and report

After execution:
- Run any quick, safe validation available (e.g. `git status`, confirm new files
  exist, lint/parse a modified workflow YAML if a linter is available) — don't
  invent a validation step that isn't meaningful for the change made.
- Report: files changed (created/modified/removed-from-tracking), a one-line summary
  per change, validation performed, any findings from the audit that were **not**
  included in the approved plan (still open issues), and anything that couldn't be
  safely automated (e.g. rotating a leaked secret, purging git history — flag as
  the user's action item, don't attempt it).

### Share a written audit report

Always produce a Markdown report of the full audit as its own file, in addition to
the short chat summary — this is the one file write in this skill that isn't gated
behind plan approval, because it's a record of the audit itself, not a change to
the repository's standards/config. Still, mention you're writing it.

- Write it to `CODING_STANDARDS_AUDIT.md` at the audited repo's root (or the
  relevant subproject root, for a single-subproject audit). If one already exists
  from a prior run, overwrite it — the report reflects the latest audit, it isn't a
  history log.
- Contents: audit date, repo/subproject(s) covered, every finding from checks 1–5
  grouped by check (including ones not acted on), the plan that was proposed, which
  items were approved/executed vs. declined/left open, and for check 5, each app's
  structural-gap findings (pattern + every affected file, per check 5's format)
  plus the refactor-plan discussion outcome (asked, and what the user decided).
  For a monorepo, break this down **per app/subproject**, not one merged list — a
  reader needs to see that `apps/web` has flagged structural gaps (and opted into
  a refactor plan) while `apps/admin` has none, not a single flattened summary
  that hides which app each finding belongs to.
- This report is for the team, not just this session — write it so someone who
  wasn't in the conversation can read it standalone and understand what was found
  and what happened.
- Do not commit or push it — creating the file is exempt from the approval gate,
  but putting it under version control is still a repository change like any
  other, and stays behind the same explicit-approval rule as everything else.

## Hard rule recap

No repository *fix* — anything that changes standards, config, or tracked
content — happens before explicit approval of a specific, itemized plan.
Inspection is always read-only. The one exception is the audit report file
itself (see "Share a written audit report"), which documents the audit rather
than changing the repo. When in doubt about whether something counts as
approval, ask instead of assuming.
