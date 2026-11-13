# 0003. Dónde viven los datos

- Estado: Aceptado
- Fecha: 2026-11-13
- Decide: Luis Alberto Méndez Pozo, Desarrollador Full-Stack

## Contexto
El sistema `tienda_online` gestiona información sensible de clientes (correos, teléfonos, direcciones) y registros comerciales que deben resguardarse adecuadamente.

## Decisión
- Los datos relacionales del sistema se almacenan en una base de datos **MySQL** local (o instancia segura en servidor cloud).
- Las tablas principales (`clientes`, `productos`, `pedidos`, `proveedores`) mantienen la integridad referencial mediante claves foráneas explícitas (`FOREIGN KEY`).
- Los datos personales de los clientes no se utilizan para entrenar modelos públicos de lenguaje.

## Alternativas descartadas
- Almacenar la información de pedidos y clientes en archivos planos o bases de datos no relacionales: descartado para conservar las restricciones de unicidad y normalización.

## Consecuencias
- Estructura de datos limpia, segura y con relaciones optimizadas para consultas complejas de inventario y facturación.

## Cómo se verifica
- Verificando el script DDL de creación de la base de datos `tienda_online` y las restricciones de integridad definidas en las tablas.