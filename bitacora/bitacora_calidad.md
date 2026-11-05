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
| fact_Compras | Hechos | Una factura de balanceado | N_Factura | Proveedor, Calendario | 748 |
| fact_Kardex | Hechos | Una salida de bodega | N_Documento | Piscina, Calendario | 56.690 |
| fact_ParametrosAgua | Hechos (se promedia) | Una medición por piscina y día | Fecha + Piscina | Piscina, Calendario | 24.462 |
| fact_Liquidaciones | Hechos | Una línea (clase + talla) de liquidación | N_Liquidacion + Clase + Talla | Piscina, Empacadora, Calendario | 3.438 |
| fact_Gastos | Hechos (mensual) | Gasto de un rubro, sector y mes | Periodo + Centro_Costo + Rubro | Sector, Calendario | 1.863 |

## Inventario de fuentes

Archivo: `data/raw/AcuicolaSofrevi_BD_2021-2026.xlsx`. Sistema de origen y responsable de cada hoja: por confirmar con el cliente.

| Hoja | Tabla destino | Filas | Particularidades |
|---|---|---|---|
| MAESTRO_PISCINAS | dim_Piscina | 34 | Código duplicado (P-12); Area_ha con punto y coma |
| CICLOS_PRODUCCION | fact_Ciclos | 550 | Fechas en 4 formatos; separador decimal mezclado |
| ALIMENTACION_DIARIA | fact_Alimentacion | 57.480 | "S/D" en Kg_Aplicados; kilos negativos |
| KARDEX_BODEGA | fact_Kardex | 56.690 | Solo salidas; hora en columna aparte |
| COMPRAS_BALANCEADO | fact_Compras | 748 | IVA 0; diferencias de centavos en Subtotal |
| PARAMETROS_AGUA | fact_ParametrosAgua | 24.462 | pH 14 y 0 °C imposibles; vacíos |
| LIQUIDACIONES_EMPACADORAS | fact_Liquidaciones | 3.438 | Precio como texto con $; Precio como texto con $; 6 liquidaciones duplicadas |
| GASTOS_OPERATIVOS | fact_Gastos | 1.863 | Mensual, por sector; Periodo en 2 formatos |

## Log de novedades

Resumen del Log de calidad (`Log_Calidad.xlsx`, hoja 4). El detalle completo de cada novedad (evidencia, impacto y paso en Power Query) está en el Excel.

| N° | Tabla | Columna | Categoría | Problema | Filas | Severidad | Decisión | Estado |
|---|---|---|---|---|---|---|---|---|
| 1 | dim_Piscina | Codigo | Duplicados | P-12 aparece dos veces (área 7,5 vs 8,8 ha) | 2 | Alta | Se conserva 7,5 ha: cuadra con sus 17 ciclos | Escalado |
| 2 | dim_Piscina | Sector | Consistencia | Prefijo "Sector " mezclado con la letra sola | 6 | Media | Quitar prefijo, recortar, mayúsculas | Resuelto |
| 3 | dim_Piscina | Area_ha | Tipos | Texto con coma decimal | 5 | Alta | Coma a número, independiente de la región | Resuelto |
| 4 | fact_Ciclos | Piscina | Consistencia | 135 formas para 32 piscinas | 493 | Alta | Solo los dígitos, formato P-00 | Resuelto |
| 5 | fact_Ciclos | Fecha_Siembra, Fecha_Cosecha | Tipos | 4 formatos de fecha en la misma columna | 550 | Alta | fnTextoAFecha (día/mes probado) | Resuelto |
| 6 | fact_Ciclos | Fecha_Cosecha | Validez | 30/02/2021 en C-2021-013 | 1 | Media | Estimada (siembra + 106 días) y marcada | Escalado |
| 7 | fact_Ciclos | Tipo_Cosecha | Consistencia | 7 formas para 3 tipos | 160 | Media | N = NORMAL, probado por duración | Resuelto |
| 8 | fact_Ciclos | 4 columnas de cantidades | Tipos | Números como texto con coma decimal | 323 | Alta | fnTextoANumero | Resuelto |
| 9 | fact_Ciclos | Larvas_Sembradas_miles | Validez | C-2022-104 escrito en unidades | 1 | Alta | Dividido entre 1.000; la densidad no cuadra | Escalado |
| 10 | fact_Ciclos | Peso_Muestreo_g | Validez | SIG-000441 con 245 g | 1 | Media | Corregido a 24,5 g, confirmado con biomasa | Resuelto |
| 11 | fact_Ciclos | Laboratorio | Consistencia | 10 formas para 3 laboratorios y 4 vacíos | 364 | Alta | Palabra clave; vacío = Desconocido | Escalado |
| 12 | fact_Ciclos | ID_Ciclo | Duplicados | 3 ciclos cargados dos veces | 3 | Alta | Quitar solo filas idénticas | Resuelto |
| 13 | fact_Ciclos | Observaciones | Completitud | "-", "s/n" y "ok" como vacíos disfrazados | 109 | Media | Pasados a vacío | Resuelto |
| 14 | fact_Ciclos | Observaciones | Completitud | 8 emergencias sin causa registrada | 8 | Media | No se infiere la causa | Escalado |
| 15 | fact_Ciclos | Biomasa | Consistencia | Kilos en SIG y libras en C- | 326 | Alta | Unificada en libras (Biomasa_lb) | Escalado |
| 16 | fact_Ciclos | Sobrevivencia_pct | Completitud | El sistema anterior no la registraba | 221 | Media | Se deja vacía | Escalado |
| 17 | fact_Compras | Subtotal_USD / Precio_Saco_USD | Consistencia | Subtotal ≠ Cantidad × Precio por redondeo del precio | 655 | Baja | Sin cambios: Subtotal es el monto oficial | Resuelto |
| 18 | fact_Compras | N_Factura | Duplicados | 4 facturas cargadas dos veces (82.279 USD) | 4 | Alta | Quitar solo filas idénticas | Resuelto |
| 19 | fact_Compras | Proveedor | Consistencia | 13 formas para 3 proveedores | 154 | Alta | Palabra clave sin espacios ni puntos | Resuelto |

**Resumen:** 16 novedades · 9 resueltas · 7 escaladas al cliente.

## Preguntas para el cliente

| N° | Pregunta |
|---|---|
| 1 | P-12: ¿el área es 7,5 ha? ¿El año de construcción es 2018 o 2011? |
| 6 | ¿La cosecha del ciclo C-2021-013 fue el 30/04/2021? |
| 9 | En C-2022-104, ¿se sembraron 1.150,3 mil larvas o 793,6 mil? ¿Cuál es la densidad real? |
| 11 | ¿Cuál es el nombre oficial de cada laboratorio? |
| 14 | ¿Cuál fue la causa de las 8 cosechas de emergencia sin observación? |
| 15 | ¿Confirman libras en el sistema anterior y kilos en SIG? ¿El dashboard debe mostrar libras? |
| 16 | ¿Quieren una sobrevivencia estimada para 2021–2023, marcada como estimada? |
