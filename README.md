# Vitest A11y Testing Skill

> Senior developer expert skill for accessibility testing with Vitest and W3C ARIA APG patterns

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Vitest](https://img.shields.io/badge/Vitest-6E9F18?logo=vitest&logoColor=white)](https://vitest.dev/)
[![WCAG 2.2](https://img.shields.io/badge/WCAG-2.2%20AA-blue)](https://www.w3.org/WAI/WCAG22/quickref/)

## 🎯 Overview

This skill provides comprehensive guidance for writing precise, non-flaky accessibility tests using **Vitest**. It covers both **jsdom mode** and **Browser Mode**, with deep expertise in:

- ✅ **W3C ARIA Authoring Practices Guide (APG)** patterns
- ✅ **WCAG 2.2 AA/AAA compliance** testing
- ✅ **Real browser testing** with Vitest Browser Mode
- ✅ **Automated accessibility scans** with vitest-axe
- ✅ **Focus management** and keyboard navigation
- ✅ **Screen reader compatibility** verification

## 🚀 Quick Start

### Installation

```bash
# For jsdom mode (structural tests + axe scans)
npm install --save-dev vitest vitest-axe @testing-library/react jsdom

# For Browser Mode (interactive tests + keyboard navigation)
npm install --save-dev @vitest/browser playwright vitest-browser-react
npx playwright install chromium

# For Angular
npm install --save-dev vitest vitest-axe @testing-library/angular
npm install --save-dev @vitest/browser playwright vitest-browser-angular
```

### Basic Configuration

**vitest.config.ts** (Browser Mode - Recommended for a11y)
```ts
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

### Your First Accessibility Test

```ts
import { page, userEvent } from 'vitest/browser';
import { render } from 'vitest-browser-react';
import { test, expect } from 'vitest';

test('Modal is accessible', async () => {
  // WCAG 4.1.2: Name, Role, Value
  // APG: Dialog Pattern
  
  const { getByRole } = render(<Modal isOpen title="Settings" />);
  
  // Given: Modal is rendered
  const modal = getByRole('dialog');
  
  // Then: Has correct ARIA attributes
  await expect.element(modal).toHaveAttribute('aria-modal', 'true');
  await expect.element(modal).toHaveAccessibleName('Settings');
  
  // When: User presses Escape
  await userEvent.keyboard('{Escape}');
  
  // Then: Modal closes (WCAG 2.1.2: No Keyboard Trap)
  await expect.element(modal).not.toBeInTheDocument();
});
```

## 📚 Documentation

### Core Guides

- **[SKILL.md](./SKILL.md)** - Complete skill documentation with all patterns and best practices
- **[WCAG 2.2 Reference](./reference/wcag-2.2.md)** - WCAG 2.2 success criteria with examples
- **[APG Patterns](./reference/apg-patterns.md)** - All 17 W3C APG patterns with test contracts
- **[Common Mistakes](./reference/common-mistakes.md)** - 15 common mistakes and how to fix them

### Examples

- **[Angular Examples](./examples/angular/)** - Complete Angular component tests
- **[React Examples](./examples/react/)** - React component accessibility tests
- **[Vue Examples](./examples/vue/)** - Vue component accessibility tests

## 🎓 Key Concepts

### Browser Mode vs jsdom

| Feature | Browser Mode | jsdom Mode |
|---------|--------------|------------|
| **Focus Testing** | ✅ Real focus behavior | ❌ Simulated |
| **Keyboard Nav** | ✅ Real browser events | ⚠️ Limited |
| **Visual Testing** | ✅ Computed styles | ❌ Not available |
| **Speed** | Slower (real browser) | Faster (simulated) |
| **Use Case** | Interactive a11y tests | Structural a11y tests |

**Rule of Thumb:**
- **Browser Mode**: For APG patterns (Dialog, Tabs, Combobox, etc.)
- **jsdom Mode**: For axe scans and semantic HTML verification

### Test Format: Gherkin (Given-When-Then)

All tests follow Gherkin format for clarity:

```ts
test('description of what is being tested', async () => {
  // WCAG X.X.X: Criterion name
  // APG: "Specific rule from APG" (if applicable)
  
  // Given: Setup - render component with specific state
  const { container } = await render(Component, { props });
  
  // When: Action - user interaction (if applicable)
  await userEvent.keyboard('{Tab}');
  
  // Then: Assertion - verify accessibility contract
  await expect.element(element).toHaveFocus();
});
```

### Supported APG Patterns

| Pattern | Status | Documentation |
|---------|--------|---------------|
| Dialog (Modal) | ✅ | [APG Patterns](./reference/apg-patterns.md#dialog-modal) |
| Tabs | ✅ | [APG Patterns](./reference/apg-patterns.md#tabs) |
| Combobox | ✅ | [APG Patterns](./reference/apg-patterns.md#combobox) |
| Menu Button | ✅ | [APG Patterns](./reference/apg-patterns.md#menu-button) |
| Accordion | ✅ | [APG Patterns](./reference/apg-patterns.md#accordion) |
| Listbox | ✅ | [APG Patterns](./reference/apg-patterns.md#listbox) |
| Slider | ✅ | [APG Patterns](./reference/apg-patterns.md#slider) |
| Tooltip | ✅ | [APG Patterns](./reference/apg-patterns.md#tooltip) |
| Tree View | ✅ | [APG Patterns](./reference/apg-patterns.md#tree-view) |
| Toolbar | ✅ | [APG Patterns](./reference/apg-patterns.md#toolbar) |
| Breadcrumb | ✅ | [APG Patterns](./reference/apg-patterns.md#breadcrumb) |
| Alert | ✅ | [APG Patterns](./reference/apg-patterns.md#alert) |
| Alert Dialog | ✅ | [APG Patterns](./reference/apg-patterns.md#alert-dialog) |

## 🔍 WCAG 2.2 Coverage

This skill covers all WCAG 2.2 Level AA criteria, including the 9 new success criteria:

- **2.4.11** Focus Not Obscured (Minimum) - AA
- **2.4.12** Focus Not Obscured (Enhanced) - AAA
- **2.5.7** Dragging Movements - AA
- **2.5.8** Target Size (Minimum) - AA (24x24 CSS pixels)
- **3.2.6** Consistent Help - A
- **3.3.7** Redundant Entry - A
- **3.3.8** Accessible Authentication (Minimum) - AA
- **3.3.9** Accessible Authentication (Enhanced) - AAA

See [WCAG 2.2 Reference](./reference/wcag-2.2.md) for complete details.

## 🛠️ Features

### Mandatory Autofix Behavior

The skill automatically detects and fixes common issues:

1. ✅ Detects failures (a11y assertions, test environment, imports, DI, aliases)
2. ✅ Applies minimal safe edits to code/config/spec files
3. ✅ Re-runs the same failing test scope
4. ✅ Repeats until green or blocked by missing product decision
5. ✅ Reports exactly what was changed and why

### Angular v20+ Support

- ✅ Automatic detection of Angular projects
- ✅ Uses `ng test` (required for Angular v20+)
- ✅ Handles `templateUrl`, `styleUrls`, and path aliases
- ✅ Automatic DI provider patching

### Framework Support

- ✅ **React** - Full support with vitest-browser-react
- ✅ **Angular** - Full support with @testing-library/angular
- ✅ **Vue** - Full support with vitest-browser-vue
- ✅ **Svelte** - Full support with vitest-browser-svelte

## 📖 Usage Examples

### Testing Focus Management

```ts
test('Modal receives focus when opened', async () => {
  // WCAG 2.4.3: Focus Order
  // APG: "Focus moves to element inside dialog"
  
  const { getByRole } = render(<App />);
  const trigger = getByRole('button', { name: /open/i });
  
  // When: User opens modal
  await trigger.click();
  
  // Then: Focus moves inside modal
  const modal = getByRole('dialog');
  await expect.element(modal).toHaveFocus();
});
```

### Testing Keyboard Navigation

```ts
test('Tab key navigates through menu items', async () => {
  // APG: "Tab — Moves focus to next focusable element"
  // WCAG 2.1.1: Keyboard accessible
  
  const { getByRole } = render(<Menu />);
  const button = getByRole('button', { name: /menu/i });
  
  // Given: Menu is open
  await button.click();
  
  // When: User presses Tab
  await userEvent.keyboard('{Tab}');
  
  // Then: Focus moves to first menu item
  const firstItem = getByRole('menuitem', { name: /save/i });
  await expect.element(firstItem).toHaveFocus();
});
```

### Testing with axe-core

```ts
import { axe } from 'vitest-axe';

test('Component has no WCAG AA violations', async () => {
  const { container } = render(<MyComponent />);
  
  const results = await axe(container, {
    runOnly: { 
      type: 'tag', 
      values: ['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa'] 
    },
  });
  
  expect(results).toHaveNoViolations();
});
```

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

## 📝 License

MIT License - see [LICENSE](./LICENSE) file for details.

## 🔗 Resources

- [Vitest Documentation](https://vitest.dev/)
- [Vitest Browser Mode](https://vitest.dev/guide/browser/)
- [W3C ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
- [WCAG 2.2 Guidelines](https://www.w3.org/WAI/WCAG22/quickref/)
- [vitest-axe](https://github.com/chaance/vitest-axe)
- [Testing Library](https://testing-library.com/)

## 💡 Tips

1. **Always read the component first** - Never generate tests from a description alone
2. **Use Browser Mode for interactive tests** - Focus, keyboard, and visual testing require a real browser
3. **Cite WCAG and APG rules** - Every test should reference the specific criterion it verifies
4. **Follow Gherkin format** - Given-When-Then makes tests readable and maintainable
5. **Test accessibility contracts, not implementation** - Use semantic queries (`getByRole`) over test IDs

## 🆘 Support

- **Issues**: Report bugs or request features via GitHub Issues
- **Discussions**: Ask questions in GitHub Discussions
- **Documentation**: Check [SKILL.md](./SKILL.md) for complete reference

