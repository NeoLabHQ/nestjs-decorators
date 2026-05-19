# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Use @README.md for project overview, @CONTRIBUTING.md for contributing guidelines, @package.json for avaiable commands.

## Commit Conventions

This project uses **semantic-release** with **Conventional Commits**. Follow the `type(scope): subject` format (feat, fix, docs, style, refactor, perf, test, chore, ci) for commits.

## Architecture

**Purpose:** A zero-dependency TypeScript decorator library for NestJS providing `@Log` (structured logging), `@Catch` (error handling), and `@Validate`/`@ValidateObject` (input validation) decorators. The toolkit eliminates try-catch logging boilerplate, configurable error-handling boilerplate, and pre-execution input-validation boilerplate from service methods.

**Build:** tsdown compiles `src/` to `dist/` as CommonJS (CJS only). TypeScript declarations are emitted via `emitDeclarationOnly` in tsconfig. Peer dependencies: `@nestjs/common` (required) and `class-validator >=0.14.0` (optional — only needed by `@ValidateObject`).

**Decorator entry points:**

1. `@Log()` / `@NoLog()` (src/log.decorator.ts) — Unified decorator that detects whether it's applied to a class or method via argument count, then delegates to `Effect` (class) or `EffectOnMethod` (method). Builds a `HooksOrFactory` function that creates a `LogWrapper` once per invocation and wires up `onInvoke`, `onReturn`, and `onError` hooks. `@NoLog()` marks methods with a Symbol to exclude them from class-level logging.
2. `@Catch()` (src/catch.decorator.ts) — Method decorator that wraps an async method in a configurable try-catch. Built on top of `OnErrorHook` from `base-decorators`. Accepts a `CatchConfig` whose `on` field is an error class constructor, an error predicate function, or `undefined` (catch-all); when the filter matches, the `handle` callback runs bound to the class instance (so `this` refers to the service) with the caught error and the original method arguments. Logs through `createLogWrapper` and supports `formatArgs` for redaction and a custom `message`. Multiple `@Catch` decorators stack on a single method via a unique `CATCH_EXCLUSION_KEY` Symbol.
3. `@Validate()` / `@ValidateObject()` (src/validate.decorator.ts) — Method decorators that run a validation callback BEFORE the decorated method executes. `Validate` wraps `descriptor.value` directly (so async rejections propagate into the method's return-promise chain instead of becoming unhandled rejections), awaits `config.validate(...)` with the call-time arguments, and on failure logs via `createLogWrapper` and re-throws. `ValidateObject` delegates to `Validate`: it `extract`s an object from the arguments, runs class-validator's `validate()` function against it, and on violations calls a `handleErrors(args, errors, formattedMessages)` callback that is expected to throw a domain exception. `validatorOptions` (groups, whitelist, etc.) and `ValidationError` are re-exposed from `class-validator`.

**Shared primitives (used by all three decorator families):**

- `Effect` / `EffectOnMethod` / `EffectOnClass` (src/decorators/) — Logger-agnostic decorator primitives from `base-decorators`. `EffectOnMethod` wraps a single method: extracts parameter names, builds a `HookContext` (args object, target, propertyKey, descriptor, parameterNames, className), and invokes lifecycle hooks. `EffectOnClass` iterates prototype methods and applies `EffectOnMethod` to each. `Effect` dispatches to one or the other based on argument count.
- `buildArgsObject` (from `base-decorators`) — Maps parameter names to their call-time values to produce the pre-built `args` object passed in every `HookContext` and used by `Validate`'s manual descriptor wrap to build a name-keyed args object for logging.
- `LogWrapper` (src/LogWrapper.ts) — Formats and outputs structured log entries (`invoked`, `success`, `error` states) through the NestJS Logger. Contains `createLogWrapper` (auto-injects a NestJS Logger when the instance has no `logger` property) and `isLoggable` type guard. Consumed by `@Log`, `@Catch`, and `@Validate`/`@ValidateObject`.

**Axios handling:** The library has no runtime axios dependency. `src/axios/axios.stub.ts` defines local interfaces mirroring Axios types, and `isAxiosError` checks `payload.isAxiosError === true`. `prettifyAxiosError` in `axios.logger.ts` formats Axios errors with structured request/response data when detected.
