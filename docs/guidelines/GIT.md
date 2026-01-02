# Git & Workflow Guidelines

> **Scope:** Managing the Polyrepo Ecosystem
> **Tooling:** Git, PNPM Workspaces, Conventional Commits

---

## 1. Multi-Repo Management

We use a "Virtual Monorepo" strategy for local development, but distinct Git histories for each repo.

### Setup
Your local directory should look like this:
```text
~/projects/rota-workspace/
├── rota-frontend/       (.git)
├── rota-ai-core/        (.git)
├── rota-canvas-engine/  (.git)
└── pnpm-workspace.yaml  (Ignored)
```

### Dependency Rules
1.  **Never commit `node_modules`**.
2.  **Always use `pnpm`**. We rely on its symlinking behavior.
3.  **Lockfiles:** Each repo has its own `pnpm-lock.yaml`.

---

## 2. Branching Strategy (GitFlow Lite)

Each repository follows its own versioning.

*   **`main`**: Production-ready code. Matches the latest NPM tag (e.g., `v1.2.0`).
*   **`develop`**: Integration branch. Feature branches merge here.
*   **`feature/XYZ`**: Active development.

**Rule:** When working on a cross-cutting feature (e.g., "Add Zoom Tool"):
1.  Create `feature/zoom-tool` in `rota-canvas-engine`.
2.  Create `feature/zoom-tool` in `rota-ai-core`.
3.  Create `feature/zoom-tool` in `rota-frontend`.
4.  Merge bottom-up (Canvas -> AI -> Frontend).

---

## 3. Commit Messages

We enforce **Conventional Commits** to automate versioning.

*   `feat: ...` -> Triggers Minor version bump (v1.1.0 -> v1.2.0)
*   `fix: ...` -> Triggers Patch version bump (v1.2.0 -> v1.2.1)
*   `BREAKING CHANGE: ...` -> Triggers Major version bump (v1.0 -> v2.0)

**Example:**
```bash
git commit -m "feat(layout): add radial layout support to ELK engine"
```
