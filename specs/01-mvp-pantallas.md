# SPEC 01 — MVP visual de Arcade Vault (5 pantallas)

> **Status:** Borrador
> **Depends on:** ninguna
> **Date:** 2026-10-29
> **Objective:** Implementar en Next.js las 5 pantallas de `references/templates/` (Biblioteca, Detalle, Reproductor, Auth y Salón de la Fama) con datos mock y sin ningún juego real.

---

## Por qué existe esta spec

El repo es un scaffold de Create Next App. Las plantillas en `references/templates/` son un prototipo en React con scripts en el navegador (hash JSON como router, globals en `window`). Esta spec las traduce al App Router con rutas reales y TypeScript, sin cambiar el diseño. El tema y las fuentes ya están portados en `app/globals.css` y `app/layout.tsx` (commit `c1b5cb2`).

---

## Alcance

**Dentro:**

- Cinco rutas del App Router: `/`, `/juegos/[id]`, `/juegos/[id]/jugar`, `/auth`, `/salon`.
- Nav (con menú móvil) y footer compartidos desde `app/layout.tsx`.
- Biblioteca con búsqueda por nombre y filtro por categoría, tarjetas con efecto tilt.
- Detalle de juego con ranking de 10 puntuaciones semilla.
- Reproductor copiado tal cual de `references/templates/reproductor.jsx` (HUD, pausa, fin de juego, modal de guardado, arena CRT decorativa y puntaje simulado por intervalo). Es un placeholder: la arena CRT es el punto donde luego se montarán las pantallas de los juegos reales.
- Auth con pestañas "Iniciar sesión" / "Crear cuenta". Es solo visual: no guarda sesión.
- Salón de la Fama con pestañas por juego, podio y tabla de 12 filas semilla.
- Datos ficticios en `app/data.ts`, portados de `data.jsx`. Es el único punto de acceso a los datos: eventualmente se reemplazará por una base de datos sin tocar las pantallas.
- Estado 404 (`notFound()`) cuando `[id]` no existe.
- Diseño de UI guiado por `/frontend-design` y por las plantillas como referencia visual.

**Fuera de alcance (para futuras specs):**

- Cualquier juego jugable (lógica, canvas, input de teclado o táctil).
- Autenticación real (backend, hashing, sesiones, OAuth). Los botones GOOGLE y GITHUB son decorativos.
- Base de datos o API para juegos y puntuaciones.
- Fila "Tu mejor marca" del Salón (requiere sesión de usuario).
- Mostrar en Salón o Detalle las puntuaciones guardadas desde el reproductor.
- Sistema real de créditos (el contador "CRÉDITOS · 03" es texto fijo).
- Tests automatizados (no hay runner configurado).

---

## Modelo de datos

```ts
// app/data.ts — datos ficticios; eventualmente vendrán de una base de datos
export type Category = "ARCADE" | "PUZZLE" | "SHOOTER" | "VERSUS";
export type CardColor = "cyan" | "magenta" | "yellow" | "green";

export type Game = {
  id: string;        // slug de URL, ej. "bloque-buster"
  title: string;     // "BLOQUE BUSTER"
  short: string;     // descripción de tarjeta
  long: string;      // descripción de detalle
  cat: Category;
  cover: string;     // clase CSS en globals.css, ej. "cover-bricks"
  color: CardColor;  // color del botón JUGAR
  best: number;
  plays: string;     // "12.4K"
};

export type ScoreRow = { rank: number; name: string; score: number; date: string };

export const GAMES: Game[];                 // los 8 juegos de data.jsx
export const CATS: ("TODOS" | Category)[];  // filtro de la biblioteca
export function seededScores(seed: number, count?: number): ScoreRow[]; // determinista
```

Persistencia: el reproductor, al guardar, agrega `{ game, score, name, at }` a `localStorage["av_scores"]` dentro de un `try/catch`. Nada lo lee en este MVP.

Convenciones:

- Los números se formatean con `toLocaleString("es-ES")`.
- `seededScores` usa el mismo generador congruencial que la plantilla, para que Detalle y Salón sean deterministas entre renders (sin errores de hidratación).
- Semillas: Detalle `id.length * 17 + 3` con 10 filas; Salón `id.length * 23 + 7` con 12 filas.

---

## Plan de implementación

Antes de escribir código Next.js, leer la guía relevante en `node_modules/next/dist/docs/` (App Router, `cacheComponents`, `params` asíncrono, `generateStaticParams`).

1. Crear `app/data.ts` con los tipos, `GAMES`, `CATS` y `seededScores`. Verificar con `npx tsc --noEmit`.
2. Crear `components/nav.tsx` (client component: menú móvil, enlace activo vía `usePathname`) y el footer. Integrarlos en `app/layout.tsx` dentro de `<main className="av-main">`. La app compila y muestra nav y footer.
3. Reemplazar `app/page.tsx` con la Biblioteca. Crear `components/game-card.tsx` (client, tilt) y `components/library.tsx` (client, búsqueda y chips). Cada tarjeta enlaza a `/juegos/[id]`. Incluir el estado "NO HAY RESULTADOS".
4. Crear `app/juegos/[id]/page.tsx` (Detalle) con `generateStaticParams`, `notFound()` y los botones a `/juegos/[id]/jugar` y `/`.
5. Crear `app/auth/page.tsx` y `components/auth-form.tsx` (client). Las pestañas muestran u ocultan el campo de correo. El envío y "JUGAR COMO INVITADO" navegan a `/` con `useRouter`. No se guarda nada.
6. Crear `app/salon/page.tsx` y `components/hall-of-fame.tsx` (client: pestañas por juego, podio, tabla). No incluye la fila "Tu mejor marca". El botón vuelve a `/`.
7. Crear `app/juegos/[id]/jugar/page.tsx` y `components/game-player.tsx` (client), portando `reproductor.jsx` sin cambios de diseño: HUD, pausa, FIN, modal de fin de juego con input de iniciales (máx. 10, mayúsculas, valor inicial `INVITADO`) y guardado en `av_scores`. SALIR vuelve a `/juegos/[id]`.
8. Revisar los estilos de las 5 pantallas en 375 px, 768 px y 1280 px. Ajustar `app/globals.css` solo si falta alguna clase usada por los componentes.
9. Ejecutar `npm run lint` y `npm run build`, y corregir lo que falle.

---

## Criterios de aceptación

- [ ] `npm run build` y `npm run lint` terminan sin errores.
- [ ] `/` muestra 8 tarjetas de juego bajo el título "ARCADE VAULT".
- [ ] Escribir "caí" en el buscador deja solo la tarjeta CAÍDA (búsqueda sin distinguir mayúsculas).
- [ ] Elegir el chip SHOOTER muestra exactamente INVASORES y ROCAS.
- [ ] Una búsqueda sin coincidencias muestra "NO HAY RESULTADOS".
- [ ] Clic en una tarjeta o en su botón JUGAR navega a `/juegos/<id>`.
- [ ] `/juegos/<id>` muestra título, descripción larga, etiquetas, partidas, mejor global y 10 filas de ranking.
- [ ] `/juegos/no-existe` devuelve la página 404 de Next.
- [ ] "JUGAR AHORA" navega a `/juegos/<id>/jugar`; "VOLVER AL VAULT" navega a `/`.
- [ ] En el reproductor el puntaje aumenta solo; PAUSA lo detiene y muestra "EN PAUSA"; REANUDAR lo reanuda.
- [ ] FIN abre el modal "FIN DEL JUEGO" con la puntuación final.
- [ ] GUARDAR PUNTUACIÓN agrega una entrada a `localStorage["av_scores"]` y muestra "PUNTUACIÓN GUARDADA".
- [ ] El input de iniciales no admite más de 10 caracteres y los pasa a mayúsculas.
- [ ] JUGAR DE NUEVO reinicia puntaje, vidas y nivel; SALIR vuelve al detalle del juego.
- [ ] `/auth` alterna entre pestañas; "CREAR CUENTA" muestra el campo de correo y "INICIAR SESIÓN" lo oculta.
- [ ] Enviar el formulario o pulsar "JUGAR COMO INVITADO" navega a `/` sin escribir en `localStorage`.
- [ ] `/salon` muestra podio (1.º, 2.º, 3.º) y 12 filas; cambiar de pestaña cambia los datos.
- [ ] Recargar `/salon` y `/juegos/<id>` muestra las mismas puntuaciones, sin warning de hidratación en consola.
- [ ] El nav marca como activo "Biblioteca" en `/`, `/juegos/*` y "Salón de la Fama" en `/salon`.
- [ ] A 375 px el menú hamburguesa abre y cierra el panel móvil y no hay scroll horizontal.
- [ ] La consola del navegador no muestra errores en ninguna de las 5 rutas.

---

## Decisiones tomadas y descartadas

- **Sí:** rutas reales del App Router. URLs compartibles y es lo idiomático en Next.
- **No:** réplica del router por hash con `useState`. Solo existía por ser un prototipo de un solo HTML.
- **Sí:** auth solo visual, sin sesión. Una auth real es una spec propia y el MVP es visual.
- **No:** guardar `av_user` en `localStorage` como hace la plantilla. Sin sesión no tiene función y obligaría a leer `localStorage` en el nav (riesgo de hidratación).
- **Sí:** quitar la fila "Tu mejor marca" del Salón. Depende de un usuario con sesión.
- **Sí:** datos ficticios en `app/data.ts`, con `seededScores` determinista. Todas las pantallas importan de ahí, de modo que cambiar la fuente a una base de datos no afecta a los componentes. Sin BD ni API en este MVP.
- **No:** `lib/data.ts`. El usuario prefirió `app/data.ts`; no es una ruta porque Next solo expone carpetas con `page.tsx` y archivos especiales.
- **Sí:** copiar el reproductor de la plantilla sin rediseñarlo, como placeholder de la pantalla de juego. Mientras no haya juegos reales permite validar HUD, pausa y modal, y la arena CRT queda como el espacio donde se integrarán los juegos.
- **No:** rediseñar o abstraer el reproductor para alojar juegos reales (props, contrato de juego). Eso se define en la spec que implemente el primer juego.
- **No:** lógica de juegos reales. El usuario lo excluyó explícitamente.
- **Sí:** guardar puntuaciones en `localStorage["av_scores"]` aunque nada las lea todavía. Deja el flujo del modal completo para una spec posterior.
- **Sí:** reutilizar el CSS ya portado en `app/globals.css`. Se ajusta solo si falta una clase.
- **Sí:** componentes client solo donde hay interacción (nav, biblioteca, tarjeta, auth, salón, reproductor). Las páginas siguen siendo server components.

---

## Riesgos identificados

| Riesgo | Mitigación |
| ------ | ---------- |
| `cacheComponents: true` cambia la semántica de datos dinámicos y `params` | Leer la guía en `node_modules/next/dist/docs/` antes de implementar; usar `generateStaticParams` para `[id]`. |
| Error de hidratación por `Math.random` en el reproductor | El puntaje solo cambia dentro de `useEffect`; el render inicial empieza en 0. |
| Falta de alguna clase CSS de la plantilla en `globals.css` (por ejemplo `cover-*`) | El paso 8 revisa las 5 pantallas y añade lo que falte copiándolo de `references/templates/styles.css`. |
| Los enlaces del nav y de las tarjetas usan `onClick` en la plantilla | Usar `<Link>` de `next/link` para mantener accesibilidad y prefetch. |

---

## Lo que **no** está en esta spec

- Juegos reales (ninguno de los 8).
- Autenticación real, sesión de usuario u OAuth.
- Backend, base de datos o API de puntuaciones.
- Fila "Tu mejor marca" y lectura de `av_scores`.
- Sistema de créditos.
- Tests automatizados.

Cada uno, si se aborda, va en su propia spec.
