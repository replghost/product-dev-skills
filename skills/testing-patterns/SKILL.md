---
name: testing-patterns
description: Testing patterns and best practices for this project. Auto-load when writing tests, creating mocks, or following TDD workflow.
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

# Testing Patterns

## Framework: Vitest + React Testing Library

## Test File Location
- Unit tests: `tests/unit/[module].test.ts`
- Integration: `tests/integration/[feature].test.ts`
- E2E: `tests/e2e/[flow].spec.ts`

## Running Tests
```bash
# All tests
npm test

# Single file
npm test -- tests/unit/auth.test.ts

# Watch mode
npm test -- --watch

# Coverage
npm test -- --coverage
```

## Patterns

### Mock Factory
```typescript
// tests/factories/user.ts
export const createMockUser = (overrides = {}): User => ({
  id: 'user-123',
  email: 'test@example.com',
  name: 'Test User',
  ...overrides,
});
```

### API Mocking
```typescript
import { vi } from 'vitest';
import * as api from '@/api';

vi.mock('@/api');
const mockApi = vi.mocked(api);

beforeEach(() => {
  mockApi.fetchUser.mockResolvedValue(createMockUser());
});
```

### Component Testing
```typescript
import { render, screen, userEvent } from '@testing-library/react';

it('submits form with valid data', async () => {
  const onSubmit = vi.fn();
  render(<Form onSubmit={onSubmit} />);
  
  await userEvent.type(screen.getByLabelText('Email'), 'test@example.com');
  await userEvent.click(screen.getByRole('button', { name: 'Submit' }));
  
  expect(onSubmit).toHaveBeenCalledWith({ email: 'test@example.com' });
});
```

### Async Testing
```typescript
it('loads data on mount', async () => {
  render(<DataLoader />);
  
  expect(screen.getByText('Loading...')).toBeInTheDocument();
  
  await waitFor(() => {
    expect(screen.getByText('Data loaded')).toBeInTheDocument();
  });
});
```

## Anti-Patterns to Avoid
- ❌ Testing implementation details
- ❌ Multiple assertions testing different behaviors
- ❌ Mocking internal modules
- ❌ Hardcoded timeouts instead of waitFor
