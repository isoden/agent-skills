# Choosing queries: accessibility-first, and why

The query priority list looks like a rule to memorize, but its value is the
*reasoning*. A linter can nudge you toward `screen` and `findBy`; it cannot tell you
whether the query you wrote reflects how a real person locates the element. That is
the judgment this reference is about.

## The mental model: instruct a manual human tester

Write each step as if you were telling a person to test the app by hand. A human
finds a control by its visible label, its role ("the Submit button"), or its text —
never by a CSS class or its index among siblings. Your queries should locate
elements the same way. When a query can only be expressed in terms the user can't
perceive (a class name, a DOM path), that is a signal the test is coupling to
implementation rather than behavior.

## The priority order and the rationale

Prefer queries higher in this list; drop down only when the ones above genuinely
don't fit.

1. **Accessible to everyone** — works for sighted *and* assistive-technology users:
   - **`getByRole`** — the default for almost everything. It queries the
     accessibility tree and can be narrowed by accessible name, so it matches how
     both users and screen readers perceive the UI. It also handles text split
     across child elements and produces helpful error output.
   - **`getByLabelText`** — ideal for form fields; mirrors how users navigate forms
     by their labels.
   - **`getByPlaceholderText`** — a fallback when there is no label.
   - **`getByText`** — for non-interactive content located by what it says.
   - **`getByDisplayValue`** — form elements by their current value.
2. **Semantic queries** — based on HTML/ARIA attributes with inconsistent assistive-
   technology support: `getByAltText`, `getByTitle` (`title` is least reliable — not
   consistently announced).
3. **Test IDs** — **`getByTestId`**, a deliberate last resort (see below).

The order is not arbitrary: it mirrors how humans actually perceive and operate a
UI. A pleasant side effect is that choosing a higher-priority query simultaneously
exercises your app's accessibility — a good test doubles as an accessibility check,
and reaching for a lower-priority query often reveals missing semantics (a control
with no role or accessible name).

## Lean on the accessibility tree

Because role-based queries read the accessibility tree, they push you toward
accessible markup. Use semantic HTML first (it carries implicit roles —
`<nav>` is `navigation`, `<button>` is `button`), and add ARIA only to *augment*
what semantics can't express, never to paper over non-semantic markup. When you're
unsure which role an element exposes, log the roles present in the rendered output to
discover the right `getByRole` query rather than guessing (these debug aids are for
authoring only — remove them before committing, since the linter flags leftover ones).

## When is `data-testid` legitimate?

A test ID is invisible to users, so it sits at the bottom of the list — but it is a
sanctioned **escape hatch**, not a failure. Reach for it only when role, label, and
text genuinely cannot identify the element (e.g. a purely presentational container
with no semantic meaning, or a dynamic region you must target unambiguously).

When you do use one, prefer it over a CSS class or DOM index precisely because it is
**intentional metadata**: it makes the test-to-code relationship explicit, can be
safely removed when knowingly no longer needed, and won't break when styling
changes. A class selector hijacks something that exists for another purpose; a test
ID says "a test depends on this" out loud.

Conversely, reaching for a test ID *instead of* visible text when text would work is
a smell: it skips verification of the content (and any i18n) that users actually see,
hiding regressions that should have failed the test.

## Good / Bad in practice

```js
// ❌ Bad — coupled to markup a user can't perceive; breaks on restyle/reorder
const button = container.querySelector('.btn-primary')
const second = container.querySelectorAll('.btn')[1]

// ✅ Good — found the way a user (and a screen reader) finds it
const button = screen.getByRole('button', { name: /save/i })
const email = screen.getByLabelText(/email/i)
```

```js
// ❌ Bad — test ID where visible text would work; skips content/i18n verification
screen.getByTestId('submit-button')

// ✅ Good — asserts the user-facing label too
screen.getByRole('button', { name: /create account/i })

// ✅ Acceptable — last resort for a non-semantic element, as intentional metadata
screen.getByTestId('chart-canvas') // a purely presentational element with no implicit role or accessible name
```

```js
// ❌ Bad — queryBy for a positive existence check (null-deref, unhelpful failure)
expect(screen.queryByRole('alert')).toBeInTheDocument()

// ✅ Good — getBy throws with a DOM dump when present-check fails;
//          queryBy only for asserting absence
expect(screen.getByRole('alert')).toBeInTheDocument()
expect(screen.queryByRole('alert')).not.toBeInTheDocument()
```

## Query variants — pick by intent

Choosing the variant is a statement of intent (whether to await something is the
linter's job; *which* variant communicates your meaning is yours):

- **`getBy*`** — the element should already be there; throws (with a DOM dump) if not.
- **`queryBy*`** — returns `null` when absent; use it to assert that something is
  *not* present. Don't use it for positive existence checks — `getBy*`'s throw is
  more informative.
- **`findBy*`** — the element will appear asynchronously; retries until it does.

## Further reading

- Testing Library — About Queries (priority): <https://testing-library.com/docs/queries/about/>
- Testing Library — Accessibility API: <https://testing-library.com/docs/dom-testing-library/api-accessibility/>
- Kent C. Dodds — Making Your UI Tests Resilient to Change: <https://kentcdodds.com/blog/making-your-ui-tests-resilient-to-change>
