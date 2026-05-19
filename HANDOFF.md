# FrankNDesign — Web Handoff

Premium single-page mockup para FrankNDesign (virtual production / themed entertainment). Stack flat: **HTML + CSS + JS plano, sin build step**. Librerías por CDN.

> Última actualización: 2026-05-18

---

## 1. Cómo correrlo

```bash
cd /ruta/a/frankdesign
python3 -m http.server 8000
# abre http://localhost:8000
```

⚠️ **No abrir con `file://`** — algunos navegadores bloquean el canvas o el preloader.

Cualquier static host sirve (Vercel, Netlify, S3, GitHub Pages, Cloudflare Pages). Drag & drop la carpeta entera.

---

## 2. Estructura

```
frankdesign/
├── index.html                # TODO el código (HTML + CSS + JS inline)
├── HANDOFF.md                # este archivo
└── assets/
    ├── hero-frames/          # 64 webp 1600x900 — secuencia del hero (7.4 MB)
    │   ├── 00.webp
    │   ├── ...
    │   └── 63.webp
    ├── hero-frank.webp       # poster del hero (fallback)
    ├── FrankNDesign Logo.webp
    ├── FrankNDesign Virtual Production*.webp  # 4 imágenes (no usadas en la página actual)
    ├── francisco.webp        # team
    ├── jeremy.webp
    ├── carianne.webp
    └── patrick.webp
```

**Todo el código vive en `index.html` (~900 líneas).** No hay imports, no hay build.

---

## 3. Stack

| Librería | Versión | Uso | Carga |
|---|---|---|---|
| GSAP | 3.12.5 | Animaciones, timelines | CDN cdnjs |
| ScrollTrigger | 3.12.5 | Scroll-driven (canvas scrub, reveals, parallax) | CDN cdnjs |
| Lenis | 1.1.13 | Smooth scroll desktop | CDN unpkg |

Sin npm, sin webpack, sin TypeScript. Editás `index.html` directo.

**Tipografías (Google Fonts):**
- `Bebas Neue` → headings (variable `--font-d`)
- `DM Sans` → body, navegación (variable `--font-b`)

---

## 4. Paleta de marca

Definida en `:root` (líneas ~11–28 de `index.html`):

| Variable | Valor | Uso |
|---|---|---|
| `--black` | `#111111` | fondos primarios, hero |
| `--white` | `#ffffff` | tipografía sobre oscuro |
| `--off-white` / `--gray-50` | `#f4f4f4` | fondos secundarios (intro, process) |
| `--orange` | `#e30613` | **acento de marca (rojo FrankNDesign)** — CTAs, eyebrows |
| `--orange-hover` | `#ff1a2a` | hover de CTAs |
| `--orange-pale` | `#fdecee` | fondos suaves rojos (hover svc, tags) |
| `--muted` | `#8a8a8a` | textos secundarios |
| `--border` | `#e6e6e6` | líneas separadoras |

> ⚠️ Las variables se llaman `--orange*` por legado del primer mockup. **El valor es rojo.** Si renombrás, hacer find/replace en TODO el archivo.

> Confirmar tono exacto del rojo con el cliente. `#e30613` es un rojo estándar "industrial".

---

## 5. Estructura de la página

| Sección | ID | Comentario |
|---|---|---|
| Nav | `#nav` | sticky, cambia a fondo blanco al scrollear (`#nav.scrolled`) |
| Hero scroll-driven | `#hero-scroll` | canvas con secuencia + 4 escenas de texto |
| Stats | `#stats` | 4 números animados con counter |
| Intro / About | `#intro` | bloque 2 col con imagen + texto |
| Work (portfolio) | `#work` | grid masonry de 7 cards |
| Brands marquee | `#brands` | scroll infinito de logos |
| Services | `#services` | grid 3×2 de 6 categorías |
| Process | `#process` | 5 pasos + imágenes |
| Case Study Tokyo Streets | `#casestudy` | 5 fases editoriales con galería |
| Team | `#team` | 4 fotos del leadership |
| CTA | `#cta` | call to action final |
| Footer | `footer` | 4 columnas + socials |

Hay un **lightbox** global (`#lb`) que se activa con cualquier elemento que tenga clase `.lb-trigger` + `data-src` + `data-cap`.

---

## 6. Hero scroll-driven (la parte compleja)

**Concepto:** estilo Apple iPhone/AirPods. Una `<canvas>` que pinta el frame correspondiente al progreso del scroll. La sección es `500vh` de alto (`350vh` en mobile) con `position: sticky` para pinear el viewport mientras se scrollea.

### Cómo funciona

1. JS preload de **24 frames críticos** (frames 0-23) mientras el preloader muestra %.
2. Apenas terminan los críticos → fade out del preloader.
3. Los 40 frames restantes cargan en background.
4. ScrollTrigger anima `state.frame` de 0 → 63 ligado al scroll.
5. En cada update, `draw()` pinta el frame al canvas con `object-fit: cover` calculado a mano.
6. En paralelo, las 4 `.hero-scene` (texto) hacen crossfade según el rango del scroll.

### Variables clave (en el IIFE `heroScroll`)

```js
const FRAME_COUNT      = 64;   // total de frames
const PRELOAD_CRITICAL = 24;   // frames antes de soltar el preloader
const FRAME_PATH       = i => `assets/hero-frames/${String(i).padStart(2,'0')}.webp`;
```

### Cómo cambiar el copy del hero

Buscar `<section id="hero-scroll">` (línea ~360 aprox). Adentro hay 4 `<div class="hero-scene" data-scene="N">`. Editar texto directamente. **No tocar las clases ni el atributo `data-scene`** porque la animación se ata a esos.

### Cómo reemplazar la secuencia de imágenes

Tenés que generar una nueva secuencia de 64 frames WebP @ 1600×900, numerados `00.webp` a `63.webp`, en `assets/hero-frames/`.

**Pipeline desde un .mp4 o secuencia JPG (macOS):**
```bash
# Requiere: brew install webp ffmpeg

# 1) Extraer frames del video original (saltea cada N para llegar a ~64 total)
mkdir -p /tmp/raw
ffmpeg -i source.mp4 -vf "fps=15" /tmp/raw/frame-%06d.jpg

# 2) Resize + convertir cada uno a WebP, renumerando 00-63
rm -rf assets/hero-frames && mkdir -p assets/hero-frames
TMP=$(mktemp -d)
COUNT=$(ls /tmp/raw/*.jpg | wc -l)
STEP=$(( (COUNT + 63) / 64 ))
i=0
for src in /tmp/raw/frame-*.jpg; do
  src_idx=$(echo "$src" | grep -oE '[0-9]+' | head -1)
  if [ $((10#$src_idx % STEP)) -eq 0 ] && [ $i -lt 64 ]; then
    sips -z 900 1600 "$src" --out "$TMP/r.jpg" >/dev/null 2>&1
    cwebp -q 82 -m 6 -quiet "$TMP/r.jpg" -o "$(printf 'assets/hero-frames/%02d.webp' $i)"
    i=$((i+1))
  fi
done
rm -rf "$TMP" /tmp/raw
```

Si la nueva secuencia tiene más/menos frames, actualizar `FRAME_COUNT` en JS.

---

## 7. Animaciones (qué hace cada cosa)

| Feature | Cómo se activa | Dónde está |
|---|---|---|
| Smooth scroll Lenis | automático en desktop | top del `<script>` |
| Reveals al scroll | clases `.reveal`, `.reveal-l`, `.reveal-r`, `.reveal-scale` + `.d1`..`.d6` para delay | `revealMap` en JS |
| Hero scrub | automático con canvas + ScrollTrigger | función `heroScroll()` |
| Stats counter | automático al entrar al viewport | bloque "STATS COUNTER" |
| Brands marquee | scroll infinito loop, pausa hover | bloque "BRANDS MARQUEE" |
| Parallax intro/work | `yPercent: -10` ligado a scroll | bloque "PARALLAX" |
| Magnetic CTAs | hover sigue el cursor con offset | bloque "MAGNETIC CTA" — desactivado en mobile |
| Lightbox galería | click en cualquier `.lb-trigger` | función IIFE al final |
| Mobile menu | burger toggle full-screen overlay | bloque "MOBILE BURGER MENU" |

---

## 8. Responsive

Breakpoint principal: **`@media (max-width: 900px)`**.

- Hero: misma animación scroll-scrub, altura `350vh` (en lugar de `500vh`).
- Nav: links se convierten en overlay full-screen, se accede con el burger.
- Stats: 4 col → 2×2.
- Services: 3 col → 2 col (1 col en `<540px`).
- Team: 4 col → 2 col.
- Intro/Process/Case-study: pasan a 1 columna.
- Lenis se **desactiva** en mobile (touch scroll nativo es mejor).
- Magnetic buttons se desactivan.
- Los frames del hero igual se cargan (7.4 MB) — si quieren un fallback más liviano en mobile, ver "Mejoras pendientes".

---

## 9. Mejoras pendientes (backlog)

### Alta prioridad
- [ ] **Copy de las 4 escenas del hero** — son borrador, validar con cliente.
- [ ] **Versión real del logo** — confirmar si el `assets/FrankNDesign Logo.webp` es la versión oficial vectorial. El nav usa filtro CSS `invert` para mostrarlo blanco sobre el hero — si el logo no es monocromo, dará feo. Solución: dos archivos (logo-white.webp + logo-dark.webp) y swap por clase.
- [ ] **Verificar rojo de marca** — actualmente `#e30613`. Pedir el HEX oficial.
- [ ] **Imágenes del portfolio + case study** apuntan a `https://frankndesign.com/wp-content/uploads/2026/03/...` (remotas del sitio actual). Convendría descargarlas a `assets/` y servir local — más rápido + sin dependencia del dominio viejo.

### Media prioridad
- [ ] **Fallback mobile más liviano para el hero** — opciones: (a) reducir a 32 frames en mobile, (b) usar la versión video mp4 en `<video autoplay loop>` solo en mobile.
- [ ] **Copy de servicios** — la lista actual es densa (5 categorías × 5-6 items = 28 servicios). Considerar mostrar 3 destacados por categoría con "+ more" expandible.
- [ ] **Idioma** — todo en inglés. Si el público objetivo incluye LatAm/España, hacer versión ES.
- [ ] **SEO** — falta meta description, OpenGraph, Twitter Cards, JSON-LD `Organization`.
- [ ] **Favicon** — no hay.
- [ ] **404 page** — no hay.

### Baja prioridad / nice-to-have
- [ ] Lottie animations en sección Process (iconos animados por paso).
- [ ] Custom cursor (anillo + punto que sigue al mouse).
- [ ] Page transitions con barba.js o algo similar si crece a multi-página.
- [ ] Tests visuales con Playwright en CI.

---

## 10. Performance

| Métrica | Estimado |
|---|---|
| HTML+CSS+JS (gzipped) | ~12 KB |
| CDN libs (GSAP+ScrollTrigger+Lenis, gzipped) | ~52 KB |
| Hero secuencia completa | 7.4 MB |
| Preloader crítico (24 frames) | ~2.8 MB |
| LCP esperado en 4G | ~2.5s |
| LCP esperado en WiFi | <1s |

### Si necesitan optimizar más
- Servir `assets/hero-frames/*.webp` desde un CDN (Cloudflare, BunnyCDN) con cache largo.
- Considerar AVIF en lugar de WebP (~30% más chico) — soporte: Safari 16+, Chrome 85+, Firefox 113+.
- Lazy-load el hero canvas: arrancar la carga solo cuando el usuario se acerca al hero (si hay otra cosa arriba — actualmente el hero es lo primero).

---

## 11. Deploy

Cualquier static hosting funciona:

**Vercel:**
```bash
npx vercel --prod
```

**Netlify drag & drop:** arrastrar la carpeta entera a https://app.netlify.com/drop

**S3 + CloudFront:**
```bash
aws s3 sync . s3://tu-bucket/ --exclude "*.md" --exclude ".git/*"
```

**Cache headers recomendados (`.htaccess` / CDN config):**
```
*.html       → no-cache
*.webp       → public, max-age=31536000, immutable
*.css *.js   → public, max-age=31536000, immutable (no aplicamos hoy porque todo está inline en HTML)
```

---

## 12. Contacto / decisiones pendientes

Antes de continuar conviene confirmar con cliente:

1. ✅ Logo definitivo (¿hay versión SVG? ¿variante blanca?)
2. ✅ Rojo exacto de marca
3. ✅ Copy final del hero (4 escenas)
4. ✅ ¿Hay versiones definitivas de las fotos del equipo? (las actuales son OK pero pueden tener mejores)
5. ✅ ¿Multi-idioma (ES) o solo EN?
6. ✅ ¿Qué proyectos del portfolio se pueden mostrar legalmente y con qué crédito? (Universal, Nintendo, etc. requieren confirmar permisos de imagen y uso de marcas ™)

---

**Cualquier duda revisar el archivo `index.html` — está comentado por secciones.** Las animaciones GSAP están agrupadas en bloques marcados con `// ── NOMBRE ──`.
