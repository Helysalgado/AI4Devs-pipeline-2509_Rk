# Prompts utilizados – Pipeline CI/CD Backend

Este documento recoge los prompts utilizados para diseñar y generar el pipeline de CI/CD del backend mediante asistentes de IA, como parte del ejercicio.

---

## 1. Prompt para tests de backend

**Objetivo:**  
Definir un paso de pipeline que ejecute automáticamente los tests del backend en cada push con Pull Request abierto.

**Prompt utilizado:**

> “Genera un workflow de GitHub Actions para un backend en Node.js con TypeScript que instale dependencias y ejecute tests usando npm y jest.”

---

## 2. Prompt para build del backend

**Objetivo:**  
Automatizar la compilación del backend TypeScript antes del despliegue.

**Prompt utilizado:**

> “Añade un paso al pipeline de GitHub Actions que genere el build de un proyecto Node.js con TypeScript utilizando el comando `npm run build`.”

---

## 3. Prompt para despliegue en EC2

**Objetivo:**  
Desplegar automáticamente el backend en una instancia EC2 utilizando SSH desde GitHub Actions.

**Prompt utilizado:**

> “Genera un job de GitHub Actions que se conecte por SSH a una instancia EC2 y despliegue un backend Node.js, copiando los archivos necesarios y reiniciando el servicio.”

---

## 4. Prompt para Prisma y base de datos

**Objetivo:**  
Aplicar migraciones de Prisma en el entorno de producción usando variables de entorno.

**Prompt utilizado:**

> “Configura el pipeline para ejecutar `prisma migrate deploy` utilizando la variable de entorno DATABASE_URL definida como secret en GitHub Actions.”

---

## 5. Prompt para gestión del proceso con PM2

**Objetivo:**  
Asegurar que el backend se ejecute de forma persistente tras el despliegue.

**Prompt utilizado:**

> “Añade al pipeline un paso para reiniciar o levantar el backend usando PM2 después del despliegue.”


