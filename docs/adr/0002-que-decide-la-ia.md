# ADR-0002: Separación entre recomendaciones analíticas de IA y decisiones vinculantes humanas

- Estado: Aceptado
- Fecha: 2026-11-13
- Decide: Daniel Ramírez, Líder Técnico y Responsable de Arquitectura de Almacéntrico

## Contexto

Almacéntrico gestiona el inventario de una distribuidora de suministros donde una falta de existencias (stockout) detiene operaciones de clientes y un sobreabastecimiento genera costos financieros y de almacenamiento. El sistema incorpora un módulo multiagente con 4 agentes (Coordinador, Inventario, Demanda y Proveedores) para evaluar necesidades de compra basadas en existencias mínimas, rotación histórica de kardex y listas de precios de proveedores homologados.

Delegar la emisión automática de órdenes de compra a un modelo de lenguaje (LLM) conllevaría riesgos severos de compras erróneas, alucinación de cantidades, compromisos financieros no autorizados o pedidos a proveedores desactualizados.

## Decisión

Se adopta una arquitectura estricta de **Solo Lectura con Aprobación Humana en el Bucle (Human-in-the-Loop)**:

1. **Lo que la IA decide y hace autónomamente:**
   - La IA consulta en modo de solo lectura las tablas de existencias, kardex y proveedores mediante herramientas SQL parametrizadas (`consultarExistencias`, `consultarSalidas`, `consultarPedidosPendientes`, `consultarProveedoresProducto`).
   - Calcula rotación promedio diaria y sugiere de forma no vinculante cantidades recomendadas de reabastecimiento y selección preliminar de proveedores por costo o plazo.
   - Consolida una propuesta estructurada en formato JSON validada por el servidor antes de ser expuesta.

2. **Lo que la IA NUNCA hace ni decide:**
   - La IA **nunca** crea, confirma ni emite órdenes de compra directamente en la base de datos (`ordenes_compra`).
   - La IA **nunca** realiza ajustes de inventario, bajas físicas ni transferencias de stock entre almacenes.
   - La IA **nunca** evalúa ni califica el desempeño laboral de colaboradores ni toma determinaciones sobre clientes individuales.
   - La IA **nunca** tiene permisos de escritura SQL (`INSERT`, `UPDATE`, `DELETE`) en las tablas de negocio.

3. **Intervención y aprobación humana:**
   - Las sugerencias se presentan exclusivamente en la interfaz web (`CentroAgentesPage.jsx`) con badges explícitos de revisión.
   - El operador o comprador humano debe auditar la justificación, verificar disponibilidad de presupuesto y decidir manualmente si genera la orden de compra a través del formulario oficial de adquisiciones.

## Alternativas descartadas

- **Abastecimiento autónomo sin supervisión:** Descartado porque cualquier alucinación en el contexto de inferencia generaría órdenes vinculantes a proveedores con impacto de flujo de caja y responsabilidad contractual irreversible.
- **Reglas estáticas heurísticas simples (sin IA):** Descartado como única solución porque no permiten ponderar dinámicamente ventanas de estacionalidad, advertencias cualitativas de proveedores ni justificaciones semánticas consolidadas.

## Consecuencias

- **Positivas:** Se eliminan compras accidentales o no autorizadas por fallos del modelo; los auditores disponen de trazabilidad en la tabla `ejecuciones_agentes`; los datos enviados a la API son agregados y anónimos.
- **Negativas:** Requiere tiempo de revisión por parte del personal de compras antes de que los insumos sean ordenados formalmente.
- **Costo operacional:** Las peticiones al endpoint de DeepSeek consumen tokens controlados mediante cuotas fijas (`AI_MAX_CALLS_PER_EXECUTION=10`) y límite de tasa (`aiRateLimiter`), evitando bucles infinitos de gasto.

## Cómo se verifica

- En el código del servicio [`backend/src/services/analisisAbastecimiento.service.js`](file:///c:/Users/calva/taller-cli/Almacéntrico/backend/src/services/analisisAbastecimiento.service.js), todas las consultas de los agentes se limitan a sentencias `SELECT` sin mutación sobre las tablas de inventario ni de órdenes de compra.
- En la suite automatizada [`backend/tests/agentes.test.js`](file:///c:/Users/calva/taller-cli/Almacéntrico/backend/tests/agentes.test.js), la prueba *«Herramientas controladas: consultarExistencias identifica déficit y estados»* comprueba que la ejecución devuelve propuestas en memoria sin mutar la base de datos.
- En la interfaz de usuario [`frontend/src/pages/CentroAgentesPage.jsx`](file:///c:/Users/calva/taller-cli/Almacéntrico/frontend/src/pages/CentroAgentesPage.jsx), el badge superior indica permanentemente: *«Análisis de solo lectura · Sin mutaciones automáticas de stock»*.
