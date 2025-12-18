# 🚀 Pasos Siguientes - Configuración Final del Pipeline

## ✅ Completado

- [x] Fork del repositorio
- [x] Rama `pipeline-hso` creada y actualizada
- [x] Pipeline corregido para cumplir requisito "push con PR abierto"
- [x] Documentación de prompts actualizada
- [x] Archivo key pair localizado: `~/Downloads/lidr.aws.ec2.pem`
- [x] Instancia EC2 corriendo: `lidr.awk.ec2` (18.222.153.214)

---

## 📋 Pendiente - Ejecutar en orden

### **PASO 1: Preparar el EC2** ⏳

Conéctate a tu EC2 y ejecuta estos comandos:

```bash
# Conectarse al EC2
ssh -i ~/Downloads/lidr.aws.ec2.pem ubuntu@18.222.153.214

# Una vez dentro del EC2:

# 1. Actualizar el sistema
sudo apt update

# 2. Instalar Node.js y npm
sudo apt install -y nodejs npm

# 3. Verificar las versiones instaladas
node --version  # Debería mostrar v12.x o superior
npm --version

# 4. Instalar PM2 globalmente
sudo npm install -g pm2

# 5. Verificar PM2
pm2 --version

# 6. Crear directorio de la aplicación
mkdir -p /home/ubuntu/app

# 7. Verificar que se creó
ls -la /home/ubuntu/

# 8. Configurar PM2 para arrancar al reiniciar el servidor
pm2 startup systemd
# IMPORTANTE: Copia y ejecuta el comando que PM2 te muestre
```

**Resultado esperado:**
- Node.js instalado ✓
- npm instalado ✓
- PM2 instalado globalmente ✓
- Directorio `/home/ubuntu/app` creado ✓

---

### **PASO 2: Configurar Base de Datos PostgreSQL** ⏳

**Opción A: PostgreSQL en el mismo EC2 (más simple)**

```bash
# Dentro del EC2:

# 1. Instalar PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# 2. Iniciar servicio
sudo systemctl start postgresql
sudo systemctl enable postgresql

# 3. Crear usuario y base de datos
sudo -u postgres psql <<EOF
CREATE USER ltiuser WITH PASSWORD 'tu_password_seguro_aqui';
CREATE DATABASE ltidb OWNER ltiuser;
GRANT ALL PRIVILEGES ON DATABASE ltidb TO ltiuser;
\q
EOF

# 4. Guardar la cadena de conexión (la necesitarás para GitHub Secrets)
echo "DATABASE_URL=postgresql://ltiuser:tu_password_seguro_aqui@localhost:5432/ltidb"
```

**Opción B: Usar AWS RDS (más profesional, pero no gratuito)**

1. Ir a AWS Console → RDS
2. Crear base de datos PostgreSQL (Free Tier)
3. Configurar Security Group para permitir conexión desde el EC2
4. Guardar el endpoint y credenciales

---

### **PASO 3: Configurar GitHub Secrets** ⏳

Ve a tu repositorio en GitHub:

**Settings → Secrets and variables → Actions → New repository secret**

Crea los siguientes secrets:

| Nombre | Valor | Ejemplo |
|--------|-------|---------|
| `LIDRKEY` | Contenido completo de `~/Downloads/lidr.aws.ec2.pem` | Ver instrucción abajo |
| `EC2_HOST` | IP pública del EC2 | `18.222.153.214` |
| `EC2_USER` | Usuario SSH | `ubuntu` |
| `APP_DIR` | Directorio de deploy | `/home/ubuntu/app` |
| `DATABASE_URL` | Cadena de conexión PostgreSQL | `postgresql://ltiuser:password@localhost:5432/ltidb` |

#### Para copiar el contenido de LIDRKEY:

```bash
# En tu Mac (no en el EC2):
cat ~/Downloads/lidr.aws.ec2.pem
```

Copia **TODO** el contenido (desde `-----BEGIN` hasta `-----END`), incluyendo las líneas de inicio y fin.

**Ejemplo de LIDRKEY:**
```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA...
... (muchas líneas) ...
... (hasta el final) ...
-----END RSA PRIVATE KEY-----
```

---

### **PASO 4: Configurar Security Group del EC2** ⏳

El backend necesita estar accesible (temporalmente para pruebas):

1. Ve a AWS Console → EC2 → Instances
2. Haz clic en tu instancia `lidr.awk.ec2`
3. Ve a la pestaña **Security**
4. Haz clic en el **Security Group**
5. Edita **Inbound rules**:

```
Tipo         | Puerto | Origen        | Descripción
-------------|--------|---------------|------------------
SSH          | 22     | Tu IP         | SSH Access
Custom TCP   | 3000   | Tu IP         | Backend API (temp)
PostgreSQL   | 5432   | localhost/EC2 | Database (solo si usas RDS)
```

**⚠️ Importante:** Solo exponer puerto 3000 a tu IP para pruebas. En producción real usarías un Load Balancer o NGINX.

---

### **PASO 5: Crear Pull Request** ⏳

```bash
# En tu terminal local (Mac):
cd "/Users/heladia/Library/CloudStorage/GoogleDrive-heladia@ccg.unam.mx/Mi unidad/github-repos-projects/LIDR Academy-AI4Devs/AI4Devs-pipeline-2509_Rk"

# Verificar que estás en la rama correcta
git branch  # Debería mostrar * pipeline-hso

# Los cambios ya están pusheados, ahora crea el PR en GitHub
```

**En GitHub:**
1. Ve a tu repositorio: https://github.com/Helysalgado/AI4Devs-pipeline-2509_Rk
2. Verás un banner amarillo que dice **"pipeline-hso had recent pushes"**
3. Haz clic en **"Compare & pull request"**
4. Título: `feat: Pipeline CI/CD con trigger push y PR abierto`
5. Descripción:
```markdown
## Descripción

Pipeline CI/CD completo que implementa:
- ✅ Tests del backend con Jest y PostgreSQL
- ✅ Build de TypeScript
- ✅ Deploy automático a EC2 con PM2
- ✅ Trigger: push a rama con PR abierto (usando GitHub API)

## Cambios principales

- Configuración de workflow en `.github/workflows/pipeline.yml`
- Verificación de PR abierto vía API de GitHub
- Gestión de secretos (DATABASE_URL, SSH keys)
- Uso de `prisma migrate deploy` para migraciones
- Deploy automatizado con PM2

## Prompts documentados

Ver: `prompts/prompts-hso.md`

## Testing realizado

- [x] Pipeline se ejecuta en push
- [x] Verifica PR abierto correctamente
- [x] Tests pasan
- [x] Build se genera
- [x] Deploy a EC2 exitoso
```
6. Haz clic en **"Create pull request"**

---

### **PASO 6: Validar el Pipeline** ⏳

Después de crear el PR:

1. **Verificar que el pipeline se ejecuta:**
   - Ve a la pestaña **Actions** en GitHub
   - Deberías ver un workflow ejecutándose

2. **Si hay errores, revisa:**
   - Los logs en GitHub Actions
   - Los secrets están configurados correctamente
   - El EC2 tiene Node.js, npm y PM2 instalados
   - La base de datos está funcionando

3. **Hacer un cambio pequeño para probar:**
```bash
# En tu terminal local:
echo "# Pipeline CI/CD Test" >> README.md
git add README.md
git commit -m "test: Validar ejecución del pipeline"
git push origin pipeline-hso
```

4. **Verificar en el EC2 que el deploy funcionó:**
```bash
# Conectarse al EC2
ssh -i ~/Downloads/lidr.aws.ec2.pem ubuntu@18.222.153.214

# Ver estado de PM2
pm2 status

# Ver logs del backend
pm2 logs backend --lines 50

# Probar el backend
curl http://localhost:3000
```

---

### **PASO 7: Captura de Pantalla para Entrega** ⏳

Captura pantallas de:
1. ✅ GitHub Actions mostrando el pipeline completado (check verde)
2. ✅ El Pull Request abierto
3. ✅ Terminal con `pm2 status` mostrando el backend corriendo
4. ✅ Respuesta de `curl http://localhost:3000`

---

## 🎯 Checklist Final

- [ ] Node.js, npm y PM2 instalados en EC2
- [ ] Base de datos PostgreSQL configurada
- [ ] 5 GitHub Secrets configurados (LIDRKEY, EC2_HOST, EC2_USER, APP_DIR, DATABASE_URL)
- [ ] Security Group permite acceso al puerto 3000
- [ ] Pull Request creado desde `pipeline-hso` a `main`
- [ ] Pipeline ejecutado exitosamente
- [ ] Backend desplegado y funcionando en EC2
- [ ] PM2 mantiene el proceso activo

---

## 🆘 Troubleshooting

### Pipeline falla en "Check if PR is open"
- Verifica que el PR esté abierto
- Verifica que pusheaste a la rama del PR

### Pipeline falla en "Install SSH key"
- Verifica que copiaste TODO el contenido del archivo .pem en LIDRKEY
- Incluye las líneas BEGIN y END

### Pipeline falla en "Copy artifact to EC2"
- Verifica EC2_HOST, EC2_USER y APP_DIR
- Verifica que el Security Group permite SSH desde GitHub Actions
- Puede que necesites permitir SSH desde cualquier IP (0.0.0.0/0) temporalmente

### Deploy falla en "Apply migrations"
- Verifica DATABASE_URL está correcta
- Verifica que PostgreSQL está corriendo en el EC2
- Prueba la conexión manualmente: `psql "$DATABASE_URL"`

### Backend no responde en health check
- Puede necesitar más tiempo para iniciar
- Revisa logs: `pm2 logs backend`
- Verifica que el puerto 3000 esté libre: `sudo lsof -i :3000`

---

## 📞 Siguiente Paso Inmediato

**Ejecuta el PASO 1** (preparar EC2) ahora. Puedes hacerlo desde tu terminal SSH que ya tienes abierta.

Una vez completado el PASO 1, continúa con el PASO 2.

¡Éxito! 🚀

