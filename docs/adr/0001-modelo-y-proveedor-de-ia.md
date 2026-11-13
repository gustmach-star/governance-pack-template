# ADR-0001: OpenAI con gpt-4.1-mini para extraer datos de facturas

- Estado: Aceptado
- Fecha: 2026-11-13
- Decide: Hellen Quesada, Desarrolladora y responsable técnica de VisionAI

## Contexto

VisionAI necesita leer imágenes de facturas y extraer proveedor, número de factura, fecha, productos, cantidades, precios, impuestos, total y moneda. Las facturas pueden variar de formato y contener datos personales. El sistema no tiene autenticación ni política de conservación implementada, y los resultados se guardan como texto libre en MySQL. No se han medido precisión, latencia ni costo.

## Decisión

Se usa el proveedor OpenAI con el modelo `gpt-4.1-mini` para analizar la imagen completa de la factura y devolver los campos solicitados. La llamada se hace desde el backend Node.js y la llave de API se lee de una variable de entorno del servidor, no del cliente. El resultado se almacena como texto en `base_facturas.facturas.resultado_ia`; la imagen original no se guarda en MySQL ni en disco. Actualmente no se aplica anonimización ni ocultamiento de datos antes del envío.

## Alternativas descartadas

- Modelos locales de código abierto: considerados solo como opción documental; no se ejecutaron pruebas de precisión, latencia ni costo, por lo que no se adoptaron en esta versión.
- Otro proveedor comercial de visión: considerado para evaluación documental; no se realizó una comparación ejecutada de resultados ni de condiciones de tratamiento.
- Extracción manual sin IA: descartada porque el objetivo del proyecto es automatizar la extracción; no se midió el ahorro de tiempo.

## Consecuencias

Se gana extracción automatizada de datos de facturas sin conservar las imágenes originales. Se asume dependencia de OpenAI y el envío de la imagen completa, que puede incluir datos personales sin ocultamiento previo. No existen mediciones de costo mensual, precisión ni latencia, ni pruebas de carga. Quedan pendientes: definir conservación y borrado, evaluar el ocultamiento de datos personales antes del envío, documentar las condiciones de tratamiento del proveedor, implementar autenticación y monitorear el gasto. Si el proveedor falla o cambia sus condiciones, el análisis queda indisponible hasta definir una alternativa.

## Cómo se verifica

Se comprueba por inspección del backend Node.js: la llamada a OpenAI usa el modelo `gpt-4.1-mini` y toma la llave de una variable de entorno del servidor. En MySQL, el esquema de `base_facturas.facturas` confirma que solo se guarda `resultado_ia` como texto y que no hay columna para la imagen. No existen pruebas automatizadas que verifiquen este comportamiento; la revisión es manual.
