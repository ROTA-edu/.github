# CI/CD Guidelines

> **Purpose:** Automated testing, building, and deployment across all ROTA repos
> **Last Updated:** 2026-01-02

---

## Overview

ROTA uses **GitHub Actions** for all CI/CD operations. We use **reusable workflows** to avoid duplication and ensure consistency across repos.

---

## 1. Workflow Types

### Continuous Integration (CI)
**Triggers:** Push to main/develop, Pull Request
**Purpose:** Validate code quality before merge

```yaml
# Used by: ALL repos
.github/workflows/reusable-ci.yml
├─ Lint (ESLint + Prettier)
├─ Type Check (TypeScript)
├─ Unit Tests (Vitest)
├─ Integration Tests
├─ Coverage Check (80% threshold)
└─ Build Verification
```

### Publish (Packages)
**Triggers:** Version tag push (`v*.*.*`)
**Purpose:** Publish packages to GitHub Packages

```yaml
# Used by: course-engine, shared-utils, canvas-engine, ai-engine
.github/workflows/reusable-publish.yml
├─ Run full CI pipeline
├─ Build package
├─ Publish to GitHub Packages
├─ Generate changelog (AI-powered)
└─ Create GitHub Release
```

### Deploy (Apps)
**Triggers:** Push to main, manual dispatch
**Purpose:** Deploy apps to production

```yaml
# Used by: atlas-app, platform
.github/workflows/reusable-deploy.yml
├─ Run full CI pipeline
├─ Build application
├─ Deploy to Vercel/Railway
└─ Run smoke tests
```

---

## 2. Using Reusable Workflows

### Example: Package CI (course-engine)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  ci:
    uses: ROTA-edu/.github/.github/workflows/reusable-ci.yml@main
    with:
      node-version: '18'
      coverage-threshold: 80
      run-integration-tests: true
```

### Example: Package Publish (course-engine)

```yaml
# .github/workflows/publish.yml
name: Publish to GitHub Packages

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  publish:
    uses: ROTA-edu/.github/.github/workflows/reusable-publish.yml@main
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

---

## 3. Branch Protection Rules

### main/master Branch
**Required for:** ALL repos

```yaml
Protection Rules:
├─ Require pull request before merging
├─ Require status checks to pass
│  ├─ CI / Lint
│  ├─ CI / Type Check
│  ├─ CI / Tests
│  └─ CI / Build
├─ Require conversation resolution
├─ Require branches to be up to date
└─ Do NOT allow bypassing (even for admins)
```

**Setup:**
```bash
# Via GitHub CLI
gh api repos/ROTA-edu/rota-course-engine/branches/main/protection \
  --method PUT \
  --field required_status_checks[strict]=true \
  --field required_status_checks[contexts][]=ci \
  --field required_pull_request_reviews[required_approving_review_count]=1 \
  --field enforce_admins=true \
  --field allow_force_pushes=false
```

**Or via GitHub UI:**
1. Repo Settings → Branches → Add rule
2. Branch name pattern: `main`
3. ✅ Require pull request before merging
4. ✅ Require status checks to pass before merging
5. ✅ Require branches to be up to date before merging
6. ✅ Include administrators
7. Save

---

## 4. Secrets Management

### Repository Secrets
**Path:** Settings → Secrets and variables → Actions

**Required for ALL repos:**
- `ANTHROPIC_API_KEY` - For AI-powered features (code review, changelog)

**Required for PACKAGES:**
- `GITHUB_TOKEN` - Auto-provided by GitHub Actions

**Required for APPS:**
- `VERCEL_TOKEN` - For deployment
- `DATABASE_URL` - For migrations (if applicable)

**Setup:**
```bash
# Via GitHub CLI
gh secret set ANTHROPIC_API_KEY -b"sk-ant-..." --repo ROTA-edu/rota-course-engine
```

---

## 5. AI-Powered Automation

### AI Code Review
**Triggers:** Pull Request opened/synchronized
**What it does:**
- Analyzes PR diff
- Checks for security issues (XSS, SQL injection, prompt injection)
- Identifies performance problems
- Suggests improvements
- Leaves review comments on PR

**Example:**
```yaml
# .github/workflows/ai-review.yml
name: AI Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  ai-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: AI Code Review
        uses: ROTA-edu/.github/actions/ai-code-review@v1
        with:
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
          focus-areas: security,performance,testing
```

### AI Changelog Generator
**Triggers:** Version tag push
**What it does:**
- Analyzes commits since last tag
- Categorizes changes (Features, Bug Fixes, Breaking Changes)
- Generates CHANGELOG.md
- Creates GitHub Release with notes

**Example:**
```yaml
# Part of reusable-publish.yml
- name: Generate Changelog
  uses: ROTA-edu/.github/actions/ai-changelog@v1
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    previous-tag: ${{ github.event.before }}
    current-tag: ${{ github.ref }}
```

### AI Test Suggestions
**Triggers:** Pull Request with low coverage
**What it does:**
- Reads coverage report
- Identifies untested code
- Suggests test cases
- Generates example test code
- Comments on PR

---

## 6. Local Development Workflow

### Before Creating PR

```bash
# 1. Create feature branch
git checkout -b feature/your-feature

# 2. Make changes
# ... code ...

# 3. Run local checks (same as CI)
pnpm lint              # ESLint + Prettier
pnpm type-check        # TypeScript
pnpm test              # Vitest
pnpm test:coverage     # Coverage check
pnpm build             # Build verification

# 4. Fix issues if any
pnpm lint:fix          # Auto-fix linting issues

# 5. Commit (follow conventions)
git add .
git commit -m "feat: add new feature"

# 6. Push
git push -u origin feature/your-feature

# 7. Create PR
gh pr create --fill
```

### PR Workflow

```
1. Developer creates PR
   ↓
2. GitHub Actions runs CI
   ├─ Lint
   ├─ Type Check
   ├─ Tests
   ├─ Coverage
   └─ Build
   ↓
3. AI Code Review (optional)
   └─ Leaves review comments
   ↓
4. Developer addresses feedback
   └─ Push fixes → CI runs again
   ↓
5. Reviewer approves PR
   ↓
6. Merge (squash & merge preferred)
   ↓
7. Delete feature branch
```

---

## 7. Deployment Workflow

### Packages (course-engine, shared-utils, etc.)

```bash
# 1. Ensure all changes merged to main
git checkout main
git pull origin main

# 2. Update version
npm version patch  # 1.0.0 → 1.0.1
# or
npm version minor  # 1.0.0 → 1.1.0
# or
npm version major  # 1.0.0 → 2.0.0

# This creates:
# - Updated package.json
# - Git commit
# - Git tag

# 3. Push with tags
git push --follow-tags

# 4. GitHub Actions automatically:
# - Runs CI
# - Builds package
# - Publishes to GitHub Packages
# - Generates changelog
# - Creates GitHub Release

# 5. Verify
gh release view v1.0.1
```

### Apps (atlas-app, platform)

```bash
# 1. Merge PR to main
# GitHub Actions automatically:
# - Runs CI
# - Builds app
# - Deploys to Vercel/Railway
# - Runs smoke tests

# 2. Monitor deployment
gh run watch

# 3. Verify production
# Check deployment URL
```

---

## 8. Troubleshooting

### CI Failing: "Lockfile out of sync"

**Problem:** `pnpm install --frozen-lockfile` fails

**Solution:**
```bash
# Update lockfile
pnpm install

# Commit updated lockfile
git add pnpm-lock.yaml
git commit -m "chore: update lockfile"
git push
```

### CI Failing: "Tests not found"

**Problem:** Test files not detected

**Solution:**
```bash
# Check test pattern in package.json
{
  "scripts": {
    "test": "vitest"
  }
}

# Check Vitest config (vitest.config.ts)
export default defineConfig({
  test: {
    include: ['tests/**/*.test.ts', 'src/**/*.test.ts']
  }
});
```

### Publish Failing: "Package marked as private"

**Problem:** `npm error This package has been marked as private`

**Solution:**
```json
// package.json - Remove or set to false
{
  "private": false,  // or remove this line
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  }
}
```

### Deploy Failing: "Environment variable not found"

**Problem:** Missing secrets

**Solution:**
```bash
# Add secret via GitHub CLI
gh secret set DATABASE_URL -b"postgres://..." --repo ROTA-edu/rota-atlas-app

# Or via GitHub UI: Settings → Secrets → New repository secret
```

---

## 9. Best Practices

### ✅ DO

- Run `pnpm lint && pnpm type-check && pnpm test` before creating PR
- Write meaningful commit messages (follow [Conventional Commits](https://www.conventionalcommits.org/))
- Keep PRs small and focused (1 feature = 1 PR)
- Add tests for new features
- Update docs when changing behavior
- Use semantic versioning (semver.org)

### ❌ DON'T

- Don't bypass CI checks (emergency only)
- Don't commit secrets to code
- Don't merge without approval
- Don't push directly to main (use PRs)
- Don't ignore linting/type errors
- Don't skip writing tests

---

## 10. GitHub Copilot Integration

### Enabling Copilot

1. Install Copilot extension (VS Code)
2. Sign in with GitHub account
3. Ensure you have Copilot subscription

### Copilot Best Practices

**✅ Use Copilot for:**
- Boilerplate code generation
- Test case generation
- JSDoc comments
- Repetitive patterns

**❌ Don't rely on Copilot for:**
- Security-critical code (always review!)
- Complex algorithms (verify logic)
- AI prompts (these are IP, write manually)

### Copilot + CI Integration

Copilot can help write tests, but CI validates them:

```typescript
// 1. Write function
export function validateEmail(email: string): boolean {
  // implementation
}

// 2. Ask Copilot: "Write tests for validateEmail"
// Copilot suggests:
describe('validateEmail', () => {
  it('should accept valid email', () => {
    expect(validateEmail('test@example.com')).toBe(true);
  });

  it('should reject invalid email', () => {
    expect(validateEmail('invalid')).toBe(false);
  });
});

// 3. CI runs tests automatically
// 4. Coverage report shows if edge cases missed
```

---

## 11. Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Reusable Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Semantic Versioning](https://semver.org/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [GitHub Copilot](https://github.com/features/copilot)

---

**Questions?** Post in #engineering-help or create an issue in [.github repo](https://github.com/ROTA-edu/.github/issues)
