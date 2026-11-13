# Inventario de datos · tienda_online

| Dato | ¿Personal? | Categoría (Ley 8968) | Origen | Destino | Almacenamiento | Retención | Seguridad |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Nombre del cliente | Sí | Acceso restringido | Registro de usuario | Tabla `clientes` | Base de datos MySQL | Indefinida | Campo obligatorio `NOT NULL` |
| Correo electrónico | Sí | Acceso restringido | Registro de usuario | Tabla `clientes` y `proveedores` | Base de datos MySQL | Indefinida | Restricción `UNIQUE` |
| Teléfono | Sí | Acceso restringido | Registro de usuario | Tabla `clientes` y `proveedores` | Base de datos MySQL | Indefinida | Tipado `VARCHAR(20)` |
| Dirección y ciudad | Sí | Acceso restringido | Formulario de envío | Tabla `direcciones` | Base de datos MySQL | Indefinida | Relación por FK (`id_cliente`) |
| Stock y precios | No | Acceso irrestricto | Gestión de inventario | Tabla `productos` | Base de datos MySQL | Permanente | Tipos numéricos (`DECIMAL`, `INT`) |

## Lo que NO recolectamos
- **Información biomédica, de salud u origen étnico:** La tienda no almacena datos catalogados como sensibles bajo el artículo 9 de la Ley 8968, ya que el giro comercial es exclusivamente la venta de componentes y artículos tecnológicos.
- **Credenciales bancarias directas:** Las transacciones se derivan a pasarelas externas seguras y no guardamos números completos de tarjetas de crédito.