# Delegate deterministic checks to eslint-plugin-testing-library

This skill deliberately does **not** police mechanical, deterministic mistakes — the
kind a static analyzer can detect with certainty from the syntax tree. Those are
better enforced automatically and consistently by
[`eslint-plugin-testing-library`](https://github.com/testing-library/eslint-plugin-testing-library).
A linter never forgets, runs on every save, and gives an exact line number. Prose
guidance cannot compete with that for rules that are truly black-and-white.

## What the linter owns (do not re-teach these as judgment)

The plugin ships framework-specific recommended configs. Representative rules and
the class of mistake each one catches:

- **Async correctness** — `await-async-queries`, `await-async-utils`,
  `await-async-events`, `no-await-sync-queries`, `no-promise-in-fire-event`.
  Forgetting to `await` a `findBy*`/`waitFor`, or awaiting something synchronous.
- **Query hygiene** — `prefer-screen-queries` (use `screen` over destructured
  queries), `prefer-find-by` (use `findBy*` instead of `waitFor` + `getBy*`),
  `prefer-presence-queries`, `prefer-query-by-disappearance`,
  `prefer-query-matchers`.
- **Don't reach into the DOM** — `no-node-access`, `no-container`. Discourages
  `container.querySelector` and traversing DOM nodes directly.
- **Imports & setup** — `no-dom-import` (import from the framework package, not the
  low-level DOM package), `no-manual-cleanup`, `no-render-in-lifecycle`,
  `no-unnecessary-act`.
- **Interaction** — `prefer-user-event` (prefer `user-event` over `fireEvent`).
- **Debug leftovers** — `no-debugging-utils` (stray `screen.debug()`,
  `logRoles`, etc.).
- **Naming** — `render-result-naming-convention`, `consistent-data-testid`.

If you find yourself wanting to write "always await `findBy`" or "use `screen`
instead of destructuring," stop — that belongs to the linter, not to this skill.

## What this skill owns instead

The **judgment** a linter cannot make:

- *What* is worth testing, and how much (see [guiding-principles.md](./guiding-principles.md)).
- *Whether* a given query reflects how a user finds an element, vs. merely passing
  (see [queries.md](./queries.md)).
- *Whether* an assertion proves the behavior you care about.
- *Whether* a test is coupled to implementation details that will break on refactor
  (see [avoiding-implementation-details.md](./avoiding-implementation-details.md)).

## Recommendation to surface to the user

When helping someone set up or review a Testing Library codebase, recommend they
adopt `eslint-plugin-testing-library` with the recommended config for their
framework, so the deterministic rules are enforced automatically and this skill can
focus on the parts that require human/agent judgment.

```bash
npm install --save-dev eslint-plugin-testing-library
```

```js
// eslint.config.js (flat config)
import testingLibrary from 'eslint-plugin-testing-library'

export default [
  // pick the config matching your framework: react / dom / vue / angular / marko
  testingLibrary.configs['flat/react'],
]
```

Pair it with the jest-dom companion plugin, which autofixes raw-property assertions
into semantic matchers — see the dedicated section in
[jest-dom.md](./jest-dom.md#enforce-the-matchers-automatically-eslint-plugin-jest-dom).

## Further reading

- eslint-plugin-testing-library — <https://github.com/testing-library/eslint-plugin-testing-library>
- Supported rules list — <https://github.com/testing-library/eslint-plugin-testing-library#supported-rules>
