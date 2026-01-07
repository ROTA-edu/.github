# ROTA Organization Infrastructure

This repository contains the central documentation, reusable workflows, and shared actions for all ROTA repositories.

## Documentation

All organizational documentation lives in [`docs/`](./docs/):

- **[Getting Started](./docs/README.md)** - Overview and quick links
- **[Architecture](./docs/architecture/POLYREPO_ARCHITECTURE.md)** - System design and repo structure
- **[Guidelines](./docs/guidelines/)** - Coding standards, testing, CI/CD, git workflow

## Reusable Workflows

Located in [`.github/workflows/`](./.github/workflows/):

### [`reusable-ci.yml`](./.github/workflows/reusable-ci.yml)
Runs linting, type-checking, tests, and build for any repo.

**Usage:**
```yaml
jobs:
  ci:
    uses: ROTA-edu/.github/.github/workflows/reusable-ci.yml@main
    with:
      node-version: '18'
      coverage-threshold: 80
```

### [`reusable-publish.yml`](./.github/workflows/reusable-publish.yml)
Publishes packages to GitHub Packages with AI-generated changelog.

**Usage:**
```yaml
jobs:
  publish:
    uses: ROTA-edu/.github/.github/workflows/reusable-publish.yml@main
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

## Custom AI Agents

Located in [`actions/`](./actions/):

### [`wiki-issue-agent`](./actions/wiki-issue-agent/) - DeepSeek V3.2
Dual-purpose agent for wiki maintenance and issue triage.

**Wiki Mode (Weekly):**
```yaml
- uses: ROTA-edu/.github/actions/wiki-issue-agent@v1
  with:
    openrouter-api-key: ${{ secrets.OPENROUTER_API_KEY }}
    mode: 'wiki'
```

**Issue Mode (Auto on issue open):**
```yaml
- uses: ROTA-edu/.github/actions/wiki-issue-agent@v1
  with:
    openrouter-api-key: ${{ secrets.OPENROUTER_API_KEY }}
    mode: 'issue'
    issue-number: ${{ github.event.issue.number }}
```

### [`bug-finder-agent`](./actions/bug-finder-agent/) - Grok Code Fast 1
Daily security and bug scanning.

**Daily Scan:**
```yaml
- uses: ROTA-edu/.github/actions/bug-finder-agent@v1
  with:
    openrouter-api-key: ${{ secrets.OPENROUTER_API_KEY }}
    scan-mode: 'daily'
```

**PR Scan:**
```yaml
- uses: ROTA-edu/.github/actions/bug-finder-agent@v1
  with:
    openrouter-api-key: ${{ secrets.OPENROUTER_API_KEY }}
    scan-mode: 'pr'
    pr-number: ${{ github.event.pull_request.number }}
```

### [`ai-code-review`](./actions/ai-code-review/) - Claude 3.5 Sonnet
Comprehensive PR code review.

**Usage:**
```yaml
- uses: ROTA-edu/.github/actions/ai-code-review@v1
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    focus-areas: security,performance,testing
```

**Cost:** ~$2-7/month per repo (using cheap models)

## Repository Structure

```
.github/
├── .github/
│   └── workflows/           # Reusable workflows
│       ├── reusable-ci.yml
│       └── reusable-publish.yml
├── actions/                 # Custom GitHub Actions
│   └── ai-code-review/
│       └── action.yml
├── docs/                    # Central documentation
│   ├── README.md
│   ├── architecture/
│   ├── guidelines/
│   ├── core/
│   └── api/
├── workflow-templates/      # Starter templates for new repos
└── README.md               # This file
```

## Quick Start for New Repos

1. **Copy workflow template:**
   ```bash
   cp .github/workflow-templates/package-ci.yml your-repo/.github/workflows/ci.yml
   ```

2. **Update repo-specific details** (if needed)

3. **Enable branch protection:**
   - Settings → Branches → Add rule
   - Branch name pattern: `main`
   - Require pull request before merging
   - Require status checks to pass

4. **Add secrets** (if using AI features):
   ```bash
   gh secret set ANTHROPIC_API_KEY -b"sk-ant-..." --repo ROTA-edu/your-repo
   ```

## Contributing

### Documentation Updates

1. Create branch: `git checkout -b docs/update-xyz`
2. Edit relevant files in `docs/`
3. Create PR with clear description
4. Tag `@tech-leads` for review

### Workflow Updates

1. Test changes in a separate test repo first
2. Create PR with explanation of changes
3. Update version tags after merge

## Resources

- [ROTA Organization](https://github.com/ROTA-edu)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Reusable Workflows Guide](https://docs.github.com/en/actions/using-workflows/reusing-workflows)

---

**Questions?** Create an issue or post in #engineering-help
# Test
