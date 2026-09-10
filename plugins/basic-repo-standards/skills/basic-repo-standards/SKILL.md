---
name: basic-repo-standards
description: Identify each project/app in a repo (or the repo itself, if single-project) and make sure each has a real README.md (not just a template), a CLAUDE.md with the baseline coding-standards goal statement, .gitignore coverage for env files plus a .env.example, and GitHub Actions CI covering the repo's actual working branches — fixing any gaps directly, no confirmation needed — then write a BASIC_REPO_STANDARDS.md report. Use when the user asks to "audit this repo", "standardize this repo", "check repo hygiene/standards", "onboard this repo", or similar.
---

# Basic Repo Standards

This is stack independent. Identify the apps, run the four checks below on each one,
fix any gap directly (these are additive, low-risk changes — no need to ask
first), then write the report.

## 1. Identify projects/apps

Work out whether the repository is a single project or a monorepo (workspace manifest,
several independent manifests, an `apps/*`/`packages/*` layout). List each
app found. Repeat checks 2–5 for every app (and the repo root, if it's a
monorepo with its own root-level concerns too).

## 2. README.md

Does each app have a `README.md`, and is it real content rather than a
scaffold template (framework default, "# Project Name", no real content)?

If missing or just a template, write/rewrite one covering:
- **Name & description** — what it is, what it does.
- **Tech stack** — actual languages/frameworks/tools in use.
- **Steps to install** — the real setup commands.
- **How to run / build / test** — matching actual scripts that exist.

## 3. CLAUDE.md

Every app must have a `CLAUDE.md`, and it must open with this goal statement,
verbatim, as the very first thing in the file:

> The goal is to produce code that is:
>
> - Maintainable
> - Readable
> - Consistent
> - Reusable where appropriate
> - Easy to extend
> - Safe to modify

- If `CLAUDE.md` exists but doesn't open with this statement, prepend it to
  the top of the file — leave the rest of the existing content as is.
- If `CLAUDE.md` doesn't exist at all, create one with the goal statement at
  the top, followed by sections written specific to this project (never a
  generic template): **Tech Stack** (actual languages/frameworks/tools in
  use), **Project Structure** (what lives where), **Do's and Don'ts**
  (concrete conventions for this codebase — naming, error handling, testing,
  etc.), and any other standards this project clearly needs.

## 4. .gitignore / env files

Does `.gitignore` cover `.env` and other env-shaped files? Is there a
`.env.example` documenting the required variables? Add/fix whichever is
missing. If an env file with real secrets is already tracked, flag it instead
of silently untracking it — that needs a rotation decision, not an automatic fix.

## 5. GitHub Actions

Does `.github/workflows/` exist, and does it trigger on the branches this repo
actually uses (check what's real, e.g. `dev`/`main` — don't assume)? Add or
fix the workflow if it's missing or targets the wrong branch.

## 6. Write the report

Write `BASIC_REPO_STANDARDS.md` at the repo root (if one already
exists, add a suffix to the name). For each app/project, list what was found and what was fixed for each of
checks 2–5.
