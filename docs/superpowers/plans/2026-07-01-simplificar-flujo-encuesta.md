# Simplificar Flujo de Encuesta — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the conditional CLP/UF-modality question flow in the Estándar and FOGAPE blocks of `public/index.html` with two fixed, always-visible rate questions per block (CLP and UF) plus the existing monto máximo question, per `docs/superpowers/specs/2026-07-01-simplificar-flujo-encuesta-design.md`.

**Architecture:** Single-file static HTML/CSS/JS (`public/index.html`, no build step). Work is a direct edit of the markup inside `#bloque-2` (Estándar) and `#bloque-3` (FOGAPE), a relabel inside `#bloque-4` (Requisitos), and removal of the now-dead conditional-branching JS (`handleConditional`, `show`/`hide`/`clearRadios`, `skipFogape`) plus updates to every place that reads/writes the old field names (`validateBloque`, `submitForm`, `savePartial`, `getFormSnapshot`, `restoreLocalDraft`, `resetFormExceptRut`).

**Tech Stack:** Vanilla HTML/CSS/JS, no framework, no build tooling, no test runner. Verification is done by (a) `grep` structural checks against `public/index.html` and (b) manually loading the page via `python3 -m http.server 8000 --directory public` and exercising the form in a browser, per `CLAUDE.local.md`.

## Global Constraints

- Reuse existing numeric rate/amount option text and ordering verbatim — no changes to the brackets themselves (spec "Fuera de alcance").
- New radio `name` attributes: `q_std_clp`, `q_std_uf`, `q_std_monto`, `q_fogape_clp`, `q_fogape_uf`, `q_fogape_monto`. Requisitos keeps `q9`/`q10` unchanged (spec "Payload / integración").
- Do not touch Contacto (`bloque-1`) or Tamaño (`bloque-0`, `q0`) markup or logic.
- Do not touch the Google Apps Script or the Google Sheet — only the payload shape sent from the client.
- Do not add new CSS or restructure `.subpregunta` styles — they become unused but are left in place (spec "Fuera de alcance").
- No `alert()` wording changes beyond what's needed to reference the new question numbers.

---

### Task 1: Replace Bloque 1 Estándar markup with fixed CLP/UF/monto questions

**Files:**
- Modify: `public/index.html:233-313` (the `#bloque-2` div, from `<!-- BLOQUE 2: CAPITAL DE TRABAJO -->` through its closing `</div>`)

**Interfaces:**
- Produces: three always-visible `.pregunta` blocks with radio groups `name="q_std_clp"`, `name="q_std_uf"`, `name="q_std_monto"` inside `#bloque-2`. Task 5 (`validateBloque`) and Task 6 (`submitForm`/`savePartial`/`getFormSnapshot`) read these names.

- [ ] **Step 1: Confirm current markup matches expectations**

Run:
```bash
grep -n 'name="q1"\|name="q2"\|name="q3"\|name="q3a"\|name="q3b"\|name="q4"' public/index.html
```
Expected: hits at lines 240-242 (`q1`), 250-256 and 262-267 (`q2`), 275-277 (`q3`), 283-286 (`q3a`), 293-296 (`q3b`), 305-310 (`q4`) — confirming the old conditional block is still in place before editing.

- [ ] **Step 2: Replace the `#bloque-2` div contents**

Use the Edit tool to replace the entire block from `<!-- BLOQUE 2: CAPITAL DE TRABAJO -->` (line 232) through the `</div>` that closes `#bloque-2` (line 313) with:

```html
  <!-- BLOQUE 2: CAPITAL DE TRABAJO -->
  <div class="bloque" id="bloque-2">
    <span class="bloque-tag">Bloque 1</span>
    <div class="bloque-title">Préstamo de Capital de Trabajo Estándar (Sin Garantía)</div>

    <div class="pregunta" id="pq-q-std-clp">
      <div class="pregunta-label">Q1. Seleccione el rango de tasa anual obtenido en su última operación principal en <strong>Pesos (CLP)</strong>:</div>
      <div class="opciones">
        <label class="opcion"><input type="radio" name="q_std_clp"> <span class="opcion-text">A menos de 8,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_std_clp"> <span class="opcion-text">Entre 8,00% y 9,50% anual</span></label>
        <label class="opcion"><input type="radio" name="q_std_clp"> <span class="opcion-text">Entre 9,51% y 11,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_std_clp"> <span class="opcion-text">Entre 11,01% y 12,50% anual</span></label>
        <label class="opcion"><input type="radio" name="q_std_clp"> <span class="opcion-text">Entre 12,51% y 14,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_std_clp"> <span class="opcion-text">Entre 14,01% y 15,50% anual</span></label>
        <label class="opcion"><input type="radio" name="q_std_clp"> <span class="opcion-text">Más de 15,51% anual</span></label>
        <label class="opcion"><input type="radio" name="q_std_clp"> <span class="opcion-text">No aplica / no usa este tipo de financiamiento</span></label>
      </div>
    </div>

    <div class="pregunta" id="pq-q-std-uf">
      <div class="pregunta-label">Q2. Seleccione el rango de tasa anual obtenido en su última operación principal en <strong>UF (Tasa Real)</strong>:</div>
      <div class="opciones">
        <label class="opcion"><input type="radio" name="q_std_uf"> <span class="opcion-text">A menos de UF + 5,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_std_uf"> <span class="opcion-text">Entre UF + 5,01% y UF + 6,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_std_uf"> <span class="opcion-text">Entre UF + 6,01% y UF + 7,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_std_uf"> <span class="opcion-text">Entre UF + 7,01% y UF + 8,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_std_uf"> <span class="opcion-text">Entre UF + 8,01% y UF + 9,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_std_uf"> <span class="opcion-text">Más de UF + 9,01% anual</span></label>
        <label class="opcion"><input type="radio" name="q_std_uf"> <span class="opcion-text">No aplica / no usa este tipo de financiamiento</span></label>
      </div>
    </div>

    <div class="pregunta" id="pq-q-std-monto">
      <div class="pregunta-label">Q3. ¿Cuál es el monto máximo que le ofrecen los bancos comerciales en préstamos estándar sin garantía estatal para plazos de hasta 12 meses?</div>
      <div class="opciones">
        <label class="opcion"><input type="radio" name="q_std_monto"> <span class="opcion-text">Hasta UF 2.500 (Aprox. $100M CLP / $110.000 USD)</span></label>
        <label class="opcion"><input type="radio" name="q_std_monto"> <span class="opcion-text">Entre UF 2.501 y UF 5.000 (Aprox. $100M a $200M CLP / $220.000 USD)</span></label>
        <label class="opcion"><input type="radio" name="q_std_monto"> <span class="opcion-text">Entre UF 5.001 y UF 10.000 (Aprox. $200M a $400M CLP / $440.000 USD)</span></label>
        <label class="opcion"><input type="radio" name="q_std_monto"> <span class="opcion-text">Entre UF 10.001 y UF 25.000 (Aprox. $400M a $1.000M CLP / $1,1 Millones USD)</span></label>
        <label class="opcion"><input type="radio" name="q_std_monto"> <span class="opcion-text">Entre UF 25.011 y UF 50.000 (Aprox. $1.000M a $2.000M CLP / $2,2 Millones USD)</span></label>
        <label class="opcion"><input type="radio" name="q_std_monto"> <span class="opcion-text">Más de UF 50.001 (Más de $2.000M CLP / $2,2 Millones USD)</span></label>
      </div>
    </div>
  </div>
```

- [ ] **Step 3: Verify the replacement**

Run:
```bash
grep -n 'name="q1"\|name="q3a"\|name="q3b"' public/index.html
grep -c 'name="q_std_clp"' public/index.html
grep -c 'name="q_std_uf"' public/index.html
grep -c 'name="q_std_monto"' public/index.html
```
Expected: first command returns no matches (old filter/complementary fields gone from this block — `q3` and `q4` may still appear elsewhere until Task 4/5 touch the JS, that's fine). `q_std_clp` count is 8, `q_std_uf` count is 7, `q_std_monto` count is 6.

- [ ] **Step 4: Commit**

```bash
git add public/index.html
git commit -m "feat: replace conditional Estándar rate questions with fixed CLP/UF questions"
```

---

### Task 2: Replace Bloque 2 FOGAPE markup with fixed CLP/UF/monto questions

**Files:**
- Modify: `public/index.html` (the `#bloque-3` div, `<!-- BLOQUE 3: FOGAPE -->` through its closing `</div>` — line numbers shifted after Task 1, re-locate with grep)

**Interfaces:**
- Consumes: none from Task 1 (independent block).
- Produces: three always-visible `.pregunta` blocks with radio groups `name="q_fogape_clp"`, `name="q_fogape_uf"`, `name="q_fogape_monto"` inside `#bloque-3`.

- [ ] **Step 1: Locate the current block**

Run:
```bash
grep -n '<!-- BLOQUE 3: FOGAPE -->\|id="bloque-3"\|id="bloque-4"' public/index.html
```
Expected: three line numbers — the FOGAPE comment, the `#bloque-3` opening div, and the `#bloque-4` opening div (which marks where `#bloque-3` ends). Use these to find the exact end of the block (the `</div>` immediately before the `#bloque-4` comment/div).

- [ ] **Step 2: Replace the `#bloque-3` div contents**

Use the Edit tool to replace the entire block from `<!-- BLOQUE 3: FOGAPE -->` through its closing `</div>` with:

```html
  <!-- BLOQUE 3: FOGAPE -->
  <div class="bloque" id="bloque-3">
    <span class="bloque-tag">Bloque 2</span>
    <div class="bloque-title">Préstamos con Garantía Estatal (FOGAPE)</div>

    <div class="pregunta" id="pq-q-fogape-clp">
      <div class="pregunta-label">Q4. Seleccione el rango de tasa anual de su última colocación FOGAPE en <strong>Pesos (CLP)</strong>:</div>
      <div class="opciones">
        <label class="opcion"><input type="radio" name="q_fogape_clp"> <span class="opcion-text">A menos de 8,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_clp"> <span class="opcion-text">Entre 8,00% y 9,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_clp"> <span class="opcion-text">Entre 9,01% y 10,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_clp"> <span class="opcion-text">Entre 10,01% y 11,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_clp"> <span class="opcion-text">Entre 11,01% y 12,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_clp"> <span class="opcion-text">Más de 12,01% anual</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_clp"> <span class="opcion-text">No aplica / no usa este tipo de financiamiento</span></label>
      </div>
    </div>

    <div class="pregunta" id="pq-q-fogape-uf">
      <div class="pregunta-label">Q5. Seleccione el rango de tasa anual de su última colocación FOGAPE en <strong>UF (Tasa Real)</strong>:</div>
      <div class="opciones">
        <label class="opcion"><input type="radio" name="q_fogape_uf"> <span class="opcion-text">A menos de UF + 4,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_uf"> <span class="opcion-text">Entre UF + 4,00% y UF + 4,50% anual</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_uf"> <span class="opcion-text">Entre UF + 4,51% y UF + 5,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_uf"> <span class="opcion-text">Entre UF + 5,01% y UF + 5,50% anual</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_uf"> <span class="opcion-text">Entre UF + 5,51% y UF + 6,00% anual</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_uf"> <span class="opcion-text">Más de UF + 6,01% anual</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_uf"> <span class="opcion-text">No aplica / no usa este tipo de financiamiento</span></label>
      </div>
    </div>

    <div class="pregunta" id="pq-q-fogape-monto">
      <div class="pregunta-label">Q6. ¿Cuál es el monto máximo que le ofrecen los bancos comerciales en préstamos con garantía FOGAPE para plazos de hasta 12 meses?</div>
      <div class="opciones">
        <label class="opcion"><input type="radio" name="q_fogape_monto"> <span class="opcion-text">Hasta UF 2.500</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_monto"> <span class="opcion-text">Entre UF 2.501 y UF 5.000</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_monto"> <span class="opcion-text">Entre UF 5.001 y UF 10.000</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_monto"> <span class="opcion-text">Entre UF 10.001 y UF 15.000</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_monto"> <span class="opcion-text">Entre UF 15.001 y UF 25.000</span></label>
        <label class="opcion"><input type="radio" name="q_fogape_monto"> <span class="opcion-text">No nos ofrecen cupo FOGAPE a pesar de cumplir con los requisitos de tamaño</span></label>
      </div>
    </div>
  </div>
```

- [ ] **Step 3: Verify the replacement**

Run:
```bash
grep -n 'name="q5"\|name="q7a"\|name="q7b"' public/index.html
grep -c 'name="q_fogape_clp"' public/index.html
grep -c 'name="q_fogape_uf"' public/index.html
grep -c 'name="q_fogape_monto"' public/index.html
```
Expected: first command returns no matches inside this block (any remaining `q5`/`q7a`/`q7b` hits at this point would only be in the still-unedited JS, addressed in Task 4). `q_fogape_clp` count is 7, `q_fogape_uf` count is 7, `q_fogape_monto` count is 6.

- [ ] **Step 4: Commit**

```bash
git add public/index.html
git commit -m "feat: replace conditional FOGAPE rate questions with fixed CLP/UF questions"
```

---

### Task 3: Renumber Requisitos question labels

**Files:**
- Modify: `public/index.html` (the `#bloque-4` div — `pq-q9` and `pq-q10` `.pregunta-label` text only)

**Interfaces:**
- Consumes: none.
- Produces: no change to `name="q9"`/`name="q10"` or any JS — only the visible label text changes from `Q9.`/`Q10.` to `Q7.`/`Q8.`.

- [ ] **Step 1: Locate current labels**

Run:
```bash
grep -n 'pregunta-label">Q9\.\|pregunta-label">Q10\.' public/index.html
```
Expected: two matches, one for each label.

- [ ] **Step 2: Edit the labels**

Use the Edit tool:

Old:
```html
      <div class="pregunta-label">Q9. ¿Cuál es la principal exigencia de los bancos para otorgar un crédito SIN garantía estatal?</div>
```
New:
```html
      <div class="pregunta-label">Q7. ¿Cuál es la principal exigencia de los bancos para otorgar un crédito SIN garantía estatal?</div>
```

Old:
```html
      <div class="pregunta-label">Q10. ¿Cuál es la principal exigencia de los bancos para otorgar un crédito CON garantía estatal (FOGAPE)?</div>
```
New:
```html
      <div class="pregunta-label">Q8. ¿Cuál es la principal exigencia de los bancos para otorgar un crédito CON garantía estatal (FOGAPE)?</div>
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -n 'pregunta-label">Q7\.\|pregunta-label">Q8\.\|pregunta-label">Q9\.\|pregunta-label">Q10\.' public/index.html
```
Expected: matches for `Q7.` and `Q8.` only, no `Q9.`/`Q10.` remaining.

- [ ] **Step 4: Commit**

```bash
git add public/index.html
git commit -m "chore: renumber Requisitos question labels to Q7/Q8"
```

---

### Task 4: Remove dead conditional-branching JS

**Files:**
- Modify: `public/index.html` (inside the second `<script>` block, roughly lines 441-497 pre-edit — re-locate with grep since line numbers shift after Tasks 1-3)

**Interfaces:**
- Consumes: none.
- Produces: removes `handleConditional`, `show`, `hide`, `clearRadios`, `skipFogape`, and their call sites. After this task, no code references `handleConditional`, `show(`, `hide(`, `clearRadios(`, or `skipFogape` anywhere in the file. This is a prerequisite for Task 5 and Task 7, which touch code that currently calls these functions.

- [ ] **Step 1: Confirm current call sites**

Run:
```bash
grep -n 'skipFogape\|handleConditional\|^function show\|^function hide\|^function clearRadios\|show(\|hide(\|clearRadios(' public/index.html
```
Expected: the declaration `let skipFogape = false;`, the `handleConditional(name, this.value);` call inside the radio `change` listener, the `handleConditional` function definition itself, the `show`/`hide`/`clearRadios` function definitions, and a `handleConditional(name, value);` call inside `restoreLocalDraft`.

- [ ] **Step 2: Remove `skipFogape` and simplify the radio change listener**

Old:
```javascript
const LOCAL_STORAGE_KEY = 'encuestaMPChile_borrador';
let currentBloque = 0;
const totalBloques = 5; // 0-4
let skipFogape = false;

// Estilo visual opciones al seleccionar
document.querySelectorAll('.opcion input[type=radio]').forEach(radio => {
  radio.addEventListener('change', function(){
    const name = this.name;
    document.querySelectorAll(`input[name="${name}"]`).forEach(r => {
      r.closest('.opcion').classList.remove('selected');
    });
    this.closest('.opcion').classList.add('selected');
    handleConditional(name, this.value);
  });
});
```
New:
```javascript
const LOCAL_STORAGE_KEY = 'encuestaMPChile_borrador';
let currentBloque = 0;
const totalBloques = 5; // 0-4

// Estilo visual opciones al seleccionar
document.querySelectorAll('.opcion input[type=radio]').forEach(radio => {
  radio.addEventListener('change', function(){
    const name = this.name;
    document.querySelectorAll(`input[name="${name}"]`).forEach(r => {
      r.closest('.opcion').classList.remove('selected');
    });
    this.closest('.opcion').classList.add('selected');
  });
});
```

- [ ] **Step 3: Delete `handleConditional`, `show`, `hide`, `clearRadios`**

Old:
```javascript
function handleConditional(name, value){
  if(name === 'q1'){
    // Mostrar Q2 según modalidad
    hide('sq-q2-clp'); hide('sq-q2-uf');
    hide('pq-q3'); hide('pq-q4');
    hide('sq-q3a'); hide('sq-q3b');
    clearRadios('q2'); clearRadios('q3'); clearRadios('q3a'); clearRadios('q3b');
    if(value === 'clp'){ show('sq-q2-clp'); show('pq-q3'); show('pq-q4'); }
    if(value === 'uf'){ show('sq-q2-uf'); show('pq-q3'); show('pq-q4'); }
    if(value === 'no'){ skipFogape = false; } // no salta, continúa a bloque FOGAPE
  }
  if(name === 'q3'){
    hide('sq-q3a'); hide('sq-q3b');
    if(value === 'clp-uf') show('sq-q3a');
    if(value === 'uf-clp') show('sq-q3b');
  }
  if(name === 'q5'){
    hide('sq-q6-clp'); hide('sq-q6-uf');
    hide('pq-q7'); hide('pq-q8');
    hide('sq-q7a'); hide('sq-q7b');
    clearRadios('q6'); clearRadios('q7'); clearRadios('q7a'); clearRadios('q7b');
    if(value === 'clp'){ show('sq-q6-clp'); show('pq-q7'); show('pq-q8'); }
    if(value === 'uf'){ show('sq-q6-uf'); show('pq-q7'); show('pq-q8'); }
    if(value === 'no'){ skipFogape = true; }
  }
  if(name === 'q7'){
    hide('sq-q7a'); hide('sq-q7b');
    if(value === 'clp-uf') show('sq-q7a');
    if(value === 'uf-clp') show('sq-q7b');
  }
}

function show(id){ const el=document.getElementById(id); if(el){ el.style.display='block'; el.classList.add('visible'); }}
function hide(id){ const el=document.getElementById(id); if(el){ el.style.display='none'; el.classList.remove('visible'); }}
function clearRadios(name){ document.querySelectorAll(`input[name="${name}"]`).forEach(r=>{ r.checked=false; r.closest('.opcion').classList.remove('selected'); }); }

function validateBloque(b){
```
New:
```javascript
function validateBloque(b){
```

- [ ] **Step 4: Verify removal**

Run:
```bash
grep -n 'skipFogape\|handleConditional\|function show(\|function hide(\|function clearRadios(' public/index.html
```
Expected: no matches. (The `restoreLocalDraft` call site is removed in Task 7; if this grep still shows a `handleConditional(name, value);` line at this point, that's expected until Task 7 runs — re-run this check again after Task 7.)

- [ ] **Step 5: Commit**

```bash
git add public/index.html
git commit -m "refactor: remove dead conditional-branching JS (handleConditional, show/hide/clearRadios)"
```

---

### Task 5: Simplify `validateBloque` for the Estándar and FOGAPE blocks

**Files:**
- Modify: `public/index.html` (the `validateBloque` function)

**Interfaces:**
- Consumes: radio `name` attributes `q_std_clp`, `q_std_uf`, `q_std_monto`, `q_fogape_clp`, `q_fogape_uf`, `q_fogape_monto` produced by Task 1 and Task 2.
- Produces: `validateBloque(b)` returns `false` and shows an `alert()` if any of the 3 required radios in bloque `2` or `3` is unanswered; otherwise returns `true`. Task 6's `goNext()`/`submitForm()` call site is unchanged (still calls `validateBloque(currentBloque)`).

- [ ] **Step 1: Confirm current validation logic**

Run:
```bash
grep -n "if(b === 2)\|if(b === 3)" public/index.html
```
Expected: two matches, marking the start of the blocks to replace.

- [ ] **Step 2: Replace the `b === 2` and `b === 3` branches**

Old:
```javascript
  if(b === 2){
    const q1 = document.querySelector('input[name="q1"]:checked');
    if(!q1){ alert('Por favor responda Q1.'); return false; }
    if(q1.value !== 'no'){
      if(!document.querySelector('input[name="q2"]:checked')){ alert('Por favor responda Q2.'); return false; }
      if(!document.querySelector('input[name="q3"]:checked')){ alert('Por favor responda Q3.'); return false; }
      const q3 = document.querySelector('input[name="q3"]:checked').value;
      if(q3 === 'clp-uf' && !document.querySelector('input[name="q3a"]:checked')){ alert('Por favor responda Q3.A.'); return false; }
      if(q3 === 'uf-clp' && !document.querySelector('input[name="q3b"]:checked')){ alert('Por favor responda Q3.B.'); return false; }
      if(!document.querySelector('input[name="q4"]:checked')){ alert('Por favor responda Q4.'); return false; }
    }
    return true;
  }
  if(b === 3){
    const q5 = document.querySelector('input[name="q5"]:checked');
    if(!q5){ alert('Por favor responda Q5.'); return false; }
    if(q5.value !== 'no'){
      if(!document.querySelector('input[name="q6"]:checked')){ alert('Por favor responda Q6.'); return false; }
      if(!document.querySelector('input[name="q7"]:checked')){ alert('Por favor responda Q7.'); return false; }
      const q7 = document.querySelector('input[name="q7"]:checked').value;
      if(q7 === 'clp-uf' && !document.querySelector('input[name="q7a"]:checked')){ alert('Por favor responda Q7.A.'); return false; }
      if(q7 === 'uf-clp' && !document.querySelector('input[name="q7b"]:checked')){ alert('Por favor responda Q7.B.'); return false; }
      if(!document.querySelector('input[name="q8"]:checked')){ alert('Por favor responda Q8.'); return false; }
    }
    return true;
  }
```
New:
```javascript
  if(b === 2){
    if(!document.querySelector('input[name="q_std_clp"]:checked')){ alert('Por favor responda Q1.'); return false; }
    if(!document.querySelector('input[name="q_std_uf"]:checked')){ alert('Por favor responda Q2.'); return false; }
    if(!document.querySelector('input[name="q_std_monto"]:checked')){ alert('Por favor responda Q3.'); return false; }
    return true;
  }
  if(b === 3){
    if(!document.querySelector('input[name="q_fogape_clp"]:checked')){ alert('Por favor responda Q4.'); return false; }
    if(!document.querySelector('input[name="q_fogape_uf"]:checked')){ alert('Por favor responda Q5.'); return false; }
    if(!document.querySelector('input[name="q_fogape_monto"]:checked')){ alert('Por favor responda Q6.'); return false; }
    return true;
  }
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -n 'name="q1"]\|name="q3a"]\|name="q3b"]\|name="q5"]\|name="q7a"]\|name="q7b"]' public/index.html
```
Expected: no matches (these old query selectors are gone from `validateBloque`; Task 6 removes the remaining references in the payload builders).

- [ ] **Step 4: Manual check — required-field alerts fire correctly**

Serve the site and click through:
```bash
python3 -m http.server 8000 --directory public
```
Open `http://localhost:8000` in a browser, get to Bloque 1 (Estándar), click "Continuar →" without answering anything. Expected: alert "Por favor responda Q1." Answer Q1 only, click again. Expected: alert "Por favor responda Q2." Answer Q1 and Q2, click again. Expected: alert "Por favor responda Q3." Answer all three, click again. Expected: advances to the FOGAPE block. Repeat the same sequence for FOGAPE (Q4/Q5/Q6 alerts). Stop the server with Ctrl+C when done.

- [ ] **Step 5: Commit**

```bash
git add public/index.html
git commit -m "feat: simplify validateBloque for fixed Estándar/FOGAPE questions"
```

---

### Task 6: Update payload builders (`submitForm`, `savePartial`, `getFormSnapshot`)

**Files:**
- Modify: `public/index.html` (the `submitForm`, `savePartial`, and `getFormSnapshot` functions)

**Interfaces:**
- Consumes: radio `name` attributes from Task 1/Task 2, and the existing `getRadio(name)` / `getRadioValue(name)` helper functions (unchanged, defined elsewhere in the file).
- Produces: the JSON payload sent to Google Apps Script now has keys `q0, q_std_clp, q_std_uf, q_std_monto, q_fogape_clp, q_fogape_uf, q_fogape_monto, q9, q10` (plus the existing `estado`, `mailEmpresa`, `rut`, `mailContacto`, `telefono`), matching the spec's payload table. `getFormSnapshot().radios` uses the same key set, consumed by Task 7's `restoreLocalDraft`.

- [ ] **Step 1: Confirm current payload shape**

Run:
```bash
grep -n "q3a: getRadio\|q3a: getRadioValue" public/index.html
```
Expected: two matches — one inside `submitForm`'s payload, one inside `savePartial`'s payload (note `getFormSnapshot` uses `getRadioValue`, not `getRadio` — check both to distinguish).

- [ ] **Step 2: Edit `submitForm`'s payload**

Old:
```javascript
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

  // Enviar a Google Sheets via Apps Script
  const btn = document.getElementById('btn-next');
```
New:
```javascript
    q0: getRadio('q0'),
    q_std_clp: getRadio('q_std_clp'),
    q_std_uf: getRadio('q_std_uf'),
    q_std_monto: getRadio('q_std_monto'),
    q_fogape_clp: getRadio('q_fogape_clp'),
    q_fogape_uf: getRadio('q_fogape_uf'),
    q_fogape_monto: getRadio('q_fogape_monto'),
    q9: getRadio('q9'),
    q10: getRadio('q10')
  };

  // Enviar a Google Sheets via Apps Script
  const btn = document.getElementById('btn-next');
```

- [ ] **Step 3: Edit `getFormSnapshot`'s `radios` object**

Old:
```javascript
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
New:
```javascript
    radios: {
      q0: getRadioValue('q0'),
      q_std_clp: getRadioValue('q_std_clp'),
      q_std_uf: getRadioValue('q_std_uf'),
      q_std_monto: getRadioValue('q_std_monto'),
      q_fogape_clp: getRadioValue('q_fogape_clp'),
      q_fogape_uf: getRadioValue('q_fogape_uf'),
      q_fogape_monto: getRadioValue('q_fogape_monto'),
      q9: getRadioValue('q9'),
      q10: getRadioValue('q10')
    }
  };
}
```

- [ ] **Step 4: Edit `savePartial`'s payload**

Old:
```javascript
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

  const saveBtn = document.getElementById('btn-save-partial');
```
New:
```javascript
    q0: getRadio('q0'),
    q_std_clp: getRadio('q_std_clp'),
    q_std_uf: getRadio('q_std_uf'),
    q_std_monto: getRadio('q_std_monto'),
    q_fogape_clp: getRadio('q_fogape_clp'),
    q_fogape_uf: getRadio('q_fogape_uf'),
    q_fogape_monto: getRadio('q_fogape_monto'),
    q9: getRadio('q9'),
    q10: getRadio('q10')
  };

  const saveBtn = document.getElementById('btn-save-partial');
```

- [ ] **Step 5: Verify**

Run:
```bash
grep -n "q3a\|q3b\|q7a\|q7b" public/index.html
grep -c "q_std_clp\|q_std_uf\|q_std_monto\|q_fogape_clp\|q_fogape_uf\|q_fogape_monto" public/index.html
```
Expected: first command returns no matches anywhere in the file. Second command returns a count ≥ 24 (6 fields × at least 4 occurrences each: markup `name=`, `submitForm`, `savePartial`, `getFormSnapshot`, plus `validateBloque` from Task 5 — exact count isn't critical, just confirm it's non-zero and the old names are gone).

- [ ] **Step 6: Manual check — submitted payload**

Serve the site, open browser devtools → Network tab, fill out the form through Bloque 2 (Requisitos) and submit. Expected: the request to the Apps Script URL has a JSON body containing `q_std_clp`, `q_std_uf`, `q_std_monto`, `q_fogape_clp`, `q_fogape_uf`, `q_fogape_monto`, `q9`, `q10` with the selected option text as values, and no `q1`, `q3`, `q3a`, `q3b`, `q5`, `q7`, `q7a`, `q7b` keys.

- [ ] **Step 7: Commit**

```bash
git add public/index.html
git commit -m "feat: update submitted payload keys to match simplified question structure"
```

---

### Task 7: Fix `restoreLocalDraft` and `resetFormExceptRut` for the new field names

**Files:**
- Modify: `public/index.html` (the `resetFormExceptRut` and `restoreLocalDraft` functions)

**Interfaces:**
- Consumes: `getFormSnapshot().radios` key set from Task 6 (now `q_std_clp`, etc.), no more `handleConditional` (removed in Task 4).
- Produces: `resetFormExceptRut` no longer references now-nonexistent subpregunta ids. `restoreLocalDraft` no longer calls `handleConditional`.

- [ ] **Step 1: Confirm current code**

Run:
```bash
grep -n "Oculta subpreguntas condicionales\|handleConditional(name, value)" public/index.html
```
Expected: two matches — the comment above the `hide(...)` list in `resetFormExceptRut`, and the `handleConditional(name, value);` call inside `restoreLocalDraft`.

- [ ] **Step 2: Remove the dead subpregunta-hiding list from `resetFormExceptRut`**

Old:
```javascript
  document.querySelectorAll('.opciones input[type=radio]').forEach(input => {
    input.checked = false;
    const opt = input.closest('.opcion');
    if(opt) opt.classList.remove('selected');
  });

  // Oculta subpreguntas condicionales que pudieran haber quedado visibles
  ['sq-q2-clp','sq-q2-uf','pq-q3','pq-q4','sq-q3a','sq-q3b',
   'sq-q6-clp','sq-q6-uf','pq-q7','pq-q8','sq-q7a','sq-q7b'].forEach(hide);

  if(currentBloque !== 0){
```
New:
```javascript
  document.querySelectorAll('.opciones input[type=radio]').forEach(input => {
    input.checked = false;
    const opt = input.closest('.opcion');
    if(opt) opt.classList.remove('selected');
  });

  if(currentBloque !== 0){
```

- [ ] **Step 3: Remove the `handleConditional` call in `restoreLocalDraft`**

Old:
```javascript
      const input = document.querySelector(`input[name="${name}"][value="${CSS.escape(value)}"]`);
      if(input){
        input.checked = true;
        input.closest('.opcion').classList.add('selected');
        handleConditional(name, value);
      }
```
New:
```javascript
      const input = document.querySelector(`input[name="${name}"][value="${CSS.escape(value)}"]`);
      if(input){
        input.checked = true;
        input.closest('.opcion').classList.add('selected');
      }
```

- [ ] **Step 4: Verify no remaining references**

Run:
```bash
grep -n "handleConditional\|sq-q2-clp\|sq-q2-uf\|sq-q6-clp\|sq-q6-uf\|sq-q3a\|sq-q3b\|sq-q7a\|sq-q7b\|pq-q3\|pq-q4\|pq-q7\|pq-q8" public/index.html
```
Expected: no matches anywhere in the file.

- [ ] **Step 5: Manual check — RUT-change reset still works**

Serve the site, go through Bloque 1 (Contacto), enter a RUT, advance and answer some questions in the Estándar block, then go back to Bloque 1 and change the RUT to a different value, then blur the field (click elsewhere). Expected: the form resets (email/telefono/radios cleared, back to Bloque 0) same as before this change — confirms `resetFormExceptRut` still works without the removed `hide` list.

- [ ] **Step 6: Commit**

```bash
git add public/index.html
git commit -m "fix: remove dead handleConditional/subpregunta references from draft restore and reset"
```

---

### Task 8: Full end-to-end manual verification

**Files:**
- None (verification only).

**Interfaces:**
- Consumes: the complete, edited `public/index.html` from Tasks 1-7.
- Produces: confidence that the full flow works before considering the plan done.

- [ ] **Step 1: Static sanity check — no leftover old field names anywhere**

Run:
```bash
grep -n 'name="q1"\|name="q3"\|name="q3a"\|name="q3b"\|name="q4"\|name="q5"\|name="q7"\|name="q7a"\|name="q7b"\|name="q8"' public/index.html
```
Expected: no matches (note `name="q10"` will legitimately match a `q1` substring search if you use a looser pattern — the patterns above are anchored with the closing quote to avoid that false positive; if you see a hit, confirm it isn't actually `q10` before treating it as a failure).

- [ ] **Step 2: Full click-through in a browser**

```bash
python3 -m http.server 8000 --directory public
```
Open `http://localhost:8000`. Walk the entire form start to finish:
1. Tamaño (Q0) — select an option, continue.
2. Contacto — fill email, RUT, continue.
3. Estándar — answer Q1 (CLP), Q2 (UF), Q3 (monto), including trying the new "No aplica" option on Q1 and Q2, continue.
4. FOGAPE — answer Q4, Q5, Q6, continue.
5. Requisitos — confirm labels read "Q7." and "Q8.", answer both, submit.

Expected: success screen appears, no `alert()` popups except when intentionally leaving a required question blank (re-test that from Task 5 Step 4 still holds), no console errors in devtools.

- [ ] **Step 3: Confirm draft autosave/restore across the new fields**

With the server still running, reload the page mid-form (after answering Q1/Q2 in Estándar but before submitting), and confirm the previously-selected options are still shown as selected and the page resumes on the same block. Stop the server with Ctrl+C.

- [ ] **Step 4: Final commit (if Step 2/3 surfaced any fixes)**

If Steps 1-3 required any code changes, stage and commit them with a descriptive message. If no changes were needed, this task requires no commit — the plan is complete as of Task 7's commit.
