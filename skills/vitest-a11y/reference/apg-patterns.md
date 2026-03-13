# W3C ARIA APG Patterns Reference

Complete reference for testing W3C ARIA Authoring Practices Guide (APG) patterns with Vitest.

Each pattern includes:
- APG specification link
- Required ARIA structure
- Keyboard interaction contracts
- WCAG criteria mapping
- Complete test examples

---

## Dialog (Modal)
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
    // WCAG 2.4.3: Focus Order
    // WCAG 4.1.2: Name, Role, Value
    // Based on Vitest's official accessibility testing example
    
    // Given: Modal is rendered in open state
    const { getByRole, getByLabelText } = render(
      <Modal isOpen={true} title="Settings">
        <SettingsForm />
      </Modal>
    );

    // Then: Modal receives focus and has correct ARIA attributes
    // This is crucial for screen reader users to know a modal opened
    const modal = getByRole('dialog');
    await expect.element(modal).toHaveFocus();
    await expect.element(modal).toHaveAttribute('aria-labelledby'); // Links to title element
    await expect.element(modal).toHaveAttribute('aria-modal', 'true'); // Indicates modal behavior
  });

  test('Escape closes dialog', async () => {
    // APG: "Escape — Closes the dialog"
    // WCAG 2.1.2: No Keyboard Trap - must be able to exit
    
    // Given: Modal is open
    const onClose = vi.fn();
    render(<Modal isOpen title="Settings" onClose={onClose} />);
    
    // When: User presses Escape key
    await userEvent.keyboard('{Escape}');
    
    // Then: Modal is removed from DOM
    // expect.element auto-retries until modal is removed
    await expect.element(page.getByRole('dialog')).not.toBeInTheDocument();
  });

  test('Tab cycles focus within dialog — focus trap', async () => {
    // APG: "Tab — Moves focus to next focusable element inside dialog."
    // WCAG 2.4.3: Focus Order must be logical and contained
    
    // Given: Modal is open with multiple focusable elements
    const { getByRole, getByLabelText } = render(
      <Modal isOpen={true} title="Settings">
        <SettingsForm />
      </Modal>
    );
    const firstInput = getByLabelText(/username/i);
    const lastButton = getByRole('button', { name: /save/i });

    // When: User focuses first input and presses Shift+Tab (backwards)
    await firstInput.click();
    await userEvent.keyboard('{Shift>}{Tab}{/Shift}');
    
    // Then: Focus wraps to last element (focus trap active)
    // This prevents users from tabbing to content behind the modal
    await expect.element(lastButton).toHaveFocus();
  });

  test('focus returns to trigger after close', async () => {
    // APG: "Focus returns to the element that invoked the dialog"
    // WCAG 2.4.3: Maintain logical focus order
    
    // Given: App with modal trigger button
    const { getByRole } = render(<App />);
    const trigger = getByRole('button', { name: /open settings/i });
    
    // When: User opens modal then closes it with Escape
    await trigger.click();
    await expect.element(page.getByRole('dialog')).toBeVisible();
    await userEvent.keyboard('{Escape}');
    
    // Then: Focus returns to the trigger button
    await expect.element(trigger).toHaveFocus();
  });
});
```

---

## Tabs
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/tabs/

| Contract | APG rule | Assertion |
|---|---|---|
| `role="tablist"` wraps tabs | Required | `getByRole('tablist')` |
| Each tab has `role="tab"` | Required | `getByRole('tab', { name })` |
| Panel has `role="tabpanel"` | Required | `getByRole('tabpanel')` |
| Active tab has `aria-selected="true"` | Required | `toHaveAttribute('aria-selected', 'true')` |
| ArrowRight moves to next tab | "Right Arrow — moves focus to next tab" | next tab `toBeFocused()` |
| ArrowLeft moves to previous | "Left Arrow — moves focus to previous tab" | prev tab `toBeFocused()` |
| Home → first tab | "Home — moves to first tab" | first tab `toBeFocused()` |
| End → last tab | "End — moves to last tab" | last tab `toBeFocused()` |
| Panel labelled by active tab | WCAG 4.1.2 | panel `toHaveAccessibleName()` |

```ts
describe('Tabs — APG Tabs Pattern', () => {
  test('tabs have correct roles and relationships', async () => {
    render(<TabGroup />);
    await expect.element(page.getByRole('tablist')).toBeVisible();
    const firstTab = page.getByRole('tab').first();
    await expect.element(firstTab).toHaveAttribute('aria-selected', 'true');
    await expect.element(page.getByRole('tabpanel')).toBeVisible();
  });

  test('ArrowRight moves focus to next tab', async () => {
    // APG: "Right Arrow — When focus is on a tab, moves focus to the next tab."
    render(<TabGroup />);
    await page.getByRole('tab').first().focus();
    await userEvent.keyboard('{ArrowRight}');
    await expect.element(page.getByRole('tab').nth(1)).toBeFocused();
  });

  test('ArrowLeft moves focus to previous tab', async () => {
    // APG: "Left Arrow — When focus is on a tab, moves focus to the previous tab."
    render(<TabGroup />);
    await page.getByRole('tab').nth(1).focus();
    await userEvent.keyboard('{ArrowLeft}');
    await expect.element(page.getByRole('tab').first()).toBeFocused();
  });

  test('Home moves to first tab, End to last', async () => {
    // APG: "Home — Moves focus to the first tab."
    render(<TabGroup />);
    await page.getByRole('tab').last().focus();
    await userEvent.keyboard('{Home}');
    await expect.element(page.getByRole('tab').first()).toBeFocused();
    await userEvent.keyboard('{End}');
    await expect.element(page.getByRole('tab').last()).toBeFocused();
  });
});
```

---

## Combobox
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/combobox/

| Contract | APG rule | Assertion |
|---|---|---|
| Input has `role="combobox"` | Required | `getByRole('combobox')` |
| `aria-expanded` reflects state | Required | `toHaveAttribute('aria-expanded', 'true/false')` |
| Popup has `role="listbox"` | Required | `getByRole('listbox')` |
| Options have `role="option"` | Required | `getByRole('option')` |
| ArrowDown opens and moves | "Down Arrow — moves focus into popup" | listbox visible + option focused |
| Enter selects option | "Enter — Accepts focused option" | listbox closed + value updated |
| Escape closes | "Escape — Dismisses the popup" | listbox not visible |

```ts
describe('Combobox — APG Combobox Pattern', () => {
  test('ArrowDown opens listbox and focuses first option', async () => {
    // APG: "Down Arrow — If popup not visible, opens it."
    render(<ComboboxComponent />);
    const input = page.getByRole('combobox');
    await expect.element(input).toHaveAttribute('aria-expanded', 'false');
    await input.focus();
    await userEvent.keyboard('{ArrowDown}');
    await expect.element(input).toHaveAttribute('aria-expanded', 'true');
    await expect.element(page.getByRole('listbox')).toBeVisible();
  });

  test('Escape closes listbox without selecting', async () => {
    // APG: "Escape — Dismisses the popup"
    render(<ComboboxComponent />);
    await page.getByRole('combobox').focus();
    await userEvent.keyboard('{ArrowDown}');
    await userEvent.keyboard('{Escape}');
    await expect.element(page.getByRole('combobox')).toHaveAttribute('aria-expanded', 'false');
  });

  test('Enter selects focused option and closes listbox', async () => {
    // APG: "Enter — Accepts the focused option."
    render(<ComboboxComponent />);
    await page.getByRole('combobox').focus();
    await userEvent.keyboard('{ArrowDown}');
    const firstOption = page.getByRole('option').first();
    const text = await firstOption.element().then(el => el.textContent ?? '');
    await userEvent.keyboard('{Enter}');
    await expect.element(page.getByRole('combobox')).toHaveAttribute('aria-expanded', 'false');
    await expect.element(page.getByRole('combobox')).toHaveValue(text.trim());
  });
});
```

---

## Menu Button
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/menu-button/

| Contract | APG rule | Assertion |
|---|---|---|
| Button has `aria-haspopup="true"` | Required | `toHaveAttribute('aria-haspopup', 'true')` |
| Button has `aria-expanded` | Required | `toHaveAttribute('aria-expanded', ...)` |
| Menu has `role="menu"` | Required | `getByRole('menu')` |
| Items have `role="menuitem"` | Required | `getByRole('menuitem')` |
| Enter/Space opens + focuses first | "Enter/Space — Opens menu, focus on first item" | first item `toBeFocused()` |
| ArrowDown/Up navigates | "Down/Up Arrow — moves focus" | adjacent item `toBeFocused()` |
| Escape closes + returns focus | "Escape — Closes. Returns to button." | button `toBeFocused()` |

```ts
describe('Menu Button — APG Menu Button Pattern', () => {
  test('Enter opens menu and focuses first item', async () => {
    // APG: "Enter — Opens the menu. Focus is set on first item."
    render(<MenuButton label="Actions" />);
    const btn = page.getByRole('button', { name: /actions/i });
    await btn.focus();
    await userEvent.keyboard('{Enter}');
    await expect.element(page.getByRole('menu')).toBeVisible();
    await expect.element(page.getByRole('menuitem').first()).toBeFocused();
  });

  test('ArrowDown moves focus to next item, ArrowUp to previous', async () => {
    // APG: "Down Arrow — Moves focus to next item."
    render(<MenuButton label="Actions" />);
    await page.getByRole('button', { name: /actions/i }).focus();
    await userEvent.keyboard('{Enter}');
    await userEvent.keyboard('{ArrowDown}');
    await expect.element(page.getByRole('menuitem').nth(1)).toBeFocused();
    await userEvent.keyboard('{ArrowUp}');
    await expect.element(page.getByRole('menuitem').first()).toBeFocused();
  });

  test('Escape closes menu and returns focus to trigger', async () => {
    // APG: "Escape — Closes the menu. Sets focus to the button."
    render(<MenuButton label="Actions" />);
    const btn = page.getByRole('button', { name: /actions/i });
    await btn.focus();
    await userEvent.keyboard('{Enter}');
    await userEvent.keyboard('{Escape}');
    await expect.element(page.getByRole('menu')).not.toBeVisible();
    await expect.element(btn).toBeFocused();
  });
});
```

---

## Accordion
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/accordion/

| Contract | APG rule | Assertion |
|---|---|---|
| Header is a `<button>` | Required | `getByRole('button', { name })` |
| Button has `aria-expanded` | Required | `toHaveAttribute('aria-expanded', ...)` |
| Panel linked via `aria-controls` | Required | `toHaveAttribute('aria-controls')` |
| Enter/Space toggles | "Enter or Space — Toggles the accordion" | `aria-expanded` flips |
| Tab moves between headers | "Tab — Focus next focusable element" | next header `toBeFocused()` |

```ts
describe('Accordion — APG Accordion Pattern', () => {
  test('Enter toggles panel open and updates aria-expanded', async () => {
    // APG: "Enter or Space — When focus is on accordion header, toggles the panel."
    render(<Accordion />);
    const header = page.getByRole('button', { name: /section 1/i });
    await expect.element(header).toHaveAttribute('aria-expanded', 'false');
    await header.focus();
    await userEvent.keyboard('{Enter}');
    await expect.element(header).toHaveAttribute('aria-expanded', 'true');
    // Panel must be visible
    const panelId = await header.element().then(el => el.getAttribute('aria-controls'));
    await expect.element(page.locator(`#${panelId}`)).toBeVisible();
  });

  test('Space also toggles panel', async () => {
    render(<Accordion />);
    const header = page.getByRole('button', { name: /section 1/i });
    await header.focus();
    await userEvent.keyboard(' ');
    await expect.element(header).toHaveAttribute('aria-expanded', 'true');
  });
});
```

---

## Checkbox
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/checkbox/

| Contract | APG rule | Assertion |
|---|---|---|
| Space toggles checked state | "Space — Toggles checkbox" | `toBeChecked()` / `not.toBeChecked()` |
| Tri-state uses `aria-checked="mixed"` | Tri-state spec | `toHaveAttribute('aria-checked', 'mixed')` |

```ts
describe('Checkbox — APG Checkbox Pattern', () => {
  test('Space toggles checked state', async () => {
    // APG: "Space — Checks/unchecks the checkbox"
    render(<Checkbox label="Accept terms" />);
    const cb = page.getByRole('checkbox', { name: /accept terms/i });
    await expect.element(cb).not.toBeChecked();
    await cb.focus();
    await userEvent.keyboard(' ');
    await expect.element(cb).toBeChecked();
  });

  test('tri-state parent shows aria-checked="mixed" when partially checked', async () => {
    // APG: tri-state uses aria-checked="mixed"
    render(<CheckboxGroup />);
    // Check one child but not all
    await page.getByRole('checkbox', { name: /option 1/i }).click();
    await expect.element(
      page.getByRole('checkbox', { name: /select all/i })
    ).toHaveAttribute('aria-checked', 'mixed');
  });
});
```

---

## Radio Group
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/radio/

| Contract | APG rule | Assertion |
|---|---|---|
| Group has `role="radiogroup"` | Required | `getByRole('radiogroup')` |
| Group has accessible name | WCAG 4.1.2 | `toHaveAccessibleName()` |
| ArrowDown/Right selects next | "Down/Right — Moves and selects next" | next `toBeChecked()` + `toBeFocused()` |
| Tab exits group | "Tab — Moves to next element in tab sequence" | focus leaves radiogroup |

```ts
describe('Radio Group — APG Radio Pattern', () => {
  test('ArrowDown moves focus and selects next option', async () => {
    // APG: "Down Arrow — Moves focus to and selects the next radio button."
    render(<RadioGroup name="size" />);
    await page.getByRole('radio').first().focus();
    await userEvent.keyboard('{ArrowDown}');
    const second = page.getByRole('radio').nth(1);
    await expect.element(second).toBeFocused();
    await expect.element(second).toBeChecked();
  });

  test('radiogroup has an accessible name', async () => {
    render(<RadioGroup name="size" legend="Shirt size" />);
    await expect.element(page.getByRole('radiogroup')).toHaveAccessibleName('Shirt size');
  });
});
```

---

## Switch
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/switch/

```ts
describe('Switch — APG Switch Pattern', () => {
  test('Space and Enter toggle aria-checked', async () => {
    // APG: "Space or Enter — Toggle the switch"
    render(<Switch label="Dark mode" />);
    const sw = page.getByRole('switch', { name: /dark mode/i });
    await expect.element(sw).toHaveAttribute('aria-checked', 'false');
    await sw.focus();
    await userEvent.keyboard(' ');
    await expect.element(sw).toHaveAttribute('aria-checked', 'true');
    await userEvent.keyboard('{Enter}');
    await expect.element(sw).toHaveAttribute('aria-checked', 'false');
  });
});
```

---

## Disclosure (Show/Hide)
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/

```ts
describe('Disclosure — APG Disclosure Pattern', () => {
  test('toggle button updates aria-expanded and shows panel', async () => {
    render(<Disclosure />);
    const btn = page.getByRole('button', { name: /show details/i });
    await expect.element(btn).toHaveAttribute('aria-expanded', 'false');
    await btn.click();
    await expect.element(btn).toHaveAttribute('aria-expanded', 'true');
    const panelId = await btn.element().then(el => el.getAttribute('aria-controls'));
    await expect.element(page.locator(`#${panelId}`)).toBeVisible();
  });
});
```

---

## Slider
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/slider/
**WCAG**: 4.1.2 (Name, Role, Value)

| Contract | APG rule | Assertion |
|---|---|---|
| Has `role="slider"` | Required | `getByRole('slider')` |
| Has `aria-valuenow` | Required | `toHaveAttribute('aria-valuenow')` |
| Has `aria-valuemin` | Required | `toHaveAttribute('aria-valuemin')` |
| Has `aria-valuemax` | Required | `toHaveAttribute('aria-valuemax')` |
| Has accessible name | WCAG 4.1.2 | `toHaveAccessibleName()` |
| ArrowRight/Up increases | "Right/Up Arrow — Increases value" | `aria-valuenow` increases |
| ArrowLeft/Down decreases | "Left/Down Arrow — Decreases value" | `aria-valuenow` decreases |
| Home → minimum | "Home — Sets to minimum" | `aria-valuenow` = `aria-valuemin` |
| End → maximum | "End — Sets to maximum" | `aria-valuenow` = `aria-valuemax` |
| PageUp/PageDown (optional) | "Page Up/Down — Increases/decreases by larger amount" | Larger step change |

```ts
describe('Slider — APG Slider Pattern', () => {
  test('has required ARIA attributes', async () => {
    // WCAG 4.1.2: Name, Role, Value
    render(<Slider label="Volume" min={0} max={100} defaultValue={50} />);
    const slider = page.getByRole('slider', { name: /volume/i });
    await expect.element(slider).toHaveAccessibleName('Volume');
    await expect.element(slider).toHaveAttribute('aria-valuenow', '50');
    await expect.element(slider).toHaveAttribute('aria-valuemin', '0');
    await expect.element(slider).toHaveAttribute('aria-valuemax', '100');
  });

  test('ArrowRight/Up increases value, ArrowLeft/Down decreases', async () => {
    // APG: "Right/Up Arrow — Increases value. Left/Down Arrow — Decreases."
    render(<Slider label="Volume" min={0} max={100} defaultValue={50} />);
    const slider = page.getByRole('slider', { name: /volume/i });
    await slider.focus();
    
    await userEvent.keyboard('{ArrowRight}');
    const afterRight = await slider.element().then(el => el.getAttribute('aria-valuenow'));
    expect(Number(afterRight)).toBeGreaterThan(50);
    
    await userEvent.keyboard('{ArrowLeft}');
    await userEvent.keyboard('{ArrowLeft}');
    const afterLeft = await slider.element().then(el => el.getAttribute('aria-valuenow'));
    expect(Number(afterLeft)).toBeLessThan(Number(afterRight));
  });

  test('Home sets minimum, End sets maximum', async () => {
    // APG: "Home — Sets to minimum. End — Sets to maximum."
    render(<Slider label="Volume" min={0} max={100} defaultValue={50} />);
    const slider = page.getByRole('slider', { name: /volume/i });
    await slider.focus();
    await userEvent.keyboard('{Home}');
    await expect.element(slider).toHaveAttribute('aria-valuenow', '0');
    await userEvent.keyboard('{End}');
    await expect.element(slider).toHaveAttribute('aria-valuenow', '100');
  });
});
```

---

## Alert
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/alert/
**WCAG**: 4.1.3 (Status Messages)

| Contract | APG rule | Assertion |
|---|---|---|
| Has `role="alert"` | Required | `getByRole('alert')` |
| Container pre-exists | "Container exists before content" | Element exists before message |
| Does not move focus | "Alerts do not receive focus" | Focus stays on trigger |
| Announced by AT | WCAG 4.1.3 | Implicit via `role="alert"` |

```ts
describe('Alert — APG Alert Pattern', () => {
  test('role="alert" container pre-exists before message injection', async () => {
    // APG critical rule: the container must exist in DOM BEFORE message is set
    // If you create and inject simultaneously, AT may miss the announcement
    // WCAG 4.1.3: Status messages can be programmatically determined
    render(<AlertContainer />);
    await expect.element(page.locator('[role="alert"]')).toBeInTheDocument();
  });

  test('alert does not move focus', async () => {
    // APG: alerts are announced without stealing focus
    render(<AlertContainer />);
    const trigger = page.getByRole('button', { name: /trigger alert/i });
    await trigger.focus();
    await trigger.click();
    await expect.element(page.getByRole('alert')).toBeVisible();
    // Focus must still be on the trigger
    await expect.element(trigger).toBeFocused();
  });
});
```

---

## Alert Dialog
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/alertdialog/
**WCAG**: 2.4.3 (Focus Order), 4.1.2 (Name, Role, Value)

| Contract | APG rule | Assertion |
|---|---|---|
| Has `role="alertdialog"` | Required | `getByRole('alertdialog')` |
| Has accessible name | Required | `toHaveAccessibleName()` |
| Has accessible description | Recommended | `toHaveAccessibleDescription()` |
| Focus moves to dialog or element inside | "Focus moves to element inside" | `toBeFocused()` |
| Contains at least one focusable element | Required | Button or link exists |

```ts
describe('Alert Dialog — APG Alert Dialog Pattern', () => {
  test('has required ARIA structure and receives focus', async () => {
    // APG: Alert dialog must have role="alertdialog" and accessible name
    render(<AlertDialog title="Delete Confirmation" message="Are you sure?" />);
    const dialog = page.getByRole('alertdialog');
    await expect.element(dialog).toHaveAccessibleName('Delete Confirmation');
    await expect.element(dialog).toHaveAccessibleDescription(/are you sure/i);
  });

  test('focus moves to element inside alert dialog', async () => {
    // APG: "When alert dialog opens, focus moves to element inside"
    render(<AlertDialog title="Delete Confirmation" />);
    const cancelBtn = page.getByRole('button', { name: /cancel/i });
    await expect.element(cancelBtn).toBeFocused();
  });
});
```

---

## Listbox
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/listbox/
**WCAG**: 4.1.2 (Name, Role, Value), 2.1.1 (Keyboard)

| Contract | APG rule | Assertion |
|---|---|---|
| Has `role="listbox"` | Required | `getByRole('listbox')` |
| Options have `role="option"` | Required | `getByRole('option')` |
| Has accessible name | WCAG 4.1.2 | `toHaveAccessibleName()` |
| Selected option has `aria-selected="true"` | Required | `toHaveAttribute('aria-selected', 'true')` |
| ArrowDown moves to next | "Down Arrow — Moves focus to next option" | Next option `toBeFocused()` |
| ArrowUp moves to previous | "Up Arrow — Moves focus to previous option" | Previous option `toBeFocused()` |
| Home → first option | "Home — Moves focus to first option" | First option `toBeFocused()` |
| End → last option | "End — Moves focus to last option" | Last option `toBeFocused()` |

```ts
describe('Listbox — APG Listbox Pattern', () => {
  test('has required ARIA structure', async () => {
    render(<Listbox label="Choose a color" options={['Red', 'Green', 'Blue']} />);
    const listbox = page.getByRole('listbox', { name: /choose a color/i });
    await expect.element(listbox).toBeVisible();
    const options = page.getByRole('option');
    expect(await options.count()).toBe(3);
  });

  test('ArrowDown moves focus to next option', async () => {
    // APG: "Down Arrow — Moves focus to the next option"
    render(<Listbox label="Colors" options={['Red', 'Green', 'Blue']} />);
    await page.getByRole('option').first().focus();
    await userEvent.keyboard('{ArrowDown}');
    await expect.element(page.getByRole('option').nth(1)).toBeFocused();
  });

  test('Home and End navigate to first and last options', async () => {
    // APG: "Home — Moves focus to first option. End — Moves focus to last option."
    render(<Listbox label="Colors" options={['Red', 'Green', 'Blue']} />);
    await page.getByRole('option').nth(1).focus();
    await userEvent.keyboard('{Home}');
    await expect.element(page.getByRole('option').first()).toBeFocused();
    await userEvent.keyboard('{End}');
    await expect.element(page.getByRole('option').last()).toBeFocused();
  });

  test('selected option has aria-selected="true"', async () => {
    // WCAG 4.1.2: States must be programmatically determinable
    render(<Listbox label="Colors" options={['Red', 'Green', 'Blue']} defaultValue="Green" />);
    const greenOption = page.getByRole('option', { name: /green/i });
    await expect.element(greenOption).toHaveAttribute('aria-selected', 'true');
  });
});
```

---

## Breadcrumb
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/breadcrumb/
**WCAG**: 2.4.8 (Location), 1.3.1 (Info and Relationships)

| Contract | APG rule | Assertion |
|---|---|---|
| Container has `aria-label="Breadcrumb"` | Required | `toHaveAttribute('aria-label', 'Breadcrumb')` |
| Uses `<nav>` or `role="navigation"` | Required | `getByRole('navigation')` |
| Current page has `aria-current="page"` | Required | `toHaveAttribute('aria-current', 'page')` |
| Links are in an ordered list | Recommended | `<ol>` contains `<li>` elements |

```ts
describe('Breadcrumb — APG Breadcrumb Pattern', () => {
  test('has required ARIA structure', async () => {
    // APG: Breadcrumb must be in a navigation landmark with label
    render(<Breadcrumb items={[{label: 'Home', href: '/'}, {label: 'Products', href: '/products'}, {label: 'Laptop'}]} />);
    const nav = page.getByRole('navigation', { name: /breadcrumb/i });
    await expect.element(nav).toBeVisible();
  });

  test('current page has aria-current="page"', async () => {
    // APG: "The last link has aria-current='page'"
    // WCAG 2.4.8: Helps users understand their location
    render(<Breadcrumb items={[{label: 'Home', href: '/'}, {label: 'Products'}]} />);
    const currentPage = page.getByRole('link', { name: /products/i });
    await expect.element(currentPage).toHaveAttribute('aria-current', 'page');
  });
});
```

---

## Tooltip
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/tooltip/
**WCAG**: 1.4.13 (Content on Hover or Focus)

| Contract | APG rule | Assertion |
|---|---|---|
| Tooltip has `role="tooltip"` | Required | `getByRole('tooltip')` |
| Trigger has `aria-describedby` | Required | `toHaveAttribute('aria-describedby')` |
| Appears on focus and hover | WCAG 1.4.13 | Visible on focus/hover |
| Dismissible with Escape | WCAG 1.4.13 | Escape closes tooltip |
| Hoverable (can move pointer to tooltip) | WCAG 1.4.13 | Tooltip stays visible |

```ts
describe('Tooltip — APG Tooltip Pattern', () => {
  test('trigger has aria-describedby linking to tooltip', async () => {
    // APG: "Element with tooltip has aria-describedby referencing tooltip ID"
    render(<TooltipButton label="Save" tooltip="Save your changes" />);
    const button = page.getByRole('button', { name: /save/i });
    const describedBy = await button.element().then(el => el.getAttribute('aria-describedby'));
    expect(describedBy).toBeTruthy();
    
    await button.focus();
    const tooltip = page.getByRole('tooltip');
    await expect.element(tooltip).toBeVisible();
    const tooltipId = await tooltip.element().then(el => el.id);
    expect(describedBy).toBe(tooltipId);
  });

  test('tooltip appears on focus and hover', async () => {
    // WCAG 1.4.13: Content on Hover or Focus
    render(<TooltipButton label="Save" tooltip="Save your changes" />);
    const button = page.getByRole('button', { name: /save/i });
    
    // Test focus
    await button.focus();
    await expect.element(page.getByRole('tooltip')).toBeVisible();
    
    await button.blur();
    await expect.element(page.getByRole('tooltip')).not.toBeVisible();
    
    // Test hover
    await button.hover();
    await expect.element(page.getByRole('tooltip')).toBeVisible();
  });

  test('Escape dismisses tooltip', async () => {
    // WCAG 1.4.13: Content can be dismissed
    render(<TooltipButton label="Save" tooltip="Save your changes" />);
    await page.getByRole('button', { name: /save/i }).focus();
    await expect.element(page.getByRole('tooltip')).toBeVisible();
    await userEvent.keyboard('{Escape}');
    await expect.element(page.getByRole('tooltip')).not.toBeVisible();
  });
});
```

---

## Tree View
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/treeview/
**WCAG**: 4.1.2 (Name, Role, Value), 2.1.1 (Keyboard)

| Contract | APG rule | Assertion |
|---|---|---|
| Container has `role="tree"` | Required | `getByRole('tree')` |
| Items have `role="treeitem"` | Required | `getByRole('treeitem')` |
| Expandable items have `aria-expanded` | Required | `toHaveAttribute('aria-expanded')` |
| ArrowDown moves to next visible | "Down Arrow — Moves focus to next node" | Next item `toBeFocused()` |
| ArrowUp moves to previous visible | "Up Arrow — Moves focus to previous node" | Previous item `toBeFocused()` |
| ArrowRight expands/moves to child | "Right Arrow — Expands node or moves to first child" | Expanded or child focused |
| ArrowLeft collapses/moves to parent | "Left Arrow — Collapses node or moves to parent" | Collapsed or parent focused |

```ts
describe('Tree View — APG Tree View Pattern', () => {
  test('has required ARIA structure', async () => {
    render(<TreeView data={treeData} />);
    await expect.element(page.getByRole('tree')).toBeVisible();
    const items = page.getByRole('treeitem');
    expect(await items.count()).toBeGreaterThan(0);
  });

  test('ArrowRight expands collapsed node', async () => {
    // APG: "Right Arrow — When focus is on a closed node, opens the node"
    render(<TreeView data={treeData} />);
    const parentNode = page.getByRole('treeitem', { name: /folder/i });
    await expect.element(parentNode).toHaveAttribute('aria-expanded', 'false');
    await parentNode.focus();
    await userEvent.keyboard('{ArrowRight}');
    await expect.element(parentNode).toHaveAttribute('aria-expanded', 'true');
  });

  test('ArrowLeft collapses expanded node', async () => {
    // APG: "Left Arrow — When focus is on an open node, closes the node"
    render(<TreeView data={treeData} defaultExpanded />);
    const parentNode = page.getByRole('treeitem', { name: /folder/i });
    await parentNode.focus();
    await userEvent.keyboard('{ArrowLeft}');
    await expect.element(parentNode).toHaveAttribute('aria-expanded', 'false');
  });
});
```

---

## Toolbar
**APG**: https://www.w3.org/WAI/ARIA/apg/patterns/toolbar/
**WCAG**: 4.1.2 (Name, Role, Value), 2.1.1 (Keyboard)

| Contract | APG rule | Assertion |
|---|---|---|
| Container has `role="toolbar"` | Required | `getByRole('toolbar')` |
| Has accessible name | Recommended | `toHaveAccessibleName()` |
| Tab moves into/out of toolbar | "Tab — Moves focus into and out of toolbar" | Focus enters/exits |
| ArrowRight/Left navigate items | "Right/Left Arrow — Moves focus among items" | Adjacent item focused |
| Home → first item | "Home — Moves focus to first item" | First item `toBeFocused()` |
| End → last item | "End — Moves focus to last item" | Last item `toBeFocused()` |

```ts
describe('Toolbar — APG Toolbar Pattern', () => {
  test('has required ARIA structure', async () => {
    render(<Toolbar label="Text Formatting" />);
    const toolbar = page.getByRole('toolbar', { name: /text formatting/i });
    await expect.element(toolbar).toBeVisible();
  });

  test('ArrowRight moves focus to next toolbar item', async () => {
    // APG: "Right Arrow — Moves focus to the next item"
    render(<Toolbar label="Formatting" />);
    const buttons = page.getByRole('toolbar').getByRole('button');
    await buttons.first().focus();
    await userEvent.keyboard('{ArrowRight}');
    await expect.element(buttons.nth(1)).toBeFocused();
  });

  test('Home and End navigate to first and last items', async () => {
    // APG: "Home — Moves focus to first item. End — Moves focus to last item."
    render(<Toolbar label="Formatting" />);
    const buttons = page.getByRole('toolbar').getByRole('button');
    await buttons.nth(1).focus();
    await userEvent.keyboard('{Home}');
    await expect.element(buttons.first()).toBeFocused();
    await userEvent.keyboard('{End}');
    await expect.element(buttons.last()).toBeFocused();
  });
});
```
