# Consultas Power Query (M)

Copia de cada consulta en texto para poder leerla en GitHub sin abrir Power BI.

## Cómo exportar una consulta

1. En Power Query: clic derecho sobre la consulta → **Editor avanzado**.
2. Seleccionar todo (Ctrl+A) y copiar.
3. Pegar en un archivo `.pq` dentro de la carpeta de su grupo.

> Si guardas el proyecto en formato PBIP, el código M también queda dentro de `pbip/` (archivos `.tmdl`). Estos `.pq` son la versión cómoda de leer.

## Convención de nombres

| Grupo | Carpeta | Ejemplo |
|---|---|---|
| 01_Origen | `01_origen/` | `src_Ventas.pq` |
| 02_Staging | `02_staging/` | `stg_Ventas.pq` |
| 03_Final | `03_final/` | `fact_Ventas.pq`, `dim_Cliente.pq` |
| QA | `qa/` | `QA_Inventario.pq` |
| Funciones y parámetros | `funciones/` | `fnPiscina.pq`, `RutaArchivo.pq` |
