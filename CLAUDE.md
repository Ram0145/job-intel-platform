# CLAUDE.md — job-intel-platform

Context and rules for Claude Code sessions on this repo. Read this first, every session.

## What this project is

A from-scratch learning project by Ram ([@Ram0145](https://github.com/Ram0145)): web scraping → secure storage → data modeling → inverted index → Azure AI Search vectors → API → UI → AI agent and chatbot. **Learning is the goal, not just a working app.** Networking, security (especially SQL injection), traffic inspection, Git/GitHub/GitLab and CI/CD are practiced at every step.

- `PLAN.md` — master plan: phases 0–11, progress dashboard, decisions, command log (Appendix A). **Source of truth for status.**
- `SECURITY_LOG.md` — threat model, findings register, per-phase Security & Traffic Gate records.

Ram is an experienced data engineer (Python, SQL). Data-engineering basics can move quickly; networking, security, containers, DevOps, frontend and agents should be explained from first principles.

## Start of every session

1. Read `PLAN.md` (dashboard, current phase, the latest Command log session) and `SECURITY_LOG.md`.
2. Tell Ram in two or three lines where we are and what the next step is.
3. Run `git status` and `git log --oneline -5` and say what they show.

## Working agreement (always follow)

1. **Show the commands for every operation.** Git, GitHub, Docker, Linux, Postgres, Azure: the exact commands, in code blocks.
2. **Explain every command**: what it does, what each flag means, why we need it now, and what output to expect. Explain new concepts *before* the code that uses them.
3. **Ram types, Claude explains.** By default give Ram the commands to run himself. Run them yourself only when he asks or the step is purely mechanical, and still show what you ran.
4. **Log commands** in `PLAN.md` Appendix A under a new session heading: copy-pasteable blocks, a `#` comment line above each command.
5. **Label OS-specific commands** (Windows PowerShell / macOS / Linux) once Ram's laptop OS is known.
6. **Every change goes through a pull request**: branch → commit → push → PR → checks → squash merge. Never push to `main`.
7. **End every session with docs updated**: `PLAN.md` checkboxes, dashboard, changelog, command log; `SECURITY_LOG.md` gate record and findings. Ask Ram for his Learning Journal entry; don't write it for him.

## Conventions

- **Branches:** `feat/<phase>-<topic>`, `fix/<topic>`, `sec/<finding-id>`, `docs/<topic>`, `lab/<topic>`. Example: `feat/p1-greenhouse-client`.
- **Commits:** Conventional Commits (`feat:`, `fix:`, `docs:`, `sec:`, `ci:`, `test:`, `refactor:`).
- **PR body:** What · Why · Security & Traffic Gate (what was tested) · How to test. Merge with **Squash and merge**.
- **Commit identity:** Ram commits as `Ram Sirusanagandla <92075495+Ram0145@users.noreply.github.com>` (GitHub noreply address, keeps his personal email out of a public repo). Once the `main` ruleset requires signed commits, commits are signed with his SSH key.
- **Stack:** Python 3.12, `uv`, `ruff`, `pytest`, `httpx`, PostgreSQL 16, FastAPI, Nginx, React + TypeScript, Docker Compose, GitHub Actions. Details and reasons: `PLAN.md` §5 and §12.
- **Layout:** `PLAN.md` §6.
- **Phase completion:** a phase is DONE only after its Security & Traffic Gate passes and is recorded in `SECURITY_LOG.md`.

## Rules of engagement (non-negotiable)

1. Only scan or attack systems we own (our containers, our Azure resources).
2. Scrape only public job-board APIs and public career pages. Respect `robots.txt`, rate-limit every host, use an honest User-Agent (project name + repo URL, **no personal email**). Nothing behind a login (no LinkedIn, no Indeed).
3. Vulnerable-on-purpose code lives only in `labs/`, binds to `127.0.0.1`, is never deployed or imported by `services/`.
4. Never commit secrets, not even fake-looking real ones. If one leaks: rotate first, then clean history.
5. Resumes are personal data: Postgres only, never in the Azure index, redacted before any LLM call.

## Free-tier limits to remember

- Azure AI Search Free: one service per subscription, 50 MB storage, 3 indexes, no private endpoints; may be deleted after long inactivity.
- Azure free account: $200 credit for the first 30 days only, so don't create the account until Phase 6. Set a $5 budget alert first.
- GitHub Models (LLM): free but rate-limited per minute and per day.
- GitLab.com Free: 400 CI compute minutes per month.

## Current status (update at the end of every session)

- **Phase 0 — Foundations: IN PROGRESS.**
- Done: public repo created; PR #1 (plan) squash-merged; PR #2 (working agreement + this file) opened.
- Unknown — ask first: Ram's laptop OS; whether Git, Python 3.12, `uv`, Docker Desktop and the GitHub CLI (`gh`) are installed.
- **Next steps, in order** (details in `PLAN.md` Phase 0):
  1. Confirm PR #2 is merged; delete merged branches; turn on "Automatically delete head branches".
  2. Laptop check: OS and tool versions (`git --version`, `python3 --version`, `docker --version`, `gh --version`).
  3. Set the commit identity (noreply email) and GitHub's "Keep my email addresses private" + "Block command line pushes that expose my email".
  4. Create an ed25519 SSH key; add it to GitHub as an authentication key **and** a signing key; switch the remote to SSH; turn on commit signing and Vigilant mode.
  5. Repo security settings: secret scanning, push protection, Dependabot alerts, private vulnerability reporting.
  6. `main` ruleset and `v*` tag ruleset exactly as in `PLAN.md` §7.3.
  7. Python project: `uv`, `ruff`, `pytest`, `.gitignore`, pre-commit (ruff, gitleaks, detect-private-key, large files), `.env.example`, settings loader.
  8. `.github/`: `dependabot.yml`, `CODEOWNERS`, PR template with the gate checklist, issue templates.
  9. `ci.yml` (lint → test → gitleaks → pip-audit; `permissions: contents: read`; actions pinned to SHAs), then make it a required check.
  10. Milestones for Phases 0–11 and a Projects board.
  11. Phase 0 traffic and security labs, then the Phase 0 gate record in `SECURITY_LOG.md`.
