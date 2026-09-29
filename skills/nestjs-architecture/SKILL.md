---
name: nestjs-architecture
description: |
  ESPAÑOL - Guía para la estructura de proyecto, convenciones de archivos y path aliases en NestJS.
  Úsala cuando se solicite definir u organizar carpetas, convenciones de nombres, path aliases
  en tsconfig.json, estructura de módulos, estructura con o sin base de datos, ubicación de
  archivos o sufijos como .core.ts, .function.ts, .dto.ts, .table.ts, .query.ts y .seed.ts.
  Define la estructura de directorios y convenciones de nombres para proyectos NestJS.
---

# NestJS Architecture

This skill defines the directory and file naming conventions for NestJS modules. Use these structures as the strict template for creating new modules.

## File Naming Terminology

All files must strictly follow the suffix pattern `[name].[type].ts` to identify their purpose instantly.

- **Core**: `*.core.ts` (Global configuration/setup, e.g., `swagger.core.ts`)
- **Functions**: `*.function.ts` (Pure utility functions, e.g., `date.function.ts`)
- **Models**: `*.dto.ts` or similar (Shared data models, e.g., `http-error.dto.ts` in `src/models`)
- **Tables**: `*.table.ts` (Drizzle schema definitions within `db/tables`, e.g., `user.table.ts`)
- **Queries (Atomic)**: `*.query.ts` (Atomic table operations, e.g., `user.query.ts` inside `src/queries/user/`)
- **Queries (Joins)**: `*-join.query.ts` (Multi-table join queries using the table as base, e.g., `user-join.query.ts` inside `src/queries/user/`)
- **Seeds**: `*.seed.ts` (Initial data scripts, e.g., `user.seed.ts`)

## 1. Basic Structure (No Database)

Use this structure for modules that do **not** require database persistence.

```text
src/
├── core/           # Global setup and configuration
├── functions/      # Shared standalone helper functions
├── models/         # Shared data models (e.g. error DTOs)
└── modules/        # Feature-specific logic
    └── [feature]/
        ├── dto/
        │   ├── create-[feature].dto.ts
        │   └── update-[feature].dto.ts
        ├── [feature].controller.ts
        ├── [feature].module.ts
        └── [feature].service.ts
```

## 2. Full Structure (With Database)

Use this structure for modules that require database interaction using Drizzle ORM.

### Root Files

```text
drizzle.config.ts
```

### Module Files

```text
src/
├── core/           # Global setup (e.g. swagger.core.ts)
├── functions/      # Shared helpers (e.g. date.function.ts)
├── models/         # Shared models (e.g. http-error.dto.ts)
└── modules/        # Domain features
    └── [feature]/
        ├── dto/
        │   ├── create-[feature].dto.ts
        │   ├── update-[feature].dto.ts
        │   └── filter-[feature].dto.ts
        ├── [feature].controller.ts
        ├── [feature].module.ts
        └── [feature].service.ts
```

### Database & Queries Files (Required)

```text
src/
├── db/
│   ├── tables/
│   │   └── [table-name].table.ts      # Schema definitions
│   ├── config.db.ts                   # Database configuration
│   ├── connection.db.ts               # Connection logic
│   ├── create.db.ts                   # Migration creation script
│   ├── reset.db.ts                    # Database reset script
│   └── seed.db.ts                     # Main seeder entry point
├── queries/                           # Data access layer organized by table
│   └── [table-name]/                  # Table-specific folder (kebab-case)
│       ├── [table-name].query.ts      # Atomic operations (CRUD single table)
│       └── [table-name]-join.query.ts # Relational queries (joins with other tables)
└── seeds/
    └── [table-name].seed.ts           # Module-specific seed data
```

### Queries Layer Conventions

The `src/queries/` directory replaces generic repositories by separating atomic single-table actions from relational join queries. Under `src/queries/[table-name]/`:

1. **Atomic Queries (`[table-name].query.ts`)**:
   - Class name: `[TableName]Query` (e.g., `BathQuery`, `UserQuery`).
   - Decorated with `@Injectable()`.
   - Dedicated exclusively to single-table operations: `create`, `update`, `delete`, `findOne`, `findAllPaginated`, `getSummary`, etc. No joins to other tables.

2. **Join Queries (`[table-name]-join.query.ts`)**:
   - Class name: `[TableName]JoinQuery` (e.g., `BathJoinQuery`, `UserJoinQuery`).
   - Decorated with `@Injectable()`.
   - Uses the table as the base query (`.from(table)`) and performs relational joins (`innerJoin`, `leftJoin`, `rightJoin`, etc.) with other related tables.

## Path Aliases (tsconfig.json)

Ensure `tsconfig.json` is configured with these strict path aliases:

```json
{
  "compilerOptions": {
    "paths": {
      "@core/*": ["src/core/*"],
      "@models/*": ["src/models/*"],
      "@db/*": ["src/db/*"],
      "@seeds/*": ["src/seeds/*"],
      "@modules/*": ["src/modules/*"],
      "@queries/*": ["src/queries/*"],
      "@functions/*": ["src/functions/*"]
    }
  }
}
```
