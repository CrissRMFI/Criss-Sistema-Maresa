# Modelo de Dominio

## Sistema de Gestión Comercial y Operativa para Negocio de Bebidas y Mercadería

**Versión:** 1.1
**Estado:** Documento vivo de análisis y diseño
**Plataforma:** Aplicación web (Node.js) con backend central.
**Relacionado con:** [especificaciones](especificaciones:md), [backlog](backlog.md), [historias_de_usuario](historias_de_usuario.md)

---

## 1. Propósito del documento

Este documento define el modelo de dominio del sistema: entidades, relaciones, responsabilidades y reglas de consistencia, como base para el diseño de base de datos, backend, casos de uso e interfaces. No describe pantallas ni detalles técnicos de implementación.

---

## 2. Alcance del modelo

Cubre la operación interna del negocio:

- productos, costos, precios y listas de precios
- clientes, proveedores y terceros
- stock por ubicación y depósito Móvil
- lotes de costo y movimientos de stock (con resolución de costo pendiente)
- compras, recepciones y pagos
- ventas, entrega total y cobros (con utilidad pendiente de cálculo)
- deuda de cajas retornables
- cuentas financieras, fondos separados, gastos y cierres de caja
- usuarios, roles y auditoría

Quedan fuera del MVP:

- integración AFIP/ARCA
- modo offline con sincronización
- aplicación móvil
- **deuda de mercadería** (todo lo facturado se entrega; no se factura sin stock en Móvil)

---

## 3. Visión general del dominio

Áreas principales:

1. **Catálogo comercial** — productos, costos, precios y listas.
2. **Relaciones comerciales** — clientes, proveedores, terceros y depósitos externos.
3. **Stock y logística** — ubicaciones (incluido el depósito Móvil), lotes de costo, movimientos y cajas retornables.
4. **Compras** — compras, recepciones, pagos y deudas con proveedores.
5. **Ventas** — ventas, entrega total, cobros y utilidad.
6. **Dinero y control interno** — cuentas, gastos, fondos separados y cierres.
7. **Seguridad y trazabilidad** — usuarios, roles, permisos y auditoría.

---

## 4. Principios de modelado adoptados

- **4.1 No borrar, siempre auditar.** No se eliminan físicamente movimientos; toda anulación/reversión queda auditada.
- **4.2 Separar operación comercial de operación física.** Una compra no equivale a una recepción; un cobro/pago no equivale al cierre.
- **4.3 Separar tipos de deuda.** El negocio registra deuda en dinero y en cajas (no en mercadería en el MVP). No se mezclan en una estructura genérica.
- **4.4 Mantener historial.** Costos, precios, movimientos y anulaciones conservan historial.
- **4.5 Priorizar consistencia operativa.** Se prefiere una entidad/relación explícita antes que resolver con observaciones de texto libre.
- **4.6 Costos por lote y visibilidad por rol.** El costo del stock se maneja por lotes; el Empleado mueve cantidades sin ver costos, y Administrador/Dueño resuelven costos y utilidad.

---

## 5. Subdominios y entidades principales

---

# 5.1 Catálogo comercial

## 5.1.1 Producto

Artículo comercializable o controlable por stock.

**Atributos principales:** `id`, `nombre`, `marca`, `presentacion`, `unidadVenta`, `factorEmpaque` (opcional, ej. 1 cajón = N unidades), `activo`, `usaCajaRetornable`, `cantidadPorCaja` (opcional), `esCajaRetornable` (marca el producto especial "caja").

**Reglas:**
- se identifica por nombre, marca y presentación
- puede quedar inactivo sin borrarse
- puede tener múltiples costos (vía lotes) y múltiples precios
- la compra puede registrarse por pack (factor de empaque) y el stock se lleva en unidad de venta
- la **caja retornable** se modela como un producto especial **sin costo**

## 5.1.2 CostoProducto / Lote

Costo histórico de un producto, asociado a un lote de ingreso.

**Atributos principales:** `id`, `productoId`, `fechaHora`, `costoUnitario`, `cantidadIngresada`, `origen` (compra/alta manual), `recepcionCompraId` (opcional), `observacion`.

**Reglas:**
- cada recepción de compra genera un lote con su costo
- puede haber múltiples costos para un mismo producto, incluso el mismo día
- el sistema consulta promedio simple de costos y permite fijarlo como referencia (Administrador/Dueño)
- el costo es visible solo para Administrador y Dueño

## 5.1.3 PrecioProducto

Precio sugerido/vigente para la venta, dentro de una lista de precios.

**Atributos principales:** `id`, `productoId`, `listaPreciosId`, `valor`, `fechaDesde`, `activo`.

**Reglas:**
- el sistema sugiere el precio vigente de la lista del cliente
- el usuario puede modificar el precio sugerido al vender
- actualizar un precio afecta sugerencias futuras, no ventas históricas

## 5.1.4 ListaPrecios

Conjunto de precios para un segmento comercial (minorista, mayorista, etc.).

**Atributos principales:** `id`, `nombre`, `activa`.

**Reglas:**
- una lista agrupa precios de productos
- un cliente referencia una lista; el precio sugerido proviene de esa lista

---

# 5.2 Relaciones comerciales

## 5.2.1 EntidadComercial

Cualquier contraparte: cliente, proveedor o tercero.

**Atributos principales:** `id`, `nombre`, `telefono`, `activo`, `limiteCredito`, `listaPreciosId` (opcional), `observaciones`.

**Reglas:**
- una misma entidad puede ser cliente, proveedor, tercero o depósito externo (múltiples roles)
- puede tener deuda monetaria y deuda de cajas (no de mercadería)
- puede tener varios domicilios

## 5.2.2 RolEntidad

Rol(es) que cumple una entidad: `CLIENTE`, `PROVEEDOR`, `TERCERO`, `DEPOSITO_EXTERNO`. Una entidad puede tener uno o más; los roles clasifican, no duplican.

## 5.2.3 Domicilio

Dirección asociada a una entidad. **Atributos:** `id`, `entidadComercialId`, `direccion`, `descripcion`, `activo`.

---

# 5.3 Stock y logística

## 5.3.1 UbicacionStock

Lugar físico o lógico donde está la mercadería o las cajas.

**Atributos principales:** `id`, `nombre`, `tipo` (`LOCAL`, `DEPOSITO`, `CONSIGNACION`, `DEPOSITO_EXTERNO`, `MOVIL`), `activa`.

**Reglas:**
- el depósito **Móvil** es la **única fuente de ventas/entregas**
- no se puede facturar un producto sin stock en Móvil

## 5.3.2 ExistenciaStock

Cantidad actual de un producto en una ubicación. **Atributos:** `productoId`, `ubicacionId`, `cantidadActual`. Puede calcularse desde movimientos o persistirse como resumen.

## 5.3.3 MovimientoStock

Cualquier variación de stock de mercadería (incluye cajas, como producto especial).

**Tipos:** compra recibida, venta entregada, ajuste manual, traslado salida, traslado entrada, devolución, consignación entrega, consignación devolución, anulación/reversión.

**Atributos principales:** `id`, `fechaHora`, `tipo`, `productoId`, `ubicacionOrigenId` (opcional), `ubicacionDestinoId` (opcional), `cantidad`, `loteId` (opcional), `estadoCosto` (`FINALIZADO` / `PENDIENTE_COSTO`), `referenciaTipo`, `referenciaId`, `usuarioId`, `observacion`.

**Reglas:**
- el stock solo cambia por movimientos válidos y toda variación queda auditada
- una compra no aumenta stock por lo no recibido
- una venta no descuenta stock por lo no entregado
- **costeo por lote en traslados:** si el producto movido tiene un único lote/costo, el movimiento se finaliza automáticamente (`FINALIZADO`); si tiene más de uno, la cantidad se mueve pero el movimiento queda `PENDIENTE_COSTO` hasta que Administrador/Dueño asignen el lote
- el Empleado mueve cantidades sin ver ni elegir costos

## 5.3.4 CajaRetornable y DeudaCajas

La caja retornable se controla como **producto especial sin costo** (ver 5.1.1). Su stock global del negocio = cajas presentes + cajas en poder de terceros.

**DeudaCajas** — cajas que una entidad debe al negocio.
**Atributos:** `id`, `entidadComercialId`, `cantidadAdeudada`, `estado` (`ABIERTA`/`PARCIAL`/`SALDADA`), `fechaOrigen`.

**Reglas:**
- importa saber **quién tiene cuántas** y **cuántas hay en total**
- la deuda de cajas se gestiona y se consulta separada del dinero
- la deuda de cajas integra el saldo de arrastre del cliente

> **Nota:** la entidad `DeudaMercaderia` del modelo anterior fue **eliminada** del MVP (todo lo facturado se entrega).

---

# 5.4 Compras

## 5.4.1 Compra

Acuerdo comercial de compra con un proveedor.
**Atributos:** `id`, `proveedorId`, `fecha`, `estado` (registrada, pendiente de retiro, recibida parcial/total, pagada parcial/total, anulada), `observaciones`.
**Reglas:** compra, recepción y pago son independientes; puede estar pagada y no recibida, o recibida y no pagada, o recibirse parcialmente.

## 5.4.2 ItemCompra

Línea de producto de una compra. **Atributos:** `id`, `compraId`, `productoId`, `cantidadPactada`, `costoUnitario`, `cantidadRecibida`.

## 5.4.3 RecepcionCompra

Retiro/recepción real de mercadería; genera el lote de costo.
**Atributos:** `id`, `compraId`, `fecha`, `usuarioId`, `observacion`.
**Reglas:** aumenta stock solo por lo recibido; lo no recibido queda pendiente.

## 5.4.4 PagoCompra

Pago asociado a una compra o a deuda con proveedor.
**Atributos:** `id`, `compraId` (opcional), `proveedorId`, `fecha`, `importe`, `cuentaFinancieraId`, `medioPago`, `observacion`.

---

# 5.5 Ventas

## 5.5.1 Venta

Operación comercial interna de venta. La entrega es **total** al cerrar.

**Atributos principales:** `id`, `clienteId`, `fecha`, `estado`, `totalBruto`, `totalCobrado`, `estadoUtilidad` (`PENDIENTE_CALCULO` / `CALCULADA`), `utilidad` (opcional), `usuarioId`, `observaciones`.

**Estados:** `BORRADOR`, `CERRADA`, `ENTREGADA`, `ANULADA`, `EN_CONSIGNACION`.

**Reglas:**
- no se puede facturar sin stock del producto en Móvil
- puede pagarse con múltiples medios
- al facturar, la utilidad queda `PENDIENTE_CALCULO`; Administrador/Dueño la resuelven especificando el costo (notificación)
- la utilidad se computa como **venta − costo** (vista única)
- una venta anulada revierte stock, cobros, deuda y utilidad, dejando auditoría
- el Empleado puede registrar la venta y su cobro, pero no resuelve utilidad ni ve saldos

## 5.5.2 ItemVenta

Línea de producto vendida. **Atributos:** `id`, `ventaId`, `productoId`, `cantidad`, `precioUnitario` (snapshot), `costoAsignadoId` (opcional, al resolver utilidad), `utilidadCalculada` (opcional).
**Reglas:** el precio aplicado queda congelado (snapshot); no se recalcula por cambios de catálogo.

## 5.5.3 EntregaVenta

Entrega física (total) asociada a la venta; descuenta stock de Móvil.
**Atributos:** `id`, `ventaId`, `fecha`, `usuarioId`, `observacion`.

## 5.5.4 CobroVenta

Cobro asociado a una venta o a saldo de arrastre del cliente.
**Atributos:** `id`, `ventaId` (opcional), `clienteId`, `fecha`, `importe`, `cuentaFinancieraId`, `medioPago`, `observacion`.

## 5.5.5 OperacionConsignacion

Mercadería entregada en consignación, su venta posterior, devolución y liquidación. Entidad **separada** de Venta porque su lógica operativa es distinta.
**Responsabilidades:** registrar lo entregado, lo vendido por el consignatario, lo devuelto y lo pendiente de liquidación; estados de liquidación parcial/total.

---

# 5.6 Dinero y control interno

## 5.6.1 CuentaFinanciera

Cuenta o canal de dinero (efectivo, Mercado Pago, banco, tarjeta, otra).
**Atributos:** `id`, `nombre`, `tipo`, `activa`. Los saldos son visibles solo a Administrador/Dueño.

## 5.6.2 MovimientoCuenta

Variación monetaria en una cuenta. **Tipos:** cobro de venta, pago de compra, gasto, transferencia salida/entrada, separación a fondo, devolución desde fondo, ajuste de cierre.
**Atributos:** `id`, `cuentaFinancieraId`, `fechaHora`, `tipo`, `importe`, `referenciaTipo`, `referenciaId`, `usuarioId`, `observacion`.

## 5.6.3 FondoSeparado y MovimientoFondo

Dinero apartado del flujo del negocio. **FondoSeparado:** `id`, `nombre`, `saldoActual`, `responsable`, `observaciones`. **MovimientoFondo:** separación desde negocio, consumo del fondo, devolución al negocio.
**Reglas:** separar dinero reduce los fondos del negocio; usar el fondo luego no impacta como gasto operativo.

## 5.6.4 Gasto

Egreso operativo del negocio. **Atributos:** `id`, `fecha`, `categoria`, `importe`, `cuentaOrigenId`, `observacion`. No se confunde con fondos separados; se reporta por categoría y cuenta.

## 5.6.5 CierreCaja

Cierre diario de una cuenta. **Atributos:** `id`, `fecha`, `cuentaFinancieraId`, `saldoTeorico`, `saldoReal`, `diferencia`, `usuarioId`, `observacion`.

---

# 5.7 Seguridad, roles y trazabilidad

## 5.7.1 Usuario y Rol

Usuario interno autenticado. **Atributos:** `id`, `nombreUsuario`, `hashPassword`, `rol` (`DUENO` / `ADMINISTRADOR` / `EMPLEADO`), `activo`.

**Matriz de permisos:**

| Acción / visibilidad                        | Dueño | Administrador | Empleado |
| ------------------------------------------- | :---: | :-----------: | :------: |
| Mover stock entre depósitos (cantidades)    |  ✔   |      ✔       |   ✔     |
| Ver costos de stock                         |  ✔   |      ✔       |   ✘     |
| Resolver lote/costo de movimiento pendiente |  ✔   |      ✔       |   ✘     |
| Facturar / registrar ventas                 |  ✔   |      ✔       |   ✔     |
| Registrar el cobro/pago de la venta         |  ✔   |      ✔       |   ✔     |
| Resolver utilidad pendiente de factura      |  ✔   |      ✔       |   ✘     |
| Ver saldos de las cuentas / dinero          |  ✔   |      ✔       |   ✘     |
| Cierre de caja                              |  ✔   |      ✔       |   ✘     |
| Configuración general / usuarios            |  ✔   |    parcial    |   ✘     |

## 5.7.2 AuditoriaOperacion

Registro histórico de acciones relevantes. **Atributos:** `id`, `fechaHora`, `usuarioId`, `tipoOperacion`, `entidadAfectada`, `entidadId`, `resumenAntes` (opcional), `resumenDespues` (opcional), `motivo`.

---

## 6. Relaciones principales del dominio

- un **Producto** tiene muchos **CostoProducto/Lote**
- un **Producto** tiene muchos **PrecioProducto**
- una **ListaPrecios** tiene muchos **PrecioProducto** y muchos **clientes** asignados
- una **EntidadComercial** tiene muchos **RolEntidad** y muchos **Domicilio**
- una **EntidadComercial** puede tener muchas **DeudaCajas**
- una **Compra** tiene muchos **ItemCompra**, muchas **RecepcionCompra** y muchos **PagoCompra**
- una **RecepcionCompra** genera uno o más **Lote**
- una **Venta** tiene muchos **ItemVenta**, una **EntregaVenta** (total) y muchos **CobroVenta**
- un **MovimientoStock** puede referenciar un **Lote** y queda `FINALIZADO` o `PENDIENTE_COSTO`
- una **CuentaFinanciera** tiene muchos **MovimientoCuenta**
- un **FondoSeparado** tiene muchos **MovimientoFondo**
- un **Usuario** genera muchas **AuditoriaOperacion**

---

## 7. Reglas de consistencia del dominio

1. Ninguna operación crítica se elimina físicamente.
2. Toda anulación o reversión conserva trazabilidad.
3. El stock solo cambia por movimientos válidos.
4. Una compra no aumenta stock hasta su recepción.
5. Una venta no descuenta stock por lo no entregado; la entrega es total y sale de Móvil.
6. No se factura un producto sin stock en Móvil (no hay deuda de mercadería).
7. Los precios actuales no modifican operaciones históricas; el precio aplicado se congela.
8. Los costos se manejan por lote y se conservan; un movimiento con más de un lote queda pendiente hasta su resolución.
9. La utilidad se calcula como venta − costo; al facturar queda pendiente hasta que Administrador/Dueño la resuelvan.
10. Los fondos separados no se reportan como gasto operativo.
11. Las deudas monetarias y de cajas se gestionan por separado.
12. Toda operación relevante registra usuario, fecha y hora.
13. La visibilidad de costos, saldos y utilidad está restringida por rol (no visible al Empleado).

---

## 8. Decisiones de modelado importantes

- **8.1 Entidad comercial única** con múltiples roles (no mundos separados de cliente/proveedor/tercero).
- **8.2 Venta y entrega** son entidades distintas, pero en el MVP la entrega es total al cerrar.
- **8.3 Compra y recepción** son entidades distintas.
- **8.4 Costo y precio congelados** en la operación; las históricas no dependen del catálogo actual.
- **8.5 Costeo por lote** con resolución pendiente cuando hay ambigüedad de costo.
- **8.6 Depósito Móvil** como única fuente de ventas.
- **8.7 Cajas retornables** como producto especial sin costo, con control de total global y deuda por entidad.
- **8.8 Utilidad única** (venta − costo) con cálculo diferido al facturar.
- **8.9 Roles en el MVP** (Dueño/Administrador/Empleado) con visibilidad restringida de costos, saldos y utilidad.

---

## 9. Riesgos de modelado identificados

- **9.1** Mezclar deuda monetaria y de cajas en una sola estructura genera ambigüedad.
- **9.2** No separar operación comercial de física rompe la lógica de stock.
- **9.3** No congelar precios/costos en operaciones pierde trazabilidad.
- **9.4** Tratar cajas retornables como comentario impide su control real.
- **9.5** No resolver a tiempo movimientos `PENDIENTE_COSTO` o ventas con utilidad pendiente deja reportes de costo/utilidad incompletos.

---

## 10. Puntos abiertos para futuras etapas

- permisos más finos dentro de cada rol
- tipos distintos de cajas retornables
- integración fiscal/legal con AFIP/ARCA
- soporte móvil
- analítica avanzada y reglas más sofisticadas de sugerencia de precios
- reincorporación eventual de venta con entrega parcial / deuda de mercadería

---

## 11. Historial de cambios

| Versión | Fecha      | Cambio                                                                                                                                                              |
| ------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.0     | 2026-04-04 | Creación inicial del documento de modelo de dominio.                                                                                                               |
| 1.1     | 2026-06-22 | Alineación con especificaciones v1.1: app web, roles, depósito Móvil, lotes de costo y movimientos/utilidad pendientes, lista de precios, eliminación de deuda de mercadería, caja retornable como producto especial. |
</content>
</invoke>
