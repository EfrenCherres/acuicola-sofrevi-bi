# Acuícola Sofrevi — De datos sucios a decisiones

> **Caso práctico de Business Intelligence** · ETL en Power Query · Modelado en Power BI · Dashboard ejecutivo
> Autor: **Efren Cherres Bowen, Mgtr.** · ECHB Consulting & Think Analytics
> *"Calidad que se mide. Datos que mejoran."*

![Estado](https://img.shields.io/badge/estado-en%20progreso-orange) ![Power BI](https://img.shields.io/badge/Power%20BI-PBIP-F2C811) ![Power Query](https://img.shields.io/badge/Power%20Query-M-217346)

> ⚠️ **Datos sintéticos.** La empresa y todos los registros son ficticios, creados para practicar. Cualquier parecido con una empresa real es coincidencia.

## Avance del proyecto

| Fase | Estado | Evidencia |
|---|---|---|
| 0. Preparación del repo y proyecto | ✅ Terminada | [Diario](bitacora/diario_avance.md) |
| 1. Entender el negocio y los datos | ✅ Terminada | [Exploración de datos](bitacora/exploracion_datos.md) · [Log de calidad](bitacora/Log_Calidad.xlsx) |
| 2. Conexión y cuadre de carga | ✅ Terminada | [Bitácora de calidad](bitacora/bitacora_calidad.md) |
| 3. Limpieza (staging) | ✅ Fase 3 terminada: 13 de 13 tablas limpias y validadas (40 novedades en el Log). Siguiente: Fase 4, modelo y relaciones. | [Log de calidad](bitacora/Log_Calidad.xlsx) · [Bitácora de calidad](bitacora/bitacora_calidad.md) · [Diario](bitacora/diario_avance.md) · [Bitácora PDF](bitacora/Bitacora_Acuicola_Sofrevi.pdf) |
| 4. Tablas finales | ⬜ Pendiente | |
| 5. Modelo y medidas DAX | ⬜ Pendiente | |
| 6. Dashboard | ⬜ Pendiente | |
| 7. Insights | ⬜ Pendiente | |

---

## 1. El problema del negocio

*[Quién es la empresa, a qué se dedica y qué necesita decidir. 3–5 líneas.]*

**Preguntas que el dashboard debe responder:**
1. [Pregunta 1]
2. [Pregunta 2]
3. [Pregunta 3]

## 2. Los datos de origen

| Archivo / hoja | Contenido | Filas | Columnas |
|---|---|---|---|
| [tabla] | [qué registra] | [n] | [n] |

## 3. Calidad de datos: hallazgos principales

Resumen de la [bitácora de calidad](bitacora/bitacora_calidad.md) (fuente: [`Log_Calidad.xlsx`](bitacora/Log_Calidad.xlsx)). El detalle día a día está en el [diario de avance](bitacora/diario_avance.md).

| # | Hallazgo | Filas afectadas | Decisión |
|---|---|---|---|
| 1 | [problema] | [n (%)] | [qué se hizo] |

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
