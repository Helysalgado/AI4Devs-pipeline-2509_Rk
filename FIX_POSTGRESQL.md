# 🔧 Solución: Error de Autenticación PostgreSQL

## ❌ Error Actual
```
psql: error: connection to server at "localhost" (127.0.0.1), port 5432 failed: 
FATAL:  password authentication failed for user "ltiuser"
```

## ✅ Solución Paso a Paso

### **Paso 1: Eliminar y recrear el usuario correctamente**

```bash
# Conectarse como postgres (usuario administrador)
sudo -u postgres psql

# Dentro de psql, ejecuta estos comandos uno por uno:

-- Eliminar el usuario si existe
DROP DATABASE IF EXISTS ltidb;
DROP USER IF EXISTS ltiuser;

-- Crear el usuario con contraseña
CREATE USER ltiuser WITH PASSWORD 'changeme123';

-- Crear la base de datos
CREATE DATABASE ltidb OWNER ltiuser;

-- Dar permisos
GRANT ALL PRIVILEGES ON DATABASE ltidb TO ltiuser;

-- Conectarse a la base de datos
\c ltidb

-- Dar permisos en el esquema público (importante para Prisma)
GRANT ALL ON SCHEMA public TO ltiuser;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO ltiuser;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO ltiuser;

-- Hacer que los permisos sean permanentes para tablas futuras
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO ltiuser;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON SEQUENCES TO ltiuser;

-- Salir
\q
```

### **Paso 2: Configurar el método de autenticación**

```bash
# Editar el archivo de configuración de autenticación
sudo nano /etc/postgresql/*/main/pg_hba.conf
```

**Busca esta línea:**
```
local   all             all                                     peer
```

**Agregar ANTES de esa línea:**
```
# Permitir autenticación con contraseña para ltiuser
local   all             ltiuser                                 md5
host    all             ltiuser         127.0.0.1/32            md5
host    all             ltiuser         ::1/128                 md5
```

**Guardar:** Ctrl+O, Enter, Ctrl+X

### **Paso 3: Reiniciar PostgreSQL**

```bash
sudo systemctl restart postgresql
sudo systemctl status postgresql
```

### **Paso 4: Probar la conexión**

```bash
# Probar conexión
psql -U ltiuser -d ltidb -h localhost -c "SELECT version();"
# Contraseña: changeme123
```

Si funciona, deberías ver la versión de PostgreSQL.

---

## 🚀 Alternativa Más Simple (Si la anterior no funciona)

Usa autenticación "trust" localmente (solo para desarrollo/testing):

```bash
# Editar configuración
sudo nano /etc/postgresql/*/main/pg_hba.conf
```

**Cambiar estas líneas:**
```
# DE ESTO:
local   all             all                                     peer

# A ESTO (temporal para testing):
local   all             all                                     trust
host    all             all         127.0.0.1/32                trust
```

**Guardar y reiniciar:**
```bash
sudo systemctl restart postgresql

# Ahora probar SIN contraseña:
psql -U ltiuser -d ltidb -h localhost -c "SELECT version();"
```

---

## 📝 Después de Solucionar

Una vez que la conexión funcione, la cadena de conexión es:

```
postgresql://ltiuser:changeme123@localhost:5432/ltidb
```

**Esta ya está configurada en tus GitHub Secrets como DATABASE_URL** ✅

---

## 🔍 Verificar que Todo Funciona

```bash
# 1. Verificar que el servicio está corriendo
sudo systemctl status postgresql

# 2. Verificar que puedes conectarte
psql -U ltiuser -d ltidb -h localhost -c "\dt"

# 3. Crear una tabla de prueba
psql -U ltiuser -d ltidb -h localhost <<EOF
CREATE TABLE IF NOT EXISTS test (id SERIAL PRIMARY KEY, name TEXT);
INSERT INTO test (name) VALUES ('Hello from PostgreSQL');
SELECT * FROM test;
DROP TABLE test;
EOF
```

Si todo esto funciona, ¡estás listo para el deploy! 🎉

---

## ⚠️ Nota de Seguridad

La contraseña `changeme123` es **solo para pruebas**. En producción real deberías:
1. Usar una contraseña fuerte
2. Configurar SSL/TLS
3. Limitar acceso por IP
4. Usar AWS RDS en lugar de PostgreSQL local

Pero para este ejercicio de aprendizaje, está bien. 👍

