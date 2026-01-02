# ROTA Organization Documentation

Welcome to the ROTA engineering knowledge base. This is the **Single Source of Truth** for all architectural decisions, coding standards, and development practices across the entire ROTA polyrepo ecosystem.

## 🗺️ The Map

| Section | Description | Target Audience |
|---------|-------------|-----------------|
| **[core/](./core/)** | The "Constitution". Project vision, manifesto, and unchangeable principles. | Everyone |
| **[architecture/](./architecture/)** | System design, polyrepo strategy, data flows, and infrastructure. | Architects, Tech Leads |
| **[guidelines/](./guidelines/)** | The "Law". Coding style, testing rules, git workflow, CI/CD, naming conventions. | All Developers |
| **[api/](./api/)** | External API docs, integration guides (Anthropic, Google, OpenAI, Stripe). | Integrators |

---

## 🏗️ ROTA Polyrepo Ecosystem

Our codebase is split into specialized repositories:

### Packages (Published to GitHub Packages)
- **[@rota-edu/course-engine](https://github.com/ROTA-edu/rota-course-engine)** - AI course generation engine
- **[@rota-edu/shared-utils](https://github.com/ROTA-edu/rota-shared-utils)** - Logging, retry, validation, caching utilities
- **[@rota-edu/canvas-engine](https://github.com/ROTA-edu/rota-canvas-engine)** - Graph visualization engine *(planned)*
- **[@rota-edu/types](https://github.com/ROTA-edu/rota-types)** - Shared TypeScript types *(planned)*
- **[@rota-edu/ai-engine](https://github.com/ROTA-edu/rota-ai-engine)** - General AI utilities (model routing, prompts) *(planned)*

### Applications
- **[rota-atlas-app](https://github.com/ROTA-edu/rota-atlas-app)** - Course viewer & material navigator
- **[rota-platform](https://github.com/ROTA-edu/rota-platform)** - Main SaaS application (monorepo)

### Infrastructure
- **[.github](https://github.com/ROTA-edu/.github)** - Central documentation & reusable workflows (this repo)

---

## 📋 The 3 Golden Rules

1. **Docs as Code:** Documentation lives with the code. If you change a feature, update relevant docs in the same PR.
2. **Single Source of Truth:** Never duplicate information. If a rule exists in `guidelines/`, link to it instead of repeating.
3. **Future-Proof:** Write docs assuming the reader knows nothing about the context. Explain *why*, not just *how*.

---

## 🚀 Quick Start for New Developers

### First Day
1. Read the **[Manifesto](./core/MANIFESTO.md)** to understand our philosophy
2. Review **[Architecture Overview](./architecture/POLYREPO_ARCHITECTURE.md)** to see the big picture
3. Set up your workspace following **[Development Setup](./guidelines/DEVELOPMENT_SETUP.md)**

### Before Writing Code
4. Check **[Coding Guidelines](./guidelines/CODE_STYLE.md)** for our standards
5. Review **[Frontend Guidelines](./guidelines/FRONTEND.md)** (for UI work)
6. Review **[Backend Guidelines](./guidelines/BACKEND.md)** (for API work)
7. Read **[Testing Guidelines](./guidelines/TESTING.md)** before writing your first test

### Before Submitting PR
8. Follow **[Git Workflow](./guidelines/GIT.md)** conventions
9. Ensure CI/CD checks pass (see **[CI/CD Guide](./guidelines/CI_CD.md)**)
10. Request review from relevant team members

---

## 📚 Individual Repo Documentation

Each repository has its own README with specific setup instructions:

- **course-engine**: See [rota-course-engine/README.md](https://github.com/ROTA-edu/rota-course-engine/blob/main/README.md)
- **atlas-app**: See [rota-atlas-app/README.md](https://github.com/ROTA-edu/rota-atlas-app/blob/main/README.md)
- **platform**: See [rota-platform/README.md](https://github.com/ROTA-edu/rota-platform/blob/main/README.md)

**Rule:** Individual repo READMEs should be SHORT and link back to this central documentation for detailed guidelines.

---

## 🔍 Finding Information

### "How do I...?"
- **Write a new component?** → [Frontend Guidelines](./guidelines/FRONTEND.md)
- **Create a new API endpoint?** → [Backend Guidelines](./guidelines/BACKEND.md)
- **Write tests?** → [Testing Guidelines](./guidelines/TESTING.md)
- **Integrate an AI model?** → [AI Integration Guide](./guidelines/AI_INTEGRATION.md) *(planned)*
- **Deploy a package?** → [Publishing Guide](./guidelines/PUBLISHING.md) *(planned)*

### "Why is it designed this way?"
- **Polyrepo vs Monorepo?** → [Architecture Principles](./guidelines/ARCHITECTURE_PRINCIPLES.md)
- **Tech stack choices?** → [Technology Decisions](./architecture/TECH_STACK.md) *(planned)*
- **Type safety rules?** → [Type Safety](./guidelines/TYPE_SAFETY.md)

---

## 🤝 Contributing to Docs

Found outdated information? See a gap in documentation?

1. Create a branch: `git checkout -b docs/fix-xyz`
2. Update the relevant `.md` file
3. Submit a PR with clear description of changes
4. Tag `@docs-maintainers` for review

**Remember:** Good documentation is a gift to future developers (including yourself!).

---

## 📞 Support

- **Technical Questions**: Post in #engineering-help (Slack/Discord)
- **Documentation Issues**: Create issue in [.github repo](https://github.com/ROTA-edu/.github/issues)
- **Architecture Discussions**: Tag @tech-leads in relevant repo

---

*"Documentation is a love letter that you write to your future self."* 💌
