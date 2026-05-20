---
title: Rename project to nestjs-decorators
---

## Initial User Prompt

rename project to nestjs-decorators

### Context

Project was previusly named as nestjs-log-decorator, it is now will be renamed to nestjs-decorators and republished with new name. It will contain new catch and validation decorators. That was copied from internal project to this one.

### Requirements

-  update catch.decorator.ts and validate.decorator.ts to use proper import structure and fix tests for them
- add class-validator as transitive dependency
- update README.md to reflect new name, new installation instructions and include new decorators. Update description, usage examples and etc.
- update package.json to reflect new name, description, repository url and etc.
- update CLAUDE.md, CONTRIBUTING.md and other documentation files 
- fix tests if there any issues left after refactoring

> **Required Skill**: You MUST use and analyse `nestjs-decorators-rename` skill before doing any modification to task file or starting implementation of it!
>
> Skill location: `.claude/skills/nestjs-decorators-rename/SKILL.md`

## Description

The repository currently ships under the npm name `nestjs-log-decorator`, but it has expanded beyond a single logging decorator: two new decorators (`@Catch` for error handling and `@Validate`/`@ValidateObject` for class-validator-based input validation) have already been copied into `src/` from an internal project. The current name misrepresents the library's scope, hurts discoverability for the new features, and constrains future growth. This task renames the package to `nestjs-decorators` and turns it into a properly integrated multi-decorator toolkit.

The rename is not a simple string replacement. The newly copied `src/catch.decorator.ts` and `src/validate.decorator.ts` currently import internal helpers (`createLogWrapper`, `LogArgsFormatter`) from the external `'nestjs-log-decorator'` package name, which does not resolve inside this workspace. Their accompanying tests import from a non-existent `@/error-handling/...` path alias, and `tests/catch.decorator.spec.ts` uses `jest.fn()` while the project's test runner is Vitest. As part of this work, all internal imports must be re-pointed to relative paths inside `src/`, the test files must be adapted to Vitest globals (`vi.fn()` etc.) and to relative paths under `tests/`, `class-validator` must be declared as a peer dependency (its `ValidationError` and `ValidatorOptions` types are exposed in the public `ValidateObjectConfig` interface), `class-transformer` must be added as a dev dependency (used by the nested-validation test scenarios), and `src/index.ts` must export the new decorator modules.

The user-facing surface - `README.md`, `CLAUDE.md`, `CONTRIBUTING.md`, and `package.json` metadata - must be rewritten to describe the broader purpose and document all three decorator families with working code samples under the new package name. The runtime behavior of `@Log()` and `@NoLog()` must remain identical: an existing consumer who only switches their import string from `'nestjs-log-decorator'` to `'nestjs-decorators'` should see no behavioral change. Consumers who use only `@Log()` must not be forced to install `class-validator` at runtime.

**Scope**:

- Included:
  - Renaming the npm package and all metadata in `package.json` (`name`, `description`, `homepage`, `repository.url`, `bugs.url`, keywords as appropriate).
  - Wiring `@Catch`, `@Validate`, and `@ValidateObject` into the public API by adding `export * from './catch.decorator'` and `export * from './validate.decorator'` to `src/index.ts`.
  - Replacing the broken `from 'nestjs-log-decorator'` imports in `src/catch.decorator.ts` and `src/validate.decorator.ts` with relative imports (`LogArgsFormatter` from `./types`, `createLogWrapper` from `./LogWrapper`).
  - Adapting `tests/catch.decorator.spec.ts` and `tests/validate.decorator.spec.ts` to (a) import the decorators via `../src/catch.decorator` and `../src/validate.decorator` instead of the broken `@/error-handling/...` alias, and (b) replace `jest.fn()` with `vi.fn()` plus the required `vitest` named imports, mirroring the convention already used by `tests/log.decorator.spec.ts`.
  - Declaring `class-validator` (range `>=0.14.0`) in `peerDependencies` alongside `@nestjs/common`, and adding `class-validator` and `class-transformer` to `devDependencies`.
  - Rewriting `README.md` to describe the multi-decorator toolkit, present the new install command (`npm install nestjs-decorators @nestjs/common`), note when `class-validator` is additionally required, and document `@Log`, `@Catch`, and `@Validate`/`@ValidateObject` each with at least one working code sample.
  - Updating `CLAUDE.md` and `CONTRIBUTING.md` to reflect the new package name, the broader purpose, and the new clone URL.
  - Ensuring `npm run build`, `npm run lint`, and `npm test` all exit cleanly on a fresh checkout.
- Excluded:
  - Designing or adding any new decorator families beyond the three already present in `src/`.
  - Deprecating the legacy `nestjs-log-decorator` package on npm (administrative follow-up).
  - Authoring a migration guide for existing `nestjs-log-decorator` consumers.
  - Renaming the GitHub repository itself (assumed to be handled out-of-band; the code change just updates URLs to the target name).
  - Resetting the package version to `1.0.0` (semantic-release continues to derive the next version from Conventional Commits).
  - Changing the runtime behavior, options, or log-output format of `@Log()` or `@NoLog()`.

**User Scenarios**:

1. **Primary Flow**: A new developer installs `nestjs-decorators @nestjs/common`, imports `@Log`, `@Catch`, or `@Validate`/`@ValidateObject` from the single package entry point, and uses each decorator exactly as documented in the new `README.md`.
2. **Alternative Flow**: An existing `nestjs-log-decorator` consumer updates only the package name in their `package.json` and their import strings; their `@Log()`-decorated services continue to behave identically with no other code changes required.
3. **Error Handling**: A consumer who attempts to use `@ValidateObject` without installing `class-validator` receives a clear module-not-found error pointing at `class-validator`, and the README's validation section directs them to install it.

---

## Acceptance Criteria

### Functional Requirements

- [X] **AC-1: Package metadata is fully renamed**
  - Given: The repository at HEAD after the change is applied
  - When: A reader opens `package.json`
  - Then: The `name` field equals `"nestjs-decorators"`, the `description` field reflects the multi-decorator scope (no longer "try catch boilerplate logging"), and `homepage`, `repository.url`, and `bugs.url` all reference `nestjs-decorators` instead of `nestjs-log-decorator`.

- [X] **AC-2: All decorators are exported from a single public entry point**
  - Given: A consumer project that has installed the built package
  - When: They write `import { Log, NoLog, Catch, Validate, ValidateObject } from 'nestjs-decorators'`
  - Then: All five symbols resolve, type-check, and are usable as decorators or decorator factories at runtime. `src/index.ts` contains explicit re-exports for `./catch.decorator` and `./validate.decorator`.

- [X] **AC-3: New decorator source files use internal relative imports**
  - Given: The current contents of `src/catch.decorator.ts` and `src/validate.decorator.ts`
  - When: A search for `from 'nestjs-log-decorator'` is run across those two files
  - Then: Zero matches are returned. `LogArgsFormatter` is imported from `./types`, and `createLogWrapper` is imported from `./LogWrapper`.

- [X] **AC-4: New decorator tests run under the project's test runner**
  - Given: A clean checkout with dependencies installed
  - When: `npm test` is executed
  - Then: `tests/catch.decorator.spec.ts` and `tests/validate.decorator.spec.ts` both execute and pass; neither file imports from the `@/error-handling/...` alias (they use `../src/...` relative paths); `tests/catch.decorator.spec.ts` contains no references to `jest.fn` / `jest.mock` (it uses `vi.fn()` and named imports from `vitest`); the full test suite reports zero failures.

- [X] **AC-5: Peer and dev dependencies are declared correctly**
  - Given: The renamed `package.json`
  - When: A consumer project that does NOT have `class-validator` installed runs `npm install nestjs-decorators @nestjs/common`
  - Then: The install completes (peer-dependency warnings are acceptable; installation failure is not), and `import { Log } from 'nestjs-decorators'` works at runtime in that consumer project without `class-validator` being present. `package.json` lists `class-validator` (range `>=0.14.0`) under `peerDependencies`, and both `class-validator` and `class-transformer` appear under `devDependencies`.

- [X] **AC-6: README documents the full toolkit under the new name**
  - Given: A new reader who opens `README.md` at HEAD
  - When: They scan the document for installation instructions and usage examples
  - Then: The install command shows `npm install nestjs-decorators @nestjs/common` (with `class-validator` mentioned as an additional install when using validation); the document includes at least one runnable code sample for each of `@Log`, `@Catch`, and `@Validate`/`@ValidateObject`; every code sample imports from `'nestjs-decorators'`.

- [X] **AC-7: No residual references to the old name remain**
  - Given: The repository at HEAD
  - When: A search for the literal string `nestjs-log-decorator` is run across `src/`, `tests/`, `README.md`, `CLAUDE.md`, `CONTRIBUTING.md`, and `package.json`
  - Then: Zero matches are returned (`package-lock.json` and git history are exempt).

- [X] **AC-8: `@Log()` backward-compatible behavior is preserved**
  - Given: The existing tests in `tests/log.decorator.spec.ts` and `tests/LogWrapper.spec.ts`
  - When: `npm test` is executed after the rename
  - Then: All pre-existing `@Log()` / `@NoLog()` / `LogWrapper` behavior assertions pass with no test logic modified, demonstrating identical runtime behavior, log fields, and log format.

- [X] **AC-9: Architecture and contributor documentation reflect the new scope**
  - Given: A contributor opens `CLAUDE.md` and `CONTRIBUTING.md` at HEAD
  - When: They read the "Architecture" / "Quick Start" sections
  - Then: `CLAUDE.md` describes the project as a multi-decorator library and names all three decorator families (`@Log`, `@Catch`, `@Validate`) with their respective source files; `CONTRIBUTING.md` references the new package name and the updated clone URL.

### Non-Functional Requirements

- [X] **NFR-1: Backward compatibility** - The runtime behavior of `@Log()` and `@NoLog()` is unchanged (same options, same log output structure, same auto-injection of NestJS Logger).
- [X] **NFR-2: Optional peer dependencies** - A consumer who installs `nestjs-decorators` and uses only `@Log` is not required to have `class-validator` installed at runtime.
- [X] **NFR-3: Build artifact integrity** - The compiled `dist/index.cjs` does not embed `class-validator`, `class-transformer`, `@nestjs/common`, or `axios`; all four remain external at runtime.
- [X] **NFR-4: Documentation accuracy** - 100% of code samples in `README.md` reference `from 'nestjs-decorators'`, and every documented feature exists in `src/`.

### Definition of Done

- [X] All Functional Requirements (AC-1 through AC-9) pass.
- [X] All Non-Functional Requirements (NFR-1 through NFR-4) pass.
- [X] `npm run build` exits with status 0 and produces `dist/index.cjs` plus `dist/index.d.cts` containing declarations for `Log`, `NoLog`, `Catch`, `Validate`, and `ValidateObject`.
- [X] `npm run lint` (which runs `tsc --noEmit`) exits with status 0.
- [X] `npm test` exits with status 0 with all spec files executing.
- [X] No file inside `src/`, `tests/`, `README.md`, `CLAUDE.md`, `CONTRIBUTING.md`, or `package.json` contains the literal string `nestjs-log-decorator`.
- [ ] Conventional Commit message used for the change (e.g., `feat!: rename package to nestjs-decorators and integrate catch/validate decorators`) so semantic-release can compute the next version.

---

## Architecture

### References

- **Skill**: `.claude/skills/nestjs-decorators-rename/SKILL.md`
- **Codebase Analysis**: `.specs/analysis/analysis-rename-project.md`
- **Scratchpad**: `.specs/scratchpad/ba418b14.md`

### Solution Strategy

**Architecture Pattern**: **Layered** (Public API barrel -> Decorator Use Cases -> Shared Logging Primitives -> External Frameworks). The codebase already exhibits this layering: `src/index.ts:1-5` is the barrel layer, `src/log.decorator.ts` is a Layer 2 use case, `src/LogWrapper.ts:3` is a Layer 3 primitive importing only inward to `./axios/axios.logger`. The rename does NOT introduce new architecture — it slots `@Catch` and `@Validate`/`@ValidateObject` into the existing layered structure as additional Layer 2 use cases.

**Approach**: Mechanical, in-place rename. Keep the existing flat module layout under `src/` and the existing `export *` barrel pattern in `src/index.ts`. Treat the work as orthogonal mechanical fixes that share a single commit: (1) replace broken `'nestjs-log-decorator'` self-imports with relative paths, (2) extend the public-API barrel with two new `export *` lines, (3) declare `class-validator` as an optional peer dependency and add `class-transformer` as a dev dep, (4) rename package metadata, (5) update Vitest test conventions in the two broken specs, (6) update user-facing and contributor docs. The existing layered structure absorbs `@Catch` and `@Validate` without modification.

**Key Decisions**:

1. **In-place fix over restructuring** — the existing `src/<verb>.decorator.ts` convention is already correct; `catch.decorator.ts` and `validate.decorator.ts` only have broken imports, not broken structure.
2. **`peerDependenciesMeta.optional: true` for `class-validator`** — the public `ValidateObjectConfig` interface exposes `ValidationError` and `ValidatorOptions` types from class-validator, but `@Log`-only consumers must not be required to install it (NFR-2). Optional peer dep is the standard npm idiom for this case.
3. **Extend the barrel, do not redesign it** — `src/index.ts` already uses five `export *` lines; adding two more preserves the established pattern and satisfies AC-2 with one-line edits.
4. **Relative imports in tests, no `tsconfig` changes** — `tsconfig.json` has no `paths` or `baseUrl`; all existing tests use `../src/...`. Adding a `@/` alias would diverge from project convention.
5. **Preserve `@Log` runtime behavior exactly** — no edits to `log.decorator.ts`, `types.ts`, or `LogWrapper.ts`; `tests/log.decorator.spec.ts` and `tests/LogWrapper.spec.ts` remain untouched (AC-8 / NFR-1).

**Trade-offs Accepted**:

- A consumer who installs `nestjs-decorators` and uses `@ValidateObject` without installing `class-validator` will see a module-resolution error at runtime. This is documented in the README's validation section (User Scenario 3). The alternative (dynamic import) would complicate the public TypeScript types unnecessarily.
- Keywords in `package.json` are extended (not replaced) — older keyword-based discoverability is preserved alongside the new ones.

### Architecture Decomposition

**Components**:

| Component | Layer | Responsibility | Dependencies |
|-----------|-------|----------------|--------------|
| `src/index.ts` | Public API barrel | Re-export every public symbol | `./log.decorator`, `./types`, `./LogWrapper`, `./axios/*`, **NEW: `./catch.decorator`, `./validate.decorator`** |
| `src/catch.decorator.ts` | Use Case | `@Catch` decorator factory | `base-decorators`, **NEW: `./types`, `./LogWrapper`** |
| `src/validate.decorator.ts` | Use Case | `@Validate` / `@ValidateObject` decorator factories | `base-decorators`, `class-validator`, **NEW: `./LogWrapper`** |
| `src/log.decorator.ts` | Use Case | `@Log` / `@NoLog` (unchanged) | `base-decorators`, `./LogWrapper`, `./types` |
| `src/LogWrapper.ts` | Shared logging primitive | `LogWrapper`, `createLogWrapper`, `isLoggable` (unchanged) | `@nestjs/common`, `./axios/axios.logger` |
| `src/types.ts` | Shared types (unchanged) | `LogArgsFormatter`, `LogOptions`, `NO_LOG_METADATA_KEY` | — |
| `tests/catch.decorator.spec.ts` | Test | Behavioral assertions for `@Catch` | `vitest`, `@nestjs/common`, **NEW: `../src/catch.decorator`** |
| `tests/validate.decorator.spec.ts` | Test | Behavioral assertions for `@Validate`/`@ValidateObject` | `vitest`, `@nestjs/common`, `class-validator`, `class-transformer`, **NEW: `../src/validate.decorator`** |
| `package.json` | Metadata | Name, deps, URLs, peer-dep declarations | — |
| `README.md` | User docs | Multi-decorator install + usage examples | — |
| `CLAUDE.md` | Internal docs | Architecture overview for AI assistants | — |
| `CONTRIBUTING.md` | Contributor docs | Clone URL + commit conventions | — |

**Interactions**:

```
       Consumer Application
             |
             v import { Log, Catch, Validate, ValidateObject } from 'nestjs-decorators'
       +-------------------------+
       |   src/index.ts (barrel) |  <-- Public API (Layer 1)
       +----+------+---------+---+
            |      |         |
            v      v         v
   +----------+ +---------+ +--------------+
   |   log.   | |  catch. | |  validate.   |  <-- Decorator Use Cases (Layer 2)
   | decorator| |decorator| |   decorator  |
   +----+-----+ +----+----+ +------+-------+
        |            |             |
        |   +--------v------+      |
        +-->|  LogWrapper +  |<----+
            |  types         |       <-- Shared Logging Primitives (Layer 3)
            +-------+--------+
                    |
                    v
            +----------------+
            | axios.logger / |       <-- Adapters
            | isTimoutError  |
            +----------------+
                    |
                    v
            @nestjs/common, base-decorators, (class-validator only for ValidateObject)
                                            <-- External Frameworks (Layer 4)
```

Dependency direction is strictly inward: tests and decorators may import from primitives; primitives never import from decorators.

### Expected Changes

```
nestjs-decorators/
|-- src/
|   |-- catch.decorator.ts          # UPDATE: lines 2 & 4 - replace 'nestjs-log-decorator' with './types' and './LogWrapper'
|   |-- validate.decorator.ts       # UPDATE: line 5 - replace 'nestjs-log-decorator' with './LogWrapper'; line 96 JSDoc cleanup
|   `-- index.ts                    # UPDATE: append `export * from './catch.decorator'` and `export * from './validate.decorator'`
|-- tests/
|   |-- catch.decorator.spec.ts     # UPDATE: line 2 import -> '../src/catch.decorator'; add `import { vi, describe, it, expect } from 'vitest'`; replace all `jest.fn()` with `vi.fn()`
|   `-- validate.decorator.spec.ts  # UPDATE: line 4 import -> '../src/validate.decorator'; add `import { vi, describe, it, expect } from 'vitest'`; replace all `jest.fn()` with `vi.fn()` (incl. chained .mockResolvedValue / .mockImplementation)
|-- package.json                    # UPDATE: name -> 'nestjs-decorators'; description, homepage, repository.url, bugs.url; add class-validator to peerDependencies (>=0.14.0) + peerDependenciesMeta.optional: true; add class-validator and class-transformer to devDependencies; extend keywords
|-- README.md                       # REWRITE: new title, new install command, sections for @Log, @Catch, @Validate/@ValidateObject; every code sample imports from 'nestjs-decorators'
|-- CLAUDE.md                       # UPDATE: Purpose line (multi-decorator), Architecture section to include catch.decorator.ts and validate.decorator.ts entry points; replace 'nestjs-log-decorator' references
`-- CONTRIBUTING.md                 # UPDATE: clone URL -> nestjs-decorators repo
```

### Workflow Steps

```
Phase 1: Fix self-imports           Phase 2: Extend barrel             Phase 3: Update deps
  src/catch.decorator.ts:2,4   -->  src/index.ts adds 2 exports   -->  package.json peerDeps + devDeps
  src/validate.decorator.ts:5,96                                       npm install
        |                                  |                                  |
        v                                  v                                  v
Phase 4: Fix tests                  Phase 5: Rename metadata           Phase 6: Update docs
  tests/catch.spec.ts          -->  package.json name/URLs/desc   -->  README, CLAUDE.md, CONTRIBUTING.md
  tests/validate.spec.ts                                                rewrite samples for new name
        |                                  |                                  |
        +----------------------------------+----------------------------------+
                                           |
                                           v
                              Phase 7: Verify (npm lint / test / build)
                                  + grep for 'nestjs-log-decorator' = 0 matches
```

Sequence rationale (no forward dependencies):

- **Phase 1 first** — `catch.decorator.ts` and `validate.decorator.ts` cannot resolve `from 'nestjs-log-decorator'` because Node won't resolve a package to itself. Fixing imports is independent of the rename and unblocks everything else.
- **Phase 2** depends on Phase 1 — extending the barrel before fixing imports would re-export broken modules.
- **Phase 3** depends on Phase 1 — adding `class-validator` to devDeps and running `npm install` only makes sense once the source compiles.
- **Phase 4** depends on Phases 1+2+3 — tests import from fixed source paths and require `class-validator` / `class-transformer` to be installed.
- **Phase 5** depends on Phase 1 — `package.json#name` is changed only AFTER source no longer self-references the old name, avoiding any state where both are wrong.
- **Phase 6** depends on Phase 5 — docs must reference the final name and final exports.
- **Phase 7** depends on all — final verification.

### Architecture Decisions

#### AD-1: Keep flat `src/` layout instead of grouping decorators in subdirectories

**Status**: Accepted

**Context**: The repo currently uses `src/<verb>.decorator.ts` files at the root of `src/`. The new `@Catch` and `@Validate` files already follow this convention. A subdirectory regroup (`src/decorators/*.ts`) was considered.

**Options**:

1. Keep flat layout — `src/<verb>.decorator.ts`.
2. Group under `src/decorators/<verb>.ts` with a sub-barrel.
3. Group by concern (`src/logging/`, `src/error-handling/`, `src/validation/`).

**Decision**: Option 1. Keep the flat layout. Files stay at `src/catch.decorator.ts` and `src/validate.decorator.ts`.

**Consequences**:

- Zero churn beyond the import-fix.
- Consistent with the existing `log.decorator.ts` placement.
- If the toolkit grows past ~6 decorators, a future, separate task can introduce a subdirectory split.

#### AD-2: Declare `class-validator` as an optional peer dependency

**Status**: Accepted

**Context**: `validate.decorator.ts` imports `validate`, `ValidationError`, and `ValidatorOptions` from `class-validator` at the module top level. These types are re-exposed in the public `ValidateObjectConfig` interface. Consumers using only `@Log` should not be forced to install `class-validator`.

**Options**:

1. Hard runtime dependency (`dependencies`) — forces every consumer to ship class-validator.
2. Optional peer dep (`peerDependencies` + `peerDependenciesMeta.optional: true`) — install-time optional, runtime required only when actually used.
3. Dynamic `await import('class-validator')` inside `ValidateObject`.
4. Bundle a vendored copy of class-validator.

**Decision**: Option 2. Add `"class-validator": ">=0.14.0"` to `peerDependencies`, add `peerDependenciesMeta: { "class-validator": { "optional": true } }`, and add `class-validator: ^0.15.1` plus `class-transformer: ^0.5.1` to `devDependencies` for the test suite.

**Consequences**:

- `npm install nestjs-decorators` completes without errors when `class-validator` is absent.
- A consumer using `@ValidateObject` without `class-validator` installed will see a clear module-not-found error at runtime; README directs them to install it (User Scenario 3).
- `dist/index.cjs` keeps `class-validator` external (NFR-3 satisfied by tsdown's default external-for-peer-deps behavior).

#### AD-3: Use Vitest globals + relative imports in test files (no `paths` alias)

**Status**: Accepted

**Context**: The two new test files use `jest.fn()` and the `@/error-handling/...` alias, neither of which is supported by the project's Vitest setup or `tsconfig.json` (no `paths` entry).

**Options**:

1. Replace `jest.fn()` with `vi.fn()` and use `../src/...` relative paths (matches `tests/log.decorator.spec.ts`).
2. Configure Vitest to alias `jest` to `vi` and add a `paths: { "@/*": ["./src/*"] }` entry to `tsconfig.json`.
3. Switch the project to Jest.

**Decision**: Option 1. Replace `jest.fn()` with `vi.fn()` (named import from `vitest`) and replace `@/error-handling/<file>` with `../src/<file>`. Do NOT add a `paths` entry to `tsconfig.json`.

**Consequences**:

- Test files match the canonical pattern in `tests/log.decorator.spec.ts`.
- No build-tool configuration changes; `tsdown` and `vitest` continue to work with stock settings.

### Contracts

**Public API Contract** (the symbols `nestjs-decorators` exports after this task):

```
// Decorators
Log(options?: LogOptions): ClassDecorator & MethodDecorator
NoLog(): MethodDecorator
Catch<T extends Error, R, V extends unknown[]>(config: CatchConfig<T, R, V>): MethodDecorator
Validate<T, TArgs, TReturn>(config: ValidateConfig<TArgs>): TypedMethodDecorator<TArgs, TReturn>
ValidateObject<T, TArgs, TReturn>(config: ValidateObjectConfig<TArgs>): TypedMethodDecorator<TArgs, TReturn>

// Types
LogOptions
LogArgsFormatter
LogResultFormatter
Loggable
CatchConfig
ErrorClassConstructor
ErrorPredicate
ValidateConfig
ValidateObjectConfig

// Utilities (unchanged)
isLoggable
LogWrapper
createLogWrapper
NO_LOG_METADATA_KEY
```

**Dependency Contract** (`package.json` after this task):

```
dependencies:
  base-decorators ^0.1.1

peerDependencies:
  @nestjs/common  *
  class-validator >=0.14.0

peerDependenciesMeta:
  class-validator: { optional: true }

devDependencies (additions):
  class-validator   ^0.15.1
  class-transformer ^0.5.1
```

---

## Implementation Process

You MUST launch for each step a separate agent, instead of performing all steps yourself. And for each step marked as parallel, you MUST launch separate agents in parallel.

**CRITICAL:** For each agent you MUST:
1. Use the **Agent** type specified in the step (e.g., `sdd:developer`, `opus`, `sdd:qa-engineer`)
2. Provide path to task file and prompt which step to implement
3. Require agent to implement exactly that step, not more, not less, not other steps

### Parallelization Overview

```
                START
                  |
        +---------+---------+
        |                   |
        v                   v
    Step 1               Step 3
 (Fix imports)    (Declare deps in pkg.json)
[sdd:developer]        [opus]
 1a + 1b parallel
        |                   |
        v                   |
    Step 2                  |
(Extend barrel)             |
[sdd:developer]             |
        |                   |
        +---------+---------+
                  |
                  v
              Step 4
   (npm install + Fix tests)
        [sdd:developer]
       4a + 4b parallel
                  |
                  v
              Step 5
        (Rename metadata)
             [opus]
                  |
        +---------+---------+
        |         |         |
        v         v         v
     Step 6    Step 7    Step 8
    (README) (CLAUDE.md) (CONTRIBUTING.md)
    [opus]    [opus]     [opus]
        |         |         |
        +---------+---------+
                  |
                  v
              Step 9
       (Final Verification)
       [sdd:qa-engineer]
                  |
                 END
```

**Critical Path:** Step 1 -> Step 2 -> Step 4 -> Step 5 -> Step 6 -> Step 9

**Parallel Opportunities:**
- Steps 1 and 3 MUST be executed in parallel at the start (both have no dependencies)
- Steps 6, 7, and 8 MUST be executed in parallel after Step 5 lands
- Within Step 1: sub-task edits to `src/catch.decorator.ts` and `src/validate.decorator.ts` MUST be performed in parallel
- Within Step 4: sub-task edits to `tests/catch.decorator.spec.ts` and `tests/validate.decorator.spec.ts` MUST be performed in parallel after `npm install`

### Implementation Strategy

**Approach**: Bottom-Up (Building-Blocks-First)

**Rationale**: The "building blocks" are the broken source files. Until they compile, nothing else (barrel, tests, build, docs) can succeed. We fix the lowest-level units first (source imports), then build outward (barrel exports -> deps -> tests -> metadata -> docs -> verify). Top-down was rejected because renaming metadata before fixing source self-imports would create a temporarily broken state the architecture sequence rationale explicitly forbids (Phase 5 must follow Phase 1), and docs written before exports exist would document a non-existent API.

### Phase Overview

```
Phase 1: Foundational (Source self-import fixes)
    |
    v
Phase 2: Implementation A (Barrel exports + Dependency declarations)
    |
    v
Phase 3: Implementation B (npm install + Test file fixes)
    |
    v
Phase 4: Implementation C (Package metadata rename)
    |
    v
Phase 5: Documentation (README, CLAUDE.md, CONTRIBUTING.md)
    |
    v
Phase 6: Verification (Build / Lint / Test / Grep)
```

---

### Step 1: Fix Broken Self-Imports in New Decorator Source Files [DONE]

**Model:** opus
**Agent:** sdd:developer
**Depends on:** None
**Parallel with:** Step 3
**Note:** Individual source file edits (1a for `src/catch.decorator.ts` and 1b for `src/validate.decorator.ts`) MUST be performed in parallel by multiple agents — the two files are independent and do not share imports.

**Goal**: Make `src/catch.decorator.ts` and `src/validate.decorator.ts` resolve their internal dependencies via relative paths so they can be imported by the barrel and by tests.

| Sub-task | Description | Agent | Can Parallel |
|----------|-------------|-------|--------------|
| 1a | Fix imports in `src/catch.decorator.ts` (lines 2 and 4) | sdd:developer | Yes |
| 1b | Fix imports in `src/validate.decorator.ts` (line 5 and JSDoc near line 96) | sdd:developer | Yes |

#### Expected Output

- `src/catch.decorator.ts` line 2: `import type { LogArgsFormatter } from './types'` (was `'nestjs-log-decorator'`)
- `src/catch.decorator.ts` line 4: `import { createLogWrapper } from './LogWrapper'` (was `'nestjs-log-decorator'`)
- `src/validate.decorator.ts` line 5: `import { createLogWrapper } from './LogWrapper'` (was `'nestjs-log-decorator'`)
- `src/validate.decorator.ts` JSDoc at line ~96: `nestjs-log-decorator` reference replaced with `./LogWrapper`

#### Success Criteria

- [X] grep `from 'nestjs-log-decorator'` in `src/catch.decorator.ts` returns 0 matches
- [X] grep `from 'nestjs-log-decorator'` in `src/validate.decorator.ts` returns 0 matches
- [X] grep `nestjs-log-decorator` in `src/validate.decorator.ts` returns 0 matches (covers JSDoc)
- [X] `npm run lint` exits 0 (typecheck passes for these source files)

#### Verification

**Level:** ✅ Per-Decorator-File Judges (2 separate evaluations in parallel)
**Artifacts:** `src/{catch.decorator,validate.decorator}.ts`
**Threshold:** 4.0/5.0

**Rubric (per source file):**

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Import Path Correctness | 0.35 | Imported symbols (`LogArgsFormatter`, `createLogWrapper`) resolve from the correct relative module (`./types`, `./LogWrapper`). |
| Behavior Preserved | 0.25 | No functional changes to the decorator body; only import statements (and JSDoc) modified. |
| Residual Old-Name References | 0.20 | Zero occurrences of `'nestjs-log-decorator'` string in the file, including JSDoc/comments. |
| Type Safety | 0.15 | TypeScript typecheck (`npm run lint`) passes for this file; correct use of `import type` where applicable. |
| Code Quality | 0.05 | Imports follow project conventions (ordering, naming). |

**Reference Pattern:** `src/log.decorator.ts` (canonical relative-import convention in `src/`)

#### Subtasks

- [X] Edit `src/catch.decorator.ts` line 2: replace `'nestjs-log-decorator'` with `'./types'`
- [X] Edit `src/catch.decorator.ts` line 4: replace `'nestjs-log-decorator'` with `'./LogWrapper'`
- [X] Edit `src/validate.decorator.ts` line 5: replace `'nestjs-log-decorator'` with `'./LogWrapper'`
- [X] Edit `src/validate.decorator.ts` JSDoc near line 96: replace `nestjs-log-decorator` with `./LogWrapper`
- [X] Run `npm run lint` to confirm typecheck passes

#### Blockers

- None.

#### Risks

| Risk | Mitigation |
|------|------------|
| Wrong relative target path chosen | Analysis confirms `LogArgsFormatter` is in `src/types.ts` and `createLogWrapper` in `src/LogWrapper.ts:93`. |
| JSDoc line number drifts | Edit by exact string match, not line number. |

#### Complexity

Small.

#### Dependencies

None.

#### Uncertainty Rating

Low.

#### Integration Points

- `src/types.ts` (re-exports `LogArgsFormatter`)
- `src/LogWrapper.ts` (re-exports `createLogWrapper`)

#### Definition of Done

- [X] All four edits applied
- [X] grep verification passes (zero `nestjs-log-decorator` in these two files)
- [X] Source files type-check via `npm run lint`

---

### Step 2: Extend Public API Barrel in src/index.ts [DONE]

**Model:** opus
**Agent:** sdd:developer
**Depends on:** Step 1
**Parallel with:** None (Step 3 may still be executing in parallel from start, but Step 2 has no shared dependency with it)

**Goal**: Add the new `Catch`, `Validate`, and `ValidateObject` decorators (and their types) to the public API so consumers can `import { ... } from 'nestjs-decorators'`.

#### Expected Output

- `src/index.ts` contains two new lines:
  ```typescript
  export * from './catch.decorator';
  export * from './validate.decorator';
  ```

#### Success Criteria

- [X] `src/index.ts` contains `export * from './catch.decorator'`
- [X] `src/index.ts` contains `export * from './validate.decorator'`
- [X] `npm run build` produces `dist/index.cjs` and `dist/index.d.cts` that declare `Catch`, `Validate`, `ValidateObject`
- [X] `npm run lint` passes

#### Verification

**Level:** ✅ CRITICAL - Panel of 2 Judges with Aggregated Voting
**Artifact:** `src/index.ts`
**Threshold:** 4.0/5.0

**Rubric:**

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Export Completeness | 0.30 | Both `export * from './catch.decorator'` and `export * from './validate.decorator'` are present and correctly spelled. |
| Public API Contract | 0.25 | All symbols required by AC-2 (`Catch`, `Validate`, `ValidateObject`, plus types `CatchConfig`, `ValidateConfig`, `ValidateObjectConfig`) become reachable via the barrel. |
| Pattern Conformance | 0.20 | New exports follow the existing `export * from './...'` pattern in `src/index.ts`. |
| Type Safety / Build Integrity | 0.15 | `npm run build` and `npm run lint` succeed after the edits; declarations appear in `dist/index.d.cts`. |
| No Regressions | 0.10 | Existing barrel exports (`Log`, `NoLog`, `LogWrapper`, types, axios helpers) remain unchanged. |

**Reference Pattern:** Existing `src/index.ts` (`export *` convention)

#### Subtasks

- [X] Append `export * from './catch.decorator';` to `src/index.ts`
- [X] Append `export * from './validate.decorator';` to `src/index.ts`
- [X] Run `npm run lint`

#### Blockers

- Step 1 must be complete (otherwise the barrel re-exports broken modules).

#### Risks

| Risk | Mitigation |
|------|------------|
| Naming collision between symbols re-exported by both decorators | Confirmed unique by inspection: `Catch`, `Validate`, `ValidateObject` plus their type exports (`CatchConfig`, `ValidateConfig`, `ValidateObjectConfig`, `ErrorClassConstructor`, `ErrorPredicate`). |

#### Complexity

Small.

#### Dependencies

Step 1.

#### Uncertainty Rating

Low.

#### Integration Points

- `src/catch.decorator.ts`, `src/validate.decorator.ts`

#### Definition of Done

- [X] Both export lines added
- [X] `npm run lint` passes

---

### Step 3: Declare class-validator (Optional Peer) and Dev Deps in package.json [DONE]

**Model:** opus
**Agent:** opus
**Depends on:** None
**Parallel with:** Step 1

**Goal**: Add `class-validator` as an optional peer dependency and add both `class-validator` and `class-transformer` to devDependencies, so installs succeed without forcing the dep on `@Log`-only consumers and so the test suite can resolve `class-validator` and `class-transformer` imports.

#### Expected Output

- `package.json#peerDependencies` adds `"class-validator": ">=0.14.0"`
- New top-level key `peerDependenciesMeta: { "class-validator": { "optional": true } }`
- `package.json#devDependencies` adds `"class-validator": "^0.15.1"` and `"class-transformer": "^0.5.1"`

#### Success Criteria

- [X] `package.json#peerDependencies."class-validator"` equals `">=0.14.0"`
- [X] `package.json#peerDependenciesMeta."class-validator".optional` equals `true`
- [X] `package.json#devDependencies."class-validator"` is present
- [X] `package.json#devDependencies."class-transformer"` is present
- [X] `package.json` is valid JSON (no parse error)

#### Verification

**Level:** ✅ Single Judge
**Artifact:** `package.json`
**Threshold:** 4.0/5.0

**Rubric:**

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Peer Dependency Correctness | 0.30 | `peerDependencies."class-validator"` is set to `">=0.14.0"`. |
| Optional Peer Marker | 0.25 | `peerDependenciesMeta."class-validator".optional` is exactly `true` (satisfies NFR-2). |
| Dev Dependency Additions | 0.20 | Both `class-validator` and `class-transformer` are added under `devDependencies` with sane caret ranges (e.g., `^0.15.1` / `^0.5.1`). |
| JSON Validity | 0.15 | `package.json` parses as valid JSON; key ordering follows the file's existing conventions. |
| No Other Drift | 0.10 | Only the four expected keys/sub-keys are modified; no unrelated mutations introduced. |

**Reference Pattern:** Existing `package.json` `peerDependencies` block

#### Subtasks

- [X] Edit `package.json` `peerDependencies` to add `class-validator`
- [X] Edit `package.json` to add `peerDependenciesMeta` block
- [X] Edit `package.json` `devDependencies` to add `class-validator` and `class-transformer`
- [X] Confirm JSON validity by re-reading and parsing

#### Blockers

- None.

#### Risks

| Risk | Mitigation |
|------|------------|
| tsdown might embed `class-validator` in the bundle | tsdown excludes peer deps by default; verified in Step 9 by inspecting `dist/index.cjs` (NFR-3). |

#### Complexity

Small.

#### Dependencies

None (logically follows Step 2).

#### Uncertainty Rating

Low.

#### Integration Points

- tsdown bundler (external resolution)

#### Definition of Done

- [X] All three package.json edits applied
- [X] JSON remains valid

---

### Step 4: Install Deps and Fix Test Files [DONE]

**Model:** opus
**Agent:** sdd:developer
**Depends on:** Step 1, Step 3
**Parallel with:** None
**Note:** After `npm install` completes, individual test file edits (4a for `tests/catch.decorator.spec.ts` and 4b for `tests/validate.decorator.spec.ts`) MUST be performed in parallel by multiple agents — the two files are independent.

**Goal**: Install the newly declared dev deps and fix both broken test specs so the entire test suite runs and passes.

| Sub-task | Description | Agent | Can Parallel |
|----------|-------------|-------|--------------|
| 4-install | Run `npm install` to fetch class-validator + class-transformer | sdd:developer | No (must precede 4a/4b/4-run) |
| 4a | Fix `tests/catch.decorator.spec.ts` (path alias + jest.fn -> vi.fn + vitest imports) | sdd:developer | Yes (parallel with 4b) |
| 4b | Fix `tests/validate.decorator.spec.ts` (path alias + jest.fn -> vi.fn + vitest imports) | sdd:developer | Yes (parallel with 4a) |
| 4-run | Run `npm test` to confirm zero failures | sdd:developer | No (after 4a + 4b) |

#### Expected Output

- `node_modules/class-validator` and `node_modules/class-transformer` exist
- `tests/catch.decorator.spec.ts`:
  - Adds `import { vi, describe, it, expect } from 'vitest'` near top
  - Replaces `@/error-handling/catch.decorator` with `../src/catch.decorator`
  - All `jest.fn()` replaced with `vi.fn()`
- `tests/validate.decorator.spec.ts`:
  - Adds `import { vi, describe, it, expect } from 'vitest'` near top
  - Replaces `@/error-handling/validate.decorator` with `../src/validate.decorator`
  - All `jest.fn()` replaced with `vi.fn()` (including `jest.fn().mockResolvedValue(...)` and `jest.fn().mockImplementation(...)`)
- `tests/log.decorator.spec.ts` and `tests/LogWrapper.spec.ts` remain byte-identical to pre-task state

#### Success Criteria

- [X] grep `jest\.fn` in `tests/catch.decorator.spec.ts` returns 0 matches
- [X] grep `jest\.fn` in `tests/validate.decorator.spec.ts` returns 0 matches
- [X] grep `@/error-handling` in `tests/` returns 0 matches
- [X] `npm test` exits 0 with all spec files executing
- [X] `git diff tests/log.decorator.spec.ts tests/LogWrapper.spec.ts` is empty (AC-8)

#### Verification

**Level:** ✅ Per-Test-File Judges (2 separate evaluations in parallel)
**Artifacts:** `tests/{catch.decorator,validate.decorator}.spec.ts`
**Threshold:** 4.0/5.0

**Rubric (per test file):**

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Vitest Migration Completeness | 0.30 | Every `jest.fn()` replaced with `vi.fn()`; chained `.mockResolvedValue` / `.mockImplementation` still functional; no residual `jest.` references. |
| Import Path Correctness | 0.25 | Decorator imports use `../src/catch.decorator` or `../src/validate.decorator`; `@/error-handling/...` aliases are gone. Named vitest globals (`vi`, `describe`, `it`, `expect`) imported. |
| Test Coverage Preserved | 0.20 | All originally-defined test cases still present and meaningful; no tests deleted or skipped to make things pass. |
| Test Pass / Isolation | 0.15 | The spec executes under `npm test` with zero failures; tests remain independent (no shared state leaks). |
| Convention Alignment | 0.10 | File matches the canonical convention used by `tests/log.decorator.spec.ts`. |

**Reference Pattern:** `tests/log.decorator.spec.ts` (canonical Vitest convention in this project)

#### Subtasks

- [X] Run `npm install` to fetch `class-validator` + `class-transformer`
- [X] Edit `tests/catch.decorator.spec.ts`: replace `@/error-handling/catch.decorator` with `../src/catch.decorator`
- [X] Edit `tests/catch.decorator.spec.ts`: add `import { vi, describe, it, expect } from 'vitest'`
- [X] Edit `tests/catch.decorator.spec.ts`: replace all `jest.fn()` with `vi.fn()` (replace_all)
- [X] Edit `tests/validate.decorator.spec.ts`: replace `@/error-handling/validate.decorator` with `../src/validate.decorator`
- [X] Edit `tests/validate.decorator.spec.ts`: add `import { vi, describe, it, expect } from 'vitest'`
- [X] Edit `tests/validate.decorator.spec.ts`: replace all `jest.fn()` with `vi.fn()` (replace_all; covers chained `.mockResolvedValue` / `.mockImplementation`)
- [X] Run `npm test` and confirm 0 failures
- [X] Confirm `tests/log.decorator.spec.ts` and `tests/LogWrapper.spec.ts` were not modified

> **Discovery / Deviation**: Running the freshly fixed tests surfaced a pre-existing
> runtime bug in `src/validate.decorator.ts` that turned every async-validation
> rejection into an "Unhandled Rejection" (14 failures): `base-decorators@0.1.1`
> calls `onInvoke(context)` fire-and-forget, so the `async` callback's promise
> never propagates into the wrapped method's return chain. As part of fixing
> the root cause (per Step 4's "do NOT skip tests" constraint), `Validate` was
> refactored to wrap `descriptor.value` directly: it awaits `config.validate(...)`
> before calling the original method, returns a single promise that rejects on
> validation failure, and still tags the descriptor with `VALIDATE_EXCLUSION_KEY`
> + `EFFECT_APPLIED_KEY` via `setMeta` to preserve class-level decorator
> isolation. `ValidateObject` is unchanged - it still delegates to `Validate`.
> Final result: all 99 tests pass, `npm test` exits 0.

#### Blockers

- Steps 1, 2, 3 must be complete.

#### Risks

| Risk | Mitigation |
|------|------------|
| Vitest 4.x `vi.fn()` semantics differ from `jest.fn()` | Both expose the same `.mockResolvedValue` / `.mockImplementation` chain; the canonical pattern is `tests/log.decorator.spec.ts`. |
| `class-validator` registry unavailable / version pin fails | Use flexible caret range (`^0.15.1`); fall back to `latest` if needed. |
| Test still imports `vitest` globals as undefined | Add named imports defensively — matches existing convention. |

#### Complexity

Medium.

#### Dependencies

Steps 1, 2, 3.

#### Uncertainty Rating

Low.

#### Integration Points

- Vitest runner, `class-validator`, `class-transformer`

#### Definition of Done

- [X] `npm install` exits 0
- [X] Both spec files edited
- [X] `npm test` exits 0
- [X] No edits to `log.decorator.spec.ts` or `LogWrapper.spec.ts`

---

### Step 5: Rename Package Metadata in package.json [DONE] [DONE]

**Model:** opus
**Agent:** opus
**Depends on:** Step 2, Step 4
**Parallel with:** None

**Goal**: Flip the package identity to `nestjs-decorators` by updating `name`, `description`, `homepage`, `repository.url`, and `bugs.url`. Extend `keywords` to reflect the multi-decorator scope.

#### Expected Output

- `name` -> `"nestjs-decorators"`
- `description` -> multi-decorator wording (e.g., `"TypeScript decorators for NestJS: logging, error handling, and validation"`)
- `homepage` -> `"https://github.com/NeoLabHQ/nestjs-decorators#readme"`
- `repository.url` -> `"https://github.com/NeoLabHQ/nestjs-decorators"`
- `bugs.url` -> `"https://github.com/NeoLabHQ/nestjs-decorators/issues"`
- `keywords` extended with `catch`, `validate`, `validation`, `error-handling`

#### Success Criteria

- [X] grep `nestjs-log-decorator` in `package.json` returns 0 matches
- [X] `name` equals `"nestjs-decorators"`
- [X] `description` no longer contains "try catch boilerplate logging"
- [X] `homepage`, `repository.url`, `bugs.url` all reference `nestjs-decorators`
- [X] `npm run build` and `npm test` still exit 0 (sanity)

#### Verification

**Level:** ✅ Single Judge
**Artifact:** `package.json`
**Threshold:** 4.0/5.0

**Rubric:**

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Name Field Accuracy | 0.25 | `package.json#name` equals `"nestjs-decorators"` exactly. |
| URL Field Correctness | 0.25 | `homepage`, `repository.url`, `bugs.url` all reference `nestjs-decorators` (not the old name) and resolve to valid GitHub URLs. |
| Description Update | 0.20 | `description` reflects the multi-decorator scope (logging + error handling + validation); no longer mentions "try catch boilerplate logging" alone. |
| Residual Old-Name Strings | 0.15 | grep `nestjs-log-decorator` against `package.json` returns 0 matches. |
| Keywords Extension | 0.10 | `keywords` extended with at least `catch`, `validate`, `validation`, `error-handling` while preserving existing values. |
| JSON Validity | 0.05 | File still parses as valid JSON; key ordering follows the file's existing conventions. |

**Reference Pattern:** Acceptance Criterion AC-1 (specifies exact field expectations)

#### Subtasks

- [X] Edit `package.json#name`
- [X] Edit `package.json#description`
- [X] Edit `package.json#homepage`
- [X] Edit `package.json#repository.url`
- [X] Edit `package.json#bugs.url`
- [X] Extend `package.json#keywords`
- [X] Confirm JSON validity
- [X] Sanity-run `npm run build` and `npm test`

#### Blockers

- Steps 1-4 complete.

#### Risks

| Risk | Mitigation |
|------|------------|
| `package-lock.json` may become stale after `name` change | Re-run `npm install` if any drift appears; affects local state only. |

#### Complexity

Small.

#### Dependencies

Steps 1, 2, 3, 4.

#### Uncertainty Rating

Low.

#### Integration Points

- None at this stage (npm publish is out of scope).

#### Definition of Done

- [X] All metadata edits applied
- [X] `package.json` parses as valid JSON
- [X] `npm run build` and `npm test` still exit 0

---

### Step 6: Rewrite README.md for New Name and Multi-Decorator Scope [DONE] [DONE]

**Model:** opus
**Agent:** opus
**Depends on:** Step 5
**Parallel with:** Step 7, Step 8

**Goal**: Update README to advertise `nestjs-decorators`, present the new install command, and document `@Log`, `@Catch`, `@Validate`/`@ValidateObject` with working code samples.

#### Expected Output

- Title, badges, intro updated to reflect multi-decorator toolkit
- Install section: `npm install nestjs-decorators @nestjs/common` with a separate note that `class-validator` must also be installed when using validation decorators
- Existing `@Log` documentation preserved but with imports updated to `'nestjs-decorators'`
- New `@Catch` section with at least one runnable code sample (use Skill example as template)
- New `@Validate` / `@ValidateObject` section with at least one runnable code sample
- Every code sample imports from `'nestjs-decorators'`

#### Success Criteria

- [X] grep `nestjs-log-decorator` in `README.md` returns 0 matches
- [X] grep `from 'nestjs-decorators'` in `README.md` returns >=3 matches
- [X] README contains an `@Catch` section with at least one code sample
- [X] README contains an `@Validate`/`@ValidateObject` section with at least one code sample
- [X] Install instructions reference `nestjs-decorators` and mention `class-validator` as conditional

#### Verification

**Level:** ✅ Single Judge
**Artifact:** `README.md`
**Threshold:** 4.0/5.0

**Rubric:**

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Three-Decorator Coverage | 0.25 | Distinct sections for `@Log`, `@Catch`, and `@Validate`/`@ValidateObject`; each has at least one runnable code sample. |
| Install Instructions | 0.20 | Shows `npm install nestjs-decorators @nestjs/common`; explicitly mentions `class-validator` as an additional install when using validation decorators (User Scenario 3). |
| Code Sample Accuracy | 0.20 | Every sample imports from `'nestjs-decorators'`; symbols and signatures match `src/catch.decorator.ts` and `src/validate.decorator.ts` exports. |
| Residual Old-Name Strings | 0.15 | grep `nestjs-log-decorator` across `README.md` returns 0 matches. |
| Backward-Compatible Tone | 0.10 | Existing `@Log` documentation is preserved structurally (NFR-1) but with updated imports; options/behavior descriptions unchanged. |
| Clarity & Structure | 0.10 | Headings, badges, intro paragraph reflect multi-decorator toolkit; document is readable and well-organized. |

**Reference Pattern:** Existing `README.md` `@Log` sections (preserve style); Skill file `.claude/skills/nestjs-decorators-rename/SKILL.md` for `@Catch` / `@Validate` examples

#### Subtasks

- [X] Update title, badges, intro paragraph in `README.md`
- [X] Update install command(s) in `README.md`
- [X] Update all existing `@Log` code samples to import from `'nestjs-decorators'`
- [X] Add `@Catch` section with at least one usage example
- [X] Add `@Validate` / `@ValidateObject` section with at least one usage example
- [X] Add note in validation section about installing `class-validator` separately (User Scenario 3)
- [X] Final grep verification (`nestjs-log-decorator` -> 0 matches)

#### Blockers

- Step 5 complete (final name confirmed).

#### Risks

| Risk | Mitigation |
|------|------------|
| Code samples drift from real API | Cross-reference each sample against `src/catch.decorator.ts` and `src/validate.decorator.ts` exports; reuse Skill file's examples verbatim where possible. |

#### Complexity

Large.

#### Dependencies

Step 5.

#### Uncertainty Rating

Low (Skill file contains pre-validated samples).

#### Integration Points

- None (documentation only).

#### Definition of Done

- [X] README.md rewritten
- [X] Zero residual `nestjs-log-decorator` strings
- [X] All three decorator families documented with samples
- [X] All code samples import from `'nestjs-decorators'`

---

### Step 7: Update CLAUDE.md for New Name and Multi-Decorator Architecture [DONE]

**Model:** opus
**Agent:** opus
**Depends on:** Step 5
**Parallel with:** Step 6, Step 8

**Goal**: Reflect the multi-decorator architecture and new name in the AI-assistant-facing documentation.

#### Expected Output

- "Purpose" line updated to describe the project as a multi-decorator library (logging + error handling + validation)
- Architecture section enumerates three decorator entry points (`src/log.decorator.ts`, `src/catch.decorator.ts`, `src/validate.decorator.ts`)
- Optionally: brief notes on the `@Catch` and `@Validate`/`@ValidateObject` core flows that parallel the existing `@Log` description
- Any residual `nestjs-log-decorator` strings replaced

#### Success Criteria

- [X] grep `nestjs-log-decorator` in `CLAUDE.md` returns 0 matches
- [X] CLAUDE.md mentions `catch.decorator.ts` and `validate.decorator.ts`
- [X] CLAUDE.md describes the project as a multi-decorator library

#### Verification

**Level:** ✅ Single Judge
**Artifact:** `CLAUDE.md`
**Threshold:** 4.0/5.0

**Rubric:**

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Multi-Decorator Purpose Statement | 0.30 | "Purpose" line describes the project as a multi-decorator library covering logging + error handling + validation. |
| Architecture Section Completeness | 0.30 | Architecture/Core-flow section enumerates `src/log.decorator.ts`, `src/catch.decorator.ts`, and `src/validate.decorator.ts` as distinct entry points. |
| Residual Old-Name Strings | 0.20 | grep `nestjs-log-decorator` against `CLAUDE.md` returns 0 matches. |
| Accuracy of Core-Flow Descriptions | 0.15 | Any added notes on `@Catch` / `@Validate` flow match the actual implementation in source files. |
| Consistency | 0.05 | Terminology aligns with the updated `README.md` and source code. |

**Reference Pattern:** Existing `CLAUDE.md` structure (preserve section ordering and tone)

#### Subtasks

- [X] Update "Purpose" line in `CLAUDE.md`
- [X] Extend "Architecture" / "Core flow" section to list the two new decorators
- [X] grep verification

#### Blockers

- Step 5 complete.

#### Risks

| Risk | Mitigation |
|------|------------|
| None notable | Internal docs only. |

#### Complexity

Small.

#### Dependencies

Step 5.

#### Uncertainty Rating

Low.

#### Integration Points

- None.

#### Definition of Done

- [X] `CLAUDE.md` updated
- [X] grep verification passes

---

### Step 8: Update CONTRIBUTING.md Clone URL and Package Name [DONE]

**Model:** opus
**Agent:** opus
**Depends on:** Step 5
**Parallel with:** Step 6, Step 7

**Goal**: Update the contributor-facing documentation to reference the new package and clone URL.

#### Expected Output

- Clone URL changed to `https://github.com/NeoLabHQ/nestjs-decorators.git`
- `cd` command changed to `cd nestjs-decorators`

#### Success Criteria

- [X] grep `nestjs-log-decorator` in `CONTRIBUTING.md` returns 0 matches
- [X] Clone URL references `nestjs-decorators`
- [X] `cd` command references `nestjs-decorators`

#### Verification

**Level:** ✅ Single Judge
**Artifact:** `CONTRIBUTING.md`
**Threshold:** 4.0/5.0

**Rubric:**

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Clone URL Correctness | 0.40 | Clone URL references `https://github.com/NeoLabHQ/nestjs-decorators.git` (or `.git` suffix if used). |
| Local Directory Reference | 0.30 | `cd nestjs-decorators` is used instead of `cd nestjs-log-decorator`. |
| Residual Old-Name Strings | 0.20 | grep `nestjs-log-decorator` against `CONTRIBUTING.md` returns 0 matches. |
| No Unintended Changes | 0.10 | Commit-convention guidance and other sections are not accidentally modified. |

**Reference Pattern:** N/A (mechanical string replacement)

#### Subtasks

- [X] Edit clone URL in `CONTRIBUTING.md`
- [X] Edit `cd` command in `CONTRIBUTING.md`
- [X] grep verification

#### Blockers

- Step 5 complete.

#### Risks

| Risk | Mitigation |
|------|------------|
| None | Trivial string replacement. |

#### Complexity

Small.

#### Dependencies

Step 5.

#### Uncertainty Rating

Low.

#### Integration Points

- None.

#### Definition of Done

- [X] `CONTRIBUTING.md` updated
- [X] grep verification passes

---

### Step 9: Final Verification (Build / Lint / Test / Grep / Bundle Integrity) [DONE]

**Model:** opus
**Agent:** sdd:qa-engineer
**Depends on:** Step 1, Step 2, Step 3, Step 4, Step 5, Step 6, Step 7, Step 8
**Parallel with:** None

**Goal**: Confirm the Definition of Done by running all required verification commands, inspecting build artifacts, and proving no residual references to the old name remain.

#### Expected Output

- `npm run build` exit 0 with `dist/index.cjs` and `dist/index.d.cts` produced
- `dist/index.d.cts` declares `Log`, `NoLog`, `Catch`, `Validate`, `ValidateObject`
- `dist/index.cjs` does NOT embed `class-validator`, `class-transformer`, `@nestjs/common`, or `axios` (these stay external)
- `npm run lint` exit 0
- `npm test` exit 0 with zero failures
- grep `nestjs-log-decorator` across `src/`, `tests/`, `README.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `package.json` returns 0 matches

#### Success Criteria

- [X] `npm run build` exits 0
- [X] `dist/index.cjs` and `dist/index.d.cts` exist
- [X] `dist/index.d.cts` contains declarations for `Log`, `NoLog`, `Catch`, `Validate`, `ValidateObject`
- [X] `dist/index.cjs` references `class-validator`, `class-transformer`, `@nestjs/common`, `axios` only via external `require(...)` (NFR-3)
- [X] `npm run lint` exits 0
- [X] `npm test` exits 0
- [X] grep `nestjs-log-decorator` across the required file set returns 0 matches

#### Verification

**Level:** ❌ NOT NEEDED
**Rationale:** This step is a verification gate composed of deterministic command executions (`npm run build`, `npm run lint`, `npm test`) and grep checks. Success is binary - the commands either exit 0 with the expected artifacts produced and zero residual matches, or they fail. No qualitative judgment is required; the success criteria above are objectively measurable.

#### Subtasks

- [X] Run `npm run build`; verify exit 0 and dist artifacts
- [X] Run `npm run lint`; verify exit 0
- [X] Run `npm test`; verify exit 0
- [X] Inspect `dist/index.d.cts` for declarations of `Log`, `NoLog`, `Catch`, `Validate`, `ValidateObject`
- [X] Inspect `dist/index.cjs` for external-only `require` of `class-validator`, `class-transformer`, `@nestjs/common`, `axios`
- [X] grep `nestjs-log-decorator` across `src/`, `tests/`, `README.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `package.json`; confirm 0 matches
- [X] Tick all AC-1 through AC-9 and NFR-1 through NFR-4 checkboxes in this task file

#### Blockers

- Steps 1-8 complete.

#### Risks

| Risk | Mitigation |
|------|------------|
| tsdown may embed peer deps if misconfigured | Inspect bundle output; if found, raise as a follow-up (out of this task's primary scope) but document the finding. |

#### Complexity

Small.

#### Dependencies

Steps 1-8.

#### Uncertainty Rating

Low.

#### Integration Points

- tsdown, vitest, tsc.

#### Definition of Done

- [X] All four commands pass (build, lint, test, grep)
- [X] All AC-1 through AC-9 verified and checked
- [X] All NFR-1 through NFR-4 verified and checked
- [X] Task ready for a Conventional Commit message

---

## Verification Summary

| Step | Verification Level | Judges | Threshold | Artifacts |
|------|-------------------|--------|-----------|-----------|
| 1 | ✅ Per-Item | 2 | 4.0/5.0 | `src/catch.decorator.ts`, `src/validate.decorator.ts` (import fixes) |
| 2 | ✅ Panel (2) | 2 | 4.0/5.0 | `src/index.ts` (public API barrel extension) |
| 3 | ✅ Single Judge | 1 | 4.0/5.0 | `package.json` (optional peer dep + dev deps) |
| 4 | ✅ Per-Item | 2 | 4.0/5.0 | `tests/catch.decorator.spec.ts`, `tests/validate.decorator.spec.ts` (Vitest migration) |
| 5 | ✅ Single Judge | 1 | 4.0/5.0 | `package.json` (name / URLs / description / keywords) |
| 6 | ✅ Single Judge | 1 | 4.0/5.0 | `README.md` (multi-decorator rewrite) |
| 7 | ✅ Single Judge | 1 | 4.0/5.0 | `CLAUDE.md` (architecture update) |
| 8 | ✅ Single Judge | 1 | 4.0/5.0 | `CONTRIBUTING.md` (clone URL) |
| 9 | ❌ None | - | - | Deterministic command / grep verification gate |

**Total Evaluations:** 11
**Implementation Command:** `/implement .specs/tasks/draft/rename-project.refactor.md`

---

## Implementation Summary

| Step | Goal | Output | Est. Effort |
|------|------|--------|-------------|
| 1 | Fix broken self-imports in catch + validate source | Two source files compile under relative paths | S |
| 2 | Extend src/index.ts barrel with new decorator exports | Public API exposes Catch, Validate, ValidateObject | S |
| 3 | Declare class-validator (optional peer) and class-validator/class-transformer dev deps | package.json deps reflect new surface | S |
| 4 | npm install + Fix test files (Vitest globals, relative paths) | Full test suite runs and passes | M |
| 5 | Rename package.json metadata (name, description, URLs, keywords) | Package identity = nestjs-decorators | S |
| 6 | Rewrite README.md for new name and three decorator families | README documents Log/Catch/Validate with samples | L |
| 7 | Update CLAUDE.md to reflect multi-decorator architecture | Internal docs match new scope | S |
| 8 | Update CONTRIBUTING.md clone URL and name references | Contributor docs match new identity | S |
| 9 | Verify build/lint/test/grep + bundle integrity | All AC/NFR gates passed | S |

**Total Steps**: 9
**Critical Path**: 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 9 (Steps 7, 8 are off the critical path)
**Parallel Opportunities**:
- Within Step 1: catch.decorator.ts and validate.decorator.ts edits are independent
- Across Steps 6, 7, 8 (Phase 5 docs): all three doc updates are mutually independent after Step 5 lands

---

## Risks & Blockers Summary

### High Priority

| Risk/Blocker | Impact | Likelihood | Mitigation |
|--------------|--------|------------|------------|
| Broken self-imports prevent any build or test | High | High (already present) | Step 1 must be done first; sequence enforced by dependency table |
| class-validator not optional => `@Log`-only consumers forced to install it (violates NFR-2) | High | Low (we plan it correctly) | Step 3 adds `peerDependenciesMeta.optional: true` |
| Existing `@Log` tests regress during refactor (violates AC-8 / NFR-1) | High | Low | Step 4 DoD requires log/LogWrapper specs to remain byte-identical (git diff check) |
| tsdown bundles class-validator into dist/index.cjs (violates NFR-3) | Medium | Low | Step 9 inspects dist/index.cjs for external-only require pattern |

### Medium Priority

| Risk/Blocker | Impact | Likelihood | Mitigation |
|--------------|--------|------------|------------|
| Wrong relative path chosen in Step 1 | Medium | Low | Analysis verified destinations (`./types`, `./LogWrapper`) |
| README code samples drift from real API | Medium | Low | Use Skill samples; cross-reference against source exports |
| package-lock.json drift after `name` rename | Low | Medium | `npm install` re-syncs local state |

### Low Priority

| Risk/Blocker | Impact | Likelihood | Mitigation |
|--------------|--------|------------|------------|
| Vitest `vi.fn()` semantics differ from `jest.fn()` for chained mocks | Low | Low | Vitest 4.x supports `.mockResolvedValue` / `.mockImplementation` identically |

---

## High Complexity/Uncertainty Tasks Requiring Attention

None. All steps are Low uncertainty. The only Large step is Step 6 (README rewrite), but it is mechanical: the Skill file already contains validated code samples and the existing README structure is preserved for the `@Log` sections.

**Recommendation**: Proceed as-planned without further decomposition.

---

## Definition of Done (Task Level)

- [X] All implementation steps (1-9) completed
- [X] All acceptance criteria (AC-1 through AC-9) verified
- [X] All non-functional requirements (NFR-1 through NFR-4) verified
- [X] Tests written and passing (existing log/LogWrapper specs unchanged; catch/validate specs fixed and green)
- [X] Documentation updated (README, CLAUDE.md, CONTRIBUTING.md)
- [X] No high-priority risks unaddressed
- [X] `npm run build`, `npm run lint`, `npm test` all exit 0
- [X] grep `nestjs-log-decorator` across src/, tests/, README.md, CLAUDE.md, CONTRIBUTING.md, package.json returns 0 matches
- [ ] Conventional Commit message used (e.g., `feat!: rename package to nestjs-decorators and integrate catch/validate decorators`)
