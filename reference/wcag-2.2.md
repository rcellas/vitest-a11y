# WCAG 2.2 Success Criteria Reference

Complete reference for testing WCAG 2.2 success criteria with Vitest Browser Mode.

## New in WCAG 2.2

WCAG 2.2 adds 9 new success criteria. All examples require **Browser Mode** for accurate testing.

---

## 2.4.11 Focus Not Obscured (Minimum) - Level AA

**Requirement**: When a user interface component receives keyboard focus, the component is not entirely hidden due to author-created content.

**Why it matters**: Users navigating with keyboard need to see which element has focus, especially with sticky headers/footers.

```ts
test('focused element is not completely obscured by other content', async () => {
  // WCAG 2.4.11: When a component receives focus, it is not entirely hidden
  render(<StickyHeaderApp />);
  const button = page.getByRole('button', { name: /submit/i });
  await button.focus();
  
  // Check that focused element is visible (not obscured by sticky header)
  const isVisible = await button.element().then(el => {
    const rect = el.getBoundingClientRect();
    return rect.top >= 0 && rect.bottom <= window.innerHeight;
  });
  expect(isVisible).toBe(true);
});
```

---

## 2.4.12 Focus Not Obscured (Enhanced) - Level AAA

**Requirement**: When a user interface component receives keyboard focus, no part of the component is hidden by author-created content.

**Why it matters**: Enhanced version ensures the entire focused element is visible, not just part of it.

```ts
test('focused element is fully visible without scrolling', async () => {
  // WCAG 2.4.12: No part of the focus indicator is hidden
  render(<App />);
  const input = page.getByRole('textbox', { name: /email/i });
  await input.focus();
  
  const fullyVisible = await input.element().then(el => {
    const rect = el.getBoundingClientRect();
    const viewportHeight = window.innerHeight;
    return rect.top >= 0 && rect.bottom <= viewportHeight;
  });
  expect(fullyVisible).toBe(true);
});
```

---

## 2.4.13 Focus Appearance - Level AAA

**Requirement**: When the keyboard focus indicator is visible, an area of the focus indicator meets all of the following:
- Is at least as large as the area of a 2 CSS pixel thick perimeter
- Has a contrast ratio of at least 3:1 between the same pixels in the focused and unfocused states

**Why it matters**: Ensures focus indicators are visible enough for users with low vision.

```ts
test('focus indicator meets minimum size and contrast requirements', async () => {
  // WCAG 2.4.13: Focus indicator has sufficient size and contrast
  // Browser Mode required for computed styles
  render(<Button>Click me</Button>);
  const button = page.getByRole('button', { name: /click me/i });
  await button.focus();
  
  // Verify focus indicator is visible (implementation-specific)
  const outlineWidth = await button.element().then(el => 
    window.getComputedStyle(el).outlineWidth
  );
  expect(outlineWidth).not.toBe('0px');
  
  // For full compliance, you'd also test contrast ratio
  // This requires color extraction and calculation
});
```

---

## 2.5.7 Dragging Movements - Level AA

**Requirement**: All functionality that uses a dragging movement for operation can be achieved by a single pointer without dragging, unless dragging is essential.

**Why it matters**: Users with motor disabilities may not be able to perform dragging movements.

```ts
test('draggable component has single-pointer alternative', async () => {
  // WCAG 2.5.7: Functionality that uses dragging has single pointer alternative
  render(<DraggableList />);
  
  // Must have buttons or other controls for reordering
  const moveUpBtn = page.getByRole('button', { name: /move up/i });
  const moveDownBtn = page.getByRole('button', { name: /move down/i });
  
  await expect.element(moveUpBtn).toBeVisible();
  await expect.element(moveDownBtn).toBeVisible();
});

test('drag and drop has keyboard alternative', async () => {
  render(<DragDropInterface />);
  
  // Test keyboard-only reordering
  const firstItem = page.getByRole('listitem').first();
  await firstItem.focus();
  
  // Should have keyboard shortcut or button to move
  await userEvent.keyboard('{Control>}{ArrowDown}{/Control}');
  
  // Verify item moved
  const items = page.getByRole('listitem');
  const firstItemText = await items.first().element().then(el => el.textContent);
  expect(firstItemText).not.toBe(await firstItem.element().then(el => el.textContent));
});
```

---

## 2.5.8 Target Size (Minimum) - Level AA

**Requirement**: The size of the target for pointer inputs is at least 24 by 24 CSS pixels, except when:
- The target is available through an equivalent link or control that is at least 24x24 pixels
- The target is inline (in a sentence or block of text)
- The size is determined by the user agent
- Essential for the information being conveyed

**Why it matters**: Small touch targets are difficult for users with motor disabilities or on touch devices.

```ts
test('interactive elements meet minimum target size', async () => {
  // WCAG 2.5.8: Target size is at least 24x24 CSS pixels
  // Browser Mode required for computed dimensions
  render(<ButtonGroup />);
  const buttons = page.getByRole('button');
  const count = await buttons.count();
  
  for (let i = 0; i < count; i++) {
    const button = buttons.nth(i);
    const { width, height } = await button.element().then(el => el.getBoundingClientRect());
    expect(width).toBeGreaterThanOrEqual(24);
    expect(height).toBeGreaterThanOrEqual(24);
  }
});

test('icon buttons have sufficient size or spacing', async () => {
  // WCAG 2.5.8: Small targets need spacing
  render(<IconButtonToolbar />);
  const iconButtons = page.getByRole('button');
  
  const firstButton = iconButtons.first();
  const { width, height } = await firstButton.element().then(el => el.getBoundingClientRect());
  
  if (width < 24 || height < 24) {
    // If button is smaller than 24x24, check spacing to next target
    const secondButton = iconButtons.nth(1);
    const rect1 = await firstButton.element().then(el => el.getBoundingClientRect());
    const rect2 = await secondButton.element().then(el => el.getBoundingClientRect());
    
    const spacing = rect2.left - rect1.right;
    expect(spacing).toBeGreaterThanOrEqual(24 - width);
  }
});
```

---

## 3.2.6 Consistent Help - Level A

**Requirement**: If a Web page contains any of the following help mechanisms, and those mechanisms are repeated on multiple Web pages within a set of Web pages, they occur in the same relative order to other page content:
- Human contact details
- Human contact mechanism
- Self-help option
- A fully automated contact mechanism

**Why it matters**: Consistent help location helps users find assistance quickly.

```ts
test('help mechanism appears in consistent location across pages', async () => {
  // WCAG 3.2.6: Help mechanism is in same relative order on each page
  render(<Page1 />);
  const help1 = page.getByRole('link', { name: /help/i });
  const help1Index = await help1.element().then(el => 
    Array.from(el.parentElement?.children || []).indexOf(el)
  );
  
  render(<Page2 />);
  const help2 = page.getByRole('link', { name: /help/i });
  const help2Index = await help2.element().then(el => 
    Array.from(el.parentElement?.children || []).indexOf(el)
  );
  
  expect(help1Index).toBe(help2Index);
});

test('contact information in consistent location', async () => {
  const pages = [<HomePage />, <AboutPage />, <ContactPage />];
  let previousPosition: number | null = null;
  
  for (const PageComponent of pages) {
    render(PageComponent);
    const contactLink = page.getByRole('link', { name: /contact/i });
    
    const position = await contactLink.element().then(el => {
      const nav = el.closest('nav');
      return Array.from(nav?.children || []).indexOf(el);
    });
    
    if (previousPosition !== null) {
      expect(position).toBe(previousPosition);
    }
    previousPosition = position;
  }
});
```

---

## 3.3.7 Redundant Entry - Level A

**Requirement**: Information previously entered by or provided to the user that is required to be entered again in the same process is either:
- Auto-populated, or
- Available for the user to select

**Why it matters**: Reduces cognitive load and errors, especially for users with cognitive disabilities.

```ts
test('previously entered information is auto-populated or selectable', async () => {
  // WCAG 3.3.7: Information previously entered is auto-populated or available to select
  render(<MultiStepForm />);
  
  // Step 1: Enter email
  await page.getByRole('textbox', { name: /email/i }).fill('user@example.com');
  await page.getByRole('button', { name: /next/i }).click();
  
  // Step 2: Go back
  await page.getByRole('button', { name: /back/i }).click();
  
  // Email should be preserved
  const emailInput = page.getByRole('textbox', { name: /email/i });
  await expect.element(emailInput).toHaveValue('user@example.com');
});

test('shipping address auto-fills from billing address', async () => {
  render(<CheckoutForm />);
  
  // Fill billing address
  await page.getByRole('textbox', { name: /billing street/i }).fill('123 Main St');
  await page.getByRole('textbox', { name: /billing city/i }).fill('Springfield');
  
  // Check "Same as billing"
  await page.getByRole('checkbox', { name: /same as billing/i }).click();
  
  // Shipping should auto-populate
  await expect.element(page.getByRole('textbox', { name: /shipping street/i }))
    .toHaveValue('123 Main St');
  await expect.element(page.getByRole('textbox', { name: /shipping city/i }))
    .toHaveValue('Springfield');
});
```

---

## 3.3.8 Accessible Authentication (Minimum) - Level AA

**Requirement**: A cognitive function test (such as remembering a password or solving a puzzle) is not required for any step in an authentication process unless that step provides at least one of:
- Alternative: Another authentication method that does not rely on a cognitive function test
- Mechanism: A mechanism is available to assist the user in completing the cognitive function test
- Object Recognition: The cognitive function test is to recognize objects
- Personal Content: The cognitive function test is to identify non-text content the user provided to the Web site

**Why it matters**: Complex authentication can exclude users with cognitive disabilities.

```ts
test('authentication does not rely on cognitive function test', async () => {
  // WCAG 3.3.8: No cognitive function test required (unless alternative provided)
  render(<LoginForm />);
  
  // Should have password manager support or alternative authentication
  const passwordInput = page.getByLabelText(/password/i);
  await expect.element(passwordInput).toHaveAttribute('autocomplete', 'current-password');
  
  // Or: Social login alternative
  const socialLogin = page.getByRole('button', { name: /sign in with/i });
  await expect.element(socialLogin).toBeVisible();
});

test('password field supports paste', async () => {
  // WCAG 3.3.8: Don't block password managers
  render(<LoginForm />);
  const passwordInput = page.getByLabelText(/password/i);
  
  // Paste should work (not blocked by JavaScript)
  await passwordInput.focus();
  await userEvent.paste('MySecurePassword123!');
  
  await expect.element(passwordInput).toHaveValue('MySecurePassword123!');
});

test('biometric authentication available as alternative', async () => {
  // WCAG 3.3.8: Alternative that doesn't require memory
  render(<LoginForm />);
  
  const biometricBtn = page.getByRole('button', { name: /use fingerprint/i });
  await expect.element(biometricBtn).toBeVisible();
});
```

---

## 3.3.9 Accessible Authentication (Enhanced) - Level AAA

**Requirement**: A cognitive function test is not required for any step in an authentication process unless that step provides at least one of:
- Alternative: Another authentication method that does not rely on a cognitive function test
- Mechanism: A mechanism is available to assist the user in completing the cognitive function test

**Why it matters**: Enhanced version removes object recognition exception.

```ts
test('authentication provides mechanism to assist users', async () => {
  // WCAG 3.3.9: Enhanced - stricter than 3.3.8
  render(<LoginForm />);
  
  // Must have one of:
  // 1. Password manager support
  const passwordInput = page.getByLabelText(/password/i);
  await expect.element(passwordInput).toHaveAttribute('autocomplete', 'current-password');
  
  // 2. Or biometric/SSO alternative
  const ssoButtons = page.getByRole('button', { name: /sign in with/i });
  expect(await ssoButtons.count()).toBeGreaterThan(0);
  
  // 3. Or email magic link
  const magicLinkBtn = page.getByRole('button', { name: /email me a link/i });
  await expect.element(magicLinkBtn).toBeVisible();
});
```

---

## Additional WCAG 2.2 Testing Patterns

### Testing autocomplete attributes (WCAG 1.3.5)

```ts
test('form fields have appropriate autocomplete attributes', async () => {
  // WCAG 1.3.5: Input Purpose
  render(<PersonalInfoForm />);
  
  await expect.element(page.getByLabelText(/email/i))
    .toHaveAttribute('autocomplete', 'email');
  await expect.element(page.getByLabelText(/first name/i))
    .toHaveAttribute('autocomplete', 'given-name');
  await expect.element(page.getByLabelText(/last name/i))
    .toHaveAttribute('autocomplete', 'family-name');
  await expect.element(page.getByLabelText(/street address/i))
    .toHaveAttribute('autocomplete', 'street-address');
  await expect.element(page.getByLabelText(/postal code/i))
    .toHaveAttribute('autocomplete', 'postal-code');
});
```

### Testing text spacing (WCAG 1.4.12)

```ts
test('content is readable with increased text spacing', async () => {
  // WCAG 1.4.12: Text Spacing
  render(<Article />);
  
  // Apply WCAG 1.4.12 spacing requirements
  await page.evaluate(() => {
    document.body.style.lineHeight = '1.5';
    document.body.style.letterSpacing = '0.12em';
    document.body.style.wordSpacing = '0.16em';
    const paragraphs = document.querySelectorAll('p');
    paragraphs.forEach(p => {
      (p as HTMLElement).style.marginBottom = '2em';
    });
  });
  
  // Content should still be readable (no clipping, overlapping)
  const article = page.getByRole('article');
  await expect.element(article).toBeVisible();
  
  // Check for overflow
  const hasOverflow = await article.element().then(el => {
    return el.scrollHeight > el.clientHeight || el.scrollWidth > el.clientWidth;
  });
  expect(hasOverflow).toBe(false);
});
```

### Testing orientation (WCAG 1.3.4)

```ts
test('content works in both portrait and landscape', async () => {
  // WCAG 1.3.4: Orientation
  render(<ResponsiveApp />);
  
  // Test portrait
  await page.setViewportSize({ width: 375, height: 667 });
  await expect.element(page.getByRole('main')).toBeVisible();
  
  // Test landscape
  await page.setViewportSize({ width: 667, height: 375 });
  await expect.element(page.getByRole('main')).toBeVisible();
  
  // Content should be accessible in both orientations
});
```
