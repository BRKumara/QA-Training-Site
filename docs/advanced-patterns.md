# Advanced Automation Patterns

This guide covers advanced patterns and techniques for sophisticated Playwright test automation on the QA Training Site.

## 🎯 Design Patterns

### 1. Page Object Model (POM) Advanced

#### Base Page Class
```javascript
// pages/BasePage.js
class BasePage {
  constructor(page) {
    this.page = page;
    this.loadingSpinner = page.locator('.loading-spinner');
    this.errorMessages = page.locator('.error-message');
  }

  async navigate(path) {
    await this.page.goto(path);
    await this.waitForPageLoad();
  }

  async waitForPageLoad() {
    await this.page.waitForLoadState('networkidle');
    await this.loadingSpinner.waitFor({ state: 'hidden', timeout: 5000 });
  }

  async waitForElement(selector, options = {}) {
    return await this.page.locator(selector).waitFor(options);
  }

  async clickElement(selector, options = {}) {
    await this.waitForElement(selector);
    await this.page.locator(selector).click(options);
  }

  async fillText(selector, text, options = {}) {
    await this.waitForElement(selector);
    await this.page.locator(selector).fill(text, options);
  }

  async verifyElementVisible(selector, options = {}) {
    await expect(this.page.locator(selector)).toBeVisible(options);
  }

  async verifyElementHidden(selector, options = {}) {
    await expect(this.page.locator(selector)).toBeHidden(options);
  }

  async takeScreenshot(name) {
    await this.page.screenshot({ 
      path: `screenshots/${name}-${Date.now()}.png`,
      fullPage: true 
    });
  }

  async waitForAPIResponse(urlPattern) {
    return await this.page.waitForResponse(urlPattern);
  }

  async handleError() {
    const errors = await this.errorMessages.all();
    if (errors.length > 0) {
      const errorTexts = await Promise.all(
        errors.map(error => error.textContent())
      );
      throw new Error(`Page errors: ${errorTexts.join(', ')}`);
    }
  }
}

module.exports = BasePage;
```

#### Enhanced Page Objects
```javascript
// pages/LoginPage.js
const BasePage = require('./BasePage');

class LoginPage extends BasePage {
  constructor(page) {
    super(page);
    this.usernameInput = page.locator('#username');
    this.passwordInput = page.locator('#password');
    this.loginButton = page.locator('button[type="submit"]');
    this.rememberMeCheckbox = page.locator('#remember-me');
    this.forgotPasswordLink = page.locator('a[href*="forgot"]');
    this.errorMessage = page.locator('.error-message');
    this.successMessage = page.locator('.success-message');
  }

  async login(credentials, options = {}) {
    const { username, password, rememberMe = false } = credentials;
    
    await this.fillText('#username', username);
    await this.fillText('#password', password);
    
    if (rememberMe) {
      await this.clickElement('#remember-me');
    }
    
    await this.clickElement('button[type="submit"]');
    
    if (options.waitForNavigation) {
      await this.page.waitForNavigation();
    }
    
    await this.handleError();
  }

  async loginWithRetry(credentials, maxRetries = 3) {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        await this.login(credentials);
        return true;
      } catch (error) {
        if (attempt === maxRetries) throw error;
        
        console.log(`Login attempt ${attempt} failed, retrying...`);
        await this.page.waitForTimeout(1000);
        await this.page.reload();
      }
    }
  }

  async verifyLoginForm() {
    await this.verifyElementVisible('#username');
    await this.verifyElementVisible('#password');
    await this.verifyElementVisible('button[type="submit"]');
    await this.verifyElementVisible('a[href*="forgot"]');
  }

  async getErrorMessage() {
    await this.waitForElement('.error-message');
    return await this.errorMessage.textContent();
  }

  async isLoginSuccessful() {
    try {
      await this.page.waitForURL(/welcome/, { timeout: 5000 });
      return true;
    } catch {
      return false;
    }
  }
}

module.exports = { LoginPage };
```

### 2. Factory Pattern for Test Data

#### Test Data Factory
```javascript
// factories/UserFactory.js
class UserFactory {
  static createValidUser(overrides = {}) {
    return {
      username: `testuser_${Date.now()}`,
      email: `test${Date.now()}@example.com`,
      password: 'TestPassword123!',
      firstName: 'Test',
      lastName: 'User',
      phone: '1234567890',
      ...overrides
    };
  }

  static createInvalidUser(overrides = {}) {
    return {
      username: '',
      email: 'invalid-email',
      password: '123',
      firstName: '',
      lastName: '',
      phone: 'abc',
      ...overrides
    };
  }

  static createAdminUser(overrides = {}) {
    return {
      ...this.createValidUser(),
      role: 'admin',
      permissions: ['read', 'write', 'delete'],
      ...overrides
    };
  }

  static createUsers(count, factoryMethod = 'createValidUser') {
    return Array.from({ length: count }, (_, index) => 
      this[factoryMethod]({ username: `user${index + 1}` })
    );
  }
}

module.exports = UserFactory;
```

#### Form Data Factory
```javascript
// factories/FormFactory.js
class FormFactory {
  static createContactForm(overrides = {}) {
    return {
      name: 'John Doe',
      email: 'john.doe@example.com',
      subject: 'Test Subject',
      message: 'This is a test message',
      phone: '1234567890',
      ...overrides
    };
  }

  static createRegistrationForm(overrides = {}) {
    return {
      firstName: 'Jane',
      lastName: 'Smith',
      email: 'jane.smith@example.com',
      password: 'SecurePass123!',
      confirmPassword: 'SecurePass123!',
      termsAccepted: true,
      ...overrides
    };
  }

  static createInvalidForm(fieldToInvalidate = 'email') {
    const baseForm = this.createContactForm();
    
    const invalidations = {
      email: { ...baseForm, email: 'invalid-email' },
      phone: { ...baseForm, phone: 'abc' },
      name: { ...baseForm, name: '' },
      message: { ...baseForm, message: '' }
    };
    
    return invalidations[fieldToInvalidate] || baseForm;
  }
}

module.exports = FormFactory;
```

### 3. Builder Pattern for Complex Objects

#### Test Scenario Builder
```javascript
// builders/TestScenarioBuilder.js
class TestScenarioBuilder {
  constructor() {
    this.scenario = {
      name: '',
      description: '',
      steps: [],
      assertions: [],
      setup: [],
      teardown: [],
      tags: [],
      timeout: 30000
    };
  }

  withName(name) {
    this.scenario.name = name;
    return this;
  }

  withDescription(description) {
    this.scenario.description = description;
    return this;
  }

  addStep(action, locator, value = null) {
    this.scenario.steps.push({ action, locator, value });
    return this;
  }

  addAssertion(type, locator, expected) {
    this.scenario.assertions.push({ type, locator, expected });
    return this;
  }

  addSetup(action, details) {
    this.scenario.setup.push({ action, details });
    return this;
  }

  addTeardown(action, details) {
    this.scenario.teardown.push({ action, details });
    return this;
  }

  withTags(...tags) {
    this.scenario.tags = tags;
    return this;
  }

  withTimeout(timeout) {
    this.scenario.timeout = timeout;
    return this;
  }

  build() {
    return { ...this.scenario };
  }
}

module.exports = TestScenarioBuilder;
```

## 🔄 Advanced Test Patterns

### 1. Data-Driven Testing

#### Parameterized Tests
```javascript
// tests/auth/login-data-driven.spec.js
const { test, expect } = require('@playwright/test');
const { UserFactory } = require('../factories/UserFactory');
const { LoginPage } = require('../pages/LoginPage');

const testCases = [
  {
    name: 'valid login',
    user: UserFactory.createValidUser(),
    shouldSucceed: true,
    expectedUrl: /welcome/
  },
  {
    name: 'invalid email',
    user: UserFactory.createInvalidUser({ email: 'invalid' }),
    shouldSucceed: false,
    expectedError: 'Invalid email format'
  },
  {
    name: 'empty password',
    user: UserFactory.createInvalidUser({ password: '' }),
    shouldSucceed: false,
    expectedError: 'Password is required'
  },
  {
    name: 'non-existent user',
    user: UserFactory.createValidUser({ email: 'nonexistent@example.com' }),
    shouldSucceed: false,
    expectedError: 'User not found'
  }
];

test.describe('Data-Driven Login Tests', () => {
  testCases.forEach(({ name, user, shouldSucceed, expectedUrl, expectedError }) => {
    test(`should handle ${name}`, async ({ page }) => {
      const loginPage = new LoginPage(page);
      await loginPage.goto();
      
      await loginPage.login({
        username: user.email,
        password: user.password
      });
      
      if (shouldSucceed) {
        await expect(page).toHaveURL(expectedUrl);
        await expect(page.locator('text=Welcome')).toBeVisible();
      } else {
        const errorText = await loginPage.getErrorMessage();
        expect(errorText).toContain(expectedError);
      }
    });
  });
});
```

#### CSV Data Provider
```javascript
// utils/CSVDataProvider.js
const fs = require('fs');
const path = require('path');

class CSVDataProvider {
  static async loadTestData(filename) {
    const csvPath = path.join(__dirname, '../test-data', filename);
    const csvContent = fs.readFileSync(csvPath, 'utf8');
    
    return csvContent.split('\n')
      .filter(line => line.trim())
      .map(line => {
        const [username, password, expected] = line.split(',').map(item => item.trim());
        return { username, password, expected };
      });
  }
}

module.exports = CSVDataProvider;
```

### 2. Custom Test Fixtures

#### Enhanced Test Fixtures
```javascript
// fixtures/custom-fixtures.js
const { test as base } = require('@playwright/test');

// Extend base test with custom fixtures
const test = base.extend({
  // Custom page fixture with automatic error handling
  page: async ({ page }, use) => {
    // Handle uncaught errors
    page.on('pageerror', error => {
      console.error('Page error:', error);
    });
    
    // Handle console errors
    page.on('console', msg => {
      if (msg.type() === 'error') {
        console.error('Console error:', msg.text());
      }
    });
    
    await use(page);
  },
  
  // Authenticated page fixture
  authenticatedPage: async ({ page }, use) => {
    // Auto-login before each test
    await page.goto('/auth/login.html');
    await page.fill('#username', 'test@example.com');
    await page.fill('#password', 'password123');
    await page.click('button[type="submit"]');
    await page.waitForURL(/welcome/);
    
    await use(page);
  },
  
  // Test data fixture
  testData: async ({}, use) => {
    const data = {
      user: {
        valid: { email: 'test@example.com', password: 'password123' },
        invalid: { email: 'invalid@example.com', password: 'wrong' }
      },
      forms: {
        contact: { name: 'Test User', email: 'test@example.com' }
      }
    };
    
    await use(data);
  }
});

module.exports = { test };
```

### 3. API Testing Patterns

#### API Response Validation
```javascript
// tests/api/users-api.spec.js
const { test, expect } = require('@playwright/test');

test.describe('Users API', () => {
  const apiBase = 'http://localhost:3000/api';
  
  test('should get user list', async ({ request }) => {
    const response = await request.get(`${apiBase}/users`);
    
    expect(response.status()).toBe(200);
    
    const users = await response.json();
    expect(Array.isArray(users)).toBe(true);
    expect(users.length).toBeGreaterThan(0);
    
    // Validate user structure
    users.forEach(user => {
      expect(user).toHaveProperty('id');
      expect(user).toHaveProperty('name');
      expect(user).toHaveProperty('email');
    });
  });

  test('should create new user', async ({ request }) => {
    const newUser = {
      name: 'Test User',
      email: 'test@example.com',
      role: 'user'
    };
    
    const response = await request.post(`${apiBase}/users`, {
      data: newUser
    });
    
    expect(response.status()).toBe(201);
    
    const createdUser = await response.json();
    expect(createdUser.id).toBeDefined();
    expect(createdUser.name).toBe(newUser.name);
    expect(createdUser.email).toBe(newUser.email);
  });

  test('should handle API errors gracefully', async ({ request }) => {
    const response = await request.get(`${apiBase}/users/999`);
    
    expect(response.status()).toBe(404);
    
    const error = await response.json();
    expect(error.message).toContain('User not found');
  });
});
```

#### API Mocking in UI Tests
```javascript
// tests/auth/login-with-api-mock.spec.js
const { test, expect } = require('@playwright/test');

test.describe('Login with API Mocking', () => {
  test('should handle slow API response', async ({ page }) => {
    // Mock slow API response
    await page.route('**/api/login', route => {
      setTimeout(() => {
        route.fulfill({
          status: 200,
          contentType: 'application/json',
          body: JSON.stringify({ success: true, token: 'mock-token' })
        });
      }, 2000); // 2 second delay
    });
    
    await page.goto('/auth/login.html');
    await page.fill('#username', 'test@example.com');
    await page.fill('#password', 'password123');
    await page.click('button[type="submit"]');
    
    // Verify loading state
    await expect(page.locator('.loading')).toBeVisible();
    
    // Wait for completion
    await expect(page.locator('.loading')).toBeHidden();
    await expect(page).toHaveURL(/welcome/);
  });

  test('should handle API error', async ({ page }) => {
    // Mock API error
    await page.route('**/api/login', route => {
      route.fulfill({
        status: 401,
        contentType: 'application/json',
        body: JSON.stringify({ error: 'Invalid credentials' })
      });
    });
    
    await page.goto('/auth/login.html');
    await page.fill('#username', 'test@example.com');
    await page.fill('#password', 'wrongpassword');
    await page.click('button[type="submit"]');
    
    // Verify error message
    await expect(page.locator('.error-message')).toContainText('Invalid credentials');
  });
});
```

## 🎪 Advanced Interaction Patterns

### 1. Complex Form Handling

#### Multi-Step Form Wizard
```javascript
// tests/forms/form-wizard.spec.js
const { test, expect } = require('@playwright/test');

test.describe('Multi-Step Form Wizard', () => {
  test('should complete all steps', async ({ page }) => {
    await page.goto('/forms/wizard.html');
    
    // Step 1: Personal Information
    await page.fill('#firstName', 'John');
    await page.fill('#lastName', 'Doe');
    await page.fill('#email', 'john@example.com');
    await page.click('button:has-text("Next")');
    
    // Verify step 1 completion
    await expect(page.locator('.step-1')).toHaveClass(/completed/);
    await expect(page.locator('.step-2')).toHaveClass(/active/);
    
    // Step 2: Address Information
    await page.fill('#address', '123 Main St');
    await page.fill('#city', 'Anytown');
    await page.selectOption('#state', 'CA');
    await page.fill('#zipCode', '12345');
    await page.click('button:has-text("Next")');
    
    // Step 3: Review and Submit
    await expect(page.locator('.review-section')).toBeVisible();
    await expect(page.locator('text=John Doe')).toBeVisible();
    await expect(page.locator('text=john@example.com')).toBeVisible();
    
    await page.click('button:has-text("Submit")');
    
    // Verify completion
    await expect(page.locator('.success-message')).toBeVisible();
    await expect(page.locator('text=Form submitted successfully')).toBeVisible();
  });

  test('should validate each step', async ({ page }) => {
    await page.goto('/forms/wizard.html');
    
    // Try to proceed without filling step 1
    await page.click('button:has-text("Next")');
    
    // Verify validation errors
    await expect(page.locator('.error-message:has-text("First name is required")')).toBeVisible();
    await expect(page.locator('.error-message:has-text("Email is required")')).toBeVisible();
    
    // Fill step 1 and proceed
    await page.fill('#firstName', 'John');
    await page.fill('#lastName', 'Doe');
    await page.fill('#email', 'invalid-email'); // Invalid format
    await page.click('button:has-text("Next")');
    
    // Verify email validation
    await expect(page.locator('.error-message:has-text("Invalid email format")')).toBeVisible();
  });
});
```

### 2. Drag and Drop Patterns

#### Advanced Drag and Drop
```javascript
// tests/dragdrop/advanced-dragdrop.spec.js
const { test, expect } = require('@playwright/test');

test.describe('Advanced Drag and Drop', () => {
  test('should handle drag and drop with validation', async ({ page }) => {
    await page.goto('/dragdrop/index.html');
    
    // Get drag and drop elements
    const source = page.locator('.draggable-item[data-id="item1"]');
    const target = page.locator('.drop-zone[data-zone="zone1"]');
    
    // Verify initial state
    await expect(source).toBeVisible();
    await expect(target).toBeVisible();
    await expect(target).toHaveText(/Drop items here/);
    
    // Perform drag and drop
    await source.dragTo(target);
    
    // Verify drop success
    await expect(target.locator('.dropped-item')).toBeVisible();
    await expect(target.locator('.dropped-item')).toHaveText('Item 1');
    await expect(source).toBeHidden(); // Source should be hidden after drop
    
    // Verify zone validation
    await expect(target).toHaveClass(/valid-drop/);
  });

  test('should handle multiple drag and drop operations', async ({ page }) => {
    await page.goto('/dragdrop/index.html');
    
    const items = [
      { selector: '.draggable-item[data-id="item1"]', zone: '.drop-zone[data-zone="zone1"]' },
      { selector: '.draggable-item[data-id="item2"]', zone: '.drop-zone[data-zone="zone2"]' },
      { selector: '.draggable-item[data-id="item3"]', zone: '.drop-zone[data-zone="zone1"]' }
    ];
    
    // Perform multiple drag and drop operations
    for (const item of items) {
      const source = page.locator(item.selector);
      const target = page.locator(item.zone);
      
      await source.dragTo(target);
      await expect(target.locator('.dropped-item')).toBeVisible();
    }
    
    // Verify final state
    const zone1Items = page.locator('.drop-zone[data-zone="zone1"] .dropped-item');
    const zone2Items = page.locator('.drop-zone[data-zone="zone2"] .dropped-item');
    
    await expect(zone1Items).toHaveCount(2);
    await expect(zone2Items).toHaveCount(1);
  });
});
```

### 3. File Operations

#### Complex File Upload
```javascript
// tests/upload/advanced-upload.spec.js
const { test, expect } = require('@playwright/test');
const path = require('path');

test.describe('Advanced File Upload', () => {
  test('should handle multiple file upload', async ({ page }) => {
    await page.goto('/upload/index.html');
    
    // Prepare test files
    const files = [
      path.join(__dirname, '../test-files/file1.txt'),
      path.join(__dirname, '../test-files/file2.json'),
      path.join(__dirname, '../test-files/file3.csv')
    ];
    
    // Select multiple files
    const fileInput = page.locator('input[type="file"][multiple]');
    await fileInput.setInputFiles(files);
    
    // Verify file selection
    await expect(page.locator('.file-list')).toBeVisible();
    await expect(page.locator('.file-item')).toHaveCount(3);
    
    // Upload files
    await page.click('button:has-text("Upload All")');
    
    // Verify upload progress
    await expect(page.locator('.upload-progress')).toBeVisible();
    await expect(page.locator('.progress-bar')).toHaveAttribute('value', '100');
    
    // Verify upload completion
    await expect(page.locator('.upload-success')).toBeVisible();
    await expect(page.locator('.file-item.uploaded')).toHaveCount(3);
  });

  test('should handle file upload with validation', async ({ page }) => {
    await page.goto('/upload/index.html');
    
    // Try to upload invalid file type
    const invalidFile = path.join(__dirname, '../test-files/invalid.exe');
    const fileInput = page.locator('input[type="file"]');
    await fileInput.setInputFiles(invalidFile);
    
    // Verify validation error
    await expect(page.locator('.error-message')).toContainText('Invalid file type');
    
    // Try to upload oversized file
    const largeFile = path.join(__dirname, '../test-files/large-file.txt');
    await fileInput.setInputFiles(largeFile);
    
    // Verify size validation
    await expect(page.locator('.error-message')).toContainText('File size too large');
  });
});
```

## 🔄 State Management Patterns

### 1. Session Management
```javascript
// utils/SessionManager.js
class SessionManager {
  constructor(page) {
    this.page = page;
  }

  async saveSession(name) {
    const cookies = await this.page.context().cookies();
    const localStorage = await this.page.evaluate(() => {
      const storage = {};
      for (let i = 0; i < localStorage.length; i++) {
        const key = localStorage.key(i);
        storage[key] = localStorage.getItem(key);
      }
      return storage;
    });
    
    const sessionStorage = await this.page.evaluate(() => {
      const storage = {};
      for (let i = 0; i < sessionStorage.length; i++) {
        const key = sessionStorage.key(i);
        storage[key] = sessionStorage.getItem(key);
      }
      return storage;
    });
    
    return {
      name,
      cookies,
      localStorage,
      sessionStorage,
      timestamp: Date.now()
    };
  }

  async restoreSession(sessionData) {
    // Restore cookies
    await this.page.context().addCookies(sessionData.cookies);
    
    // Restore localStorage
    await this.page.evaluate((data) => {
      Object.keys(data).forEach(key => {
        localStorage.setItem(key, data[key]);
      });
    }, sessionData.localStorage);
    
    // Restore sessionStorage
    await this.page.evaluate((data) => {
      Object.keys(data).forEach(key => {
        sessionStorage.setItem(key, data[key]);
      });
    }, sessionData.sessionStorage);
    
    // Refresh page to apply session
    await this.page.reload();
  }
}

module.exports = SessionManager;
```

### 2. Test State Management
```javascript
// utils/TestStateManager.js
class TestStateManager {
  constructor() {
    this.state = new Map();
  }

  set(key, value) {
    this.state.set(key, value);
  }

  get(key) {
    return this.state.get(key);
  }

  has(key) {
    return this.state.has(key);
  }

  delete(key) {
    return this.state.delete(key);
  }

  clear() {
    this.state.clear();
  }

  // Store user data across tests
  storeUser(userData) {
    this.set('currentUser', userData);
  }

  getCurrentUser() {
    return this.get('currentUser');
  }

  // Store API responses
  storeApiResponse(endpoint, response) {
    const responses = this.get('apiResponses') || {};
    responses[endpoint] = response;
    this.set('apiResponses', responses);
  }

  getApiResponse(endpoint) {
    const responses = this.get('apiResponses') || {};
    return responses[endpoint];
  }
}

module.exports = TestStateManager;
```

## 🎯 Performance Testing Patterns

### 1. Load Testing
```javascript
// tests/performance/load-testing.spec.js
const { test, expect } = require('@playwright/test');

test.describe('Load Testing', () => {
  test('should handle concurrent users', async ({ browser }) => {
    const concurrentUsers = 10;
    const promises = [];
    
    for (let i = 0; i < concurrentUsers; i++) {
      promises.push(createUserSession(browser, i));
    }
    
    const results = await Promise.all(promises);
    
    // Verify all sessions succeeded
    results.forEach((result, index) => {
      expect(result.success).toBe(true);
      console.log(`User ${index + 1}: ${result.duration}ms`);
    });
    
    // Calculate performance metrics
    const avgDuration = results.reduce((sum, r) => sum + r.duration, 0) / results.length;
    expect(avgDuration).toBeLessThan(5000); // 5 second average
  });

  async function createUserSession(browser, userId) {
    const context = await browser.newContext();
    const page = await context.newPage();
    const startTime = Date.now();
    
    try {
      await page.goto('/auth/login.html');
      await page.fill('#username', `user${userId}@example.com`);
      await page.fill('#password', 'password123');
      await page.click('button[type="submit"]');
      await page.waitForURL(/welcome/);
      
      return {
        success: true,
        duration: Date.now() - startTime,
        userId
      };
    } catch (error) {
      return {
        success: false,
        duration: Date.now() - startTime,
        userId,
        error: error.message
      };
    } finally {
      await context.close();
    }
  }
});
```

### 2. Memory Usage Monitoring
```javascript
// tests/performance/memory-monitoring.spec.js
const { test, expect } = require('@playwright/test');

test.describe('Memory Usage Monitoring', () => {
  test('should monitor memory usage during navigation', async ({ page }) => {
    const memorySnapshots = [];
    
    // Monitor memory usage
    page.on('metrics', (metrics) => {
      memorySnapshots.push({
        timestamp: Date.now(),
        jsHeapUsedSize: metrics.metrics.JSHeapUsedSize,
        jsHeapTotalSize: metrics.metrics.JSHeapTotalSize
      });
    });
    
    // Navigate through multiple pages
    const pages = [
      '/auth/login.html',
      '/forms/index.html',
      '/tables/table.html',
      '/dynamic/index.html'
    ];
    
    for (const pageUrl of pages) {
      await page.goto(pageUrl);
      await page.waitForLoadState('networkidle');
      
      // Force garbage collection if available
      await page.evaluate(() => {
        if (window.gc) {
          window.gc();
        }
      });
    }
    
    // Analyze memory usage
    const initialMemory = memorySnapshots[0].jsHeapUsedSize;
    const finalMemory = memorySnapshots[memorySnapshots.length - 1].jsHeapUsedSize;
    const memoryGrowth = finalMemory - initialMemory;
    
    console.log(`Initial memory: ${initialMemory / 1024 / 1024} MB`);
    console.log(`Final memory: ${finalMemory / 1024 / 1024} MB`);
    console.log(`Memory growth: ${memoryGrowth / 1024 / 1024} MB`);
    
    // Verify memory usage is reasonable
    expect(memoryGrowth).toBeLessThan(50 * 1024 * 1024); // 50MB growth limit
  });
});
```

## 🎪 Visual Testing Patterns

### 1. Screenshot Comparison
```javascript
// tests/visual/visual-comparison.spec.js
const { test, expect } = require('@playwright/test');

test.describe('Visual Testing', () => {
  test('should match login page screenshot', async ({ page }) => {
    await page.goto('/auth/login.html');
    
    // Take full page screenshot
    await expect(page).toHaveScreenshot('login-page.png');
  });

  test('should match component screenshots', async ({ page }) => {
    await page.goto('/forms/index.html');
    
    // Take component screenshots
    const formElement = page.locator('.contact-form');
    await expect(formElement).toHaveScreenshot('contact-form.png');
    
    const buttonElement = page.locator('.submit-button');
    await expect(buttonElement).toHaveScreenshot('submit-button.png');
  });

  test('should handle responsive screenshots', async ({ page }) => {
    await page.goto('/');
    
    // Test different viewports
    const viewports = [
      { width: 1920, height: 1080, name: 'desktop' },
      { width: 768, height: 1024, name: 'tablet' },
      { width: 375, height: 667, name: 'mobile' }
    ];
    
    for (const viewport of viewports) {
      await page.setViewportSize(viewport);
      await page.waitForTimeout(1000); // Allow for responsive adjustments
      
      await expect(page).toHaveScreenshot(`homepage-${viewport.name}.png`);
    }
  });
});
```

## 🎯 Key Takeaways

1. **Design Patterns** - Use POM, Factory, and Builder patterns for maintainable code
2. **Data-Driven Testing** - Parameterize tests with various data sources
3. **Custom Fixtures** - Extend Playwright with domain-specific fixtures
4. **API Integration** - Combine UI and API testing for comprehensive coverage
5. **State Management** - Handle complex state across test sessions
6. **Performance Testing** - Monitor load times and memory usage
7. **Visual Testing** - Ensure UI consistency with screenshot comparison
8. **Error Handling** - Implement robust error handling and retry logic
9. **Mocking** - Use API mocking for isolated test scenarios
10. **Advanced Interactions** - Handle complex UI interactions like drag-and-drop

These advanced patterns will help you build sophisticated, maintainable, and robust automation frameworks with Playwright.
