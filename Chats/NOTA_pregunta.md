# NOTA pregunta
_Exported on 4/21/2026 at 11:25:08 GMT+2 from Cursor (2.6.21)_

---

**User**

1. sigues las reglas de .curor rules ? 

---

**Cursor**

Sí, sigo las reglas de `.cursor/rules/.rules_instrucciones_para_desarrollo_API.mdc` del workspace. Te resumo los puntos clave que aplico siempre en este proyecto:

## Reglas que sigo activamente

**Idioma y comunicación**
- Respondo siempre en español (regla de usuario).
- Respuestas paso a paso, breves, sin teoría innecesaria.
- Indico en qué archivo va cada parte del código.

**Arquitectura del MVP**
- Estructura mínima: `index.html` + `/examenes/` con CSVs + opcional JS/CSS separado.
- Un solo archivo autocontenido cuando sea posible.
- Diseño modular, escalable y desacoplado (UI / lógica / datos separadas).
- Nada de hardcoding de valores críticos → todo configurable.
- Sin backend, sin login, sin BBDD en el MVP.

**UI / UX**
- Paleta NOrlexia: verde `#0d4a45` + dorado `#c9a04a` + crema `#f5f0e8` + fondo `#f0f2f0`.
- Dashboard: Materias Comunes centrada arriba grande + grid 2×2 de especialidades.
- Colores por materia: verde (comunes), ámbar (laboral), rojo (penal), azul (civ

---

**User**

quiero INSERTAr en el cuadro de repuestas de la pregunta un cuadro que permita añadir una nota .Esta nota se guardará asociada a esa pregunta. Cuando se vuelva a hacer ell test y se llegue a esa pregunta se mostrará la nota cuando se responda antes de pasar a la siguiente.
De que maneras se puede hacer que sea lo mas sencillo sin tocar casi nada de programa

---

**Cursor**

Te muestro 3 opciones ordenadas de más sencilla a más completa. Todas son cambios mínimos sobre el `index.html` actual y no tocan nada de la lógica de examen.

---

## Contexto del código actual

La pregunta se pinta en `renderQ()` (líneas 1371–1395 de `index.html`) dentro de `.q-card`:

```938:954:index.html
        <div class="q-card">
          <div class="q-num" id="qNum"></div>
          <div class="q-text" id="qText"></div>
          <div class="options" id="opts"></div>
          <div class="nav-bar">
```

Ahí es donde iría el cuadro de notas. Y el almacenaje debe sobrevivir al cierre del navegador → `localStorage`.

El único "problema" técnico: las preguntas no tienen `id` único. Se resuelve usando como clave un hash del texto de la pregunta (o el propio texto).

---

## Opción A — La más simple (RECOMENDADA) ⭐

**Qué hace:** un `<textarea>` siempre visible bajo las opciones. Se guarda solo al salir del foco. Si la pregunta ya tenía nota de una sesión anterior, aparece precargada; además, al responder se resalta con un banner dorado.

**Cambios:** ~25 líneas. No toca nada del flujo del examen.

- **HTML** (1 línea nueva en `.q-card`, tras `#opts`):

```html
<div id="qNota"></div>
```

- **JS** (nuevo bloque + 3 líneas dentro de `renderQ()`):

```js
const NOTAS_KEY = "norlexia_notas_v1";
const loadNotas = () => JSON.parse(localStorage.getItem(NOTAS_KEY) || "{}");
const saveNota  = (k, v) => {
  const n = loadNotas();
  if (v.trim()) n[k] = v.trim(); else delete n[k];
  localStorage.setItem(NOTAS_KEY, JSON.stringify(n));
};
const keyQ = q => q.pregunta;  // clave simple por texto

function renderNota(q, answered) {
  const prev = loadNotas()[keyQ(q)] || "";
  const aviso = (answered && prev)
    ? `<div class="nota-aviso">📝 Nota previa: ${prev}</div>` : "";
  document.getElementById("qNota").innerHTML = `
    ${aviso}
    <textarea class="nota-ta" placeholder="Escribe una nota para esta pregunta…"
              onblur="saveNota('${keyQ(q).replace(/'/g,"\\'")}', this.value)"
    >${prev}</textarea>`;
}
```

- En `renderQ()`, añade al final:

```js
renderNota(q, answered[cur] !== undefined);
```

- **CSS** (3 reglas pequeñas):

```css
.nota-ta{width:100%;min-height:60px;margin-top:.8rem;padding:.55rem;
  border:1px solid #d9d3c6;border-radius:8px;font:inherit;resize:vertical}
.nota-aviso{background:#fff7e0;border-left:3px solid var(--dorado);
  padding:.5rem .7rem;margin-top:.8rem;border-radius:6px;font-size:.85rem}
```

**Ventajas:** cero dependencias, funciona offline, persistente entre sesiones, responsive.
**Limitación:** si editas el texto de la pregunta en el CSV, se "pierde" la nota (quedaría huérfana). Aceptable para un MVP.

---

## Opción B — Igual que A pero desplegable

Añade un botón "📝 Nota" que muestra/oculta el textarea (UI más limpia, útil en móvil). Suma solo 5 líneas más (un `<button>` con `onclick` que hace `classList.toggle("open")`).

Recomendada si te preocupa que el textarea ocupe espacio visual siempre.

---

## Opción C — Mostrar la nota como "pop-up" al responder

Si quieres que la nota previa **aparezca solo después de contestar** (tal como pides: "se mostrará la nota cuando se responda antes de pasar a la siguiente"):

- Mantienes el textarea para crear/editar la nota.
- Al hacer click en una opción (`answer(l)`), si existe nota previa, se muestra un aviso flotante tipo toast o un banner animado sobre el `.q-card`.
- El botón "Siguiente →" ya existe → el usuario la lee y avanza cuando quiera.

**Cambios extra sobre la opción A:** ~8 líneas en `answer()` para mostrar el banner con animación (`@keyframes slideDown`).

---

## Comparativa rápida

| Opción | Líneas nuevas | UI | Cumple "mostrar al responder" |
|---|---|---|---|
| **A** | ~25 | Textarea fijo + banner si hay nota previa y ya respondida | ✅ (banner aparece al re-renderizar tras responder) |
| **B** | ~30 | Botón plegable | ✅ igual que A |
| **C** | ~35 | Textarea + toast animado tras click | ✅✅ más visual |

---

## Mi recomendación

**Opción A**. Es literalmente añadir una función `renderNota()`, un `div` y una línea en `renderQ()`. Cumple todo lo que pides porque `renderQ()` se vuelve a ejecutar en `answer()` → el banner "📝 Nota previa" aparece automáticamente justo al contestar, antes de pulsar "Siguiente".

Extras opcionales muy baratos que se pueden añadir después sin tocar nada:
- **Exportar/importar notas** como JSON (2 botones en el panel).
- **Contador de notas** en el dashboard (badge "N preguntas con nota").
- **Migrar a hash** (`btoa(q.pregunta).slice(0,16)`) si te preocupa que el texto cambie.

¿Quieres que te pase el parche exacto listo para pegar con la opción A (cuando cambies a modo Agent)?

---

**User**

las notas qeu creee con su texto y demas donde se almacenan

---

**Cursor**

Buena pregunta. Con la propuesta de la Opción A las notas se guardan en **`localStorage` del navegador**. Te explico qué significa eso en concreto, dónde vive el dato y qué limitaciones tiene.

---

## Dónde se guardan exactamente

`localStorage` es una pequeña base de datos **clave-valor** que cada navegador mantiene en tu propio equipo, vinculada al **origen** (protocolo + dominio + puerto, o la ruta del archivo si lo abres como `file://`).

En el código propuesto guardaríamos **una sola clave** con todas las notas en formato JSON:

```js
localStorage.setItem("norlexia_notas_v1", JSON.stringify({
  "¿Qué plazo tiene el trabajador para...?": "Art. 59 ET — 1 año. Ojo con despido: 20 días hábiles.",
  "¿Cuándo procede el recurso de casación?": "Revisar LEC art. 477.",
  "...": "..."
}));
```

Es decir: **un único objeto JSON** cuyas claves son el texto de la pregunta y cuyos valores son tu nota.

---

## Ubicación física en tu PC

Depende del navegador, pero siempre queda en tu perfil de usuario de Windows:

| Navegador | Ruta aproximada |
|---|---|
| **Chrome** | `C:\Users\norik\AppData\Local\Google\Chrome\User Data\Default\Local Storage\leveldb\` |
| **Edge** | `C:\Users\norik\AppData\Local\Microsoft\Edge\User Data\Default\Local Storage\leveldb\` |
| **Firefox** | `C:\Users\norik\AppData\Roaming\Mozilla\Firefox\Profiles\<perfil>\storage\default\` |

Son archivos binarios (LevelDB / SQLite) — **no los edites a mano**. Para verlos y modificarlos con comodidad: abre el navegador en tu `index.html`, pulsa **F12 → pestaña Application (Chrome/Edge) o Storage (Firefox) → Local Storage → la entrada de tu archivo → clave `norlexia_notas_v1`**.

---

## Propiedades importantes

| Aspecto | Comportamiento |
|---|---|
| **Persistencia** | Sobrevive cerrar el navegador, reiniciar el PC y reiniciar la app. |
| **Capacidad** | ~5–10 MB por origen. Sobra de largo para miles de notas de texto. |
| **Ámbito** | Por origen. Si abres el HTML con doble clic (`file://...index.html`) las notas quedan **atadas a esa ruta exacta**. Si mueves el archivo de carpeta, el navegador lo ve como otro origen y **no verá las notas viejas**. |
| **Dispositivo** | Solo local. **No se sincronizan** entre PC y móvil, ni entre Chrome y Edge. |
| **Privacidad** | Si alguien entra en "Borrar datos de navegación → cookies y datos de sitios", **se borran**. También con modo incógnito (no persiste al cerrar). |
| **Usuarios** | Cada usuario de Windows tiene su propio perfil → sus propias notas. |

---

## Alternativas según lo que necesites

| Necesidad | Dónde guardarlas |
|---|---|
| "Que no dependa del navegador ni se pierdan si limpio cookies" | Botón **Exportar JSON** → descarga `notas.json` a tu carpeta + botón **Importar**. Cero infraestructura. |
| "Quiero acceder a las notas desde PC y móvil" | Subir a Railway/Supabase o un simple endpoint con Postgres. Ya no es MVP. |
| "Solo para esta sesión, que no queden guardadas" | `sessionStorage` en vez de `localStorage`. Se borran al cerrar pestaña. |
| "Que se pueda editar el JSON como un archivo" | Exportar/importar JSON (lo mismo que la primera fila). |

---

## Recomendación alineada con las reglas del proyecto

Para el MVP → **`localStorage`** (sin backend, sin dependencias, escalable luego).
Añade también un par de botones pequeños en el dashboard:

- **⬇ Exportar notas** → descarga `norlexia_notas_YYYY-MM-DD.json`
- **⬆ Importar notas** → selecciona un JSON y lo fusiona

Así, aunque borres el navegador o cambies de PC, puedes llevarte tus notas en un archivo. Son ~15 líneas extra.

¿Quieres que te prepare ese parche (Opción A + exportar/importar) para cuando pases a modo Agent?

---

**User**

cuando añado la nota desde el movil uqier qeu se almacene y cuando haga el test en el pc pueda verla , debera estar asociada esa nota a esa pregunta 
2 entiendo que tendra qeu alamacenarse en alguna base de datos 
3. para ayudar podria servir renmobrar las preguntas con numero del 1 al 56 , mas el mnombre del test por ejemplo 2025_1 comun , asi se indexa a esa pregunta.
4. con estas necesidades como se podri hacer sin tocar casi nada del programa  solo una llamada o de alguna manera mas facil


---

**Cursor**

Perfecto, vamos por partes. Tu intuición es correcta: **para sincronizar móvil ↔ PC sí hace falta un servidor/BBDD en la nube**. `localStorage` es solo local al navegador. No hay atajo posible. Pero hay formas muy sencillas de hacerlo sin montar backend propio.

---

## 1. Sobre la clave de la pregunta (punto 3) — sí, es buena idea

Muy acertado. Lo mejor es **no renombrar los CSV** sino generar la clave al vuelo en el código a partir del nombre del examen y el índice de la pregunta. Una sola línea:

```js
const keyQ = (examName, idx) => 
  `${examName.toLowerCase().replace(/\s+/g,"_")}__q${String(idx+1).padStart(2,"0")}`;

// Ejemplos:
// "Materias Comunes 2025.2" → idx 0  → "materias_comunes_2025.2__q01"
// "Laboral 2024"            → idx 15 → "laboral_2024__q16"
```

Ventajas sobre usar el texto de la pregunta como clave:
- **Estable**: aunque corrijas una errata en el CSV, la nota sigue asociada.
- **Corta y legible**: fácil de inspeccionar en la BBDD.
- **Escalable**: por materia, por año, por convocatoria — lista para filtros y estadísticas futuras.

---

## 2. Dónde almacenar las notas para que se sincronicen

Tres opciones, de **más sencilla** a **más completa**:

### 🟢 Opción 1 — JSONBin.io (la más simple, ~15 líneas de código)

Es un servicio que guarda un JSON en la nube y lo recupera con `fetch`. Cero backend propio.

- **Registro**: gratis, email.
- **Te dan**: un `bin_id` + una `API_KEY`.
- **Código total**: 2 funciones (guardar / cargar).

```js
const BIN_ID  = "XXXXXXXX";   // tu bin
const API_KEY = "$2a$10$...";  // tu clave maestra

async function cargarNotas() {
  const r = await fetch(`https://api.jsonbin.io/v3/b/${BIN_ID}/latest`, {
    headers: { "X-Master-Key": API_KEY }
  });
  return (await r.json()).record || {};
}

async function guardarNotas(obj) {
  await fetch(`https://api.jsonbin.io/v3/b/${BIN_ID}`, {
    method: "PUT",
    headers: { "Content-Type": "application/json", "X-Master-Key": API_KEY },
    body: JSON.stringify(obj)
  });
}
```

**Ventajas**: literalmente 2 llamadas. Funciona en móvil y PC al instante.
**Pegas**: gratis hasta 10 000 peticiones/mes (suficiente), y la clave va en el HTML (cualquiera que vea el código podría escribir). **Para uso personal, perfecto**.

---

### 🟡 Opción 2 — Supabase (la que recomiendo — alineada con las reglas del proyecto)

Postgres en la nube con API REST automática y SDK JS. Encaja con Railway/Vercel (mencionados en tus reglas).

- **Registro**: gratis (500 MB DB, más que sobra).
- **Te dan**: `URL` del proyecto + `ANON_KEY`.
- **Creas una tabla** desde el panel web (clicks, sin SQL):
  - `notas` con columnas `id_pregunta` (text, PK), `nota` (text), `updated_at` (timestamp).
- **Código**: incluyes el CDN del SDK + ~15 líneas.

```html
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
```

```js
const sb = supabase.createClient("https://xxx.supabase.co", "eyJ...anon_key");

async function cargarNotas() {
  const { data } = await sb.from("notas").select("id_pregunta, nota");
  const map = {};
  data?.forEach(r => map[r.id_pregunta] = r.nota);
  return map;
}

async function guardarNota(id, texto) {
  await sb.from("notas").upsert({ id_pregunta: id, nota: texto });
}
```

**Ventajas**:
- Escalable a estadísticas, usuarios, ranking, etc. (punto 25–27 de tus reglas).
- Consultas SQL reales desde el panel.
- Si luego quieres autenticación (Google/email) es activar un interruptor.
- Cumple con "evitar dependencias innecesarias" — es el mínimo posible para tener BBDD real.

**Pegas**: la `anon_key` también va en el HTML. Para proteger escritura se añade una Row Level Security en 2 clicks.

---

### 🟠 Opción 3 — Sincronización manual vía archivo (cero servidores)

Botones **Exportar** e **Importar** JSON. Cuando acabas en el móvil, exportas, te mandas el archivo (WhatsApp / email / Drive) y lo importas en el PC. No automático, pero 0 código de red y 0 dependencias.

**Pegas**: manual. No es lo que has pedido, pero lo menciono por completitud.

---

## 3. Comparativa rápida

| Criterio | JSONBin | **Supabase** ⭐ | Export/Import manual |
|---|---|---|---|
| Sincronización automática móvil↔PC | ✅ | ✅ | ❌ |
| Registro gratis | ✅ | ✅ | — |
| Líneas de código | ~15 | ~20 | ~15 |
| Base de datos real | ❌ (solo JSON) | ✅ Postgres | ❌ |
| Escalable a estadísticas/usuarios | Limitado | ✅ | ❌ |
| Dependencias externas | `fetch` | 1 SDK (CDN) | Ninguna |
| Coste | Gratis (10k req/mes) | Gratis (500 MB) | 0 |
| Alineado con reglas del proyecto | Parcial | **Total** | Parcial |

---

## 4. Lo "casi sin tocar nada" concreto

Con cualquiera de las dos opciones online, el impacto sobre tu `index.html` es este:

1. **1 `<script>` en el `<head>`** (solo Supabase; JSONBin no necesita nada).
2. **1 bloque `<div id="qNota"></div>`** dentro de `.q-card` (ya lo vimos antes).
3. **~20 líneas JS** con las funciones `cargarNotas()`, `guardarNota(id, texto)` y la clave `keyQ()`.
4. **2 líneas dentro de `renderQ()`**: cargar nota de la pregunta actual + pintar textarea.
5. **1 evento `onblur`** en el textarea que llama a `guardarNota()`.

Total: **~30 líneas nuevas + 2 modificaciones mínimas**. Ningún cambio en los CSV. Ningún cambio en el flujo del examen. Ningún cambio en el dashboard.

Patrón recomendado para no bloquear la UI:
- Al **iniciar la app** (o al entrar al examen) se hace **una sola llamada** `cargarNotas()` y se cachea en memoria + `localStorage` (offline fallback).
- Al **escribir una nota** se guarda optimista en memoria + `localStorage` y se dispara un `guardarNota()` en background. Si falla la red, se reintenta.

Así el examen sigue siendo instantáneo incluso sin conexión y las notas se sincronizan en cuanto haya red.

---

## Mi recomendación

**Supabase + clave `examname__qNN`**.

Motivos:
- Cumple tus 4 puntos con el mínimo código.
- Escalable a las funciones que ya tienes previstas en las reglas (estadísticas, progreso por materia, usuarios).
- Compatible con Railway/Vercel (tu stack objetivo).
- Si algún día quieres que cada persona tenga sus propias notas, solo hay que añadir `user_id` a la tabla y activar login — no hay que reescribir nada.

Si quieres algo funcionando **hoy en 10 minutos** sin registrarte en nada nuevo, usa **Opción 3 (export/import JSON)** como puente y luego migras a Supabase.

---

¿Te preparo el parche exacto con Supabase (crear tabla + código listo) para cuando pases a modo Agent? Necesitaría que primero:
1. Crees cuenta gratis en [supabase.com](https://supabase.com).
2. Crees un proyecto y me pases la `URL` y la `anon_key` (las tienes en *Project Settings → API*).

O si prefieres JSONBin, lo mismo pero con `bin_id` y `API_KEY`.

---

**User**

tengo vercel y supabase vamos a crear un commit antes para volver aqui en caso de que falle 

---

**Cursor**

Estoy en **modo Ask**, así que no puedo ejecutar `git` ni hacer commits yo directamente. Tienes dos caminos:

---

## Opción 1 — Cambia a modo Agent (recomendado)

Pulsa el selector de modo en Cursor y elige **Agent**. Entonces te lanzo la secuencia completa (status → add → commit → verificación) con un mensaje de commit alineado con el estado del proyecto antes de tocar nada de notas/Supabase.

Propuesta de flujo que ejecutaría:

1. `git status` para ver qué hay pendiente.
2. `git diff` para revisar cambios.
3. `git log --oneline -5` para ver el estilo de mensajes previos.
4. `git add -A`
5. `git commit -m "checkpoint: estado estable previo a integración de notas por pregunta con Supabase"`
6. `git status` de verificación.

Y como red de seguridad extra, te crearía también una **etiqueta** para volver fácilmente:

```bash
git tag -a v-pre-notas -m "Punto de retorno antes de añadir notas + Supabase"
```

Así luego basta con `git checkout v-pre-notas` si quieres volver.

---

## Opción 2 — Hazlo tú mismo ahora (3 comandos)

Abre la terminal de Cursor (Ctrl+ñ) en la raíz del proyecto y ejecuta:

```bash
git status
```

Para revisar qué vas a incluir. Luego:

```bash
git add -A
git commit -m "checkpoint: estado estable previo a integracion de notas por pregunta con Supabase"
git tag -a v-pre-notas -m "Punto de retorno antes de notas + Supabase"
```

Verifica con:

```bash
git log --oneline -3
```

### Si luego necesitas volver a este punto

- **Ver diferencias respecto al checkpoint**: `git diff v-pre-notas`
- **Deshacer cambios no commiteados**: `git checkout -- .`
- **Volver completamente a ese commit** (descartando cambios posteriores): `git reset --hard v-pre-notas` ⚠️ destructivo.
- **Crear una rama desde ese punto** (opción segura): `git checkout -b backup-pre-notas v-pre-notas`

### Extra de seguridad — push al remoto

Si tienes GitHub configurado, sube el checkpoint a la nube para no depender solo del disco local:

```bash
git push
git push --tags
```

---

## ¿Y después?

Cuando tengas el commit hecho, pásame:

1. La **URL** del proyecto Supabase (la que aparece en *Project Settings → API → Project URL*).
2. La **anon/public key** (misma pantalla).
3. Si quieres que active **Row Level Security** desde el principio (recomendado) o lo dejamos abierto para el MVP.

Con esos 2 datos + modo Agent activo, te preparo:

- La tabla `notas` en Supabase (te doy el SQL listo para pegar en *SQL Editor*).
- El parche mínimo (~30 líneas) sobre `index.html` con la clave `examname__qNN`, textarea bajo las opciones, banner "📝 Nota previa" tras responder, y sync automático móvil↔PC.

Dime cuando tengas el commit + los datos y cambia a Agent.

---

**User**

vamos a implementar las modificaciones de nota 

---

**Cursor**

El árbol de trabajo está limpio (último commit `3b40ceb`). No hace falta commit, basta con crear una **etiqueta de seguridad** que apunte al estado actual para poder volver.

Etiqueta `v-pre-notas` creada. Si algo falla: `git reset --hard v-pre-notas`.

Ahora antes de implementar necesito que me confirmes 2 datos. Mientras los buscas, voy preparando el código.

> **¿Tienes ya creado el proyecto en Supabase y a mano la URL + anon key?**
> - Sí, los tengo (te los pego en el siguiente mensaje)
> - No, créame el SQL de la tabla y dime qué hacer en el panel ✔
> - Empieza solo con localStorage y dejas un hueco para Supabase luego

> **¿Las notas serán privadas por usuario o compartidas (todos ven las mismas)?**
> - Compartidas: las notas son mías, accedo desde móvil y PC con el mismo origen (más simple, sin login) ✔
> - Privadas con login (Google/email) — añade complejidad pero es escalable a multiusuario

Tengo todo el contexto. Voy a implementar paso a paso. El diseño es:

**Estrategia técnica:**
- ID estable por pregunta: `slug(nombreExamen)__qNN` (p.ej. `materias_comunes_2025.2__q01`)
- Cache en memoria + `localStorage` como fallback offline
- Sync con Supabase en background (la app funciona aunque no haya red)
- Si Supabase no está configurado, todo va a `localStorage` (el código no rompe)

## 1) SDK de Supabase en `<head>`

## 2) CSS de la nota

## 3) Contenedores en `.q-card` (examen y repaso)

## 4) Módulo NOTAS en JS (insertado antes de `// ══ STATE ══`)

## 5) `startExam()` — añadir `_origIdx` a cada pregunta

## 6) Llamadas `renderNota()` en `renderQ()` y `renderRepaso()`

## 7) Init: cargar notas al arranque

## 8) Guía de configuración Supabase

## 9) Verificar lints

Implementación terminada. Resumen de lo añadido:

## Archivos modificados

- **`index.html`** (~150 líneas nuevas):
  - SDK Supabase vía CDN en `<head>`
  - CSS para textarea, banner dorado y estados de sincronización
  - 2 contenedores `<div class="nota-wrap">` (uno examen, uno repaso)
  - Módulo **NOTAS** completo: cache + Supabase + fallback localStorage
  - 1 línea modificada en `startExam()` (añade `_origIdx`)
  - 1 línea añadida en `renderQ()` y otra en `renderRepaso()`
  - 1 línea añadida al init (`initNotas()`)

- **`SUPABASE_SETUP.md`** (nuevo): guía paso a paso con el SQL listo para pegar.

## Cómo funciona

1. **Sin Supabase configurado** (estado actual): la app sigue funcionando, las notas se guardan en `localStorage`. Verás el indicador **"Solo local"** en cada nota.
2. **Con Supabase configurado**: 
   - Al arrancar la app descarga todas las notas a memoria + cache local.
   - Al editar una nota → se guarda local instantáneo + se sube a Supabase en background.
   - Indicador en tiempo real: `Guardando…` → `Sincronizada ✓`.
   - Si no hay red → `Error sync · guardada local` (se reintenta al volver a editar).
3. **Identificador estable** por pregunta: `materias_comunes_2025.2__q01`, `laboral_2024__q16`, etc. (visible en la UI dentro de la propia nota — útil para debug).
4. **Banner de nota previa**: cuando respondes a una pregunta que ya tenía nota, aparece un recuadro dorado **"📝 Nota: ..."** encima del cuadro de edición, antes de pulsar "Siguiente". Justo lo que pediste.

## Lo que tienes que hacer ahora

Sigue **`SUPABASE_SETUP.md`** (5 minutos):

1. Crear proyecto Supabase (gratis).
2. Copiar/pegar el SQL en el SQL Editor → crea la tabla `notas`.
3. Copiar `Project URL` y `anon key` en el bloque CONFIG de `index.html` (líneas ~1148-1149):

```js
const SUPABASE_URL = "";   // ← pega aquí
const SUPABASE_KEY = "";   // ← pega aquí
```

4. Recarga la página y prueba.

## Punto de retorno

Si algo falla, vuelves al estado anterior con:

```bash
git reset --hard v-pre-notas
```

¿Quieres que haga commit del estado actual ahora con un mensaje tipo "feat: notas por pregunta con Supabase + fallback local"?

---

**User**

PUEDO PROBARLO ANTES , COMO LO HACIA

---

**Cursor**

Sí, puedes probarlo **ahora mismo sin tocar Supabase**. La app funciona en modo **"Solo local"** (las notas se guardan en `localStorage` del navegador) hasta que rellenes las credenciales.

## Cómo probar (igual que antes)

### Opción A — Doble clic (lo más rápido)

1. Abre el Explorador de Windows en la carpeta del proyecto.
2. Doble clic en **`index.html`** → se abre en tu navegador por defecto.
3. Selecciona un examen, pulsa **Iniciar examen** y dentro de cualquier pregunta verás el cuadro **📝 Nota personal**.
4. Escribe algo, haz click fuera del cuadro → verás el indicador **"Solo local"** en dorado.
5. Responde a la pregunta → aparece el banner dorado **"📝 Nota: ..."** encima del cuadro.

### Opción B — Desde Cursor (doble clic también funciona)

En Cursor, click derecho sobre `index.html` en el panel de archivos → **"Reveal in File Explorer"** → doble clic.

### Opción C — Con servidor local (si ves errores al cargar CSVs)

Si al abrir con `file://` no cargan los CSVs de `/examenes/`, abre una terminal en la carpeta y lanza:

```bash
python -m http.server 8000
```

Luego abre `http://localhost:8000` en el navegador.

---

## Qué debes comprobar

| Prueba | Resultado esperado |
|---|---|
| 1. Inicia examen, mira debajo de las opciones | Cuadro **📝 Nota personal** con ID `materias_comunes_..._q01` |
| 2. Escribe nota + click fuera del cuadro | Indicador cambia a **"Solo local"** (dorado) |
| 3. Cambia de pregunta (Siguiente) y vuelve (Anterior) | La nota persiste |
| 4. Responde a la pregunta | Aparece banner dorado **"📝 Nota: ..."** encima del textarea |
| 5. Cierra el navegador y vuelve a abrir | La nota sigue ahí (está en `localStorage`) |
| 6. Abre en móvil (misma PC, IP local) | La nota **NO** aparece (todavía no hay Supabase) — esto es lo que arreglará Supabase |

---

## Ver las notas guardadas en `localStorage`

Útil para depurar: abre el navegador → pulsa **F12** → pestaña **Application** (Chrome/Edge) o **Storage** (Firefox) → **Local Storage** → selecciona tu archivo → busca la clave `norlexia_notas_v1`.

Verás algo como:

```json
{
  "materias_comunes_2025.2__q01": "Art. 24 CE — tutela judicial efectiva",
  "laboral_2024__q15": "Plazo de 20 días hábiles para despido"
}
```

---

## Si algo va raro

- **La página está en blanco / da error**: abre la consola con F12 → pestaña **Console** → mándame la traza.
- **No sale el cuadro de nota**: puede ser caché. Fuerza recarga con **Ctrl + F5**.
- **Quieres volver al estado anterior de golpe**:

```bash
git reset --hard v-pre-notas
```

¿Lo pruebas y me cuentas si se ve bien antes de configurar Supabase?

---

**User**

MODIFICAR 
1. LA CASILLA DE NOTA SE ACTIVA CUANDO SE CONTESTA A LA PREGUNTA /  EN LA CREACION DE LA NOTA /  HAY UN BOTON PEQUEÑO QUE INDICA "CREAR NOTA "  / SI NO SE HACE CLIC EN ESE BOTON Y SE DA " SIGUIENTE"  NO SE ABRE EL CAMPO  DE RELLENAR NOTA Y SALTA A LA SIGUIENTE.
2 - EN SUCESIVAS APERTURAS DEL TEST , SI SE HAN CREADO NOTAS , LA NOTA  SE VISUALIZA NADA MAS CONTESTARLA ANTES DE PASAR A LA SIGUIENTE 
3. SE TIENE QUE GUARDAR EN LA BASE DE DATOS PARA VER CON EL LINK DE LA APP EN CUALQIER PC O MOVIL 
 SI HAY LA NOTA SI TIENE LA PREGUNTA UNA VEZ AÑADIDA LA NOTA
4 .ANTES DE HACER NADA PREGUNTAME O DIME SI LO HE EXPLICADO BIEN

---

**Cursor**

Antes de tocar código, te recapitulo lo que entiendo y pregunto lo dudoso. Dime si está bien.

## Cómo entiendo el nuevo flujo

### Pregunta NO respondida todavía
- Debajo de las opciones **NO aparece nada** relativo a notas (ni textarea, ni botón).
- Solo aparecen las 4 opciones y los botones de navegación.

### Pregunta RESPONDIDA (has hecho clic en una opción)

**Caso A — La pregunta NO tiene nota guardada previamente:**
- Aparece un **botón pequeño "➕ Crear nota"** debajo de las opciones.
- Si pulsas "Siguiente →" sin hacer nada → salta a la siguiente, no se crea nada.
- Si pulsas el botón "Crear nota" → se abre el textarea para escribir. Al salir del campo (`blur`) se guarda automáticamente (local + Supabase).

**Caso B — La pregunta YA tiene nota (de una sesión anterior):**
- Aparece **directamente el banner dorado "📝 Nota: [tu texto]"** con la nota que tenías.
- Justo debajo, un botón pequeño **"✏ Editar nota"** por si quieres modificarla.
- Si pulsas "Siguiente" sin tocar nada → avanza, la nota sigue ahí intacta.
- Si pulsas "Editar nota" → se abre el textarea con el texto actual, editable.

### Persistencia (móvil ↔ PC)
- Notas guardadas en **Supabase** (base de datos en la nube).
- El link de la app (Vercel) sirve las mismas notas en cualquier dispositivo.
- Si no hay red: `localStorage` como respaldo y sync cuando vuelva.

---

## Dudas que necesito aclarar antes de tocar código

> **¿El flujo que he descrito arriba es correcto?**
> - Sí, exactamente así. Adelante.
> - Casi, pero tengo matices (te los explico después de responder las demás) ✔
> - No, re-explico desde cero

> **En la pantalla de REPASO (preguntas falladas al final del test), ¿aplica el mismo flujo de botón 'Crear nota' / 'Editar nota'?**
> - Sí, mismo comportamiento que en el examen
> - No, en repaso no mostrar nada de notas
> - En repaso solo mostrar la nota existente si hay (banner), sin opción de crear/editar ✔

> **Cuando se abre el textarea para escribir/editar, ¿cómo se guarda la nota?**
> - Automático al salir del campo (click fuera) — como ahora, más fluido ✔
> - Con un botón explícito 'Guardar' — más control
> - Ambos: automático al salir + botón 'Guardar y cerrar' para colapsar el campo

> **¿Ya has rellenado las credenciales de Supabase en el index.html (SUPABASE_URL y SUPABASE_KEY) o sigues en modo 'Solo local'?**
> - Sí, ya funciona la sincronización con Supabase ✔
> - No, todavía no — necesito ayuda con eso antes
> - Quiero probar primero solo local, y configurar Supabase al final

Perfecto, casi todo claro. Solo me queda saber los matices que tienes sobre el flujo principal.

Cuéntame **qué quieres cambiar exactamente** respecto a lo que describí. Por ejemplo, piensa en estas dudas concretas:

1. **Visibilidad del botón "Crear nota"**: ¿quieres que aparezca **solo después de responder**, o ya está visible siempre y es el textarea lo que solo aparece tras clic?

2. **Caso B (pregunta con nota previa)**: ¿el banner dorado con la nota se muestra **automáticamente al responder**, o también detrás de un botón tipo "Ver mi nota"?

3. **Posición del botón "Crear nota" / "Editar nota"**: ¿dónde exactamente?
   - (a) Debajo de las opciones, alineado a la izquierda, pequeño tipo link.
   - (b) En la barra de navegación inferior junto a "Siguiente →".
   - (c) Otro sitio.

4. **Texto del botón**: ¿prefieres…?
   - `➕ Crear nota` / `✏ Editar nota`
   - `📝 Añadir nota` / `📝 Ver/editar nota`
   - Otro.

5. **Al abrir el textarea para escribir**: ¿quieres un botón "✕ Cerrar / Cancelar" para ocultarlo sin guardar si te arrepientes, o basta con click fuera?

6. **Cuando escribes una nota nueva y sales del campo**: ¿el textarea se cierra y queda el banner "📝 Nota: ..." + botón "Editar nota", o sigue abierto mientras respondes la pregunta?

Dime los matices y ajusto el plan antes de tocar nada.

---

**User**

EN ESTE APARTADO Caso B — La pregunta YA tiene nota (de una sesión anterior):

Aparece directamente  con la nota que tenías ABAJO AL CONTESTARLA ANTES DE DAR SIGUIENTE.
ADEMAS , un botón pequeño "✏ Editar nota" por si quieres modificarla.

---

**Cursor**

Perfecto, entendido. Caso B confirmado: al responder, aparece el texto de la nota + botón "✏ Editar nota" debajo, antes de pulsar Siguiente.

Voy a cerrar los detalles pendientes con **decisiones por defecto razonables** (todas son reversibles). Dime si te valen o si cambiamos algo.

## Diseño final propuesto

### ▶ Antes de responder
Nada de notas. Solo las 4 opciones y los botones de navegación.

### ▶ Después de responder — Caso A (sin nota previa)
Debajo de las opciones aparece un enlace/botón pequeño:

```
➕ Añadir nota
```

- Discreto, alineado a la izquierda, con estilo de enlace (no un botón grande).
- Click → se despliega el textarea. Escribes → click fuera (blur) → se guarda automáticamente (Supabase + local).
- Una vez escrita → el textarea se colapsa y queda como **Caso B**.
- Si no pulsas el enlace → Siguiente avanza sin crear nada.

### ▶ Después de responder — Caso B (con nota previa o recién creada)
Debajo de las opciones aparece:

```
📝 Nota: [el texto que guardaste]                    ✏ Editar
```

- Banner dorado con el texto completo de la nota.
- Botón pequeño "✏ Editar" al lado (o debajo).
- Click en "✏ Editar" → se abre textarea con el texto → escribes → click fuera → se guarda → vuelve al banner actualizado.
- Si borras todo el texto y sales → la nota se elimina y vuelve al **Caso A**.

### ▶ Pantalla de repaso
Solo lectura: si la pregunta tenía nota, se muestra el banner "📝 Nota: ..." al responder. Sin botones de crear/editar.

### ▶ Cerrar el editor sin guardar
No hay botón "Cancelar" explícito. El guardado es por blur. Si escribes algo y no quieres guardarlo → borra el texto antes de salir del campo. **Simple y alineado con el "toca poco código"**.

### ▶ Posición y texto
- **Posición**: debajo de las opciones, dentro de la `.q-card`, antes de la barra de navegación.
- **Textos exactos**:
  - `➕ Añadir nota` (botón link pequeño)
  - `📝 Nota: ...` (banner dorado)
  - `✏ Editar` (botón link pequeño junto al banner)

---

> **¿Procedo a implementarlo exactamente así?**
> - Sí, adelante tal cual ✔
> - Sí pero cambiar los textos (dime después cuáles)
> - No, hay algo del diseño que quiero cambiar

Perfecto, voy al lío.

## 1) CSS para los botones-enlace de nota

## 2+3) Reescribir `renderNota()` + añadir helpers

## 4) Actualizar `renderQ()` y `renderRepaso()`

Ahora simplifico las llamadas: uso los helpers `renderQNota()` y `renderRepasoNota()`, y limpio el estado de edición al cambiar de pregunta.

## 5) Verificar

Hecho. Resumen de los cambios:

## Nuevo flujo implementado

### En el examen (pantalla de preguntas)

| Momento | Qué se ve debajo de las opciones |
|---|---|
| Pregunta sin responder | **Nada**. Solo opciones y nav. |
| Pregunta respondida, **sin nota previa** | Botón pequeño `➕ Añadir nota` |
| Pregunta respondida, **con nota previa** | Banner dorado `📝 Nota: [texto]` + botón `✏ Editar nota` |
| Durante edición (tras clic en el botón) | Textarea con autofocus. Al hacer clic fuera → guarda y cierra. Queda en estado "con nota previa". |

### En la pantalla de repaso (preguntas falladas al final)

- Si la pregunta tiene nota → banner `📝 Nota: [texto]` (solo lectura).
- Si no tiene → no aparece nada relativo a notas.

### Persistencia

- **Guardado**: al salir del campo (blur). Cache en memoria → `localStorage` → Supabase (async).
- **Sincronización móvil↔PC**: automática vía Supabase si tienes las credenciales configuradas.
- **Sin red / sin Supabase**: sigue funcionando con `localStorage` y muestra "Solo local".

### Detalles técnicos

- Estado de edición por pregunta controlado con un `Set` (`notasEditing`) — siempre hay como máximo una abierta.
- Al navegar entre preguntas se limpia el estado (`notasEditing.clear()` en `renderQ`), así no quedan editores abiertos.
- Si borras todo el texto y sales del campo → la nota se elimina de la BBDD y vuelve al botón "➕ Añadir nota".
- El ID de la pregunta (p.ej. `materias_comunes_2025.2__q01`) se muestra junto al editor en modo edición, útil para depurar.

## Cómo probarlo ahora

1. Recarga la página (Ctrl+F5 para forzar caché).
2. Inicia un examen.
3. Prueba el ciclo completo:
   - Contesta una pregunta → clic en `➕ Añadir nota` → escribe algo → clic fuera.
   - Debería aparecer el banner `📝 Nota: ...` + `✏ Editar nota`.
   - Siguiente pregunta (Caso A sin nota). Anterior vuelve y la nota sigue.
   - En un test futuro sobre el mismo examen, al contestar esa pregunta verás el banner directamente.

## Rollback por si algo falla

```bash
git reset --hard v-pre-notas
```

¿Pruebas y me dices cómo se ve? Si todo va bien, cuando confirmes puedo preparar el commit.

---

**User**

1. en local si funciona,  crea la nota , cuando salgo del examen y vuelvo a entrar me la ha guardado 
2. como se asocia la pregunta co nla nota  ?  
3. simplifica si se numeran las preguntas del 1 al 56 ?

---

**Cursor**

¡Genial que funcione en local! Te respondo las dos dudas.

## 2) Cómo se asocia la pregunta con la nota

La asociación se hace con un **ID único generado** para cada pregunta, que combina:

**`{nombre_examen_en_slug}__q{número_pregunta_01-99}`**

Esto vive en la función `keyQ()` del código:

```1168:1171:index.html
    function keyQ(examName, origIdx) {
      const n = (origIdx == null ? 0 : origIdx) + 1;
      return slugify(examName) + "__q" + String(n).padStart(2, "0");
    }
```

### Ejemplos reales con tus exámenes

| Examen + pregunta nº | ID generado |
|---|---|
| Materias Comunes 2025.2, pregunta 1 | `materias_comunes_2025.2__q01` |
| Materias Comunes 2025.2, pregunta 56 | `materias_comunes_2025.2__q56` |
| Laboral 2024, pregunta 7 | `laboral_2024__q07` |
| Civil 2025.1, pregunta 23 | `civil_2025.1__q23` |

### Por qué es robusto

El **`origIdx`** (índice original) se asigna al arrancar el examen, **antes** de que se mezclen las preguntas en modo aleatorio:

```1471:1471:index.html
      let qs = ex.preguntas.map((p, i) => ({ ...p, _origIdx: i }));
```

Así, la **pregunta 5 del CSV siempre será `__q05`**, independientemente de si te sale en la posición 1, 17 o 42 del test aleatorio.

### Dónde puedes verlo funcionando

1. **Dentro de la app**: cuando editas una nota, el ID aparece en el `chip` gris junto al título "Editar nota".
2. **En `localStorage`**: F12 → Application → Local Storage → clave `norlexia_notas_v1` → verás un JSON como:
   ```json
   {
     "materias_comunes_2025.2__q01": "Art. 24 CE...",
     "laboral_2024__q15": "Plazo 20 días..."
   }
   ```
3. **En Supabase** (cuando lo conectes): Table Editor → `notas` → verás una fila por nota con `id_pregunta`, `nota`, `updated_at`.

---

## 3) ¿Simplificar numerando del 1 al 56?

**Buena noticia: ya está así.** La numeración actual es `__q01`, `__q02`, ..., `__q56` — exactamente lo que pedías. Lo único que añade el sistema es el **nombre del examen** delante para que no se mezclen las notas entre distintos exámenes (por ejemplo la pregunta 1 de Laboral y la pregunta 1 de Civil no deben compartir nota).

### Si aun así lo ves largo, hay tres grados de simplificación

| Opción | Ejemplo ID | Ventaja | Pega |
|---|---|---|---|
| **A. Actual** (recomendada) | `materias_comunes_2025.2__q01` | Legible, autodescriptivo, único | Largo (28 caracteres) |
| **B. Abreviado** | `mc25.2__q01` | Compacto y legible | Ambiguo si creces (¿`mc` es Mercantil o Materias Comunes?) |
| **C. Solo número global** | `q001` | Ultra-corto | **Se rompe al cambiar CSV**: si reordenas o añades exámenes, las notas se pegan a preguntas distintas |

### Mi recomendación

**Deja la Opción A**. Es la que tienes ahora y cumple todos los requisitos:
- Se lee solo: `materias_comunes_2025.2__q01` → obvio a qué pregunta apunta.
- Escalable cuando añadas más exámenes (no hay colisiones).
- Si algún día borras un examen y vuelves a subirlo, las notas encajan por sí solas.
- Compatible con futuras estadísticas ("¿cuántas notas tengo por materia?" → 1 SQL).

El "ruido visual" del ID solo se ve al editar una nota (en el chip gris). En el banner dorado `📝 Nota: ...` y en el botón `➕ Añadir nota` no aparece.

---

## Siguiente paso: probar con Supabase

Ahora que confirmas que local funciona, el siguiente test importante es **sincronizar móvil ↔ PC**. Para eso:

1. Si todavía no has configurado Supabase, sigue los pasos 1-3 de `SUPABASE_SETUP.md` (crear proyecto, ejecutar SQL, pegar URL+KEY en `index.html`).
2. Recarga.
3. Verás que el indicador pasa de **"Solo local"** (dorado) a **"Sincroniza con Supabase"** (verde) cuando abres una nota.

¿Te ayudo con algo de la configuración de Supabase, o quieres que avance con otra cosa (commit, deploy a Vercel, etc.)?

---

**User**

1. dame los pasos para probar con supa base 
2. como obtengo la url del servidor , de github o vercel , no se con cual 

---

**Cursor**

Te aclaro primero un malentendido importante, porque se mezclan **3 URLs distintas** que cumplen funciones diferentes:

## Las 3 URLs del proyecto — qué hace cada una

| URL | Para qué sirve | De dónde sale | ¿La necesitas para las notas? |
|---|---|---|---|
| **URL de GitHub** | Guardar el código fuente | Tu repo `github.com/janorlex-dev/EE-examen-de-estado-API-CURSOR_V2` | ❌ No |
| **URL de Vercel** | Abrir la app desde el móvil/PC con un link público tipo `tu-app.vercel.app` | Vercel (hay que hacer deploy) | ✅ Sí, para abrirla desde el móvil |
| **URL de Supabase** | La base de datos donde se guardan las notas | Supabase (proyecto creado) | ✅ Sí, va dentro del `index.html` |

**Resumen:**
- **Supabase URL** → va DENTRO del código, para que la app sepa dónde guardar.
- **Vercel URL** → es la que tú copias al móvil para abrir la app.
- **GitHub** → solo almacena el código; NO sirve para abrir la app.

---

## 1) Pasos para probar con Supabase (solo la BBDD, sin Vercel aún)

Con esto las notas ya se sincronizan entre PCs/móviles **siempre que abras el mismo `index.html`** (ya sea local o en Vercel).

### Paso 1 — Crear el proyecto Supabase

1. Abre [https://supabase.com](https://supabase.com) → **Sign in** (puedes usar GitHub).
2. **New project**:
   - Name: `norlexia-test-abogacia`
   - Database Password: genera una segura, guárdala (no se usa en el HTML).
   - Region: `West EU (Ireland)` o la más cercana.
   - Plan: `Free`.
3. Espera ~1 minuto.

### Paso 2 — Crear la tabla `notas`

1. Panel izquierdo → **SQL Editor** → **New query**.
2. Pega este SQL completo y pulsa **Run** (▶):

```sql
create table if not exists public.notas (
  id_pregunta text primary key,
  nota        text not null default '',
  updated_at  timestamptz not null default now()
);
create index if not exists notas_updated_at_idx on public.notas (updated_at desc);
alter table public.notas enable row level security;
drop policy if exists "open_select" on public.notas;
drop policy if exists "open_insert" on public.notas;
drop policy if exists "open_update" on public.notas;
drop policy if exists "open_delete" on public.notas;
create policy "open_select" on public.notas for select using (true);
create policy "open_insert" on public.notas for insert with check (true);
create policy "open_update" on public.notas for update using (true) with check (true);
create policy "open_delete" on public.notas for delete using (true);
```

Debes ver `Success. No rows returned`.

3. Verifica: panel izquierdo → **Table Editor** → debería aparecer `notas` con 3 columnas.

### Paso 3 — Copiar credenciales Supabase al `index.html`

1. Panel izquierdo → **Project Settings** (engranaje) → **API**.
2. Copia:
   - **Project URL**: algo como `https://abcdefghijklmno.supabase.co`
   - **Project API keys → `anon` `public`**: un token largo `eyJ...`
3. Abre `index.html` y busca el bloque (líneas 1144-1147 aprox):

```1145:1147:index.html
    const SUPABASE_URL = "";   // ej: "https://xxxxxxxx.supabase.co"
    const SUPABASE_KEY = "";   // anon/public key
```

4. Pega los valores entre las comillas y guarda.

### Paso 4 — Probar

1. En tu terminal, donde tienes `npx live-server` corriendo, recarga el navegador con **Ctrl+F5**.
2. Inicia un examen.
3. Contesta una pregunta → pulsa **➕ Añadir nota** → escribe algo → clic fuera.
4. El indicador debería pasar de **"Solo local"** (dorado) a **"Sincronizada ✓"** (verde).
5. Vuelve a Supabase → **Table Editor** → `notas` → debería aparecer una fila con tu nota.

### Paso 5 — Probar sincronización PC ↔ PC (opcional, sin móvil aún)

- Abre `index.html` en Chrome → escribe nota.
- Abre el **mismo archivo** en Firefox (o en otro perfil de Chrome) → **Ctrl+F5** → haz el test → al responder la pregunta debería cargar la nota desde Supabase.

---

## 2) Para abrirlo desde el móvil — Deploy a Vercel

Para que el móvil pueda acceder a la app sin cable USB y sin tener el código, necesitas "publicarla" en Vercel (es gratis, 2 minutos).

### Paso A — Ir a Vercel

1. Abre [https://vercel.com](https://vercel.com) → **Sign up** con GitHub (usa la misma cuenta `janorlex-dev`).
2. En el dashboard → **Add New...** → **Project**.
3. Vercel te lista tus repos de GitHub → busca `EE-examen-de-estado-API-CURSOR_V2` → pulsa **Import**.
4. En la pantalla de configuración:
   - **Framework Preset**: `Other`
   - **Root Directory**: déjalo vacío (raíz)
   - **Build Command**: déjalo vacío
   - **Output Directory**: déjalo vacío
5. Pulsa **Deploy** → espera ~30 segundos.

### Paso B — Obtener la URL pública

Te dará una URL tipo:

```
https://ee-examen-de-estado-api-cursor-v2.vercel.app
```

Esta es la URL que **copias al móvil** (por WhatsApp, email, o escaneando un QR). Al abrirla, ya tienes la app funcionando con las notas sincronizadas contra Supabase.

### Paso C — Despliegue automático

Cada vez que hagas `git push` a GitHub, Vercel redespliega solo. Es decir:
- Editas código en tu PC → `git push`
- En 30 segundos la versión en `vercel.app` está actualizada
- Móvil ve los cambios al recargar

---

## Orden recomendado que sugiero seguir

1. ✅ Hacer Paso 1-4 de Supabase → probar local con sincronización real.
2. ✅ Confirmar que se sincroniza entre navegadores distintos en el mismo PC.
3. ✅ Subir a GitHub (`git push`) para dejar el código actualizado.
4. ✅ Deploy Vercel → obtienes URL pública.
5. ✅ Abrir la URL en el móvil → probar nota desde móvil → verificar que aparece en PC al refrescar.

---

## ¿Por dónde empezamos?

> **¿Qué quieres que haga ahora?**
> - Voy a hacer los pasos 1-3 de Supabase yo mismo. Avísame cuando vuelva con la URL y la anon key pegadas.
> - Acompáñame paso a paso con Supabase (dime qué hago en cada pantalla)
> - Primero hagamos commit de las notas (estado actual funcionando en local) y después Supabase
> - Prefiero hacer primero el deploy a Vercel y después Supabase

Tranquilo, puedes pulsar **"Run this query"** con total seguridad. ✅

## Por qué sale ese aviso

Supabase detecta las palabras `drop policy if exists` en el SQL y te muestra esa advertencia **por precaución**, para avisarte si vas a borrar algo importante. Es una salvaguarda estándar.

## Por qué en TU caso es seguro

Las líneas `drop policy if exists ...` del SQL hacen esto:

```sql
drop policy if exists "open_select" on public.notas;
-- ...inmediatamente después...
create policy "open_select" on public.notas for select using (true);
```

Tres razones por las que no pasa nada malo:

1. **Estás en una tabla recién creada**: `notas` acaba de nacer en la línea anterior del script, no tiene políticas previas, así que el `drop` no borra nada.
2. **Cada `drop` va seguido del `create` equivalente**: incluso si hubiera políticas previas, se recrean al instante.
3. **La cláusula `if exists`** hace que si no existe la política, simplemente no haga nada (no falla).

El motivo de poner esos `drop` es para que **puedas volver a ejecutar el SQL las veces que quieras** sin errores (idempotente). Si lo ejecutas dos veces sin los `drop`, fallaría con "policy already exists".

## Qué hacer

👉 Pulsa **"Run this query"** (botón amarillo).

Deberías ver `Success. No rows returned`.

Después confirma en **Table Editor** (panel izquierdo) que aparece la tabla `notas` con 3 columnas: `id_pregunta`, `nota`, `updated_at`.

Cuando lo tengas, pasa al **Paso 3** de la guía: copiar `Project URL` y `anon key` desde Project Settings → API, y pegarlos en el `index.html` (líneas 1145-1146).

Avísame cuando esté hecho o si algo va raro.

---

**User**

Success. No rows returned

---

**Cursor**

¡Perfecto! "Success. No rows returned" es exactamente lo que tenía que salir. La tabla `notas` ya está creada en Supabase. 

Ahora te resuelvo las dudas que tenías:

---

## Aclaración IMPORTANTE: hay 3 URLs distintas, no las confundas

| URL | Para qué sirve | Dónde se usa |
|---|---|---|
| **Supabase URL** | Backend donde se guardan las notas (BBDD) | Dentro del `index.html` (constante `SUPABASE_URL`) |
| **GitHub URL** | Solo guarda el código fuente | Para commits, no sirve para abrir la app |
| **Vercel URL** | URL pública para abrir la app en móvil/PC | La compartes contigo mismo (móvil ↔ PC) |

👉 La que tú preguntabas es la **Supabase URL** (para que el código sepa a qué BBDD hablar) y luego la **Vercel URL** (para abrir la app desde el móvil).

---

## Pasos para probar con Supabase (de menos a más)

### ✅ PASO 1 — Tabla creada
Ya lo has hecho.

### 📋 PASO 2 — Verificar que la tabla existe

En Supabase: panel izquierdo → **Table Editor** (icono de cuadrícula) → deberías ver la tabla **`notas`** con 3 columnas: `id_pregunta`, `nota`, `updated_at`.

Si la ves, todo OK.

### 🔑 PASO 3 — Copiar las 2 credenciales de Supabase

En Supabase:

1. Engranaje de abajo a la izquierda → **Project Settings** → **API**.
2. Verás 2 valores a copiar:
   - **Project URL** → algo tipo `https://abcdefghij.supabase.co`
   - **Project API keys** → el que pone **`anon`** `public` → un token larguísimo que empieza por `eyJ...`
3. Cópialos a un bloc de notas temporal.

### ✏ PASO 4 — Pegarlos en `index.html`

Abre `index.html` en Cursor, pulsa **Ctrl + G**, escribe `1155` y ve a esa línea. Verás:

```1154:1157:index.html
    // Rellena estos 2 valores tras crear el proyecto en Supabase (ver SUPABASE_SETUP.md).
    // Si los dejas vacíos, las notas se guardan SOLO en este navegador (no sincroniza).
    const SUPABASE_URL = "";   // ej: "https://xxxxxxxx.supabase.co"
    const SUPABASE_KEY = "";   // anon/public key
```

Pégalos entre las comillas:

```js
const SUPABASE_URL = "https://abcdefghij.supabase.co";
const SUPABASE_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6...";
```

**Guarda el archivo** (Ctrl + S).

### 🧪 PASO 5 — Probar en local

1. Ve al navegador donde tienes la app corriendo (live-server).
2. **Ctrl + F5** (recarga forzada, sin caché).
3. Inicia un examen → contesta → click en **➕ Añadir nota** → escribe algo → click fuera.
4. Ahora el indicador en la esquina derecha del editor debe pasar de `"Solo local"` (dorado) a `"Sincronizada ✓"` (verde).
5. Verifica en Supabase: **Table Editor** → `notas` → deberías ver una fila nueva con tu nota.

Si llega a Supabase → **la parte backend está lista**.

---

## Ahora lo del "URL del servidor" para móvil ↔ PC

Para abrir la app desde el móvil necesitas **subir la app a Vercel** (no a GitHub, aunque Vercel se conecta a GitHub). Te lo explico en 3 pasos:

### OPCIÓN A — Subir a Vercel conectando con GitHub (recomendado)

1. Tu repo ya está en GitHub: `https://github.com/janorlex-dev/EE-examen-de-estado-API-CURSOR_V2`
2. Primero hay que **subir los últimos cambios** (notas + Supabase) a GitHub con un commit y push.
3. Entra en [https://vercel.com](https://vercel.com) → **Sign in with GitHub**.
4. **Add New → Project** → elige `EE-examen-de-estado-API-CURSOR_V2`.
5. En la configuración deja todo por defecto (es un sitio estático, Vercel lo detecta solo) → **Deploy**.
6. Al acabar te da una URL tipo `https://ee-examen-de-estado-api-cursor-v2.vercel.app`.
7. Esa URL la abres **en el móvil** y **en el PC** → ambas ven las mismas notas porque las dos hablan con la misma Supabase.

### OPCIÓN B — Solo probar móvil ↔ PC en red local (sin Vercel aún)

Mientras tienes `npx live-server` corriendo, te da una URL tipo `http://192.168.1.50:8080`. El móvil **conectado a la misma WiFi** puede abrir esa URL. Pero solo funciona en casa.

---

## Plan recomendado de acciones

1. **Ahora**: completa los PASOS 2-5 (pegar credenciales + probar local con Supabase). Dime cuando veas "Sincronizada ✓".
2. **Después**: hacemos commit + push a GitHub.
3. **Después**: conectamos Vercel para desplegar.
4. **Final**: abres la URL de Vercel en el móvil → creas una nota → la ves en el PC.

¿Vamos con el PASO 2? Mírate si la tabla `notas` aparece en el Table Editor y me confirmas.

---

**User**

en table editor no hay nada, en database esto de l aimagen

---

**Cursor**

¡Perfecto! La tabla está **creada correctamente**. Lo que ves es exactamente lo esperado:

- ✅ **"Success. No rows returned"** → el `CREATE TABLE` funcionó.
- ✅ **En Database** ves la tabla `notas` con las 3 columnas (`id_pregunta`, `nota`, `updated_at`).
- ✅ **En Table Editor está vacía** → normal, todavía no hay notas. Cuando añadas una desde la app, aparecerá la primera fila.

Ahora aclaro la confusión de las URLs (son **3 cosas distintas**) y te doy los pasos en orden.

---

## Las 3 URLs que se mezclan

| URL | Qué es | De dónde se saca | Para qué la usamos |
|---|---|---|---|
| **Supabase URL** | La BBDD en la nube donde viven las notas | Supabase → Settings → API | Se pega **dentro del `index.html`** para que la app sepa dónde guardar |
| **GitHub URL** | El repo del código fuente | `github.com/janorlex-dev/EE-examen-de-estado-API-CURSOR_V2` | Solo almacena el código. **No sirve tu app** al público. |
| **Vercel URL** | El hosting público de tu app | Vercel → Deployments | Es el enlace que compartes/usas en el móvil: `tu-app.vercel.app` |

**Regla fácil**:
- **Supabase** = tu base de datos (backend).
- **Vercel** = tu aplicación web (frontend).
- **GitHub** = donde vive el código para que Vercel lo despliegue.

---

## PASO A PASO para probar

### Paso 1 — Sacar la Supabase URL + anon key

1. En Supabase, panel izquierdo, icono **⚙ (Project Settings)** → **API**.
2. Ahí verás dos datos:

   - **Project URL**: algo como `https://abcdef1234567890.supabase.co`
   - **Project API keys** → fila **`anon` `public`** → clic en **Reveal** → copia el token largo `eyJhbGci...` (muy largo).

   ⚠ **NO uses** la `service_role` key — esa es secreta para el servidor. Solo queremos `anon / public`.

3. Pégame los dos valores aquí en el chat (la `anon` key no es peligrosa en este MVP, va en el HTML público) **o** si prefieres los pegas tú mismo en el archivo (te indico dónde abajo).

### Paso 2 — Pegarlos en `index.html`

Abre `index.html` y busca estas dos líneas (línea ~1148):

```1146:1149:index.html
    // Rellena estos 2 valores tras crear el proyecto en Supabase (ver SUPABASE_SETUP.md).
    // Si los dejas vacíos, las notas se guardan SOLO en este navegador (no sincroniza).
    const SUPABASE_URL = "";   // ej: "https://xxxxxxxx.supabase.co"
    const SUPABASE_KEY = "";   // anon/public key
```

Pega tus valores entre las comillas vacías:

```js
const SUPABASE_URL = "https://abcdef1234567890.supabase.co";
const SUPABASE_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOi...";
```

Guarda el archivo.

### Paso 3 — Probar en local

1. Recarga la página en el navegador (**Ctrl + F5** para forzar caché).
2. Inicia un examen, contesta una pregunta.
3. Haz clic en **➕ Añadir nota** → escribe algo → clic fuera del cuadro.
4. Fíjate en la esquina derecha del editor. Debe pasar de **Solo local** (dorado) a:
   - `Guardando…` (gris, medio segundo)
   - `Sincronizada ✓` (verde)

### Paso 4 — Verificar en Supabase

1. Vuelve a Supabase → **Table Editor** → `notas`.
2. Pulsa el icono de refrescar ↻.
3. Debería aparecer **1 fila** con tu `id_pregunta`, `nota` y `updated_at`.

### Paso 5 — Probar en móvil (DOS opciones)

**Opción 5.A — Misma red WiFi con `live-server` (la más rápida, sin Vercel):**

1. En tu PC, el servidor `npx live-server` ya está corriendo (lo veo en el terminal).
2. Averigua la IP de tu PC: en PowerShell ejecuta `ipconfig` → busca "IPv4 Address" (tipo `192.168.1.35`).
3. En el móvil, conectado al mismo WiFi, abre en el navegador: `http://192.168.1.35:8080` (el puerto que use live-server, normalmente `:8080`).
4. Ve a la misma pregunta → **verás la nota que creaste en el PC**.

**Opción 5.B — Desplegar en Vercel (público, funciona desde cualquier sitio):**

Esto lo explicamos cuando quieras. Es 5 minutos más y ya te queda una URL tipo `https://tu-app.vercel.app` que funciona desde cualquier red, no solo el WiFi de casa.

---

## ¿Cómo quieres seguir?

> **¿Por dónde seguimos?**
> - Te pego aquí la URL y anon key y las configuras tú en el index.html
> - Ya las tengo, las pego yo en el index.html y pruebo
> - Primero confígurame Supabase y luego guiame para desplegar en Vercel
> - No encuentro Project Settings → API, guíame con más detalle

---

**User**

https://kyedoeauobqqfcpkihfs.supabase.co  poyect url
sb_publishable_RZq0UKBRO74pPP0hCjxcYg_E4NwCe7s pulish key
postgresql://postgres:[YOUR-PASSWORD]@db.kyedoeauobqqfcpkihfs.supabase.co:5432/postgres direct conection string
supabase login
supabase init
supabase link --project-ref kyedoeauobqqfcpkihfs cli setup commands


---

**Cursor**

Perfecto, tengo lo que necesito. Aclaro lo que es útil y lo que no:

| Dato que has pegado | ¿Lo usamos? |
|---|---|
| **Project URL** `https://kyedoeauobqqfcpkihfs.supabase.co` | ✅ SÍ |
| **Publishable key** `sb_publishable_RZq0UKBRO74pPP0hCjxcYg_E4NwCe7s` | ✅ SÍ (es la nueva "anon key") |
| **Direct connection string** (postgresql://...) | ❌ NO (es para conectar servidores, no un HTML) |
| **Supabase CLI** (`supabase login`...) | ❌ NO (es para desarrollo local de BBDD, no para tu caso) |

Voy a pegar las credenciales en `index.html`:

Hecho. Ahora dos cosas importantes antes de probar:

## ⚠ Importante — Proteger la publishable key en git

La `sb_publishable_...` es pública por diseño (va en el navegador), pero igualmente es buena práctica no commitearla en el repo público de GitHub. Tienes dos opciones:

### Opción A — Aceptarlo tal cual (MVP rápido)
La `publishable key` está pensada para ir expuesta en el frontend. La seguridad depende del **Row Level Security** que activamos en el SQL. Para tu caso personal es aceptable.

### Opción B — Moverla a un archivo ignorado por git (más limpio)
Creamos `config.local.js` (ignorado por git) que define las claves, y lo cargamos antes del main script. Más trabajo pero más limpio para cuando el repo sea público.

De momento seguimos con Opción A para no bloquearnos. Ahora vamos a probar.

---

## PROBAR AHORA

### 1) Recarga forzada en el navegador
**Ctrl + F5** en la página donde tienes la app (la que sirve `live-server`).

### 2) Abre la consola del navegador
**F12** → pestaña **Console**. Déjala abierta mientras pruebas. Si algo falla aparecerá aquí.

### 3) Prueba el ciclo completo
1. Selecciona un examen → **Iniciar examen**.
2. Contesta una pregunta.
3. Clic en **➕ Añadir nota** → escribe algo → clic fuera del cuadro.
4. **Debe aparecer**:
   - Indicador `Guardando…` (gris, medio segundo)
   - Indicador `Sincronizada ✓` (verde)

### 4) Verifica en Supabase
1. Ve a Supabase → **Table Editor** → selecciona la tabla `notas`.
2. Pulsa **Refresh** (↻).
3. Deberías ver 1 fila con:
   - `id_pregunta`: p.ej. `comun_2018_1__q01`
   - `nota`: tu texto
   - `updated_at`: fecha actual

### 5) Si algo falla
- Si ves **"Error sync · guardada local"** en rojo → hay problema de permisos/red. Copia el error de la Console y me lo pegas.
- Si sigue poniendo **"Solo local"** → no ha cargado la config. Verifica que Ctrl+F5 refrescó.
- Si ves en la Console algo como `JWSError` o `invalid api key` → la key copiada mal (espacios/guiones de más).

---

¿Cómo va? Pruébalo y cuéntame qué aparece en el indicador cuando escribes tu primera nota.

---

**User**

he creado un .gitignore para que n se suba lo que ahi escriba

---

**Cursor**

⚠ Hay un problema importante. Te explico lo que encuentro.

## El problema

Has creado `.gitignore` como **carpeta**, pero en git eso no funciona. `.gitignore` debe ser un **archivo** (con una lista de patrones dentro). 

Lo confirmo con dos pruebas:

1. `git check-ignore .gitignore/pasw_janorlex.md` devuelve código 1 → **NO está siendo ignorado**.
2. `git status` muestra:
   ```
   M index.html
   ?? .gitignore/       ← carpeta sin rastrear (pero se subiría si haces `git add .`)
   ?? SUPABASE_SETUP.md
   ```

El `??` significa "archivo nuevo, se subirá al próximo commit si lo añades". **Si haces `git add . && git push` ahora mismo, tu `pasw_janorlex.md` se subiría a GitHub junto con tus contraseñas**.

## Por qué pasa

En Windows NTFS una carpeta y un archivo no pueden tener el mismo nombre en la misma ruta. Como ya tienes la **carpeta** `.gitignore`, no puedes crear un **archivo** `.gitignore` ahí al lado. Hay que renombrar la carpeta.

## La solución

Renombramos la carpeta de secretos a un nombre claro (p.ej. `.secretos/`) y creamos el archivo `.gitignore` real que ignore esa carpeta + otras cosas sensibles.

### Propuesta concreta

1. **Renombrar** `.gitignore\` → `.secretos\` (con tu `pasw_janorlex.md` dentro).
2. **Crear** el archivo `.gitignore` con:
   ```
   # Carpeta privada con credenciales, contraseñas, notas personales
   .secretos/

   # IDE / editores
   .vscode/
   .idea/

   # Node (por si añades dependencias)
   node_modules/
   npm-debug.log
   package-lock.json

   # Archivos del sistema
   Thumbs.db
   .DS_Store
   desktop.ini

   # Logs
   *.log

   # Variables de entorno (si las uso mañana)
   .env
   .env.local
   ```
3. **Verificar** con `git check-ignore -v .secretos/pasw_janorlex.md` que ya está ignorado.

---

> **¿Procedo a renombrar la carpeta y crear el .gitignore real?**
> - Sí, renombra a .secretos/ y crea el .gitignore con los patrones propuestos ✔
> - Sí, pero usa otro nombre para la carpeta (te lo digo)
> - No renombres, solo quiero ver el contenido propuesto para copiarlo yo
> - Explícame más cómo funciona .gitignore antes de decidir

Voy al lío.

Ahora verifico que funciona:

Todo funciona correctamente:

## Verificación

```
.gitignore:4:.secretos/	.secretos/pasw_janorlex.md
```
↑ Git confirma que la regla de la línea 4 del `.gitignore` está **ignorando correctamente** tu archivo de contraseñas.

Y el `git status` ahora muestra:

```
 M index.html           ← modificado (credenciales Supabase)
?? .gitignore           ← nuevo (el .gitignore REAL como archivo)
?? SUPABASE_SETUP.md    ← nuevo (documentación)
```

**La carpeta `.secretos/` YA NO aparece** → perfecto, está oculta a git. ✅

---

## Resumen de lo que hay ahora

| Elemento | Estado | Se sube a GitHub |
|---|---|---|
| `.secretos/pasw_janorlex.md` | Carpeta renombrada, con tus contraseñas dentro | ❌ NO (ignorada) |
| `.gitignore` | Archivo nuevo con las reglas | ✅ SÍ (debe subirse) |
| `index.html` | Modificado (notas + Supabase) | ✅ SÍ |
| `SUPABASE_SETUP.md` | Doc nueva | ✅ SÍ |
| `examenes/*.csv` | Sin cambios | ya estaban subidos |

## Sobre las credenciales Supabase en `index.html`

Están pegadas ahí:
- `SUPABASE_URL = "https://kyedoeauobqqfcpkihfs.supabase.co"`
- `SUPABASE_KEY = "sb_publishable_..."`

La `publishable key` está **pensada para ir en el navegador** (es pública por diseño). La seguridad la garantizan las políticas RLS que creamos en el SQL. Para tu caso personal es correcto. Si algún día quieres moverla a un archivo `.gitignored`, te lo puedo preparar en 5 min — pero no es urgente.

---

## Y ahora — ¿probamos Supabase antes del commit?

Te recuerdo los pasos:

1. **Ctrl + F5** en la app del navegador (para recargar con las credenciales nuevas).
2. Abre **F12 → Console** para ver si hay errores.
3. Inicia un examen, contesta una pregunta, **➕ Añadir nota** → escribe algo → clic fuera.
4. Indicador debe pasar de `Guardando…` (gris) a **`Sincronizada ✓`** (verde).
5. Vuelve a Supabase → **Table Editor** → `notas` → Refresh → aparece 1 fila.

Cuando confirmes que sincroniza, hacemos el commit con mensaje tipo:

```
feat: notas por pregunta con Supabase + .gitignore
```

¿Pruebas y me cuentas?

---

**User**

102Tracking Prevention blocked access to storage for <URL>.
11jbm0fsk6t~_.js?dpl=dpl_b6dG3bvN8sh5pcuPeXKj7jrAVy42:13 Supabase Studio is running commit 9c716a6a483f098649d2cd1d1ca3721357d592c1 deployed at 2026-04-20 21:45:22 +02:00.
0i8wy8tbht2ow.js?dpl=dpl_b6dG3bvN8sh5pcuPeXKj7jrAVy42:2 [Intervention] Images loaded lazily and replaced with placeholders. Load events are deferred. See https://go.microsoft.com/fwlink/?linkid=2048113
11jbm0fsk6t~_.js?dpl=dpl_b6dG3bvN8sh5pcuPeXKj7jrAVy42:13 [Warning] Please install/enable Redux devtools extension

---

**Cursor**

Esos mensajes son del **Supabase Studio** (la web del panel de Supabase donde creaste la tabla), no de tu app. Te lo confirmo por estas pistas:

- `Supabase Studio is running commit...` → es la consola de la página de Supabase.
- `Please install/enable Redux devtools` → Supabase Studio usa Redux, tu app no.
- `Tracking Prevention blocked...` → advertencia del navegador, no error.

**Ninguno de esos mensajes indica problema**. Pero tampoco son de tu app, así que no nos sirven para saber si la sincronización funciona.

## Hay que mirar la consola DE TU APP

1. Cambia a la pestaña del navegador donde tienes **la app de test** (la que abriste con `live-server`, URL tipo `http://127.0.0.1:5500` o `http://localhost:8080`).
2. En **esa pestaña** pulsa **F12** → pestaña **Console**.
3. Ahí debería salir solo algún mensaje tipo:
   ```
   [vite] connected.   (o similar de live-server)
   ```
   Si aparece algo en rojo (error), cópialo.

## Prueba la sincronización

Con la consola de la app abierta:

1. **Ctrl + F5** para recargar con las credenciales Supabase.
2. Selecciona un examen → **Iniciar examen**.
3. Contesta una pregunta.
4. Clic en **➕ Añadir nota** → escribe "prueba sync" → clic fuera.
5. **Observa el indicador** arriba a la derecha del cuadro de nota:
   - ¿Pone **"Solo local"** (dorado)?
   - ¿Pone **"Guardando…"** (gris)?
   - ¿Pone **"Sincronizada ✓"** (verde)?
   - ¿Pone **"Error sync · guardada local"** (rojo)?

## Verifica en Supabase

1. Ve al **Supabase Studio** → **Table Editor** (menú izquierdo) → selecciona tabla **`notas`**.
2. Pulsa el botón **Refresh** ↻ arriba.
3. ¿Aparece una fila con tu nota?

---

Dime:
- **(a)** Qué indicador sale en la esquina derecha del cuadro de nota.
- **(b)** Si aparece algo en la consola de TU APP (F12 sobre la pestaña de `localhost`).
- **(c)** Si se ve la fila en Table Editor de Supabase.

Con esos 3 datos sé si está funcionando o qué hay que arreglar.

---

**User**

si las veo 

---

**Cursor**

¡Excelente! 🎉 Sincronización funcionando:

- ✅ Creas nota en PC → se guarda en Supabase.
- ✅ Aparece la fila en Table Editor.

## Falta la prueba crítica: móvil ↔ PC

Para confirmar que la sincronización cruzada funciona:

### Prueba en otro dispositivo (móvil o el mismo PC en otro navegador)

**Opción rápida — mismo WiFi:**

1. En tu PC, averigua la IP local. Abre PowerShell:

Tu IP local es `192.168.137.16`. Live-server usa el puerto **8080**.

Para el móvil: `http://192.168.137.16:8080`

⚠ Nota: el rango `192.168.137.x` suele ser de **red compartida/hotspot** de Windows. Si el móvil y el PC están en **el mismo WiFi doméstico** normal, es posible que no se vean. Si falla, el PC tendrá una IP tipo `192.168.1.x` o `192.168.0.x` (cuando estás en tu WiFi de casa).

## La prueba más rápida (sin salir del PC)

Más fácil que ir al móvil: **abre OTRO navegador** en el mismo PC (si usas Edge, abre Chrome; o una ventana **Incógnito**). Esto simula otro dispositivo porque no comparte `localStorage`.

### Pasos

1. Si `live-server` sigue corriendo, abre otra ventana/navegador en: `http://127.0.0.1:8080`
2. Si no corre, relánzalo:

---

**User**

vamos a hacer que corra con la url de vercel

---

**Cursor**

Para que corra con URL de Vercel hay 3 pasos:

1. **Commit + push** del estado actual a GitHub (Vercel despliega desde ahí).
2. **Conectar el repo a Vercel** (web UI, 2 min).
3. **Obtener la URL pública** y probar desde el móvil.

Primero compruebo el estado y te pregunto antes de hacer commit.

---

**User**

como se en qeu cuenta de git esta 

añade las instrucciones nuevas qeu hemos creado con la integracion de la creacion de notas en en rules_intrucciones_para _desarroolo_api

---

**Cursor**

Añado una sección nueva al final con todo lo que hemos implementado, en el mismo estilo del resto de la regla.

Hecho. Añadida una sección completa al final de `.cursor/rules/.rules_instrucciones_para_desarrollo_API.mdc` con todo lo que hemos construido.

## Qué he documentado

### 1. Flujo UX obligatorio (3 estados + repaso readonly)
- Sin responder → nada.
- Respondida sin nota → `➕ Añadir nota`.
- Respondida con nota → banner + `✏ Editar nota`.
- Repaso → solo lectura.

### 2. Regla de identificadores
- Formato: `slug(examName) + "__q" + NN` (ej. `materias_comunes_2025.2__q01`).
- `_origIdx` siempre del CSV original, no de la posición aleatoria.
- Prohibido usar texto de pregunta como clave.
- Numeración 1-based con 2 dígitos.

### 3. Arquitectura de datos (3 capas)
- `notasCache` en memoria (fuente de verdad en sesión).
- `localStorage` (respaldo offline).
- Supabase (sync entre dispositivos).
- Orden de escritura y de lectura bien definido.

### 4. Esquema Supabase
- Tabla `notas` con 3 columnas.
- RLS abierto para MVP.
- Camino de migración a multiusuario (añadir `user_id`).

### 5. Separación de secretos
- `.secretos/` ignorada por git.
- `.gitignore` con reglas para `.env*`, PDFs, etc.
- Obligación de verificar con `git check-ignore -v`.

### 6. Reglas para futuros cambios
- No romper el flujo UX de 3 estados.
- Toda escritura pasa por `guardarNota()`.
- Toda lectura desde `notasCache`.
- Si se cambia la forma del ID → migración obligatoria (los IDs escritos son contrato).
- Nunca exponer `service_role` key.

### 7. Checklist del agente
Lista de 7 preguntas que cualquier futuro agente (o tú mismo en otra sesión) debe validar antes de tocar este módulo.

---

Esto asegura que en futuras sesiones, cuando Cursor lea las reglas, tendrá todo el contexto del módulo de notas y no romperá el diseño ni las invariantes importantes (clave de pregunta, fallback local, RLS, etc.).

¿Quieres que sigamos con **commit + push + Vercel** ahora, o prefieres descansar y lo retomas después?

