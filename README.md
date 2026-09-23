# 📚 Documentación Técnica — Comparador de Precios de Medicamentos (El Salvador)

Bienvenido a la documentación técnica y operativa del ecosistema **Comparador de Precios de Medicamentos — El Salvador**.

---

## 🗂️ Índice de Documentación

1. **[Arquitectura del Sistema (`architecture.md`)](./architecture.md)**
   - Topología de microservicios independientes.
   - Diagrama de flujo de datos y scraping.
   - Integración de Inteligencia Artificial (Gemini) para normalización y desambiguación.

2. **[Base de Datos y Modelo de Datos (`database.md`)](./database.md)**
   - Diccionario de tablas, relaciones y llaves foráneas.
   - Catálogo maestro vs productos por farmacia.
   - Histórico inmutable de precios y referencias regulatorias SRS (PVMP).

3. **[Guía y Especificación de Scrapers (`scraping.md`)](./scraping.md)**
   - Farmacias CEFAFA (Firma criptográfica HMAC-SHA256).
   - Farmacias Económicas (API REST POST).
   - Farmacias San Nicolás (Parser Blazor SSR).
   - Farmacias Camila (Adaptador web adaptativo con fallback).
   - Superintendencia de Regulación Sanitaria (SRS - Consulta Integral).

4. **[Guía de Despliegue en Railway (`deployment.md`)](./deployment.md)**
   - Pasos para desplegar la base de datos PostgreSQL.
   - Configuración de variables de entorno para backend, scraper-service y frontend.
   - Conexión CI/CD mediante los repositorios de GitHub en `nevorasoftware`.

---

## 🏛️ Repositorios en GitHub (`nevorasoftware`)

- **Backend**: [`https://github.com/nevorasoftware/farmacia-comparator-backend.git`](https://github.com/nevorasoftware/farmacia-comparator-backend.git)
- **Scraper Service**: [`https://github.com/nevorasoftware/farmacia-comparator-scraper-service.git`](https://github.com/nevorasoftware/farmacia-comparator-scraper-service.git)
- **Frontend Web App**: [`https://github.com/nevorasoftware/farmacia-comparator-frontend.git`](https://github.com/nevorasoftware/farmacia-comparator-frontend.git)
- **Documentación**: [`https://github.com/nevorasoftware/farmacia-comparator-docs.git`](https://github.com/nevorasoftware/farmacia-comparator-docs.git)
