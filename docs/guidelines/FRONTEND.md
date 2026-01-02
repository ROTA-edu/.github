# Frontend Guidelines

> **Philosophy:** The Frontend is a "Dumb Shell". It orchestrates, it does not think.

---

## 1. The "Shell" Concept

`rota-frontend` is purely an integration layer. It connects the user to the `rota-ai-core` (Brain) and displays the `rota-canvas-engine` (Hand).

### Anti-Patterns (Forbidden)
*   **Business Logic:** Writing complex algorithms in `+page.svelte`.
*   **Direct API Calls:** Calling `fetch('https://api.openai.com')` directly from the client.
*   **Prompt Storage:** Storing system instructions or prompts in `src/lib/prompts`. (They belong in `@rota/ai`).
*   **Canvas Logic:** Calculating graph layouts or physics in components. (They belong in `@rota/canvas`).

### Correct Usage
*   **Importing Logic:**
    ```typescript
    import { startSession } from '@rota/ai';
    import { RotaCanvas } from '@rota/canvas';
    ```
*   **Orchestration:** Listening to events from AI and passing them to Canvas.

---

## 2. Component Architecture (Svelte 5)

We use **Svelte 5 Runes** for all state management.

### Structure
```text
src/lib/components/
├── ui/              # Dumb atoms (Button, Card, Input) - Tailwind only
├── layout/          # Application shell (Header, Sidebar)
├── integration/     # Smart wrappers connecting @rota packages
│   ├── AIStatus.svelte
│   └── CanvasWrapper.svelte
```

### Rules
1.  **Dumb Components:** UI components (`ui/`) must accept data via `props` and emit events. They should not import stores.
2.  **Smart Components:** Integration components (`integration/`) handle the glue logic between packages.
3.  **No `any`:** Props must be strictly typed using `@rota/types`.

---

## 3. State Management

*   **Global State:** Managed by `@rota/ai` and `@rota/canvas` internal stores.
*   **Local UI State:** Managed by Svelte 5 Runes (`$state`, `$derived`) within the shell.
*   **Persistence:** `localStorage` logic is handled by the shell, not the packages.

---

## 4. Contractor Workflow

If you are a contractor working on `rota-frontend`:

1.  **Do not touch `@rota/*` packages.** You treat them as black boxes.
2.  **Use Mocks:** Enable `USE_MOCK_AI=true` in your `.env`. This provides fake AI responses so you can style the UI without burning API credits.
3.  **Focus on UX:** Your job is to make the interface beautiful and responsive. The "thinking" happens elsewhere.
