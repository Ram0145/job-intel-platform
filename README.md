# Job Intel Platform

An end-to-end learning project, built from scratch: web scraping → secure storage → data modeling → inverted index → Azure AI Search vectors → API → UI → AI agent and chatbot. Networking, security, traffic inspection and CI/CD are practiced at every step.

**Status:** Phase 0 (foundations) in progress.

| Document | What it is |
|---|---|
| [PLAN.md](PLAN.md) | The master plan: phases, tasks, progress, decisions |
| [SECURITY_LOG.md](SECURITY_LOG.md) | Threat model, findings, per-phase security & traffic gate records |
| [CLAUDE.md](CLAUDE.md) | Context and working rules for Claude Code sessions |

## Stack (planned)

Python 3.12 · httpx · PostgreSQL 16 · FastAPI · Nginx · React + TypeScript · sentence-transformers · Azure AI Search · GitHub Models · Docker Compose · GitHub Actions · GitLab CI

## Data

Only publicly published job postings from companies' own job boards, collected politely: `robots.txt` respected, rate-limited, honestly identified. No sites behind a login.
