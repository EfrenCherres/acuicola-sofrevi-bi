# Medidas DAX · Acuícola Sofrevi

**50 medidas en 8 carpetas** (01. Producción, 02. Ventas, 03. Alimentación, 04. Compras, 05. Hallazgos, 06. Agua, 07. Costos y Margen, 99. Reels).

Todas las medidas viven en la tabla **Medidas DAX**, organizadas en carpetas. Cada medida se probó en la vista de consultas DAX (DEFINE + EVALUATE) y se comparó con los totales de control del ETL antes de guardarla en el modelo.

## Reglas que siguen todas las medidas

- **Promedios ponderados, no AVERAGE**, cuando las filas pesan distinto. Un ciclo de 50.000 lb no puede contar igual que uno de 5.000 lb.
- **Un precio es un cociente** (ventas ÷ libras), nunca el promedio de una columna de precios.
- **Fechas inactivas con `USERELATIONSHIP`**: siembra de ciclos y cosecha de liquidaciones.
- **El nivel correcto antes de dividir**: la conversión alimenticia se calcula por ciclo, no por mes.
- **`DIVIDE`** en lugar de `/` para no mostrar errores cuando el divisor es cero o vacío.

---

## 01. Producción

| Medida | Formato |
|---|---|
| Ciclos Cosechados | Entero |
| Ciclos Sembrados | Entero |
| Biomasa Cosechada (lb) | Entero |
| Biomasa Cosechada (kg) | Entero |
| Larvas Sembradas (miles) | Entero |
| Peso Promedio (g) | 1 decimal |
| Sobrevivencia % | Porcentaje, 1 decimal |
| Días de Cultivo Promedio | 1 decimal |

```dax
Ciclos Cosechados =
CALCULATE ( COUNTROWS ( fact_Ciclos ), KEEPFILTERS ( NOT ISBLANK ( fact_Ciclos[Fecha_Cosecha] ) ) )
```
Solo cuenta ciclos con fecha de cosecha; sin el filtro, un total sin fechas sumaría también los 25 EN CURSO.

```dax
Ciclos Sembrados =
CALCULATE ( COUNTROWS ( fact_Ciclos ), USERELATIONSHIP ( Dim_Calendario[Date], fact_Ciclos[Fecha_Siembra] ) )

Larvas Sembradas (miles) =
CALCULATE ( SUM ( fact_Ciclos[Larvas_Sembradas_miles] ), USERELATIONSHIP ( Dim_Calendario[Date], fact_Ciclos[Fecha_Siembra] ) )
```
Activan la fecha de siembra, que en el modelo es una relación inactiva.

```dax
Biomasa Cosechada (lb) = SUM ( fact_Ciclos[Biomasa_lb] )

Biomasa Cosechada (kg) = [Biomasa Cosechada (lb)] / 2.20462
```
La versión en kg existe para dividir contra el alimento, que está en kg.

```dax
Peso Promedio (g) =
VAR T = FILTER ( fact_Ciclos, NOT ISBLANK ( fact_Ciclos[Peso_Muestreo_g] ) && NOT ISBLANK ( fact_Ciclos[Biomasa_lb] ) )
RETURN DIVIDE ( SUMX ( T, fact_Ciclos[Peso_Muestreo_g] * fact_Ciclos[Biomasa_lb] ), SUMX ( T, fact_Ciclos[Biomasa_lb] ) )
```
Ponderado por biomasa.

```dax
Sobrevivencia % =
VAR T = FILTER ( fact_Ciclos, NOT ISBLANK ( fact_Ciclos[Sobrevivencia_pct] ) )
RETURN DIVIDE ( SUMX ( T, fact_Ciclos[Larvas_Sembradas_miles] * fact_Ciclos[Sobrevivencia_pct] ), SUMX ( T, fact_Ciclos[Larvas_Sembradas_miles] ) ) / 100
```
Ponderada por larvas sembradas. La columna viene en escala 0–100. Vacía en 2021–2022: el sistema anterior (C-) no la registraba.

```dax
Días de Cultivo Promedio = AVERAGE ( fact_Ciclos[Dias_Cultivo] )
```
Promedio simple: aquí cada ciclo es una unidad de análisis.

## 02. Ventas

| Medida | Formato |
|---|---|
| Libras Vendidas | Entero |
| Libras en Guía | Entero |
| % Recibido en Planta | Porcentaje, 1 decimal |
| Ventas USD | Moneda, 0 decimales |
| Ventas por Fecha de Cosecha | Moneda, 0 decimales |
| Precio Promedio (USD/lb) | Moneda, 2 decimales |

```dax
Libras Vendidas = SUM ( fact_Liquidaciones[Lbs_Recibidas] )

Ventas USD = SUM ( fact_Liquidaciones[Total_USD] )

Precio Promedio (USD/lb) = DIVIDE ( [Ventas USD], [Libras Vendidas] )
```

```dax
Libras en Guía =
SUMX ( VALUES ( fact_Liquidaciones[N_Liquidacion] ), CALCULATE ( MAX ( fact_Liquidaciones[Lbs_Guia_Remision] ) ) )
```
La guía es el peso total de la cosecha **repetido en cada fila** de la liquidación (una fila por clase y talla). Se toma una guía por liquidación y después se suman. Por eso la columna está en "No resumir".

```dax
% Recibido en Planta = DIVIDE ( [Libras Vendidas], [Libras en Guía] )

Ventas por Fecha de Cosecha =
CALCULATE ( [Ventas USD], USERELATIONSHIP ( Dim_Calendario[Date], fact_Liquidaciones[Fecha_Cosecha] ) )
```

## 03. Alimentación

| Medida | Formato |
|---|---|
| Alimento Aplicado (kg) | Entero |
| Alimento Despachado (kg) | Entero |
| Diferencia Bodega vs Piscina (kg) | Entero |
| % Diferencia Bodega vs Piscina | Porcentaje, 1 decimal |
| Alimento de Ciclos Cosechados (kg) | Entero |
| FCA | 2 decimales |
| FCA con Despacho | 2 decimales |

```dax
Alimento Aplicado (kg) = SUM ( fact_Alimentacion[Kg_Aplicados] )

Alimento Despachado (kg) = SUM ( fact_Kardex[Kg_Despachados] )

Diferencia Bodega vs Piscina (kg) = [Alimento Despachado (kg)] - [Alimento Aplicado (kg)]

% Diferencia Bodega vs Piscina = DIVIDE ( [Diferencia Bodega vs Piscina (kg)], [Alimento Despachado (kg)] )
```

```dax
Alimento de Ciclos Cosechados (kg) =
VAR Ciclos =
    CALCULATETABLE ( VALUES ( fact_Ciclos[ID_Ciclo] ), KEEPFILTERS ( NOT ISBLANK ( fact_Ciclos[Fecha_Cosecha] ) ) )
RETURN
    CALCULATE ( [Alimento Aplicado (kg)], REMOVEFILTERS ( Dim_Calendario ), TREATAS ( Ciclos, dim_Ciclo[ID_Ciclo] ) )

FCA = DIVIDE ( [Alimento de Ciclos Cosechados (kg)], [Biomasa Cosechada (kg)] )
```
El alimento se aplica durante todo el ciclo y la cosecha ocurre al final. La medida toma los ciclos cosechados en el periodo, quita el filtro de fecha y trae el alimento de toda su vida. Los filtros de piscina y sector se mantienen.

```dax
FCA con Despacho =
VAR Ciclos =
    CALCULATETABLE ( VALUES ( fact_Ciclos[ID_Ciclo] ), KEEPFILTERS ( NOT ISBLANK ( fact_Ciclos[Fecha_Cosecha] ) ) )
RETURN
    DIVIDE (
        CALCULATE ( [Alimento Despachado (kg)], REMOVEFILTERS ( Dim_Calendario ), TREATAS ( Ciclos, dim_Ciclo[ID_Ciclo] ) ),
        [Biomasa Cosechada (kg)]
    )
```
La misma conversión, con el alimento que salió de bodega en lugar del registrado en piscina. Sirve para probar si una FCA "buena" es real o es falta de registro.

## 04. Compras

| Medida | Formato |
|---|---|
| Alimento Comprado (kg) | Entero |
| Compras Alimento USD | Moneda, 0 decimales |
| Precio Alimento (USD/kg) | Moneda, 2 decimales |

```dax
Alimento Comprado (kg) = SUM ( fact_Compras[Kg_Comprados] )

Compras Alimento USD = SUM ( fact_Compras[Subtotal_USD] )

Precio Alimento (USD/kg) = DIVIDE ( [Compras Alimento USD], [Alimento Comprado (kg)] )
```

## 01. Producción (agregada)

```dax
Densidad Promedio (m2) =
CALCULATE ( AVERAGE ( fact_Ciclos[Densidad_m2] ), USERELATIONSHIP ( Dim_Calendario[Date], fact_Ciclos[Fecha_Siembra] ) )
```
Por fecha de siembra: la densidad se decide al sembrar.

## 02. Ventas (agregadas)

```dax
Libras Vendidas por Fecha de Cosecha =
CALCULATE ( [Libras Vendidas], USERELATIONSHIP ( Dim_Calendario[Date], fact_Liquidaciones[Fecha_Cosecha] ) )

Precio por Fecha de Cosecha (USD/lb) = DIVIDE ( [Ventas por Fecha de Cosecha], [Libras Vendidas por Fecha de Cosecha] )
```
Para comparar el precio con el ciclo que lo produjo, en el año en que se cosechó.

## 03. Alimentación (agregadas)

```dax
% Alimento por Proveedor =
DIVIDE ( [Alimento Aplicado (kg)], CALCULATE ( [Alimento Aplicado (kg)], REMOVEFILTERS ( dim_Proveedor ) ) )

Costo Alimento de Ciclos (USD) =
SUMX (
    VALUES ( dim_Proveedor[Proveedor] ),
    [Alimento de Ciclos Cosechados (kg)] * CALCULATE ( [Precio Alimento (USD/kg)], REMOVEFILTERS ( Dim_Calendario ) )
)

Costo Alimento por kg Camarón = DIVIDE ( [Costo Alimento de Ciclos (USD)], [Biomasa Cosechada (kg)] )
```
El costo real del alimento: los kilos de cada proveedor por su propio precio. Responde cuánto cuesta en alimento cada kilo de camarón, que es lo que importa al negocio, más que el precio del saco.

**Columna calculada `dim_Ciclo[Grupo Alimento]`**
```dax
Grupo Alimento =
VAR Total = CALCULATE ( SUM ( fact_Alimentacion[Kg_Aplicados] ) )
VAR Aquafeed = CALCULATE ( SUM ( fact_Alimentacion[Kg_Aplicados] ), dim_Proveedor[Proveedor] = "AQUAFEED LITORAL S.A." )
RETURN IF ( DIVIDE ( Aquafeed, Total ) >= 0.5, "Aquafeed", "Otros" )
```
Clasifica cada ciclo por el alimento que recibió. Filtrar dim_Proveedor no sirve para comparar la FCA por proveedor: reduce el alimento a un proveedor pero deja la biomasa completa del ciclo.

## 05. Hallazgos

```dax
Ventas Perdidas por Precio (Aquafeed) =
SUMX (
    VALUES ( Dim_Calendario[Año] ),
    VAR LbA = CALCULATE ( [Libras Vendidas por Fecha de Cosecha], dim_Ciclo[Grupo Alimento] = "Aquafeed" )
    VAR PO = CALCULATE ( [Precio por Fecha de Cosecha (USD/lb)], dim_Ciclo[Grupo Alimento] = "Otros" )
    VAR PA = CALCULATE ( [Precio por Fecha de Cosecha (USD/lb)], dim_Ciclo[Grupo Alimento] = "Aquafeed" )
    RETURN IF ( LbA > 0, LbA * ( PO - PA ) )
)

Sobrecosto Alimento (Aquafeed) =
SUMX (
    VALUES ( Dim_Calendario[Año] ),
    VAR KgA = CALCULATE ( [Biomasa Cosechada (kg)], dim_Ciclo[Grupo Alimento] = "Aquafeed" )
    VAR CA = CALCULATE ( [Costo Alimento por kg Camarón], dim_Ciclo[Grupo Alimento] = "Aquafeed" )
    VAR CO = CALCULATE ( [Costo Alimento por kg Camarón], dim_Ciclo[Grupo Alimento] = "Otros" )
    RETURN IF ( KgA > 0, KgA * ( CA - CO ) )
)

Impacto Estimado Aquafeed (USD) = [Ventas Perdidas por Precio (Aquafeed)] + [Sobrecosto Alimento (Aquafeed)]
```
Año por año, se compara Aquafeed contra los demás proveedores **en el mismo año**, con el mismo mercado.

```dax
Precio Base 2022 (USD/lb) =
CALCULATE ( [Precio Promedio (USD/lb)], REMOVEFILTERS ( Dim_Calendario ), Dim_Calendario[Año] = 2022 )

Precio Solo Mercado (USD/lb) =
VAR Combos = SUMMARIZE ( ALL ( fact_Liquidaciones ), fact_Liquidaciones[Clase], fact_Liquidaciones[Talla] )
VAR X = ADDCOLUMNS ( Combos,
    "S", CALCULATE ( [Libras Vendidas], REMOVEFILTERS ( Dim_Calendario ), Dim_Calendario[Año] = 2022 ),
    "P", [Precio Promedio (USD/lb)] )
VAR Y = FILTER ( X, NOT ISBLANK ( [P] ) && [S] > 0 )
RETURN DIVIDE ( SUMX ( Y, [S] * [P] ), SUMX ( Y, [S] ) )

Precio Solo Mezcla (USD/lb) =
VAR Combos = SUMMARIZE ( ALL ( fact_Liquidaciones ), fact_Liquidaciones[Clase], fact_Liquidaciones[Talla] )
VAR X = ADDCOLUMNS ( Combos,
    "S", [Libras Vendidas],
    "P", CALCULATE ( [Precio Promedio (USD/lb)], REMOVEFILTERS ( Dim_Calendario ), Dim_Calendario[Año] = 2022 ) )
VAR Y = FILTER ( X, NOT ISBLANK ( [P] ) && [S] > 0 )
RETURN DIVIDE ( SUMX ( Y, [S] * [P] ), SUMX ( Y, [S] ) )

Efecto Mezcla (USD/lb) =
VAR Mezcla = [Precio Solo Mezcla (USD/lb)]
RETURN IF ( NOT ISBLANK ( Mezcla ), Mezcla - [Precio Base 2022 (USD/lb)] )

Efecto Mezcla (USD) = SUMX ( VALUES ( Dim_Calendario[Año] ), [Efecto Mezcla (USD/lb)] * [Libras Vendidas] )
```
Separan la caída del precio en dos: **mercado** (las tallas de 2022 pagadas al precio del periodo) y **mezcla** (las tallas del periodo pagadas a precios de 2022). El `IF` del efecto mezcla evita que el miembro en blanco del calendario (los ciclos en curso sin fecha de cosecha) muestre −2,88: un vacío menos un número no da vacío.

## 06. Agua

```dax
% Mediciones OD Crítico =
DIVIDE (
    COUNTROWS ( FILTER ( fact_ParametrosAgua, fact_ParametrosAgua[OD_Critico] = TRUE () ) ),
    COUNTROWS ( FILTER ( fact_ParametrosAgua, NOT ISBLANK ( fact_ParametrosAgua[OD_Critico] ) ) )
)
```
El denominador excluye las mediciones vacías (centinelas convertidos en la Fase 3): no se cuentan como "OD normal".

## 07. Costos y Margen

```dax
Gastos Operativos USD = SUM ( fact_Gastos[Monto_USD] )

Costo Larvas USD =
CALCULATE (
    SUMX ( fact_Ciclos, fact_Ciclos[Larvas_Sembradas_miles] * fact_Ciclos[Costo_Larva_USD_mil] ),
    USERELATIONSHIP ( Dim_Calendario[Date], fact_Ciclos[Fecha_Siembra] )
)

Costo Total USD = [Gastos Operativos USD] + [Compras Alimento USD] + [Costo Larvas USD]

Ventas (meses cerrados) =
CALCULATE ( [Ventas USD],
    KEEPFILTERS ( FILTER ( VALUES ( Dim_Calendario[Inicio Mes] ), CALCULATE ( NOT ISEMPTY ( fact_Gastos ) ) ) ) )
```
`Costo Total (meses cerrados)` y `Libras (meses cerrados)` usan el mismo filtro. Un mes está **cerrado** si tiene gastos registrados: octubre de 2026 tiene ventas pero no gastos (novedad 40), y sin este filtro sería el mes más rentable de la historia. Cuando el cliente cierre el mes, entra solo.

```dax
Margen USD = [Ventas (meses cerrados)] - [Costo Total (meses cerrados)]

Margen % = DIVIDE ( [Margen USD], [Ventas (meses cerrados)] )

Costo por Libra (USD) = DIVIDE ( [Costo Total (meses cerrados)], [Libras (meses cerrados)] )
```

---

## Validación contra los totales del ETL

| Medida (total) | Medida DAX | Control ETL |
|---|---|---|
| Ciclos Sembrados | 547 | 547 ciclos |
| Ciclos Cosechados | 522 | 547 − 25 en curso = 522 liquidados |
| Alimento Aplicado (kg) | 12.909.873,5 | 12.909.874 |
| Alimento Despachado (kg) | 13.153.630 | 13.153.630 |
| Diferencia Bodega vs Piscina | 243.756,5 kg (1,9 %) | 243.757 kg |
| Precio Alimento | 1,10 USD/kg | 1,10 USD/kg |
| % Recibido en Planta | 97–99 % por año | merma de transporte de la Fase 1 |
| Gastos Operativos USD (total) | 17.386.725,67 | total de control de fact_Gastos |
| Compras Alimento USD (total) | 14.489.129 | 13.171.175 kg × 1,10 USD/kg |
| Precio Solo Mercado y Solo Mezcla en 2022 | 2,88 = Precio real | el año base se reproduce a sí mismo |
| Alimento de Ciclos Cosechados | 12.649.424,5 | aplicado − alimento de los 25 en curso |

Pendiente de revisar: en 2021 la biomasa cosechada es 5,8 % mayor que las libras en guía (2,72 M contra 2,57 M). Desde 2022 son parecidas.

## Hallazgos que salieron de las medidas

**La conversión empeoró en toda la camaronera desde 2024.** FCA de 1,26–1,29 (2021–2023) a 1,42–1,44 (2024) y 1,54–1,61 (2025–2026), en los cuatro sectores. Al mismo tiempo el peso de cosecha bajó (de ~26 g a ~23,5 g), los días de cultivo subieron (de 105 a 109) y el precio de venta cayó (de 2,88 a 2,13 USD/lb). Causa por investigar.

**El Sector C parecía el más eficiente, y no lo era.** Con el alimento registrado, el Sector C tenía la mejor FCA (1,31–1,35 contra 1,42–1,60). Con el alimento despachado sale 1,37–1,54, igual que los demás sectores. El alimento que falta en el registro de C (novedad 33) sí parece haber llegado a las piscinas: los camarones convirtieron como en el resto. La hipótesis más consistente es un problema de registro, no de desvío. No prueba la causa.

**Un factor a la vez: era el alimento.** En 2024 cambiaron tres cosas: entró un proveedor nuevo y más barato (Aquafeed, de 0 a 69 % del alimento), se sembró 30 % más denso y las mediciones con oxígeno crítico pasaron de 1 % a 23 %. Comparando ciclos del mismo año que difieren en un solo factor, el proveedor explica la FCA (1,60 contra 1,30); la densidad (1,30 contra 1,29) y el oxígeno (1,62 contra 1,60) casi no la mueven.

**El alimento más barato salió caro.** Aquafeed costaba 0,97 USD/kg, 17 % menos que los otros (los tres de 35 % de proteína), pero hacía falta 25 % más: el alimento por kilo de camarón costó igual o más (1,55–1,57 contra 1,50–1,55 USD). Además, el camarón salió entre 1,1 y 2,4 g más pequeño y se vendió entre 7 y 19 centavos menos por libra. Impacto estimado: **679.678 USD** (2024 a septiembre de 2026), 90 % en ventas. Es una estimación: el cliente debe confirmar cómo se asignó Aquafeed a las piscinas.

**El margen cayó por el precio, no por los costos.** Del 45 % (2022) al 21–22 % (2024–2025). El costo por libra apenas subió (de 1,58 a 1,68 USD); el precio cayó de 2,88 a 2,06. 2021 es año de arranque: gastos desde enero, primera venta en abril.

**Mercado contra mezcla.** 2023 fue 100 % mercado: las mismas tallas, pagadas más baratas. Desde 2024 se suma la mezcla: se vende camarón más pequeño, que resta 18–30 centavos por libra y unos **2,48 M USD** en total. El mercado se está recuperando (precio "solo mercado" de 2,22 a 2,50), pero la mezcla ya explica cerca del 40 % de la caída. Aquafeed es una parte de ese problema, no todo.
