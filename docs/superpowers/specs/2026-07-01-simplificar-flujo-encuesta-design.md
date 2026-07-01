# Simplificar flujo de preguntas de la encuesta

**Fecha:** 2026-07-01
**Estado:** Aprobado, pendiente de plan de implementación

## Contexto

`public/index.html` implementa hoy una encuesta de 5 pasos (`bloque-0` a `bloque-4`) con lógica condicional: cada bloque de tasas (Estándar y FOGAPE) empieza con una pregunta de "modalidad principal" (CLP vs UF) que determina qué subpregunta de rango de tasa mostrar (`handleConditional()` + `.subpregunta`), seguida de una pregunta de "financiamiento complementario" que puede abrir una segunda subpregunta de rango de tasa en la otra moneda.

El archivo `docs_local/diagrama_simplificado_svg.xml.pdf` define un flujo nuevo, más lineal: por cada bloque (Estándar, FOGAPE) se pregunta directamente la tasa CLP (corto plazo) y la tasa UF (largo plazo), sin pregunta de filtro previa ni rama condicional, y sin pregunta de financiamiento complementario (queda redundante porque ambas tasas ya se preguntan siempre). El bloque de Requisitos y Colaterales (Q9/Q10 actuales) no aparece en el diagrama pero el usuario confirmó que se mantiene como paso final.

## Objetivo

Simplificar la estructura de preguntas de Financiamiento Estándar y FOGAPE eliminando la lógica condicional de modalidad y las preguntas de financiamiento complementario, dejando dos preguntas de tasa fijas (CLP y UF) por bloque más la pregunta de monto máximo, tal como muestra el diagrama.

## Estructura final (5 pasos)

1. **Contacto** (`bloque-1`) — sin cambios.
2. **Tamaño** (`bloque-0`, Q0) — sin cambios.
3. **Bloque 1: Financiamiento Estándar** (`bloque-2`) — 3 preguntas obligatorias, sin ramas:
   - `q_std_clp` — tasa CLP corto plazo. Mismos 7 tramos que la actual `sq-q2-clp` (Opción A), más una opción nueva **"No aplica / no usa este tipo de financiamiento"**.
   - `q_std_uf` — tasa UF largo plazo. Mismos 6 tramos que la actual `sq-q2-uf` (Opción B), más **"No aplica / no usa este tipo de financiamiento"**.
   - `q_std_monto` — monto máximo sin garantía. Igual al Q4 actual (6 opciones), sin cambios.
4. **Bloque 2: FOGAPE** (`bloque-3`) — mismo patrón:
   - `q_fogape_clp` — tasa CLP FOGAPE. Mismos 6 tramos que la actual `sq-q6-clp`, más **"No aplica / no usa este tipo de financiamiento"**.
   - `q_fogape_uf` — tasa UF FOGAPE. Mismos 6 tramos que la actual `sq-q6-uf`, más **"No aplica / no usa este tipo de financiamiento"**.
   - `q_fogape_monto` — monto máximo FOGAPE. Igual al Q8 actual (6 opciones, incluye "No nos ofrecen cupo FOGAPE..."), sin cambios.
5. **Requisitos** (`bloque-4`, Q9/Q10) — sin cambios, se mantiene como paso final.

## Qué se elimina

- Preguntas de filtro de modalidad principal: Q1 actual (`pq-q1`, name="q1") en Estándar y Q5 actual (`pq-q5`, name="q5") en FOGAPE.
- Preguntas de financiamiento complementario y sus subpreguntas: Q3/Q3.A/Q3.B (`pq-q3`, `sq-q3a`, `sq-q3b`) en Estándar; Q7/Q7.A/Q7.B (`pq-q7`, `sq-q7a`, `sq-q7b`) en FOGAPE.
- La función `handleConditional()` y el uso de clases `.subpregunta`/`.subpregunta.visible` para mostrar/ocultar preguntas de tasa según modalidad — las preguntas de tasa pasan a ser siempre visibles (ya no son `.subpregunta`, son `.pregunta` normales).
- Los `<div class="pregunta" ... style="display:none">` que hoy ocultan Q3/Q7/Q4/Q8 hasta que se responde la pregunta anterior — ya no hace falta ese show/hide secuencial dentro del bloque, todas las preguntas del bloque están visibles desde el inicio (aparecen en el orden del diagrama).

## Numeración visible

Las etiquetas visibles de pregunta (`Q1.`, `Q2.`, etc. en `.pregunta-label`) se renumeran para reflejar el nuevo orden dentro de cada bloque, siguiendo el diagrama:
- Estándar: Q1 (CLP), Q2 (UF), Q3 (monto máximo).
- FOGAPE: Q4 (CLP), Q5 (UF), Q6 (monto máximo).
- Requisitos: Q7, Q8 (antes Q9, Q10).

## Payload / integración con Google Apps Script

El payload enviado a GAS (`submitForm()` y `savePartial()`) cambia sus keys para reflejar la nueva estructura:

```
q0            -> tamaño de empresa (sin cambios)
q_std_clp     -> tasa CLP Estándar
q_std_uf      -> tasa UF Estándar
q_std_monto   -> monto máximo Estándar
q_fogape_clp  -> tasa CLP FOGAPE
q_fogape_uf   -> tasa UF FOGAPE
q_fogape_monto -> monto máximo FOGAPE
q9            -> Requisitos sin garantía (sin cambios de contenido)
q10           -> Requisitos con garantía (sin cambios de contenido)
```

Se eliminan del payload: `q1`, `q3`, `q3a`, `q3b`, `q5`, `q7`, `q7a`, `q7b`.

El usuario actualizará el Google Apps Script / la hoja de cálculo por su cuenta para reflejar las nuevas columnas — no es parte de este trabajo.

## Validación por paso

Cada bloque valida en `goNext()`/`submitForm()` que sus preguntas obligatorias estén respondidas, usando el mismo patrón `alert()` que existe hoy. Como ya no hay ramas condicionales, la validación se simplifica: por bloque, simplemente comprobar que las 2-3 preguntas de radio del bloque tengan una opción marcada (no hace falta lógica condicional tipo `if(q3 === 'clp-uf' && ...)`).

## Fuera de alcance

- No se toca el paso de Contacto ni el paso de Tamaño (Q0).
- No se toca el bloque de Requisitos (Q9/Q10, ahora Q7/Q8) más allá de renumerar su etiqueta visible.
- No se cambian los rangos numéricos de tasas ni de montos — se reusan tal cual existen hoy, solo se convierten de condicionales a preguntas fijas y se les agrega "No aplica" a las dos preguntas de tasa.
- No se toca el guardado de borrador local (`saveLocalDraft`/`restoreLocalDraft`), el guard de cambio de RUT, ni el tracking de Clarity/Meta Pixel — deberán seguir funcionando pero no requieren cambios de diseño más allá de ajustar qué nombres de campo se guardan/restauran.
- No se actualiza el Google Apps Script ni la Google Sheet — el usuario lo hará por su cuenta.

## Riesgos / notas de implementación

- `saveLocalDraft()`/`restoreLocalDraft()` deben actualizarse para guardar/restaurar los nuevos `name` de radio (`q_std_clp`, etc.) en vez de los actuales `q1`..`q10`, o el autosave de borrador quedará roto para los bloques Estándar/FOGAPE.
- El array `nombresPasos` usado para Clarity y el indicador mobile no necesita cambios (sigue siendo por bloque, no por pregunta).
- Los estilos `.subpregunta` pueden quedar sin uso tras el cambio; no es necesario eliminarlos del CSS a menos que se quiera limpieza adicional (fuera de alcance salvo que el usuario lo pida).
