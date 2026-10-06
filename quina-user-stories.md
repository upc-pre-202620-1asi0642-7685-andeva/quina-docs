# Quina - Historias de Usuario (User Stories para SCRUM)

Este documento define el conjunto exhaustivo de Historias de Usuario (_User
Stories_) para el desarrollo ágil de **Quina** mediante el marco de trabajo
**SCRUM**. Cubre el 100 % de las características definidas en la
[documentación maestra de Quina](quina-documentation.md), organizadas en cinco
ciclos de iteración (_Sprints_).

Cada historia sigue el estándar **3 C's** (Tarjeta, Conversación y Confirmación),
cumple los criterios **INVEST** (Independiente, Negociable, Valiosa, Estimable,
Pequeña y Testeable) y se articula con la
[priorización de características](feature-prioritization.md), la
[estrategia de precios](pricing-strategy.md) y la
[arquitectura webapp](webapp-documentation/quina-webapp-documentation.md).

---

## 1. Roles de usuario del sistema

Para contextualizar el valor de cada historia, se definen los cuatro actores
principales:

- **Administrador de Plataforma (`Platform Admin`):** gestiona el alta y baja de
  los comercios (_tenants_) y audita el estado general del sistema.
- **Dueño del Negocio / Comerciante (`Store Owner` - "Don Lucho"):** fija precios,
  pacta condiciones de crédito, registra compras, gestiona empleados y consulta la
  liquidez de su cartera.
- **Cajero / Ayudante de Mostrador (`Store Employee` - "Brayan"):** atiende en
  el mostrador, despacha ventas rápidas al contado y al fiado, y registra cobros.
- **Cliente Vecino / Casero (`Customer` - "Doña Rosa"):** adquiere productos al
  fiado, consulta su estado de cuenta gratuito y recibe avisos de pago.

---

## 2. Mapa general de Sprints y Épicas

Las 23 historias de usuario se distribuyen en cinco Sprints de desarrollo:

```
[ Sprint 1: Cimientos y Seguridad ] ──► Multi-tenant, Login seguro Turnstile, Catálogo dual, Clientes Ley 29733, RBAC
               │
[ Sprint 2: Ventas y Finanzas ] ──────► Condiciones de crédito, Venta al contado, Despacho fiado 3 toques, Método Francés
               │
[ Sprint 3: Corte y Cobranza ] ───────► Liquidación mensual, Avisos de cargo 48h, Recordatorios preventivos, Mora, Portal Vecino
               │
[ Sprint 4: Inventario y Auditoría ] ─► Compras con facturas en Supabase, Kardex costo promedio, Stock mínimo, Bitácora de auditoría
               │
[ Sprint 5: Analítica, PWA y SaaS ] ──► Panel de cartera, Reportes con IA Groq, Asistente Nuevo RUS, PWA offline, Mercado Pago
```

---

## Sprint 1: Cimientos, Seguridad, Catálogo y Roles

### US01: Alta de comercio y configuración multi-tienda
- **Épica:** Identidad y Configuración.
- **Prioridad:** Must Have.
- **Estimación:** 5 Story Points.

**Tarjeta (Card):**
> **Como** Administrador de Plataforma,  
> **quiero** registrar una nueva tienda en el sistema con sus datos comerciales y credenciales iniciales,  
> **para que** el comercio comience a operar de forma aislada y segura en la plataforma.

**Conversación (Conversation):**
Cada tienda debe crearse como un inquilino independiente (_tenant_). Los datos
incluyen nombre comercial, giro de negocio (bodega, botica, carnicería), RUC o
DNI del dueño, correo de contacto y dirección física. El sistema debe asociar un
identificador único inmutable `store_id` que se inyectará en todas las consultas
de la base de datos para garantizar aislamiento estricto de información.

**Criterios de Aceptación (Confirmation):**
1. El formulario de alta valida campos obligatorios: nombre comercial, documento de
   identidad (RUC de 11 dígitos o DNI de 8 dígitos) y correo electrónico válido.
2. Al crearse la tienda, se genera automáticamente la cuenta del usuario dueño con
   rol `STORE_OWNER` y contraseña temporal segura enviada por correo.
3. El sistema asigna un `store_id` único y crea el registro de suscripción inicial
   en el plan gratuito (Plan Casero).
4. No se permite registrar dos tiendas con el mismo RUC o correo de administrador.
5. El administrador puede suspender (_dar de baja lógica_) una tienda, impidiendo el
   inicio de sesión de todos sus usuarios asociados sin eliminar su historial.
6. La operación registra un log de auditoría con fecha, hora y usuario ejecutor.

---

### US02: Autenticación segura, verificación OTP, Google OAuth y bloqueo por fuerza bruta
- **Épica:** Seguridad y Autenticación.
- **Prioridad:** Must Have.
- **Estimación:** 5 Story Points.

**Tarjeta (Card):**
> **Como** Usuario del Sistema (Dueño, Empleado o Cliente),  
> **quiero** iniciar sesión con credenciales seguras, código OTP, cuenta de Google o enlace de recuperación,  
> **para que** mi cuenta esté blindada contra accesos no autorizados y ataques automatizados sin complicaciones.

**Conversación (Conversation):**
Cumple estrictamente con el requisito de autenticación segura del enunciado.
Soporta dos modalidades de acceso: credenciales tradicionales (correo y contraseña
segura con Argon2id) y acceso federado con Google OAuth 2.0 en un solo toque.
Incluye verificación de correo con código OTP de 6 dígitos numéricos, protección
contra bots con Cloudflare Turnstile (100 % gratuito y no intrusivo), política
anti-fuerza bruta con bloqueo automático temporal tras 5 fallos consecutivos y
recuperación de contraseña mediante enlace tokenizado de un solo uso.

**Criterios de Aceptación (Confirmation):**
1. La pantalla de login incluye acceso tradicional, botón **"Continuar con Google"**
   (OAuth 2.0 gratuito) y verificación perimetral de **Cloudflare Turnstile** invisible
   o de un solo clic.
2. El registro inicial o cambio de correo despacha un código OTP numérico de 6
   dígitos vía Resend con expiración de 15 minutos y tasa limitada (máximo 5
   intentos y 60 segundos entre reenvíos).
3. Las contraseñas comerciales se almacenan mediante hash seguro Argon2id (o bcrypt)
   y validan una longitud mínima de 8 caracteres con mayúsculas, números y símbolos.
4. Si un usuario ingresa 5 contraseñas erróneas consecutivas, la cuenta se bloquea
   automáticamente durante 15 minutos (`locked_until`) y se le notifica por correo.
5. El flujo *"Olvidé mi contraseña"* genera un token criptográfico seguro de un solo
   uso con expiración de 30 minutos enviado por correo, dirigiendo a
   `/auth/reset-password?token=...` sin revelar si el correo existe o no en el sistema.
6. La sesión emite un token JWT en cookie `HttpOnly`, `SameSite=Lax` y `Secure`, y
   el cierre de sesión invalida la cookie inmediatamente.

---

### US03: Catálogo de productos y servicios con precios duales e historial de cambios
- **Épica:** Catálogo e Inventario.
- **Prioridad:** Must Have.
- **Estimación:** 5 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** registrar mis productos y servicios especificando precio al contado y precio al crédito,  
> **para que** en el mostrador se cobre el importe correcto según la modalidad de venta sin alterar ventas pasadas.

**Conversación (Conversation):**
Los pequeños negocios manejan frecuentemente dos precios: uno al contado en
efectivo/Yape y un precio de lista superior al fiado. El dueño puede modificar
estos precios cuando lo desee. Las ventas ya realizadas conservan el precio
histórico con el que se despacharon y cada cambio queda en
`product_price_history`.

**Criterios de Aceptación (Confirmation):**
1. El formulario solicita nombre, descripción, marca, unidad de medida (código
   de unidad), precio al contado (`cash_price`) y precio al crédito (`credit_price`).
2. Permite marcar si el ítem es un producto físico (con stock) o un servicio (sin
   stock), y si admite financiamiento en cuotas (`allows_installments`).
3. Permite cargar una imagen del producto que se almacena en el bucket público de
   Supabase Storage.
4. Si el dueño modifica alguno de los precios, el sistema guarda de forma atómica
   el precio anterior, el nuevo precio, la fecha y el usuario en el historial.
5. Las ventas registradas previamente no modifican su subtotal ante cambios
   posteriores en el catálogo.
6. El sistema alerta visualmente si el precio al crédito es menor que el de contado.

---

### US04: Registro de clientes con consentimiento de la Ley N.° 29733
- **Épica:** Identidad y Cumplimiento Legal.
- **Prioridad:** Must Have.
- **Estimación:** 3 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** registrar a mis clientes vecinos capturando sus datos generales y su consentimiento de privacidad,  
> **para que** sus compras a crédito queden respaldadas legalmente bajo la Ley de Protección de Datos Personales.

**Conversación (Conversation):**
Para cumplir con la Ley N.° 29733 y los requisitos de la rúbrica, el registro del
cliente recaba nombre completo, DNI, dirección, teléfono celular y correo
electrónico. El sistema presenta una casilla obligatoria de consentimiento
informado donde se deja constancia digital de la autorización del vecino para la
gestión de su cuenta corriente.

**Criterios de Aceptación (Confirmation):**
1. El registro exige DNI válido (8 dígitos numéricos), nombres, apellidos, teléfono
   móvil y correo electrónico.
2. Muestra un enlace legible a la Política de Privacidad adaptada al comercio.
3. El botón de guardado permanece deshabilitado hasta marcar la casilla de
   consentimiento expreso.
4. La base de datos guarda la fecha exacta, hora y versión de la política en la
   tabla de auditoría de consentimiento.
5. No permite registrar dos clientes con el mismo DNI dentro de la misma tienda.
6. Se envía un correo de bienvenida automático al vecino informándole de su registro.

---

### US05: Configuración de roles y empleados por tienda (RBAC)
- **Épica:** Identidad y Seguridad.
- **Prioridad:** Should Have.
- **Estimación:** 5 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** crear cuentas para mis empleados asignándoles roles con permisos limitados,  
> **para que** puedan despachar en el mostrador sin tener acceso a mis costos, ganancias ni reportes financieros.

**Conversación (Conversation):**
Don Lucho necesita que su ayudante Brayan registre ventas al contado y al fiado,
pero no quiere que modifique precios del catálogo, ni que vea los reportes de
ganancias o acceda a la configuración tributaria. El sistema implementa RBAC con
roles `STORE_OWNER` y `STORE_EMPLOYEE`.

**Criterios de Aceptación (Confirmation):**
1. El dueño puede invitar a un nuevo empleado ingresando su nombre, DNI y correo.
2. El rol `STORE_EMPLOYEE` tiene permisos habilitados para: registrar ventas, buscar
   clientes, consultar stock disponible y registrar pagos de clientes.
3. El rol `STORE_EMPLOYEE` tiene bloqueado el acceso a: reportes financieros,
   márgenes de ganancia, compras a proveedores, edición de precios y asistente RUS.
4. Las Server Actions verifican el rol del usuario en la sesión antes de ejecutar
   cualquier mutación protegida.
5. El dueño puede desactivar la cuenta de un empleado en cualquier momento.
6. Cada venta registrada guarda el ID del empleado que atendió para auditoría.

---

## Sprint 2: Ventas en Mostrador y Motor Financiero

### US06: Configuración de condiciones de crédito por vecino
- **Épica:** Crédito y Cuentas Corrientes.
- **Prioridad:** Must Have.
- **Estimación:** 5 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** definir las condiciones financieras personalizadas para cada vecino (límite, tasa, corte y pago),  
> **para que** cada cuenta corriente opere bajo reglas claras acordadas previamente y sin ambigüedades.

**Conversación (Conversation):**
Para cada cliente, Don Lucho asigna: límite de crédito en soles (ej. S/ 150),
moneda (PEN/USD), plazo máximo en meses (ej. 3 meses), tasa de interés
compensatoria (nominal o efectiva, ej. TNA 24 % o TEA 20 %), tasa moratoria
pactada, día del mes de corte (ej. día 20) y día del mes de pago pactado (ej. día
26).

**Criterios de Aceptación (Confirmation):**
1. Permite seleccionar el tipo de tasa (Nominal o Efectiva) y el período de
   capitalización o vigencia (anual, mensual o diaria).
2. Valida que las tasas no excedan los topes máximos vigentes publicados por el BCRP
   para operaciones fuera del sistema financiero.
3. El límite de crédito debe ser un número mayor o igual a cero.
4. Los días de corte y pago deben ser enteros entre 1 y 28.
5. Permite configurar tasa compensatoria del 0 % para clientes de absoluta confianza.
6. Los cambios en las condiciones crediticias solo aplican para compras futuras.

---

### US07: Despacho rápido de ventas al contado con cálculo de vuelto
- **Épica:** Venta en Mostrador.
- **Prioridad:** Must Have.
- **Estimación:** 3 Story Points.

**Tarjeta (Card):**
> **Como** Cajero o Dueño del Negocio,  
> **quiero** registrar ventas al contado cobradas en efectivo o Yape descontando el stock automáticamente,  
> **para que** la caja cuadre al final del día sin generar deuda a ningún cliente.

**Conversación (Conversation):**
En una bodega la mayoría de ventas son al contado. Don Lucho o Brayan seleccionan
los productos, eligen modalidad "Contado", ingresan el monto recibido en efectivo
o marcan Yape/Plin, el sistema calcula el vuelto instantáneamente, descuenta el
stock de los productos en el kardex y registra el ingreso en la caja diaria.

**Criterios de Aceptación (Confirmation):**
1. La venta al contado no requiere asociar obligatoriamente a un cliente registrado
   (admite cliente genérico / anónimo).
2. Aplica automáticamente el precio al contado (`cash_price`) de cada producto.
3. Permite ingresar el monto recibido en efectivo y calcula el vuelto en pantalla.
4. Soporta registrar el medio de pago: Efectivo, Yape, Plin o Tarjeta.
5. Descuenta de inmediato las unidades del stock físico registrando el movimiento en
   el kardex (`SALIDA_VENTA`).
6. Si algún producto no cuenta con stock suficiente, la venta no se procesa.

---

### US08: Despacho veloz de venta al crédito con validación de límite y stock
- **Épica:** Venta en Mostrador.
- **Prioridad:** Must Have.
- **Estimación:** 8 Story Points.

**Tarjeta (Card):**
> **Como** Cajero o Dueño del Negocio,  
> **quiero** registrar una compra fiada en menos de 10 segundos con validación automática de límite y stock,  
> **para que** la atención en mostrador sea tan rápida como el cuaderno y no se entregue mercadería sin saldo disponible.

**Conversación (Conversation):**
En horas punta, se selecciona al cliente vecino, se agregan los productos y se
confirma en 3 toques. El sistema ejecuta una transacción ACID que: 1) verifica
stock, 2) verifica que no supere el crédito disponible (`límite - deuda_actual`),
3) descuenta stock y 4) asienta la venta al precio de crédito (`credit_price`).

**Criterios de Aceptación (Confirmation):**
1. El flujo de selección de cliente, productos y confirmación se completa en un
   máximo de 3 toques/clics en pantalla.
2. Si el importe supera el saldo disponible, el sistema bloquea la venta con un
   aviso: *"Excede el límite disponible por S/ X.XX"*.
3. Si algún producto no tiene stock suficiente, la transacción se cancela y se
   informa qué producto carece de existencias.
4. Aplica el precio de lista al crédito (`credit_price`) de cada producto.
5. Al confirmar, muestra un toast de Sileo y actualiza la deuda mediante UI optimista.
6. Congela el costo promedio vigente (`unit_cost_snapshot`) en cada ítem vendido.

---

### US09: Cálculo de cronograma francés con capitalización en período de gracia
- **Épica:** Motor Financiero.
- **Prioridad:** Must Have.
- **Estimación:** 8 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** que el sistema calcule automáticamente las cuotas fijas bajo el método francés vencido ordinario capitalizando los días de gracia previa,  
> **para que** el vecino cuente con un plan de pagos matemáticamente exacto y formal.

**Conversación (Conversation):**
Cumple estrictamente el punto 4 del enunciado. Si una compra a 3 meses se realiza
el 15 de setiembre, la fecha de corte es el 20 y la de pago es el 26: existen 11
días de gracia total (del 15 al 26). La deuda se capitaliza con la tasa diaria
compensatoria por esos 11 días antes de calcular la cuota fija mensual. El
cronograma arranca en el mes siguiente (26 de octubre, 26 de noviembre y 26 de
diciembre), bajo año comercial de 360 días y meses base de 30 días.

**Criterios de Aceptación (Confirmation):**
1. Calcula los días exactos entre la fecha de compra y la fecha de pago pactada del
   ciclo corriente como período de gracia total.
2. Capitaliza el saldo principal mediante la fórmula financiera correspondiente
   según sea tasa efectiva o nominal.
3. Calcula la cuota periódica uniforme usando la fórmula del método francés
   vencido: $R = S_{cap} \times \left[ \frac{i \times (1+i)^m}{(1+i)^m - 1} \right]$.
4. Genera la tabla de amortización detallando: número de cuota, fecha de vencimiento,
   saldo inicial, cuota fija, interés, amortización y saldo final.
5. La suma de las amortizaciones coincide exactamente con el capital capitalizado
   (tolerancia de redondeo $\pm$ 0.01 soles).
6. El cronograma queda grabado y visible para el dueño y para el cliente.

---

## Sprint 3: Corte, Liquidación y Cobranza Desatendida

### US10: Generación de liquidación en fecha de corte e interés compensatorio
- **Épica:** Crédito y Cuentas Corrientes.
- **Prioridad:** Must Have.
- **Estimación:** 8 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** que en cada fecha de corte el sistema consolide todas las compras del mes y liquide los intereses por días transcurridos,  
> **para que** cada vecino reciba el detalle exacto de lo que debe pagar en su fecha de pago pactada.

**Conversación (Conversation):**
En la fecha de corte, el sistema recorre las cuentas corrientes. Para compras bajo
modalidad de pago único a fin de mes, calcula los intereses compensatorios por los
días transcurridos entre cada compra y la fecha de pago. Las compras hechas después
de la fecha de corte se postergan automáticamente para el siguiente ciclo.

**Criterios de Aceptación (Confirmation):**
1. En la fecha de corte, agrupa todas las compras realizadas antes de la hora límite
   del día de corte.
2. Calcula los intereses individuales aplicando la tasa pactada por los días entre la
   compra y la fecha de pago pactada.
3. Incorpora las cuotas del mes para compras financiadas bajo el método francés.
4. Genera el documento de liquidación consolidando: lista de compras por fecha,
   intereses compensatorios, cuotas del mes y total a pagar.
5. Compras con fecha posterior al corte se excluyen y se asignan al siguiente ciclo.
6. El estado de la cuenta del cliente pasa a `LIQUIDADO_PENDIENTE_PAGO`.

---

### US11: Notificación de cargos por correo con conformidad liviana de 48 horas
- **Épica:** Notificaciones y Respaldo Legal.
- **Prioridad:** Must Have.
- **Estimación:** 5 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** que cada cargo fiado envíe un correo al vecino otorgándole 48 horas para observaciones,  
> **para que** la deuda quede reconocida automáticamente sin discusiones posteriores de "yo no saqué eso".

**Conversación (Conversation):**
Al registrarse una venta fiada, el sistema despacha un correo vía Resend con el
desglose de productos, cantidades, fecha e importe. Indica que el cliente dispone
de 48 horas para reportar cualquier discrepancia; transcurrido ese lapso sin
reclamos, el cargo se considera aceptado de mutuo acuerdo.

**Criterios de Aceptación (Confirmation):**
1. Tras registrar la venta al crédito, encola el envío asíncrono vía Resend sin
   demorar la interfaz.
2. El correo contiene: nombre del comercio, fecha/hora, productos adquiridos con
   precios y saldo deudor acumulado.
3. Incluye el texto legal de 48 horas para observaciones antes de considerarse
   conforme de mutuo acuerdo.
4. Si no se registra objeción en 48 horas, el estado del cargo pasa a
   `CONFORME_TACITO`.
5. Si el correo rebota, marca una alerta en la ficha del cliente en el sistema.
6. Guarda registro inmutable del envío en la tabla de logs de notificación.

---

### US12: Recordatorios automáticos preventivos previos a la fecha de pago
- **Épica:** Cobranza Desatendida.
- **Prioridad:** Should Have.
- **Estimación:** 3 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** que el sistema envíe recordatorios por correo a los vecinos 3 días antes de su fecha de pago,  
> **para que** acudan a cancelar puntualmente sin que yo tenga que cobrarles en persona.

**Conversación (Conversation):**
Al bodeguero le da "roche" cobrar en persona. El sistema asume el rol de cobrador
neutral enviando un correo automático 3 días antes del vencimiento con un tono
amable y respetuoso (*"Estimada Doña Rosa, le recordamos que su fecha de pago es el
26..."*), adjuntando el enlace a su estado de cuenta.

**Criterios de Aceptación (Confirmation):**
1. Un proceso programado verifica diariamente las liquidaciones pendientes cuya fecha
   de pago venza en 3 días calendario.
2. Envía un correo con tono respetuoso y cercano recordando la fecha límite y el
   monto exacto a pagar.
3. Incluye un enlace seguro directo al portal del cliente para revisar el desglose.
4. No envía recordatorios a clientes cuya deuda haya sido cancelada anticipadamente.
5. Registra el envío en el historial del cliente para evitar recordatorios duplicados.
6. Permite al dueño consultar en una pantalla qué recordatorios fueron enviados hoy.

---

### US13: Registro de pagos y liquidación automática de intereses moratorios
- **Épica:** Cobranza y Liquidación.
- **Prioridad:** Must Have.
- **Estimación:** 5 Story Points.

**Tarjeta (Card):**
> **Como** Cajero o Dueño del Negocio,  
> **quiero** registrar el pago total de la deuda liquidando automáticamente los días de mora si paga después de la fecha pactada,  
> **para que** el comercio recupere su capital más el interés moratorio pactado sin cálculos manuales al ojo.

**Conversación (Conversation):**
El cliente acude a pagar. Si lo hace en o antes de la fecha pactada, paga el monto
liquidado. Si paga con atraso, el sistema calcula los días de mora y aplica la tasa
moratoria pactada sobre el saldo impago, desglosándola por separado en el recibo.
No se permiten pagos parciales a la fecha de pago pactada.

**Criterios de Aceptación (Confirmation):**
1. Si el pago se realiza dentro del plazo, no aplica recargos y cancela la deuda del
   ciclo al 100 %.
2. Si paga con atraso, calcula los días de mora: $días\_mora = fecha\_real - fecha\_pactada$.
3. Aplica la tasa moratoria pactada y desglosa: *"Intereses moratorios por X días: S/ Y.YY"*.
4. Rechaza cualquier intento de pago parcial, exigiendo la cancelación total a la fecha.
5. Al registrarse el pago, el saldo deudor se reduce a cero, liberando la línea de
   crédito.
6. Emite comprobante digital de cancelación despachado por correo al vecino.

---

### US14: Portal gratuito y acceso sin contraseñas para el vecino cliente
- **Épica:** Experiencia del Cliente.
- **Prioridad:** Should Have.
- **Estimación:** 5 Story Points.

**Tarjeta (Card):**
> **Como** Cliente Vecino ("Doña Rosa"),  
> **quiero** consultar mi estado de cuenta y cuotas desde mi celular en un solo clic o con mi cuenta de Google sin tener que recordar contraseñas complejas,  
> **para que** conozca mis fechas de pago con total transparencia, sin frustración y mantenga mis cuentas claras.

**Conversación (Conversation):**
Doña Rosa no paga nada por usar Quina y no se le obliga a recordar una contraseña
compleja de 12 caracteres. El acceso está diseñado sin fricción (*passwordless*):
puede ingresar tocando el enlace directo (*Magic Link*) incluido en cada correo de
notificación de compra o aviso de corte, o tocando el botón *"Continuar con Google"*
con el correo Gmail de su celular, o ingresando su DNI para recibir un código OTP de
6 números. Accede a una vista limpia y en letra grande donde revisa: deuda acumulada,
próxima fecha de corte, fecha de pago pactada, cronograma de cuotas y los datos de
pago de su casero (número de Yape o cuenta bancaria).

**Criterios de Aceptación (Confirmation):**
1. El acceso al portal del vecino es 100 % gratuito y no solicita tarjetas bancarias
   ni requiere instalar una aplicación obligatoria.
2. Permite acceso directo de un solo clic mediante **Magic Link** temporal (72 horas)
   enviado en los correos de compra y corte mensual.
3. Permite acceso en un toque con **"Continuar con Google"** (OAuth 2.0) si el correo
   coincide con el registrado por la tienda.
4. Permite acceso sin contraseña ingresando el **DNI** del cliente y validando un
   **código OTP** de 6 dígitos despachado a su correo electrónico.
5. Permite opcionalmente crear una contraseña personalizada permanente si el vecino
   prefiere el método tradicional.
6. La pantalla móvil destaca en formato legible: saldo actual, fecha de corte, fecha
   límite de pago, botón para ver comprobantes de compras y datos de Yape de la bodega.
7. Para compras en cuotas, presenta la tabla francesa con cuotas canceladas y cuotas
   futuras por vencer con desglose de amortización e intereses.
8. Datos transmitidos mediante conexión cifrada HTTPS/TLS gestionada por Cloudflare.

---

## Sprint 4: Inventario Valorizado, Compras y Trazabilidad

### US15: Registro de compras a proveedores con comprobantes adjuntos
- **Épica:** Compras e Inventario.
- **Prioridad:** Should Have.
- **Estimación:** 5 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** registrar las compras que realizo a mis proveedores adjuntando una foto o PDF de la factura o boleta,  
> **para que** mis costos queden respaldados digitalmente y no se traspapelen los comprobantes físicos.

**Conversación (Conversation):**
Cuando llega el proveedor, Don Lucho registra la compra: proveedor, productos con
cantidad y costo unitario de compra, y toma una foto de la factura con su celular.
El archivo se sube de forma privada a Supabase Storage y se asocia a la compra.

**Criterios de Aceptación (Confirmation):**
1. Permite registrar datos del comprobante: tipo (Factura, Boleta, Guía), serie y
   número.
2. Permite capturar fotos directamente desde la cámara del celular o subir PDF/imágenes.
3. Valida formatos (JPG, PNG, WEBP, PDF) con tamaño máximo de 5 MB.
4. Guarda el archivo en el bucket privado de Supabase Storage bajo la ruta de la tienda.
5. Solo se visualiza mediante URLs firmadas temporales generadas para usuarios de la
   tienda.
6. Enlaza la compra al proveedor para reportes históricos de abastecimiento.

---

### US16: Kardex inmutable y valoración por costo promedio ponderado
- **Épica:** Inventario y Finanzas.
- **Prioridad:** Should Have.
- **Estimación:** 8 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** que cada entrada o salida de mercadería alimente un kardex inmutable y recalcule el costo promedio ponderado,  
> **para que** conozca en todo momento el valor real de mi almacén y el margen de ganancia de mis ventas.

**Conversación (Conversation):**
Cada compra recalcula el costo promedio ponderado del producto según la fórmula
financiera. Cada venta descuenta stock sin alterar el costo promedio. Ningún
registro del kardex puede ser editado o eliminado; correcciones se procesan
mediante mermas o ajustes de conteo físico.

**Criterios de Aceptación (Confirmation):**
1. Cada compra aprobada recalcula automáticamente el `avg_cost` del producto.
2. Salidas por venta reducen stock registrando `SALIDA_VENTA` con el costo vigente.
3. Prohíbe la modificación o eliminación directa de registros en `inventory_movements`.
4. El módulo de mermas permite registrar productos dañados o vencidos con motivo
   obligatorio.
5. Soporta cantidades con hasta 3 decimales para productos a granel (kilos, litros).
6. El reporte de inventario muestra el valor monetario total: $\sum (stock \times avg\_cost)$.

---

### US17: Alertas automáticas de stock mínimo y reposición
- **Épica:** Inventario y Abastecimiento.
- **Prioridad:** Could Have.
- **Estimación:** 3 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** recibir alertas visuales cuando un producto alcance o baje de su nivel de stock mínimo configurado,  
> **para que** pueda reponer mercadería a tiempo con mis proveedores antes de que se agote.

**Conversación (Conversation):**
En la ficha del producto se fija un umbral de `stock_min`. Cuando una venta reduce
las existencias por debajo de ese número, el sistema destaca el producto en color
de advertencia y lo agrupa en una lista de reposición sugerida para el próximo
pedido.

**Criterios de Aceptación (Confirmation):**
1. Permite configurar el valor numérico de `stock_min` en cada producto.
2. Cuando `stock_current <= stock_min`, muestra un badge visual de alerta en catálogo
   y mostrador.
3. La barra superior muestra un contador con productos en estado crítico de reposición.
4. Permite filtrar el catálogo en un clic para ver únicamente productos bajo mínimo.
5. Ofrece un botón para agregar automáticamente los productos bajo mínimo al pedido
   de compra en borrador.
6. Los servicios quedan exentos de alertas de stock mínimo.

---

### US18: Bitácora de auditoría y trazabilidad inmutable de operaciones
- **Épica:** Seguridad y Auditoría.
- **Prioridad:** Must Have.
- **Estimación:** 5 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio o Administrador,  
> **quiero** contar con una bitácora inmutable de auditoría de todas las operaciones realizadas en el sistema,  
> **para que** quede constancia exacta de quién modificó datos, registró cobros o cambió condiciones de crédito.

**Conversación (Conversation):**
Cubre la exigencia de trazabilidad y auditoría de la rúbrica y las directrices
OWASP. Cada acción crítica (creación de tienda, ajuste de límite de crédito, venta,
pago, cambio de precio de catálogo o ajuste de stock) genera un registro inmutable
en la tabla `audit_logs` con usuario, fecha, IP y valores antes/después.

**Criterios de Aceptación (Confirmation):**
1. La tabla `audit_logs` registra de forma automática e inmutable: `id`, `store_id`,
   `user_id`, `action`, `entity_name`, `entity_id`, `old_values`, `new_values`, `timestamp`.
2. Prohíbe mediante restricciones de base de datos operaciones de `UPDATE` o `DELETE`
   sobre la tabla de auditoría (solo inserción).
3. Permite al dueño filtrar la bitácora por fecha, usuario (ej. ventas hechas por el
   cajero Brayan) o entidad.
4. Registra los cambios en las condiciones crediticias de los vecinos indicando la
   tasa y límite anteriores y los nuevos.
5. Registra los intentos de acceso fallidos bloqueados por el sistema de seguridad.
6. Permite exportar la bitácora de auditoría a formato CSV para revisiones.

---

## Sprint 5: Analítica de Cartera, Asistente Tributario, PWA y SaaS

### US19: Panel de cartera y puntaje interno de riesgo del cliente
- **Épica:** Analítica y Gestión de Riesgo.
- **Prioridad:** Should Have.
- **Estimación:** 5 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** visualizar un panel de cartera con la deuda total en la calle y un puntaje de puntualidad por vecino,  
> **para que** identifique rápidamente a quién puedo ampliarle el límite de crédito y a quién debo frenarle el fiado.

**Conversación (Conversation):**
Responde cuánta plata está en la calle y quién es buen o mal pagador. El sistema
calcula un puntaje de 1 a 100 basado en pagos puntuales frente a días de mora,
clasificando a los clientes en categorías: Verde (Excelente), Amarillo (En riesgo)
y Rojo (Moroso crítico).

**Criterios de Aceptación (Confirmation):**
1. Muestra cuatro métricas clave: Capital Prestado, Total Vencido en Mora, Proyección
   de Cobros del Mes y Clientes Morosos Activos.
2. Cada cliente muestra un indicador visual de puntaje de 1 a 100 calculado
   automáticamente.
3. Descuenta puntos por cada día de mora incurrido en los últimos 6 meses y premia
   cancelaciones en fecha pactada.
4. Sugiere reducir el límite si el cliente cae en categoría Roja en dos ciclos
   consecutivos.
5. Permite ordenar la cartera de mayor a menor deuda pendiente.
6. Gráficos de evolución de cartera construidos con Recharts.

---

### US20: Reportes por rango de fechas con recomendaciones de IA (Groq)
- **Épica:** Analítica e Inteligencia Artificial.
- **Prioridad:** Could Have.
- **Estimación:** 5 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** generar un reporte financiero por rango de fechas y recibir recomendaciones automáticas con IA,  
> **para que** entienda la rentabilidad de mi negocio y tome decisiones comerciales informadas.

**Conversación (Conversation):**
Don Lucho elige un rango de fechas. El sistema calcula: ventas (contado vs fiado),
intereses cobrados, costo de lo vendido y margen bruto. Al presionar "Analizar con
IA", el resumen numérico anónimo se envía a Groq (Llama 3) y devuelve sugerencias
prácticas en lenguaje sencillo, descontando una consulta de su cuota mensual.

**Criterios de Aceptación (Confirmation):**
1. Consolida ventas al contado, compras fiadas, intereses compensatorios y moratorios,
   costo de ventas y margen bruto.
2. Análisis de IA ejecutado con Groq Cloud API en menos de 3 segundos.
3. El prompt de IA solo transmite métricas agregadas anónimas; jamás envía datos
   personales de los vecinos.
4. Valida la cuota mensual de la tienda (Plan Pro: 20 consultas/mes).
5. Las recomendaciones se presentan en tarjetas visuales claras con lenguaje cercano
   al bodeguero peruano.
6. Permite exportar o imprimir el reporte en formato PDF limpio.

---

### US21: Asistente tributario informativo de topes del Nuevo RUS
- **Épica:** Cumplimiento Tributario Informativo.
- **Prioridad:** Could Have.
- **Estimación:** 3 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** que Quina me alerte cuando mis ventas acumuladas del mes se acerquen al tope de mi categoría del Nuevo RUS,  
> **para que** evite sanciones o cambios forzados de régimen tributario por parte de la SUNAT.

**Conversación (Conversation):**
Suma las ventas del mes en curso y, al alcanzar el 85 % del tope de la categoría del
Nuevo RUS (Categoría 1 hasta S/ 5,000 o Categoría 2 hasta S/ 8,000), emite una alerta
preventiva para que el comerciante administre sus ventas. Es puramente informativo.

**Criterios de Aceptación (Confirmation):**
1. Permite seleccionar la categoría tributaria del Nuevo RUS en la configuración.
2. Montos límites de cada categoría configurables desde la administración del sistema.
3. Barra de progreso que compara ventas totales del mes contra el límite de la
   categoría.
4. Al superar el 85 % del tope mensual, activa alerta visual preventiva en pantalla.
5. Incluye descargo visible: *"Información referencial. Quina no constituye asesoría
   ni emite declaraciones ante SUNAT"*.
6. El cálculo reinicia automáticamente a las 00:00 del primer día de cada mes.

---

### US22: Experiencia PWA instalable en móvil y tolerancia a desconexión
- **Épica:** Experiencia de Usuario y Despliegue.
- **Prioridad:** Should Have.
- **Estimación:** 5 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio o Cajero,  
> **quiero** instalar Quina como una aplicación en la pantalla de inicio de mi teléfono celular y consultar datos aunque la señal falle,  
> **para que** la herramienta esté accesible al instante sin abrir navegadores y no se bloquee ante caídas breves de internet.

**Conversación (Conversation):**
La webapp debe configurarse como Progressive Web App (PWA) mediante Service Workers
y Web App Manifest. Permite al comerciante "Instalar Quina" en su Android o iPhone,
abriendo a pantalla completa con icono propio. Los activos estáticos y catálogos se
almacenan en caché para lectura rápida en caso de intermitencia celular.

**Criterios de Aceptación (Confirmation):**
1. Incluye `manifest.json` válido con iconos en múltiples resoluciones, nombre corto
   "Quina" y tema de color configurado.
2. El navegador móvil muestra el banner nativo de instalación ("Añadir a pantalla de
   inicio").
3. Una vez instalada, abre en modo `standalone` a pantalla completa sin barra de
   direcciones del navegador.
4. En caso de pérdida de conexión a internet, la app muestra una pantalla amigable de
   aviso y permite consultar en solo lectura el catálogo y deudas cacheadas.
5. Las operaciones de escritura (ventas, cobros) exigen conexión activa para
   garantizar consistencia financiera estricta y evitar desfases.
6. El tamaño total de los activos cacheados iniciales no supera los 5 MB para carga
   rápida en redes 3G/4G.

---

### US23: Facturación y suscripción SaaS del comercio vía Mercado Pago
- **Épica:** Facturación y Monetización B2B.
- **Prioridad:** Should Have.
- **Estimación:** 5 Story Points.

**Tarjeta (Card):**
> **Como** Dueño del Negocio,  
> **quiero** suscribirme al Plan Bodega Pro mediante Mercado Pago pagando con tarjeta o Yape en soles,  
> **para que** active clientes de crédito ilimitados, múltiples usuarios y almacenamiento de comprobantes.

**Conversación (Conversation):**
Cuando el comerciante supera los 15 clientes del Plan Casero gratuito o desea
habilitar el kardex avanzado y recomendaciones de IA, ingresa a la sección de
planes y selecciona el Plan Bodega Pro (S/ 29/mes o S/ 279/año). Paga mediante
Mercado Pago y su cuenta se actualiza automáticamente a Pro.

**Criterios de Aceptación (Confirmation):**
1. Muestra con transparencia los planes: Plan Casero (Gratis), Plan Bodega Pro
   (S/ 29/mes) y Plan Negocio Plus (S/ 69/mes).
2. Checkout seguro de Mercado Pago con soporte de tarjetas peruanas y billeteras.
3. Webhook de Mercado Pago procesa la notificación de pago exitoso de forma
   idempotente, actualizando el plan en Supabase PostgreSQL.
4. Si el pago falla, otorga período de gracia de 5 días hábiles notificando por correo.
5. Permite descargar comprobantes de pago de la suscripción desde el panel.
6. Permite cancelar la renovación automática conservando funciones Pro hasta el fin
   del ciclo.

---

## 3. Matriz de cobertura integral de características (`quina-documentation.md`)

| Característica en `quina-documentation.md` | Historias de Usuario de cobertura |
| :--- | :--- |
| **Gestión de tiendas (Multi-tenant)** | US01 |
| **Autenticación segura y captcha Turnstile** | US02 |
| **Catálogo de productos/servicios y precios duales** | US03 |
| **Historial de cambios de precios (`product_price_history`)** | US03 |
| **Clientes y consentimiento Ley N.° 29733** | US04 |
| **Roles de usuario y control de accesos (RBAC)** | US05 |
| **Condiciones de crédito personalizadas por cliente** | US06 |
| **Venta al contado con cálculo de vuelto** | US07 |
| **Venta al crédito veloz en 3 toques** | US08 |
| **Validación de límite de crédito no sobrepasado** | US08 |
| **Método francés vencido ordinario con gracia capitalizada** | US09 |
| **Liquidación en fecha de corte e interés compensatorio** | US10 |
| **Notificación de cargos con conformidad liviana (48 h)** | US11 |
| **Recordatorios automáticos preventivos de pago** | US12 |
| **Registro de pagos y liquidación de mora por atraso** | US13 |
| **Portal gratuito de consulta para el vecino** | US14 |
| **Compras a proveedores con comprobantes en Supabase Storage** | US15 |
| **Inventario valorizado, kardex y costo promedio ponderado** | US16 |
| **Alertas automáticas de stock mínimo** | US17 |
| **Bitácora de auditoría y trazabilidad inmutable** | US18 |
| **Panel de cartera y puntaje interno de riesgo** | US19 |
| **Reportes por rango de fechas con recomendaciones IA (Groq)** | US20 |
| **Asistente tributario informativo del Nuevo RUS** | US21 |
| **Aplicación instalable (PWA) y modo resiliente** | US22 |
| **Suscripción SaaS del negocio con Mercado Pago** | US23 |
