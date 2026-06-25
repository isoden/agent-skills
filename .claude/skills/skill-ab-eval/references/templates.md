# Templates: prompts, rubric, scoring

Copy and fill these. Replace `{{...}}` placeholders. Keep both arms byte-for-byte
identical except for the marked block.

## Control prompt (no skill)

```
{{TASK INSTRUCTION — what to produce}}

EVERYTHING you need is in this prompt. Do NOT read, list, grep, or search any files on
disk, and do NOT explore the repository — work purely from the material below.
(Using file/search tools is out of scope for this task.)

{{ARTIFACT — inline the full code/spec/content here}}

Do not ask questions — make reasonable assumptions. Output ONLY the final deliverable
({{format, e.g. "the complete file in one code block"}}), plus a 2–3 line note on the
key choices you made and why. Your entire final message is the deliverable.
```

## Treatment prompt (with skill)

Identical to the control prompt, with this block inserted at the TOP (and the
"do NOT read files" sentence removed — the treatment must read the skill):

```
FIRST, read and follow this skill and the reference files it links. Apply its guidance:
- {{ABSOLUTE PATH TO SKILL.md}}
- and the files under {{.../references/}} that it links to.
```

## Launch settings

- `subagent_type: "general-purpose"` (never `fork`).
- `model`: same value for all 2N generation agents (default `sonnet`).
- Send all 2N `Agent` calls in **one message** so they run concurrently.
- Keep a private mapping, e.g. `A=control, B=treatment, C=control, D=treatment, …`,
  shuffled so arms don't cluster.

## Rubric (derive signals from the skill under test)

Turn the skill's claims into binary, evidence-quotable signals.

```
GOOD signals (from the skill's Non-negotiables / "✅ Good" / principles):
- G1 {{...}}
- G2 {{...}}
- ...

BAD signals (from the skill's "⚠️ Bad" / anti-patterns; presence is negative):
- B1 {{...}}
- B2 {{...}}
- ...
```

Prefer signals that are **judgment calls a linter/static tool can't enforce** — those
are where a philosophy skill can actually move the needle, and where separation is most
meaningful.

## Scorer prompt (blind)

```
You are a strict, objective reviewer. {{N*2}} independently-written deliverables for the
SAME task are at {{scratch dir}}/sample-A … sample-{{last}}. Read all of them. You do NOT
know who wrote each — score each only on its own content, with quoted evidence.

Context of the task: {{1–2 sentences + the artifact, so the scorer knows "correct"}}.

Score each on these signals (mark present/absent, quote a snippet as proof):
{{paste the GOOD/BAD rubric above}}

For each sample: list GOOD present (G# + evidence), BAD present (B# + evidence), a 1–2
sentence verdict, and a 0–10 score. Scoring rule: start at 10; −3 per BAD signal present;
−2 per missing core GOOD signal; structure/bonus signals noted but don't change score.

Then output a compact table: Sample | Score | {{2–3 key discriminating columns, e.g.
"mocking approach", "impl-detail assertions Y/N", "structure"}}.

Be rigorous and concise. Return only the scored report.
```

Use a strong model for the scorer (`model: "opus"`).

## Aggregation

Map each sample back to its arm and compute, per arm:

- **mean** and **min** score
- **regression count** = samples scoring below the quality threshold (default `< 7`)
- the signal(s) that most separated the arms

A skill "earns its keep" when the treatment arm has **fewer/no regressions** than
control, especially on signals no linter would have caught. Equal means with the same
regression count = the skill added little for this task (try a messier artifact before
concluding).
