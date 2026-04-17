# Workflow Structure & References

## 📋 Complete Workflow Map

```
reusable-ci.yml (Workflow Principal)
├── Job 1: check-with-lint
│   └── uses: ./.github/actions/check ✅
│
├── Job 2: test-and-coverage
│   └── (Standalone steps, no custom action)
│
├── Job 3: check
│   └── uses: ./.github/actions/check ✅
│
├── Job 4: build-and-push
│   └── uses: ./.github/actions/build ✅
│
└── Job 5: build-docker-image
    └── uses: ./.github/actions/build-and-push-image ✅
```

## 🔗 Actions References

| Job | Action | Path | Descripción |
|-----|--------|------|-------------|
| check | check | `./.github/actions/check` | Lint & SonarCloud |
| build-and-push | build | `./.github/actions/build` | Construir aplicación |
| build-docker-image | build-and-push-image | `./.github/actions/build-and-push-image` | Docker build & push |

## 📁 Directory Structure

```
.github/
│
├── actions/ (CUSTOM ACTIONS)
│   ├── build/
│   │   ├── action.yml            ← Acción para compilar
│   │   └── README.md
│   │
│   ├── check/
│   │   └── action.yml            ← Acción para checks/linting
│   │
│   └── build-and-push-image/
│       ├── action.yml            ← Acción para Docker
│       └── README.md
│
└── workflows/ (WORKFLOWS)
    ├── reusable-ci.yml           ← Workflow principal (usa las actions)
    └── reusable-cd.yml           ← Workflow CD
```

## ✨ Actions Detalles

### 1️⃣ .github/actions/check/action.yml
**Propósito:** Ejecuta linting y análisis SonarCloud

**Inputs:** working-directory, install-command, lint-command, setup-node, setup-python, setup-java, etc.

**Usado por:** Job `check` (línea 250)

```yaml
- uses: ./.github/actions/check
  with:
    working-directory: ${{ inputs.working-directory }}
    install-command: ${{ inputs.install-command }}
    lint-command: ${{ inputs.lint-command }}
    # ... más inputs
```

---

### 2️⃣ .github/actions/build/action.yml
**Propósito:** Compila la aplicación

**Inputs:** working-directory, build-command, install-command, artifact-path, setup-node, setup-python, setup-java, etc.

**Outputs:** artifact-name

**Usado por:** Job `build-and-push` (línea 277)

```yaml
- uses: ./.github/actions/build
  with:
    working-directory: ${{ inputs.working-directory }}
    build-command: ${{ inputs.build-command }}
    install-command: ${{ inputs.install-command }}
    artifact-path: ${{ inputs.artifact-path }}
    # ... más inputs
```

---

### 3️⃣ .github/actions/build-and-push-image/action.yml
**Propósito:** Build y push de imágenes Docker a múltiples registros

**Inputs:** image-name, dockerfile-path, context, registry, registry-username, registry-password, etc.

**Outputs:** image-digest, image-url

**Usado por:** Job `build-docker-image` (línea 330)

```yaml
- uses: ./.github/actions/build-and-push-image
  with:
    image-name: ${{ inputs.docker-image-name }}
    dockerfile-path: ${{ inputs.dockerfile-path }}
    context: ${{ inputs.working-directory }}
    registry: ${{ inputs.docker-registry }}
    registry-username: ${{ steps.creds.outputs.registry_username }}
    registry-password: ${{ steps.creds.outputs.registry_password }}
    # ... más inputs
```

---

## 🔄 Job Dependencies & Flow

```
check-with-lint
    ↓ (si lint-command != '')
test-and-coverage
    ↓ (siempre)
check
    ↓ (si setup-java)
build-and-push
    ↓ (si docker-image-name != '')
build-docker-image
    ✅ COMPLETE
```

---

## 🚀 Credential Resolution in build-docker-image

El job `build-docker-image` selecciona credenciales automáticamente basado en `docker-registry`:

```bash
Prepare credentials (paso intermedio)
│
├─ si registry == 'docker'
│  └─ usa DOCKERHUB_USERNAME + DOCKERHUB_TOKEN
│
├─ si registry == 'ghcr'
│  └─ usa github.actor + GITHUB_TOKEN
│
├─ si registry == 'acr'
│  └─ usa ACR_USERNAME + ACR_PASSWORD
│
└─ si registry == 'custom'
   └─ usa ACR_USERNAME + ACR_PASSWORD
```

---

## ✅ Verificación de Referencias

```
✅ check-with-lint → uses: ./.github/actions/check (línea 250)
✅ build-and-push → uses: ./.github/actions/build (línea 277)
✅ build-docker-image → uses: ./.github/actions/build-and-push-image (línea 330)
```

Todas las referencias son correctas y relativas al repositorio.

---

## 📝 Outputs Workflow

| Output | Fuente | Descripción |
|--------|--------|-------------|
| artifact-name | build-and-push | Nombre del artefacto compilado |
| image-digest | build-docker-image | Digest SHA256 de la imagen Docker |
| image-url | build-docker-image | URL completa de la imagen con registro |

---

## 🎯 Uso en Repositorio Consumer

```yaml
name: My App CI/CD

on: [push, pull_request]

jobs:
  ci:
    uses: my-org/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'npm install'
      build-command: 'npm run build'
      test-command: 'npm test'
      lint-command: 'npm run lint'
      setup-node: true
      docker-image-name: 'myusername/myapp'
      docker-registry: 'docker'
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

---

## 🔍 Validación Final

Todas las actions están:
- ✅ En el directorio correcto: `.github/actions/`
- ✅ Referenciadas correctamente en el workflow
- ✅ Fuera de la carpeta `workflows/`
- ✅ Con documentación README.md
- ✅ Con definición action.yml válida
