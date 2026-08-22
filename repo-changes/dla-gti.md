# dla-gti — repo changes for the packet

**Repo:** https://github.com/GPS-Demos/dla-gti
**Packet role:** primary evidence for Security/Privacy/Compliance, Reliability & Resilience,
Operational Excellence, and — if change 1 lands — Decision Records.
**Verified against `origin/main` @ `203d044` on 2026-08-22.**

This repo is the strongest operational-engineering evidence in the slate. Everything below is
about a single event: on 2026-08-21, `203d044` reverted the six commits sitting on top of
`4de6ca9`, restoring the tree byte-for-byte. That was the right call for the commits that change
deployed infrastructure. It also took three commits with it that change **no infrastructure at
all**, and those three are exactly the ones the packet needs.

---

## 1. Restore the ADRs — **blocking**

**Status quo.** `git ls-tree -r origin/main | grep -i adr` returns nothing. The ADRs landed in
`0346350 docs: add architecture decision records` and were rolled back four hours later.

**Why it blocks a requirement.** *Decision Records — "ADRs, documenting trade-offs, alternatives
considered, rationale"* is a named sub-competency under Scoping and Documentation, and this commit
is the only artifact in the five-repo slate that satisfies it in the form the rubric names. The
five records are:

| ADR | Decision | Why it reads well |
|---|---|---|
| 0001 | Chunk BigQuery writes on payload size, not row count | The rejected alternative was *implemented and reverted* — `366d08b` routed fat batches to the Storage Write API, `87d0b55` undid it after a partial write plus a 500 left four copies of every CVE |
| 0002 | Reach Gemini through Vertex AI, not a Developer API key | Records that rotating the key was the obvious move and would not have helped |
| 0003 | Hold chat rate-limit state in Memorystore, keyed on the authenticated principal | States plainly that the Redis/Firestore/in-process cascade **degrades open** |
| 0004 | Generate the OpenAPI spec from handlers, fail CI on drift | Contract-first with an enforcement mechanism |
| 0005 | Defer VPC-SC and CMEK to runbooks | Carries a written **reversal condition**: before live DLA data, CUI, or an RMF assessment |

Each carries Status / Context / Decision / Alternatives considered and rejected / Consequences /
Reversal condition, plus the source finding and its severity, indexed by `documentation/adr/README.md`.

**The change.** Cherry-pick the docs-only commit back. It touches `documentation/adr/` only — no
Terraform, no application code, nothing that a `terraform apply` can act on.

```bash
cd ~/Documents/Google/dla-gti
git checkout main && git pull --ff-only
git cherry-pick 0346350        # docs: add architecture decision records
git push origin main
```

**Risk:** none. Verify with `git show --stat 0346350` before picking — if any path outside
`documentation/` appears, stop and split the commit.

---

## 2. Get `security` and `terraform` green again — **high value, non-blocking**

**Status quo, verified 2026-08-22 via `gh run list`:** `frontend-tests`, `schema-regression` and
`openapi-contract` pass. **`terraform` and `security` fail on every run.**

- `terraform` dies at `terraform validate`:
  `hashicorp/null 3.2.4 … does not match any of the checksums recorded in the dependency lock file`
  — the lock file records `darwin_arm64` only, so no Ubuntu runner can ever validate.
- `security` fails **six of its jobs**: gitleaks full-history scan, npm audit (high/critical),
  pip-audit × 4 (common, dashboard-api, ai-analyzer, gti-ingestion, score-calculator), and trivy
  (which dies at `Set up job` because `aquasecurity/trivy-action@0.28.0` stopped resolving).

**Why it matters.** This repo anchors *CI/CD & Deployment*. A reviewer who follows the packet link
to a Terraform-plan-gated pipeline and then clicks the Actions tab sees two permanently red
workflows — including the security one — on the project the entry cites for security engineering.
That reads worse than having no CI at all.

**The change.** Restore the two reverted fix commits. Both were verified locally before they were
first pushed and neither creates, modifies or destroys a GCP resource:

- `5871127 fix(ci): make the terraform and dashboard-api-tests workflows able to pass` — adds
  `linux_amd64` + `darwin_amd64` checksums to `.terraform.lock.hcl` and mocks `_get_bq` in two
  tests that were constructing a real BigQuery client. Two files, 13 lines.
- `3d57f00 fix(security): clear every finding in the security workflow` — dependency upgrades
  (next, sharp, protobufjs override, and five regenerated Python locks compiled on 3.12 with
  `--allow-unsafe`), the trivy action pinned by commit SHA, and a `.gitleaksignore` allowlisting
  four historical occurrences of one Firebase Web API key **by per-commit fingerprint**, with the
  reasoning and the unresolved residue written out in the file.

```bash
git cherry-pick 5871127 3d57f00
git push origin main
gh run list --limit 6          # confirm terraform + security go green
```

**Two things to know before doing this.**

1. `5871127`'s message says the three `GCP_*` Actions **repository variables** for WIF were set as
   part of that work. Those are repo settings, not files — the revert did not unset them, but
   confirm with `gh variable list` before assuming the auth step passes.
2. `3d57f00` touches `infra/envs/dev/backend.tf` (+22 lines) for the inline trivy `GCP-0077`
   suppression on the state bucket. Read that hunk before pushing: it should be comment/tag only.

**Alternative if you'd rather not carry the dependency bumps:** cherry-pick `5871127` alone (green
`terraform`), and disable the `security` workflow's schedule rather than leaving it red. Do not
delete the workflow — its existence is part of the evidence.

---

## 3. Leave the monitoring commits reverted — and say so

`89ee945` (deploy score-calc + dashboard-api 5xx alert policies), `1c1754b` (three defects the
first apply exposed) and `1b25490` (defer the orphaned `google_project` data source) all change
what `terraform apply` will do to the project. `203d044`'s own message already records the
consequence: the next apply would remove the alert policies and the `score_calc_failures` log
metric, redeploy `dashboard-api` against the previous dependency lock, and drop the
`ciCloudRunIamSetter` custom role.

**Recommendation:** leave them out. The packet's Observability claim for this repo rests on the
metrics stories (10-3, 10-5, 10-6) and the runbook
`documentation/runbooks/monitoring-alert-policies.md`, all of which are on `main` — it does not
need the deployed alert policies. If you *do* want to re-land them, that is a deploy exercise with
a real apply behind it, not a packet task, and it should happen before the packet is submitted
rather than during review.

**One packet consequence to accept:** the coverage matrix lists Observability's primary evidence as
DLA's metrics stories rather than the alert policies. That is accurate as `main` stands.

---

## 4. Optional: index the runbooks

`documentation/runbooks/` holds ten runbooks with no index, and the README does not link them. A
six-line `documentation/runbooks/README.md` — one row per runbook, what it covers, and whether it
is *applied* or *deferred with a reversal condition* — makes *Operational Documentation* citable
with one link instead of ten. Worth ten minutes; skip if time is short, since the directory
listing is legible on its own.

---

## Order of operations

```bash
git cherry-pick 0346350            # 1 — blocking, zero risk
git cherry-pick 5871127 3d57f00    # 2 — after reading the backend.tf hunk
git push origin main
gh run list --limit 6              # confirm green before citing CI in the packet
```

Changes 1 and 2 are mechanical and I can run them on request. Change 3 is a decision that is
already made correctly; change 4 is discretionary.
