# PoC Implementation Plan — Hardware Service Decision Copilot

**Date:** 2026-10-21
**Status:** Proposed for orchestration (orchestrator does not implement; specialized agents implement)
**Binding inputs:** `docs/PRD.md` (with revisions R1–R6), `docs/ADR/000` to `005`, `docs/design-guidelines.md` (neutral token use, see section 2), `AGENTS.md`

---

## 1. Decisions taken for this plan

| # | Decision | Source |
|---|---|---|
| D1 | Scope: all acceptance criteria AC-01 to AC-36 | User answer |
| D2 | Base: `vercel/chatbot` template reworked inside `app/` | User answer, ADR-005 |
| D3 | Model: `openai/gpt-6-luna` via OpenRouter (`LLM_MODEL`). Spike: HTTP 200, schema-valid output, damage detected; uses reasoning tokens (265 in the spike), so latency must be measured against the 30 s limit. `xiaomi/mimo-v2.6-flash` was the first candidate and stays as fallback | User answer, spike test HTTP 200 |
| D4 | Brand: neutral tokens from `design-guidelines.md`; no x-kom logo or x-kom font | User answer; license note in the guideline |
| D5 | Agents: `fe-developer`, `be-developer`, `qa-engineer` in `.claude/agents/` | User answer |
| D6 | Commits: one local commit per completed TDD step, format `Area: summary`. No push from the agents. Push is done by the user with `! ` | User answer, AGENTS.md, sandbox policy |
| D7 | Decision response is complete JSON (no streaming). Chat replies stream. Revised decisions go through `reviseDecision` (ADR-002 §3) | ADR-002, PRD R1 |
| D8 | Rule documents are shipped as a committed copy in `app/rules/` with a drift test against `docs/examples/rules/` | ADR-002 §6 |
| D9 | Client-side timeout 30 s is binding; server budget 25 s (compression 5 s) | ADR-000 §6 |
| D10 | Photo released after successful analysis; no image bytes to the client | PRD AC-35, ADR-003 |
| D11 | Chat SDK not used now (OI-005-1 closed) | ADR-005 §4 |

**Assumption to confirm (A1):** the PoC font is a system sans-serif stack. Instrument Sans is x-kom's font under a licence that must be checked first (`design-guidelines.md`, Assets). Tokens for colours, radius and spacing come from `design-guidelines.md` and `assets/design-tokens.json`.

**Assumption to confirm (A2):** `docs/PRD.md` is the file the agents read. The user's name `docs/PRD-Product-Requirements-Document.md` does not exist in this repository.

---

## 2. Operating rules for every agent

Every task brief the orchestrator sends must contain the sections in section 6 (template). The following rules apply to all agents:

1. **TDD order.** Write the failing test, run it, confirm it fails for the expected reason, implement the minimum, run the scope's tests, refactor only while green. Record the failing output and the passing output in the report.
2. **Verification per commit.** Scope tests pass, `pnpm exec tsc --noEmit` passes, `pnpm check` (Ultracite) passes. No new warnings. Pre-existing failures are reported, not hidden.
3. **Commit.** One logical change per commit, format `Area: short summary` (e.g. `Backend: add decision post-validation`). Local only. Message ends with the Co-Authored-By line given in the session instructions. Never `git push`.
4. **No secret handling.** Do not read `app/.env.local`, do not print keys, do not write keys into tracked files. Tests that need the live model read the key through the runtime only and are run by the user (section 5, risk R1).
5. **Context7 first.** Before writing code against a library, query its current docs with Context7 (IDs in ADR-000 §2 and ADR-005 §2).
6. **File ownership.** Each agent edits only the paths in its brief. Shared paths have one owner (section 4).
7. **No questions to the orchestrator for details already in the brief.** If the brief is ambiguous, the agent reports the ambiguity and stops that step.
8. **Language.** Code and comments in English. UI strings and model replies in Polish (PRD AC-26, AC-33). Only Polish strings come from `app/lib/copy/pl.ts` (see S1.2).

---

## 3. Phases and steps

Legend: **Owner** = agent. **Depends on** = steps that must be committed first. **Par** = steps that may run in parallel with this one.

### Phase 0 — Foundation

| Step | Task | Owner | Depends on | Par | Output / verification |
|---|---|---|---|---|---|
| S0.1 | Create agent definitions `fe-developer`, `be-developer`, `qa-engineer` in `.claude/agents/`, each short (role, paths owned, rules from section 2) | Orchestrator (definition files only) | — | — | Three files; Claude Code lists them. Commit `Agents: add PoC subagent definitions` |
| S0.2 | Test tooling: Vitest with `tests/unit/**` and `tests/integration/**`; Testing Library + jsdom for components; Playwright config without mock mode; scripts `test:unit`, `test:integration`, `test:e2e` | be-developer | S0.1 | — | Each script reports discovery count (a smoke test: red, then green). Commit `Backend: add unit and integration test runner` |
| S0.3 | Remove the mock model path: `isTestEnvironment`, `lib/ai/models.mock.ts`, `lib/ai/models.test.ts`, `PLAYWRIGHT` switch | be-developer | S0.2 | S0.4 | Test that the provider module has no test branch (red, then green). Commit `Backend: remove mock model path` |
| S0.4 | Design tokens into the Tailwind theme (colours, radius, 4 px spacing from `assets/design-tokens.json`); system font per A1 | fe-developer | S0.2 | S0.3 | Snapshot of token values in a unit test (red, then green). Commit `Frontend: apply neutral brand tokens` |
| S0.5 | Remove auth: `proxy.ts`, `app/(auth)/**`, Auth.js config, guest login, `AUTH_SECRET` use | be-developer | S0.3 | S0.6 | Build and tsc pass; test that no auth route exists (red, then green). Commit `Backend: remove authentication` |
| S0.6 | Remove database and history: `lib/db/**`, Drizzle config, `db:*` scripts, history routes, sidebar history (UI part by fe-developer in S0.7) | be-developer | S0.5 | — | `build` runs without migration step. Commit `Backend: remove database and chat history` |
| S0.7 | Remove UI features not in the PRD: artifacts, document tools, weather tool, suggestions, model selector, chat attachments, sidebar history, analytics components | fe-developer | S0.6 | — | Chat input accepts text only (AC-24); tests for the removed component list (red, then green). Commit `Frontend: remove features outside PRD` |
| S0.8 | Remove infrastructure not in the PRD: Vercel Blob, Redis resumable streams, BotID, IP rate limit, otel instrumentation | be-developer | S0.6 | S0.7 | `package.json` has none of these packages; tsc passes. Commit `Backend: remove blob, redis, botid and otel` |
| S0.9 | Environment: `app/.env.example` with only `OPENROUTER_API_KEY` and `LLM_MODEL` (empty values); test that no other key is required | be-developer | S0.8 | — | Commit `Backend: reduce env example to course keys` |
| S0.10 | Baseline gate: `pnpm exec tsc --noEmit`, `pnpm check`, `pnpm exec next build`, unit and integration runners discover tests | qa-engineer | S0.9 | — | Report with each check separately (build, lint, tests, startup not run). No commit unless fixes are needed |

### Phase 1 — Shared contracts

| Step | Task | Owner | Depends on | Par | Output / verification |
|---|---|---|---|---|---|
| S1.1 | Zod schemas in `app/lib/schemas/`: CaseForm (AC-01 to AC-07), ImageDescription for both case types incl. `photoUsable` and `photoIssue` (ADR-002 §3), DecisionResult (AC-17, AC-19, AC-20), CaseContext, error codes (ADR-000 §5), chat message length (AC-24) | be-developer | S0.10 | S1.2 | One unit test per AC row (red, then green). Commit `Backend: add shared case and decision schemas` |
| S1.2 | Polish copy module `app/lib/copy/pl.ts` with every string from PRD §13 (validation, loading, errors, buttons, category labels, disclaimer, fallback ESCALATE texts) | fe-developer | S0.10 | S1.1 | Test: every key exists, no empty value, disclaimer exact text (AC-21). Commit `Frontend: add Polish UI copy module` |

### Phase 2 — Backend core

Sequential chain in `app/lib/server/**` (be-developer). Steps S2.1 to S2.3 may run in parallel with each other and with Phase 3 steps.

| Step | Task | Owner | Depends on | Par | Output / verification |
|---|---|---|---|---|---|
| S2.1 | `llm-client` (server-only): creates the OpenRouter provider from `OPENROUTER_API_KEY` and `LLM_MODEL`; throws a typed error when either is missing; single place that reads them | be-developer | S1.1 | S2.2, S2.3 | Unit tests with the provider factory mocked (red, then green). Context7: `@openrouter/ai-sdk-provider` `createOpenRouter`. Commit `Backend: add OpenRouter client` |
| S2.2 | Rules loader: reads `app/rules/complaint-rules.md` or `return-rules.md` by fixed case-type table; parses `R-<number>` rule headings; fails if file missing (ADR-002 §6). Copy the two files into `app/rules/`. Drift test that the copy equals `docs/examples/rules/` | be-developer | S1.1 | S2.1, S2.3 | Unit tests (red, then green); drift test is a unit test reading both files. Commit `Backend: add rule document loader` |
| S2.3 | Photo preparation: content-signature check for JPG, PNG, WEBP (AC-09); size ≤ 10 MB (AC-10); compression ≤ 1 MB with longest side ≥ 1024 px, else `UNREADABLE_PHOTO` (AC-11, OI-003-1); no disk write. Choose the image library after Context7 check (candidate: `sharp`) | be-developer | S1.1 | S2.1, S2.2 | Unit tests for checks; integration tests with real image fixtures (generated in test setup, not mocked). Commit `Backend: add photo validation and compression` |
| S2.4 | Analysis module: prompt per case type only (AC-14); structured output ImageDescription; outcome mapping (ADR-002 §3 table); `photoUsable` handling | be-developer | S2.1, S2.2, S2.3 | — | Unit tests of prompt selection and outcome mapping (red, then green). Integration test replays a **captured** real response (see R1). Commit `Backend: add image analysis module` |
| S2.5 | Decision module: decision prompt with case-type rules only (AC-16); complete JSON output; post-validation (ADR-002 §7 table); composition order: greeting, bold category in sentence 2 (AC-18), justification, rule references, next steps 2 to 5 (AC-20), disclaimer (AC-21); ESCALATE fallback texts from PRD §13 | be-developer | S2.4 | — | Unit tests, one per row of ADR-002 §7 table (red, then green). Commit `Backend: add decision module with post-validation` |
| S2.6 | Route `POST /api/cases/analyze` (multipart, JSON response, no image bytes, 25 s server budget) | be-developer | S2.4, S0.5 | — | Integration tests with captured responses; TAC-003-07 (no image data). Commit `Backend: add analyze route` |
| S2.7 | Route `POST /api/cases/decision` (JSON, composed text plus DecisionResult, CaseContext returned for chat) | be-developer | S2.5 | S2.6 | Integration tests (captured responses); fallback ESCALATE path. Commit `Backend: add decision route` |
| S2.8 | Route `POST /api/chat` (rewritten from the template chat route): full messages and CaseContext, rebuilt context on server, streaming replies, `reviseDecision` tool with post-validation, disclaimer appended when a reply states a category without the tool (ADR-002 §3), message length 1 to 2000 (AC-24) | be-developer | S2.5, S0.6 | — | Integration tests (captured stream); test that the rule document of the other case type never appears (TAC-002-02, chat route included). Commit `Backend: rework chat route for PoC` |
| S2.9 | Remove template chat code that is no longer used: chat history reads, `prepareSendMessagesRequest` backend parts, Guest, unused tools | be-developer | S2.8 | — | tsc and build pass; no dead route. Commit `Backend: remove unused chat code` |

### Phase 3 — Frontend

Owner: fe-developer. Uses `app/lib/schemas/` and `app/lib/copy/pl.ts` from Phase 1 and mocks `fetch` in unit tests (allowed: unit mocks all dependencies).

| Step | Task | Owner | Depends on | Par | Output / verification |
|---|---|---|---|---|---|
| S3.1 | Case form: fields in order (category, model name, purchase date, reason, photo, Submit); case-type selector changes reason required state (AC-05, AC-06); validation messages (AC-03 to AC-05); Submit disabled rules (AC-08) | fe-developer | S1.1, S1.2, S0.7 | S2.x | Component tests (red, then green). Commit `Frontend: add case form` |
| S3.2 | Photo field: exactly one file (AC-07), type and size messages (AC-09, AC-10), thumbnail, file name, Remove | fe-developer | S3.1 | — | Component tests. Commit `Frontend: add photo field` |
| S3.3 | Page state machine: form, analyzing, deciding, analysisFailed, decisionFailed, unreadable, chat (ADR-004 §7.2); photo released after analysis success (D10); Cancel aborts; 30 s client timeout (D9) | fe-developer | S3.2, S1.2 | S2.x | State transition unit tests with mocked fetch (red, then green). Commit `Frontend: add case state machine` |
| S3.4 | Loading and failure views: texts from PRD §9.2, §9.3, §9.4; Try again repeats only the failed step (PRD 4.5) | fe-developer | S3.3 | — | Component tests. Commit `Frontend: add loading and failure views` |
| S3.5 | Chat screen: AI Elements `Conversation`, `Message`, `MessageResponse`, `PromptInput` (text only); `DecisionMessage` pinned as first item and not removable (PRD 9.5); timestamps (AC-29); typing indicator; Send disabled while pending (PRD 9.5); Retry under failed user message (AC-25); input 1 to 2000 (AC-24) | fe-developer | S3.3, S2.8 contract | — | Component tests with mocked `useChat` transport; test that no decision text renders before validated response (TAC-004-08). Commit `Frontend: add chat screen` |
| S3.6 | Browser history: one history entry for the chat step with a step marker only, no data in URL (ADR-004 §3) | fe-developer | S3.5 | — | Unit test on history handling. Commit `Frontend: add chat history step marker` |
| S3.7 | New case and discard dialog (AC-32), reload resets to empty form (AC-31), no localStorage or sessionStorage for session data (TAC-004-03) | fe-developer | S3.5 | — | Unit tests; grep test. Commit `Frontend: add new case and discard dialog` |
| S3.8 | Layout: 360 px to 1440 px without horizontal scroll (AC-34); tokens applied; Polish text everywhere (AC-33) | fe-developer | S3.7 | — | Component test of layout classes is weak; the real check is in S4.4. Commit `Frontend: responsive layout` |

### Phase 4 — End-to-end and Manual QA

Owner: qa-engineer. E2E uses no mocks (AGENTS.md). Only the browser may be taken offline, and only in dedicated failure specs (ADR-004 §8).

| Step | Task | Owner | Depends on | Par | Output / verification |
|---|---|---|---|---|---|
| S4.1 | Playwright setup for the app: projects Chromium desktop and mobile viewport 360 px; web server started with `pnpm dev`; synthetic image fixtures in `tests/fixtures/images/` (clear and blank) | qa-engineer | S3.8, S2.9 | — | Discovery count reported. Commit `QA: add Playwright project and image fixtures` |
| S4.2 | E2E complaint happy path: form, analysis, decision, chat with first message (AC-18 to AC-23) | qa-engineer | S4.1 | S4.3 | Red until implementation, then green. Real model call (R1). Commit `QA: add complaint happy path E2E` |
| S4.3 | E2E return happy path, unreadable photo (form kept, photo cleared), follow-up reply, new case with confirmation, reload resets | qa-engineer | S4.1 | S4.2 | Commit `QA: add return, unreadable photo and chat E2E` |
| S4.4 | Manual QA with Playwright CLI: screenshots of every touched screen at 360 px and 1440 px, console error check, comparison with `assets/homepage.png` tokens (colours, spacing, buttons) | qa-engineer | S4.2, S4.3 | — | QA report in `docs/plans/` with screenshot paths. No commit of screenshots unless the user asks |
| S4.5 | Failure E2E: offline browser for analysis failure and Try again; invalid-key project for decision failure (ADR-004 §8) | qa-engineer | S4.3 | — | Commit `QA: add failure path E2E` |

### Phase 5 — Release readiness

| Step | Task | Owner | Depends on | Output |
|---|---|---|---|---|
| S5.1 | `app/README.md` (course file) is not overwritten; setup notes go to `app/TEMPLATE_README.md` or a new `app/SETUP.md` | be-developer | S4.5 | Commit `Docs: add PoC setup notes` |
| S5.2 | Final gate: tsc, Ultracite, unit, integration, E2E, build, startup check with recorded results separately | qa-engineer | S5.1 | Report; orchestrator decides on the commit range |

---

## 4. Ownership and conflict matrix

| Path | Owner | Notes |
|---|---|---|
| `.claude/agents/*` | Orchestrator | Definitions only |
| `app/lib/schemas/**` | be-developer | fe-developer reads only |
| `app/lib/copy/**` | fe-developer | be-developer reads only (fallback texts go through the copy module) |
| `app/lib/server/**`, `app/rules/**`, `app/app/api/**` | be-developer | |
| `app/app/**` (pages), `app/components/**`, `app/hooks/**` | fe-developer | Route handlers under `app/app/api/` are be-developer |
| `app/tailwind*`, `app/app/globals.css` | fe-developer | |
| `app/tests/unit/**`, `app/tests/integration/**` | Owner of the code under test | |
| `app/tests/e2e/**`, `app/playwright.config.ts`, `app/tests/fixtures/images/**` | qa-engineer | |
| `app/package.json` | be-developer | Other agents request additions through the orchestrator |
| `docs/**` | Orchestrator | Only plan and QA reports; ADR and PRD changes are not made by agents |

**Rule:** two agents never edit the same file in the same time window. The orchestrator starts parallel steps only when their paths do not overlap.

---

## 5. Dependency matrix (summary)

| Step | Depends on | Blocks |
|---|---|---|
| S0.1 | — | S0.2 |
| S0.2 | S0.1 | S0.3, S0.4 |
| S0.3 | S0.2 | S0.5 |
| S0.4 | S0.2 | S3.x (styles) |
| S0.5 | S0.3 | S0.6, S2.6 |
| S0.6 | S0.5 | S0.7, S0.8, S2.8 |
| S0.7 | S0.6 | S1.x UI parts, S3.1 |
| S0.8 | S0.6 | S0.9 |
| S0.9 | S0.8 | S0.10 |
| S0.10 | S0.9 | S1.1, S1.2 |
| S1.1 | S0.10 | S2.1, S2.2, S2.3, S3.1 |
| S1.2 | S0.10 | S3.1, S3.3 |
| S2.1–S2.3 | S1.1 | S2.4 |
| S2.4 | S2.1–S2.3 | S2.5, S2.6 |
| S2.5 | S2.4 | S2.7, S2.8 |
| S2.6–S2.8 | S2.4 / S2.5 | S2.9, S4.x |
| S3.1 → S3.8 | S1.x, S0.7 | S4.x |
| S4.1 | S3.8, S2.9 | S4.2–S4.5 |
| S5.x | S4.5 | — |

**Parallel windows:**
- Window A: S0.3 (be) || S0.4 (fe) after S0.2.
- Window B: S2.1, S2.2, S2.3 (be) || S1.2 (fe) after S1.1 / S0.10.
- Window C: S3.1 to S3.4 (fe) || S2.4 to S2.7 (be).
- Window D: S4.2 || S4.3 (qa), after both agents finish their Phase 2 and 3 steps.

---

## 6. Brief template for every subagent task

The orchestrator fills this for each step. Only the lines that apply to the step are used.

```
Task ID: <S-x.y>
Agent: <fe-developer | be-developer | qa-engineer>
Goal: <one sentence>
Read first (only these):
  - PRD: <section and AC numbers>
  - ADR: <file and section>
  - Design: <token names or section of design-guidelines.md>
Paths you may edit: <list>
Paths you must not edit: <list>
Depends on (already committed): <step IDs>
Current state: <branch main, last commit hash, what exists>
Steps:
  1. Write failing tests for: <list of behaviours with AC numbers>
  2. Run <command>; expect failure for reason: <reason>
  3. Implement minimum
  4. Run <scope command>; all green
  5. Run tsc and Ultracite; no new warnings
  6. Commit: "<Area>: <summary>" with the Co-Authored-By line
Output report: failing output, passing output, discovery count, commit hash, open questions (no questions in the middle of the step)
Do not: push, read app/.env.local, print keys, edit other owners' files
```

---

## 7. Risks and blockers

| ID | Risk | Effect | Mitigation |
|---|---|---|---|
| R1 | Live model tests need `OPENROUTER_API_KEY`. Agents are denied reading `app/.env.local`. | Capture of real responses (TAC-002-05) and E2E with the real model cannot run from agents | Captures and E2E are run by the user with `! ` or from a terminal where the key is set; agents consume the stored captures. Agents never read the key file |
| R2 | Sandbox classifier blocks some actions: git push, settings edits (self-modification), and possibly data-related commands | Steps stop | Agents report the block and stop; the user decides. Commits stay local |
| R3 | Template removal (S0.5 to S0.9) breaks build in the middle | Working state lost | One removal per commit; tsc and build must pass before each commit; if not, the step is split further |
| R4 | `openai/gpt-6-luna` may exceed the 30 s limit (reasoning tokens) or be weak in Polish on real photos | Timeouts (AC-25, section 4.5) and decision quality | Measure latency per request in S4.2 and S4.3 with synthetic cases; if too slow, fallback to `xiaomi/mimo-v2.6-flash` by changing `LLM_MODEL` only |
| R5 | Image library choice (S2.3) and Vercel body limit (OI-000-1) | Upload size and compression | Context7 check before S2.3; body limit check before deployment, not blocking for local PoC |
| R6 | Retention of data at OpenRouter (OI-000-2) | Real customer data not allowed | Synthetic data only (PRD R6) |

---

## 8. Open items for the orchestrator

- **A1** Font decision (system stack assumed).
- **A2** PRD file name in the user's brief (`docs/PRD.md` used).
- **O1** Who runs live captures and E2E with the real model (user with `! ` assumed).
- **O2** Permission to create `.claude/agents/*.md` may be blocked by the sandbox (self-modification rule). If blocked, the user creates the three files from the templates in section 6 of this plan.
