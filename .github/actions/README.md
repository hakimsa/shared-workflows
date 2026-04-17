# Custom GitHub Actions

Este directorio contiene las actions personalizadas reutilizables del proyecto.

## 📦 Actions Disponibles

### 1. build
**Directorio:** `./build`

Construye la aplicación con soporte para múltiples lenguajes.

- **Lenguajes:** Node.js, Python, Java
- **Entradas:** build-command, install-command, artifact-path, setup-node, setup-python, setup-java
- **Salidas:** artifact-name
- **Usado en:** Workflow `reusable-ci.yml` - Job `build-and-push`

### 2. check
**Directorio:** `./check`

Ejecuta linting y análisis SonarCloud.

- **Herramientas:** Linting, SonarCloud
- **Entradas:** lint-command, install-command, sonar-token, setup-node, setup-python, setup-java
- **Usado en:** Workflow `reusable-ci.yml` - Job `check`

### 3. build-and-push-image
**Directorio:** `./build-and-push-image`

Construye y pushea imágenes Docker a múltiples registros.

- **Registros:** Docker Hub, GHCR, Azure Container Registry, Custom
- **Características:** Multi-plataforma, Caching automático
- **Entradas:** image-name, dockerfile-path, registry, registry-username, registry-password
- **Salidas:** image-digest, image-url
- **Usado en:** Workflow `reusable-ci.yml` - Job `build-docker-image`

## 📖 Documentación

Cada action tiene su propio `README.md`:

- [build/README.md](./build/README.md)
- [check/README.md](./check/README.md)
- [build-and-push-image/README.md](./build-and-push-image/README.md)

## 🔄 Diagrama de Flujo

```
Workflow Input Parameters
        ↓
    ┌───────────────────────────────────┐
    │    reusable-ci.yml Workflow       │
    └───────────────────────────────────┘
        ↓
    ┌─────────────────────────────┐
    │ Job: check-with-lint        │
    │ └─ Action: check            │
    └─────────────────────────────┘
        ↓
    ┌─────────────────────────────┐
    │ Job: test-and-coverage      │
    │ (no custom action)          │
    └─────────────────────────────┘
        ↓
    ┌─────────────────────────────┐
    │ Job: check                  │
    │ └─ Action: check            │
    └─────────────────────────────┘
        ↓
    ┌─────────────────────────────┐
    │ Job: build-and-push         │
    │ └─ Action: build            │
    │   Output: artifact-name     │
    └─────────────────────────────┘
        ↓
    ┌─────────────────────────────────────────┐
    │ Job: build-docker-image                 │
    │ └─ Action: build-and-push-image         │
    │   Output: image-digest, image-url       │
    └─────────────────────────────────────────┘
```

## 🚀 Desarrollo de Actions

Para crear una nueva action:

1. Crea un directorio: `mkdir ./nombre-action`
2. Crea `action.yml` con la definición de la action
3. Implementa los steps en formato composite (shell scripts)
4. Crea `README.md` con documentación
5. Referencia en el workflow usando `uses: ./.github/actions/nombre-action`

### Template action.yml

```yaml
name: 'Action Name'
description: 'Action description'

inputs:
  param1:
    required: true
    description: 'Parameter description'
  param2:
    required: false
    default: 'default value'
    description: 'Optional parameter'

outputs:
  result:
    description: 'Output description'
    value: ${{ steps.step-id.outputs.result }}

runs:
  using: 'composite'
  steps:
    - name: Step 1
      shell: bash
      run: echo "Hello"
    
    - name: Step 2
      id: step-id
      shell: bash
      run: echo "result=value" >> $GITHUB_OUTPUT
```

## 📝 Convenciones

- **Nombres:** kebab-case (ej: `build-and-push-image`)
- **action.yml:** Siempre incluir name, description, inputs, outputs, runs
- **shell:** Usar `shell: bash` para paso portabilidad
- **outputs:** Usar `$GITHUB_OUTPUT` para pasar datos entre pasos
- **Documentación:** README.md con ejemplos de uso

## 🔗 Referencias en el Workflow

Las actions se referencian usando rutas relativas:

```yaml
uses: ./.github/actions/nombre-action
```

Esto funciona porque GitHub Actions busca el directorio `.github/actions/` relativo al repositorio.

## ✅ Checklist para Nueva Action

- [ ] Crear directorio en `.github/actions/`
- [ ] Crear `action.yml` con inputs/outputs
- [ ] Crear `README.md` con documentación
- [ ] Implementar steps en formato composite
- [ ] Testar localmente en un workflow
- [ ] Documentar ejemplos de uso
- [ ] Actualizar este README
