# Lectura de archivos .rule en cursor
_Exported on 4/12/2026 at 22:10:55 GMT+2 from Cursor (2.6.21)_

---

**User**

como se hece en cursor que lea los .rule las instrucciones cada vez que se ccarga el proyecto , ocomo se hace ?

---

**Cursor**


Revisando la estructura del proyecto y la documentación de reglas de Cursor para darte una respuesta precisa.




## Cómo hace Cursor que se “lean” instrucciones al abrir el proyecto

Cursor **no busca automáticamente** una carpeta llamada `.rule` ni archivos `.md` sueltos con ese nombre. Lo que sí reconoce de forma estándar son las **reglas del proyecto** bajo **`.cursor/rules/`**.

### Qué debes usar

1. **Crea** la carpeta **` .cursor/rules/ `** en la raíz del workspace (si no existe).
2. **Añade** archivos de regla en formato **`.mdc`** (Markdown con frontmatter YAML al inicio).
3. Para que esas instrucciones entren **en prácticamente todas las conversaciones** con el agente en ese repo, pon en el frontmatter:

```yaml
---
description: Instrucciones generales del proyecto
alwaysApply: true
---
```

El contenido Markdown va **debajo** del frontmatter.

- **`alwaysApply: true`**: la regla se considera **siempre aplicable** al trabajar en ese proyecto (equivalente a “cárgalo siempre”).
- **`alwaysApply: false`** con **`globs`**: solo cuando trabajas con archivos que coinciden con el patrón (por ejemplo `**/*.ts`).

### Tu caso concreto (`.rule/`)

Tienes cosas como `.rule\.rules_instrucciones_para_desarrollo_API.md`. Eso **no es** la convención de Cursor para inyección automática: **hay que migrar o duplicar** el contenido (o la parte relevante) a **`.cursor/rules/*.mdc`** con `alwaysApply: true` si quieres el mismo efecto “global”.

### Resumen

| Ubicación | ¿Cursor lo usa solo al abrir el proyecto? |
|-----------|-------------------------------------------|
| `.rule/*.md` (carpeta propia) | No, salvo que tú referencies el archivo o lo copies a reglas oficiales |
| `.cursor/rules/*.mdc` con `alwaysApply: true` | Sí, es la forma prevista de “instrucciones persistentes” del repo |

No hace falta ningún ajuste extra en “cargar el proyecto”: al abrir el workspace, Cursor enlaza esas reglas del proyecto con el agente según `alwaysApply` y `globs`.

Si quieres que te ayude a **convertir** tus `.md` de `.rule/` a `.mdc` en `.cursor/rules/`, activa **modo Agent** y lo podemos hacer en el repo.

---

**User**

hazlo

---

**Cursor**


Leyendo los archivos en `.rule/` y creando reglas en `.cursor/rules/`.




Creando `.cursor/rules/` con un archivo `.mdc` consolidado que incluye ambos documentos (el segundo ya está contenido en el primero; eliminamos duplicados) y formato Markdown claro.




Listo. Creé la regla de proyecto que Cursor cargará automáticamente en este workspace.

## Qué se hizo

- **Archivo nuevo:** `.cursor/rules/examen-abogacia-estado.mdc`
- **Frontmatter:** `alwaysApply: true` y `description` para que aparezca en el selector de reglas.
- **Contenido:** Unificación de `.rule\.rules_instrucciones_para_desarrollo_API.md` y `.rule\.rules para el agente IA.md` (el segundo ya estaba casi todo en el primero), en Markdown con secciones claras: contexto, funcionalidades, arquitectura, CSV, pantallas, paleta, MVP vs v2, y cómo debe responder el agente.

Los archivos en `.rule/` **siguen ahí** como copia de referencia; la fuente que Cursor usa por defecto es **`.cursor/rules/*.mdc`**.

Al abrir el proyecto de nuevo o en nuevos chats del agente, esta regla debería aplicarse de forma continua. Si no la ves, revisa en Cursor **Settings → Rules** (o el panel de reglas del proyecto) que las reglas del repo estén activadas.

---

**User**

en  \.cursor\rules\examen-abogacia-estado.mdc me has añadido todas las instrucciones qeu he puesto en los dos archivos , uno es la estructura de proyecto y otra son las instrucciones para ti  el agente , lo veo muuy simplificado , solo dime brevemente

---

**Cursor**


No: **no está todo literal ni al mismo nivel de detalle** que los dos `.md` originales.

Hice una **regla única condensada**: mismas **ideas** (MVP, dashboard, examen, resultados, CSV, paleta, footer, arquitectura, cómo responder el agente, verificación, fuera de MVP), pero **muchas viñetas concretas** (pantallas línea a línea, duplicados entre archivos, matices repetidos del PDF→CSV, etc.) quedaron **resumidas o fusionadas** para que el `.mdc` fuera manejable y sin repetición.

Si quieres que el agente tenga **el 100 % del texto** de ambos archivos, habría que **pegáis el contenido completo** en una o más reglas `.mdc` (o partir en 2–3 `.mdc` temáticos) sin resumir tanto. En modo Agent puedo hacerlo si lo pides.

---

**User**

renombro los dos .md a .mdc y los paaso a .cursor\rules ? estaria bien

---

**Cursor**


**Casi.** Renombrar a `.mdc` y moverlos a `.cursor/rules` es el sitio correcto, pero **no basta con cambiar la extensión**.

Para que Cursor los trate bien como reglas de proyecto, cada `.mdc` debe empezar con **frontmatter YAML**, por ejemplo:

```yaml
---
description: Breve título de qué cubre esta regla
alwaysApply: true
---
```

(sustituye `alwaysApply` por `false` y añade `globs: ...` si solo quieres que aplique al abrir ciertos archivos.)

**Resumen:** Sí, puedes tener **dos** `.mdc` en `.cursor/rules` (estructura del proyecto en uno, instrucciones del agente en otro). Solo asegúrate de **insertar ese bloque `---` al inicio** en cada archivo; el resto puede ser tu Markdown tal cual.

Si dejas el `.mdc` **sin** frontmatter, puede que se cargue de forma rara o no aparezca como regla “oficial” en la UI.

---

**User**

ya los he movido , compueba si cumplen lo qeu hemos dicho de leerlas cuando se abre el proyecto , ambos archivos

---

**Cursor**


Revisando los archivos en `.cursor/rules/` para comprobar formato y frontmatter.




## Comprobación

### Lo que está bien
- **Ambos archivos** tienen **frontmatter válido** al inicio con `description` y **`alwaysApply: true`** (líneas 1–4 en cada uno). Eso es lo que pedíamos para que la regla se considere **siempre aplicable** al trabajar en el proyecto.
- Están en **`.cursor/rules/`**, que es la carpeta correcta.

### Punto a revisar (importante)
- Siguen siendo **`.md`**, no **`.mdc`**. En la documentación de Cursor las reglas de proyecto suelen definirse como **`.mdc`**. Es posible que el IDE **sí** los indexe igual, pero para alinearte con lo acordado y evitar sorpresas, lo más seguro es **renombrarlos a `.mdc`** (por ejemplo `instrucciones-desarrollo-api.mdc` y `reglas-agente-ia.mdc`), manteniendo el mismo contenido y el mismo frontmatter.

### Efecto práctico
- Con **`alwaysApply: true` en los dos**, Cursor intentará aplicar **las dos reglas** en cada contexto donde cuenten las reglas del repo (más tokens y algo de solapamiento si el contenido se repite). Si notas ruido o límite de contexto, podrías unificar en un solo `.mdc` o dejar una regla con `alwaysApply: false` y `globs` solo para ciertos archivos.

**Resumen:** Cumplen la parte de **“leerlas al usar el proyecto”** por el **frontmatter + `alwaysApply: true`**. Lo único que te recomiendo ajustar por compatibilidad con lo hablado: **pasar la extensión a `.mdc`**. Si quieres que lo haga yo en el repo, activa modo Agent.

---

**User**

ya estan renombrados , si los uno y especifico reglas de diseño para el desarrollo y reglas para el agente ed ia 

---

**Cursor**


Sí, **puedes unirlos en un solo `.mdc`**: un solo frontmatter al inicio y el resto en secciones claras. Así evitas duplicar `alwaysApply: true` y reduces solapamiento entre archivos.

## Cómo organizarlo

1. **Un archivo**, por ejemplo `examen-abogacia-reglas.mdc`.

2. **Frontmatter una sola vez** (arriba del todo):

```yaml
---
description: Examen de Estado — diseño del producto y normas para el agente
alwaysApply: true
---
```

3. **Dos bloques grandes** con títulos que no se pisen, por ejemplo:

- `## Reglas de diseño y producto (desarrollo)` — MVP, pantallas, CSV, paleta, arquitectura, stack, footer, etc.
- `## Reglas para el agente de IA` — criterios de calidad, formato de respuesta (paso a paso, qué archivos tocar), verificación, prioridades, cuándo preguntar.

4. **Orden recomendado:** primero **producto/diseño** (qué construir), después **agente** (cómo debe trabajar). Así el modelo primero entiende el contexto y luego el estilo de trabajo.

5. **Qué hacer con los dos `.mdc` actuales:** tras copiar todo al unificado, **borra o archiva** los dos antiguos para no tener **tres** reglas con `alwaysApply: true` cargando el mismo contenido dos veces.

Si quieres que te deje el archivo unificado en el repo, cambia a **modo Agent** y lo aplicamos sobre tus textos actuales.

---

**User**

ya esta comprueba y solo marcame lo que eliminariamos para que no este duplicado 

---

**Cursor**


Revisando el contenido actual de `.cursor/rules/` para localizar duplicados.




He leído `.cursor/rules/.rules_instrucciones_para_desarrollo_API.mdc` completo. Esto es lo que **duplica** contenido o **sobra** y conviene quitar o unificar.

---

### 1. Error grave: segundo frontmatter (líneas 231-234)

Hay un bloque repetido:

```text
---
description: ...
alwaysApply: true
---
```

en medio del archivo, **después** del contenido de diseño. Eso **no debe estar ahí**: las reglas de Cursor solo llevan **un** frontmatter al **inicio** (líneas 1-4). **Elimínalo por completo** (líneas 231-234).

---

### 2. Duplicado exacto: bloque “agente” en dos sitios

Está **dos veces** el mismo texto:

| Ubicación | Líneas aprox. |
|-----------|----------------|
| Dentro de “Reglas de diseño y producto” | **71-103** (desde `CRITERIOS DE CALIDAD` hasta `INSTRUCCIÓN DE TRABAJO`) |
| En “Reglas para el agente de IA” | **239-272** |

**Qué hacer:** dejar **solo una** copia, la que va bajo `## Reglas para el agente de IA`. **Borra 71-103** del primer bloque (o borra 239-272 si prefieres el orden inverso; lo lógico es mantener la sección “agente” al final y vaciar el duplicado del principio).

---

### 3. Repeticiones de producto / diseño (unificar en un solo sitio)

Mismo documento, mismo significado repetido:

| Tema | Dónde se repite | Nota |
|------|------------------|------|
| **Formato CSV** (cabecera + ejemplo) | **67-69**, **146-148**, y otra variante **227-229** (`materia` vs `bloque`) | Dejar **una** subsección “Formato CSV” con las **dos** variantes de columnas en una sola tabla o dos líneas. |
| **Pantalla 1 — Dashboard** (título, cards, modos, cargar CSV) | **111-119** y otra vez **200-205** (muy similar) | Fusionar en un solo listado de dashboard. |
| **Pantalla 2 — Examen** (cabecera, barra, opciones, hover/click, botones) | **121-130** y otra vez **213-219** | Quedarte con un solo bloque “Pantalla 2”. |
| **Pantalla 3 — Resultados** | **131-135** y parte de **221-223** | Unificar en un solo bloque “Pantalla 3”. |
| **Lógica de falladas** | **137-142** y **219-221** | Un solo bloque “Lógica de falladas”. |
| **Botón Cargar CSV / exámenes en carpeta** | **105-110** y contexto en **179**, **157** | Se puede condensar en un párrafo sin repetir la misma idea tres veces. |

No hace falta borrar *todo* el segundo bloque de cada pare si prefieres dejar “resumen + detalle”; lo importante es **no tener la misma lista entera dos veces**.

---

### 4. Línea 236 vacía y título

Tras quitar el frontmatter falso, `## Reglas para el agente de IA` quedará pegado al contenido; revisa que no queden `---` sueltos que confundan el parser.

---

**Resumen:** Lo imprescindible es **eliminar el segundo frontmatter (231-234)** y **una de las dos copias del bloque agente (71-103 o 239-272)**. Lo demás es **deduplicar** CSV, pantallas 1-3 y falladas dentro de la parte de diseño. Si quieres que lo aplique yo en el archivo, activa modo Agent y lo edito.

