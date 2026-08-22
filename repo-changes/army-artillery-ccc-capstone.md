# army-artillery-ccc-capstone — repo changes for the packet

**Repo:** https://github.com/GPS-Demos/army-artillery-ccc-capstone
**Packet role:** primary evidence for Retrieval & Data Engineering (RAG) and Resource Efficiency;
secondary for Domain-Applied AI and System Design Artifacts.
**Verified against `origin/main` @ `cb12eec` (2026-04-27) on 2026-08-22.**

---

## 1. Push and merge the Vertex AI migration — **blocking**

**Status quo.** `origin/main` and `origin/kim-dev` are identical, and both are from April. The only
newer work is **one unpushed commit on this laptop**, `7174ca4` (2026-08-21) — 398 insertions
across six files. If this machine dies, it is gone.

**Why it blocks a requirement.** The commit is not a tidy-up; it is three packet-relevant decisions
at once, all with the failing symptom recorded in the message:

- **Vertex AI over an API key.** The `GOOGLE_API_KEY` secret held a key Argolis rejects with
  `API_KEY_INVALID` against `generativelanguage.googleapis.com`, which broke generation *and*
  embedding — every `/chat` request errored. Now authenticates with ADC / the runtime service
  account, granted `roles/aiplatform.user`, against the `global` endpoint (`us-central1` 404s for
  Gemini 3.x in this project). This is the same conclusion FINRA reached independently and DLA
  wrote up as ADR 0002 — three projects, one verified constraint.
- **Configuration single-source.** Model selection, retrieval parameters and env config moved into
  `backend/constants.py` with a **startup check that fails fast on a retired model id**. That is
  direct evidence for *Configuration Management*, which this repo currently contributes nothing to.
- **An embedding-space compatibility decision.** `text-embedding-004` is deliberately held at 768
  dimensions because the stored corpus was embedded with it, so query vectors must stay in the same
  space — no re-ingest. That is the kind of RAG-specific reasoning the rubric's Retrieval line is
  looking for, and it is one sentence in a commit message nobody will find.

Also in the commit: the DIVAD exercise scenario now loads from GCS because the Docker build context
is `backend/`, so the old relative path never resolved inside the container — the simulator could
not answer questions about its own exercise, and `PYTHONUNBUFFERED` is what made that visible.
Without this, `main` is a build that fails at runtime.

**The change.**

```bash
cd ~/Documents/Google/army-artillery-ccc-capstone
git checkout kim-dev
git push origin kim-dev
gh pr create --base main --head kim-dev \
  --title "Migrate Gemini to Vertex AI; centralise config; load the exercise scenario from GCS" \
  --body "See commit 7174ca4 — the API key path was dead (API_KEY_INVALID), model/retrieval config \
moves to backend/constants.py with a startup guard, and the DIVAD scenario loads from GCS because \
the Docker build context made the relative path unresolvable."
gh pr merge --merge
```

Redeploy after merging (`deploy-back-end.sh` changed by 42 lines) — the packet should not link to a
`main` whose deployed counterpart is a different build.

---

## 2. Consolidate twenty scattered design notes — **high value**

**Status quo.** Twenty markdown files, 4,400 lines, with no index and no obvious entry point:
`rag-plan.md` (1,129 lines) at the root, four terrain documents under `backend/terrain/`, three
loose notes under `notes/`, two demo scripts, `instructions.md`, `MAP_IMPLEMENTATION_GUIDE.md`. The
README is 138 lines and titled "Army Artillery Capstone Architecture", but it covers the frontend
and a system-context diagram only — nothing about the RAG pipeline that the packet cites this repo
for.

**Why it matters.** Two of this entry's claims are buried in files a reviewer has no reason to
open. `backend/sql-gin-index/INDEX_OPTIMIZATION.md` and `backend/terrain/
ELEVATION_GRID_OPTIMIZATION.md` are the packet's primary evidence for *Resource Efficiency*, and
they are three directories deep with names that read like scratch notes.

**The change.** Two documents, assembled from what already exists — no new claims:

- **`docs/ARCHITECTURE.md`** — promote the README's system-context and class diagrams, then add the
  RAG pipeline the README omits: PDF ingestion and overlap chunking with page/section provenance
  (`backend/pdf_processor.py`), embedding generation (`backend/embeddings.py`), hybrid semantic +
  keyword retrieval over pgvector (`backend/rag_search.py`), citation formatting
  (`backend/citations.py`). `rag-plan.md` already contains this material in planning form; this is
  the built version of it.
- **`docs/PERFORMANCE.md`** — merge `INDEX_OPTIMIZATION.md`, `ELEVATION_GRID_OPTIMIZATION.md`,
  `VIEWPORT_ENHANCEMENT_SUMMARY.md` and `BUSINESS_FILTERING_SOLUTION.md` into one document with the
  measured before/after numbers up front, and cite `backend/sql-gin-index/test_index_performance.py`
  as the harness. One link, four optimizations, numbers first.

Keep the originals where they are and link them from the new documents — the goal is an entry
point, not a rewrite. Leave `demo_script.md` / `demo_script_air_force.md` alone; they are customer
artifacts and they date the Air Force interest that `impact-2025.md` records.

---

## 3. Hygiene — **small, do it while you're there**

- `backend/terrain/__pycache__/api_key_utils.cpython-311.pyc` is committed. Remove it and add
  `__pycache__/` to `.gitignore`:
  ```bash
  git rm --cached backend/terrain/__pycache__/api_key_utils.cpython-311.pyc
  printf '__pycache__/\n*.pyc\n' >> .gitignore
  ```
  A compiled artifact named `api_key_utils` sitting in a repo is also the wrong first impression on
  a packet that claims secrets discipline, whatever the file actually contains.
- There are 25 open Dependabot PRs against this repo (21 npm, 5 pip). Merging even the security
  ones changes what a reviewer sees on the front page from "abandoned" to "maintained."

---

## 4. Optional — a test entry point

`backend/` holds roughly twenty `test_*.py` scripts (`test_rag_pipeline.py`, `test_embeddings.py`,
`test_retriever.py`, `test_citations.py`, `test_terrain_integration.py` …) written as standalone
scripts rather than as a suite — several require live GCP credentials and a database. The packet
does **not** claim Testing & Quality Engineering for this repo (DLA, FINRA and LOGCAP carry it), so
this is optional.

If you want it: add `backend/pytest.ini` with markers separating unit from live-integration, mark
the credential-dependent modules `@pytest.mark.integration`, and make `pytest -m "not integration"`
pass from a clean checkout. That is real work — half a day at least — and it is the only item in
this file that is not assembly of existing material. Skip it unless the packet ends up thin on
testing, which the matrix says it is not.

---

## Order of operations

1. `git push origin kim-dev` — **today**, before anything else; that commit exists in one place.
2. PR to `main`, merge, redeploy.
3. `docs/ARCHITECTURE.md` + `docs/PERFORMANCE.md`.
4. `__pycache__` removal and `.gitignore`.
5. Dependabot triage (optional), pytest harness (optional, probably skip).

Steps 1, 2 and 4 are mechanical and I can run them on request. Step 3 I can draft for review.
