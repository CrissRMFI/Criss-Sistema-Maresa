# Modelo de Dominio

## Sistema de Gestión Comercial y Operativa para Negocio de Bebidas y Mercadería

**Versión:** 1.0  
**Estado:** Borrador de trabajo  
**Tipo de documento:** Documento vivo de análisis y diseño  
**Relacionado con:** [especificaciones](especificaciones:md)

---

## 1. Propósito del documento

Este documento define el modelo de dominio inicial del sistema. Su objetivo es describir las entidades principales del negocio, sus relaciones, responsabilidades y reglas de consistencia, de modo que sirva como base para el diseño de base de datos, backend, casos de uso e interfaces.

Este documento no describe pantallas ni detalles técnicos de implementación. Describe el negocio y cómo se estructura conceptualmente dentro del sistema.

---

## 2. Alcance del modelo

El modelo de dominio cubre la operación interna inicial del negocio, incluyendo:

- productos
- costos y precios
- clientes, proveedores y terceros
- stock por ubicación
- compras, recepciones y pagos
- ventas, entregas y cobros
- deuda de mercadería
- deuda de cajas retornables
- cuentas financieras
- fondos separados
- gastos
- cierres de caja
- auditoría de operaciones

Quedan fuera de este modelo inicial:

- integración AFIP/ARCA
- permisos refinados por rol
- modo offline con sincronización
- aplicación móvil

---

## 3. Visión general del dominio

El dominio del sistema se organiza en seis áreas principales:

1. **Catálogo comercial**  
   Define qué productos existen, cómo se identifican, qué costos tienen y qué precios pueden sugerirse.

2. **Relaciones comerciales**  
   Define con quién opera el negocio: clientes, proveedores, terceros y depósitos externos.

3. **Stock y logística**  
   Define dónde está la mercadería, cómo se mueve, qué queda pendiente de entregar y cómo se controlan las cajas retornables.

4. **Compras**  
   Define cómo se registran compras, recepciones de mercadería, pagos y deudas con proveedores.

5. **Ventas**  
   Define cómo se registran ventas, entregas físicas, cobros y saldos pendientes.

6. **Dinero y control interno**  
   Define cuentas financieras, gastos, fondos separados, cierres de caja y trazabilidad.

---

## 4. Principios de modelado adoptados

### 4.1 No borrar, siempre auditar

El sistema no debe eliminar físicamente movimientos operativos. Toda anulación o reversión debe quedar registrada y auditada.

### 4.2 Separar operación comercial de operación física

Una venta no equivale automáticamente a una entrega.  
Una compra no equivale automáticamente a una recepción.  
Un cobro o pago tampoco equivale automáticamente al cierre de una operación.

### 4.3 Separar tipos de deuda

El negocio puede registrar deuda en:

- dinero
- mercadería
- cajas retornables

Estas deudas no deben mezclarse en una única estructura genérica.

### 4.4 Mantener historial

Costos, precios, movimientos y anulaciones deben conservar historial.

### 4.5 Priorizar consistencia operativa

La consistencia del negocio es más importante que la flexibilidad técnica. El sistema debe evitar mezclar estados ambiguos o resolver con observaciones lo que requiere una entidad o relación explícita.

---

## 5. Subdominios y entidades principales

---

# 5.1 Catálogo comercial

## 5.1.1 Producto

### Descripción

Representa un artículo comercializable o controlable por stock dentro del negocio.

### Responsabilidades

- identificar un artículo del negocio
- almacenar su información básica
- indicar si se encuentra activo o inactivo
- permitir asociar costos y precios históricos
- indicar si usa cajas retornables

### Atributos principales

- `id`
- `nombre`
- `marca`
- `presentacion`
- `unidadBase`
- `activo`
- `usaCajaRetornable`
- `cantidadPorCaja` (opcional)

### Reglas asociadas

- un producto se identifica operativamente por nombre, marca y presentación
- un producto puede quedar inactivo sin borrarse
- un producto puede tener múltiples costos históricos
- un producto puede tener múltiples precios históricos o vigentes

### Observaciones

No se recomienda guardar el “precio actual” o el “costo actual” directamente en la entidad Producto como única fuente de verdad. Es preferible mantener historial separado.

---

## 5.1.2 CostoProducto

### Descripción

Representa un costo histórico registrado para un producto.

### Responsabilidades

- conservar historial de costos
- permitir seleccionar un costo al momento de vender
- servir de base para reportes de utilidad
- permitir cálculo de promedio simple de costos

### Atributos principales

- `id`
- `productoId`
- `fechaHora`
- `costoUnitario`
- `origen`
- `observacion`

### Reglas asociadas

- puede haber múltiples costos para un mismo producto incluso el mismo día
- el usuario puede elegir desde qué costo vender
- el sistema debe poder consultar promedio simple de costos por producto

### Observaciones

El costo histórico debe quedar asociado a operaciones de compra o alta manual cuando corresponda.

---

## 5.1.3 PrecioProducto

### Descripción

Representa un precio sugerido o vigente para la venta de un producto.

### Responsabilidades

- mantener historial de precios
- sugerir precio de venta vigente
- permitir diferentes precios según contexto comercial
- no reescribir operaciones pasadas

### Atributos principales

- `id`
- `productoId`
- `valor`
- `tipoPrecio`
- `entidadComercialId` (opcional)
- `fechaDesde`
- `activo`

### Reglas asociadas

- el sistema debe sugerir un precio vigente
- el usuario puede modificar manualmente el precio sugerido al vender
- actualizar un precio afecta sugerencias futuras
- cambiar un precio no modifica ventas históricas

---

# 5.2 Relaciones comerciales

## 5.2.1 EntidadComercial

### Descripción

Representa cualquier persona, cliente, proveedor o tercero con quien opera el negocio.

### Responsabilidades

- identificar a una contraparte comercial
- registrar información básica y contacto
- permitir múltiples roles simultáneos
- servir como referencia para deudas, compras, ventas y movimientos

### Atributos principales

- `id`
- `nombre`
- `telefono`
- `activo`
- `limiteCredito`
- `observaciones`

### Reglas asociadas

- una misma entidad puede ser cliente, proveedor, tercero o depósito externo
- una entidad puede tener deuda monetaria, de mercadería y de cajas
- una entidad puede tener varios domicilios

### Observaciones

Se modela como entidad única con múltiples roles para evitar duplicación conceptual y problemas cuando una misma contraparte cumple varias funciones.

---

## 5.2.2 RolEntidad

### Descripción

Representa el rol o conjunto de roles que una entidad comercial puede cumplir dentro del sistema.

### Valores iniciales posibles

- `CLIENTE`
- `PROVEEDOR`
- `TERCERO`
- `DEPOSITO_EXTERNO`

### Reglas asociadas

- una entidad puede tener uno o más roles
- los roles ayudan a clasificar, no a duplicar entidades

---

## 5.2.3 Domicilio

### Descripción

Representa una dirección asociada a una entidad comercial.

### Responsabilidades

- permitir registrar múltiples domicilios
- facilitar entregas, contacto y trazabilidad operativa

### Atributos principales

- `id`
- `entidadComercialId`
- `direccion`
- `descripcion`
- `activo`

---

# 5.3 Stock y logística

## 5.3.1 UbicacionStock

### Descripción

Representa un lugar físico o lógico donde puede encontrarse mercadería o cajas.

### Ejemplos

- local
- depósito principal
- consignación
- depósito externo
- otra ubicación definida por el negocio

### Responsabilidades

- clasificar dónde se encuentra la mercadería
- permitir stock separado por ubicación
- facilitar movimientos y reportes

### Atributos principales

- `id`
- `nombre`
- `tipo`
- `activa`

---

## 5.3.2 ExistenciaStock

### Descripción

Representa la cantidad actual de un producto en una ubicación dada.

### Responsabilidades

- mostrar stock actual por ubicación
- servir como resultado acumulado de movimientos válidos

### Atributos principales

- `productoId`
- `ubicacionId`
- `cantidadActual`

### Observaciones

Conceptualmente existe como agregado del sistema, aunque técnicamente puede calcularse desde movimientos o persistirse como resumen optimizado.

---

## 5.3.3 MovimientoStock

### Descripción

Representa cualquier variación de stock de mercadería.

### Responsabilidades

- registrar entradas y salidas
- vincular cambios de stock con operaciones del negocio
- servir como base de trazabilidad e inventario

### Tipos iniciales

- compra recibida
- venta entregada
- ajuste manual
- traslado salida
- traslado entrada
- devolución
- consignación entrega
- consignación devolución
- anulación o reversión

### Atributos principales

- `id`
- `fechaHora`
- `tipo`
- `productoId`
- `ubicacionOrigenId` (opcional)
- `ubicacionDestinoId` (opcional)
- `cantidad`
- `referenciaTipo`
- `referenciaId`
- `usuarioId`
- `observacion`

### Reglas asociadas

- el stock solo cambia por movimientos válidos
- toda variación de stock debe quedar auditada
- una venta no descuenta stock por lo no entregado
- una compra no aumenta stock por lo no recibido

---

## 5.3.4 DeudaMercaderia

### Descripción

Representa mercadería pendiente de entregar a un cliente luego de una venta cerrada.

### Responsabilidades

- registrar cantidades pendientes por cliente
- permitir consulta de pendientes de entrega
- permitir saldar deuda de mercadería con entregas posteriores

### Atributos principales

- `id`
- `ventaId`
- `clienteId`
- `productoId`
- `cantidadPendiente`
- `estado`
- `fechaOrigen`

### Reglas asociadas

- una venta puede cerrarse aunque no se entregue todo
- la deuda de mercadería debe quedar visible en un listado específico
- la deuda se salda con una entrega posterior
- lo pendiente no sale de stock hasta que se entrega

---

## 5.3.5 CajaRetornable

### Descripción

Representa el control de cajas retornables del negocio.

### Observación importante

En la etapa inicial todas las cajas retornables se consideran del mismo tipo.

### Responsabilidades

- controlar stock físico de cajas
- registrar deuda de cajas de terceros
- reflejar el stock total considerando cajas presentes y cajas adeudadas

---

## 5.3.6 DeudaCajas

### Descripción

Representa la cantidad de cajas retornables que una entidad comercial debe al negocio.

### Responsabilidades

- registrar deuda de cajas por entidad
- permitir conciliación en conteos y reportes
- mostrar deuda separada de dinero y mercadería

### Atributos principales

- `id`
- `entidadComercialId`
- `cantidadAdeudada`
- `estado`
- `fechaOrigen`

### Reglas asociadas

- las cajas forman parte del stock total del negocio
- si no están físicamente presentes, deben reflejarse como deuda de cajas de terceros
- la deuda de cajas se consulta en forma separada

---

# 5.4 Compras

## 5.4.1 Compra

### Descripción

Representa el acuerdo comercial de compra con un proveedor o tercero.

### Responsabilidades

- registrar intención y detalle de compra
- separar la compra de la recepción física
- separar la compra del pago
- permitir deuda con proveedor

### Atributos principales

- `id`
- `proveedorId`
- `fecha`
- `estado`
- `observaciones`

### Reglas asociadas

- compra, recepción y pago son eventos independientes
- una compra puede estar pagada y no recibida
- una compra puede estar recibida y no pagada
- una compra puede recibirse parcialmente

---

## 5.4.2 ItemCompra

### Descripción

Representa una línea de producto dentro de una compra.

### Responsabilidades

- indicar qué producto se compra
- registrar cantidades y costos
- controlar cantidades recibidas y dañadas

### Atributos principales

- `id`
- `compraId`
- `productoId`
- `cantidadPactada`
- `costoUnitario`
- `cantidadRecibida`
- `cantidadDanada`

---

## 5.4.3 RecepcionCompra

### Descripción

Representa un evento de retiro o recepción real de mercadería asociada a una compra.

### Responsabilidades

- registrar cuándo y cuánto se recibió
- aumentar stock únicamente por lo efectivamente recibido
- dejar pendiente lo no recibido

### Atributos principales

- `id`
- `compraId`
- `fecha`
- `usuarioId`
- `observacion`

### Reglas asociadas

- una compra no incrementa stock hasta la recepción
- si la recepción es parcial, el resto queda pendiente de retiro

---

## 5.4.4 PagoCompra

### Descripción

Representa un pago asociado a una compra o a una deuda con proveedor.

### Responsabilidades

- registrar pagos parciales o totales
- asociar pagos con cuentas financieras
- reflejar deuda monetaria pendiente con proveedor

### Atributos principales

- `id`
- `compraId` (opcional)
- `proveedorId`
- `fecha`
- `importe`
- `cuentaFinancieraId`
- `medioPago`
- `observacion`

---

# 5.5 Ventas

## 5.5.1 Venta

### Descripción

Representa una operación comercial interna de venta.

### Responsabilidades

- registrar la operación comercial
- vincular cliente, productos, totales y cobros
- distinguir entre cierre comercial y entrega física
- soportar anulación auditada

### Atributos principales

- `id`
- `clienteId`
- `fecha`
- `estado`
- `totalBruto`
- `totalCobrado`
- `observaciones`

### Estados previstos

- `BORRADOR`
- `CERRADA`
- `ENTREGADA_PARCIAL`
- `ENTREGADA_COMPLETA`
- `ANULADA`
- `EN_CONSIGNACION`
- `LIQUIDADA_PARCIAL`
- `LIQUIDADA_COMPLETA`

### Reglas asociadas

- una venta puede pagarse con múltiples medios
- una venta puede cerrarse aunque exista mercadería pendiente
- una venta anulada debe revertir efectos y dejar auditoría

---

## 5.5.2 ItemVenta

### Descripción

Representa una línea de producto dentro de una venta.

### Responsabilidades

- registrar qué producto se vendió
- conservar el precio de venta aplicado
- conservar el costo elegido al momento de vender
- permitir cálculo de utilidad

### Atributos principales

- `id`
- `ventaId`
- `productoId`
- `cantidadVendida`
- `precioUnitario`
- `costoElegidoId` (opcional)
- `cantidadEntregadaAcumulada`
- `utilidadCalculadaCerrada`

### Reglas asociadas

- el precio y costo usados en la venta deben quedar congelados como snapshot histórico
- no deben recalcularse automáticamente por cambios posteriores de catálogo

---

## 5.5.3 EntregaVenta

### Descripción

Representa una entrega física parcial o total asociada a una venta.

### Responsabilidades

- registrar qué se entregó realmente
- descontar stock solo por lo entregado
- vincular entregas posteriores a la venta original

### Atributos principales

- `id`
- `ventaId`
- `fecha`
- `usuarioId`
- `observacion`

### Reglas asociadas

- el stock baja solo por lo efectivamente entregado
- una venta puede requerir una o más entregas

---

## 5.5.4 CobroVenta

### Descripción

Representa un cobro asociado a una venta o a un saldo de arrastre del cliente.

### Responsabilidades

- registrar cobros parciales o totales
- permitir múltiples medios de pago
- impactar cuentas financieras
- reflejar cuenta corriente del cliente

### Atributos principales

- `id`
- `ventaId` (opcional)
- `clienteId`
- `fecha`
- `importe`
- `cuentaFinancieraId`
- `medioPago`
- `observacion`

---

## 5.5.5 OperacionConsignacion

### Descripción

Representa mercadería entregada en consignación, su venta posterior, devolución o liquidación.

### Responsabilidades

- registrar lo entregado en consignación
- registrar lo vendido por el cliente o tercero
- registrar lo devuelto
- registrar lo pendiente de liquidación

### Observaciones

Aunque puede modelarse como extensión de Venta, se mantiene como concepto propio porque la lógica operativa es distinta.

---

# 5.6 Dinero y control interno

## 5.6.1 CuentaFinanciera

### Descripción

Representa una cuenta o canal de dinero del negocio.

### Ejemplos

- efectivo
- Mercado Pago
- banco
- tarjeta
- otra plataforma

### Responsabilidades

- registrar saldo actual
- recibir y emitir movimientos
- participar en cierres diarios

### Atributos principales

- `id`
- `nombre`
- `tipo`
- `activa`

---

## 5.6.2 MovimientoCuenta

### Descripción

Representa cualquier variación monetaria en una cuenta financiera.

### Responsabilidades

- registrar ingresos y egresos
- vincular operaciones a cuentas
- permitir auditoría y cierres

### Tipos iniciales

- cobro de venta
- pago de compra
- gasto
- transferencia salida
- transferencia entrada
- separación a fondo
- devolución desde fondo
- ajuste de cierre

### Atributos principales

- `id`
- `cuentaFinancieraId`
- `fechaHora`
- `tipo`
- `importe`
- `referenciaTipo`
- `referenciaId`
- `usuarioId`
- `observacion`

---

## 5.6.3 FondoSeparado

### Descripción

Representa dinero apartado del flujo normal del negocio.

### Responsabilidades

- registrar reservas o cuentas especiales
- mostrar saldo actual
- diferenciar estos fondos de gastos operativos

### Atributos principales

- `id`
- `nombre`
- `saldoActual`
- `responsable`
- `observaciones`

### Reglas asociadas

- separar dinero reduce los fondos del negocio
- usar el fondo luego no impacta como gasto operativo del negocio

---

## 5.6.4 MovimientoFondo

### Descripción

Representa ingresos y egresos dentro de un fondo separado.

### Tipos iniciales

- separación desde negocio
- consumo del fondo
- devolución al negocio

### Atributos principales

- `id`
- `fondoSeparadoId`
- `fechaHora`
- `tipo`
- `importe`
- `usuarioId`
- `observacion`

---

## 5.6.5 Gasto

### Descripción

Representa un gasto operativo del negocio.

### Responsabilidades

- registrar egresos reales del negocio
- clasificar por categoría
- vincular cuenta de origen

### Atributos principales

- `id`
- `fecha`
- `categoria`
- `importe`
- `cuentaOrigenId`
- `observacion`

### Reglas asociadas

- no debe confundirse con fondos separados
- debe poder reportarse por categoría y cuenta

---

## 5.6.6 CierreCaja

### Descripción

Representa el cierre diario de una cuenta financiera.

### Responsabilidades

- comparar saldo teórico y saldo real
- detectar diferencias
- dejar trazabilidad del cierre

### Atributos principales

- `id`
- `fecha`
- `cuentaFinancieraId`
- `saldoTeorico`
- `saldoReal`
- `diferencia`
- `usuarioId`
- `observacion`

---

# 5.7 Seguridad y trazabilidad

## 5.7.1 Usuario

### Descripción

Representa un usuario interno autenticado del sistema.

### Responsabilidades

- acceder al sistema
- ejecutar operaciones auditables
- quedar asociado a movimientos y cierres

### Atributos principales

- `id`
- `nombreUsuario`
- `hashPassword`
- `activo`

### Observaciones

En la primera etapa no se endurecerán permisos finos. Todos los empleados tendrán acceso operativo amplio.

---

## 5.7.2 AuditoriaOperacion

### Descripción

Representa el registro histórico de acciones relevantes del sistema.

### Responsabilidades

- registrar qué operación se realizó
- registrar quién la realizó
- registrar cuándo se realizó
- conservar referencia a la entidad afectada
- permitir reconstrucción histórica

### Atributos principales

- `id`
- `fechaHora`
- `usuarioId`
- `tipoOperacion`
- `entidadAfectada`
- `entidadId`
- `resumenAntes` (opcional)
- `resumenDespues` (opcional)
- `motivo`

---

## 6. Relaciones principales del dominio

Las relaciones principales son:

- un **Producto** tiene muchos **CostoProducto**
- un **Producto** tiene muchos **PrecioProducto**
- una **EntidadComercial** tiene muchos **RolEntidad**
- una **EntidadComercial** tiene muchos **Domicilio**
- una **Compra** tiene muchos **ItemCompra**
- una **Compra** tiene muchas **RecepcionCompra**
- una **Compra** puede tener muchos **PagoCompra**
- una **Venta** tiene muchos **ItemVenta**
- una **Venta** puede tener muchas **EntregaVenta**
- una **Venta** puede tener muchos **CobroVenta**
- una **Venta** puede generar muchas **DeudaMercaderia**
- una **CuentaFinanciera** tiene muchos **MovimientoCuenta**
- un **FondoSeparado** tiene muchos **MovimientoFondo**
- una **EntidadComercial** puede tener muchas **DeudaCajas**
- un **Usuario** puede generar muchas **AuditoriaOperacion**

---

## 7. Reglas de consistencia del dominio

1. Ninguna operación crítica debe eliminarse físicamente.
2. Toda anulación o reversión debe conservar trazabilidad.
3. El stock solo puede cambiar por movimientos válidos.
4. Una compra no aumenta stock hasta su recepción.
5. Una venta no descuenta stock por lo que no fue entregado.
6. La deuda de mercadería debe saldarse mediante entregas posteriores.
7. Los precios actuales no deben modificar operaciones históricas.
8. Los costos históricos deben conservarse.
9. Los fondos separados no deben reportarse como gasto operativo.
10. Las deudas monetarias, de mercadería y de cajas deben gestionarse por separado.
11. Toda operación relevante debe registrar usuario, fecha y hora.
12. El sistema debe diferenciar utilidad por ventas cerradas y utilidad por entregas reales.

---

## 8. Decisiones de modelado importantes

### 8.1 Entidad comercial única

Se adopta una entidad comercial general con múltiples roles.  
No se modelan clientes, proveedores y terceros como mundos completamente separados.

### 8.2 Venta y entrega son entidades distintas

Una venta representa el acuerdo comercial.  
Una entrega representa el movimiento físico real de mercadería.

### 8.3 Compra y recepción son entidades distintas

Una compra representa el acuerdo comercial.  
Una recepción representa el retiro o ingreso real de stock.

### 8.4 El costo usado al vender debe quedar congelado

Las operaciones históricas no deben depender de costos o precios “actuales”.

### 8.5 Las cajas retornables se controlan explícitamente

No se modelan como simple observación.  
Deben integrarse al stock total y a la deuda por tercero.

---

## 9. Riesgos de modelado identificados

### 9.1 Mezclar tipos de deuda

Si se mezclan deuda monetaria, deuda de mercadería y deuda de cajas en una sola estructura, el sistema se vuelve ambiguo.

### 9.2 No separar operación comercial de operación física

Si venta y entrega se modelan como lo mismo, se rompe la lógica de stock y deuda de mercadería.

### 9.3 No congelar precios y costos en operaciones

Si una venta depende del catálogo actual en vez de snapshots históricos, se pierde trazabilidad.

### 9.4 Tratar cajas retornables como comentario

Si las cajas se registran solo en texto libre, no habrá control real ni conciliación confiable.

---

## 10. Puntos abiertos para futuras etapas

Estos puntos no bloquean la primera etapa, pero pueden requerir extensión del modelo en el futuro:

- permisos refinados por usuario
- tipos distintos de cajas retornables
- integración fiscal/legal con AFIP/ARCA
- soporte móvil
- analítica avanzada
- reglas más sofisticadas de sugerencia de precios
- reportes contables adicionales

---

## 11. Historial de cambios

| Versión | Fecha      | Cambio                                              |
| ------- | ---------- | --------------------------------------------------- |
| 1.0     | 2026-04-04 | Creación inicial del documento de modelo de dominio |
