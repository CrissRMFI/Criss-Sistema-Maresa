# Backlog — Sistema de Gestión Comercial y Operativa

**Versión:** 0.2 (borrador)
**Fecha:** 2026-06-22
**Plataforma:** Aplicación web (Node.js) con backend central.
**Relacionado con:** [especificaciones](especificaciones:md), [modelo_de_dominio](modelo_de_dominio.md)

> Estado: backlog estructural (épicas + features + historias propuestas).
> Actualizado tras la revisión de especificaciones del 2026-06-22 (app web, roles en alcance,
> depósito Móvil, costeo por lotes, sin deuda de mercadería).
> Cada ítem referencia los requisitos de origen (RF / RN / RNF).

---

## Convenciones

- **Épica (E):** gran bloque funcional.
- **Feature (F):** agrupación entregable dentro de una épica.
- **Historia (US):** unidad de trabajo para una iteración.
- Prioridad MVP según especificación §13: `Imprescindible`, `Alta`, `Media`, `Posterior`.

---

## Modelo de roles (en alcance del MVP)

| Acción / visibilidad                         | Dueño | Administrador | Empleado |
| -------------------------------------------- | :---: | :-----------: | :------: |
| Mover stock entre depósitos (cantidades)     |  ✔   |      ✔       |   ✔     |
| Ver costos de stock                          |  ✔   |      ✔       |   ✘     |
| Resolver lote/costo de movimiento pendiente  |  ✔   |      ✔       |   ✘     |
| Facturar / registrar ventas                  |  ✔   |      ✔       |   ✔     |
| Registrar el cobro/pago de la venta          |  ✔   |      ✔       |   ✔     |
| Resolver utilidad pendiente de factura       |  ✔   |      ✔       |   ✘     |
| Ver saldos de las cuentas / dinero           |  ✔   |      ✔       |   ✘     |
| Cierre de caja                               |  ✔   |      ✔       |   ✘     |
| Configuración general / usuarios             |  ✔   |    parcial    |   ✘     |

> El Empleado puede facturar y registrar el cobro, y mover stock entre depósitos, pero no ve costos ni saldos de cuentas ni resuelve la utilidad (§4).

---

## Mapa de épicas

| Épica | Nombre | Prioridad dominante |
| ----- | ------ | ------------------- |
| E0 | Fundaciones técnicas y arquitectura (web/Node.js) | Imprescindible |
| E1 | Usuarios, roles y permisos | Imprescindible |
| E2 | Catálogo: productos, costos y precios | Imprescindible |
| E3 | Entidades comerciales (clientes, proveedores, terceros) | Alta |
| E4 | Stock, ubicaciones y depósito Móvil | Imprescindible |
| E5 | Costeo por lotes y movimientos pendientes | Imprescindible |
| E6 | Cajas retornables | Media |
| E7 | Compras, recepción y pagos a proveedores | Imprescindible |
| E8 | Ventas, entrega y cobros | Imprescindible |
| E9 | Consignación | Media |
| E10 | Cuentas financieras, transferencias y caja | Imprescindible |
| E11 | Gastos y fondos separados | Alta |
| E12 | Cierre de caja | Alta |
| E13 | Reportes y utilidad | Media |
| E14 | Auditoría y seguridad operativa | Alta |

---

## E0 — Fundaciones técnicas y arquitectura
_Origen: RNF-02..RNF-10, §12_

### F0.1 Backend central y persistencia
- US-001 Definir arquitectura web (Node.js) con backend central y base de datos compartida. _(RNF-02, §12)_
- US-002 Esquema de base de datos con soporte multiusuario concurrente. _(RNF-08)_
- US-003 Borrado lógico (soft delete) transversal a entidades operativas. _(RN-01, RF-45)_

### F0.2 Disponibilidad y respaldo
- US-004 Respaldo automático periódico (mínimo semanal) con restauración verificada. _(RNF-05)_
- US-005 Consistencia de datos como prioridad de diseño. _(RNF-04)_

### F0.3 Preparación para extensión futura
- US-006 Backend/API reutilizable para futura app móvil. _(RNF-09)_
- US-007 Estructura preparada para futura integración fiscal (sin implementarla). _(RNF-10)_

---

## E1 — Usuarios, roles y permisos
_Origen: §3, §4, RF-47, RN-04, RF-18/19 (9.3)_

- US-010 Autenticación de usuarios internos. _(RF-47)_
- US-011 Definir roles Dueño, Administrador y Empleado con sus permisos. _(§4)_
- US-012 Permitir al Empleado mover stock, facturar y registrar el cobro, pero sin ver costos, sin ver saldos de cuentas y sin resolver utilidad. _(§4, RF-18 9.3)_
- US-013 Habilitar a Administrador y Dueño la gestión de costos, cuentas, saldos y resolución de pendientes (lotes y utilidad). _(§4, RN-04)_
- US-014 Aplicar control de acceso por rol en cada operación y vista. _(§3)_

---

## E2 — Catálogo: productos, costos y precios
_Origen: RF-01..RF-06, RN-02..RN-07_

### F2.1 Productos
- US-020 Crear, editar, inactivar/reactivar y consultar productos (nombre, marca, presentación, unidad de venta), sin borrado físico. _(RF-01, RF-02, RN-02, §6)_
- US-021 Definir factor de empaque (ej. 1 cajón = N unidades): compra por pack, stock en unidades de venta. _(decisión previa #6)_
- US-022 Marcar si el producto usa caja retornable y su `cantidadPorCaja`. _(modelo 5.1.1)_

### F2.2 Costos
- US-023 Registrar múltiples costos por producto (incluso varios el mismo día), visibles solo a Administrador/Dueño. _(RF-03, RN-03, RN-04)_
- US-024 Consultar historial de costos y promedio simple; permitir que Administrador/Dueño fijen ese promedio como referencia. _(RF-06, RN-05)_

### F2.3 Precios y listas
- US-025 Registrar múltiples precios de venta por producto dentro de una lista y marcar el vigente. _(RF-04, RN-06)_
- US-026 Gestionar `ListaPrecios` (minorista/mayorista/etc.) y asignarla al cliente. _(RF-04, RF-07)_
- US-027 Sugerir precio vigente según la lista del cliente, editable al vender. _(RF-04, RF-05, RN-06)_
- US-028 Mostrar historial de precios por producto; los cambios no reescriben operaciones históricas. _(RF-06, RN-07)_

---

## E3 — Entidades comerciales
_Origen: RF-07..RF-11, RN-18_

### F3.1 Alta y roles
- US-030 Crear/editar entidad comercial con datos básicos y contacto. _(RF-07, RF-10)_
- US-031 Asignar uno o varios roles (cliente, proveedor, tercero, depósito externo). _(RF-11, modelo 5.2.2)_
- US-032 Gestionar múltiples domicilios por entidad. _(RF-07, modelo 5.2.3)_

### F3.2 Crédito y cuenta corriente
- US-033 Configurar límite de crédito por cliente. _(RF-07)_
- US-034 Alerta no bloqueante al superar el límite de crédito. _(RF-08)_
- US-035 Consultar cuenta corriente: movimientos, pagos y saldo de arrastre. _(RF-09)_
- US-036 Visualización separada de deuda en dinero y en cajas. _(RN-18, §5)_
- US-037 Saldo de arrastre que acumula deuda monetaria **y** deuda de cajas/cajones. _(§5)_

---

## E4 — Stock, ubicaciones y depósito Móvil
_Origen: RF-12..RF-17, RF-18..RF-21 (9.3), RN-14, RN-16_

### F4.1 Ubicaciones y existencias
- US-040 Gestionar ubicaciones de stock (local, depósito, consignación, depósito externo). _(RF-12)_
- US-041 Definir el depósito especial **Móvil** como única fuente de ventas/entregas. _(§5, RF-24)_
- US-042 Consultar existencia actual por producto y ubicación. _(modelo 5.3.2)_

### F4.2 Movimientos entre depósitos
- US-043 Mover stock entre depósitos (Empleado, Administrador, Dueño). _(RF-18 9.3, decisión #2)_
- US-044 El Empleado mueve cantidades sin ver costos. _(§4, RF-18 9.3)_
- US-045 Mostrar el stock resultante en cada depósito inmediatamente tras el movimiento. _(RF-20 9.3)_
- US-046 Alertar cuando el movimiento excede la cantidad disponible. _(RF-21 9.3)_

### F4.3 Control y alertas
- US-047 Registrar movimientos de stock tipados (compra, venta, ajuste, traslado, devolución, consignación, reversión). _(RF-13, RN-01)_
- US-048 Ajuste manual de stock con motivo y auditoría. _(RF-14)_
- US-049 Definir stock mínimo y alertar por stock bajo. _(RF-15)_
- US-050 Conteo físico y comparación teórico vs real, con registro de diferencias. _(RF-16, RF-46)_

---

## E5 — Costeo por lotes y movimientos pendientes
_Origen: RF-21/RF-19 (9.3), RF-25 (9.5), RN-03, RN-04, utilidad_

### F5.1 Lotes y movimientos pendientes de costo
- US-051 Modelar lotes de costo por producto (cada ingreso/compra define un costo). _(RN-03)_
- US-052 Al mover stock con **un solo** lote/costo posible, finalizar el movimiento automáticamente. _(RF-21 9.3)_
- US-053 Al mover stock con **más de un** lote/costo posible, dejar el movimiento **pendiente** de asignación de lote. _(RF-21 9.3)_
- US-054 Notificar (🔔) al Administrador/Dueño los movimientos pendientes de lote. _(RF-19 9.3)_
- US-055 Permitir que Administrador/Dueño asignen el lote/costo y finalicen el movimiento. _(RF-19 9.3, RN-04)_

### F5.2 Utilidad pendiente de cálculo en facturas
- US-056 Al facturar desde Móvil, dejar la factura **pendiente de cálculo de utilidad**. _(decisión #1)_
- US-057 Notificar (🔔) al Administrador/Dueño las facturas pendientes de cálculo de utilidad. _(decisión #1)_
- US-058 Permitir especificar el costo a usar (según lotes en Móvil) y calcular utilidad = venta − costo. _(RN-04, decisión #1)_

---

## E6 — Cajas retornables
_Origen: RF-17, RN-19, modelo 5.3.5/5.3.6_

- US-060 Modelar la caja retornable como producto especial sin costo. _(RF-17, decisión #4)_
- US-061 Registrar deuda de cajas por entidad comercial (quién tiene cuántas). _(RN-19, modelo 5.3.6)_
- US-062 Saldar/ajustar deuda de cajas (devolución) con estados abierta/parcial/saldada. _(estados §10)_
- US-063 Mostrar total global de cajas del negocio = presentes + en poder de terceros. _(RN-19, decisión #4)_
- US-064 Integrar la deuda de cajas al saldo de arrastre del cliente. _(§5)_

---

## E7 — Compras, recepción y pagos a proveedores
_Origen: RF-18..RF-22 (9.4), RN-15..RN-17_

### F7.1 Compra
- US-070 Registrar compra (proveedor, fecha, ítems, cantidad, costo unitario, forma de pago, observaciones). _(RF-18 9.4)_
- US-071 Dejar compra pendiente de retiro total o parcial. _(RF-19 9.4, RN-17)_

### F7.2 Recepción
- US-072 Registrar recepción parcial/total y aumentar stock solo por lo recibido, generando el lote de costo. _(RF-20 9.4, RN-16, RN-03)_
- US-073 _(Diferida, fuera del MVP)_ Registrar mercadería dañada/no apta. _(RF-22 9.4, decisión #9)_

### F7.3 Pago y deuda
- US-074 Registrar pagos parciales/totales asociados a compra o deuda con proveedor. _(RF-21 9.4, modelo 5.4.4)_
- US-075 Estados de compra (registrada, pendiente retiro, recibida parcial/total, pagada parcial/total, anulada). _(§10)_
- US-076 Anular compra con reversión y auditoría. _(RN-01)_

---

## E8 — Ventas, entrega y cobros
_Origen: RF-23..RF-30, RN-08..RN-14_

### F8.1 Venta
- US-080 Registrar venta interna con uno o más productos y precio editable (Empleado, Administrador, Dueño). _(RF-23, RF-05, §4)_
- US-081 Pago con múltiples medios de pago. _(RF-23, RN-08)_
- US-082 **Bloquear la venta si no hay stock del producto en el depósito Móvil.** _(RF-24, RF-25, RN-12, §5)_
- US-083 Congelar snapshot de precio en cada ítem. _(modelo 8.4, RN-07)_
- US-084 Estados de venta: borrador, cerrada, entregada, anulada, en consignación. _(§10)_

### F8.2 Entrega
- US-085 Entrega total al cerrar la venta, descontando stock de Móvil. _(RF-24, RN-10, RN-14)_

### F8.3 Cobro
- US-086 Registrar cobros parciales/totales contra venta o saldo de arrastre; el Empleado puede registrar el cobro sin ver saldos de cuentas. _(modelo 5.5.4, §4)_
- US-087 Impacto de cobros en cuentas financieras y cuenta corriente (saldos visibles solo a Administrador/Dueño). _(RF-32, §4)_

### F8.4 Comprobantes y anulación
- US-088 Emitir remitos y recibos internos. _(RF-28)_
- US-089 Anular venta con reversión completa de stock, pagos, deuda y utilidad + auditoría. _(RF-30, RN-09)_

---

## E9 — Consignación
_Origen: RF-29, modelo 5.5.5_

- US-090 Crear `OperacionConsignacion` (entidad separada) y registrar mercadería entregada. _(RF-29, decisión #2)_
- US-091 Registrar lo vendido por el consignatario, lo devuelto y lo pendiente de liquidación. _(RF-29)_
- US-092 Liquidación parcial/total de consignación con estados propios. _(modelo 5.5.5)_

---

## E10 — Cuentas financieras, transferencias y caja
_Origen: RF-31..RF-35_

- US-100 Gestionar cuentas financieras (efectivo, Mercado Pago, bancos, tarjetas, otras). _(RF-31)_
- US-101 Saldo por cuenta en tiempo real. _(RF-32)_
- US-102 Transferencias entre cuentas (salida + entrada). _(RF-33)_
- US-103 Movimientos de cuenta tipados con referencia a operación origen. _(modelo 5.6.2)_

---

## E11 — Gastos y fondos separados
_Origen: RF-36..RF-38, RN-20_

### F11.1 Gastos
- US-110 Registrar gasto por categoría y cuenta de origen. _(RF-36)_
- US-111 Reportar gastos por categoría y cuenta. _(modelo 5.6.5)_

### F11.2 Fondos separados
- US-112 Crear fondo separado (saldo, responsable, observaciones). _(RF-37)_
- US-113 Registrar separación desde el negocio (reduce fondos del negocio). _(RF-38, RN-20)_
- US-114 Registrar consumo del fondo y devolución al negocio sin computarlos como gasto operativo. _(RF-38, RN-20)_

---

## E12 — Cierre de caja
_Origen: RF-34, RF-35_

- US-120 Cierre diario por cuenta financiera. _(RF-34)_
- US-121 Comparar saldo teórico vs real y registrar diferencia. _(RF-35, RF-46)_
- US-122 Historial de cierres y diferencias. _(RF-46)_

---

## E13 — Reportes y utilidad
_Origen: RF-39..RF-43, RN-11_

### F13.1 Reportes operativos
- US-130 Reportes de stock actual y stock valorizado. _(RF-39)_
- US-131 Reportes de ventas, compras, gastos y cuentas. _(RF-39)_
- US-132 Reportes de clientes, proveedores, consignación y cajas. _(RF-39)_
- US-133 Mostrar saldos de arrastre y deudas asociadas junto a reportes. _(RF-42)_

### F13.2 Utilidad
- US-134 Vista única de utilidad por venta (venta − costo). _(RF-40, RF-41, decisión #1)_

### F13.3 Exportación
- US-135 Exportar reportes a PDF. _(RF-43)_

---

## E14 — Auditoría y seguridad operativa
_Origen: RF-44..RF-46, RNF-06, RNF-07_

- US-140 Registrar usuario, fecha y hora en toda operación relevante. _(RF-44)_
- US-141 Bitácora de auditoría (antes/después, motivo, entidad afectada). _(modelo 5.7.2)_
- US-142 Historial de anulaciones, reversiones y diferencias de cierre/conteo. _(RF-46, RNF-06)_
- US-143 Garantizar no eliminación física de movimientos operativos. _(RF-45)_

---

## Decisiones registradas

### Revisión 2026-06-22 (cambio a app web + roles + Móvil)

1. **Plataforma** → aplicación web en **Node.js**, backend central. Sin modo offline. Se descarta el requisito previo de "2 PCs / conexión obligatoria".
2. **Permisos por rol en el alcance del MVP** → Dueño (todo), Administrador (operativo con costos/cuentas), Empleado (mueve stock + factura + registra cobro, pero sin ver costos ni saldos de cuentas y sin resolver utilidad). Ver matriz arriba.
3. **Depósito Móvil** → única fuente de ventas/entregas; no se factura sin stock en Móvil.
4. **Costeo por lotes** → Administrador/Dueño definen costos. Movimiento del Empleado con 1 lote → finaliza solo; con >1 lote → pendiente, resuelto por Administrador (notificación 🔔).
5. **Utilidad** → vista única, **venta − costo**. Al facturar, la utilidad queda **pendiente de cálculo**; Administrador/Dueño la resuelven especificando el costo (notificación 🔔). Se descarta la doble vista de utilidad.
6. **Deuda de mercadería** → **eliminada del MVP** (todo lo facturado se entrega). Se quita la épica de deuda de mercadería y se simplifican los estados de venta.
7. **Saldo de arrastre** → incluye deuda monetaria **y** deuda de cajas/cajones.
8. **RF-19 (9.3) "Dueño no mueve stock"** → se descarta por error de redacción; el Dueño hace todo.

### Decisiones previas que siguen vigentes

- **Consignación** → entidad `OperacionConsignacion` separada.
- **Listas de precios** → entidad `ListaPrecios` asignable al cliente.
- **Cajas retornables** → producto especial sin costo; total global + deuda por entidad.
- **Unidades** → unidad de venta + factor de empaque.
- **Devolución de venta** → solo vía anulación.
- **Mercadería dañada en compra** → fuera del MVP.

---

## Inconsistencias del documento de especificaciones a corregir

Estas quedan **en el documento fuente** ([especificaciones](especificaciones:md)) y conviene resolverlas allí:

1. **Códigos RF duplicados:** la sección 9.3 usa `RF-18, RF-19, RF-20, RF-21` que ya existen en 9.4; además repite `RF-21` y `RF-19` dos veces dentro de 9.3. Sugerido: renumerar los nuevos como `RF-48` en adelante.
2. **RN-10** menciona "deuda de mercadería asociada al cliente y listado específico", contradice §5 (no hay deuda de mercadería). Reescribir.
3. **RF-42** menciona "pendientes de entrega" — ya no aplica.
4. **§13 (prioridades MVP)** lista "deuda de mercadería" en Media — ya no aplica.
5. **RF-40/RN-11** hablan de doble vista de utilidad ("ventas cerradas" vs "entregada"); con entrega total coinciden → dejar una sola.
6. **modelo_de_dominio.md** quedó desactualizado: dice "no se endurecerán permisos", no incluye roles de usuario, depósito Móvil, lotes de costo ni la eliminación de la deuda de mercadería (5.3.4). Requiere actualización.
</content>
</invoke>
