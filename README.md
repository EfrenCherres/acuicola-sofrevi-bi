# Acuícola Sofrevi — De datos sucios a decisiones

> **Caso práctico de Business Intelligence** · ETL en Power Query · Modelado en Power BI · Dashboard ejecutivo
> Autor: **Efren Cherres Bowen, Mgtr.** · ECHB Consulting & Think Analytics
> *"Calidad que se mide. Datos que mejoran."*

![Estado](https://img.shields.io/badge/estado-en%20progreso-orange) ![Power BI](https://img.shields.io/badge/Power%20BI-PBIP-F2C811) ![Power Query](https://img.shields.io/badge/Power%20Query-M-217346)

> ⚠️ **Sobre los datos.** Los datos son simulados. En proyectos con clientes, la información operativa y financiera es confidencial y no se publica; por eso este caso usa una camaronera simulada que reproduce los problemas de una operación real: errores de digitación, cambios de sistema, duplicados, valores centinela y meses sin cerrar. El proceso, las decisiones y la documentación son los mismos que se aplican con un cliente.

## Avance del proyecto

| Fase | Estado | Evidencia |
|---|---|---|
| 0. Preparación del repo y proyecto | ✅ Terminada | [Diario](bitacora/diario_avance.md) |
| 1. Entender el negocio y los datos | ✅ Terminada | [Exploración de datos](bitacora/exploracion_datos.md) · [Log de calidad](bitacora/Log_Calidad.xlsx) |
| 2. Conexión y cuadre de carga | ✅ Terminada | [Bitácora de calidad](bitacora/bitacora_calidad.md) |
| 3. Limpieza (staging) | ✅ Fase 3 terminada: 13 de 13 tablas limpias y validadas (40 novedades en el Log). Siguiente: Fase 4, modelo y relaciones. | [Log de calidad](bitacora/Log_Calidad.xlsx) · [Bitácora de calidad](bitacora/bitacora_calidad.md) · [Diario](bitacora/diario_avance.md) · [Bitácora PDF](bitacora/Bitacora_Acuicola_Sofrevi.pdf) |
| 4. Tablas finales | ✅ **Fase 4 · Modelo:** terminada. Modelo en estrella con 7 dimensiones, 7 tablas de hechos y 26 relaciones validadas con consultas DAX (0 filas huérfanas). | [relaciones y decisiones](modelo/relaciones.md) · [diagrama](modelo/diagrama_modelo.png) | |
| 5. Modelo y medidas DAX | 🔄 **Fase 5 · Medidas DAX:** en curso. 26 medidas de producción, ventas, alimentación y compras, validadas contra los totales del ETL. | [medidas y hallazgos](modelo/medidas_dax.md)| |
| 6. Dashboard | ⬜ Pendiente | |
| 7. Insights | ⬜ Pendiente | |

---

## 1. El problema del negocio

*[Quién es la empresa, a qué se dedica y qué necesita decidir. 3–5 líneas.]*

**Preguntas que el dashboard debe responder:**
1. ¿Cuánto producimos y vendemos, a qué precio, y cómo cambia por sector, piscina y año?
2. ¿Qué tan eficiente es el uso del alimento (conversión alimenticia) y dónde se pierde entre bodega y piscina?
3. ¿Cuánto cuesta producir una libra de camarón y cuál es el margen?
4. ¿Qué condiciones del agua ponen en riesgo los ciclos?


## 2. Los datos de origen

Un libro de Excel (`AcuicolaSofrevi_BD_2021-2026.xlsx`) con 8 hojas:

| Archivo / hoja | Contenido | Filas | Columnas |
|---|---|---|---|
| MAESTRO_PISCINAS | Catálogo de piscinas: sector, área, aireadores, año de construcción, estado | 33 | 6 |
| CICLOS_PRODUCCION | Un ciclo de cultivo por fila: siembra, cosecha, larvas, peso, biomasa, sobrevivencia | 550 | 13 |
| ALIMENTACION_DIARIA | Una ración por piscina, día y turno | 57.480 | 6 |
| KARDEX_BODEGA | Despachos de alimento de bodega a piscina | 56.690 | 9 |
| COMPRAS | Facturas de compra de alimento | 748 | 10 |
| PARAMETROS_AGUA | Mediciones de oxígeno, temperatura, salinidad, pH y transparencia | 24.462 | 9 |
| LIQUIDACIONES_EMPACADORAS | Ventas a empacadoras por liquidación, clase y talla | 3.438 | 11 |
| GASTOS_OPERATIVOS | Gastos mensuales por centro de costo y rubro | 1.863 | 4 |

## 3. Calidad de datos: hallazgos principales

Resumen de la [bitácora de calidad](bitacora/bitacora_calidad.md) (fuente: `Log_Calidad.xlsx`). El detalle día a día está en el [diario de avance](bitacora/diario_avance.md). En total se registraron **40 novedades**; estas son las de mayor impacto.

| # | Hallazgo | Filas afectadas | Decisión |
|---|---|---|---|
| 1 | La biomasa venía en kilos en el sistema nuevo y en libras en el anterior. Sin corregirla, la producción parecía caer a menos de la mitad desde marzo de 2023 | 326 ciclos (60 %) | Unificada en libras, la unidad de venta; validada con larvas × sobrevivencia × peso |
| 2 | Raciones de alimento duplicadas y con signo negativo que se compensaban: el total cuadraba (0,3 %) pero 128.097 kg estaban mal registrados (~141.000 USD) | 343 duplicadas + 226 negativas (1,0 %) | Duplicados eliminados y signo corregido |
| 3 | El 12 % del alimento despachado por bodega al Sector C no aparece como ración desde 2024 (185.687 kg, ~204.000 USD). La conversión alimenticia indica que el alimento sí llegó a las piscinas: falla el registro | 76 % de la diferencia total | Sin cambios en los datos; escalado al cliente |
| 4 | Valores centinela en el agua: OD de 99,9 mg/L, temperatura de 0 °C y pH 14, imposibles en una piscina | 292 (1,2 %) | Convertidos a vacío con reglas físicas; no se estiman |
| 5 | 6 liquidaciones cargadas dos veces (489.902,56 USD de ventas duplicadas) | 38 (1,1 %) | Eliminadas; clave única verificada |
| 6 | Piscina escrita de muchas formas (P-01, PISC-01, P1, p 1…) en varias tablas | Todas las tablas operativas | Una sola regla: conservar los dígitos y escribir P-00 |
| 7 | Gastos con el periodo en dos formatos ("ene-21" y "2023-07") y octubre de 2026 sin gastos (mes no cerrado) | 1.863 / 27 faltantes | Fecha del mes unificada; el costo por libra se calcula solo en meses con gastos |
| 8 | Raciones en 0 kg dentro de ciclos activos, sin explicación en los datos (se descartó el oxígeno bajo como causa) | 1.645 (2,9 %) | Sin cambios; escalado al cliente |

## 4. Proceso ETL (Power Query)

Las consultas están organizadas en grupos y exportadas en [`power-query/`](power-query/):

- **01_Origen**: conexión a los archivos, sin transformar
- **02_Staging**: limpieza y estandarización por tabla
- **03_Final**: tablas listas para el modelo (hechos y dimensiones)
- **QA**: consultas de control (inventario, duplicados, conciliaciones)

## 5. Modelo de datos

![Modelo](img/modelo.png)

*[Esquema estrella: tabla(s) de hechos, dimensiones, granularidad y relaciones.]* Las medidas DAX están documentadas en [`modelo/medidas_dax.md`](modelo/medidas_dax.md).

## 6. Dashboard

![Dashboard](img/dashboard_01.png)

*[Una línea por página: qué muestra y para quién.]*

## 7. Insights y recomendaciones

1. **[Insight]**: [dato que lo respalda] → [recomendación].

## 8. Cómo reproducirlo

1. Clonar el repositorio.
2. Abrir `pbip/[Nombre].pbip` con Power BI Desktop.
3. Cambiar el parámetro `RutaArchivo` a la carpeta local de `data/raw/`.
4. Actualizar.

## Estructura del repositorio

```
├── data/raw/        datos sucios originales
├── power-query/     consultas M exportadas (.pq)
├── pbip/            proyecto Power BI (formato PBIP)
├── bitacora/        bitácora de calidad + diario de avance
├── modelo/          diagrama y medidas DAX
└── img/             capturas del modelo y del dashboard
```

---

📫 [LinkedIn](#) · [TikTok](#) · [Instagram](#)
