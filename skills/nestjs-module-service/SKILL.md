---
name: nestjs-module-service
description: |
  ESPAÑOL - Guía para CREAR SERVICIOS (*.service.ts) en NestJS.
  Usa esta skill cuando se solicite crear un servicio, implementar lógica de negocio,
  trazabilidad con @Trace (traceflow), inyectar queries, enriquecimiento de datos en memoria,
  sanitización con StringFunction o paginación con PaginationFunction.
---

# NestJS Service (`*.service.ts`)

This skill defines the essential standards for creating and maintaining services in NestJS.

## Core Rules

1. **`@Trace` on EVERY method**:
   - **All methods** (public and private) **MUST** be decorated with `@Trace` from `traceflow`.
   - **`type: 'service'`**: ONLY for methods that are called directly by the Controller (`findAllPaginated`, `findOne`, `create`, `update`, `delete`).
   - **`type: 'transformation'`** or **`type: 'method'`**: For internal methods or private helpers NOT called directly by the Controller (e.g. `enrichBaths`, formatters, calculations).
   - Multi-entity services adjust `name` to the specific entity (e.g. `'bath'` vs `'bathType'`).

2. **Access Data via Queries (`@queries/*`)**:
   - Never import database connection or write raw SQL in services; always inject table Query classes.

3. **Input Sanitization**:
   - In `create` and `update`, sanitize payloads with `StringFunction.sanitizeStringFields(data)`.

4. **Standard Pagination**:
   - Extract `{ page, limit, ...rest }` from filters and return `{ data, meta: PaginationFunction.createPaginationMeta(total, page, limit) }`.

5. **In-Memory Enrichment (Prevent N+1)**:
   - Collect unique IDs with `new Set(...)`, query dependencies in parallel with `Promise.all(...)`, and index with `new Map(...)` for O(1) correlation.

---

## Pattern Example

```typescript
import { Injectable } from '@nestjs/common';
import { BathQuery } from '@queries/bath/bath.query';
import { PetQuery } from '@queries/pet/pet.query';
import type { Bath } from '@db/tables/bath.table';
import { BathCreateDto } from './dto/bath/bath-create.dto';
import { BathUpdateDto } from './dto/bath/bath-update.dto';
import { BathListFiltersDto } from './dto/bath/bath-list.dto';
import { PaginationFunction } from '@functions/pagination.function';
import { StringFunction } from '@functions/string.function';
import { Trace } from 'traceflow';

@Injectable()
export class BathService {
  constructor(
    private readonly bathQuery: BathQuery,
    private readonly petQuery: PetQuery,
  ) {}

  // Methods called by Controller -> type: 'service'
  @Trace({ name: 'bath', type: 'service' })
  async findAllPaginated(filters: BathListFiltersDto) {
    const { page, limit, ...rest } = filters;
    const { data, total } = await this.bathQuery.findAllPaginated(page, limit, rest);
    const enriched = await this.enrichBaths(data);

    return {
      data: enriched,
      meta: PaginationFunction.createPaginationMeta(total, page, limit),
    };
  }

  @Trace({ name: 'bath', type: 'service' })
  async findOne(id: number) {
    const result = await this.bathQuery.findOne(id);
    return result ? (await this.enrichBaths([result]))[0] : result;
  }

  @Trace({ name: 'bath', type: 'service' })
  create(data: BathCreateDto) {
    return this.bathQuery.create(StringFunction.sanitizeStringFields(data));
  }

  @Trace({ name: 'bath', type: 'service' })
  update(id: number, data: BathUpdateDto) {
    return this.bathQuery.update(id, StringFunction.sanitizeStringFields(data));
  }

  @Trace({ name: 'bath', type: 'service' })
  delete(id: number) {
    return this.bathQuery.delete(id);
  }

  // Internal helper method NOT called by Controller -> type: 'transformation' (or 'method')
  @Trace({ name: 'bath', type: 'transformation' })
  private async enrichBaths(data: Bath[]) {
    const petIds = [...new Set(data.map((item) => item.petId))];
    const pets = await this.petQuery.findByIds(petIds);
    const petsById = new Map(pets.map((pet) => [pet.id, pet]));

    return data.map((item) => ({
      ...item,
      petName: petsById.get(item.petId)?.name || '',
    }));
  }
}
```
