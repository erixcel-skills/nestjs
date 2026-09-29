---
name: nestjs-swagger
description: |
  ESPAÑOL - Guía para configurar Swagger y validación global en NestJS.
  Úsala cuando se solicite configurar Swagger, OpenAPI, ValidationPipe,
  class-transformer, class-validator, main.ts, autenticación Bearer en la
  documentación o validación automática de datos de entrada. Incluye la
  configuración de swagger.core.ts y transformer.core.ts.
---

# NestJS Swagger & Validation Setup

This skill provides the standard configuration for API documentation and input validation.

## 1. Install Dependencies

```bash
npm install @nestjs/swagger class-transformer class-validator
```

## 2. Core Configuration

Create these files in `src/core/` to modularize the setup.

### Swagger Configuration

**File**: `src/core/swagger.core.ts`

```typescript
import { INestApplication } from '@nestjs/common';
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';

export function setupSwagger(app: INestApplication) {
  const config = new DocumentBuilder()
    .setTitle('API Documentation')
    .setDescription('API endpoints for backend-cachorros')
    .setVersion('1.0')
    .addBearerAuth()
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);
}
```

### Global Validation Pipe

**File**: `src/core/transformer.core.ts`

```typescript
import { INestApplication, ValidationPipe } from '@nestjs/common';

export function setupTransformer(app: INestApplication) {
  app.useGlobalPipes(
    new ValidationPipe({
      transform: true,
      transformOptions: {
        enableImplicitConversion: true,
      },
      whitelist: true,
    }),
  );
}
```

## 3. Main Integration

Update `src/main.ts` to apply the configurations.

```typescript
import { setupSwagger } from '@core/swagger.core';
import { setupTransformer } from '@core/transformer.core';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  setupTransformer(app);
  setupSwagger(app);

  await app.listen(3000);
}
bootstrap();
```
