# DEBUG_LOG — Galería de Proyectos Técnicos

## Registro de errores

### Error #1 — Rol ARIA redundante en `<nav>`

- **Herramienta que lo detectó:** W3C Nu HTML Checker
- **Mensaje exacto:** *"The navigation role is unnecessary for element nav."*
- **Captura antes:** `capturas/error1-antes.png`
- **Causa raíz (análisis propio):**
  El elemento `<nav>` ya tiene el rol semántico `navigation` de forma implícita según la
  especificación HTML. Al escribir `role="navigation"` de forma explícita, se duplica
  información que el navegador y los lectores de pantalla ya deducen del propio tag.
  No rompe la accesibilidad, pero el validador lo reporta como advertencia. Lo detecté
  al pasar el HTML por el Nu Checker antes de empezar a tocar el CSS.
- **Corrección manual aplicada:**
  Eliminé el atributo `role="navigation"` del `<nav>` conservando `aria-label="Navegación principal"`,
  porque el `aria-label` sí aporta información nueva (distingue este nav de otros futuros).
  Verifiqué en la especificación WAI-ARIA que `<nav>` mapea a `role=navigation` por defecto.
- **Captura después:** `capturas/error1-despues.png`
- **Verificación:** W3C → 0 errores y 0 advertencias sobre roles.

---

### Error #2 — Enlace roto a `#nosotros`

- **Herramienta que lo detectó:** WAVE (extensión Chrome)
- **Mensaje exacto:** *"Broken same-page link: A link contains a target that doesn't exist (#nosotros)"*
- **Captura antes:** `capturas/error2-antes.png`
- **Causa raíz (análisis propio):**
  En el `<nav>` puse cuatro enlaces (`#inicio`, `#proyectos`, `#nosotros`, `#contacto`) pero al
  maquetar solo construí las secciones `#inicio`, `#proyectos` y el `<footer id="contacto">`.
  La sección "Nosotros" nunca existió, así que el ancla quedó huérfana. Lo confirmé
  haciendo clic con teclado (Tab + Enter) y viendo que el foco no se movía.
- **Corrección manual aplicada:**
  Eliminé el enlace `<a href="#nosotros">Nosotros</a>` del nav en lugar de crear una sección
  vacía, porque la práctica pide una galería de proyectos, no un sitio institucional completo.
  Después usé Ctrl+F en el HTML para confirmar que no quedaran otras referencias a `#nosotros`.
- **Captura después:** `capturas/error2-despues.png`
- **Verificación:** WAVE → 0 errores de "Broken same-page link". Probé de nuevo la navegación
  por teclado y los tres enlaces restantes saltan correctamente.

---

### Error #3 — Enlaces nulos (`href="#"`) en "Ver proyecto"

- **Herramienta que lo detectó:** WAVE
- **Mensaje exacto:** *"Null link: A link contains a null or empty destination (#)"*
  (reportado 4 veces, uno por tarjeta)
- **Captura antes:** `capturas/error3-antes.png`
- **Causa raíz (análisis propio):**
  Usé `<a href="#">` como placeholder en los 4 botones "Ver proyecto" porque todavía no
  existían páginas de detalle. WAVE lo marca porque un `href="#"` no navega a ningún sitio
  real: para lectores de pantalla y para SEO es un enlace vacío. Semánticamente, además,
  "Ver proyecto" no es una navegación a un ancla del documento, es una acción.
- **Corrección manual aplicada:**
  Sustituí cada `<a href="#" class="project-link">` por `<button type="button" class="project-link">`.
  Elegí `<button>` en lugar de un `<a href>` real porque no tenemos páginas de proyecto
  todavía, y `button` es el elemento correcto para una acción sin destino. En el CSS tuve
  que resetear los estilos por defecto del botón (`border: 0`, `font-family: inherit`,
  `font-size: 1rem`, `cursor: pointer`) para que la apariencia fuera idéntica.
- **Captura después:** `capturas/error3-despues.png`
- **Verificación:** WAVE → 0 "Null links". Probé con teclado que los 4 botones siguen
  siendo enfocables y mantienen el `:focus-visible`.

---

### Error #4 — Contraste insuficiente en `.intro-section p` (modo claro)

- **Herramienta que lo detectó:** WAVE
- **Mensaje exacto:** *"Very low contrast — Contrast ratio: 4.45:1 (required 4.5:1)"*
  sobre el párrafo introductorio.
- **Captura antes:** `capturas/error4-antes.png`
- **Causa raíz (análisis propio):**
  El párrafo `.intro-section p` usaba `color: var(--color-text-light)` = `#64748b` sobre
  `background: var(--color-background)` = `#f5f7fa`. Calculé la relación con el contrast
  checker de WAVE y dio **4.45:1**, apenas por debajo del mínimo WCAG AA para texto normal
  (4.5:1). El error era mínimo, pero WAVE lo marcaba igual. Lo verifiqué también manualmente
  con la fórmula de luminancia relativa para asegurarme de que no era un falso positivo.
- **Corrección manual aplicada:**
  Oscurecí la variable `--color-text-light` en `:root` de `#64748b` a `#4a5568`. Con ese
  valor el contraste pasa a **7.04:1** sobre `#f5f7fa` y **7.55:1** sobre blanco. Al cambiarlo
  en la variable raíz, se corrigió también el texto de las tarjetas sin tocar cada selector.
  No usé IA para el valor: probé varios grises en el contrast checker hasta encontrar uno
  que superara 7:1 sin perder la estética.
- **Captura después:** `capturas/error4-despues.png`
- **Verificación:** WAVE → 0 contrastes fallidos. Confirmé en DevTools con el color picker
  que el ratio mostrado es 7.04:1.

---

### Error #5 — Contraste insuficiente en `.project-link` (modo oscuro)

- **Herramienta que lo detectó:** WAVE con `prefers-color-scheme: dark` simulado
- **Mensaje exacto:** *"Very low contrast — Contrast ratio: 2.54:1 (required 4.5:1)"*
  sobre el botón "Ver proyecto" y sobre los enlaces del nav en hover.
- **Captura antes:** `capturas/error5-antes.png`
- **Causa raíz (análisis propio):**
  En modo oscuro redefiní `--color-primary: #6ea8d8` (un azul claro) y dejé `.project-link`
  con `background-color: var(--color-primary)` y `color: #ffffff`. Texto blanco sobre azul
  claro = **2.54:1**, muy por debajo de WCAG AA. El mismo problema aparecía en
  `.main-nav a:hover` porque también usa `--color-primary` como fondo. No lo detecté al
  principio porque solo estaba probando la página en modo claro.
- **Corrección manual aplicada:**
  Añadí dentro del bloque `@media (prefers-color-scheme: dark)` una regla que fuerza
  `color: #121820` (casi negro, igual al fondo oscuro) cuando `--color-primary` se usa como
  fondo. También ajusté el hover para que use `--color-secondary` como fondo y mantenga el
  texto oscuro. Así el contraste pasa a **7.04:1**. La decisión de qué color usar la tomé yo
  iterando con el contrast checker de WAVE.
- **Captura después:** `capturas/error5-despues.png`
- **Verificación:** WAVE con modo oscuro simulado → 0 contrastes fallidos. Probé también
  manualmente alternando el tema del sistema operativo.

---

## 3. Resultados finales

| Herramienta | Antes | Después |
|---|---|---|
| W3C Validator | 1 advertencia (`role`) | 0 errores / 0 advertencias |
| WAVE | 6 errores (1 roto, 4 nulos, 2 contraste) | 0 errores · 0 contrastes fallidos |
| Lighthouse — Accesibilidad | <puntaje> | <puntaje ≥ 90> |
| Lighthouse — SEO | <puntaje> | <puntaje ≥ 90> |

- **Captura Lighthouse final:** `capturas/lighthouse-final.png`
- **Captura WAVE final:** `capturas/wave-final.png`

---

## 4. Bitácora de uso de IA

| Fecha | Herramienta | Prompt / Consulta | Uso que le di | % código manual |
|---|---|---|---|---|
| <fecha> | ChatGPT | "¿Por qué W3C marca role='navigation' como redundante en <nav>?" | Entendí el mapeo implícito ARIA; eliminé el atributo a mano | 100% |
| <fecha> | ChatGPT | "Interpreta este reporte de WAVE" | Identifiqué los null links y los dos contrastes; decidí las correcciones yo mismo | 100% |
| <fecha> | ChatGPT | "¿Cómo funciona container-type: inline-size?" | Entendí la diferencia con @media; escribí yo el fallback con @supports | 100% |
| <fecha> | — | (sin IA) | Redacté la meta description, los alt de las imágenes y todos los textos | 100% |

> Todo bloque de código asistido por IA quedó marcado en `styles.css` con:
> `/* IA: [consulta] → Corrección manual: [explicación] */`

---

## 5. Notas de compatibilidad (Can I Use)

- `clamp()` — soporte en todos los navegadores modernos desde 2020. Sin fallback necesario.
- `@container` / `container-type` — soporte desde Chrome 105 / Safari 16 / Firefox 110.
  Fallback aplicado con `@supports not (container-type: inline-size)` + `@media` (ver `styles.css`).
- `:focus-visible` — soporte universal desde 2022.
- `prefers-color-scheme` — soporte universal. Sin fallback necesario.
- `prefers-reduced-motion` — soporte universal. Sin fallback necesario.
- `aspect-ratio` — soporte desde Chrome 88 / Safari 15 / Firefox 89. Sin fallback necesario.
- `inset-inline-start` / `inset-block-start` — propiedades lógicas, soporte universal.

---

## 6. Notas de depuración manual

- El CSS fue escrito y ajustado a mano tras cada iteración en Live Server.
- Ningún bloque fue pegado directamente desde una IA sin adaptación y verificación en DevTools.
- Los valores de contraste (`#4a5568`, `#121820`) fueron elegidos probando en el contrast
  checker de WAVE, no sugeridos por IA.