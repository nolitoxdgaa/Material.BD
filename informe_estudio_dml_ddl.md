# Informe de Estudio: DML y DDL en SQL

Este tercer informe se enfoca en ir más allá de solo consultar información. Aprenderemos a **crear estructuras** (DDL) y a **manipular la información** (DML) dentro de nuestra base de datos, basándonos en los conceptos del Laboratorio 04 (Casos Dulcería y Hotel).

---

## 1. DDL: Lenguaje de Definición de Datos (Data Definition Language)

El DDL se encarga de definir o modificar la **estructura** de los objetos de la base de datos (como crear tablas o alterar columnas). No toca los datos en sí, sino el "molde" que los contiene.

### A. Crear Tablas (`CREATE TABLE`)
Se usa para crear una nueva tabla desde cero, especificando el nombre de sus columnas, sus tipos de datos y restricciones (como llaves primarias o foráneas).

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

```sql
-- Agregando una nueva columna llamada 'Comentarios' a la tabla Dulces
ALTER TABLE Dulces 
ADD Comentarios VARCHAR(100);
```

### C. Eliminar Estructuras (`DROP TABLE`)
Destruye por completo una tabla y todos los datos que tenga adentro. **Es una operación irreversible y muy peligrosa en entornos de producción.**

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
