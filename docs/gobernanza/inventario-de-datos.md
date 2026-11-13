# Inventario de datos

**Proyecto:** VisionAI — Sistema de análisis de facturas con inteligencia artificial.

**Objetivo:** Identificar los datos que el sistema recibe, procesa, almacena o transmite a terceros, así como los riesgos asociados a su tratamiento.

**Alcance:** Versión revisada del proyecto `trabajo agente ia`, con backend Node.js, base de datos MySQL y servicio de inteligencia artificial de OpenAI.

| Dato | ¿Personal? | Categoría (Ley 8968, art. 9) | De dónde viene | Dónde se guarda | ¿Sale a un tercero? ¿A quién? | Cuánto se conserva | Quién puede verlo |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Imagen original de la factura | Sí | Acceso restringido cuando contiene datos personales; depende del contenido | Imagen JPG, PNG o WebP subida por el usuario | Temporalmente en memoria del backend y navegador; no se guarda en MySQL ni en disco | Sí, a OpenAI para analizar su contenido | Durante el procesamiento local; retención en OpenAI no verificada | Usuario que la carga, proceso del backend y proveedor de IA |
| Nombre del archivo | Sí | Acceso restringido si identifica a una persona; de lo contrario, no aplica | Nombre original del archivo subido | MySQL: `base_facturas.facturas.nombre_archivo` | No se envía como campo independiente a OpenAI | Indefinidamente, hasta eliminación manual; no existe política implementada | Personas con acceso a la base de datos; no existe consulta pública implementada |
| Resultado completo de la IA | Sí | Acceso restringido cuando contiene datos personales; pueden existir otras categorías según el contenido | Respuesta de OpenAI después de analizar la imagen | MySQL: `base_facturas.facturas.resultado_ia` | Se genera mediante OpenAI, que recibe la imagen original | Indefinidamente, hasta eliminación manual | Personas con acceso a MySQL y usuario que recibe la respuesta de su análisis |
| Nombre del proveedor o emisor | Sí | Acceso restringido si identifica a una persona física; no aplica cuando es información exclusivamente empresarial no personal | Contenido de la factura | Dentro del texto `resultado_ia`, si la IA lo extrae | Sí, a OpenAI mediante la imagen completa | Mientras se conserve el registro de MySQL | Personas con acceso al resultado almacenado o devuelto |
| Número de factura | Sí | Acceso restringido cuando permite identificar a una persona; depende del contenido | Contenido de la factura | Dentro de `resultado_ia`, si la IA lo extrae | Sí, a OpenAI mediante la imagen completa | Mientras se conserve el registro de MySQL | Personas con acceso al resultado almacenado o devuelto |
| Fecha de emisión | No | No aplica, salvo que forme parte de información personal | Contenido de la factura | Dentro de `resultado_ia`, si la IA lo extrae | Sí, a OpenAI mediante la imagen completa | Mientras se conserve el registro de MySQL | Personas con acceso al resultado almacenado o devuelto |
| Productos o servicios y cantidades | Sí | Acceso restringido si revelan información personal; clasificación específica según contenido | Detalle de la factura | Dentro de `resultado_ia`, si la IA lo extrae | Sí, a OpenAI mediante la imagen completa | Mientras se conserve el registro de MySQL | Personas con acceso al resultado almacenado o devuelto |
| Precios, subtotal, impuestos, total y moneda | Sí | No aplica para datos comerciales generales; acceso restringido si están vinculados a una persona | Contenido de la factura | Dentro de `resultado_ia`, si la IA lo extrae | Sí, a OpenAI mediante la imagen completa | Mientras se conserve el registro de MySQL | Personas con acceso al resultado almacenado o devuelto |
| Identificadores fiscales, direcciones o datos bancarios visibles | Sí | Acceso restringido u otra categoría aplicable según el dato | Contenido de la factura | Pueden quedar incluidos en `resultado_ia` porque se solicita extraer toda la información visible | Sí, a OpenAI mediante la imagen completa | Mientras se conserve el registro si se extraen; retención externa no verificada | Personas con acceso al resultado y proveedor de IA |
| Identificador interno de factura | No | No aplica | Generado automáticamente por MySQL | MySQL: `base_facturas.facturas.id_factura` | No se envía a OpenAI | Mientras exista el registro | Personas con acceso a la base de datos y usuario que recibe la respuesta |
| Fecha de registro | No | No aplica | Generada por MySQL al guardar el análisis | MySQL: `base_facturas.facturas.fecha_registro` | No se envía a OpenAI | Mientras exista el registro | Personas con acceso a la base de datos |

## Lo que NO recolectamos

- **Contraseñas de usuarios:** el sistema no dispone actualmente de cuentas de usuario ni inicio de sesión.
- **Datos biométricos:** no se solicitan para analizar facturas.
- **Información médica:** no se solicita como un campo específico de extracción.
- **Opiniones políticas y creencias religiosas:** no forman parte de los datos necesarios para el análisis de facturas.
- **Imagen original como archivo permanente:** no se almacena en MySQL ni en una carpeta del servidor.

**Importante:** aunque el sistema no solicita expresamente datos sensibles, una factura podría contenerlos. Actualmente la imagen completa se envía a OpenAI sin ocultar previamente información privada.

## Observaciones de gobernanza

### 1. Almacenamiento

La base de datos `base_facturas` contiene la tabla `facturas`, con cuatro campos:

- `id_factura`
- `nombre_archivo`
- `resultado_ia`
- `fecha_registro`

El sistema almacena el resultado de la IA como texto libre. No existen columnas independientes para proveedor, número de factura o fecha de emisión.

### 2. Uso de inteligencia artificial

El proveedor utilizado es OpenAI, con el modelo `gpt-4.1-mini`.

El sistema envía la imagen completa de la factura para extraer información. No se identificaron mecanismos implementados de anonimización, ocultamiento de datos personales ni eliminación de metadatos.

### 3. Conservación de datos

Actualmente no existe una política implementada de conservación o eliminación automática.

Los registros permanecen almacenados en MySQL hasta que se eliminen manualmente. La conservación de información por parte de OpenAI no pudo verificarse durante la revisión.

### 4. Acceso a la información

No se identificaron mecanismos de autenticación, roles ni permisos de acceso en la aplicación.

El endpoint de análisis es público para los clientes que puedan alcanzar el servidor. No existe todavía un endpoint para listar o buscar facturas almacenadas.

### 5. Medidas recomendadas, todavía no implementadas

- Definir quién puede cargar, consultar y eliminar facturas.
- Establecer una política de conservación y eliminación de registros.
- Evaluar qué datos personales deben ocultarse antes de enviar imágenes a OpenAI.
- Implementar autenticación y autorización.
- Registrar quién realiza cada análisis.
- Proteger las credenciales y limitar los privilegios de acceso a MySQL.
- Documentar las condiciones de tratamiento y conservación de datos del proveedor de IA.
- Establecer un procedimiento de revisión y corrección de resultados incorrectos.

Estas medidas son propuestas de gobernanza y no deben interpretarse como funcionalidades ya implementadas.
