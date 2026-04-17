# Build Action

Acción reutilizable para compilar aplicaciones con soporte para Node.js, Python y Java.

## Descripción

Esta acción automatiza el proceso de compilación de aplicaciones, incluyendo:
- Setup de entornos (Node.js, Python, Java)
- Instalación de dependencias
- Ejecución del build
- Generación y carga de artefactos

## Inputs

### Requeridos

- `build-command` - Comando para compilar la aplicación (ej: `npm run build`, `mvn clean package`)
- `install-command` - Comando para instalar dependencias (ej: `npm install`, `pip install -r requirements.txt`)

### Opcionales

| Input | Descripción | Por defecto |
|-------|-------------|-------------|
| `working-directory` | Directorio de trabajo | `.` |
| `artifact-path` | Ruta a artefactos de build | `dist/` |
| `setup-node` | Activar setup de Node.js | `false` |
| `node-version` | Versión de Node.js | `20` |
| `setup-python` | Activar setup de Python | `false` |
| `python-version` | Versión de Python | `3.12` |
| `setup-java` | Activar setup de Java | `false` |
| `java-version` | Versión de Java | `21` |
| `package-token` | Token para registry de paquetes | `` |

## Outputs

- `artifact-name` - Nombre del artefacto generado (formato: `build-{sha}`)

## Uso en Workflow

```yaml
- name: Build application
  uses: ./.github/actions/build
  with:
    build-command: 'npm run build'
    install-command: 'npm install'
    setup-node: true
    node-version: '20'
    artifact-path: 'dist/'
```

## Integración en reusable-ci.yml

La acción está integrada en el job `build-and-push` del workflow reutilizable `reusable-ci.yml`:

```yaml
- name: Build application
  id: build
  uses: ./.github/actions/build
  with:
    working-directory: ${{ inputs.working-directory }}
    build-command: ${{ inputs.build-command }}
    install-command: ${{ inputs.install-command }}
    artifact-path: ${{ inputs.artifact-path }}
    setup-node: ${{ inputs.setup-node }}
    node-version: ${{ inputs.node-version }}
    setup-python: ${{ inputs.setup-python }}
    python-version: ${{ inputs.python-version }}
    setup-java: ${{ inputs.setup-java }}
    java-version: ${{ inputs.java-version }}
    package-token: ${{ secrets.PACKAGE_TOKEN }}
```

## Ejemplo completo

Para usar el workflow reutilizable con esta acción:

```yaml
name: CI
on: [push, pull_request]

jobs:
  build:
    uses: ./.github/workflows/reusable-ci.yml
    with:
      install-command: 'npm install'
      build-command: 'npm run build'
      test-command: 'npm test'
      setup-node: true
      node-version: '20'
    secrets:
      PACKAGE_TOKEN: ${{ secrets.PACKAGE_TOKEN }}
```
