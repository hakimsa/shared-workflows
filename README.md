# 🔄 shared-workflows

Repositorio centralizado de **Reusable Workflows** de GitHub Actions para la organización. Evita duplicar lógica de CI/CD en cada proyecto ofreciendo pipelines configurables y reutilizables.

---

## 📋 Índice

- [Workflows disponibles](#-workflows-disponibles)
- [Reusable CI](#-reusable-ci--integración-continua)
  - [Flujo de jobs](#flujo-de-jobs)
  - [Inputs](#inputs)
  - [Secrets](#secrets)
  - [Outputs](#outputs)
- [Reusable CD](#-reusable-cd--despliegue-a-kubernetes)
  - [Flujo de jobs](#flujo-de-jobs-1)
  - [Inputs](#inputs-1)
  - [Secrets](#secrets-1)
  - [Outputs](#outputs-1)
- [Integración CI + CD](#-integración-ci--cd)
- [Buenas prácticas](#-buenas-prácticas)

---

## 📦 Workflows disponibles

| Archivo | Propósito |
|---|---|
| [`reusable-ci.yml`](.github/workflows/reusable-ci.yml) | Integración continua: lint, análisis estático, tests, build y push Docker |
| [`reusable-cd.yml`](.github/workflows/reusable-cd.yml) | Despliegue continuo: deploy a Kubernetes con Helm o kubectl |

---

## 🏗️ Reusable CI — Integración Continua

Pipeline genérico de CI compatible con **Node.js**, **Python** y **Java**. Automatiza la verificación de calidad del código, ejecución de tests, construcción del artefacto y publicación de imágenes Docker.

### Flujo de jobs

```
check-with-lint ──► check-with-sonar ──► test-and-coverage ──► build-and-push
```

> **Nota:** Si `lint-command` está vacío, los tres primeros jobs se omiten. `build-and-push` se ejecuta siempre que no haya habido fallos previos y `build-command` esté definido.

| Job | Condición de ejecución |
|---|---|
| `check-with-lint` | `lint-command != ''` |
| `check-with-sonar` | `lint-command != ''` (necesita `check-with-lint`) |
| `test-and-coverage` | `run-tests: true` AND `test-command != ''` AND `lint-command != ''` |
| `build-and-push` | `build-command != ''` AND sin fallos previos |

### Inputs

#### General

| Input | Tipo | Requerido | Default | Descripción |
|---|---|---|---|---|
| `working-directory` | `string` | No | `.` | Directorio raíz del proyecto |
| `run-tests` | `boolean` | No | `true` | Habilita o deshabilita los tests |
| `install-command` | `string` | **Sí** | — | Comando de instalación de dependencias |
| `lint-command` | `string` | No | `''` | Comando de linting. Vacío omite lint, sonar y tests |
| `test-command` | `string` | No | `''` | Comando de tests |
| `build-command` | `string` | No | `''` | Comando de build |
| `artifact-path` | `string` | No | `dist/` | Ruta del artefacto generado (relativa a `working-directory`) |

#### Entornos de desarrollo

| Input | Tipo | Requerido | Default | Descripción |
|---|---|---|---|---|
| `setup-node` | `boolean` | No | `false` | Activa el entorno Node.js con cache npm |
| `node-version` | `string` | No | `20` | Versión de Node.js |
| `setup-python` | `boolean` | No | `false` | Activa el entorno Python con cache pip |
| `python-version` | `string` | No | `3.12` | Versión de Python |
| `setup-java` | `boolean` | No | `false` | Activa el entorno Java (Temurin) con cache Maven |
| `java-version` | `string` | No | `21` | Versión de Java |

#### Docker

| Input | Tipo | Requerido | Default | Descripción |
|---|---|---|---|---|
| `docker-image-name` | `string` | No | `''` | Nombre de la imagen. Vacío omite el build/push de Docker |
| `dockerfile-path` | `string` | No | `Dockerfile` | Ruta al Dockerfile |

### Secrets

| Secret | Requerido | Descripción |
|---|---|---|
| `PACKAGE_TOKEN` | No | Token para registros privados de paquetes (npm, PyPI, Maven) |
| `DOCKERHUB_USERNAME` | No | Usuario de Docker Hub (requerido si `docker-image-name` está definido) |
| `DOCKERHUB_TOKEN` | No | Token de Docker Hub (requerido si `docker-image-name` está definido) |
| `SONAR_TOKEN` | No | Token de SonarCloud (requerido para análisis Java con Maven) |
| `KUBE_CONFIG` | No | Kubeconfig en base64 |

### Outputs

| Output | Descripción |
|---|---|
| `artifact-name` | Nombre del artefacto generado: `build-{github.sha}` |

### Ejemplo de uso

```yaml
jobs:
  ci:
    uses: hakimsa/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'npm ci'
      setup-node: true
      node-version: '20'
      lint-command: 'npm run lint'
      test-command: 'npm test'
      build-command: 'npm run build'
      docker-image-name: 'my-app'
    secrets: inherit
```

---

## 🚀 Reusable CD — Despliegue a Kubernetes

Pipeline genérico de CD que despliega aplicaciones en clústeres Kubernetes. Soporta **Helm** y **kubectl** con rollback automático ante fallos.

### Flujo de jobs

```
checkout app ──► checkout helm charts ──► [download artifact] ──► setup kubectl
      ──► [helm upgrade | kubectl apply] ──► get service URL ──► [rollback on failure]
```

### Inputs

| Input | Tipo | Requerido | Default | Descripción |
|---|---|---|---|---|
| `environment` | `string` | **Sí** | — | Entorno de GitHub Actions (ej: `staging`, `production`) |
| `runner` | `string` | No | `ubuntu-latest` | Runner donde se ejecuta el job |
| `helm-release-name` | `string` | **Sí** | — | Nombre del release de Helm |
| `helm-namespace` | `string` | No | `default` | Namespace de Kubernetes |
| `helm-charts-repo` | `string` | No | `hakimsa/helm-releases` | Repositorio de Helm charts |
| `helm-charts-ref` | `string` | No | `main` | Rama o tag del repositorio de charts |
| `image-name` | `string` | No | `''` | Nombre de la imagen Docker (`image.repository` en Helm) |
| `image-tag` | `string` | No | `''` | Tag de la imagen Docker (`image.tag` en Helm) |
| `helm-values-file` | `string` | No | `''` | Ruta al values.yaml. Si vacío usa `{environment}/values.yaml` |
| `deploy-tool` | `string` | No | `helm` | Herramienta de despliegue: `helm` o `kubectl` |
| `kubectl-namespace` | `string` | No | `default` | Namespace para despliegues con kubectl |
| `artifact-name` | `string` | No | `''` | Nombre del artefacto a descargar antes del despliegue |

### Secrets

| Secret | Requerido | Descripción |
|---|---|---|
| `KUBE_CONFIG` | **Sí** | Kubeconfig del clúster codificado en base64 |
| `CHARTS_TOKEN` | No | Token de acceso al repositorio privado de Helm charts |

### Outputs

| Output | Descripción |
|---|---|
| `deploy-url` | IP/URL del LoadBalancer del servicio. Valor `pending` si aún no tiene IP asignada |

### Rollback automático

Si el job `deploy` falla y `deploy-tool` es `helm`, se ejecuta automáticamente:

```bash
helm rollback <helm-release-name> --namespace <helm-namespace>
```

> ⚠️ El rollback automático **no aplica** a despliegues con `kubectl`.

### Ejemplo de uso

```yaml
jobs:
  cd:
    uses: hakimsa/shared-workflows/.github/workflows/reusable-cd.yml@main
    with:
      environment: 'staging'
      helm-release-name: 'my-app'
      helm-namespace: 'staging'
      image-name: 'myorg/my-app'
      image-tag: ${{ github.sha }}
    secrets: inherit
```

---

## 🔗 Integración CI + CD

Encadena ambos workflows pasando el `artifact-name` del CI al CD:

```yaml
jobs:
  ci:
    uses: hakimsa/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'npm ci'
      setup-node: true
      build-command: 'npm run build'
      docker-image-name: 'my-app'
    secrets: inherit

  cd:
    needs: ci
    uses: hakimsa/shared-workflows/.github/workflows/reusable-cd.yml@main
    with:
      environment: 'staging'
      helm-release-name: 'my-app'
      helm-namespace: 'staging'
      image-name: 'myorg/my-app'
      image-tag: ${{ github.sha }}
      artifact-name: ${{ needs.ci.outputs.artifact-name }}
    secrets: inherit
```

---

## ✅ Buenas prácticas

**Versioning estable**
Apunta a un tag en lugar de `@main` para garantizar estabilidad en producción:
```yaml
uses: hakimsa/shared-workflows/.github/workflows/reusable-ci.yml@v1.2.0
```

**Un solo entorno por ejecución**
Activa solo uno de `setup-node`, `setup-python` o `setup-java`. Activar varios no da error, pero instala software innecesario y alarga la ejecución.

**SonarCloud solo para Java/Maven**
El job `check-with-sonar` ejecuta `mvn sonar:sonar` con los parámetros del proyecto `hakimsa`. Para otros proyectos o lenguajes, ajusta `sonar.projectKey` y `sonar.organization` directamente en el workflow.

**Propagación de secrets**
Usa `secrets: inherit` para propagar todos los secrets del repositorio automáticamente. Pásalos de forma explícita solo si necesitas mayor control sobre qué se comparte.

**Dependencia lint → sonar → test**
Si `lint-command` está vacío, también se omiten `check-with-sonar` y `test-and-coverage`. Asegúrate de definirlo si necesitas cobertura de tests o análisis estático.
