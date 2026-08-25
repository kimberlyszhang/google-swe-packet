# Coverage matrix — competency → project → file-level evidence

Maps every sub-competency in [categories.md](categories.md) to specific files on the `main` branch
of the five packet repos. Built by reading the files, not the READMEs' claims about them; every
path below was confirmed present on `origin/main` (or is flagged as a gap when it is not).

**Projects.** FINRA = [finra-multimodal-compliance](https://github.com/GPS-Demos/finra-multimodal-compliance) ·
DLA = [dla-gti](https://github.com/GPS-Demos/dla-gti) ·
ARMY = [army-artillery-ccc-capstone](https://github.com/GPS-Demos/army-artillery-ccc-capstone) ·
TRAVEL = [dod-travel-agent](https://github.com/GPS-Demos/dod-travel-agent) ·
LOGCAP = [logcap-dfac](https://github.com/GPS-Demos/logcap-dfac)

**Gap column.** `—` means the evidence is live on `main` today. Anything else names the repo change
required before the packet can cite it (see `repo-changes/`).

---

## 1. AI/ML Engineering

| Sub-competency | Primary | Secondary | Gap |
|---|---|---|---|
| Agentic & Multi-Agent Systems | **FINRA** — `packages/worker/src/stages/07-adjudicate.ts`: one agent per applicable 2210(d) clause, ≤6 in flight, each shown only its own clause text + ancestors + incorporated paragraphs; schema-rejected answers are re-asked with the validator's refusal, then recorded as `analysis_failed` rather than killing the run. Orchestration in `packages/worker/src/main.ts` (17 stage modules under 12 numbers). | **TRAVEL** — ADK `LlmAgent` with three function-callable tools (`dod_travel_agent/agent.py`, `tools/lookup_per_diem.py`, `search_flights.py`, `search_hotels.py`), exposed over A2A via `agent_card.json`. **DLA** — Gemini tool-calling registry, `infra/src/apis/dashboard-api/chat_tools/registry.py` (8 dispatchable tools). | — |
| Retrieval & Data Engineering (RAG) | **ARMY** — hybrid semantic + keyword search over Army doctrine PDFs: `backend/rag_search.py` (pgvector + full-text), `backend/pdf_processor.py` (overlap chunking with page/section provenance), `backend/embeddings.py` (`text-embedding-005`, batched), `backend/citations.py`, `backend/sql-gin-index/INDEX_OPTIMIZATION.md`. | **FINRA** — evidence-first retrieval over video: everything citable (`transcript_segment`, `onscreen_text`) is extracted and persisted *before* anything is judged; retrieval for adjudication reads stored evidence, never the raw video (`stages/05-build-evidence.ts`, `stages/09-cross-modal.ts`). | — |
| Model Selection & Tuning | **FINRA** — per-stage model + thinking-level tiering with written justification, all in `packages/shared/src/constants.ts`; the determinism boundary table in `README.md` states which stages may use a model at all. Structured output via `packages/shared/src/schemas.ts` + `worker/src/lib/generate-json.ts`. | **LOGCAP** — `functions/src/config/models.ts` (four roles, four models, env-overridable) and the phase split in `functions/src/generateMenu/` where Gemini picks and TypeScript solves (`phase4-solve.ts`, `deterministic.ts`). | — |
| LLM Ops and Evaluation | **FINRA** — three corpora with three separate answer keys under `eval/`: `generate-synthetic.ts` (authored ads + per-clause labels), `generate-variants.ts` (derived non-compliant variants), `blind-set-catalogue.ts` (26 real ads); readers `analyse-blind-set.ts` / `analyse-synthetic.ts` and the deploy gate `confirm-clean-controls.ts`. Measured results with recall/specificity/cost in `documentation/results.md`. Run provenance: `packages/worker/src/lib/prompt-bundle-hash.ts`, `run-provenance.ts`. | **DLA** — output-side quality/safety scanning of every Gemini response (`infra/src/common/ai_safety.py`) with regression tests (`tests/test_ai_summary_sanitization.py`). | FINRA blind-set numbers are `0.5.0` (2026-08-10) while the build is `0.11.1`; README already labels them "not re-measured since" — decide re-run vs. cite-as-dated. |
| Domain-Applied AI/ML Expertise | **FINRA** — FINRA Rule 2210(d) encoded as a clause registry with citation-incorporation semantics (`packages/shared/src/clauses.ts`); measured legibility (dwell, glyph height, required WPM) in `packages/shared/src/legibility.ts`. | **TRAVEL** — Joint Travel Regulations Q&A with mandatory section citations + GSA per-diem caps enforced deterministically. **ARMY** — MIL-STD-2525 symbology and slope-based traversability per unit type (`backend/terrain_traversability.py`). **DLA** — CVE/CVSS/EPSS + eMASS RMF/ATO scoring (`infra/src/common/score_engine.py`). | — |

## 2. Scoping and Documentation

| Sub-competency | Primary | Secondary | Gap |
|---|---|---|---|
| Problem Definition | **TRAVEL** — `_bmad-output/planning-artifacts/prd.md` (27 FRs across 7 capability areas, with the value proposition stated before any design). | **FINRA** — README opening states the backlog problem (hundreds of thousands of pending video ad approvals) and derives cost/throughput as product constraints from it. | — |
| Technical Scope & Constraints | **TRAVEL** — `architecture.md` § Technical Constraints & Dependencies: FedRAMP High as a hard deployment boundary, 1M-token context ceiling against a ~600-page JTR, per-integration rate/auth constraints. | **FINRA** — README § Architecture states the four deployable units and why the worker must be a Cloud Run job (ffmpeg is impossible on the managed Functions base image). | — |
| Stakeholder Alignment & Success Criteria | **TRAVEL** — 12 NFRs with numeric ceilings (30s JTR Q&A, 3s per diem, 10s flights) carried into `_bmad-output/implementation-artifacts/*.md` as per-story acceptance criteria; `epics.md` is the phased delivery plan (Phase 1a/1b). | **DLA** — `_bmad-output/implementation-artifacts/sprint-status.yaml` + ~90 story artifacts, each with its acceptance evidence. | — |
| System Design Artifacts | **DLA** — two Mermaid diagrams in `README.md` (system overview, data pipeline) rendering the real deployed topology. **FINRA** — Mermaid flowchart plus a "boundaries the diagram is asserting" section that states what the picture is *claiming*. | **ARMY** — system-context + frontend class diagrams in `README.md`; `MAP_IMPLEMENTATION_GUIDE.md`. **LOGCAP** — ASCII architecture + BigQuery ERD in `README.md`, expanded in `docs/TECHNICAL.md`. | — |
| **Decision Records** | **DLA** — five ADRs (chunking BigQuery writes by payload size; Vertex AI over a Developer API key; Memorystore-backed rate limiting that degrades open; generated OpenAPI with CI drift gate; deferring VPC-SC/CMEK with a written reversal condition), each with Status / Context / Decision / Alternatives rejected / Consequences / Reversal condition. | **FINRA** — decision-with-alternatives prose in README (`## Authentication` documents reversing an earlier "no auth needed" decision and why). | **BLOCKING.** The ADR commit `0346350` was rolled back by `203d044` on 2026-08-21 and is **not on `main`**. This is the only strong artifact for a named sub-competency. → `repo-changes/dla-gti.md` |
| API Documentation | **DLA** — `infra/src/apis/dashboard-api/openapi.yaml` generated from handler annotations by `openapi_spec.py`, with `.github/workflows/openapi-contract.yml` failing CI on spec drift, generated-client drift, or URL-registry drift. | **LOGCAP** — README § API Reference over 15 endpoints; `docs/TECHNICAL.md` § 3. **FINRA** — the ten-endpoint contract and its `{ data } / { error }` envelope in README. | — |
| Operational Documentation | **DLA** — ten runbooks under `documentation/runbooks/` (VPC-SC perimeter, CMEK enablement, Cloud Armor WAF, IAP front door, org-policy constraints, Chronicle workforce identity, monitoring alert policies, Gemini budget cap, gitleaks secret rotation, dependabot). | **FINRA** — README § Build and deploy + § Verification and evaluation (eight live re-runnable checks). **LOGCAP** — `docs/TECHNICAL.md` § 7 and § 9 (migration run order, partition-pruning verification query). | — |

## 3. Security, Privacy, and Compliance

| Sub-competency | Primary | Secondary | Gap |
|---|---|---|---|
| Authentication & Authorization | **FINRA** — `packages/api/src/with-handler.ts` verifies the Firebase ID token and checks the reviewer allowlist **before dispatch**, so the property holds for endpoints added later; `updated_by` is taken from the verified token, never the request body (`handlers/update-compliance-policy.ts`); live proof in `scripts/verify-auth.ts`. | **DLA** — auth-on-all-GETs, domain allowlist, token revocation and RBAC custom claims (stories 1-1…1-12), enforced by `tests/test_auth_gate.py`; per-service runtime service accounts in `infra/modules/iam/`. | — |
| Infrastructure & Network Security | **DLA** — `infra/modules/network/vpc.tf`, `infra/envs/dev/flow_logs.tf`, internal-only Cloud Run ingress (story 11-1), scoped run-invoker (11-8), additive-only IAM bindings (11-9), plus VPC-SC and Cloud Armor runbooks with the deferral written down. | **FINRA** — private videos bucket reachable only through V4 signed URLs with bounded TTLs (15 min upload / 1 h playback); bytes never transit a function. | — |
| Data Protection & Privacy | **DLA** — `infra/envs/dev/secrets.tf` + Secret Manager for the dashboard API key (story 1-8), gitleaks in CI *and* pre-commit pinned to one ruleset, CMEK runbook, deploy-bucket hardening (11-7). | **FINRA** — `packages/shared/src/genai-client.ts` is the only place a model client may be constructed and passes no API key; the `google-api-key` secret is retained but unused. **LOGCAP** — `functions/src/lib/secrets.ts`. | — |
| AI-Specific Security | **DLA** — `infra/src/common/ai_safety.py`: input-side sanitisation + sentinel wrapping of attacker-controllable CVE/alert text, **and** output-side injection-marker scanning that redacts before the response reaches the UI. Tests: `test_ai_summary_sanitization.py`. | **FINRA** — two citation gates (`stages/10-assert-verbatim.ts` deterministic verbatim match, `stages/11-verify-citations.ts` blind re-check) discard any finding a model could not ground. **LOGCAP** — `containsInjectionMarker` enforced at the HTTP boundary (`functions/src/lib/http.ts`). | — |
| Compliance & Governance | **DLA** — `infra/envs/dev/audit_log_sink.tf` with retention (story 12-4), eMASS RMF/POA&M modelling (`emass/EMASS.md`), Chronicle workforce-identity runbook. | **FINRA** — append-only versioned compliance policy: saving inserts, restoring inserts a copy, nothing updates or deletes, so who moved the bar and when is a `SELECT` (`packages/api/src/repositories/policy-repository.ts`). **TRAVEL** — FedRAMP-High fail-closed constraint documented in `architecture.md`. | — |

## 4. Reliability & Resilience

| Sub-competency | Primary | Secondary | Gap |
|---|---|---|---|
| Availability Design | **DLA** — scheduled ingestion with retry config (12-7), Cloud Run min-instances and no-CPU-throttling asserted in tests (`test_cloud_run_min_instances.py`), Memorystore-backed shared rate-limit state so limits hold across instances (1-4, 9-7). | **FINRA** — each submission is an independent Cloud Run job execution; concurrency is bounded client-side by a semaphore of three (`packages/frontend/src/hooks/use-upload-queue.ts`), and BigQuery DML contention under 13-way parallelism is documented with its measured safe operating point. | — |
| Observability | **DLA** — structured logging plus purpose-built metrics: campaign parse-failure metric (10-3), score-calculation metrics (10-5), suppression-free ingestion errors (10-6), Cloud Monitoring alert policies. | **LOGCAP** — `functions/src/lib/logger.ts` (structured fields, `requestId` per request). **TRAVEL** — `terraform/monitoring.tf` (5xx rate policy, scrape-job failure policy). **FINRA** — `runs.stage` published at every stage boundary; token/cost accounting per call site in `packages/shared/src/cost.ts`. | DLA's monitoring alert policies were reverted with `203d044` → `repo-changes/dla-gti.md` |
| Failure & Recovery Testing | **DLA** — `tests/load/sse-chat.k6.js` drives 50 concurrent SSE chat sessions against real Cloud Run with p95 TTFB < 1s / p95 full-stream < 30s / <1% error SLOs; concurrency + rate-limit tests (13-6); config-table snapshots and TF state `prevent_destroy` for restore (12-5, 12-6). | **FINRA** — `packages/api/scripts/verify-independent-runs.ts` and `verify-batch-progress.ts` prove behaviour under real concurrent load; `documentation/results.md` § 1 reports a serialization failure the sweep exposed *and* the reporting defect it uncovered, uncorrected numbers included. | — |
| Graceful Degradation | **FINRA** — failure policy differs by stage on purpose: evidence-spine failure (01–05) aborts the run, a verdict stage (07–09) records `insufficient_evidence` and continues, screening (06) fails **open**; retries in `packages/worker/src/lib/with-retry.ts`. | **TRAVEL** — per-dependency degradation matrix in `architecture.md` ("never hallucinate when an API is down — say unavailable"); on `kim-dev`, hotel search is **blocked** when the per-diem lookup fails rather than returning unfiltered results. **DLA** — GTI client 5xx retry (10-2), Gemini + handler timeouts (10-4). | The TRAVEL degradation code is on `kim-dev`, not `main` → `repo-changes/dod-travel-agent.md` |

## 5. Performance & Cost Optimization

| Sub-competency | Primary | Secondary | Gap |
|---|---|---|---|
| Scalability & Elasticity | **FINRA** — measured concurrency work: 13 parallel pipeline runs took the 26-ad sweep from ~2 hours to 23.5 minutes; the safe operating point is documented together with the BigQuery per-table DML contention that bounds it (README § Verification and evaluation). | **DLA** — Cloud Run autoscaling config in `infra/modules/apis/dashboard-api.tf`, batch ingestion split from the serving path so a 200k-CVE refresh never touches request latency. | — |
| Resource Efficiency | **ARMY** — `backend/sql-gin-index/INDEX_OPTIMIZATION.md` + `test_index_performance.py` (measured before/after), `backend/terrain/ELEVATION_GRID_OPTIMIZATION.md` and `VIEWPORT_ENHANCEMENT_SUMMARY.md` (grid sampling and viewport-bounded queries instead of whole-region analysis). | **LOGCAP** — `bigquery/migrations/001_partition_clustering.sql` (partition + cluster the three time-series tables, with a row-count-verified copy-back procedure and a pruning verification query). **DLA** — required partition filter (12-1), `functools` client caching (16-13). | — |
| AI Cost Management | **FINRA** — `packages/shared/src/cost.ts` computes run cost from reported token counts against a pinned price table, one entry per attempt that reached the model so retries are billed visibly; cheapest-capable model per stage; the screening stage exists purely as a cost decision and therefore fails open. Measured: **$23.50 for a 26-advertisement sweep**. | **DLA** — `documentation/runbooks/gemini-budget-cap.md`, Redis-cached scoring keys (7-2), cache invalidation (7-4). **LOGCAP** — model roles split so only menu generation pays Pro pricing (`config/models.ts`). | — |

## 6. Operational Excellence

| Sub-competency | Primary | Secondary | Gap |
|---|---|---|---|
| CI/CD & Deployment | **DLA** — six GitHub Actions workflows: `terraform.yml` (plan on PR with sticky diff comment, apply on protected merge, `-detailed-exitcode` handling), `security.yml` (gitleaks pinned by version + SHA-256, npm audit, pip-audit, nightly), `openapi-contract.yml`, `schema-regression.yml`, `dashboard-api-tests.yml`, `frontend-tests.yml`; plus `.pre-commit-config.yaml` and dependabot. | **FINRA** — `deploy.sh` per unit, idempotently re-granting the object-level Storage roles the signer needs; `packages/api/scripts/migrate.ts` for schema migration. | CI-fix commits `5871127` / `3d57f00` were reverted; workflows on `main` may fail → `repo-changes/dla-gti.md` |
| Infrastructure as Code | **DLA** — module-structured Terraform (`infra/modules/{apis,data,ingestion,iam,network,redis,apis-enable}`) over `infra/envs/dev`, remote GCS backend (`backend.tf`), `moved.tf` for safe refactors, `check` blocks as invariant guards, `.tflint.hcl`, WIF-authenticated CI apply (`ci_terraform_wif.tf`). | **TRAVEL** — `terraform/{main,iam,bigquery,gcs,scheduler,monitoring,variables}.tf`. | — |
| AI Lifecycle Management | **FINRA** — every run records `pipeline_version` and a `prompt_bundle_hash` (`packages/worker/src/lib/prompt-bundle-hash.ts`), prompts are versioned files under `packages/worker/src/prompts/`, and `PIPELINE_STAGES` is a registry a test enforces against `main.ts` so the pipeline cannot tick a stage the registry never heard of. Compliance policy is versioned and append-only, re-judging the backlog on read without re-running a model. | **LOGCAP** — models selected by env-overridable role constants. **DLA** — `gemini_models.py` centralises model choice. | — |
| Testing & Quality Engineering | **DLA** — ~40 pytest modules covering routes, auth gate, rate limiting, tool dispatch, schema consistency, Cloud Run config; Vitest/RTL frontend harness; k6 load test. **FINRA** — unit tests beside every module *plus* eight live verification scripts that exercise real GCP services, on the stated principle that mocked tests prove parsing while these prove integration. | **LOGCAP** — 38 test files including BigQuery SQL tests (`bigquery/tests/*.sql`). **TRAVEL** — 15 pytest modules, one per tool/story. | **ARMY** has ~20 ad-hoc `test_*.py` scripts with no runner → `repo-changes/army-artillery-ccc-capstone.md` |

## 7. Designing for Change

| Sub-competency | Primary | Secondary | Gap |
|---|---|---|---|
| Modularity & Abstraction | **FINRA** — `packages/shared` is the single source for clauses, types, schemas, constants and the one Vertex client factory; it imports from nobody, and `api` and `worker` never import from each other. The worker is the sole writer to the four analysis tables; the API is read-only against all four. | **DLA** — handlers / services / chat_tools / common split, with cross-module IAM isolated in `infra/modules/iam/cross-module.tf` so producer modules stay independent. **LOGCAP** — `shared/types.ts` shared across frontend and functions; `generateMenu` split into five phase modules. | — |
| Configuration Management | **FINRA** — every model, thinking level, token ceiling, timeout, retry policy, threshold and weight lives in `packages/shared/src/constants.ts` with a written justification and no literal at any call site. | **LOGCAP** — `functions/src/config/models.ts`, `config/mealRatios.ts`. **DLA** — `infra/src/common/config_loader.py` + `env.py`, with env defaults removed from code (12-8) and config held in a BigQuery config table with snapshots. | — |
| API Design & Versioning | **DLA** — contract-first: the OpenAPI spec is generated from handler annotations and CI fails on drift between spec, generated TypeScript client and runtime URL registry. | **FINRA** — `expectedVersion` on policy updates answers `409` instead of letting one reviewer silently discard another's change; `{ data } / { error }` envelope on all ten endpoints. **TRAVEL** — `agent_card.json` as the A2A contract. | — |
| Extensibility | **FINRA** — adding a clause is a registry entry (`packages/shared/src/clauses.ts`) that the adjudication fan-out picks up automatically; adding a stage is a registry entry the drift test enforces. | **TRAVEL** — ADK tools are drop-in functions (`dod_travel_agent/tools/`), and the agent gains a capability by registering one. **DLA** — `chat_tools/registry.py` is the same pattern for Gemini tool dispatch. | — |

---

## Coverage summary

All 7 categories have a primary **and** at least one secondary source; all 30 sub-competencies have
named file evidence. Distribution across the slate:

| Project | Primary for | Secondary for |
|---|---|---|
| FINRA | Agentic systems · Model selection · LLM ops & eval · Domain AI · AuthN/Z · Graceful degradation · Scalability · AI cost · AI lifecycle · Testing · Modularity · Config mgmt · Extensibility · System design artifacts | 8 more |
| DLA | Decision records · API docs · Operational docs · System design artifacts · Infra & network security · Data protection · AI-specific security · Compliance & governance · Availability · Observability · Failure testing · CI/CD · IaC · Testing · API versioning | 7 more |
| TRAVEL | Problem definition · Technical scope · Stakeholder alignment & success criteria | Agentic systems · Domain AI · Graceful degradation · IaC · Observability · Extensibility · API versioning |
| ARMY | Retrieval & data engineering (RAG) · Resource efficiency | Domain AI · System design artifacts |
| LOGCAP | — | Model selection · Modularity · Config mgmt · Resource efficiency · AI cost · Testing · API docs · Observability |

**Reading of that table.** ARMY and LOGCAP are thin as primaries, and that is a deliberate
consequence of what the repos hold — but LOGCAP being primary for nothing is the one result worth
acting on. Its strongest claim is the **hybrid LLM/deterministic architecture** (Gemini picks the
menu, TypeScript solves servings and feasibility, and the deterministic half is testable), which
belongs under Model Selection & Tuning; the packet entry should lead with that rather than with
the feature list, and the FINRA entry — which is over-subscribed — should cede the sub-competency
in its Demonstrated Competencies list.

### Blocking gaps (packet cannot cite these until fixed)

1. **DLA ADRs are off `main`** — Decision Records has no other strong artifact in the slate.
2. **TRAVEL degradation + guardrail chain is on `kim-dev`** — the per-diem-capped hotel search is
   that entry's best Reliability evidence.
3. **ARMY `main` is four months stale** — the Vertex AI migration exists only on a local branch.

### Non-blocking, worth fixing

4. DLA CI: reviewers judging Operational Excellence will open the Actions tab.
5. FINRA: code comments cite `architecture.md`, which is gitignored under `_bmad-output/` and so
   resolves to nothing for a reader.
6. ARMY: 12 scattered design notes, committed `__pycache__`, and unrunnable test scripts.
7. LOGCAP: a dangling `kim-dev2` (one feature commit and its revert) sits outside `main`.
