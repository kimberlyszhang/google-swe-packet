# Repo changes — triage index

What to change in each packet repo, why, and in what order. Every claim in these documents was
verified with `git` and `gh` against `origin/main` on **2026-08-22**; nothing here is inferred from
a README.

The packet itself (`documentation/packet.md`) is written **after** you decide on these, so that
every Supporting Link resolves and no entry cites evidence that is sitting on a branch.

Evidence mapping for all of it: [`documentation/coverage-matrix.md`](../documentation/coverage-matrix.md).

---

## Blocking — the packet cannot cite this evidence until it lands

| # | Repo | Change | Effort | Risk |
|---|---|---|---|---|
| 1 | [army-artillery](army-artillery-ccc-capstone.md) | **Push `kim-dev`.** Commit `7174ca4` (Vertex AI migration, `constants.py` single-source config, GCS scenario load) exists **only on this laptop**. `main` is a build that fails at runtime. | 5 min | none |
| 2 | [dla-gti](dla-gti.md) | **Cherry-pick `0346350`** — five ADRs, rolled back on 2026-08-21 with the infra commits. Docs-only. Decision Records has no other artifact in the slate. | 5 min | none |
| 3 | [dod-travel-agent](dod-travel-agent.md) | **Merge `kim-dev` (11 commits)** — per-diem-capped hotel search, hard price filter behind the vendor's, hotel search blocked when per diem fails. Two of the eleven are unpushed. `main` still wires the agent to FedRooms, shut down by GSA in 2024. | 30 min | run `pytest tests/` first |

Do #1 today regardless of the packet — that commit exists in exactly one place.

## High value — fixes something a reviewer will notice

| # | Repo | Change | Effort |
|---|---|---|---|
| 4 | [dla-gti](dla-gti.md) | **Cherry-pick `5871127` + `3d57f00`** — the `terraform` and `security` workflows fail on every run today (provider lock records `darwin_arm64` only; six security jobs red). This repo anchors CI/CD & Deployment, and the Actions tab is one click from the packet link. | 20 min |
| 5 | [finra](finra-multimodal-compliance.md) | **Publish `architecture.md`** — 65 files carry 127 anchored citations to a document that is gitignored and has never been pushed. Copy to `documentation/`, `git add -f`. | 15 min |
| 6 | [army-artillery](army-artillery-ccc-capstone.md) | **Consolidate 20 scattered notes** into `docs/ARCHITECTURE.md` (add the RAG pipeline the README omits) + `docs/PERFORMANCE.md` (four optimization write-ups, measured numbers first). Assembly, not authorship. | 2 h |
| 7 | [logcap-dfac](logcap-dfac.md) | **Rewrite `docs/TECHNICAL.md` § 4** — it cites `generateMenu.ts:2475`, a file that became a five-module phased pipeline. The undocumented refactor *is* the repo's primary packet claim. | 1 h |
| 8 | [dod-travel-agent](dod-travel-agent.md) | **Replace the README** — currently documents `run-workflow.sh` and nothing about the agent. Pull the constraints and the degradation matrix up from `_bmad-output/planning-artifacts/architecture.md`. | 1 h |

## Hygiene — cheap, do them while you're in the repo

| # | Repo | Change |
|---|---|---|
| 9 | army-artillery | `git rm --cached` the committed `__pycache__/*.pyc`; add `__pycache__/` to `.gitignore`. 25 open Dependabot PRs also make the front page read "abandoned". |
| 10 | logcap-dfac | `git branch -D kim-dev2` (a feature and its own revert, against a deleted file, superseded on `main`); recount the endpoints the README claims. |
| 11 | dod-travel-agent | `architecture.md` frontmatter says `project_name: 'dla-gti'`; decide the fate of the uncommitted `lookup_per_diem.py` change (+83 lines). |
| 12 | dla-gti | Optional: a `documentation/runbooks/README.md` index so Operational Documentation is one link instead of ten. |

## Decisions, not changes

- **dla-gti monitoring commits stay reverted.** `89ee945` / `1c1754b` / `1b25490` change what
  `terraform apply` does to the project. The packet's Observability claim rests on the metrics
  stories and the runbook, all on `main`, and does not need them. Re-landing them is a deploy
  exercise, not a packet task.
- **FINRA blind-set numbers: cite as dated.** The README already labels them `0.5.0`, not
  re-measured, and publishes the noise floor beside them. That is a stronger evaluation artifact
  than a fresh number. Re-running is ~25 min and ~$25 if you want the headline to match the build.
- **No new CI for FINRA or LOGCAP.** DLA anchors CI/CD with six workflows. A workflow added in
  August 2026 to a repo last touched in May reads as packet dressing.

---

## What I can run for you

Mechanical, on request: #1, #2, #3, #4, #9, #10, #11 (the branch/frontmatter parts).
Drafts for your review: #6, #7, #8 — all assembly from committed sources, no new claims.
Yours to decide: the dla-gti monitoring question, the FINRA re-run, and whether the uncommitted
travel-agent airport-code table ships.
