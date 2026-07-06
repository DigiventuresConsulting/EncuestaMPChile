# Diseño — Bloque "Proceso de solicitud" (reemplaza paso 4 "Garantías")

**Fecha:** 2026-07-06
**Branch:** alternativa
**Archivo afectado:** `public/index.html` (single-file app)

## 1. Objetivo

Sumar a la encuesta un bloque que **caracterice el flujo de solicitud del préstamo Pyme** en los bancos chilenos (opción B — inteligencia de mercado / descripción del estado actual, sin diagnóstico de fricción ni scoring de lead). El output alimenta un mapa descriptivo de cómo operan hoy los bancos con Pymes: canal, asistencia humana vs. automatizada, requisitos para cotizar, tiempo punta a punta y conformidad general.

## 2. Decisión de alcance y trade-off asumido

El paso 4 actual ("Garantías", `bloque-3`) contiene **Q7 y Q8** sobre exigencias de garantías/colaterales (con y sin FOGAPE). Este bloque **se reemplaza por completo**: se dropea el dato de garantías a cambio del dato de proceso.

- **Consecuencia aceptada:** se pierde la caracterización de garantías/colaterales exigidos.
- **Coherencia:** ambos son inteligencia de mercado descriptiva, así que el reemplazo no cambia la naturaleza de la encuesta.
- **No se hace:** no se reubican Q7/Q8 a otro paso (fue decisión explícita: reemplazo, no mudanza).

## 3. Contenido del bloque (validado)

5 preguntas núcleo + 1 opcional. Todas **single-choice** (radio), ancladas a la última operación de financiamiento, en usted y contexto Chile. Ninguna es obligatoria para avanzar (consistente con FOGAPE/Garantías, que hoy son opcionales; solo Capital de Trabajo y email bloquean el avance).

**Q7. ¿A través de qué canal inició su última solicitud de financiamiento?**
- Ejecutivo de cuenta asignado (presencial o remoto)
- Sucursal / presencial, sin ejecutivo asignado
- Sitio web / banca en línea (autoservicio)
- App móvil
- Teléfono / call center
- Correo electrónico

**Q8. Durante la solicitud, ¿quién lo asistió principalmente?**
- Un ejecutivo humano dedicado de principio a fin
- Ejecutivos humanos, pero rotando de interlocutor
- Asistente automatizado / bot, con humano solo a pedido
- 100% autoservicio, sin asistencia
- No recuerdo / no aplica

**Q9. Para informarle la tasa, ¿le exigieron documentación previa?**
- Sí, carpeta completa (EEFF, carpeta tributaria) antes de cotizar
- Sí, documentación básica (identificación, ventas)
- No, tasa referencial inmediata; la documentación vino después
- Me dieron la tasa sin pedir documentación

**Q10. Punta a punta —desde que inició la solicitud hasta el desembolso o rechazo—, ¿cuánto tardó?**
- Menos de 48 horas
- 2 a 5 días hábiles
- 1 a 2 semanas
- 2 a 4 semanas
- Más de 1 mes

**Q11. En general, ¿qué tan conforme quedó con el proceso (independiente de la tasa)?**
- Muy conforme
- Conforme
- Neutral
- Disconforme
- Muy disconforme

**Q12 (opcional). ¿Qué fue lo más engorroso del proceso?**
- La cantidad de documentación exigida
- La lentitud de las respuestas
- La falta de un interlocutor claro
- La poca transparencia sobre cómo se define la tasa
- Nada, fue ágil

## 4. Cambios técnicos en `public/index.html`

Es un archivo único; el cambio toca cuatro lugares acoplados.

### 4.1 HTML — `bloque-3` (líneas ~336–358)
Reemplazar el contenido del bloque:
- `bloque-title`: "Garantías y Colaterales Exigidos" → **"El proceso de solicitud"**.
- Quitar las preguntas `pq-q9` (Q7 garantías) y `pq-q10` (Q8 FOGAPE).
- Insertar las 6 preguntas de arriba con la misma estructura (`.pregunta > .pregunta-label + .opciones > label.opcion > input[type=radio] + .opcion-text`).
- Nombres de inputs propuestos (uno por pregunta): `q_canal`, `q_asistencia`, `q_docs_tasa`, `q_tiempo`, `q_conformidad`, `q_engorroso`.
- Q12: marcar el label como "(opcional)" en el texto, igual que otros opcionales.

### 4.2 Stepper — label visible (línea ~208)
`<span class="step-label">Garantías</span>` → **`Proceso`**.

### 4.3 `nombresPasos` (línea ~526)
`['Tamaño','Capital','FOGAPE','Garantías','Contacto']` → `['Tamaño','Capital','FOGAPE','Proceso','Contacto']`.

### 4.4 `collectAnswers()` (líneas ~730–733)
Quitar `q9` y `q10`. Agregar las seis claves nuevas:
```js
q_canal: getRadio('q_canal'),
q_asistencia: getRadio('q_asistencia'),
q_docs_tasa: getRadio('q_docs_tasa'),
q_tiempo: getRadio('q_tiempo'),
q_conformidad: getRadio('q_conformidad'),
q_engorroso: getRadio('q_engorroso')
```
Estas claves viajan en el payload JSON tanto en el autosave parcial (`autosaveStep`) como en el envío final (`submitForm`) al endpoint de Apps Script.

## 5. Decisiones de analítica (a confirmar)

- **GA4 `etapa_nombre`:** el array `etapas` en `goNext` (línea ~452) nombra este paso como `'Requisitos'`, deliberadamente idéntico a la otra branch para poder comparar embudos. **Se mantiene `'Requisitos'` sin cambios** aunque el label visible pase a "Proceso", para no romper la comparabilidad histórica del evento `etapa_completada`. El label del stepper es cosmético; el nombre de evento es analítico.
- No se agregan eventos GA4/Pixel nuevos por pregunta: el bloque se mide a nivel de paso completado, como el resto.

## 6. Dependencia externa (fuera de este repo)

El endpoint es un Google Apps Script que escribe a una planilla:
`ENDPOINT = https://script.google.com/.../exec` (línea ~384).

- Las nuevas claves (`q_canal`, `q_asistencia`, `q_docs_tasa`, `q_tiempo`, `q_conformidad`, `q_engorroso`) **solo se persisten si la planilla/Apps Script tiene columnas para recibirlas**. Si el script mapea headers dinámicamente, basta agregar columnas; si mapea posiciones fijas, hay que actualizar el script.
- `q9`/`q10` dejan de enviarse: sus columnas históricas quedan vacías de aquí en más (no se borran datos previos).
- **Acción requerida por fuera del código:** confirmar/actualizar la planilla antes de publicar, o los datos del bloque se pierden silenciosamente (el envío es `no-cors`, sin feedback de error).

## 7. Fuera de alcance (YAGNI)

- No se hace responsive/estilo nuevo: se reusa el CSS existente de `.bloque`/`.pregunta`/`.opcion`.
- No se agregan subpreguntas condicionales (el bloque es plano).
- No se diferencia el flujo por producto (estándar vs. FOGAPE): una sola batería genérica sobre la operación principal.
- No se agrega validación obligatoria al bloque.
