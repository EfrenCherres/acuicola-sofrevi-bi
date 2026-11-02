# Bitácora de calidad de datos

Versión legible en GitHub de [`Log_Calidad.xlsx`](Log_Calidad.xlsx). Allí están las tablas del modelo, el inventario de fuentes, la matriz de controles y los totales de control; aquí se publica el resumen y el log de novedades. Se actualiza al cerrar cada tabla.

**Severidad:** Alta = cambia cifras o rompe el modelo · Media = distorsiona un análisis · Baja = estética o estandarización.
**Estado:** Pendiente → Resuelto, o Escalado (enviado al cliente, esperando respuesta).

## Resumen

| Indicador | Valor |
|---|---|
| Tablas del modelo | 13 (6 dimensiones, 7 hechos) |
| Fuentes inventariadas | 8 |
| Controles registrados / completados | 0 / 0 (Fase 3) |
| Novedades (Alta / Media / Baja) | Se registran en la Fase 3 |
| Fuentes cuyo cuadre de carga no cuadra | Se mide en la Fase 2 |

## Tablas del modelo

| Tabla | Tipo | Granularidad | Clave | Se relaciona con | Filas |
|---|---|---|---|---|---|
| dim_Calendario | Calendario | Un día | Fecha | Todas las tablas de hechos | Se crea |
| dim_Piscina | Dimensión | Una piscina | Codigo | Ciclos, Alimentación, Agua, Kardex, Liquidaciones, Sector | 34 (33 distintas) |
| dim_Sector | Dimensión | Un sector o centro de costo | Sector | Piscina, Gastos | Derivada |
| dim_Proveedor | Dimensión | Un proveedor | Proveedor | Compras, Alimentación | Derivada |
| dim_Empacadora | Dimensión | Una empacadora | Empacadora | Liquidaciones | Derivada |
| dim_Laboratorio | Dimensión | Un laboratorio | Laboratorio | Ciclos | Derivada |
| fact_Ciclos | Hechos | Un ciclo de producción | ID_Ciclo | Piscina, Laboratorio, Calendario | 550 |
| fact_Alimentacion | Hechos | Una aplicación por piscina, día y turno | Fecha + Piscina + Turno | Piscina, Proveedor, Calendario | 57.480 |
| fact_Compras | Hechos | Una factura de balanceado | N_Factura | Proveedor, Calendario | 7.489 |
| fact_Kardex | Hechos | Una salida de bodega | N_Documento | Piscina, Calendario | 56.690 |
| fact_ParametrosAgua | Hechos (se promedia) | Una medición por piscina y día | Fecha + Piscina | Piscina, Calendario | 24.462 |
| fact_Liquidaciones | Hechos | Una línea (clase + talla) de liquidación | N_Liquidacion + Clase + Talla | Piscina, Empacadora, Calendario | 3.300 |
| fact_Gastos | Hechos (mensual) | Gasto de un rubro, sector y mes | Periodo + Centro_Costo + Rubro | Sector, Calendario | 1.863 |

## Inventario de fuentes

Archivo: `data/raw/AcuicolaSofrevi_BD_2021-2026.xlsx`. Sistema de origen y responsable de cada hoja: por confirmar con el cliente.

| Hoja | Tabla destino | Filas | Particularidades |
|---|---|---|---|
| MAESTRO_PISCINAS | dim_Piscina | 34 | Código duplicado (P-12); Area_ha con punto y coma |
| CICLOS_PRODUCCION | fact_Ciclos | 550 | Fechas en 4 formatos; separador decimal mezclado |
| ALIMENTACION_DIARIA | fact_Alimentacion | 57.480 | "S/D" en Kg_Aplicados; kilos negativos |
| KARDEX BODEGA | fact_Kardex | 56.690 | Solo salidas; hora en columna aparte |
| COMPRAS_BALANCEADO | fact_Compras | 7.489 | IVA 0; diferencias de centavos en Subtotal |
| PARAMETROS_AGUA | fact_ParametrosAgua | 24.462 | pH 14 y 0 °C imposibles; vacíos |
| LIQUIDACIONES_EMPACADORAS | fact_Liquidaciones | 3.300 | Precio como texto con $; liquidación 23101 incompleta |
| GASTOS_OPERATIVOS | fact_Gastos | 1.863 | Mensual, por sector; Periodo en 2 formatos |

## Log de novedades

Se llena en la Fase 3, una fila por cada problema encontrado al limpiar cada tabla.

| N° | Tabla | Columna | Categoría | Problema detectado | Cómo lo detecté (evidencia) | Filas afectadas | % de la tabla | Severidad | Impacto en el análisis | Decisión / tratamiento | Paso en Power Query | Validar con cliente | Pregunta al cliente | Respuesta del cliente | Estado | Fecha |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | | | | | | | | | | | | | | | | |