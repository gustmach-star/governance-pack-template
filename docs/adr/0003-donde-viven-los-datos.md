# ADR-0003: datos en MySQL local y envío de la imagen completa a OpenAI

- Estado: Propuesto
- Fecha: 2026-11-13
- Decide: Hellen Quesada, Desarrolladora y responsable técnica de VisionAI

## Contexto

VisionAI recibe imágenes de facturas, las procesa temporalmente y envía la imagen completa a OpenAI con el modelo `gpt-4.1-mini`. El backend es Node.js con Express y MySQL, base `base_facturas`, tabla `facturas`. Las facturas pueden contener datos personales y el resultado de la IA puede incluirlos. No hay autenticación, autorización, auditoría, política de conservación ni eliminación automática implementadas. Tampoco se ha verificado la retención de OpenAI, la existencia o configuración de copias de seguridad, la infraestructura de producción ni la ubicación geográfica del almacenamiento externo.

## Decisión

Se documenta el flujo actual y las decisiones de gobernanza propuestas, separando lo vigente de lo pendiente:

1. En el navegador: la persona selecciona la imagen y el navegador la mantiene en memoria para enviarla; no se implementó almacenamiento persistente local.
2. En el backend: Express recibe el archivo con Multer y lo procesa temporalmente en memoria; no se escribe la imagen en disco de forma permanente.
3. A OpenAI: se envía la imagen completa para extraer la información visible. El nombre del archivo no se envía como campo independiente. La retención del proveedor no está verificada.
4. En MySQL: se guarda `id_factura` (autogenerado), `nombre_archivo`, `resultado_ia` como texto libre y `fecha_registro`. El resultado puede contener proveedor, número de factura, fecha, productos, cantidades, precios, impuestos, total, moneda e identificadores fiscales, direcciones o datos bancarios visibles.
5. No se conserva localmente: el binario de la imagen original no queda en MySQL ni en el disco del backend; solo existe durante el procesamiento.
6. Riesgos actuales: acceso sin autenticación ni roles, cuenta de MySQL con privilegios excesivos, ausencia de auditoría, conservación indefinida sin borrado automático, retención externa no verificada, copias de seguridad no verificadas, infraestructura y ubicación externa no verificadas, y posible exposición de datos personales en la imagen y en el resultado.
7. Decisiones propuestas, pendientes de implementación: autenticación y autorización por roles; cuenta de MySQL de mínimo privilegio; política de conservación y borrado con mecanismo automático; verificar las condiciones y retención de OpenAI; verificar copias de seguridad y su ubicación; documentar la región del alojamiento; evaluar el ocultamiento de datos personales antes del envío; y registrar auditoría de accesos y cambios.

Ninguna de las medidas del punto 7 está implementada a la fecha de este ADR.

## Alternativas descartadas

- Guardar la imagen original en disco o en MySQL: descartada porque aumenta la exposición y el plazo de conservación sin ser necesaria para la función actual, aunque impide volver a analizar la misma imagen sin una nueva carga.
- Usar un almacenamiento externo de imágenes: descartada por ahora porque agrega otro tercero y otra ubicación por verificar, sin una necesidad operativa demostrada.
- Enviar solo texto extraído localmente en lugar de la imagen completa: considerada como opción documental; no se evaluó ni implementó un OCR local, por lo que no se adoptó.

## Consecuencias

El diseño reduce la exposición local al no conservar la imagen, pero no elimina el riesgo: la imagen completa sale a un tercero y el resultado puede contener datos personales almacenados indefinidamente. Para la Ley 8968, esto implica posibles transferencias internacionales cuya ubicación no se ha verificado, dificultad para atender derechos de acceso, rectificación y supresión mientras no exista identificación de titulares ni mecanismo de borrado, y un riesgo elevado por el acceso sin autenticación y la cuenta de MySQL con privilegios excesivos. Quedan pendientes las medidas del punto 7 y la verificación de contratos, regiones, respaldos y retención del proveedor, que no deben darse por existentes.

## Cómo se verifica

La condición actual se comprueba por inspección: el esquema de `base_facturas.facturas` muestra solo los cuatro campos indicados; la configuración de Multer procesa el archivo en memoria y no hay escritura permanente de la imagen; el backend usa Express y la llamada a OpenAI con `gpt-4.1-mini`. No existen pruebas automatizadas, políticas de acceso ni migraciones que demuestren controles de conservación, borrado o mínimo privilegio. Cuando se implementen, la verificación deberá incluir la política de conservación, la cuenta restringida de MySQL, el registro de auditoría y pruebas que confirmen el borrado y la restricción de accesos.
