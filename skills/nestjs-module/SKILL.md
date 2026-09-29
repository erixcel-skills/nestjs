---
name: nestjs-module
description: |
  ESPAÑOL - Guía para CREAR MÓDULOS, SERVICIOS y CONTROLADORES en NestJS.
  Usa esta skill cuando el usuario pida: crear módulo, nuevo servicio, nuevo controlador,
  crear endpoint, nueva ruta API, implementar lógica de negocio, agregar método al servicio,
  crear CRUD, exponer API REST, inyección de dependencias, o cualquier tarea
  relacionada con la capa de aplicación (service, controller, module).
  Define responsabilidades y patrones de arquitectura limpia.
---

# NestJS Module

This skill defines the responsibilities of the core module files. A module consists of:

1.  **Service** (`.service.ts`): Business logic, calculations, and DTO transformation.
2.  **Controller** (`.controller.ts`): HTTP routing, documentation (Swagger), and request delegation.
3.  **Module** (`.module.ts`): Dependency Injection wiring.

---

## 1. Service Pattern (`.service.ts`)

For comprehensive service guidelines, in-memory enrichment patterns, and tracing standards, see the `nestjs-module-service` skill.

**Responsibilities**:

- Decorates **every method** with `@Trace` from `traceflow`.
- Injects **Queries** (`@queries/*`) to access data.
- Implements business rules and input sanitization (using `StringFunction`).
- Formats pagination with `PaginationFunction`.
- **NEVER** accesses the database directly (always use Queries).

**Example**: `user.service.ts`

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { UserQuery } from '@queries/user/user.query';
import { UserCreateDto } from './dto/user-create.dto';
import { StringFunction } from '@functions/string.function';
import { Trace } from 'traceflow';
import * as bcrypt from 'bcrypt';

@Injectable()
export class UserService {
  constructor(private readonly userQuery: UserQuery) {}

  @Trace({ name: 'user', type: 'service' })
  async create(dto: UserCreateDto) {
    const sanitized = StringFunction.sanitizeStringFields(dto);
    const hashedPassword = await bcrypt.hash(sanitized.password, 10);
    const fullName = `${sanitized.firstName} ${sanitized.lastName}`;

    const user = await this.userQuery.create({
      ...sanitized,
      password: hashedPassword,
      fullName,
    });

    const { password, ...cleanUser } = user;
    return cleanUser;
  }
}
```

---

## 2. Controller Pattern (`.controller.ts`)

For detailed routing conventions, OpenAPI documentation, and tracing standards, see the `nestjs-module-controller` skill.

**Responsibilities**:

- Decorates **every endpoint** with `@TraceNode` from `traceflow`.
- Defines routes and HTTP verbs (`@Get`, `@Post`, etc.).
- Documents the API using **Swagger** (`@ApiTags`, `@ApiOperation`, `@ApiParam`, `@ApiResponse`).
- Validates inputs using DTOs (`@Body`, `@Query`, `@Param`).
- **Strictly** delegates logic to the Service.
- **NEVER** use `any` in DTOs or parameters.

**Example**: `user.controller.ts`

```typescript
import { Controller, Get, Post, Body, Query, UseGuards } from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse, ApiBearerAuth } from '@nestjs/swagger';
import { UserService } from './user.service';
import { UserListDto, UserListFiltersDto } from './dto/user-list.dto';
import { UserCreateDto } from './dto/user-create.dto';
import { HttpErrorDto } from '@models/http-error.dto';
import { TraceNode } from 'traceflow';

@ApiTags('user')
@ApiBearerAuth()
@Controller('admin/user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get('find-all')
  @ApiOperation({ summary: 'Get all users paginated' })
  @ApiResponse({ status: 200, type: UserListDto })
  @ApiResponse({ status: 400, type: HttpErrorDto })
  @TraceNode({ name: 'Listar usuarios', type: 'controller' })
  findAll(@Query() filters: UserListFiltersDto) {
    return this.userService.findAllPaginated(filters);
  }

  @Post('create')
  @ApiOperation({ summary: 'Create a new user' })
  @ApiResponse({ status: 200, type: UserListDto })
  @ApiResponse({ status: 400, type: HttpErrorDto })
  @TraceNode({ name: 'Crear usuario', type: 'controller' })
  create(@Body() dto: UserCreateDto) {
    return this.userService.create(dto);
  }
}
```

---

## 3. Module Wiring (`.module.ts`)

**Responsibilities**:

- Registers Controllers and Providers (Services, Queries).
- Exports Services/Queries if they need to be used by other modules.

**Example**: `user.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { UserService } from './user.service';
import { UserController } from './user.controller';
import { UserQuery } from '@queries/user/user.query';

@Module({
  controllers: [UserController],
  providers: [UserService, UserQuery],
  exports: [UserService, UserQuery],
})
export class UserModule {}
```

---

## Global Models

Common DTOs like error responses should be placed in `src/models/`.

**File**: `src/models/http-error.dto.ts`

```typescript
import { ApiProperty } from '@nestjs/swagger';

export class HttpErrorDto {
  @ApiProperty({
    description: 'Error message(s)',
    example: ['Validation failed'],
  })
  message: string | string[];

  @ApiProperty({ description: 'Error type', example: 'Bad Request' })
  error: string;

  @ApiProperty({ description: 'Status code', example: 400 })
  statusCode: number;
}
```
