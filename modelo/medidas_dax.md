# Medidas DAX

Agrupadas por carpeta de visualización. Cada medida con su código, qué calcula y un valor de control para validarla.

## [Carpeta: p. ej. Ventas]

### [Nombre de la medida]

```dax
[Medida] =
    -- código
```

- **Qué calcula:** [descripción en lenguaje de negocio]
- **Valor de control:** [cifra esperada y contra qué se validó]

---

## Diagrama del modelo

Guarda la captura de la vista de modelo como `img/modelo.png`.

| Tabla | Tipo | Granularidad | Clave |
|---|---|---|---|
| fact_[...] | Hechos | [una fila por...] | [...] |
| dim_Calendario | Dimensión | un día | Fecha |
