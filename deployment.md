# 🚀 Guía de Despliegue en Railway

Esta guía explica paso a paso cómo desplegar todo el ecosistema del **Comparador de Precios de Medicamentos — El Salvador** en [Railway.app](https://railway.app/).

---

## 🏗️ Estructura de Servicios en Railway

Recomendamos crear un único **Proyecto** en Railway y dentro de él crear los siguientes 5 servicios:

1. **PostgreSQL** (Base de datos gestionada por Railway).
2. **Backend API** (Spring Boot 3.3).
3. **Scraper Service** (Spring Boot 3.3).
4. **Frontend Web App** (Vite + React con Nginx o Node).
5. **(Opcional) Redis** para caché de consultas frecuentes.

---

## 📋 Paso a Paso

### 1. Crear la Base de Datos PostgreSQL
1. En Railway, haz clic en **New** -> **Database** -> **Add PostgreSQL**.
2. Una vez creada, ve a la pestaña **Data** o **Query**.
3. Copia el contenido de [`database/railway_full_init.sql`](../database/railway_full_init.sql) y ejecútalo para crear las tablas y datos iniciales salvadoreños.
4. En la pestaña **Variables**, toma nota de:
   - `DATABASE_URL`
   - O los valores individuales: `PGHOST`, `PGPORT`, `PGUSER`, `PGPASSWORD`, `PGDATABASE`.

---

### 2. Desplegar el Backend API (`farmacia-comparator-backend`)
1. En el mismo proyecto, haz clic en **New** -> **GitHub Repo**.
2. Selecciona: `nevorasoftware/farmacia-comparator-backend`.
3. En la pestaña **Settings**:
   - Railway detectará el `Dockerfile` existente en la raíz del backend o el buildpack de Maven con Java 17.
4. En la pestaña **Variables**, añade:
   - `SERVER_PORT`: `8080`
   - `SPRING_DATASOURCE_URL`: `jdbc:postgresql://${{Postgres.PGHOST}}:${{Postgres.PGPORT}}/${{Postgres.PGDATABASE}}`
   - `SPRING_DATASOURCE_USERNAME`: `${{Postgres.PGUSER}}`
   - `SPRING_DATASOURCE_PASSWORD`: `${{Postgres.PGPASSWORD}}`
5. Railway generará un dominio público (ejemplo: `https://farmacia-backend-production.up.railway.app`).
   - Puedes verificar la documentación OpenAPI en `/swagger-ui.html`.

---

### 3. Desplegar el Scraper Service (`farmacia-comparator-scraper-service`)
1. Haz clic en **New** -> **GitHub Repo**.
2. Selecciona: `nevorasoftware/farmacia-comparator-scraper-service`.
3. En la pestaña **Variables**, añade:
   - `SPRING_DATASOURCE_URL`: `jdbc:postgresql://${{Postgres.PGHOST}}:${{Postgres.PGPORT}}/${{Postgres.PGDATABASE}}`
   - `SPRING_DATASOURCE_USERNAME`: `${{Postgres.PGUSER}}`
   - `SPRING_DATASOURCE_PASSWORD`: `${{Postgres.PGPASSWORD}}`
   - `GEMINI_API_KEY`: Tu clave API de Google Gemini (para normalización).
   - `SCRAPER_CRON`: `0 0 8-17 * * *` (o la frecuencia deseada en zona de El Salvador).
4. El servicio correrá de fondo ejecutando los scrapers en el horario programado.

---

### 4. Desplegar el Frontend Web App (`farmacia-comparator-frontend`)
1. Haz clic en **New** -> **GitHub Repo**.
2. Selecciona: `nevorasoftware/farmacia-comparator-frontend`.
3. En la pestaña **Variables**, añade:
   - `VITE_API_URL`: La URL pública de tu backend (ej. `https://farmacia-backend-production.up.railway.app`).
4. Railway compilará con `npm run build` y servirá la aplicación.
5. Genera un dominio público para el frontend en la pestaña **Settings** -> **Networking** -> **Generate Domain**.

---

## ✅ Verificación del Despliegue

1. Abre la URL del Frontend: la página de inicio debe mostrar el buscador con autocompletado y los medicamentos populares.
2. Ingresa a la ficha de cualquier producto (ej. `/comparar/1`): se desplegará la comparativa de farmacias, el gráfico histórico y la referencia oficial de la SRS con el cálculo de ahorro.
3. Ingresa a `/admin`: verás el panel de control de scrapers con telemetría en tiempo real.
