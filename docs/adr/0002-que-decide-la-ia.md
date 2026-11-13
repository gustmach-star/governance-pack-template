# 0002. Qué decide la IA

- Estado: Aceptado
- Fecha: 2026-11-13
- Decide: Luis Alberto Méndez Pozo, Desarrollador Full-Stack

## Contexto
En la arquitectura del sistema de comercio electrónico `tienda_online`, es fundamental establecer los límites operativos de la tecnología para proteger la integridad de los datos de los clientes y el inventario.

## Decisión
La Inteligencia Artificial **no toma decisiones autónomas** sobre las transacciones del negocio. Específicamente:
- La IA no modifica stocks, no cancela pedidos ni altera precios de manera automática.
- Toda actualización de inventario se ejecuta mediante lógica determinista en el servidor utilizando procedimientos almacenados (`ActualizarStockProducto`).
- Cualquier cambio o inserción masiva de datos requiere la validación y ejecución manual por parte del administrador del sistema.

## Alternativas descartadas
- Delegar la gestión de inventarios y estados de pedidos a un agente autónomo de IA: descartado por el riesgo de errores lógicos o pérdidas de stock involuntarias.

## Consecuencias
- Cumplimiento estricto del principio de supervisión humana y trazabilidad absoluta de las operaciones de compra-venta.

## Cómo se verifica
- Inspeccionando el código fuente y la base de datos para confirmar que no existen triggers o procesos automáticos de IA que modifiquen tablas transaccionales sin intervención humana.