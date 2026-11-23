# Fase 6 · Dashboard

Dashboard en Power BI con tema oscuro, 8 páginas y navegación lateral entre páginas. Cada página responde una pregunta del negocio, y los títulos de los gráficos son la conclusión, no una descripción del gráfico.

| # | Página | Pregunta que responde | Estado |
|---|---|---|---|
| 00 | Portada | ¿De qué trata el caso? | ✅ |
| 01 | Resumen | ¿La camaronera es rentable y dónde se pierde dinero? | ✅ |
| 02 | Producción | ¿Cómo rinden las piscinas y los sectores? | ✅ |
| 03 | Alimentación | ¿Qué tan eficiente es el alimento? | ✅ |
| 04 | Agua | ¿La calidad del agua afecta el resultado? | ⬜ |
| 05 | Ventas | ¿Qué pasa con el precio y las tallas? | ⬜ |
| 06 | Costos | ¿Cuánto cuesta producir una libra? | ⬜ |
| 07 | Calidad de datos | ¿Qué tan confiables son los datos? | ⬜ |

**Diseño**
- Tema: `tema/Sofrevi_tema_oscuro.json`, que define colores, fuentes y deja apagados los fondos y bordes de los visuales.
- Lienzos de fondo: 1280 × 720, con el riel lateral reservado para el navegador de páginas.
- Colores con significado fijo en todo el informe:
  - Azul `#7FA7E0`: referencia o precio.
  - Ámbar `#D99A2B`: costo o dato a resaltar.
  - Coral `#E8705F`: problema (Aquafeed, pérdidas).
  - Verde `#5BBF8E`: margen.

---

## 01 · Resumen ejecutivo

![Resumen ejecutivo](capturas/01_Resumen.png)

### Qué contiene

**Segmentaciones:** Año, Sector y Piscina, en formato desplegable. Filtran las tarjetas y los gráficos.

**Tarjetas KPI (período completo)**

| KPI | Valor | Medida |
|---|---|---|
| Ventas (meses cerrados) | $47,7 M | `Ventas (meses cerrados)`: solo meses con gastos registrados, para comparar ventas y costos del mismo período |
| Margen % | 30,3 % | `Margen %` |
| Costo por libra | $1,67 | `Costo por Libra (USD)` |
| FCA | 1,37 | `FCA`: kg de alimento por kg de camarón, calculado por ciclo cosechado |
| Impacto estimado Aquafeed | $679,7 mil | `Impacto Estimado Aquafeed (USD)` |

**Gráficos**

| # | Título (conclusión) | Tipo | Campos |
|---|---|---|---|
| 1 | El margen cayó de 45 % a 21 %: bajó el precio, no subió el costo | Columnas agrupadas + línea | Año · Precio Promedio (USD/lb), Costo por Libra (USD) · Margen % |
| 2 | Aquafeed convierte peor: FCA 1,60 vs 1,30 | Columnas agrupadas | Año · FCA · Grupo Alimento |
| 3 | Desde 2024, la mezcla también baja el precio | Líneas | Año · Precio real, Precio Solo Mercado, Precio Solo Mezcla |
| 4 | Aquafeed costó $679,7 mil: 90 % en ventas perdidas | Columnas apiladas | Año · Ventas Perdidas por Precio, Sobrecosto Alimento |
| 5 | Hallazgos clave | Cuadro de texto | — |

### Qué se encontró

1. **El precio, no el costo, explica la caída del margen.** El margen pasó de 44,6 % (2022) a 21,3 % (2025). En ese período el precio bajó de $2,88 a $2,13 por libra, mientras el costo apenas cambió (de $1,59 a $1,68).
2. **El alimento barato salió caro.** Desde 2024 parte de las piscinas usa alimento Aquafeed, que es 17 % más barato por kg pero convierte peor: FCA 1,60–1,62 frente a 1,26–1,34 del resto de proveedores. El camarón sale más pequeño y se vende más barato. El impacto estimado es de $679,7 mil, repartido así:

   | Año | Impacto |
   |---|---|
   | 2024 | $249,2 mil |
   | 2025 | $267,1 mil |
   | 2026 (meses cerrados) | $163,4 mil |

   Cerca del 90 % del impacto son ventas perdidas por menor precio; el resto es sobrecosto de alimento.
3. **Desde 2024, la mezcla de tallas también baja el precio.** Para separar los dos efectos, el precio se descompuso con base 2022:
   - **Solo mercado:** cómo habría cambiado el precio si la mezcla de tallas siguiera igual que en 2022.
   - **Solo mezcla:** cómo habría cambiado si el precio de cada talla siguiera igual que en 2022.

   | Año | Caída total vs 2022 | Por mercado | Por mezcla |
   |---|---|---|---|
   | 2023 | −0,46 | −0,46 | 0,00 |
   | 2024 | −0,82 | −0,66 | −0,18 |
   | 2025 | −0,75 | −0,47 | −0,31 |
   | 2026 | −0,64 | −0,38 | −0,28 |

   En 2023 toda la caída fue del mercado. En 2025–2026 la mezcla explica cerca del 40 % de la caída (≈ −$2,48 M acumulado). El mercado se recupera, pero las tallas menores frenan la recuperación. Los dos efectos son una aproximación y no suman exactamente la caída total.

### Decisiones y advertencias

- **2021 es año de arranque:** hubo pocas cosechas y los costos fijos se reparten entre pocas libras. En el gráfico 1 se muestra con nota; en los gráficos 3 y 4 se excluye (el 3 usa 2022 como base y el 4 solo cubre los años con Aquafeed).
- **2026 es un año parcial:** incluye solo los meses cerrados, es decir, con gastos registrados.
- **El cuadro de hallazgos es fijo:** describe el período completo y no cambia con las segmentaciones.
- **Comprobación de cifras:** todos los números del texto se contrastaron con las etiquetas de los gráficos y con las consultas DAX de la Fase 5 (`modelo/medidas_dax.md`).
- **Línea recta en los gráficos de línea:** se usa interpolación lineal, porque la curva suavizada dibuja valores entre años que no existen.

---

## 02 · Producción y ciclos

![Producción y ciclos](capturas/02_Produccion.png)

### Qué contiene

**Tarjetas KPI:** Ciclos cosechados (522) · Biomasa cosechada (20,3 M lb) · Peso promedio (25,0 g) · Sobrevivencia (57,6 %)

| # | Título (conclusión) | Tipo | Campos |
|---|---|---|---|
| 1 | Se cosecha más, pero más pequeño: el peso cayó de 26 g a 23,4 g desde 2024 | Columnas + línea | Año · Biomasa Cosechada (lb) · Peso Promedio (g) |
| 2 | Con Aquafeed el camarón sale 1 a 2 g más pequeño | Columnas agrupadas | Año · Peso Promedio (g) · Grupo Alimento |
| 3 | El Sector B sobrevive menos: 51,9 % frente a ~59 % de los demás | Tabla con barras de datos | Sector · Ciclos, Biomasa, Peso, Sobrevivencia, Días de cultivo |

### Qué se encontró

1. **Se produce más, pero más pequeño.** La biomasa subió de 3,6 M lb (2023) a 3,8 M lb (2025), mientras el peso promedio bajó de 26,0 g a 23,4 g.
2. **Aquafeed explica parte del camarón pequeño, pero no todo.**

   | Año | Aquafeed | Otros | Diferencia |
   |---|---|---|---|
   | 2024 | 23,3 g | 25,6 g | −2,3 g |
   | 2025 | 23,0 g | 24,2 g | −1,2 g |
   | 2026 | 23,3 g | 24,4 g | −1,1 g |

   Desde 2025 el peso también baja con los otros proveedores (de 25,6 a 24,2 g), así que hay otra causa por revisar. Se analiza en la página Agua.
3. **El Sector B sobrevive menos.** Con casi los mismos ciclos que los demás (130–131) y un peso similar, su sobrevivencia es de 51,9 % frente a ~59,3 %, y cosecha entre 15 % y 25 % menos biomasa.

### Decisiones y advertencias

- El eje del gráfico 2 empieza en 20 g para que se vea la diferencia; las etiquetas muestran el valor real.
- La tabla no incluye FCA a propósito: el FCA del Sector C sale mejor por una falla de registro del alimento, y eso se explica en la página Alimentación.
- 2026 es un año en curso, por eso su biomasa es menor.

---

## 03 · Alimentación y conversión

![Alimentación y conversión](capturas/03_Alimentacion.png)

### Qué contiene

**Tarjetas KPI:** FCA registrado (1,37) · FCA con despacho (1,40) · Costo de alimento por kg de camarón ($1,51) · % diferencia bodega vs piscina (1,85 %)

| # | Título (conclusión) | Tipo | Campos |
|---|---|---|---|
| 1 | El Sector C no es más eficiente: su FCA sube de 1,30 a 1,38 con el alimento despachado | Columnas agrupadas | Sector · FCA, FCA con Despacho |
| 2 | Desde 2024 Aquafeed es el principal proveedor: 47 % → 69 % del alimento | Columnas 100 % apiladas | Año · Alimento Aplicado (kg) · Proveedor |
| 3 | Aquafeed es 16 % más barato, pero el camarón cuesta 5 % más en alimento | Columnas agrupadas | Grupo Alimento · Precio Alimento por Grupo, Costo Alimento por kg Camarón |
| 4 | Desde 2024 el Sector C registra 6–12 % menos alimento del que sale de bodega | Matriz con escala de color | Sector × Año · % Diferencia Bodega vs Piscina |

### Qué se encontró

1. **El FCA "bueno" del Sector C es un problema de registro.** Con el alimento anotado en piscina, C tiene FCA 1,30 y parece el sector más eficiente. Con el alimento que salió de bodega sube a 1,38, igual que A, B y D (1,40–1,41).
2. **La brecha del Sector C empezó en 2024.** De 2021 a 2023 su diferencia entre bodega y piscina era de 0,3–0,6 %, como los demás sectores. En 2024 sube a 5,6 %, y en 2025–2026 a 11,9–12,4 %. Los otros sectores se mantienen por debajo de 1 %. Algo cambió ese año en la forma de registrar en C; la pregunta quedó en el Log para el cliente.
3. **Aquafeed pasó a ser el principal proveedor.** No aparece hasta 2023; en 2024 cubre el 47 % del alimento aplicado, en 2025 el 69 % y en 2026 el 64 %.
4. **El alimento barato no ahorró dinero.** En 2024–2026, Aquafeed costó $0,97 por kg frente a $1,16 de los otros proveedores (16 % menos). Pero como convierte peor, cada kg de camarón necesitó $1,56 de alimento frente a $1,49 (5 % más).

### Decisiones y advertencias

- **Medida nueva `Precio Alimento por Grupo (USD/kg)`.** Las compras no están relacionadas con los ciclos, así que la medida original daba el mismo precio a los dos grupos. La nueva toma el precio de compra de Aquafeed o del resto de proveedores, según el grupo que muestra el gráfico.
- **Eje desde 1,0 en el gráfico de FCA.** Las diferencias son de décimas; las etiquetas muestran el valor real.
- **El gráfico de proveedores dibuja proporciones con los kg,** pero la etiqueta muestra `% Alimento por Proveedor`.
- **"Apunta a una falla de registro", no "es".** Los datos no permiten descartar otra causa; se pidió confirmación al cliente.

---

*Datos sintéticos: la camaronera y sus registros son simulados.*
