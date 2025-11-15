# QA Training Site

A comprehensive web application designed for automation testing with Playwright. This project provides a complete testing playground with various web elements, interactions, and scenarios that automation engineers commonly encounter.

## 🚀 Features

### Testing Pages
- **Authentication** - Login forms, validation, session management
- **Forms** - Complex form scenarios with validation
- **Buttons** - Various button types and interactions
- **Tables** - Data tables with sorting, pagination, and search
- **API Testing** - REST API calls with mock responses
- **File Operations** - Upload and download functionality
- **Dynamic Content** - AJAX-loaded content and state changes
- **Drag & Drop** - Interactive drag and drop elements
- **iFrames** - Multi-frame testing scenarios
- **Shadow DOM** - Web components with Shadow DOM
- **Alerts** - JavaScript dialogs (alert, confirm, prompt)

### Key Capabilities
- **Element Locators** - ID, CSS selectors, XPath, text content
- **User Interactions** - Click, type, hover, drag & drop
- **Wait Strategies** - Implicit and explicit waits
- **Form Validation** - Required fields, email format, password strength
- **Dynamic Elements** - AJAX-loaded content, conditional rendering
- **File Handling** - Upload, download, file type validation
- **Dialog Handling** - Alerts, confirms, prompts, modals

## 🛠️ Technologies Used

- **HTML5** - Semantic markup
- **CSS3** - Modern styling with Grid and Flexbox
- **JavaScript (ES6+)** - Vanilla JavaScript, no external dependencies
- **Responsive Design** - Mobile-first approach

## 📁 Project Structure

```
SampleWeb/
├── index.html              # Main landing page
├── about.html              # Project information
├── alerts/                 # JavaScript dialogs
│   └── index.html
├── api/                    # API testing interface
│   ├── index.html
│   └── fakeapi.json
├── auth/                   # Authentication forms
│   ├── login.html
│   └── welcome.html
├── buttons/                # Various button types
│   └── index.html
├── dragdrop/               # Drag & drop functionality
│   └── index.html
├── download/               # File download testing
│   └── index.html
├── dynamic/                # Dynamic content loading
│   └── index.html
├── forms/                  # Complex form scenarios
│   └── index.html
├── iframe/                 # Multi-frame testing
│   └── iframe.html
├── shadowdom/              # Shadow DOM elements
│   └── index.html
├── tables/                 # Data tables with sorting
│   └── table.html
└── upload/                 # File upload testing
    └── index.html
```

## 🎯 Testing Scenarios

Each page is designed with specific testing scenarios in mind:

### Element Locators
- ID-based locators
- CSS selectors
- XPath expressions
- Text content matching
- Data attributes

### User Interactions
- Click events (single, double, right-click)
- Form input and validation
- Dropdown selection
- Checkbox and radio buttons
- Hover effects
- Drag and drop operations

### Advanced Features
- Shadow DOM traversal
- iFrame switching
- File upload/download
- JavaScript dialog handling
- AJAX content loading
- Session management

## 🚀 Getting Started

### Prerequisites
- Modern web browser (Chrome, Firefox, Safari, Edge)
- Playwright framework (for automation testing)
- Basic understanding of HTML/CSS/JavaScript

### Running the Project
1. Clone or download the project
2. Open `index.html` in your web browser
3. Navigate to different testing pages using the main navigation

### Automation Testing with Playwright

```javascript
const { test, expect } = require('@playwright/test');

test.describe('QA Training Site', () => {
  test('should load main page', async ({ page }) => {
    await page.goto('http://localhost:8000');
    await expect(page).toHaveTitle(/QA Training Site/);
  });

  test('should navigate to login page', async ({ page }) => {
    await page.goto('http://localhost:8000');
    await page.click('text=Authentication');
    await expect(page).toHaveURL(/.*login\.html/);
  });
});
```

## 📊 Project Statistics

- **15 Testing Pages**
- **60+ Test Scenarios**
- **100% Responsive Design**
- **0 External Dependencies**

## 🗺️ Roadmap

### Completed ✅
- Phase 1: Core Features (Authentication, Forms, Buttons, Alerts)
- Phase 2: Advanced Interactions (Tables, iFrames, Shadow DOM, Drag & Drop)
- Phase 3: File Operations (Upload, Download, API Testing)

### Planned 📋
- Phase 4: Performance Testing (Load testing, timing measurements)
- Phase 5: Accessibility Testing (ARIA labels, keyboard navigation)

## 📈 Version History

### v1.0 (2025-11)
- Initial release with 14 testing pages
- Core automation scenarios
- Responsive design implementation
- Zero external dependencies

### v1.1 (2025-11)
- Added download testing functionality
- Improved API testing capabilities
- Enhanced activity logging
- Bug fixes and optimizations

## 🧪 Testing Best Practices

### Page Object Model
```javascript
class LoginPage {
  constructor(page) {
    this.page = page;
    this.usernameInput = page.locator('#username');
    this.passwordInput = page.locator('#password');
    this.loginButton = page.locator('button[type="submit"]');
  }

  async login(username, password) {
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }
}
```

### Wait Strategies
```javascript
// Wait for element to be visible
await page.waitForSelector('#dynamic-element', { state: 'visible' });

// Wait for API response
await page.waitForResponse('**/api/data');

// Wait for navigation
await page.waitForNavigation();
```

## 🔧 Configuration

### Browser Support
- Chrome/Chromium
- Firefox
- Safari
- Microsoft Edge

### Viewport Testing
- Desktop: 1920x1080
- Tablet: 768x1024
- Mobile: 375x667

## 📚 Learning Resources

### Documentation
- [Playwright Documentation](https://playwright.dev/)
- [Best Practices Guide](./docs/best-practices.md)
- [Locator Strategies](./docs/locators.md)

### Tutorials
- [Getting Started with Playwright](./docs/getting-started.md)
- [Advanced Automation Patterns](./docs/advanced-patterns.md)
- [Debugging Test Failures](./docs/debugging.md)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Add your improvements
4. Submit a pull request

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

## 👨‍💻 Developer

**SQA Buddhika Rangana**
- Quality Assurance Engineer
- Automation Testing Specialist
- Web Development Enthusiast

## 📞 Support

For questions, issues, or contributions:
- Create an issue on GitHub
- Contact the development team
- Check the documentation

---

Built with ❤️ for Playwright Automation Testing
