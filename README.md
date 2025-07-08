**🚀 Taller de Triggers en MySQL
📌 Objetivo
En este taller aprenderás a usar Triggers en MySQL mediante escenarios prácticos. Implementarás triggers para validaciones, registros automáticos y restricciones.

🔹 Caso 1: Control de Stock de Productos
🧩 Escenario
Una tienda en línea debe asegurarse de que los clientes no puedan comprar más unidades de las disponibles en el inventario. Si lo intentan, la venta debe ser bloqueada.

✅ Tareas
Crear las tablas productos y ventas.
Implementar un trigger BEFORE INSERT que impida ventas donde la cantidad supere el stock.
Probar el trigger.
💻 Implementación SQL
CREATE TABLE productos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50),
    stock INT
);

CREATE TABLE ventas (
    id INT PRIMARY KEY AUTO_INCREMENT,
    id_producto INT,
    cantidad INT,
    FOREIGN KEY (id_producto) REFERENCES productos(id)
);
DELIMITER //

CREATE TRIGGER antes_insertar_venta
BEFORE INSERT ON ventas
FOR EACH ROW
BEGIN
    DECLARE stock_actual INT;

    SELECT stock INTO stock_actual
    FROM productos
    WHERE id = NEW.id_producto;

    IF NEW.cantidad > stock_actual THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Stock insuficiente para completar la venta.';
    END IF;
END;
//

DELIMITER ;
🔍 Prueba
INSERT INTO productos (nombre, stock) VALUES
('Tablet', 5),
('Mouse', 10);

-- ✅ Válido
INSERT INTO ventas (id_producto, cantidad) VALUES (1, 3);

-- ❌ Inválido: excede el stock
INSERT INTO ventas (id_producto, cantidad) VALUES (1, 10);
🔹 Caso 2: Registro Automático de Cambios Salariales
🧩 Escenario
EmpresaSoft desea mantener un registro histórico de todos los cambios salariales de sus empleados.

✅ Tareas
Crear las tablas empleados e historial_sueldos.
Implementar un trigger BEFORE UPDATE que registre cualquier cambio en el salario.
Probar el trigger.
💻 Implementación SQL
CREATE TABLE empleados (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50),
    sueldo DECIMAL(10,2)
);

CREATE TABLE historial_sueldos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    id_empleado INT,
    sueldo_anterior DECIMAL(10,2),
    sueldo_nuevo DECIMAL(10,2),
    fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_empleado) REFERENCES empleados(id)
);
DELIMITER //

CREATE TRIGGER antes_actualizar_sueldo
BEFORE UPDATE ON empleados
FOR EACH ROW
BEGIN
    IF OLD.sueldo <> NEW.sueldo THEN
        INSERT INTO historial_sueldos (id_empleado, sueldo_anterior, sueldo_nuevo)
        VALUES (OLD.id, OLD.sueldo, NEW.sueldo);
    END IF;
END;
//

DELIMITER ;
🔍 Prueba
INSERT INTO empleados (nombre, sueldo) VALUES
('Mateo Díaz', 2500.00),
('Laura Moreno', 3200.00);

UPDATE empleados SET sueldo = 2800.00 WHERE id = 1;

SELECT * FROM historial_sueldos;
🔹 Caso 3: Auditoría de Eliminación de Clientes
🧩 Escenario
DatosSeguros S.A.S. quiere registrar toda eliminación de clientes en una tabla de auditoría para evitar pérdida de información.

✅ Tareas
Crear las tablas clientes y clientes_auditoria.
Implementar un trigger AFTER DELETE que registre los datos eliminados.
Probar el trigger.
💻 Implementación SQL
CREATE TABLE clientes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50),
    correo VARCHAR(50)
);

CREATE TABLE clientes_auditoria (
    id INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT,
    nombre VARCHAR(50),
    correo VARCHAR(50),
    fecha_eliminacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
DELIMITER //

CREATE TRIGGER despues_eliminar_cliente
AFTER DELETE ON clientes
FOR EACH ROW
BEGIN
    INSERT INTO clientes_auditoria (id_cliente, nombre, correo)
    VALUES (OLD.id, OLD.nombre, OLD.correo);
END;
//

DELIMITER ;
🔍 Prueba
INSERT INTO clientes (nombre, correo) VALUES
('Camila Rodríguez', 'camila@correo.com'),
('Pedro Alvarez', 'pedro@correo.com');

DELETE FROM clientes WHERE id = 1;

SELECT * FROM clientes_auditoria;
🔹 Caso 4: Impedir Eliminación de Pedidos Pendientes
🧩 Escenario
En un sistema de ventas, no se deben eliminar pedidos con estado pendiente.

✅ Tareas
Crear la tabla pedidos.
Implementar un trigger BEFORE DELETE que impida la eliminación de pedidos pendientes.
Probar el trigger.
💻 Implementación SQL
CREATE TABLE pedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente VARCHAR(100),
    estado ENUM('pendiente', 'completado')
);
DELIMITER //

CREATE TRIGGER antes_eliminar_pedido
BEFORE DELETE ON pedidos
FOR EACH ROW
BEGIN
    IF OLD.estado = 'pendiente' THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'No se puede eliminar un pedido pendiente.';
    END IF;
END;
//

DELIMITER ;
🔍 Prueba
INSERT INTO pedidos (cliente, estado) VALUES
('Isabel Castro', 'pendiente'),
('Nicolás Rojas', 'completado');

-- ❌ Debería fallar
DELETE FROM pedidos WHERE id = 1;

-- ✅ Debería funcionar
DELETE FROM pedidos WHERE id = 2;
✅ Resumen
Caso	Tipo de Trigger	Evento	Propósito
1	BEFORE INSERT	ventas	Evitar sobreventa
2	BEFORE UPDATE	empleados	Registrar cambios salariales
3	AFTER DELETE	clientes	Auditar clientes eliminados
4	BEFORE DELETE	pedidos	Bloquear eliminación de pedidos pendientes**
