# Playwright Locator Strategies Guide

This guide covers comprehensive locator strategies for finding elements in the QA Training Site using Playwright.

## 🎯 Locator Fundamentals

### 1. What is a Locator?
A locator is a way to find an element on a web page. Playwright provides multiple strategies to locate elements reliably.

```javascript
const element = page.locator('selector');
```

### 2. Auto-Waiting
Playwright automatically waits for elements to be actionable before performing actions.

```javascript
// ✅ Playwright waits for element to be visible and enabled
await page.locator('button').click();
```

## 🏷️ Basic Locator Strategies

### 1. By ID
```javascript
// ✅ Most reliable when available
await page.locator('#username').fill('testuser');
await page.locator('#login-button').click();
```

### 2. By Class Name
```javascript
// ✅ Good for styled elements
await page.locator('.btn-primary').click();
await page.locator('.form-group').locator('input').fill('text');
```

### 3. By Tag Name
```javascript
// ✅ Simple but less specific
await page.locator('button').click();
await page.locator('input[type="text"]').fill('text');
```

### 4. By Attribute
```javascript
// ✅ Useful for specific attributes
await page.locator('[data-testid="submit"]').click();
await page.locator('[name="email"]').fill('test@example.com');
await page.locator('input[required]').fill('required-field');
```

## 🎨 Text-Based Locators

### 1. Exact Text Match
```javascript
// ✅ Find elements by their exact text
await page.locator('text=Login').click();
await page.locator('text=Welcome to QA Training Site').isVisible();
```

### 2. Partial Text Match
```javascript
// ✅ Find elements containing text
await page.locator('text=Welcome').isVisible();
await page.locator('text=Training').click();
```

### 3. Text with CSS Selector
```javascript
// ✅ Combine text with CSS selector
await page.locator('button:has-text("Submit")').click();
await page.locator('.nav-item:has-text("Home")').click();
```

### 4. Case-Insensitive Text
```javascript
// ✅ Case-insensitive matching
await page.locator('text=/login/i').click();
```

## 🔗 CSS Selectors

### 1. Basic CSS Selectors
```javascript
// ✅ Element with class
await page.locator('.container').isVisible();

// ✅ Element with ID
await page.locator('#main-content').isVisible();

// ✅ Element with attribute
await page.locator('[data-role="button"]').click();
```

### 2. CSS Combinators
```javascript
// ✅ Descendant selector
await page.locator('.form-group input').fill('text');

// ✅ Child selector
await page.locator('.menu > .menu-item').click();

// ✅ Adjacent sibling
await page.locator('.label + .input').fill('text');

// ✅ General sibling
await page.locator('.error ~ .input').fill('text');
```

### 3. CSS Pseudo-classes
```javascript
// ✅ First/last element
await page.locator('li:first-child').click();
await page.locator('tr:last-child').isVisible();

// ✅ Nth element
await page.locator('li:nth-child(2)').click();
await page.locator('tr:nth-of-type(even)').isVisible();

// ✅ Element states
await page.locator('input:enabled').fill('text');
await page.locator('button:disabled').isDisabled();
await page.locator('input:checked').isChecked();
```

### 4. CSS Attribute Selectors
```javascript
// ✅ Attribute contains
await page.locator('[class*="btn"]').click();

// ✅ Attribute starts with
await page.locator('[class^="btn"]').click();

// ✅ Attribute ends with
await page.locator('[class$="primary"]').click();

// ✅ Attribute equals
await page.locator('[type="submit"]').click();
```

## 🎯 XPath Selectors

### 1. Basic XPath
```javascript
// ✅ Find by tag name
await page.locator('xpath=//button').click();

// ✅ Find by attribute
await page.locator('xpath=//button[@type="submit"]').click();

// ✅ Find by text
await page.locator('xpath=//button[text()="Submit"]').click();
```

### 2. XPath Functions
```javascript
// ✅ Contains text
await page.locator('xpath=//button[contains(text(), "Submit")]').click();

// ✅ Starts with text
await page.locator('xpath=//button[starts-with(text(), "Sub")]').click();

// ✅ Normalize space (handles whitespace)
await page.locator('xpath=//button[normalize-space(text())="Submit"]').click();
```

### 3. XPath Axes
```javascript
// ✅ Parent
await page.locator('xpath=//input/../label').click();

// ✅ Following sibling
await page.locator('xpath=//label/following-sibling::input').fill('text');

// ✅ Preceding sibling
await page.locator('xpath=//input/preceding-sibling::label').click();

// ✅ Ancestor
await page.locator('xpath=//input/ancestor::form').locator('button').click();
```

## 🎭 Advanced Locator Techniques

### 1. Chaining Locators
```javascript
// ✅ Chain multiple locators for specificity
await page.locator('.form-group')
  .locator('input[type="email"]')
  .fill('test@example.com');

// ✅ Filter locators
await page.locator('.button')
  .filter({ hasText: 'Submit' })
  .click();
```

### 2. Locating by Role
```javascript
// ✅ Find by ARIA role
await page.locator('role=button[name="Submit"]').click();
await page.locator('role=textbox[name="Email"]').fill('test@example.com');
await page.locator('role=link[name="Home"]').click();
```

### 3. Locating by Label
```javascript
// ✅ Find form controls by their labels
await page.locator('label:has-text("Email")').locator('input').fill('test@example.com');
await page.locator('text=Password').locator('input').fill('password');
```

### 4. Locating by Placeholders
```javascript
// ✅ Find inputs by placeholder text
await page.locator('input[placeholder="Enter your email"]').fill('test@example.com');
await page.locator('input[placeholder*="password"]').fill('password');
```

## 🔄 Dynamic Element Locators

### 1. Handling Dynamic IDs
```javascript
// ✅ Use partial attribute matching
await page.locator('[id*="username"]').fill('testuser');
await page.locator('[id^="user-"]').fill('testuser');
await page.locator('[id$="-input"]').fill('testuser');
```

### 2. Handling Dynamic Classes
```javascript
// ✅ Use class contains
await page.locator('[class*="active"]').click();
await page.locator('[class*="error"]').isVisible();
```

### 3. Using Data Attributes
```javascript
// ✅ Best practice for test-specific elements
await page.locator('[data-testid="login-button"]').click();
await page.locator('[data-cy="submit-form"]').click();
await page.locator('[test-id="user-menu"]').click();
```

## 🎪 Frame and Shadow DOM Locators

### 1. iFrame Locators
```javascript
// ✅ Get frame content
const frame = page.frameLocator('iframe[name="content"]');
await frame.locator('#inner-button').click();

// ✅ Nested frames
const nestedFrame = frame.frameLocator('iframe[name="nested"]');
await nestedFrame.locator('.deep-element').click();
```

### 2. Shadow DOM Locators
```javascript
// ✅ Pierce shadow DOM
await page.locator('my-component').locator('::shadow(.shadow-element)').click();

// ✅ Deep shadow piercing
await page.locator('my-component')
  .locator('::shadow(nested-component')
  .locator('::shadow(.deep-element'))
  .click();
```

## 🎯 Best Practices for Locators

### 1. Priority Order
1. **Test IDs** - `data-testid`, `data-cy`
2. **User-facing text** - `text=`, `role=` 
3. **Semantic HTML** - `label`, `aria-label`
4. **CSS selectors** - `.class`, `#id`
5. **XPath** - When other methods fail

### 2. Avoid Brittle Locators
```javascript
// ❌ Bad - Implementation details
await page.locator('.btn.btn-primary.btn-lg').click();
await page.locator('div:nth-child(2) > div > button').click();

// ✅ Good - User-facing
await page.locator('button:has-text("Submit")').click();
await page.locator('[data-testid="submit-button"]').click();
```

### 3. Make Tests Resilient
```javascript
// ✅ Use multiple strategies as fallback
const submitButton = page.locator(
  '[data-testid="submit"], button:has-text("Submit"), input[type="submit"]'
);
await submitButton.click();
```

## 🔍 Locator Debugging

### 1. Count Elements
```javascript
// ✅ Check how many elements match
const count = await page.locator('.button').count();
console.log(`Found ${count} buttons`);
```

### 2. Get All Elements
```javascript
// ✅ Get all matching elements
const buttons = await page.locator('.button').all();
for (const button of buttons) {
  const text = await button.textContent();
  console.log(`Button text: ${text}`);
}
```

### 3. First and Last Elements
```javascript
// ✅ Get specific elements
const firstButton = page.locator('.button').first();
const lastButton = page.locator('.button').last();
const nthButton = page.locator('.button').nth(2);
```

### 4. Element Visibility
```javascript
// ✅ Check element state
const isVisible = await page.locator('.button').isVisible();
const isEnabled = await page.locator('.button').isEnabled();
const isChecked = await page.locator('.checkbox').isChecked();
```

## 🎪 Special Scenarios

### 1. Table Elements
```javascript
// ✅ Find table cells by content
await page.locator('td:has-text("John Doe")').click();

// ✅ Find by column and row
await page.locator('table tr:nth-child(2) td:nth-child(3)').click();

// ✅ Find table headers
await page.locator('th:has-text("Name")').isVisible();
```

### 2. List Elements
```javascript
// ✅ Find list items
await page.locator('ul li:has-text("Option 1")').click();

// ✅ Find dropdown options
await page.locator('select option[value="option1"]').click();
```

### 3. Modal and Dialog Elements
```javascript
// ✅ Find modal content
await page.locator('.modal').locator('h2:has-text("Confirmation")').isVisible();
await page.locator('.modal').locator('button:has-text("OK")').click();
```

## 🚀 Performance Tips

### 1. Use Specific Locators
```javascript
// ✅ More specific = faster
await page.locator('#login-form input[name="username"]').fill('test');

// ❌ Less specific = slower
await page.locator('input').fill('test');
```

### 2. Avoid Complex XPath
```javascript
// ✅ Simple CSS is faster
await page.locator('.submit-button').click();

// ❌ Complex XPath is slower
await page.locator('//div[@class="container"]/div[2]/div[1]/button').click();
```

### 3. Cache Locators
```javascript
// ✅ Cache frequently used locators
const loginButton = page.locator('button:has-text("Login")');
await loginButton.click();
await loginButton.waitFor({ state: 'hidden' });
```

## 🎯 Key Takeaways

1. **Prefer test IDs** for test-specific elements
2. **Use user-facing text** when test IDs aren't available
3. **Chain locators** for better specificity
4. **Avoid implementation details** in locators
5. **Use role-based locators** for accessibility
6. **Handle dynamic elements** with partial matching
7. **Debug locators** with count and visibility checks
8. **Consider performance** when writing complex selectors
9. **Test locators** in browser console first
10. **Document locator strategies** for team consistency

## 📚 Quick Reference

| Strategy | Example | Best For |
|----------|---------|----------|
| Test ID | `[data-testid="submit"]` | Test-specific elements |
| Text | `text=Submit` | User-facing elements |
| Role | `role=button[name="Submit"]` | Accessible elements |
| CSS | `.btn-primary` | Styled elements |
| XPath | `//button[text()="Submit"]` | Complex selections |
| Label | `label:has-text("Email")` | Form controls |

Mastering these locator strategies will help you write robust, maintainable, and reliable Playwright tests.
