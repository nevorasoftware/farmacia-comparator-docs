# 🕷️ Especificación Técnica de Scrapers Oficiales

El microservicio `scraper-service` implementa recolectores especializados para cada fuente salvadoreña.

---

## 1. Farmacias CEFAFA (`portal.farmaciascefafa.com.sv`)
- **Tipo**: API REST protegida por firma digital.
- **Endpoint**: `GET https://portal.farmaciascefafa.com.sv/api-ecommerce/api/productos/buscador/{query}`
- **Autenticación y Encabezados Requeridos**:
  - `X-App-Key`: Clave pública de la aplicación cliente.
  - `X-Timestamp`: Marca de tiempo Unix en milisegundos.
  - `X-Signature`: Firma criptográfica calculada mediante **HMAC-SHA256**:
    $$\text{Signature} = \text{HMAC-SHA256}(\text{X-App-Key} + \text{X-Timestamp} + \text{query}, \text{secret})$$
- **Datos extraídos**:
  - `producto_id`, `nombre`, `marca`, `precio_regular`, `precio_oferta`, `imagen_url`, `url_slug`.
- **Implementación**: [`CefafaScraper.java`](file:///C:/Users/jonathan.giron/Documents/farmacia-comparator/scraper-service/src/main/java/com/luppo/farmacia/scraper/service/scraper/CefafaScraper.java)

---

## 2. Farmacias Económicas (`farmaciaseconomicaselsalvador.com`)
- **Tipo**: Web API oficial (JSON POST).
- **Endpoint**: `POST https://www.farmaciaseconomicaselsalvador.com/PROD/ECOMMERCE/API/Articulo/ObtenerArticulosPorNombre`
- **Payload**:
  ```json
  {
    "ArticuloNombre": "{query}",
    "Pagina": 1,
    "TamanoPagina": 20,
    "Ordenamiento": 0
  }
  ```
- **Datos extraídos**:
  - `ArticuloID`, `NombreArticulo`, `Precio`, `PrecioDescuento`, `Categoria`, `URLImagen`, `TieneExistencia`.
- **Implementación**: [`EconomicasScraper.java`](file:///C:/Users/jonathan.giron/Documents/farmacia-comparator/scraper-service/src/main/java/com/luppo/farmacia/scraper/service/scraper/EconomicasScraper.java)

---

## 3. Farmacias San Nicolás (`farmaciasannicolas.com`)
- **Tipo**: Blazor Server-Side Rendering (SSR) con páginas HTML estáticas.
- **URL Base**: `https://www.farmaciasannicolas.com/productos/landing/...`
- **Técnica de Extracción**:
  - Petición HTTP con encabezados de navegador real (User-Agent, Accept-Language).
  - Parser HTML determinístico mediante **Jsoup**:
    - Nombre del producto: `.prod-name`, `h2`
    - Precio regular: `.before`, `.regular-price`
    - Precio especial/tarjeta: `.pp-price`, `.offer-price`
    - SKU / Enlace: atributo `href` en tarjetas de catálogo.
- **Implementación**: [`SanNicolasScraper.java`](file:///C:/Users/jonathan.giron/Documents/farmacia-comparator/scraper-service/src/main/java/com/luppo/farmacia/scraper/service/scraper/SanNicolasScraper.java)

---

## 4. Farmacias Camila (`farmaciascamila.com`)
- **Tipo**: Catálogo web corporativo.
- **Estado Técnico Actual**:
  - Certificado SSL con advertencia de validación y `/productos.html` actualmente responde con 404 (sin tienda e-commerce abierta al público).
- **Estrategia Adaptativa**:
  - Conexión con `SSLContext` permisivo para evitar fallos de red por certificados expirados.
  - Inyección de catálogo de referencia semilla para mantener la comparativa mientras habilitan su portal transaccional.
- **Implementación**: [`CamilaScraper.java`](file:///C:/Users/jonathan.giron/Documents/farmacia-comparator/scraper-service/src/main/java/com/luppo/farmacia/scraper/service/scraper/CamilaScraper.java)

---

## 5. Superintendencia de Regulación Sanitaria (SRS)
- **Portal**: `http://info.medicamentos.gob.sv`
- **Consulta**: Catálogo integral de Registros Sanitarios y Precios Máximos de Venta al Público (PVMP) amparados por la Ley de Medicamentos de El Salvador.
- **Implementación**: [`SrsDataProvider.java`](file:///C:/Users/jonathan.giron/Documents/farmacia-comparator/scraper-service/src/main/java/com/luppo/farmacia/scraper/service/scraper/SrsDataProvider.java)
