# Diseño — Revisión de encuesta: banco principal (Paso 1) + bloque "Proceso de solicitud" (reemplaza Paso 4)

**Fecha:** 2026-07-06
**Branch:** alternativa
**Archivo afectado:** `public/index.html` (single-file app)

## 1. Objetivo

Dos cambios coordinados sobre la encuesta de benchmark de tasas Pyme, implementados juntos porque comparten plumbing (`collectAnswers`, validación y la planilla destino):

- **Cambio A — Banco principal (Paso 1):** capturar con qué banco concretó el encuestado su operación, para poder cruzar **banco × tasa × proceso** (el corazón del benchmark). Ver §2.
- **Cambio B — Bloque "Proceso de solicitud" (Paso 4):** caracterizar el flujo de solicitud del préstamo (opción B — inteligencia de mercado, descripción del estado actual, sin diagnóstico de fricción ni scoring de lead): canal, asistencia humana vs. automatizada, requisitos para cotizar, tiempo punta a punta y conformidad. Reemplaza el bloque de Garantías. Ver §3–§4.

## 2. Cambio A — Banco principal (Paso 1)

### 2.1 Ubicación y racional (CRO)
Nueva pregunta en el **Paso 1 (`bloque-0`, "Caracterización de la Empresa"), debajo de Q0 Tamaño**. No es un paso nuevo ni va al final.

- **Momentum:** elegir el banco de una lista es fricción casi nula → buena pregunta temprana para bajar el abandono aguas abajo.
- **Ancla de contexto:** fija el banco como entidad de referencia para tasas, FOGAPE y Proceso; las preguntas siguientes dejan de ser abstractas.
- **Encaje temático:** el bloque se llama "Caracterización de la Empresa"; el banco principal es un atributo de caracterización.
- **Sin costo de longitud:** no agrega un círculo al stepper (siguen 5 pasos) ni un punto de caída extra.

### 2.2 Contenido
Single-choice (**radio**), **obligatoria**.

**Q0b. ¿Cuál es su banco principal?** *(la entidad con la que concretó su última operación de financiamiento)*
- Banco de Chile
- BCI
- Santander
- BancoEstado
- Scotiabank
- Bice
- Itaú
- Falabella
- Otro → habilita un campo de texto **opcional** para especificar

Notas de diseño:
- **"Otro":** evita el callejón sin salida para Pymes con banco no listado (Security, Consorcio, Coopeuch) o financiamiento vía factoring/fintech. Seleccionar "Otro" ya satisface el requerido; especificar el nombre en el texto es opcional.
- **Obligatoria:** bloquea el avance del Paso 1 si no hay selección (nueva validación en `validateBloque(0)`, ver §5.6). Nota: Q0 Tamaño hoy **no** es obligatoria y no se toca; el único bloqueo nuevo en el Paso 1 es el banco.
- **Orden de la lista:** se respeta el orden provisto. Reordenar por peso en Pyme (BancoEstado / Santander / Banco de Chile / BCI arriba) es opcional y queda a criterio.
- **Sin GA4:** el banco no emite evento ni parámetro GA4; viaja solo en el payload a la planilla (el cruce banco × tasa se hace sobre los datos de la planilla, no en GA).

## 3. Cambio B — alcance y trade-off asumido

El Paso 4 actual ("Garantías", `bloque-3`) contiene **Q7 y Q8** sobre exigencias de garantías/colaterales (con y sin FOGAPE). Este bloque **se reemplaza por completo**: se dropea el dato de garantías a cambio del dato de proceso.

- **Consecuencia aceptada:** se pierde la caracterización de garantías/colaterales exigidos.
- **Coherencia:** ambos son inteligencia de mercado descriptiva, así que el reemplazo no cambia la naturaleza de la encuesta.
- **No se hace:** no se reubican Q7/Q8 a otro paso (decisión explícita: reemplazo, no mudanza).

## 4. Contenido del bloque "Proceso de solicitud"

1 pregunta-compuerta + 5 núcleo + 1 opcional. Todas **single-choice** (radio), ancladas a la última operación de financiamiento, en usted y contexto Chile. Ninguna del bloque bloquea el avance (los únicos bloqueos de la encuesta son: Capital de Trabajo en Paso 2, banco principal en Paso 1 y email en Contacto).

La compuerta **Q7** filtra la población: si el encuestado nunca solicitó financiamiento, Q8–Q13 se ocultan y no se responden. Así los blancos por debajo de un "No" son semánticamente correctos (no aplica) en vez de ambiguos, y el embudo GA4 puede segmentarse por aplicabilidad (ver §6). Para el que responde "No", el bloque colapsa a una sola pregunta.

**Q7 (compuerta). ¿Su empresa ha pasado por un proceso de solicitud de financiamiento bancario en los últimos 24 meses?**
- Sí → se muestran Q8–Q13
- No, no hemos solicitado → se ocultan Q8–Q13

**Q8. ¿A través de qué canal inició su última solicitud de financiamiento?**
- Ejecutivo de cuenta asignado (presencial o remoto)
- Sucursal / presencial, sin ejecutivo asignado
- Sitio web / banca en línea (autoservicio)
- App móvil
- Teléfono / call center
- Correo electrónico

**Q9. Durante la solicitud, ¿quién lo asistió principalmente?**
- Un ejecutivo humano dedicado de principio a fin
- Ejecutivos humanos, pero rotando de interlocutor
- Asistente automatizado / bot, con humano solo a pedido
- 100% autoservicio, sin asistencia
- No recuerdo / no aplica

**Q10. Para informarle la tasa, ¿le exigieron documentación previa?**
- Sí, carpeta completa (EEFF, carpeta tributaria) antes de cotizar
- Sí, documentación básica (identificación, ventas)
- No, tasa referencial inmediata; la documentación vino después
- Me dieron la tasa sin pedir documentación

**Q11. Punta a punta —desde que inició la solicitud hasta el desembolso o rechazo—, ¿cuánto tardó?**
- Menos de 48 horas
- 2 a 5 días hábiles
- 1 a 2 semanas
- 2 a 4 semanas
- Más de 1 mes

**Q12. En general, ¿qué tan conforme quedó con el proceso (independiente de la tasa)?**
- Muy conforme
- Conforme
- Neutral
- Disconforme
- Muy disconforme

**Q13 (opcional). ¿Qué fue lo más engorroso del proceso?**
- La cantidad de documentación exigida
- La lentitud de las respuestas
- La falta de un interlocutor claro
- La poca transparencia sobre cómo se define la tasa
- Nada, fue ágil

## 5. Cambios técnicos en `public/index.html`

Es un archivo único; los cambios tocan varios lugares acoplados. Se listan por ubicación.

### 5.1 HTML — `bloque-0`, banco principal (después de `pq-q0`, línea ~224) [Cambio A]
- Insertar una `.pregunta` (id `pq-q-banco`) con `.pregunta-label` "¿Cuál es su banco principal?" + `.req` (asterisco, igual que email), subtítulo anclando a "última operación de financiamiento".
- 9 opciones radio `name="q_banco"` (8 bancos + "Otro"), misma estructura `.opcion > input[type=radio] + .opcion-text`.
- **"Otro" con texto:** un input de texto `id="q_banco_otro"` oculto por defecto (patrón `.subpregunta`/`.visible`); un listener `change` sobre `q_banco` lo muestra cuando el valor es "Otro" y lo oculta (y limpia) en cualquier otra opción. El texto es opcional.

### 5.2 HTML — `bloque-3`, bloque Proceso (líneas ~336–358) [Cambio B]
- `bloque-title`: "Garantías y Colaterales Exigidos" → **"El proceso de solicitud"**.
- Quitar `pq-q9` (Q7 garantías) y `pq-q10` (Q8 FOGAPE).
- Insertar las 7 preguntas de §4 (1 compuerta + 6) con la estructura estándar (`.pregunta > .pregunta-label + .opciones > label.opcion > input[type=radio] + .opcion-text`).
- Nombres de inputs: compuerta `q_solicito`; luego `q_canal`, `q_asistencia`, `q_docs_tasa`, `q_tiempo`, `q_conformidad`, `q_engorroso`.
- **Condicional (compuerta):** envolver Q8–Q13 en un contenedor oculto por defecto (patrón `.subpregunta`/`.visible`). Listener `change` sobre `q_solicito`: muestra el contenedor si "Sí", lo oculta si "No". Al ocultar, **resetear** los radios de Q8–Q13 (`.checked = false`) para no arrastrar respuestas viejas al payload.
- Q13: label marcado "(opcional)".

### 5.3 Stepper — label visible (línea ~208) [Cambio B]
`<span class="step-label">Garantías</span>` → **`Proceso`**.

### 5.4 `nombresPasos` (línea ~526) [Cambio B]
`['Tamaño','Capital','FOGAPE','Garantías','Contacto']` → `['Tamaño','Capital','FOGAPE','Proceso','Contacto']`.

### 5.5 `collectAnswers()` (líneas ~730–733) [Cambios A + B]
Quitar `q9` y `q10`. Agregar banco (A) + compuerta y proceso (B):
```js
q_banco: getRadio('q_banco'),
q_banco_otro: document.getElementById('q_banco_otro')?.value.trim() || '',
q_solicito: getRadio('q_solicito'),
q_canal: getRadio('q_canal'),
q_asistencia: getRadio('q_asistencia'),
q_docs_tasa: getRadio('q_docs_tasa'),
q_tiempo: getRadio('q_tiempo'),
q_conformidad: getRadio('q_conformidad'),
q_engorroso: getRadio('q_engorroso')
```
Estas claves viajan en el payload JSON tanto en el autosave parcial (`autosaveStep`) como en el envío final (`submitForm`).

### 5.6 `validateBloque()` (línea ~433) — banco obligatorio [Cambio A]
Agregar el caso del Paso 1:
```js
if(b === 0){
  if(!document.querySelector('input[name="q_banco"]:checked'))
    return showError('pq-q-banco','Seleccione su banco principal para continuar.');
  return true;
}
```
No se agrega validación al bloque Proceso (Paso 4): sigue siendo opcional como FOGAPE.

## 6. Decisiones de analítica (a confirmar)

- **GA4 `etapa_nombre`:** el array `etapas` en `goNext` (línea ~452) nombra el Paso 4 como `'Requisitos'`, deliberadamente idéntico a la otra branch para comparar embudos. **Se mantiene `'Requisitos'` sin cambios** aunque el label visible pase a "Proceso", para no romper la comparabilidad histórica de `etapa_completada`. El label del stepper es cosmético; el nombre de evento es analítico.
- **Segmentación solicitó vs. no solicitó (compuerta Q7):** al completar el Paso 4 (`goNext` con `currentBloque === 3`) se agrega a `etapa_completada` el parámetro `solicito_financiamiento` con valor `'si'` / `'no'` / `''` (vacío si no respondió la compuerta), derivado de `getRadio('q_solicito')`. El mismo parámetro se replica en `form_completado` (submit final) para segmentar el embudo completo por aplicabilidad. Sin esto, quien nunca solicitó cuenta idéntico a quien sí.
- **Banco principal:** sin cambios GA4 (ver §2.2). Solo payload a la planilla.
- El único agregado analítico es el parámetro `solicito_financiamiento`: un parámetro, no un evento nuevo.

## 7. Dependencia externa (fuera de este repo)

El endpoint es un Google Apps Script que escribe a una planilla:
`ENDPOINT = https://script.google.com/.../exec` (línea ~384).

- Las nuevas claves (`q_banco`, `q_banco_otro`, `q_solicito`, `q_canal`, `q_asistencia`, `q_docs_tasa`, `q_tiempo`, `q_conformidad`, `q_engorroso`) **solo se persisten si la planilla/Apps Script tiene columnas para recibirlas**. Si el script mapea headers dinámicamente, basta agregar columnas; si mapea posiciones fijas, hay que actualizar el script.
- `q9`/`q10` dejan de enviarse: sus columnas históricas quedan vacías de aquí en más (no se borran datos previos).
- **Acción requerida por fuera del código:** confirmar/actualizar la planilla antes de publicar, o los datos se pierden silenciosamente (el envío es `no-cors`, sin feedback de error).

## 8. Fuera de alcance (YAGNI)

- No se hace responsive/estilo nuevo: se reusa el CSS existente de `.bloque`/`.pregunta`/`.opcion`.
- Los únicos condicionales son la compuerta Q7 (mostrar/ocultar Q8–Q13) y el "Otro" del banco (mostrar/ocultar su texto). No hay otras subpreguntas anidadas ni ramas por respuesta.
- No se captura *footprint* bancario (con cuántos/qué bancos opera): el banco es single-select "principal" para poder linkearlo a la operación. Un checkbox de footprint sería otra pregunta, opcional, y hoy no se necesita.
- No se diferencia el flujo por producto (estándar vs. FOGAPE): una sola batería genérica sobre la operación principal.
- No se agrega validación obligatoria al bloque Proceso.
