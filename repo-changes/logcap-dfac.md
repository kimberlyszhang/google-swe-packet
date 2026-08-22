# logcap-dfac — repo changes for the packet

**Repo:** https://github.com/GPS-Demos/logcap-dfac
**Packet role:** should become **primary** for Model Selection & Tuning (the hybrid LLM/
deterministic menu pipeline); secondary for Modularity, Configuration Management, Resource
Efficiency, AI Cost Management and Testing.
**Verified against `origin/main` @ `7d29964` (2026-05-21) on 2026-08-22.**

`main` is current and clean — this repo needs the least work of the five. The one real problem is
that its best architectural evidence is undocumented, and the document that would describe it
points at a file that no longer exists.

---

## 1. Document the five-phase menu pipeline — **high value**

**Status quo.** `docs/TECHNICAL.md` § 4 cites
`[generateMenu.ts:2475](functions/src/generateMenu.ts#L2475)` for the menu model configuration.
That file is gone. It was refactored into a directory:

```
functions/src/generateMenu/
  index.ts            orchestrator
  phase1-pick.ts      Gemini picks dishes
  phase2-servings.ts  servings computed + leftovers injected
  phase3-filter.ts    inventory and expiry validation
  phase4-solve.ts     the solve
  phase5-assemble.ts  assembly
  deterministic.ts    deterministic rotation fallback
  json-repair.ts      strips Gemini's stray bytes around the JSON object
  bigquery.ts  prompts.ts  swaps.ts  types.ts
```

and the model IDs moved to `functions/src/config/models.ts`, where all four are env-overridable
(`CHAT_MODEL`, `MENU_MODEL`, `VISION_MODEL`, `AI_INSIGHTS_MODEL`).

**Why it matters.** The coverage matrix has LOGCAP as primary for nothing, and that is a
documentation problem rather than a substance problem. What this repo actually demonstrates —
better than any other in the slate except FINRA — is **the division of labour between a model and
code**: Gemini picks dishes (a judgment call over recipes, patron history and events), and
TypeScript computes servings, validates inventory and expiry, and solves feasibility (arithmetic
with a right answer). `deterministic.ts` is the fallback when the model is unavailable, so the
capability degrades to a working rotation rather than to nothing. `json-repair.ts` exists because
production LLM output needs a parser that tolerates preamble and trailing bytes.

None of that is described anywhere a reviewer will look.

**The change.** Rewrite `docs/TECHNICAL.md` § 4 as **"Menu generation — what the model decides and
what the code decides"**:

- A phase table: phase → what it does → model or deterministic → the file.
- The current model table, with `config/models.ts` as the source and a note that each is
  env-overridable (that is the *Configuration Management* claim, and it is one line).
- Two paragraphs on the fallbacks: `deterministic.ts` when the model call fails, `json-repair.ts`
  when the response is not clean JSON.
- Fix the four dead line-anchored links in the existing model table while you are in there.

Then add a short **Architecture** pointer in `README.md` § AI Features to the new section — the
README currently describes menu generation as a bulleted list of inputs, which reads as a feature
description rather than as a design.

---

## 2. Delete the `kim-dev2` branch — **trivial**

Two commits from 2026-03-16, local only (nothing matching on `origin`), and self-cancelling:
`6e8e070 fix: include incoming shipment inventory in recipe feasibility filter` followed ten
minutes later by `6b41676 revert: restore generateMenu.ts to pre-incoming-inventory state`. Both
patch `functions/src/generateMenu.ts`, the file that no longer exists — and incoming inventory is
in `main` today anyway, via `fetchIncomingInventory` in `generateMenu/bigquery.ts`.

Net effect zero, applies to a deleted file, superseded by shipped work:

```bash
cd ~/Documents/Google/logcap-dfac
git branch -D kim-dev2
```

---

## 3. Recount the endpoints — **trivial**

`README.md` says "15 HTTP endpoints"; `main` exports 16 handlers (11 `onRequest` in `index.ts`,
plus `generateMenu`, three from `leftovers.ts`, and `chatFeedback`). Recount and fix, or drop the
number. A miscount in the first screen of the architecture diagram is the kind of thing a reviewer
notices and then wonders what else drifted.

---

## 4. Not recommended: adding CI

This repo has no `.github/workflows/`, despite 38 test files including BigQuery SQL tests under
`bigquery/tests/`. Adding a workflow would be real, useful engineering — and it is **not** worth
doing for the packet. DLA already anchors CI/CD & Deployment with six workflows including a
Terraform plan gate, and a workflow added in August 2026 to a repo whose last real commit is May
2026 reads as packet dressing rather than as practice. Cite LOGCAP for what it genuinely shows:
the hybrid pipeline, the shared type contract, and the partition/cluster migration.

---

## Order of operations

1. Rewrite `docs/TECHNICAL.md` § 4 and fix the dead anchors.
2. `git branch -D kim-dev2`.
3. Recount the endpoints in the README.

All three are small. I can draft § 4 for review and run items 2 and 3 on request.
