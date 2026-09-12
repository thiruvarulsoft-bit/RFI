# DUTAM-RFI — Model Selection & Session-Start Prompt Pack

**Owner:** Annadurai MA, RHCE — Software Owner / Product Owner
**Project:** DUTAM-RFI — Customer Clarification & Engineering Knowledge System
**Baseline:** v0.16.0 · 468 tests · 107 endpoints · 41 tables · 9 migrations
**Deployment target:** Docker on port 8000, behind Apache HTTPD at `http://192.168.1.50/rfi/`
**Plan:** Claude Pro — one new chat per session/module

---

## 1. How many models

You do not need a different model per module. You need **three models rotated by SDLC phase**:

| Model | Role in this project | Cost against your Pro limit |
|---|---|---|
| **Claude Opus 5** | Thinking phases: SRS, architecture, data model, security, hard root-cause | Highest — reserve it |
| **Claude Sonnet 5** | Building phases: schema, API, React frontend, tests, packaging | Moderate — your workhorse |
| **Claude Haiku 4.5** | Mechanical phases: log triage, RTM updates, doc formatting, repeat runs | Lightest — stretches your limit |

A fourth model, **Claude Fable 5.1**, sits on the Mythos tier. It is not required for this project and some queries on it are routed to Opus 5 by safeguards. Ignore it for DUTAM-RFI planning.

**Rule of thumb:** Opus decides *what* to build. Sonnet builds it. Haiku cleans up after it.

---

## 2. Phase-to-model map (remaining work)

| # | SDLC phase | Model | Output of the session |
|---|---|---|---|
| P1 | SRS v1.1 — RFI functional requirements | Opus 5 | Approved SRS + RTM rows |
| P2 | Architecture & API contract | Opus 5 | ADR + endpoint contract + auth matrix |
| P3 | Database schema + migrations | Sonnet 5 | Migration files + rollback + schema tests |
| P4 | Backend API implementation | Sonnet 5 | Express routes + unit/integration tests |
| P5 | Frontend implementation (React/TS/Vite) | Sonnet 5 | Built UI served under `/rfi/` |
| P6 | Test execution — API/UI/E2E | Sonnet 5 | Test run report + defect list |
| P7 | Defect fix + regression | Sonnet 5 | Fixes + full regression pass |
| P8 | Security review | Opus 5 | Threat model + fail-closed verification |
| P9 | Deployment package + Apache/Docker | Sonnet 5 | DeploymentPackage tar.gz |
| P10 | Documentation, RTM, release notes | Haiku 4.5 | Docs + honest RTM status |
| P11 | Hard defect — root cause unknown | Opus 5 | Root cause + fix + regression |

**Usage pacing on Pro:** run Opus phases early in your usage window when you have headroom. If you hit the limit mid-session, the session package is still your recovery point — that is why every session must end with a package.

---

## 3. Standard header (paste at the top of EVERY session prompt)

```
ROLE: You are my complete autonomous engineering team for DUTAM-RFI —
architect, full-stack developer, QA engineer, security engineer, DevOps
engineer, code reviewer, release engineer. I am the Software Owner and
Product Owner and the sole decision authority.

PROJECT: DUTAM-RFI — AI Engineering Knowledge & RFI Automation System,
DUTAM Engineering Services.

STACK (do not substitute):
- Backend: Node.js / Express
- Database: PostgreSQL + pgvector
- AI runtime: Ollama, local only, no internet retrieval
- Frontend: React + TypeScript + Vite
- Web server: Apache HTTPD only (HTTP on LAN, no TLS, no Nginx)
- Auth: Argon2id
- Deployment: Docker on port 8000 behind Apache at http://192.168.1.50/rfi/

NON-NEGOTIABLE CONSTRAINTS:
- Fail-closed security. No permissive fallbacks. Access matrix as data.
- No hard-coded secrets. Use .env.example templates only.
- Honest RTM. A blocked requirement stays BLOCKED with the blocker named.
  Never report a requirement as done when it is not. "It starts" is not
  "it works."
- Bounded resource usage on OCR and other heavy operations.

WORKING METHOD:
- Inspect the repository BEFORE writing any code.
- Work autonomously. On failure: INVESTIGATE -> ROOT CAUSE -> FIX ->
  RETEST -> REGRESSION -> CONTINUE.
- Interrupt me only for: contradictory requirements, missing business
  information, unavailable credentials, destructive production actions,
  major architecture decisions, or security decisions needing owner
  approval.
- One topic per response. Do not bundle unrelated items.

END OF SESSION, DELIVER:
1. CompletePackage.<YYYY-MM-DD>.tar.gz — full source + prompts + docs
2. DeploymentPackage.<YYYY-MM-DD>.tar.gz — for my manual install
3. Updated RTM with honest status
4. The copy-paste prompt for the next session
```

---

## 4. Session-start prompts

### P1 — SRS v1.1 (model: **Opus 5**)

```
[PASTE STANDARD HEADER]

SESSION: P1 — SRS v1.1, RFI functional requirements.

CONTEXT: RFI functionality has not been invented yet. It is blocked
pending SRS v1.1. Everything else through v0.16.0 is built.

TASK:
1. Read the existing repository and current SRS baseline.
2. Draft SRS v1.1 covering the RFI lifecycle end to end: raise, route,
   clarify, respond, approve, close, reopen, audit.
3. Define roles and the authorization matrix as DATA, not code.
4. Define the RFI data model at requirement level, not schema level.
5. List every assumption you had to make, as a numbered decision list
   for my approval. Do NOT invent business rules silently.
6. Produce RTM rows for every new requirement, all initially NOT STARTED.

DO NOT write any implementation code in this session.
```

### P2 — Architecture & API contract (model: **Opus 5**)

```
[PASTE STANDARD HEADER]

SESSION: P2 — Architecture and API contract for the RFI module.

INPUT: Approved SRS v1.1 from session P1.

TASK:
1. Produce an ADR for the RFI module: options considered, decision,
   trade-offs, consequences.
2. Define the full REST API contract: path, method, request, response,
   error codes, required role. Extend the existing 107 endpoints —
   do not redesign what already works.
3. Define the authorization policy entries for every new endpoint,
   fail-closed by default.
4. Define the state machine for an RFI, including illegal transitions.
5. Map every SRS v1.1 requirement to an endpoint or explain why it has
   none. Update the RTM.

DO NOT write implementation code. Design artifacts only.
```

### P3 — Database schema & migrations (model: **Sonnet 5**)

```
[PASTE STANDARD HEADER]

SESSION: P3 — Database schema and migrations for the RFI module.

INPUT: Approved ADR and API contract from session P2.
CURRENT: 41 tables, 9 migrations, PostgreSQL + pgvector.

TASK:
1. Inspect existing migrations before writing migration 10.
2. Write forward migrations AND rollback scripts for every change.
3. Add indexes justified by the actual query patterns in the API
   contract — state the query each index serves.
4. Write schema-level tests: constraints, foreign keys, cascade
   behaviour, illegal state transitions rejected at DB level.
5. Run the full existing test suite. Report the before/after counts.
6. Update the RTM.

Report any requirement that cannot be satisfied by the schema and name
the blocker.
```

### P4 — Backend API implementation (model: **Sonnet 5**)

```
[PASTE STANDARD HEADER]

SESSION: P4 — Backend API implementation for the RFI module.

INPUT: Schema from P3, API contract from P2.

TASK:
1. Inspect the repository. Follow the existing Express route, service
   and error-handling patterns exactly. Do not introduce a new style.
2. Implement every endpoint in the contract.
3. Enforce authorization through the existing policy engine. No inline
   role checks. No permissive fallback.
4. Validate every input at the boundary. Reject unknown fields.
5. Write unit AND integration tests for each endpoint including the
   403 and 422 paths, not just the happy path.
6. Run the full suite. Report pass/fail honestly with counts.
7. Update the RTM.
```

### P5 — Frontend implementation (model: **Sonnet 5**)

```
[PASTE STANDARD HEADER]

SESSION: P5 — DUTAM-RFI employee web application, RFI screens.

TARGET: React + TypeScript + Vite, served at http://192.168.1.50/rfi/
behind Apache. The Vite base path must match the /rfi/ subpath.

TASK:
1. Inspect the existing Session 17 frontend before adding anything.
2. Build the RFI screens: list, detail, raise, respond, approve, audit
   trail. Match the existing component and styling conventions.
3. Wire to the P4 API. Handle loading, empty, error and 403 states —
   a 403 must show a clear denial, not a blank screen.
4. Hide actions the user's role cannot perform, but rely on the server
   as the real gate.
5. Add component tests and at least one E2E flow per screen.
6. Produce a production build and confirm it loads under /rfi/.
7. Update the RTM.
```

### P6 — Test execution (model: **Sonnet 5**)

```
[PASTE STANDARD HEADER]

SESSION: P6 — Full test execution pass.

TASK:
1. Run every suite: unit, integration, API, UI, E2E, regression.
2. Report actual counts: total, passed, failed, skipped. Do not round
   or summarise away failures.
3. For each failure: file, test name, expected vs actual, and your
   first hypothesis. Do NOT fix anything in this session.
4. Produce a defect list ranked by severity.
5. Flag any requirement marked DONE in the RTM that has no test
   proving it. That is a false completion — correct the RTM.
```

### P7 — Defect fix & regression (model: **Sonnet 5**)

```
[PASTE STANDARD HEADER]

SESSION: P7 — Defect fixes from the P6 defect list.

TASK: For each defect, in severity order:
INVESTIGATE -> ROOT CAUSE -> FIX -> RETEST -> REGRESSION -> CONTINUE.

RULES:
- State the root cause before the fix. A fix without a stated root
  cause is not accepted.
- After every fix, run the FULL regression suite, not just the one test.
- If a fix breaks something else, say so and stop for my decision.
- If a defect cannot be fixed, mark it BLOCKED and name the blocker.

Deliver a before/after test count and an updated RTM.
```

### P8 — Security review (model: **Opus 5**)

```
[PASTE STANDARD HEADER]

SESSION: P8 — Security review of the RFI module.

TASK:
1. Threat model the RFI module: assets, actors, trust boundaries,
   attack paths.
2. Verify fail-closed behaviour: attempt to reach every endpoint with
   no role, wrong role and expired session. Every one must deny.
3. Audit authorization data for gaps — any endpoint with no policy
   entry is a finding.
4. Review input validation, file upload handling, SVG sanitizer
   allow-list, and OCR resource bounds.
5. Run a secret scan across the whole repository. Any hard-coded
   secret is a release blocker.
6. Verify Argon2id parameters and session handling.
7. Produce findings ranked Critical / High / Medium / Low with a fix
   recommendation each.

Bring me any finding that needs an owner decision. Do not silently
accept risk.
```

### P9 — Deployment package (model: **Sonnet 5**)

```
[PASTE STANDARD HEADER]

SESSION: P9 — Deployment package for manual installation by me.

TARGET: Ubuntu server 192.168.1.50. Docker container on port 8000.
Apache HTTPD reverse proxy at /rfi/. Ollama already installed.
I perform the installation manually — you produce the package only.

TASK:
1. Produce Dockerfile and compose config, pinned versions, no :latest.
2. Produce the Apache vhost/proxy config for the /rfi/ subpath,
   including correct handling of the SPA fallback route.
3. Produce .env.example with every variable documented. No real
   secrets anywhere in the package.
4. Produce migration run order and rollback instructions.
5. Produce a step-by-step INSTALL.md I can follow on the console.
6. Produce a post-install smoke-test checklist with expected output
   for each step.
7. Run a secret scan before building the package.
8. Deliver DeploymentPackage.<date>.tar.gz and
   CompletePackage.<date>.tar.gz.
```

### P10 — Documentation & RTM (model: **Haiku 4.5**)

```
[PASTE STANDARD HEADER]

SESSION: P10 — Documentation and RTM consolidation.

TASK:
1. Update the RTM with the true status of every requirement. BLOCKED
   stays BLOCKED with the blocker named.
2. Write release notes for this version: added, changed, fixed, known
   issues, blocked items.
3. Update the API reference from the actual implemented routes, not
   from the design document.
4. Write the user guide for the RFI screens.
5. List every document that is now stale and needs rewriting.

Do not change any code in this session.
```

### P11 — Hard defect, root cause unknown (model: **Opus 5**)

```
[PASTE STANDARD HEADER]

SESSION: P11 — Root cause analysis.

SYMPTOM: <describe exactly what you observe>
EXPECTED: <what should happen>
ENVIRONMENT: <dev / deployed server>
WHAT I ALREADY TRIED: <list>

TASK:
1. Reproduce the fault deterministically. If you cannot reproduce it,
   say so — do not guess a fix.
2. Isolate the failing layer: Apache, Docker, Express, PostgreSQL,
   pgvector, Ollama, or frontend.
3. State the root cause in one sentence before proposing any fix.
4. Fix, retest, run the full regression suite.
5. Add a test that would have caught this.
```

---

## 5. How to run the cycle

1. Open a **new chat** for each phase.
2. Select the model from the table in section 2 before typing.
3. Paste the standard header, then the phase prompt.
4. Let the session run to completion without interrupting unless one of
   your six interrupt conditions is hit.
5. Collect both tar.gz packages and the next-session prompt.
6. Close the chat. Do not continue a phase in a chat that has already
   delivered its package.

Phases P3 to P7 will repeat per module. P1, P2, P8 run once per major
scope change. P9 and P10 run once per release.
