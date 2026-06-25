---
name: skill-ab-eval
description: Run an A/B usefulness check on an agent-skill — does it actually change agent behavior? Spawns N control agents (no skill) and N treatment agents (with the skill) on the same task, scores their output blind against a rubric derived from the skill's own claims, and reports control-vs-treatment. Use when asked to "check if a skill is useful", "A/B test a skill", "measure a skill's effect", "skill 有用性チェック", or to validate a skill before/after editing it. Project-local tool, not for distribution.
---

# Skill A/B usefulness check

Measures whether a target skill actually shifts agent behavior, instead of guessing.
A capable model already *knows* most best practices; the open question is whether it
*deploys* them by default. This harness answers that empirically: same task, with and
without the skill, scored blind.

The core finding it surfaces is usually **variance reduction**, not a uniform lift —
so always report the **regression rate** (how often the no-skill arm falls into a bad
pattern), not just the mean.

## Inputs (ask if unclear)

- **Skill under test** — path to the `SKILL.md` (+ its `references/`).
- **Representative task** — a prompt + artifact the skill is meant to influence (e.g.
  "write tests for this component", "review this code", "design this API"). If none is
  given, derive one from the skill's domain. Pick something with **room to diverge** —
  not so trivial that every run is identical (see the cleanliness caveat).
- **N per arm** — default **3** each (6 generation agents). Raise for more signal.
- **Models** — generation: default **sonnet** (same for both arms — fairness). Scoring:
  default **opus** (stronger judge).

## Method (follow exactly — the isolation rules are load-bearing)

1. **Build the rubric from the skill itself.** Read the skill under test and extract
   its stated GOOD signals (Non-negotiables, "✅ Good" examples, principles) and BAD
   signals ("⚠️ Bad" / anti-patterns). The rubric is the skill's own claims turned into
   checkable, evidence-quotable signals. See [references/templates.md](./references/templates.md).

2. **Prepare the artifact** in the scratchpad (outside the repo tree the control agents
   might explore). Keep the secret arm→label mapping yourself.

3. **Launch 2N generation agents in parallel** (one message, multiple `Agent` calls):
   - Use **fresh `general-purpose` agents** — **NOT `subagent_type: fork`** (a fork
     inherits your context, which is contaminated by the skill → invalid control).
   - **Same `model` for every agent**, both arms.
   - **Control prompt:** the task + artifact **inlined**, plus an explicit instruction
     to NOT read/list/grep/search any files or explore the repo (work only from the
     prompt). Never mention the skill or its path. → contamination guard.
   - **Treatment prompt:** identical task + artifact, plus "read and follow this skill:
     `<path>` and the references it links."
   - Both arms: "output ONLY the final deliverable" so results are comparable.

4. **Watch for contamination.** When an agent returns, a control agent with many tool
   calls or output that echoes the skill's vocabulary probably found the skill on disk —
   discard and re-run it with a hard "do not access files" instruction. (This happens;
   the default working directory is the repo root.)

5. **Save outputs to neutral, shuffled filenames** (`sample-A…F`) in the scratchpad,
   mixing arms so order doesn't leak which is which.

6. **Score blind.** Launch one scorer agent (opus) that reads the neutral files and
   scores each 0–10 against the rubric, quoting evidence per signal, **without knowing
   the arms**. Scoring rule in [references/templates.md](./references/templates.md).

7. **Map back and aggregate.** Using your secret mapping, compute per arm: **mean**,
   **min**, and **regression count** (samples below a quality threshold, e.g. <7). Note
   *which signals separated* the arms (often the linter-invisible judgment calls).

## Reporting (mandatory caveats — never omit)

Report control vs treatment as a table, then state every caveat that applies:

- **Small n** — with N=3 the regression rate has wide error bars; don't over-claim a %.
- **Generation-model caveat** — results are for the generation model used (e.g. sonnet);
  the user's everyday model may have a different baseline. The *direction* transfers
  better than the absolute numbers.
- **Task-cleanliness bias** — a clean, canonical artifact inflates the baseline (less
  room to go wrong). If control looks "too good", say the task may be too easy and a
  messier artifact would likely widen the gap.
- **Scorer is an LLM**, and a hard 0/10 from a penalty rubric means "wrong philosophy",
  not "broken/unrunnable" — describe quality qualitatively too.

Frame the verdict around **what the skill prevents** (the regressions) and whether those
failures are ones a linter/other tooling would *not* have caught — that's where a
philosophy/judgment skill earns its keep.

## Artifacts

Put all generated files (artifact, `sample-*`, scorer output) under the session
scratchpad so re-runs are clean and the repo stays untouched.
