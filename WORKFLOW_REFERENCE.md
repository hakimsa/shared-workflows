# Reusable CI Workflow Reference

## Overview

El workflow `reusable-ci.yml` es un workflow genérico y reutilizable que automatiza el ciclo completo de CI/CD:

1. **LINT** - Análisis de código estático
2. **TESTS** - Ejecución de tests y coverage
3. **CHECK** - Checks adicionales (SonarCloud, etc.)
4. **BUILD** - Construcción de la aplicación
5. **DOCKER** - Construcción y push de imagen Docker

## Estructura de Actions

```
.github/
├── actions/
│   ├── build/
│   │   ├── action.yml         # Construye la aplicación
│   │   └── README.md
│   ├── check/
│   │   └── action.yml         # Lint y SonarCloud
│   └── build-and-push-image/
│       ├── action.yml         # Build y push Docker
│       └── README.md
└── workflows/
    ├── reusable-ci.yml        # Workflow principal
    └── reusable-cd.yml        # CD workflow
```

## Flujo de Jobs

```
check-with-lint ─┐
                 ├─→ test-and-coverage ─┐
                 │                      ├─→ check ─┐
                 │                                  ├─→ build-and-push ─→ build-docker-image
```

## Uso Básico

### Node.js Application

```yaml
name: CI - Node.js
on: [push, pull_request]

jobs:
  ci:
    uses: org/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'npm install'
      build-command: 'npm run build'
      test-command: 'npm test'
      lint-command: 'npm run lint'
      setup-node: true
      node-version: '20'
      artifact-path: 'dist/'
    secrets:
      PACKAGE_TOKEN: ${{ secrets.PACKAGE_TOKEN }}
```

### Python Application

```yaml
name: CI - Python
on: [push, pull_request]

jobs:
  ci:
    uses: org/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'pip install -r requirements.txt'
      build-command: 'python setup.py build'
      test-command: 'pytest'
      lint-command: 'flake8'
      setup-python: true
      python-version: '3.12'
    secrets:
      PACKAGE_TOKEN: ${{ secrets.PACKAGE_TOKEN }}
```

### Java Application

```yaml
name: CI - Java
on: [push, pull_request]

jobs:
  ci:
    uses: org/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'mvn clean install'
      build-command: 'mvn clean package'
      test-command: 'mvn test'
      lint-command: 'mvn checkstyle:check'
      setup-java: true
      java-version: '21'
    secrets:
      PACKAGE_TOKEN: ${{ secrets.PACKAGE_TOKEN }}
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

## Con Docker Image

### Docker Hub

```yaml
jobs:
  ci:
    uses: org/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'npm install'
      build-command: 'npm run build'
      setup-node: true
      # Docker
      docker-image-name: 'username/myapp'
      docker-registry: 'docker'
      dockerfile-path: 'Dockerfile'
      environment: 'production'
      image-tag: 'v1.0.0'
    secrets:
      PACKAGE_TOKEN: ${{ secrets.PACKAGE_TOKEN }}
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

### GitHub Container Registry (GHCR)

```yaml
jobs:
  ci:
    uses: org/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'npm install'
      build-command: 'npm run build'
      setup-node: true
      # Docker GHCR
      docker-image-name: 'myapp'
      docker-registry: 'ghcr'
      dockerfile-path: 'Dockerfile'
      environment: 'production'
    secrets:
      PACKAGE_TOKEN: ${{ secrets.PACKAGE_TOKEN }}
```

### Azure Container Registry (ACR)

```yaml
jobs:
  ci:
    uses: org/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'mvn clean install'
      build-command: 'mvn clean package'
      setup-java: true
      # Docker ACR
      docker-image-name: 'myapp'
      docker-registry: 'acr'
      docker-registry-url: 'myregistry.azurecr.io'
      dockerfile-path: 'Dockerfile'
      environment: 'production'
    secrets:
      PACKAGE_TOKEN: ${{ secrets.PACKAGE_TOKEN }}
      ACR_USERNAME: ${{ secrets.ACR_USERNAME }}
      ACR_PASSWORD: ${{ secrets.ACR_PASSWORD }}
```

## Input Parameters

### Build Configuration
- `install-command` *(required)* - Comando para instalar dependencias
- `build-command` - Comando para compilar
- `test-command` - Comando para tests
- `lint-command` - Comando para linter
- `artifact-path` - Ruta de artefactos (default: `dist/`)

### Environment Setup
- `setup-node` - Setup Node.js (default: false)
- `node-version` - Versión Node.js (default: '20')
- `setup-python` - Setup Python (default: false)
- `python-version` - Versión Python (default: '3.12')
- `setup-java` - Setup Java (default: false)
- `java-version` - Versión Java (default: '21')

### Docker Configuration
- `docker-image-name` - Nombre de la imagen Docker
- `docker-registry` - Tipo de registro: `docker`, `ghcr`, `acr`, `custom` (default: 'docker')
- `docker-registry-url` - URL personalizada del registro (requerido para ACR y custom)
- `dockerfile-path` - Ruta al Dockerfile (default: 'Dockerfile')
- `image-tag` - Tag de la imagen
- `docker-build-args` - Build arguments (multilinea)
- `docker-platforms` - Plataformas soportadas (default: 'linux/amd64')
- `environment` - Entorno (dev, staging, production) (default: 'dev')

### Other
- `working-directory` - Directorio de trabajo (default: '.')
- `run-tests` - Ejecutar tests (default: true)

## Secret Parameters

Según el tipo de registry:

### Docker Hub
- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`

### GitHub Container Registry
- Usa `GITHUB_TOKEN` automáticamente

### Azure Container Registry
- `ACR_USERNAME`
- `ACR_PASSWORD`

### Comunes
- `PACKAGE_TOKEN` - Token para registros de paquetes
- `SONAR_TOKEN` - Token para SonarCloud

## Output Values

```yaml
outputs:
  artifact-name: Nombre del artefacto generado
  image-digest: Digest de la imagen Docker
  image-url: URL completa de la imagen Docker
```

Uso en workflow consumer:

```yaml
jobs:
  ci:
    uses: ./.github/workflows/reusable-ci.yml@main
    with:
      # ... inputs
    secrets:
      # ... secrets

  deploy:
    needs: ci
    runs-on: ubuntu-latest
    steps:
      - run: echo "Image URL: ${{ needs.ci.outputs.image-url }}"
```

## Jobs Detalles

### 1. check-with-lint
- Ejecuta linting si `lint-command` está configurado
- Condición: `inputs.lint-command != ''`

### 2. test-and-coverage
- Depende de: `check-with-lint`
- Ejecuta tests y genera coverage
- Condición: `inputs.run-tests && inputs.test-command != ''`

### 3. check
- Depende de: `test-and-coverage`
- Ejecuta SonarCloud para Java
- Condición: `inputs.lint-command != ''`

### 4. build-and-push
- Depende de: `check-with-lint`, `test-and-coverage`, `check`
- Construye la aplicación usando `./.github/actions/build`
- Condición: `inputs.build-command != ''`
- Outputs: `artifact-name`

### 5. build-docker-image
- Depende de: `build-and-push`
- Construye y pushea imagen Docker usando `./.github/actions/build-and-push-image`
- Selecciona credenciales según `docker-registry`
- Condición: `inputs.docker-image-name != ''`
- Outputs: `image-digest`, `image-url`

## Ejemplo Completo: Node.js + Docker Hub

```yaml
name: CI/CD - Node.js + Docker

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  ci:
    uses: my-org/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      # Build configuration
      install-command: 'npm install'
      build-command: 'npm run build'
      test-command: 'npm test'
      lint-command: 'npm run lint'
      artifact-path: 'dist/'
      
      # Environment
      setup-node: true
      node-version: '20'
      
      # Docker
      docker-image-name: 'mycompany/myapp'
      docker-registry: 'docker'
      dockerfile-path: 'Dockerfile'
      image-tag: ${{ github.ref_name }}
      environment: ${{ github.ref_name == 'main' && 'production' || 'staging' }}
      
    secrets:
      PACKAGE_TOKEN: ${{ secrets.NPM_TOKEN }}
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  notify:
    needs: ci
    runs-on: ubuntu-latest
    steps:
      - run: echo "✅ Build successful! Image: ${{ needs.ci.outputs.image-url }}"
```
