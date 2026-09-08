# Análisis de Requerimientos y Diseño Conceptual - Base de Datos Licorería

Este documento contiene el levantamiento de información, la especificación de entidades, diccionario de datos, relaciones justificadas y reglas de integridad para el sistema de inventario, ventas y gestión de caja de una licorería familiar.

---

## 1. Narración del Cliente, Actores, Eventos y Restricciones

### Descripción del Negocio
La dueña opera un negocio familiar enfocado en la venta de bebidas alcohólicas (cervezas, licores finos/caros), gaseosas, golosinas y helados. Su modelo combina la venta minorista directa en mostrador con el abastecimiento al por mayor a través de agencias distribuidoras o compras independientes.

"Hola, miren, tengo mi licorería y la verdad es tardado llevar la cuenta de todo a mano. Trabajo con mi familia y a veces no sé exactamente qué tenemos o qué falta comprar hasta que la repisa queda vacía. Todos los días vendo de todo un poco: soda, helados y chocolates, hasta licores caros. La gente me paga en efectivo y mucho por QR. Al final del día (o a la semana), nos sentamos con mi familia a sumar en cuadernos todo lo que vendimos y el dinero que hay en caja para ver cuánto ganamos o si nos falta plata. Además, los proveedores de las agencias de cerveza y licores vienen ciertos días, y nos cuesta saber rápido cuánto les debemos o cuánto producto necesitamos pedirles. Quiero algo sencillo para registrar las ventas rápido (que no me atrase la atención al cliente) y que al final del día saber cuánto vendí en efectivo, cuánto por QR, qué cosas se están terminando y si ganamos plata o no."

### Actores
* **Atendientes (Familiares / Dueña):** Registran las ventas diarias en mostrador, realizan los cierres de caja y gestionan el stock.
* **Clientes:** Compran productos al detalle y pagan mediante Efectivo o Transferencia QR.
* **Proveedores / Agencias:** Visitan el local en días específicos para entregar mercadería por cajas o paquetes al por mayor, generando cuentas pagadas o deudas a crédito.

### Eventos del Negocio
1. **Venta Rápida en Mostrador:** Salida de productos al detalle (unidades) que requiere un registro inmediato para no demorar la atención.
2. **Cobro Multimétodo:** Recepción del dinero dividiendo el ingreso exacto entre Efectivo y QR.
3. **Recepción de Mercadería al Por Mayor:** Ingreso de productos por empaque (cajas, fardos), los cuales se traducen a unidades individuales para el inventario de venta.
4. **Arqueo y Cierre de Caja:** Consolidación diaria/semanal para verificar el total en efectivo, el total recaudado en QR y contrastar contra lo vendido.
5. **Evaluación de Pérdidas y Ganancias:** Cálculo de ganancia neta restando el costo al por mayor de los productos vendidos.
6. **Control de Stock y Pedidos:** Verificación visual o por sistema de productos por agotarse para hacer pedidos en el día de visita del proveedor.
7. **Gestión de Deudas a Proveedores:** Seguimiento de facturas pendientes de pago a crédito.

### Restricciones Operativas
* **Velocidad en Caja:** El proceso de registro de ventas no puede requerir más de 2 o 3 pasos para evitar filas.
* **Desglose de Pago:** Una sola venta puede ser pagada combinando efectivo y QR.
* **Control de Stock por Unidades:** Aunque el producto se compre por cajas de 12 o 24 unidades, el inventario debe restar unidades sueltas.

---

## 2. Entidades y Atributos

* **PRODUCTO:** Representa cada ítem individual que se comercializa en la licorería.
  * *Atributos:* Identificador único, nombre comercial, precio de venta al detalle, costo promedio de adquisición, inventario actual en unidades, inventario mínimo de alerta.
* **CATEGORIA:** Clasificación para agrupar los artículos (ej. Licores, Cervezas, Sodas, Golosinas, Helados).
  * *Atributos:* Identificador único, nombre de la categoría.
* **PROVEEDOR:** Empresas, agencias de licor/cerveza o distribuidores independientes que surten el negocio.
  * *Atributos:* Identificador único, nombre de la empresa/agencia, teléfono de contacto, día fijado de visita, indicador de agencia oficial.
* **VENTA:** Evento que registra la transacción comercial completa en el mostrador.
  * *Atributos:* Identificador único, fecha y hora exacta de la transacción, monto total acumulado.
* **DETALLE_VENTA:** Desglose línea por línea de los productos incluidos en una venta específica.
  * *Atributos:* Identificador único, cantidad de unidades vendidas, precio unitario aplicado al momento de vender, subtotal de la línea.
* **PAGO:** Registro contable de cómo se cobra una venta (Efectivo o QR).
  * *Atributos:* Identificador único, método utilizado (Efectivo / QR), monto abonado con dicho método.
* **COMPRA:** Transacción de abastecimiento realizada con un proveedor (ingreso de mercadería).
  * *Atributos:* Identificador único, fecha de recepción, monto total de la compra, estado del pago (Pagado, Pendiente, Parcial).
* **DETALLE_COMPRA:** Desglose de los empaques al por mayor recibidos en una compra específica.
  * *Atributos:* Identificador único, cantidad de empaques (cajas/packs) comprados, unidades que contiene cada empaque (factor de conversión), costo pagado por empaque, subtotal de la línea.

---

## 3. Claves Primarias y Tipos Básicos para los Atributos

| Entidad | Atributo | Tipo de Dato | Rol | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| **CATEGORIA** | `id_categoria` | `INT` | **PK** | Clave primaria autonumérica. |
| | `nombre` | `VARCHAR(50)` | Atributo | Nombre de la categoría (ej. Licores). |
| **PRODUCTO** | `id_producto` | `INT` | **PK** | Clave primaria autonumérica. |
| | `nombre` | `VARCHAR(100)`| Atributo | Nombre o descripción del producto. |
| | `precio_venta_unitario`| `DECIMAL(10,2)`| Atributo | Precio de venta por unidad al cliente. |
| | `costo_promedio_unitario`| `DECIMAL(10,2)`| Atributo | Costo unitario para cálculo de ganancia. |
| | `stock_actual_unidades`| `INT` | Atributo | Cantidad de unidades físicas en estante. |
| | `stock_minimo_unidades`| `INT` | Atributo | Umbral de alerta para reposición. |
| **PROVEEDOR** | `id_proveedor` | `INT` | **PK** | Clave primaria autonumérica. |
| | `nombre_empresa` | `VARCHAR(100)`| Atributo | Razón social o nombre comercial. |
| | `telefono` | `VARCHAR(20)` | Atributo | Teléfono del preventista o agencia. |
| | `dia_visita` | `VARCHAR(20)` | Atributo | Día de la semana en que pasa a tomar pedido. |
| | `es_agencia` | `BOOLEAN` | Atributo | `TRUE` si es agencia oficial, `FALSE` si es independiente. |
| **VENTA** | `id_venta` | `INT` | **PK** | Clave primaria autonumérica. |
| | `fecha_hora` | `DATETIME` | Atributo | Estampa de tiempo exacta del cobro. |
| | `monto_total` | `DECIMAL(10,2)`| Atributo | Total cobrado en la nota de venta. |
| **DETALLE_VENTA**| `id_detalle_venta`| `INT` | **PK** | Clave primaria autonumérica. |
| | `cantidad` | `INT` | Atributo | Unidades vendidas. |
| | `precio_unitario` | `DECIMAL(10,2)`| Atributo | Precio congelado al instante de la venta. |
| | `subtotal` | `DECIMAL(10,2)`| Atributo | `cantidad` * `precio_unitario`. |
| **PAGO** | `id_pago` | `INT` | **PK** | Clave primaria autonumérica. |
| | `metodo_pago` | `VARCHAR(20)` | Atributo | Especifica si es `'Efectivo'` o `'QR'`. |
| | `monto` | `DECIMAL(10,2)`| Atributo | Cantidad de dinero recibida por este método. |
| **COMPRA** | `id_compra` | `INT` | **PK** | Clave primaria autonumérica. |
| | `fecha_compra` | `DATETIME` | Atributo | Fecha de recepción de la mercadería. |
| | `monto_total` | `DECIMAL(10,2)`| Atributo | Importe total de la nota de abastecimiento. |
| | `estado_pago` | `VARCHAR(20)` | Atributo | Estado comercial (`'Pagado'`, `'Pendiente'`, `'Parcial'`). |
| **DETALLE_COMPRA**| `id_detalle_compra`| `INT` | **PK** | Clave primaria autonumérica. |
| | `cantidad_empaques`| `INT` | Atributo | Número de cajas/packs adquiridos. |
| | `unidades_por_empaque`| `INT` | Atributo | Unidades dentro de cada caja (ej. 12, 24). |
| | `costo_por_empaque`| `DECIMAL(10,2)`| Atributo | Precio pagado por la caja completa. |
| | `subtotal` | `DECIMAL(10,2)`| Atributo | `cantidad_empaques` * `costo_por_empaque`. |

---

## 4. Relaciones entre Entidades y Cardinalidades

* **CATEGORIA (1) a PRODUCTO (N) — [1:N]**
  * *Cardinalidad:* `1:N` (Una categoría contiene de 1 a N productos; un producto pertenece estrictamente a 1 categoría).
  * *Justificación:* Permite clasificar el inventario. Un helado no puede pertenecer a dos categorías al mismo tiempo dentro del catálogo.
* **VENTA (1) a DETALLE_VENTA (N) — [1:N]**
  * *Cardinalidad:* `1:N` (Una venta posee de 1 a N renglones de detalle; un renglón pertenece a 1 sola venta).
  * *Justificación:* Un cliente puede llevarse en la misma compra una soda y un chocolate. Cada artículo requiere su propio renglón dentro del comprobante de venta.
* **PRODUCTO (1) a DETALLE_VENTA (N) — [1:N]**
  * *Cardinalidad:* `1:N` (Un producto puede registrarse en 0 a N detalles de venta; un detalle de venta se refiere a 1 solo producto).
  * *Justificación:* El producto "Cerveza 620ml" puede venderse muchas veces a lo largo del día en distintas transacciones. *(Resuelve la relación conceptual N:M entre VENTA y PRODUCTO)*.
* **VENTA (1) a PAGO (N) — [1:N]**
  * *Cardinalidad:* `1:N` (Una venta recibe de 1 a N registros de pago; un pago pertenece a 1 sola venta).
  * *Justificación:* Permite dividir la cuenta. Si una venta totaliza $100, el cliente puede pagar $40 en Efectivo y $60 mediante QR.
* **PROVEEDOR (1) a COMPRA (N) — [1:N]**
  * *Cardinalidad:* `1:N` (Un proveedor abastece de 0 a N compras; una compra se realiza a 1 solo proveedor).
  * *Justificación:* Las compras de mercadería se le realizan a un distribuidor específico en su día de visita.
* **COMPRA (1) a DETALLE_COMPRA (N) — [1:N]**
  * *Cardinalidad:* `1:N` (Una compra incluye de 1 a N detalles de mercadería; un detalle pertenece a 1 sola compra).
  * *Justificación:* En una sola nota de abastecimiento se pueden pedir 5 cajas de ron y 10 cajas de cerveza.
* **PRODUCTO (1) a DETALLE_COMPRA (N) — [1:N]**
  * *Cardinalidad:* `1:N` (Un producto puede ser abastecido en 0 a N detalles de compra; un detalle de compra abastece a 1 solo producto).
  * *Justificación:* Un producto se reabastece periódicamente en distintas fechas. *(Resuelve la relación conceptual N:M entre COMPRA y PRODUCTO)*.

---

## 5. Restricciones e Integridad de Datos

### Restricciones de Dominio y Chequeo (`CHECK`)
* **Precios y Costos Positivos:** `precio_venta_unitario > 0`, `costo_promedio_unitario >= 0`, `costo_por_empaque >= 0`.
* **Cantidades no Negativas:** `stock_actual_unidades >= 0`, `stock_minimo_unidades >= 0`, `cantidad > 0`, `cantidad_empaques > 0`, `unidades_por_empaque > 0`.
* **Valores Permitidos:** `metodo_pago IN ('Efectivo', 'QR')`, `estado_pago IN ('Pagado', 'Pendiente', 'Parcial')`.

### Restricciones de Unicidad (`UNIQUE`) y Obligatoriedad (`NOT NULL`)
* **`NOT NULL`:** Obligatorio en montos, fechas, cantidades y llaves foráneas. Evita datos inconsistentes o sin referencia.
* **`UNIQUE`:** `CATEGORIA.nombre` y `PRODUCTO.nombre`.

### Ciclos de Vida: Dependencias de Existencia (Composición vs. Agregación)
* **Relaciones por Composición (Entidades Débiles):**
  * `DETALLE_VENTA` respecto a `VENTA`: Si se elimina la venta padre, se eliminan sus detalles en cascada (`ON DELETE CASCADE`).
  * `DETALLE_COMPRA` respecto a `COMPRA`: Si se elimina la compra, se eliminan sus renglones en cascada (`ON DELETE CASCADE`).
  * `PAGO` respecto a `VENTA`: Si la venta se cancela, los pagos asociados se eliminan en cascada (`ON DELETE CASCADE`).
* **Relaciones por Agregación (Entidades Independientes):**
  * `PRODUCTO` en `DETALLE_VENTA` / `DETALLE_COMPRA`: No se permite eliminar un producto si ya registra transacciones históricas (`ON DELETE RESTRICT`).
  * `PROVEEDOR` en `COMPRA`: No se borra un proveedor con facturas registradas (`ON DELETE RESTRICT`).
  * `CATEGORIA` en `PRODUCTO`: Restringido mientras existan productos asociados (`ON DELETE RESTRICT`).
