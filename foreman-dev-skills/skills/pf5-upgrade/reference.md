# PF5 Upgrade — Reference

Companion to [SKILL.md](SKILL.md). Read when mapping components, migrating legacy UI, or documenting an AI-assisted upgrade.

---

## Canonical docs (read before inventing patterns)

| Topic | Location |
|-------|----------|
| PF3→PF5 process | `foreman/developer_docs/pf3-to-pf5-migration-guide.asciidoc` |
| Component layout, registry, ERB, `reactProps` | `foreman/developer_docs/adding-new-components.asciidoc` |
| Plugin reuse / mounting core components | `foreman/developer_docs/plugins.asciidoc` |
| RTL / Capybara / mocking / Enzyme replacement | `foreman/developer_docs/ui-testing-guidelines.asciidoc` |
| Legacy assets JS + `tfm` store bridges | `foreman/developer_docs/legacy-js.asciidoc` |
| Where to put state | `foreman/developer_docs/state-management.asciidoc` |
| PR expectations | `foreman/developer_docs/pr_review.asciidoc` |
| Upstream tree | https://github.com/theforeman/foreman/tree/develop/developer_docs |
| PF5 component API (use this, not PF6) | https://v5-archive.patternfly.org/ |

**Note:** `legacy-js.asciidoc` documents the Rails assets pipeline and `tfm` — it does **not** document Angular. Angular may appear in some plugins; use Path B replacement goals without assuming Foreman core Angular docs exist.

**Note:** `adding-new-components.asciidoc` links to `patternfly.org` for “does this component exist?” — during PF5 work, verify equivalents on **v5-archive** instead (main site is PF6).

---

## Foreman reuse hotspots

Search these before adding new UI (paths under `webpack/assets/javascripts/react_app/`).

**Index/list pages:** start with **`TableIndexPage`** or **`PageLayout`** — do not hand-roll pagination, search, breadcrumbs, and toolbar from scratch when these cover the use case.

| Need | Look first |
|------|------------|
| **Index/list page** (API-backed table, search, pagination, create/export) | `components/PF4/TableIndexPage/TableIndexPage` — bundles `PageLayout`, `SearchBar`, `Pagination`, toolbar, empty state. Copy `routes/Models/ModelsPage` or `components/HostsIndex` before composing `Pagination` + `SearchBar` yourself |
| **Page chrome** (title, breadcrumbs, toolbar, optional search — no standard index table) | `routes/common/PageLayout/PageLayout` — use for forms, detail pages, or custom content; pass `children` instead of rolling header/toolbar markup |
| Page breadcrumbs only | `components/BreadcrumbBar` (usually via `PageLayout` / `TableIndexPage` `breadcrumbOptions`) |
| Confirm destroy / dangerous action | `components/ConfirmModal` (also `foreman_tools` `openConfirmModal` for legacy callers) |
| Pagination | `components/Pagination` — **only** when not using `TableIndexPage` / `PageLayout` (they already include it) |
| Search | `components/SearchBar` — **only** when not using `TableIndexPage` / `PageLayout` (they already include it) |
| Toasts | `components/ToastsList` |
| Loading / empty | `components/Loading`, PF5 `EmptyState` compositions in existing pages |
| Hosts patterns | `components/HostDetails`, `components/HostsIndex`, `components/hosts` |
| Modal composition | `components/ForemanModal` (context for modal internals), or PF5 `Modal` directly — avoid new thin wrappers |
| Tables | `@patternfly/react-table` via `TableIndexPage` when possible; otherwise copy recent PF5 table patterns — avoid deprecated `components/common/table` |
| Test helpers | `common/rtlTestHelpers.js` (`rtlHelpers`), `common/testInitialReduxStore.js` |
| Deprecation | `common/DeprecationService.js` (re-exported from `foreman_tools.js` → `tfm.tools`) |

Plugins: three reuse modes per `plugins.asciidoc` — mount registered components from ERB; import via Webpack/`foremanReact` (e.g. `foremanReact/components/PF4/TableIndexPage/TableIndexPage`, `foremanReact/routes/common/PageLayout/PageLayout`); do not vendor copies of core components.

---

## PF3 → PF5 mapping (common)

Not exhaustive; always confirm on v5-archive. Names/props differ; prefer composition over forcing old APIs.

| PF3 / old pattern | PF5 direction |
|-------------------|---------------|
| `Button` (`patternfly-react`) | `Button` from `@patternfly/react-core` (`variant`, `isDisabled`, …) |
| `Modal` | `Modal`, `ModalVariant` (small/medium — avoid magic pixel widths) |
| `Form`, `FormGroup`, `FormControl` | `Form`, `FormGroup`, `TextInput`, `TextArea`, `FormSelect`, … |
| `Checkbox` / `Radio` | PF5 `Checkbox` / `Radio` (event/`onChange` signatures differ) |
| `Alert` | `Alert`, `AlertActionLink` / `AlertActionCloseButton` as needed |
| `Row` / `Col` (BS grid) | `Grid` / `GridItem`, or `Flex` / `FlexItem` |
| `Spinner` / loading markup | `Spinner`, `Bullseye`, existing `Loading` |
| Empty illustration blocks | `EmptyState`, `EmptyStateHeader`, `EmptyStateBody`, `EmptyStateFooter` |
| Tabs via `Nav` hacks | PF5 `Tabs` / `Tab` (avoid heavy CSS overrides) |
| Icon + text buttons | PF5 `Button` + `@patternfly/react-icons` |
| Dropdowns | PF5 dropdown patterns or `@patternfly/react-templates` where Foreman already does |
| Data tables | `@patternfly/react-table` — preserve sort/pagination semantics |
| Tooltip / OverlayTrigger | PF5 `Tooltip`, `Popover` |
| Truncation CSS | `<Truncate content={…} />` when appropriate |
| `pf-c-*` classes | Prefer components that emit `pf-v5-c-*` — do not hand-maintain old PF3 class strings |

**Prop gotchas:** controlled vs uncontrolled inputs; `onChange` shapes; `isDisabled` / `isOpen` naming; icon props expect elements, not class strings.

---

## New React component layout (from docs)

```
components/<COMPONENT_NAME>/
  <COMPONENT_NAME>.js            # prefer ≤100 lines; split if larger
  <COMPONENT_NAME>.scss          # if needed
  <COMPONENT_NAME>.fixtures.js
  <COMPONENT_NAME>.test.js       # or __tests__/
  components/                    # nested pieces
  index.js                       # optional: Redux connect / API wiring
```

ERB mount after registry entry:

```erb
<%= react_component('MyComponent', id: record.id, url: some_path) %>
```

Legacy root prop update (limited cases only — re-renders the tree):

```js
var el = document.getElementById('…').getElementsByTagName('foreman-react-component')[0];
var newProps = el.reactProps;
newProps.errorText = '…';
el.reactProps = newProps;
```

---

## Legacy JS / ERB → React checklist

```
- [ ] Listed templates, assets JS, and data/permissions for the page
- [ ] Designed React component tree + props from ERB
- [ ] Registered component in componentRegistry.js
- [ ] Mounted with react_component in the ERB (or plugin view)
- [ ] Reused Foreman/PF5 building blocks (no Bootstrap page clone in divs)
- [ ] Removed or gated legacy JS/templates after parity check
- [ ] No jQuery writing into the React subtree
- [ ] Redux only via actions if legacy still dispatches (`tfm.store`)
- [ ] RTL and/or Capybara per ui-testing-guidelines
- [ ] i18n on all new strings
- [ ] Documented QE selectors / ouiaId changes
```

If the surface is **Angular** (plugin-specific): same checklist; replace controllers/templates with React — do not restyle Angular in place.

**Anti-patterns**

- Restyling legacy/Angular with PF5 CSS class names only
- Keeping business logic in legacy services that the new React view still drives via DOM events
- Global CSS to “make Bootstrap look like PF5”
- New wrapper components that only re-export a PF5 widget
- Updating React DOM from jQuery (`legacy-js.asciidoc`)

---

## CSS rules of thumb (migration guide)

- Audit in the running app; delete only rules that no longer match DOM, duplicate PF5, or patched PF3 quirks.
- Scope under a component root class/id; specific selectors only.
- Prefer PF5 variables (`--pf-v5-global--*`) and spacing utilities.
- Only delete from a **global** stylesheet when the rule is clearly component-specific.
- Never bare `label { }`, `input { }`, or unscoped utility classes that affect other pages.
- Watch mixed `pf-c-` vs `pf-v5-c-` after find-replace.

---

## Testing (ui-testing-guidelines)

**RTL when:** props-driven, self-contained, no routing/API side effects that need a real browser.

**Capybara when:** full page/route, real API/FactoryBot data, multi-step flows, many interacting nested components.

**Helpers (actual API):**

```js
// core
import { rtlHelpers } from '../../../common/rtlTestHelpers';
const { renderWithStore, renderWithStoreAndI18n } = rtlHelpers;

// plugins
import { rtlHelpers } from 'foremanReact/common/rtlTestHelpers';
```

**Mocking:** API only by default; prefer store initial state via `renderWithStore` over mocking selectors. Avoid mocking children/`foremanReact` unless unavoidable. Prefer `userEvent` over `fireEvent`. Assert with `toBeInTheDocument()` on user-visible output — not CSS class names.

**PF3 + old Enzyme tests:** migrate the component to PF5 first, then write RTL; if the component is unused, delete it and its tests instead of rewriting.

### Commands (always from Foreman root)

```bash
npm run lint
npm run lint -- --fix          # eslint fix; intentional scope only
npm run test
npm run test -- <path-or-name>  # e.g. BreadcrumbBar.test.js

npm run lint:plugins <plugin-name>
npm run test:plugins <plugin-name>   # omit name → all plugins; failures listed at end
```

(`npm test` also works as an npm script alias; prefer `npm run test` to match ui-testing / migration guide.)

---

## AI upgrade session log (template)

Use when documenting research / scale-up recommendations:

```markdown
## Target
- Page/section:
- Repo (core vs plugin):
- Legacy stack: PF3 React | assets JS / ERB | Angular (plugin) | mixed

## Baseline inventory
- Components/widgets:
- Behaviors to preserve:
- Reuse candidates found in core:

## Effort
- Start → first lint/tests green:
- Extra human interventions:
- Reverted or rewritten AI hunks:

## Outcomes
- Structural PF5 substitutions (no 1:1):
- CSS removed (approx):
- Tests: Enzyme removed? RTL/Capybara added?
- Defects in self-review / CI:

## Recommendations
- What AI handled well:
- What needed a human (API shape, permissions, QE, design):
- Good next candidate pages:
```

### Metrics definitions

| Metric | How to measure |
|--------|----------------|
| Time saved | Compare to similar prior manual PF5 PR if known; else record absolute hours |
| Code quality | Lint/CI pass; reviewer critical comments count; reuse ratio (new components vs core imports) |
| Developer effort | Interventions + time in visual QA + test fixing |
| Validation | Migration guide checklist + `foreman-ui-reviewer` on the PR |

---

## PR description extras for migrations

- Screenshots of **all** affected views (empty, error, long text if relevant).
- Bullet list of non-1:1 PF substitutions and why.
- Note plugin impact if shared exports, Slot & Fill, or ouiaIds changed.
- Redmine: `Fixes #XXXXX - …` per Foreman commit rules when committing.
