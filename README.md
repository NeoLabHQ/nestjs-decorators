<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo-small.svg" width="120" alt="Nest Logo" /></a>
</p>

<div align="center">

<h1>NestJS Decorators</h1>

![Build Status](https://github.com/neolabhq/nestjs-decorators/actions/workflows/build.yaml/badge.svg)
[![npm version](https://img.shields.io/npm/v/nestjs-decorators)](https://www.npmjs.com/package/nestjs-decorators)
[![Bundle Size](https://img.shields.io/bundlephobia/minzip/nestjs-decorators)](https://www.npmjs.com/package/nestjs-decorators)
[![NPM Downloads](https://img.shields.io/npm/dw/nestjs-decorators)](https://www.npmjs.com/package/nestjs-decorators)
[![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)](https://opensource.org/licenses/AGPL-3.0)

TypeScript decorators for NestJS: logging, error handling, and validation.

[Quick Start](#quick-start) •
[How It Works](#how-it-works) •
[Usage](#usage) •
[Options](#options) •
[API Reference](#api-reference) •
[Advanced Example](#advanced-example)

</div>

## Description

Zero-dependency TypeScript decorator primitives for NestJS applications, that eliminate the most common boilerplate:

- **`@Log`** — replaces the try-catch logging pattern by automatically logging method success and errors, with optional invocation and result logging.
- **`@Catch`** — wraps a method in declarative `try/catch` so failures are routed to a handler callback (with optional error-type filtering) instead of bubbling raw exceptions.
- **`@Validate` / `@ValidateObject`** — runs validation logic integrated with [class-validator](https://github.com/typestack/class-validator) before the method executes, so business code never receives invalid input.

**Key Features**

- By default uses structured output
- Prettifies Axios errors
- Zero configuration
- Minimal dependencies
- Uses the default `@nestjs/common` Logger instance

## Installation

```bash
npm install nestjs-decorators @nestjs/common
```

`class-validator` is an **optional peer dependency**. Install it only when you intend to use the `@Validate` or `@ValidateObject` decorators:

```bash
npm install class-validator
```

## Quick Start

### @Log()

Simply apply `@Log()` to your class or method:

```typescript
import { Log } from 'nestjs-decorators';

class UserService {
  @Log()
  createUser(name: string, email: string) {
    return { id: 1, name, email };
  }
}
```

Once a service method is called, it will log the method invocation with all arguments.

```typescript
const result = service.createUser('John', 'john@example.com');
// console output:
// [UserService] { method: 'createUser', state: 'success', args: { name: 'John', email: 'john@example.com' } }
```

### @Catch()

```typescript
import { Catch } from 'nestjs-decorators';

const IncreaseErrorMetricOnError = Catch({
    handle(error, [userId, message]) {
      // Swallow notification failures so the parent flow continues
      this.metrics.increment('notification.failed');
    },
  })

class NotificationService {
  constructor(private readonly metrics: MetricsService) {}

  @IncreaseErrorMetricOnError()
  async notify(userId: string, message: string) {
    await this.transport.send(userId, message);
  }
}
```

### `@Validate` — custom validation callback

`@Validate` allow to decompose the validation logic into a separate callback, that is called before the method executes.

```typescript
import { Validate } from 'nestjs-decorators';

const ValidateUser = Validate<[User, string]>({
  message: 'User validation failed',
  validate(user, userId) {
    if (!user.email) {
      throw new Error('User email is required'); // error automatically logged
    }
  },
})

class UserService {
  @ValidateUser()
  async createUser(user: User, userId: string) {
    // if validation fails, this method will not run
    return await this.userRepository.create(user);
  }
}
```

### `@ValidateObject` — class-validator integration

`@ValidateObject` is a thin wrapper around `@Validate` that uses [class-validator](https://github.com/typestack/class-validator) to check a DTO-like object extracted from the method arguments.

```typescript
import { ValidateObject } from 'nestjs-decorators';
import { IsEmail, IsNotEmpty, IsString } from 'class-validator';

class CreateUserDto {
  @IsNotEmpty()
  @IsString()
  name!: string;

  @IsEmail()
  email!: string;
}

const ValidateUser = ValidateObject<[CreateUserDto]>({
  extract: (dto) => dto,
  // If `dto` fails any `class-validator` constraint, `handleErrors` is invoked
  // expects `handleErrors` to throw a domain exception
  handleErrors: ([dto], errors, messages) => {
    // `errors` is the raw class-validator ValidationError[]
    // `messages` is a pre-formatted human-readable string[]
    throw new BadRequestException(messages);
  },
})

class UserService {
  @ValidateUser()
  async createUser(dto: CreateUserDto) {
    // if validation fails, this method will not run
    return await this.repo.create(dto);
  }
}
```

## Usage

### @Log()

The `@Log()` decorator wraps your methods with automatic try-catch logging. It extracts parameter names, captures arguments, and logs structured output on success or error.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            @Log() Decorator Flow                            │
└─────────────────────────────────────────────────────────────────────────────┘

  Method Call
       │
       ▼
┌──────────────────┐
│ Extract Args     │  ──▶  { id: 1, name: 'John' }
│ (auto or custom) │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐       ┌─────────────────────────────────────┐
│ onInvoke: true?  │──YES─▶│ logger.log({ state: 'invoked' })    │
└────────┬─────────┘       └─────────────────────────────────────┘
         │ NO
         ▼
┌──────────────────────┐
│ Execute Original     │
│ Method (sync/async)  │
└────────┬─────────────┘
         │
    ┌────┴────┐
    ▼         ▼
 SUCCESS    ERROR
    │         │
    ▼         ▼
┌────────┐ ┌──────────────────────────────────────────────────────┐
│log()   │ │ logger.error({ state: 'error', error: prettify(e) })│
│success │ │ (Axios errors auto-prettified)                      │
└────────┘ └──────────────────────────────────────────────────────┘
    │         │
    ▼         ▼
 Return    Re-throw
 Result    Error
```

#### Error Logging

If a method throws an error, by default the decorator logs and throws it, preserving the original stack trace.

```typescript
@Log()
createUser(name: string) {
  throw new Error('Validation failed');
}
```

Example call with error:

```typescript
const result = service.createUser('John');
// console output:
// [UserService] { method: 'createUser', state: 'error', args: { name: 'John' }, error: Error: Validation failed }
```

#### Invocation Logging

If you want to log the method invocation, you can use the `onInvoke` option.

```typescript
@Log({ onInvoke: true })
async createUser(name: string) {
  return await Promise.resolve({ name });
}
```

Example call with invocation logging:

```typescript
const resultPromise = service.createUser('John');
// [UserService] { method: 'createUser', state: 'invoked', args: { name: 'John' } }
const result = await resultPromise;
// [UserService] { method: 'createUser', state: 'success', args: { name: 'John' } }
```

#### Result Logging

If you want to include method results in success logs, use the `result` option.

```typescript
class UserService {
  // Logs result as-is
  @Log({ result: true })
  findUser(id: number) {
    return { id, name: 'John', email: 'john@example.com' };
  }

  // Logs formatted result
  @Log({
    result: (res: { id: number; name: string; email: string }) => ({ id: res.id, name: res.name }),
  })
  findPublicUser(id: number) {
    return { id, name: 'John', email: 'john@example.com' };
  }
}
```

Example success output:

```typescript
// [UserService] { method: 'findUser', state: 'success', args: { id: 1 }, result: { id: 1, name: 'John', email: 'john@example.com' } }
// [UserService] { method: 'findPublicUser', state: 'success', args: { id: 1 }, result: { id: 1, name: 'John' } }
```

#### Explicit Logger

If you need a custom logger (e.g., for testing or a different context), you can still define your own:

```typescript
import { Logger } from '@nestjs/common';
import { Log } from 'nestjs-decorators';

@Log()
class PaymentService {
  // Explicit logger takes precedence over auto-injected one
  readonly logger = new Logger('CustomPaymentContext');

  async processPayment(amount: number, currency: string) {
    // Logs using the explicit logger with 'CustomPaymentContext' context
    return await this.gateway.processPayment(amount, currency);
  }
}
```

#### Class-Level Decorator

If you want to log all methods in a class, use the `@Log()` decorator on its definition:

```typescript
import { Log } from 'nestjs-decorators';

@Log()
@Injectable()
class PaymentService {
  processPayment(amount: number, currency: string) {
    // Automatically logged on success or error
    return { status: 'completed', amount, currency };
  }

  async refund(transactionId: string) {
    // Async methods are also logged
    return await this.gateway.refund(transactionId);
  }
}
```

#### Excluding Methods with `@NoLog()`

When using class-level `@Log()`, you can exclude specific methods with `@NoLog()`:

```typescript
import { Log, NoLog } from 'nestjs-decorators';

@Log()
class UserService {
  createUser(name: string) {
    // Logged
    return { name };
  }

  @NoLog()
  internalHelper() {
    // NOT logged
    return 'helper';
  }
}
```

### `@Catch`

The `@Catch()` decorator wraps a method in a declarative `try/catch` block. When the method throws, the decorator routes the error to a handler callback instead of letting it bubble. The error is also logged through the same structured logger used by `@Log()`.

You can:

- catch **all** errors (omit `on`)
- catch only instances of a specific error **class** (`on: AxiosError`)
- catch errors matched by a **predicate** function (`on: (e) => ...`)

Multiple `@Catch()` decorators can be stacked on a single method — they are applied bottom-up, and unmatched errors propagate to the next outer decorator.

#### Catching a specific error class

```typescript
import { AxiosError } from 'axios';

interface Data {
  id: string;
  payload: unknown;
}

class DataService {
  @Catch({
    on: AxiosError,
    handle(error, [id]) {
      // Returned value becomes the method's return value
      return null;
    },
  })
  async fetchData(id: string): Promise<Data | null> {
    return await this.httpClient.get(`/data/${id}`);
  }
}
```

If `fetchData` throws an `AxiosError`, the error is logged and `null` is returned. Any other error type re-throws normally.

#### Catching with a predicate

```typescript
import { AxiosError } from 'axios';

class DataService {
  @Catch({
    on: (e) => e instanceof AxiosError && e.response?.status === 404,
    handle(error, [id]) {
      // Only 404s are swallowed — other Axios errors still throw
      return null;
    },
    message: 'Resource not found, returning null',
  })
  async findOptional(id: string) {
    return await this.httpClient.get(`/data/${id}`);
  }
}
```

### `@Validate` — custom validation callback

`@Validate` runs a validation callback **before** the decorated method executes.
Use `@Validate` when validation is simple or domain-specific and you don't need `class-validator` decorators on the input.

```typescript

const ValidateUser = Validate<[User, string]>({
  message: 'User validation failed',
  validate(user, userId) {
    if (!user.email) {
      throw new Error('User email is required');
    }
  },
})

class UserService {
  @ValidateUser()
  async createUser(user: User, userId: string) {
    return await this.userRepository.create(user);
  }
}
```

If the `validate` callback throws (sync or async), the error is logged and re-thrown — the original `computeScore` body never runs.

### `@ValidateObject` — class-validator integration

`@ValidateObject` is a thin wrapper around `@Validate` that uses [class-validator](https://github.com/typestack/class-validator) to check a DTO-like object extracted from the method arguments.
Use it when the input is a DTO decorated with `class-validator` constraints. 

**Requires `class-validator`**. 
```bash
npm install class-validator
```

```typescript
class CreateUserDto {
  @IsNotEmpty()
  @IsString()
  name!: string;

  @IsEmail()
  email!: string;
}

const ValidateUser = ValidateObject<[CreateUserDto]>({
  extract: (dto) => dto,
  // If `dto` fails any `class-validator` constraint, `handleErrors` is invoked
  // expects `handleErrors` to throw a domain exception
  handleErrors: ([dto], errors, messages) => {
    // `errors` is the raw class-validator ValidationError[]
    // `messages` is a pre-formatted human-readable string[]
    throw new BadRequestException(messages);
  },
})

class UserService {
  @ValidateUser()
  async createUser(dto: CreateUserDto) {
    // if validation fails, this method will not run
    return await this.repo.create(dto);
  }
}
```

If `dto` fails any `class-validator` constraint, `handleErrors` is invoked. The decorator expects `handleErrors` to throw a domain exception — the original `createUser` body will not run.

## Options

### `@Log()` Options

#### `onInvoke`

Log method invocation (before execution), not just completion:

```typescript
@Log({ onInvoke: true })
async fetchExternalData(url: string) {
  const response = await fetch(url);
  return response.json();
}
// Logs: { method: 'fetchExternalData', state: 'invoked', args: { url: '...' } }
// Logs: { method: 'fetchExternalData', state: 'success', args: { url: '...' } }
```

Class-level with `onInvoke`:

```typescript
@Log({ onInvoke: true })
class ApiService {
  // All methods will log invocation + completion
}
```

#### `args` — Custom Argument Formatting

Control what arguments are logged. Useful for:

- Excluding large objects from logs
- Hiding sensitive data (passwords, tokens)
- Logging only specific arguments

```typescript
interface LargePayload {
  data: Buffer;
  metadata: object;
}

class SyncService {
  // Only log the ID, exclude the large payload
  @Log({ args: (id: number, _payload: LargePayload) => ({ id }) })
  async syncData(id: number, payload: LargePayload) {
    return await this.process(id, payload);
  }

  // Log multiple specific args
  @Log({ args: (userId: number, txId: string, _data: object) => ({ userId, txId }) })
  async processTransaction(userId: number, txId: string, data: object) {
    return await this.execute(userId, txId, data);
  }

  // Return a custom string
  @Log({ args: (id: number, name: string) => `${id}:${name}` })
  lookupUser(id: number, name: string) {
    return this.users.find(id, name);
  }
}
```

**Output:**

```
[SyncService] { method: 'syncData', state: 'success', args: { id: 123 } }
[SyncService] { method: 'processTransaction', state: 'success', args: { userId: 1, txId: 'tx_abc' } }
[SyncService] { method: 'lookupUser', state: 'success', args: '1:John' }
```

#### `result` — Success Result Logging

Control how return values are included in success logs:

- `result: true` logs the raw return value
- `result: (value) => ...` logs a formatted value

```typescript
class PaymentService {
  // Log full return value
  @Log({ result: true })
  createPayment(id: number) {
    return { id, status: 'success', cardToken: 'tok_123' };
  }

  // Log only safe result fields
  @Log({
    result: (res: { id: number; status: string; cardToken: string }) => ({ id: res.id, status: res.status }),
  })
  createPaymentSafe(id: number) {
    return { id, status: 'success', cardToken: 'tok_123' };
  }
}
```

### `@Catch` Options

| Option | Type | Description |
|--------|------|-------------|
| `on` | `ErrorClassConstructor \| ErrorPredicate \| undefined` | Filter for which errors to catch. Omit to catch all errors. |
| `handle` | `(error, methodArgs) => R` | Handler invoked when an error matches. Bound to the class instance (`this` is the service). Its return value becomes the method's return value. |
| `formatArgs` | `(...args) => any` | Optional formatter for the args logged alongside the error (same shape as `@Log`'s `args` option). |
| `message` | `string` | Optional custom log message attached to the error log entry. |

### `@ValidateObject` Options

| Option | Type | Description |
|--------|------|-------------|
| `extract` | `(...args) => object` | Pulls the object to validate out of the method arguments. |
| `handleErrors` | `(args, errors, messages) => void` | Called when `class-validator` finds violations. Should throw a domain exception. Receives the original args, raw `ValidationError[]`, and pre-formatted message strings. |
| `validatorOptions` | `ValidatorOptions` | Optional [class-validator options](https://github.com/typestack/class-validator#passing-options) (`groups`, `whitelist`, etc.). |
| `message` | `string` | Optional custom log message attached to the error log entry. |

### `@Validate` Options

| Option | Type | Description |
|--------|------|-------------|
| `validate` | `(...args) => void \| Promise<void>` | Validation callback. Throw (or reject) to fail validation. May be sync or async. |
| `message` | `string` | Optional custom log message attached to the error log entry. |

## API Reference

### `Log(options?)`

Decorator that can be applied to classes or methods. When applied to a class, by default all methods are logged.

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `onInvoke` | `boolean` | `false` | Log method invocation before execution |
| `args` | `(...args) => any` | `undefined` | Custom function to format logged arguments |
| `result` | `true \| (result) => any` | `undefined` | Include and optionally format successful method result |

### `NoLog()`

Method decorator that excludes a method from class-level `@Log()` logging.

### `Catch(config)`

Method decorator that wraps a method in a declarative `try/catch`. 

### `Validate(config)`

Method decorator that runs a validation callback before the decorated method. 

### `ValidateObject(config)`

Method decorator that runs `class-validator` against an extracted object before the decorated method. 

## Advanced

### Log Format

All logs are structured JSON objects:

#### Success Log

```typescript
{
  method: 'methodName',
  state: 'success',
  args: { param1: value1, param2: value2 },
  // Present only when `result` option is configured
  result: { any: 'value' }
}
```

#### Invocation Log (when `onInvoke: true`)

```typescript
{
  method: 'methodName',
  state: 'invoked',
  args: { param1: value1, param2: value2 }
}
```

#### Error Log

```typescript
{
  method: 'methodName',
  state: 'error',
  args: { param1: value1, param2: value2 },
  error: Error | PrettifiedAxiosError
}
```

#### Methods with No Arguments

```typescript
{
  method: 'methodName',
  state: 'success',
  args: undefined
}
```

### Error Handling

#### Standard Errors

Regular JavaScript errors are logged as-is and re-thrown:

```typescript
@Log()
processPayment(amount: number) {
  if (amount <= 0) {
    throw new Error('Invalid amount');
  }
  return { status: 'success' };
}
```

**Log Output:**

```json
{
  method: 'processPayment',
  state: 'error',
  args: { amount: -10 },
  error: {
    message: 'Invalid amount',
    stack: '...',
    ...
  }
}
```

#### Axios Errors (Auto-Prettified)

Axios errors are automatically formatted with structured request/response info:

```typescript
@Log()
async fetchData(url: string) {
  const response = await this.httpClient.get(url);
  return response.data;
}
```

**Prettified Axios Error Output:**

```typescript
{
  method: 'fetchData',
  state: 'error',
  args: { url: 'http://api.example.com/data' },
  error: {
    name: 'AxiosError',
    error: 'Request failed with status code 404',
    code: 'ERR_BAD_REQUEST',
    config: {
      method: 'get',
      url: 'http://api.example.com/data',
      headers: { ... },
    },
    response: {
      status: 404,
      statusText: 'Not Found',
      data: { message: 'Resource not found' },
      headers: { ... }
    }
  }
}
```

### Example

```typescript
import { Injectable, Logger, BadRequestException } from '@nestjs/common';
import { AxiosError } from 'axios';
import { IsNotEmpty, IsString } from 'class-validator';
import { Log, NoLog, Catch, ValidateObject } from 'nestjs-decorators';

class CreateOrderDto {
  @IsNotEmpty()
  @IsString()
  userId!: string;

  @IsNotEmpty()
  items!: OrderItem[];
}

@Log()
@Injectable()
export class OrderService {
  // Optional: explicit logger takes precedence over auto-injected one
  readonly logger = new Logger(OrderService.name);

  constructor(
    readonly orderRepo: OrderRepository,
    readonly paymentGateway: PaymentGateway,
    readonly externalApi: ExternalApiClient,
  ) {}

  // Logged + validates the DTO with class-validator before running
  @ValidateObject<[CreateOrderDto]>({
    extract: (dto) => dto,
    handleErrors: (_args, _errors, messages) => {
      throw new BadRequestException(messages);
    },
  })
  async createOrder(dto: CreateOrderDto) {
    return await this.orderRepo.create(dto);
  }

  // Logged with invocation + custom args (exclude sensitive card data)
  @Log({
    onInvoke: true,
    args: (orderId: number, _cardDetails: CardDetails) => ({ orderId }),
  })
  async processPayment(orderId: number, cardDetails: CardDetails) {
    const result = await this.paymentGateway.charge(orderId, cardDetails);
    return result;
  }

  // Not logged - internal helper
  @NoLog()
  calculateTotal(items: OrderItem[]): number {
    return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  // Logged + transparently swallows 404s from the external API
  @Catch({
    on: (e) => e instanceof AxiosError && e.response?.status === 404,
    handle: () => null,
    message: 'Remote order not found',
  })
  async syncWithExternalSystem(orderId: number) {
    const response = await this.externalApi.post('/orders', { orderId });
    return response.data;
  }
}
```
