# ROTA Test Automation Guide

**Last Updated:** 2025-12-28
**Owner:** Development Team

---

## Overview

ROTA uses **Vitest** for unit/integration testing. Tests focus on server-side security modules, stores, and orchestration logic.

---

## Quick Start

```bash
# Run all tests
npm test

# Run with coverage
npm test:coverage

# Watch mode (development)
npm test -- --watch

# Run specific test file
npm test -- tests/server/session-tokens.test.ts
```

---

## Test Structure

```
tests/
├── server/                    # Server module tests (highest priority)
│   ├── session-tokens.test.ts   # Security token lifecycle
│   ├── validation.test.ts       # Zod schema validation
│   └── rate-limiter.test.ts     # Rate limiting logic
├── stores/                    # Svelte store tests
│   ├── courses.test.ts          # Course data + localStorage
│   └── session-history.test.ts  # Session tracking + streaks
└── orchestrator/              # AI orchestration tests
    ├── event-bus.test.ts        # Event system
    └── aggregator.test.ts       # Decision engine
```

---

## Test Categories

### 1. Server Module Tests (Priority: Critical)

| Module | Tests | Purpose |
|--------|-------|---------|
| `session-tokens.ts` | 29 | Token creation, validation, expiration, single-use connection info |
| `validation.ts` | 39 | Zod schema validation for all API inputs |
| `rate-limiter.ts` | 17 | Rate limit enforcement, IP extraction |

**When to add tests:**
- New API endpoint → Add validation schema tests
- Security-related changes → Add session-token tests
- Rate limit rule changes → Add rate-limiter tests

### 2. Store Tests (Priority: High)

| Store | Tests | Purpose |
|-------|-------|---------|
| `courses.ts` | ~10 | CRUD operations, localStorage persistence |
| `session-history.ts` | ~12 | Session tracking, streak calculation |

**When to add tests:**
- New store method → Add corresponding test
- localStorage schema change → Update existing tests

### 3. Orchestrator Tests (Priority: Medium)

| Module | Tests | Purpose |
|--------|-------|---------|
| `event-bus.ts` | 16 | Event subscription, emission, wildcard handling |
| `aggregator.ts` | 6 | Decision logic for pedagogy/visual/content |

**When to add tests:**
- New event type → Add to event-bus tests
- New decision path → Add to aggregator tests

---

## Writing Tests

### Mocking Store Adapter

For session-tokens, rate-limiter, and other server modules that use `getStore()`:

```typescript
vi.mock('$lib/server/store-adapter', () => {
  const stores = new Map<string, { value: string; expiresAt: number }>();
  
  return {
    getStore: vi.fn(async () => ({
      get: async (key) => stores.get(key)?.value ?? null,
      set: async (key, value, ttl) => {
        stores.set(key, { value, expiresAt: ttl ? Date.now() + ttl * 1000 : 0 });
      },
      del: async (key) => stores.delete(key),
      keys: async (pattern) => {
        const prefix = pattern.replace('*', '');
        return Array.from(stores.keys()).filter(k => k.startsWith(prefix));
      }
    })),
    __resetStores: () => stores.clear()
  };
});
```

> [!NOTE]
> The actual `store-adapter.ts` automatically falls back to in-memory when `REDIS_URL` is not set.
> Tests always use the mocked in-memory adapter shown above.

### Redis Integration Tests (Future)

Redis-specific tests should be in a separate suite:

```
tests/
├── server/           # Unit tests (mocked store)
├── integration/      # Integration tests
│   └── redis/        # Redis-specific tests (requires REDIS_URL)
```

Run Redis tests only when REDIS_URL is available:
```bash
REDIS_URL=redis://localhost:6379 npm test -- --run tests/integration/redis/
```

---

### Test File Naming
```
tests/[category]/[module].test.ts
```

### Basic Test Structure
```typescript
import { describe, it, expect, beforeEach, vi } from 'vitest';

describe('Module Name', () => {
  beforeEach(() => {
    // Reset mocks
    vi.clearAllMocks();
  });

  describe('function/feature', () => {
    it('does expected behavior', () => {
      // Arrange
      const input = {};
      
      // Act
      const result = functionUnderTest(input);
      
      // Assert
      expect(result).toBe(expected);
    });
  });
});
```

### Mocking External Dependencies
```typescript
// Mock store adapter (for session-tokens, rate-limiter)
vi.mock('$lib/server/store-adapter', () => ({
  getStore: vi.fn(async () => ({
    get: async (key) => null,
    set: async (key, value, ttl) => {},
    del: async (key) => {},
    keys: async (pattern) => []
  }))
}));

// Mock localStorage (for stores)
const localStorageMock = {
  getItem: vi.fn(),
  setItem: vi.fn(),
  removeItem: vi.fn()
};
Object.defineProperty(global, 'localStorage', { value: localStorageMock });
```

---

## Coverage Goals

| Category | Target | Current |
|----------|--------|---------|
| Server modules | >80% | ~85% |
| Stores | >70% | ~75% |
| Orchestrator | >60% | ~65% |

Run `npm test:coverage` and check `coverage/index.html` for detailed report.

---

## CI/CD Integration (Future)

Tests can be integrated into GitHub Actions:

```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm test -- --run
```

---

## Troubleshooting

### Tests hang or timeout
- Check for unresolved promises
- Ensure mocks return immediately
- Use `--run` flag for single execution

### Mock not working
- Verify mock path matches import path
- Use `vi.resetModules()` in beforeEach
- Check mock is defined before imports

### localStorage tests fail
- Ensure `global.localStorage` is mocked before store import
- Reset mock stores in beforeEach
