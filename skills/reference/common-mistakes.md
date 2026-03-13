# Common Accessibility Testing Mistakes

Learn from these common mistakes when writing accessibility tests with Vitest.

---

## ❌ Mistake 1: Testing CSS class instead of ARIA state

```ts
// ❌ Wrong — tests implementation
expect(dialog.classList.contains('open')).toBe(true);

// ✅ Correct — tests the accessibility contract
await expect.element(page.getByRole('dialog')).toBeVisible();
await expect.element(btn).toHaveAttribute('aria-expanded', 'true');
```

**Why**: Screen readers don't see CSS classes. They rely on ARIA attributes and semantic HTML.

---

## ❌ Mistake 2: Using `@testing-library/user-event` in Browser Mode

```ts
// ❌ Wrong in Browser Mode — simulated events, not real
import userEvent from '@testing-library/user-event';
await userEvent.keyboard('{ArrowDown}');

// ✅ Correct in Browser Mode — real CDP events
import { userEvent } from 'vitest/browser';
await userEvent.keyboard('{ArrowDown}');
```

**Why**: Browser Mode uses Chrome DevTools Protocol for real browser events. `@testing-library/user-event` simulates events in jsdom, which doesn't match real user behavior.

---

## ❌ Mistake 3: axe without AA tags

```ts
// ❌ Wrong — does not enforce WCAG AA
const results = await axe(container);

// ✅ Correct — explicitly enforces WCAG 2.2 AA
const results = await axe(container, {
  runOnly: { 
    type: 'tag', 
    values: ['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa', 'wcag22aa'] 
  },
});
expect(results).toHaveNoViolations();
```

**Why**: axe-core's default configuration doesn't enforce WCAG AA compliance. You must specify tags explicitly.

---

## ❌ Mistake 4: Not citing the APG rule in the test comment

```ts
// ❌ Vague — what contract does this test?
test('arrow key works', async () => { ... });

// ✅ Traceable — APG rule is cited
test('ArrowDown moves focus to next tab', async () => {
  // APG: "Right Arrow — When focus is on a tab element, moves focus to the next tab."
  ...
});
```

**Why**: Tests should document which accessibility requirement they verify. This makes failures easier to understand and fix.

---

## ❌ Mistake 5: `expect()` instead of `expect.element()` in Browser Mode

```ts
// ❌ Wrong in Browser Mode — no retry, flaky on async state
expect(dialog).toHaveAttribute('aria-modal', 'true');

// ✅ Correct — retries until condition is met
await expect.element(dialog).toHaveAttribute('aria-modal', 'true');
```

**Why**: `expect.element()` has built-in retry logic for async state changes. Regular `expect()` doesn't wait and will fail on timing-dependent assertions.

---

## ❌ Mistake 6: Testing focus with jsdom when Browser Mode is available

```ts
// ❌ Unreliable in jsdom — focus is simulated
fireEvent.keyDown(tab, { key: 'ArrowRight' });
expect(nextTab).toHaveFocus();

// ✅ Real in Browser Mode
await userEvent.keyboard('{ArrowRight}');
await expect.element(nextTab).toBeFocused();
```

**Why**: jsdom simulates focus behavior, which doesn't match real browser focus management. Use Browser Mode for focus testing.

---

## ❌ Mistake 7: Not testing focus-visible (WCAG 2.4.7)

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

**Why**: WCAG 2.4.7 requires a visible focus indicator. Testing only that an element can receive focus doesn't verify the indicator is visible.

---

## ❌ Mistake 8: Not using semantic HTML

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

**Why**: Semantic HTML provides built-in accessibility. Divs don't have keyboard support, focus management, or screen reader announcements.

---

## ❌ Mistake 9: Testing with mouse clicks only

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

**Why**: WCAG 2.1.1 requires all functionality to be keyboard accessible. Mouse-only tests miss keyboard accessibility issues.

---

## ❌ Mistake 10: Not testing with actual assistive technology patterns

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

**Why**: Assistive technology uses the computed accessible name, which can come from multiple sources. Testing the specific attribute is too restrictive.

---

## ❌ Mistake 11: Using test IDs instead of semantic queries

```ts
// ❌ Wrong — doesn't test accessibility
const dialog = page.locator('[data-testid="modal"]');

// ✅ Correct — semantic query tests ARIA role
const dialog = page.getByRole('dialog', { name: /settings/i });
```

**Why**: If `getByRole` fails, your component has an accessibility issue. Test IDs bypass accessibility verification.

---

## ❌ Mistake 12: Not testing error announcements

```ts
// ❌ Wrong — only tests visual error
test('shows error message', async () => {
  await page.getByRole('button', { name: /submit/i }).click();
  await expect.element(page.getByText(/required/i)).toBeVisible();
});

// ✅ Correct — tests that error is linked to input
test('error is announced to screen readers', async () => {
  render(<Form />);
  await page.getByRole('button', { name: /submit/i }).click();
  
  const input = page.getByRole('textbox', { name: /email/i });
  
  // WCAG 3.3.1: Error must be programmatically associated
  await expect.element(input).toBeInvalid();
  await expect.element(input).toHaveAccessibleErrorMessage(/required/i);
});
```

**Why**: Visual error messages aren't enough. Screen readers need `aria-invalid` and `aria-describedby` linking to error text.

---

## ❌ Mistake 13: Forgetting to test focus trap

```ts
// ❌ Wrong — doesn't verify focus stays in modal
test('modal opens', async () => {
  await page.getByRole('button', { name: /open/i }).click();
  await expect.element(page.getByRole('dialog')).toBeVisible();
});

// ✅ Correct — tests focus trap (WCAG 2.4.3)
test('Tab cycles focus within modal', async () => {
  const { getByRole } = render(<Modal isOpen={true} />);
  const firstInput = getByRole('textbox').first();
  const lastButton = getByRole('button').last();
  
  await firstInput.click();
  await userEvent.keyboard('{Shift>}{Tab}{/Shift}');
  await expect.element(lastButton).toHaveFocus(); // Should wrap
});
```

**Why**: Modals must trap focus to prevent keyboard users from tabbing to content behind the modal.

---

## ❌ Mistake 14: Not testing with reduced motion

```ts
// ❌ Wrong — doesn't respect prefers-reduced-motion
test('animation plays', async () => {
  render(<AnimatedComponent />);
  // Animation always plays
});

// ✅ Correct — respects user preference (WCAG 2.3.3)
test('respects prefers-reduced-motion', async () => {
  // Set reduced motion preference
  await page.emulateMedia({ reducedMotion: 'reduce' });
  
  render(<AnimatedComponent />);
  
  // Animation should be disabled or simplified
  const element = page.getByRole('region');
  const animationDuration = await element.element().then(el =>
    window.getComputedStyle(el).animationDuration
  );
  expect(animationDuration).toBe('0s');
});
```

**Why**: WCAG 2.3.3 requires respecting `prefers-reduced-motion` for users with vestibular disorders.

---

## ❌ Mistake 15: Testing color only

```ts
// ❌ Wrong — color alone conveys information
<span style={{ color: 'red' }}>Error</span>

// ✅ Correct — uses icon + text + ARIA
<span role="alert">
  <ErrorIcon aria-hidden="true" />
  Error: Email is required
</span>

test('error uses more than color', async () => {
  // WCAG 1.4.1: Use of Color
  render(<Form />);
  await page.getByRole('button', { name: /submit/i }).click();
  
  // Error should be in an alert or have aria-invalid
  const alert = page.getByRole('alert');
  await expect.element(alert).toContainText(/error/i);
});
```

**Why**: WCAG 1.4.1 prohibits using color as the only means of conveying information. Users with color blindness need alternatives.

---

## Best Practices Summary

1. **Always use semantic queries** (`getByRole`, `getByLabelText`)
2. **Test keyboard navigation** for every interactive element
3. **Use Browser Mode** for focus, keyboard, and visual testing
4. **Cite APG/WCAG rules** in test comments
5. **Test with assistive technology patterns** (accessible name, not attributes)
6. **Enforce WCAG AA explicitly** in axe scans
7. **Use `expect.element()`** in Browser Mode for retry logic
8. **Test error announcements**, not just visual errors
9. **Verify focus management** (trap, return, visible indicator)
10. **Respect user preferences** (reduced motion, high contrast)
