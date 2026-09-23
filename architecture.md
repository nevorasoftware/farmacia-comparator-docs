# 🏛️ Arquitectura del Sistema

El **Comparador de Precios de Medicamentos — El Salvador** está diseñado siguiendo una arquitectura de microservicios contenerizados y desacoplados, listos para despliegue automatizado en Railway.

---

## 📐 Diagrama de Arquitectura

```mermaid
flowchart TD
    subgraph FuentesExternas["Fuentes de Datos Oficiales en El Salvador"]
        F1["Farmacias San Nicolás<br/>(Blazor SSR)"]
        F2["Farmacias CEFAFA<br/>(API REST con HMAC)"]
        F3["Farmacias Económicas<br/>(Web API POST)"]
        F4["Farmacias Camila<br/>(Web estática)"]
        SRS["Superintendencia Regulación Sanitaria<br/>(PVMP y Registros)"]
    end

    subgraph ScraperMicroservice["Microservicio Scraper (Spring Boot 3.3)"]
        Scheduler["Scheduler Programado<br/>America/El_Salvador (Hora)"]
        Adapters["Adaptadores de Scraping"]
        Normalizer["Normalizador & Hashing SHA-256"]
        GeminiAI["Google Gemini API<br/>(Desambiguación & Normalización)"]
    end

    subgraph DatabaseLayer["Capa de Persistencia (Railway PostgreSQL)"]
        DB[(PostgreSQL 16<br/>Esquema Relacional + UUID)]
    end

    subgraph BackendMicroservice["Microservicio Backend API (Spring Boot 3.3)"]
        RestControllers["Controladores REST<br/>/api/products, /api/pharmacies, /api/scraping"]
        ComparisonEngine["Motor de Comparativa & Cálculo de Ahorro"]
        Swagger["OpenAPI / Swagger UI<br/>/swagger-ui.html"]
    end

    subgraph FrontendMicroservice["Frontend Web App (React 18 + Vite)"]
        UI["Portal Web Responsive<br/>(Buscador, Comparador, Catálogo, Dashboard)"]
    end

    F1 --> Adapters
    F2 --> Adapters
    F3 --> Adapters
    F4 --> Adapters
    SRS --> Adapters

    Scheduler --> Adapters
    Adapters --> Normalizer
    Normalizer -.->|Sólo si hash cambia| GeminiAI
    Normalizer --> DB

    DB --> BackendMicroservice
    BackendMicroservice --> FrontendMicroservice
```

---

## 🧩 Componentes

### 1. Backend REST API (`backend/`)
- **Framework**: Spring Boot 3.3.3, Java 17.
- **Persistencia**: Spring Data JPA + Hibernate con PostgreSQL y migraciones Flyway.
- **Documentación**: Springdoc OpenAPI / Swagger (`/swagger-ui.html`).
- **Seguridad**: Configuración CORS para el frontend de Vite y dominios de Railway.
- **Endpoints clave**:
  - `GET /api/products`: Catálogo paginado con filtros.
  - `GET /api/products/search?q={query}`: Búsqueda rápida.
  - `GET /api/products/{id}/comparison`: Comparación detallada de precios por farmacia, referencias SRS, cálculo de diferencia vs PVMP y sustitutos equivalentes.
  - `GET /api/pharmacies`: Directorio de farmacias.
  - `GET /api/scraping/status` y `POST /api/scraping/trigger`: Monitoreo y control manual.

### 2. Scraper & Scheduler Service (`scraper-service/`)
- **Framework**: Spring Boot 3.3.3, Java 17.
- **Orquestación**: Spring `@Scheduled` con zona horaria `America/El_Salvador` (cron: `0 0 8-17 * * *`).
- **Técnicas de extracción**:
  - **CEFAFA**: Firma HMAC-SHA256 en encabezados HTTP (`X-App-Key`, `X-Timestamp`, `X-Signature`).
  - **Económicas**: Peticiones JSON POST a `ObtenerArticulosPorNombre`.
  - **San Nicolás**: Parser HTML con Jsoup sobre tarjetas Blazor.
  - **Camila**: Adaptador con SSL flexible y fallback controlado.
  - **SRS**: Proveedor de registros sanitarios y Precios Máximos de Venta al Público (PVMP).
- **Control de Costos de IA**:
  - `ContentHashService`: Si el hash SHA-256 del contenido crudo no cambia, se reutiliza el resultado previo sin invocar a Gemini.

### 3. Frontend Web App (`frontend/`)
- **Stack**: React 18, TypeScript, Vite, React Router v6, Lucide React, Recharts.
- **Diseño**: Paleta institucional médica salvadoreña (`#112A46`, `#19A7A0`, `#2F9E6E`), tipografía *Plus Jakarta Sans*, 100% responsive para móviles, tablets y escritorio.
- **Vistas**:
  - **Home**: Buscador principal, KPIs generales, farmacias monitoreadas y productos destacados.
  - **Comparador**: Ficha del medicamento, banner regulatorio oficial de la SRS, tabla comparativa con ahorro vs PVMP, gráfico histórico con Recharts y alternativas genéricas/de marca.
  - **Catálogo**: Directorio filtrable por principio activo, presentación y ordenamiento por precio.
  - **Panel de Control**: Monitor de ejecuciones de scrapers y botón de disparo manual.

### 4. Base de Datos PostgreSQL (`database/`)
- Alojada en Railway.
- Tablas particionadas lógicamente entre catálogo maestro, productos por farmacia, histórico inmutable de precios, referencias regulatorias SRS y auditoría.
