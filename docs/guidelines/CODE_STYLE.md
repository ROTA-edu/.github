# Code Style Guidelines

> **Philosophy:** Strict boundaries make for happy teams.

---

## 1. The "Zero Any" Policy

*   **Rule:** The use of `any` is strictly forbidden in production code.
*   **Reason:** `any` defeats the purpose of TypeScript and breaks the contract between `rota-frontend` and `@rota/ai`.
*   **Exception:** In test mocks, `any` is permitted but discouraged.

---

## 2. Package Boundaries (The Wall)

In a Polyrepo system, you cannot simply import a file from another folder. You must respect the Public API.

### ❌ Illegal Import
```typescript
// Importing from deep inside a package's source
import { internalHelper } from '@rota/ai/src/internal/helper';
```

### ✅ Legal Import
```typescript
// Importing from the package's public export
import { helper } from '@rota/ai';
```

---

## 3. Naming Conventions

*   **Packages:** `@rota/[name]` (e.g., `@rota/ai`, `@rota/canvas`).
*   **Components:** PascalCase (`RotaCanvas.svelte`).
*   **Stores:** camelCase (`sessionStore`).
*   **Events:** `verbNoun` (`onMessage`, `onConnect`).

---

## 4. Git Workflow

### Commit Messages
We follow **Conventional Commits**:
*   `feat: add new voice tool`
*   `fix: resolve websocket timeout`
*   `docs: update architecture diagram`
*   `chore: bump version to 1.2.0`

### Branches
*   `main`: Production-ready code.
*   `develop`: Integration branch.
*   `feature/xyz`: Active development.

### Private Repos
*   Commits in `rota-ai-core` must not leak into `rota-frontend` commit history (they are separate Git histories).