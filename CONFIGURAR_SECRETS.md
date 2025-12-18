# 🔐 Configurar GitHub Secrets - Guía Paso a Paso

## 📋 Contenido del archivo .pem encontrado

El archivo `lidr.aws.pem` contiene la clave privada SSH para conectarse al EC2.

**⚠️ Este archivo NO está en el repositorio por seguridad (está en .gitignore)**

---

## 🚀 PASO 1: Ir a GitHub Settings

1. Ve a tu repositorio: https://github.com/Helysalgado/AI4Devs-pipeline-2509_Rk

2. Haz clic en **⚙️ Settings** (arriba a la derecha)

3. En el menú lateral izquierdo, busca la sección **Security**

4. Haz clic en **Secrets and variables** → **Actions**

5. Verás una página con el botón verde **"New repository secret"**

---

## 🔑 PASO 2: Crear los 5 Secrets Requeridos

### Secret 1: LIDRKEY 🔴 MUY IMPORTANTE

**Name:** `LIDRKEY`

**Secret:** Copia TODO este contenido (COMPLETO):

```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAswD1OqBJ9/qqEbz9ixFH7bvICCperIINjjtT4tyGCo0+Q5qi
BiSz6ERopzp7SQSCUJr0YZROEZDxLlRaPQl0KH0EeAJe9HOUULyFGvrIez6ayGfD
jxGsqzH5ZiW3bygf1HzlJrnxdkkhQu1ggJfFWVKdckGGSiwCDK3Y0KJnAQAWoQyv
IMTf/3rjwIervOiUkt25nJ3sp2ggbIl447XyjASeWq/uwR73YqPhOkFMaeC/fNEB
Vi/uE6AI0O0xRsnjUGb49qpRzp1PBeXJpFh/K+LF9So+a2CxUk5TWfYAj+a1/NuB
MvgakxWLI+K7pmkOUrV5pYj6qvovP1teUxwhxQIDAQABAoIBAQCl4H5fTIgfFxcg
huzBQUtRX1EXWAQRghcDtbCfqtE/1/FZ/mKkpMBevX3pccUrPdVw0pciEadimdkV
oK9anncYyfkpKLFLgAtx4bDK9phvtO0ERzuuSaejoRTG9q6MgDc21mIXR/odLdl3
xrdFkt8bTfZ/GX9THoLrsvIXLFedUSVFpYf6HPYCcmZwrHP3I6z9QzYWtcZxLkze
vqWGUijP381+uCPwE0/7Qdy5ab/aFRBrze69/BOie0Ae+ieA2zx9rE0kI18unOod
044uRyvIW/dOaHzQnbpoWrX4AfhoAw4iYvjtue0RDztWf+6NbyCbL3eal9HgwMDq
yqhhNK2lAoGBAOKJDdPXo0SVwWM/zgdAuXqDdlYNjEmCZKHuZ5/Khc5LA+7PYpXJ
Ryza5/pRgIy7+OErgfokP4dXn9SiRjzHCy7mqcJ08kJnbsutfL8F/vuObgG5IIMy
X9zn5HRZcR2nKoFTBGrqEddMJS6aV9jIuROfxs2xHaZ+3V2eYaNfkhZ3AoGBAMpJ
Pmu7sw9f2+uidweNAkErikteFmqzpwOjJh30o3AvJN9ivnRFEP0KNPlleTjlPqNy
eBQmKYmHh3MT8m/FgEDiaWNWwjNrMbsBYeWA4hcEnCsf3ApoJjG0UN1RLNKUPurr
P6rllDVcyAlAl1rYRJPFvItLVlBA/hhIliJpl8yjAoGBANSpGrz7Gv8gorocRLpU
TYqwbN+dukurx/KoDslX4sLlcxy1vPOmT1XRbqJz7nyvXZVsNYlwi97vKFEBwXP+
2wW9bjHpfR9PYBh+lbPx2gunCqg9dUMUgB+t5a4/5MuUiXd8SpJfiD4X8nPMWplc
3TmJ7aRdF9ucDw16yGgJKOhDAoGAFqyTFJxbe9Ow4P66/NuvbwKkY8JOHPO6Oswk
z6LGVyLRrUByPLIpL1PfkDzxk5EOrl98WjXU3heU9S89M44dzCgUzA/DgOP5FQ8Y
nBMQRKg9oQ/XKEt4TIX7snMQ5SG807Q+1LcbH8ggm/jjfklTloTJl4uAR0qhsLMi
MQVJAKUCgYAp6a6dIZJDAX0zIDxs9jkIo13HNPgLqP/ctNZvwOdTWEwZxR/ZLAcY
62Ec2Osw1xYBNbB0xgtnuS7DBtm2KoI8khbKaVH3yK1dNqC6GCq1lAJaCjnmqp6K
v89Z7hae17fDtKCbZXuYgJntZUtGJjAr0NwXZPsb6RY4Vgrg9aE8/w==
-----END RSA PRIVATE KEY-----
```

✅ Clic en **Add secret**

---

### Secret 2: EC2_HOST

**Name:** `EC2_HOST`

**Secret:** `18.222.153.214`

✅ Clic en **Add secret**

---

### Secret 3: EC2_USER

**Name:** `EC2_USER`

**Secret:** `ubuntu`

✅ Clic en **Add secret**

---

### Secret 4: APP_DIR

**Name:** `APP_DIR`

**Secret:** `/home/ubuntu/app`

✅ Clic en **Add secret**

---

### Secret 5: DATABASE_URL (provisional)

**Name:** `DATABASE_URL`

**Secret:** `postgresql://ltiuser:changeme123@localhost:5432/ltidb`

✅ Clic en **Add secret**

> **Nota:** Este valor será actualizado después de configurar PostgreSQL en el EC2.

---

## ✅ Verificación

Después de crear los 5 secrets, deberías ver en la lista:

```
LIDRKEY         Updated X seconds ago
EC2_HOST        Updated X seconds ago
EC2_USER        Updated X seconds ago
APP_DIR         Updated X seconds ago
DATABASE_URL    Updated X seconds ago
```

---

## 🎯 Siguiente Paso

Una vez configurados los secrets, el siguiente paso es:

1. **Preparar el EC2** (instalar Node.js, npm, PM2, PostgreSQL)
2. **Actualizar DATABASE_URL** con las credenciales reales
3. **Crear el Pull Request**
4. **Validar el pipeline**

Ver detalles en: `PASOS_SIGUIENTES.md`

---

## 🆘 Problemas Comunes

### ❌ Error: "SSH key invalid"
- Asegúrate de copiar TODO el contenido del .pem (incluye `-----BEGIN` y `-----END`)
- No debe haber espacios extra al inicio o final
- Copia desde la terminal con: `cat lidr.aws.pem | pbcopy`

### ❌ Error: "Permission denied (publickey)"
- Verifica que EC2_HOST sea la IP correcta
- Verifica que EC2_USER sea `ubuntu`
- Verifica que el Security Group permita SSH

---

## 🔒 Seguridad

✅ El archivo `lidr.aws.pem` está protegido:
- Está en `.gitignore`
- NO se subirá al repositorio
- Solo su contenido está en GitHub Secrets (encriptado)

❌ NUNCA:
- Compartas este archivo públicamente
- Lo subas a GitHub
- Lo incluyas en el código

