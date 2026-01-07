## Description

<!-- Provide a concise description of what this PR does and why it's needed -->

## Type of Change

<!-- Mark the relevant option with an 'x' -->

- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update
- [ ] Style/UI update (no functional changes)
- [ ] Refactoring (no functional changes)
- [ ] Performance improvement
- [ ] Test update
- [ ] Build/configuration change
- [ ] CI/CD change
- [ ] Chore (maintenance tasks)

## Related Issues

<!-- Link related issues using keywords (closes, fixes, resolves) -->
<!-- Example: Closes #123, Fixes #456 -->

- Related to:
- Closes:

## Changes Made

<!-- List the main changes in this PR -->

- Change 1
- Change 2
- Change 3

## Testing Checklist

<!-- Mark completed items with an 'x' -->

### Test Coverage

- [ ] Unit tests added/updated for new code
- [ ] Integration tests added/updated (if applicable)
- [ ] E2E tests added/updated (if applicable)
- [ ] All existing tests pass
- [ ] Test coverage maintained or improved (≥80%)

### Manual Testing

- [ ] Tested locally in development environment
- [ ] Tested on staging environment (if applicable)
- [ ] Tested edge cases and error scenarios
- [ ] Tested with different user roles/permissions (if applicable)
- [ ] Tested cross-browser compatibility (if frontend change)
- [ ] Tested mobile responsiveness (if UI change)

### Test Plan

<!-- Describe how you tested this change -->

```
1. Step 1
2. Step 2
3. Expected result
```

## Code Quality Checklist

<!-- Mark completed items with an 'x' -->

- [ ] Code follows project style guidelines
- [ ] ESLint passes with no errors
- [ ] TypeScript types are properly defined (no `any` types)
- [ ] Code is self-documenting with clear variable/function names
- [ ] Complex logic includes comments explaining the 'why'
- [ ] No console.log or debugging code left in
- [ ] No commented-out code blocks
- [ ] Functions are small and focused (single responsibility)
- [ ] No code duplication (DRY principle followed)
- [ ] Error handling is comprehensive and user-friendly

## Documentation Checklist

<!-- Mark completed items with an 'x' -->

- [ ] JSDoc/TSDoc comments added for new functions/classes
- [ ] README updated (if needed)
- [ ] API documentation updated (if API changes)
- [ ] Architecture documentation updated (if architecture changes)
- [ ] Migration guide provided (if breaking changes)
- [ ] Changelog updated
- [ ] User-facing documentation updated (if UI/UX changes)

## Architecture Compliance

<!-- Mark completed items with an 'x' -->

- [ ] Follows monorepo package structure
- [ ] Respects package boundaries (no circular dependencies)
- [ ] Uses shared types from `@rota-edu/types`
- [ ] Follows event-driven architecture patterns (if applicable)
- [ ] Implements proper error handling and logging
- [ ] Uses dependency injection where appropriate
- [ ] Follows SOLID principles
- [ ] No tight coupling between modules

## Security Checklist

<!-- Mark completed items with an 'x' -->

- [ ] No sensitive data (API keys, passwords, tokens) in code
- [ ] Input validation implemented for user inputs
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS prevention (proper escaping/sanitization)
- [ ] CSRF protection implemented (if applicable)
- [ ] Authentication/authorization checks in place
- [ ] No security vulnerabilities introduced (run `pnpm audit`)
- [ ] Secrets stored in environment variables
- [ ] Dependencies are up to date and secure

## Performance Checklist

<!-- Mark completed items with an 'x' -->

- [ ] No N+1 queries or performance regressions
- [ ] Database queries are optimized with proper indexing
- [ ] Large operations use pagination/cursors
- [ ] Expensive operations are cached where appropriate
- [ ] Bundle size impact is minimal (if frontend change)
- [ ] Images/assets are optimized (if applicable)
- [ ] No memory leaks introduced
- [ ] Asynchronous operations use proper concurrency control

## Database Changes

<!-- If this PR includes database changes, provide details -->

- [ ] N/A - No database changes
- [ ] Migration scripts created and tested
- [ ] Migrations are reversible (down migrations)
- [ ] Seed data updated (if needed)
- [ ] Database indexes added for new queries
- [ ] Data migration plan documented (if large dataset)
- [ ] Backwards compatibility maintained

## Breaking Changes

<!-- If this PR includes breaking changes, describe them -->

- [ ] N/A - No breaking changes
- [ ] Breaking changes are documented below
- [ ] Migration guide provided
- [ ] Dependent packages/services identified and updated
- [ ] Version bump follows semver (major version)

### Breaking Change Details

<!-- Describe what breaks and how to migrate -->

```
Before:
// Old code example

After:
// New code example

Migration steps:
1. Step 1
2. Step 2
```

## Deployment Notes

<!-- Any special deployment instructions or considerations -->

- [ ] N/A - Standard deployment
- [ ] Requires environment variable changes (listed below)
- [ ] Requires database migration
- [ ] Requires infrastructure changes
- [ ] Requires cache invalidation
- [ ] Requires manual deployment steps (documented below)
- [ ] Rollback plan documented (if risky change)

### Environment Variables

<!-- List any new or changed environment variables -->

```env
NEW_VAR=description
UPDATED_VAR=new_description
```

### Deployment Steps

<!-- Detailed deployment steps if non-standard -->

```
1. Step 1
2. Step 2
3. Step 3
```

### Rollback Plan

<!-- How to revert this change if something goes wrong -->

```
1. Rollback step 1
2. Rollback step 2
```

## Screenshots/Videos

<!-- If this PR includes UI changes, add screenshots or videos -->

### Before

<!-- Screenshot/video of old behavior -->

### After

<!-- Screenshot/video of new behavior -->

## Additional Context

<!-- Add any other context about the PR here -->

## Reviewer Notes

<!-- Anything specific you want reviewers to focus on -->

- [ ] Please pay special attention to:
- [ ] Known limitations:
- [ ] Future improvements planned:

## Pre-merge Checklist

<!-- Final checks before merging - DO NOT MERGE UNTIL ALL ARE CHECKED -->

- [ ] All CI/CD checks pass (lint, test, build, security)
- [ ] Code has been reviewed and approved by required reviewers
- [ ] All review comments have been addressed and resolved
- [ ] Branch is up to date with base branch
- [ ] No merge conflicts
- [ ] Documentation is complete and accurate
- [ ] Breaking changes are well documented
- [ ] Tests provide adequate coverage
- [ ] Performance impact is acceptable
- [ ] Security checklist is complete

---

<!--
By submitting this PR, I confirm that:
- I have read the contributing guidelines
- My code follows the project's coding standards
- I have tested my changes thoroughly
- I have updated documentation as needed
- I understand this will be reviewed before merging
-->

**Ready to merge:** Yes / No

<!-- If "No", explain what's remaining: -->
