# Browser Automation QA Skill

**Version**: 1.0.0
**Category**: Testing, Automation
**Author**: ClawFarm QA Team

---

## Description

This skill provides browser automation capabilities for QA testing of the ClawFarm web application. It uses Playwright to automate browser interactions for UI testing, end-to-end workflows, and visual regression testing.

## Capabilities

- Navigate to web pages
- Take screenshots
- Fill forms
- Click elements
- Wait for elements
- Extract page content
- Run JavaScript in browser context
- Handle dialogs and alerts
- Upload files
- Download files
- Execute multi-step workflows

## Installation

```bash
# Install Playwright
npm install -D playwright

# Install browsers
npx playwright install chromium
```

## Usage

### Basic Navigation

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("https://clawfarm.ca")
    browser.close()
```

### Take Screenshot

```python
page.screenshot(path="screenshot.png")
```

### Fill Form

```python
page.fill("input[name='email']", "test@example.com")
page.fill("input[name='password']", "password123")
page.click("button[type='submit']")
```

### Wait for Element

```python
page.wait_for_selector(".dashboard", timeout=5000)
```

### Extract Text

```python
text = page.locator("h1").inner_text()
```

## Actions Reference

| Action | Parameters | Description |
|--------|------------|-------------|
| `navigate` | url | Navigate to URL |
| `screenshot` | path, fullPage | Take screenshot |
| `click` | selector, button, modifiers | Click element |
| `fill` | selector, text | Fill input field |
| `type` | selector, text, slowly | Type text character by character |
| `wait` | selector, timeout | Wait for element |
| `select` | selector, value | Select dropdown option |
| `hover` | selector | Hover over element |
| `evaluate` | script | Execute JavaScript |
| `pdf` | path | Save page as PDF |

## Workflows

### Login Workflow

```python
def login(page, email, password):
    page.goto("https://clawfarm.ca/login")
    page.fill("input[name='email']", email)
    page.fill("input[name='password']", password)
    page.click("button[type='submit']")
    page.wait_for_url("**/dashboard")
```

### Create Agency Workflow

```python
def create_agency(page, name, plan):
    page.goto("https://clawfarm.ca/agencies")
    page.click("text=Create Agency")
    page.fill("input[name='name']", name)
    page.select_option("select[name='plan']", plan)
    page.click("button[type='submit']")
    page.wait_for_selector(".agency-item")
```

### Health Check Workflow

```python
def health_check():
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()

        # Check main page
        page.goto("https://clawfarm.ca")
        assert page.title() != ""

        # Check API health
        response = page.request.get("https://api.clawfarm.ca/v1/health")
        assert response.status == 200

        browser.close()
```

## Error Handling

```python
try:
    page.wait_for_selector(".element", timeout=5000)
except Exception as e:
    page.screenshot(path="error.png")
    raise e
```

## Tips

1. Use `headless=False` during development to see what's happening
2. Use `page.pause()` for debugging
3. Use `page.locator()` instead of `page.$()` for better performance
4. Use data-testid attributes for more reliable selectors
5. Set appropriate timeouts for each action

## Examples

See `qa_browser_automation.md` for complete test cases using this skill.
