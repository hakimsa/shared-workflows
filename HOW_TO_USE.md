# 🚀 Cómo Usar el Workflow Reusable en tu Repo Java

## Para el Repo Consumidor (bakend-mgt)

### 1. Crear workflow en el repo consumidor

Crea un archivo en tu repo Java:

```
bakend-mgt/
└── .github/
    └── workflows/
        └── ci.yml
```

### 2. Contenido del workflow (ci.yml)

```yaml
name: CI - Java Backend

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  ci:
    # 👇 LLAMAR AL WORKFLOW REUSABLE DEL SHARED-WORKFLOWS
    uses: hakimsa/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      # Build Configuration
      install-command: 'mvn clean install'
      build-command: 'mvn clean package'
      test-command: 'mvn test'
      lint-command: 'mvn checkstyle:check'
      
      # Environment
      setup-java: true
      java-version: '21'
      
      # Docker (Opcional)
      docker-image-name: 'hakimsa/bakend-mgt'
      docker-registry: 'docker'
      dockerfile-path: 'Dockerfile'
      image-tag: ${{ github.ref_name }}
      environment: ${{ github.ref_name == 'main' && 'production' || 'staging' }}
      
    secrets:
      PACKAGE_TOKEN: ${{ secrets.PACKAGE_TOKEN }}
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  # (Opcional) Job para usar los outputs del workflow
  notify:
    needs: ci
    runs-on: ubuntu-latest
    if: success()
    steps:
      - run: |
          echo "✅ Build exitoso!"
          echo "Imagen Docker: ${{ needs.ci.outputs.image-url }}"
          echo "Artifact: ${{ needs.ci.outputs.artifact-name }}"
```

### 3. Qué Sucede Automáticamente

Cuando empujas código, GitHub Actions:

1. ✅ Ejecuta el workflow `reusable-ci.yml` del repo `hakimsa/shared-workflows`
2. ✅ Las actions (`check`, `build`, `build-and-push-image`) se descargan desde shared-workflows
3. ✅ Se ejecutan con los parámetros que proporcionaste
4. ✅ Los outputs están disponibles en tu job `notify`

## 🔄 Flujo Completo

```
Tu Repo (bakend-mgt)
└─ .github/workflows/ci.yml
   └─ Llama a: hakimsa/shared-workflows/.github/workflows/reusable-ci.yml@main
      └─ Usa actions:
         ├─ hakimsa/shared-workflows/.github/actions/check@main
         ├─ hakimsa/shared-workflows/.github/actions/build@main
         └─ hakimsa/shared-workflows/.github/actions/build-and-push-image@main
```

## ⚙️ Variables de Entorno en shared-workflows

El workflow `reusable-ci.yml` ahora usa referencias EXTERNAS:

```yaml
# ✅ CORRECTO - referencias externas
uses: hakimsa/shared-workflows/.github/actions/check@main
uses: hakimsa/shared-workflows/.github/actions/build@main
uses: hakimsa/shared-workflows/.github/actions/build-and-push-image@main
```

Esto permite que:
- 🎯 Se ejecute desde cualquier repo consumidor
- 🔀 Las actions se descarguen del repo shared-workflows
- 🔒 No dependa de rutas locales del consumidor

## 🔐 Requisitos

Para que funcione, asegurate de que:

1. ✅ El repo `hakimsa/shared-workflows` sea **PÚBLICO**
2. ✅ Tengas configurados los `secrets` en tu repo bakend-mgt:
   - `SONAR_TOKEN` (si usas SonarCloud)
   - `DOCKERHUB_USERNAME` y `DOCKERHUB_TOKEN` (si usas Docker)
   - `PACKAGE_TOKEN` (si tienes registros privados)

## 📝 Ejemplo Para Otras Tecnologías

### Node.js

```yaml
jobs:
  ci:
    uses: hakimsa/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'npm install'
      build-command: 'npm run build'
      test-command: 'npm test'
      lint-command: 'npm run lint'
      setup-node: true
      node-version: '20'
      docker-image-name: 'hakimsa/myapp'
      docker-registry: 'docker'
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

### Python

```yaml
jobs:
  ci:
    uses: hakimsa/shared-workflows/.github/workflows/reusable-ci.yml@main
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

## 🎯 Outputs Disponibles

Después de que el workflow termine, puedes usar los outputs:

```yaml
- run: |
    echo "Artifact Name: ${{ needs.ci.outputs.artifact-name }}"
    echo "Image Digest: ${{ needs.ci.outputs.image-digest }}"
    echo "Image URL: ${{ needs.ci.outputs.image-url }}"
```

## ✅ Ventajas de Esta Estructura

| Ventaja | Descripción |
|---------|-------------|
| 🔄 **Reutilizable** | Úsalo en múltiples repos |
| 🚀 **Mantenible** | Actualiza una vez en shared-workflows |
| 📦 **Limpio** | No duplicas código en cada repo |
| 🔒 **Seguro** | Usa secrets de cada repo |
| 🌐 **Universal** | Soporta Node, Python, Java, etc. |

---

**¿Tienes dudas sobre cómo configurar esto en tu repo bakend-mgt?**
