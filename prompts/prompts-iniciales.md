# Prompts utilizados – Pipeline CI/CD Backend

Este documento recoge los prompts utilizados para diseñar y generar el pipeline de CI/CD del backend mediante asistentes de IA, como parte del ejercicio AI4Devs Pipeline.

---

## 1. Prompt para trigger "push con PR abierto"

**Objetivo:**  
Configurar el pipeline para que se ejecute en cada push, pero solo continúe si la rama tiene un Pull Request abierto.

**Prompt utilizado:**

> "Crea un workflow de GitHub Actions que se ejecute en cada push a cualquier rama, pero que primero verifique si existe un Pull Request abierto para esa rama usando la GitHub API y GITHUB_TOKEN. Si no hay PR abierto, debe terminar sin ejecutar nada. Si hay PR abierto, debe continuar con los siguientes jobs."

**Decisión técnica:**  
GitHub Actions no tiene un trigger nativo para "push con PR abierto", por lo que implementamos un job inicial (`check-pr-open`) que consulta la API de GitHub para verificar la existencia de PRs abiertos antes de continuar.

---

## 2. Prompt para tests del backend

**Objetivo:**  
Ejecutar automáticamente los tests del backend usando Jest con una base de datos PostgreSQL efímera.

**Prompt utilizado:**

> "Genera un job de GitHub Actions para un backend Node.js con TypeScript que:
> 1. Configure un servicio PostgreSQL efímero
> 2. Instale dependencias con npm ci
> 3. Genere el Prisma Client
> 4. Aplique migraciones con prisma migrate deploy
> 5. Ejecute los tests con npm test"

**Decisión técnica:**  
Se usa `prisma migrate deploy` (no `db push`) porque el proyecto cuenta con migraciones versionadas en `backend/prisma/migrations/`, siguiendo las recomendaciones oficiales de Prisma para CI/CD.

---

## 3. Prompt para build del backend

**Objetivo:**  
Compilar el código TypeScript a JavaScript y empaquetar el artefacto para deploy.

**Prompt utilizado:**

> "Añade pasos al workflow para:
> 1. Generar el build del backend TypeScript usando npm run build
> 2. Empaquetar dist/, package.json, package-lock.json y la carpeta prisma en un tar.gz
> 3. Subir el artefacto para usarlo en el job de deploy"

**Decisión técnica:**  
Se empaquetan solo los archivos necesarios para reducir el tamaño del artefacto y acelerar la transferencia al EC2.

---

## 4. Prompt para despliegue en EC2

**Objetivo:**  
Desplegar automáticamente el backend en una instancia EC2 utilizando SSH y gestionar el proceso con PM2.

**Prompt utilizado:**

> "Crea un job de GitHub Actions que:
> 1. Descargue el artefacto del job anterior
> 2. Se conecte por SSH a un EC2 usando una key privada almacenada en secrets
> 3. Copie el artefacto al EC2 con scp
> 4. Ejecute remotamente: desempaquetar, instalar dependencias, aplicar migraciones de Prisma y reiniciar el backend con PM2"

**Decisión técnica:**  
Se utiliza PM2 para mantener el backend ejecutándose de forma persistente y permitir reinicios sin downtime (`pm2 restart` o `pm2 start`).

---

## 5. Prompt para manejo de variables de entorno

**Objetivo:**  
Asegurar que las credenciales de la base de datos y configuración sensible se manejen de forma segura.

**Prompt utilizado:**

> "Modifica el workflow para que:
> 1. El schema.prisma use env('DATABASE_URL') en lugar de credenciales hardcodeadas
> 2. La DATABASE_URL se pase como secret de GitHub Actions al EC2
> 3. Se cree un archivo .env en el EC2 durante el deploy con las variables necesarias"

**Decisión técnica:**  
Se corrigió el `schema.prisma` para usar variables de entorno, cumpliendo con las buenas prácticas de DevSecOps y evitando exponer credenciales en el código.

---

## 6. Prompt para validación y health checks

**Objetivo:**  
Verificar que el deploy fue exitoso y el backend está funcionando correctamente.

**Prompt utilizado:**

> "Añade al final del deploy:
> 1. Un comando que muestre el estado de PM2
> 2. Un health check con curl al localhost:3010 para confirmar que el backend responde
> 3. Un resumen del deploy en el GITHUB_STEP_SUMMARY"

**Decisión técnica:**  
Se incluyen validaciones automáticas post-deploy para detectar problemas inmediatamente y facilitar la depuración.

---

## 📋 Secrets requeridos en GitHub Actions

Para que el pipeline funcione correctamente, se deben configurar los siguientes secrets en:  
**Settings → Secrets and variables → Actions**

| Secret | Descripción | Ejemplo |
|--------|-------------|---------|
| `LIDRKEY` | Contenido completo de la clave privada SSH (.pem) | `-----BEGIN RSA PRIVATE KEY-----\n...` |
| `EC2_HOST` | IP pública o DNS del EC2 | `18.222.153.214` |
| `EC2_USER` | Usuario SSH del EC2 | `ubuntu` |
| `APP_DIR` | Directorio de la aplicación en EC2 | `/home/ubuntu/app` |
| `DATABASE_URL` | Cadena de conexión PostgreSQL de producción | `postgresql://ltiuser@localhost:5433/ltidb` |

---

## 🔧 Estructura final del pipeline

```
on: push (cualquier rama)
  ↓
Job 1: check-pr-open
  → Verifica si hay PR abierto via API
  → Si NO hay PR: termina ✓
  → Si SÍ hay PR: continúa ↓
  
Job 2: ci-backend (needs: check-pr-open)
  → Instala dependencias
  → Genera Prisma Client
  → Aplica migraciones
  → Ejecuta tests
  → Genera build
  → Empaqueta artefacto
  → Sube artefacto
  
Job 3: deploy-backend (needs: ci-backend)
  → Descarga artefacto
  → Copia al EC2 via SSH
  → Desempaqueta
  → Instala deps en producción
  → Aplica migraciones en prod
  → Reinicia con PM2
  → Health check
```

---

## ✅ Buenas prácticas aplicadas

1. **DevSecOps**: Secrets protegidos, no hay credenciales en el código
2. **CI/CD**: Pipeline automatizado con validación en cada push
3. **Testing**: Tests ejecutados antes del deploy
4. **Migrations**: Uso correcto de `prisma migrate deploy`
5. **Process Management**: PM2 para alta disponibilidad
6. **Artifact Management**: Empaquetado eficiente de artefactos
7. **Error Handling**: `set -euo pipefail` en todos los scripts bash
8. **Safety Checks**: Validaciones de directorios antes de operaciones destructivas
9. **Logging**: Outputs claros y resumen del deploy

---

## 📚 Referencias

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Prisma Migrations in Production](https://www.prisma.io/docs/guides/migrate/production-troubleshooting)
- [PM2 Process Management](https://pm2.keymetrics.io/docs/usage/quick-start/)
- [GitHub REST API - Pull Requests](https://docs.github.com/en/rest/pulls/pulls)
