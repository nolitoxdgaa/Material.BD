# Informe de Estudio: Fundamentos de Consultas SQL

Este documento es una guía de estudio diseñada para ayudarte a comprender los conceptos básicos de las consultas en SQL (Structured Query Language). Abarcaremos desde la estructura más fundamental hasta el uso de funciones de cálculo avanzado.

Para que los ejemplos sean más claros, vamos a simular que tenemos la siguiente tabla base llamada `Empleados`:

| ID | Nombre | Apellido | Departamento | Ciudad | Salario |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Ana | Gomez | 01 | Lima | 2500 |
| 2 | Luis | Perez | 02 | Lima | 3000 |
| 3 | Rosa | Torres | 02 | Cuzco | 2100 |
| 4 | Juan | Ruiz | 01 | Arequipa| 1800 |
| 5 | Jose | Silva | 02 | Lima | 3500 |

---

## 1. La Estructura Básica: `SELECT`, `FROM` y Alias (`AS`)

La operación más básica en cualquier base de datos es la extracción de información. Su estructura fundamental consta de dos partes obligatorias:

*   **`SELECT <campos>`:** Indica **qué columnas** queremos recuperar. Si queremos traer todas, usamos un asterisco (`*`).
*   **`FROM <tablas>`:** Indica **de dónde** sacar la información.

**El uso de `AS` (Alias):**
A veces, los nombres de las columnas en la base de datos no son claros o simplemente queremos cambiarlos en el resultado visual por conveniencia. Para esto usamos la palabra reservada `AS` (como), que permite dar un "apodo" temporal a una columna o tabla.

### Ejemplos y Resultados Simulados

**Consulta 1: Seleccionar todo**
```sql
SELECT * FROM Empleados;
```

*Resultado Simulado:*
| ID | Nombre | Apellido | Departamento | Ciudad | Salario |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Ana | Gomez | 01 | Lima | 2500 |
| 2 | Luis | Perez | 02 | Lima | 3000 |
| 3 | Rosa | Torres | 02 | Cuzco | 2100 |
| 4 | Juan | Ruiz | 01 | Arequipa| 1800 |
| 5 | Jose | Silva | 02 | Lima | 3500 |

**Consulta 2: Seleccionar columnas específicas usando `AS`**
```sql
-- Obtenemos el nombre y apellido, pero cambiamos el título de las columnas en el resultado
SELECT 
    Nombre AS Primer_Nombre, 
    Apellido AS Apellido_Paterno 
FROM Empleados;
```

*Resultado Simulado:*
| Primer_Nombre | Apellido_Paterno |
| :--- | :--- |
| Ana | Gomez |
| Luis | Perez |
| Rosa | Torres |
| Juan | Ruiz |
| Jose | Silva |

---

## 2. Filtrado de Datos: La Cláusula `WHERE`

Rara vez necesitamos todos los registros de una tabla. El `WHERE` evalúa cada fila; si la condición es verdadera (`TRUE`), se incluye en el resultado. Puedes usar operadores matemáticos (`=`, `>`, `<`, `>=`, `<=`, `!=`) y lógicos (`AND`, `OR`, `NOT`).

### Ejemplos y Resultados Simulados

**Consulta 1: Filtrar por igualdad**
```sql
-- Empleados que pertenecen al departamento '02'
SELECT Nombre, Apellido, Departamento
FROM Empleados
WHERE Departamento = '02';
```

*Resultado Simulado:*
| Nombre | Apellido | Departamento |
| :--- | :--- | :--- |
| Luis | Perez | 02 |
| Rosa | Torres | 02 |
| Jose | Silva | 02 |

**Consulta 2: Filtros múltiples con `AND`**
```sql
-- Empleados que ganan más de 2000 y pertenecen a 'Lima'
SELECT Nombre, Salario, Ciudad
FROM Empleados
WHERE Salario > 2000 AND Ciudad = 'Lima';
```

*Resultado Simulado:*
| Nombre | Salario | Ciudad |
| :--- | :--- | :--- |
| Ana | 2500 | Lima |
| Luis | 3000 | Lima |
| Jose | 3500 | Lima |

---

## 3. Agrupación de Datos: La Cláusula `GROUP BY`

La cláusula `GROUP BY` se utiliza para agrupar filas que tienen los mismos valores en columnas específicas. Si solo se usa `GROUP BY` sin funciones de cálculo, su comportamiento es similar a eliminar duplicados.

> [!IMPORTANT]
> Regla de oro: Cualquier columna que aparezca en el `SELECT` y que **no** sea parte de una función de agregado, **debe** aparecer obligatoriamente en el `GROUP BY`.

### Ejemplos y Resultados Simulados

**Demostración de la Regla de Oro:**
Si queremos contar cuántos empleados hay considerando tanto su departamento como su ciudad, ambas columnas deben estar en el `GROUP BY`.

```sql
-- CORRECTO: Ambas columnas 'Departamento' y 'Ciudad' están en el SELECT y en el GROUP BY
SELECT Departamento, Ciudad, COUNT(*) AS Total_Empleados
FROM Empleados
GROUP BY Departamento, Ciudad;

/*
INCORRECTO (Daría error en SQL):
SELECT Departamento, Ciudad, COUNT(*) AS Total_Empleados
FROM Empleados
GROUP BY Departamento;
-> Error porque la base de datos no sabría qué ciudad mostrar si el grupo es solo por departamento.
*/
```

*Resultado Simulado (Consulta Correcta):*
| Departamento | Ciudad | Total_Empleados |
| :--- | :--- | :--- |
| 01 | Lima | 1 |
| 02 | Lima | 2 |
| 02 | Cuzco | 1 |
| 01 | Arequipa | 1 |

**Consulta 1: Agrupar para ver valores únicos**
```sql
-- Mostrar qué departamentos existen en la tabla (sin repetir)
SELECT Departamento
FROM Empleados
GROUP BY Departamento;
```

*Resultado Simulado:*
| Departamento |
| :--- |
| 01 |
| 02 |

---

## 4. Funciones de Agregado

Realizan un cálculo sobre un conjunto de valores y devuelven un único valor resumido.

1.  **`COUNT(columna)`:** Cuenta filas donde la columna *no es nula*.
2.  **`COUNT(*)`:** Cuenta el número total de filas, **incluyendo** nulos.
3.  **`SUM(columna)`:** Suma valores numéricos.
4.  **`AVG(columna)`:** Calcula el promedio (Average).
5.  **`MIN(columna)`:** Devuelve el valor mínimo.
6.  **`MAX(columna)`:** Devuelve el valor máximo.

### Ejemplos Individuales por Función (sin `GROUP BY`)

**1. Función `COUNT(columna)`:**
Cuenta cuántos registros tienen un valor (no nulo) en esa columna específica.
```sql
SELECT COUNT(Departamento) AS Empleados_Asignados
FROM Empleados;
```
*Resultado Simulado:*
| Empleados_Asignados |
| :--- |
| 5 |

**2. Función `COUNT(*)`:**
Cuenta el total absoluto de filas.
```sql
SELECT COUNT(*) AS Total_Empleados
FROM Empleados;
```
*Resultado Simulado:*
| Total_Empleados |
| :--- |
| 5 |

**3. Función `SUM(columna)`:**
Suma todos los valores numéricos.
```sql
SELECT SUM(Salario) AS Suma_Salarios
FROM Empleados;
```
*Resultado Simulado:*
| Suma_Salarios |
| :--- |
| 12900 |

**4. Función `AVG(columna)`:**
Calcula el promedio de los valores numéricos.
```sql
SELECT AVG(Salario) AS Promedio_Salarial
FROM Empleados;
```
*Resultado Simulado:*
| Promedio_Salarial |
| :--- |
| 2580 |

**5. Función `MIN(columna)`:**
Encuentra el valor más bajo.
```sql
SELECT MIN(Salario) AS Salario_Minimo
FROM Empleados;
```
*Resultado Simulado:*
| Salario_Minimo |
| :--- |
| 1800 |

**6. Función `MAX(columna)`:**
Encuentra el valor más alto.
```sql
SELECT MAX(Salario) AS Salario_Maximo
FROM Empleados;
```
*Resultado Simulado:*
| Salario_Maximo |
| :--- |
| 3500 |

### Diferencia clave entre `COUNT(columna)` y `COUNT(*)`

Para entender la diferencia, imagina que agregamos un nuevo empleado a nuestra tabla base, pero aún no le hemos asignado un `Departamento` (es decir, su valor en esa columna es `NULL` o nulo). Si ahora tenemos 6 empleados en total, pero solo 5 tienen un departamento asignado:

*   **`COUNT(*)`** devolverá **6**, porque cuenta *todas* las filas de la tabla sin importar si tienen datos nulos.
*   **`COUNT(Departamento)`** devolverá **5**, porque *ignora* las filas donde la columna que indicaste está vacía (nula).

```sql
-- Cuenta absolutamente todas las filas (ej. 6)
SELECT COUNT(*) AS Total_Registros FROM Empleados; 

-- Cuenta solo las filas con datos en esa columna (ej. 5)
SELECT COUNT(Departamento) AS Con_Departamento FROM Empleados;
```

**Combinación Potente: Funciones de agregado con `GROUP BY`**

```sql
-- Conocer el costo total en salarios y la cantidad de empleados por cada departamento
SELECT 
    Departamento, 
    SUM(Salario) AS Costo_Total_Salarios,
    COUNT(*) AS Numero_De_Empleados
FROM Empleados
GROUP BY Departamento;
```

*Resultado Simulado:*
| Departamento | Costo_Total_Salarios | Numero_De_Empleados |
| :--- | :--- | :--- |
| 01 | 4300 | 2 |
| 02 | 8600 | 3 |

*(Explicación del cálculo simulado: En el departamento 01 están Ana(2500) y Juan(1800), sumando 4300. En el departamento 02 están Luis(3000), Rosa(2100) y Jose(3500), sumando 8600).*

---

## 5. Ordenamiento de Datos: La Cláusula `ORDER BY`

La cláusula `ORDER BY` se utiliza para ordenar el conjunto de resultados de una consulta basándose en una o más columnas. Por defecto, el ordenamiento es ascendente (de menor a mayor o de la A a la Z). 

*   `ASC`: Ascendente (es el valor por defecto).
*   `DESC`: Descendente (de mayor a menor o de la Z a la A).

> [!TIP]
> El `ORDER BY` siempre es la **última** instrucción en escribirse en una consulta SQL.

### Ejemplos y Resultados Simulados

**Consulta 1: Ordenamiento Ascendente (por defecto)**
```sql
-- Queremos ver a los empleados ordenados por su salario de menor a mayor
SELECT Nombre, Salario
FROM Empleados
ORDER BY Salario ASC;
```

*Resultado Simulado:*
| Nombre | Salario |
| :--- | :--- |
| Juan | 1800 |
| Rosa | 2100 |
| Ana | 2500 |
| Luis | 3000 |
| Jose | 3500 |

**Consulta 2: Ordenamiento Descendente**
```sql
-- Queremos ver a los empleados ordenados por su salario de mayor a menor
SELECT Nombre, Salario
FROM Empleados
ORDER BY Salario DESC;
```

*Resultado Simulado:*
| Nombre | Salario |
| :--- | :--- |
| Jose | 3500 |
| Luis | 3000 |
| Ana | 2500 |
| Rosa | 2100 |
| Juan | 1800 |

**Consulta 3: Ordenamiento por múltiples columnas**
```sql
-- Ordenamos primero por Departamento (Ascendente) y luego por Salario (Descendente)
SELECT Nombre, Departamento, Salario
FROM Empleados
ORDER BY Departamento ASC, Salario DESC;
```

*Resultado Simulado:*
| Nombre | Departamento | Salario |
| :--- | :--- | :--- |
| Ana | 01 | 2500 |
| Juan | 01 | 1800 |
| Jose | 02 | 3500 |
| Luis | 02 | 3000 |
| Rosa | 02 | 2100 |

---

## 6. Práctica Final: Combinando Todo

Para finalizar esta guía, aquí tienes 5 ejemplos de consultas que combinan `SELECT`, `WHERE`, `GROUP BY`, Funciones de Agregado y `ORDER BY`.

### Ejemplo Integrador 1: Conteo y Ordenamiento Básico
Queremos saber cuántos empleados hay en cada departamento, ordenados desde el departamento que tiene más empleados hasta el que tiene menos.

```sql
SELECT 
    Departamento, 
    COUNT(*) AS Total_Empleados
FROM Empleados
GROUP BY Departamento
ORDER BY Total_Empleados DESC;
```

*Resultado Simulado:*
| Departamento | Total_Empleados |
| :--- | :--- |
| 02 | 3 |
| 01 | 2 |

### Ejemplo Integrador 2: Filtrado Previo y Promedios
Queremos el salario promedio por ciudad, pero **solo** tomando en cuenta a los empleados que ganan más de 2000, ordenado alfabéticamente por el nombre de la ciudad.

```sql
SELECT 
    Ciudad, 
    AVG(Salario) AS Salario_Promedio
FROM Empleados
WHERE Salario > 2000
GROUP BY Ciudad
ORDER BY Ciudad ASC;
```

*Resultado Simulado:*
| Ciudad | Salario_Promedio |
| :--- | :--- |
| Cuzco | 2100 |
| Lima | 3000 |

### Ejemplo Integrador 3: Gastos Máximos y Mínimos en un Departamento Específico
Queremos conocer cuál es el salario más alto y el más bajo dentro del departamento '02', y cambiar el nombre de las columnas resultantes. (En este caso, al ser un solo departamento filtrado, no es estrictamente necesario el `GROUP BY`).

```sql
SELECT 
    MAX(Salario) AS Sueldo_Tope, 
    MIN(Salario) AS Sueldo_Base
FROM Empleados
WHERE Departamento = '02';
```

*Resultado Simulado:*
| Sueldo_Tope | Sueldo_Base |
| :--- | :--- |
| 3500 | 2100 |

### Ejemplo Integrador 4: Agrupación Múltiple con Filtro y Orden
Queremos saber cuánto se gasta en salarios (Suma) separando a la gente por su `Departamento` y también por su `Ciudad`. Solo queremos considerar a los empleados cuyo apellido no sea 'Perez'. Queremos ordenar los resultados por la ciudad y luego por el total gastado de mayor a menor.

```sql
SELECT 
    Departamento, 
    Ciudad, 
    SUM(Salario) AS Gasto_Total
FROM Empleados
WHERE Apellido != 'Perez'
GROUP BY Departamento, Ciudad
ORDER BY Ciudad ASC, Gasto_Total DESC;
```

*Resultado Simulado:*
| Departamento | Ciudad | Gasto_Total |
| :--- | :--- | :--- |
| 01 | Arequipa | 1800 |
| 02 | Cuzco | 2100 |
| 02 | Lima | 3500 |
| 01 | Lima | 2500 |

### Ejemplo Integrador 5: El "Todo en Uno"
Queremos un reporte que muestre el nombre del departamento, la cantidad de empleados válidos (que tengan ciudad registrada), y el salario promedio. Solo evaluaremos a aquellos con salario mayor o igual a 2000. Finalmente, ordenaremos por el salario promedio de mayor a menor.

```sql
SELECT 
    Departamento, 
    COUNT(Ciudad) AS Empleados_Validos, 
    AVG(Salario) AS Promedio_Ganancia
FROM Empleados
WHERE Salario >= 2000
GROUP BY Departamento
ORDER BY Promedio_Ganancia DESC;
```

*Resultado Simulado:*
| Departamento | Empleados_Validos | Promedio_Ganancia |
| :--- | :--- | :--- |
| 02 | 3 | 2866.67 |
| 01 | 1 | 2500 |
