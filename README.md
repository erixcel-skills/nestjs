# NestJS Skills

## Consumir desde otro proyecto

En el `apm.yml` del proyecto consumidor, declara solo las skills necesarias:

```yaml
name: mi-proyecto
version: "1.0.0"
targets:
  - codex
dependencies:
  apm:
    - git: https://github.com/erixcel-skills/nestjs.git
      path: skills/nestjs-module
      ref: "^0.1.0"
    - git: https://github.com/erixcel-skills/nestjs.git
      path: skills/nestjs-swagger
      ref: "^0.1.0"
```

Ejecuta `apm install`. APM crea `apm.lock.yaml` con las revisiones resueltas y instala las skills para Codex en `.agents/skills/`. Guarda el manifest y el lockfile en Git. En instalaciones posteriores, `apm install --frozen` respeta el lockfile.
