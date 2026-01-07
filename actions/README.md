# ROTA Custom GitHub Actions

Custom AI-powered automation actions for ROTA repositories.

## 🤖 Available Actions

### 1. Wiki & Issue Agent
**Path:** `wiki-issue-agent/`
**Model:** DeepSeek V3.2 (164K context, $0.25/M input, $0.38/M output)
**Purpose:** Dual-purpose agent for wiki maintenance and issue triage

#### Wiki Mode (Weekly)
- Analyzes recent code changes (last 7 days)
- Suggests wiki/documentation updates
- Creates issues for needed documentation
- **Trigger:** Every Monday at 9 AM UTC

#### Issue Mode (Instant)
- Analyzes new issues automatically
- Suggests labels and priority
- Identifies potential root cause
- Provides next steps for assignee
- **Trigger:** When issue is opened

**Usage:**
```yaml
- uses: ROTA-edu/.github/actions/wiki-issue-agent@v1
  with:
    openrouter-api-key: ${{ secrets.OPENROUTER_API_KEY }}
    mode: 'wiki'  # or 'issue'
    issue-number: 123  # required for issue mode
```

---

### 2. Bug Finder Agent
**Path:** `bug-finder-agent/`
**Model:** Grok Code Fast 1 (2M context, $0.20/M input, $0.50/M output)
**Purpose:** Proactive bug and security vulnerability detection

#### Daily Scan
- Scans files changed in last 24 hours
- If no changes, samples random files
- Creates issues for critical/high-priority bugs
- **Trigger:** Daily at 2 AM UTC

#### PR Scan
- Scans only files changed in PR
- Posts review comment with findings
- Fails CI if critical issues found
- **Trigger:** On PR open/update

**Severity Levels:**
- 🔴 **Critical:** Security vulnerabilities, data loss risks → Blocks PR
- 🟠 **High:** Logic errors, performance issues → Creates issue
- 🟡 **Medium:** Code quality, best practices → Advisory

**Usage:**
```yaml
- uses: ROTA-edu/.github/actions/bug-finder-agent@v1
  with:
    openrouter-api-key: ${{ secrets.OPENROUTER_API_KEY }}
    scan-mode: 'daily'  # or 'pr' or 'full'
    pr-number: 123  # required for pr mode
```

---

### 3. AI Code Review
**Path:** `ai-code-review/`
**Model:** GLM-4.7 (OpenRouter)
**Purpose:** Comprehensive PR code review

**Features:**
- Security analysis (XSS, injection, secrets)
- Performance review
- Testing suggestions
- Best practice enforcement

**Usage:**
```yaml
- uses: ROTA-edu/.github/actions/ai-code-review@v1
  with:
    openrouter-api-key: ${{ secrets.OPENROUTER_API_KEY }}
    focus-areas: 'security,performance,testing'
```

---

## 🔧 Setup Requirements

### Required Secrets

All repos using these actions need:

```bash
# OpenRouter API (for wiki, bug-finder, and code-review)
gh secret set OPENROUTER_API_KEY -b"sk-or-..." --repo ROTA-edu/your-repo

# GitHub token is auto-provided
```

### Cost Estimates

**Typical Monthly Usage (per repo):**

| Agent | Frequency | Cost/Run | Monthly Cost |
|-------|-----------|----------|--------------|
| Wiki Agent | Weekly (4x/month) | ~$0.01 | $0.04 |
| Issue Agent | Per issue (~20/month) | ~$0.005 | $0.10 |
| Bug Finder (Daily) | Daily (30x/month) | ~$0.02 | $0.60 |
| Bug Finder (PR) | Per PR (~40/month) | ~$0.03 | $1.20 |
| **Total** | | | **~$2/month** |

*Using cheap models (DeepSeek, Grok) keeps costs minimal!*

---

## 📋 Action Development Guidelines

### Creating New Actions

1. Create directory: `actions/your-action-name/`
2. Create `action.yml` with inputs/outputs
3. Use shell scripts or Node.js for logic
4. Test with `act` (local GitHub Actions simulator)
5. Document in this README

### Best Practices

- ✅ Use cheap models for frequent tasks
- ✅ Implement token limits to avoid runaway costs
- ✅ Cache API responses where possible
- ✅ Fail gracefully if API is down
- ✅ Add timeout limits (max 5 minutes)
- ❌ Don't call expensive models on every commit
- ❌ Don't expose API keys in logs

---

## 🐛 Troubleshooting

### "API key not found"

**Fix:**
```bash
gh secret set OPENROUTER_API_KEY -b"your-key" --repo ROTA-edu/your-repo
```

### "Action failed: model not found"

**Fix:** Check model name in action.yml. OpenRouter models:
- `deepseek/deepseek-v3.2` (wiki)
- `x-ai/grok-code-fast-1` (bug-finder)

### "Rate limit exceeded"

OpenRouter has generous rate limits. If hit:
1. Reduce scan frequency
2. Limit files per scan
3. Use smaller context windows

---

## 📚 Resources

- [DeepSeek V3.2 on OpenRouter](https://openrouter.ai/deepseek/deepseek-v3.2)
- [Grok Code Fast 1 on OpenRouter](https://openrouter.ai/x-ai/grok-code-fast-1)
- [OpenRouter Pricing](https://openrouter.ai/models?o=pricing-low-to-high)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

---

**Questions?** Create an issue or post in #engineering-help
