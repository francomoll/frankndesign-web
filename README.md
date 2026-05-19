# FrankNDesign — Web

Mockup premium para el sitio de FrankNDesign (virtual production / themed entertainment).

## 🚀 Quick start

```bash
python3 -m http.server 8000
# abrir http://localhost:8000
```

Stack flat: HTML + CSS + JS en `index.html`. Sin build, sin npm. Librerías por CDN (GSAP + ScrollTrigger + Lenis).

## 📚 Documentación

- **[HANDOFF.md](./HANDOFF.md)** ← leé esto primero (onboarding, stack, paleta, mejoras pendientes)
- **[CLAUDE.md](./CLAUDE.md)** ← contexto para Claude Code (se carga solo si lo usás)

## 📁 Estructura

```
.
├── index.html         # Todo el código (~900 líneas)
├── HANDOFF.md         # Doc principal para el equipo
├── CLAUDE.md          # Memoria de Claude Code
└── assets/
    ├── hero-frames/   # Secuencia del hero (64 webp, 7.4 MB)
    └── *.webp         # Logo, equipo, posters
```

## 🚢 Deploy

Cualquier static host (Vercel, Netlify, Cloudflare Pages, S3). Drag & drop la carpeta entera. Detalle en `HANDOFF.md`.
