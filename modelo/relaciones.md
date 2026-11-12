# Modelo de datos · Acuícola Sofrevi

Modelo en estrella con 7 dimensiones y 7 tablas de hechos. Las 26 relaciones son **varios a uno (\*:1)** con **filtro en una sola dirección**. Ninguna relación une dos tablas de hechos.

## Tablas

| Tipo | Tabla | Filas | Clave o granularidad |
|---|---|---|---|
| Dimensión | Dim_Calendario (DAX) | 2.191 | Date |
| Dimensión | dim_Piscina | 33 | Piscina |
| Dimensión | dim_Sector | 5 | Sector |
| Dimensión | dim_Ciclo | 547 | ID_Ciclo |
| Dimensión | dim_Laboratorio | 4 | Laboratorio |
| Dimensión | dim_Proveedor | 3 | Proveedor |
| Dimensión | dim_Empacadora | 3 | Empacadora |
| Hechos | fact_Ciclos | 547 | ID_Ciclo |
| Hechos | fact_Liquidaciones | 3.400 | N_Liquidacion + Clase + Talla |
| Hechos | fact_Alimentacion | 57.137 | Fecha + Piscina + Turno |
| Hechos | fact_Kardex | 56.690 | N_Documento |
| Hechos | fact_ParametrosAgua | 24.462 | Fecha + Piscina |
| Hechos | fact_Compras | 744 | Una línea de factura |
| Hechos | fact_Gastos | 1.863 | Fecha_Mes + Centro_Costo + Rubro |

## Relaciones

| N° | Lado uno | Lado varios | Estado |
|---|---|---|---|
| 1 | Dim_Calendario[Date] | fact_Alimentacion[Fecha] | Activa |
| 2 | Dim_Calendario[Date] | fact_Kardex[Fecha] | Activa |
| 3 | Dim_Calendario[Date] | fact_ParametrosAgua[Fecha] | Activa |
| 4 | Dim_Calendario[Date] | fact_Compras[Fecha_Factura] | Activa |
| 5 | Dim_Calendario[Date] | fact_Gastos[Fecha_Mes] | Activa |
| 6 | Dim_Calendario[Date] | fact_Ciclos[Fecha_Cosecha] | Activa |
| 7 | Dim_Calendario[Date] | fact_Ciclos[Fecha_Siembra] | Inactiva |
| 8 | Dim_Calendario[Date] | fact_Liquidaciones[Fecha_Liquidacion] | Activa |
| 9 | Dim_Calendario[Date] | fact_Liquidaciones[Fecha_Cosecha] | Inactiva |
| 10 | dim_Piscina[Piscina] | fact_Ciclos[Piscina] | Activa |
| 11 | dim_Piscina[Piscina] | fact_Liquidaciones[Piscina] | Activa |
| 12 | dim_Piscina[Piscina] | fact_Alimentacion[Piscina] | Activa |
| 13 | dim_Piscina[Piscina] | fact_Kardex[Piscina] | Activa |
| 14 | dim_Piscina[Piscina] | fact_ParametrosAgua[Piscina] | Activa |
| 15 | dim_Ciclo[ID_Ciclo] | fact_Ciclos[ID_Ciclo] | Activa |
| 16 | dim_Ciclo[ID_Ciclo] | fact_Liquidaciones[ID_Ciclo] | Activa |
| 17 | dim_Ciclo[ID_Ciclo] | fact_Alimentacion[ID_Ciclo] | Activa |
| 18 | dim_Ciclo[ID_Ciclo] | fact_Kardex[ID_Ciclo] | Activa |
| 19 | dim_Ciclo[ID_Ciclo] | fact_ParametrosAgua[ID_Ciclo] | Activa |
| 20 | dim_Sector[Sector] | dim_Piscina[Sector] | Activa (copo de nieve) |
| 21 | dim_Sector[Sector] | fact_Gastos[Centro_Costo] | Activa |
| 22 | dim_Proveedor[Proveedor] | fact_Compras[Proveedor] | Activa |
| 23 | dim_Proveedor[Proveedor] | fact_Alimentacion[Proveedor] | Activa |
| 24 | dim_Proveedor[Proveedor] | fact_Kardex[Proveedor] | Activa |
| 25 | dim_Laboratorio[Laboratorio] | fact_Ciclos[Laboratorio] | Activa |
| 26 | dim_Empacadora[Empacadora] | fact_Liquidaciones[Empacadora] | Activa |

## Decisiones de diseño

**dim_Ciclo separada de fact_Ciclos.** fact_Ciclos tiene una fila por ciclo y también es el ciclo al que apuntan alimentación, kardex, agua y liquidaciones. Si se usaba como dimensión, el filtro de piscina llegaba a esas tablas por dos caminos (directo y a través del ciclo) y el modelo quedaba ambiguo. dim_Ciclo solo tiene atributos (ID_Ciclo, Sistema, Tipo_Cosecha, Cosecha_Estimada) y no se relaciona con dim_Piscina ni con el calendario: cada filtro tiene un solo camino.

**dim_Ciclo → fact_Ciclos se declara varios a uno, aunque los datos son 1 a 1.** Power BI proponía uno a uno, que obliga a filtrar en ambas direcciones y abría un camino ambiguo hacia dim_Piscina. La cardinalidad decide quién filtra a quién, no solo cómo son los datos.

**Fechas activas e inactivas.**
- fact_Ciclos: activa **Fecha_Cosecha**, porque la producción, el peso y la sobrevivencia se conocen al cosechar. Fecha_Siembra se usa con `USERELATIONSHIP` (ciclos y larvas sembradas en el mes).
- fact_Liquidaciones: activa **Fecha_Liquidacion**, que responde a cuánto dinero entró en el mes. Fecha_Cosecha se usa con `USERELATIONSHIP`.

**fact_Gastos es mensual.** Se relaciona por Fecha_Mes (primer día del mes) y con dim_Sector por Centro_Costo, que incluye ADMINISTRACION.

**Fecha y hora automática apagada.** Power BI había creado 4 tablas de fechas ocultas (LocalDateTable) para las columnas de fecha del calendario. Se apagó la opción, en el archivo y en la configuración global.

**Relaciones automáticas descartadas.** Power BI había creado unas 30 relaciones por coincidencia de nombres, entre ellas relaciones entre tablas de hechos y otras desactivadas por ambigüedad. Se borraron todas y el modelo se construyó desde el diseño.

## Validación

| Control | Resultado |
|---|---|
| Relaciones (`INFO.VIEW.RELATIONSHIPS`) | 26, todas Many → One, OneDirection; 2 inactivas |
| Filas huérfanas (`RELATED` en blanco) | 0 en las 23 relaciones activas revisadas; 25 en fact_Ciclos → calendario = ciclos EN CURSO sin cosecha (esperado) |
| Columnas visibles que se suman solas (`INFO.VIEW.COLUMNS`) | 16, todas cantidades sumables |

## Limpieza del panel de Datos

- **Claves ocultas en las tablas de hechos** (Piscina, ID_Ciclo, Proveedor, Laboratorio, Empacadora, Centro_Costo y fechas). Así los filtros se toman siempre de las dimensiones.
- **"No resumir"** en todo número que no tiene un total lógico: precios, pH, oxígeno, temperatura, salinidad, transparencia, peso, densidad, sobrevivencia, días de cultivo, año de construcción, Orden_Talla y las columnas numéricas del calendario. Se analizan con medidas DAX (promedios o ponderados).
- **Columnas de orden del calendario ocultas.** Siguen ordenando Mes, Trimestre, etc.
- **Visibles a propósito:** fact_Kardex[Hora] (análisis por hora del día) y fact_Liquidaciones[Piscina_Origen] (marca de las piscinas inferidas).

## Para las medidas DAX (Fase 5)

- El costo por libra y el margen se calculan **solo en meses con gastos**: octubre 2026 no tiene gastos (novedad 40).
- Ventas por fecha de cosecha y ciclos sembrados: `USERELATIONSHIP`.
- El calendario tiene "Con Datos" hasta el 20/10/2026 para ocultar fechas sin operación.
