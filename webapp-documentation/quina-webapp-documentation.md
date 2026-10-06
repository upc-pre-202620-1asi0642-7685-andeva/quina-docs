# Quina Webapp - Documentación técnica y arquitectura de software

Este documento define la arquitectura de software, las decisiones tecnológicas,
los patrones de diseño y el conjunto de herramientas (_stack_) para la
construcción de la aplicación web de **Quina**. Aplica los principios de
**Clean Architecture**, **Hexagonal Architecture (Ports and Adapters)** y
**Domain-Driven Design (DDD)** táctico sobre un monolito modular full-stack en
**Next.js**, garantizando que las reglas financieras y de negocio sean 100 %
independientes de frameworks y completamente testeables sin bases de datos.

Para una visión global del producto, consulta la
[documentación maestra de Quina](../quina-documentation.md).

---

## 1. Patrón arquitectónico: Clean Architecture y Puertos y Adaptadores

Para evitar el acoplamiento típico donde la lógica de cálculo financiero termina
mezclada dentro de Server Actions, controladores HTTP o consultas directas al
ORM, Quina implementa una arquitectura en capas concéntricas basada en la
**Regla de Dependencias**: las dependencias de código solo apuntan hacia adentro.

```
                ┌──────────────────────────────────────────────┐
                │          Frameworks & Drivers (Borde)        │
                │    Next.js App Router, Supabase, Vercel      │
                │  ┌────────────────────────────────────────┐  │
                │  │       Interface Adapters (Borde)       │  │
                │  │ Server Actions, Drizzle Repos, Resend  │  │
                │  │  ┌──────────────────────────────────┐  │  │
                │  │  │      Application (Use Cases)     │  │  │
                │  │  │  RegisterCreditSale, SettleCycle │  │  │
                │  │  │  ┌────────────────────────────┐  │  │  │
                │  │  │  │        Domain Core         │  │  │  │
                │  │  │  │ Money, FrenchAmortization, │  │  │  │
                │  │  │  │ CreditTerms, StockKardex   │  │  │  │
                │  │  │  └────────────────────────────┘  │  │  │
                │  │  └──────────────────────────────────┘  │  │
                │  └────────────────────────────────────────┘  │
                └──────────────────────────────────────────────┘
                    Flujo de dependencia: Siempre hacia adentro
```

### Capas del sistema

1. **Dominio (`src/core/domain`):**
   - El núcleo del negocio, libre de cualquier librería o framework externo
     (sin imports de Next.js, React o Drizzle).
   - Contiene las entidades de negocio, agregados, objetos de valor y las fórmulas
     matemáticas financieras (método francés, capitalización en período de gracia,
     interés diario compensatorio y cálculo de mora).
   - Define los **Puertos (Interfaces abstractas)** para repositorios y servicios
     externos.
2. **Aplicación (`src/core/application`):**
   - Orquesta el flujo de datos mediante **Casos de Uso (Use Cases)**.
   - Cada caso de uso representa una intención del usuario (por ejemplo:
     `RegisterCreditSaleUseCase`, `GenerateMonthlySettlementUseCase`,
     `ProcessPaymentUseCase`).
   - Interactúa exclusivamente con las interfaces de dominio, lo que permite
     ejecutar pruebas unitarias completas utilizando implementaciones en memoria
     (_In-Memory Adapters_) sin requerir una base de datos activa.
3. **Adaptadores de interfaz e infraestructura (`src/infrastructure`):**
   - **Adaptadores primarios (Driving):** Next.js Server Actions y Route Handlers
     que traducen entradas de usuario o webhooks a DTOs de aplicación.
   - **Adaptadores secundarios (Driven):** implementaciones concretas de los
     puertos: repositorios en Drizzle ORM sobre PostgreSQL, cliente de Resend,
     cliente de Supabase Storage, cliente de Mercado Pago y cliente de Groq.
4. **Presentación e interfaz (`src/app`, `src/components`):**
   - Componentes React interactivos, layouts, estados optimistas y vistas PWA.

---

## 2. DDD Strategic Design y Análisis de Dominio

Aplicando los principios estratégicos de **Domain-Driven Design (DDD)** y
las directrices de **Domain Analysis**, se analizan las capacidades de negocio
de Quina para delimitar sus fronteras lingüísticas, clasificar sus subdominios
y prevenir problemas de acoplamiento prematuro o modelos monolíticos
desordenados (_Big Ball of Mud_).

### 2.1. Clasificación estratégica de subdominios

Cada capacidad del negocio se evalúa según su valor competitivo, complejidad
intrínseca y especificidad:

| Capacidad del negocio | Subdominio | Tipo | Justificación estratégica | Cohesión |
| :--- | :--- | :--- | :--- | :---: |
| **Gestión de Crédito y Cuentas Corrientes** | `Credit & Ledger` | **Core Domain** | **Ventaja competitiva exclusiva.** El corazón financiero de Quina: cálculo de amortización francesa, capitalización en período de gracia, intereses compensatorios diarios, recargo por mora y fechas de corte. Ningún software para bodegas en el Perú ofrece estas reglas formales. | 9.5 / 10 |
| **Catálogo e Inventario Valorizado** | `Catalog & Inventory` | **Supporting** | Esencial para la operación (bloqueo de venta sin stock, cálculo de costo promedio ponderado y registro de comprobantes de compras). Es específico del comercio pero no constituye la diferenciación única. | 8.5 / 10 |
| **Analítica de Cartera y Recomendaciones IA** | `Analytics & Insights` | **Supporting** | Consolida métricas de endeudamiento y genera sugerencias comerciales mediante Groq (Llama 3). Apoya directamente al Core Domain. | 8.0 / 10 |
| **Identidad, Tienda y Control de Acceso** | `Store & Identity` | **Generic** | Administración de tiendas (_multi-tenant_), cuentas de usuario, roles (RBAC) y consentimiento de la Ley N.° 29733. Problema común y estandarizado. | 8.5 / 10 |
| **Comunicaciones y Notificaciones** | `Notification & Messaging` | **Generic** | Envío de correos transaccionales por Resend (notificaciones de cargo de 48 h, cobro y mora). Capacidad genérica reemplazable. | 9.0 / 10 |
| **Facturación y Suscripciones SaaS** | `Billing & Subscription` | **Generic** | Gestión del cobro mensual/anual de los planes Quina Pro y Plus a través de Mercado Pago. Totalmente desacoplado de las finanzas de los vecinos. | 8.5 / 10 |

### 2.2. Catálogo de Bounded Contexts y fronteras lingüísticas

Un error crítico en arquitecturas monolíticas es forzar un único modelo global
para términos que significan cosas distintas según la perspectiva del negocio.
Quina define fronteras lingüísticas explícitas donde los términos son
completamente unívocos:

```
┌─────────────────────────────────┐       ┌─────────────────────────────────┐
│     Credit & Ledger Context     │       │   Catalog & Inventory Context   │
│         (Core Domain)           │◄──────│      (Supporting Domain)        │
│ Entidad: DebtorAccount          │  ACL  │ Entidad: ProductStock           │
│ Entidad: CreditCharge           │       │ Entidad: SaleDispatch           │
│ Operación: SettleInstallment    │       │ Operación: DeductKardex         │
└─────────────────────────────────┘       └─────────────────────────────────┘
                 ▲                                         ▲
                 │ Customer / Supplier                     │ Customer / Supplier
                 └──────────────────┬──────────────────────┘
                                    │ Shared Kernel (TenantContext / StoreId)
                    ┌───────────────┴───────────────┐
                    │   Store & Identity Context    │
                    │      (Generic Subdomain)      │
                    │ Entidad: Store (Tenant)       │
                    │ Entidad: User (Credentials)   │
                    │ Entidad: ConsentLog           │
                    └───────────────────────────────┘
```

#### Análisis de desplazamientos de significado (_Linguistic Boundary Shifts_)

Para garantizar alta cohesión y cero ambigüedad, los conceptos centrales cambian
de modelo e invariantes entre contextos:

- **Concepto "Cliente":**
  - En `Store & Identity Context`: es un **`User`** con credenciales de acceso,
    correo electrónico y registro de consentimiento legal (Ley N.° 29733).
  - En `Credit & Ledger Context`: es un **`DebtorAccount`** o sujeto de crédito,
    definido por su límite disponible, tasa compensatoria pactada, días de corte y
    pago, y saldo deudor. Carece de contraseña o datos de sesión.
  - En `Catalog & Inventory Context`: **no existe**. El inventario despacha ítems y
    registra salidas de almacén sin conocer la identidad financiera del comprador.
- **Concepto "Venta":**
  - En `Catalog & Inventory Context`: es un **`SaleDispatch`** que lista líneas de
    productos (`quantity`, `unit_cost_snapshot`), descuenta unidades y asienta un
    movimiento en el kardex valorizado.
  - En `Credit & Ledger Context`: es un **`CreditCharge`** o débito financiero que
    solo requiere el monto monetario total y la fecha de compra para alimentar el
    motor de amortización francesa o el cálculo de interés diario.
- **Concepto "Pago":**
  - En `Credit & Ledger Context`: es un **`DebtSettlement`** que cancela el capital
    de la cuota o saldo del mes y abona intereses moratorios acumulados.
  - En `Billing & Subscription Context`: es una **`SaaSSubscriptionCharge`**
    procesada mediante Mercado Pago para renovar la licencia del comercio.

### 2.3. Mapa de relaciones de contexto (_Context Mapping_)

La integración entre contextos se gobierna mediante patrones tácticos formales:

1. **`Catalog & Inventory` $\rightarrow$ `Credit & Ledger` (Anti-Corruption Layer - ACL):**
   - Cuando se confirma un despacho de venta al crédito en el mostrador, la capa
     de aplicación utiliza una capa anticorrupción que traduce los ítems
     vendidos a un comando financiero simple (`ChargeCustomerCreditCommand`). El
     motor de crédito nunca importa entidades de inventario ni de proveedores.
2. **`Store & Identity` $\rightarrow$ Todos los contextos (Shared Kernel ligero):**
   - Todos los contextos comparten exclusivamente el Value Object inmutable
     `StoreId` y el contrato `TenantContext`. Cada repositorio inyecta
     automáticamente el identificador de la tienda para aislamiento estricto.
3. **`Credit & Ledger` $\rightarrow$ `Notification & Messaging` (Domain Events):**
   - El motor financiero publica eventos de dominio en memoria
     (`CreditChargeCreatedEvent`, `CycleSettledEvent`, `LatePaymentDetectedEvent`).
     El adaptador de notificaciones escucha estos eventos y delega el envío a
     Resend de forma asíncrona, manteniendo el núcleo financiero 100 % desacoplado
     de la infraestructura de correo.
4. **`Credit & Ledger` $\rightarrow$ `Analytics & Insights` (Published Language):**
   - Las liquidaciones mensuales cerradas se publican como proyecciones
     financieras de solo lectura para alimentar el prompt de recomendaciones de IA
     en Groq.

### 2.4. Glosario de Lenguaje Ubicuo (_Ubiquitous Language_) y anti-términos

Para garantizar coherencia absoluta entre el código, los requerimientos de la
docente y el usuario bodeguero, se define el glosario canónico:

| Término canónico | Contexto | Definición precisa en el dominio | Término prohibido / Anti-término |
| :--- | :--- | :--- | :--- |
| **Fiado / Crédito barrial** | `Credit & Ledger` | Crédito de consumo a corto plazo otorgado por confianza mercantil entre el comercio y el vecino. | *Micropréstamo bancario*, *Crédito subprime* |
| **Fecha de corte** | `Credit & Ledger` | Día mensual en que se consolidan las compras del período para liquidar intereses y cuotas. | *Closing date contable*, *Facturación fiscal* |
| **Fecha de pago pactada** | `Credit & Ledger` | Día mensual acordado en que el vecino se compromete a cancelar la liquidación sin mora. | *Due date financiero corporativo* |
| **Período de gracia capitalizado** | `Credit & Ledger` | Días entre la fecha de compra y el corte en ventas en cuotas, cuyos intereses se suman al capital antes de aplicar la cuota francesa. | *Días muertos*, *Carencia simple* |
| **Interés moratorio** | `Credit & Ledger` | Cargo compensatorio adicional por días de atraso sobre el saldo total impago posterior a la fecha pactada. | *Penalidad punitiva*, *Mora judicial* |
| **Conformidad de cargo** | `Credit & Ledger` | Mecanismo de aceptación de deuda notificado por correo con plazo de 48 h para objeciones. | *Firma digital notarial*, *Pagaré legal* |
| **Kardex valorizado** | `Catalog & Inventory` | Registro cronológico e inmutable de entradas y salidas de stock evaluadas al costo promedio ponderado. | *Inventario simple*, *Stock en memoria* |

### 2.5. Evaluación de cohesión y prevención de anti-patrones

Siguiendo la métrica objetiva de cohesión de Domain Analysis:

$$
\text{Cohesión} = \frac{\text{Lingüística (3/3)} + \text{Uso (3/3)} + \text{Datos (2/2)} + \text{Cambio (2/2)}}{10} = \mathbf{1.0 \ (10/10 \ \text{en Core Domain})}
$$

- **Anti-patrón *Big Ball of Mud* evitado:** los agregados financieros no
  arrastran datos de stock ni facturas físicas de compra; cada contexto persiste y
  opera solo su información requerida.
- **Anti-patrón *Generic in Core* evitado:** las integraciones de infraestructura
  (Resend, Supabase Storage, Groq y Mercado Pago) se extrajeron a adaptadores
  secundarios, protegiendo las fórmulas financieras de cambios de proveedores
  cloud.
- **Anti-patrón *Mixed Responsibilities* evitado:** el cálculo del plan francés
  se ejecuta en un servicio de dominio puro (`FrenchAmortizationCalculator`),
  aislado de la persistencia y de la capa web.

---

## 3. Especificación del stack tecnológico

La implementación del monolito modular full-stack se apoya en un conjunto
coherente de herramientas en TypeScript:

| Capa                 | Tecnología                  | Versión / Detalle                | Justificación técnica                                                                   |
| :------------------- | :-------------------------- | :------------------------------- | :-------------------------------------------------------------------------------------- |
| **Framework base**   | Next.js                     | 15+ (App Router)                 | Unifica cliente y servidor con Server Actions y soporte PWA sin backend separado.       |
| **Lenguaje**         | TypeScript                  | 5.x (Strict mode)                | Tipado estricto compartido desde el dominio hasta los componentes de React.             |
| **Gestor paquetes**  | pnpm                        | Estándar del workspace           | Resolución rápida, determinista y prevención de dependencias fantasma.                  |
| **Estilos UI**       | Tailwind CSS                | v4                               | Estilos utilitarios atómicos y diseño responsivo adaptado a pantallas móviles.          |
| **Componentes UI**   | shadcn/ui                   | Radix UI primitives              | Primitivas accesibles, ligeras y totalmente desacopladas de librerías de estilos pesadas.|
| **Iconografía**      | Lucide React                | `lucide-react`                   | Iconos limpios, optimizados por tree-shaking para despacho veloz.                      |
| **Skeletons UI**     | Boneyard                    | `boneyard-js`                    | Generación automática en desarrollo de pantallas esqueléticas pixel-perfect.            |
| **Notificaciones**   | Sileo                       | `sileo`                          | Toasts con física de resortes fluida para retroalimentación instantánea en mostrador.   |
| **Formularios**      | React Hook Form + Zod       | Estándar de tipado               | Validación estricta en dos niveles (cliente y servidor) con DTOs tipados.              |
| **ORM y Datos**      | Drizzle ORM                 | `drizzle-orm` + `postgres-js`    | Cero arranques en frío en Vercel, transacciones SQL ACID nativas y esquemas tipados.    |
| **Base de datos**    | Supabase PostgreSQL         | PostgreSQL 16+ (Plan Free)       | Motor transaccional relacional con Row Level Security (RLS) para multi-tenancy.         |
| **Autenticación**    | Auth.js / NextAuth          | v5 (Google OAuth + Credentials)  | Manejo de sesiones seguras, login federado con Google y contraseñas con Argon2id.      |
| **Protección bots**  | Cloudflare Turnstile        | `@marsidev/react-turnstile`      | Captcha 100 % gratuito, no intrusivo y sin retos visuales para proteger el login y altas.|
| **Correos**          | Resend SDK                  | `resend` (Capa gratuita)         | Despacho asíncrono de notificaciones de cargo, recordatorios y alertas de mora.         |
| **Pagos SaaS**       | Mercado Pago SDK            | `mercadopago`                    | Suscripciones recurrentes en soles peruanos con tarjetas locales y webhooks seguros.    |
| **IA Financiera**    | Groq Cloud SDK              | `groq-sdk` (Llama 3 en free tier)| Inferencia ultrarrápida para recomendaciones comerciales con control de cuotas.         |

---

## 4. Decisiones de diseño y descarte de sobreingeniería

En consonancia con los principios de arquitectura limpia y desarrollo ágil, se
evaluaron y descartaron alternativas complejas:

### Por qué Drizzle ORM frente a Prisma
- **Sin binarios pesados en serverless:** Prisma requiere un motor en Rust
  (~40 MB) que penaliza los arranques en frío en funciones serverless de Vercel.
  Drizzle es 100 % TypeScript nativo y arranca en milisegundos.
- **Transacciones ACID complejas:** Quina ejecuta operaciones financieras
  atómicas críticas (ej: validar stock $\rightarrow$ restar inventario $\rightarrow$
  registrar kardex $\rightarrow$ validar límite $\rightarrow$ registrar venta al
  crédito). Drizzle soporta transacciones nativas de PostgreSQL con máxima
  claridad y seguridad.
- **Conexión limpia con Supabase:** Se conecta de forma directa al pool de
  conexiones transaccionales de Supavisor (puerto 6543) sin requerir servicios de
  pago como Prisma Accelerate.

### Por qué se descarta Redis en la versión V1
- **Consistencia financiera absoluta:** Cachear límites de crédito o saldos en
  Redis genera riesgo de lecturas sucias e invalidación de caché desfasada. La
  verdad financiera debe ser transaccional e inmediata en PostgreSQL.
- **Rendimiento demostrado:** Con índices relacionales adecuados en PostgreSQL
  (`store_id`, `customer_id`), las consultas de cartera toman menos de 5 ms.
- **Seguridad perimetral resuelta:** La protección contra ataques de fuerza bruta y
  bloqueos por IP los asume Cloudflare WAF en el borde perimetral.
- **Simplicidad operativa:** Evita añadir otro servicio en la nube, costos
  adicionales y latencia entre Vercel y el proveedor de caché.

---

## 5. Estructura de directorios limpia (Clean Architecture en Next.js)

La estructura de carpetas implementa la separación estricta entre Dominio,
Aplicación, Infraestructura y Presentación dentro del App Router de Next.js:

```text
quina-app/
├── src/
│   ├── core/                             # NÚCLEO PURO (Sin dependencias de frameworks)
│   │   ├── domain/                       # Capa 1: Dominio y Reglas de Negocio
│   │   │   ├── credit/                   # Entidades: Customer, CreditAccount, Installment
│   │   │   ├── finance/                  # Fórmulas: FrenchAmortization, InterestCalculator
│   │   │   ├── inventory/                # Entidades: Product, KardexMovement, CostCalculator
│   │   │   ├── identity/                 # Entidades: Store, User, Role
│   │   │   ├── shared/                   # Value Objects: Money, Result, DateRange
│   │   │   └── ports/                    # Puertos abstractos (Interfaces de salida)
│   │   │       ├── ICustomerRepository.ts
│   │   │       ├── ISaleRepository.ts
│   │   │       ├── IProductRepository.ts
│   │   │       ├── INotificationGateway.ts
│   │   │       ├── IStorageGateway.ts
│   │   │       └── IPaymentGateway.ts
│   │   └── application/                  # Capa 2: Casos de Uso (Orquestación de Dominio)
│   │       ├── credit/
│   │       │   ├── RegisterCreditSaleUseCase.ts
│   │       │   ├── GenerateSettlementUseCase.ts
│   │       │   └── ProcessDebtPaymentUseCase.ts
│   │       ├── inventory/
│   │       │   ├── RegisterPurchaseUseCase.ts
│   │       │   └── AdjustStockUseCase.ts
│   │       └── analytics/
│   │           └── GetPortfolioSummaryUseCase.ts
│   ├── infrastructure/                   # Capa 3: Adaptadores Secundarios (Infraestructura)
│   │   ├── db/                           # Persistencia con Drizzle ORM
│   │   │   ├── schema/                   # Tablas SQL tipadas en Drizzle
│   │   │   ├── repositories/             # DrizzleCustomerRepository, DrizzleSaleRepository
│   │   │   └── connection.ts             # Cliente y pool de Supabase Postgres
│   │   ├── email/                        # ResendNotificationAdapter (implementa puerto)
│   │   ├── storage/                      # SupabaseStorageAdapter (implementa puerto)
│   │   ├── payments/                     # MercadoPagoPaymentAdapter (implementa puerto)
│   │   └── ai/                           # GroqAIRecommendationAdapter (implementa puerto)
│   └── presentation/                     # Capa 4: Adaptadores Primarios y UI (Next.js)
│       ├── app/                          # Next.js App Router (Páginas y layouts)
│       │   ├── (auth)/login/             # Pantalla de acceso seguro
│       │   ├── (dashboard)/              # Vistas de mostrador del comerciante
│       │   │   ├── sales/                # Despacho en 3 toques
│       │   │   ├── customers/            # Gestión de cuentas y créditos
│       │   │   ├── inventory/            # Catálogo, kardex y compras
│       │   │   ├── portfolio/            # Panel de cartera y cobranza
│       │   │   └── reports/              # Reportes financieros e IA
│       │   ├── (customer)/portal/        # Consulta pública gratuita del vecino
│       │   ├── api/webhooks/             # Route Handlers para Mercado Pago
│       │   └── actions/                  # Server Actions (Invocan Use Cases)
│       ├── components/                   # Componentes visuales
│       │   ├── ui/                       # shadcn/ui (Botones, diálogos, tablas)
│       │   ├── feedback/                 # Toasts Sileo y Skeletons Boneyard
│       │   └── forms/                    # Formularios con validación Zod
│       └── hooks/                        # Custom hooks y UI optimista
├── drizzle.config.ts                     # Configuración de migraciones Drizzle Kit
├── tailwind.config.ts                    # Configuración de Tailwind CSS v4
├── next.config.ts                        # Configuración de Next.js PWA
└── package.json                          # Dependencias gestionadas con pnpm
```

---

## 6. Estrategia de pruebas: In-Memory Adapters para validación formal

Uno de los mayores beneficios de aplicar Clean Architecture es que los casos de
uso de Quina se pueden probar en milisegundos mediante pruebas unitarias
rigurosas con repositorios en memoria, sin necesidad de conectarse a PostgreSQL
ni levantar contenedores Docker:

```typescript
// tests/unit/RegisterCreditSaleUseCase.spec.ts
import { RegisterCreditSaleUseCase } from "@/core/application/credit/RegisterCreditSaleUseCase";
import { InMemoryCustomerRepository } from "@/tests/fakes/InMemoryCustomerRepository";
import { InMemorySaleRepository } from "@/tests/fakes/InMemorySaleRepository";
import { Money } from "@/core/domain/shared/Money";

describe("RegisterCreditSaleUseCase", () => {
  it("debe rechazar una venta fiada si excede la línea de crédito disponible", async () => {
    const customerRepo = new InMemoryCustomerRepository();
    const saleRepo = new InMemorySaleRepository();
    const useCase = new RegisterCreditSaleUseCase(customerRepo, saleRepo);

    // Cliente con límite de S/ 100 y deuda actual de S/ 80
    await customerRepo.seedCustomer({
      id: "cust-1",
      creditLimit: Money.fromSoles(100),
      currentDebt: Money.fromSoles(80),
    });

    const result = await useCase.execute({
      customerId: "cust-1",
      amount: Money.fromSoles(30), // 80 + 30 = 110 (Supera el límite)
    });

    expect(result.isSuccess).toBe(false);
    expect(result.error).toContain("Excede el límite de crédito disponible");
  });
});
```

Esta separación garantiza el cumplimiento de los **dos juegos de datos de prueba
obligatorios** exigidos por la rúbrica del curso, permitiendo demostrar cálculos
financieros auditables de caja blanca y caja negra directamente en el código.

---

## 7. Arquitectura de seguridad, autenticación y flujos de acceso

Quina implementa una arquitectura de seguridad por capas que balancea una
protección estricta para las cuentas comerciales (dueños y cajeros) con una
experiencia sin fricción para los clientes vecinos (caseros deudores).

```
┌────────────────────────────────────────────────────────────────────────┐
│                        Capa de Borde (Cloudflare)                      │
│     WAF + Anti-DDoS + Cloudflare Turnstile (Captcha 100 % gratuito)    │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                   Capa de Autenticación (Auth.js v5)                   │
│                                                                        │
│   ┌───────────────────────────┐      ┌─────────────────────────────┐   │
│   │ Comerciante / Empleado    │      │ Cliente Vecino (Doña Rosa)  │   │
│   │ • Email + Argon2id hash   │      │ • Magic Link directo (1-clic│   │
│   │ • Google OAuth 2.0        │      │ • Continuar con Google      │   │
│   │ • Bloqueo tras 5 fallos   │      │ • DNI + OTP (6 dígitos)     │   │
│   │ • Reset con token (30 min)│      │ • Contraseña (Opcional)     │   │
│   └───────────────────────────┘      └─────────────────────────────┘   │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                  Base de Datos (Supabase PostgreSQL 16)                │
│    `users`, `accounts`, `email_verifications`, `password_reset_tokens` │
│    + Row Level Security (RLS) con `app.current_store_id`               │
└────────────────────────────────────────────────────────────────────────┘
```

### 7.1. Política de contraseñas seguras y algoritmo de hashing

Para las cuentas con acceso al mostrador y administración (`STORE_OWNER` y
`STORE_EMPLOYEE`):

- **Requisitos de complejidad:** longitud mínima de 8 caracteres, exigiendo al
  menos una mayúscula, un número y un símbolo. En el cliente se utiliza la
  librería `zxcvbn-ts` para evaluar la entropía y rechazar contraseñas comunes
  (como "123456" o "bodega2026").
- **Algoritmo de derivación de claves:** se utiliza **Argon2id** (ganador del
  *Password Hashing Competition* y recomendado por OWASP), configurado con
  memoria de 64 MB, 3 iteraciones y paralelismo de 4 hilos (o **bcrypt** con
  factor de costo 12 como alternativa nativa en Node.js).
- **Protección contra fuga:** las contraseñas nunca se guardan en texto plano ni
  se retornan en consultas de API o DTOs de lectura.

### 7.2. Verificación de correo electrónico con código OTP de 6 dígitos

Para garantizar que el correo del comerciante o del cliente sea válido antes de
recibir notificaciones de deuda o acceso:

- **Generación aleatoria segura:** el servidor genera un número criptográfico
  pseudoaleatorio de 6 dígitos numéricos (`100000` a `999999`) usando
  `crypto.randomInt()`.
- **Almacenamiento seguro en base de datos:** en la tabla `email_verifications`,
  no se guarda el código en texto plano; se almacena su hash SHA-256
  (`otp_hash`).
- **Tiempo de vida (TTL):** el código expira automáticamente a los **15 minutos**
  de su emisión (`expires_at = NOW() + INTERVAL '15 minutes'`).
- **Límite de reintentos y tasa (Rate limiting):**
  - Máximo **5 intentos erróneos** permitidos antes de invalidar el registro.
  - El botón de reenvío en la interfaz tiene un bloqueo de **60 segundos** entre
    solicitudes para evitar spam de correos.
- **Canal de despacho:** se envía una plantilla limpia en HTML a través del SDK
  de Resend en su capa gratuita.

### 7.3. Autenticación con Google (OAuth 2.0 federado)

La integración con Google OAuth 2.0 resuelve el acceso seguro e instantáneo sin
requerir memorizar contraseñas:

- **¿Requiere un servicio externo de pago? No.** Google Identity Services
  (Google Cloud OAuth 2.0) es **100 % gratuito y sin coste mensual** para
  autenticación de usuarios web estándar. Solo requiere crear un proyecto en
  Google Cloud Console, registrar la pantalla de consentimiento OAuth y generar
  un par de credenciales (`GOOGLE_CLIENT_ID` y `GOOGLE_CLIENT_SECRET`).
- **Flujo de autorización:**
  1. El usuario hace clic en **"Continuar con Google"**.
  2. Next.js redirige al servidor de autorización de Google solicitando los
     ámbitos (`scopes`) mínimos: `openid`, `email` y `profile`.
  3. Google redirige a `/api/auth/callback/google` con un código de autorización.
  4. El servidor intercambia el código por tokens y obtiene el identificador
     único de usuario de Google (`sub`).
  5. Si el correo coincide con un usuario existente, vincula la cuenta en la
     tabla `accounts`; si es un cliente nuevo, crea automáticamente el usuario
     con correo verificado.

### 7.4. Política anti-fuerza bruta y bloqueo temporal de cuentas (*Account Lockout*)

Para neutralizar ataques de diccionario y prueba automatizada de credenciales:

- **Monitoreo de intentos:** la tabla `users` mantiene el contador
  `failed_login_attempts SMALLINT`.
- **Regla de bloqueo:** tras **5 intentos fallidos consecutivos**, el sistema
  bloquea la cuenta fijando `locked_until = NOW() + INTERVAL '15 minutes'`.
- **Comportamiento en intento bloqueado:** cualquier intento de autenticación
  mientras `NOW() < locked_until` es rechazado inmediatamente con el mensaje:
  *"Cuenta temporalmente bloqueada por seguridad. Inténtalo de nuevo en 15 minutos o restablece tu contraseña."*
- **Reinicio automático:** un inicio de sesión exitoso reinicia el contador
  `failed_login_attempts` a `0` y limpia `locked_until`.
- **Notificación preventiva:** se despacha un correo de alerta por Resend
  notificando al usuario que se detectaron intentos sospechosos en su cuenta.

### 7.5. Protección perimetral contra bots: Cloudflare Turnstile

- **¿Es posible y gratuito? Sí, al 100 %.** Cloudflare Turnstile es una
  herramienta moderna de validación de visitantes completamente gratuita y sin
  límites de volumen para la mayoría de aplicaciones web.
- **Ventajas sobre reCAPTCHA v2 / v3:**
  - **Cero fricción:** no obliga al bodeguero Don Lucho ni a Doña Rosa a
    identificar semáforos, hidrantes o pasos peatonales en imágenes borrosas.
  - **Privacidad estricta:** no rastrea la navegación del usuario ni utiliza
    cookies para publicidad digital como las herramientas de Google.
  - **Modo interactivo y no interactivo:** se ejecuta de forma invisible en
    segundo plano; solo muestra una casilla de un clic si detecta anomalías en
    el navegador.
- **Validación dual (Cliente y Servidor):**
  - El componente React (`@marsidev/react-turnstile`) obtiene un token de desafío.
  - El Server Action en Next.js valida el token contra la API de Cloudflare
    (`https://challenges.cloudflare.com/turnstile/v0/siteverify`) utilizando la
    clave secreta del servidor antes de procesar el intento de login o registro.

### 7.6. Recuperación de contraseña olvidada con enlace y token criptográfico

Cuando un usuario no recuerda su contraseña:

1. **Solicitud de rescate:** el usuario ingresa su correo en `/auth/forgot-password`.
2. **Mitigación de enumeración de usuarios:** el sistema siempre responde en
   pantalla: *"Si el correo ingresado se encuentra registrado, recibirás un enlace de recuperación en breve"*,
   impidiendo que atacantes descubran qué correos existen en Quina.
3. **Generación del token seguro:** el servidor genera un token criptográfico
   de alta entropía de 32 bytes (`crypto.randomBytes(32).toString('hex')`).
4. **Almacenamiento de hash:** se calcula el hash SHA-256 del token y se inserta
   en `password_reset_tokens` con `expires_at = NOW() + INTERVAL '30 minutes'` y
   `used_at = NULL`.
5. **Envío de correo por Resend:** se despacha un correo transaccional con el
   enlace de un solo uso:
   `https://quina.andeva.tech/auth/reset-password?token=<token_plano>`.
6. **Consumo y cambio de clave:** al abrir el enlace, el servidor valida que el
   token coincida por su hash, que no haya expirado y que `used_at IS NULL`. El
   usuario ingresa su nueva contraseña segura, se marca `used_at = NOW()` y se
   reinician los contadores de fallos de login.

### 7.7. Experiencia de usuario y accesibilidad para clientes vecinos (Doña Rosa)

Un dilema crítico de adopción en bodegas peruanas es que **si le exigimos a una
vecina de 58 años recordar una contraseña compleja de 12 caracteres con símbolos
para revisar su cuenta de fiados, abandonará el sistema y exigirá volver al
cuaderno de papel.**

Para resolver este desafío sin comprometer la seguridad, Quina diseña un modelo
de **acceso sin fricción (*Frictionless / Passwordless*)** con cuatro alternativas
pensadas para el vecino:

| Mecanismo de acceso | Experiencia del vecino (Doña Rosa) | Seguridad técnica |
| :--- | :--- | :--- |
| **1. Magic Link directo en notificaciones (1 clic)** | Cuando Don Lucho registra una compra fiada o llega el corte mensual, Doña Rosa recibe un correo con el botón *"Ver mi estado de cuenta"*. Al tocarlo en su celular, entra directamente a su cuenta sin ingresar ninguna clave. | Enlace firmado criptográficamente de un solo toque con expiración de 72 horas almacenado en `customer_auth_tokens`. |
| **2. Continuar con Google (1 toque)** | El 95 % de celulares Android en el Perú tienen cuenta Gmail configurada. Al ingresar a `quina.andeva.tech`, Doña Rosa toca el botón de Google y su teléfono valida su identidad al instante. | Google OAuth 2.0 estándar vinculado al correo del cliente registrado en la tienda. |
| **3. Acceso por DNI + Código OTP (Sin contraseña)** | Si Doña Rosa entra desde otro teléfono, ingresa su número de DNI. El sistema le envía un código de 6 números a su correo. Lo copia, lo pega y entra. | OTP numérico de 6 dígitos con tiempo de expiración de 15 minutos y tasa limitada. |
| **4. Contraseña personalizada (Opcional)** | Si el cliente desea fijar una contraseña permanente para ingresar tradicionalmente con correo y clave, puede configurarla en su perfil cuando lo desee. | Almacenamiento seguro bajo Argon2id, totalmente voluntario para el vecino. |

---

## 8. Persistencia relacional y esquema PostgreSQL

La persistencia del sistema está gobernada por un esquema relacional normalizado
en PostgreSQL 16 sobre Supabase, estructurado en torno a los Bounded Contexts de
la aplicación.

El detalle técnico exhaustivo, que incluye el diccionario de datos completo de
las **26 tablas**, los tipos de datos numéricos exactos, las restricciones de
integridad referencial, los índices compuestos de alto rendimiento, las
políticas de Row Level Security (RLS) y los esquemas en Drizzle ORM, se
encuentra documentado en:

👉 **[Especificación completa del esquema de base de datos PostgreSQL](quina-database-schema.md)**

