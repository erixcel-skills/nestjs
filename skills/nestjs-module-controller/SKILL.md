---
name: nestjs-module-controller
description: |
  ESPAÑOL - Guía para CREAR CONTROLADORES (*.controller.ts) en NestJS.
  Usa esta skill cuando el usuario pida: crear controlador, endpoints API REST,
  trazabilidad con @TraceNode (traceflow), documentación OpenAPI/Swagger,
  decoradores HTTP (@Get, @Post, @Patch, @Delete), guards de autenticación o enrutamiento.
---

# NestJS Controller (`*.controller.ts`)

This skill defines the essential standards for creating and maintaining controllers in NestJS.

## Core Rules

1. **`@TraceNode` on EVERY Endpoint**:
   - Every route handler **MUST** be decorated with `@TraceNode` from `traceflow`:
     ```typescript
     @TraceNode({
       name: '<HumanReadableAction>', // e.g. 'Listar baños', 'Crear baño'
       type: 'controller',
     })
     ```

2. **Class-Level Standards**:
   - `@ApiTags('<feature>')`: OpenAPI group tag.
   - `@ApiBearerAuth()`: Requires JWT authentication.
   - `@UseGuards(JwtAuthGuard)`: Enforces route protection.
   - `@Controller('admin/<feature>')`: Route prefix.

3. **Complete OpenAPI / Swagger Documentation**:
   - `@ApiOperation({ summary: '...' })` for every route.
   - `@ApiParam({ name: 'id', type: 'number', description: '...' })` whenever `:id` is in the path.
   - `@ApiResponse({ status: 200, type: ResponseDto })` and `@ApiResponse({ status: 400, type: HttpErrorDto })`.

4. **Strict Delegation & Casting**:
   - The controller contains **no business logic**; it validates via DTOs and immediately delegates to the service.
   - Convert numeric path parameters with `+id` when passing to service.

5. **Sub-entity Route Prefixing**:
   - If a controller manages sub-entities, prefix their routes (e.g. `type/find-all`, `type/create`, `type/update/:id`).

---

## Pattern Example

```typescript
import { Controller, Get, Post, Body, Patch, Param, Delete, Query, UseGuards } from '@nestjs/common';
import { ApiTags, ApiOperation, ApiParam, ApiResponse, ApiBearerAuth } from '@nestjs/swagger';
import { HttpErrorDto } from 'src/dto/http-error.dto';
import { BathService } from './bath.service';
import { BathCreateDto } from './dto/bath/bath-create.dto';
import { BathUpdateDto } from './dto/bath/bath-update.dto';
import { BathListDto, BathListFiltersDto } from './dto/bath/bath-list.dto';
import { BathResultDto } from './dto/bath/bath-result.dto';
import { BathTypeListDto, BathTypeListFiltersDto } from './dto/bath-type/bath-type-list.dto';
import { JwtAuthGuard } from '@modules/auth/jwt-auth.guard';
import { TraceNode } from 'traceflow';

@ApiTags('bath')
@ApiBearerAuth()
@UseGuards(JwtAuthGuard)
@Controller('admin/bath')
export class BathController {
  constructor(private readonly bathService: BathService) {}

  // List Paginated
  @Get('find-all')
  @ApiOperation({ summary: 'Get all baths paginated' })
  @ApiResponse({ status: 200, type: BathListDto })
  @ApiResponse({ status: 400, type: HttpErrorDto })
  @TraceNode({ name: 'Listar baños', type: 'controller' })
  findAll(@Query() filters: BathListFiltersDto) {
    return this.bathService.findAllPaginated(filters);
  }

  // Find One by ID
  @Get('find-one/:id')
  @ApiOperation({ summary: 'Get a bath by ID' })
  @ApiParam({ name: 'id', type: 'number', description: 'Bath ID' })
  @ApiResponse({ status: 200, type: BathResultDto })
  @ApiResponse({ status: 400, type: HttpErrorDto })
  @TraceNode({ name: 'Obtener baño', type: 'controller' })
  async findOne(@Param('id') id: string) {
    return await this.bathService.findOne(+id);
  }

  // Create
  @Post('create')
  @ApiOperation({ summary: 'Create a new bath' })
  @ApiResponse({ status: 200, type: BathResultDto })
  @ApiResponse({ status: 400, type: HttpErrorDto })
  @TraceNode({ name: 'Crear baño', type: 'controller' })
  create(@Body() dto: BathCreateDto) {
    return this.bathService.create(dto);
  }

  // Update
  @Patch('update/:id')
  @ApiOperation({ summary: 'Update a bath' })
  @ApiParam({ name: 'id', type: 'number', description: 'Bath ID' })
  @ApiResponse({ status: 200, type: BathResultDto })
  @ApiResponse({ status: 400, type: HttpErrorDto })
  @TraceNode({ name: 'Actualizar baño', type: 'controller' })
  update(@Param('id') id: string, @Body() dto: BathUpdateDto) {
    return this.bathService.update(+id, dto);
  }

  // Delete
  @Delete('delete/:id')
  @ApiOperation({ summary: 'Delete a bath' })
  @ApiParam({ name: 'id', type: 'number', description: 'Bath ID' })
  @ApiResponse({ status: 200, type: BathResultDto })
  @ApiResponse({ status: 400, type: HttpErrorDto })
  @TraceNode({ name: 'Eliminar baño', type: 'controller' })
  remove(@Param('id') id: string) {
    return this.bathService.delete(+id);
  }

  // Sub-entity Route Example
  @Get('type/find-all')
  @ApiOperation({ summary: 'Get all bath types paginated' })
  @ApiResponse({ status: 200, type: BathTypeListDto })
  @ApiResponse({ status: 400, type: HttpErrorDto })
  @TraceNode({ name: 'Listar tipos de baño', type: 'controller' })
  findAllBathTypes(@Query() filters: BathTypeListFiltersDto) {
    return this.bathService.findAllBathTypesPaginated(filters);
  }
}
```
