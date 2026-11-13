# 0001. Modelo y proveedor de IA

- Estado: Aceptado
- Fecha: 2026-11-13
- Decide: Luis Alberto Méndez Pozo, Desarrollador Full-Stack

## Contexto
Para el desarrollo, diseño de esquemas relacionales y optimización de consultas SQL del proyecto `tienda_online`, se requiere documentar el uso de herramientas de asistencia basada en Inteligencia Artificial.

## Decisión
Se utiliza el modelo **Claude (Anthropic)** a través de asistentes de desarrollo en el IDE (como Cursor y Copilot) para la estructuración del código backend y la depuración de procedimientos almacenados (`Stored Procedures`). Las llaves de API o credenciales de conexión se gestionan de manera local y nunca se publican en el repositorio.

## Alternativas descartadas
- Conectar agentes de IA directamente a la base de datos de producción con permisos de escritura autónomos: descartado por el riesgo crítico de alterar datos transaccionales sin supervisión.

## Consecuencias
- Agilidad en la creación de esquemas normalizados (1FN, 2FN, 3FN) y validación sintáctica de scripts en MySQL.

## Cómo se verifica
- Revisando el historial de commits y el archivo `.gitignore` para asegurar que no se filtren credenciales ni configuraciones locales.