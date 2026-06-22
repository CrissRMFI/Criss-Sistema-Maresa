# Historias de Usuario y Criterios de Aceptación

**Versión:** 0.1 (borrador)
**Fecha:** 2026-06-22
**Relacionado con:** [especificaciones](especificaciones:md), [modelo_de_dominio](modelo_de_dominio.md), [backlog](backlog.md)

> Este documento detalla las historias de usuario (US) del backlog con sus criterios de aceptación (CA).
> Formato de US: **Como** \<rol\>, **quiero** \<objetivo\>, **para** \<beneficio\>.
> Formato de CA: **Dado que** \<contexto\>, **cuando** \<acción\>, **entonces** \<resultado esperado\>.
> Alcance de esta versión: épicas Imprescindibles del MVP (E0, E1, E2, E4, E5, E7, E8, E10).

---

## E0 — Fundaciones técnicas y arquitectura

### US-001 — Arquitectura web con backend central
**Como** equipo de desarrollo, **quiero** una aplicación web (Node.js) con backend central y base de datos compartida, **para** que varios usuarios operen sobre los mismos datos de forma consistente.
- **CA1 — Dado que** el sistema está desplegado, **cuando** dos usuarios acceden desde navegadores distintos, **entonces** ambos ven y operan sobre la misma base de datos central.
- **CA2 — Dado que** un usuario realiza una operación, **cuando** otro usuario consulta el dato afectado, **entonces** ve el cambio sin necesidad de reinstalar ni sincronizar manualmente.

### US-003 — Borrado lógico transversal
**Como** dueño, **quiero** que ninguna operación se borre físicamente, **para** conservar trazabilidad y auditoría.
- **CA1 — Dado que** existe un movimiento operativo, **cuando** un usuario lo "elimina" o anula, **entonces** el registro se marca como inactivo/anulado pero permanece en la base.
- **CA2 — Dado que** un registro fue anulado, **cuando** se consulta la auditoría, **entonces** se ve quién, cuándo y por qué se anuló.

### US-004 — Respaldo periódico
**Como** dueño, **quiero** respaldos automáticos al menos semanales, **para** poder recuperar la información ante una falla.
- **CA1 — Dado que** está configurado el respaldo, **cuando** transcurre el período definido (≤ 1 semana), **entonces** se genera un respaldo sin intervención manual.
- **CA2 — Dado que** existe un respaldo, **cuando** se ejecuta una restauración de prueba, **entonces** los datos se recuperan íntegros.

---

## E1 — Usuarios, roles y permisos

### US-010 — Autenticación de usuarios internos
**Como** usuario interno, **quiero** autenticarme con mis credenciales, **para** acceder a las funciones que me corresponden.
- **CA1 — Dado que** tengo un usuario activo, **cuando** ingreso credenciales correctas, **entonces** accedo al sistema con mi rol asignado.
- **CA2 — Dado que** ingreso credenciales incorrectas, **cuando** intento entrar, **entonces** el sistema rechaza el acceso y no revela cuál dato falló.
- **CA3 — Dado que** mi usuario está inactivo, **cuando** intento ingresar, **entonces** el sistema deniega el acceso.

### US-011 — Definición de roles
**Como** dueño, **quiero** definir los roles Dueño, Administrador y Empleado, **para** acotar lo que cada uno puede hacer y ver.
- **CA1 — Dado que** soy dueño, **cuando** creo o edito un usuario, **entonces** puedo asignarle uno de los roles Dueño, Administrador o Empleado.
- **CA2 — Dado que** un usuario tiene un rol asignado, **cuando** opera el sistema, **entonces** solo accede a las acciones y vistas permitidas por su rol.

### US-012 — Restricciones del Empleado
**Como** dueño, **quiero** que el Empleado pueda mover stock, facturar y cobrar pero sin ver costos ni saldos ni resolver utilidad, **para** proteger la información sensible del negocio.
- **CA1 — Dado que** soy Empleado, **cuando** muevo stock entre depósitos, **entonces** opero cantidades pero no veo ningún costo.
- **CA2 — Dado que** soy Empleado, **cuando** registro una venta y su cobro, **entonces** puedo completarla pero no veo los saldos de las cuentas.
- **CA3 — Dado que** soy Empleado, **cuando** intento acceder a costos, saldos de cuentas o a resolver la utilidad de una venta, **entonces** el sistema no muestra esas opciones.

### US-013 — Capacidades de Administrador y Dueño
**Como** administrador o dueño, **quiero** gestionar costos, cuentas, saldos y resolver pendientes, **para** controlar la operación completa del negocio.
- **CA1 — Dado que** soy administrador o dueño, **cuando** abro un producto, **entonces** veo sus costos y lotes.
- **CA2 — Dado que** soy administrador o dueño, **cuando** existen movimientos o ventas pendientes, **entonces** puedo resolver el lote/costo y la utilidad.

### US-014 — Control de acceso por operación
**Como** dueño, **quiero** que cada operación valide el rol del usuario, **para** evitar que alguien ejecute algo fuera de sus permisos.
- **CA1 — Dado que** un usuario sin permiso intenta una acción restringida (vía UI o petición directa), **cuando** se procesa, **entonces** el sistema la rechaza y registra el intento.

---

## E2 — Catálogo: productos, costos y precios

### US-020 — Gestión de productos
**Como** administrador, **quiero** crear, editar, inactivar y consultar productos, **para** mantener el catálogo del negocio sin perder historia.
- **CA1 — Dado que** quiero un producto nuevo, **cuando** lo creo con nombre, marca, presentación y unidad de venta, **entonces** queda disponible para operar.
- **CA2 — Dado que** existe un producto, **cuando** lo edito, **entonces** se guardan los cambios sin alterar operaciones históricas ya registradas.
- **CA3 — Dado que** un producto ya no se usa, **cuando** lo inactivo, **entonces** deja de aparecer en nuevas operaciones pero permanece en consultas históricas.
- **CA4 — Dado que** un producto está inactivo, **cuando** lo reactivo, **entonces** vuelve a estar disponible para operar.

### US-021 — Factor de empaque
**Como** administrador, **quiero** definir el factor de empaque (ej. 1 cajón = N unidades), **para** comprar por pack y llevar el stock en unidades de venta.
- **CA1 — Dado que** un producto tiene factor de empaque N, **cuando** registro una compra de X packs, **entonces** el stock se incrementa en X·N unidades de venta.
- **CA2 — Dado que** un producto no usa empaque, **cuando** lo opero, **entonces** la cantidad se maneja directamente en unidades de venta.

### US-022 — Marca de caja retornable
**Como** administrador, **quiero** indicar si un producto usa caja retornable y su `cantidadPorCaja`, **para** controlar las cajas asociadas.
- **CA1 — Dado que** un producto usa caja retornable, **cuando** lo configuro, **entonces** queda vinculado al control de cajas.
- **CA2 — Dado que** un producto no usa caja retornable, **cuando** lo opero, **entonces** no genera movimientos de cajas.

### US-023 — Registro de costos por producto
**Como** administrador, **quiero** registrar múltiples costos por producto (incluso varios el mismo día), **para** conservar el historial de costos. Solo Administrador y Dueño los ven.
- **CA1 — Dado que** soy administrador, **cuando** registro un costo para un producto, **entonces** se agrega al historial con fecha/hora y origen, sin pisar costos anteriores.
- **CA2 — Dado que** existen costos cargados, **cuando** un Empleado consulta el producto, **entonces** no ve ningún costo.

### US-024 — Historial y promedio de costos
**Como** administrador, **quiero** consultar el historial y el promedio simple de costos, **para** decidir precios y referencias.
- **CA1 — Dado que** un producto tiene varios costos, **cuando** consulto su historial, **entonces** veo todos los costos con su fecha.
- **CA2 — Dado que** un producto tiene varios costos, **cuando** pido el promedio simple, **entonces** el sistema lo calcula y permite fijarlo como referencia.

### US-025 — Precios por lista
**Como** administrador, **quiero** registrar precios de venta dentro de una lista y marcar el vigente, **para** sugerir precios según el tipo de cliente.
- **CA1 — Dado que** una lista existe, **cuando** asigno un precio a un producto en esa lista, **entonces** queda como precio vigente de esa lista.
- **CA2 — Dado que** actualizo un precio, **cuando** se registra, **entonces** afecta las sugerencias futuras pero no las ventas históricas.

### US-026 — Gestión de listas de precios
**Como** administrador, **quiero** crear listas de precios (minorista, mayorista, etc.) y asignarlas a clientes, **para** que cada cliente reciba su precio correspondiente.
- **CA1 — Dado que** creo una lista, **cuando** la asigno a un cliente, **entonces** ese cliente queda vinculado a esa lista.
- **CA2 — Dado que** un cliente tiene una lista asignada, **cuando** se le vende, **entonces** el precio sugerido proviene de esa lista.

### US-027 — Sugerencia de precio editable
**Como** vendedor (Empleado/Administrador/Dueño), **quiero** que el precio se sugiera según la lista del cliente y poder modificarlo, **para** agilizar la venta sin perder flexibilidad.
- **CA1 — Dado que** selecciono un cliente con lista, **cuando** agrego un producto a la venta, **entonces** se completa el precio sugerido de su lista.
- **CA2 — Dado que** veo el precio sugerido, **cuando** lo modifico manualmente, **entonces** la venta usa el precio modificado y queda registrado el valor aplicado.

### US-028 — Historial de precios
**Como** administrador, **quiero** ver el historial de precios por producto, **para** auditar cambios.
- **CA1 — Dado que** un producto tuvo cambios de precio, **cuando** consulto su historial, **entonces** veo cada precio con su fecha de vigencia.

---

## E4 — Stock, ubicaciones y depósito Móvil

### US-040 — Gestión de ubicaciones
**Como** administrador, **quiero** gestionar ubicaciones de stock (local, depósito, consignación, externo), **para** saber dónde está la mercadería.
- **CA1 — Dado que** necesito una ubicación, **cuando** la creo, **entonces** queda disponible para movimientos y reportes.
- **CA2 — Dado que** una ubicación ya no se usa, **cuando** la inactivo, **entonces** no admite nuevos movimientos pero conserva su historia.

### US-041 — Depósito Móvil como única fuente de ventas
**Como** dueño, **quiero** que el depósito Móvil sea la única fuente de ventas/entregas, **para** controlar qué stock está disponible para facturar.
- **CA1 — Dado que** existe el depósito Móvil, **cuando** se registra una venta, **entonces** el stock se descuenta exclusivamente del Móvil.
- **CA2 — Dado que** un producto está en otro depósito pero no en Móvil, **cuando** intento venderlo, **entonces** el sistema no permite facturarlo (ver US-082).

### US-042 — Consulta de existencias
**Como** usuario, **quiero** consultar la existencia por producto y ubicación, **para** conocer el stock disponible.
- **CA1 — Dado que** hay movimientos registrados, **cuando** consulto un producto, **entonces** veo su cantidad actual por cada ubicación.
- **CA2 — Dado que** soy Empleado, **cuando** consulto existencias, **entonces** veo cantidades pero no costos.

### US-043 — Movimiento de stock entre depósitos
**Como** empleado/administrador/dueño, **quiero** mover stock entre depósitos, **para** abastecer el Móvil y otros depósitos.
- **CA1 — Dado que** hay stock en el depósito origen, **cuando** registro un movimiento al depósito destino, **entonces** disminuye el origen y aumenta el destino por la cantidad movida.
- **CA2 — Dado que** soy Empleado, **cuando** muevo stock, **entonces** opero cantidades sin ver ni elegir costos.

### US-045 — Stock resultante tras el movimiento
**Como** quien mueve stock, **quiero** ver el stock resultante en cada depósito inmediatamente, **para** conocer la disponibilidad sin otra consulta.
- **CA1 — Dado que** confirmo un movimiento, **cuando** finaliza, **entonces** el sistema muestra el nuevo stock de origen y destino.

### US-046 — Alerta por exceso de cantidad
**Como** quien mueve stock, **quiero** una alerta si intento mover más de lo disponible, **para** evitar movimientos inválidos.
- **CA1 — Dado que** el origen tiene cantidad C, **cuando** intento mover una cantidad mayor a C, **entonces** el sistema alerta y no permite completar el movimiento.

### US-048 — Ajuste manual de stock
**Como** administrador, **quiero** ajustar stock manualmente con motivo, **para** corregir diferencias.
- **CA1 — Dado que** detecto una diferencia, **cuando** registro un ajuste con motivo, **entonces** el stock se corrige y queda auditado (usuario, fecha, motivo).

### US-049 — Stock mínimo y alerta
**Como** administrador, **quiero** definir stock mínimo y recibir alertas, **para** reponer a tiempo.
- **CA1 — Dado que** un producto tiene mínimo M, **cuando** su existencia cae por debajo de M, **entonces** el sistema genera una alerta de stock bajo.

### US-050 — Conteo físico
**Como** administrador, **quiero** registrar conteos físicos y compararlos con el teórico, **para** detectar diferencias.
- **CA1 — Dado que** realizo un conteo, **cuando** ingreso las cantidades reales, **entonces** el sistema muestra la diferencia contra el stock teórico.
- **CA2 — Dado que** hay diferencias, **cuando** confirmo el conteo, **entonces** se registran y quedan en el historial de diferencias.

---

## E5 — Costeo por lotes y movimientos pendientes

### US-051 — Lotes de costo por producto
**Como** administrador, **quiero** que cada ingreso de mercadería genere un lote con su costo, **para** poder rastrear de qué costo proviene el stock.
- **CA1 — Dado que** recibo una compra, **cuando** se registra la recepción, **entonces** se crea un lote con la cantidad y el costo unitario de esa recepción.
- **CA2 — Dado que** un producto tiene varios lotes, **cuando** consulto su stock, **entonces** veo la composición por lote (Administrador/Dueño).

### US-052 — Movimiento auto-finalizado con un solo lote
**Como** sistema, **quiero** finalizar automáticamente el movimiento cuando el producto tiene un único lote/costo, **para** no generar pendientes innecesarios.
- **CA1 — Dado que** un producto movido tiene un único lote posible, **cuando** el Empleado completa el movimiento, **entonces** el sistema lo finaliza automáticamente asignando ese costo.

### US-053 — Movimiento pendiente con múltiples lotes
**Como** sistema, **quiero** dejar pendiente el movimiento cuando hay más de un lote/costo posible, **para** que el costo correcto lo defina quien tiene permiso.
- **CA1 — Dado que** un producto movido tiene más de un lote posible, **cuando** el Empleado completa el movimiento, **entonces** la cantidad se mueve pero el movimiento queda en estado "pendiente de asignación de costo".
- **CA2 — Dado que** un movimiento está pendiente, **cuando** se consultan reportes de costo/utilidad, **entonces** ese movimiento figura como pendiente y no afecta cálculos de costo hasta resolverse.

### US-054 — Notificación de movimientos pendientes
**Como** administrador, **quiero** recibir una notificación 🔔 de los movimientos pendientes de lote, **para** resolverlos a tiempo.
- **CA1 — Dado que** se genera un movimiento pendiente, **cuando** ingreso al sistema como Administrador/Dueño, **entonces** veo una notificación con los movimientos pendientes de asignación de costo.

### US-055 — Resolución de lote/costo
**Como** administrador o dueño, **quiero** asignar el lote/costo a un movimiento pendiente, **para** finalizarlo correctamente.
- **CA1 — Dado que** un movimiento está pendiente, **cuando** asigno el lote/costo correspondiente, **entonces** el movimiento pasa a "finalizado" y el costo queda registrado.
- **CA2 — Dado que** resolví el movimiento, **cuando** se recalculan costos/utilidad, **entonces** el sistema ya considera el costo asignado.

### US-056 — Venta pendiente de cálculo de utilidad
**Como** sistema, **quiero** dejar la venta pendiente de cálculo de utilidad al facturar desde Móvil, **para** que el costo lo defina quien tiene permiso.
- **CA1 — Dado que** se factura una venta desde Móvil, **cuando** se confirma, **entonces** la venta queda en estado "utilidad pendiente de cálculo".
- **CA2 — Dado que** una venta tiene utilidad pendiente, **cuando** la registró un Empleado, **entonces** la venta se completa igual pero su utilidad no se calcula hasta que la resuelva Administrador/Dueño.

### US-057 — Notificación de utilidad pendiente
**Como** administrador o dueño, **quiero** recibir una notificación 🔔 de las facturas con utilidad pendiente, **para** resolverlas.
- **CA1 — Dado que** existe una venta con utilidad pendiente, **cuando** ingreso como Administrador/Dueño, **entonces** veo la notificación con esas facturas.

### US-058 — Cálculo de utilidad
**Como** administrador o dueño, **quiero** especificar el costo de la venta pendiente y calcular su utilidad, **para** cerrar el cálculo (venta − costo).
- **CA1 — Dado que** una venta tiene utilidad pendiente, **cuando** especifico el costo según los lotes en Móvil, **entonces** el sistema calcula utilidad = venta − costo y la venta deja de estar pendiente.
- **CA2 — Dado que** resolví la utilidad, **cuando** consulto reportes de utilidad, **entonces** la venta figura con su utilidad calculada.

---

## E7 — Compras, recepción y pagos a proveedores

### US-070 — Registro de compra
**Como** administrador, **quiero** registrar una compra con proveedor, productos, cantidades, costos y forma de pago, **para** documentar el acuerdo comercial.
- **CA1 — Dado que** quiero registrar una compra, **cuando** ingreso proveedor, ítems, cantidades y costo unitario, **entonces** la compra queda registrada en estado "registrada".
- **CA2 — Dado que** registro una compra, **cuando** la confirmo, **entonces** el stock no se modifica hasta que haya recepción.

### US-071 — Compra pendiente de retiro
**Como** administrador, **quiero** que una compra pueda quedar pendiente de retiro total o parcial, **para** reflejar que la mercadería aún no ingresó.
- **CA1 — Dado que** una compra no fue retirada, **cuando** la consulto, **entonces** figura como pendiente de retiro (total o parcial).

### US-072 — Recepción de mercadería
**Como** administrador, **quiero** registrar recepciones parciales o totales, **para** aumentar el stock solo por lo recibido y generar el lote de costo.
- **CA1 — Dado que** una compra tiene ítems pendientes, **cuando** registro una recepción parcial, **entonces** el stock aumenta solo por lo recibido y el resto queda pendiente.
- **CA2 — Dado que** registro una recepción, **cuando** se confirma, **entonces** se genera el lote de costo correspondiente a esa recepción.
- **CA3 — Dado que** se recibió todo lo pendiente, **cuando** se confirma la última recepción, **entonces** la compra pasa a "recibida total".

### US-074 — Pagos y deuda con proveedor
**Como** administrador, **quiero** registrar pagos parciales o totales, **para** reflejar la deuda con el proveedor.
- **CA1 — Dado que** una compra no está totalmente pagada, **cuando** registro un pago parcial, **entonces** disminuye la deuda y la compra figura "pagada parcial".
- **CA2 — Dado que** registro un pago, **cuando** se confirma, **entonces** impacta la cuenta financiera de origen.

### US-076 — Anulación de compra
**Como** administrador, **quiero** anular una compra con reversión y auditoría, **para** corregir errores sin perder trazabilidad.
- **CA1 — Dado que** una compra tiene recepciones y pagos, **cuando** la anulo, **entonces** se revierten stock, lotes y pagos asociados, y queda registrada la anulación (usuario, fecha, motivo).

---

## E8 — Ventas, entrega y cobros

### US-080 — Registro de venta
**Como** empleado, administrador o dueño, **quiero** registrar una venta con uno o más productos y precio editable, **para** facturar al cliente.
- **CA1 — Dado que** selecciono un cliente y productos, **cuando** confirmo la venta, **entonces** queda registrada con los precios aplicados.
- **CA2 — Dado que** soy Empleado, **cuando** registro la venta, **entonces** puedo completarla sin ver costos ni utilidad.

### US-081 — Múltiples medios de pago
**Como** vendedor, **quiero** cobrar una venta con varios medios de pago, **para** reflejar cómo pagó el cliente.
- **CA1 — Dado que** una venta tiene un total, **cuando** asigno varios medios de pago, **entonces** la suma de los medios debe coincidir con lo cobrado y cada medio impacta su cuenta.

### US-082 — Bloqueo de venta sin stock en Móvil
**Como** dueño, **quiero** que no se pueda facturar si no hay stock del producto en Móvil, **para** que todo lo facturado se entregue.
- **CA1 — Dado que** un producto no tiene stock suficiente en Móvil, **cuando** intento agregarlo o confirmar la venta, **entonces** el sistema bloquea la operación e informa la falta de stock.
- **CA2 — Dado que** todos los productos tienen stock en Móvil, **cuando** confirmo la venta, **entonces** la venta se registra y el stock de Móvil se descuenta.

### US-083 — Snapshot de precio
**Como** dueño, **quiero** que el precio aplicado quede congelado en cada ítem, **para** que cambios futuros de catálogo no alteren ventas pasadas.
- **CA1 — Dado que** registro una venta con un precio, **cuando** luego cambia el precio del producto, **entonces** la venta histórica conserva el precio original aplicado.

### US-084 — Estados de venta
**Como** sistema, **quiero** manejar los estados de venta (borrador, cerrada, entregada, anulada, en consignación), **para** reflejar su situación.
- **CA1 — Dado que** una venta se confirma con entrega, **cuando** se cierra, **entonces** pasa a "entregada".
- **CA2 — Dado que** una venta se anula, **cuando** se confirma la anulación, **entonces** pasa a "anulada" y se revierten sus efectos.

### US-085 — Entrega total y descuento de stock
**Como** vendedor, **quiero** que la entrega sea total al cerrar la venta, descontando stock de Móvil, **para** que no queden pendientes de entrega.
- **CA1 — Dado que** se confirma la venta, **cuando** se cierra, **entonces** el stock entregado se descuenta de Móvil por la cantidad vendida.

### US-086 — Registro de cobro
**Como** vendedor, **quiero** registrar el cobro (total o parcial) de una venta o saldo de arrastre, **para** reflejar el ingreso de dinero.
- **CA1 — Dado que** una venta tiene saldo, **cuando** registro un cobro, **entonces** disminuye el saldo del cliente y se registra el ingreso.
- **CA2 — Dado que** soy Empleado, **cuando** registro el cobro, **entonces** lo completo sin ver los saldos de las cuentas.

### US-087 — Impacto de cobros en cuentas
**Como** administrador, **quiero** que los cobros impacten las cuentas financieras y la cuenta corriente, **para** mantener saldos correctos.
- **CA1 — Dado que** se registra un cobro, **cuando** se confirma, **entonces** la cuenta financiera correspondiente aumenta su saldo (visible solo a Administrador/Dueño).

### US-088 — Comprobantes internos
**Como** vendedor, **quiero** emitir remitos y recibos internos, **para** documentar la entrega y el cobro.
- **CA1 — Dado que** se cierra una venta con entrega, **cuando** la confirmo, **entonces** puedo generar el remito interno.
- **CA2 — Dado que** se registra un cobro, **cuando** lo confirmo, **entonces** puedo generar el recibo interno.

### US-089 — Anulación de venta
**Como** administrador, **quiero** anular una venta con reversión completa y auditoría, **para** corregir errores.
- **CA1 — Dado que** una venta está cerrada, **cuando** la anulo, **entonces** se revierten stock, cobros, deuda y utilidad asociados.
- **CA2 — Dado que** anulo una venta, **cuando** se confirma, **entonces** queda registro de auditoría (usuario, fecha, motivo) y la venta pasa a "anulada".

---

## E10 — Cuentas financieras, transferencias y caja

### US-100 — Gestión de cuentas financieras
**Como** administrador, **quiero** gestionar cuentas (efectivo, Mercado Pago, bancos, tarjetas, otras), **para** registrar el dinero del negocio.
- **CA1 — Dado que** necesito una cuenta, **cuando** la creo, **entonces** queda disponible para recibir y emitir movimientos.

### US-101 — Saldo por cuenta en tiempo real
**Como** administrador o dueño, **quiero** ver el saldo de cada cuenta en tiempo real, **para** conocer la posición de dinero.
- **CA1 — Dado que** una cuenta tiene movimientos, **cuando** la consulto, **entonces** veo su saldo actualizado.
- **CA2 — Dado que** soy Empleado, **cuando** intento ver saldos, **entonces** el sistema no los muestra.

### US-102 — Transferencias entre cuentas
**Como** administrador, **quiero** transferir dinero entre cuentas, **para** mover fondos internos.
- **CA1 — Dado que** dos cuentas existen, **cuando** registro una transferencia, **entonces** disminuye el saldo de origen y aumenta el de destino por igual importe.
- **CA2 — Dado que** registro una transferencia, **cuando** se confirma, **entonces** quedan dos movimientos vinculados (salida y entrada) auditables.

### US-103 — Movimientos de cuenta tipados
**Como** administrador, **quiero** que cada movimiento tenga tipo y referencia a su operación origen, **para** auditar de dónde viene cada importe.
- **CA1 — Dado que** se genera un movimiento (cobro, pago, gasto, transferencia, etc.), **cuando** lo consulto, **entonces** veo su tipo y la operación que lo originó.

---

## Pendiente de detallar (próxima iteración)

- E3 — Entidades comerciales (US-030..037)
- E6 — Cajas retornables (US-060..064)
- E9 — Consignación (US-090..092)
- E11 — Gastos y fondos separados (US-110..114)
- E12 — Cierre de caja (US-120..122)
- E13 — Reportes y utilidad (US-130..135)
- E14 — Auditoría y seguridad operativa (US-140..143)
</content>
</invoke>
