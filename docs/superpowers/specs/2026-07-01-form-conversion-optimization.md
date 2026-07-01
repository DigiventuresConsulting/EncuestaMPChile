# Formulario — Optimización de Conversión (Paso 1) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reducir la fricción del primer paso del formulario (`public/index.html`) para tráfico frío de Meta Ads, moviendo el compromiso de bajo esfuerzo (tamaño de empresa) antes del pedido de datos personales, y recortando campos innecesarios en el paso de contacto.

**Architecture:** Cambios contenidos enteramente en `public/index.html` (HTML/CSS/JS inline, sin build step). No se toca el backend de Google Apps Script — el payload sigue enviando las claves `mailEmpresa` y `mailContacto` para no romper las columnas del Google Sheet, aunque ahora se completan desde un único input.

**Tech Stack:** HTML, CSS y JavaScript vanilla inline, sin dependencias ni framework.

## Global Constraints

- No hay build tooling, linter ni test suite en este proyecto — cada tarea se verifica cargando la página directamente (`python3 -m http.server 8000 --directory public`), no con comandos de test automatizados.
- Un solo archivo (`public/index.html`) concentra markup, CSS y JS — todas las tareas modifican ese archivo.
- No modificar el endpoint de Google Apps Script ni su schema de columnas — el payload debe seguir enviando `mailEmpresa` y `mailContacto` como claves separadas, aunque ambas tomen el mismo valor.
- Mantener el resto del formulario (bloques Capital, FOGAPE, Requisitos, lógica condicional Q1–Q10) sin cambios — el scope de esta spec es exclusivamente el paso de Contacto/Tamaño y la navegación entre pasos.
- Todo texto visible va en español (rioplatense/chileno neutro, como el resto del copy existente).

---

## Decisiones de producto (ya confirmadas, no reabrir)

1. **Orden de pasos:** Tamaño de empresa (hoy `bloque-1`) pasa a ser el **paso 1**. Contacto (hoy `bloque-0`) pasa a ser el **paso 2**. El resto de los bloques (Capital, FOGAPE, Requisitos) no cambian de posición relativa.
2. **Mail Empresa + Mail Contacto → un solo campo** "Correo electrónico". El payload sigue mandando ambas claves (`mailEmpresa`, `mailContacto`) con el mismo valor, para no requerir cambios en el Apps Script/Sheet.
3. **Teléfono pasa a ser opcional** en el paso de Contacto (se sigue pidiendo, pero no bloquea el avance).
4. **Orden de campos en Contacto:** Correo electrónico → RUT → Teléfono (opcional). El campo más fácil primero, el más sensible al final.
5. Se agrega una línea de confianza + estimador de tiempo junto a los campos de Contacto.
6. El botón del primer paso cambia su copy a "Comenzar encuesta →"; el resto de los pasos mantienen "Continuar →"; el último mantiene "Enviar encuesta".
7. **Prioridad mobile: la gran mayoría del tráfico es Android**, no iOS. Los fixes de layout de la Task 5 priorizan anchos de pantalla Android típicos (~360-412px, hasta 320px en equipos de gama baja) y el estándar de touch target de Material Design (48dp mínimo) por sobre las guías de iOS.

---

## File Structure

Un solo archivo modificado en las 5 tareas:

- Modify: `public/index.html`
  - Markup: bloques `bloque-0`/`bloque-1` (reorder + merge de campos), progress bar (`#progress-bar` labels + nuevo indicador `#step-label-mobile`), línea de confianza nueva.
  - CSS: nueva clase `.form-trust` para la línea de confianza; nueva clase `.step-label-mobile`; extensión del `@media(max-width:600px)` existente con paddings/tipografía/touch targets mobile.
  - JS: `validateBloque()`, `goNext()`, `updateProgress()`, `submitForm()`, `savePartial()`, `getFormSnapshot()`, `restoreLocalDraft()`, `resetFormExceptRut()`.

---

### Task 1: Reordenar los pasos — Tamaño primero, Contacto segundo

**Files:**
- Modify: `public/index.html` (bloque de progress bar, bloques `bloque-0`/`bloque-1`, funciones `validateBloque`, `goNext`, `updateProgress`)

**Interfaces:**
- Produces: `bloque-0` = contenido de Tamaño (antes en `bloque-1`); `bloque-1` = contenido de Contacto (antes en `bloque-0`). Todo lo que referencie estos IDs por índice numérico (`currentBloque`) ahora corresponde al nuevo orden.

- [ ] **Step 1: Actualizar las etiquetas de la progress bar**

Ubicar el bloque `<div class="progress-wrap">` (contiene `id="progress-bar"`) y reemplazar los `step-label` para que reflejen el nuevo orden:

```html
  <!-- PROGRESS BAR -->
  <div class="progress-wrap">
    <div class="progress" id="progress-bar">
      <div class="step-circle active" id="step-1"><span>1</span><span class="step-label">Tamaño</span></div>
      <div class="step-line" id="line-1-2"></div>
      <div class="step-circle pending" id="step-2"><span>2</span><span class="step-label">Contacto</span></div>
      <div class="step-line" id="line-2-3"></div>
      <div class="step-circle pending" id="step-3"><span>3</span><span class="step-label">Capital</span></div>
      <div class="step-line" id="line-3-4"></div>
      <div class="step-circle pending" id="step-4"><span>4</span><span class="step-label">FOGAPE</span></div>
      <div class="step-line" id="line-4-5"></div>
      <div class="step-circle pending" id="step-5"><span>5</span><span class="step-label">Requisitos</span></div>
    </div>
  </div>
```

- [ ] **Step 2: Intercambiar el contenido de `bloque-0` y `bloque-1`**

`bloque-0` pasa a contener la pregunta de Tamaño (con `class="bloque active"`, porque es el primer paso visible), y `bloque-1` pasa a contener los campos de Contacto (con `class="bloque"`, sin `active`):

```html
  <!-- BLOQUE 0: TAMAÑO -->
  <div class="bloque active" id="bloque-0">
    <div class="bloque-title">Caracterización de la Empresa</div>
    <div class="pregunta" id="pq-q0">
      <div class="pregunta-label">Q0. Tamaño de la Empresa (Facturación Anual Neta)</div>
      <div class="opciones">
        <label class="opcion"><input type="radio" name="q0" value="micro"> <span class="opcion-text"><strong>Microempresa:</strong> Hasta 2.400 UF (Hasta $105.000 USD)</span></label>
        <label class="opcion"><input type="radio" name="q0" value="pequena"> <span class="opcion-text"><strong>Pequeña Empresa:</strong> Entre 2.401 y 25.000 UF (De $105.000 a $1,1 Millones USD)</span></label>
        <label class="opcion"><input type="radio" name="q0" value="mediana"> <span class="opcion-text"><strong>Mediana Empresa:</strong> Entre 25.001 y 100.000 UF (De $1,1 a $4,4 Millones USD)</span></label>
        <label class="opcion"><input type="radio" name="q0" value="gran"> <span class="opcion-text"><strong>Gran Empresa:</strong> Más de 100.000 UF (Más de $4,4 Millones USD)</span></label>
      </div>
    </div>
  </div>

  <!-- BLOQUE 1: CONTACTO -->
  <div class="bloque" id="bloque-1">
    <div class="bloque-title">Información de Contacto</div>
    <div class="field" id="f-rut">
      <label>RUT <span class="req">*</span></label>
      <input type="text" id="rut" placeholder="00-00000000-0" maxlength="12">
    </div>
    <div class="grid-2">
      <div class="field" id="f-email-empresa">
        <label>Mail Empresa <span class="req">*</span></label>
        <input type="email" id="email-empresa" placeholder="empresa@ejemplo.cl">
      </div>
      <div class="field" id="f-email">
        <label>Mail Contacto <span class="req">*</span></label>
        <input type="email" id="email" placeholder="nombre@ejemplo.cl">
      </div>
    </div>
    <div class="field" id="f-telefono">
      <label>Teléfono <span class="req">*</span></label>
      <input type="tel" id="telefono" placeholder="+56 9 XXXX XXXX">
    </div>
  </div>
```

(El contenido interno de Contacto se termina de editar en la Task 2 — acá solo importa que quedó en `bloque-1` con `class="bloque"` sin `active`.)

- [ ] **Step 3: Invertir las validaciones `b === 0` / `b === 1` en `validateBloque()`**

```javascript
function validateBloque(b){
  if(b === 0){
    if(!document.querySelector('input[name="q0"]:checked')){
      alert('Por favor seleccione el tamaño de su empresa.'); return false;
    }
    return true;
  }
  if(b === 1){
    const emailEmpresa = document.getElementById('email-empresa').value.trim();
    const rut = document.getElementById('rut').value.trim();
    const email = document.getElementById('email').value.trim();
    const telefono = document.getElementById('telefono').value.trim();
    if(!emailEmpresa || !rut || !email || !telefono){
      alert('Por favor complete todos los campos obligatorios.');
      return false;
    }
    if(!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(emailEmpresa)){
      alert('Por favor ingrese un email de empresa válido.');
      return false;
    }
    if(!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)){
      alert('Por favor ingrese un email de contacto válido.');
      return false;
    }
    return true;
  }
```

(Esta versión todavía valida los 2 campos de mail y el teléfono obligatorio — se simplifica en la Task 2. Acá el objetivo único es que `b===0` valide Tamaño y `b===1` valide Contacto.)

- [ ] **Step 4: Invertir los eventos de Meta Pixel en `goNext()`**

```javascript
function goNext(){
  if(!validateBloque(currentBloque)) return;
  // Evento Meta Pixel en el primer botón Continuar (paso Tamaño)
  if(currentBloque === 0 && typeof fbq !== 'undefined') {
    fbq('trackCustom', 'InicioEncuesta', {
      content_name: 'Encuesta MP Chile',
      tamanio: getRadio('q0')
    });
  }
  // Evento Meta Pixel en el botón Continuar del paso Contacto
  if(currentBloque === 1 && typeof fbq !== 'undefined') {
    fbq('trackCustom', 'IniciarPago', {
      content_name: 'Encuesta MP Chile - Contacto completado'
    });
  }
  if(currentBloque < totalBloques - 1){
    document.getElementById('bloque-'+currentBloque).classList.remove('active');
    currentBloque++;
    document.getElementById('bloque-'+currentBloque).classList.add('active');
    updateProgress();
    saveLocalDraft();
    window.scrollTo({top:0, behavior:'smooth'});
  } else {
    // Submit final
    submitForm();
  }
}
```

- [ ] **Step 5: Reordenar `nombresPasos` en `updateProgress()`**

```javascript
  // Clarity: etiquetar el paso actual para segmentar sesiones por avance
  const nombresPasos = ['Tamaño','Contacto','Capital','FOGAPE','Requisitos'];
  if(window.clarity) clarity('set','paso_encuesta', nombresPasos[currentBloque]);
```

(El resto de `updateProgress()` no cambia.)

- [ ] **Step 6: Verificación manual**

```bash
python3 -m http.server 8000 --directory public
```

Abrir `http://localhost:8000` en el navegador y verificar:
- La página carga con la pregunta "Q0. Tamaño de la Empresa" visible primero (no los campos de contacto).
- El círculo 1 de la progress bar dice "Tamaño" y está activo (verde/borde resaltado); el círculo 2 dice "Contacto" y está pendiente.
- Seleccionar un tamaño y hacer click en el botón → avanza al paso de Contacto (con los 4 campos, sin cambios visuales todavía respecto a antes).
- Botón "← Anterior" desde Contacto vuelve a Tamaño y la selección previa sigue marcada.
- Completar el resto del flujo hasta el final (Capital → FOGAPE → Requisitos → enviar) sin errores de consola.
- Abrir DevTools → Network, filtrar por el request al `script.google.com` en el envío final, y confirmar que el payload sigue incluyendo `q0`, `mailEmpresa`, `mailContacto`, `rut`, `telefono`, etc. (aunque el request es `no-cors` y no se puede leer la respuesta, sí se puede ver el body enviado en la pestaña "Payload"/"Request" del navegador).

- [ ] **Step 7: Commit**

```bash
git add public/index.html
git commit -m "reorder form steps: Tamaño first, Contacto second"
```

---

### Task 2: Simplificar el paso de Contacto — fusionar mails, teléfono opcional, reordenar campos

**Files:**
- Modify: `public/index.html` (bloque `bloque-1` de Contacto, `validateBloque()`, `submitForm()`, `savePartial()`, `getFormSnapshot()`, `restoreLocalDraft()`, `resetFormExceptRut()`)

**Interfaces:**
- Consumes: `bloque-1` con `class="bloque"` producido en la Task 1, conteniendo los campos `rut`, `email-empresa`, `email`, `telefono`.
- Produces: campo único `id="email"` (label "Correo electrónico") reemplaza a `email-empresa` + `email` combinados. El campo `email-empresa` deja de existir en el DOM. El input `telefono` deja de ser obligatorio.

- [ ] **Step 1: Reescribir el markup de `bloque-1` (Contacto) con el nuevo orden y campos**

```html
  <!-- BLOQUE 1: CONTACTO -->
  <div class="bloque" id="bloque-1">
    <div class="bloque-title">Información de Contacto</div>
    <div class="field" id="f-email">
      <label>Correo electrónico <span class="req">*</span></label>
      <input type="email" id="email" placeholder="nombre@empresa.cl">
    </div>
    <div class="field" id="f-rut">
      <label>RUT <span class="req">*</span></label>
      <input type="text" id="rut" placeholder="00-00000000-0" maxlength="12">
    </div>
    <div class="field" id="f-telefono">
      <label>Teléfono <span style="color:var(--text3);font-weight:400">(opcional)</span></label>
      <input type="tel" id="telefono" placeholder="+56 9 XXXX XXXX">
    </div>
  </div>
```

- [ ] **Step 2: Simplificar la validación de `b === 1` en `validateBloque()`**

```javascript
  if(b === 1){
    const rut = document.getElementById('rut').value.trim();
    const email = document.getElementById('email').value.trim();
    if(!rut || !email){
      alert('Por favor complete todos los campos obligatorios.');
      return false;
    }
    if(!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)){
      alert('Por favor ingrese un email válido.');
      return false;
    }
    return true;
  }
```

- [ ] **Step 3: Actualizar la construcción del payload en `submitForm()`**

```javascript
  // Recopilar todas las respuestas
  const correo = document.getElementById('email').value.trim();
  const payload = {
    estado: 'completo',
    mailEmpresa: correo,
    rut: document.getElementById('rut').value.trim(),
    mailContacto: correo,
    telefono: document.getElementById('telefono').value.trim(),
    q0: getRadio('q0'),
    q1: getRadio('q1'),
    q2: getRadio('q2'),
    q3: getRadio('q3'),
    q3a: getRadio('q3a'),
    q3b: getRadio('q3b'),
    q4: getRadio('q4'),
    q5: getRadio('q5'),
    q6: getRadio('q6'),
    q7: getRadio('q7'),
    q7a: getRadio('q7a'),
    q7b: getRadio('q7b'),
    q8: getRadio('q8'),
    q9: getRadio('q9'),
    q10: getRadio('q10')
  };
```

(`mailEmpresa` y `mailContacto` siguen viajando como claves separadas en el JSON — mismo valor, sin cambios en el Apps Script.)

- [ ] **Step 4: Actualizar la construcción del payload en `savePartial()`**

```javascript
function savePartial(){
  const rut = document.getElementById('rut').value.trim();
  if(!rut){
    alert('Para guardar sus respuestas parciales, complete el RUT en el primer paso.');
    return;
  }

  const correo = document.getElementById('email').value.trim();
  const payload = {
    estado: 'parcial',
    mailEmpresa: correo,
    rut: rut,
    mailContacto: correo,
    telefono: document.getElementById('telefono').value.trim(),
    q0: getRadio('q0'),
    q1: getRadio('q1'),
    q2: getRadio('q2'),
    q3: getRadio('q3'),
    q3a: getRadio('q3a'),
    q3b: getRadio('q3b'),
    q4: getRadio('q4'),
    q5: getRadio('q5'),
    q6: getRadio('q6'),
    q7: getRadio('q7'),
    q7a: getRadio('q7a'),
    q7b: getRadio('q7b'),
    q8: getRadio('q8'),
    q9: getRadio('q9'),
    q10: getRadio('q10')
  };
```

(El resto de la función `savePartial()` — botón de estado, `fetch`, `.then/.catch` — no cambia.)

- [ ] **Step 5: Actualizar `getFormSnapshot()` para usar un único campo de correo**

```javascript
function getFormSnapshot(){
  return {
    currentBloque: currentBloque,
    correo: document.getElementById('email').value.trim(),
    rut: document.getElementById('rut').value.trim(),
    telefono: document.getElementById('telefono').value.trim(),
    radios: {
      q0: getRadioValue('q0'),
      q1: getRadioValue('q1'),
      q2: getRadioValue('q2'),
      q3: getRadioValue('q3'),
      q3a: getRadioValue('q3a'),
      q3b: getRadioValue('q3b'),
      q4: getRadioValue('q4'),
      q5: getRadioValue('q5'),
      q6: getRadioValue('q6'),
      q7: getRadioValue('q7'),
      q7a: getRadioValue('q7a'),
      q7b: getRadioValue('q7b'),
      q8: getRadioValue('q8'),
      q9: getRadioValue('q9'),
      q10: getRadioValue('q10')
    }
  };
}
```

- [ ] **Step 6: Actualizar `restoreLocalDraft()` — leer `correo` con fallback a los nombres viejos**

Los borradores guardados en `localStorage` antes de este cambio tienen `mailEmpresa`/`mailContacto` en vez de `correo`. El fallback evita perder el borrador de alguien que tenga la encuesta a medio completar en el navegador al momento del deploy:

```javascript
  document.getElementById('email').value = snap.correo || snap.mailContacto || snap.mailEmpresa || '';
  document.getElementById('rut').value = snap.rut || '';
  document.getElementById('telefono').value = snap.telefono || '';
```

- [ ] **Step 7: Actualizar `resetFormExceptRut()` — quitar la referencia a `email-empresa`**

```javascript
function resetFormExceptRut(newRut){
  // Limpia todos los campos de contacto (excepto el RUT que se está escribiendo) y todas las selecciones
  document.getElementById('email').value = '';
  document.getElementById('telefono').value = '';

  document.querySelectorAll('.opciones input[type=radio]').forEach(input => {
    input.checked = false;
    const opt = input.closest('.opcion');
    if(opt) opt.classList.remove('selected');
  });

  // Oculta subpreguntas condicionales que pudieran haber quedado visibles
  ['sq-q2-clp','sq-q2-uf','pq-q3','pq-q4','sq-q3a','sq-q3b',
   'sq-q6-clp','sq-q6-uf','pq-q7','pq-q8','sq-q7a','sq-q7b'].forEach(hide);

  if(currentBloque !== 0){
    document.getElementById('bloque-'+currentBloque).classList.remove('active');
    currentBloque = 0;
    document.getElementById('bloque-0').classList.add('active');
    updateProgress();
  }

  document.getElementById('rut').value = newRut;
  clearLocalDraft();
}
```

- [ ] **Step 8: Verificación manual**

```bash
python3 -m http.server 8000 --directory public
```

En el navegador:
- Avanzar al paso de Contacto: debe verse un único campo "Correo electrónico" arriba, "RUT" en el medio, "Teléfono (opcional)" abajo, sin el grid de 2 columnas.
- Dejar el teléfono vacío e intentar avanzar: debe permitir continuar (no bloquea).
- Dejar el correo o el RUT vacíos e intentar avanzar: debe mostrar el alert de campos obligatorios.
- Poner un correo inválido (sin `@`): debe mostrar el alert de email inválido.
- Completar correo + RUT, avanzar, volver atrás con "← Anterior": los valores deben seguir cargados (usa `saveLocalDraft`/`restoreLocalDraft` internamente al recargar la página — para probarlo: completar el paso, recargar la página con F5, y confirmar que el correo/RUT/teléfono se restauran).
- Completar toda la encuesta y enviarla; en DevTools → Network, inspeccionar el body del request final y confirmar que `mailEmpresa` y `mailContacto` llegan con el mismo valor (el que se escribió en el único campo de correo).
- Probar también "Guardar y continuar más tarde" desde el paso de Contacto con el correo completo: no debe tirar error de consola.

- [ ] **Step 9: Commit**

```bash
git add public/index.html
git commit -m "merge email fields, make phone optional, reorder contact fields"
```

---

### Task 3: Agregar línea de confianza + estimador de tiempo en el paso de Contacto

**Files:**
- Modify: `public/index.html` (CSS dentro de `<style>`, markup de `bloque-1`)

**Interfaces:**
- Consumes: `bloque-1` con la estructura de campos producida en la Task 2 (`f-email`, `f-rut`, `f-telefono`).
- Produces: nuevo elemento `<p class="form-trust">` visible solo en el paso de Contacto.

- [ ] **Step 1: Agregar la clase CSS `.form-trust`**

Ubicar la sección `/* ERROR */` dentro del `<style>` y agregar el bloque nuevo justo antes:

```css
/* TRUST COPY */
.form-trust{font-size:12px;color:var(--text3);margin-top:16px;line-height:1.6;display:flex;align-items:flex-start;gap:6px}
.form-trust span{flex-shrink:0}

/* ERROR */
```

- [ ] **Step 2: Agregar el párrafo de confianza al final de `bloque-1` (Contacto)**

```html
  <!-- BLOQUE 1: CONTACTO -->
  <div class="bloque" id="bloque-1">
    <div class="bloque-title">Información de Contacto</div>
    <div class="field" id="f-email">
      <label>Correo electrónico <span class="req">*</span></label>
      <input type="email" id="email" placeholder="nombre@empresa.cl">
    </div>
    <div class="field" id="f-rut">
      <label>RUT <span class="req">*</span></label>
      <input type="text" id="rut" placeholder="00-00000000-0" maxlength="12">
    </div>
    <div class="field" id="f-telefono">
      <label>Teléfono <span style="color:var(--text3);font-weight:400">(opcional)</span></label>
      <input type="tel" id="telefono" placeholder="+56 9 XXXX XXXX">
    </div>
    <p class="form-trust"><span>🔒</span> Sus datos son confidenciales y se usan únicamente para este estudio. No los compartimos con bancos ni terceros. Le tomará 2 minutos.</p>
  </div>
```

- [ ] **Step 3: Verificación manual**

```bash
python3 -m http.server 8000 --directory public
```

Avanzar al paso de Contacto y verificar visualmente que el texto de confianza aparece debajo del campo Teléfono, en gris tenue (mismo tono que el texto secundario del resto del form), sin romper el layout ni superponerse con el botón "Continuar". Revisar también en la vista mobile (DevTools → toggle device toolbar, ancho 375px) que el texto no desborda el card.

- [ ] **Step 4: Commit**

```bash
git add public/index.html
git commit -m "add trust copy and time estimate to contact step"
```

---

### Task 4: Copy del botón por paso

**Files:**
- Modify: `public/index.html` (`updateProgress()`)

**Interfaces:**
- Consumes: `currentBloque` (0–4), `totalBloques` (5), ambos ya existentes.
- Produces: texto del botón `#btn-next` varía según `currentBloque`.

- [ ] **Step 1: Reemplazar la línea de texto del botón en `updateProgress()`**

```javascript
  const backBtn = document.getElementById('btn-back');
  backBtn.style.visibility = currentBloque > 0 ? 'visible' : 'hidden';
  const nextBtn = document.getElementById('btn-next');
  const textosBoton = ['Comenzar encuesta →','Continuar →','Continuar →','Continuar →','Enviar encuesta'];
  nextBtn.textContent = textosBoton[currentBloque];
```

(Esto reemplaza la línea `nextBtn.textContent = currentBloque === totalBloques - 1 ? 'Enviar encuesta' : 'Continuar →';`. El resto de `updateProgress()` no cambia.)

- [ ] **Step 2: Verificación manual**

```bash
python3 -m http.server 8000 --directory public
```

Verificar que:
- En el paso 1 (Tamaño) el botón dice "Comenzar encuesta →".
- En los pasos 2, 3 y 4 (Contacto, Capital, FOGAPE) dice "Continuar →".
- En el paso 5 (Requisitos) dice "Enviar encuesta".
- Al volver con "← Anterior" desde cualquier paso, el texto del botón se actualiza correctamente al del paso actual (no queda pegado el texto del paso anterior).

- [ ] **Step 3: Commit**

```bash
git add public/index.html
git commit -m "add per-step button copy"
```

---

### Task 5: Ajustes mobile — layout, touch targets y progress bar

**Files:**
- Modify: `public/index.html` (CSS dentro de `<style>`, markup de `.progress-wrap`, función `updateProgress()`)

**Interfaces:**
- Consumes: `nombresPasos` y `totalBloques` (definidos en `updateProgress()`, reordenados en la Task 1); estructura de `.progress-wrap` / `#progress-bar` producida en la Task 1.
- Produces: nuevo elemento `#step-label-mobile` dentro de `.progress-wrap`, actualizado en cada llamada a `updateProgress()`.

Motivación: con un solo `@media` en todo el archivo (el de `.grid-2`), el resto del layout es fijo. En una pantalla Android de 360-375px de ancho, `.form-wrap` (`padding:0 40px 80px`) + `.form-card` (`padding:40px`) apilados dejan solo ~215px de ancho útil para el contenido — las opciones de radio con texto largo (Q1, Q4, Q9, etc.) wrappean en varias líneas y el form se siente apretado en todas las pantallas, no solo en Contacto.

- [ ] **Step 1: Subir el `font-size` base de los inputs de texto a 16px**

```css
.field input[type=text],.field input[type=email],.field input[type=tel]{width:100%;padding:11px 14px;border:1.5px solid var(--border);border-radius:var(--radius-sm);font-size:16px;color:var(--text);background:var(--bg);outline:none;transition:border-color .15s}
```

(Único cambio: `font-size:14px` → `font-size:16px`. Evita el auto-zoom de Safari en iOS al enfocar el campo — no afecta a Android, pero no tiene costo y deja el sitio a salvo para la minoría de tráfico iOS.)

- [ ] **Step 2: Extender el `@media(max-width:600px)` existente con paddings reducidos**

Ubicar la línea `@media(max-width:600px){.grid-2{grid-template-columns:1fr}}` y reemplazarla por:

```css
@media(max-width:600px){
  .grid-2{grid-template-columns:1fr}
  .header{padding:14px 20px 4px}
  .logo img{height:56px}
  .hero{padding:32px 20px 24px}
  .intro-section{padding:4px 20px 6px;text-align:left}
  .form-wrap{padding:0 16px 60px}
  .form-card{padding:24px 20px}
  .btn-back{min-height:48px}
  .step-label{display:none}
  .step-label-mobile{display:block}
}
```

Con estos valores, en una pantalla de 375px de ancho el contenido útil pasa de ~215px a ~303px (`.form-wrap` 16px×2 + `.form-card` 20px×2 = 72px de padding total, contra los 160px actuales). El `text-align:left` en `.intro-section` evita el espaciado irregular que genera `justify` en columnas angostas. `.logo img{height:56px}` y el padding reducido de `.header`/`.hero` adelantan la propuesta de valor dentro del viewport visible sin scroll.

- [ ] **Step 3: Agregar la clase base `.step-label-mobile` (oculta por defecto, fuera del media query)**

Ubicar la regla `.step-label{...}` dentro del `<style>` y agregar justo después:

```css
.step-label{position:absolute;top:40px;left:50%;transform:translateX(-50%);font-size:11px;white-space:nowrap;color:var(--text3);font-weight:500}
.step-circle.active .step-label{color:var(--verde-accent)}
.step-circle.done .step-label{color:var(--verde)}
.step-label-mobile{display:none;text-align:center;font-size:12px;color:var(--text3);font-weight:600;margin-top:10px}
.progress-wrap{position:relative;margin-bottom:20px}
```

(Los `step-label` individuales — "Tamaño", "Contacto", "Capital", etc. — se ocultan en `<600px` porque con 5 círculos comprimidos en ~300px de ancho corren riesgo de superponerse o cortarse contra el borde. Se reemplazan por un único indicador de texto "Paso X de Y" debajo de la barra, que no depende del ancho disponible.)

- [ ] **Step 4: Agregar el elemento `#step-label-mobile` al markup de la progress bar**

```html
  <!-- PROGRESS BAR -->
  <div class="progress-wrap">
    <div class="progress" id="progress-bar">
      <div class="step-circle active" id="step-1"><span>1</span><span class="step-label">Tamaño</span></div>
      <div class="step-line" id="line-1-2"></div>
      <div class="step-circle pending" id="step-2"><span>2</span><span class="step-label">Contacto</span></div>
      <div class="step-line" id="line-2-3"></div>
      <div class="step-circle pending" id="step-3"><span>3</span><span class="step-label">Capital</span></div>
      <div class="step-line" id="line-3-4"></div>
      <div class="step-circle pending" id="step-4"><span>4</span><span class="step-label">FOGAPE</span></div>
      <div class="step-line" id="line-4-5"></div>
      <div class="step-circle pending" id="step-5"><span>5</span><span class="step-label">Requisitos</span></div>
    </div>
    <div class="step-label-mobile" id="step-label-mobile"></div>
  </div>
```

- [ ] **Step 5: Actualizar el texto de `#step-label-mobile` en `updateProgress()`**

```javascript
  // Clarity: etiquetar el paso actual para segmentar sesiones por avance
  const nombresPasos = ['Tamaño','Contacto','Capital','FOGAPE','Requisitos'];
  if(window.clarity) clarity('set','paso_encuesta', nombresPasos[currentBloque]);
  // Indicador de paso para mobile (los step-label individuales se ocultan en <600px)
  const stepLabelMobile = document.getElementById('step-label-mobile');
  if(stepLabelMobile) stepLabelMobile.textContent = `Paso ${currentBloque+1} de ${totalBloques} — ${nombresPasos[currentBloque]}`;
```

(Esto va inmediatamente después de la línea `if(window.clarity) clarity(...)` dentro de `updateProgress()`, sin tocar el resto de la función.)

- [ ] **Step 6: Verificación manual**

```bash
python3 -m http.server 8000 --directory public
```

En Chrome DevTools → toggle device toolbar, elegir un preset Android (ej. "Galaxy S8+", 360×740) o setear un ancho custom de 360px, y verificar:
- El logo se ve más chico, el hero y la propuesta de valor quedan visibles con menos scroll que antes.
- El párrafo de la propuesta de valor (`.intro-section`) está alineado a la izquierda, sin espaciados irregulares entre palabras.
- Debajo de la progress bar aparece un texto tipo "Paso 1 de 5 — Tamaño" (no los 5 labels individuales "Tamaño/Contacto/Capital/FOGAPE/Requisitos" apretados bajo los círculos).
- Ese texto cambia correctamente al avanzar/retroceder entre pasos (probar con los botones "Continuar" y "← Anterior").
- Las opciones de radio de preguntas largas (Q1, Q4, Q9) ya no wrappean en tantas líneas — hay más ancho disponible dentro del `.form-card`.
- Tocar el campo de correo/RUT en un emulador o dispositivo iOS (si hay alguno a mano) y confirmar que no hay salto de zoom al enfocar. En Android no debería notarse cambio en el comportamiento de foco (ya funcionaba bien).
- Repetir la revisión en un ancho de escritorio (>600px, ej. 1280px) y confirmar que no cambió nada visualmente respecto al comportamiento actual — todos los cambios de este Task están dentro del `@media(max-width:600px)` o son cambios neutros (el `font-size:16px` de los inputs).

- [ ] **Step 7: Commit**

```bash
git add public/index.html
git commit -m "mobile CSS fixes: reduce padding stacking, step indicator, touch targets"
```

---

## Fuera de alcance (no incluido en esta spec)

Discutido en el análisis de CRO pero no forma parte de esta implementación — requieren información o decisiones adicionales:

- **Message match anuncio ↔ headline:** falta el copy real del anuncio de Meta Ads para auditar si el headline de la landing coincide con la promesa del ad.
- **Instrumentación de drop-off real (focus/blur por campo):** agregar eventos de Clarity/Meta Pixel en el primer `focus` de cada campo del paso de Contacto, para medir abandono campo por campo en vez de solo por paso completado.
- **A/B testing formal:** esta spec implementa directamente los cambios recomendados (no hay herramienta de A/B testing instalada en el sitio); si se quiere testear antes de aplicar a todo el tráfico, es una spec aparte.
