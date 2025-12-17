Vamos a ejecutar este plan como si fuera una checklist operativa.
Indícame el Paso 1 exacto que debo ejecutar ahora y espera a que lo confirme antes de continuar.


# Plan paso a paso — Pipeline GitHub Actions (Backend + Prisma + EC2)

> **Objetivo del ejercicio**  
Crear un pipeline en **GitHub Actions** que se ejecute **al hacer push a una rama que tenga un Pull Request abierto** y que realice, en orden:
>
> 1. **Tests del backend**
> 2. **Build del backend**
> 3. **Despliegue del backend en un EC2**
>
> Todo siguiendo **buenas prácticas de CI/CD y DevSecOps**, usando **Prisma con migraciones**, y documentando los **prompts utilizados**.

---

## 0️⃣ Pre-requisitos (antes de empezar)

- Cuenta de **GitHub** con permisos para:
  - crear forks
  - crear secrets
  - abrir Pull Requests
- Cuenta de **AWS** con una instancia **EC2 Ubuntu (free tier)**.
- **Cursor IDE** instalado.
- Acceso SSH funcional al EC2 (key pair).
- Conocimientos básicos de Git y línea de comandos.

---

## 1️⃣ Preparación del repositorio

### 1.1 Fork y clone
1. Haz **fork** del repositorio base en tu cuenta.
2. Clona tu fork:
   ```bash
   git clone <TU_FORK_URL>
   cd AI4Devs-pipeline-2509_Rk
   ```

### 1.2 Rama del entregable
Crea la rama exigida por el ejercicio:
```bash
git checkout -b pipeline-hso
```

---

## 2️⃣ Verificación técnica del backend

En `backend/package.json` se confirma:

- **Tests**: `npm test` (Jest)
- **Build**: `npm run build` (TypeScript → `dist/`)
- **Arranque prod**: `node dist/index.js`
- **ORM**: Prisma
- **Migraciones**: presentes en `backend/prisma/migrations/`

### Conclusión técnica
- El pipeline **DEBE** usar `prisma migrate deploy`
- **NO** se debe usar `prisma db push` en CI/CD

---

## 3️⃣ Ajuste obligatorio de seguridad (Prisma)

### 3.1 Problema detectado
El archivo `schema.prisma` tenía credenciales hardcodeadas.

### 3.2 Corrección requerida
Modificar el datasource a:

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

✅ Esto es obligatorio para:
- CI en GitHub Actions
- Producción en EC2
- Cumplir buenas prácticas DevSecOps

Este cambio **debe incluirse en el PR**.

---

## 4️⃣ Preparación del EC2 (entorno de despliegue)

### 4.1 Crear y configurar instancia
- AMI: **Ubuntu LTS**
- Security Group:
  - Puerto **22 (SSH)** → solo tu IP
  - Puerto del backend (ej. **3000**) → solo tu IP (temporal)

### 4.2 Instalación de dependencias
En el EC2:
```bash
sudo apt update
sudo apt install -y nodejs npm
sudo npm install -g pm2
mkdir -p /home/ubuntu/app
```

PM2 se usará para mantener el backend corriendo tras el deploy.

---

## 5️⃣ Secrets en GitHub Actions

En **Settings → Secrets and variables → Actions**, crear:

### 5.1 Secrets de infraestructura
- `EC2_HOST` → IP o DNS del EC2
- `EC2_USER` → `ubuntu`
- `EC2_SSH_KEY` → private key PEM (contenido completo)
- `APP_DIR` → `/home/ubuntu/app`

### 5.2 Secrets de aplicación
- `DATABASE_URL` → cadena de conexión real de producción
- Otros envs si aplica (`PORT`, `NODE_ENV`, etc.)

📌 **Nunca** guardar secretos en el repo ni en el YAML.

---

## 6️⃣ Diseño del workflow de GitHub Actions

### 6.1 Trigger requerido
El pipeline debe ejecutarse cuando:
- hay un **push**
- la rama del push tiene un **Pull Request abierto**

👉 GitHub Actions no ofrece este trigger directo, por lo que se implementa con:
- `on: push`
- un **job previo** que consulta la GitHub API

---

## 7️⃣ Estructura final del pipeline

### Job 1 — `check_pr_open`
Responsabilidad:
- Consultar la GitHub API
- Verificar si existe un PR abierto para `owner:branch`
- Si **no hay PR**, terminar el pipeline sin error

### Job 2 — `test_build_deploy`
Solo se ejecuta si el job anterior confirma PR abierto.

Incluye:
1. Servicio **PostgreSQL** efímero
2. Instalación de dependencias
3. Prisma:
   - `prisma generate`
   - `prisma migrate deploy`
4. Tests (Jest)
5. Build (TypeScript)
6. Empaquetado del backend
7. Deploy por SSH al EC2
8. Reinicio del backend con PM2

---

## 8️⃣ Flujo correcto de Prisma en CI/CD

### En GitHub Actions (CI)
Orden exacto:
1. `npm ci`
2. `npm run prisma:generate`
3. `npx prisma migrate deploy`
4. `npm test`
5. `npm run build`

### En EC2 (producción)
Orden exacto:
```bash
npm ci --omit=dev
npm run prisma:generate
npx prisma migrate deploy
pm2 restart backend || pm2 start dist/index.js --name backend
pm2 save
```

---

## 9️⃣ Prompt Engineering (uso de Cursor)

### Prompt A — Tests y build
> Genera un workflow de GitHub Actions para un backend Node.js + TypeScript con Prisma y Jest que instale dependencias, genere Prisma Client, aplique migraciones con `prisma migrate deploy`, ejecute tests y genere el build.

### Prompt B — Trigger con PR abierto
> Modifica el workflow para que se ejecute en `push`, pero solo continúe si la rama tiene un Pull Request abierto, usando la GitHub API y `GITHUB_TOKEN`.

### Prompt C — Deploy a EC2
> Extiende el workflow para empaquetar el backend, copiarlo por SSH a un EC2 y reiniciar la aplicación con PM2 siguiendo buenas prácticas.

### Prompt D — Documentación
> Genera un archivo `prompts/prompts-hso.md` documentando los prompts usados, las decisiones técnicas y los secrets requeridos.

---

## 🔟 Validación del pipeline

1. Abre un Pull Request desde `pipeline-hso`.
2. Realiza un push adicional a esa rama.
3. Verifica en **Actions**:
   - `check_pr_open` detecta el PR
   - Tests pasan
   - Build se genera
   - Deploy se ejecuta sin errores
4. En el EC2:
   ```bash
   pm2 ls
   pm2 logs backend
   ```

---

## 1️⃣1️⃣ Entrega final

El Pull Request debe incluir:

- `.github/workflows/pipeline.yml`
- `schema.prisma` corregido
- `prompts/prompts-hso.md`

---

## 1️⃣2️⃣ Checklist final

- [ ] `schema.prisma` usa `env("DATABASE_URL")`
- [ ] Se usa `prisma migrate deploy` (no `db push`)
- [ ] Pipeline solo corre con PR abierto
- [ ] Tests, build y deploy funcionan
- [ ] Secrets correctamente configurados
- [ ] PM2 mantiene el backend activo
- [ ] Prompts documentados

---

## 1️⃣3️⃣ Nota para evaluación

> *Se utiliza `prisma migrate deploy` en CI y producción porque el proyecto cuenta con migraciones versionadas, siguiendo las recomendaciones oficiales de Prisma para entornos no interactivos.*

Esto demuestra **criterio técnico**, no solo ejecución.
