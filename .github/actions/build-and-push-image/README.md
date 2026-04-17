# Build and Push Docker Image Action

Acción reutilizable para construir y pushear imágenes Docker a diferentes registros.

## Descripción

Esta acción automatiza el proceso de build y push de imágenes Docker con soporte para:
- Múltiples registros (Docker Hub, GitHub Container Registry, Azure Container Registry, custom)
- Multi-plataforma (linux/amd64, linux/arm64, etc.)
- Caching automático con GitHub Actions Cache
- Etiquetado flexible
- Build arguments personalizados

## Inputs

### Requeridos

- `image-name` - Nombre de la imagen (ej: `myapp`, `org/myapp`)

### Configuración de Imagen

| Input | Descripción | Por defecto |
|-------|-------------|-------------|
| `dockerfile-path` | Ruta al Dockerfile | `Dockerfile` |
| `context` | Contexto de build | `.` |
| `tags` | Tags separados por coma (ej: `latest,v1.0.0`) | `{sha},{environment},latest` |
| `build-args` | Build args separados por salto de línea | `` |

### Registro

| Input | Descripción | Por defecto |
|-------|-------------|-------------|
| `registry` | Tipo de registro: `docker`, `ghcr`, `acr`, `custom` | `docker` |
| `registry-username` | Usuario del registro | `` |
| `registry-password` | Contraseña/token del registro | `` |
| `registry-url` | URL personalizada del registro | `` |

### Configuración de Push

| Input | Descripción | Por defecto |
|-------|-------------|-------------|
| `push` | Push la imagen después de buildear | `true` |
| `environment` | Entorno (dev, staging, production) | `dev` |
| `platforms` | Plataformas soportadas | `linux/amd64` |

## Outputs

- `image-digest` - Digest de la imagen Docker
- `image-url` - URL completa de la imagen con tag

## Ejemplos

### Docker Hub
```yaml
- name: Build and Push to Docker Hub
  uses: ./.github/actions/build-and-push-image
  with:
    image-name: myusername/myapp
    registry: docker
    registry-username: ${{ secrets.DOCKERHUB_USERNAME }}
    registry-password: ${{ secrets.DOCKERHUB_TOKEN }}
    tags: 'latest,v1.0.0'
    environment: production
```

### GitHub Container Registry
```yaml
- name: Build and Push to GHCR
  uses: ./.github/actions/build-and-push-image
  with:
    image-name: myapp
    registry: ghcr
    registry-username: ${{ github.actor }}
    registry-password: ${{ secrets.GITHUB_TOKEN }}
    environment: production
```

### Azure Container Registry
```yaml
- name: Build and Push to ACR
  uses: ./.github/actions/build-and-push-image
  with:
    image-name: myapp
    registry: acr
    registry-url: myregistry.azurecr.io
    registry-username: ${{ secrets.ACR_USERNAME }}
    registry-password: ${{ secrets.ACR_PASSWORD }}
    environment: production
```

### Con Build Arguments
```yaml
- name: Build and Push with Args
  uses: ./.github/actions/build-and-push-image
  with:
    image-name: myapp
    registry: docker
    registry-username: ${{ secrets.DOCKERHUB_USERNAME }}
    registry-password: ${{ secrets.DOCKERHUB_TOKEN }}
    build-args: |
      NODE_ENV=production
      BUILD_VERSION=1.0.0
      API_URL=https://api.example.com
```

### Multi-plataforma
```yaml
- name: Build and Push Multi-arch
  uses: ./.github/actions/build-and-push-image
  with:
    image-name: myapp
    registry: docker
    registry-username: ${{ secrets.DOCKERHUB_USERNAME }}
    registry-password: ${{ secrets.DOCKERHUB_TOKEN }}
    platforms: linux/amd64,linux/arm64,linux/arm/v7
```

## Tipos de Aplicación Soportadas

Esta action es agnóstica al tipo de aplicación. Funciona con:
- **Node.js** - Aplicaciones Express, Next.js, etc.
- **Python** - Django, Flask, FastAPI
- **Java** - Spring Boot, microservicios
- **Go** - Aplicaciones compiladas
- **Rust** - Aplicaciones de alto rendimiento
- **.NET** - Aplicaciones ASP.NET Core
- **Cualquier** - Mientras tengas un Dockerfile válido

## Integración en Workflow

```yaml
jobs:
  build-image:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build and Push Docker Image
        uses: ./.github/actions/build-and-push-image
        with:
          image-name: ${{ secrets.DOCKERHUB_USERNAME }}/myapp
          registry: docker
          registry-username: ${{ secrets.DOCKERHUB_USERNAME }}
          registry-password: ${{ secrets.DOCKERHUB_TOKEN }}
          tags: ${{ github.ref_name }}
          environment: ${{ github.ref_name }}
```

## Características

- ✅ Multi-plataforma (x86_64, ARM64, etc.)
- ✅ Caching automático con GitHub Actions
- ✅ Soporte para múltiples registros
- ✅ Etiquetado flexible y automático
- ✅ Build arguments personalizables
- ✅ Salida de digest e imagen URL
- ✅ Resumen en GitHub Actions
