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
