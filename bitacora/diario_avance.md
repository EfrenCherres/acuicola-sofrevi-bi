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

---
