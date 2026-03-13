# Contributing to Vitest A11y Testing Skill

Thank you for your interest in contributing to the Vitest A11y Testing Skill! This document provides guidelines and instructions for contributing.

## 📋 Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Development Setup](#development-setup)
- [Contribution Guidelines](#contribution-guidelines)
- [Documentation Standards](#documentation-standards)
- [Testing Standards](#testing-standards)
- [Pull Request Process](#pull-request-process)

## 🤝 Code of Conduct

This project adheres to a code of conduct that all contributors are expected to follow:

- Be respectful and inclusive
- Welcome newcomers and help them learn
- Focus on what is best for the community
- Show empathy towards other community members

## 🎯 How Can I Contribute?

### Reporting Bugs

Before creating bug reports, please check existing issues to avoid duplicates. When creating a bug report, include:

- **Clear title and description**
- **Steps to reproduce** the issue
- **Expected behavior** vs **actual behavior**
- **Code samples** demonstrating the issue
- **Environment details** (Vitest version, Node version, OS, etc.)
- **WCAG criterion** or APG pattern affected (if applicable)

### Suggesting Enhancements

Enhancement suggestions are tracked as GitHub issues. When creating an enhancement suggestion, include:

- **Clear title and description**
- **Use case** - why is this enhancement needed?
- **Proposed solution** - how should it work?
- **Alternatives considered**
- **WCAG/APG references** (if applicable)

### Adding New APG Patterns

To add a new APG pattern:

1. Read the [W3C APG documentation](https://www.w3.org/WAI/ARIA/apg/) for the pattern
2. Create a new section in `reference/apg-patterns.md`
3. Include:
   - Pattern name and APG link
   - WCAG criteria covered
   - Test contract table (what must be tested)
   - Complete Browser Mode example
   - jsdom fallback example (if applicable)
4. Add pattern to the quick reference table in `SKILL.md`
5. Create example implementation in `examples/`

### Adding WCAG Criteria

To add or update WCAG criteria:

1. Reference the [official WCAG 2.2 documentation](https://www.w3.org/WAI/WCAG22/quickref/)
2. Update `reference/wcag-2.2.md` with:
   - Criterion number and name
   - Level (A, AA, AAA)
   - Clear explanation
   - Code examples (both correct and incorrect)
   - Testing approach
3. Link from relevant APG patterns

### Improving Documentation

Documentation improvements are always welcome:

- Fix typos or grammatical errors
- Clarify confusing sections
- Add more examples
- Improve code samples
- Update outdated information

## 🛠️ Development Setup

### Prerequisites

- Node.js 18+ or 20+
- pnpm, npm, or yarn
- Git

### Local Setup

```bash
# Clone the repository
git clone https://github.com/yourusername/vitest-a11y-skill.git
cd vitest-a11y-skill

# Install dependencies (if you want to test examples)
pnpm install

# Run example tests
pnpm test:angular
pnpm test:react
pnpm test:vue
```

### Project Structure

```
vitest-a11y-skill/
├── SKILL.md                    # Main skill documentation
├── README.md                   # Project overview
├── CONTRIBUTING.md             # This file
├── LICENSE                     # MIT License
├── CHANGELOG.md                # Version history
├── reference/
│   ├── apg-patterns.md        # All 17 APG patterns
│   ├── wcag-2.2.md            # WCAG 2.2 criteria
│   └── common-mistakes.md     # Common mistakes and fixes
└── examples/
    ├── angular/               # Angular examples
    ├── react/                 # React examples
    └── vue/                   # Vue examples
```

## 📝 Contribution Guidelines

### Code Style

- **Follow existing patterns** - Match the style of surrounding code
- **Use TypeScript** - All examples should be in TypeScript
- **Include types** - Don't use `any` unless absolutely necessary
- **Comment WCAG/APG references** - Every test must cite the criterion it verifies

### Example Code Standards

```ts
// ✅ Good - Clear, cited, follows Gherkin format
test('Modal closes on Escape key', async () => {
  // WCAG 2.1.2: No Keyboard Trap
  // APG: "Escape — Closes the dialog"
  
  // Given: Modal is open
  const { getByRole } = render(<Modal isOpen title="Settings" />);
  const modal = getByRole('dialog');
  
  // When: User presses Escape
  await userEvent.keyboard('{Escape}');
  
  // Then: Modal is removed from DOM
  await expect.element(modal).not.toBeInTheDocument();
});

// ❌ Bad - No citation, unclear, no Gherkin format
test('escape works', async () => {
  render(<Modal isOpen />);
  await userEvent.keyboard('{Escape}');
  expect(screen.queryByRole('dialog')).toBeNull();
});
```

### Documentation Standards

#### Markdown Formatting

- Use **ATX-style headers** (`#`, `##`, `###`)
- Include **table of contents** for long documents
- Use **code fences** with language identifiers
- Use **tables** for structured data
- Include **emoji** for visual hierarchy (sparingly)

#### Code Examples

- Always include **complete, runnable examples**
- Show both **correct** (✅) and **incorrect** (❌) approaches
- Include **imports** - never assume context
- Add **comments** explaining why, not what
- Use **realistic component names** and props

#### WCAG/APG Citations

Every test example must include:

```ts
// WCAG X.X.X: Criterion Name
// APG: "Exact quote from APG documentation"
```

Example:
```ts
// WCAG 2.4.7: Focus Visible
// APG: "When the dialog opens, focus moves to an element inside the dialog"
```

### Testing Standards

#### Browser Mode Tests (Preferred)

```ts
import { page, userEvent } from 'vitest/browser';
import { render } from 'vitest-browser-react';
import { test, expect, describe } from 'vitest';

describe('ComponentName — APG Pattern Name', () => {
  test('specific accessibility contract', async () => {
    // WCAG citation
    // APG citation
    
    // Given
    const { getByRole } = render(<Component />);
    
    // When
    await userEvent.keyboard('{Tab}');
    
    // Then
    await expect.element(element).toHaveFocus();
  });
});
```

#### jsdom Tests (Fallback)

```ts
import { render, screen } from '@testing-library/react';
import { axe } from 'vitest-axe';
import { test, expect } from 'vitest';

test('no WCAG AA violations', async () => {
  const { container } = render(<Component />);
  
  const results = await axe(container, {
    runOnly: { 
      type: 'tag', 
      values: ['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa'] 
    },
  });
  
  expect(results).toHaveNoViolations();
});
```

## 🔄 Pull Request Process

### Before Submitting

1. **Read the documentation** - Ensure you understand the skill's approach
2. **Check existing PRs** - Avoid duplicate work
3. **Test your changes** - Run examples to verify correctness
4. **Update documentation** - Keep docs in sync with code changes
5. **Follow the style guide** - Match existing patterns

### PR Checklist

- [ ] Code follows the project's style guidelines
- [ ] All examples are complete and runnable
- [ ] WCAG/APG citations are included and accurate
- [ ] Documentation is updated (if applicable)
- [ ] Examples follow Gherkin format (Given-When-Then)
- [ ] Tests use semantic queries (`getByRole`, not `getByTestId`)
- [ ] Browser Mode is used for interactive tests
- [ ] axe scans include WCAG AA tags
- [ ] Commit messages are clear and descriptive

### PR Title Format

Use conventional commits format:

- `feat: Add Disclosure pattern to APG reference`
- `fix: Correct Focus Trap example in Dialog pattern`
- `docs: Improve WCAG 2.5.8 Target Size examples`
- `refactor: Simplify Tabs pattern keyboard navigation`
- `test: Add React example for Combobox pattern`

### PR Description Template

```markdown
## Description
Brief description of the changes

## Motivation
Why is this change needed? What problem does it solve?

## Changes Made
- List of specific changes
- Include file paths

## WCAG/APG References
- WCAG X.X.X: Criterion Name
- APG Pattern: [Link to APG documentation]

## Testing
How were these changes tested?

## Screenshots (if applicable)
Add screenshots for visual changes

## Checklist
- [ ] Code follows style guidelines
- [ ] Documentation updated
- [ ] Examples are runnable
- [ ] WCAG/APG citations included
```

## 🎓 Learning Resources

### Required Reading

- [Vitest Documentation](https://vitest.dev/)
- [Vitest Browser Mode Guide](https://vitest.dev/guide/browser/)
- [W3C ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)

### Recommended Reading

- [WebAIM Articles](https://webaim.org/articles/)
- [Inclusive Components](https://inclusive-components.design/)
- [A11y Project](https://www.a11yproject.com/)
- [MDN Accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility)

## 🐛 Common Pitfalls

### 1. Using Wrong Import for userEvent

```ts
// ❌ Wrong in Browser Mode
import userEvent from '@testing-library/user-event';

// ✅ Correct in Browser Mode
import { userEvent } from 'vitest/browser';
```

### 2. Missing WCAG Tags in axe Scans

```ts
// ❌ Wrong - doesn't enforce AA
const results = await axe(container);

// ✅ Correct - enforces WCAG 2.2 AA
const results = await axe(container, {
  runOnly: { 
    type: 'tag', 
    values: ['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa'] 
  },
});
```

### 3. Using expect() Instead of expect.element()

```ts
// ❌ Wrong in Browser Mode - no retry
expect(element).toHaveFocus();

// ✅ Correct - retries until condition met
await expect.element(element).toHaveFocus();
```

## 💬 Questions?

- **GitHub Discussions** - Ask questions and share ideas
- **GitHub Issues** - Report bugs or request features
- **Documentation** - Check [SKILL.md](./SKILL.md) for detailed reference

## 🙏 Thank You!

Your contributions help make the web more accessible for everyone. Every improvement, no matter how small, makes a difference.

---

**Happy contributing! 🎉**
