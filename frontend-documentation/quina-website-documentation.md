# Quina Website - Documentación técnica y especificación del stack

Este documento define la arquitectura técnica, las herramientas, los frameworks
y el conjunto de librerías (*stack*) para el desarrollo y despliegue del sitio web
público de **Quina** (`quina.andeva.tech`). Su enfoque es exclusivamente técnico y
operativo, detallando el propósito de cada herramienta en el ecosistema sin
abordar decisiones de diseño visual o maquetación gráfica.

Para consultar la arquitectura de la aplicación interna de gestión comercial,
revisa la [documentación de la webapp](../webapp-documentation/quina-webapp-documentation.md).
Para la identidad visual y tokens de estilos, consulta el
[sistema de diseño](design-system.md).

---

## 1. Arquitectura general y propósito del website

El sitio web público de Quina opera como la puerta de entrada digital del
producto. Cumple tres propósitos principales:

1. **Atracción y conversión de comercios:** presentar la propuesta de valor para
   los dueños de bodegas y pequeños negocios peruanos, con demostraciones
   interactivas del retorno de inversión (ROI) y planes de precios.
2. **Educación y contenido:** alojar artículos informativos, casos de éxito de
   comerciantes y guías de gestión financiera barrial mediante colecciones de
   contenido estático optimizadas para motores de búsqueda (SEO).
3. **Blindaje legal y cumplimiento normativo:** proveer el marco formal exigido
   por la legislación peruana para plataformas comerciales electrónicas,
   incluyendo políticas de privacidad (Ley N.° 29733), términos del servicio,
   política de cookies y el Libro de Reclamaciones Virtual obligatorio
   (D.S. 011-2011-PCM).

---

## 2. Especificación exhaustiva del stack tecnológico

A continuación se detalla cada componente del conjunto de herramientas,
agrupado por su función en la arquitectura del proyecto:

### 2.1. Núcleo, compilación y entorno de ejecución

| Herramienta | Versión / Detalle | Propósito técnico específico |
| :--- | :--- | :--- |
| **Astro** | 5.x | **Framework principal centrado en contenido (*Content-Driven*).** Implementa la arquitectura de islas (*Islands Architecture*), generando páginas HTML estáticas por defecto con cero JavaScript en el cliente (*Zero JS by default*). Esto garantiza tiempos de carga casi instantáneos (Time to First Byte y First Contentful Paint sub-segundo) en conexiones móviles peruanas (redes 3G/4G inestables). |
| **Node.js** | 20+ LTS | **Entorno de ejecución de servidor.** Soporta las herramientas de compilación, ejecución de scripts de empaquetado, pruebas automatizadas en integración continua (CI) y renderizado bajo demanda si se requieren endpoints de backend ligeros. |
| **pnpm** | 9.x+ (Estándar de workspace) | **Gestor de paquetes determinista.** Proporciona instalación ultrarrápida mediante almacenamiento direccionable por contenido y enlaces duros (*hard links*), evitando dependencias fantasma y garantizando paridad exacta de dependencias entre entornos. |
| **TypeScript** | 5.x (Strict Mode) | **Lenguaje de desarrollo tipado.** Asegura tipado estático riguroso en componentes `.astro`, componentes de React, esquemas de colecciones de contenido y utilidades, previniendo errores de tipos en tiempo de compilación. |
| **Vite** | 6.x (Integrado en Astro) | **Motor de desarrollo y empaquetador ultrarrápido.** Proporciona recarga de módulos en caliente (*Hot Module Replacement - HMR*) instantánea en desarrollo y optimización avanzada de activos estáticos en producción. |

### 2.2. Capa de componentes e interactividad (Islas de interfaz)

| Herramienta | Versión / Detalle | Propósito técnico específico |
| :--- | :--- | :--- |
| **@astrojs/react** | Oficial Astro | **Adaptador oficial de islas React.** Permite incrustar componentes dinámicos de React dentro del marcado estático de Astro utilizando directivas de hidratación granular (`client:load`, `client:visible`, `client:idle`), cargando JavaScript exclusivamente cuando el componente lo requiere. |
| **React** | 19.x | **Librería de interfaces de usuario.** Se emplea para los componentes altamente dinámicos del sitio: calculadora interactiva de financiamiento barrial para comerciantes, selector interactivo de planes de suscripción, acordeones dinámicos de preguntas frecuentes (FAQ) y modales de diálogo accesibles. |
| **React DOM** | 19.x | **Punto de montaje del árbol de React.** Permite la hidratación de los componentes de React sobre el DOM del navegador. |
| **Lucide Astro** | Paquete nativo | **Librería de iconos estáticos para Astro.** Renderiza iconos vectoriales SVG como componentes nativos de Astro sin despachar JavaScript al navegador, utilizándose en encabezados, pies de página, listas de características y tarjetas informativas estáticas. |
| **Lucide React** | Paquete para React | **Librería de iconos interactivos para React.** Se utiliza dentro de las islas de React donde los iconos cambian de estado de forma reactiva (por ejemplo: alternar estado abierto/cerrado en menús o acordeones). |

### 2.3. Motor de estilos y procesamiento CSS

| Herramienta | Versión / Detalle | Propósito técnico específico |
| :--- | :--- | :--- |
| **Tailwind CSS** | v4 | **Framework utilitario de estilos.** Sistema de estilos de alto rendimiento basado en variables nativas de CSS (`@theme`). Compila únicamente las clases utilizadas mediante un motor en Rust, reduciendo la hoja de estilos de producción a pocos kilobytes. |
| **@tailwindcss/vite** | Oficial v4 | **Plugin oficial para Vite.** Conecta el compilador de Tailwind CSS directamente con el pipeline de build de Vite, eliminando la necesidad de archivos de configuración pesados y acelerando la compilación. |
| **PostCSS** | 8.x | **Procesador de CSS.** Aplica transformaciones sintácticas complementarias, prefijos de proveedores para compatibilidad entre navegadores y optimizaciones de empaquetado. |

### 2.4. Animación, microinteracciones y experiencia de desplazamiento

| Herramienta | Versión / Detalle | Propósito técnico específico |
| :--- | :--- | :--- |
| **GSAP (GreenSock)** | 3.x (`gsap`) | **Motor de animación por JavaScript.** Ejecuta animaciones de alto rendimiento (60 a 120 FPS estables) para transiciones de entrada, desvanecimientos progresivos y transformaciones cinemáticas sin causar saltos de fotogramas (*layout shifts*). |
| **GSAP ScrollTrigger** | Plugin oficial GSAP | **Disparador de animaciones según scroll.** Sincroniza las animaciones de GSAP con la posición del desplazamiento del usuario en la pantalla: revelado de secciones al entrar en el viewport, fijado de elementos comparativos (*pinning*) y barras de progreso de lectura. |
| **Lenis** | `lenis` | **Normalizador de scroll suave (*Smooth Scroll*).** Proporciona una experiencia de desplazamiento inercial fluida y unificada entre sistemas operativos (Windows, macOS, Android, iOS), sincronizándose nativamente con GSAP ScrollTrigger y respetando la preferencia del sistema de movimiento reducido (`prefers-reduced-motion`). |

### 2.5. Gestión de contenidos, esquemas y tipado de datos

| Herramienta | Versión / Detalle | Propósito técnico específico |
| :--- | :--- | :--- |
| **Astro Content Collections** | Integrado en Astro | **Gestor de colecciones de contenido estático.** Permite organizar el contenido editorial en directorios dedicados (`src/content/blog/`, `src/content/legal/`), validando sus metadatos (*frontmatter*) antes de generar las páginas estáticas. |
| **Zod** | 3.x | **Validación estricta de esquemas de datos.** Define los esquemas de tipado para cada colección de contenido (por ejemplo: título obligatorio, fecha válida, autor, etiquetas, resumen y descripción SEO), garantizando que ningún artículo con datos faltantes pase a producción. |
| **@astrojs/mdx** | Oficial Astro | **Soporte para Markdown enriquecido (MDX).** Permite redactar artículos de blog y documentos legales en formato Markdown estándar mientras se intercalan componentes interactivos de React (como tablas interactivas o llamadas a la acción) dentro del texto. |

### 2.6. Internacionalización (i18n) y accesibilidad web (a11y)

| Herramienta | Versión / Detalle | Propósito técnico específico |
| :--- | :--- | :--- |
| **Astro i18n (`astro:i18n`)** | Integrado en Astro | **Enrutamiento y localización de contenidos.** Gestiona prefijos de ruta para múltiples idiomas (`/es/` como idioma principal para Perú y Latinoamérica; `/en/` para evaluación académica e inversionistas internacionales), control de idioma alternativo mediante etiquetas `<link rel="alternate" hreflang="...">` y redirección basada en cabeceras de navegación. |
| **eslint-plugin-jsx-a11y** | Estándar de linting | **Auditoría estática de accesibilidad en código.** Verifica en tiempo de desarrollo que todos los componentes interactivos cumplan con estándares WCAG: etiquetas `alt` en imágenes, roles ARIA válidos, contraste semántico, botones accesibles y navegación completa por teclado. |
| **@axe-core/playwright** | Integración de testing | **Motor de auditoría automatizada de accesibilidad.** Ejecuta pruebas de cumplimiento WCAG 2.1 nivel AA en cada página estática generada antes de autorizar el despliegue a producción. |

### 2.7. Calidad de código, formateo y automatización

| Herramienta | Versión / Detalle | Propósito técnico específico |
| :--- | :--- | :--- |
| **ESLint** | 9.x | **Analizador estático de código.** Detecta errores sintácticos, variables no utilizadas, malas prácticas en TypeScript y patrones anti-rendimiento en todo el repositorio. |
| **Prettier** | 3.x | **Formateador determinista de código.** Estandariza la indentación, uso de comillas y saltos de línea de forma unificada entre todos los desarrolladores. |
| **prettier-plugin-astro** | Plugin oficial | **Formateo para sintaxis Astro.** Habilita el formateo consistente del bloque de metadatos (*frontmatter*) y del bloque de marcado en archivos con extensión `.astro`. |
| **prettier-plugin-tailwindcss** | Plugin oficial | **Ordenamiento automático de clases utilitarias.** Reorganiza automáticamente las clases de Tailwind CSS según las reglas oficiales de especificidad y jerarquía visual. |
| **Playwright (`@playwright/test`)** | 1.x | **Framework de pruebas integrales extremo a extremo (E2E).** Simula la navegación real de usuarios sobre motores Chromium, Firefox y WebKit para validar: envío correcto del formulario del Libro de Reclamaciones, visibilidad de los banners legales, navegación del menú móvil y tiempos de respuesta. |

### 2.8. Despliegue, red perimetral e infraestructura

| Herramienta | Versión / Detalle | Propósito técnico específico |
| :--- | :--- | :--- |
| **@astrojs/vercel** | Oficial Astro | **Adaptador oficial de despliegue para Vercel.** Configura la compilación de Astro para aprovechar las capacidades de la plataforma Vercel, optimizando la generación estática (SSG) y las funciones serverless de borde. |
| **Vercel Edge Network** | Infraestructura Cloud | **Red de distribución de contenido (CDN) global.** Aloja las páginas pre-renderizadas en servidores perimetrales cercanos a los usuarios en Perú, ofreciendo compresión moderna (Brotli/Gzip), certificados SSL/TLS automáticos y alta disponibilidad sin costos fijos en plan Hobby. |

---

## 3. Blindaje legal, privacidad y protección contra contingencias jurídicas

Para proteger a la plataforma Quina de contingencias legales, multas de
organismos reguladores peruanos (como INDECOPI o la Autoridad Nacional de
Protección de Datos Personales) y eventuales demandas de usuarios o competidores,
el website integra componentes y páginas legales obligatorias modeladas en el
código fuente.

### 3.1. Gestor de consentimiento de cookies y trazabilidad técnica

- **Herramienta:** módulo de consentimiento granular (Klaro! o componente
  reactivo personalizado con almacenamiento en `localStorage`).
- **Propósito legal:** impedir la inyección de cookies analíticas o de
  seguimiento antes de que el visitante otorgue su consentimiento explícito
  (*opt-in* obligatorio).
- **Categorización de cookies:**
  1. **Cookies técnicas o estrictamente necesarias:** requeridas para la
     navegación básica, seguridad perimetral (Cloudflare Turnstile) y sesión; no
     requieren consentimiento pero se informan.
  2. **Cookies analíticas:** destinadas a medir visitas y rendimiento; bloqueadas
     por defecto hasta que el usuario las acepte expresamente.
  3. **Cookies de preferencias:** guardan la configuración de idioma y tema.
- **Mecanismo de revocación:** el pie de página incluye un enlace permanente
  *"Configurar cookies"* que permite al usuario revocar o modificar sus
  preferencias en cualquier momento con un solo clic.

### 3.2. Catálogo de páginas legales obligatorias (Astro Content Collections)

El directorio `src/content/legal/` almacena los documentos jurídicos en formato
Markdown/MDX con control de versiones y fecha de última actualización:

1. **Política de Privacidad y Tratamiento de Datos Personales (`privacy-policy.mdx`):**
   - **Normativa aplicable:** Ley N.° 29733 (Ley de Protección de Datos Personales
     del Perú) y su Reglamento aprobado por Decreto Supremo N.° 003-2013-JUS.
   - **Cláusulas exigidas:**
     - Identificación del titular del banco de datos personales y domicilio legal.
     - Finalidad expresa del tratamiento de los datos capturados (nombres,
       teléfonos, correos, RUC/DNI).
     - Mecanismo para el ejercicio de los **Derechos ARCO** (Acceso, Rectificación,
       Cancelación y Oposición) a través de un correo de contacto formal.
     - Declaración de no transferencia de datos a terceros sin autorización
       expresa.
2. **Términos y Condiciones de Uso del Servicio (`terms-and-conditions.mdx`):**
   - **Normativa aplicable:** Código Civil Peruano y Código de Protección y
     Defensa del Consumidor (Ley N.° 29571).
   - **Cláusulas de blindaje:**
     - Delimitación del alcance: Quina es una plataforma de software como
       servicio (SaaS) que provee herramientas de control contable y financiero;
       no es una entidad financiera, no capta dinero del público ni otorga
       préstamos directos con fondos propios.
     - Responsabilidad del comerciante: el dueño del negocio es el único
       responsable de los pactos de crédito y cobranza con sus clientes vecinos.
     - Propiedad intelectual: protección expresa sobre el software, marcas,
       logotipos y algoritmos contra ingeniería inversa, copia o distribución no
       autorizada.
     - Jurisdicción y competencia: sometimiento expreso a las leyes de la
       República del Perú y a la jurisdicción de los jueces y tribunales del
       distrito judicial de Lima.
3. **Política de Cookies (`cookie-policy.mdx`):**
   - Inventario detallado de cada cookie utilizada: nombre, proveedor, finalidad,
     caducidad y enlace para deshabilitarla en navegadores populares.
4. **Libro de Reclamaciones Virtual (`libro-de-reclamaciones.astro`):**
   - **Normativa obligatoria:** Decreto Supremo N.° 011-2011-PCM (Reglamento del
     Libro de Reclamaciones) y directivas de INDECOPI para comercio electrónico.
   - **Requisitos técnicos implementados:**
     - Acceso visible mediante el logotipo oficial del Libro de Reclamaciones en
       el pie de todas las páginas del sitio web.
     - Formulario digital con campos obligatorios para: datos del consumidor
       reclamante, identificación del bien contratado (producto o servicio) y
       detalle del reclamo (disconformidad relacionada a los bienes) o queja
       (malestar o descontento respecto a la atención).
     - Generación de código correlativo único de hoja de reclamación
       (ejemplo: `LR-2026-00001`).
     - Envío automático de copia en PDF o correo formal al consumidor con acuse
       de recibo inmediato.
     - Plazo legal de respuesta: aviso visible indicando el compromiso de atención
       en un plazo no mayor a **15 días hábiles improrrogables** conforme a la
       normativa peruana vigente.

### 3.3. Cabeceras de seguridad perimetral HTTP (Security Headers)

Para mitigar vulnerabilidades web comunes y proteger los datos en tránsito, la
configuración de Vercel (`vercel.json`) implementa cabeceras de respuesta
estrictas:

```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "Content-Security-Policy",
          "value": "default-src 'self'; script-src 'self' 'unsafe-inline' https://challenges.cloudflare.com; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self' https://challenges.cloudflare.com; frame-src https://challenges.cloudflare.com;"
        },
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "Referrer-Policy",
          "value": "strict-origin-when-cross-origin"
        },
        {
          "key": "Strict-Transport-Security",
          "value": "max-age=63072000; includeSubDomains; preload"
        },
        {
          "key": "Permissions-Policy",
          "value": "camera=(), microphone=(), geolocation=()"
        }
      ]
    }
  ]
}
```

---

## 4. Estructura de directorios del proyecto del website

La estructura de carpetas implementa la separación limpia entre páginas estáticas,
islas interactivas, contenido tipado y recursos legales:

```text
quina-website/
├── public/                               # Activos estáticos públicos
│   ├── branding/                         # Logotipos, isotipos e iconos vectoriales
│   ├── fonts/albert-sans/                # Archivos WOFF2 de Albert Sans
│   ├── favicon.ico                       # Favicon principal
│   └── robots.txt                        # Directivas para rastreadores de motores de búsqueda
├── src/
│   ├── content/                          # Colecciones de contenido tipadas con Zod
│   │   ├── config.ts                     # Definición de esquemas de colecciones
│   │   ├── blog/                         # Artículos del blog en Markdown / MDX
│   │   └── legal/                        # Documentos legales (privacidad, términos, cookies)
│   ├── components/                       # Componentes de interfaz
│   │   ├── astro/                        # Componentes estáticos nativos (Header, Footer, Hero)
│   │   ├── react/                        # Islas interactivas React (Calculadora, Selector Planes)
│   │   └── legal/                        # Formulario interactivo del Libro de Reclamaciones
│   ├── layouts/                          # Plantillas base de página
│   │   ├── BaseLayout.astro              # Layout maestro con metadatos SEO y fuentes
│   │   └── LegalLayout.astro             # Layout tipográfico para documentos jurídicos
│   ├── pages/                            # Enrutamiento basado en archivos
│   │   ├── index.astro                   # Portada principal en español
│   │   ├── nosotros.astro                # Historia y propósito de Quina
│   │   ├── planes.astro                  # Detalle de suscripciones para bodegas
│   │   ├── blog/                         # Índice y páginas dinámicas del blog
│   │   ├── legal/                        # Rutas de privacidad, términos y cookies
│   │   ├── libro-de-reclamaciones.astro  # Libro de Reclamaciones Virtual
│   │   └── en/                           # Rutas localizadas en inglés (i18n)
│   ├── styles/                           # Hojas de estilo globales
│   │   ├── global.css                    # Hoja maestra con importaciones y utilidades
│   │   ├── theme.css                     # Tokens de diseño Tailwind CSS v4 (@theme)
│   │   ├── typography.css                # Declaraciones @font-face de Albert Sans
│   │   └── index.css                     # Punto de entrada de estilos
│   └── utils/                            # Funciones auxiliares
│       ├── lenis.ts                      # Inicialización y puente entre Lenis y ScrollTrigger
│       ├── gsap.ts                       # Registro de plugins y configuraciones de animación
│       └── i18n.ts                       # Helpers para rutas y traducciones
├── tests/                                # Pruebas automatizadas con Playwright
│   ├── e2e/navigation.spec.ts            # Pruebas de navegación y enlaces legales
│   └── a11y/accessibility.spec.ts        # Pruebas automatizadas de accesibilidad con axe-core
├── astro.config.mjs                      # Configuración de Astro (integraciones y Vercel adapter)
├── package.json                          # Dependencias gestionadas con pnpm
├── tsconfig.json                         # Configuración estricta de TypeScript
└── vercel.json                           # Cabeceras de seguridad perimetral y redirecciones
```

---

## 5. Próximos pasos y sincronización

Este documento técnico se articula con:

- [Sistema de diseño de Quina](design-system.md): especificación visual de tokens cromáticos, tipografía Albert Sans y catálogo de componentes de interfaz.
- [Arquitectura de la Webapp](../webapp-documentation/quina-webapp-documentation.md): especificación técnica del backend y de la aplicación interna multi-tienda en Next.js.
- [Historias de usuario para SCRUM](../quina-user-stories.md): trazabilidad de requerimientos en Sprint 1 al Sprint 5.
- [Documentación técnica maestra](../quina-documentation.md): fuente de la verdad general del proyecto.
