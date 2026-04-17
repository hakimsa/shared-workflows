# 🔗 Complete References Diagram

## Workflow Action References

```
.github/workflows/reusable-ci.yml
│
├─── Job 1: check-with-lint
│    ├─ Line 250: uses: ./.github/actions/check ✅
│    └─ Context: Lint stage
│
├─── Job 2: test-and-coverage  
│    └─ (No custom action - inline steps)
│
├─── Job 3: check
│    ├─ Line 250: uses: ./.github/actions/check ✅
│    └─ Context: SonarCloud analysis
│
├─── Job 4: build-and-push
│    ├─ Line 277: uses: ./.github/actions/build ✅
│    ├─ Inputs from: workflow inputs
│    └─ Outputs to: artifact-name
│
└─── Job 5: build-docker-image
     ├─ Line 325: Prepare credentials (helper step)
     ├─ Line 330: uses: ./.github/actions/build-and-push-image ✅
     ├─ Inputs from: workflow inputs + prepared creds
     └─ Outputs to: image-digest, image-url
```

## Data Flow: Inputs → Actions → Outputs

```
┌─────────────────────────────────────────────────────────────┐
│ WORKFLOW INPUTS (from consumer repository)                  │
├─────────────────────────────────────────────────────────────┤
│ • install-command                                           │
│ • build-command                                             │
│ • test-command                                              │
│ • lint-command                                              │
│ • artifact-path                                             │
│ • setup-node, setup-python, setup-java (+ versions)        │
│ • docker-image-name                                         │
│ • docker-registry (docker|ghcr|acr|custom)                │
│ • docker-build-args, docker-platforms                      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │   ACTION: ./.github/actions/check     │
        ├───────────────────────────────────────┤
        │ ✓ Lint execution                      │
        │ ✓ SonarCloud analysis (Java)          │
        └───────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │   ACTION: ./.github/actions/build     │
        ├───────────────────────────────────────┤
        │ ✓ Install dependencies                │
        │ ✓ Build application                   │
        │ ✓ Upload artifacts                    │
        │                                        │
        │ OUTPUT: artifact-name                 │
        └───────────────────────────────────────┘
                            │
                            ▼
   ┌─────────────────────────────────────────────┐
   │ CREDENTIAL SELECTION (Prepare credentials)  │
   ├─────────────────────────────────────────────┤
   │ if registry == 'docker'                     │
   │   → use DOCKERHUB_USERNAME/TOKEN            │
   │ else if registry == 'ghcr'                  │
   │   → use github.actor / GITHUB_TOKEN         │
   │ else if registry == 'acr'                   │
   │   → use ACR_USERNAME/PASSWORD               │
   │ else (custom)                               │
   │   → use ACR_USERNAME/PASSWORD               │
   └─────────────────────────────────────────────┘
                            │
                            ▼
  ┌──────────────────────────────────────────────┐
  │ ACTION: ./.github/actions/build-and-push-image │
  ├──────────────────────────────────────────────┤
  │ ✓ Setup QEMU (multi-arch)                   │
  │ ✓ Setup Buildx                              │
  │ ✓ Login to registry                         │
  │ ✓ Build image                               │
  │ ✓ Push to registry                          │
  │                                              │
  │ OUTPUTS:                                     │
  │   • image-digest                            │
  │   • image-url                               │
  └──────────────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │ WORKFLOW OUTPUTS                      │
        ├───────────────────────────────────────┤
        │ • artifact-name                       │
        │ • image-digest                        │
        │ • image-url                           │
        └───────────────────────────────────────┘
                            │
                            ▼
              (Consumer workflow can use outputs)
```

## Actions Location Reference

```
Repository Structure
│
├─ .github/
│  ├─ actions/                      ← All custom actions here
│  │  ├─ build/
│  │  │  ├─ action.yml             ← Build application
│  │  │  └─ README.md
│  │  │
│  │  ├─ check/
│  │  │  ├─ action.yml             ← Lint & SonarCloud
│  │  │  └─ (no README)
│  │  │
│  │  └─ build-and-push-image/
│  │     ├─ action.yml             ← Docker build & push
│  │     └─ README.md
│  │
│  └─ workflows/                    ← Workflow files
│     ├─ reusable-ci.yml           ← Uses actions from ../actions/
│     └─ reusable-cd.yml
│
└─ [Rest of repository]
```

## Reference Path Syntax

### Local Action Reference (within same repository)
```yaml
uses: ./.github/actions/action-name
```

### External Action Reference (from another repository)
```yaml
uses: owner/repo/.github/actions/action-name@ref
```

### Standard Actions Reference
```yaml
uses: actions/checkout@v4
uses: docker/setup-buildx-action@v3
```

## Workflow to Action Parameter Mapping

### build-and-push Job → build Action
```
Workflow Input          →  Action Input
─────────────────────────────────────────
working-directory       →  working-directory
build-command          →  build-command
install-command        →  install-command
artifact-path          →  artifact-path
setup-node             →  setup-node
node-version           →  node-version
setup-python           →  setup-python
python-version         →  python-version
setup-java             →  setup-java
java-version           →  java-version
(secrets)PACKAGE_TOKEN →  package-token
```

### build-docker-image Job → build-and-push-image Action
```
Workflow Input                Prepared Value              →  Action Input
──────────────────────────────────────────────────────────────────────────
docker-image-name                                         →  image-name
dockerfile-path                                           →  dockerfile-path
working-directory                                         →  context
docker-registry               (enum: docker|ghcr|acr|custom) → registry
docker-registry-url                                       →  registry-url
(dynamic from creds step)                                 →  registry-username
(dynamic from creds step)                                 →  registry-password
image-tag OR github.sha                                   →  tags
docker-build-args                                         →  build-args
docker-platforms                                          →  platforms
environment                                               →  environment
(hardcoded)                   'true'                      →  push
```

## Usage Example: End-to-End

### Consumer Repository
```yaml
# .github/workflows/ci.yml (in consumer repo)
name: CI

on: [push]

jobs:
  build:
    uses: my-org/shared-workflows/.github/workflows/reusable-ci.yml@main
    with:
      install-command: 'npm install'
      build-command: 'npm run build'
      docker-image-name: 'myapp'
      docker-registry: 'docker'
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying ${{ needs.build.outputs.image-url }}"
```

### Shared Workflows Repository
```
shared-workflows/
├── .github/actions/build/action.yml
├── .github/actions/check/action.yml
├── .github/actions/build-and-push-image/action.yml
└── .github/workflows/reusable-ci.yml (uses all 3 actions)
```

### Flow
1. Consumer calls `reusable-ci.yml@main`
2. `reusable-ci.yml` executes Job 1-5
3. Jobs call actions: `check`, `build`, `build-and-push-image`
4. Actions execute their composite steps
5. Outputs passed back to consumer

## 🎯 All Reference Paths (Final Checklist)

✅ Action Reference in reusable-ci.yml (line 250):
```yaml
uses: ./.github/actions/check
```

✅ Action Reference in reusable-ci.yml (line 277):
```yaml
uses: ./.github/actions/build
```

✅ Action Reference in reusable-ci.yml (line 330):
```yaml
uses: ./.github/actions/build-and-push-image
```

All references are valid and correctly point to local actions.
