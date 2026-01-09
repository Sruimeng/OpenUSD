---
id: doc-standard
type: guide
related_ids: []
---

# Documentation Standard (Doc-Standard)

## Purpose

This guide defines the documentation standards for the `/llmdoc` system. All documentation must follow these rules to ensure consistency, machine-readability, and high information density.

## Core Principles

### 1. Frontmatter Required

Every document MUST begin with YAML frontmatter:

```yaml
---
id: unique-identifier
type: reference | architecture | guide | agent
related_ids: [id1, id2]
---
```

### 2. Type-First Definitions

Define interfaces, types, and data structures BEFORE explaining logic:

```typescript
// GOOD: Type first
interface Config {
  timeout: number;
  retries: number;
}

// Then explain usage
```

### 3. Pseudocode Over Prose

Use pseudocode for logic explanations instead of long paragraphs:

```
// GOOD
FUNCTION processData(input):
  IF input.isValid:
    RETURN transform(input)
  ELSE:
    THROW ValidationError

// BAD
"The processData function first checks if the input is valid.
If it is valid, it transforms the input and returns the result.
Otherwise, it throws a validation error."
```

### 4. Negative Constraints

Explicitly list "DO NOT" rules:

```markdown
## Constraints

- DO NOT use raw pointers for ownership
- DO NOT call blocking functions in async context
- DO NOT modify shared state without locks
```

### 5. Tables Over Lists

Use tables for structured information:

| Component | Purpose | Location |
|-----------|---------|----------|
| Stage | Scene container | pxr/usd/usd/stage.h |
| Prim | Scene object | pxr/usd/usd/prim.h |

### 6. Code References

Include file paths with line numbers:

```
See: pxr/base/gf/matrix4d.h:48-69
```

## Document Types

### Reference (`type: reference`)

- Technical specifications
- API documentation
- Standards and conventions
- "The Constitution"

### Architecture (`type: architecture`)

- System overviews
- Data flow diagrams
- Component relationships
- Critical paths

### Guide (`type: guide`)

- Step-by-step procedures
- How-to instructions
- Best practices

### Agent (`type: agent`)

- Strategy documents
- Investigation reports
- Decision records

## File Naming

- Use kebab-case: `tech-stack.md`, `data-models.md`
- Be descriptive: `constitution.md` not `rules.md`
- No spaces or special characters

## Constraints

- DO NOT write "wall of text" paragraphs
- DO NOT omit frontmatter
- DO NOT use ambiguous language
- DO NOT duplicate information across documents
- DO NOT include time-sensitive information without dates
