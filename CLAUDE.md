# CLAUDE.md — Memoria del proyecto FrankNDesign

> Este archivo lo lee Claude Code automáticamente al abrir el proyecto. Es la "memoria" entre sesiones — decisiones tomadas, contexto del cliente, gotchas, y dónde quedamos. Para handoff humano leer `HANDOFF.md`.

---

## Contexto

- **Cliente:** FrankNDesign (estudio de virtual production / themed entertainment, Clermont FL).
- **Trabajando:** Franco Moll (info@francomoll.com).
- **Proyecto:** redesign premium del sitio público (actualmente live en frankndesign.com).
- **Fecha inicio:** 2026-05-18.
- **Forma de trabajo:** iteración rápida sobre `index.html` con preview live. Franco escribe en español, a veces con typos — interpretar la intención, no pedir clarificación si es obvio.

---

## Decisiones arquitectónicas (NO revisitar sin razón)

| Decisión | Razón |
|---|---|
| **Stack flat HTML/CSS/JS, sin build** | Franco eligió esto en plan inicial. Iteración rápida, deploy a cualquier static host. |
| **GSAP + ScrollTrigger + Lenis vía CDN** | Premium feel sin npm. |
| **Remotion descartado** | No aplica (renderiza video con React, no es lib de animación web). |
| **Three.js descartado** | Overkill para esta versión. |
| **Vite/Next descartado** | Complejidad innecesaria para single-page. |
| **Hero como canvas + secuencia de imágenes** | Mejor scrub que `video.currentTime` (keyframes esparcidos del H.264 lo hacen saltón). Técnica de Apple. |
| **64 frames WebP @ 1600×900** | Sweet spot peso/calidad — 7.4MB total, indistinguible de 192 frames JPG @ 1920×1080. |
| **Lenis OFF en mobile** | Touch nativo se siente mejor. |
| **Magnetic CTAs OFF en mobile** | Sin hover. |
| **Mobile: misma animación scroll-scrub** | Franco lo pidió explícitamente. Altura 350vh en mobile vs 500vh desktop. |

---

## Convenciones / gotchas que muerden

1. **Variables CSS se llaman `--orange*` pero el valor es ROJO** (`#e30613`). Legado del primer mockup. Si renombrás, hacer find/replace en todo `index.html`. No mezclar nombre semántico vs valor.
2. **`section { padding: 6rem 3.5rem; }` aplica globalmente** — `#hero-scroll` necesita `padding: 0 !important` para ser full-bleed.
3. **`.reveal*` no debe tener `opacity:0` por defecto** — solo con `html.preloading` o cuando GSAP ya tomó control. Si no, se rompe sin JS.
4. **Preloader bloquea scroll con `html.preloading { overflow: hidden }`** seteado inicialmente inline en `<html lang="en" class="preloading">`. JS lo remueve al dismissear.
5. **Logo nav usa filter `brightness(0) invert(1)` sobre el hero** para verse blanco. Si el logo no es monocromo se va a romper visualmente — pendiente versión dual (logo-white + logo-dark).
6. **`patick.webp` se renombró a `patrick.webp`** (typo original en filename).
7. **`hero-section-frank-design.mp4` ya no existe en `/assets/`** — Franco lo movió afuera para no inflar el repo. La carpeta `assets/hero-section1/` también se borró (eran los JPG originales, ahora reemplazados por `assets/hero-frames/*.webp`).
8. **`window.load`** se usa para el boot script (no DOMContentLoaded) — necesitamos que los CDN deferred (GSAP) estén listos.

---

## Estado actual (al 2026-05-18)

### ✅ Hecho
- Auditoría de contenido del sitio actual (frankndesign.com).
- Mockup base reescrito en single-file `index.html` (~900 líneas).
- Paleta swap: naranja `#e8620a` → rojo `#e30613` + grises/negro/blanco.
- Logo real integrado en nav (con invert filter) y footer.
- Hero rediseñado: scroll-driven canvas estilo Apple, 4 escenas de texto con crossfade.
- Secuencia hero optimizada: 64 frames WebP @ 1600×900 → 7.4 MB total (de 68 MB original).
- Preloader minimal con barra de progreso (logo + % + barra roja).
- Mobile responsive completo:
  - Hero conserva scroll-scrub (altura 350vh).
  - Burger menu overlay full-screen.
  - Stats 4→2×2, services 3→2 cols (1 col <540px), team 4→2 cols.
- Assets locales:
  - Hero: poster `hero-frank.webp` + secuencia `hero-frames/00-63.webp`
  - Team: francisco, jeremy, carianne, patrick (.webp)
  - Logo: `FrankNDesign Logo.webp`
- INTRO usa imágenes de proyectos remotas (Hagrid's + modelo 3D multidisciplinario), no las fotos del equipo.
- Animaciones premium: reveals con stagger, counter en stats, parallax intro/work, marquee brands con GSAP, magnetic CTAs, pin reveals en case study.
- Backdrop direccional + text-shadow en cada hero scene para legibilidad sobre el video.
- `HANDOFF.md` creado para pasar el proyecto a otro miembro del equipo.

### 🟡 Pendiente — decisiones del cliente
- [ ] Confirmar tono exacto del rojo de marca (hoy `#e30613`).
- [ ] Confirmar si el logo actual (`FrankNDesign Logo.webp`) es monocromo. Si no → necesitamos `logo-white.webp` + `logo-dark.webp` y swap por clase.
- [ ] Validar copy de las 4 escenas del hero (es borrador).
- [ ] Confirmar permisos de marcas/imágenes (Universal, Nintendo, Warner, etc.) para uso comercial en el sitio.
- [ ] ¿Multi-idioma (versión ES) o solo EN?

### 🟡 Pendiente — trabajo técnico
- [ ] Descargar imágenes del portfolio + case study (hoy apuntan a `frankndesign.com/wp-content/uploads/2026/03/...`) y servir local.
- [ ] SEO: meta description, OG, Twitter Card, JSON-LD Organization.
- [ ] Favicon.
- [ ] Página 404.
- [ ] Considerar Lottie en sección Process.
- [ ] Fallback hero más liviano para mobile (opcional — hoy carga los 7.4 MB también).

---

## Lecciones aprendidas del proyecto

- Franco prefiere ver progreso visual rápido en el preview panel — por eso CADA edición se reporta como "visible en el panel Launch preview".
- Cuando algo se ve mal en pantalla, Franco manda screenshot — leer atentamente y fixear sin pedir aclaración si la causa es obvia (ej: "se corta" → padding global mordiendo el ancho).
- Recomendar paleta/aproximaciones concretas cuando pregunta abierto ("¿qué creés mejor?") — no devolverle la decisión.
- Las preguntas con AskUserQuestion deben tener opciones realmente diferentes y útiles. Franco ya descartó muchas opciones (Remotion, Three.js, Vite) — no las revivas.

---

## Comandos útiles

```bash
# Correr local
cd /Users/francomoll/Desktop/frankdesign && python3 -m http.server 8000

# Re-generar la secuencia de hero desde un nuevo .mp4 fuente
# (instalar: brew install webp ffmpeg)
mkdir -p /tmp/raw
ffmpeg -i source.mp4 -vf "fps=15" /tmp/raw/frame-%06d.jpg
rm -rf assets/hero-frames && mkdir -p assets/hero-frames
TMP=$(mktemp -d); i=0
for src in /tmp/raw/frame-*.jpg; do
  [ $i -ge 64 ] && break
  sips -z 900 1600 "$src" --out "$TMP/r.jpg" >/dev/null 2>&1
  cwebp -q 82 -m 6 -quiet "$TMP/r.jpg" -o "$(printf 'assets/hero-frames/%02d.webp' $i)"
  i=$((i+1))
done
rm -rf "$TMP" /tmp/raw

# Ver peso de assets
du -sh assets/*

# Buscar referencias a un asset
grep -n "patrick\|francisco" index.html
```

---

## Log de cambios mayores

| Fecha | Cambio | Por |
|---|---|---|
| 2026-05-18 | Auditoría inicial del sitio actual + mockup base | Claude |
| 2026-05-18 | Swap paleta naranja → rojo | Claude |
| 2026-05-18 | Hero scroll-driven con `<video>` mp4 | Claude |
| 2026-05-18 | Hero migrado a canvas + secuencia 192 JPG | Claude |
| 2026-05-18 | Logo real en nav/footer | Claude |
| 2026-05-18 | INTRO imágenes: team photos → proyectos | Claude |
| 2026-05-18 | Gradients direccionales + text-shadow para legibilidad hero | Claude |
| 2026-05-18 | `#hero-scroll` full-bleed (override padding global) | Claude |
| 2026-05-18 | Preloader con barra y % | Claude |
| 2026-05-18 | Reducción 192 → 64 frames (FRAME_STEP=3) | Claude |
| 2026-05-18 | Mobile: stack vertical de escenas (luego revertido) | Claude |
| 2026-05-18 | Mobile: vuelve a scroll-scrub completo, altura 350vh | Claude |
| 2026-05-18 | Optimización: 64 JPG @ 1920×1080 → 64 WebP @ 1600×900, 23MB → 7.4MB | Claude |
| 2026-05-18 | Borrada carpeta `assets/hero-section1/`; mp4 movido por Franco | Franco + Claude |
| 2026-05-18 | `HANDOFF.md` + `CLAUDE.md` creados | Claude |

> Mantener este log breve. Para detalles, `git log` o el plan archivado en `~/.claude/plans/haz-una-audotoria-de-crispy-hammock.md`.
