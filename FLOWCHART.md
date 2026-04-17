# 🔄 Flujo Completo: PR → Merge → Build → Deploy

## Visual Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                      DEVELOPER WORKFLOW                             │
└─────────────────────────────────────────────────────────────────────┘

1️⃣  Developer commits en rama feature
        │
        ├─ git checkout -b feature/new-feature
        ├─ [Hacer cambios]
        ├─ git commit -m "feat: new feature"
        └─ git push origin feature/new-feature
                │
                ▼
2️⃣  Crea PR a develop en GitHub
        │
        └─ Create Pull Request → develop
                │
                ▼

┌─────────────────────────────────────────────────────────────────────┐
│                  ⚡ PR VALIDATION WORKFLOW                          │
│              (.github/workflows/pr-validation.yml)                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Trigger: on pull_request (opened, synchronize)                    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────┐       │
│  │ Usa: hakimsa/shared-workflows/.../reusable-ci.yml@main │       │
│  ├─────────────────────────────────────────────────────────┤       │
│  │ ✓ mvn clean install          (Install)                │       │
│  │ ✓ mvn checkstyle:check       (Lint)                   │       │
│  │ ✓ mvn test                   (Tests)                  │       │
│  │ ✓ mvn clean package          (Build)                  │       │
│  │ ✓ SonarCloud Analysis        (Code Quality)           │       │
│  │ ✗ NO Docker Build            (no docker-image-name)   │       │
│  └─────────────────────────────────────────────────────────┘       │
│                                                                      │
│  Result: SUCCESS ✅ o FAILURE ❌                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
                │
                ├─── SI FALLA ❌
                │     └─→ PR Comments: "Validation failed"
                │         Developer arregla y pushea cambios
                │         Workflow se ejecuta nuevamente
                │
                └─── SI PASA ✅
                      └─→ PR marked as "Ready for merge"
                          │
                          ▼

┌─────────────────────────────────────────────────────────────────────┐
│                ⚡ AUTO MERGE WORKFLOW                               │
│             (.github/workflows/auto-merge-pr.yml)                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Trigger: workflow_run [PR Validation completed successfully]      │
│                                                                      │
│  ✓ Get PR number                                                    │
│  ✓ gh pr merge --squash --delete-branch                           │
│  ✓ PR merged to develop                                            │
│  ✓ Feature branch deleted                                          │
│                                                                      │
│  Result: SUCCESS ✅                                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
                │
                ▼ MERGE COMPLETE - Develop updated
                │
                ├─ Develop branch now contains PR changes
                ├─ Feature branch removed
                └─ Push to develop triggers next workflow
                      │
                      ▼

┌─────────────────────────────────────────────────────────────────────┐
│              ⚡ DEVELOP CI/CD WORKFLOW                              │
│          (.github/workflows/develop-ci-cd.yml)                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Trigger: push [develop] OR workflow_run [Auto Merge completed]    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────┐       │
│  │ Usa: hakimsa/shared-workflows/.../reusable-ci.yml@main │       │
│  ├─────────────────────────────────────────────────────────┤       │
│  │ ✓ mvn clean install          (Install)                │       │
│  │ ✓ mvn checkstyle:check       (Lint)                   │       │
│  │ ✓ mvn test                   (Tests) - Again!         │       │
│  │ ✓ mvn clean package          (Build) - Full branch     │       │
│  │ ✓ SonarCloud Analysis        (Code Quality)           │       │
│  │ ✓ Docker Build & Push        (BUILD IMAGE!) 🐳         │       │
│  │   └─ Image: hakimsa/bakend-mgt:develop-{sha}          │       │
│  │   └─ Push to Docker Hub                               │       │
│  └─────────────────────────────────────────────────────────┘       │
│                                                                      │
│  Outputs Available:                                                │
│    • artifact-name: Nombre del JAR/ZIP                            │
│    • image-url: URL completa de la imagen                         │
│    • image-digest: SHA256 de la imagen                            │
│                                                                      │
│  Post-Actions:                                                     │
│    ✓ Notifications                                                 │
│    ✓ Trigger deployment (webhook a staging)                       │
│    ✓ Update deployment tracking                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
                │
                ▼ DEPLOYMENT COMPLETE
                │
        ┌───────┴────────┬──────────────┐
        │                │              │
    Staging       Monitoring      Logs & Alerts
    Ready         Active          Enabled
        │                │              │
        └────────────────┴──────────────┘
                │
                ▼

📊 FINAL STATE
├─ ✅ Code validated (PR validation)
├─ ✅ Auto merged to develop (auto-merge)
├─ ✅ Tests passed on full branch (develop workflow)
├─ ✅ Build successful
├─ 🐳 Docker image in registry
└─ 🚀 Ready for production release
```

---

## 📋 Summary Table

| Phase | Workflow | Trigger | Actions | Result |
|-------|----------|---------|---------|--------|
| **1. Validation** | PR Validation | PR open/updated | Lint, Test, Build | ✅/❌ |
| **2. Merge** | Auto Merge | PR Validation success | Squash merge | Develop updated |
| **3. Build** | Develop CI/CD | Merge complete | Full test + Docker build | 🐳 Image ready |
| **4. Deploy** | (Optional) | Docker build success | Webhook to K8s/etc | 🚀 Live |

---

## 🔄 Git Flow Overview

```
┌─────────────────────────────────────────────────────┐
│         YOUR REPOSITORY (bakend-mgt)                │
├─────────────────────────────────────────────────────┤
│                                                      │
│  develop (main integration branch)                  │
│     ▲                                                │
│     │ (squash merge from feature)                   │
│     │                                                │
│  feature/new-feature (your work)                    │
│     │                                                │
│     └─ Create PR ──→ PR Validation ──→ Auto Merge  │
│                                                      │
│  main (production, created from release)            │
│     ▲                                                │
│     │ (merge from develop when ready)                │
│     │                                                │
│     └─ [Manual or automated release process]        │
│                                                      │
└─────────────────────────────────────────────────────┘
```

---

## 🚀 Workflow Coordination

```
┌─────────────────────────────────────────────────────────────┐
│          SHARED WORKFLOWS REPO                              │
│      (hakimsa/shared-workflows)                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  .github/workflows/reusable-ci.yml                          │
│      ├─ Job: check-with-lint                               │
│      ├─ Job: test-and-coverage                             │
│      ├─ Job: check (SonarCloud)                            │
│      ├─ Job: build-and-push                                │
│      └─ Job: build-docker-image                            │
│                                                              │
│  .github/actions/                                           │
│      ├─ check (uses external actions)                      │
│      ├─ build (uses external actions)                      │
│      └─ build-and-push-image (uses Docker buildx)          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
         │
         │ Used by multiple consumer repos
         │
         ├─ bakend-mgt (Java)
         ├─ frontend-app (Node.js)
         ├─ api-python (Python)
         └─ ...
```

---

## ⚙️ Configuration Checklist

Para que todo funcione en tu repo consumidor:

```
bakend-mgt Repository
├─ Settings
│  ├─ Actions
│  │  ├─ ✅ Read and write permissions
│  │  └─ ✅ Allow GitHub Actions to create and approve PRs
│  │
│  ├─ Pull Requests
│  │  ├─ ✅ Allow auto-merge
│  │  ├─ ✅ Delete head branches
│  │  └─ ✅ Require status checks to pass
│  │
│  ├─ Branch protection (develop)
│  │  ├─ ✅ Require PR reviews (optional)
│  │  ├─ ✅ Require status checks to pass
│  │  └─ ✅ Require branches to be up to date
│  │
│  └─ Secrets
│     ├─ ✅ PACKAGE_TOKEN
│     ├─ ✅ SONAR_TOKEN
│     ├─ ✅ DOCKERHUB_USERNAME
│     └─ ✅ DOCKERHUB_TOKEN
│
└─ .github/workflows/
   ├─ ✅ pr-validation.yml
   ├─ ✅ auto-merge-pr.yml
   └─ ✅ develop-ci-cd.yml
```

---

## 📊 Timing & Performance

```
PR Created
    │
    ├─ PR Validation: ~5-10 min ⏱️
    │  (install + lint + test + build)
    │
    └─ Status: Ready to merge ✅
         │
         ├─ Auto Merge: ~1 min ⏱️
         │  (merge + delete branch)
         │
         └─ Develop CI/CD starts
              │
              ├─ Full validation: ~5-10 min ⏱️
              │  (install + lint + test + build)
              │
              ├─ Docker build: ~3-5 min ⏱️
              │  (buildx + push)
              │
              └─ Total: ~12-20 min ⏱️
                   Image available in Docker Hub ✅
```

---

## 🎯 Benefits of This Approach

| Benefit | Why |
|---------|-----|
| **Automatic Merging** | No manual PR merges needed |
| **Continuous Validation** | Tests run twice (PR + develop) |
| **Clean Git History** | Squash merges = linear history |
| **Fast Feedback** | Developers know status in <15 min |
| **Auto Deployment** | Image ready without manual steps |
| **Production Ready** | Same workflow for main branch |

---

**This is a production-ready CI/CD strategy! 🚀**
