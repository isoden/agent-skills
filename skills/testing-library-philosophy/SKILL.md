---
name: testing-library-philosophy
description: Philosophy and judgment-level best practices for Testing Library (React Testing Library, user-event, jest-dom) plus Kent C. Dodds' testing ideas. Use when writing, reviewing, or designing frontend component/integration tests — deciding what to test, choosing accessible queries, simulating real user interaction, asserting on user-facing behavior, and avoiding implementation-detail coupling. Defers deterministic, mechanically-checkable rules to eslint-plugin-testing-library.
---

# Testing Library Philosophy

This skill captures the *reasoning* behind good Testing Library tests — the
judgment a static linter cannot make. It does **not** restate mechanical rules
(await your async queries, use `screen`, prefer `findBy`, …); those are
deterministic and belong to `eslint-plugin-testing-library`
(see [references/eslint-plugin.md](./references/eslint-plugin.md)). Recommend that
plugin to the user and spend your attention on the parts that require thought.

## The one principle everything follows from

> "The more your tests resemble the way your software is used, the more confidence
> they can give you." — Kent C. Dodds / Testing Library guiding principles

The goal of a test is **confidence** that a real user can do a real thing.
Resemblance to actual usage is the proxy for that confidence. Almost every
recommendation below is a corollary: test through the user-facing surface (the
accessibility tree and visible DOM), drive it with realistic interactions, assert
on observable outcomes, and stay decoupled from internals so refactors don't lie to
you.

## Non-negotiables (the linter won't catch these)

These are the highest-leverage judgment calls — the ones `eslint-plugin-testing-library`
cannot flag, and the ones tests most often get subtly wrong even when they "pass."
Apply them by default; the linked references explain the why.

- **Don't mock the module under test.** Mocking your own `./api` (or other code you're
  testing) — e.g. `vi.mock('./api')` / `jest.mock('./api')` — tests a fake, not your
  app. Mock at the **network boundary** (MSW) instead. → [mocking.md](./references/mocking.md)
- **Don't assert on implementation details.** No assertions on *internal* collaborators
  you mocked out (`expect(mockedApi).toHaveBeenCalledWith(...)`), no internal state, no
  private methods — assert what the user observes. (Asserting that a component invoked
  its own public **callback prop**, e.g. `onSubmit`, is fine: that's its developer-user
  contract, not an internal.) → [avoiding-implementation-details.md](./references/avoiding-implementation-details.md)
- **Refactor-resilience is the test of a test.** If a behavior-preserving refactor
  would break it, it's coupled to implementation — fix the coupling.
- **Query the way a user finds things:** role → label → text, with `getByTestId` as a
  last resort. → [queries.md](./references/queries.md)
- **Drive interactions like a user** (`userEvent`, not a single synthetic event), and
  **assert in user terms** (jest-dom matchers, not raw DOM properties).

These restate, in their crispest form, the most error-prone parts of the references
below — promoted here so they apply even at a glance.

## How to apply this skill

When writing or reviewing a frontend test, work through these questions. Each links
to a reference with the full reasoning.

1. **Is this worth testing, and at what layer?** Chase use-case coverage, not line
   coverage; favor integration tests. → [references/guiding-principles.md](./references/guiding-principles.md)
2. **Does the query reflect how a user finds the element?** Prefer role/label/text
   over test IDs; understand *why* the priority order exists rather than memorizing
   it. → [references/queries.md](./references/queries.md)
3. **Does the interaction resemble what a user actually does?** Use `user-event`'s
   intent-level actions, which also enforce that the element is really
   interactable. → [references/user-event.md](./references/user-event.md)
4. **Does the assertion express the requirement in user terms?** Use jest-dom
   matchers so the test reads as the behavior it verifies. → [references/jest-dom.md](./references/jest-dom.md)
5. **Is the test coupled to implementation details?** If a behavior-preserving
   refactor would break it, the test is brittle — fix the coupling. → [references/avoiding-implementation-details.md](./references/avoiding-implementation-details.md)
6. **Should this be mocked at all, and at which boundary?** Every mock trades away
   confidence; prefer faking the network (MSW) over your own code. → [references/mocking.md](./references/mocking.md)
7. **Is the test readable and maintainable?** Avoid hasty abstraction; use a custom
   render for providers + user-event; write fewer, longer workflow tests. → [references/test-maintainability.md](./references/test-maintainability.md)
8. **Are we falling into a known judgment-level trap?** → [references/common-mistakes.md](./references/common-mistakes.md)

For the deterministic stuff (and the recommendation to adopt it), see
[references/eslint-plugin.md](./references/eslint-plugin.md). Source articles and
docs are collected in [references/further-reading.md](./references/further-reading.md).

## Scope boundary

- **In scope:** philosophy, mental models, and judgment calls — what to test,
  which query semantically fits, when a `data-testid` is a legitimate escape hatch,
  why user-resemblance beats internal-state assertions.
- **Out of scope (delegated to `eslint-plugin-testing-library`):** anything a linter
  can decide from syntax alone — awaiting async queries, `screen` usage,
  `findBy` vs `waitFor`+`getBy`, `no-container`, `no-node-access`, `prefer-user-event`,
  debug leftovers, naming conventions. Recommend the plugin; don't re-teach its rules.
