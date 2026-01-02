# Type Safety Guidelines

## Mandatory Typing Rules

### Strict TypeScript Configuration
The project uses strict TypeScript mode. All files MUST comply:
```json
{
  "strict": true,
  "noImplicitAny": true,
  "strictNullChecks": true,
  "strictFunctionTypes": true
}
```

### Function Signatures
**DO:** Explicitly type all function parameters and return types
```typescript
function createNode(
  id: string,
  type: NodeType,
  position: Position
): EducationNode {
  return { id, type, position, data: { label: '' } };
}

async function fetchCourse(courseId: string): Promise<Course | null> {
  // Implementation
}
```

**DON'T:** Rely on type inference for public APIs
```typescript
// FORBIDDEN
function createNode(id, type, position) {
  return { id, type, position };
}
```

### Variable Declarations
**DO:** Let TypeScript infer simple literal types
```typescript
let count = 0;  // number
let name = 'test';  // string
let isActive = false;  // boolean
```

**DO:** Explicitly type complex structures
```typescript
let node: EducationNode | null = null;
let config: LayoutConfig = createLayoutConfig();
let nodes: EducationNode[] = [];
```

## Forbidden Patterns

### No `any` Type
**FORBIDDEN:** Never use `any` - it defeats type safety
```typescript
// FORBIDDEN
let data: any = fetchData();
function process(input: any): any {}
```

**DO:** Use proper types or `unknown` for truly dynamic data
```typescript
// If type is genuinely unknown
let data: unknown = fetchData();

// Better: define the actual type
interface ApiResponse {
  status: string;
  data: CourseStructure;
}
let data: ApiResponse = fetchData();
```

### No Type Assertions Without Validation
**DON'T:** Use `as` without runtime validation
```typescript
// FORBIDDEN
const data = response as CourseStructure;
```

**DO:** Validate before asserting or use type guards
```typescript
// With Zod validation
const validation = CourseSchema.safeParse(response);
if (validation.success) {
  const data: CourseStructure = validation.data;
}

// With type guard
if (isEducationNode(data)) {
  const node: EducationNode = data;
}
```

### No Non-Null Assertions Without Justification
**DON'T:** Use `!` operator carelessly
```typescript
// FORBIDDEN without proof
const element = document.getElementById('id')!;
const value = map.get(key)!;
```

**DO:** Check for null/undefined or document why it's safe
```typescript
const element = document.getElementById('id');
if (!element) {
  throw new Error('Element not found');
}

// If guaranteed by business logic, comment why
const value = map.get(key)!; // Safe: key validated above
```

## Type Import Conventions

### Import Type Syntax
**DO:** Use `import type` for type-only imports
```typescript
import type { CourseStructure, Module, Lesson } from '$lib/types';
import type { RequestHandler } from './$types';
```

**DON'T:** Mix value and type imports unnecessarily
```typescript
// FORBIDDEN - use import type
import { CourseStructure } from '$lib/types';
```

### Re-export Pattern
**DO:** Use centralized type exports from `$lib/types/index.ts`
```typescript
// In component
import type { EducationNode, EducationEdge } from '$lib/types';

// NOT from individual files
import type { EducationNode } from '$lib/types/canvas';
```

## Generic Usage Guidelines

### Generic Function Pattern
**DO:** Use generics for reusable, type-safe functions
```typescript
function validateInput<T>(
  schema: z.ZodSchema<T>,
  input: unknown
): ValidationResult<T> {
  const result = schema.safeParse(input);
  if (result.success) {
    return { success: true, data: result.data };
  }
  return { success: false, error: result.error.message };
}

// Usage with full type safety
const result = validateInput(CourseSchema, rawData);
if (result.success) {
  const course: Course = result.data; // Fully typed
}
```

### Generic Constraints
**DO:** Constrain generics when needed
```typescript
function findById<T extends { id: string }>(
  items: T[],
  id: string
): T | undefined {
  return items.find(item => item.id === id);
}
```

### Avoid Over-Generalization
**DON'T:** Use generics when concrete types suffice
```typescript
// FORBIDDEN - unnecessary generic
function addNode<T>(node: T): void {}

// Better - specific type
function addNode(node: EducationNode): void {}
```

## Zod Schema Patterns

### Schema Definition
**DO:** Define Zod schemas alongside TypeScript types
```typescript
export const NodeSchema = z.object({
  id: z.string().min(1),
  label: z.string().min(1).max(100),
  type: NodeTypeSchema,
  data: z.object({
    description: z.string().optional()
  }).optional()
});

export type GraphNode = z.infer<typeof NodeSchema>;
```

### Schema Validation
**DO:** Use `safeParse` for validation with error handling
```typescript
const result = GraphGenerationSchema.safeParse(input);

if (result.success) {
  const graph: GraphGeneration = result.data;
  // Process valid data
} else {
  console.error('Validation failed:', result.error);
  return validationError(result.error.message);
}
```

**DON'T:** Use `parse` which throws exceptions
```typescript
// FORBIDDEN in API routes
const graph = GraphGenerationSchema.parse(input); // Throws!
```

### Schema Composition
**DO:** Compose schemas for reusability
```typescript
const PositionSchema = z.object({
  x: z.number(),
  y: z.number()
});

const NodeSchema = z.object({
  id: z.string(),
  position: PositionSchema,  // Reuse
  // ...
});
```

## Type Guards

### Custom Type Guards
**DO:** Create type guards for runtime type checking
```typescript
export function isEducationNode(node: unknown): node is EducationNode {
  return (
    typeof node === 'object' &&
    node !== null &&
    'id' in node &&
    'type' in node &&
    'position' in node &&
    'data' in node
  );
}

// Usage
if (isEducationNode(data)) {
  const node: EducationNode = data; // Type narrowed
}
```

### Array Type Guards
**DO:** Validate array contents
```typescript
function isNodeArray(arr: unknown): arr is EducationNode[] {
  return Array.isArray(arr) && arr.every(isEducationNode);
}
```

## Utility Types

### Use Built-in Utility Types
**DO:** Leverage TypeScript utility types
```typescript
// Partial - make all properties optional
type PartialNode = Partial<EducationNode>;

// Pick - select specific properties
type NodePosition = Pick<EducationNode, 'id' | 'position'>;

// Omit - exclude properties
type NodeWithoutData = Omit<EducationNode, 'data'>;

// Record - type-safe key-value pairs
type NodeMap = Record<string, EducationNode>;
```

### Readonly Types
**DO:** Use `readonly` for immutable data
```typescript
interface CanvasAction {
  readonly type: string;
  readonly payload: unknown;
}

type ReadonlyNodes = readonly EducationNode[];
```
