# Foreman UI Reviewer — Full Reference

Detailed checklist for reviewing or preparing UI/frontend PRs in theforeman/foreman.

---

## 1. Commit Conventions

- Format: `Fixes #XXXXX - imperative mood description` (Redmine issue required)
- First line ≤50 characters; blank line; then optional body wrapped at ~72 characters
- Use `Fixes` for the main PR, `Refs` only for follow-ups on open issues (never on closed issues)
- Multiple issues: `Fixes #9998,#9999 - description`
- Describe the change outcome, not the bug title (e.g., "X now accepts Y when doing Z")
- Squash to a single commit before merge for **small PRs**; **large PRs** may keep multiple logical commits (each commit one coherent step) — reviewers may still ask to squash before merge, but do not require a single commit when the diff is hard to review as one lump
- Create a Redmine issue at projects.theforeman.org if one does not exist
- The Redmine issue must be in the **Foreman** project, not Katello or another project — PRProcessor rejects cross-project references
- Never force push; use `git revert` for reversals
- Maintain linear history via `git pull --rebase upstream develop` (no merge commits — enables clean bisect)
- Reference: https://cbea.ms/git-commit/

**Cherry-pick commits:** Use `git cherry-pick -x` to maintain attribution. Use the correct merge commit SHA from the target branch, not the original commit SHA — GitHub merge commits have different SHAs.

**Common mistakes caught by PRProcessor:**
- Missing `Fixes #` or `Refs #` prefix
- Issue number from wrong Redmine project
- Non-imperative mood ("Fixed" instead of "Fix", "Adding" instead of "Add")

---

## 2. PR Review Process (from Handbook & dev docs)

### Label Workflow

- `Not reviewed` and `Needs testing` applied automatically on submission
- Mark as `Waiting on contributor` when changes are needed
- `Needs re-review` and `Needs testing` set automatically when branch updates
- Use `Reached an impasse` for serious disagreements or blocking dependencies

### Official Review Checklist (from `developer_docs/pr_review.asciidoc`)

- [ ] Ticket number, title, and description are correct
- [ ] Fixes the problem described in the issue — no unrelated changes
- [ ] Redmine opened for the correct project
- [ ] No use of `.to_sym`, `.send`, or other reflection on untrusted inputs
- [ ] All strings extracted (use `:mark_translated: true` in `config/settings.yaml`)
- [ ] All string extractions follow i18n rules
- [ ] Appropriate permissions for non-admin users added
- [ ] No copy-pasted code
- [ ] All new AR fields have appropriate validators
- [ ] No exceptions swallowed/squashed
- [ ] New exceptions are `Foreman::Exception` or `WrappedException`
- [ ] Covered with meaningful unit/functional/integration tests
- [ ] View templates use `.html.erb` extension (not just `.erb`)
- [ ] Mixins use `ActiveSupport::Concern` (not `class_eval`)
- [ ] Concerns and new classes in `app/`, not `lib/`
- [ ] Apipie documentation is correct: HTTP methods, parameter names, required true/false
- [ ] Scoped_search definitions added for new model attributes where appropriate
- [ ] Virtual fields use `:only_explicit` in scoped_search (prevents free text search on non-existent fields)
- [ ] Appropriate Rails logging with `logger.debug` blocks (lazy evaluation)
- [ ] Deprecations use `Foreman::Deprecation` with deadline = latest stable + 3 versions
- [ ] New controllers/actions have API counterparts
- [ ] New controllers/actions have Hammer CLI counterparts (or a Redmine ticket filed)
- [ ] Public HTTP API not changed in incompatible way
- [ ] No memory or performance concerns
- [ ] Necessary packaging done
- [ ] Agreed on who provides user documentation and community demo

### Contributor Guidelines (from Handbook)

- Write clear descriptions explaining problem, solution, and reproduction steps
- Assume reviewers lack background context
- Discuss significant changes beforehand with maintainers on Matrix or forums
- Submit changes incrementally; maintain API compatibility
- Use feature flags for incomplete features

---

## 3. Testing

### React Testing Library (RTL)

- **All React components must have RTL tests** — snapshot tests and Enzyme tests are not acceptable
- Use `renderWithStore` from `react_app/common/rtlTestHelpers.js` for components with Redux
- Test user-visible behavior: text content, roles, ARIA attributes, interactions (click, type)
- **Do not test implementation details:** class names, internal state, component hierarchy
- **Do not mock Redux selectors** — use a real store with initial state fixtures
- `redux-mock-store` is deprecated — never use it in new tests
- Wrap `jest.advanceTimersByTime` in `act()` to avoid React act warnings
- `*.test.js` files are excluded from `.eslintignore` — linting won't catch formatting issues in tests
- Use `foremanReact/common/testHelpers` import alias (not relative paths) for test helpers in plugins

**Test assertions — what reviewers look for:**

```javascript
// GOOD: tests user-visible behavior
expect(screen.getByText('Delete')).toBeInTheDocument();
expect(screen.getByRole('button', { name: 'Submit' })).toBeEnabled();
await userEvent.click(screen.getByRole('button', { name: 'Cancel' }));
expect(screen.queryByText('Modal Title')).not.toBeInTheDocument();

// BAD: tests implementation details
expect(wrapper.find('.pf-v5-c-button')).toHaveLength(1);
expect(component.state().isOpen).toBe(true);
```

**Mocking strategies (from `ui-testing-guidelines.asciidoc`):**
1. Add mock data to the store via `renderWithStore` with `extraReduxStoreItems`
2. Mock API calls directly: `jest.mock('../../redux/API/API'); API.get.mockImplementation(serverMock);`
3. Mock selector return values — last resort only

**Plugin test setup:** Plugins can define `webpack/test_setup.js` (loaded with Foreman's global setup) and `jest.config.js` (merged with Foreman's config). To reuse core setup: `import 'foremanJSTestSetup';`

### System / Integration Tests (Capybara)

- Use Capybara best practices — reference https://devhints.io/capybara
- **Do not copy patterns from existing Foreman integration tests** — many are outdated
- Use `wait_for_ajax` for timing issues in Capybara tests
- Randomly failing tests: check for timing issues, cached assets, or headless-mode-specific behavior (`DEBUG_JS_TEST=1` may not reproduce headless failures)
- Note unrelated test failures explicitly: "Test failures not related to this PR" (with specifics)

### Ruby Testing Standards (from Handbook)

- Use `test 'description' do` syntax (not `def test_`)
- Use custom assertions (`assert_empty` vs `assert variable.empty?`)
- Use symbolic status codes (`:success`, `:not_authorized`) over numeric HTTP codes
- Prefer `FactoryBot.build` over `.create` to avoid database writes
- Use `setup_user` for users with special permissions
- Test valid cases before invalid ones in API tests
- Extract shared test behavior into `context` blocks or shared concerns
- Use stubs/mocks for external and slow calls

### Plugin Test Infrastructure

- Plugins use `npm run test:plugins <plugin-name>`, `npm run lint:plugins <plugin-name>` and `npm run stylelint:plugins <plugin-name>` from Foreman root
- If plugin name is omitted, all plugins are tested; failed plugins are listed at the end
- If a plugin has no `lint` script in `package.json`, Foreman's default lint config runs
- Plugin tests that need more memory: set `NODE_OPTIONS='--max-old-space-size=8096'`

---

## 4. PatternFly 5 Migration

Reference: `developer_docs/pf3-to-pf5-migration-guide.asciidoc`

### Migration Process

1. **Run the UI and locate the component** — some components are embedded from ERB via `react_component` helper, not mounted from React code. Check `.html.erb` files and the component registry
2. **Map PF3 to PF5** — names and APIs may differ; PF5 may split one component into a composition
3. **Read PF5 docs** — use https://v5-archive.patternfly.org/ (not `patternfly.org` which documents PF6)
4. **Update imports** — replace `patternfly-react` (PF3) with `@patternfly/react-core`, `@patternfly/react-icons`, `@patternfly/react-table`, `@patternfly/react-templates`
5. **Prefer direct imports** from the concrete module over importing through `index.js`

### Component Selection

- Use PF5 components from `@patternfly/react-core` — never import from `patternfly-react` (PF3) or deprecated `@theforeman/*` packages
- Use `Tabs` component instead of `Nav` for tab-like UI — avoids excessive CSS overrides
- Use `ModalVariant.small` or `ModalVariant.medium` instead of hardcoded pixel widths
- Use `<Truncate content={text} />` for text truncation instead of CSS-only approaches
- **Do not keep or add thin wrappers** that only re-export a PF widget with no meaningful logic — use the PF5 import directly at the call site
- Convert class components to function components (hooks) during migration

### Feature Parity

- **Every PF5 migration must maintain feature parity with the legacy UI:**
  - Tooltips must still appear and be correctly positioned
  - Popovers, keyboard navigation, loading/error states must work
  - Sort behavior must be preserved (check `isSorted` — set to `false` if sorting doesn't actually work)
  - Pagination type must match (PF5 component from ForemanCore used in TableIndexPage)
- Document in the PR what changed structurally and why, so reviewers can verify parity
- Include screenshots of all affected views in the PR description

### State Simplification During Migration

- **Use component/hook state** (`useState`, `useReducer`) for UI-only concerns: open/close panels, wizard steps, form drafts, transient loading flags
- **Keep Redux** when multiple distant screens or middleware need the data
- Don't change Redux shape as a drive-by unless PF5 work truly requires it
- Delete unused actions, reducers, and selectors after confirming no other subscribers

### Removing Obsolete Code

- Search the codebase AND sibling plugins for imports, re-exports, and string references before deleting
- **If still in use:** deprecate using `import { deprecate } from '…/common/DeprecationService'` — do not hard-delete
- **Shared/common components:** treat as public API. Deprecate first; avoid hard deletes until all known call sites are migrated
- **Plugin-only components:** safe to delete if not used in the plugin

### ouiaId Requirements

- `ouiaId` must be stable and descriptive — **never use UUIDs**
- Pattern: `description-${name}` (e.g., `host-details-${hostname}`)
- When a PF5 component lacks `ouiaId` support (e.g., `TimePicker`), use `id` as fallback
- Used by QE automation (robottelo/airgun) — unstable IDs break downstream tests
- See https://www.patternfly.org/developer-resources/open-ui-automation/

### Deprecation Lifecycle

- JS: `deprecate('OldComponent', 'NewComponent from @patternfly/react-core', '3.XX')`
- Ruby: `Foreman::Deprecation.deprecation_warning(deadline_version, message)` — deadline = latest stable + 3
- API: `Foreman::Deprecation.api_deprecation_warning(message)`
- Legacy global JS: `tfm.tools.deprecate` before removing global functions
- Check if deprecated component is used in plugins before removing
- Deprecate first, remove in a follow-up release
- Open related Redmine issues for plugin cleanup

### Data Sources

- **Use `reported_data`, not raw facts** for hardware info in UI components
- Frontend code should never query the fact store directly — if reported_data doesn't have what you need, enhance the reported_data model first
- This is a core design principle: "reported data should be used and facts should be avoided"

---

## 5. CSS and Styling

### Scoping Rules

- **Scope all CSS to a component-specific class or ID** — global rules affect all plugins
- Never write bare element selectors (`label { }`, `input { }`) — these break PF5 alignment across the app
- When overriding PF3 global rules (e.g., `label` font-weight from bootstrap-sass), scope to PF5 form containers
- Group icon-related classes under a host classname to prevent cross-page side effects
- Use specific CSS selectors to prevent conflicts with plugins or core (Handbook rule)

**Example — scoping CSS:**
```scss
// BAD: global override affects all plugins
.icon-size { font-size: 24px; }

// GOOD: scoped to component
.my-component-name .icon-size { font-size: var(--pf-v5-global--icon--FontSize--lg); }
```

### PF5 Variables and Utilities

- **Use PF5 CSS variables for all values** — never hardcode colors, font sizes, or spacing:
  - Colors: `--pf-v5-global--palette--white`, `--pf-v5-global--Color--100`
  - Font sizes: `--pf-v5-global--icon--FontSize--lg`
  - Spacing: use PF5 utility classes
- Use `overflow-wrap: anywhere` instead of hardcoded widths for text overflow
- Move inline styles to `.scss` files
- Prefer PF5 utility classes over custom CSS where possible

### CSS During PF5 Migration

- PF5 brings its own layout and tokens — old PF3 CSS often becomes redundant
- Audit styles tied to the old component: open the screen and confirm each rule still affects the PF5 markup
- Delete rules that no longer match any DOM, duplicate PF5, or only patched PF3 quirks
- Only delete global CSS if it is component-specific and declared in a global file

### Common CSS Issues

| Issue | Fix |
|-------|-----|
| `!important` in CSS | Remove it — fix specificity instead |
| Hardcoded pixel widths | Use PF5 variables or responsive units |
| Inline styles | Move to `.scss` file |
| Background image breaks at extreme resolutions | Use media queries for edge cases |
| PF3 class name in PF5 component (`pf-c-` vs `pf-v5-c-`) | Double-check automated find-replace didn't catch non-component classes like `pf-color-white` |
| Checkbox/icon misalignment after migration | PF3 global rules break PF5 alignment — scope overrides |

---

## 6. Internationalization (i18n)

### Ruby Translations (from Handbook)

- Basic: `_('My string')`
- Single parameter: `_("String with param: %s") % param`
- Multiple named parameters: `_("Params: %{a} and %{b}") % {:a => foo, :b => bar}`
- Mark for translation without translating: `N_("String")` — translate before display with `_(variable)`
- Plurals: `n_("One", "Two", number)` — with params: `n_("%s minute", "%s minutes", @param) % @param`
- ERB escaping: `_('Test <%%= %{var1} %%>') % { :var1 => "xxx" }`
- **Critical:** Gettext does not extract from interpolation — `"blah #{_('key')} blah"` will NOT work

### JavaScript Translations

- Wrap all user-visible strings in `__()` for translation
- **Variables cannot be inside translated strings** — use `sprintf`:
  ```javascript
  // BAD: variable not translatable
  __(`Template for ${name}`)

  // GOOD: use sprintf
  sprintf(__('Template for %s'), name)

  // GOOD: named parameters
  sprintf(__('Example: %(foo) %(bar)'), {foo: "a", bar: "b"})
  ```
- JS template strings (`` `${var}` ``) cannot be used with `__()` — common mistake reviewers catch
- Strings marked for translation in Ruby with `N_()` or `n_()` can be translated in JS with `__()`
- Use `documentLocale()` for locale-dependent formatting — never hardcode `en-US`
- Set `:mark_translated: true` in `config/settings.yaml` to make untranslated strings visible

### Casing and Terminology

- **UI labels: sentence case** — not Title Case (except page titles)
- Use "hosts" not "systems" — Foreman/Katello standardized on "hosts"
- Avoid branded/product-specific terms in upstream — use generic equivalents

### Translation of dynamic content

- Taxonomy names (`organization`, `location`) in composed strings must themselves be translated — otherwise an English word appears in non-English UI
- Error messages shown to users must be translatable — use `render_error` with translated messages
- API descriptions must be wrapped with `N_(...)` — used for building Hammer CLI help

---

## 7. Cross-Plugin Impact

### Before Modifying Shared Code

1. **Search within the foreman repo** for usage of any component, export, or API being modified — check `all_react_app_exports.js` to determine if it is exposed to plugins
2. **Do NOT scan local plugin directories** — plugin repos may not be checked out or up to date. Instead, flag the change as affecting a public API and note which plugins may need coordinated PRs
3. Check `foreman_register.rb` pagelets and `ForemanContext` metadata for shared state
4. Check if Slot & Fill extensions exist for modified components (`<Slot>` in core, `<Fill>` in plugins)

### Plugin Architecture (from dev docs)

- Plugins extend core via **Slot & Fill** (weight-based rendering), **`foremanReact` imports**, and **`react_component` ERB helper**
- Plugin webpack requires: `./webpack/` folder, `package.json`, and entry point (`./webpack/index.js` or defined in `package.json`)
- Plugins use `addGlobalFill` and `register_global_file 'fills'` for cross-page extensions
- When modifying a Slot in core, check all plugins for Fills targeting that slot ID
- Verify plugin integration with `./script/plugin_webpack_directories.rb`

### Coordinated Changes

- **Open coordinated PRs in affected plugins before merging core changes** — don't break plugins
- If CSS selectors change, ask QE (robottelo/airgun) if it will break downstream automation
- When removing or renaming an export, check `webpack/index.js` and the vendor entry files
- When adding to webpack vendor config, also check if `config/webpack.vendor.js` needs updates

### Packaging Coordination

- New npm dependencies require a foreman-packaging PR (https://github.com/theforeman/foreman-packaging)
- Dependencies vendored inside `foreman-js` do not need separate packaging
- Version-locked dependencies (`"1.13.5"` vs `"^1.13.5"`) — only lock if you've verified breakage with newer versions; prefer ranges when upstream follows SemVer
- When adding packages to the vendor list, check if they need to be in the packaging exclude list too
- **NPM dependencies** are managed through `@theforeman/vendor` in the `foreman-js` monorepo

### API Backward Compatibility

- When changing API parameters (e.g., replacing a boolean with an enum), add a transitional parameter for the old name
- Consider older Smart Proxies and Foreman Ansible Modules — users update ansible collections more often than Foreman servers
- Return `html_url` from API responses so frontends don't construct URLs
- Link to resources by ID rather than name for persistence (hosts can be renamed)

---

## 8. JavaScript and React Patterns

### General (from Handbook + PR comments)

- Place new JS files in `webpack/assets/javascripts/` — use ES2015 syntax (Babel transpiles to ES5)
- Export/import for dependency management — no global namespace pollution
- If you must expose a function globally, use `window.tfm` object
- Never remove global functions without deprecating via `tfm.tools.deprecate`
- Remove unused imports and variables in the same PR — reviewers flag dead code consistently
- Remove `console.log` and debugging artifacts before submitting
- Revert accidental `package.json` / `package-lock.json` changes unrelated to the PR
- Add missing newline at end of file
- **React component files are limited to 100 lines** — split into subcomponents if larger

### Component Structure (from `adding-new-components.asciidoc`)

```
components/<COMPONENT_NAME>/
  <COMPONENT_NAME>.js            — react component
  <COMPONENT_NAME>.scss          — styles if needed
  <COMPONENT_NAME>.fixtures.js   — constants for testing, initial state
  <COMPONENT_NAME>.test.js       — RTL test file
  components/                    — nested components if needed
```

For Redux-backed components, the folder may also include `<COMPONENT_NAME>Actions.js`, `<COMPONENT_NAME>Reducer.js`, `<COMPONENT_NAME>Selectors.js`, and `<COMPONENT_NAME>Constants.js`. Plugins (e.g. Katello) commonly add `<COMPONENT_NAME>Helpers.js` alongside these files.

**Constants and helpers — do not define inline when a dedicated file exists:**

- Place constants in the component's `<COMPONENT_NAME>Constants.js` when one exists — do not define module-level constants inline in the component file (action types, modal IDs, tab keys, API keys, etc.).
- Place helpers in the component's `<COMPONENT_NAME>Helpers.js` when one exists — do not define pure helper functions inline in the component file (data transforms, formatters, filter/sort logic).
- Small one-off values used only inside the component body (e.g. a local `useState` default) may stay in the component file.
- When introducing a new feature to an existing component folder that already has `*Constants.js` or `*Helpers.js`, extend those files rather than growing the component file.

**Example (Katello):** `ManageManifestModal.js` imports `MANAGE_MANIFEST_MODAL_ID` from `./ManifestConstants.js`; `SubscriptionHelpers.js` holds `filterRHSubscriptions` and related transforms — not the React component file.

### State Management (from `state-management.asciidoc`)

- **Local state** (`useState`, `useReducer`): for data used only by the component and children (input values, form state, UI element states)
- **Redux store**: for data shared across many unrelated components, or API responses via middleware. Don't default to Redux — consider local state or context first
- **ForemanContext**: for rarely-changing global metadata (taxonomy, user, version, settings). Use hooks: `useForemanOrganization()`, `useForemanLocation()`, `useForemanUser()`, `useForemanSettings()`, `useForemanVersion()`
- **`window.tfm`**: only for legacy JS interop. Avoid if possible
- **`sessionStorage`**: only for volatile, temporary state that won't impact users if lost
- Prefer hooks over Redux where possible (Handbook rule)

### Redux / API Selectors

- `selectAPIStatus` accepts a single argument (the state) when used with a pre-bound key
- `selectAPIStatusData` returns the response data — use optional chaining:
  ```javascript
  export const selectApiDataErrorCode = state =>
    selectAPIStatusData(state)?.response?.status;
  ```
- Handle HTTP status codes correctly: 401 = session expired (show login page), 403 = permission denied (show error)

### Permissions in Frontend (from `handling_user_permissions.asciidoc`)

- **Frontend permission checks are NOT a replacement for backend authorization**
- Prefer context-based approach (`<Permitted>` component or `usePermissions` hook) — cached, fast
- API-based approach (`/api/v2/permissions/current_permissions`) adds 200-250ms — use near top of component tree
- Use `authorized` scope in backend models to filter permissions

### Legacy JS Interaction (from `legacy-js.asciidoc`)

- Do not update React content with jQuery or other DOM manipulation
- Do not alter Redux state manually — only via actions
- Avoid changing the actual DOM with React when legacy JS manages it
- Access webpack JS from legacy code via `tfm.store.dispatch('actionName', arg1)` and `tfm.store.observeStore('path.to.store', callback)`

### Type Checking

- `typeof value === 'string'` is sufficient — `instanceof String` is unnecessary
- Prefer simple type checks over complex validation for internal code

---

## 9. Ruby / Backend Patterns (from Handbook + PR comments)

### Do

- Use `blank?` over `empty?` for strings
- Filter models through `authorized` scope for permission handling
- Raise `Foreman::Exception` types exclusively
- Ensure migrations are reversible
- Use Rails validator syntax: `validates :name, :uniqueness => true, :presence => true`
- Use `.html.erb` extension for view templates; use `%>` not `-%>`
- Leverage `ActiveSupport::Concern` in mixins (not `class_eval` or `InstanceMethods`)
- Document new model fields in API docs; verify Apipie accuracy
- Add non-admin permissions and test routes with restricted users
- Place concerns and new classes in `app/`, not `lib/`
- Favor `.present?` over `.nil?`
- Add scoped_search definitions for new model attributes; use `:only_explicit` for virtual fields
- Catch only expected exceptions, wrap in `Foreman::WrappedException`
- Display ERF codes for caught `Foreman::Exception` errors
- Pass blocks to `logger.debug` for lazy evaluation
- Use symbolic status codes (`:success`, `:not_authorized`) over HTTP codes
- Use `Foreman::Deprecation` with deadline = latest stable + 3 versions
- Use `update` not `update_attribute` — `update` runs validations

### Don't

- Use `.to_sym`, `.send`, `eval` on untrusted inputs
- Catch unexpected exceptions or `Foreman::Exception` broadly
- Use single-character variable names or abbreviations
- Squash exceptions without handling them
- Parse error messages for control flow — use HTTP status codes

### API Design (from `api.asciidoc`)

- Use Apipie documentation framework — Hammer CLI relies on it
- All API descriptions must be wrapped with `N_(...)` for translation
- Nest parameters in a root node named by the resource: `{ "architecture": { "name": "i386" } }`
- Use `render_error` for error responses — don't render error JSON directly
- Available error templates: `access_denied`, `not_found`, `param_error`, `standard_error`, `unauthorized`, `unprocessable_entity`, `unsupported_content_type`
- New API endpoints should have corresponding Hammer CLI commands (or a Redmine ticket filed)
- Don't simplify parameter names too much — be explicit (don't use `environment` when resource is `FavoriteEnvironment`)
- Index responses have standard metadata: `@total`, `@per_page`, `@page`, `@subtotal`, `@search`

### Performance

- `update_all` returns number of affected rows, not input count — use for accurate reporting
- Prefer `!collection.any?` over `collection.count == 0` (avoids full count query)
- Error messages shown to users must be actionable — "check server logs" is not valid for regular users

---

## 10. PR Description and Process

### Required Elements

- **Screenshots or screen recordings** showing before and after for all visual changes
- Clear reproduction steps for testing
- For PF5 migrations: screenshots of **all** affected views, including edge cases (empty states, long text, extreme resolutions)
- Note unrelated test failures explicitly: "Test failures not related to this PR" with specifics
- If AI was used to generate code, note it in the PR and **review the output thoroughly**

### Process

- **Small PRs:** squash to a single commit before merge. **Large PRs:** multiple logical commits are acceptable when each commit is a reviewable unit (e.g. routing, then UI, then tests) — note the commit order in the PR description; squash only if the reviewer requests it
- Rebase on latest develop before requesting merge, especially if there are conflicts
- Katello plugin test failures during PF5 migration are often expected — note this
- packit build failures may be expected if packaging PRs haven't merged yet — note this too
- Test the feature in a running application, not just unit tests — UI correctness requires visual verification
- For snapshot test updates: either read the entire updated snapshot to verify correctness, or replace with RTL/Capybara tests

### Branching (from Handbook)

- **develop**: target for new PRs — nightly code, relatively stable
- **release-stable (e.g., 3.14-stable)**: cherry-picked commits for current release cycle
- All cherry-picks use `git cherry-pick -x` to maintain attribution

---

## 11. Form and Input Patterns

- Add `autoComplete` attributes to login/password inputs: `autoComplete="current-password"`, `autoComplete="username"`
- Use PF5 form validation patterns — inline validation on submit is acceptable for now, but file Redmine for as-you-type validation
- When checkboxes override each other (e.g., two boolean options that are mutually exclusive), convert to a dropdown/select
- For readonly values, use `readOnly` prop rather than disabling the input

---

## 12. Timezones and Dates (from Handbook)

- Store all date/time data in **UTC** in the database
- Display dates to users with timezone consideration via `User.current.timezone`
- Controllers call `set_timezone` per request to adjust Ruby Time objects

---

## 13. Common Review Feedback Quick Reference

| Issue | Fix |
|-------|-----|
| Global CSS override affecting plugins | Scope CSS to component ID/class |
| Component ID collision | Use unique, descriptive IDs — check for duplicates |
| Unused imports/variables left in code | Remove dead code in the same PR |
| `package.json` changes unrelated to PR | Revert accidental changes before committing |
| Missing newline at end of file | Add trailing newline |
| AI-generated code not reviewed | Always review and test AI output — note in PR if AI was used |
| Empty `grep` or temp files committed | Check `git status` before committing |
| `update_attribute` used instead of `update` | Use `update` to run validations |
| Hardcoded color/size values in CSS | Use `--pf-v5-global--*` CSS variables |
| `instanceof String` check | Use `typeof value === 'string'` instead |
| Tooltip next to wrong element | Place tooltip next to the element it explains |
| Sort enabled but not functional | Set `isSorted={false}` if sorting doesn't work |
| English word in translated string | Translate taxonomy names and other dynamic words |
| Compose strings with variable concatenation | Use `sprintf(__('message %s'), variable)` |
| Test mocking Redux selectors | Use real store with initial state fixtures |
| Links constructed from names | Link by ID for persistence (resources can be renamed) |
| Swallowed exception | Always handle or re-raise — never squash |
| `.to_sym`/`.send` on user input | Security risk — never use reflection on untrusted input |
| Missing Apipie docs for new API | Document all new APIs with correct HTTP methods, params, required fields |
| Missing Hammer CLI counterpart | Create Hammer command or file Redmine ticket |
| Component >100 lines | Split into subcomponents; use index.js for orchestration |
| Constants/helpers inline in component file | Move to existing `<Name>Constants.js` / `<Name>Helpers.js` in the same folder |
| Thin PF wrapper component | Remove the wrapper — use PF5 import directly |
| Missing `:only_explicit` on virtual scoped_search | Add it to prevent broken free text search |
| Unused `className` on element | Remove it — only add class names referenced in CSS/SCSS, test selectors, or needed by PF/third-party libraries |

---

## 14. ESLint Configuration

- When adding new domain-specific terms (e.g., `rfc4519`, `posix`), add them to `.eslintrc` globals or word lists to prevent lint warnings
- `eslint-disable` comments for `useEffect` dependency warnings are acceptable when intentional — add a brief reason
- Run `npm run lint` before submitting — CI catches issues but reviewing lint output locally saves round-trips
- Disable ESLint rules only with documented, strong justification
- Latest rules: https://github.com/theforeman/foreman/blob/develop/.eslintrc
