---
name: basic-coding-standards
description: Audit a repository (and any subprojects/monorepo packages) against baseline hygiene standards — README specificity, CLAUDE.md presence/coding-standards content/tech-stack documentation, GitHub Actions CI coverage and branch-name correctness, committed secrets/.env files, .gitignore, .env.example — plus, for large/lengthy codebases, flagging structural debt and (only if the user wants one) drafting a refactoring plan built around their stated priorities. Proposes a fix plan and only applies changes after explicit user confirmation. Use when the user asks to "audit this repo", "standardize this repo", "check repo hygiene/standards", "onboard this repo", or similar. Never modifies files during inspection; strict Plan → Confirm → Execute gate before any write.
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

### 0. Map the repo shape first — detect subprojects / monorepo

Before auditing individual files, determine whether this is a single project or a
monorepo, since every later check must be repeated per subproject.

Signals to check:
- Workspace manifests: `pnpm-workspace.yaml`, `lerna.json`, `nx.json`, `turbo.json`,
  a `"workspaces"` field in root `package.json`, `go.work`, a multi-module
  `settings.gradle`.
- Multiple independent manifests below root: several `package.json` / `pyproject.toml`
  / `go.mod` / `pom.xml` / `Cargo.toml` files in different top-level directories that
  each look like an installable/buildable unit (has its own lockfile or build config).
- A top-level layout like `apps/*`, `packages/*`, `services/*`, each with its own
  README/CI/config.

If subprojects exist, record the list and note that each one needs its own pass
through checks 1–4 below (a repo-level finding like "root README is generic" is
distinct from "packages/api has no README at all"). Don't silently audit only the
root when subprojects exist.

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

- Does `CLAUDE.md` exist at the root (and per subproject, if subprojects have
  meaningfully different stacks/conventions)?
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

### 3. GitHub Actions / CI

- Does `.github/workflows/` contain any workflow files?
- If yes, is coverage **sufficient for what the repo actually has**? Don't apply a
  generic "must have lint+test+build" template — derive expectations from the repo:
  if there's a test script/test directory, CI should run tests; if there's a build
  step (compiled language, bundler config), CI should build; if there's a linter
  config, CI should lint. Flag gaps where the repo clearly has the capability but CI
  doesn't exercise it, not stylistic preferences.
- If workflows reference environment variables/secrets, check whether they're pulled
  from `secrets.*` / `vars.*` properly or hardcoded inline.
- **Confirm the branch name actually matches the repo.** Find the repo's real
  default branch (`git symbolic-ref refs/remotes/origin/HEAD` or `git remote show
  origin`, or ask the user if that's not conclusive — don't just assume `main`).
  Then check every `on: push`/`on: pull_request` (and any other branch-filtered
  trigger, protection rule references, deploy-target branch, etc.) `branches:` list
  in every workflow file against that real name. Flag any workflow hardcoded to a
  branch that isn't the actual default (most commonly `master` referenced in a repo
  whose default branch is now `main`, or vice versa) — this is a **silent CI
  failure mode**: the workflow file is present and looks correct, but never
  triggers, because it filters on a branch nobody pushes to.

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

### 5. Codebase scale — flag refactor opportunities in large/lengthy projects

This check is about the *code itself*, not repo hygiene files, so it's additive to
1–4, not a replacement. Only run it if the repo is actually large/lengthy — a small
project doesn't need this.

- Get a quick read-only signal of scale and structural health, e.g.:
  `git ls-files | xargs wc -l 2>/dev/null | sort -rn | head -20` for the biggest
  files, a rough file count per top-level directory, and a skim for obvious smells
  (a handful of "god files" far larger than the rest, deeply nested logic, the same
  block of logic copy-pasted across several files, no discernible module
  boundaries).
- Treat the repo as "lengthy" when it shows real scale (well beyond a small
  script/app) **and** structural symptoms like the above — size alone without
  symptoms isn't a finding.
- If it qualifies, record it as a distinct finding category — **structural debt**
  — separate from the hygiene findings in checks 1–4. Name the specific symptoms
  observed (e.g. "3 files over 1000 lines mixing unrelated responsibilities",
  "the same validation logic duplicated in 4 places") rather than a vague "code
  could be cleaner."
- Judge and state **how strongly refactoring is recommended, and why** — this is
  not optional hedging, give a real opinion:
  - **Strongly recommend** when the symptoms are actively costing the team now —
    the duplication has already caused a bug fixed in one copy but not another,
    the god files are the ones getting touched most often (check recent commit
    activity on them), or the tangled structure makes onboarding/reviewing
    genuinely hard.
  - **Recommend** when the debt is real but currently contained — it will bite
    later (as the team or codebase grows) but isn't actively causing incidents.
  - **Optional / low priority** when the smells are present but isolated,
    low-churn, or in code nobody expects to extend further.
  - Base the level on concrete evidence (churn/`git log --oneline -- <file>`
    frequency on the worst files, duplication that has actually diverged and
    caused a bug, size relative to the rest of the repo) — not a gut feeling.

Do **not** draft a refactor plan yet at this point — see the refactor-plan handling
in Phase 2, which requires asking the user first.

Repeat checks 1–4 (and 5, where it applies) for every subproject identified in
step 0.

## Phase 2 — Audit findings → Plan → Confirm → Execute

### Structural debt (check 5) gets a separate track — ask before planning

Hygiene findings (checks 1–4) go straight into the itemized plan below. **Structural
debt does not** — it's a bigger, more subjective decision than "add a missing
`.env.example`," so never draft a refactoring plan unprompted. Instead:

1. Report the structural-debt finding alongside the others, but expand this one
   beyond a single line: state the symptoms, your recommendation strength
   (strongly recommend / recommend / optional — from check 5) **and the concrete
   reason behind it**, then ask whether they want a refactor plan. e.g.: "This repo
   also has notable structural debt: 3 files over 1000 lines mixing
   responsibilities, and duplicated validation logic in 4 places that has already
   diverged (one copy is missing a check the others have). I'd strongly recommend
   refactoring this — it's an active bug source, not just untidiness. Want me to
   put together a refactoring plan for it?" Give your honest assessment even when
   it's "optional" — don't inflate urgency to push action, and don't soften a real
   risk to avoid sounding alarmist.
2. If they say yes, ask for their priorities/constraints before drafting anything —
   don't assume scope. Useful questions: which parts of the codebase matter most
   right now, incremental (small PRs alongside ongoing feature work) vs. a
   dedicated refactor push, any areas that are off-limits (e.g. "don't touch the
   payments module, it's being rewritten separately"), and any deadline pressure
   that should shape how aggressive to be.
3. Only then draft the refactor plan, and present it as its **own** itemized list
   with its own confirmation question — never bundled into the same yes/no as the
   hygiene fixes. Someone may want the quick hygiene fixes applied immediately while
   still thinking over a multi-week refactor.
4. If they say no (or don't want a refactor plan right now), just note the finding
   in the report as an open item and move on — don't push.

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
  commit conventions) written in this repo's actual language/tooling terms — the
  goal stated to the user should be recognizable as "standards for clean, reusable,
  structured code," not a generic template.
- **Assumptions or risks**, especially for anything destructive or security-sensitive
  (e.g. "removing `.env.staging` from git tracking does not remove it from history —
  if it contains real secrets, they should be rotated; git history purge is out of
  scope unless you ask for it explicitly").

Present the plan as: numbered issues found, then a bulleted list of proposed changes,
ending with an explicit question asking whether to proceed — always shown to the
user before any file is touched, no exceptions. Example:

> I found the following issues:
>
> 1. `README.md` is generic and does not document this repository.
> 2. `CLAUDE.md` is missing.
> 3. GitHub Actions only runs lint; tests and build are not covered.
> 4. CI workflows trigger on `master`, but the repo's default branch is `main` —
>    CI has not run on pushes/PRs to `main`.
> 5. `.env.staging` is committed.
>
> Proposed changes:
> - Update `README.md` with repository-specific setup instructions.
> - Add `CLAUDE.md` with the required repository standards.
> - Update GitHub Actions to run lint, tests, and build.
> - Update workflow trigger branches from `master` to `main`.
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
  items were approved/executed vs. declined/left open, and the refactor-plan
  discussion outcome from check 5 if it came up (asked, and what the user decided).
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
