# ROTA Polyrepo Architecture

> **Scope:** High-Level Repository Structure & Dependencies
> **Status:** Active (2026)
> **Last Updated:** 2026-01-02

---

## 1. The ROTA Ecosystem

ROTA is not a single codebase. It is a **cluster of specialized repositories** working together, each with clear responsibilities and boundaries.

```mermaid
graph TD
    DEV((Developer))

    subgraph "Foundation Layer"
        TYPES[@rota-edu/types]
        UTILS[@rota-edu/shared-utils]
    end

    subgraph "Engine Layer (Private IP)"
        COURSE[@rota-edu/course-engine]
        AI[@rota-edu/ai-engine]
        CANVAS[@rota-edu/canvas-engine]
        META[@rota-edu/metahuman-engine]
    end

    subgraph "Application Layer"
        ATLAS[rota-atlas-app]
        PLATFORM[rota-platform]
    end

    subgraph "Infrastructure"
        GITHUB[.github]
    end

    DEV -->|Clones| ATLAS
    DEV -->|Clones| PLATFORM

    TYPES -->|Published to| REGISTRY[(GitHub Packages)]
    UTILS -->|Published to| REGISTRY
    COURSE -->|Published to| REGISTRY
    AI -->|Published to| REGISTRY
    CANVAS -->|Published to| REGISTRY
    META -->|Published to| REGISTRY

    REGISTRY -->|npm install| ATLAS
    REGISTRY -->|npm install| PLATFORM

    GITHUB -->|Workflows & Docs| ATLAS
    GITHUB -->|Workflows & Docs| PLATFORM
    GITHUB -->|Workflows & Docs| COURSE
```

---

## 2. Repository Definitions

### Foundation Layer

#### @rota-edu/types (Public - Planned)
- **Role:** The Contract
- **Content:** TypeScript interfaces, Zod schemas, API types
- **Dependencies:** NONE (foundation)
- **Used By:** ALL repos
- **Rule:** If a type is shared between 2+ repos, it MUST be defined here
- **Examples:** `UserProfile`, `CoursePackage`, `GraphNode`, `Edge`

#### @rota-edu/shared-utils (Private)
- **Role:** The Toolbox
- **Content:** Logging (Pino), retry logic, validation, caching, error handling
- **Dependencies:** `pino`, `zod`, minimal third-party
- **Used By:** All packages and apps
- **Exports:**
  - `Logger`, `createRequestLogger()` (logging)
  - `retry()`, `retryAI()`, `CircuitBreaker` (retry logic)
  - `validateWithSchema()`, `sanitizeUserInput()` (validation)
  - `getCache()`, `withCache()` (caching)
- **Quality:** >90% test coverage required

---

### Engine Layer (Private IP Protection)

#### @rota-edu/course-engine (Private)
- **Role:** The Course Generator
- **Content:**
  - Orchestrator & Worker agents
  - AI prompts for course generation
  - CASE standards integration
  - Bloom taxonomy logic
  - Course structure generation
- **Dependencies:** `@rota-edu/types`, `@rota-edu/shared-utils`, `@rota-edu/ai-engine`, AI SDKs
- **Exports:**
  - `generateCoursePackage(profile): Promise<CoursePackage>`
  - `generateCoursePackageWithProgress(profile, callbacks)`
  - `createQuickProfile(topic, options)`
- **IP Protection:** Prompts and orchestration logic are proprietary

#### @rota-edu/ai-engine (Private - Planned)
- **Role:** The AI Abstraction Layer
- **Content:**
  - Model routing (Gemini, GPT, DeepSeek selection)
  - Prompt utilities and templates
  - AI SDK wrappers
  - Token counting & cost calculation
  - Response streaming helpers
- **Dependencies:** `@rota-edu/types`, `@rota-edu/shared-utils`, AI SDKs
- **Exports:**
  - `selectModel(taskType): ModelConfig`
  - `createAIProvider(config): AIProvider`
  - `streamResponse(prompt, callbacks)`
  - `calculateTokens(text): number`
- **Why Separate:** Canvas AI needs are different from Course AI needs
  - Canvas: Quick responses, layout suggestions, visual reasoning
  - Course: Deep reasoning, structured output, multi-step orchestration

#### @rota-edu/canvas-engine (Private - Planned)
- **Role:** The Visualization Engine
- **Content:**
  - Svelte Flow graph components
  - ELK layout algorithms
  - Force-directed graph physics
  - Interactive navigation (drag, zoom, pan)
  - Animation & transitions
- **Dependencies:** `@rota-edu/types`, `@rota-edu/ai-engine` (for AI-assisted layout)
- **Exports:**
  - `<RotaCanvas />` (Svelte component)
  - `useGraphStore()` (state management)
  - `calculateLayout(nodes, edges): Layout`
- **Why Separate:** Reusable across Atlas, Platform, and future products

#### @rota-edu/metahuman-engine (Private - Future)
- **Role:** The 3D Avatar System
- **Content:**
  - 3D avatar rendering (Three.js/WebGL)
  - Facial animations & lip sync
  - Voice synchronization
  - Gesture control
- **Dependencies:** `@rota-edu/types`, `@rota-edu/ai-engine`
- **Tech Stack:** Three.js, WebGL, potentially Unity WebGL
- **Why Separate:** Completely different tech stack from 2D canvas

---

### Application Layer

#### rota-atlas-app (Private/Semi-Public)
- **Role:** Course Material Viewer
- **Content:**
  - SvelteKit routes & components
  - Course navigation UI
  - Material viewer (lessons, quizzes, resources)
  - Progress tracking
- **Dependencies:**
  - `@rota-edu/course-engine` (to generate courses)
  - `@rota-edu/canvas-engine` (to visualize course graph)
  - `@rota-edu/shared-utils` (logging, validation)
- **Logic:** Minimal business logic, mostly UI and integration
- **Access Control:** Frontend team can work here without seeing AI prompts

#### rota-platform (Private Monorepo)
- **Role:** Main SaaS Application
- **Structure:**
  ```
  apps/
    web/              # Main SvelteKit app
  packages/
    ai-core/          # Voice AI (to be extracted to @rota-edu/ai-engine)
    canvas-engine/    # Graph viz (to be extracted to @rota-edu/canvas-engine)
    types/            # Shared types (to be extracted to @rota-edu/types)
  ```
- **Dependencies:** Same as Atlas + additional packages
- **Migration Plan:** Gradually extract packages to separate repos

---

### Infrastructure

#### ROTA-edu/.github (Private)
- **Role:** Central Documentation & CI/CD
- **Content:**
  - Architecture docs (this file)
  - Coding guidelines
  - Reusable GitHub Actions workflows
  - Custom actions (AI code review, changelog generation)
  - Organizational profile
- **Special:** GitHub automatically uses this repo for org-wide settings

---

## 3. Dependency Graph

```
┌──────────────────────────────────────────────────────┐
│  DEPENDENCY FLOW (Bottom-up)                          │
└──────────────────────────────────────────────────────┘

Layer 0: Foundation (NO dependencies)
   @rota-edu/types

Layer 1: Utilities (depends on types only)
   @rota-edu/shared-utils
   └─ types

Layer 2: AI Abstraction (depends on utils + types)
   @rota-edu/ai-engine
   ├─ types
   └─ shared-utils

Layer 3: Domain Engines (depends on ai + utils + types)
   @rota-edu/course-engine
   ├─ types
   ├─ shared-utils
   └─ ai-engine

   @rota-edu/canvas-engine
   ├─ types
   ├─ shared-utils
   └─ ai-engine (optional, for AI-assisted layout)

   @rota-edu/metahuman-engine
   ├─ types
   ├─ shared-utils
   └─ ai-engine

Layer 4: Applications (depends on ALL engines)
   rota-atlas-app
   ├─ types
   ├─ shared-utils
   ├─ course-engine
   └─ canvas-engine

   rota-platform
   ├─ types
   ├─ shared-utils
   ├─ course-engine
   ├─ canvas-engine
   ├─ ai-engine
   └─ metahuman-engine (future)
```

**Critical Rules:**
- ✅ Lower layers NEVER import from higher layers
- ✅ Engines are independent (course-engine doesn't import canvas-engine)
- ❌ NO circular dependencies allowed
- ❌ Apps cannot be imported by packages

---

## 4. Data Flow Example: Course Generation with Visualization

```
┌─────────────────────────────────────────────────────────────┐
│  USER FLOW: Generate Course & View on Graph                 │
└─────────────────────────────────────────────────────────────┘

1. User fills form in Atlas/Platform
   └─> Atlas UI collects UserProfile data

2. Atlas calls course-engine
   └─> import { generateCoursePackage } from '@rota-edu/course-engine'
   └─> coursePackage = await generateCoursePackage(profile)

3. course-engine orchestrates AI generation
   ├─> Uses ai-engine.selectModel('complex-reasoning')
   ├─> Orchestrator agent creates blueprint
   ├─> Worker agents generate content
   └─> Returns CoursePackage (nodes + edges + metadata)

4. Atlas visualizes with canvas-engine
   └─> import { RotaCanvas } from '@rota-edu/canvas-engine'
   └─> <RotaCanvas nodes={coursePackage.nodes} edges={coursePackage.edges} />

5. canvas-engine renders graph
   ├─> Uses ELK layout algorithm
   ├─> Optionally calls ai-engine for layout optimization
   └─> User sees interactive course graph
```

---

## 5. Team Access Control Matrix

| Repo | Frontend Dev | Backend Dev | AI Engineer | Full-Stack | Admin |
|------|-------------|-------------|-------------|------------|-------|
| **types** | Read | Read | Read | Write | Admin |
| **shared-utils** | Read | Read | Read | Write | Admin |
| **ai-engine** | No Access | Read | Write | Write | Admin |
| **course-engine** | No Access | Read | Write | Write | Admin |
| **canvas-engine** | Read | Read | Read | Write | Admin |
| **metahuman-engine** | Read | Read | Write | Write | Admin |
| **atlas-app** | Write | Read | Read | Write | Admin |
| **platform** | Write | Write | Read | Write | Admin |
| **.github** | Read | Read | Read | Write | Admin |

**Key:**
- **No Access:** Cannot see repo at all
- **Read:** Can clone, view code, install package
- **Write:** Can push commits, create PRs
- **Admin:** Full control (dangerous!)

---

## 6. Publishing Strategy

### GitHub Packages (Private Registry)

All packages published to: `https://npm.pkg.github.com/@rota-edu/*`

**Publishing Flow:**
```bash
# 1. Update version
npm version patch  # or minor, major

# 2. Push tag
git push --tags

# 3. GitHub Actions auto-publishes
# - Runs CI (tests, type-check, build)
# - Publishes to GitHub Packages
# - Creates GitHub Release with AI-generated notes
```

**Consuming Flow:**
```bash
# In atlas-app or platform

# 1. Configure .npmrc
echo "@rota-edu:registry=https://npm.pkg.github.com" >> .npmrc

# 2. Authenticate (one-time)
npm login --registry=https://npm.pkg.github.com

# 3. Install
pnpm add @rota-edu/course-engine
pnpm add @rota-edu/canvas-engine
```

---

## 7. Migration Roadmap

### Phase 1: Foundation (✅ Completed)
- [x] Extract course-engine from platform
- [x] Create .github central docs repo
- [x] Set up reusable CI/CD workflows

### Phase 2: Utilities (In Progress)
- [ ] Create @rota-edu/shared-utils
- [ ] Extract logging, retry, validation from course-engine
- [ ] Migrate course-engine to use shared-utils
- [ ] Migrate atlas-app to use shared-utils

### Phase 3: AI Abstraction (Planned)
- [ ] Create @rota-edu/ai-engine
- [ ] Extract model routing, prompt utilities
- [ ] Migrate course-engine to use ai-engine
- [ ] Extract platform ai-core to ai-engine

### Phase 4: Visualization (Planned)
- [ ] Create @rota-edu/canvas-engine
- [ ] Extract from platform packages/canvas-engine
- [ ] Add AI-assisted layout features
- [ ] Migrate atlas & platform to use canvas-engine

### Phase 5: Types (Planned)
- [ ] Create @rota-edu/types (public)
- [ ] Extract types from all repos
- [ ] Migrate all repos to use central types

### Phase 6: Metahuman (Future)
- [ ] Create @rota-edu/metahuman-engine
- [ ] 3D avatar rendering
- [ ] Voice synchronization
- [ ] Integration with platform

---

## 8. Architectural Principles

### 1. **Separation of Concerns**
Each repo has ONE clear responsibility. No overlap.

### 2. **Dependency Inversion**
High-level modules (apps) depend on low-level modules (packages), not vice versa.

### 3. **Single Direction**
Dependencies flow in one direction: Apps → Engines → Utils → Types

### 4. **No Circular Dependencies**
Enforced by tooling (dependency-cruiser). Build fails if detected.

### 5. **Package Independence**
Engines don't depend on each other. Canvas doesn't import Course.

### 6. **Clear Contracts**
All inter-repo communication happens through well-defined TypeScript interfaces.

### 7. **IP Protection**
Proprietary logic (AI prompts, orchestration) isolated in private packages.

### 8. **Gradual Migration**
Don't extract everything at once. Stabilize, then extract.

---

## 9. Decision Log

### Why Polyrepo instead of Monorepo?
- **Team Isolation:** Frontend devs don't need to see AI prompts
- **Access Control:** Granular permissions (some engineers see only canvas)
- **Independent Versioning:** course-engine v2.0.0 doesn't force atlas-app update
- **Build Speed:** CI only runs for changed repo (not entire monorepo)
- **IP Protection:** Private packages keep prompts & algorithms secret

### Why Extract ai-engine separately from course-engine?
- **Different Use Cases:** Canvas needs quick AI, Course needs deep reasoning
- **Reusability:** Voice AI, Canvas AI, Course AI all use same core utilities
- **Testing:** Easier to test AI utilities in isolation
- **Cost Optimization:** Model routing logic benefits all AI features

### Why Keep Platform as Monorepo?
- **Tight Integration:** Web app, packages, shared configs work together
- **Faster Development:** Change multiple packages in single PR
- **Gradual Migration:** Extract when packages mature

---

## 10. FAQ

**Q: Which repo should I work in?**
- AI prompts/logic → course-engine or ai-engine
- Graph visualization → canvas-engine (or platform/packages/canvas-engine for now)
- UI components → atlas-app or platform/apps/web
- Shared utilities → shared-utils
- Types → types (or define in individual repos for now)

**Q: How do I test changes across multiple repos?**
- Use `pnpm link` for local development
- Or publish pre-release version: `pnpm publish --tag beta`

**Q: What if I need to update both course-engine and atlas-app?**
1. Update course-engine, publish new version
2. Update atlas-app to use new version
3. Test integration
4. Deploy

**Q: Why can't I just keep everything in one repo?**
See "Why Polyrepo" decision log above. TL;DR: Team isolation + IP protection.

---

**Document Owner:** Tech Lead
**Last Review:** 2026-01-02
**Next Review:** 2026-04-01
