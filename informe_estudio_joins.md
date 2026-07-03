# Informe de Estudio: JOINS y Vistas en SQL

Este segundo informe se centra en cómo conectar y consultar datos que están repartidos en múltiples tablas utilizando la sentencia `JOIN`, así como en el uso de **Vistas (Views)**, conceptos clave del Laboratorio 03.

Para los ejemplos de este informe, simularemos dos tablas que están relacionadas entre sí: una de `Profesores` y otra de `Cursos`.

**Tabla `Profesores`** (Tabla Izquierda / Tabla 1)
| Id | Nombre | Apellido | Edad |
| :--- | :--- | :--- | :--- |
| 1 | Agustín | Oviedo | 24 |
| 2 | Ana | Gadea | 15 |
| 3 | Mariela | Lima | 20 |
| 4 | Francisco | Chirino | 30 |

**Tabla `Cursos`** (Tabla Derecha / Tabla 2)
| Id_Curso | Nombre_curso | Costo | Id_profesor |
| :--- | :--- | :--- | :--- |
| 1 | Curso de Programación | 1000 | 1 |
| 2 | Curso de Mecánica | 2000 | 2 |
| 3 | Curso de Cocina | 500 | 3 |
| 4 | Curso de Natación | 600 | NULL |

> [!NOTE]
> Observa que el profesor "Francisco" (Id 4) no dicta ningún curso. Por otro lado, el "Curso de Natación" no tiene un profesor asignado (`NULL`).

---

## 1. ¿Qué es un JOIN?

En bases de datos relacionales, la información se divide en múltiples tablas para evitar repetir datos. El propósito del `JOIN` es unir esa información consultando datos de 2 o más tablas a través de una columna que tengan en común (generalmente una Llave Primaria y una Llave Foránea).

Existen 4 tipos principales de `JOIN`:

---

## 2. INNER JOIN (Intersección)

Es el `JOIN` por defecto. Combina cada fila de la primera tabla con cada fila de la segunda tabla **SOLO SI cumplen la condición de enlace**. Si un registro está en una tabla pero no tiene su par en la otra, es descartado.

### Ejemplo y Resultado Simulado
Queremos saber el nombre del profesor y qué curso dicta.
```sql
SELECT Profesores.Nombre, Cursos.Nombre_curso
FROM Profesores 
INNER JOIN Cursos 
ON Profesores.Id = Cursos.Id_profesor;
```

*Resultado Simulado:*
| Nombre | Nombre_curso |
| :--- | :--- |
| Agustín | Curso de Programación |
| Ana | Curso de Mecánica |
| Mariela | Curso de Cocina |

*(Explicación: Francisco no sale porque no tiene curso, y Natación no sale porque no tiene profesor. Solo salen las coincidencias exactas).*

---

## 3. LEFT JOIN (Todo lo de la Izquierda)

Combina los valores de la primera tabla (la que escribes después del `FROM`) con los de la segunda tabla (la del `JOIN`). **Siempre devolverá TODAS las filas de la primera tabla (izquierda)**, incluso si no cumplen la condición. Si no hay coincidencia, pondrá valores `NULL` en las columnas de la segunda tabla.

### Ejemplo y Resultado Simulado
Queremos una lista de TODOS los profesores, indicando su curso si es que dictan alguno.
```sql
SELECT Profesores.Nombre, Cursos.Nombre_curso
FROM Profesores 
LEFT JOIN Cursos 
ON Profesores.Id = Cursos.Id_profesor;
```

*Resultado Simulado:*
| Nombre | Nombre_curso |
| :--- | :--- |
| Agustín | Curso de Programación |
| Ana | Curso de Mecánica |
| Mariela | Curso de Cocina |
| Francisco | NULL |

*(Explicación: Francisco aparece porque el LEFT JOIN obliga a mostrar toda la tabla Profesores).*

---

## 4. RIGHT JOIN (Todo lo de la Derecha)

Funciona exactamente al revés que el LEFT JOIN. **Siempre devolverá TODAS las filas de la segunda tabla (derecha)**, incluso si no cumplen la condición.

### Ejemplo y Resultado Simulado
Queremos una lista de TODOS los cursos, indicando el profesor si es que tienen alguno asignado.
```sql
SELECT Profesores.Nombre, Cursos.Nombre_curso
FROM Profesores 
RIGHT JOIN Cursos 
ON Profesores.Id = Cursos.Id_profesor;
```

*Resultado Simulado:*
| Nombre | Nombre_curso |
| :--- | :--- |
| Agustín | Curso de Programación |
| Ana | Curso de Mecánica |
| Mariela | Curso de Cocina |
| NULL | Curso de Natación |

*(Explicación: El Curso de Natación aparece porque el RIGHT JOIN obliga a mostrar toda la tabla Cursos).*

---

## 5. FULL JOIN (Unión Completa)

Es la combinación del LEFT y el RIGHT JOIN. Devuelve **todas las filas de ambas tablas**, coincidan o no. Rellenará con `NULL` donde falte información.

### Ejemplo y Resultado Simulado
Queremos un reporte total cruzando a todos los profesores y a todos los cursos.
```sql
SELECT Profesores.Nombre, Cursos.Nombre_curso
FROM Profesores 
FULL JOIN Cursos 
ON Profesores.Id = Cursos.Id_profesor;
```

*Resultado Simulado:*
| Nombre | Nombre_curso |
| :--- | :--- |
| Agustín | Curso de Programación |
| Ana | Curso de Mecánica |
| Mariela | Curso de Cocina |
| Francisco | NULL |
| NULL | Curso de Natación |

---

## 6. Creación de Vistas (Views)

Un tema importante cubierto en el Laboratorio 3 es la creación de **Vistas**. Una Vista no es más que **una consulta SQL que se guarda en la base de datos con un nombre**. Actúa como una tabla "virtual". 

Es muy útil cuando tienes un `JOIN` muy grande y complejo que usas constantemente; en lugar de escribir todo el código cada vez, lo guardas como una vista y luego solo haces un `SELECT * FROM MiVista`.

### Ejemplo
Creando una vista para el reporte completo de profesores y sus cursos:
```sql
CREATE VIEW Reporte_Profesores_Cursos AS
SELECT Profesores.Nombre, Profesores.Apellido, Cursos.Nombre_curso, Cursos.Costo
FROM Profesores
INNER JOIN Cursos ON Profesores.Id = Cursos.Id_profesor;
```
Una vez ejecutado eso, puedes consultar la vista como si fuera una tabla normal:
```sql
SELECT * FROM Reporte_Profesores_Cursos;
```

---

## 7. Práctica Final: Ejemplos de Alto Nivel

Aquí combinamos `JOIN` con otros comandos analíticos (`GROUP BY`, `HAVING`, `ORDER BY`, `SUM`).

### Ejemplo 1: Cálculo de Ingresos con JOIN
Queremos saber el costo total de los cursos que dicta cada profesor, pero ordenado desde el que genera más ingresos hasta el que menos. (Uso de alias de tablas `P` y `C` para simplificar).

```sql
SELECT 
    P.Nombre, 
    P.Apellido, 
    SUM(C.Costo) AS Ingreso_Total
FROM Profesores P
INNER JOIN Cursos C ON P.Id = C.Id_profesor
GROUP BY P.Nombre, P.Apellido
ORDER BY Ingreso_Total DESC;
```
*Resultado Simulado:*
| Nombre | Apellido | Ingreso_Total |
| :--- | :--- | :--- |
| Ana | Gadea | 2000 |
| Agustín| Oviedo | 1000 |
| Mariela| Lima | 500 |

### Ejemplo 2: LEFT JOIN + NULL Check (Encontrar los huérfanos)
Queremos saber qué profesores **NO** tienen cursos asignados actualmente. Para lograrlo usamos un `LEFT JOIN` y filtramos buscando los que resultaron con valor `NULL`.

```sql
SELECT P.Nombre, P.Apellido
FROM Profesores P
LEFT JOIN Cursos C ON P.Id = C.Id_profesor
WHERE C.Id_Curso IS NULL;
```
*Resultado Simulado:*
| Nombre | Apellido |
| :--- | :--- |
| Francisco | Chirino |

### Ejemplo 3: JOIN Múltiple con Condicionales (HAVING)
Supongamos que los profesores dieran múltiples cursos. Queremos saber qué profesores están impartiendo cursos cuya suma de costos es mayor a 1000.

```sql
SELECT 
    P.Nombre, 
    COUNT(C.Id_Curso) AS Cantidad_Cursos,
    SUM(C.Costo) AS Suma_Costos
FROM Profesores P
INNER JOIN Cursos C ON P.Id = C.Id_profesor
GROUP BY P.Nombre
HAVING SUM(C.Costo) > 1000;
```
*Resultado Simulado:*
| Nombre | Cantidad_Cursos | Suma_Costos |
| :--- | :--- | :--- |
| Ana | 1 | 2000 |
