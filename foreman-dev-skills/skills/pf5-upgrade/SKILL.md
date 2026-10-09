---
name: pf5-upgrade
description: >-
  Upgrades Foreman/Satellite UI from PatternFly 3 or legacy JS (assets pipeline,
  Bootstrap ERB; Angular in some plugins) to PatternFly 5 React, reusing Foreman
  core components and following developer docs. Use when migrating PF3,
  patternfly-react, legacy JS, Angular, Bootstrap ERB views, or when the user
  asks to upgrade/migrate UI to PF5, PatternFly 5, or modern React.
---

# PatternFly 5 Upgrade (PF3 / legacy JS → PF5)

Behavior-preserving UI kit migration for Foreman and plugins (Satellite UI). Prefer PF5 components; avoid reinventing layout with raw HTML/CSS.

## Standing rules (always)

1. **Always check developer docs** for guidance — start with local `foreman/developer_docs/`, especially:
   - `pf3-to-pf5-migration-guide.asciidoc`
   - `adding-new-components.asciidoc`
   - `ui-testing-guidelines.asciidoc`
   - `legacy-js.asciidoc` (Rails assets pipeline + `tfm` bridges — not Angular-specific)
   - `plugins.asciidoc` (reusing / mounting core components from plugins)
   - `client-routing.asciidoc` (Rails `react#index` + `react-router` client routes)
   - `state-management.asciidoc`
   - Upstream: https://github.com/theforeman/foreman/tree/develop/developer_docs
2. **Check Foreman core for reusable components** before writing new UI — search `webpack/assets/javascripts/react_app/components/` and `common/` (and `foremanReact/*` from plugins). Extend or compose existing pieces; do not duplicate Pagination, ConfirmModal, SearchBar, ToastsList, Loading, etc.
3. **Use PF5 components and avoid pure HTML/CSS styling** — layout with `@patternfly/react-core` (Flex, Grid, Stack, Bullseye, EmptyState, …). Prefer PF5 tokens/utility classes over custom SCSS. No bare element selectors. No thin PF re-export wrappers.
4. **PF5 docs only for this work** — https://v5-archive.patternfly.org/. Do not use `patternfly.org` (documents PF6). Ignore the PatternFly.org link in `adding-new-components.asciidoc` while doing PF5 migrations.
5. **Parity first** — same workflows, labels, permissions, loading/error/empty states, keyboard behavior, tooltips. Document structural UX changes in the PR.

## When to apply

| Signal | Path |
|--------|------|
| Imports from `patternfly-react`, `pf-c-*` classes, PF3 React | [PF3 React → PF5](#path-a-pf3-react--pf5) |
| Legacy assets JS (`app/assets/javascripts`), jQuery/`tfm`, Bootstrap ERB | [Legacy JS / ERB → React PF5](#path-b-legacy-js--erb--react-pf5) |
| Angular controllers/templates (some plugins / Satellite — **not** covered by Foreman `legacy-js.asciidoc`) | Same Path B goals; inventory Angular surface, replace with React PF5 |
# Using this skill to Angular update is experimental

Copy and track:

```
Upgrade progress:
- [ ] Run UI and locate the screen (baseline behaviors)
- [ ] Scope & baseline inventory
- [ ] Reuse scan (Foreman core)
- [ ] PF5 mapping + v5-archive docs check
- [ ] Implement (function components)
- [ ] Rails routes → `react#index` + client `react-router` paths (redirects for old URLs)
- [ ] CSS audit / delete dead PF3 styles (carefully)
- [ ] RTL tests (no Enzyme/snapshots); Capybara if needed
- [ ] Lint + tests from Foreman root
- [ ] Visual parity in running app
- [ ] PR notes (structural diffs, screenshots)
```

---

## Path A: PF3 React → PF5

Follow `pf3-to-pf5-migration-guide.asciidoc` (summarized below).

### 1. Run the UI and locate the component

- Start Foreman and open the screen that uses the file.
- Find mounts: React callers **and** ERB `react_component('Name', …)`; check `componentRegistry.js` if renaming or changing server props.
- Confirm user-visible behavior: layout, labels, keyboard flow, loading/error states, edge cases. Goal: *no workflow or behavior change*.

### 2. Reuse before rewrite

Search core for the same UX (see [reference.md](reference.md)). Prefer existing Foreman PF5 patterns over new one-off markup.

### 3. Map and read PF5 docs

For each PF3 building block, pick the closest PF5 API ([reference.md](reference.md)). Open the v5-archive page for props, variants, a11y. If no 1:1 match, compose PF5 primitives for the **same user outcome** and document the structural change in the PR.

### 4. Implement

- Replace `patternfly-react` (and related PF3 packages) with `@patternfly/*`:
  - UI: `@patternfly/react-core`
  - Icons: `@patternfly/react-icons`
  - Tables: `@patternfly/react-table`
  - Charts: `@patternfly/react-charts`
  - Simple dropdowns: `@patternfly/react-templates`
- Prefer **direct imports** from the concrete module over folder `index.js` when that avoids ambiguous re-exports.
- Convert **class → function** components (`useState`, `useEffect`, `useRef`, and related hooks) in files you touch; keep behavior identical.
- Prefer **local React state** (`useState`, `useReducer`, `useContext` for subtree-only sharing) for UI-only concerns; keep Redux when distant screens/middleware need the data or ripping it out is an unrelated refactor. Do not reshape Redux as a drive-by.
- i18n: user-visible strings in `__()` / `_()`; variables via `sprintf`, never inside the gettext string.
- Stable `ouiaId`s (never UUIDs) for QE — pattern `description-${name}`.

### 5. CSS

Per migration guide:

- Audit styles tied to the old component; confirm each rule still affects PF5 markup in the running app.
- Delete rules that no longer match any DOM, duplicate PF5, or only patched PF3 quirks.
- Update remaining rules; use **specific selectors** so they do not bleed into core/plugins.
- Only delete CSS from a **global** file when it is clearly component-specific.

### 6. Obsolete code

- Grep callers (core + sibling plugins) before delete.
- Still used → keep surface and `import { deprecate } from '…/common/DeprecationService'` (path relative under `react_app/`), or the re-export from `foreman_tools.js` / `tfm.tools` for legacy globals.
- Shared/common → deprecate first; do not hard-delete until call sites migrate.
- Plugin-private unused helpers → safe to delete after search.

### 7. Tests and verify

Per `ui-testing-guidelines.asciidoc` and the migration guide:

- If the code is still PF3, migrate the component **before** investing in new tests.
- Replace Enzyme/snapshots with **RTL**; prefer `getByRole` → `getByLabelText` → `getByText`; prefer `userEvent` over `fireEvent`.
- Import helpers as `rtlHelpers` from `common/rtlTestHelpers.js` (core) or `foremanReact/common/rtlTestHelpers` (plugins); use `renderWithStore` / `renderWithStoreAndI18n`.
- Mock **API/network only**; avoid mocking `foremanReact`, child components, Redux selectors, or utilities unless there is no other way. Delete leftover `__snapshots__` / unused `__mocks__`.
- Use **Capybara** for full pages, API-heavy flows, multi-step wizards, or deep component interaction that RTL cannot realistically cover.
- Always run from **Foreman root** (never from a plugin dir): `npm run lint` / `npm run lint:plugins <plugin>`; `npm run test` / `npm run test:plugins <plugin>` (or `npm run test -- <path>`).
- Re-check visual and interaction parity in the running app.

---

## Path B: Legacy JS / ERB → React PF5

Foreman `legacy-js.asciidoc` covers the **Rails assets pipeline** (`app/assets/javascripts`), the global `tfm` object, and Redux observe/dispatch — not Angular. Some plugins may still have Angular; treat that as the same replacement goal below.

Goal: replace the legacy surface with React PF5 registered for ERB — not a partial restyle of Bootstrap/jQuery/Angular.

### 1. Inventory the legacy surface

- Templates/partials, assets JS, services, and data the view needs (API endpoints, props, permissions).
- Do **not** update React content with jQuery/DOM manipulation; do **not** alter Redux state except via actions (`legacy-js.asciidoc`).

### 2. Design the React replacement

- Mount with `react_component` from ERB; register in `componentRegistry.js` (`adding-new-components.asciidoc`, `plugins.asciidoc`).
- Pass server props via the helper; React renders as a `foreman-react-component` web component.
- Follow folder layout from `adding-new-components.asciidoc`; keep component files ~**100 lines** (split / index for logic and API calls when larger).
- Reuse Foreman core + PF5; same standing rules as Path A.
- Prefer local state / ForemanContext over new Redux (`state-management.asciidoc`).

### 3. Rails routing → React routing

Legacy pages use **Rails controller routes** that render ERB templates and assets-pipeline JS — every link causes a full page reload. React pages use a **two-layer model**: Rails hands off to Foreman's React shell (`react#index`), then **`react-router`** handles in-app navigation without reloads. See `client-routing.asciidoc`.

**Rails layer — point URLs at the React shell**

Replace controller/action routes with `react#index` (Foreman's `ReactController`):

```ruby
# Before (legacy): controller renders ERB + assets JS
get '/subscriptions' => 'katello/subscriptions#index'

# After: Rails serves the React app shell; client router picks the page
match '/subscriptions' => 'react#index', :via => [:get]
match '/subscriptions/*page' => 'react#index', :via => [:get]  # nested client paths
```

- Add **`redirect` routes** for old URLs users, docs, or QE may still hit — Katello does this extensively (e.g. `/content_hosts/:id` → host details, `/katello/sync_management` → `/sync_management`).
- Remove or gate the old controller action and ERB template once the React route is live.
- Plugin routes go in the plugin's `config/routes.rb`; core routes in `foreman/config/routes.rb`.

**Client layer — register `react-router` paths**

Pick the pattern your repo already uses:

| Pattern | Where | When to use |
|---------|-------|-------------|
| **Core routes** | `react_app/routes/<Page>/` → import in `routes.js` | New pages in Foreman core |
| **Plugin `registerRoutes`** | `registerRoutes('plugin', routes)` via `foremanReact/routes/RoutingService`; load from `global_index.js` or `register_global_js_file 'routes'` | Standalone plugin pages (e.g. `foreman_rh_cloud`) |
| **Plugin monolith app** | Root component in `componentRegistry` + internal `Routes.js` / `config.js` | Katello-style multi-scene apps mounted once from `react.html.erb` |

**Core example** (`client-routing.asciidoc`):

```js
// react_app/routes/MyPage/index.js
export default {
  path: MY_PAGE_PATH,   // from ./constants.js
  exact: true,
  render: props => <MyPage {...props} />,
};
```

**Plugin `registerRoutes` example** (`foreman_rh_cloud`):

```js
import { registerRoutes } from 'foremanReact/routes/RoutingService';

export const routes = [
  { path: '/foreman_rh_cloud/inventory_upload', exact: true, render: props => <InventoryUpload {...props} /> },
];

registerRoutes('foreman_rh_cloud', routes);
```

**Katello monolith example** — add the scene to `webpack/containers/Application/config.js` and `Routes.js`; Rails already maps `react#index` for the base path.

**Navigation and parity checks**

- Replace `<a href="…">`, `window.location`, and hardcoded paths with `Link` / `useHistory` from `react-router-dom`.
- Define path strings in a `constants.js` file (e.g. `MODELS_PATH` in core) — do not scatter URL strings across components.
- For widgets embedded in ERB outside a full react route page, wrap with `withReactRoutes` only when they need router context (`foremanReact/common/withReactRoutes`).
- Verify **browser back/forward**, deep links, and query strings still work after migration.
- Update menu entries, breadcrumbs, and any hardcoded links in Ruby helpers or other plugins.
- Routing-heavy flows: prefer **Capybara** tests in addition to RTL (`ui-testing-guidelines.asciidoc`).

### 4. Wire and remove legacy UI

- Mount React from the ERB that previously hosted the legacy UI; remove or gate old JS/templates after parity.
- Limited legacy→React bridge (only when needed): `tfm.store.dispatch` / `observeStore`, or root prop updates via `reactProps` / `data-props` on the web component (`adding-new-components.asciidoc`). Prefer moving logic into React/API.
- Deprecate shared legacy surfaces via DeprecationService / `foreman_tools.js` as in Path A.

### 5. Tests

- RTL for self-contained React; Capybara for full-page / multi-step / API flows (`ui-testing-guidelines.asciidoc`).

---

## Validation (AI-generated code)

Before calling the work done (aligned with migration guide checklist):

| Check | Pass criteria |
|-------|----------------|
| Imports | No new `patternfly-react` / PF3; `@patternfly/*` + Foreman reuse |
| Markup | PF5 components for structure; no ad-hoc page layout in raw HTML/CSS |
| Parity | Behaviors from baseline inventory still present (verified in running app) |
| Routing | Old URLs redirect; client routes registered; in-app nav without full reload |
| i18n | All new user-visible strings extracted |
| Tests | RTL (and Capybara if appropriate); no new Enzyme/snapshots |
| Lint/tests | Clean from Foreman root on touched package |
| CSS | Scoped; PF3 dead rules removed only when justified |
| Docs | PR lists structural PF differences + screenshots of affected views |

For PR review readiness, also apply the `foreman-ui-reviewer` skill.

---

## Selecting a good pilot page (scale-up research)

Pick a section that mixes several of: form controls, modal, table/list + pagination, tabs, empty/loading states, ERB mount, and (optionally) a permission-gated action. Avoid a pure icon-only tweak and avoid a multi-plugin epic for the first AI-driven pass.

**Suggested metrics** (capture in the PR or a short migration note):

- Wall time: inventory → green lint/tests → visual sign-off
- Human intervention count (blocked decisions, reverted AI edits)
- Files / LOC touched vs reused from core
- Defects found in review or QE (parity, a11y, i18n, CSS bleed)
- Test debt closed (Enzyme→RTL) vs left behind

---

## Additional resources

- Component mapping, pitfalls, checklists: [reference.md](reference.md)
- Foreman core: https://github.com/theforeman/foreman
- PF5 docs: https://v5-archive.patternfly.org/
- Handbook JS: https://theforeman.org/handbook.html
