# Quina - Sistema de diseño (Design System)

Este documento especifica el **Sistema de diseño (*Design System*)** oficial de
**Quina**, estructurado sobre los recursos de marca de
[`docs/frontend-documentation/branding/`](branding/) y los tokens de estilos
compilados en [`docs/frontend-documentation/styles/`](styles/). Define la
paleta cromática, la jerarquía tipográfica, el catálogo de activos de identidad,
las reglas de espaciado, los componentes de interfaz y las directrices de
animación accesible para el website y la aplicación.

Para consultar la arquitectura técnica del sitio público, revisa la
[documentación técnica del website](quina-website-documentation.md).
Para la hoja de estilos global lista para copiar y pegar, consulta
[`styles/global.css`](styles/global.css).

---

## 1. Fundamentos e identidad de marca

La identidad de **Quina** nace en la intersección entre la tradición comercial
del barrio peruano y la innovación financiera digital:

- **Origen del nombre:** rinde homenaje al **árbol de la quina**, emblema patrio
  del escudo nacional del Perú que simboliza resiliencia, crecimiento natural y
  salud económica; y conecta con la expresión popular barrial *"la quina"*, que
  evoca dinero, intercambio cotidiano y valor ganado con esfuerzo.
- **Personalidad visual:** cercana, confiable, moderna y sin pretensiones
  corporativas intimidantes. Diseñada para que el bodeguero Don Lucho sienta una
  herramienta sólida y la vecina Doña Rosa perciba transparencia y respeto.

```
┌────────────────────────────────────────────────────────────────────────┐
│                        QUINA DESIGN SYSTEM                             │
│                                                                        │
│   Primario: Verde Quina (#2bdd66)  │  Fondo: Blanco Hueso (#f8f8fa)    │
│   Contraste: Verde Oscuro (#0a2e17)│  Tarjeta: Blanco Puro (#ffffff)   │
│   Acento: Ámbar Dorado (#ffb020)   │  Texto: Antracita (#262626)       │
│                                                                        │
│   Tipografía: Albert Sans (Variable 100-900)                           │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Paleta cromática y tokens de color

Los colores de Quina están definidos en variables CSS nativas dentro de
[`styles/theme.css`](styles/theme.css) para el compilador de **Tailwind CSS v4**:

### 2.1. Colores de marca y acento

| Token | Valor Hex | Muestra | Rol semántico en el sistema |
| :--- | :--- | :---: | :--- |
| `--color-primary` | `#2bdd66` | 🟢 | **Verde Quina principal:** color de acción primaria, botones destacados, indicadores de crecimiento y salud financiera. |
| `--color-primary-dark` | `#0a2e17` | 🌲 | **Verde bosque profundo:** contraste máximo para texto sobre fondos verdes y elementos institucionales de alta solidez. |
| `--color-primary-soft` | `#e3f9ea` | 🍃 | **Verde menta suave:** fondo de insignias (*badges*), estados seleccionados y alertas positivas. |
| `--color-accent` | `#ffb020` | 🟡 | **Ámbar cálido de acento:** llamadas de atención secundarias, períodos de gracia y avisos preventivos. |
| `--color-accent-dark` | `#b86e00` | 🍯 | **Ámbar profundo:** texto legible sobre fondos ámbar suaves. |
| `--color-accent-soft` | `#fff3d6` | 🍞 | **Ámbar pálido:** fondos de tarjetas informativas de corte y advertencias leves. |

### 2.2. Superficies, tarjetas y fondos

| Token | Valor Hex | Modo | Rol semántico |
| :--- | :--- | :--- | :--- |
| `--color-surface-white` | `#f8f8fa` | Claro | Fondo principal del sitio web y aplicaciones en modo diurno. |
| `--color-surface-black` | `#262626` | Oscuro | Fondo principal en modo oscuro y secciones de alto contraste. |
| `--color-card-white` | `#ffffff` | Claro | Fondo de tarjetas, modales y campos de formulario en modo claro. |
| `--color-card-black` | `#272727` | Oscuro | Fondo de tarjetas elevadas en modo oscuro. |

### 2.3. Tipografía y bordes

| Token | Valor Hex | Rol semántico |
| :--- | :--- | :--- |
| `--color-text-black` | `#262626` | Texto principal de párrafos y encabezados (máxima legibilidad). |
| `--color-text-white` | `#ffffff` | Texto principal sobre fondos oscuros o botones oscuros. |
| `--color-text-subtitles`| `#c6c6c6` | Texto secundario, descripciones atenuadas y leyendas. |
| `--color-border-card-dark`| `#ebebeb` | Bordes sutiles para tarjetas y divisores en modo claro. |
| `--color-border-card-light`| `#c6c6c6` | Bordes de inputs inactivos y separadores de sección. |

### 2.4. Estados funcionales del sistema

| Token | Valor Hex | Muestra | Caso de uso financiero |
| :--- | :--- | :---: | :--- |
| `--color-success` | `#3ac530` | 🟢 | Pago registrado con éxito, cliente al día, conformidad aceptada. |
| `--color-warning` | `#f0b100` | 🟡 | Cuenta próxima a la fecha de pago pactada, alerta de proximidad al tope del Nuevo RUS. |
| `--color-error` | `#fb2c36` | 🔴 | Crédito en mora con intereses moratorios, límite de crédito agotado, stock en cero. |

---

## 3. Tipografía oficial: Albert Sans

Quina adopta **Albert Sans** como su tipografía corporativa y de interfaz. Es una
familia tipográfica sans-serif geométrica con terminaciones limpias, inspirada en
la cartelería moderna y optimizada para pantallas móviles de baja resolución.

### 3.1. Archivos tipográficos locales (`branding/albert-sans/`)

Para garantizar independencia de servidores externos y máxima velocidad de
carga, las fuentes se alojan localmente en formato **WOFF2** optimizado:

- `AlbertSans-VariableFont_wght.woff2` (Eje de pesos variable: 100 a 900).
- `AlbertSans-Italic-VariableFont_wght.woff2` (Eje de cursivas variable).
- Pesos discretos adicionales: `Thin` (100), `Light` (300), `Regular` (400),
  `Medium` (500), `SemiBold` (600), `Bold` (700), `ExtraBold` (800) y `Black` (900).

Las directrices `@font-face` y la asignación al cuerpo del documento se
encuentran en [`styles/typography.css`](styles/typography.css).

### 3.2. Escala tipográfica fluida

| Nivel | Tamaño (rem / px) | Peso sugerido | Interlineado | Caso de uso principal |
| :--- | :--- | :--- | :--- | :--- |
| **Display** | 3.5rem (56px) | ExtraBold (800) | 1.1 | Título principal de la portada del website (*Hero*). |
| **H1** | 2.5rem (40px) | Bold (700) | 1.2 | Título principal de página o módulo. |
| **H2** | 2.0rem (32px) | Bold (700) | 1.25 | Encabezados de sección y comparativas de planes. |
| **H3** | 1.5rem (24px) | SemiBold (600) | 1.3 | Títulos de tarjetas de características y precios. |
| **H4** | 1.25rem (20px) | SemiBold (600) | 1.4 | Subtítulos de bloques informativos y modales. |
| **Body Large**| 1.125rem (18px) | Regular (400) | 1.55 | Párrafo introductorio (*lead*) y artículos de blog. |
| **Body** | 1.0rem (16px) | Regular (400) | 1.5 | Texto general, descripciones y tablas de datos. |
| **Small** | 0.875rem (14px) | Medium (500) | 1.45 | Etiquetas de formularios, fechas y notas al pie. |
| **Caption** | 0.75rem (12px) | SemiBold (600) | 1.3 | Textos legales, *badges* y códigos correlativos. |

---

## 4. Identidad visual y catálogo de activos (`branding/`)

El directorio [`branding/`](branding/) almacena los logotipos e isotipos
vectoriales y de mapa de bits en alta definición:

```
docs/frontend-documentation/branding/
├── quina-isologo-verde.svg           # Isologo principal sobre fondos claros
├── quina-isologo-verde-oscuro.svg    # Isologo institucional sobre blanco
├── quina-isologo-blanco.svg          # Isologo para fondos oscuros y cabeceras
├── quina-isologo-negro.svg           # Isologo monocromático para documentos
├── quina-isotipo-verde.svg           # Isotipo cuadrado verde para aplicaciones
├── quina-isotipo-verde-oscuro.svg    # Isotipo cuadrado verde oscuro
├── quina-isotipo-blanco.svg          # Isotipo cuadrado blanco
├── quina-isotipo-negro.svg           # Isotipo cuadrado negro
├── quina-icon-primary.svg            # Favicon e icono principal web
├── quina-icon-primary.ico            # Favicon para navegadores de escritorio
├── quina-icon-primary-dark.svg       # Favicon alternativo modo oscuro
└── quina-icon-primary-dark.ico       # Favicon escritorio modo oscuro
```

### 4.1. Variantes de logotipo e isologo

1. **Isologo principal (`quina-isologo-verde.svg`):**
   - Combina el símbolo de la quina estilizado con la tipografía nominal "Quina".
   - Aplicación recomendada: encabezado del sitio web sobre fondo claro (`#f8f8fa`
     o `#ffffff`).
2. **Isologo invertido (`quina-isologo-blanco.svg`):**
   - Símbolo y tipografía en blanco puro.
   - Aplicación recomendada: pie de página oscuro, menús nocturnos y fondos
     fotográficos de alto contraste.
3. **Isologo institucional (`quina-isologo-verde-oscuro.svg`):**
   - Presentación sobria en tono verde bosque (`#0a2e17`).
   - Aplicación recomendada: reportes formales, exportaciones en PDF y membretes
     legales.
4. **Isotipo independiente (`quina-isotipo-verde.svg`):**
   - El icono representativo sin texto.
   - Aplicación recomendada: icono de acceso directo en celulares (PWA), favicons,
     botones de menú en pantallas móviles y avatares de redes.

### 4.2. Reglas de uso y área de protección

- **Área de protección:** el isologo debe conservar un margen libre perimetral
  equivalente a la altura de la letra "Q" del logotipo, libre de otros textos,
  bordes o elementos visuales que resten protagonismo.
- **Tamaño mínimo:**
  - Isologo completo: ancho mínimo de 120 px en digital (o 30 mm en impresión).
  - Isotipo independiente: ancho mínimo de 24 px en digital (o 8 mm en impresión).
- **Usos incorrectos:**
  - No distorsionar las proporciones de aspecto (escalado no proporcional).
  - No alterar los colores aprobados ni aplicar degradados ajenos a la paleta.
  - No colocar el isologo verde sobre fondos saturados donde el contraste sea
    inferior a 4.5:1.

---

## 5. Espaciado, elevaciones y radios de borde

El sistema adopta una escala geométrica de 4 px como módulo base:

### 5.1. Escala de radios de curvatura (*Border Radius*)

- `rounded-lg` (0.5rem / 8px): campos de texto (*inputs*), selectores y botones
  compactos de mostrador.
- `rounded-xl` (0.75rem / 12px): botones principales de acción (`.btn-primary`,
  `.btn-secondary`), menús desplegables y tarjetas compactas.
- `rounded-2xl` (1.0rem / 16px): tarjetas principales de contenido
  (`.card-quina`), paneles de precios y ventanas modales de diálogo.
- `rounded-full` (9999px): insignias (*badges*), píldoras de filtros y avatares.

### 5.2. Elevaciones y sombras (*Box Shadows*)

- **Nivel 1 (Superficie / Reposo):** `box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);`
  con borde perimetral sutil (`border: 1px solid var(--color-border-card-dark)`).
- **Nivel 2 (Interacción / Hover):** `box-shadow: 0 4px 12px rgba(43, 221, 102, 0.25);`
  para botones primarios y tarjetas interactivas.
- **Nivel 3 (Elevación / Modales):** `box-shadow: 0 12px 32px rgba(0, 0, 0, 0.12);`
  para modales de confirmación y menús flotantes.

---

## 6. Catálogo de componentes fundamentales de interfaz

Los siguientes componentes están formalizados en [`styles/global.css`](styles/global.css)
listos para aplicarse con clases utilitarias:

### 6.1. Botón primario de acción (`.btn-primary`)

```html
<button class="btn-primary">
  <span>Registrar mi bodega gratis</span>
  <svg class="w-4 h-4" ...></svg>
</button>
```

- Fondo: `--color-primary` (`#2bdd66`).
- Texto: `--color-primary-dark` (`#0a2e17`), peso 600.
- Interacción hover: oscurece sutilmente a `#24c65a`, se eleva 1 px y proyecta un
  resplandor verde translúcido.

### 6.2. Botón secundario oscuro (`.btn-secondary`)

```html
<button class="btn-secondary">
  <span>Ver demostración en video</span>
</button>
```

- Fondo: `--color-card-black` (`#272727`).
- Texto: `--color-text-white` (`#ffffff`), peso 600 con borde fino semitransparente.
- Interacción hover: se ilumina a `#333333` y se eleva 1 px.

### 6.3. Tarjeta estándar de Quina (`.card-quina`)

```html
<article class="card-quina">
  <h3 class="text-xl font-bold mb-2">Plan Bodega Pro</h3>
  <p class="text-sm text-[var(--color-text-subtitles)]">Hasta 100 clientes al crédito</p>
</article>
```

- Fondo: `--color-card-white` en modo claro y `--color-card-black` en modo oscuro.
- Borde sutil y esquinas redondeadas de 16 px (`rounded-2xl`).

### 6.4. Insignia o píldora de estado (`.badge-quina`)

```html
<span class="badge-quina">
  <span class="w-2 h-2 rounded-full bg-[var(--color-primary)]"></span>
  <span>Cuenta al día</span>
</span>
```

- Fondo suave `--color-primary-soft` con texto de alto contraste `--color-primary-dark`.

---

## 7. Directrices de movimiento y animación (GSAP y Lenis)

Las animaciones en el website de Quina enriquecen la narrativa del producto sin
comprometer el rendimiento ni la accesibilidad:

### 7.1. Estandarización de transiciones y *easings*

- **Microinteracciones (hover, clics, acordeones):** duración de 150 a 250 ms con
  curva cúbica rápida `cubic-bezier(0.16, 1, 0.3, 1)`.
- **Revelado de secciones con GSAP ScrollTrigger:** duración de 600 a 800 ms con
  *easing* suave `power2.out`, desplazando el contenido verticalmente 24 px
  hacia arriba mientras la opacidad transiciona de 0 a 1.
- **Scroll inercial con Lenis:** configurado con una duración de 1.2 segundos y
  *easing* `(t) => Math.min(1, 1.001 - Math.pow(2, -10 * t))` para una sensación
  sedosa al navegar por la página.

### 7.2. Cumplimiento de accesibilidad de movimiento (`prefers-reduced-motion`)

El sistema detecta automáticamente la preferencia de reducción de movimiento del
sistema operativo del usuario:

- La hoja [`styles/global.css`](styles/global.css) fuerza la duración de
  animaciones a `0.01ms` e inhabilita el comportamiento de *smooth scroll*.
- En los scripts de JavaScript de inicialización de GSAP y Lenis, se verifica
  `window.matchMedia('(prefers-reduced-motion: reduce)').matches` para detener
  los disparadores de ScrollTrigger y permitir el desplazamiento nativo estándar.
