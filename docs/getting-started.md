# Getting Started with Playwright

This guide helps you get started with Playwright automation testing on the QA Training Site.

## 🚀 Prerequisites

### System Requirements
- **Node.js** version 14 or higher
- **npm** or **yarn** package manager
- **Modern web browser** (Chrome, Firefox, Safari, Edge)
- **Code editor** (VS Code recommended)

### Installation
```bash
# Check Node.js version
node --version

# Install Playwright
npm init playwright@latest

# Or install globally
npm install -g playwright
```

## 📁 Project Setup

### 1. Initialize Project
```bash
# Create project directory
mkdir qa-training-tests
cd qa-training-tests

# Initialize npm project
npm init -y

# Install Playwright
npm install --save-dev @playwright/test
```

### 2. Install Browsers
```bash
# Install browser binaries
npx playwright install

# Install specific browsers
npx playwright install chromium firefox webkit

# Install browser dependencies (Linux)
npx playwright install-deps
```

### 3. Configure Playwright
Create `playwright.config.js`:
```javascript
const { defineConfig, devices } = require('@playwright/test');

module.exports = defineConfig({
  // Test directory
  testDir: './tests',
  
  // Global settings
  timeout: 30000,
  expect: { timeout: 5000 },
  
  // Retry failed tests
  retries: process.env.CI ? 2 : 0,
  
  // Reporter configuration
  reporter: [
    ['html', { outputFolder: 'playwright-report' }],
    ['json', { outputFile: 'test-results.json' }],
    ['junit', { outputFile: 'test-results.xml' }]
  ],
  
  // Global setup and teardown
  globalSetup: require('./tests/global-setup'),
  globalTeardown: require('./tests/global-teardown'),
  
  // Use configuration
  use: {
    // Browser settings
    headless: process.env.CI === 'true',
    viewport: { width: 1280, height: 720 },
    
    // Screenshots and videos
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'retain-on-failure',
    
    // Base URL for tests
    baseURL: 'http://localhost:8000',
    
    // Ignore HTTPS errors
    ignoreHTTPSErrors: true,
  },
  
  // Configure projects for different browsers
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    
    // Mobile testing
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],
  
  // Test organization
  testMatch: [
    '**/*.spec.js',
    '**/*.test.js'
  ],
  
  // Ignore patterns
  testIgnore: [
    '**/node_modules/**',
    '**/dist/**'
  ],
});
```

## 📂 Project Structure

```
qa-training-tests/
├── tests/
│   ├── auth/
│   │   ├── login.spec.js
│   │   └── logout.spec.js
│   ├── forms/
│   │   ├── contact.spec.js
│   │   └── registration.spec.js
│   ├── api/
│   │   └── users.spec.js
│   ├── fixtures/
│   │   └── test-data.js
│   ├── pages/
│   │   ├── LoginPage.js
│   │   └── HomePage.js
│   ├── utils/
│   │   └── helpers.js
│   ├── global-setup.js
│   └── global-teardown.js
├── playwright.config.js
├── package.json
├── .env
└── README.md
```

## 🎯 Your First Test

### 1. Create Test File
Create `tests/auth/login.spec.js`:
```javascript
const { test, expect } = require('@playwright/test');
const { LoginPage } = require('../pages/LoginPage');

test.describe('Authentication', () => {
  let loginPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    await page.goto('/auth/login.html');
  });

  test('should show login form', async ({ page }) => {
    // Verify page title
    await expect(page).toHaveTitle(/Login/);
    
    // Verify form elements
    await expect(loginPage.usernameInput).toBeVisible();
    await expect(loginPage.passwordInput).toBeVisible();
    await expect(loginPage.loginButton).toBeVisible();
  });

  test('should login with valid credentials', async ({ page }) => {
    // Perform login
    await loginPage.login('testuser@example.com', 'password123');
    
    // Verify successful login
    await expect(page).toHaveURL(/welcome/);
    await expect(page.locator('text=Welcome')).toBeVisible();
  });

  test('should show error with invalid credentials', async ({ page }) => {
    // Attempt login with invalid credentials
    await loginPage.login('invalid@example.com', 'wrongpassword');
    
    // Verify error message
    await expect(loginPage.errorMessage).toBeVisible();
    await expect(loginPage.errorMessage).toContainText('Invalid credentials');
  });
});
```

### 2. Create Page Object
Create `tests/pages/LoginPage.js`:
```javascript
class LoginPage {
  constructor(page) {
    this.page = page;
    this.usernameInput = page.locator('#username');
    this.passwordInput = page.locator('#password');
    this.loginButton = page.locator('button[type="submit"]');
    this.errorMessage = page.locator('.error-message');
    this.successMessage = page.locator('.success-message');
  }

  async goto() {
    await this.page.goto('/auth/login.html');
  }

  async login(username, password) {
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }

  async getErrorMessage() {
    return await this.errorMessage.textContent();
  }

  async getSuccessMessage() {
    return await this.successMessage.textContent();
  }

  async isLoggedIn() {
    return await this.page.locator('text=Welcome').isVisible();
  }
}

module.exports = { LoginPage };
```

## 🏃 Running Tests

### 1. Run All Tests
```bash
# Run all tests
npx playwright test

# Run tests with specific browser
npx playwright test --project=chromium

# Run tests in headed mode (show browser)
npx playwright test --headed
```

### 2. Run Specific Tests
```bash
# Run specific test file
npx playwright test tests/auth/login.spec.js

# Run tests with pattern
npx playwright test --grep "login"

# Run specific test line
npx playwright test tests/auth/login.spec.js:15
```

### 3. Debug Tests
```bash
# Run in debug mode
npx playwright test --debug

# Run with trace viewer
npx playwright test --trace on

# Run with inspector
npx playwright test --inspect
```

## 📊 Viewing Results

### 1. HTML Report
```bash
# Open HTML report
npx playwright show-report

# Open specific report
npx playwright show-report playwright-report
```

### 2. Test Results
```bash
# View test results
cat test-results.json

# View JUnit XML
cat test-results.xml
```

### 3. Screenshots and Videos
```bash
# View screenshots
ls test-results/

# View videos
ls test-results/videos/
```

## 🔧 Configuration Options

### 1. Environment Variables
Create `.env` file:
```bash
# Base URL
BASE_URL=http://localhost:8000

# Test credentials
TEST_EMAIL=test@example.com
TEST_PASSWORD=password123

# Browser settings
HEADED=false
CI=false
```

### 2. Custom Configurations
```javascript
// playwright.config.js
const config = {
  // Development configuration
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:8000',
    headless: process.env.HEADED === 'true' ? false : true,
  },
  
  // CI configuration
  ...(process.env.CI && {
    retries: 3,
    workers: 4,
    reporter: [['html'], ['json'], ['github']],
  }),
};
```

## 📝 Test Data Management

### 1. Test Fixtures
Create `tests/fixtures/test-data.js`:
```javascript
const testUsers = {
  valid: {
    email: 'testuser@example.com',
    password: 'ValidPassword123',
    name: 'Test User'
  },
  invalid: {
    email: 'invalid@example.com',
    password: 'WrongPassword'
  },
  admin: {
    email: 'admin@example.com',
    password: 'AdminPassword123',
    role: 'admin'
  }
};

const testForms = {
  contact: {
    name: 'John Doe',
    email: 'john@example.com',
    message: 'This is a test message'
  },
  registration: {
    firstName: 'Jane',
    lastName: 'Smith',
    email: 'jane@example.com',
    password: 'SecurePass123'
  }
};

module.exports = { testUsers, testForms };
```

### 2. Using Test Data
```javascript
const { testUsers } = require('../fixtures/test-data');

test('should login with valid user data', async ({ page }) => {
  const user = testUsers.valid;
  await loginPage.login(user.email, user.password);
  await expect(page.locator(`text=${user.name}`)).toBeVisible();
});
```

## 🎯 Common Test Patterns

### 1. BeforeEach/AfterEach Hooks
```javascript
test.describe('Form Testing', () => {
  test.beforeEach(async ({ page }) => {
    // Navigate to form before each test
    await page.goto('/forms/index.html');
  });

  test.afterEach(async ({ page }) => {
    // Clean up after each test
    await page.evaluate(() => {
      localStorage.clear();
      sessionStorage.clear();
    });
  });
});
```

### 2. Test Groups
```javascript
test.describe('Authentication Flow', () => {
  test.describe('Login', () => {
    test('valid credentials', async ({ page }) => {
      // Test implementation
    });

    test('invalid credentials', async ({ page }) => {
      // Test implementation
    });
  });

  test.describe('Logout', () => {
    test('successful logout', async ({ page }) => {
      // Test implementation
    });
  });
});
```

### 3. Parameterized Tests
```javascript
const browsers = ['chromium', 'firefox', 'webkit'];
const viewports = ['Desktop', 'Mobile', 'Tablet'];

browsers.forEach(browser => {
  test.describe(`${browser} browser`, () => {
    test.use({ browserName: browser });
    
    test('should work on all browsers', async ({ page }) => {
      // Cross-browser test
    });
  });
});
```

## 🛠️ Advanced Features

### 1. Network Mocking
```javascript
test('should handle API responses', async ({ page }) => {
  // Mock API response
  await page.route('**/api/login', route => {
    route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify({ success: true, token: 'abc123' })
    });
  });

  // Continue with test
  await page.goto('/auth/login.html');
  await loginPage.login('test@example.com', 'password');
});
```

### 2. File Upload/Download
```javascript
test('should upload file', async ({ page }) => {
  await page.goto('/upload/index.html');
  
  // Select file
  const fileInput = page.locator('input[type="file"]');
  await fileInput.setInputFiles('path/to/test-file.txt');
  
  // Submit form
  await page.locator('button[type="submit"]').click();
  
  // Verify upload
  await expect(page.locator('text=File uploaded successfully')).toBeVisible();
});
```

### 3. Handling Alerts
```javascript
test('should handle JavaScript alerts', async ({ page }) => {
  await page.goto('/alerts/index.html');
  
  // Handle alert
  page.on('dialog', async dialog => {
    expect(dialog.message()).toBe('Test alert');
    await dialog.accept();
  });
  
  // Trigger alert
  await page.locator('button:has-text("Show Alert")').click();
});
```

## 🐛 Debugging Tips

### 1. Using Playwright Inspector
```bash
# Run with inspector
npx playwright test --debug

# Or add to test
test('debug example', async ({ page }) => {
  await page.pause(); // Pause execution
  // Test code here
});
```

### 2. Screenshots on Failure
```javascript
test.afterEach(async ({ page }, testInfo) => {
  if (testInfo.status !== 'passed') {
    await page.screenshot({ 
      path: `screenshots/${testInfo.title}.png`,
      fullPage: true 
    });
  }
});
```

### 3. Console Logging
```javascript
test('debug with console', async ({ page }) => {
  page.on('console', msg => {
    console.log('PAGE LOG:', msg.text());
  });
  
  // Test code
});
```

## 🚀 Next Steps

1. **Explore more features** - Check the [Best Practices Guide](./best-practices.md)
2. **Learn locator strategies** - Read the [Locator Guide](./locators.md)
3. **Advanced patterns** - See [Advanced Automation Patterns](./advanced-patterns.md)
4. **Debugging techniques** - Check [Debugging Guide](./debugging.md)

## 📚 Resources

- [Official Playwright Documentation](https://playwright.dev/)
- [Playwright API Reference](https://playwright.dev/docs/api/class-playwright)
- [Community Discord](https://discord.gg/playwright)
- [GitHub Discussions](https://github.com/microsoft/playwright/discussions)

## 🎯 Quick Checklist

- [ ] Install Node.js and Playwright
- [ ] Set up project structure
- [ ] Configure playwright.config.js
- [ ] Create first test file
- [ ] Set up page objects
- [ ] Run tests locally
- [ ] Check HTML report
- [ ] Configure CI/CD if needed

You're now ready to start automating tests with Playwright on the QA Training Site! 🎉
