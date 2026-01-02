# The ROTA Engineering Manifesto

> **"Voice First. Visual Second. Text Last."**

This manifesto defines the engineering culture and core principles of ROTA. It is the compass that guides our architectural decisions, code quality, and product development.

---

## 1. Core Philosophy

### Voice is the Interface
ROTA is not a chat app with a microphone button. It is a **Voice-Native OS**.
*   **Rule:** If a feature requires a mouse click to be usable, it is an accessibility fallback, not the primary flow.
*   **Metric:** Can a user complete a full lesson with their screen turned off? (If yes, we succeeded).

### The "Shared Mental Model"
The AI and the User must share the same context.
*   **Visuals:** If the AI explains a triangle, the Canvas MUST show a triangle.
*   **Vision:** If the user draws a circle, the AI MUST "see" the circle.
*   **Sync:** Latency between "User Speaks" and "AI Sees" must be <500ms.

### Modular Monolith (Future-Ready)
We build today as if we are splitting into microservices tomorrow.
*   **Isolation:** The `Canvas` engine should not know that `Voice` exists. They communicate via the `Orchestrator`.
*   **Feature-First:** Code is organized by Domain (e.g., `features/profiler`), not by Type (`components/buttons`).

---

## 2. Engineering Standards

### Quality over Speed
*   **Typed:** `any` is forbidden. If you can't type it, you don't understand it.
*   **Tested:** Critical logic (Pedagogy, State Sync) must have unit tests. UI can be snapshot tested.
*   **Clean:** Delete dead code immediately. Don't comment it out.

### State Management
*   **Svelte 5 Runes:** We use Runes (`$state`, `$derived`) for all new state logic.
*   **Single Truth:** Data lives in `services` or `stores`. Components are just dumb renderers.

---

## 3. The "No-Go" Zones (Anti-Patterns)

1.  **"God Components":** If a file exceeds 300 lines, it's doing too much. Break it down.
2.  **Hardcoded Prompts:** Prompts live in the `ai/` registry, never inside React/Svelte components.
3.  **Client-Side Secrets:** API keys never touch the browser. Use the Proxy.

---

## 4. Documentation Strategy

*   **Living Docs:** Every major folder (`features/xyz`) must have a `README.md`.
*   **Architecture Decision Records (ADR):** Major changes (like switching to Gemini) must be documented in `docs/architecture/`.

---

*This manifesto is a living document. Challenge it if it slows us down, but respect it while it stands.*
