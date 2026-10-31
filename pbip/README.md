# Proyecto Power BI (PBIP)

Guarda el archivo aquí con **Archivo → Guardar como → Proyecto de Power BI (*.pbip)**.

Power BI crea:

```
[Nombre].pbip
[Nombre].SemanticModel/   ← modelo, consultas M y medidas DAX (texto TMDL)
[Nombre].Report/          ← páginas y visuales
```

**Por qué PBIP y no .pbix:** el .pbix es binario y GitHub no puede mostrar qué cambió entre versiones. Con PBIP cada commit muestra exactamente qué consulta, relación o medida se modificó.

> Si la opción no aparece: Archivo → Opciones → Características de versión preliminar → activar **Guardar como proyecto de Power BI**. Activa también **Formato TMDL**.
