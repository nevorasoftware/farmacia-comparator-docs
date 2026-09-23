# 🗄️ Modelo de Datos y Diccionario de Base de Datos

La base de datos utiliza **PostgreSQL 16** con la extensión `uuid-ossp` habilitada. Los scripts completos se encuentran en `database/railway_full_init.sql`.

---

## 📊 Diccionario de Tablas

### 1. `pharmacy`
Almacena las cadenas de farmacias activas en El Salvador.
- `id` (BIGSERIAL, PK): Identificador único.
- `name` (VARCHAR): Nombre oficial (ej. 'Farmacias San Nicolás').
- `code` (VARCHAR, UNIQUE): Código interno (SAN_NICOLAS, CEFAFA, CAMILA, ECONOMICAS).
- `base_url` (VARCHAR): URL base del sitio web o portal de compras.
- `logo_url` (VARCHAR): Enlace a logo representativo.
- `is_active` (BOOLEAN): Estado operativo.

### 2. `master_product`
Catálogo unificado y normalizado de medicamentos.
- `id` (BIGSERIAL, PK): Identificador del producto unificado.
- `name` (VARCHAR): Nombre representativo (ej. 'Acetaminofén 500 mg').
- `active_ingredient` (VARCHAR): Principio activo principal (ej. 'Acetaminofén').
- `concentration` (VARCHAR): Concentración (ej. '500 mg', '120 mg/5 mL').
- `pharmaceutical_form` (VARCHAR): Forma farmacéutica (Tableta, Cápsula, Jarabe).
- `administration_route` (VARCHAR): Vía de administración (Oral, Tópica, Oftálmica).
- `brand` (VARCHAR): Marca comercial de referencia o 'Genérico'.
- `laboratory` (VARCHAR): Laboratorio fabricante.
- `health_registration` (VARCHAR): Número de Registro Sanitario salvadoreño.
- `chm` (VARCHAR): Código Homologado de Medicamentos de la SRS.
- `presentation` (VARCHAR): Descripción de la presentación comercial.
- `quantity` (INTEGER): Cantidad de unidades en el empaque.
- `unit` (VARCHAR): Unidad de medida (tableta, frasco, ampolla).

### 3. `pharmacy_product`
Registro específico del producto publicado por cada cadena de farmacias.
- `id` (BIGSERIAL, PK): Identificador único.
- `pharmacy_id` (BIGINT, FK -> pharmacy.id): Farmacia que comercializa.
- `master_product_id` (BIGINT, FK -> master_product.id): Enlace al producto unificado.
- `external_id` (VARCHAR): SKU o identificador del producto en la farmacia.
- `original_name` (VARCHAR): Nombre exacto tal como aparece en la farmacia.
- `brand` (VARCHAR): Marca detectada en la farmacia.
- `url` (VARCHAR): Enlace directo de compra o ficha del producto.
- `image_url` (VARCHAR): Enlace a la imagen del producto.
- `current_price` (DECIMAL): Precio regular actual en USD.
- `current_offer_price` (DECIMAL): Precio de oferta vigente (si aplica).
- `is_available` (BOOLEAN): Disponibilidad o stock reportado.
- `content_hash` (VARCHAR): Hash SHA-256 para control de cambios.
- `last_scraped_at` (TIMESTAMP): Fecha y hora de la última recolección.

### 4. `price_history`
Historial cronológico inmutable de variaciones de precios para análisis y gráficos.
- `id` (BIGSERIAL, PK).
- `pharmacy_product_id` (BIGINT, FK -> pharmacy_product.id).
- `price` (DECIMAL): Precio regular registrado.
- `offer_price` (DECIMAL): Precio de descuento registrado.
- `is_available` (BOOLEAN): Estado de inventario al momento del registro.
- `checked_at` (TIMESTAMP): Fecha y hora del registro.

### 5. `srs_product`
Información regulatoria oficial provista por la Superintendencia de Regulación Sanitaria.
- `id` (BIGSERIAL, PK).
- `master_product_id` (BIGINT, FK -> master_product.id).
- `health_registration` (VARCHAR, UNIQUE): Número oficial de registro sanitario.
- `chm` (VARCHAR): Código Homologado de Medicamentos.
- `product_name` (VARCHAR): Denominación oficial registrada en el país.
- `active_ingredient` (VARCHAR): Denominación Común Internacional (DCI).
- `source_url` (VARCHAR): Enlace de consulta en el portal de la SRS.

### 6. `srs_price`
Precios regulados y techos oficiales establecidos por la SRS.
- `id` (BIGSERIAL, PK).
- `srs_product_id` (BIGINT, FK -> srs_product.id).
- `pvmp` (DECIMAL): **Precio Máximo de Venta al Público Regulado** (techo legal).
- `pvmp_unit` (DECIMAL): Precio máximo por unidad dosificada.
- `pvmp_presentation` (DECIMAL): Precio máximo por presentación comercial.
- `market_price` (DECIMAL): Precio promedio de mercado reportado por la SRS.
- `effective_date` (DATE): Fecha de entrada en vigencia del acuerdo de precios.

### 7. `product_match`
Auditoría y trazabilidad del emparejamiento entre productos de farmacias y el catálogo maestro.
- `match_score` (DECIMAL): Nivel de confianza (0.0000 a 1.0000).
- `match_method` (VARCHAR): EXACT, EMBEDDING o AI.
- `match_status` (VARCHAR): PENDING, CONFIRMED, REJECTED.

### 8. `scraping_execution`
Registro de telemetría y auditoría de cada ejecución del scraper programado.
- `pharmacy_id` (BIGINT, FK).
- `started_at` y `finished_at` (TIMESTAMP).
- `status` (VARCHAR): COMPLETED, FAILED, RUNNING.
- `records_found`, `records_created`, `records_updated`, `records_failed` (INTEGER).
- `error_message` (TEXT).

### 9. `ai_processing_log`
Auditoría de consumo y costo de la API de Gemini.
- `model_name` (VARCHAR), `operation` (VARCHAR).
- `content_hash` (VARCHAR): Hash evaluado.
- `tokens_prompt`, `tokens_response` (INTEGER).
- `cost_usd` (DECIMAL), `latency_ms` (BIGINT).
