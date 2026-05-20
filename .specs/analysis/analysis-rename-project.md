---
title: Codebase Impact Analysis - Rename project to nestjs-decorators
task_file: .specs/tasks/draft/rename-project.refactor.md
scratchpad: .specs/scratchpad/1daab77f.md
created: 2026-05-15
status: complete
---

# Codebase Impact Analysis: Rename project to nestjs-decorators

## Summary

- **Files to Modify**: 9 files
- **Files to Create**: 0 files
- **Files to Delete**: 0 files
- **Test Files Affected**: 2 files
- **Risk Level**: Low-Medium

---

## Files to be Modified/Created

### Primary Changes

```
nestjs-decorators/
├── src/
│   ├── catch.decorator.ts          # FIX: 2 broken imports from 'nestjs-log-decorator'
│   ├── validate.decorator.ts       # FIX: 1 broken import from 'nestjs-log-decorator'; JSDoc comment
│   └── index.ts                    # ADD: export * for catch.decorator and validate.decorator
├── tests/
│   ├── catch.decorator.spec.ts     # FIX: @/ alias -> relative import; jest.fn() -> vi.fn()
│   └── validate.decorator.spec.ts  # FIX: @/ alias -> relative import; jest.fn() -> vi.fn()
└── package.json                    # UPDATE: name, description, URLs, peerDeps, devDeps
```

### Documentation Updates

```
nestjs-decorators/
├── README.md                       # REWRITE: new name, install cmd, all 3 decorator families
├── CLAUDE.md                       # UPDATE: package name, architecture section, clone URL
└── CONTRIBUTING.md                 # UPDATE: package name, clone URL
```

---

## Broken Import Fixes (Critical — Build Blockers)

### src/catch.decorator.ts

| Line | Current (Broken) | Fix |
|------|-----------------|-----|
| 2 | `import type { LogArgsFormatter } from 'nestjs-log-decorator'` | `import type { LogArgsFormatter } from './types'` |
| 4 | `import { createLogWrapper } from 'nestjs-log-decorator'` | `import { createLogWrapper } from './LogWrapper'` |

LogArgsFormatter is exported from src/types.ts:1. createLogWrapper is exported from src/LogWrapper.ts:93.

### src/validate.decorator.ts

| Line | Current (Broken) | Fix |
|------|-----------------|-----|
| 5 | `import { createLogWrapper } from 'nestjs-log-decorator'` | `import { createLogWrapper } from './LogWrapper'` |
| 96 | JSDoc comment references "nestjs-log-decorator" | Update to "./LogWrapper" |

---

## Missing Exports (Critical — Public API Blocker)

### src/index.ts

Current exports: log.decorator, types, LogWrapper, axios/axios.logger, axios/isTimoutError.

Add these two lines:
```typescript
export * from './catch.decorator';
export * from './validate.decorator';
```

This exposes: Catch, CatchConfig, ErrorClassConstructor, ErrorPredicate, Validate, ValidateObject, ValidateConfig, ValidateObjectConfig.

---

## Test File Fixes (Critical — npm test Blocker)

Both test files were written for a project using Jest + a @/ path alias. This project uses Vitest and has NO path aliases in tsconfig.json (confirmed: no "paths" key present).

### tests/catch.decorator.spec.ts

Line 2 - fix import:
- Remove: `import { Catch } from '@/error-handling/catch.decorator'`
- Add: `import { Catch } from '../src/catch.decorator'`

Add Vitest import at top:
```typescript
import { vi, describe, it, expect } from 'vitest'
```

Replace all jest.fn() with vi.fn() — appears on lines 6, 7, 8 (silentLogger setup) and throughout test bodies for handlers.

### tests/validate.decorator.spec.ts

Line 4 - fix import:
- Remove: `import { Validate, ValidateObject } from '@/error-handling/validate.decorator'`
- Add: `import { Validate, ValidateObject } from '../src/validate.decorator'`

Add Vitest import at top:
```typescript
import { vi, describe, it, expect } from 'vitest'
```

Replace all jest.fn() with vi.fn(), including chained calls:
- `jest.fn().mockResolvedValue(...)` -> `vi.fn().mockResolvedValue(...)`
- `jest.fn().mockImplementation(...)` -> `vi.fn().mockImplementation(...)`

---

## Package.json Changes

| Field | Current Value | New Value |
|-------|--------------|-----------|
| name | "nestjs-log-decorator" | "nestjs-decorators" |
| description | "Decorator that removes try catch boilerplate..." | Multi-decorator scope description |
| homepage | "...nestjs-log-decorator#readme" | "...nestjs-decorators#readme" |
| repository.url | "...nestjs-log-decorator" | "...nestjs-decorators" |
| bugs.url | "...nestjs-log-decorator/issues" | "...nestjs-decorators/issues" |
| peerDependencies | { "@nestjs/common": "*" } | Add "class-validator": ">=0.14.0" |
| peerDependenciesMeta | absent | Add class-validator: { optional: true } |
| devDependencies | no class-validator/class-transformer | Add both |

---

## Key Interfaces and Contracts

### Functions with No Signature Changes (import-only fixes)

| Location | Name | Signature |
|----------|------|-----------|
| src/catch.decorator.ts:72 | Catch | function Catch<T,R,V>(config: CatchConfig<T,R,V>): MethodDecorator |
| src/validate.decorator.ts:116 | Validate | function Validate<T,TArgs,TReturn>(config: ValidateConfig<TArgs>): TypedMethodDecorator<TArgs,TReturn> |
| src/validate.decorator.ts:65 | ValidateObject | function ValidateObject<T,TArgs,TReturn>(config: ValidateObjectConfig<TArgs>): TypedMethodDecorator<TArgs,TReturn> |

### Import Targets (no changes needed)

| Location | Export |
|----------|--------|
| src/types.ts:1 | LogArgsFormatter |
| src/LogWrapper.ts:93 | createLogWrapper |

---

## Integration Points

| File | Relationship | Impact | Action Needed |
|------|--------------|--------|---------------|
| src/index.ts | Re-exports all public API | High | Add 2 export lines |
| src/types.ts | Exports LogArgsFormatter for catch.decorator | High | No change; fix the import in catch.decorator |
| src/LogWrapper.ts | Exports createLogWrapper for both new decorators | High | No change; fix imports in catch/validate decorators |
| tests/log.decorator.spec.ts | Vitest reference implementation | Low | No change; use as pattern |
| tests/LogWrapper.spec.ts | Existing passing tests (AC-8) | Low | No change; must stay green |

---

## Similar Implementations

### Vitest test file convention

- Location: tests/log.decorator.spec.ts:1-5
- Why relevant: Canonical pattern this project uses for all test files
- Key pattern: `import { expect, describe, it, vi } from 'vitest'` + `import { ... } from '../src/...'`

---

## Test Coverage

### Tests to Fix

| Test File | Issue | Fix |
|-----------|-------|-----|
| tests/catch.decorator.spec.ts | @/ alias + jest globals | Fix import, add vitest import, replace jest.fn() |
| tests/validate.decorator.spec.ts | @/ alias + jest globals + mockImplementation | Fix import, add vitest import, replace all jest.* |

### Tests to Preserve Unchanged

| Test File | Requirement |
|-----------|-------------|
| tests/log.decorator.spec.ts | AC-8: must pass without modification |
| tests/LogWrapper.spec.ts | AC-8: must pass without modification |

---

## Risk Assessment

| Area | Risk | Mitigation |
|------|------|------------|
| catch.decorator.ts broken imports | Build fails - nestjs-log-decorator not resolvable in this workspace | Fix lines 2 and 4 first |
| validate.decorator.ts broken import | Build fails | Fix line 5 |
| jest globals in test files | ReferenceError: jest is not defined at runtime | Add vitest import, replace all jest.fn() |
| @/ path alias in tests | Module not found at runtime | Replace with ../src/ relative paths |
| class-validator not in devDependencies | Test suite fails (IsString etc. decorators unresolvable) | npm install --save-dev class-validator class-transformer |
| src/index.ts missing exports | Consumer cannot import Catch/Validate | Add two export * lines |

---

## Recommended Exploration

1. src/catch.decorator.ts (full file) - 2 broken import lines at top; all other logic is correct
2. src/validate.decorator.ts (full file) - 1 broken import + JSDoc comment referencing old name at line 96
3. tests/log.decorator.spec.ts lines 1-12 - exact Vitest import pattern to replicate in fixed test files

---

## Verification Summary

| Check | Status | Notes |
|-------|--------|-------|
| All affected files identified | OK | 9 files confirmed by reading each |
| Integration points mapped | OK | types.ts and LogWrapper.ts confirmed as targets |
| Similar patterns found | OK | log.decorator.spec.ts is the Vitest reference |
| Test coverage analyzed | OK | 2 broken; 2 must stay green |
| Risks assessed | OK | All high-risk: import/test-runner mismatches |
| No @/ alias in tsconfig.json | OK | Confirmed: no "paths" key in tsconfig.json |
| Broken imports verified by file read | OK | Confirmed lines 2,4 in catch; line 5 in validate |

Limitations: The nestjs-decorators GitHub repo URL may not exist yet (repo rename is out-of-band per task scope). Update package.json URLs optimistically.
