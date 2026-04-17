# 📑 Complete Documentation Index

## 📚 Quick Navigation

- [WORKFLOW_REFERENCE.md](./WORKFLOW_REFERENCE.md) - Guía de uso y ejemplos del workflow
- [WORKFLOW_STRUCTURE.md](./WORKFLOW_STRUCTURE.md) - Estructura interna y mapeo de actions
- [REFERENCES_DIAGRAM.md](./REFERENCES_DIAGRAM.md) - Diagramas de flujo y referencias
- [.github/actions/README.md](./.github/actions/README.md) - Documentación de las custom actions

## 🏗️ Project Structure

```
shared-workflows/
│
├── .github/
│   ├── actions/                       ← Custom GitHub Actions
│   │   ├── README.md                  ← Actions overview
│   │   ├── build/
│   │   │   ├── action.yml             ← Build application
│   │   │   └── README.md              ← Build action docs
│   │   ├── check/
│   │   │   └── action.yml             ← Lint & SonarCloud
│   │   └── build-and-push-image/
│   │       ├── action.yml             ← Docker build & push
│   │       └── README.md              ← Docker action docs
│   │
│   └── workflows/
│       ├── reusable-ci.yml            ← Main CI workflow (uses all 3 actions)
│       └── reusable-cd.yml            ← CD workflow
│
├── WORKFLOW_REFERENCE.md              ← Usage guide with examples
├── WORKFLOW_STRUCTURE.md              ← Internal structure & mapping
├── REFERENCES_DIAGRAM.md              ← Data flow diagrams
├── INDEX.md                           ← This file
└── README.md                          ← Project readme
```

## 🔗 Action References in Workflow

| Action | Job | Line | Used For |
|--------|-----|------|----------|
| [./.github/actions/check](./.github/actions/check/action.yml) | check-with-lint / check | 250 | Linting & SonarCloud |
| [./.github/actions/build](./.github/actions/build/action.yml) | build-and-push | 277 | Build application |
| [./.github/actions/build-and-push-image](./.github/actions/build-and-push-image/action.yml) | build-docker-image | 330 | Docker build & push |

## 📖 Documentation Files Explained

### WORKFLOW_REFERENCE.md
**For:** Developers using the shared workflow

**Contains:**
- Overview of the workflow
- How to call it from other repositories
- Input parameters & secrets
- Examples for Node.js, Python, Java
- Examples for Docker Hub, GHCR, ACR
- Complete workflow examples

**Read this to:** Understand how to use `reusable-ci.yml` in your project

---

### WORKFLOW_STRUCTURE.md
**For:** Developers maintaining the workflow

**Contains:**
- Job dependency graph
- Which action each job uses
- Credential resolution logic
- Input/output mappings
- Job validation checklist

**Read this to:** Understand how the workflow works internally

---

### REFERENCES_DIAGRAM.md
**For:** Visual learners and architects

**Contains:**
- Reference path syntax
- Data flow diagrams
- Action location structure
- Parameter mapping tables
- End-to-end usage example

**Read this to:** See how data flows through the workflow

---

### .github/actions/README.md
**For:** Developers working with custom actions

**Contains:**
- Overview of each action
- Development guidelines
- Creating new actions
- Naming conventions
- Checklist for new actions

**Read this to:** Learn how to create or modify actions

---

## 🎯 Job Execution Flow

```
check-with-lint
    └─ Action: check
       
test-and-coverage
    └─ (inline steps)
       
check
    └─ Action: check
    └─ (SonarCloud)
       
build-and-push
    └─ Action: build
    └─ Output: artifact-name
       
build-docker-image
    ├─ Step: Prepare credentials (based on registry type)
    └─ Action: build-and-push-image
       └─ Output: image-digest, image-url
```

## ✅ Validation Checklist

- ✅ YAML syntax is valid (all files)
- ✅ 3 custom actions created
- ✅ All actions properly referenced in workflow
- ✅ Job dependencies are correct
- ✅ Credential selection logic works
- ✅ Complete documentation provided

## 🚀 Quick Start

### I want to...

**Use the workflow in my project:**
→ Read [WORKFLOW_REFERENCE.md](./WORKFLOW_REFERENCE.md)

**Understand how the workflow works:**
→ Read [WORKFLOW_STRUCTURE.md](./WORKFLOW_STRUCTURE.md)

**See data flow visually:**
→ Read [REFERENCES_DIAGRAM.md](./REFERENCES_DIAGRAM.md)

**Create a new action:**
→ Read [.github/actions/README.md](./.github/actions/README.md)

**Check action details (build):**
→ Read [.github/actions/build/README.md](./.github/actions/build/README.md)

**Check action details (Docker):**
→ Read [.github/actions/build-and-push-image/README.md](./.github/actions/build-and-push-image/README.md)

## 📊 Statistics

- **Total Actions:** 3
- **Total Jobs in Main Workflow:** 5
- **Supported Languages:** Node.js, Python, Java
- **Supported Docker Registries:** 4 (Docker Hub, GHCR, ACR, Custom)
- **Documentation Files:** 7

## 🔍 File Locations Reference

| What | Where | Type |
|------|-------|------|
| Main workflow | `.github/workflows/reusable-ci.yml` | YAML |
| Build action | `.github/actions/build/action.yml` | YAML |
| Check action | `.github/actions/check/action.yml` | YAML |
| Docker action | `.github/actions/build-and-push-image/action.yml` | YAML |
| Usage guide | `WORKFLOW_REFERENCE.md` | Markdown |
| Internal docs | `WORKFLOW_STRUCTURE.md` | Markdown |
| Flow diagrams | `REFERENCES_DIAGRAM.md` | Markdown |
| Actions docs | `.github/actions/README.md` | Markdown |

---

**Last Updated:** 2026-04-17
**Status:** ✅ Production Ready
