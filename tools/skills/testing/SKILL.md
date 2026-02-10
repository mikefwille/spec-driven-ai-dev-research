---
name: testing
description: Write tests for the workflow builder application. Use this skill when creating E2E tests, unit tests, or integration tests. Covers Playwright patterns, test data setup, and mocking strategies.
---

This skill guides creation of tests for the workflow builder. Focus on E2E testing with Playwright for user flows and unit tests for utility functions.

The user provides testing requirements: a feature to test, edge cases to cover, or test infrastructure to set up.

## Testing Stack

- **E2E**: Playwright
- **Unit**: Vitest (if needed)
- **Location**: `tests/` directory

## Directory Structure

```
tests/
├── e2e/
│   ├── workflows/
│   │   ├── create.spec.ts
│   │   ├── edit.spec.ts
│   │   └── execute.spec.ts
│   ├── integrations/
│   │   └── connect.spec.ts
│   └── auth/
│       └── login.spec.ts
├── integration/
│   └── api/
│       └── workflows.test.ts
└── fixtures/
    └── test-data.ts
```

## Playwright E2E Test Pattern

### Basic Test Structure

```typescript
// tests/e2e/workflows/create.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Workflow Creation', () => {
  test.beforeEach(async ({ page }) => {
    // Set up authentication before page load
    await page.addInitScript(() => {
      // Set auth token in localStorage before navigation
      localStorage.setItem('auth-token', 'test-token');
    });

    await page.goto('/workflows');
  });

  test('should create a new workflow', async ({ page }) => {
    // Click create button
    await page.getByRole('button', { name: 'Create Workflow' }).click();

    // Fill in workflow name
    await page.getByLabel('Workflow Name').fill('My Test Workflow');

    // Submit
    await page.getByRole('button', { name: 'Create' }).click();

    // Verify redirect to editor
    await expect(page).toHaveURL(/\/workflows\/[\w-]+/);

    // Verify workflow name is shown
    await expect(page.getByText('My Test Workflow')).toBeVisible();
  });

  test('should show validation error for empty name', async ({ page }) => {
    await page.getByRole('button', { name: 'Create Workflow' }).click();

    // Try to submit without name
    await page.getByRole('button', { name: 'Create' }).click();

    // Check for error message
    await expect(page.getByText('Name is required')).toBeVisible();
  });
});
```

### Authentication Setup

```typescript
// tests/e2e/fixtures/auth.ts
import { test as base, expect } from '@playwright/test';

// Extend base test with authenticated context
export const test = base.extend<{ authenticatedPage: Page }>({
  authenticatedPage: async ({ page }, use) => {
    // Set up auth before navigation
    await page.addInitScript(() => {
      // Simulate logged-in user
      const mockSession = {
        user: {
          id: 'test-user-id',
          email: 'test@example.com',
          name: 'Test User',
        },
        expires: new Date(Date.now() + 86400000).toISOString(),
      };
      localStorage.setItem('session', JSON.stringify(mockSession));
    });

    await use(page);
  },
});

export { expect };
```

### Testing Workflow Canvas

```typescript
// tests/e2e/workflows/canvas.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Workflow Canvas', () => {
  test.beforeEach(async ({ page }) => {
    // Navigate to existing workflow
    await page.goto('/workflows/test-workflow-id');
  });

  test('should add a new action node', async ({ page }) => {
    // Find the add node button
    const addButton = page.getByTestId('add-node-button');
    await addButton.click();

    // Select action type
    await page.getByText('Send Slack Message').click();

    // Verify node appears on canvas
    await expect(page.getByTestId('action-node-')).toBeVisible();
  });

  test('should connect two nodes', async ({ page }) => {
    // Get source handle
    const sourceHandle = page.locator('[data-handleid="source"]').first();

    // Get target handle
    const targetHandle = page.locator('[data-handleid="target"]').last();

    // Drag to connect
    await sourceHandle.dragTo(targetHandle);

    // Verify edge exists
    await expect(page.locator('.react-flow__edge')).toBeVisible();
  });

  test('should configure node settings', async ({ page }) => {
    // Click on node to select
    await page.getByTestId('action-node-node-1').click();

    // Config panel should open
    await expect(page.getByText('Node Configuration')).toBeVisible();

    // Fill in config
    await page.getByLabel('Channel').fill('#general');
    await page.getByLabel('Message').fill('Hello from test!');

    // Verify auto-save (check for saved indicator)
    await expect(page.getByText('Saved')).toBeVisible({ timeout: 5000 });
  });
});
```

### API Mocking

```typescript
// tests/e2e/workflows/with-mocks.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Workflow with Mocked API', () => {
  test('should handle API errors gracefully', async ({ page }) => {
    // Mock API to return error
    await page.route('**/api/workflows', (route) => {
      route.fulfill({
        status: 500,
        contentType: 'application/json',
        body: JSON.stringify({ error: 'Internal server error' }),
      });
    });

    await page.goto('/workflows');

    // Should show error state
    await expect(page.getByText('Failed to load workflows')).toBeVisible();
  });

  test('should display workflow list from API', async ({ page }) => {
    // Mock API response
    await page.route('**/api/workflows', (route) => {
      route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify([
          {
            id: 'wf-1',
            name: 'Test Workflow 1',
            createdAt: '2024-01-01T00:00:00Z',
            updatedAt: '2024-01-01T00:00:00Z',
          },
          {
            id: 'wf-2',
            name: 'Test Workflow 2',
            createdAt: '2024-01-02T00:00:00Z',
            updatedAt: '2024-01-02T00:00:00Z',
          },
        ]),
      });
    });

    await page.goto('/workflows');

    // Verify workflows displayed
    await expect(page.getByText('Test Workflow 1')).toBeVisible();
    await expect(page.getByText('Test Workflow 2')).toBeVisible();
  });
});
```

### Integration Selection Test

```typescript
// tests/e2e/integrations/select.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Integration Selection', () => {
  test('should show warning for unconfigured integration', async ({ page }) => {
    await page.goto('/workflows/new');

    // Add node that requires integration
    await page.getByTestId('add-action').click();
    await page.getByText('Send Slack Message').click();

    // Should show warning badge
    await expect(
      page.locator('[data-testid="action-node-"] .bg-orange-500')
    ).toBeVisible();
  });

  test('should allow selecting configured integration', async ({ page }) => {
    // Mock integrations API
    await page.route('**/api/integrations*', (route) => {
      route.fulfill({
        status: 200,
        body: JSON.stringify([
          {
            id: 'int-1',
            name: 'My Slack',
            type: 'slack',
            createdAt: '2024-01-01T00:00:00Z',
            updatedAt: '2024-01-01T00:00:00Z',
          },
        ]),
      });
    });

    await page.goto('/workflows/new');

    // Add Slack node
    await page.getByTestId('add-action').click();
    await page.getByText('Send Slack Message').click();

    // Click node to configure
    await page.getByTestId('action-node-').click();

    // Select integration
    await page.getByLabel('Integration').selectOption('int-1');

    // Warning should be gone
    await expect(
      page.locator('[data-testid="action-node-"] .bg-orange-500')
    ).not.toBeVisible();
  });
});
```

## Test Utilities

### Page Object Model

```typescript
// tests/e2e/pages/workflow-editor.ts
import type { Page, Locator } from '@playwright/test';

export class WorkflowEditorPage {
  readonly page: Page;
  readonly canvas: Locator;
  readonly toolbar: Locator;
  readonly configPanel: Locator;

  constructor(page: Page) {
    this.page = page;
    this.canvas = page.locator('.react-flow');
    this.toolbar = page.getByTestId('workflow-toolbar');
    this.configPanel = page.getByTestId('config-panel');
  }

  async goto(workflowId: string) {
    await this.page.goto(`/workflows/${workflowId}`);
  }

  async addNode(nodeType: string) {
    await this.toolbar.getByRole('button', { name: 'Add' }).click();
    await this.page.getByText(nodeType).click();
  }

  async selectNode(nodeId: string) {
    await this.page.getByTestId(`action-node-${nodeId}`).click();
  }

  async setNodeConfig(field: string, value: string) {
    await this.configPanel.getByLabel(field).fill(value);
  }

  async saveWorkflow() {
    await this.toolbar.getByRole('button', { name: 'Save' }).click();
  }
}

// Usage in test
test('workflow editor page object', async ({ page }) => {
  const editor = new WorkflowEditorPage(page);
  await editor.goto('workflow-id');
  await editor.addNode('Send Slack Message');
  await editor.selectNode('node-1');
  await editor.setNodeConfig('Channel', '#general');
});
```

### Test Data Factories

```typescript
// tests/fixtures/factories.ts
export function createWorkflow(overrides = {}) {
  return {
    id: `wf-${Date.now()}`,
    name: 'Test Workflow',
    description: 'A test workflow',
    nodes: [],
    edges: [],
    visibility: 'private',
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
    ...overrides,
  };
}

export function createIntegration(type: string, overrides = {}) {
  return {
    id: `int-${Date.now()}`,
    name: `Test ${type}`,
    type,
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
    ...overrides,
  };
}

export function createNode(type: string, overrides = {}) {
  return {
    id: `node-${Date.now()}`,
    type: 'action',
    position: { x: 100, y: 100 },
    data: {
      label: type,
      config: {
        actionType: type,
      },
    },
    ...overrides,
  };
}
```

## Playwright Config

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',

  use: {
    baseURL: 'http://localhost:3002',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],

  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3002',
    reuseExistingServer: !process.env.CI,
  },
});
```

## Running Tests

```bash
# Run all E2E tests
npx playwright test

# Run in headed mode (see browser)
npx playwright test --headed

# Run specific test file
npx playwright test tests/e2e/workflows/create.spec.ts

# Run with UI mode
npx playwright test --ui

# Debug a test
npx playwright test --debug

# Show report
npx playwright show-report
```

## Test Checklist

When writing tests:

1. **Isolation**: Each test should be independent
2. **Authentication**: Set up auth before page navigation
3. **Mocking**: Mock external APIs for reliability
4. **Selectors**: Prefer `getByRole`, `getByLabel`, `getByTestId`
5. **Assertions**: Use specific assertions with timeouts
6. **Cleanup**: Tests should clean up after themselves
7. **Data**: Use factories for test data

## Anti-Patterns to Avoid

- Hard-coded waits (`page.waitForTimeout`)
- Flaky selectors (CSS classes that change)
- Tests that depend on other tests
- Not mocking external services
- Missing error case coverage
- Overly complex test setups

## Code Structure & Organization

### DRY Principle - Zero Tolerance for Duplication

**Test utilities prevent copy-paste nightmares.**

```typescript
// WRONG - duplicated mock setup in every test
await page.route('**/api/workflows', (route) => {
  route.fulfill({
    status: 200,
    body: JSON.stringify([{ id: 'wf-1', name: 'Test' }]),
  });
});

// RIGHT - extracted mock helper
// tests/fixtures/mocks.ts
export const mockWorkflows = (page: Page, workflows: Workflow[]) =>
  page.route('**/api/workflows', (route) =>
    route.fulfill({ status: 200, body: JSON.stringify(workflows) })
  );

// Usage
await mockWorkflows(page, [createWorkflow({ name: 'Test' })]);
```

### Smart Code Over Long Code

```typescript
// WRONG - verbose assertions
await expect(page.getByText('Workflow 1')).toBeVisible();
await expect(page.getByText('Workflow 2')).toBeVisible();
await expect(page.getByText('Workflow 3')).toBeVisible();

// RIGHT - concise loop
for (const name of ['Workflow 1', 'Workflow 2', 'Workflow 3']) {
  await expect(page.getByText(name)).toBeVisible();
}

// OR - parallel assertions
await Promise.all(
  ['Workflow 1', 'Workflow 2', 'Workflow 3'].map((name) =>
    expect(page.getByText(name)).toBeVisible()
  )
);
```

**File Size Limits**:

- Maximum 500 lines per test file
- Split large test suites by feature or scenario

**Test Structure**:

```
tests/
├── e2e/
│   ├── workflows/
│   │   ├── create.spec.ts      # Creation tests (<200 lines)
│   │   ├── edit.spec.ts        # Editing tests
│   │   ├── execute.spec.ts     # Execution tests
│   │   └── canvas.spec.ts      # Canvas interaction tests
│   └── fixtures/
│       ├── auth.ts             # Auth helpers
│       └── mocks.ts            # API mock helpers
├── pages/                      # Page objects
│   ├── workflow-editor.ts
│   └── dashboard.ts
└── fixtures/
    └── factories.ts            # Test data factories
```

**Extraction Triggers**:

- Test file exceeds 300 lines → Split by scenario
- Repeated setup code → Extract to fixture
- Repeated assertions → Extract to helper function
- Complex page interactions → Extract to page object

## Output Standards

**No Emojis**: Never use emojis in test names, comments, or assertions.

**Console Log Discipline**:

- Never use `console.log` in test files
- Use Playwright's built-in tracing and screenshots for debugging
- Remove any debug statements before committing
- Use `test.only` and `--debug` flag for debugging, not logs
