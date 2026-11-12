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
| 20 | fact_Liquidaciones | Empacadora | Consistencia | Empacadora escrita de 10 formas para 3 empresas | 2.328 | Media | Sin espacios ni puntos + palabra clave → razón social oficial | Resuelto |
| 21 | fact_Liquidaciones | Precio_USD_lb | Tipos | 172 precios como texto con signo $ | 172 | Alta | fnTextoANumero ampliada para quitar $ y espacios; prueba de regresión en ciclos y compras | Resuelto |
| 22 | fact_Liquidaciones | (toda la fila) | Duplicados | 6 liquidaciones cargadas dos veces: 38 filas, 489.902,56 USD | 38 | Alta | Table.Distinct sin columnas: 3.438 → 3.400; clave N_Liquidacion + Clase + Talla única | Resuelto |
| 23 | fact_Liquidaciones | Piscina | Consistencia | Piscina en dos formatos: P-01 y PISC n | 1.699 | Media | Solo dígitos → P-00, igual que en ciclos | Resuelto |
| 24 | fact_Liquidaciones | Piscina | Completitud | Piscina vacía en 22 liquidaciones | 138 | Alta | Deducida por fecha de cosecha + libras más cercanas; validado 499/499; marcada en Piscina_Origen | Resuelto |
| 25 | fact_Alimentacion | Piscina | Consistencia | 96 formas (P-01, P01, PISC-01) para 32 piscinas | 50.479 | Media | Regla de ciclos: dígitos → P-00; probada: 32 piscinas, 0 huérfanas | Resuelto |
| 26 | fact_Alimentacion | Proveedor | Consistencia | 11 formas para 3 proveedores (incluye "Acuafeed" con C) | 46.146 | Media | Regla de compras sin cambios: palabra clave FEED / ANDINOS / NUTRIACUA | Resuelto |
| 27 | fact_Alimentacion | Kg_Aplicados | Completitud | 289 sin dato: 174 vacíos y 115 "S/D" | 289 | Baja | "S/D" → vacío; no se inventa ni se pone 0 | Resuelto |
| 28 | fact_Alimentacion | Kg_Aplicados | Validez | 226 raciones con kilos negativos | 226 | Alta | Error de signo: Number.Abs (sin signo se parecen a los positivos; repartidas entre los 4 registradores) | Resuelto |
| 29 | fact_Alimentacion | Kg_Aplicados | Plausibilidad | 1.645 raciones en 0 kg dentro del ciclo (los 56 de los 3 días previos a cosecha son ayuno normal) | 1.645 | Media | Sin cambios; validar con fact_ParametrosAgua y con el cliente | Escalado |
| 30 | fact_Alimentacion | (toda la fila) | Duplicados | 343 raciones cargadas dos veces | 343 | Alta | Table.Distinct: 57.480 → 57.137; clave Fecha + Piscina + Turno única | Resuelto |
| 31 | fact_Alimentacion | Turno | Consistencia | "mañana" en minúsculas junto a "MAÑANA" | 2.837 | Baja | Text.Upper | Resuelto |
| 32 | fact_Kardex | Tipo_Mov | Completitud | El kardex solo trae SALIDAS: no tiene entradas ni saldos de bodega | 56.690 | Media | Usar fact_Compras como entradas; no se calcula saldo de inventario | Escalado |
| 33 | fact_Kardex | Kg_Despachados | Consistencia entre tablas | Desde 2024, en P-17 a P-24 (Sector C), el 12 % del alimento despachado no aparece como ración: 185.687 kg ≈ 204.000 USD. Mismas piscinas 2021–2023: 0,3–0,6 % | 6.788 | Alta | Sin cambios en los datos; hallazgo confidencial para el cliente (en un caso real no se publicaría) | Escalado 
| 34 | fact_ParametrosAgua | Piscina | Consistencia | Piscina escrita como "PISC-01" en lugar de P-01 | ___ | Baja | Regla de siempre: solo dígitos → P-00 | Resuelto |
| 35 | fact_ParametrosAgua | OD_Madrugada_mgL | Validez | 245 vacíos + 96 con el centinela 99,9 (máximo físico ~15 mg/L) | 341 | Media | Centinela → vacío. No se estima | Resuelto |
| 36 | fact_ParametrosAgua | Temperatura_C | Validez | 245 vacíos + 98 con 0 °C (imposible en Ecuador) | 343 | Media | Centinela → vacío | Resuelto |
| 37 | fact_ParametrosAgua | pH | Validez | 245 vacíos + 98 con pH 14 (tope de la escala) | 343 | Media | Centinela → vacío (pH > 10 imposible) | Resuelto |
| 38 | fact_Gastos | Periodo | Tipos | Periodo en dos formatos: "ene-21" (hasta jun-23) y "2023-07" (desde jul-23) | 1863 | Media | Fecha_Mes = primer día del mes; abreviatura desconocida da error visible | Resuelto |
| 39 | fact_Gastos | Centro_Costo | Consistencia | Sector escrito corto ("A") en lugar de "Sector A" | 84 | Baja | Quitar "SECTOR " → clave A–D | Resuelto |
| 40 | fact_Gastos | Fecha_Mes | Completitud | Octubre 2026 sin gastos (mes no cerrado) | 27 | Media | No se estima; medidas de costo solo para meses con gastos | Escalado |

**Resumen:** 16 novedades · 9 resueltas · 7 escaladas al cliente.

## Preguntas para el cliente

| N° | Pregunta |
|---|---|
| 1 | P-12: ¿el área es 7,5 ha? ¿El año de construcción es 2018 o 2011? |
| 9 | En C-2022-104, ¿se sembraron 1.150,3 mil larvas o 793,6 mil? ¿Cuál es la densidad real? |
| 11 | ¿Cuál es el nombre oficial de cada laboratorio? |
| 14 | ¿Cuál fue la causa de las 8 cosechas de emergencia sin observación? |
| 15 | ¿Confirman libras en el sistema anterior y kilos en SIG? ¿El dashboard debe mostrar libras? |
| 16 | ¿Quieren una sobrevivencia estimada para 2021–2023, marcada como estimada? |
| 29 | ¿Los registros de alimentación con 0 kg dentro del ciclo son días sin alimentar (oxígeno bajo, muda, lluvia) o falta de registro? |
| 32 | ¿Existe un registro de entradas y saldos de bodega? El kardex recibido solo tiene salidas. |
| 33 | (Confidencial) Desde 2024, en las piscinas P-17 a P-24, el 12 % del alimento despachado por bodega no aparece como ración (185.687 kg). En las mismas piscinas, entre 2021 y 2023, la diferencia fue de 0,3–0,6 %. ¿Se puede revisar con guías de despacho e inventario físico? |
| — | ¿Por qué Empacadora Costa Dorada liquida en promedio 97,1 % de las libras de la guía, contra 99,0 % de las otras dos (209 liquidaciones)? |
| — | ¿Las ventas totales por año y por empacadora coinciden con contabilidad? |
| — | ¿Con qué criterio se reparten los gastos de ADMINISTRACION entre los sectores (área, número de piscinas, producción)? |
| 40 | ¿Confirman que octubre 2026 no estaba cerrado contablemente al generar el archivo? | Pendiente |

*La pregunta de la novedad 6 (fecha de cosecha de C-2021-013) se retiró: la liquidación 21908 confirmó el 10/05/2021.*
