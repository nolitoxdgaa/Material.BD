# Informe de Estudio: DML y DDL en SQL

Este tercer informe se enfoca en ir más allá de solo consultar información. Aprenderemos a **crear estructuras** (DDL) y a **manipular la información** (DML) dentro de nuestra base de datos, basándonos en los conceptos del Laboratorio 04 (Casos Dulcería y Hotel).

---

## 1. DDL: Lenguaje de Definición de Datos (Data Definition Language)

El DDL se encarga de definir o modificar la **estructura** de los objetos de la base de datos (como crear tablas o alterar columnas). No toca los datos en sí, sino el "molde" que los contiene.

### A. Crear Tablas (`CREATE TABLE`)
Se usa para crear una nueva tabla desde cero, especificando el nombre de sus columnas, sus tipos de datos y restricciones (como llaves primarias o foráneas).

**Estructura Base (Esqueleto):**
```sql
CREATE TABLE nombre_tabla (
    columna1 tipo_dato,
    columna2 tipo_dato,
    ...
);
```

```sql
-- Creando la tabla de Marcas
CREATE TABLE Marcas (
    Clave INT PRIMARY KEY,
    Nombre VARCHAR(45)
);

-- Creando la tabla de Dulces y relacionándola con Marcas
CREATE TABLE Dulces (
    Clave INT PRIMARY KEY,
    Nombre VARCHAR(45),
    Descripcion VARCHAR(45),
    Unidades VARCHAR(45),
    Precio_venta DECIMAL(10,2),
    Marca INT,
    FOREIGN KEY (Marca) REFERENCES Marcas(Clave)
);
```

### B. Modificar Estructuras (`ALTER TABLE`)
Se utiliza cuando la tabla ya existe y necesitas cambiar su estructura (por ejemplo, agregar una columna nueva o cambiar el tipo de dato).

**Estructura Base (Esqueleto para agregar columna):**
```sql
ALTER TABLE nombre_tabla 
ADD nombre_columna tipo_dato;
```

```sql
-- Agregando una nueva columna llamada 'Comentarios' a la tabla Dulces
ALTER TABLE Dulces 
ADD Comentarios VARCHAR(100);
```

### C. Eliminar Estructuras (`DROP TABLE`)
Destruye por completo una tabla y todos los datos que tenga adentro. **Es una operación irreversible y muy peligrosa en entornos de producción.**

**Estructura Base (Esqueleto):**
```sql
DROP TABLE nombre_tabla;
```

```sql
-- Borrando una tabla llamada 'Dotacion' que ya no nos sirve
DROP TABLE Dotacion;
```

---

## 2. DML: Lenguaje de Manipulación de Datos (Data Manipulation Language)

El DML es lo que usamos en el día a día para gestionar el **contenido** (los registros) dentro de las tablas que ya creamos.

> [!WARNING]
> Ten mucho cuidado al usar `UPDATE` o `DELETE`. **SIEMPRE** debes usar la cláusula `WHERE` para especificar qué registro afectar. Si la omites, ¡modificarás o borrarás TODOS los datos de la tabla accidentalmente!

### A. Insertar Datos (`INSERT INTO`)
Sirve para agregar nuevas filas de información a una tabla.

**Estructura Base (Esqueleto):**
```sql
INSERT INTO nombre_tabla (columna1, columna2, ...)
VALUES (valor1, valor2, ...);
```

```sql
-- Insertando registros múltiples en la tabla Marcas
INSERT INTO Marcas (Clave, Nombre) 
VALUES (1, 'De la rosa'), (2, 'Ricolino'), (3, 'Adams');

-- Insertando un registro en la tabla Dulces
INSERT INTO Dulces (Clave, Nombre, Descripcion, Unidades, Precio_venta, Marca)
VALUES (4001, 'Gussi', 'Galleta rellena de chocolate', 'Piezas', 2.50, 1);
```

### B. Actualizar Datos (`UPDATE`)
Permite modificar información que ya existe en una o varias filas.

**Estructura Base (Esqueleto):**
```sql
UPDATE nombre_tabla
SET columna = nuevo_valor
WHERE condicion;
```

**Ejemplo y Explicación:**
Imaginemos que el dulce "Lunetas" (Clave 4003) tenía como descripción "Chocolate confitado" y queremos cambiarla a "Dulce macizo", tal como lo solicita el laboratorio 4.

```sql
-- Cambiando la descripción del dulce específico
UPDATE Dulces
SET Descripcion = 'Dulce macizo'
WHERE Clave = 4003;
```
*(Explicación: La base de datos busca el registro cuya Clave sea exactamente 4003, y **solo a ese** le sobrescribe el valor de la columna Descripción).*

### C. Eliminar Datos (`DELETE`)
Borra filas completas de la tabla según la condición dada en el WHERE.

**Estructura Base (Esqueleto):**
```sql
DELETE FROM nombre_tabla
WHERE condicion;
```

**Ejemplo y Explicación:**
Queremos eliminar de nuestra base de datos todos los dulces que pertenezcan a la marca con clave '002' (Ricolino).

```sql
-- Borrando los dulces de cierta marca
DELETE FROM Dulces
WHERE Marca = 2;
```
*(Explicación: Cualquier fila en la tabla `Dulces` donde la columna `Marca` valga 2 será eliminada de forma permanente).*

---

## 3. Práctica Final: Casos Combinados (DML de Alto Nivel)

A veces, para modificar o borrar datos, necesitas usar lógica un poco más avanzada. Aquí te muestro ejemplos útiles:

### Caso 1: Actualizar con una operación matemática
Queremos aumentarle `1.00` al precio de todos los dulces cuyo formato de `Unidades` sea 'Piezas'.

```sql
UPDATE Dulces
SET Precio_venta = Precio_venta + 1.00
WHERE Unidades = 'Piezas';
```

### Caso 2: Insertar datos copiándolos de otra consulta
Podemos usar un `SELECT` dentro de un `INSERT` para poblar una nueva tabla de forma masiva. Supongamos que creamos una tabla de "Dulces Caros" y queremos pasar ahí los de precio mayor a 5.00.

```sql
INSERT INTO Dulces_Caros (Clave, Nombre, Precio)
SELECT Clave, Nombre, Precio_venta 
FROM Dulces 
WHERE Precio_venta > 5.00;
```

### Caso 3: Borrar registros validando con una Subconsulta
A veces no sabemos el ID a borrar, pero sí el nombre. Queremos borrar los dulces de la marca 'Adams'. Podemos usar un `SELECT` (subconsulta) dentro del `DELETE` para encontrar su clave dinámicamente.

```sql
DELETE FROM Dulces
WHERE Marca = (
    SELECT Clave FROM Marcas WHERE Nombre = 'Adams'
);
```
*(Explicación: Primero la base de datos resuelve el SELECT interno y encuentra que la clave de 'Adams' es el 3. Luego, la consulta principal se convierte en `DELETE FROM Dulces WHERE Marca = 3` y procede a borrar los dulces correspondientes).*

---

## 4. Solucionario: Laboratorio 04

A continuación, se presentan las sentencias SQL completas para resolver los dos ejercicios prácticos planteados en el Laboratorio 04.

### Ejercicio 1: Base de Datos "Dulcería"

**a) Mostrar el nombre y las unidades de los dulces cuyo precio de venta sea mayor a 2.**
```sql
SELECT Nombre, Unidades 
FROM Dulces 
WHERE Precio_venta > 2.00;
```

**b) Mostrar el nombre de los dulces y el nombre de las marcas cuya unidad sea piezas.**
```sql
SELECT D.Nombre AS Nombre_Dulce, M.Nombre AS Nombre_Marca
FROM Dulces D
INNER JOIN Marcas M ON D.Marca = M.Clave
WHERE D.Unidades = 'Piezas';
```

**c) Mostrar cuántos dulces hay por cada marca ordenándolos de forma descendente por marca.**
```sql
SELECT Marca, COUNT(*) AS Cantidad_Dulces
FROM Dulces
GROUP BY Marca
ORDER BY Marca DESC;
```

**d) Mostrar cuál es el dulce con menor precio.**
```sql
SELECT Nombre, Precio_venta
FROM Dulces
ORDER BY Precio_venta ASC
LIMIT 1;
-- (Alternativa con Subconsulta):
-- SELECT Nombre, Precio_venta FROM Dulces WHERE Precio_venta = (SELECT MIN(Precio_venta) FROM Dulces);
```

**e) Insertar un registro en cada tabla (marcas, dulces).**
```sql
INSERT INTO Marcas (Clave, Nombre) VALUES (4, 'Sonrics');
INSERT INTO Dulces (Clave, Nombre, Descripcion, Unidades, Precio_venta, Marca) 
VALUES (4004, 'Gudu Pop', 'Paleta masticable', 'Piezas', 1.50, 4);
```

**f) Cambiar la descripción de lunetas por dulce macizo.**
```sql
UPDATE Dulces 
SET Descripcion = 'Dulce macizo' 
WHERE Nombre = 'lunetas';
```

**g) Borrar los dulces con clave 002 (asumiendo que se refiere a la marca 002).**
```sql
DELETE FROM Dulces WHERE Marca = 2;
```

**h) Eliminar las unidades con clave 001 (asumiendo que se refiere a la marca 001).**
```sql
DELETE FROM Marcas WHERE Clave = 1; 
-- Nota: Para que esto funcione, primero deben borrarse los dulces asociados o tener eliminación en cascada.
```

### Ejercicio 2: Base de Datos "Hotel"

**1. Crear la BD.**
```sql
CREATE DATABASE Hotel;
USE Hotel;
```

**2. Crear las tablas y relacionarlas a través del cliente.**
```sql
CREATE TABLE Clientes (
    Id INT PRIMARY KEY,
    Nombre VARCHAR(50),
    A_paterno VARCHAR(50),
    A_materno VARCHAR(50),
    F_nacimiento DATE
);

CREATE TABLE Reservaciones (
    Clave INT PRIMARY KEY,
    Num_habitacion INT,
    Cliente INT,
    F_ingreso DATE,
    F_egreso DATE,
    Promocion VARCHAR(10),
    FOREIGN KEY (Cliente) REFERENCES Clientes(Id)
);
```

**3. Ingresar los registros.**
```sql
INSERT INTO Clientes (Id, Nombre, A_paterno, A_materno, F_nacimiento) VALUES 
(10001, 'Esteban', 'Pérez', 'Iturralde', '1980-10-10'),
(10456, 'Israel', 'Méndez', 'Arteaga', '1979-11-09'),
(10845, 'Juan', 'Navarrete', 'Ortiz', '1970-03-05');

INSERT INTO Reservaciones (Clave, Num_habitacion, Cliente, F_ingreso, F_egreso, Promocion) VALUES 
(90001, 2034, 10001, '2014-05-16', '2014-05-18', 'TI001'),
(90002, 2020, 10456, '2014-05-16', '2014-05-20', 'SI001'),
(90003, 1018, 10379, '2014-05-23', '2014-05-25', 'TI003'),
(90004, 1029, 10986, '2014-05-24', '2014-05-27', 'TI002'),
(90005, 4035, 10845, '2014-05-24', '2014-05-29', 'SI001');
```

**4. Borrar la reservación del cliente 10986.**
```sql
DELETE FROM Reservaciones WHERE Cliente = 10986;
```

**5. Cambiar la fecha de egreso de la reservación 90005 por 31/05/2014.**
```sql
UPDATE Reservaciones 
SET F_egreso = '2014-05-31' 
WHERE Clave = 90005;
```

**6. A través de código agregar una columna que se llame comentarios a la tabla Reservaciones.**
```sql
ALTER TABLE Reservaciones 
ADD Comentarios VARCHAR(255);
```

**7. A través de código crear una tabla que se llame dotación (Clave, Descripción).**
```sql
CREATE TABLE Dotacion (
    Clave INT PRIMARY KEY,
    Descripcion VARCHAR(100)
);
```

**8. A través de código borrar la tabla dotación.**
```sql
DROP TABLE Dotacion;
```
