# Registro de riesgos · tienda_online

| # | Riesgo | Quién sale afectado | Probabilidad | Impacto | Mitigación | Estado | Responsable |
| --- | --- | --- | --- | --- | --- | --- | --- |
| R1 | Inyección SQL al invocar el procedimiento almacenado de actualización de stock (`ActualizarStockProducto`). | Base de datos e integridad del inventario | M | A | Uso estricto de parámetros tipados de entrada (`IN p_producto_id INT`, `IN p_cantidad INT`) en los procedimientos. | Hecho | Luis Alberto Méndez |
| R2 | Duplicidad de registros o conflictos de unicidad en correos electrónicos de clientes. | Usuarios y plataforma | B | M | Configuración de la restricción `UNIQUE` en la columna `correo` de la tabla `clientes`. | Hecho | Luis Alberto Méndez |
| R3 | Manipulación de precios o existencias negativas mediante parámetros inválidos en las operaciones. | La tienda y los clientes | M | A | Restricciones de validación de tipos y validación lógica en el backend antes de realizar transacciones. | En curso | Luis Alberto Méndez |