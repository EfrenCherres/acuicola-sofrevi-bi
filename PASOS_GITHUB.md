# Cómo subir este caso a GitHub (borra este archivo después)

## Opción A: GitHub Desktop (sin comandos)

1. Copia esta plantilla y renombra la carpeta, p. ej. `acuicola-sofrevi-bi`.
2. GitHub Desktop → **File → Add local repository** → elige la carpeta → **create a repository**.
3. Escribe un mensaje de commit (p. ej. `Estructura inicial del caso`) → **Commit to main**.
4. **Publish repository** → desmarca "Keep this code private" cuando quieras que sea público.

## Opción B: terminal

```bash
cd acuicola-sofrevi-bi
git init
git add .
git commit -m "Estructura inicial del caso"
git branch -M main
git remote add origin https://github.com/EfrenCherres/acuicola-sofrevi-bi.git
git push -u origin main
```

## Ritmo de commits sugerido

Un commit al cerrar cada etapa, para que el historial cuente la historia:

```
Estructura inicial del caso
Carga de datos de origen y QA_Inventario
Staging de [TABLA]: N hallazgos documentados
Tablas finales y modelo estrella
Medidas DAX base
Dashboard: página [nombre]
README final con insights
```

## Antes de cada commit

- [ ] ¿Ningún archivo de solucionario en la lista de cambios?
- [ ] ¿Actualicé el diario de avance?
- [ ] ¿Las novedades nuevas de Log_Calidad.xlsx están copiadas en bitacora_calidad.md?
