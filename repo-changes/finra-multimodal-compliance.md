# finra-multimodal-compliance — repo changes for the packet

**Repo:** https://github.com/GPS-Demos/finra-multimodal-compliance
**Packet role:** primary evidence for Agentic Systems, Model Selection & Tuning, LLM Ops &
Evaluation, Domain-Applied AI, AuthN/Z, Graceful Degradation, Scalability, AI Cost Management, AI
Lifecycle Management, Modularity, Configuration Management and Extensibility.
**Verified against `origin/main` @ `e01be07` (2026-08-19) on 2026-08-22.**

This is the strongest repo in the slate and needs the least defending. `main` is current, the
README is already a design document, and the evaluation story is unusually honest — it reports a
red gate, a defect the sweep exposed in its own reporting code, and the noise floor that bounds
what any single sweep can claim. One change is worth making.

---

## 1. Commit the architecture document — **high value**

**Status quo.** 65 source files carry **127 citations** to `architecture.md`, anchored to specific
sections:

```
14 ×  architecture.md#Communication Patterns
11 ×  architecture.md#Naming Patterns
 8 ×  architecture.md#Anti-patterns
 7 ×  architecture.md#Format Patterns
 7 ×  architecture.md#Enforcement Guidelines
 7 ×  architecture.md#Architectural Boundaries
 …
```

`git ls-tree -r origin/main | grep -i architecture` returns nothing. The document exists at
`_bmad-output/planning-artifacts/architecture.md` (489 lines) but `_bmad-output/` is in the global
gitignore, so it has never been pushed.

**Why it matters.** These are not decorative comments — they are the mechanism by which the
codebase explains *why* a rule exists. `packages/shared/src/cost.ts` says cost is computed rather
than generated because the architecture forbids asking a model for a computable value
`[architecture.md#Implementation Patterns & Consistency Rules — Anti-patterns]`, and that it may
not reach into `api`, `worker` or `frontend` `[architecture.md#Architectural Boundaries]`. A
reviewer who follows one of those 127 pointers finds nothing, and the discipline the packet claims
for this repo reads as assertion rather than as a documented contract.

Every anchor referenced in the code exists as a heading in the local file — I checked the six most
cited. So this is a publish, not a rewrite.

**The change.**

```bash
cd ~/Documents/Google/finra-multimodal-compliance
mkdir -p documentation
cp _bmad-output/planning-artifacts/architecture.md documentation/architecture.md
git add -f documentation/architecture.md      # -f: _bmad-output is gitignored, documentation/ is not
git commit -m "docs: publish the architecture document the codebase cites 127 times"
git push origin main
```

Two things to do first:

1. **Read it before pushing** — it is a design document for a customer demo. Confirm it carries no
   customer-confidential material beyond what the README already states (project ID, region,
   dataset names are all public in the README already).
2. **Decide on the reference form.** The citations are bare (`architecture.md#Section`), so a
   reader has to find the file. Either leave them and rely on `documentation/architecture.md`
   being discoverable, or do a mechanical pass rewriting them to
   `documentation/architecture.md#Section`. The second is 127 substitutions across 65 files and
   should be one commit doing nothing else — it is not required for the packet.

**Note the naming collision.** The BMAD template titles the file "Architecture Decision Document",
but it is not a set of ADRs — it is one design document with a decisions section. The packet cites
DLA for Decision Records; this file supports *System Design Artifacts* and *Technical Scope &
Constraints*. Don't let the title imply otherwise in the entry.

---

## 2. The blind-set numbers: cite them as dated, don't re-run — **decision, no code change**

**Status quo.** `documentation/results.md` reports the 26-advertisement blind set at
`pipeline_version 0.5.0`, measured 2026-08-10: case recall 12/13, clause recall 10/14, specificity
2/10, `mustNotFlag` breaches 0, $23.50 for the sweep in 23.5 minutes. The build is now `0.11.1`,
and the README's own Results section labels this **"Blind set — `0.5.0`, 2026-08-10, not
re-measured since."** A newer clean-controls sweep at `0.11.1` (2026-08-12) is published beside it:
specificity 6/10, integrity holds 0, $9.14 across 13 runs.

**Recommendation: leave it and cite it as dated.** The disclosure is already exactly right, and the
document goes further — it publishes the run-to-run noise floor and states that it is larger than
any effect a single sweep can attribute to a build. That is a stronger LLM-Ops artifact than a
fresh number would be, because it demonstrates knowing what the measurement can and cannot support.

The packet entry should therefore say, precisely:

> Recall and cost figures are from the `0.5.0` sweep of 2026-08-10; specificity improved from 2/10
> to 6/10 by `0.11.1` (2026-08-12, clean-controls corpus). The gate remains red by design — the
> answer key is FINRA's, not the pipeline's.

**If you would rather have one current number for both corpora**, the sweep is one command and
about 25 minutes of wall clock:

```bash
cd eval
npx tsx analyse-blind-set.ts --force --concurrency 13
```

Do not run it through the root `npm run` aliases — the README documents that npm eats the flags and
the script then silently reports the previous build's results as if they were new. Budget ~$25 and
re-verify with `SELECT COUNT(*) FROM runs WHERE pipeline_version = '0.11.1'` before quoting
anything. Only worth doing if you want the headline numbers to match the build; the packet is
defensible either way.

---

## 3. Not recommended: adding CI

There is no `.github/workflows/` here. That is a real gap in the abstract and a non-issue for the
packet: this repo's testing claim rests on unit tests beside every module **plus** eight live
verification scripts that exercise real GCP services (`verify:auth`, `verify:batch-progress`,
`verify:compliance-policy`, …) on the stated principle that mocked tests prove parsing while these
prove integration. That is a more interesting claim than a green checkmark, and DLA already anchors
CI/CD. Adding a workflow now would add nothing the packet needs.

---

## Order of operations

1. Read `_bmad-output/planning-artifacts/architecture.md`, then publish it to `documentation/`.
2. Decide re-run vs. cite-as-dated for the blind set (recommendation: cite as dated).
3. Optionally, the 127-reference path rewrite as its own commit.

Item 1 is mechanical once you have read the file and I can run it on request.
