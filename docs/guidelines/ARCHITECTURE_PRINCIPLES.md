# Architecture Principles

## Separation of Concerns Rules

### Module Responsibilities

**Routes** (`src/routes/`)
- Page orchestration and layout
- Route-level data loading
- Server-side rendering setup
- NO business logic, NO direct API calls

**Components** (`src/lib/components/`)
- UI presentation only
- Receive data via props
- Emit events via callbacks
- NO direct store mutations, NO API calls

**Stores** (`src/lib/stores/`)
- Application state management
- State derivations and computed values
- State persistence (localStorage)
- NO API calls, NO side effects beyond state updates

**Services** (`src/lib/services/`)
- External API communication
- WebSocket management
- Audio processing
- Complex business logic
- MAY read stores, MUST NOT mutate stores directly

**API Routes** (`src/routes/api/`)
- Server-side endpoints
- Secret management (API keys)
- Request validation
- Rate limiting
- Database operations

### Forbidden Cross-Boundary Violations

**DON'T:** Make API calls from components
```svelte
<!-- FORBIDDEN -->
<script>
  async function handleClick() {
    const res = await fetch('/api/courses');
    const data = await res.json();
  }
</script>
```

**DO:** Call services from components
```svelte
<script lang="ts">
  import { coursesStore } from '$lib/stores/courses';

  function handleClick() {
    coursesStore.refresh(); // Store coordinates the fetch
  }
</script>
```

**DON'T:** Mutate stores from services
```typescript
// FORBIDDEN in services
import { canvasStore } from '$lib/stores/canvas';

export function processData(data: unknown) {
  canvasStore.update(state => ({ ...state, data })); // NO!
}
```

**DO:** Return data from services, let stores update
```typescript
// Service returns data
export async function fetchCanvasData(): Promise<CanvasData> {
  const response = await fetch('/api/canvas');
  return response.json();
}

// Store coordinates the update
export const canvasStore = {
  async load() {
    const data = await fetchCanvasData();
    this.set(data);
  }
};
```

## Dependency Injection Patterns

### Service Initialization
**DO:** Create services with dependency injection
```typescript
export class GeminiRealtimeService {
  constructor(
    private apiKey: string,
    private eventBus: EventBus,
    private sessionStore: SessionStore
  ) {}

  async connect(config: ConnectionConfig): Promise<void> {
    // Use injected dependencies
  }
}

// Factory function
export function createGeminiService(apiKey: string): GeminiRealtimeService {
  return new GeminiRealtimeService(apiKey, eventBus, sessionStore);
}
```

### Store Dependencies
**DO:** Inject dependencies into store creators
```typescript
function createInteractiveCanvasStore(
  eventBus?: EventBus
) {
  const { subscribe, set, update } = writable<InteractiveCanvasState>(
    createEmptyCanvasState()
  );

  return {
    subscribe,
    addNode: (node: EducationNode) => {
      update(state => ({
        ...state,
        nodes: [...state.nodes, node]
      }));
      eventBus?.emit({ type: 'node_added', node });
    }
  };
}
```

### Avoid Global Singletons for Stateful Services
**DON'T:** Export initialized service instances
```typescript
// FORBIDDEN
export const geminiService = new GeminiRealtimeService(API_KEY);
```

**DO:** Export factory functions
```typescript
export function createGeminiService(config: Config): GeminiRealtimeService {
  return new GeminiRealtimeService(config);
}

// Or use singleton pattern with lazy initialization
let instance: GeminiRealtimeService | null = null;

export function getGeminiService(): GeminiRealtimeService {
  if (!instance) {
    instance = new GeminiRealtimeService(config);
  }
  return instance;
}
```

## Module Boundaries

### Clear Interface Contracts
**DO:** Define explicit interfaces between modules
```typescript
// Service interface
export interface AudioProcessor {
  start(): Promise<void>;
  stop(): void;
  on(event: string, handler: Function): void;
}

// Multiple implementations can satisfy this interface
export class WebAudioProcessor implements AudioProcessor {
  // Implementation
}
```

### Encapsulation
**DO:** Hide implementation details
```typescript
// Public API
export const canvasStore = {
  subscribe,
  addNode,
  removeNode,
  reset
};

// Private internals
const internalState = writable<InternalState>({});
```

**DON'T:** Expose internal implementation
```typescript
// FORBIDDEN
export const canvasStore = writable<CanvasState>({});
export const _internal = { /* ... */ };
```

## Store vs Service vs Component Responsibilities

### When to Use Stores
Use stores for:
- Application-wide state
- State that multiple components need
- State that needs persistence
- Derived/computed state

**Example:** User session, course list, canvas state

### When to Use Services
Use services for:
- External API communication
- Complex algorithms
- Stateful clients (WebSocket, audio)
- Browser API interaction

**Example:** Gemini WebSocket client, audio processing, image generation

### When to Use Components
Use components for:
- UI presentation
- User interaction handling
- Layout and styling
- Local UI state (dropdowns, modals)

**Example:** Buttons, cards, forms, dialogs

### Responsibility Matrix

| Concern | Component | Store | Service | API Route |
|---------|-----------|-------|---------|-----------|
| Render UI | YES | NO | NO | NO |
| Manage state | Local only | YES | NO | NO |
| API calls | NO | Coordinate | Execute | Handle |
| Business logic | NO | Simple only | YES | YES |
| Secrets | NO | NO | NO | YES |
| Side effects | Minimal | NO | YES | YES |

## When to Create New Files vs Extend Existing

### Create New Component When:
- Component exceeds 200 lines
- Component has distinct responsibility
- Component is reused in 2+ places
- Component manages complex local state

**DO:** Extract to new file
```svelte
<!-- Before: 300-line mega component -->
<Canvas />

<!-- After: Composition -->
<Canvas>
  <CanvasToolbar />
  <CanvasViewport />
  <CanvasStatusBar />
</Canvas>
```

### Create New Store When:
- State domain is distinct
- State has different lifecycle
- State needs different persistence strategy

**Example:**
- `canvasStore` - Canvas visual state
- `sessionStore` - Active session state
- `coursesStore` - Course library

**DON'T:** Create per-component stores
```typescript
// FORBIDDEN
const buttonClickStore = writable(0);
const modalOpenStore = writable(false);
```

### Create New Service When:
- Integrating new external system
- Service exceeds 300 lines
- Service has multiple distinct responsibilities

**DO:** Split by responsibility
```typescript
// Before: Monolithic
class AudioService {
  startRecording() {}
  processAudio() {}
  synthesizeSpeech() {}
}

// After: Single responsibility
class AudioRecorder {
  startRecording() {}
  stopRecording() {}
}

class AudioProcessor {
  process(audio: AudioData) {}
}

class SpeechSynthesizer {
  synthesize(text: string) {}
}
```

### Extend Existing File When:
- Adding helper function to existing module
- Adding new method to existing service
- Adding new property to existing type
- Change is < 50 lines
- Change is cohesive with existing code

**DO:** Add related functionality
```typescript
// Existing: canvas.ts
export const canvasStore = {
  addNode,
  removeNode,
  // NEW: Related operation
  updateNode
};
```

## Event Bus Pattern

### Centralized Events
**DO:** Use event bus for cross-cutting concerns
```typescript
import { eventBus } from '$lib/orchestrator/event-bus';

// Publisher
eventBus.emit({
  type: 'user_turn_complete',
  transcript: text,
  timestamp: Date.now()
});

// Subscriber
eventBus.subscribe('user_turn_complete', (event) => {
  console.log('[Listener] User spoke:', event.transcript);
});
```

### When to Use Event Bus
Use for:
- Cross-module communication
- Analytics tracking
- Debug logging
- Orchestration events

**DON'T:** Use for parent-child component communication
```typescript
// FORBIDDEN in components
eventBus.emit({ type: 'button_clicked' });

// Use callbacks instead
let { onClick } = $props();
onClick();
```

## Provider Agnosticism

### The Agnostic Rule

Never import AI provider SDKs directly in business logic (services, stores, components).
Always use interfaces and adapters.

**DON'T:** Import provider SDKs in business logic
```typescript
// FORBIDDEN in services or components
import { GoogleGenerativeAI } from '@google/generative-ai';
import OpenAI from 'openai';

async function generateContent(prompt: string) {
    const ai = new GoogleGenerativeAI(apiKey);  // Tight coupling!
    return ai.generateContent(prompt);
}
```

**DO:** Use interfaces and adapters
```typescript
// Define interface in types/
export interface IContentGenerator {
    generate(prompt: string): Promise<string>;
}

// Implement in services/adapters/
export class GeminiAdapter implements IContentGenerator {
    async generate(prompt: string): Promise<string> {
        // Gemini-specific implementation
    }
}

// Business logic uses interface
export class ContentService {
    constructor(private generator: IContentGenerator) {}

    async process(prompt: string) {
        return this.generator.generate(prompt);
    }
}
```

### Provider Adapter Pattern

When integrating external AI services:

```
src/lib/services/
├── adapters/                    # Provider-specific implementations
│   ├── gemini-realtime.ts       # Gemini Native Audio adapter
│   └── (future) openai-realtime.ts
├── interfaces/                  # Abstract contracts
│   └── ISpeechService.ts        # Voice service interface
└── audio-processor.ts           # Provider-agnostic audio logic
```

**Interface Example:**
```typescript
// src/lib/services/interfaces/ISpeechService.ts
export interface ISpeechService {
    connect(context: LessonContext): Promise<void>;
    disconnect(): void;
    isConnected(): boolean;
    sendAudio(buffer: ArrayBuffer): void;
    onTranscript(handler: (text: string) => void): void;
    onToolCall(handler: (call: ToolCall) => void): void;
    onAudioResponse(handler: (audio: ArrayBuffer) => void): void;
}
```

### When to Apply Provider Agnosticism

| Scenario | Apply Agnosticism? |
|----------|-------------------|
| External AI APIs (OpenAI, Gemini, Claude) | YES - Always |
| Internal utility libraries | NO - Overkill |
| Database clients (Prisma) | NO - Unlikely to change |
| Third-party SDKs with no alternatives | MAYBE - Cost/benefit |

### Benefits

1. **Swap providers without rewriting business logic**
2. **Test services with mock implementations**
3. **Compare providers during evaluation**
4. **Gradual migration between providers**

## Tool Definition Standards

### Single Source of Truth

Tool schemas (for AI function calling) MUST be defined in ONE place only.

**Current Location:** `src/lib/config/gemini-tools.ts`

**Future Pattern (when adding providers):**
```
src/lib/config/tools/
├── definitions.ts           # Canonical tool schemas
├── index.ts                 # Re-exports
└── adapters/
    ├── gemini.ts            # toGeminiFormat()
    └── openai.ts            # toOpenAIFormat()
```

### Canonical Schema Format

Define tools in a provider-agnostic format:

```typescript
// src/lib/config/tools/definitions.ts
export const TOOL_SCHEMAS = {
    create_education_graph: {
        name: 'create_education_graph',
        description: 'Create an interactive educational graph visualization',
        parameters: {
            title: { type: 'string', required: true },
            nodes: {
                type: 'array',
                items: {
                    id: { type: 'string' },
                    label: { type: 'string' },
                    type: { type: 'string', enum: ['concept', 'definition', 'example'] }
                }
            },
            edges: { type: 'array', items: { source: 'string', target: 'string' } }
        }
    }
} as const;
```

### Provider Adapters

Each provider may require different formatting:

```typescript
// src/lib/config/tools/adapters/gemini.ts
import { TOOL_SCHEMAS } from '../definitions';

export function toGeminiFormat() {
    return [{
        function_declarations: Object.values(TOOL_SCHEMAS).map(tool => ({
            name: tool.name,
            description: tool.description,
            parameters: {
                type: 'object',
                properties: convertToGeminiProperties(tool.parameters),
                required: getRequiredFields(tool.parameters)
            }
        }))
    }];
}

// KNOWN QUIRK: Gemini Live API struggles with nested arrays
// Use JSON strings for complex nested structures:
// nodes_json: string (JSON.stringify of nodes array)
```

### Tool Handler Pattern

Tool execution logic lives separately from definitions:

```typescript
// src/lib/services/GeminiToolHandler.ts
export class GeminiToolHandler implements IToolHandler {
    async handleToolCall(call: ToolCall): Promise<ToolResponse> {
        switch (call.name) {
            case 'create_education_graph':
                return this.handleCreateGraph(call.args);
            case 'draw_diagram':
                return this.handleDrawDiagram(call.args);
            default:
                return { error: `Unknown tool: ${call.name}` };
        }
    }
}
```

## Testing Principles

### No Network in Unit Tests

**DON'T:** Call real AI APIs in tests
```typescript
// FORBIDDEN in tests
const result = await geminiService.generateContent('test prompt');
```

**DO:** Mock external dependencies
```typescript
const mockGenerator: IContentGenerator = {
    generate: vi.fn().mockResolvedValue('mock response')
};

const service = new ContentService(mockGenerator);
const result = await service.process('test');

expect(mockGenerator.generate).toHaveBeenCalledWith('test');
```

### Test Boundaries

| Layer | What to Test | How to Test |
|-------|--------------|-------------|
| Services | Business logic | Unit tests with mocks |
| Adapters | API integration | Integration tests (manual or CI) |
| Stores | State transitions | Unit tests |
| Components | Rendering | Component tests with testing-library |
| API Routes | Request/response | Integration tests |

### Mocking WebSockets

```typescript
// Mock WebSocket for Gemini realtime tests
class MockWebSocket {
    onopen: (() => void) | null = null;
    onmessage: ((event: { data: string }) => void) | null = null;

    send(data: string) {
        // Record or validate sent messages
    }

    simulateMessage(data: object) {
        this.onmessage?.({ data: JSON.stringify(data) });
    }
}
```

## Dead Code Prevention

### Before Committing

1. **Check for unused exports:**
   ```bash
   # Manual check - search for import of your new export
   grep -r "import.*YourNewExport" src/
   ```

2. **Remove commented code:**
   - Don't commit commented-out code blocks
   - Use git history for "just in case" code

3. **Clean up feature flags:**
   - When a feature is fully rolled out, remove the flag
   - Don't leave dead code paths behind flags

### Periodic Cleanup

Run cleanup analysis periodically:
- See `docs/CLEANUP_PLAN.md` for current cleanup tasks
- Consider tools like `knip` for automated dead code detection

### Export Discipline

**DON'T:** Export everything "just in case"
```typescript
// FORBIDDEN
export const INTERNAL_CONSTANT = 'value';
export function helperUsedOnce() { }
export interface InternalType { }
```

**DO:** Export only public API
```typescript
// Internal (no export)
const INTERNAL_CONSTANT = 'value';
function helperUsedOnce() { }

// Public API (exported)
export function publicFunction() {
    return helperUsedOnce();
}
