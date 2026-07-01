# Fixes de estilo mobile-first (Android)

**Fecha:** 2026-07-01
**Estado:** Aprobado, pendiente de plan de implementación

## Contexto

Un chequeo de estilo mobile-first (foco Android) sobre `public/index.html` encontró tres problemas concretos en el CSS global (bloque `<style>`, líneas ~15-149). Los últimos commits ya corrigieron el touch target de `.btn-next`/`.btn-back` y el padding stacking en mobile; estos tres quedan pendientes.

Esta spec asume que el plan de simplificación del flujo de preguntas (`docs/superpowers/specs/2026-07-01-simplificar-flujo-encuesta-design.md`) ya está implementado, pero es independiente de él — toca únicamente CSS global que no depende del contenido de las preguntas, así que puede implementarse en cualquier orden respecto a ese plan.

## Objetivo

Corregir tres problemas de legibilidad/usabilidad en mobile Android: un bug de contraste que hace invisible el footer, un touch target por debajo del estándar Material Design en el control más repetido de la encuesta, y un color de texto secundario que falla el mínimo de contraste WCAG AA.

## Cambios

### 1. Footer invisible (bug)

`footer{color:rgba(255,255,255,.5);border-top:1px solid rgba(255,255,255,.15)}` (línea 148) renderiza texto blanco semi-transparente sobre fondo blanco — el copyright y el disclaimer de confidencialidad son ilegibles en cualquier pantalla.

- `color: rgba(255,255,255,.5)` → `var(--text3)` (una vez aplicado el fix del punto 3).
- `border-top: 1px solid rgba(255,255,255,.15)` → `1px solid var(--border)`.

### 2. Touch target de `.opcion` a 48dp

`.opcion{padding:10px 14px}` (línea 97), combinado con el radio de 16px y texto de 13px/line-height 1.5, da una altura real de ~40-42px — por debajo del mínimo Material Design de 48dp. Es el control interactivo más repetido de toda la encuesta (decenas de opciones por bloque), así que es el punto de mayor fricción táctil en Android.

- Agregar `min-height:48px` a `.opcion`, manteniendo `align-items:flex-start` para que las opciones con texto largo (dos líneas) sigan alineando bien el radio arriba.
- No tocar `.subpregunta .opcion` (padding reducido, 8px 12px) salvo que quede código muerto tras el otro plan — fuera de alcance acá.

### 3. Contraste de `--text3`

`--text3:#8a9a90` (línea 19) sobre fondo blanco da ~2.6:1 de contraste — falla WCAG AA (mínimo 4.5:1 para texto normal). Se usa en `.step-label`, `.pregunta-sub`, `.hero .confidencial`, placeholders de inputs, y (tras el fix del punto 1) en el footer.

- Cambiar el valor del token a `#6b7a70` (~4.6:1 sobre blanco), sin tocar el nombre de la variable ni ningún otro token de la paleta.

## Verificación

- Cargar `public/index.html` localmente (`python3 -m http.server 8000 --directory public`) y revisar visualmente que el footer sea legible.
- Confirmar el contraste de `--text3` con un checker (o cálculo manual del ratio) contra `#ffffff`.
- En un viewport de ancho mobile típico (~375-412px, perfil Android), confirmar con las devtools que `.opcion` mide ≥48px de alto tanto en opciones de una línea como de dos líneas de texto.

## Fuera de alcance

- Migración de tamaños tipográficos de `px` a `rem` (evaluada y descartada para esta ronda).
- Cualquier cambio de paleta, tipografía o layout más allá de los tres puntos de arriba.
- Cambios al contenido/estructura de preguntas — eso corresponde al plan de simplificación del flujo, que se asume ya implementado.
- `.subpregunta` y estilos asociados que puedan quedar sin uso tras la simplificación del flujo — limpieza fuera de alcance de esta spec.
