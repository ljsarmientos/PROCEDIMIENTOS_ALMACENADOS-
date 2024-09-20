# PROCEDIMIENTOS_ALMACENADOS-

#1. Escribe un procedimiento que no tenga ningún parámetro de entrada ni de salida y que
muestre el texto ¡Hola mundo! 
CREATE DEFINER='root'@'localhost' PROCEDURE 'mostarHolaMundo'()
BEGIN
  SELECT '¡Hola mundo!';
END

Call modava.mostrarHolaMundo();
--------------------------------------------------------------------------------------------------
#2. . Escribe un procedimiento que reciba un número real de entrada, que
representa el valor de la nota de un alumno, y muestre un mensaje indicando qué nota ha
obtenido teniendo en cuenta las siguientes condiciones:
[0,5) = Insuficiente
[5,6) = Aprobado
[6, 7) = Bien
[7, 9) = Notable
[9, 10] = Sobresaliente.
En cualquier otro caso la nota no será válida.

DELIMITER //
CREATE PROCEDURE mostrarNota(IN nota DECIMAL(3, 2))
BEGIN
  IF nota >= 0 AND nota < 5 THEN
    SELECT 'Insuficiente';
  ELSEIF nota >= 5 AND nota < 6 THEN
    SELECT 'Aprobado';
  ELSEIF nota >= 6 AND nota < 7 THEN
    SELECT 'Bien';
  ELSEIF nota >= 7 AND nota < 9 THEN
    SELECT 'Notable';
  ELSEIF nota >= 9 AND nota < 10 THEN
    SELECT 'Sobresaliente';
  ELSE
    SELECT 'Nota no valida';
  END IF;
EDN //

Call modava.mostrarNota(5);

--------------------------------------------------------------------------------------------
#3. Escriba un procedimiento llamado cantidadProductos que reciba como
entrada el nombre del tipo de producto y devuelva el número de productos que existen
dentro de esa categoría

SELECT * FROM myfood.productos;

SELECT count(productos.idTipoproducto) FROM productos
  join tipoproductos
  ON productos.idTipoproducto = tipoproductos.idTipoProducto
  WHERE tipoproductos.nombreTipoProducto = "frutas"
  GROUP BY productos.idtipoproducto;
call myfood.cantidadProductos('frutas');

-------------------------------------------------------------------------------------------
#4. Escribe un procedimiento que se llame preciosProductos, que reciba como
parámetro de entrada el nombre del tipo de producto y devuelva como salida tres
parámetros. El precio máximo, el precio mínimo y la media de los productos que existen
en esa categoría.

DELIMITER //
CREATE PROCEDURE preciosProductos1 (IN tipoProducto VARCHAR(50))

BEGIN   
  DECLARE Maximo INT;
  DECLARE Minimo INT;
  DECLARE Media INT;

  SET Maximo = (SELECT max(valor) from inventario WHERE idProducto = tipoproducto);
  SET Minimo = (SELECT min(valor) from inventario WHERE idProducto = tipoproducto);
  SET Media = (SELECT avg(valor) from inventario WHERE idProducto = tipoproducto);

  SELECT Maximo;
  SELECT Minimo;
  SELECT Media;
END;
//
Call myfood.preciosProductos1('4');

--------------------------------------------------------------------------------------------
#5. Realice un procedimiento que se llame funcionIVA que incluya una función
que calcule el total con el incremento del iva.

DELIMITER // 

CREATE PROCEDURE funcionIVA(IN precio DECIMAL(10, 2), OUT totalConIVA DECIMAL(10, 2))
BEGIN
  SET totalConIVA = precio * 1.19; --Asumiendo un IVA del 19%
END //

DELIMITER ;

----------------------------------------------------------------------------------------------------
# 6. Escribe un procedimiento que reciba el nombre de un país como parámetro de
entrada y realice una consulta sobre la tabla sucursal para obtener todas las sucursales
que existen en la tabla de ese país.

DELIMITER //
CREATE PROCEDURE obtenerSursalesPorPais(IN nombrePais(IN nombrePais VARCHAR(50))
BEGIN 
  SLEECT s.nombre
  FROM sucursal s 
  JOIN paises p ON s.pais_id =p.id 
  WHERE p.nombre = nombrePais:
END //
DELIMITER ;

SELECT count(s.nombre)
  FROM sucursal s
  JOIN paises p ON s.pais_id = p.id_pais
    WHERE p.nombre = "Colombia"
  group by p.id_pais;

----------------------------------------------------------------------------------------------------
#7. Una vez creada la tabla se decide añadir una nueva columna a la tabla
llamada edad que será un valor calculado a partir de la columna fecha_nacimiento.
Escriba la sentencia SQL necesaria para modificar la tabla y añadir la nueva columna.

ALTER TABLE personas 
ADD edad INT;

UPDATE personas
SET edad = DATEDIFF(YEAR, fecha_nacimiento, GETDATE());

DELIMITER //

CREATE FUNCTION calcularEdad(fechaNacimiento DATE)
RETURNS INT
DETERMINISTIC
BEGIN
  DECLARE edad INT;

DELIMITER // 
CREATE FUNCTION calcularEdad(fechaNacimiento DATE)
RETURNS INT 
DETERMINISTIC
BEGIN
  DECLARE edad INT;
  SET edad = TIMESTAMPDIFF(YEAR, fechaNacimiento, CURDATE());
  IF(MONTH(CURDATE()) < MONTH(fechaNacimiento)) OR
  (MONTH(CURDATE()) = MONTH(FechaNacimiento) AND DAY(CURDATE()) < DAY(fechaNacimiento)) THEN
      SET edad = edad - 1 ;
    END IF;
    RETURN edad;
END //

DELIMITER ;
SELECT calcularEdad('1990-05-15');

--------------------------------------------------------------------------------------------------------------
#8. Escriba una función llamada calcularEdad que reciba una fecha y devuelva el
número de años que han pasado desde la fecha actual hasta la fecha pasada como:

DELIMITER // 

CREATE FUNCTION calcularEdad(fechaNacimiento DATE)
RETURNS INT
DETERMINISTIC
BEGIN
  DECLARE edad INT;
  SET edad TIMESTAMPDIFF(YEAR, fechaNacimiento, CURDATE ());
  -- Adjust the age if the birhday has not occurred this year
  IF(MONTH(CURDATE()) < MONTH(fechaNacimiento) OR 
    (MONTH(CURDATE()) = MONTH(fechaNacimiento) AND DAT(CURDATE()) < DAY(fechaNacimiento)) THEN
    SET edad = edad - 1;

-----------------------------------------------------------------------------------------------------------------
#9. Escriba un procedimiento que permita calcular la edad de todos los usuarios
que ya existen en la tabla. Para esto será necesario crear un procedimiento llamado
actualizarColumnaEdad que calcule la edad de cada usuario y actualice la tabla. Este
procedimiento hará uso de la función calcularEdad que hemos creado en el paso anterior.

DELIMITER // 

CREATE FUNCTION calcularEdad(fechaNacimiento DATE)
RETURNS INT 
DETERMINISTIC
BEGIN
  DECLARE edad INT;
  SET edad = TOMESTAMPDIFF(YEAR, fechaNacimiento, CURDATE());

  IF(MONTH(CURDATE()) < MONTH(fechaNacimiento)) OR 
    (MONTH(CURDATE()) = MONTH(fechaNacimiento) AND DAY(CURDATE()) < DAY(fechaNacimiento)) THEN
    SET edad = edad - 1;
  EDN IF;

  RETURN edad;
EDN //

DELIMITER ;

SELECT nombre, fecha_nacimiento, calcularedad(fecha_nacimiento) AS edad
FROM personas;

-----------------------------------------------------------------------------------------------------------------------
#10. Escribe un procedimiento almacenado para su proyecto integrador que sea
útil.
Bono a profesional con mas clientes atendidos.

DELIMITER //

CREATE PROCEDURE otorgarBonoProfesionalVariable()
BEGIN
  DECLARE id_profesional INT;
  DECLARE nombre_profesional VARCHAR(100);
  DECLARE clientes_atendidos INT;
  DECLARE bono_total DECIMAL(10, 2);
  --Seleccionar el profesional que ha atendido mas clientes y el número de clientes atendidos 
  SELECT p.id_profesional, p.nombre, COUNT(s.id_clientes) AS total_clientes
  INTO id_profesional, nombre_profesional, clientes_atendidos 
  FROM profesionales p
  JOIN servicios s ON p.id_profesional = s.id_profesional
  GROUP BY p.id_profesional
  ORDER BY total_clientes DESC
  LIMIT 1;

  --Calcular el registro basado en el número de clientes atendidos (por ejemplo, 50 por clientes)
  SET bono_total = clientes_atendidos * 50;

  --Actualizar el registro del profesional para otorgarle el bono
  UPDATE profesionales
  SET bono = bono + bono_total
  WHERE id_profesional = id_profesional 

  --Mostrar el nombre del profesional y el bono recibido
  SELECT CONCAT('El bono de', bono_total, 'ha sido otorgado a: ', nombre_profesional1) AS mensaje;

END //


#PROGRAMA DESCUENTO REALES 

DELIMITER //

CREATE PROCEDURE otorgarDescuentoClientesFielesVariable()
BEGIN 
  DECLARE id_cliente INT;
  DECLARE nombre_clientes VARCHAR(100);
  DECLARE total_visitas INT;
  DECLARE descuento DECIMAL(5, 2);

--Seleccionar los clientes más fieles y el número de visitas que han hecho
SELECT c.id_clientes, c.nombre, COUNT(v.id_visita) AS total_visitas
INTO id_clientes, nombre_cliente, total_visitas 
FROM clientes c
JOIN visitas v ON c.id_clientes = v.id_cliente
GROUP BY c.id_cliente
HAVING total_visitas >= 5; -- Clientes con al menos 5 visitas 

  --Calcular el descuente en funcion del número de visitas (por ejemplo, 2% por cada visita realizada)
  SET descuento = LEAST(total_visitas * 2, 20); -- Descuento máximo del 20%

  --Actualizar el registro del cliente para aplicar el descuento
  UPDATE clientes
  SET descuento = descuento
  WHERE id_cliente = id_cliente;

  --Mostrar el nombre del cliente y el descuento otrogado 
  SELECT CONCAT('Un descuento de' , decuento '% ha sido otorgado a: ', nombre_cliente AS mensaje;
END //

DELIMETER ;

----------------------------------------------------------------------------------------------------------------------------------------
#Conclusión: El uso de procedimientos almacenados en bases de datos no solo
reduce la complejidad del código en las aplicaciones que interactúan con la base de
datos, sino que también mejora el rendimiento y la seguridad al centralizar la lógica de
negocio en el servidor de la base de datos. Esto permite un control más preciso sobre las transacciones y procesos repetitivos, minimizando el riesgo de errores humanos y
garantizando la integridad de los datos.
Este trabajo ha contribuido al desarrollo de habilidades clave en el diseño e
implementación de soluciones eficientes basadas en bases de datos, resaltando el valor
de los procedimientos almacenados como una herramienta fundamental para cualquier
proyecto de software moderno. Así, se fortalece la capacidad de los futuros
desarrolladores para enfrentar los desafíos de la industria del software, aplicando
principios sólidos de programación de bases de datos.
Finalmente, la experiencia adquirida a lo largo de este trabajo refuerza la importancia de la
formación técnica y teórica para resolver problemas reales mediante la aplicación de
tecnologías de bases de datos, consolidando el aprendizaje en la gestión de datos,
automatización de procesos y diseño eficiente de sistemas de información.
 
