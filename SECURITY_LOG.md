# Security Log

The record of **what we tested, what we saw on the wire, what we found, and how we fixed it**, phase by phase. `PLAN.md` says what to do; this file proves it was done.

**Rules for this file**
- Every phase gets a Gate Record (Section 5), filled in before the phase is marked DONE.
- Every finding gets an ID in the register (Section 3), even small ones. Fixed findings stay listed; that history is the point.
- Never paste a real secret, token, password or personal data here, even a "test" one. Describe it instead.

---

## 1. Severity scale

| Severity | Meaning | Example | Gate rule |
|---|---|---|---|
| Critical | Remote compromise, data loss, or secret exposure with no preconditions | SQL injection on a public endpoint; leaked cloud key | Must fix before continuing |
| High | Serious impact with minor preconditions | Any logged-in user can read others' data (BOLA) | Must fix before the phase is done |
| Medium | Real weakness, limited impact or harder to exploit | Missing rate limit on login; verbose error messages | Fix, or accept with a written reason |
| Low | Hardening gap | A missing security header on a non-sensitive page | Track; fix when convenient |
| Info | Observation, no direct risk | Server version visible in a header | Note only |

---

## 2. System threat model

*Initial version 2026-11-06. Expanded in Phase 2 and updated by every phase's Gate Record (check A).*

### 2.1 Assets

| Asset | Classification | Where it lives |
|---|---|---|
| Scraped job postings | Public | Raw files, Postgres, Azure AI Search |
| User accounts & password hashes | Confidential | Postgres |
| Resumes | **Personal data — restricted** | Postgres only; redacted before any LLM call |
| API keys, DB passwords, signing secrets | Secret | `.env` (local), GitHub Environments, Azure Key Vault |
| Source code & CI/CD workflows | Public (integrity matters) | GitHub, GitLab |
| Deploy credentials | Secret | None long-lived (OIDC) |

### 2.2 Actors

| Actor | Motivation | Capability |
|---|---|---|
| Malicious or compromised job board | Inject payloads into our data | Controls content we scrape |
| Anonymous internet user / bot | Data theft, abuse, resource use | Sends any HTTP request to the public app |
| Logged-in user | Access other users' data, escalate role | Valid account and token |
| Prompt injector | Hijack the agent | Text inside postings, resumes or chat messages |
| Supply-chain attacker | Run code in our CI or app | Malicious dependency, image or GitHub Action |
| You, by accident | (none) | Committing a secret, opening a port, misconfiguring a rule |

### 2.3 Trust boundaries

See `PLAN.md` Section 4: TB1–TB7.

### 2.4 Top threats & planned mitigations

| ID | Threat | STRIDE | Boundary | Planned mitigation | Phase |
|---|---|---|---|---|---|
| T-01 | SSRF through URLs discovered while crawling | Information disclosure / Elevation | TB1 | Scheme allowlist, IP blocklist after DNS resolution, re-check on redirect | 1 |
| T-02 | Stored XSS from scraped HTML | Tampering | TB1 → TB2 | `nh3` server-side + DOMPurify client-side + strict CSP | 1, 8 |
| T-03 | SQL injection (direct and second-order) | Tampering / Information disclosure | TB2, TB3 | Bind parameters, allowlists, read-only roles | 1, 4, 5, 7, 9 |
| T-04 | Secrets leaked in git, images, logs or CI | Information disclosure | TB7 | gitleaks, push protection, multi-stage builds, log redaction, OIDC | 0, 2, 3, 6 |
| T-05 | Database reachable from outside its network | Information disclosure | TB3 | Internal Docker network, `127.0.0.1` binding, private subnet in Azure | 3, 10 |
| T-06 | Plaintext database traffic | Information disclosure | TB3 | Postgres TLS with `verify-full` | 4 |
| T-07 | Broken object-level authorization | Elevation of privilege | TB2 | Per-object checks, RLS, authorization matrix in CI | 7 |
| T-08 | Token forgery / weak JWTs | Spoofing | TB2 | Strong secret, fixed algorithm, short expiry, refresh in HttpOnly cookie | 7 |
| T-09 | Resource exhaustion (huge queries, scraping loops, LLM cost) | Denial of service | TB1, TB2, TB5 | Size limits, timeouts, rate limits, step/token caps | 1, 5, 7, 9 |
| T-10 | Indirect prompt injection through postings | Tampering / Elevation | TB6 | Untrusted-data labeling, tool authorization as the user, confirmation for writes | 9 |
| T-11 | Personal data sent to the LLM provider | Information disclosure | TB5 | Redaction before calls; egress logging | 9 |
| T-12 | Poisoned CI workflow or dependency | Tampering / Elevation | TB7 | SHA-pinned actions, least-privilege `GITHUB_TOKEN`, Dependabot, CODEOWNERS on workflows | 0, 2 |
| T-13 | Internet-facing VM compromise | Elevation of privilege | TB2 | NSGs, key-only SSH from one IP, fail2ban, patching | 10 |

---

## 3. Findings register

| ID | Phase | Title | Severity | Status | Found by | Fixed in (PR / commit) | Notes |
|---|---|---|---|---|---|---|---|
| — | — | *No findings yet* | — | — | — | — | — |

Status values: `Open` · `Fixing` · `Fixed` · `Accepted` · `False positive`.
ID format: `F-<phase>-<number>`, e.g. `F-1-03`.

---

## 4. Accepted risks

| Finding ID | Risk | Why accepted | Review date | Upgrade path |
|---|---|---|---|---|
| — | *None yet* | — | — | — |

---

## 5. Gate records

### Template (copy for each phase)

```markdown
### Phase N — <name> · Gate record · <date>

**A. Threat model delta**
- New components / data flows:
- What an attacker can now control:
- STRIDE notes:
- Trust boundary table updated? yes / no

**B. Secrets**
- [ ] gitleaks full-history scan clean
- [ ] No secrets in logs, images, URLs, CI output
- [ ] `.env.example` updated

**C. Dependencies & images**
- [ ] pip-audit / npm audit clean (or findings logged)
- [ ] Trivy clean on new images (Phase 3+)

**D. Static analysis**
- [ ] bandit / semgrep / CodeQL reviewed — Highs fixed, Mediums fixed or accepted

**E. Injection checks**
| Interpreter (SQL / shell / HTML / regex / LLM) | Location | Test performed | Result |
|---|---|---|---|

**F. Input & output**
- Boundaries validated:
- Size / time limits:
- Output encoding:

**G. Traffic notes**
| What I captured | Tool | Plaintext or encrypted? | Ports / interfaces | What left my machine, to whom |
|---|---|---|---|---|

What I learned from looking at the traffic:

**H. Least privilege**
- DB roles / keys / container users / `GITHUB_TOKEN` permissions checked:

**I. Logging**
- Security events logged:
- Confirmed nothing sensitive is logged:

**J. Findings**
- New finding IDs:

**Gate result:** PASS / FAIL — reason:
```

---

### Phase 0 — Foundations: Git, GitHub & CI · Gate record · *pending*

*To be filled when Phase 0 finishes.*

---

## 6. Lab notes

Step-by-step write-ups of the hands-on attack/defense labs (SQL injection playground, malicious job board, XSS, JWT forgery, prompt injection, …). Each lab: goal, setup, attack steps, what happened, the fix, proof the fix works.

*No labs yet.*

---

## 7. Final penetration test report (Phase 11)

*Executive summary, scope, methodology, findings, fixes, residual risk — written in Phase 11.*
