# ADR-0001: Selección de DeepSeek API como motor de inferencia analítica con contingencia en Google Gen AI

- Estado: Aceptado
- Fecha: 2026-11-13
- Decide: Daniel Ramírez, Líder Técnico y Responsable de Arquitectura de Almacéntrico

## Contexto

Almacéntrico requiere un motor de inferencia de lenguaje natural para ejecutar el flujo colegiado de cuatro agentes especializados (Coordinador, Inventario, Demanda y Proveedores) que analizan existencias, rotación de kardex y listas de precios. El sistema demanda alta capacidad de razonamiento estructurado en español, estricta adherencia a esquemas JSON y llamadas de herramientas (tool calling), baja latencia de respuesta (inferior a 15 segundos para la deliberación completa) y costos operativos sostenibles para una PYME sin comprometer datos personales de clientes ni credenciales.

## Decisión

Se adopta **DeepSeek API** (`deepseek-chat`) como proveedor y modelo primario para la inferencia de los agentes, con un adaptador de contingencia opcional hacia **Google Generative AI** (`gemini-2.5-flash`):

1. **Aislamiento en backend:** El modelo se consume exclusivamente desde el servidor a través de `backend/src/providers/ai/proveedor.adapter.js` y `backend/src/services/analisisAbastecimiento.service.js`. El frontend jamás interactúa directamente con los endpoints del LLM.
2. **Gestión de secretos:** La llave de acceso (`DEEPSEEK_API_KEY` o `GEMINI_API_KEY`) reside únicamente en el archivo de variables de entorno del servidor (`backend/.env`), cargada en tiempo de ejecución vía `process.env` y excluida del control de versiones mediante `.gitignore`.
3. **Resiliencia y degradación elegante:** Cada llamada al proveedor cuenta con control de excepciones y un timeout de 45 segundos por defecto (configurable hasta 30 segundos mediante la variable de entorno `AI_TIMEOUT_MS`, gestionado vía `AbortController`). Si el proveedor externo no responde, excede la cuota (HTTP 429) o sufre una interrupción de servicio (HTTP 5xx), el sistema entra en modo degradado controlado, emitiendo una respuesta estructurada que notifica al usuario sin derribar la aplicación.
4. **Perímetro de datos:** Hacia el modelo solo se envían identificadores SKU de productos, descripciones técnicas de catálogo, cantidades numéricas de existencias, sumatorias de rotación mensual y precios de compra de proveedores. Ningún dato personal (identificación de clientes, teléfonos, correos o contraseñas) se incluye en los prompts.

## Alternativas descartadas

- **OpenAI GPT-4o:** Descartado como motor principal por su costo significativamente mayor por millón de tokens de entrada/salida para las ráfagas repetitivas de análisis de stock multiagente, el cual superaba el presupuesto operativo previsto.
- **Modelos locales (Ollama / Llama 3 8B en servidor propio):** Descartado para la fase actual por los altos requisitos de hardware con GPU dedicada (VRAM de al menos 16 GB), complejidad de despliegue en infraestructura on-premise y menor consistencia en seguimiento de esquemas JSON complejos en español en máquinas sin aceleración.
- **No usar IA y basarse únicamente en alertas estáticas fijas:** Descartado porque las reglas estáticas no sintetizan tendencias estacionales de kardex, no comparan dinámicamente ventajas cualitativas de múltiples proveedores ni generan resúmenes ejecutivos comprensibles para el jefe de bodega.

## Consecuencias

- **Positivas:** Reducción sustancial del costo de inferencia manteniendo alta precisión en la salida JSON requerida por el frontend (`CentroAgentesPage.jsx`). Arquitectura desacoplada mediante patrón Adapter que permite alternar entre DeepSeek y Gemini sin modificar la lógica de negocio de los agentes.
- **Negativas / Riesgos:** Dependencia de conectividad externa y de la disponibilidad del proveedor. Riesgo de latencia variable en horas pico de la API externa.
- **Costos estimados:** Menos de $5 USD mensuales para un volumen proyectado de 1.000 deliberaciones analíticas mensuales con DeepSeek.
- **Mitigaciones operativas:** Implementación de limitador de tasa (`rateLimit.middleware.js` a nivel de ruta `/api/agentes`), caché de resultados analíticos recientes y manejo de errores mediante clase `AppError` con respuestas estandarizadas.

## Cómo se verifica

- **Implementación del adaptador:** `backend/src/providers/ai/proveedor.adapter.js` implementa la clase `ProveedorAI` y el método `generarRespuestaAgente` consumiendo `process.env.DEEPSEEK_API_KEY`.
- **Invocación orquestada:** `backend/src/services/analisisAbastecimiento.service.js` orquesta las cuatro fases de deliberación con manejo de errores y validación de esquemas.
- **Pruebas automatizadas:** La suite de pruebas `backend/tests/agentes.test.js` verifica la generación estructurada de respuestas y el aislamiento de claves de entorno.

