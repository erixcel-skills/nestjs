# NestJS Skills

## Organización

Cada directorio `skills/<nombre>/` es una skill y un paquete APM independiente: contiene `SKILL.md` y `apm.yml` con su propia `version`. El repositorio no tiene una versión global.

## Agregar o publicar una skill

1. Crea `skills/<nombre>/SKILL.md` con frontmatter `name` (igual al directorio) y `description`.
2. Crea `skills/<nombre>/apm.yml` con `name`, `version` (SemVer) y `description`.
3. Para cada publicación, haz commit y etiqueta ese commit con `<nombre>-v<versión>`. Al modificar una skill existente, incrementa **solo su versión**. Publica el commit y la etiqueta; por ejemplo, para la versión inicial: `git tag nestjs-module-v0.1.0` y `git push origin nestjs-module-v0.1.0`.

Cada etiqueta apunta a un commit del repositorio completo, pero APM instala únicamente el subdirectorio declarado en `path`.

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

## Actualizar una skill

Para buscar una nueva etiqueta dentro del rango declarado, ejecuta `apm update nestjs-module` en el consumidor. Si la nueva versión queda fuera del rango, cambia solo su `ref` en `apm.yml` y ejecuta `apm install`. Guarda el `apm.lock.yaml` actualizado.
