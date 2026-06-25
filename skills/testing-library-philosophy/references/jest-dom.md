# Assertions that express intent (jest-dom)

## Make the assertion read like the requirement

`@testing-library/jest-dom` adds custom DOM matchers so an assertion states *what* it
verifies in user terms instead of poking at raw DOM properties. The judgment here is
about **clarity of intent**: choose the matcher that names the behavior you care
about, so the test reads as the requirement and a failure message explains itself.

Compare the intent conveyed:

- `expect(button).toBeDisabled()` says "the user can't activate this."
- `expect(button.disabled).toBe(true)` says "this object has a property set" — the
  reader has to translate that back into user meaning, and the failure message is a
  bare `true !== false`.

The first is self-documenting and produces a human-readable failure; the second is
mechanical DOM-poking. Prefer matchers that describe the user-facing fact.

## Representative matchers and what they assert

- `toBeVisible` — the user can actually see it (accounts for hidden ancestors, etc.).
- `toBeInTheDocument` — the element is present in the document.
- `toHaveTextContent` — it shows the expected text.
- `toHaveAccessibleName` — it exposes the expected accessible name to assistive tech.
- `toBeDisabled` / `toBeEnabled` — its interactability from the user's point of view.
- `toBeEmptyDOMElement` — it has no rendered content.

Reach for the most specific matcher that captures the behavior, rather than asserting
on an attribute that merely happens to correlate with it.

## Good / Bad in practice

```js
// ❌ Bad — mechanical DOM-poking; opaque failure ("true !== false")
expect(button.disabled).toBe(true)
expect(input.value).toBe('hello')
expect(el.textContent).toContain('Welcome')
expect(node).not.toBeNull() // "is it on the page?" expressed as a null check

// ✅ Good — intent-revealing; failures read in user terms
expect(button).toBeDisabled()
expect(input).toHaveValue('hello')
expect(el).toHaveTextContent('Welcome')
expect(node).toBeInTheDocument()
```

```js
// ❌ Bad — checking a class to infer visibility couples to styling
expect(modal.className).toContain('is-open')

// ✅ Good — assert the user-facing fact directly
expect(modal).toBeVisible()
```

## Presence/absence: match the matcher to the query variant

Some matchers pair with `queryBy*`, not `getBy*`. To assert that something is *not*
present, query with `queryBy*` (which returns `null`) and assert
`expect(thing).not.toBeInTheDocument()`. You can't express absence with `getBy*`
because it throws before your assertion runs. (Which variant to use is a deterministic
concern the linter helps with via `prefer-presence-queries`; the *intent* — proving
absence vs. presence — is yours to get right.)

## Don't assert nothing

A `getBy*` query throws when the element is missing, so it can feel like the query
itself is the test. Still write the explicit assertion: it communicates intent to the
next reader and makes the test's purpose obvious rather than relying on a side effect
of the query throwing.

## Enforce the matchers automatically: eslint-plugin-jest-dom

The "use the semantic matcher, not the raw property" choice has a *deterministic*
core — a linter can recognize when an assertion reimplements a matcher by hand and
autofix it. That belongs to
[`eslint-plugin-jest-dom`](https://github.com/testing-library/eslint-plugin-jest-dom),
the jest-dom companion to `eslint-plugin-testing-library`. Recommend adopting it
alongside the main plugin so this skill can focus on *which* fact to assert, while
the linter enforces *how* to phrase it.

What it catches (most rules are auto-fixable):

- `prefer-in-document` — `toBeInTheDocument()` instead of `toHaveLength(1)` /
  `not.toBeNull()` style presence checks.
- `prefer-to-have-text-content` — `toHaveTextContent()` instead of asserting on
  `.textContent`.
- `prefer-to-have-value` — `toHaveValue()` instead of asserting on `.value`.
- `prefer-to-have-attribute` — `toHaveAttribute()` instead of `getAttribute()`
  comparisons.
- `prefer-to-have-class` — `toHaveClass()` instead of inspecting `className`.
- `prefer-to-have-style` — `toHaveStyle()` instead of reading `.style`.
- `prefer-enabled-disabled` — `toBeEnabled()` / `toBeDisabled()` instead of
  asserting on the `disabled` property.
- `prefer-checked` — `toBeChecked()` instead of asserting on `checked`.
- `prefer-required` — `toBeRequired()` instead of asserting on `required`.
- `prefer-empty` — `toBeEmptyDOMElement()` instead of empty-content checks.
- `prefer-focus` — `toHaveFocus()` instead of comparing against
  `document.activeElement`.

Setup (flat config) — enable both plugins together:

```bash
npm install --save-dev eslint-plugin-jest-dom eslint-plugin-testing-library
```

```js
// eslint.config.js
import testingLibrary from 'eslint-plugin-testing-library'
import jestDom from 'eslint-plugin-jest-dom'

export default [
  testingLibrary.configs['flat/react'], // your framework's config
  jestDom.configs['flat/recommended'],
]
```

Because most of these rules autofix, adopting the plugin retroactively cleans up a
codebase's raw-property assertions in one pass — leaving this skill's prose to cover
the genuine judgment: *which* matcher captures the behavior you actually care about.

## Further reading

- Testing Library — jest-dom: <https://testing-library.com/docs/ecosystem-jest-dom/>
- Matcher reference: <https://github.com/testing-library/jest-dom>
- eslint-plugin-jest-dom: <https://github.com/testing-library/eslint-plugin-jest-dom>
