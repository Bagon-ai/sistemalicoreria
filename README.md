# sistemalicoreria
docs/README.md
# 🍾 Sistema de Gestión y POS para Licorería

Este proyecto contiene el diseño de arquitectura, modelo relacional y especificidad de requerimientos para la automatización de ventas, inventario y arqueo de caja de un negocio de licorería familiar.

---

## 📋 1. Historia de Usuario y Contexto del Cliente

### Entrevista / Problema Planteado
*-La dueña atiende una licorería familiar junto a sus parientes. Comercializa bebidas alcohólicas, gaseosas, dulces y helados.-*

"Hola, miren, la verdad es tardado llevar la cuenta de todo a mano. Trabajo con mi familia y a veces no sé exactamente qué tenemos o qué falta comprar hasta que la repisa queda vacía.
Todos los días vendo de todo un poco: soda, helados y chocolates, hasta licores caros. La gente me paga en efectivo y mucho por QR. Al final del día (o a la semana), nos sentamos con mi familia a sumar en cuadernos todo lo que vendimos y el dinero que hay en caja para ver cuánto ganamos o si nos falta plata.
Además, los proveedores de las agencias de cerveza vienen ciertos días, y nos cuesta saber rápido cuánto les debemos o cuánto producto necesitamos pedirles.
Quiero algo sencillo para registrar las ventas rápido (que no me atrase la atención al cliente) y que al final del día saber cuánto vendí en efectivo, cuánto por QR, qué cosas se están terminando y si ganamos plata o no."

**Puntos de dolor identificados:**
- **Control manual a papel y lápiz:** El recuento de ganancias, cuadre de caja y control de inventario se hace manualmente al final de la jornada.
- **Doble canal de cobro:** Manejo de pagos en **Efectivo** y mediante **QR**.
- **Gestión de Stock Mayorista:** Adquisición de productos por cajas/fardos a agencias distribuidoras, pero venta por unidades individuales al cliente final.

---

## 🎯 2. Requerimientos del Sistema

1. **Punto de Venta Ágil (POS):** Registro de ventas diferencando pago en Efectivo y QR.
2. **Gestión de Inventario y Despiece:** Entrada por cajas/fardos desde proveedores y conversión automática a stock de unidades individuales.
3. **Cierre de Caja (Arqueo Automatizado):** Comparación automática entre el dinero esperado por el sistema (Efectivo vs. QR) y el dinero real en caja para detectar sobrantes/faltantes.
4. **Alerta de Reposición:** Control de stock mínimo por producto.

---

## 🗄️ 3. Modelo Entidad-Relación (MER)

### Entidades y Atributos

#### 1. USUARIO (Fuerte)
* `id_usuario` (PK, Entero, Autoincremental)
* `nombre` (VARCHAR 100, Not Null)
* `rol` (VARCHAR 20, Not Null) -> *CHECK ('ADMIN', 'CAJERO')*
* `estado` (BOOLEAN, Default true)

#### 2. PROVEEDOR (Fuerte)
* `id_proveedor` (PK, Entero, Autoincremental)
* `nombre_empresa` (VARCHAR 120, Unique, Not Null)
* `telefono` (VARCHAR 20, Nullable)

#### 3. PRODUCTO (Fuerte)
* `id_producto` (PK, Entero, Autoincremental)
* `codigo_barras` (VARCHAR 50, Unique, Nullable)
* `nombre` (VARCHAR 150, Not Null)
* `categoria` (VARCHAR 50, Not Null) -> *CHECK ('LICOR', 'BEBIDA', 'DULCE', 'HELADO', 'OTROS')*
* `unidades_por_caja` (INT, Default 1) -> *Factor de conversión mayorista*
* `precio_compra_unitario` (DECIMAL 10,2, Not Null)
* `precio_venta_unitario` (DECIMAL 10,2, Not Null)
* `stock_actual` (INT, Default 0)
* `stock_minimo` (INT, Default 5)

#### 4. COMPRA (Fuerte)
* `id_compra` (PK, Entero, Autoincremental)
* `id_proveedor` (FK -> PROVEEDOR)
* `id_usuario` (FK -> USUARIO)
* `fecha_compra` (TIMESTAMP, Default NOW())
* `monto_total` (DECIMAL 10,2)

#### 5. DETALLE_COMPRA (Débil / Intermedia)
* `id_detalle_compra` (PK, Entero, Autoincremental)
* `id_compra` (FK -> COMPRA)
* `id_producto` (FK -> PRODUCTO)
* `cajas_compradas` (INT, Not Null)
* `cantidad_unidades_total` (INT) -> *Calculado: cajas_compradas * unidades_por_caja*
* `precio_costo_caja` (DECIMAL 10,2)
* `precio_costo_unitario` (DECIMAL 10,2)
* `subtotal` (DECIMAL 10,2)

#### 6. VENTA (Fuerte)
* `id_venta` (PK, Entero, Autoincremental)
* `id_usuario` (FK -> USUARIO)
* `fecha_hora` (TIMESTAMP, Default NOW())
* `metodo_pago` (VARCHAR 20) -> *CHECK ('EFECTIVO', 'QR', 'MIXTO')*
* `monto_total` (DECIMAL 10,2)

#### 7. DETALLE_VENTA (Débil / Intermedia)
* `id_detalle_venta` (PK, Entero, Autoincremental)
* `id_venta` (FK -> VENTA)
* `id_producto` (FK -> PRODUCTO)
* `cantidad` (INT, Not Null)
* `precio_unitario` (DECIMAL 10,2)
* `subtotal` (DECIMAL 10,2)

#### 8. CIERRE_CAJA (Fuerte)
* `id_cierre` (PK, Entero, Autoincremental)
* `id_usuario` (FK -> USUARIO)
* `fecha_apertura` (TIMESTAMP)
* `fecha_cierre` (TIMESTAMP, Default NOW())
* `monto_efectivo_sistema` (DECIMAL 10,2)
* `monto_qr_sistema` (DECIMAL 10,2)
* `monto_efectivo_real` (DECIMAL 10,2)
* `diferencia` (DECIMAL 10,2)

---

## 🔄 4. Relaciones y Cardinalidades

| Entidad Origen | Relación | Entidad Destino | Cardinalidad |
| :--- | :--- | :--- | :--- |
| `PROVEEDOR` | Surtir | `COMPRA` | 1 : N |
| `USUARIO` | Registrar | `COMPRA` | 1 : N |
| `COMPRA` | Contener | `DETALLE_COMPRA` | 1 : N |
| `PRODUCTO` | Pertenecer | `DETALLE_COMPRA` | 1 : N |
| `USUARIO` | Atender | `VENTA` | 1 : N |
| `VENTA` | Contener | `DETALLE_VENTA` | 1 : N |
| `PRODUCTO` | Pertenecer | `DETALLE_VENTA` | 1 : N |
| `USUARIO` | Realizar | `CIERRE_CAJA` | 1 : N |

---

## 🛠️ 5. Estructura de Archivos del Repositorio

```text
├── database/
│   ├── schema.sql         # Script de creación de tablas (DDL)
│   └── triggers.sql       # Triggers para la actualización de stock
├── docs/
│   └── especificaciones.md # Especificación detallada del MER y negocio
└── README.md              # Documentación general
