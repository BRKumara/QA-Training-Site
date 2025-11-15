# Playwright Best Practices Guide

This guide outlines the best practices for effective automation testing with Playwright on the QA Training Site.

## 🎯 General Principles

### 1. Test Organization
- **Describe blocks**: Group related tests together
- **Test naming**: Use descriptive names that explain what the test does
- **File organization**: Keep tests in logical folders by feature

```javascript
test.describe('Authentication', () => {
  test('should login with valid credentials', async ({ page }) => {
    // Test implementation
  });
  
  test('should show error with invalid credentials', async ({ page }) => {
    // Test implementation
  });
});
```

### 2. Page Object Model (POM)
- **Separate locators from test logic**
- **Create reusable page classes**
- **Enculate page-specific actions**

```javascript
// pages/LoginPage.js
export class LoginPage {
  constructor(page) {
    this.page = page;
    this.usernameInput = page.locator('#username');
    this.passwordInput = page.locator('#password');
    this.loginButton = page.locator('button[type="submit"]');
    this.errorMessage = page.locator('.error-message');
  }

  async login(username, password) {
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }

  async getErrorMessage() {
    return await this.errorMessage.textContent();
  }
}
```

## 🔍 Locator Strategies

### 1. Prefer User-Facing Locators
```javascript
// ✅ Good - Use what users see
await page.locator('text=Login').click();
await page.locator('button:has-text("Submit")').click();

// ❌ Avoid - Implementation details
await page.locator('.btn-primary').click();
```

### 2. Use Test IDs When Available
```javascript
// ✅ Best for test-specific elements
await page.locator('[data-testid="login-button"]').click();
```

### 3. Chain Locators for Specificity
```javascript
// ✅ More specific chaining
await page.locator('.form-group').locator('input[type="email"]').fill('test@example.com');
```

## ⏱️ Wait Strategies

### 1. Use Built-in Auto-Waiting
```javascript
// ✅ Playwright automatically waits
await page.click('button');
await page.fill('input', 'text');
```

### 2. Explicit Waits When Needed
```javascript
// ✅ Wait for specific conditions
await page.waitForSelector('.loading', { state: 'hidden' });
await page.waitForResponse('**/api/login');
await page.waitForURL('**/dashboard');
```

### 3. Avoid Fixed Timeouts
```javascript
// ❌ Bad practice
await page.waitForTimeout(3000);

// ✅ Wait for element state
await page.waitForSelector('#dynamic-element', { state: 'visible' });
```

## 🧪 Test Data Management

### 1. Use Test Fixtures
```javascript
// ✅ Reusable test data
test.use({ 
  testData: {
    username: 'testuser@example.com',
    password: 'TestPassword123'
  }
});

test('should login with test data', async ({ page, testData }) => {
  await loginPage.login(testData.username, testData.password);
});
```

### 2. Generate Dynamic Data
```javascript
// ✅ Unique data for each test
const uniqueEmail = `test${Date.now()}@example.com`;
const randomUsername = `user_${Math.random().toString(36).substr(2, 9)}`;
```

## 🔧 Configuration Best Practices

### 1. Use Playwright Config
```javascript
// playwright.config.js
module.exports = {
  timeout: 30000,
  retries: 2,
  use: {
    headless: process.env.CI === 'true',
    viewport: { width: 1280, height: 720 },
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
  }
};
```

### 2. Environment-Specific Settings
```javascript
// config/environments.js
export const environments = {
  development: {
    baseURL: 'http://localhost:8000'
  },
  staging: {
    baseURL: 'https://staging.example.com'
  },
  production: {
    baseURL: 'https://example.com'
  }
};
```

## 📊 Assertions and Validations

### 1. Use Appropriate Assertions
```javascript
// ✅ Text content
await expect(page.locator('.welcome-message')).toHaveText('Welcome back!');

// ✅ Element visibility
await expect(page.locator('#dashboard')).toBeVisible();

// ✅ URL validation
await expect(page).toHaveURL(/.*dashboard/);

// ✅ Element count
await expect(page.locator('.menu-item')).toHaveCount(5);
```

### 2. Validate Multiple Conditions
```javascript
// ✅ Comprehensive validation
await expect(page.locator('.user-profile')).toBeTruthy();
await expect(page.locator('.user-name')).toHaveText('John Doe');
await expect(page.locator('.user-email')).toHaveText('john@example.com');
```

## 🔄 Handling Dynamic Content

### 1. Wait for AJAX Content
```javascript
// ✅ Wait for API response
const response = await page.waitForResponse('**/api/users');
const data = await response.json();
```

### 2. Handle Loading States
```javascript
// ✅ Wait for loading to complete
await page.waitForSelector('.loading', { state: 'hidden' });
await page.waitForSelector('.content-loaded', { state: 'visible' });
```

## 📱 Responsive Testing

### 1. Test Multiple Viewports
```javascript
const devices = ['Desktop Chrome', 'iPhone 12', 'iPad'];

devices.forEach(device => {
  test(`should work on ${device}`, async ({ page }) => {
    // Test implementation
  });
});
```

### 2. Use Viewport Configuration
```javascript
test.use({ viewport: { width: 375, height: 667 } }); // Mobile
test.use({ viewport: { width: 768, height: 1024 } }); // Tablet
```

## 🚨 Error Handling

### 1. Graceful Failure Handling
```javascript
try {
  await page.click('button');
} catch (error) {
  console.log('Element not clickable, trying alternative');
  await page.evaluate(() => document.querySelector('button').click());
}
```

### 2. Retry Failed Actions
```javascript
// ✅ Built-in retry mechanism
test.configure({ retries: 3 });

// ✅ Manual retry for specific actions
for (let i = 0; i < 3; i++) {
  try {
    await page.click('button');
    break;
  } catch (error) {
    if (i === 2) throw error;
    await page.waitForTimeout(1000);
  }
}
```

## 📝 Test Reporting

### 1. Use Built-in Reporters
```javascript
// playwright.config.js
reporter: [
  ['html', { outputFolder: 'playwright-report' }],
  ['json', { outputFile: 'test-results.json' }],
  ['junit', { outputFile: 'test-results.xml' }]
]
```

### 2. Add Custom Test Metadata
```javascript
test('should login successfully', async ({ page }) => {
  // Test implementation
}, {
  annotation: {
    type: 'issue',
    description: 'https://github.com/project/issues/123'
  }
});
```

## 🔒 Security Best Practices

### 1. Handle Sensitive Data
```javascript
// ✅ Use environment variables
const password = process.env.TEST_PASSWORD;

// ❌ Don't hardcode credentials
const password = 'plaintext-password';
```

### 2. Clean Up Test Data
```javascript
// ✅ Cleanup after each test
test.afterEach(async ({ page }) => {
  await page.evaluate(() => {
    localStorage.clear();
    sessionStorage.clear();
  });
});
```

## 🎭 Mocking and Stubbing

### 1. Mock API Responses
```javascript
// ✅ Mock API calls
await page.route('**/api/users', route => {
  route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify([{ id: 1, name: 'Test User' }])
  });
});
```

### 2. Handle Network Conditions
```javascript
// ✅ Simulate slow network
await page.context().setOffline(true);
await page.context().setOffline(false);
```

## 📈 Performance Considerations

### 1. Optimize Test Execution
```javascript
// ✅ Reuse browser context
test.beforeAll(async ({ browser }) => {
  context = await browser.newContext();
});

test.afterAll(async () => {
  await context.close();
});
```

### 2. Parallel Test Execution
```javascript
// playwright.config.js
workers: process.env.CI ? 4 : 2,
```

## 🎯 Key Takeaways

1. **Use Page Object Model** for maintainable tests
2. **Prefer user-facing locators** over implementation details
3. **Leverage Playwright's auto-waiting** capabilities
4. **Write clear, descriptive test names**
5. **Handle dynamic content with proper waits**
6. **Use environment-specific configurations**
7. **Implement proper cleanup and teardown**
8. **Use appropriate assertions for validation**
9. **Mock external dependencies when needed**
10. **Follow consistent code organization patterns**

Following these best practices will help you create robust, maintainable, and reliable automation tests with Playwright.
