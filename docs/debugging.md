# Debugging Guide for Playwright

This comprehensive guide covers debugging techniques and troubleshooting strategies for Playwright test automation on the QA Training Site.

## 🔍 Debugging Tools and Techniques

### 1. Playwright Inspector

#### Using the Inspector
```bash
# Run tests with inspector
npx playwright test --debug

# Run specific test with inspector
npx playwright test tests/auth/login.spec.js --debug

# Run with inspector in VS Code
# Add breakpoint in test and run with --debug
```

#### Inspector Features
- **Step-by-step execution** - Pause at each step
- **Element inspection** - Hover to highlight elements
- **Console access** - Execute JavaScript in browser context
- **Network monitoring** - View network requests
- **Timeline recording** - Record and replay interactions

#### Programmatic Debugging
```javascript
// Pause execution at specific point
test('debug example', async ({ page }) => {
  await page.goto('/auth/login.html');
  await page.pause(); // Execution pauses here
  
  await page.fill('#username', 'test@example.com');
  await page.pause(); // Another pause point
  
  await page.fill('#password', 'password123');
  await page.click('button[type="submit"]');
});
```

### 2. Trace Viewer

#### Enabling Trace Recording
```javascript
// playwright.config.js
module.exports = {
  use: {
    trace: 'retain-on-failure', // or 'on', 'off', 'retain-on-failure'
  },
};

// Or programmatically
test('with trace', async ({ page }) => {
  await page.goto('/auth/login.html');
  // Test code here
});
```

#### Viewing Traces
```bash
# Open trace viewer
npx playwright show-trace trace.zip

# View latest trace
npx playwright show-trace test-results/trace.zip
```

#### Trace Analysis
- **Timeline view** - See execution timeline
- **Network requests** - Analyze API calls
- **Console logs** - View browser console output
- **Screenshots** - See page state at each step
- **Source code** - Navigate to test code locations

### 3. Screenshots and Videos

#### Automatic Screenshots
```javascript
// playwright.config.js
module.exports = {
  use: {
    screenshot: 'only-on-failure', // or 'on', 'off'
    video: 'retain-on-failure',   // or 'on', 'off'
  },
};
```

#### Manual Screenshots
```javascript
test('manual screenshots', async ({ page }) => {
  await page.goto('/auth/login.html');
  
  // Full page screenshot
  await page.screenshot({ 
    path: 'screenshots/login-page.png',
    fullPage: true 
  });
  
  // Element screenshot
  await page.locator('.login-form').screenshot({ 
    path: 'screenshots/login-form.png' 
  });
  
  // Screenshot with clipping
  await page.screenshot({ 
    path: 'screenshots/viewport.png',
    clip: { x: 0, y: 0, width: 800, height: 600 }
  });
});
```

## 🐛 Common Debugging Scenarios

### 1. Element Not Found Issues

#### Troubleshooting Steps
```javascript
test('debug element not found', async ({ page }) => {
  await page.goto('/forms/index.html');
  
  // Step 1: Check if element exists
  const element = page.locator('#nonexistent-element');
  const exists = await element.count() > 0;
  console.log(`Element exists: ${exists}`);
  
  // Step 2: Wait for element with timeout
  try {
    await element.waitFor({ timeout: 5000 });
    console.log('Element found after wait');
  } catch (error) {
    console.log('Element not found after timeout');
  }
  
  // Step 3: Check page content
  const pageContent = await page.content();
  console.log('Page contains element:', pageContent.includes('target-id'));
  
  // Step 4: Try alternative selectors
  const alternatives = [
    '[data-testid="target"]',
    '.target-class',
    'text=Target Text',
    'xpath=//*[@id="target"]'
  ];
  
  for (const selector of alternatives) {
    const count = await page.locator(selector).count();
    console.log(`${selector}: ${count} elements found`);
  }
});
```

#### Dynamic Element Debugging
```javascript
test('debug dynamic elements', async ({ page }) => {
  await page.goto('/dynamic/index.html');
  
  // Wait for dynamic content
  await page.waitForFunction(() => {
    return document.querySelector('.dynamic-element') !== null;
  });
  
  // Or wait for specific condition
  await page.waitForSelector('.dynamic-element', { state: 'visible' });
  
  // Debug with console
  await page.evaluate(() => {
    console.log('Current DOM:', document.documentElement.innerHTML);
  });
});
```

### 2. Timing and Wait Issues

#### Wait Strategy Debugging
```javascript
test('debug timing issues', async ({ page }) => {
  await page.goto('/api/index.html');
  
  // Method 1: Wait for specific network request
  console.time('api-response');
  const response = await page.waitForResponse('**/api/data');
  console.timeEnd('api-response');
  console.log('Response status:', response.status());
  
  // Method 2: Wait for URL change
  console.time('navigation');
  await page.waitForURL('**/success');
  console.timeEnd('navigation');
  
  // Method 3: Wait for element state
  console.time('element-visibility');
  await page.waitForSelector('.success-message', { state: 'visible' });
  console.timeEnd('element-visibility');
  
  // Method 4: Custom wait condition
  console.time('custom-condition');
  await page.waitForFunction(() => {
    return window.appState === 'loaded';
  });
  console.timeEnd('custom-condition');
});
```

#### Timeout Debugging
```javascript
test('debug timeouts', async ({ page }) => {
  await page.goto('/slow-page.html');
  
  // Increase timeout for specific operation
  await page.locator('.slow-element').waitFor({ timeout: 30000 });
  
  // Or configure globally
  test.setTimeout(60000); // 60 seconds
  
  // Debug with progress indicators
  let attempts = 0;
  const maxAttempts = 30;
  
  while (attempts < maxAttempts) {
    const element = page.locator('.target-element');
    const isVisible = await element.isVisible().catch(() => false);
    
    if (isVisible) {
      console.log(`Element found after ${attempts} attempts`);
      break;
    }
    
    attempts++;
    await page.waitForTimeout(1000);
    console.log(`Attempt ${attempts}: Element not yet visible`);
  }
});
```

### 3. Form Submission Issues

#### Form Debugging
```javascript
test('debug form submission', async ({ page }) => {
  await page.goto('/forms/contact.html');
  
  // Debug form elements
  const formElements = {
    name: page.locator('#name'),
    email: page.locator('#email'),
    message: page.locator('#message'),
    submit: page.locator('button[type="submit"]')
  };
  
  // Check element states
  for (const [name, element] of Object.entries(formElements)) {
    const isVisible = await element.isVisible();
    const isEnabled = await element.isEnabled();
    const isRequired = await element.getAttribute('required');
    
    console.log(`${name}: visible=${isVisible}, enabled=${isEnabled}, required=${isRequired}`);
  }
  
  // Fill form with debugging
  await formElements.name.fill('Test User');
  await formElements.email.fill('test@example.com');
  await formElements.message.fill('Test message');
  
  // Check form validation
  const isValid = await page.evaluate(() => {
    const form = document.querySelector('form');
    return form.checkValidity();
  });
  console.log('Form is valid:', isValid);
  
  // Monitor network request
  page.on('request', request => {
    console.log('Request:', request.url(), request.method());
  });
  
  page.on('response', response => {
    console.log('Response:', response.url(), response.status());
  });
  
  // Submit form
  await formElements.submit.click();
  
  // Debug response
  await page.waitForResponse('**/submit');
  const responseText = await page.locator('.response-message').textContent();
  console.log('Form response:', responseText);
});
```

### 4. Navigation Issues

#### Navigation Debugging
```javascript
test('debug navigation', async ({ page }) => {
  // Monitor navigation events
  page.on('load', () => console.log('Page loaded'));
  page.on('domcontentloaded', () => console.log('DOM loaded'));
  page.on('framenavigated', () => console.log('Frame navigated'));
  
  await page.goto('/');
  
  // Debug current URL
  console.log('Current URL:', page.url());
  
  // Wait for specific navigation
  await Promise.all([
    page.waitForNavigation(),
    page.click('a[href="/about"]')
  ]);
  
  console.log('Navigated to:', page.url());
  
  // Debug navigation history
  const history = await page.evaluate(() => ({
    length: history.length,
    current: history.state?.index,
    states: history.length
  }));
  console.log('Navigation history:', history);
});
```

## 🔧 Advanced Debugging Techniques

### 1. Browser Console Integration

#### Console Logging
```javascript
test('console debugging', async ({ page }) => {
  // Capture console messages
  const consoleMessages = [];
  page.on('console', msg => {
    consoleMessages.push({
      type: msg.type(),
      text: msg.text(),
      location: msg.location()
    });
    console.log(`[${msg.type()}] ${msg.text()}`);
  });
  
  // Execute JavaScript in browser context
  await page.goto('/dynamic/index.html');
  
  const elementCount = await page.evaluate(() => {
    console.log('Counting elements in browser context');
    const elements = document.querySelectorAll('.dynamic-element');
    console.log(`Found ${elements.length} elements`);
    return elements.length;
  });
  
  console.log('Element count from browser:', elementCount);
  
  // Debug with browser dev tools
  await page.evaluate(() => {
    debugger; // Breakpoint in browser dev tools
  });
});
```

#### Error Monitoring
```javascript
test('error monitoring', async ({ page }) => {
  // Capture page errors
  page.on('pageerror', error => {
    console.error('Page error:', error.message);
    console.error('Stack trace:', error.stack);
  });
  
  // Capture JavaScript errors
  page.on('console', msg => {
    if (msg.type() === 'error') {
      console.error('Console error:', msg.text());
    }
  });
  
  // Capture request failures
  page.on('requestfailed', request => {
    console.error('Request failed:', request.url(), request.failure());
  });
  
  await page.goto('/page-with-errors.html');
});
```

### 2. Network Debugging

#### Request/Response Monitoring
```javascript
test('network debugging', async ({ page }) => {
  // Monitor all requests
  const requests = [];
  page.on('request', request => {
    requests.push({
      url: request.url(),
      method: request.method(),
      headers: request.headers(),
      postData: request.postData()
    });
    console.log(`Request: ${request.method()} ${request.url()}`);
  });
  
  // Monitor responses
  const responses = [];
  page.on('response', response => {
    responses.push({
      url: response.url(),
      status: response.status(),
      headers: response.headers()
    });
    console.log(`Response: ${response.status()} ${response.url()}`);
  });
  
  await page.goto('/api/index.html');
  
  // Wait for specific API call
  const apiResponse = await page.waitForResponse('**/api/users');
  const responseData = await apiResponse.json();
  console.log('API response data:', responseData);
});
```

#### Network Mocking for Debugging
```javascript
test('mock network for debugging', async ({ page }) => {
  // Mock slow response
  await page.route('**/api/slow', route => {
    console.log('Intercepted slow request');
    setTimeout(() => {
      route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify({ message: 'Delayed response' })
      });
    }, 3000);
  });
  
  // Mock error response
  await page.route('**/api/error', route => {
    console.log('Intercepted error request');
    route.fulfill({
      status: 500,
      contentType: 'application/json',
      body: JSON.stringify({ error: 'Internal server error' })
    });
  });
  
  await page.goto('/api/index.html');
  
  // Test with mocked responses
  await page.click('button:has-text("Call Slow API")');
  await page.waitForTimeout(3500); // Wait for delayed response
  
  await page.click('button:has-text("Call Error API")');
  await expect(page.locator('.error-message')).toBeVisible();
});
```

### 3. Element Inspection

#### Element State Debugging
```javascript
test('element inspection', async ({ page }) => {
  await page.goto('/forms/index.html');
  
  const element = page.locator('.submit-button');
  
  // Get element properties
  const elementInfo = await element.evaluate(el => ({
    tagName: el.tagName,
    className: el.className,
    id: el.id,
    textContent: el.textContent,
    innerHTML: el.innerHTML,
    isVisible: el.offsetParent !== null,
    isEnabled: !el.disabled,
    isRequired: el.required,
    value: el.value
  }));
  
  console.log('Element info:', elementInfo);
  
  // Get computed styles
  const styles = await element.evaluate(el => {
    const computed = window.getComputedStyle(el);
    return {
      display: computed.display,
      visibility: computed.visibility,
      opacity: computed.opacity,
      position: computed.position,
      zIndex: computed.zIndex
    };
  });
  
  console.log('Computed styles:', styles);
  
  // Check element boundaries
  const boundingBox = await element.boundingBox();
  console.log('Bounding box:', boundingBox);
  
  // Check if element is in viewport
  const isInViewport = await element.evaluate(el => {
    const rect = el.getBoundingClientRect();
    return (
      rect.top >= 0 &&
      rect.left >= 0 &&
      rect.bottom <= window.innerHeight &&
      rect.right <= window.innerWidth
    );
  });
  
  console.log('Element in viewport:', isInViewport);
});
```

### 4. Performance Debugging

#### Performance Metrics
```javascript
test('performance debugging', async ({ page }) => {
  await page.goto('/performance-heavy.html');
  
  // Monitor performance metrics
  const metrics = await page.evaluate(() => {
    const navigation = performance.getEntriesByType('navigation')[0];
    return {
      domContentLoaded: navigation.domContentLoadedEventEnd - navigation.domContentLoadedEventStart,
      loadComplete: navigation.loadEventEnd - navigation.loadEventStart,
      firstPaint: performance.getEntriesByType('paint')[0]?.startTime,
      firstContentfulPaint: performance.getEntriesByType('paint')[1]?.startTime
    };
  });
  
  console.log('Performance metrics:', metrics);
  
  // Monitor memory usage
  const memoryInfo = await page.evaluate(() => {
    return performance.memory ? {
      usedJSHeapSize: performance.memory.usedJSHeapSize,
      totalJSHeapSize: performance.memory.totalJSHeapSize,
      jsHeapSizeLimit: performance.memory.jsHeapSizeLimit
    } : null;
  });
  
  console.log('Memory usage:', memoryInfo);
});
```

## 🛠️ Debugging Tools and Utilities

### 1. Custom Debug Utilities

#### Debug Helper Class
```javascript
// utils/DebugHelper.js
class DebugHelper {
  constructor(page) {
    this.page = page;
    this.logs = [];
  }

  async logPageState() {
    const state = await this.page.evaluate(() => ({
      url: window.location.href,
      title: document.title,
      readyState: document.readyState,
      timestamp: Date.now()
    }));
    
    this.logs.push({ type: 'page-state', data: state });
    console.log('Page state:', state);
  }

  async logElementInfo(selector) {
    const element = this.page.locator(selector);
    const count = await element.count();
    
    if (count === 0) {
      console.log(`No elements found for selector: ${selector}`);
      return;
    }
    
    const info = await element.first().evaluate(el => ({
      tagName: el.tagName,
      className: el.className,
      id: el.id,
      textContent: el.textContent?.substring(0, 100),
      isVisible: el.offsetParent !== null,
      boundingBox: el.getBoundingClientRect()
    }));
    
    this.logs.push({ type: 'element-info', selector, data: info });
    console.log(`Element info for ${selector}:`, info);
  }

  async logNetworkActivity() {
    const requests = [];
    const responses = [];
    
    this.page.on('request', request => {
      requests.push({
        url: request.url(),
        method: request.method(),
        timestamp: Date.now()
      });
    });
    
    this.page.on('response', response => {
      responses.push({
        url: response.url(),
        status: response.status(),
        timestamp: Date.now()
      });
    });
    
    return { requests, responses };
  }

  async takeDebugScreenshot(name) {
    const screenshotPath = `debug-screenshots/${name}-${Date.now()}.png`;
    await this.page.screenshot({ 
      path: screenshotPath, 
      fullPage: true 
    });
    console.log(`Debug screenshot saved: ${screenshotPath}`);
    return screenshotPath;
  }

  async captureConsoleLogs() {
    const messages = [];
    this.page.on('console', msg => {
      messages.push({
        type: msg.type(),
        text: msg.text(),
        location: msg.location(),
        timestamp: Date.now()
      });
    });
    return messages;
  }

  getLogs() {
    return this.logs;
  }

  clearLogs() {
    this.logs = [];
  }
}

module.exports = DebugHelper;
```

#### Using Debug Helper
```javascript
// tests/debug-example.spec.js
const DebugHelper = require('../utils/DebugHelper');

test('debug with helper', async ({ page }) => {
  const debug = new DebugHelper(page);
  
  await debug.logPageState();
  await debug.logElementInfo('.login-form');
  await debug.takeDebugScreenshot('before-login');
  
  const consoleLogs = await debug.captureConsoleLogs();
  
  await page.goto('/auth/login.html');
  await page.fill('#username', 'test@example.com');
  await page.fill('#password', 'password123');
  await page.click('button[type="submit"]');
  
  await debug.logPageState();
  await debug.takeDebugScreenshot('after-login');
  
  console.log('All debug logs:', debug.getLogs());
});
```

### 2. VS Code Integration

#### Launch Configuration
```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Playwright Test",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/node_modules/.bin/playwright",
      "args": ["test", "--debug", "${input:testFile}"],
      "console": "integratedTerminal",
      "env": {
        "PWDEBUG": "1"
      }
    }
  ],
  "inputs": [
    {
      "id": "testFile",
      "description": "Test file to debug",
      "default": "tests/**/*.spec.js",
      "type": "promptString"
    }
  ]
}
```

#### VS Code Extensions
- **Playwright Test** - Official Playwright extension
- **Debugger for Chrome** - Debug in Chrome browser
- **Auto Rename Tag** - HTML tag renaming
- **Path Intellisense** - File path suggestions

## 🚨 Common Debugging Scenarios and Solutions

### 1. Test Flakiness

#### Identifying Flaky Tests
```javascript
// Run test multiple times to identify flakiness
for (let i = 0; i < 10; i++) {
  try {
    await runTest();
    console.log(`Test ${i + 1}: PASSED`);
  } catch (error) {
    console.log(`Test ${i + 1}: FAILED - ${error.message}`);
  }
}
```

#### Solutions for Flaky Tests
- **Add explicit waits** instead of relying on auto-waiting
- **Use retry mechanisms** for network-dependent operations
- **Implement proper cleanup** between test runs
- **Use stable locators** that don't change frequently

### 2. Timeout Issues

#### Common Causes and Solutions
```javascript
// Increase timeout for specific operations
await element.click({ timeout: 30000 });

// Use wait instead of timeout
await page.waitForSelector('.dynamic-element', { state: 'visible' });

// Configure global timeouts
test.setTimeout(60000);
```

### 3. Selector Issues

#### Debugging Selectors
```javascript
// Test multiple selector strategies
const selectors = [
  '#element-id',
  '.element-class',
  '[data-testid="element"]',
  'text=Element Text',
  'xpath=//div[contains(text(), "Element")]'
];

for (const selector of selectors) {
  const count = await page.locator(selector).count();
  console.log(`${selector}: ${count} elements`);
}
```

## 📊 Debugging Checklist

### Before Debugging
- [ ] Check test logs for error messages
- [ ] Verify test data and environment
- [ ] Check network connectivity
- [ ] Verify browser compatibility

### During Debugging
- [ ] Use Playwright Inspector for step-by-step debugging
- [ ] Enable trace recording for detailed analysis
- [ ] Take screenshots at key points
- [ ] Monitor console for JavaScript errors
- [ ] Check network requests and responses

### After Debugging
- [ ] Document root cause and solution
- [ ] Add proper error handling
- [ ] Improve test reliability
- [ ] Update test documentation

## 🎯 Best Practices for Debugging

1. **Start simple** - Begin with basic debugging techniques
2. **Use tools wisely** - Leverage Playwright's built-in debugging tools
3. **Document findings** - Keep track of debug sessions and solutions
4. **Prevent flakiness** - Write stable, reliable tests
5. **Monitor performance** - Keep an eye on test execution times
6. **Collaborate** - Share debugging insights with team members
7. **Automate where possible** - Create reusable debug utilities
8. **Learn continuously** - Stay updated with new debugging features

## 📚 Additional Resources

- [Playwright Debugging Documentation](https://playwright.dev/docs/debug)
- [Trace Viewer Guide](https://playwright.dev/docs/trace-viewer)
- [Playwright VS Code Extension](https://marketplace.visualstudio.com/items?itemName=ms-playwright.playwright)
- [Community Debugging Tips](https://github.com/microsoft/playwright/discussions)

Mastering these debugging techniques will help you quickly identify and resolve issues in your Playwright test automation.
