# ADR-0003: Almacenamiento local en MySQL InnoDB y perímetro de datos enviados a terceros

- Estado: Aceptado
- Fecha: 2026-11-13
- Decide: Daniel Ramírez, Líder Técnico y Responsable de Arquitectura de Almacéntrico

## Contexto

Almacéntrico procesa datos comerciales, transaccionales y de inventario esenciales para la cadena de suministro, además de registrar cuentas de usuarios internos del sistema, datos de clientes (personas físicas y jurídicas) y representantes de proveedores. Conforme a la Ley de Protección de la Persona frente al Tratamiento de sus Datos Personales de Costa Rica (Ley 8968), el sistema debe clasificar rigurosamente sus repositorios, aislar la información de acceso restringido, garantizar la integridad transaccional del kardex y definir claramente qué datos pueden cruzar las fronteras hacia servicios de inferencia externos.

## Decisión

Se define una arquitectura de base de datos relacional on-premise / contenedorizada con perímetro de privacidad estricto:

1. **Ubicación, región y motor de datos:** Todos los datos persistentes del sistema residen en una instancia de **MySQL 8.4** con motor transaccional **InnoDB**, alojada en la infraestructura on-premise de la distribuidora en San José, Costa Rica (`127.0.0.1:3306`), gestionada mediante el pool de conexiones parametrizadas de `mysql2/promise` (`backend/src/config/database.js`). Para la inferencia analítica de IA, las solicitudes hacia DeepSeek API (`https://api.deepseek.com`) se transmiten de forma cifrada mediante HTTPS a los centros de datos del proveedor externo bajo condiciones de servicio de API técnica sin persistencia para entrenamiento.
2. **Perímetro hacia proveedores de IA (DeepSeek / Google):**
   - **Lo que SÍ se envía:** Exclusivamente datos operativos y técnicos agregados: código SKU, descripción de producto en catálogo, nivel de stock actual, umbral mínimo configurado, consumo mensual consolidado (sumatoria de salidas en kardex) y listas comparativas de precios/plazos de proveedores.
   - **Lo que NUNCA se envía:** Datos personales de usuarios (nombres, correos, hashes de contraseñas), cédulas o nombres de clientes, direcciones físicas, información crediticia y cualquier token o secreto del sistema.
3. **Seguridad y control de acceso:**
   - Acceso autenticado mediante tokens JWT (firmados con secreto `JWT_SECRET` en servidor, expiración de 8 horas).
   - Control de acceso basado en roles (RBAC) mediante middleware (`backend/src/middleware/rbac.middleware.js`), restringiendo las operaciones de escritura y consulta sensible según el rol (`admin`, `bodeguero`, `auditor`).
   - Contraseñas almacenadas de forma irreversible mediante hash Bcrypt con costo computacional de 12 rondas (`backend/src/controllers/auth.controller.js`).
4. **Conservación y supresión:**
   - Los registros de movimientos de inventario (kardex) y facturación se conservan durante un mínimo de 5 años para cumplimiento tributario y comercial según el Código de Comercio de Costa Rica.
   - La retención y depuración se gestionan mediante políticas operativas y procedimientos de mantenimiento administrativo controlado (respaldos de base de datos y depuración manual asistida), sin procesos de purga autónomos desatendidos para prevenir cualquier eliminación accidental del histórico contable.
   - Los logs técnicos y registros de peticiones HTTP locales se rotan periódicamente con vigencia operativa recomendada de 90 días.
   - Para solicitudes de supresión de datos personales (derecho de cancelación bajo Ley 8968), el administrador ejecuta la anonimización de datos de contacto de clientes inactivos sin alterar los totales contables e históricos de los movimientos pasados.

## Alternativas descartadas

- **Base de datos como servicio en la nube pública (AWS RDS / Supabase en región US):** Descartada para la versión actual debido a los requerimientos de la empresa de operar con latencia mínima de red local en bodega y mantener la soberanía física directa sobre los registros de kardex sin incurrir en costos fijos mensuales en moneda extranjera.
- **Envío directo de volcados de base de datos o esquemas completos al LLM:** Descartado categóricamente para evitar la fuga involuntaria de datos personales (PII) o información comercial estratégica hacia servidores de terceros.
- **Almacenamiento NoSQL documental (MongoDB):** Descartado porque la naturaleza del negocio exige consistencia ACID estricta en operaciones de débito/crédito de stock en el kardex, respaldada por claves foráneas y bloqueos de fila (`FOR UPDATE`).

## Consecuencias

- **Cumplimiento legal (Ley 8968):** Se salvaguardan los datos de acceso restringido (correos, teléfonos, credenciales). Las transferencias internacionales se limitan estrictamente a datos comerciales no personales (códigos SKU y métricas cuantitativas desasociadas).
- **Auditabilidad:** Toda deliberación del centro de agentes y cada modificación del inventario queda registrada con ID de usuario y marca de tiempo en tablas locales de auditoría.
- **Cargas de mantenimiento:** La empresa asume la responsabilidad del mantenimiento, respaldos diarios periódicos (scripts de `mysqldump` cifrados) y custodia física del servidor de base de datos.

## Cómo se verifica

- **Configuración de base de datos:** `backend/src/config/database.js` demuestra la conexión al pool local de MySQL con codificación `utf8mb4`.
- **Estructura relacional y claves foráneas:** `backend/src/database/schema.sql` y `docs/modelo-datos.md` detallan las tablas y restricciones referenciales.
- **Sanitización de parámetros para IA:** `backend/src/services/analisisAbastecimiento.service.js` muestra cómo se proyectan únicamente columnas técnicas (`sku`, `nombre_producto`, `stock_actual`, `stock_minimo`) antes de construir el prompt hacia el adaptador de IA.
- **Protección de endpoints:** `backend/src/middleware/rbac.middleware.js` y las suites de pruebas `backend/tests/middlewaresNuevos.test.js` comprueban el rechazo de peticiones no autorizadas.

