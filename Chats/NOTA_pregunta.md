# NOTA pregunta
_Exported on 4/20/2026 at 14:17:12 GMT+2 from Cursor (2.6.21)_

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

