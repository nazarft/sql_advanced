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

### 💡 Ejemplo Práctico

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
