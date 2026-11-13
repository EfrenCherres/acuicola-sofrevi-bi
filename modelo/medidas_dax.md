# Medidas DAX · Acuícola Sofrevi

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
| Alimento de Ciclos Cosechados | 12.649.424,5 | aplicado − alimento de los 25 en curso |

Pendiente de revisar: en 2021 la biomasa cosechada es 5,8 % mayor que las libras en guía (2,72 M contra 2,57 M). Desde 2022 son parecidas.

## Hallazgos que salieron de las medidas

**La conversión empeoró en toda la camaronera desde 2024.** FCA de 1,26–1,29 (2021–2023) a 1,42–1,44 (2024) y 1,54–1,61 (2025–2026), en los cuatro sectores. Al mismo tiempo el peso de cosecha bajó (de ~26 g a ~23,5 g), los días de cultivo subieron (de 105 a 109) y el precio de venta cayó (de 2,88 a 2,13 USD/lb). Causa por investigar.

**El Sector C parecía el más eficiente, y no lo era.** Con el alimento registrado, el Sector C tenía la mejor FCA (1,31–1,35 contra 1,42–1,60). Con el alimento despachado sale 1,37–1,54, igual que los demás sectores. El alimento que falta en el registro de C (novedad 33) sí parece haber llegado a las piscinas: los camarones convirtieron como en el resto. La hipótesis más consistente es un problema de registro, no de desvío. No prueba la causa.
