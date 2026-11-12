# Diario de avance

Registro fechado del trabajo: qué hice, qué encontré y **por qué** tomé cada decisión. Una entrada por sesión.

---

## 2026-10-31 · Preparación del repo y proyecto

**Objetivo de hoy:** Dejar listo el repositorio del caso en GitHub y el proyecto de Power BI vacío, para empezar el ETL desde cero con todo versionado.

**Lo que hice:**
- Creé la carpeta del repo en C:\Portafolio
- Copié la plantilla del portafolio y la base sucia AcuicolaSofrevi_BD_2021-2026.xlsx en data/raw
- Creé el repositorio con GitHub Desktop, revisé en History que no se subiera el solucionario y corregí mi nombre de autor antes de publicar
- Publiqué el repositorio como público en GitHub
- Guardé un informe en blanco de Power BI en formato PBIP dentro de la carpeta pbip

**Lo que encontré:**
- El log de calidad tenía fórmulas que apuntaban al archivo del café; se reparó.

**Decisiones y por qué:**
- Repo fuera de OneDrive: OneDrive y Git sincronizan los mismos archivos cada uno por su lado y se pisan; en C:\Portafolio solo Git controla los cambios.
- Repo público desde el inicio: para que se vea el proceso completo, paso a paso, y no solo el resultado final.
- Formato PBIP en vez de .pbix: guarda el modelo y el informe como texto, así GitHub muestra exactamente qué cambió en cada commit; el .pbix es binario y no se puede comparar.

**Pendiente para la próxima sesión:**
- [ ] Fase 1: entender el negocio y los datos (hojas 1 y 2 del Log)

## 2026-10-31 · Exploración de datos (hojas 1 a 4)

**Objetivo de hoy:** Conocer la base antes de abrir Power Query: qué registra cada hoja, qué es una fila, cuál es la clave y qué problemas se ven a simple vista.

**Lo que hice:**
- Creé bitacora/exploracion_datos.md con un bloque por hoja: granularidad, filas, columnas, clave y hallazgos
- Exploré MAESTRO_PISCINAS, CICLOS_PRODUCCION, ALIMENTACION_DIARIA y KARDEX BODEGA
- Revisé cada columna con cuatro preguntas: escritura, formato de fechas, separador decimal y vacíos o relleno
- Hice un commit parcial (4 de 8 hojas) para guardar el avance en GitHub

**Lo que encontré:**
- Piscina viene escrita de varias formas en cada hoja (P-25, P09, p 12, PISC-30), algunas con espacios al final
- Separador decimal mezclado: 1489,7 en el ciclo C-2021-020 podría leerse como 14897
- 3 laboratorios escritos de ~9 formas y 3 proveedores de ~10 formas
- El Kardex solo tiene salidas: sin entradas no se puede calcular stock, solo consumo

**Decisiones y por qué:**
- No modificar el Excel de data/raw: cualquier corrección se hace en Power Query y queda registrada
- Corregí la granularidad de CICLOS de "una piscina" a "un ciclo": la misma piscina aparece en varios ciclos, y esa diferencia define la relación del modelo
- Verifiqué la hipótesis de entradas y salidas en el Kardex con un filtro antes de anotarla

**Pendiente para la próxima sesión:**
- [ ] Explorar las 4 hojas restantes
- [ ] Verificar el conteo de MAESTRO_PISCINAS y revisar Kg_Aplicados y Fecha en ALIMENTACION_DIARIA

## 2026-11-01 · Exploración de datos (hojas 5 a 8) y cierre

**Objetivo de hoy:** Terminar la exploración de las 8 hojas y cerrar los pendientes del día anterior.

**Lo que hice:**
- Exploré COMPRAS_BALANCEADO, PARAMETROS_AGUA, LIQUIDACIONES_EMPACADORAS y GASTOS_OPERATIVOS
- Sumé dos preguntas al método: si los cálculos cuadran y si hay valores imposibles para el negocio
- Comparé libras recibidas contra libras de la guía en cuatro liquidaciones
- Cerré las verificaciones pendientes de MAESTRO_PISCINAS y ALIMENTACION_DIARIA

**Lo que encontré:**
- pH 14 y temperatura de 0 °C en PARAMETROS_AGUA: números válidos para Power BI, imposibles en una piscina de Guayas. El 0 °C es un vacío disfrazado de dato
- Tres liquidaciones llegan a planta al ~99 % de la guía (merma normal); la 23101 llega al 87,8 % y le falta la talla 30/40: posible fila faltante
- MAESTRO_PISCINAS tiene 34 filas pero 33 piscinas: un código duplicado en la tabla que debe ser dimensión
- Kg_Aplicados tiene valores "S/D" mezclados con números
- Los gastos vienen por mes y por sector, no por piscina: el costo por ciclo requerirá repartirlos

**Decisiones y por qué:**
- IVA 0 en compras de balanceado: lo anoté como observación por confirmar, no como error, porque el balanceado tiene tarifa 0 %
- Anoté "revisado, sin hallazgos" en las columnas limpias, para que se vea que se revisaron y no que se saltaron
- Cuando dos conteos no coincidieron (Recuento contra Ctrl + Fin), investigué la diferencia en vez de elegir uno

**Pendiente para la próxima sesión:**
- [ ] Llenar las hojas 1 y 2 de Log_Calidad.xlsx (tablas del modelo e inventario de fuentes)

## 2026-11-01 · Fase 2: conexión en Power Query y cuadre de carga

**Objetivo de hoy:** Conectar el Excel en Power Query sin escribir rutas fijas y comprobar que se cargaron todas las filas.

**Lo que hice:**
- Creé el parámetro RutaArchivo y los grupos 00_Parametros, 01_Origen, 02_Staging, 03_Final y QA
- Conecté el libro con src_SofreviBD y creé una consulta de origen por hoja (8), sin cambiar tipos y sin carga al modelo
- Creé QA_Inventario para comparar las filas cargadas contra el inventario
- Registré el cuadre en la hoja 5. Totales de control del Log

**Lo que encontré:**
- COMPRAS_BALANCEADO tiene 748 filas, no 7.489: error de digitación en el conteo manual
- LIQUIDACIONES_EMPACADORAS tiene 3.438 filas, no 3.300: conté con un filtro activo en Excel, y el Recuento solo cuenta filas visibles
- La hoja se llama KARDEX_BODEGA (con guion bajo); en Power Query el nombre tiene que ser exacto

**Decisiones y por qué:**
- En Origen no se cambian tipos: los datos entran tal como llegaron y se corrigen en Staging, donde quedan documentados
- No corregí las diferencias de QA_Inventario hasta encontrar la causa de cada una

**Pendiente para la próxima sesión:**
- [ ] Fase 3: staging tabla por tabla, con la matriz de controles y el log de novedades

## 2-nov-2026 · Fase 3: dim_Piscina

**Qué hice**
- Creé stg_Piscina (Referencia de src_MaestroPiscinas) y dim_Piscina (03_Final, 33 filas).
- Encontré P-12 duplicado (7,5 y 8,8 ha). Lo resolví con evidencia de otra tabla: en los 17 ciclos de P-12, larvas / (densidad × 10) da entre 7,48 y 7,53 ha. Me quedé con 7,5. El año (2018 o 2011) queda como pregunta para el cliente.
- Limpié Sector (prefijo "Sector ", espacios, mayúsculas) y Area_ha (texto con coma a número, sin depender de la configuración regional).
- Registré 3 novedades en el Log y llené la Matriz con 24 controles.

**Qué aprendí**
- En tipo Any, Reemplazar valores no cambia texto parcial: primero se pone el tipo texto.
- Un cambio de tipo por sí solo no es novedad. El Log es para problemas de los datos.
- Un 0 no siempre es error: P-33 tiene 0 aireadores porque está en construcción y no tiene ciclos.
- Un duplicado se decide con evidencia de otra tabla, no a ojo.

**Siguiente:** stg_Ciclos (fechas como número de serie, IDs C- y SIG-, kg vs lb, formatos de piscina).

## 2-nov (tarde) y 3-nov-2026 · Fase 3: stg_Ciclos

**Qué hice**
- Piscina: 135 formas para 32 piscinas (P-01, P01, PISC-01, p 1). Regla: solo los dígitos y formato P-00. 493 de 550 filas estaban mal escritas; QA de huérfanas vacía.
- Fechas: 4 formatos en la misma columna (número de serie, dd/mm/aaaa, dd-mmm-aa, ISO). Probé que es día/mes (162 contra 0) y creé la función fnTextoAFecha. Una fecha imposible (30/02/2021 en C-2021-013): la estimé con la duración promedio de P-12, la marqué con Cosecha_Estimada y la escalé.
- Tipo_Cosecha: 7 formas a 3. N = NORMAL, probado por la duración del ciclo. EMERGENCIA explica los ciclos cortos y EN CURSO explica los 26 vacíos de cosecha.
- Cantidades: texto con coma decimal en 4 columnas, convertido con la función fnTextoANumero. Dos valores imposibles: larvas 1.150.300 (en unidades) y peso 245 g (sin coma).
- Laboratorio: 10 formas a 3 por palabra clave; 4 vacíos pasan a Desconocido.
- ID_Ciclo: 3 ciclos cargados dos veces (550 filas, 547 distintos). Quité solo las filas idénticas.
- Observaciones: 109 vacíos disfrazados ("-", "s/n", "ok"). Las 7 observaciones reales son mancha blanca y están todas en EMERGENCIA.
- Biomasa: el sistema anterior (C-, hasta marzo de 2023) registraba en libras y el nuevo (SIG-) en kilos. Unificada en libras (Biomasa_lb).
- Log: novedades 4 a 16.

**Qué aprendí**
- Distintos vs. únicos en el perfil: es la forma rápida de ver si una clave tiene duplicados.
- Un filtro mal escrito no da error, da otro número creíble (524 en vez de 57).
- Cuando la evidencia contradice la hipótesis, se busca otra columna que desempate.
- Los problemas más caros no dan error: sin unificar la biomasa, la producción habría parecido caer a la mitad.
- Un dato mal escrito se corrige; uno que nunca se registró se deja vacío y se explica.

**Tiempo:** unas 9 horas en total para CICLOS, la tabla más sucia de la base.

**Siguiente:** Matriz de fact_Ciclos, fact_Ciclos y dim_Laboratorio en 03_Final.

## 4-nov-2026 · Fase 3: fact_Ciclos, dim_Laboratorio, fact_Compras y dim_Proveedor

**Qué hice**
- fact_Ciclos en 03_Final con dos columnas calculadas: Sistema (C o SIG) y Dias_Cultivo.
- dim_Laboratorio desde los ciclos: 4 miembros, incluido Desconocido. QA de huérfanos vacía.
- COMPRAS: proveedor de 13 formas a 3 razones sociales; 4 facturas duplicadas (82.279 USD); Subtotal ≠ Cantidad × Precio por redondeo del precio (no se corrige: el Subtotal es el monto oficial).
- fact_Compras con Kg_Por_Saco, Kg_Comprados y Precio_Kg_USD: Aquafeed vende en saco de 40 kg y es el más barato por kilo, aunque parece el más caro por saco.
- dim_Proveedor: 3 proveedores; QA de huérfanos vacía y probada.
- Gráfico en Power BI para el Reel 17 (producción sin corregir vs corregida), sobre el lienzo ECHB.

**Qué aprendí**
- No todo lo que no cuadra se corrige: hay que saber cuál columna manda.
- El número de factura lo pone el proveedor; la clave es N_Factura + Proveedor.
- Comparar precios por unidad real (kg), no por presentación (saco).
- Un control se prueba en los dos sentidos: que diga OK cuando todo está bien y que avise cuando algo falla.

**Siguiente:** fact_Liquidaciones y dim_Empacadora. Reels 17 y 18 con mi voz.

## 5 de noviembre de 2026

**Tablas trabajadas:** fact_Liquidaciones y dim_Empacadora (limpias y validadas; falta completar el Log).

- Empacadora: 10 formas de escritura reducidas a 3 razones sociales.
- Precio con signo $: se amplió fnTextoANumero y se probó que ciclos y compras siguen sin errores.
- 6 liquidaciones cargadas dos veces (38 filas, 489.902,56 USD): quitadas (3.438 → 3.400).
- Piscina vacía en 22 liquidaciones: inferida por fecha de cosecha y libras, validado 499 de 499; marcada en Piscina_Origen.
- La liquidación 21908 confirmó la fecha de cosecha de C-2021-013 (10/05/2021).
- fact_Liquidaciones enlazada a fact_Ciclos con ID_Ciclo: 522 ciclos cosechados = 522 liquidaciones.
- Controles: 0 liquidaciones antes de la cosecha, precios 0,88–3,96 USD/lb, Total = Lbs × Precio en todas las filas.

**Pendiente:** novedades 22–24 en el Log, matrices, bitácora de calidad y cierre de la tabla.

## 7 de noviembre de 2026

**Tablas cerradas:** fact_Liquidaciones y dim_Empacadora (7 de 13).

- Novedades 20–24 registradas en el Log; matrices de control pegadas.
- La novedad 6 queda Resuelta: la liquidación 21908 confirmó la cosecha de C-2021-013 (10/05/2021).
- Controles finales: 3.400 filas, 0 liquidaciones antes de la cosecha, Total = Lbs × Precio en todas las filas, 522 ciclos = 522 liquidaciones.
- Hallazgo de negocio para el dashboard: Costa Dorada rinde 97,1 % contra 99,0 % de las otras empacadoras.

**Siguiente:** fact_Alimentacion y dim_Sector.

## 8 de noviembre de 2026

**Tablas cerradas:** fact_Alimentacion y dim_Sector (9 de 13).

**fact_Alimentacion (57.480 → 57.137 filas)**
- Reglas reusadas: Piscina (96 formas → 32) y Proveedor (11 → 3) se resolvieron con las reglas de ciclos y compras, pero antes se probaron en esta tabla: 0 piscinas huérfanas y 57.480 filas conservadas.
- Kg_Aplicados tenía cinco problemas distintos; cada uno se trató por separado: "S/D" → vacío, negativos → sin signo (error de digitación), ceros previos a cosecha = ayuno (se quedan), ceros dentro del ciclo = escalados al cliente.
- 343 duplicados exactos quitados; Fecha + Piscina + Turno queda como clave única.
- Evidencia cruzada: 0 raciones con Aquafeed antes de su primera compra (feb-2024).
- Cada ración recibió su ID_Ciclo con una búsqueda por rango (siembra ≤ fecha ≤ cosecha): 100 % válido, 547 ciclos enlazados; QA_Ciclos_Solapados confirmó que ningún ciclo se cruza con otro en la misma piscina.
- Lección: supuse un rango "normal" de 15 a 80 kg mirando solo las primeras filas; la mediana real es 222 kg. Un valor se juzga con la distribución completa, no con una muestra.

**dim_Sector (5 filas)**
- Los gastos se registran por sector (Centro_Costo), no por piscina; por eso el sector necesita su propia dimensión, compartida por gastos y piscinas.
- Clave corta (A–D) igual a stg_Piscina; "Sector A" va solo como nombre para mostrar.
- ADMINISTRACION entra como miembro de tipo "Administrativo", para que el costo total cuadre sin mezclarlo con el costo productivo. El criterio de prorrateo queda como pregunta para el cliente.

**Siguiente:** fact_Kardex y fact_ParametrosAgua.

## 9 y 10 de noviembre de 2026

**Tabla cerrada:** fact_Kardex (10 de 13).

**Regresión detectada y corregida**
- fact_Compras había perdido sus 3 columnas calculadas (Kg_Por_Saco, Kg_Comprados, Precio_Kg_USD), probablemente por un archivo no guardado. Se restauraron y se creó QA_Columnas_Esperadas: revisa que cada tabla de hechos conserve sus columnas calculadas.

**fact_Kardex (56.690 egresos)**
- La tabla más limpia del caso: 0 errores de fecha y hora, 0 sacos inválidos, 0 duplicados, N_Documento único, piscinas ya en formato P-00. La stg solo fija tipos: cada paso responde a un problema que una QA encontró.
- Enriquecimiento en fact: Proveedor y Kg_Por_Saco salen del código del producto (NA-PL35-25 → Nutriacua, 25 kg); Kg_Despachados = sacos × kg; ID_Ciclo por rango de fechas (100 % válido, 547 ciclos).
- Novedad 32: el kardex solo tiene salidas; no se puede calcular el saldo de bodega.

**Cruce bodega contra piscina (hallazgo principal del caso)**
- A nivel diario el cruce no sirve: bodega despacha sacos enteros y lo que sobra se usa al día siguiente. Por eso se midió por ciclo completo.
- Bodega despachó 13.153.630 kg y se registraron 12.909.874 kg como ración: 243.757 kg de diferencia (1,9 %).
- Cuatro supervisores tienen una merma de 0,4–0,6 %. En las piscinas P-17 a P-24, desde 2024, la diferencia es del 12 % cada año (12,11 / 12,30 / 12,02 %): 185.687 kg, el 76 % de toda la diferencia.
- Experimento natural: en las mismas piscinas, entre 2021 y 2023 y con otro supervisor, la diferencia fue de 0,3–0,6 %. La diferencia sigue al supervisor, no a la piscina.
- Novedad 33 (Alta, Escalado, confidencial): no prueba causa ni intención; se entrega al cliente para investigar con guías e inventario físico.

**Siguiente:** fact_ParametrosAgua (y el cruce de los ceros de alimentación con el oxígeno bajo).

## 11 nov 2026 — fact_ParametrosAgua y Dim_Calendario

**ParametrosAgua (tabla 11).** 24,462 mediciones de 32 piscinas, una por fecha y piscina,
cada 2–3 días. Encontré valores centinela: datos vacíos disfrazados de números
(OD 99,9 · Temperatura 0 · pH 14). Los convertí a vacío con reglas físicas,
no por muestra. Los errores son aleatorios entre los 3 técnicos.
Usé esta tabla para probar una hipótesis de la novedad 29 (raciones en 0 por oxígeno
bajo) y quedó descartada: 8,6 % vs 8,8 %.

**Dim_Calendario (tabla 12).** Usé mi plantilla DAX genérica. El calendario empezaba
en 2006, aunque ninguna tabla tenía fechas anteriores a 2021 (lo comprobé con
QA_Rango_Fechas). La causa: la columna Hora de fact_Kardex. Power BI guarda las
horas como 30/12/1899 y CALENDARAUTO las cuenta como fechas.
Solución: rango fijo 2021–2026 (2,191 días) y corte al 20/10/2026.
No cambié la columna Hora, porque es un dato bueno.

**Lección:** cuando un resultado no cuadra y los datos están limpios, revisa cómo
la herramienta lee los datos.

**Siguiente:** fact_Gastos, la última tabla de Power Query.
