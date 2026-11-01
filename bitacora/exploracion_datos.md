## MAESTRO_PISCINAS
- Qué es una fila: una piscina
- Filas: 34
- Columnas (6): Codigo, Sector, Area_ha, Aireadores_HP,	Anio_Construccion, Estado
- Clave: Codigo
- Me llamó la atención:
    - Sector viene escrito de varias formas: A, B, Sector B
    - Area_ha mezcla separador decimal: unos valores con punto y otros con coma (algunos se leen como texto)

## CICLOS_PRODUCCION
- Qué es una fila: un ciclo de producción (una siembra y su cosecha en una piscina)
- Filas: 550
- Columnas (13): ID_Ciclo, Piscina, Fecha_Siembra, Fecha_Cosecha, Laboratorio, Larvas_Sembradas_miles, Costo_Larva_USD_mil, Densidad_m2, Peso_Muestreo_g, Biomasa_Cosechada, Sobrevivencia_pct, Tipo_Cosecha, Observaciones
- Clave: ID_Ciclo
- Me llamó la atención:
	- Piscina con formatos distintos: P-25, P09, p 12; algunos con espacio al final ("P09 ")
	- Fecha_Siembra y Fecha_Cosecha con formatos mezclados; Fecha_Cosecha tiene 4: 23-jun-22, 19/06/2022, 2021-05-11, 2022-08-14 00:00:00
        - Laboratorio: son 3 laboratorios escritos de ~9 formas, p. ej. Lab. Punta Blanca / Larvas Punta Blanca / LARVAS PUNTA BLANCA
        - Larvas_Sembradas_miles, Costo_Larva_USD_mil, Densidad_m2 y Peso_Muestreo_g mezclan punto y coma decimal; 1489,7 en C-2021-020 podría leerse como 14897
	- Sobrevivencia con nulls
	- Tipo de Cosecha escrito de varias formas N, Normal, NORMAL, Emerg., EMERGENCIA
	- Observaciones tiene -, ok, s/n y vacíos

## ALIMENTACION_DIARIA
- Qué es una fila: una aplicación de alimento en una piscina, en un día y un turno
- Filas: 57480
- Columnas (6): Fecha, Piscina, Kg_Aplicados, Proveedor, Turno, Registrado_Por
- Clave: no tiene ID propio; clave compuesta candidata = Fecha + Piscina + Turno
- Me llamó la atención:
	- Piscina con formatos distintos: PISC-30, P30, P-24
	- Kilos aplicados con valores en negativo -720
	- Proveedor: son 3 escritos de ~10 formas pj Nutriacua / NUTRIACUA S.A
	- Turno: son 2, Mañana escrito de dos formas pj Mañana / MAÑANA

## KARDEX BODEGA
- Qué es una fila: una salida de bodega de un producto
- Filas: 56690
- Columnas (9): N_Documento, Fecha, Hora, Tipo_Mov, Codigo_Producto, Piscina, Sacos, Entregado_A,	Registrado_Por
- Clave: N_Documento (verificado: no se repite)
- Me llamó la atención:
	- Fecha trae una hora 00:00:00 sin valor; la hora real está en la columna Hora
        - Tipo_Mov solo tiene salidas: no hay entradas registradas
        - Sacos, Entregado_A y Registrado_Por: revisados, sin hallazgos



 