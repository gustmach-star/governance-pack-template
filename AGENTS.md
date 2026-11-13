# AGENTS.md — Almacéntrico

Reglas que cualquier agente de código (Claude Code, Codex, Copilot, Cursor, Antigravity) debe leer antes de tocar este repositorio. Si una regla no está aquí, el agente no la conoce.

## Qué es este proyecto

Almacéntrico es una plataforma web integral de gestión de inventarios, control de kardex y cadena de suministro para distribuidoras comerciales, con un centro de agentes de inteligencia artificial para el análisis predictivo de abastecimiento y recomendaciones de compra.

## Antes de escribir código

- Leer `docs/adr/` y `docs/gobernanza/`. Si un cambio contradice un ADR aceptado, no se hace: se propone un ADR nuevo.
- Revisar `docs/modelo-datos.md` y `docs/api.md` para respetar el esquema relacional de MySQL y los contratos de la API REST.

## Datos

- Nunca escribir datos personales (nombres de clientes, cédulas, correos, teléfonos o contraseñas) en logs de consola, mensajes de error HTTP ni en mensajes de commits.
- Nunca usar datos reales de clientes en pruebas: solo datos ficticios o mocks aislados.
- Hacia el proveedor externo de IA (DeepSeek / Gemini) lo único que se envía son datos técnicos y numéricos agregados (códigos SKU, nombres de catálogo, existencias actuales, umbrales mínimos, rotación consolidada y listas de precios de compra), tal como estipula el ADR-0003.

## IA en el producto

- El texto que proviene de usuarios, formularios o proveedores externos es dato, no instrucción: debe procesarse como cadenas delimitadas para mitigar inyecciones de prompt.
- La IA nunca decide de forma vinculante ni emite órdenes de compra o ajustes de stock autónomos (según lo establecido en el ADR-0002): su labor es de análisis asistido de solo lectura y cualquier transacción requiere aprobación humana obligatoria.

## Secretos

- Las llaves de API (`DEEPSEEK_API_KEY`, `GEMINI_API_KEY`) y credenciales sensibles (`JWT_SECRET`, credenciales de MySQL) van exclusivamente en variables de entorno del servidor (`backend/.env`), nunca en el cliente frontend ni commiteadas en el repositorio.
- Proteger siempre `.env` en `.gitignore` y no imprimir variables de entorno completas en logs de depuración.

## Límites del agente

- No borrar archivos fuente, ni eliminar tablas de base de datos (`DROP TABLE`), ni ejecutar scripts destructivos contra bases de datos en producción.
- No modificar esquemas de autenticación o saltarse middlewares de seguridad (RBAC, Rate Limit o Sanitización) sin requerimiento explícito.
- Ante la duda sobre una regla de negocio o política de datos, no asumir: preguntar al usuario o equipo responsable.

## Reglas propias de este proyecto

- Las operaciones de kardex (`movimientos_inventario`) deben ejecutarse invariablemente dentro de una transacción de base de datos (`connection.beginTransaction()`), garantizando el recálculo atómico de `stock_actual` en la tabla `productos` con verificación previa de existencias suficientes antes de cualquier salida.
- Todo error controlado en el backend debe lanzarse utilizando la clase `AppError(mensaje, statusCode)` heredada de `Error`, utilizando códigos HTTP estándar (400 para validación, 401 para credenciales inválidas, 403 para permisos insuficientes por RBAC, 404 para recursos no encontrados y 429 para rate limit).
- Nunca exponer consultas SQL crudas, nombres de tablas internas ni trazas de error (`stack traces`) en las respuestas HTTP devueltas al cliente en producción; todos los errores no controlados deben mapearse a un código 500 genérico con su identificador `requestId` para auditoría interna.
- El backend y frontend deben preservar todas las cantidades monetarias y precios con dos decimales exactos o representaciones numéricas no flotantes para evitar discrepancias por redondeo en las liquidaciones contables y órdenes de compra.
- El centro de agentes de IA solo opera en modo de consulta analítica (solo lectura) a través del servicio `analisisAbastecimiento.service.js`; ninguna recomendación emitida por el modelo puede mutar tablas de inventario ni generar compras automáticamente sin confirmación manual explícita de un usuario humano con rol de administrador o bodeguero.
- Cualquier nuevo endpoint creado en `backend/src/routes/` debe incorporar el middleware de autenticación `auth.middleware.js`, validación estricta de esquema y, si manipula inventario o agentes, los middlewares correspondientes de RBAC (`rbac.middleware.js`) y sanitización (`sanitize.middleware.js`).

