# Prompts utilizados – Pipeline CI/CD Backend

Este documento recoge los prompts utilizados para diseñar y generar el pipeline de CI/CD del backend mediante asistentes de IA, como parte del ejercicio AI4Devs Pipeline.

---

## 1. Prompt para trigger "push con PR abierto"

**Objetivo:**  
Configurar el pipeline para que se ejecute cuando hay un Pull Request abierto.

**Prompt inicial:**

> "Crea un workflow de GitHub Actions que se ejecute en cada push a cualquier rama, pero que primero verifique si existe un Pull Request abierto para esa rama usando la GitHub API y GITHUB_TOKEN. Si no hay PR abierto, debe terminar sin ejecutar nada. Si hay PR abierto, debe continuar con los siguientes jobs."

**Decisión técnica final:**  
Después de evaluar la complejidad con forks y la API de GitHub, **simplificamos la solución** usando los triggers nativos de GitHub Actions:

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened]
  push:
    branches: [pipeline-hso]  # Para testing directo
```

Esta solución es más robusta, compatible con forks y no requiere consultas a la API.

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
- Se utiliza **`webfactory/ssh-agent`** para manejar las SSH keys de forma segura y robusta
- PM2 mantiene el backend ejecutándose de forma persistente
- Security Group del EC2 configurado para permitir SSH desde IPs de GitHub Actions (`0.0.0.0/0` en puerto 22)

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
> 2. Un health check con curl para confirmar que el backend responde
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
on: pull_request + push (pipeline-hso)
  ↓
Job 1: ci-backend
  → Instala dependencias
  → Genera Prisma Client
  → Aplica migraciones
  → Ejecuta tests
  → Genera build
  → Empaqueta artefacto
  → Sube artefacto
  ↓
Job 2: deploy-backend (needs: ci-backend)
  → Descarga artefacto
  → Instala SSH key (webfactory/ssh-agent)
  → Añade EC2 a known_hosts
  → Copia al EC2 via SSH/SCP
  → Desempaqueta en EC2
  → Instala deps en producción
  → Genera Prisma Client
  → Aplica migraciones en prod (puerto 5433)
  → Reinicia con PM2 (puerto 3010)
  → Health check
  → Deployment summary
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

## 🐛 Troubleshooting y lecciones aprendidas

Durante la implementación del pipeline, se encontraron y resolvieron varios problemas críticos:

### 1. Puerto no estándar de PostgreSQL
**Problema:** PostgreSQL en EC2 estaba corriendo en puerto **5433** en lugar del estándar 5432.  
**Solución:** Ajustar `DATABASE_URL` a `postgresql://ltiuser@localhost:5433/ltidb`  
**Detección:** `sudo ss -tlnp | grep postgres` reveló el puerto real

### 2. Puerto del backend
**Problema:** El backend estaba configurado para correr en puerto **3010**, no 3000.  
**Solución:** Actualizar el health check en el pipeline y documentación

### 3. Configuración de pg_hba.conf
**Problema:** Errores de autenticación "P1000: Authentication failed"  
**Solución:** Configurar `trust` authentication para `ltiuser` en las primeras líneas de `pg_hba.conf`
```
local   all   ltiuser   trust
host    all   ltiuser   127.0.0.1/32   trust
```
**Lección:** Las reglas de `pg_hba.conf` se evalúan en orden; las más específicas deben ir primero

### 4. SSH desde GitHub Actions
**Problema:** GitHub Actions usa IPs dinámicas, bloqueadas por Security Group  
**Solución:** Abrir puerto 22 a `0.0.0.0/0` o configurar un rango de IPs de GitHub Actions  
**Alternativa segura:** Usar VPN o Bastion Host en producción

### 5. Manejo de SSH keys
**Problema inicial:** Manejo manual de keys con `echo` y `chmod` fallaba  
**Solución final:** Usar `webfactory/ssh-agent` action para manejo robusto de keys

### 6. PostgreSQL no iniciaba
**Problema:** Errores de sintaxis en `pg_hba.conf` impedían que PostgreSQL iniciara  
**Detección:** `sudo systemctl status postgresql@16-main` mostró "FATAL: could not load pg_hba.conf"  
**Solución:** Validar sintaxis YAML-style del archivo, asegurar campos obligatorios (ADDRESS para tipo `host`)

## 📚 Referencias

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Prisma Migrations in Production](https://www.prisma.io/docs/guides/migrate/production-troubleshooting)
- [PM2 Process Management](https://pm2.keymetrics.io/docs/usage/quick-start/)
- [PostgreSQL pg_hba.conf Documentation](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html)
- [webfactory/ssh-agent GitHub Action](https://github.com/webfactory/ssh-agent)
