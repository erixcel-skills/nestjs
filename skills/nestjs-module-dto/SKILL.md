---
name: nestjs-module-dto
description: |
  ESPAÑOL - Guía esencial para CREAR DTOs con VALIDACIÓN y DOCUMENTACIÓN Swagger.
  Usa esta skill cuando el usuario pida: crear DTO, validar datos, documentar API,
  trazabilidad con @Trace({ type: "validation" }), class-validator, class-transformer,
  CreateDto, UpdateDto, FilterDto, ListDto o ResultDto.
---

# NestJS Module DTOs

This skill defines the standards for Data Transfer Objects (DTOs) with validation and OpenAPI documentation.

## Core Rules

1. **`@Trace` on DTO Classes**:
   - Every DTO class **MUST** be decorated with `@Trace({ type: "validation" })` from `traceflow`.

2. **Directory Organization (`src/modules/[module]/dto/`)**:
   - **Single entity / flat**: `dto/[name]-[type].dto.ts` (e.g. `dto/dashboard-filter.dto.ts`).
   - **Multi-entity module**: Subfolder per entity `dto/[entity]/[entity]-[type].dto.ts` (e.g. `dto/bath/bath-create.dto.ts`, `dto/bath-type/bath-type-create.dto.ts`).

3. **Strict Typing with Table DTO**:
   - `CreateDto` should implement `Omit<TableDTO, 'id' | 'createdAt' | 'updatedAt' | 'deletedAt'>` to ensure alignment with Drizzle schema.
   - `UpdateDto` extends `PartialType(CreateDto)` from `@nestjs/swagger`.

4. **Synchronize Validation & Documentation**:
   - Required fields: `@ApiProperty(...)` + `@IsNotEmpty()` + type validator (`@IsString()`, `@IsInt()`, etc.).
   - Optional fields: `@ApiPropertyOptional(...)` + `@IsOptional()`.
   - Query params & Dates: Use `@Type(() => Number)` or `@Type(() => Date)` from `class-transformer`.

---

## 1. Create DTO (`*-create.dto.ts`)

```typescript
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';
import { IsNotEmpty, IsString, IsOptional, IsEnum, IsArray, IsInt, IsDate } from 'class-validator';
import { Type } from 'class-transformer';
import { bathStatusEnum, BathDTO } from '@db/tables/bath.table';
import { Trace } from 'traceflow';

@Trace({ type: "validation" })
export class BathCreateDto implements Omit<BathDTO, 'id' | 'createdAt' | 'updatedAt' | 'deletedAt'> {
  @ApiProperty({ example: 1, description: 'Pet ID' })
  @IsNotEmpty()
  @IsInt()
  petId: number;

  @ApiProperty({ example: '2024-12-01T10:00:00Z', description: 'Scheduled date' })
  @IsNotEmpty()
  @Type(() => Date)
  @IsDate()
  scheduledDate: Date;

  @ApiPropertyOptional({
    enum: bathStatusEnum.enumValues,
    default: bathStatusEnum.enumValues[0],
    description: 'Bath status',
  })
  @IsOptional()
  @IsEnum(bathStatusEnum.enumValues)
  status?: (typeof bathStatusEnum.enumValues)[number];

  @ApiPropertyOptional({ example: ['https://photo.jpg'], description: 'Photos array' })
  @IsOptional()
  @IsArray()
  @IsString({ each: true })
  photos_before?: string[];

  @ApiPropertyOptional({ example: 'Some notes', description: 'Additional notes' })
  @IsOptional()
  @IsString()
  notes?: string;
}
```

---

## 2. Update DTO (`*-update.dto.ts`)

```typescript
import { PartialType } from '@nestjs/swagger';
import { BathCreateDto } from './bath-create.dto';
import { Trace } from 'traceflow';

@Trace({ type: "validation" })
export class BathUpdateDto extends PartialType(BathCreateDto) {}
```

---

## 3. Filter / Query DTO (`*-filter.dto.ts` or `*-list.dto.ts`)

```typescript
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';
import { IsDateString, IsNumber, IsOptional, IsIn, IsInt, Min } from 'class-validator';
import { Type } from 'class-transformer';
import { Trace } from 'traceflow';

@Trace({ type: "validation" })
export class DashboardFilterDto {
  @ApiProperty({ example: '2024-01-01', description: 'Start date YYYY-MM-DD' })
  @IsDateString()
  startDate: string;

  @ApiProperty({ example: '2024-01-31', description: 'End date YYYY-MM-DD' })
  @IsDateString()
  endDate: string;

  @ApiPropertyOptional({ example: 1, description: 'Branch ID filter' })
  @IsOptional()
  @Type(() => Number)
  @IsNumber()
  branchId?: number;

  @ApiPropertyOptional({ example: 1, default: 1 })
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page?: number = 1;

  @ApiPropertyOptional({ example: 10, default: 10 })
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  limit?: number = 10;

  @ApiPropertyOptional({ enum: ['all', 'bath', 'treatment'], default: 'all' })
  @IsOptional()
  @IsIn(['all', 'bath', 'treatment'])
  type?: 'all' | 'bath' | 'treatment' = 'all';
}
```
