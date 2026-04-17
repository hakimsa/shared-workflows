# 📋 Workflows para Repo Consumidor (bakend-mgt)

Esta guía te muestra cómo configurar 3 workflows coordinados en tu repo consumidor.

## 🏗️ Estructura de Workflows

```
bakend-mgt/.github/workflows/
├── pr-validation.yml          ← Valida PRs
├── auto-merge-pr.yml          ← Auto merge si todo está bien
└── develop-ci-cd.yml          ← Build & deploy en develop
```

## 1️⃣ PR Validation Workflow

**Archivo:** `bakend-mgt/.github/workflows/pr-validation.yml`

```yaml
name: PR Validation

on:
  pull_request:
    branches: [develop]
    types: [opened, synchronize]

jobs:
  # Usa el workflow reutilizable para validar la PR
  ci:
    uses: hakimsa/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'mvn clean install'
      build-command: 'mvn clean package'
      test-command: 'mvn test'
      lint-command: 'mvn checkstyle:check'
      setup-java: true
      java-version: '21'
      # No hacer build de Docker en PRs
      docker-image-name: ''
    secrets:
      PACKAGE_TOKEN: ${{ secrets.PACKAGE_TOKEN }}
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  # Mark as success si todo pasó
  mark-success:
    needs: ci
    runs-on: ubuntu-latest
    if: success()
    steps:
      - run: echo "✅ PR validation passed!"
```

---

## 2️⃣ Auto Merge Workflow

**Archivo:** `bakend-mgt/.github/workflows/auto-merge-pr.yml`

```yaml
name: Auto Merge PR

on:
  workflow_run:
    workflows: ["PR Validation"]
    types: [completed]

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    # Solo si PR Validation pasó
    if: github.event.workflow_run.conclusion == 'success'
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}

      # Obtener la PR asociada
      - name: Get PR number
        id: get-pr
        run: |
          PR_NUMBER=$(gh pr list \
            --state open \
            --base develop \
            --head ${{ github.event.workflow_run.head_branch }} \
            --json number \
            --jq '.[0].number')
          echo "pr_number=${PR_NUMBER}" >> $GITHUB_OUTPUT
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      # Auto merge a develop
      - name: Auto merge PR
        if: steps.get-pr.outputs.pr_number != ''
        run: |
          gh pr merge ${{ steps.get-pr.outputs.pr_number }} \
            --auto \
            --squash \
            --delete-branch
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: PR auto-merged
        if: steps.get-pr.outputs.pr_number != ''
        run: echo "✅ PR #${{ steps.get-pr.outputs.pr_number }} merged to develop"
```

---

## 3️⃣ Develop CI/CD Workflow

**Archivo:** `bakend-mgt/.github/workflows/develop-ci-cd.yml`

```yaml
name: Develop CI/CD

on:
  push:
    branches: [develop]
  workflow_run:
    workflows: ["Auto Merge PR"]
    types: [completed]
    branches: [develop]

jobs:
  # Ejecutar checks + tests sobre toda develop
  ci:
    uses: hakimsa/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'mvn clean install'
      build-command: 'mvn clean package'
      test-command: 'mvn test'
      lint-command: 'mvn checkstyle:check'
      setup-java: true
      java-version: '21'
      # AHORA SÍ: build y push de Docker
      docker-image-name: 'hakimsa/bakend-mgt'
      docker-registry: 'docker'
      dockerfile-path: 'Dockerfile'
      image-tag: 'develop-${{ github.sha }}'
      environment: 'staging'
    secrets:
      PACKAGE_TOKEN: ${{ secrets.PACKAGE_TOKEN }}
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  # Notificar resultado
  notify:
    needs: ci
    runs-on: ubuntu-latest
    if: always()
    steps:
      - name: Build completed
        run: |
          echo "## 🎉 Build on develop completed"
          echo ""
          echo "**Status:** ${{ job.status }}"
          echo "**Artifact:** ${{ needs.ci.outputs.artifact-name }}"
          echo "**Image:** ${{ needs.ci.outputs.image-url }}"
          echo "**Digest:** ${{ needs.ci.outputs.image-digest }}"
        
      - name: Upload to deployment system (opcional)
        if: success()
        run: |
          echo "🚀 Triggering deployment to staging..."
          # Aquí puedes llamar a otro sistema de deployment
```

---

## 🔄 Flujo Completo

```
1. Developer hace commit en rama feature
        ↓
2. Crea PR hacia develop
        ↓
3. ⚡ PR VALIDATION inicia
   - Check (linting)
   - Tests
   - SonarCloud
        ↓
4. Si TODO pasó ✅
   - AUTO MERGE activa
   - Merge automático a develop (squash)
   - Elimina rama
        ↓
5. ⚡ DEVELOP CI/CD inicia
   - Check (linting)
   - Tests
   - Build Docker
   - Push a Docker Hub
   - SonarCloud sobre develop
        ↓
6. ✨ Imagen disponible en Docker Hub
```

---

## 🛠️ Configuración Necesaria

### 1. Enable Auto Merge en GitHub

Repo Settings → Pull Requests:
- ✅ Allow auto-merge
- ✅ Auto-merge commits
- ✅ Delete head branches

### 2. Secrets en el Repo Consumidor

Necesitas configurar en Settings → Secrets:

```
PACKAGE_TOKEN          ← Token para registros de paquetes
SONAR_TOKEN            ← SonarCloud token
DOCKERHUB_USERNAME     ← Tu usuario Docker Hub
DOCKERHUB_TOKEN        ← Token de Docker Hub
GITHUB_TOKEN           ← Auto generado (ya existe)
```

### 3. Permisos de Workflow

Settings → Actions → Workflow permissions:
- ✅ Read and write permissions
- ✅ Allow GitHub Actions to create and approve pull requests

---

## 📊 Matriz de Decisiones

| Evento | Workflow | Acciones |
|--------|----------|----------|
| **Crear PR a develop** | PR Validation | Check + Tests |
| **PR con todo OK** | Auto Merge | Merge automático |
| **After Merge** | Develop CI/CD | Check + Tests + Build + Push |
| **Push directo a develop** | Develop CI/CD | Check + Tests + Build + Push |

---

## 🎯 Variantes Según Rama

### Para Rama `main` (Production)

```yaml
name: Main CI/CD

on:
  push:
    branches: [main]

jobs:
  ci:
    uses: hakimsa/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'mvn clean install'
      build-command: 'mvn clean package'
      test-command: 'mvn test'
      lint-command: 'mvn checkstyle:check'
      setup-java: true
      docker-image-name: 'hakimsa/bakend-mgt'
      docker-registry: 'docker'
      image-tag: 'latest,v${{ github.ref_name }}'
      environment: 'production'
    secrets:
      # ... secrets
```

### Para Rama `feature/*` (Development)

```yaml
name: Feature Branch CI

on:
  push:
    branches: [feature/**]

jobs:
  ci:
    uses: hakimsa/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'mvn clean install'
      build-command: 'mvn clean package'
      test-command: 'mvn test'
      setup-java: true
      docker-image-name: ''  # No construir Docker en feature
    secrets:
      # ... secrets
```

---

## ⚙️ Configuración de Git Strategy

Recomendado: Usar **Squash and Merge** para develop

```yaml
# En auto-merge-pr.yml
gh pr merge ${{ steps.get-pr.outputs.pr_number }} \
  --auto \
  --squash \                    # ← Squash commits
  --delete-branch               # ← Limpiar rama
```

---

## 🔔 Notificaciones Opcionales

Agregar notificaciones a Slack/Teams cuando:

1. ✅ PR pasó validación
2. ✅ Merge automático realizado
3. ✅ Imagen publicada en Docker Hub
4. ❌ Algún paso falló

Ejemplo para Slack:

```yaml
- name: Notify Slack
  if: always()
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    text: 'Build ${{ job.status }}: ${{ needs.ci.outputs.image-url }}'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

## 🚀 Quick Start para Repo Consumidor

1. Crea los 3 archivos en `.github/workflows/`:
   - `pr-validation.yml`
   - `auto-merge-pr.yml`
   - `develop-ci-cd.yml`

2. Configura los secrets en Settings → Secrets

3. Enable auto-merge en Settings → Pull Requests

4. Push a GitHub y prueba creando una PR

5. ¡Listo! Los workflows harán todo automáticamente

---

## 📝 Troubleshooting

### ❌ Auto merge no funciona
- Verifica que `GITHUB_TOKEN` tenga permisos correctos
- Verifica que auto-merge esté enabled en repo settings

### ❌ Workflow no se ejecuta
- Verifica que el evento está configurado correctamente
- Revisa los logs en Actions tab

### ❌ Secrets no disponibles
- Verifica que existen en Settings → Secrets
- Verifica que el nombre es exacto (case-sensitive)

---

**¿Necesitas ayuda configurando estos workflows en tu repo?**
