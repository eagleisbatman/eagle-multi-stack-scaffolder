# Node.js + Express Reference

## Research Queries
- "Node.js best practices 2025 2026"
- "Express.js vs Fastify vs Hono 2025"
- "TypeScript Node.js setup 2025"
- "Bun vs Node.js production 2025"

## Package Manager
**Bun** - Faster runtime, built-in TypeScript, native .env support.

```bash
bun init
bun add express @types/express typescript
```

## Project Structure

```
src/
├── index.ts                  # Entry point
├── app.ts                    # Express setup
├── config/
│   ├── index.ts
│   ├── database.ts
│   └── env.ts
├── api/
│   ├── routes/
│   │   ├── index.ts
│   │   └── farmers.routes.ts
│   ├── controllers/
│   │   └── farmers.controller.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   ├── error.ts
│   │   └── validate.ts
│   └── validators/
├── services/
│   └── farmer.service.ts
├── repositories/
│   └── farmer.repository.ts
├── models/
├── types/
└── utils/
    ├── logger.ts
    └── errors.ts
```

## Essential Libraries

```bash
# Core
bun add express cors helmet
bun add @types/express @types/cors typescript

# Database
bun add prisma @prisma/client

# Validation
bun add zod

# Auth
bun add jsonwebtoken bcryptjs
bun add @types/jsonwebtoken @types/bcryptjs

# Logging
bun add pino pino-pretty

# Dev
bun add -D @types/node tsx
```

## Code Patterns

### App Setup
```typescript
// src/app.ts
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import { errorHandler } from './api/middleware/error';
import { apiRouter } from './api/routes';

export function createApp() {
  const app = express();
  app.use(helmet());
  app.use(cors());
  app.use(express.json());
  app.get('/health', (_, res) => res.json({ status: 'ok' }));
  app.use('/api/v1', apiRouter);
  app.use(errorHandler);
  return app;
}
```

### Controller
```typescript
export const farmerController = {
  async getAll(req: Request, res: Response, next: NextFunction) {
    try {
      const farmers = await farmerService.findAll(req.query);
      res.json({ success: true, data: farmers });
    } catch (error) {
      next(error);
    }
  },
};
```

### Service
```typescript
export const farmerService = {
  async findAll({ page = 1, limit = 20 }) {
    return prisma.farmer.findMany({
      skip: (page - 1) * limit,
      take: limit,
      orderBy: { createdAt: 'desc' },
    });
  },
};
```

### Error Handling
```typescript
export class AppError extends Error {
  constructor(message: string, public statusCode = 500) {
    super(message);
  }
  static notFound(msg: string) { return new AppError(msg, 404); }
  static badRequest(msg: string) { return new AppError(msg, 400); }
}
```

## Setup Commands

```bash
mkdir my-api && cd my-api
bun init -y

# Dependencies
bun add express cors helmet zod pino
bun add prisma @prisma/client
bun add @types/express @types/cors typescript

# Prisma
bunx prisma init
bunx prisma migrate dev

# Run
bun --watch src/index.ts
```

## tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "baseUrl": ".",
    "paths": { "@/*": ["src/*"] }
  }
}
```
