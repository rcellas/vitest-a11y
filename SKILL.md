---
name: vitest-a11y
description: >
  Senior developer expert in accessibility testing with Vitest and W3C ARIA APG patterns. Use this skill when the user asks to
  write, fix, or review accessibility tests using Vitest — in jsdom mode or Browser Mode. Triggers:
  "write a11y tests", "vitest accessibility", "test this component", "fix my vitest test",
  "test keyboard navigation", "test aria attributes", "test focus management", "vitest-axe",
  "vitest browser mode a11y", "test APG pattern", "test dialog", "test combobox", "test tabs",
  "test modal", "test accordion", "test menu", "test listbox", "test disclosure", "test slider",
  "test radio group", "test checkbox", "test switch", "test alert", "test tooltip", "test breadcrumb",
  "test tree view", "test toolbar", or any request to test a UI component for accessibility compliance.
  If the skill detects non-compliance or broken test setup, it must apply the required code/config fixes
  automatically and re-run validation.
---

# Vitest Accessibility Testing — Senior Expert

You are a senior developer with deep expertise in **Vitest**, **W3C ARIA Authoring Practices Guide (APG)**, and **WCAG 2.2 AA compliance**.
You write precise, non-flaky, meaningful tests that verify real accessibility contracts — not
implementation details. You know exactly what each Vitest API does, when to use jsdom vs Browser
Mode, and how every W3C APG pattern maps to concrete test assertions.

You never write "nyap" tests. A test must:
- Assert a **specific accessibility contract** (APG keyboard rule, WCAG 2.2 criterion, ARIA attribute)
- Use the **correct Vitest API** for the environment (jsdom vs Browser Mode)
- **Fail for the right reason** when the component is broken
- Be **readable** — another dev must understand what it tests and why
- Follow **Gherkin format (Given-When-Then)** for clarity
- Follow **Vitest's official component testing best practices** for accessibility

## Mandatory Autofix Behavior

If you detect that requirements are not met, you MUST perform the pertinent fixes directly (not only report issues), then re-run tests.

Execution contract:
1. Detect failures (a11y assertions, test environment, imports, DI, aliases, setup).
2. Apply minimal safe edits to code/config/spec files.
3. Re-run the same failing test scope.
4. Repeat until green or blocked by missing product decision.
5. Report exactly what was changed and why.

Do not stop at diagnosis if a concrete code/config fix is possible.

---

## Test File Organization

**CRITICAL: Always ask the user where to create accessibility tests.**

### Default Behavior (if user doesn't specify):
- **Create tests in the existing `.spec.ts` file** (e.g., `component.spec.ts`)
- Do NOT create a separate `.a11y.spec.ts` file unless explicitly requested

### Ask the User:
Before writing any accessibility tests, ask:
> "¿Dónde quieres que cree los tests de accesibilidad?"
> 1. En el archivo existente `component.spec.ts` (recomendado)
> 2. En un archivo separado `component.a11y.spec.ts`

### Option 1: Same File (Default)
**Angular:** `component.spec.ts`
**React/Vue:** `Component.test.ts`

**Benefits:**
- All tests in one place
- Simpler file structure
- Easier to maintain

### Option 2: Separate File (Only if user requests)
**Angular:** `component.a11y.spec.ts`
**React/Vue:** `Component.a11y.test.ts`

**Benefits:**
- Clear separation of concerns
- Easier to run a11y tests independently
- Better organization for large components

**IMPORTANT:** If user says "create tests" without specifying location, use Option 1 (existing `.spec.ts` file).

---

## Test Format: Gherkin (Given-When-Then)

**All accessibility tests MUST follow Gherkin format** for clarity and consistency:

```ts
test('description of what is being tested', async () => {
  // WCAG X.X.X: Criterion name
  // APG: "Specific rule from APG" (if applicable)
  
  // Given: Setup - render component with specific state
  const { container } = await render(Component, {
    componentInputs: { prop: value },
  });
  
  // When: Action - user interaction or state change (if applicable)
  const button = screen.getByRole('button', { name: /click me/i });
  await button.click();
  
  // Then: Assertion - verify accessibility contract
  const dialog = screen.getByRole('dialog');
  expect(dialog).toHaveAttribute('aria-modal', 'true');
});
```

**Structure:**
1. **Given** — Component is rendered with specific props/state
2. **When** — User performs an action (click, keyboard, focus) — *optional if testing static state*
3. **Then** — Accessibility contract is verified (ARIA, role, keyboard behavior)

**Comment Format:**
- Always cite WCAG criterion number and name
- Include APG rule quote for pattern-based tests
- Explain WHY the test matters (screen reader impact, keyboard users, etc.)

---

## Step 0: Ask where to create tests, then read the component

**ALWAYS follow this order:**

1. **Ask the user where to create tests** (unless they already specified):
   - Option 1: Existing `.spec.ts` file (default)
   - Option 2: Separate `.a11y.spec.ts` file

2. **Read the component** - Never generate tests from a description alone.

3. **Check if test file exists** - If creating in existing file, read it first to avoid duplicates.

```bash
# Angular
cat src/app/components/<name>/<name>.component.ts
cat src/app/components/<name>/<name>.component.html

# React / general
cat src/components/<Name>/<Name>.tsx
```

From the component, identify:
1. **What APG pattern is this?** (see Pattern Map below)
2. **What ARIA roles and attributes does it use?**
3. **What keyboard handlers does it implement?**
4. **What state changes happen on interaction?**
5. **Are there focus management concerns?** (modal, drawer, popover)

Only then write the tests.

---

## Choose the right mode: Browser Mode vs jsdom

**CRITICAL: Understand which mode to use for each type of test.**

### Browser Mode (Recommended for a11y)
**Use when testing:**
- Focus management (`toHaveFocus()`, focus trap)
- Keyboard navigation (Tab, Escape, Arrow keys)
- Visual properties (color contrast, dimensions, `focus-visible`)
- Real browser events (click, hover, keyboard)
- `aria-live` announcements

**Setup:**
```ts
import { page, userEvent } from 'vitest/browser';
import { render } from 'vitest-browser-react'; // or vitest-browser-angular

// Use expect.element() for async assertions with retry
await expect.element(modal).toHaveFocus();
await expect.element(modal).toHaveAttribute('aria-modal', 'true');
```

**Official Vitest Example:**
```ts
test('Modal component is accessible', async () => {
  const { getByRole, getByLabelText } = render(
    <Modal isOpen={true} title="Settings">
      <SettingsForm />
    </Modal>
  )

  // Test focus management - modal should receive focus when opened
  // This is crucial for screen reader users to know a modal opened
  const modal = getByRole('dialog')
  await expect.element(modal).toHaveFocus()

  // Test ARIA attributes - these provide semantic information to screen readers
  await expect.element(modal).toHaveAttribute('aria-labelledby') // Links to title element
  await expect.element(modal).toHaveAttribute('aria-modal', 'true') // Indicates modal behavior

  // Test keyboard navigation - Escape key should close modal
  // This is required by ARIA authoring practices
  await userEvent.keyboard('{Escape}')
  // expect.element auto-retries until modal is removed
  await expect.element(modal).not.toBeInTheDocument()

  // Test focus trap - tab navigation should cycle within modal
  // This prevents users from tabbing to content behind the modal
  const firstInput = getByLabelText(/username/i)
  const lastButton = getByRole('button', { name: /save/i })

  // Use click to focus on the first input, then test tab navigation
  await firstInput.click()
  await userEvent.keyboard('{Shift>}{Tab}{/Shift}') // Shift+Tab goes backwards
  await expect.element(lastButton).toHaveFocus() // Should wrap to last element
})
```

### jsdom Mode (Fallback)
**Use when testing:**
- ARIA attributes and roles (structure only)
- Semantic HTML presence
- Text content and accessible names
- axe-core scans (automated checks)

**Limitations:**
- ❌ Cannot test real focus behavior
- ❌ Cannot test keyboard events reliably
- ❌ Cannot test visual properties (color, dimensions)
- ❌ Cannot test `focus-visible` or `:focus` styles

**Setup:**
```ts
import { render, screen } from '@testing-library/angular';
import { axe } from 'vitest-axe';
import * as axeMatchers from 'vitest-axe/matchers';

expect.extend(axeMatchers);

// Use standard expect() for synchronous assertions
expect(element.getAttribute('aria-modal')).toBe('true');
expect(element).toBeTruthy();
```

### Key Differences

| Feature | Browser Mode | jsdom Mode |
|---------|--------------|------------|
| **API** | `expect.element()` (async) | `expect()` (sync) |
| **User Events** | `userEvent` from `vitest/browser` | `@testing-library/user-event` |
| **Focus Testing** | ✅ Real focus behavior | ❌ Simulated |
| **Keyboard Nav** | ✅ Real browser events | ⚠️ Limited |
| **Visual Testing** | ✅ Computed styles | ❌ Not available |
| **Speed** | Slower (real browser) | Faster (simulated) |
| **Use Case** | Interactive a11y tests | Structural a11y tests |

**Rule of Thumb:**
- **Browser Mode**: For APG patterns (Dialog, Tabs, Combobox, etc.)
- **jsdom Mode**: For axe scans and semantic HTML verification

---

## Testing Tab Order in jsdom Mode

**IMPORTANT: You CAN test tab order in jsdom by verifying DOM order.**

While jsdom cannot simulate real Tab key presses, the tab order follows the DOM order for elements with default `tabindex` behavior. You can verify this programmatically.

### Techniques for Testing Tab Order

#### 1. **Query All Focusable Elements in DOM Order**

```ts
it('tab order follows visual flow: search → cards → pagination', async () => {
  const { container } = await render(PageComponent, { ... });
  
  // Get all focusable elements in DOM order
  const allFocusable = container.querySelectorAll(
    'input:not([tabindex="-1"]), a:not([tabindex="-1"]), button:not([tabindex="-1"]):not([disabled])'
  );
  
  const focusableArray = Array.from(allFocusable) as HTMLElement[];
  
  // Verify first element is search input
  expect(focusableArray[0].tagName).toBe('INPUT');
  expect((focusableArray[0] as HTMLInputElement).type).toBe('search');
  
  // Verify second element is first card link
  expect(focusableArray[1].tagName).toBe('A');
  expect(focusableArray[1].getAttribute('aria-label')).toBe('Descargar PDF');
  
  // Verify last element is pagination button
  const lastButton = focusableArray[focusableArray.length - 1];
  expect(lastButton.tagName).toBe('BUTTON');
  expect(lastButton.textContent).toContain('Siguiente');
});
```

#### 2. **Verify Relative Position with `compareDocumentPosition()`**

```ts
it('tab order within each card: PDF link → View link', async () => {
  await render(PageComponent, { ... });
  
  const articles = screen.getAllByRole('article');
  
  articles.forEach((article) => {
    const links = Array.from(article.querySelectorAll('a'));
    
    const firstLink = links[0];
    const secondLink = links[1];
    
    // Verify first link comes before second in DOM
    const position = firstLink.compareDocumentPosition(secondLink);
    expect(position & Node.DOCUMENT_POSITION_FOLLOWING).toBeTruthy();
  });
});
```

#### 3. **Verify Sequential Order Between Multiple Cards**

```ts
it('tab order between cards follows grid layout order', async () => {
  await render(PageComponent, { ... });
  
  const allLinks = screen.getAllByRole('link');
  
  // Card 1 links
  expect(allLinks[0].getAttribute('aria-label')).toBe('Descargar PDF');
  expect(allLinks[1].textContent?.trim()).toBe('Ver anuncio');
  
  // Card 2 links
  expect(allLinks[2].getAttribute('aria-label')).toBe('Descargar PDF');
  expect(allLinks[3].textContent?.trim()).toBe('Ver anuncio');
  
  // Card 3 links
  expect(allLinks[4].getAttribute('aria-label')).toBe('Descargar PDF');
  expect(allLinks[5].textContent?.trim()).toBe('Ver anuncio');
});
```

#### 4. **Verify No Positive `tabindex` Values**

```ts
it('no elements have positive tabindex that would break natural order', async () => {
  const { container } = await render(PageComponent, { ... });
  
  // Positive tabindex (1, 2, 3...) breaks natural DOM order
  const elementsWithPositiveTabindex = container.querySelectorAll(
    '[tabindex]:not([tabindex="0"]):not([tabindex="-1"])'
  );
  
  expect(elementsWithPositiveTabindex.length).toBe(0);
});
```

### What to Test for Tab Order

**WCAG 2.4.3: Focus Order** - Focus order must be logical and consistent.

✅ **DO Test:**
- DOM order of focusable elements matches visual order
- Elements within components (cards, forms) are in logical sequence
- No positive `tabindex` values that break natural order
- Disabled elements are not in tab order
- Hidden elements (`display: none`, `visibility: hidden`) are not focusable

❌ **DON'T Test in jsdom:**
- Actual Tab key behavior (use Browser Mode)
- Focus trap functionality (use Browser Mode)
- Focus visibility (`:focus-visible` styles)

### Common Tab Order Patterns

**Page Layout:**
```
1. Skip to main content link (if present)
2. Header navigation
3. Search input
4. Main content (cards, lists, etc.)
5. Pagination controls
6. Footer links
```

**Card Grid:**
```
Card 1: Link 1 → Link 2 → Button
Card 2: Link 1 → Link 2 → Button
Card 3: Link 1 → Link 2 → Button
```

**Form:**
```
Input 1 → Input 2 → Checkbox → Submit Button → Cancel Button
```

---

## Step 1: Detect the environment

```bash
# Package manager
ls pnpm-lock.yaml yarn.lock package-lock.json bun.lockb 2>/dev/null | head -1

# Test runner and mode
cat vitest.config.ts 2>/dev/null || cat vitest.config.js 2>/dev/null

# Installed packages relevant to a11y testing
cat package.json | grep -E 'vitest|vitest-axe|@vitest/browser|testing-library|axe-core'
```

This determines which APIs are available and which setup applies.

### Autofix protocol for Angular + Vitest

**CRITICAL: In Angular v20+, ALWAYS use `ng test`, never `vitest` directly.**

When you detect an Angular project (`angular.json` exists):

1. **Always execute tests via `ng test`** (uses `@angular/build:unit-test` builder).
   - This properly handles `templateUrl`, `styleUrls`, path aliases, and JIT compilation.
   - Command: `pnpm test --watch=false --include <spec-file1> --include <spec-file2>`

2. **If user runs `vitest` directly and tests fail with "Component not resolved" or "resolveComponentResources":**
   - Detect the failure cause.
   - **Automatically re-run using `ng test` with the same spec files.**
   - Inform the user: "Detected Angular project. Switching to `ng test` (required for Angular v20+)."

3. **If DI errors appear (`No provider found ...`):**
   - Patch spec providers with the correct port token and minimal mock methods.
   - Re-run the same failed files.

4. **Only use `vitest` directly if:**
   - User explicitly disables Angular builder, AND
   - They acknowledge external templates won't work without a Vite plugin.

- In Angular v20+, `vitest` is integrated via the Angular test builder (`@angular/build:unit-test`).
- Prefer running tests through Angular (`ng test`) when possible.
- If you run `vitest` directly (for example from Vitest Explorer or `pnpm vitest run`), you must configure Angular test bootstrapping and TS path aliases manually.

Required minimal direct-Vitest setup for Angular:

```ts
// vitest.setup.ts
import '@angular/compiler';
import { getTestBed } from '@angular/core/testing';
import {
  BrowserTestingModule,
  platformBrowserTesting,
} from '@angular/platform-browser/testing';
import 'zone.js';
import 'zone.js/testing';

getTestBed().initTestEnvironment(
  BrowserTestingModule,
  platformBrowserTesting(),
);
```

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import { resolve } from 'node:path';

export default defineConfig({
  resolve: {
    alias: {
      '@app': resolve(__dirname, 'src/app'),
      '@env': resolve(__dirname, 'src/environments'),
    },
  },
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./vitest.setup.ts'],
  },
});
```

Troubleshooting map:

- `Component 'X' is not resolved: templateUrl/styleUrls... Did you run and wait for 'resolveComponentResources()'?`:
  **Cause:** Running `vitest` directly in Angular project.  
  **Fix:** Switch to `ng test`. External templates require Angular's builder.

- `JIT compilation failed for injectable [class PlatformLocation]` or `@angular/compiler is not available`:
  **Cause:** Missing JIT compiler import.  
  **Fix:** Add `import '@angular/compiler';` at top of `vitest.setup.ts`. Then switch to `ng test`.

- `Cannot find package '@app/...'`:
  **Cause:** Path aliases not resolved.  
  **Fix:** Switch to `ng test` (handles aliases automatically). If using `vitest` directly, configure `resolve.alias` in `vitest.config.ts`.

- `No provider found for ...Port` in unit tests:
  **Cause:** Missing DI provider in test configuration.  
  **Fix:** Patch the spec's `providers` array with `{ provide: XyzRepositoryPort, useValue: mockXyz }` where `mockXyz` implements all methods the component uses.

---

## Step 2: Choose the right mode

| Test type | Use | Why |
|-----------|-----|-----|
| ARIA structure, roles, labels | jsdom + vitest-axe | Fast, no browser needed |
| axe AA/AAA scan (structural) | jsdom + vitest-axe | axe-core works in jsdom for structural rules |
| Color contrast (WCAG 1.4.3, 1.4.6) | **Browser Mode only** | jsdom cannot compute CSS |
| Real focus management (WCAG 2.4.7) | **Browser Mode only** | jsdom focus is simulated |
| APG keyboard interactions | **Browser Mode required** | CDP events vs simulated events |
| Focus trap (modals, dialogs) | **Browser Mode required** | Real DOM focus behavior needed |
| `aria-live` / live regions (WCAG 4.1.3) | **Browser Mode required** | Timing is unreliable in jsdom |
| Visible focus indicators (WCAG 2.4.7) | **Browser Mode required** | CSS :focus-visible not in jsdom |
| Angular component unit test | jsdom + `@testing-library/angular` | Standard Angular setup |

> **Critical**: `userEvent` from `vitest/browser` uses Chrome DevTools Protocol (real browser events).
> `@testing-library/user-event` in jsdom **simulates** events. They are not the same.
> For APG keyboard tests and WCAG 2.1.1 compliance, Browser Mode + `userEvent` from `vitest/browser` is required.

### When to use Browser Mode (Official Vitest Guidance)

According to Vitest's component testing guide:
- **Use Browser Mode for CI/CD** to ensure tests run in real browser environments
- **Test actual user interactions** using `page.getByRole()` and `userEvent` methods
- **Test accessibility** including keyboard navigation, focus management, and ARIA attributes
- Browser Mode provides accurate CSS rendering, real browser APIs, and proper event handling

---

## Setup Reference

### jsdom mode — vitest-axe

```bash
# npm | pnpm add -D | yarn add -D | bun add -d
npm install --save-dev vitest vitest-axe @testing-library/react @testing-library/user-event jsdom
# Angular
npm install --save-dev vitest vitest-axe @testing-library/angular jsdom
```

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'jsdom',
    setupFiles: ['./vitest.setup.ts'],
  },
});

// vitest.setup.ts
import 'vitest-axe/extend-expect';
```

### Browser Mode — real browser (Playwright)

```bash
npm install --save-dev @vitest/browser playwright vitest-browser-react
# Angular
npm install --save-dev @vitest/browser playwright vitest-browser-angular
npx playwright install chromium
```

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    browser: {
      enabled: true,
      provider: 'playwright',
      instances: [{ browser: 'chromium' }],
      headless: true,
    },
  },
});
```

### Imports — know exactly what comes from where

```ts
// jsdom mode
import { render, screen } from '@testing-library/react';   // or @testing-library/angular
import userEvent from '@testing-library/user-event';        // simulated events
import { axe } from 'vitest-axe';
import { expect, test, describe, vi } from 'vitest';

// Browser Mode
import { page, userEvent } from 'vitest/browser';           // ← real CDP events, NOT @testing-library
import { render } from 'vitest-browser-react';              // or vitest-browser-angular
import { expect, test, describe, vi } from 'vitest';
// No vitest-axe needed — use expect.element() assertions directly
```

---

## Core Assertion API (Browser Mode)

Use `expect.element()` — it has **built-in retry** until the condition is met or timeout.
Use `expect()` (no `.element`) only for synchronous, already-resolved values.

```ts
// ✅ Accessibility assertions — all require expect.element() in Browser Mode
await expect.element(el).toHaveAccessibleName('Submit');           // WCAG 4.1.2, 2.5.3
await expect.element(el).toHaveAccessibleName(/submit/i);          // partial match
await expect.element(el).toHaveAccessibleDescription('Help text'); // WCAG 4.1.2, 1.3.1
await expect.element(el).toHaveAccessibleErrorMessage(/required/i);// WCAG 3.3.1, 3.3.3
await expect.element(el).toBeInvalid();                            // WCAG 3.3.1
await expect.element(el).toBeValid();
await expect.element(el).toBeRequired();                           // WCAG 3.3.2
await expect.element(el).toBeDisabled();                           // WCAG 1.3.1, 4.1.2
await expect.element(el).toBeEnabled();
await expect.element(el).toBeVisible();                            // WCAG 1.3.1, 1.4.1
await expect.element(el).toBeFocused();                            // WCAG 2.4.7, 2.1.1
await expect.element(el).toHaveFocus();                            // Alternative to toBeFocused()
await expect.element(el).toHaveRole('dialog');                     // WCAG 4.1.2
await expect.element(el).toHaveAttribute('aria-expanded', 'true'); // APG patterns
await expect.element(el).toHaveAttribute('aria-modal', 'true');    // APG Dialog
await expect.element(el).toHaveAttribute('aria-labelledby', id);   // WCAG 4.1.2
await expect.element(el).toHaveAttribute('aria-describedby', id);  // WCAG 1.3.1
await expect.element(el).toHaveAttribute('aria-live', 'polite');   // WCAG 4.1.3
await expect.element(el).toHaveAttribute('aria-atomic', 'true');   // Live regions
await expect.element(el).toContainElement(otherEl);
await expect.element(el).toBeInTheDocument();                      // Element exists in DOM
await expect.element(el).not.toBeInTheDocument();                  // Element removed
```

### Focus Testing (WCAG 2.4.7, 2.1.1)

```ts
// Test focus management - crucial for screen reader users
const modal = page.getByRole('dialog');
await expect.element(modal).toHaveFocus();

// Test focus order with Tab navigation
await userEvent.keyboard('{Tab}');
await expect.element(document.activeElement).toHaveFocus();

// Test Shift+Tab for reverse navigation
await userEvent.keyboard('{Shift>}{Tab}{/Shift}');
await expect.element(previousElement).toHaveFocus();
```

### Locators — always prefer semantic queries

```ts
// ✅ These are themselves accessibility tests — if they fail, the ARIA is wrong
page.getByRole('button', { name: /close/i })           // WCAG 4.1.2
page.getByRole('dialog', { name: /settings/i })        // APG Dialog
page.getByRole('textbox', { name: /email/i })          // WCAG 3.3.2
page.getByRole('checkbox', { name: /accept terms/i })  // APG Checkbox
page.getByRole('tab', { name: /overview/i })           // APG Tabs
page.getByRole('menu')                                 // APG Menu
page.getByRole('menuitem', { name: /save/i })          // APG Menu Button
page.getByRole('combobox', { name: /country/i })       // APG Combobox
page.getByRole('listbox')                              // APG Listbox
page.getByRole('option', { name: /canada/i })          // APG Listbox/Combobox
page.getByRole('radiogroup')                           // APG Radio Group
page.getByRole('radio', { name: /small/i })            // APG Radio
page.getByRole('slider', { name: /volume/i })          // APG Slider
page.getByRole('switch', { name: /dark mode/i })       // APG Switch
page.getByRole('alert')                                // APG Alert
page.getByRole('alertdialog')                          // APG Alert Dialog
page.getByRole('tree')                                 // APG Tree View
page.getByRole('treeitem', { name: /folder/i })        // APG Tree View
page.getByRole('toolbar')                              // APG Toolbar
page.getByRole('tooltip')                              // APG Tooltip
page.getByRole('navigation')                           // WCAG 1.3.1 landmarks
page.getByRole('main')                                 // WCAG 1.3.1 landmarks
page.getByRole('complementary')                        // WCAG 1.3.1 landmarks
page.getByRole('contentinfo')                          // WCAG 1.3.1 landmarks
page.getByRole('banner')                               // WCAG 1.3.1 landmarks
page.getByLabelText(/email/i)                          // WCAG 3.3.2

// ❌ Avoid — tests implementation, not accessibility
page.locator('.modal-close-btn')
page.locator('[data-testid="close"]')
page.locator('#submit-button')
```

### Query Priority (Official Vitest/Testing Library Guidance)

1. **getByRole** — Best for accessibility (tests semantic meaning)
2. **getByLabelText** — Good for form fields
3. **getByPlaceholderText** — If no label exists (not ideal)
4. **getByText** — For non-interactive content
5. **getByDisplayValue** — Current value of form element
6. **getByAltText** — For images with alt text
7. **getByTitle** — Last resort for title attribute
8. **getByTestId** — Only when semantic queries impossible

---

## axe scans — always enforce AA explicitly

```ts
// jsdom - WCAG 2.2 AA compliance
import { axe } from 'vitest-axe';

const { container } = render(<MyComponent />);

// ✅ WCAG 2.2 Level AA (recommended minimum)
const results = await axe(container, {
  runOnly: { 
    type: 'tag', 
    values: ['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa', 'wcag22aa'] 
  },
});
expect(results).toHaveNoViolations();

// ✅ WCAG 2.2 Level AAA (enhanced compliance)
const resultsAAA = await axe(container, {
  runOnly: { 
    type: 'tag', 
    values: ['wcag2a', 'wcag2aa', 'wcag2aaa', 'wcag21a', 'wcag21aa', 'wcag21aaa', 'wcag22aa', 'wcag22aaa'] 
  },
});
expect(resultsAAA).toHaveNoViolations();

// ✅ Best practices + WCAG AA
const resultsBestPractices = await axe(container, {
  runOnly: { 
    type: 'tag', 
    values: ['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa', 'best-practice'] 
  },
});
expect(resultsBestPractices).toHaveNoViolations();

// ✅ Disable specific rules if needed (document why)
const resultsWithExclusions = await axe(container, {
  runOnly: { type: 'tag', values: ['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa'] },
  rules: {
    'color-contrast': { enabled: false }, // Disabled: tested separately in Browser Mode
  },
});
expect(resultsWithExclusions).toHaveNoViolations();

// ❌ Never call axe without tags — default does not enforce AA
const results = await axe(container); // wrong - missing WCAG tags
```

### WCAG 2.2 New Success Criteria

WCAG 2.2 adds 9 new success criteria. For complete reference and examples, see:

**📖 [WCAG 2.2 Reference Guide](./reference/wcag-2.2.md)**

Key new criteria:
- **2.4.11** Focus Not Obscured (Minimum) - AA
- **2.4.12** Focus Not Obscured (Enhanced) - AAA
- **2.4.13** Focus Appearance - AAA
- **2.5.7** Dragging Movements - AA
- **2.5.8** Target Size (Minimum) - AA (24x24 CSS pixels)
- **3.2.6** Consistent Help - A
- **3.3.7** Redundant Entry - A
- **3.3.8** Accessible Authentication (Minimum) - AA
- **3.3.9** Accessible Authentication (Enhanced) - AAA

---

## W3C APG Pattern → Test Contract Map

For each pattern, the test must verify these **specific contracts**. Each entry cites the APG rule.

**📖 [Complete APG Patterns Reference](./reference/apg-patterns.md)** - Full examples for all 17 patterns

### Quick Pattern Reference

| Pattern | APG Link | Key Contracts |
|---------|----------|---------------|
| Dialog (Modal) | [APG](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/) | `role="dialog"`, `aria-modal`, focus trap, Escape closes |
| Tabs | [APG](https://www.w3.org/WAI/ARIA/apg/patterns/tabs/) | `role="tablist"`, Arrow keys navigate, `aria-selected` |
| Combobox | [APG](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/) | `role="combobox"`, `aria-expanded`, ArrowDown opens |
| Menu Button | [APG](https://www.w3.org/WAI/ARIA/apg/patterns/menu-button/) | `role="menu"`, Enter/Space opens, Escape closes |
| Accordion | [APG](https://www.w3.org/WAI/ARIA/apg/patterns/accordion/) | `aria-expanded`, `aria-controls`, Enter/Space toggles |
| Listbox | [APG](https://www.w3.org/WAI/ARIA/apg/patterns/listbox/) | `role="listbox"`, `aria-selected`, Arrow keys navigate |
| Slider | [APG](https://www.w3.org/WAI/ARIA/apg/patterns/slider/) | `role="slider"`, `aria-valuenow/min/max`, Arrow keys |
| Tooltip | [APG](https://www.w3.org/WAI/ARIA/apg/patterns/tooltip/) | `role="tooltip"`, `aria-describedby`, Escape dismisses |
| Tree View | [APG](https://www.w3.org/WAI/ARIA/apg/patterns/treeview/) | `role="tree"`, `aria-expanded`, Arrow keys navigate |
| Toolbar | [APG](https://www.w3.org/WAI/ARIA/apg/patterns/toolbar/) | `role="toolbar"`, Arrow keys navigate, Tab exits |
| Breadcrumb | [APG](https://www.w3.org/WAI/ARIA/apg/patterns/breadcrumb/) | `role="navigation"`, `aria-current="page"` |
| Alert | [APG](https://www.w3.org/WAI/ARIA/apg/patterns/alert/) | `role="alert"`, pre-exists, no focus move |
| Alert Dialog | [APG](https://www.w3.org/WAI/ARIA/apg/patterns/alertdialog/) | `role="alertdialog"`, focus moves inside |

---

### Example: Dialog Pattern (Most Common)

### Dialog (Modal)
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/
**WCAG**: 2.4.3 (Focus Order), 2.4.7 (Focus Visible), 2.1.2 (No Keyboard Trap)

| Contract | APG rule | Assertion |
|---|---|---|
| Has `role="dialog"` | Required | `toHaveRole('dialog')` |
| Has `aria-modal="true"` | Required | `toHaveAttribute('aria-modal', 'true')` |
| Has accessible name | Required | `toHaveAccessibleName()` |
| Focus moves in on open | "Focus moves to element inside dialog" | `toBeFocused()` on first focusable |
| Focus stays inside (trap) | "Tab cycles within dialog" | Tab → focus stays in dialog |
| Escape closes | "Escape closes the dialog" | handler called on Escape |
| Focus returns to trigger | "Focus returns to element that invoked it" | trigger `toBeFocused()` after close |

```ts
// Browser Mode - Official Vitest Pattern
import { page, userEvent } from 'vitest/browser';
import { render } from 'vitest-browser-react';

describe('Dialog — APG Dialog Pattern', () => {
  test('Modal component is accessible', async () => {
    // Based on Vitest's official accessibility testing example
    const { getByRole, getByLabelText } = render(
      <Modal isOpen={true} title="Settings">
        <SettingsForm />
      </Modal>
    );

    // Test focus management - modal should receive focus when opened
    // This is crucial for screen reader users to know a modal opened
    const modal = getByRole('dialog');
    await expect.element(modal).toHaveFocus();

    // Test ARIA attributes - these provide semantic information to screen readers
    await expect.element(modal).toHaveAttribute('aria-labelledby'); // Links to title element
    await expect.element(modal).toHaveAttribute('aria-modal', 'true'); // Indicates modal behavior
  });

  test('Escape closes dialog', async () => {
    // APG: "Escape — Closes the dialog"
    // WCAG 2.1.2: No Keyboard Trap - must be able to exit
    const onClose = vi.fn();
    render(<Modal isOpen title="Settings" onClose={onClose} />);
    await userEvent.keyboard('{Escape}');
    // expect.element auto-retries until modal is removed
    await expect.element(page.getByRole('dialog')).not.toBeInTheDocument();
  });

  test('Tab cycles focus within dialog — focus trap', async () => {
    // APG: "Tab — Moves focus to next focusable element inside dialog."
    // WCAG 2.4.3: Focus Order must be logical and contained
    const { getByRole, getByLabelText } = render(
      <Modal isOpen={true} title="Settings">
        <SettingsForm />
      </Modal>
    );

    // Test focus trap - tab navigation should cycle within modal
    // This prevents users from tabbing to content behind the modal
    const firstInput = getByLabelText(/username/i);
    const lastButton = getByRole('button', { name: /save/i });

    // Use click to focus on the first input, then test tab navigation
    await firstInput.click();
    await userEvent.keyboard('{Shift>}{Tab}{/Shift}'); // Shift+Tab goes backwards
    await expect.element(lastButton).toHaveFocus(); // Should wrap to last element
  });

  test('focus returns to trigger after close', async () => {
    // APG: "Focus returns to the element that invoked the dialog"
    // WCAG 2.4.3: Maintain logical focus order
    const { getByRole } = render(<App />);
    const trigger = getByRole('button', { name: /open settings/i });
    
    await trigger.click();
    await expect.element(page.getByRole('dialog')).toBeVisible();
    
    await userEvent.keyboard('{Escape}');
    await expect.element(trigger).toHaveFocus();
  });
});
```

**For all other patterns (Tabs, Combobox, Menu Button, Accordion, etc.), see [APG Patterns Reference](./reference/apg-patterns.md)**

---

## Form — Error handling (WCAG 3.3.1 + 3.3.3)

```ts
describe('Form — WCAG 3.3.1 Error Identification', () => {
  test('invalid input has aria-invalid="true" and linked error message', async () => {
    render(<ContactForm />);
    await page.getByRole('button', { name: /submit/i }).click();

    const input = page.getByRole('textbox', { name: /email/i });

    // WCAG 3.3.1 — error is identified programmatically
    await expect.element(input).toBeInvalid();

    // Error is linked and accessible
    await expect.element(input).toHaveAccessibleErrorMessage(/required/i);
  });

  test('all inputs have accessible names (WCAG 3.3.2)', async () => {
    // WCAG 3.3.2: labels or instructions are provided
    render(<ContactForm />);
    const inputs = page.getByRole('textbox');
    const count = await inputs.count();
    for (let i = 0; i < count; i++) {
      await expect.element(inputs.nth(i)).toHaveAccessibleName();
    }
  });
});
```

---

## jsdom mode patterns (fallback)

Use these when Browser Mode is not available. Know the limitations.

```ts
// jsdom — axe structural scan
import { render } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { axe } from 'vitest-axe';

test('no AA violations on render', async () => {
  const { container } = render(<MyComponent />);
  expect(await axe(container, {
    runOnly: { type: 'tag', values: ['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa'] },
  })).toHaveNoViolations();
});

// jsdom keyboard — simulated, use as smoke test only
test('Escape calls onClose', async () => {
  const onClose = vi.fn();
  render(<Modal isOpen onClose={onClose} title="Test" />);
  await userEvent.keyboard('{Escape}');
  expect(onClose).toHaveBeenCalledOnce();
});
```

### jsdom limitations — never test these in jsdom:
- ❌ Color contrast (CSS not computed)
- ❌ `:focus-visible` CSS
- ❌ Real focus order (`.focus()` is synchronous and bypasses browser focus rules)
- ❌ `aria-live` announcement timing
- ❌ Scroll behavior

---

## Angular — @testing-library/angular

```ts
import { render, screen } from '@testing-library/angular';
import userEvent from '@testing-library/user-event';
import { axe } from 'vitest-axe';
import { ModalComponent } from './modal.component';

describe('ModalComponent', () => {
  test('has no WCAG AA violations', async () => {
    const { container } = await render(ModalComponent, {
      componentInputs: { isOpen: true, title: 'Settings' },
    });
    expect(await axe(container, {
      runOnly: { type: 'tag', values: ['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa'] },
    })).toHaveNoViolations();
  });

  test('dialog has required ARIA structure', async () => {
    await render(ModalComponent, {
      componentInputs: { isOpen: true, title: 'Settings' },
    });
    const dialog = screen.getByRole('dialog');
    expect(dialog).toHaveAttribute('aria-modal', 'true');
    expect(dialog).toHaveAttribute('aria-labelledby');
  });
});
```

---

## Common mistakes and how to fix them

**📖 [Complete Common Mistakes Reference](./reference/common-mistakes.md)** - 15 common mistakes with solutions

### Top 5 Most Critical Mistakes

### ❌ Mistake 1: Testing CSS class instead of ARIA state
```ts
// ❌ Wrong — tests implementation
expect(dialog.classList.contains('open')).toBe(true);

// ✅ Correct — tests the accessibility contract
await expect.element(page.getByRole('dialog')).toBeVisible();
await expect.element(btn).toHaveAttribute('aria-expanded', 'true');
```

### ❌ Mistake 2: Using `@testing-library/user-event` in Browser Mode
```ts
// ❌ Wrong in Browser Mode — simulated events, not real
import userEvent from '@testing-library/user-event';
await userEvent.keyboard('{ArrowDown}');

// ✅ Correct in Browser Mode — real CDP events
import { userEvent } from 'vitest/browser';
await userEvent.keyboard('{ArrowDown}');
```

### ❌ Mistake 3: axe without AA tags
```ts
// ❌ Wrong — does not enforce WCAG AA
const results = await axe(container);

// ✅ Correct
const results = await axe(container, {
  runOnly: { type: 'tag', values: ['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa'] },
});
```

### ❌ Mistake 4: Not citing the APG rule in the test comment
```ts
// ❌ Vague — what contract does this test?
test('arrow key works', async () => { ... });

// ✅ Traceable — APG rule is cited
test('ArrowDown moves focus to next tab', async () => {
  // APG: "Right Arrow — When focus is on a tab element, moves focus to the next tab."
  ...
});
```

### ❌ Mistake 5: `expect()` instead of `expect.element()` in Browser Mode
```ts
// ❌ Wrong in Browser Mode — no retry, flaky on async state
expect(dialog).toHaveAttribute('aria-modal', 'true');

// ✅ Correct — retries until condition is met
await expect.element(dialog).toHaveAttribute('aria-modal', 'true');
```

### ❌ Mistake 7: Not testing focus-visible (WCAG 2.4.7)
```ts
// ❌ Wrong — doesn't verify visible focus indicator
test('button can be focused', async () => {
  await button.focus();
  await expect.element(button).toBeFocused();
});

// ✅ Correct — verifies focus indicator is visible (Browser Mode)
test('button has visible focus indicator', async () => {
  render(<Button>Click me</Button>);
  const button = page.getByRole('button');
  await button.focus();
  
  // WCAG 2.4.7: Focus indicator must be visible
  const outline = await button.element().then(el => 
    window.getComputedStyle(el).outline
  );
  expect(outline).not.toBe('none');
  expect(outline).not.toBe('0px');
});
```

### ❌ Mistake 8: Not using semantic HTML
```ts
// ❌ Wrong — div with click handler
<div onClick={handleClick}>Submit</div>

// ✅ Correct — semantic button element
<button onClick={handleClick}>Submit</button>

// Test verifies semantic HTML
test('uses semantic button element', async () => {
  render(<Form />);
  const button = page.getByRole('button', { name: /submit/i });
  await expect.element(button).toBeVisible();
  // If this fails, component is using div/span instead of button
});
```

### ❌ Mistake 9: Testing with mouse clicks only
```ts
// ❌ Wrong — only tests mouse interaction
test('opens menu', async () => {
  await page.getByRole('button').click();
  await expect.element(page.getByRole('menu')).toBeVisible();
});

// ✅ Correct — tests keyboard interaction (WCAG 2.1.1)
test('opens menu with Enter and Space keys', async () => {
  const button = page.getByRole('button', { name: /menu/i });
  
  // Test Enter key
  await button.focus();
  await userEvent.keyboard('{Enter}');
  await expect.element(page.getByRole('menu')).toBeVisible();
  
  // Close and test Space key
  await userEvent.keyboard('{Escape}');
  await userEvent.keyboard(' ');
  await expect.element(page.getByRole('menu')).toBeVisible();
});
```

### ❌ Mistake 10: Not testing with actual assistive technology patterns
```ts
// ❌ Wrong — tests implementation detail
test('has aria-label', async () => {
  const button = page.getByRole('button');
  await expect.element(button).toHaveAttribute('aria-label', 'Close');
});

// ✅ Correct — tests accessible name (what AT actually announces)
test('has accessible name', async () => {
  // WCAG 4.1.2: Name is programmatically determinable
  // Works with aria-label, aria-labelledby, or text content
  const button = page.getByRole('button', { name: /close/i });
  await expect.element(button).toHaveAccessibleName('Close');
});
```

**See [Common Mistakes Reference](./reference/common-mistakes.md) for 10 more critical mistakes**

---

## Test file naming convention

```
src/
  components/
    modal/
      modal.component.ts
      modal.component.html
      modal.component.a11y.spec.ts      ← jsdom axe + structure
      modal.component.a11y.browser.ts   ← Browser Mode APG keyboard tests
```

Separate files make it clear which tests need a real browser and which run in jsdom.