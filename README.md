# Workflow — DevOps Central de Tarffic-Simulator

Este repositorio contiene toda la lógica de CI/CD para la organización [Tarffic-Simulator](https://github.com/Tarffic-Simulator). Los demás repos no definen su pipeline internamente — solo llaman a los workflows definidos aquí.

---

## Repositorios del ecosistema

| Repo | URL | Estado |
|------|-----|--------|
| **Workflow** (este repo) | [Tarffic-Simulator/Workflow](https://github.com/Tarffic-Simulator/Workflow) | ✅ Activo |
| **Engine** | [Tarffic-Simulator/Engine](https://github.com/Tarffic-Simulator/Engine) | ✅ Activo |
| **Backend** | [Tarffic-Simulator/Backend](https://github.com/Tarffic-Simulator/Backend) | ✅ Activo |
| **mobile-app** | [Tarffic-Simulator/mobile-app](https://github.com/Tarffic-Simulator/mobile-app) | ✅ Activo |

---

## Cómo funciona

```
Engineer pushea a Engine o Backend
        │
        ▼
  CI corre en ese repo
  (llama al workflow reutilizable de aquí)
        │
        ▼
  Si el push es a main/develop,
  envía un evento al Orchestrator
        │
        ▼
  orchestrator.yml recibe el evento,
  lee config/variables.yml y coordina
  acciones centralizadas
```

---

## Setup inicial — paso a paso

### Paso 1 — Pushear este repo a GitHub

Si aún no lo has hecho, sube este repo a `Tarffic-Simulator/Workflow`:

```bash
git remote add origin https://github.com/Tarffic-Simulator/Workflow.git
git push -u origin main
```

> **Importante:** Los workflows reutilizables deben estar en `main` antes de que los otros repos los puedan llamar.

---

### Paso 2 — Crear el Personal Access Token (PAT)

Los repos Engine y Backend necesitan un token para enviar eventos al Orchestrator.

1. Ve a GitHub → tu perfil → **Settings**
2. En el menú izquierdo, baja hasta **Developer settings** → **Personal access tokens** → **Fine-grained tokens**
3. Click en **Generate new token**
4. Configura:
   - **Token name:** `DEVOPS_DISPATCH_TOKEN`
   - **Expiration:** la que prefieras (recomendado: 90 días)
   - **Resource owner:** `Tarffic-Simulator`
   - **Repository access:** Only selected repositories → selecciona **Workflow**
   - **Permissions → Repository permissions → Contents:** `Read and write`
5. Click en **Generate token**
6. **Copia el token** — solo se muestra una vez

---

### Paso 3 — Agregar el secret en el repo Engine

1. Ve a [Tarffic-Simulator/Engine](https://github.com/Tarffic-Simulator/Engine)
2. Click en **Settings** → **Secrets and variables** → **Actions**
3. Click en **New repository secret**
4. Nombre: `DEVOPS_DISPATCH_TOKEN`
5. Valor: pega el token del Paso 2
6. Click en **Add secret**

---

### Paso 4 — Agregar el secret en el repo Backend

Mismo proceso que el Paso 3, pero en [Tarffic-Simulator/Backend](https://github.com/Tarffic-Simulator/Backend).

1. Ve a **Settings** → **Secrets and variables** → **Actions**
2. **New repository secret**
3. Nombre: `DEVOPS_DISPATCH_TOKEN`, valor: el mismo token
4. **Add secret**

---

### Paso 5 — Agregar el ci.yml al repo Engine

1. En el repo [Tarffic-Simulator/Engine](https://github.com/Tarffic-Simulator/Engine), crea la carpeta `.github/workflows/` si no existe
2. Copia el archivo `templates/engine/ci.yml` de este repo
3. Pégalo en `Engine/.github/workflows/ci.yml`
4. Haz commit y push

```bash
# Desde el repo Engine:
mkdir -p .github/workflows
cp <ruta-a-este-repo>/templates/engine/ci.yml .github/workflows/ci.yml
git add .github/workflows/ci.yml
git commit -m "ci: add CI workflow from Workflow repo"
git push
```

El workflow se activará automáticamente en el siguiente push.

---

### Paso 6 — Agregar el ci.yml al repo Backend

Mismo proceso, con el archivo de backend:

```bash
# Desde el repo Backend:
mkdir -p .github/workflows
cp <ruta-a-este-repo>/templates/backend/ci.yml .github/workflows/ci.yml
git add .github/workflows/ci.yml
git commit -m "ci: add CI workflow from Workflow repo"
git push
```

---

### Paso 7 — Verificar que todo funciona

1. Haz un push cualquiera al repo Engine o Backend
2. Ve a **Actions** en ese repo — deberías ver el workflow `CI` corriendo
3. Ve a **Actions** en este repo ([Tarffic-Simulator/Workflow](https://github.com/Tarffic-Simulator/Workflow)) → workflow `Orchestrator` — deberías ver el evento recibido con el resumen del push

---

## Actualizar variables del proyecto

Todas las variables están en un solo lugar:

```
config/variables.yml
```

Edita ese archivo para cambiar nombres de repos, versiones de Python, branches, etc. No es necesario tocar los workflows directamente.

---

## Agregar el ci.yml al repo mobile-app

Mismo proceso que Engine y Backend:

1. Agrega secret `DEVOPS_DISPATCH_TOKEN` en [Tarffic-Simulator/mobile-app](https://github.com/Tarffic-Simulator/mobile-app)
2. Crea `.github/workflows/ci.yml` con el contenido de `templates/app-movil/ci.yml`
3. Llena las variables del servidor en `config/variables.yml` → `server.app_movil`

---

## Estructura de este repo

```
Workflow/
├── config/
│   └── variables.yml              # ← Variables globales del proyecto
├── templates/
│   ├── engine/
│   │   └── ci.yml                 # ← Copiar a Engine/.github/workflows/
│   ├── backend/
│   │   └── ci.yml                 # ← Copiar a Backend/.github/workflows/
│   └── app-movil/
│       └── ci.yml                 # ← Copiar a mobile-app/.github/workflows/
└── .github/
    └── workflows/
        ├── reusable-engine-ci.yml  # CI reutilizable para Python (Engine)
        ├── reusable-backend-ci.yml # CI reutilizable para Backend
        ├── reusable-app-ci.yml     # CI reutilizable para App Móvil
        └── orchestrator.yml        # Coordinador central de eventos
```
