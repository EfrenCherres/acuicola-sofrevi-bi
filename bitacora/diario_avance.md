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

-------
