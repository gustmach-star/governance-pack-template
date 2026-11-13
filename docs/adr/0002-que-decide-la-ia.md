# ADR-0002: la IA extrae y sugiere datos de facturas; la persona autorizada decide

- Estado: Propuesto
- Fecha: 2026-11-13
- Decide: Hellen Quesada, Desarrolladora y responsable técnica de VisionAI

## Contexto

Actualmente VisionAI usa OpenAI con `gpt-4.1-mini` para analizar la imagen de una factura y extraer la información visible. El resultado se guarda automáticamente como texto libre en `base_facturas.facturas.resultado_ia`, sin validación estructurada, sin revisión humana implementada, sin autenticación ni permisos, y sin mecanismo para corregir lo almacenado. Ese resultado podría confundirse con información verificada o usarse para decisiones financieras o administrativas.

## Decisión

La política de gobernanza establece lo siguiente, distinguiendo lo vigente de lo pendiente:

1. La IA puede extraer y sugerir información de las facturas. Esto ya ocurre hoy.
2. La IA no tiene autoridad para aprobar pagos, validar legalmente facturas, autorizar compras ni tomar decisiones financieras. Esta prohibición es política y hoy no está reforzada por controles técnicos.
3. Los resultados de la IA deben considerarse información preliminar, no datos verificados. Hoy el sistema no marca ese estado en el registro.
4. Una persona autorizada debe revisar los resultados antes de usarlos para decisiones financieras o administrativas. Este requisito está establecido, pero no implementado: no existe paso de revisión ni roles.
5. Si la IA comete un error, debe existir un procedimiento de corrección humana. Está definido como obligación, pero todavía no está implementado.
6. Las decisiones sensibles o con consecuencias económicas deben permanecer bajo control humano. Es una regla de la política, no un control automatizado existente.

En resumen: hoy la IA extrae y el resultado se almacena sin revisión; la política exige revisión humana y corrección, y ambas siguen pendientes de implementación.

## Alternativas descartadas

- Dejar que la IA apruebe pagos, valide facturas o autorice compras automáticamente: descartada porque trasladaría consecuencias económicas a un componente sin autoridad legal y sin evidencia de precisión.
- Usar el resultado de la IA como dato verificado sin revisión humana: descartada porque no existe validación estructurada ni medición de errores, y un dato incorrecto podría afectar decisiones financieras.
- Mantener únicamente extracción manual sin IA: descartada porque el objetivo de VisionAI es asistir la extracción; la IA se mantiene como apoyo, no como decisor.

## Consecuencias

La política exige tiempo de una persona autorizada para revisar, lo que reduce el riesgo de usar datos erróneos en decisiones financieras o administrativas. Mientras la revisión no exista, el riesgo permanece: los resultados pueden almacenarse y consultarse sin marca de "preliminar", sin autorización y sin trazabilidad de correcciones. Quedan pendientes: implementar autenticación y roles, agregar un estado de revisión y aprobación, marcar los resultados como preliminares, crear el procedimiento de corrección con registro de cambios y definir quién es la persona autorizada. No se han realizado auditorías ni pruebas de precisión.

## Cómo se verifica

Hoy la verificación es documental: este ADR y el inventario de datos describen la política y declaran que no existe control implementado. En el código no hay un paso de confirmación humana, ni roles, ni una prueba que impida a la IA ejecutar una acción financiera. Cuando se implemente, la verificación deberá mostrar en el backend el paso de revisión obligatorio, la restricción por rol y una prueba automatizada que confirme que un resultado no revisado no puede usarse para aprobar pagos o validar facturas.
