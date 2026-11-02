## MAESTRO_PISCINAS
- Qué es una fila: una piscina
- Filas: 34 (33 piscinas distintas)
- Columnas (6): Codigo, Sector, Area_ha, Aireadores_HP,	Anio_Construccion, Estado
- Clave: Codigo
- Me llamó la atención:
    - Sector viene escrito de varias formas: p. ej A, B, Sector B 4 sectores (A, B, C y D). El D apareció al revisar la columna en Power Query.
    - Area_ha mezcla separador decimal: unos valores con punto y otros con coma (algunos se leen como texto)
    - Codigo duplicado: P-12 aparece 2 veces (idénticas / con datos distintos) → 34 filas, 33 piscinas. La clave de una dimensión debe ser única

## CICLOS_PRODUCCION
- Qué es una fila: un ciclo de producción (una siembra y su cosecha en una piscina)
- Filas: 550
- Columnas (13): ID_Ciclo, Piscina, Fecha_Siembra, Fecha_Cosecha, Laboratorio, Larvas_Sembradas_miles, Costo_Larva_USD_mil, Densidad_m2, Peso_Muestreo_g, Biomasa_Cosechada, Sobrevivencia_pct, Tipo_Cosecha, Observaciones
- Clave: ID_Ciclo
- Me llamó la atención:
	- Piscina con formatos distintos: p. ej P-25, P09, p 12; algunos con espacio al final ("P09 ")
	- Fecha_Siembra y Fecha_Cosecha con formatos mezclados; Fecha_Cosecha tiene 4: 23-jun-22, 19/06/2022, 2021-05-11, 2022-08-14 00:00:00
        - Laboratorio: son 3 laboratorios escritos de ~9 formas, p. ej. Lab. Punta Blanca / Larvas Punta Blanca / LARVAS PUNTA BLANCA
        - Larvas_Sembradas_miles, Costo_Larva_USD_mil, Densidad_m2 y Peso_Muestreo_g mezclan punto y coma decimal; 1489,7 en C-2021-020 podría leerse como 14897
	- Sobrevivencia con nulls
	- Tipo de Cosecha escrito de varias formas p. ej N, Normal, NORMAL, Emerg., EMERGENCIA
	- Observaciones tiene -, ok, s/n y vacíos

## ALIMENTACION_DIARIA
- Qué es una fila: una aplicación de alimento en una piscina, en un día y un turno
- Filas: 57480
- Columnas (6): Fecha, Piscina, Kg_Aplicados, Proveedor, Turno, Registrado_Por
- Clave: no tiene ID propio; clave compuesta candidata = Fecha + Piscina + Turno
- Me llamó la atención:
	- Piscina con formatos distintos: p. ej PISC-30, P30, P-24
	- Kg_Aplicados tiene valores de texto "S/D" (sin dato) mezclados con números
	- Proveedor: son 3 escritos de ~10 formas p. ej Nutriacua / NUTRIACUA S.A
	- Turno: son 2, Mañana escrito de dos formas p. ej Mañana / MAÑANA
	- Fecha: un solo formato, con hora 00:00:00 sin valor

## KARDEX_BODEGA
- Qué es una fila: una salida de bodega de un producto
- Filas: 56690
- Columnas (9): N_Documento, Fecha, Hora, Tipo_Mov, Codigo_Producto, Piscina, Sacos, Entregado_A,	Registrado_Por
- Clave: N_Documento (verificado: no se repite)
- Me llamó la atención:
	- Fecha trae una hora 00:00:00 sin valor; la hora real está en la columna Hora
        - Tipo_Mov solo tiene salidas: no hay entradas registradas
        - Sacos, Entregado_A y Registrado_Por: revisados, sin hallazgos

## COMPRAS_BALANCEADO
- Qué es una fila: una compra de balanceado 
- Filas: 748
- Columnas (10): Fecha_Factura,	N_Factura, Proveedor,	Producto, Presentación,	Cantidad_Sacos,	Precio_Saco_USD,	Subtotal_USD,	IVA_USD, Total_USD
- Clave: N_Factura (verificado: no se repite) 
- Me llamó la atención:
	- Fecha_Factura trae una hora 00:00:00 sin valor
	- Proveedor: son 3 escritos de ~11 formas, p. ej. AQUAFEED / AQUAFEED LITORAL S.A; los mismos 3 proveedores que en ALIMENTACION_DIARIA
	- Producto, Presentación, Cantidad_Sacos, Precio_Saco_USD: revisados, sin hallazgos
 	- Subtotal_USD y Total_USD: difiere el valor en decimas p. ej la columna dice 306.45 y el valor calculado 306.48
	-  IVA_USD: 0 en todas las filas revisadas; coherente con tarifa 0% para balanceado (confirmar con el cliente). Por eso Subtotal_USD = Total_USD

## PARAMETROS_AGUA
- Qué es una fila: una medición de calidad de agua de una piscina en un día
- Filas: 24462
- Columnas (9): Fecha, Piscina,	OD_Madrugada_mgL, OD_Tarde_mgL,	Temperatura_C,	Salinidad_ppt,	pH,	Transparencia_cm, Tecnico
- Clave: no tiene ID propio; clave compuesta candidata = Fecha + Piscina
- Me llamó la atención:
    	- Fecha trae una hora 00:00:00 sin valor
	- OD_Madrugada_mgL, Temperatura_C, y ph tiene valores null
	- Piscina con formato PISC-01, distinto de las demás hojas (P-01 / P01): habrá que unificarlo para relacionar con MAESTRO_PISCINAS
	- pH llega a 14: imposible en agua de cultivo (rango real ~7–9); revisar cuántas filas superan 10
        - Temperatura_C 0 °C es imposible en la zona; probable vacío disfrazado de dato
        - Salinidad_ppt (2,3–28,3) y Transparencia_cm (6–70): rangos plausibles, sin hallazgos
	- Tecnico sin hallazgos
 
## LIQUIDACIONES_EMPACADORAS
- Qué es una fila: una línea de liquidación (una clase y talla dentro de la liquidación de una cosecha)
- Filas: 3438
- Columnas (11): N_Liquidacion, Fecha_Liquidacion, Fecha_Cosecha, Empacadora, Piscina, Lbs_Guia_Remision,	Clase, Talla, Lbs_Recibidas, Precio_USD_lb, Total_USD
- Clave: N_Liquidacion + Clase + Talla
- Conexión con CICLOS: no tiene ID_Ciclo; se une por Piscina + Fecha_Cosecha
- Me llamó la atención:
        - Fecha_Liquidacion y Fecha_Cosecha traen una hora 00:00:00 sin valor
        - Empacadora: 3 empacadoras escritas de ~9 formas, p. ej. GOLFO AZUL / Exportadora Golfo Azul S.A.
        - Piscina con formatos distintos (PISC 18, P-15) y con vacíos: esas liquidaciones no se podrán unir a su ciclo
        - Clase y Talla: revisadas, sin hallazgos (S/T en COLA/DESCARTE es legítimo)
        - Libras recibidas vs guía: en 3 liquidaciones llega ~99 % (merma normal ~1 %); la 23101 llega solo 87,8 % y le falta la talla 30/40 → posible fila faltante. Revisar todas en la Fase 3
	 - Precio_USD_lb mezcla números y texto con $ (p. ej. $2.52)
         - Total_USD = Lbs_Recibidas × Precio_USD_lb: cuadra en las filas revisadas

## GASTOS_OPERATIVOS
- Qué es una fila: el gasto de un rubro, en un centro de costo, en un mes
- Filas: 1863
- Columnas (4): Periodo, Centro_Costo, Rubro, Monto_USD
- Clave: Periodo + Centro_costo + Rubro
- Notas para el modelo:
        - Granularidad mensual (las demás hojas son diarias o por ciclo): se relaciona con el calendario por mes
        - Centro_Costo usa los Sectores de MAESTRO_PISCINAS; el costo por piscina requerirá repartir el gasto del sector (p. ej. por Area_ha). ADMINISTRACION es gasto general
- Me llamó la atención:
        - Periodo trae dos formatos: p. ej. ene-21 y 2023-07
        - Centro_Costo: 4 sectores escritos de ~2 formas, p. ej. A / Sector A, además de ADMINISTRACION
        - Rubro: revisado, sin hallazgos
          - Monto_USD: revisado (negativos, ceros, extremos), sin hallazgos