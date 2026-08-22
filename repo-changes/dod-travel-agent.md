# dod-travel-agent — repo changes for the packet

**Repo:** https://github.com/GPS-Demos/dod-travel-agent
**Packet role:** primary evidence for Problem Definition, Technical Scope & Constraints, and
Stakeholder Alignment & Success Criteria; secondary for Agentic Systems, Graceful Degradation,
IaC and Extensibility.
**Verified against `origin/main` @ `ea9e7fc` (2026-04-04) on 2026-08-22.**

---

## 1. Merge `kim-dev` into `main` — **blocking**

**Status quo.** `main` is eleven commits behind `kim-dev`. Two of those eleven are not even pushed
to `origin/kim-dev`, so they exist on this laptop only:

```
f286b81  fix: resolve airport codes and bare city names to City, ST format   ← local only
3c3cc29  fix: block hotel search when per diem lookup fails                  ← local only
136310d  feat: markdown output for Gemini Enterprise rendering
ca4a789  feat: search_hotels internally calls lookup_per_diem for a deterministic price cap
a16d0de  fix: hard price filter as a safety net for SerpApi max_price leaks
b255c74  feat: use SerpApi max_price to guarantee per-diem-compliant hotels
09dcac1  feat: increase hotel search results from 10 to 20
4a1d98c  feat: prioritize highest-rated hotels in search results
143cb78  docs: Gemini Enterprise deployment and registration instructions
fe28087  feat: add version field to agent card
36f6623  fix: improve output formatting for plain-text rendering
```

There is also an **uncommitted** change in the working tree: `dod_travel_agent/tools/
lookup_per_diem.py` (+83 lines, an IATA → City, ST lookup table). Decide whether it ships before
merging; leaving it uncommitted means it is one `git checkout` from gone.

**Why it blocks a requirement.** Four of these commits are the single best Reliability and
AI-guardrail evidence in the repo, and none of it is visible on `main`:

- `ca4a789` + `b255c74` — `search_hotels` calls `lookup_per_diem` itself and passes the returned
  lodging cap to SerpApi as `max_price`, so per-diem compliance is a **property of the query**
  rather than something the model is asked to respect. That is the same
  "never ask a model for something that can be computed" discipline the FINRA entry claims, shown
  in a second, independent codebase.
- `a16d0de` — a hard price filter behind it, because the vendor's `max_price` was observed to leak
  over-cap results. A guardrail that does not trust the upstream guardrail.
- `3c3cc29` — when the per-diem lookup fails, hotel search is **blocked** rather than degraded to
  unfiltered results. This is the concrete instance of the graceful-degradation matrix in
  `_bmad-output/planning-artifacts/architecture.md` ("never hallucinate data when an API is down").

`main` today shows the FedRooms/BigQuery approach that GSA's 2024-09-30 shutdown made obsolete.
A reviewer reading `main` sees an agent wired to a dead data source.

**The change.**

```bash
cd ~/Documents/Google/dod-travel-agent
git status                      # decide on the uncommitted lookup_per_diem.py change first
git push origin kim-dev         # get the two local-only commits off this laptop
gh pr create --base main --head kim-dev \
  --title "Per-diem-capped hotel search via SerpApi, Gemini Enterprise packaging" \
  --body "Merges the eleven commits on kim-dev: SerpApi replaces the shut-down FedRooms source, \
the lodging cap is enforced as a query parameter with a hard filter behind it, hotel search is \
blocked rather than degraded when per diem is unavailable, and the agent card + docs cover \
Gemini Enterprise registration."
gh pr merge --merge
```

Run `pytest tests/` before merging — `tests/test_search_hotels.py` grew by 118 lines on this
branch and is the thing that proves the cap.

---

## 2. Replace the README — **high value**

**Status quo.** `README.md` on `main` documents `run-workflow.sh`, the BMAD story pipeline, and its
code-review strategy. It says nothing about what the agent *is*, what it integrates, or how it is
deployed. A reader lands on a development-process document for a repo the packet cites as an
architecture-and-scoping exemplar.

**Why it matters.** *Problem Definition* and *Technical Scope & Constraints* are this entry's
primary claims. The material for both already exists — it is just buried in
`_bmad-output/planning-artifacts/`, which reads as process output rather than as the project's
front door.

**The change.** A README that pulls the existing, already-written material up to the top level:

1. **What it is** — a JTR policy-guidance agent on Gemini Enterprise, reachable over A2A, that
   answers travel-entitlement questions with mandatory JTR section citations and builds itineraries
   inside GSA per-diem caps.
2. **Constraints that shaped it** — lifted from `architecture.md` § Technical Constraints:
   FedRAMP High as a hard deployment boundary (Vertex AI only, fail closed if unreachable), the
   1M-token context ceiling against a ~600-page JTR, and the fragility of scraping travel.dod.mil.
3. **The tool contract** — the three ADK tools, their inputs, and what each one refuses to answer.
4. **The graceful-degradation matrix** — one row per external dependency (GSA per diem, SerpApi
   Google Hotels, Amadeus, the JTR scrape), its failure mode, and the user-facing behaviour. After
   change 1, `3c3cc29` makes the hotel row true in code.
5. **Deploy** — `deploy.sh`, `seed-jtr.sh`, `terraform/`, and the Gemini Enterprise registration
   steps from `143cb78`.
6. **Keep the current README content** as `docs/DEVELOPMENT.md`; the BMAD pipeline description is
   real and belongs somewhere, just not on the front page.

Every claim above already exists in a committed file — this is relocation, not authorship.

---

## 3. Optional: surface the planning artifacts

`_bmad-output/` is covered by the global gitignore in most of these projects but **is committed
here** (18 markdown files, including `prd.md`, `architecture.md`, `epics.md` and 15 per-story
specs). That is lucky and it is the packet's best Scoping and Documentation link. Two small things
make it easier to cite:

- The frontmatter in `architecture.md` reads `project_name: 'dla-gti'` — a template artefact from
  the session that produced it. One-line fix; it currently makes the document look like it belongs
  to a different project.
- Add a `_bmad-output/README.md` (or a README § Planning artifacts) that names the four documents
  a reviewer should read and in what order: PRD → architecture → epics → one representative story
  spec, e.g. `3-1-gsa-per-diem-tool.md`.

---

## Order of operations

1. Decide on the uncommitted `lookup_per_diem.py` change.
2. `git push origin kim-dev`, PR to `main`, merge (after `pytest tests/`).
3. Rewrite the README against the now-current `main`.
4. Fix the `project_name` frontmatter; add the planning-artifacts pointer.

Step 2 is mechanical and I can run it on request. Step 3 I can draft in full for your review —
it is assembly from committed sources, no new claims.
