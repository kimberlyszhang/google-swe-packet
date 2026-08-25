**Project:** FINRA — Multimodal Compliance Review for Video Advertisements (Q3 2026). Reviews video ads against FINRA Rule 2210(d), returning a clause-by-clause verdict in which every finding cites the video by timestamp and quote.

**Authorship:** Kimberly Zhang

**Supporting Links/Assets:** https://github.com/GPS-Demos/finra-multimodal-compliance
- `packages/worker/src/stages/07-adjudicate.ts` — one agent per applicable clause
- same directory: `10-assert-verbatim.ts`, `11-verify-citations.ts` — the two citation gates
- `packages/shared/src/constants.ts`, `clauses.ts` — model tiering; clause registry
- `eval/`, `documentation/results.md` — three corpora, three answer keys, results
- `packages/api/src/with-handler.ts` — token verification and allowlist before dispatch
- `documentation/architecture.md` — the design contract the source cites 127 times

**Context/Impact:** A customer engagement had built Gemini Enterprise agents that only returned errors. I stepped in six hours before the demo and got it working. The demo then failed to land for lack of a UI, so I rebuilt the prototype from scratch. Customer response: *"I love the UI... It's a perfect example of what we're looking for"* (Alain); *"You exceeded my expectations. It's great"* (Dmytro).

Member firms face hundreds of thousands of pending video ad approvals, so cost and throughput per video are product constraints. The design makes evidence precede judgment: everything citable — a time-indexed transcript, every on-screen disclosure with measured dwell time and glyph height — is extracted and persisted before anything is adjudicated, and adjudication reads that stored evidence, never the video. Two gates then separate what an agent concluded from what a reviewer sees.

Measured at the build that produced it: the blind set was last swept at `0.5.0` (2026-08-10) — case recall 12/13, `mustNotFlag` breaches 0, **$23.50 per 26-advertisement sweep**, with 13-way parallelism cutting it from ~2 hours to 23.5 minutes. Specificity improved 2/10 → 6/10 by `0.11.1`. The clean-control gate stays red by design: the answer key is FINRA's, not the pipeline's.

**Demonstrated Competencies:**
- **Agentic & Multi-Agent Systems** — bounded per-clause fan-out; schema-rejected answers re-asked with the validator's refusal.
- **LLM Ops and Evaluation** — three corpora, separate answer keys, a deploy gate, a published noise floor.
- **Domain-Applied AI/ML Expertise** — 2210(d) as a clause registry; measured legibility (dwell, glyph height, WPM).
- **Authentication & Authorization** — auth before dispatch, so endpoints added later inherit it; `updated_by` from the token.
- **Graceful Degradation** — per-stage policy: evidence failures abort, verdict stages record `insufficient_evidence`, screening fails open.
- **Scalability & Elasticity** — measured concurrency, with the BigQuery DML contention that bounds it.
- **AI Cost Management** — cost from token counts against a pinned price table, one entry per attempt, so retries bill.
- **AI Lifecycle Management** — versioned prompts, `pipeline_version` and prompt-bundle hash per run, a test-enforced stage registry.
- **Modularity & Abstraction** — `shared` is the single source of clauses, types and the Vertex client.
- **Configuration Management** — every model, thinking level and threshold in one module; no literal at any call site.
- **Extensibility** — adding a clause is a registry entry the fan-out picks up.
- **System Design Artifacts** — an architecture document plus a diagram section stating what it *claims*.

---

**Project:** Defense Logistics Agency — Cyber Health Dashboard. AI-driven vulnerability management replacing manual Excel reporting, tracking 200,000+ CVEs across Google Threat Intelligence, Chronicle and eMASS.

**Authorship:** Kimberly Zhang

**Supporting Links/Assets:** https://github.com/GPS-Demos/dla-gti
- `documentation/adr/` — five decision records
- `documentation/runbooks/` — ten runbooks (VPC-SC, CMEK, Cloud Armor, IAP, monitoring, secret rotation)
- `infra/src/common/ai_safety.py` — input sanitisation and output injection-marker scanning
- `infra/modules/`, `infra/envs/dev/` — module-structured Terraform, remote state, `check` blocks
- `.github/workflows/` — six workflows, including Terraform plan and OpenAPI drift gates
- `tests/load/sse-chat.k6.js` — 50 concurrent SSE sessions against real Cloud Run

**Context/Impact:** Delivered as a prototype in Q2 2026 and developed further live with the customer during a full-day onsite at DLA Troop Support in Philadelphia. DLA triaged vulnerabilities through a manual "report card" workflow in spreadsheets; the prototype kept that workflow and put a near-real-time platform underneath it, with Gemini ranking what to patch first.

The operational engineering is the substance here. The input is attacker-controllable — CVE and alert text written by third parties and fed to a model — so prompt-injection defence is two-sided: input is sanitised and sentinel-wrapped on the way in, and output is scanned for injection markers and redacted before reaching the UI, with regression tests for both. Access is least-privilege per service: each runs as its own service account, IAM bindings are additive-only, and ingress is internal-only.

A control deferred rather than built is a written decision with a reversal condition — ADR 0005 defers VPC-SC and CMEK and names what reverses it: live data, CUI, or an RMF assessment. ADR 0001 is the honest one: its rejected alternative shipped, then was reverted after a partial write left four copies of every CVE.

**Demonstrated Competencies:**
- **Decision Records** — five ADRs with alternatives rejected and reversal conditions; one covers an alternative that shipped, then reverted.
- **API Documentation** — OpenAPI generated from handlers; CI fails on spec/client/runtime drift.
- **Operational Documentation** — ten runbooks covering applied and deferred controls.
- **Infrastructure & Network Security** — VPC flow logs, internal-only ingress, scoped run-invoker, additive IAM.
- **Data Protection & Privacy** — Secret Manager for API keys; gitleaks in CI *and* pre-commit; bucket hardening.
- **AI-Specific Security** — two-sided prompt-injection defence, with regression tests.
- **Compliance & Governance** — audit log sink with retention, eMASS RMF/POA&M modelling.
- **Availability Design** — scheduled ingestion with retries; min-instances asserted in tests; Memorystore rate-limit state across instances.
- **Observability** — purpose-built metrics for parse failures, scoring and ingestion errors, plus a runbook.
- **Failure & Recovery Testing** — k6 at 50 concurrent SSE sessions with p95 and error SLOs.
- **CI/CD & Deployment** — Terraform plan-on-PR with sticky diff comment, apply on protected merge, security and contract gates.
- **Infrastructure as Code** — modules over environments, remote state, `check` blocks as invariant guards.
- **API Design & Versioning** — contract-first; drift between spec, client and runtime fails the build.
- **Testing & Quality Engineering** — ~40 pytest modules covering routes, auth, rate limiting, Cloud Run.

---

**Project:** Army Fires Center of Excellence — Captains Career Course Table-Top Exercise Simulator. Replaces Excel-based tabletop exercises with a RAG chatbot over Army doctrine and a Google Maps operational picture.

**Authorship:** Kimberly Zhang

**Supporting Links/Assets:** https://github.com/GPS-Demos/army-artillery-ccc-capstone
- `docs/ARCHITECTURE.md` — system architecture including the RAG pipeline
- `docs/PERFORMANCE.md` — four optimizations with before/after numbers
- `backend/rag_search.py`, `pdf_processor.py`, `embeddings.py`, `citations.py` — hybrid retrieval, chunking, embeddings, citations
- `backend/sql-gin-index/INDEX_OPTIMIZATION.md` + `test_index_performance.py` — the index work and its harness
- `backend/terrain_traversability.py`, `constants.py` — slope analysis per unit type; single-source config

**Context/Impact:** **$1,000,000**, now moving under contract and successfully transitioned to the partner; selected as one of the "Top 5" Defense projects highlighted on the screens at the Reston EBC center. After the initial demo to a General at AUSA: *"This looks great and really advances the way we train."* Final feedback: *"I'm amazed. This prototype shifts our training from manual spreadsheets to something dynamic that can keep up with the speed of combat"*, *"We see this architecture becoming the standard for Captains Career Courses across the entire Army, not just for Artillery."* It also opened a further opportunity — the Air Force wants to repurpose it for the Air Force Weapons School.

Two technical problems sat behind the spreadsheets. First, students needed doctrine answers they could trust, making retrieval quality and citation the whole game: doctrine PDFs are ingested with overlap chunking that preserves page and section provenance, retrieval is hybrid semantic-plus-keyword over pgvector, and every answer cites its source page. The embedding model is pinned at 768 dimensions because the stored corpus was embedded in that space — query vectors must match it, so changing models means a re-ingest, not a config edit.

Second, the map had to be worth looking at. Asked about bridges and crossings, I used Google Places and Maps APIs to surface every major landmark — mountains, rivers, airports, hospitals, parks, tunnels — and added terrain analysis computing slope across the region to rate mobility separately for tracked vehicles, wheeled vehicles and dismounted infantry, and to identify optimal observation points. Borders came from QGIS-generated GeoJSON; units use MIL-STD-2525 symbology. Both are expensive done naively, which is where the measured optimization work came from.

**Demonstrated Competencies:**
- **Retrieval & Data Engineering (RAG)** — overlap chunking preserving page/section provenance, batched embeddings, hybrid semantic + keyword retrieval over pgvector, citations to the source page.
- **Resource Efficiency** — four measured optimizations (index tuning, elevation-grid sampling, viewport-bounded Places queries, feature filtering) with a test harness, not estimates.
- **Domain-Applied AI/ML Expertise** — MIL-STD-2525 symbology and slope-based traversability rated per unit type; QGIS-derived exercise geography.
- **Model Selection & Tuning** — embedding dimensionality pinned to the stored corpus's space as a deliberate compatibility decision, with the reasoning recorded.
- **Configuration Management** — model, retrieval and environment config in one constants module, with a startup check that fails fast on a retired model ID.
- **System Design Artifacts** — system-context and frontend class diagrams, with `docs/ARCHITECTURE.md` as the entry point the README defers to.

---

**Project:** DoD Travel Agent on genai.mil. A Joint Travel Regulations policy agent — ADK on Cloud Run, served to Gemini Enterprise over A2A — answering entitlement questions with mandatory JTR citations and building itineraries inside GSA per-diem caps.

**Authorship:** Kimberly Zhang

**Supporting Links/Assets:** https://github.com/GPS-Demos/dod-travel-agent
- `_bmad-output/planning-artifacts/prd.md` — 27 functional requirements across 7 capability areas; 12 numeric NFRs
- `_bmad-output/planning-artifacts/architecture.md` — constraints, degradation matrix
- `_bmad-output/implementation-artifacts/` — epics as a phased plan; per-story acceptance criteria
- `dod_travel_agent/tools/` — `lookup_per_diem.py`, `search_hotels.py`, `search_flights.py`
- `agent_card.json` — the A2A contract; `terraform/`

**Context/Impact:** Delivered in Q2 2026 for genai.mil, the DoD's Gemini Enterprise environment, letting users ask questions like *"Can I get reimbursed for in-flight internet on a cross-country flight?"* and build itineraries compliant with GSA per-diem caps. Travelers check compliance by hand and submit DTS vouchers hoping they clear; Authorizing Officials review those submissions with no efficient way to verify compliance. The agent moves compliance from *rejection*, after submission, to *prevention*, before it.

I include this one mainly for what happened before the code: the scoping artifacts fix the problem, constraints and acceptance criteria ahead of the build, and those constraints shaped the design rather than decorating it. A 1M-token budget against a ~600-page JTR is what made loading the whole JTR into the agent instruction viable — buying a no-retrieval-miss property — and also what makes JTR growth a monitored risk. A daily scrape of `travel.dod.mil` was planned and then deliberately **removed**: a brittle scraper failing silently against a regulatory document is worse than a human running a seed script when the JTR is revised.

The decision I would defend hardest is that per-diem compliance is enforced in the tool, not requested of the model. `search_hotels` looks the cap up itself, passes it to the vendor as a price ceiling, and re-enforces it in code after parsing — so compliance is a property of the query, not an instruction the model is asked to honour. The architecture document also records where the build does **not** yet meet its own constraint: model calls still route through the Google AI API rather than Vertex AI, with the steps that close it.

**Demonstrated Competencies:**
- **Problem Definition** — the business problem and what success unlocks for both user types stated before any design; 27 FRs across 7 capability areas.
- **Technical Scope & Constraints** — FedRAMP High as a deployment boundary, a 1M-token ceiling against a ~600-page regulation, and an explicit record of the constraint the build does not yet satisfy.
- **Stakeholder Alignment & Success Criteria** — 12 NFRs with numeric ceilings (30s JTR Q&A, 3s per diem) carried into per-story acceptance criteria; epics as the phased plan.
- **Graceful Degradation** — a per-dependency degradation matrix whose rule is "never hallucinate when an API is down — say unavailable," implemented as refusal, not fallback.
- **Extensibility** — ADK tools are drop-in functions; the agent gains a capability by registering one.
- **API Design & Versioning** — `agent_card.json` as the published A2A contract.

---

**Project:** LOGCAP DFAC Command Center. Menu planning and inventory for military Dining Facilities at Kwajalein Atoll, generating 28-day menu plans feasible against real inventory, expiry and delivery schedules.

**Authorship:** Kimberly Zhang

**Supporting Links/Assets:** https://github.com/GPS-Demos/logcap-dfac
- `docs/TECHNICAL.md` § 4 — "Menu generation: what the model decides and what the code decides"
- `functions/src/generateMenu/` — the five phases: `phase1-pick.ts` (Gemini), `phase2-servings.ts`, `phase3-filter.ts`, `phase4-solve.ts`, `phase5-assemble.ts`
- `generateMenu/deterministic.ts`, `json-repair.ts` — always-on structural layer; tolerant parser
- `functions/src/config/models.ts`, `config/mealRatios.ts` — four model roles, env-overridable; ratio constants
- `bigquery/migrations/001_partition_clustering.sql` — partitioning with a verified copy-back and pruning check

**Context/Impact:** LOGCAP is supporting the incumbent partner on a recompete of an **$89Bn TCV** acquisition, with an estimated Google total value of **$60M**. The final sprint closed on 2026-02-17 and the customer said it looks great.

I include this one for a narrower reason than the others: it is my cleanest example of deciding what a model should and should not be asked to do. Menu planning looks like an LLM problem and mostly is not. Choosing which entrees to serve on which day is a judgment call over recipes, patron history, events and what is about to expire — that is Gemini's job, and the only thing Gemini is asked for. Everything with a right answer belongs to TypeScript: servings arithmetic, inventory and expiry validation, and the feasibility solve. The model never sees a serving count and never decides whether a menu is affordable.

That split makes the system testable and cheap. Phase 3 sums every ingredient's demand across 28 days against on-hand stock, and where an item has incoming deliveries it walks the menu in date order, requiring cumulative usage to stay under what has landed by that date. Phase 4 solves violations by swapping over-budget slots to same-category recipes avoiding the constrained ingredient. The deterministic layer is not a fallback for a failed model call — it runs first on every request, filling every non-entree slot before Gemini is invoked, so a degraded response still decodes into a full 28-day menu rather than into nothing.

**Demonstrated Competencies:**
- **Model Selection & Tuning** — an explicit determinism boundary: the model is asked for the one judgment call and nothing else, which keeps a 28-day plan inside an 8192-token ceiling; structured JSON with a tolerant parser and a retry ladder feeding validation errors back.
- **Modularity & Abstraction** — a 2,400-line function refactored into five phase modules, with a shared type contract used by frontend and Cloud Functions alike.
- **AI Cost Management** — model roles split so only menu generation pays Pro pricing; chat, insights and vision run on Flash-Lite and Flash.
- **Resource Efficiency** — three time-series tables partitioned and clustered, with a row-count-verified copy-back and a partition-pruning verification query.
- **Testing & Quality Engineering** — 38 test files, including SQL tests for the BigQuery layer.
- **API Documentation** — a 16-endpoint reference with a consistent `{ data } / { error }` envelope, anchored to the current source layout.
