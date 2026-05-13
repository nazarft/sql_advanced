## Ventanas SQL (Window Functions)

### Conceptos Clave

```
┌─────────────────────────────────────────────────────────┐
│                    WINDOW FUNCTION                       │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  SELECT column, AGGREGATE(column) OVER( ┌────────┐ )   │
│                                         │ WINDOW │      │
│  ┌──────────────────────────────┐       └────────┘      │
│  │ PARTITION BY column          │   ← Agrupa filas      │
│  │ ORDER BY column              │   ← Define orden      │
│  │ ROWS BETWEEN ... AND ...     │   ← Define rango      │
│  └──────────────────────────────┘                        │
│           ▲                                              │
│           │                                              │
│      Define cómo                                         │
│     procesamos los datos                                 │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

### 📊 Componentes Principales

| Componente | Color | Descripción |
|-----------|-------|------------|
| **Window** | 🟢 Verde | Define **cómo vemos los datos** (partición, orden, rango) |
| **Función** | 🔵 Azul | **Qué aplicamos** a la ventana (SUM, ROW_NUMBER, AVG, etc.) |

---
<img width="1420" height="666" alt="image" src="https://github.com/user-attachments/assets/e770f8c7-c94e-4c0f-8ff5-b0d35f183adf" />


### 💡 Ejemplo Práctico (SIN PARTITION BY)

**Consulta SQL:**
```sql
SELECT
    day,
    sales,
    SUM(sales) OVER(ORDER BY day) AS acumulado
FROM sales;
```

**Resultado:**

| día | ventas | acumulado | Explicación |
|-----|--------|-----------|-------------|
| 1   | 100    | 100       | suma: 100 |
| 2   | 200    | 300       | suma: 100+200 |
| 3   | 50     | 350       | suma: 100+200+50 |

---

### 🎯 Visualización del Proceso

```
Datos originales:
┌─────┬─────────┐
│ día │ ventas  │
├─────┼─────────┤
│  1  │   100   │ ─┐
│  2  │   200   │  ├─ WINDOW (ORDER BY day)
│  3  │    50   │ ─┘
└─────┴─────────┘
          │
          ▼
   SUM(sales) OVER
    (ORDER BY day)
          │
          ▼
┌─────┬─────────┬─────────────┐
│ día │ ventas  │ acumulado   │
├─────┼─────────┼─────────────┤
│  1  │   100   │     100     │
│  2  │   200   │     300     │
│  3  │    50   │     350     │
└─────┴─────────┴─────────────┘
```

---

### 💡 Ejemplo con PARTITION BY

**¿Qué es PARTITION BY?**
`PARTITION BY` divide los datos en **grupos separados** y aplica la ventana **dentro de cada grupo**.

**Imagina esto:**
- Tienes ventas de **3 tiendas** (Tienda A, B, C)
- Quieres sumar las ventas **pero de cada tienda por separado**
- `PARTITION BY tienda` hace exactamente eso

---

### 📋 Datos de Ejemplo

| tienda | día | ventas |
|--------|-----|--------|
| A      | 1   | 100    |
| A      | 2   | 200    |
| B      | 1   | 50     |
| B      | 2   | 150    |
| C      | 1   | 75     |
| C      | 2   | 125    |

---

### 🔧 Consulta SQL con PARTITION BY

```sql
SELECT
    tienda,
    day,
    ventas,
    SUM(ventas) OVER(PARTITION BY tienda ORDER BY day) AS acumulado_por_tienda
FROM sales_by_store;
```

---

### 📊 Resultado

| tienda | día | ventas | acumulado_por_tienda | Explicación |
|--------|-----|--------|----------------------|-------------|
| A      | 1   | 100    | 100                  | **Tienda A:** suma = 100 |
| A      | 2   | 200    | 300                  | **Tienda A:** suma = 100 + 200 |
| B      | 1   | 50     | 50                   | **Tienda B:** suma = 50 (comienza desde 0) |
| B      | 2   | 150    | 200                  | **Tienda B:** suma = 50 + 150 |
| C      | 1   | 75     | 75                   | **Tienda C:** suma = 75 (comienza desde 0) |
| C      | 2   | 125    | 200                  | **Tienda C:** suma = 75 + 125 |

---

### 🎨 Visualización con PARTITION BY

```
PARTITION BY tienda = Dividir en grupos separados:

┌─────────────────────────────┐
│      TIENDA A               │
├─────┬─────────┬─────────────┤
│ día │ ventas  │ acumulado   │
├─────┼─────────┼─────────────┤
│  1  │   100   │     100     │ ─┐
│  2  │   200   │     300     │ ─┤ Ventana solo dentro de A
└─────┴─────────┴─────────────┘ ─┘
        ↓↓↓ Grupo Separado ↓↓↓
┌─────────────────────────────┐
│      TIENDA B               │
├─────┬─────────┬─────────────┤
│ día │ ventas  │ acumulado   │
├─────┼─────────┼─────────────┤
│  1  │    50   │      50     │ ─┐
│  2  │   150   │     200     │ ─┤ Ventana solo dentro de B
└─────┴─────────┴─────────────┘ ─┘
        ↓↓↓ Grupo Separado ↓↓↓
┌─────────────────────────────┐
│      TIENDA C               │
├─────┬─────────┬─────────────┤
│ día │ ventas  │ acumulado   │
├─────┼─────────┼─────────────┤
│  1  │    75   │      75     │ ─┐
│  2  │   125   │     200     │ ─┤ Ventana solo dentro de C
└─────┴─────────┴─────────────┘ ─┘
```

---

### 🔑 Diferencia Clave

| Sin PARTITION BY | Con PARTITION BY |
|-----------------|-----------------|
| `SUM(ventas) OVER(ORDER BY day)` | `SUM(ventas) OVER(PARTITION BY tienda ORDER BY day)` |
| Una sola ventana con TODOS los datos | Una ventana DIFERENTE para cada tienda |
| La suma crece sin límite | La suma se reinicia en cada nueva tienda |
| Resultado: 100, 300, 350, 400, 475, 600 | Resultado: 100, 300, 50, 200, 75, 200 |

---

---

## CTEs (Common Table Expressions)

### 🎯 ¿Qué es un CTE?

Un **CTE (Expresión de Tabla Común)** es una **consulta temporal** que defines una sola vez y puedes reutilizar múltiples veces **dentro de una misma consulta principal**.

**Piénsalo así:**
- Es como crear una **tabla virtual temporal** 
- Vive **solo durante la ejecución** de tu consulta
- Te ayuda a **organizar código complejo** en partes más legibles
- Puedes usarla **una o varias veces** en la consulta principal

---

### 💡 Sintaxis Básica

```sql
WITH nombre_cte AS (
    -- Tu consulta aquí
    SELECT columnas FROM tabla WHERE condición
)
SELECT * FROM nombre_cte;
```

**Componentes:**
- `WITH` → Palabra clave para iniciar un CTE
- `nombre_cte` → Nombre que le das a tu tabla temporal
- `AS (...)` → La consulta que define el CTE
- Después puedes usarlo como si fuera una tabla real

---

### 📊 Ejemplo 1: CTE Simple

**Problema:** Queremos ver las ventas totales por tienda, pero solo mostrar las tiendas con ventas mayores a 300.

**Sin CTE (confuso):**
```sql
SELECT tienda, SUM(ventas) AS total_ventas
FROM sales
GROUP BY tienda
HAVING SUM(ventas) > 300;
```

**Con CTE (más claro):**
```sql
WITH ventas_por_tienda AS (
    SELECT tienda, SUM(ventas) AS total_ventas
    FROM sales
    GROUP BY tienda
)
SELECT * FROM ventas_por_tienda
WHERE total_ventas > 300;
```

**¿Por qué es mejor con CTE?**
- El CTE es claro: "Calcula ventas por tienda"
- El SELECT principal es simple: "Filtra por 300"
- Más fácil de leer y mantener

---

### 📋 Datos y Resultado

**Datos de entrada (tabla `sales`):**

| tienda | día | ventas |
|--------|-----|--------|
| A      | 1   | 100    |
| A      | 2   | 200    |
| A      | 3   | 150    |
| B      | 1   | 50     |
| B      | 2   | 80     |
| C      | 1   | 200    |
| C      | 2   | 150    |
| C      | 3   | 100    |

**Resultado:**

| tienda | total_ventas |
|--------|--------------|
| A      | 450          |
| C      | 450          |

---

### 🔗 Ejemplo 2: Múltiples CTEs (Reutilización)

**Problema:** Queremos combinar información de dos cálculos diferentes.

```sql
WITH ventas_por_tienda AS (
    SELECT tienda, SUM(ventas) AS total_ventas
    FROM sales
    GROUP BY tienda
),
tiendas_altas AS (
    SELECT tienda, total_ventas
    FROM ventas_por_tienda
    WHERE total_ventas > 300
),
tiendas_bajas AS (
    SELECT tienda, total_ventas
    FROM ventas_por_tienda
    WHERE total_ventas <= 300
)
SELECT 'Tiendas Altas' AS categoría, tienda, total_ventas
FROM tiendas_altas
UNION ALL
SELECT 'Tiendas Bajas' AS categoría, tienda, total_ventas
FROM tiendas_bajas;
```

**Resultado:**

| categoría      | tienda | total_ventas |
|----------------|--------|--------------|
| Tiendas Altas  | A      | 450          |
| Tiendas Altas  | C      | 450          |
| Tiendas Bajas  | B      | 130          |

**¿Por qué este enfoque?**
- Definimos `ventas_por_tienda` una sola vez
- Luego la reutilizamos en `tiendas_altas` y `tiendas_bajas`
- El código es más eficiente y legible

---

### 🎨 Visualización del Flujo con CTEs

```
┌─────────────────────────────────────────────┐
│           TABLA ORIGINAL (sales)            │
│  tienda │ día │ ventas                      │
│  ───────┼─────┼────────                     │
│  A      │ 1   │ 100                         │
│  A      │ 2   │ 200                         │
│  ...                                         │
└─────────────────────────────────────────────┘
                    │
                    ▼
         ┌──────────────────────┐
         │ CTE 1: ventas_por    │
         │      tienda          │
         │ ──────────────────── │
         │ tienda │ total_ventas│
         │ ───────┼──────────── │
         │ A      │ 450         │
         │ B      │ 130         │
         │ C      │ 450         │
         └──────────────────────┘
                │        │
         ───────┘        └───────
        │                        │
        ▼                        ▼
┌───────────────┐        ┌───────────────┐
│ CTE 2: Altas  │        │ CTE 3: Bajas  │
│ (>300)        │        │ (<=300)       │
│ ───────────── │        │ ───────────── │
│ A: 450        │        │ B: 130        │
│ C: 450        │        │               │
└───────────────┘        └───────────────┘
        │                        │
        └────────┬───────────────┘
                 │
                 ▼
        ┌──────────────────┐
        │   SELECT FINAL   │
        │ UNION ALL        │
        │ Combina ambos CTEs
        └──────────────────┘
```

---

### 💪 Ejemplo 3: CTE con Window Functions

**Problema:** Calcular el promedio de ventas por tienda Y mostrar cómo se compara cada día con el promedio.

```sql
WITH promedio_tienda AS (
    SELECT 
        tienda,
        dia,
        ventas,
        AVG(ventas) OVER(PARTITION BY tienda) AS promedio_tienda
    FROM sales
)
SELECT 
    tienda,
    dia,
    ventas,
    promedio_tienda,
    ROUND(ventas - promedio_tienda, 2) AS diferencia
FROM promedio_tienda
ORDER BY tienda, dia;
```

**Datos de entrada:**

| tienda | día | ventas |
|--------|-----|--------|
| A      | 1   | 100    |
| A      | 2   | 200    |
| A      | 3   | 150    |
| B      | 1   | 50     |
| B      | 2   | 150    |

**Resultado:**

| tienda | día | ventas | promedio_tienda | diferencia |
|--------|-----|--------|-----------------|------------|
| A      | 1   | 100    | 150.00          | -50.00     |
| A      | 2   | 200    | 150.00          | 50.00      |
| A      | 3   | 150    | 150.00          | 0.00       |
| B      | 1   | 50     | 100.00          | -50.00     |
| B      | 2   | 150    | 100.00          | 50.00      |

---

### 🎯 Ejemplo 4: CTE Recursivo (Avanzado)

**Problema:** Generar una secuencia de números del 1 al 10.

```sql
WITH RECURSIVE numeros AS (
    -- Caso base: comienza en 1
    SELECT 1 AS numero
    UNION ALL
    -- Caso recursivo: suma 1 hasta llegar a 10
    SELECT numero + 1
    FROM numeros
    WHERE numero < 10
)
SELECT * FROM numeros;
```

**Resultado:**

| numero |
|--------|
| 1      |
| 2      |
| 3      |
| 4      |
| 5      |
| 6      |
| 7      |
| 8      |
| 9      |
| 10     |

**¿Cuándo usar CTEs recursivos?**
- Generar secuencias
- Trabajar con datos jerárquicos (árboles, organigramas)
- Buscar rutas en grafos

---

### 📊 Comparativa: Sin CTE vs Con CTE

| Aspecto | Sin CTE | Con CTE |
|--------|---------|---------|
| **Legibilidad** | ❌ Consultas largas y complejas | ✅ Código dividido y claro |
| **Mantenibilidad** | ❌ Difícil de modificar | ✅ Fácil de cambiar partes |
| **Reutilización** | ❌ Código duplicado | ✅ Reutiliza subconsultas |
| **Depuración** | ❌ Difícil identificar errores | ✅ Puedes testear cada CTE |
| **Performance** | ⚠️ Similar o peor | ✅ Generalmente mejor |

---

### 🔑 Mejores Prácticas con CTEs

```sql
-- ✅ BIEN: Nombres descriptivos y estructura clara
WITH empleados_activos AS (
    SELECT id, nombre, departamento
    FROM empleados
    WHERE estado = 'activo'
),
salarios_altos AS (
    SELECT id, nombre, salario
    FROM empleados_activos
    WHERE salario > 5000
)
SELECT * FROM salarios_altos;

-- ❌ MAL: Nombres confusos y poco legibles
WITH cte1 AS (
    SELECT * FROM t1 WHERE x = 1
),
cte2 AS (
    SELECT * FROM cte1 WHERE y > 10
)
SELECT * FROM cte2;
```

**Reglas de oro:**
1. **Usa nombres descriptivos** para cada CTE
2. **Comenta qué hace** cada CTE si es complejo
3. **Ordena de arriba a abajo** (de lo general a lo específico)
4. **Evita CTEs anidadas muy profundas** (máximo 3-4 niveles)
5. **Prueba cada CTE por separado** si es posible

---

### 💡 Resumen Comparativo: Window Functions vs CTEs

| Característica | Window Functions | CTEs |
|----------------|-----------------|------|
| **Propósito** | Análisis sobre ventanas de datos | Reutilizar subconsultas |
| **Operación** | Suma acumulada, ranking, etc. | Filtrado, transformación |
| **Se reutiliza** | ❌ Solo la columna actual | ✅ Toda la subconsulta |
| **Rango de datos** | ✅ Define rangos (ROWS BETWEEN) | ❌ No define rangos |
| **Casos de uso** | Análisis secuencial, rankings | Lógica compleja, múltiples pasos |
