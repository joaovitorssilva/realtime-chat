# Agent Instructions for Realtime Chat Backend

This is a NestJS backend project. Below are the guidelines and commands for working in this codebase.

## Project Overview

- **Framework**: NestJS 11.x
- **Language**: TypeScript 5.7.x
- **Package Manager**: pnpm
- **Testing**: Jest 30.x
- **Linting**: ESLint 9.x with typescript-eslint
- **Formatting**: Prettier 3.x

## Commands

### Build & Development

```bash
pnpm build           # Build the application (dist folder)
pnpm start           # Start the application
pnpm start:dev       # Start with watch mode (hot reload)
pnpm start:debug     # Start with debug mode and watch
pnpm start:prod      # Start production build
```

### Code Quality

```bash
pnpm lint            # Lint and auto-fix all TypeScript files
pnpm format          # Format all files with Prettier
pnpm lint:check      # Check linting without auto-fixing
pnpm format:check    # Check formatting without modifying files
```

### Testing

```bash
pnpm test            # Run all tests
pnpm test:watch      # Run tests in watch mode
pnpm test:cov        # Run tests with coverage report
pnpm test:e2e        # Run end-to-end tests
pnpm test:debug      # Run tests with Node debugger
```

#### Running a Single Test

```bash
# Run a specific test file in watch mode
pnpm test:watch -- --testPathPattern=app.controller.spec

# Run a specific test file once
pnpm test -- --testPathPattern=app.controller.spec

# Run a specific test by name
pnpm test -- --testNamePattern="should return"
```

## Code Style Guidelines

### General

- Use 2 spaces for indentation (matching Prettier default)
- Files should end with a newline
- Use `auto` for line endings (handled by Prettier)
- Maximum line length should follow Prettier defaults (printWidth: 80)

### TypeScript Conventions

1. **Type Annotations**
   - Use explicit return types for public methods
   - Prefer `interface` over `type` for object shapes
   - Avoid `any` unless absolutely necessary (ESLint will warn)
   - Use `unknown` for values of unknown type

2. **Null/Undefined**
   - Use `strictNullChecks: true` (enabled in tsconfig)
   - Prefer optional properties (`prop?: Type`) over `| undefined`
   - Use nullish coalescing (`??`) and optional chaining (`?.`) when appropriate

3. **Naming Conventions**
   - Classes: PascalCase (`AppController`, `UserService`)
   - Interfaces: PascalCase with optional `I` prefix (`UserDto`, `IUserRepository`)
   - Variables/functions: camelCase (`getUserById`, `isActive`)
   - Constants: SCREAMING_SNAKE_CASE for true constants
   - Files: kebab-case (`user-service.ts`, `auth.module.ts`)

### Import Conventions

1. **Order** (enforced by typescript-eslint, grouped by empty line):
   - Node.js built-ins (`node:fs`, `node:path`)
   - External packages (`@nestjs/common`, `rxjs`)
   - Internal packages (`@/common/...`)
   - Relative imports (`./service`, `../types`)

2. **Style**
   - Use named imports where possible: `import { Controller, Get } from '@nestjs/common'`
   - Use type imports for type-only: `import type { User } from './types'`
   - Avoid default re-exports when not necessary

### NestJS Patterns

1. **Modules**
   - One module per feature/domain
   - Use `@Global()` decorator for shared modules
   - Keep modules focused, extract concerns

2. **Controllers**
   - One controller per module
   - Use appropriate HTTP verbs (`@Get`, `@Post`, `@Put`, `@Delete`)
   - Validate inputs with class-validator DTOs
   - Return consistent response structures

3. **Services**
   - Single responsibility
   - Use dependency injection via constructor
   - Mark dependencies as `readonly` when not modified

4. **Dependency Injection**

   ```typescript
   // Preferred
   constructor(private readonly userService: UserService) {}

   // Avoid (unless property injection is needed)
   @Inject(UserService)
   userService: UserService;
   ```

### Error Handling

1. **Exceptions**
   - Use built-in NestJS exceptions: `NotFoundException`, `BadRequestException`, etc.
   - Create custom exceptions for domain-specific errors
   - Use `HttpException` with appropriate status codes

2. **Async**
   - Always handle promise rejections
   - Return `Observable` from RxJS streams when using WebSockets
   - Warn on floating promises (ESLint rule enabled)

### Testing Conventions

1. **Unit Tests** (`.spec.ts`)
   - Place in same directory as source file
   - Use `@nestjs/testing` module
   - Mock external dependencies
   - Focus on behavior, not implementation

2. **E2E Tests** (`.e2e-spec.ts`)
   - Place in `/test` directory
   - Use `supertest` for HTTP assertions
   - Test real integration points

3. **Test Structure**

   ```typescript
   describe('FeatureName', () => {
     let service: MyService;

     beforeEach(async () => {
       const module = await Test.createTestingModule({
         providers: [MyService, MockDependency],
       }).compile();

       service = module.get(MyService);
     });

     it('should do something', () => {
       expect(service.doSomething()).toBe('expected');
     });
   });
   ```

## File Organization

```
src/
├── main.ts              # Application entry point
├── app.module.ts        # Root module
├── common/              # Shared utilities, decorators, guards
├── features/            # Feature modules (auth, users, chat, etc.)
│   └── feature-name/
│       ├── feature-name.module.ts
│       ├── feature-name.controller.ts
│       ├── feature-name.service.ts
│       ├── feature-name.service.spec.ts
│       └── dto/
│           └── create-feature.dto.ts
└── config/              # Configuration files
```

## Additional Notes

- Environment variables are loaded via `process.env` with defaults
- Docker Compose available for local development (`docker-compose.yml`)
- Use `strictNullChecks: true` for type safety
