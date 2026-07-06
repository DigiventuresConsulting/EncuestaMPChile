# Banco principal + Bloque "Proceso de solicitud" — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Agregar a la encuesta (a) un radio obligatorio de banco principal en el Paso 1 y (b) reemplazar el Paso 4 "Garantías" por un bloque "Proceso de solicitud" con pregunta-compuerta y segmentación GA4.

**Architecture:** App de un solo archivo estático (`public/index.html`) — HTML + CSS + JS vanilla, sin build ni framework. El wizard navega bloques (`bloque-0`..`bloque-4`) por índice `currentBloque`. Los datos se capturan por dos rutas: `collectAnswers()` (texto legible → payload JSON al Google Apps Script) y `getFormSnapshot()`/`restoreLocalDraft()` (value slug → borrador en localStorage). Se reusan patrones existentes: listener global de radios, clase `.subpregunta`/`.visible`, `.field input` para texto.

**Tech Stack:** HTML5, CSS, JavaScript vanilla, gtag.js (GA4), Meta Pixel, Clarity. Sin bundler, sin test runner.

## Global Constraints

- **Un solo archivo:** todo el cambio ocurre en `public/index.html`. No agregar dependencias, build ni framework.
- **Sin test runner:** la verificación es en navegador. Servir con `cd public && python3 -m http.server 8000` y abrir `http://localhost:8000`. "Test" = pasos observables en UI + asserts en la consola de DevTools (`collectAnswers()`, `window.dataLayer`) + inspección del payload en la pestaña Network.
- **Dos rutas de datos, ambas obligatorias para cada radio nuevo:** dar a cada radio un atributo `value` (slug corto) **y** un texto legible en `.opcion-text`. `collectAnswers()` usa `getRadio(name)` → devuelve el **texto**. `getFormSnapshot()` usa `getRadioValue(name)` → devuelve el **value**.
- **GA4 — no renombrar el evento:** el array `etapas` en `goNext` (línea ~455) mantiene `'Requisitos'` para el Paso 4. NO cambiar a "Proceso" ahí. Solo cambia el label visible del stepper y `nombresPasos`.
- **GA4 — parámetro nuevo, nombre exacto:** `solicito_financiamiento`, valores `'si'` / `'no'` / `''`.
- **Nombres de inputs exactos:** `q_banco`, `q_banco_otro`, `q_solicito`, `q_canal`, `q_asistencia`, `q_docs_tasa`, `q_tiempo`, `q_conformidad`, `q_engorroso`.
- **Obligatoriedad:** `q_banco` bloquea el avance del Paso 1. El bloque Proceso (Paso 4) es 100% opcional. Q0 (Tamaño) sigue sin ser obligatoria (no se toca).
- **Branch:** `alternativa`. Un commit por tarea, en esa branch.
- **Dependencia externa (NO es tarea de código):** la planilla / Google Apps Script destino debe tener columnas para las claves nuevas (`q_banco`, `q_banco_otro`, `q_solicito`, `q_canal`, `q_asistencia`, `q_docs_tasa`, `q_tiempo`, `q_conformidad`, `q_engorroso`) o los datos se pierden en silencio (envío `no-cors`). Confirmar por fuera antes de publicar.

---

## File Structure

Un único archivo modificado: `public/index.html`. Zonas afectadas (líneas aproximadas al estado inicial de la branch `alternativa`):

| Zona | Línea aprox. | Tarea |
|---|---|---|
| Stepper `step-4` label | 208 | 2 |
| `bloque-0` (HTML, tras `pq-q0`) | 226 | 1 |
| `bloque-3` (HTML, reemplazo completo) | 336–358 | 2 |
| Listener global de radios | 390–404 | 1, 2 |
| `goNext` — `etapa_completada` | 455–462 | 3 |
| `showSuccess` — `form_completado` | ~592 | 3 |
| `validateBloque` | 433–449 | 1 |
| `nombresPasos` (en `updateProgress`) | 526 | 2 |
| `getFormSnapshot` | 622–640 | 1, 2 |
| `collectAnswers` | 719–735 | 1, 2 |
| `restoreLocalDraft` | 755–790 | 1, 2 |

---

## Task 1: Banco principal (Paso 1) — radio obligatorio con "Otro"

**Files:**
- Modify: `public/index.html` (HTML en `bloque-0`; JS en listener, `validateBloque`, `collectAnswers`, `getFormSnapshot`, `restoreLocalDraft`)

**Interfaces:**
- Consumes: `getRadio(name)` (texto), `getRadioValue(name)` (value), `showError(containerId, msg)`, clase `.subpregunta`/`.visible`, estilos `.pregunta` / `.opcion` / `.field input[type=text]`.
- Produces: input radio `name="q_banco"` (values: `chile`, `bci`, `santander`, `bancoestado`, `scotiabank`, `bice`, `itau`, `falabella`, `otro`); input texto `id="q_banco_otro"`; función global `setBancoOtroVisible(show)`. `collectAnswers()` gana las claves `q_banco` (texto) y `q_banco_otro` (texto libre).

- [ ] **Step 1: Insertar la pregunta de banco en `bloque-0`**

Buscar el cierre del `pq-q0` y del `bloque-0` (líneas ~225–227):

```html
      </div>
    </div>
  </div>

  <!-- BLOQUE 4: CONTACTO (último paso) -->
```

Reemplazar por (inserta la nueva `.pregunta` antes de cerrar `bloque-0`):

```html
      </div>
    </div>
    <div class="pregunta" id="pq-q-banco">
      <div class="pregunta-label">Banco principal <span class="req">*</span></div>
      <div class="pregunta-sub">La entidad con la que concretó su última operación de financiamiento.</div>
      <div class="opciones">
        <label class="opcion"><input type="radio" name="q_banco" value="chile"> <span class="opcion-text">Banco de Chile</span></label>
        <label class="opcion"><input type="radio" name="q_banco" value="bci"> <span class="opcion-text">BCI</span></label>
        <label class="opcion"><input type="radio" name="q_banco" value="santander"> <span class="opcion-text">Santander</span></label>
        <label class="opcion"><input type="radio" name="q_banco" value="bancoestado"> <span class="opcion-text">BancoEstado</span></label>
        <label class="opcion"><input type="radio" name="q_banco" value="scotiabank"> <span class="opcion-text">Scotiabank</span></label>
        <label class="opcion"><input type="radio" name="q_banco" value="bice"> <span class="opcion-text">Bice</span></label>
        <label class="opcion"><input type="radio" name="q_banco" value="itau"> <span class="opcion-text">Itaú</span></label>
        <label class="opcion"><input type="radio" name="q_banco" value="falabella"> <span class="opcion-text">Falabella</span></label>
        <label class="opcion"><input type="radio" name="q_banco" value="otro"> <span class="opcion-text">Otro</span></label>
      </div>
      <div class="subpregunta" id="sub-banco-otro">
        <div class="field" style="margin:0">
          <label style="font-size:12px;font-weight:600;color:var(--text2);margin-bottom:6px;display:block">¿Cuál? <span style="color:var(--text3);font-weight:400">(opcional)</span></label>
          <input type="text" id="q_banco_otro" placeholder="Nombre del banco o financiera">
        </div>
      </div>
    </div>
  </div>

  <!-- BLOQUE 4: CONTACTO (último paso) -->
```

- [ ] **Step 2: Agregar el toggle del texto "Otro"**

Después del bloque del listener global de radios (termina en línea ~404 con `});`), insertar:

```javascript
// Banco principal: mostrar/ocultar el campo "Otro" según la selección
function setBancoOtroVisible(show){
  const sub = document.getElementById('sub-banco-otro');
  if(!sub) return;
  sub.classList.toggle('visible', show);
  if(!show){ const t = document.getElementById('q_banco_otro'); if(t) t.value = ''; }
}
document.querySelectorAll('input[name="q_banco"]').forEach(r => {
  r.addEventListener('change', function(){ setBancoOtroVisible(this.value === 'otro'); });
});
```

- [ ] **Step 3: Hacer `q_banco` obligatorio en el Paso 1**

En `validateBloque` (línea ~434), buscar:

```javascript
function validateBloque(b){
  clearErrors();
  // Obligatorios: Bloque 1 de Capital de Trabajo (paso 1) y email (Contacto, paso final). FOGAPE y Garantías son opcionales.
  if(b === 1){
```

Reemplazar por:

```javascript
function validateBloque(b){
  clearErrors();
  // Obligatorios: banco principal (paso 1), Capital de Trabajo (paso 2) y email (Contacto). FOGAPE y Proceso son opcionales.
  if(b === 0){
    if(!document.querySelector('input[name="q_banco"]:checked')) return showError('pq-q-banco','Seleccione su banco principal para continuar.');
    return true;
  }
  if(b === 1){
```

- [ ] **Step 4: Capturar banco en `collectAnswers()`**

En `collectAnswers` (línea ~726), buscar:

```javascript
    q0: getRadio('q0'),
```

Reemplazar por:

```javascript
    q0: getRadio('q0'),
    q_banco: getRadio('q_banco'),
    q_banco_otro: (document.getElementById('q_banco_otro')?.value || '').trim(),
```

- [ ] **Step 5: Persistir banco en el borrador local (`getFormSnapshot`)**

En `getFormSnapshot` (línea ~622), buscar:

```javascript
  return {
    currentBloque: currentBloque,
    correo: document.getElementById('email').value.trim(),
    rut: document.getElementById('rut').value.trim(),
    telefono: document.getElementById('telefono').value.trim(),
    radios: {
      q0: getRadioValue('q0'),
```

Reemplazar por:

```javascript
  return {
    currentBloque: currentBloque,
    correo: document.getElementById('email').value.trim(),
    rut: document.getElementById('rut').value.trim(),
    telefono: document.getElementById('telefono').value.trim(),
    bancoOtro: (document.getElementById('q_banco_otro')?.value || '').trim(),
    radios: {
      q0: getRadioValue('q0'),
      q_banco: getRadioValue('q_banco'),
```

- [ ] **Step 6: Restaurar banco al reingresar (`restoreLocalDraft`)**

En `restoreLocalDraft` (línea ~772), buscar:

```javascript
  if(snap.radios){
    Object.keys(snap.radios).forEach(name => {
      const value = snap.radios[name];
      if(!value) return;
      const input = document.querySelector(`input[name="${name}"][value="${CSS.escape(value)}"]`);
      if(input){
        input.checked = true;
        input.closest('.opcion').classList.add('selected');
      }
    });
  }
```

Reemplazar por (agrega la re-hidratación del "Otro" tras aplicar los radios):

```javascript
  if(snap.radios){
    Object.keys(snap.radios).forEach(name => {
      const value = snap.radios[name];
      if(!value) return;
      const input = document.querySelector(`input[name="${name}"][value="${CSS.escape(value)}"]`);
      if(input){
        input.checked = true;
        input.closest('.opcion').classList.add('selected');
      }
    });
  }
  if(snap.radios && snap.radios.q_banco === 'otro'){
    setBancoOtroVisible(true);
    const t = document.getElementById('q_banco_otro');
    if(t) t.value = snap.bancoOtro || '';
  }
```

- [ ] **Step 7: Verificar en navegador**

Servir: `cd public && python3 -m http.server 8000` (dejar corriendo) y abrir `http://localhost:8000`.

Verificaciones (Paso 1):
1. Se ve "Banco principal *" con 9 opciones debajo de Tamaño.
2. Sin seleccionar banco, clic en "Comenzar encuesta →" → aparece el error "Seleccione su banco principal para continuar." (confirma que es obligatoria y que Tamaño no bloquea).
3. Seleccionar "Otro" → aparece el campo de texto. Seleccionar "BCI" → el campo desaparece.
4. Seleccionar "Otro", escribir "Security". En la consola de DevTools:

```javascript
collectAnswers()
// Debe incluir: q_banco: "Otro", q_banco_otro: "Security"
```

5. Recargar la página (F5) → el banco "Otro" y el texto "Security" se restauran (borrador local).

- [ ] **Step 8: Commit**

```bash
git add public/index.html
git commit -m "feat: banco principal obligatorio con opcion Otro en Paso 1"
```

---

## Task 2: Bloque "Proceso de solicitud" reemplaza el Paso 4 "Garantías"

**Files:**
- Modify: `public/index.html` (HTML de `bloque-3` y del stepper; JS en listener/toggle, `nombresPasos`, `collectAnswers`, `getFormSnapshot`, `restoreLocalDraft`)

**Interfaces:**
- Consumes: `getRadio`, `getRadioValue`, clase `.opcion`, listener global de radios, `setProcesoVisible` (definida acá).
- Produces: radios `q_solicito` (values `si`/`no`), `q_canal`, `q_asistencia`, `q_docs_tasa`, `q_tiempo`, `q_conformidad`, `q_engorroso`; contenedor `#sub-proceso`; función global `setProcesoVisible(show)`. `collectAnswers()` pierde `q9`/`q10` y gana las 7 claves nuevas.

- [ ] **Step 1: Reemplazar el contenido de `bloque-3`**

Buscar el bloque completo (líneas ~336–358):

```html
  <!-- BLOQUE 3: REQUISITOS (Garantías) -->
  <div class="bloque" id="bloque-3">
    <div class="bloque-title">Garantías y Colaterales Exigidos</div>

    <div class="pregunta" id="pq-q9">
      <div class="pregunta-label">Q7. ¿Cuál es la principal exigencia de los bancos para otorgar un crédito SIN garantía estatal?</div>
      <div class="opciones">
        <label class="opcion"><input type="radio" name="q9"> <span class="opcion-text">Aval personal / fianza de los socios controladores de la empresa.</span></label>
        <label class="opcion"><input type="radio" name="q9"> <span class="opcion-text">Garantías reales duras (Hipotecas de naves industriales, oficinas o terrenos).</span></label>
        <label class="opcion"><input type="radio" name="q9"> <span class="opcion-text">Garantías reales líquidas (Prenda sobre depósitos a plazo o flujos de contratos).</span></label>
        <label class="opcion"><input type="radio" name="q9"> <span class="opcion-text">Flujo de caja proyectado auditado (No exigen garantías de colaterales reales).</span></label>
      </div>
    </div>

    <div class="pregunta" id="pq-q10">
      <div class="pregunta-label">Q8. ¿Cuál es la principal exigencia de los bancos para otorgar un crédito CON garantía estatal (FOGAPE)?</div>
      <div class="opciones">
        <label class="opcion"><input type="radio" name="q10"> <span class="opcion-text">Únicamente la fianza o aval personal de los socios controladores.</span></label>
        <label class="opcion"><input type="radio" name="q10"> <span class="opcion-text">Garantías reales complementarias por el porcentaje de riesgo residual (el 20% o 30% no cubierto por el Estado).</span></label>
        <label class="opcion"><input type="radio" name="q10"> <span class="opcion-text">Ninguna garantía adicional (Solo la cobertura de FOGAPE aprobada en evaluación de riesgo).</span></label>
      </div>
    </div>
  </div>
```

Reemplazar por:

```html
  <!-- BLOQUE 3: PROCESO DE SOLICITUD (reemplaza Garantías) -->
  <div class="bloque" id="bloque-3">
    <div class="bloque-title">El proceso de solicitud</div>

    <div class="pregunta" id="pq-solicito">
      <div class="pregunta-label">Q7. ¿Su empresa ha pasado por un proceso de solicitud de financiamiento bancario en los últimos 24 meses?</div>
      <div class="opciones">
        <label class="opcion"><input type="radio" name="q_solicito" value="si"> <span class="opcion-text">Sí, hemos solicitado</span></label>
        <label class="opcion"><input type="radio" name="q_solicito" value="no"> <span class="opcion-text">No, no hemos solicitado</span></label>
      </div>
    </div>

    <div id="sub-proceso" style="display:none">
      <div class="pregunta" id="pq-canal">
        <div class="pregunta-label">Q8. ¿A través de qué canal inició su última solicitud de financiamiento?</div>
        <div class="opciones">
          <label class="opcion"><input type="radio" name="q_canal" value="ejecutivo"> <span class="opcion-text">Ejecutivo de cuenta asignado (presencial o remoto)</span></label>
          <label class="opcion"><input type="radio" name="q_canal" value="sucursal"> <span class="opcion-text">Sucursal / presencial, sin ejecutivo asignado</span></label>
          <label class="opcion"><input type="radio" name="q_canal" value="web"> <span class="opcion-text">Sitio web / banca en línea (autoservicio)</span></label>
          <label class="opcion"><input type="radio" name="q_canal" value="app"> <span class="opcion-text">App móvil</span></label>
          <label class="opcion"><input type="radio" name="q_canal" value="telefono"> <span class="opcion-text">Teléfono / call center</span></label>
          <label class="opcion"><input type="radio" name="q_canal" value="mail"> <span class="opcion-text">Correo electrónico</span></label>
        </div>
      </div>

      <div class="pregunta" id="pq-asistencia">
        <div class="pregunta-label">Q9. Durante la solicitud, ¿quién lo asistió principalmente?</div>
        <div class="opciones">
          <label class="opcion"><input type="radio" name="q_asistencia" value="humano_dedicado"> <span class="opcion-text">Un ejecutivo humano dedicado de principio a fin</span></label>
          <label class="opcion"><input type="radio" name="q_asistencia" value="humano_rotando"> <span class="opcion-text">Ejecutivos humanos, pero rotando de interlocutor</span></label>
          <label class="opcion"><input type="radio" name="q_asistencia" value="bot"> <span class="opcion-text">Asistente automatizado / bot, con humano solo a pedido</span></label>
          <label class="opcion"><input type="radio" name="q_asistencia" value="autoservicio"> <span class="opcion-text">100% autoservicio, sin asistencia</span></label>
          <label class="opcion"><input type="radio" name="q_asistencia" value="no_recuerdo"> <span class="opcion-text">No recuerdo / no aplica</span></label>
        </div>
      </div>

      <div class="pregunta" id="pq-docs">
        <div class="pregunta-label">Q10. Para informarle la tasa, ¿le exigieron documentación previa?</div>
        <div class="opciones">
          <label class="opcion"><input type="radio" name="q_docs_tasa" value="carpeta_completa"> <span class="opcion-text">Sí, carpeta completa (EEFF, carpeta tributaria) antes de cotizar</span></label>
          <label class="opcion"><input type="radio" name="q_docs_tasa" value="basica"> <span class="opcion-text">Sí, documentación básica (identificación, ventas)</span></label>
          <label class="opcion"><input type="radio" name="q_docs_tasa" value="referencial_despues"> <span class="opcion-text">No, tasa referencial inmediata; la documentación vino después</span></label>
          <label class="opcion"><input type="radio" name="q_docs_tasa" value="sin_docs"> <span class="opcion-text">Me dieron la tasa sin pedir documentación</span></label>
        </div>
      </div>

      <div class="pregunta" id="pq-tiempo">
        <div class="pregunta-label">Q11. Punta a punta —desde que inició la solicitud hasta el desembolso o rechazo—, ¿cuánto tardó?</div>
        <div class="opciones">
          <label class="opcion"><input type="radio" name="q_tiempo" value="48h"> <span class="opcion-text">Menos de 48 horas</span></label>
          <label class="opcion"><input type="radio" name="q_tiempo" value="2a5d"> <span class="opcion-text">2 a 5 días hábiles</span></label>
          <label class="opcion"><input type="radio" name="q_tiempo" value="1a2sem"> <span class="opcion-text">1 a 2 semanas</span></label>
          <label class="opcion"><input type="radio" name="q_tiempo" value="2a4sem"> <span class="opcion-text">2 a 4 semanas</span></label>
          <label class="opcion"><input type="radio" name="q_tiempo" value="mas1mes"> <span class="opcion-text">Más de 1 mes</span></label>
        </div>
      </div>

      <div class="pregunta" id="pq-conformidad">
        <div class="pregunta-label">Q12. En general, ¿qué tan conforme quedó con el proceso (independiente de la tasa)?</div>
        <div class="opciones">
          <label class="opcion"><input type="radio" name="q_conformidad" value="muy_conforme"> <span class="opcion-text">Muy conforme</span></label>
          <label class="opcion"><input type="radio" name="q_conformidad" value="conforme"> <span class="opcion-text">Conforme</span></label>
          <label class="opcion"><input type="radio" name="q_conformidad" value="neutral"> <span class="opcion-text">Neutral</span></label>
          <label class="opcion"><input type="radio" name="q_conformidad" value="disconforme"> <span class="opcion-text">Disconforme</span></label>
          <label class="opcion"><input type="radio" name="q_conformidad" value="muy_disconforme"> <span class="opcion-text">Muy disconforme</span></label>
        </div>
      </div>

      <div class="pregunta" id="pq-engorroso">
        <div class="pregunta-label">Q13. ¿Qué fue lo más engorroso del proceso? <span style="color:var(--text3);font-weight:400">(opcional)</span></div>
        <div class="opciones">
          <label class="opcion"><input type="radio" name="q_engorroso" value="documentacion"> <span class="opcion-text">La cantidad de documentación exigida</span></label>
          <label class="opcion"><input type="radio" name="q_engorroso" value="lentitud"> <span class="opcion-text">La lentitud de las respuestas</span></label>
          <label class="opcion"><input type="radio" name="q_engorroso" value="interlocutor"> <span class="opcion-text">La falta de un interlocutor claro</span></label>
          <label class="opcion"><input type="radio" name="q_engorroso" value="transparencia"> <span class="opcion-text">La poca transparencia sobre cómo se define la tasa</span></label>
          <label class="opcion"><input type="radio" name="q_engorroso" value="ninguno"> <span class="opcion-text">Nada, fue ágil</span></label>
        </div>
      </div>
    </div>
  </div>
```

- [ ] **Step 2: Agregar el toggle de la compuerta Q7**

Después del bloque agregado en la Task 1 (la función `setBancoOtroVisible` y su listener), insertar:

```javascript
// Compuerta Q7: mostrar Q8–Q13 solo si respondió "Sí"; al ocultar, resetear ese sub-bloque
function setProcesoVisible(show){
  const sub = document.getElementById('sub-proceso');
  if(!sub) return;
  sub.style.display = show ? 'block' : 'none';
  if(!show){
    sub.querySelectorAll('input[type=radio]').forEach(r => {
      r.checked = false;
      const opt = r.closest('.opcion');
      if(opt) opt.classList.remove('selected');
    });
  }
}
document.querySelectorAll('input[name="q_solicito"]').forEach(r => {
  r.addEventListener('change', function(){ setProcesoVisible(this.value === 'si'); });
});
```

- [ ] **Step 3: Renombrar el label del stepper (Paso 4)**

En el stepper (línea ~208), buscar:

```html
      <div class="step-circle pending" id="step-4"><span>4</span><span class="step-label">Garantías</span></div>
```

Reemplazar por:

```html
      <div class="step-circle pending" id="step-4"><span>4</span><span class="step-label">Proceso</span></div>
```

- [ ] **Step 4: Actualizar `nombresPasos`**

En `updateProgress` (línea ~526), buscar:

```javascript
  const nombresPasos = ['Tamaño','Capital','FOGAPE','Garantías','Contacto'];
```

Reemplazar por:

```javascript
  const nombresPasos = ['Tamaño','Capital','FOGAPE','Proceso','Contacto'];
```

- [ ] **Step 5: Actualizar `collectAnswers()` (quitar q9/q10, agregar las 7 claves)**

En `collectAnswers` (línea ~731), buscar:

```javascript
    q_fogape_monto: getRadio('q_fogape_monto'),
    q9: getRadio('q9'),
    q10: getRadio('q10')
  };
```

Reemplazar por:

```javascript
    q_fogape_monto: getRadio('q_fogape_monto'),
    q_solicito: getRadio('q_solicito'),
    q_canal: getRadio('q_canal'),
    q_asistencia: getRadio('q_asistencia'),
    q_docs_tasa: getRadio('q_docs_tasa'),
    q_tiempo: getRadio('q_tiempo'),
    q_conformidad: getRadio('q_conformidad'),
    q_engorroso: getRadio('q_engorroso')
  };
```

- [ ] **Step 6: Actualizar `getFormSnapshot()` (quitar q9/q10, agregar las 7 claves)**

En `getFormSnapshot` (línea ~635), buscar:

```javascript
      q_fogape_monto: getRadioValue('q_fogape_monto'),
      q9: getRadioValue('q9'),
      q10: getRadioValue('q10')
    }
```

Reemplazar por:

```javascript
      q_fogape_monto: getRadioValue('q_fogape_monto'),
      q_solicito: getRadioValue('q_solicito'),
      q_canal: getRadioValue('q_canal'),
      q_asistencia: getRadioValue('q_asistencia'),
      q_docs_tasa: getRadioValue('q_docs_tasa'),
      q_tiempo: getRadioValue('q_tiempo'),
      q_conformidad: getRadioValue('q_conformidad'),
      q_engorroso: getRadioValue('q_engorroso')
    }
```

- [ ] **Step 7: Revelar el sub-bloque al restaurar un borrador con "Sí"**

En `restoreLocalDraft`, buscar el bloque agregado en la Task 1:

```javascript
  if(snap.radios && snap.radios.q_banco === 'otro'){
    setBancoOtroVisible(true);
    const t = document.getElementById('q_banco_otro');
    if(t) t.value = snap.bancoOtro || '';
  }
```

Reemplazar por (agrega la revelación del sub-proceso):

```javascript
  if(snap.radios && snap.radios.q_banco === 'otro'){
    setBancoOtroVisible(true);
    const t = document.getElementById('q_banco_otro');
    if(t) t.value = snap.bancoOtro || '';
  }
  if(snap.radios && snap.radios.q_solicito === 'si'){
    setProcesoVisible(true);
  }
```

- [ ] **Step 8: Verificar en navegador**

Servir (si no está corriendo): `cd public && python3 -m http.server 8000` y abrir `http://localhost:8000`. Completar Paso 1 (banco) y avanzar hasta el Paso 4.

Verificaciones:
1. El stepper marca el paso 4 como "Proceso" (desktop) y el label mobile dice "Paso 4 de 5 — Proceso".
2. El título del bloque es "El proceso de solicitud". Solo se ve Q7; Q8–Q13 ocultas.
3. Seleccionar "Sí, hemos solicitado" → aparecen Q8–Q13 (a ancho completo, sin sangría). Seleccionar "No, no hemos solicitado" → se ocultan.
4. Volver a "Sí", responder Q8 y Q11, luego cambiar a "No" y de nuevo a "Sí" → Q8–Q13 aparecen **en blanco** (se resetearon al ocultar).
5. Con "Sí" + algunas respuestas, en consola:

```javascript
collectAnswers()
// Incluye q_solicito, q_canal, ... q_engorroso con TEXTO legible.
// NO existen las claves q9 ni q10.
```

6. Recargar (F5) estando en el Paso 4 con "Sí" seleccionado → al restaurar, el sub-bloque Q8–Q13 vuelve a estar visible.

- [ ] **Step 9: Commit**

```bash
git add public/index.html
git commit -m "feat: bloque Proceso de solicitud reemplaza el Paso 4 Garantias"
```

---

## Task 3: Segmentación GA4 `solicito_financiamiento`

**Files:**
- Modify: `public/index.html` (`goNext` → `etapa_completada`; `showSuccess` → `form_completado`)

**Interfaces:**
- Consumes: `getRadioValue('q_solicito')` (devuelve `'si'`/`'no'`/`''`), definida por los radios de la Task 2.
- Produces: parámetro `solicito_financiamiento` en los eventos GA4 `etapa_completada` y `form_completado`.

- [ ] **Step 1: Agregar el parámetro a `etapa_completada`**

En `goNext` (línea ~457), buscar:

```javascript
    gtag('event', 'etapa_completada', {
      etapa_numero: currentBloque + 1,
      etapa_nombre: etapas[currentBloque] || 'Desconocida',
      form_name: 'Encuesta MP Chile'
    });
```

Reemplazar por:

```javascript
    gtag('event', 'etapa_completada', {
      etapa_numero: currentBloque + 1,
      etapa_nombre: etapas[currentBloque] || 'Desconocida',
      form_name: 'Encuesta MP Chile',
      solicito_financiamiento: getRadioValue('q_solicito')
    });
```

- [ ] **Step 2: Agregar el parámetro a `form_completado`**

En `showSuccess` (línea ~592), buscar:

```javascript
    gtag('event', 'form_completado', {
      form_name: 'Encuesta MP Chile',
      value: 1
    });
```

Reemplazar por:

```javascript
    gtag('event', 'form_completado', {
      form_name: 'Encuesta MP Chile',
      value: 1,
      solicito_financiamiento: getRadioValue('q_solicito')
    });
```

- [ ] **Step 3: Verificar en navegador**

Servir y abrir `http://localhost:8000`. Completar hasta el Paso 4, seleccionar "Sí, hemos solicitado", clic en "Continuar →". En la consola:

```javascript
window.dataLayer.filter(a => a[1] === 'etapa_completada').pop()[2]
// El objeto de params debe incluir: solicito_financiamiento: "si"
// (y etapa_nombre: "Requisitos" — NO "Proceso")
```

Completar el envío final (Paso 5, email válido, "Enviar y recibir mi informe →"). En la consola:

```javascript
window.dataLayer.filter(a => a[1] === 'form_completado').pop()[2]
// Debe incluir: solicito_financiamiento: "si", value: 1
```

Repetir eligiendo "No, no hemos solicitado" en Q7 → el parámetro debe ser `"no"`.

- [ ] **Step 4: Commit**

```bash
git add public/index.html
git commit -m "feat: segmentacion GA4 solicito_financiamiento en etapa y form completado"
```

---

## Self-Review

**1. Spec coverage** (spec `2026-07-06-bloque-proceso-solicitud-design.md`):

- §2 Banco principal (radio, Otro, obligatorio, Paso 1, values, sin GA4) → Task 1 (HTML, toggle, validación, captura, restore). ✓
- §3 Reemplazo Paso 4 (drop q9/q10) → Task 2 Step 1 + Steps 5/6. ✓
- §4 Contenido Proceso (compuerta Q7 + Q8–Q13, Q13 opcional) → Task 2 Step 1. ✓
- §5.1 HTML bloque-0 → Task 1 Step 1. §5.2 HTML bloque-3 → Task 2 Step 1. §5.3 stepper label → Task 2 Step 3. §5.4 nombresPasos → Task 2 Step 4. §5.5 collectAnswers → Task 1 Step 4 + Task 2 Step 5. §5.6 validateBloque(0) → Task 1 Step 3. ✓
- §6 GA4 (mantener 'Requisitos'; param solicito_financiamiento en etapa_completada + form_completado; banco sin GA4) → Global Constraints + Task 3. ✓
- §7 Dependencia planilla → Global Constraints (marcada como acción externa, no tarea de código). ✓
- §8 Fuera de alcance (sin footprint checkbox, sin diferenciar producto, Proceso opcional, reuso de CSS) → respetado; ningún task los agrega. ✓

Cobertura completa, sin gaps.

**2. Placeholder scan:** sin "TBD"/"TODO"/"handle edge cases". Cada step muestra el código exacto y su verificación. ✓

**3. Type/consistency check:**
- Nombres de inputs idénticos entre HTML, `collectAnswers` (getRadio) y `getFormSnapshot` (getRadioValue): `q_banco`, `q_banco_otro`, `q_solicito`, `q_canal`, `q_asistencia`, `q_docs_tasa`, `q_tiempo`, `q_conformidad`, `q_engorroso`. ✓
- Funciones definidas antes de usarse: `setBancoOtroVisible` (Task 1 Step 2) usada en restore (Task 1 Step 6); `setProcesoVisible` (Task 2 Step 2) usada en restore (Task 2 Step 7). ✓
- `getRadioValue('q_solicito')` en Task 3 devuelve `'si'`/`'no'`/`''` porque los radios `q_solicito` tienen `value="si"`/`"no"` (Task 2 Step 1). ✓
- Ordena de ediciones sobre `collectAnswers`/`getFormSnapshot`/`restoreLocalDraft`: Task 1 toca la zona de `q0` y agrega el bloque de restore del banco; Task 2 toca la zona de `q_fogape_monto`/`q9`/`q10` y extiende el bloque de restore. Regiones distintas, sin colisión, en orden. ✓
