# Backend Guidelines

> **Philosophy:** The Backend is the "Security Gateway".

---

## 1. The Gateway Pattern

Since `rota-ai-core` runs in the browser (via WebSocket), it cannot hold secrets (API Keys). The Backend (SvelteKit API Routes) acts as a proxy to inject these secrets securely.

### The Flow
1.  **Client:** Requests a session token (`POST /api/session/start`).
2.  **Server:** Validates user, generates a short-lived token.
3.  **Client:** Connects to WebSocket Proxy (`wss://api.rota.com/proxy?token=xyz`).
4.  **Proxy:** Validates token, connects to Gemini/OpenAI using the **Server-Side API Key**, and pipes the stream.

### Rules
*   **No Keys in Client:** `PUBLIC_GEMINI_KEY` is forbidden. Keys live only in `.env` on the server.
*   **Rate Limiting:** All API routes must implement `RateLimiter` middleware.
*   **Validation:** All request bodies must be validated with Zod schemas from `@rota/types`.

---

## 2. Private Package Consumption

The backend also consumes `@rota/ai` for server-side tasks (e.g., Course Generation).

*   **Authentication:** The CI/CD pipeline and local `.npmrc` must have a token to access GitHub Packages.
*   **Isolation:** Server-side AI logic (DeepSeek) runs here, separate from the client-side AI logic (Gemini Live).

---

## 3. Database Access

*   **Prisma:** We use Prisma ORM.
*   **Repo Separation:** Database schema definitions live in `rota-shared-types` (or a dedicated `rota-db` repo if complex) to ensure Typescript models match the DB.

---

## 4. API Response Standards

All API endpoints must return the standardized `ApiResponse<T>` envelope defined in `@rota/types`.

```typescript
// Good
return json({ success: true, data: user });

// Bad
return json(user);
```