# Configuración Supabase — Progreso de exámenes

Esta guía añade la tabla `progreso` al mismo proyecto Supabase que ya tienes
para `notas` y `enlaces`. Reutiliza la misma `SUPABASE_URL` y `SUPABASE_KEY`
que están en `index.html`, no hay que añadir credenciales nuevas.

## Qué hace este módulo

Permite ver de un vistazo en el dashboard qué exámenes tienes pendientes,
en progreso o finalizados, y sincroniza ese estado entre PC y móvil.

| Estado | Indicador visual | Al hacer click en el badge del año |
|---|---|---|
| **No empezado** | (sin dot) | Abre la pantalla de configuración normal |
| **En progreso** | 🟡 dot amarillo + franja "Seguir practicando" arriba | Modal: *Continuar (pregunta X de N) · Empezar de nuevo* |
| **Finalizado** | 🔴 dot rojo | Borra el estado y abre la pantalla de configuración para rehacerlo |

No se guarda historial de intentos, ni fechas, ni puntuaciones. Solo el
estado actual de cada examen y, si está en curso, dónde te quedaste.

---

## 1. Crear la tabla `progreso`

1. En Supabase abre **SQL Editor** → **New query**.
2. Pega este SQL completo y pulsa **Run** (▶):

```sql
create table if not exists public.progreso (
  exam_id         text primary key,
  estado          text not null check (estado in ('en_curso','finalizado')),
  pregunta_actual int default 0,
  respuestas      jsonb default '{}'::jsonb,
  falladas        jsonb default '[]'::jsonb,
  modo            text,
  origen_idxs     jsonb default '[]'::jsonb,
  timer_total     int default 0,
  timer_sec       int default 0,
  updated_at      timestamptz not null default now()
);

create index if not exists progreso_updated_at_idx on public.progreso (updated_at desc);
create index if not exists progreso_estado_idx on public.progreso (estado);

alter table public.progreso enable row level security;

drop policy if exists "progreso_open_select" on public.progreso;
drop policy if exists "progreso_open_insert" on public.progreso;
drop policy if exists "progreso_open_update" on public.progreso;
drop policy if exists "progreso_open_delete" on public.progreso;

create policy "progreso_open_select" on public.progreso for select using (true);
create policy "progreso_open_insert" on public.progreso for insert with check (true);
create policy "progreso_open_update" on public.progreso for update using (true) with check (true);
create policy "progreso_open_delete" on public.progreso for delete using (true);
```

Deberías ver `Success. No rows returned`.

3. Verifica que la tabla existe: **Table Editor** → `progreso` (10 columnas).

---

## 2. Probar

1. Abre `index.html`, selecciona cualquier examen y contesta 2-3 preguntas.
2. Pulsa `← Volver` o `✕ Salir` → vuelve al dashboard.
3. Debe aparecer:
   - Franja amarilla **🟡 Seguir practicando** con el examen y `3/50`.
   - Dot amarillo 🟡 en el badge del año dentro de su materia.
4. Pulsa el badge → se abre el modal con:
   - **Continuar →** retoma en la misma pregunta con el tiempo restante.
   - **Empezar de nuevo** borra el progreso y abre la configuración.
5. Termina un examen (`Finalizar ✓`) → el dot pasa a rojo 🔴.
6. Pulsa un badge rojo → entra directo a la configuración, sin modal.
7. Abre la app en el móvil o en otro navegador → debes ver los mismos dots.

---

## 3. Qué se guarda en cada fila

| Campo | Tipo | Qué contiene |
|---|---|---|
| `exam_id` | text PK | `slug(examName)`, ej: `materias_comunes_2025.2`, `laboral_2024.1` |
| `estado` | text | `'en_curso'` o `'finalizado'` |
| `pregunta_actual` | int | Índice 0-based de la pregunta donde se quedó |
| `respuestas` | jsonb | `{ "0": "a", "1": "c", ... }` respuestas dadas por índice de test |
| `falladas` | jsonb | Array con índices de preguntas marcadas como error |
| `modo` | text | `'completo'` o `'aleatorio'` |
| `origen_idxs` | jsonb | Orden de las preguntas respecto al CSV original — permite reconstruir tests aleatorios al continuar |
| `timer_total` | int | Segundos configurados al iniciar el examen |
| `timer_sec` | int | Segundos restantes al guardar (para retomar el cronómetro) |
| `updated_at` | timestamptz | Última escritura |

Cuando un examen se finaliza, la fila se reemplaza por
`{ estado: 'finalizado' }`: se borran respuestas, falladas y timer. Si
reinicias un finalizado (o eliges "Empezar de nuevo" en un en curso), la
fila se borra y queda como no empezado hasta que vuelvas a contestar.

---

## 4. Inspeccionar el progreso en Supabase

```sql
select exam_id, estado, pregunta_actual,
       jsonb_array_length(falladas) as n_falladas,
       timer_sec, updated_at
from public.progreso
order by updated_at desc;
```

Borrar todo el progreso manualmente (como si nunca hubieras empezado nada):

```sql
delete from public.progreso;
```

Borrar solo los finalizados (para reiniciar visualmente el dashboard):

```sql
delete from public.progreso where estado = 'finalizado';
```

---

## 5. Arquitectura (alineada con `notas` y `enlaces`)

Capas de datos:

1. `progresoCache` en memoria → fuente de verdad durante la sesión.
2. `localStorage["norlexia_progreso_v1"]` → espejo JSON offline.
3. Tabla `public.progreso` en Supabase → sincronización PC ↔ móvil.

Orden de escritura en `guardarProgreso(examId, data)`:

1. Actualizar `progresoCache` (síncrono).
2. Persistir `localStorage` (síncrono).
3. Encolar en `_progPendingSync` y lanzar `scheduleProgresoSync()` con
   debounce de ~900 ms (agrupa varias escrituras seguidas en un solo
   `upsert` por examen).

Orden de lectura en `initProgreso()` al arrancar:

1. Cargar `localStorage` → `progresoCache`.
2. Si hay cliente Supabase, descargar todas las filas y sobreescribir la
   cache (Supabase prevalece en caso de conflicto).
3. Persistir el resultado fusionado en `localStorage`.

Funciones públicas del módulo (en `index.html`):

- `initProgreso()` — fusiona local + remoto al cargar la app.
- `getProgreso(examId)` — lectura síncrona desde cache.
- `guardarProgreso(examId, data)` — snapshot completo de un examen en curso.
- `borrarProgreso(examId)` — elimina la fila (cache, local y Supabase).
- `marcarFinalizado(examId)` — reemplaza por `{ estado: 'finalizado' }`.
- `snapshotProgreso()` — helper que toma el estado actual del examen
  (variables `cur`, `answered`, `failedIdxs`, `timerSec`, etc.) y llama
  a `guardarProgreso`.
- `continuarExamen(idx)` — reanuda sin pasar por la pantalla de
  configuración, restaurando `questions`, `cur`, `answered`, `failedIdxs`,
  contadores y cronómetro.

Puntos del código donde se llama:

- `document.addEventListener("DOMContentLoaded")` → `initProgreso()`.
- `selectExam(i)` → si `en_curso` abre modal, si `finalizado` borra fila.
- `answer()`, `repeatQ()`, `goNext()`, `goPrev()` → `snapshotProgreso()`.
- `showDash()` → `snapshotProgreso()` antes de salir si estabas en un examen.
- `finishExam()` → `marcarFinalizado(examIdFromExam(ex))`.

---

## 6. Reglas para futuras modificaciones

- Toda escritura pasa por `guardarProgreso` / `marcarFinalizado` /
  `borrarProgreso`. No toques `progresoCache` ni `localStorage`
  directamente desde la UI.
- Toda lectura pasa por `getProgreso(examId)`. No consultes Supabase
  síncronamente desde el render del dashboard.
- El ID de examen es `slugify(ex.name)`. Si cambias el `name` de un CSV
  en `CSV_MANIFEST`, el progreso previo queda huérfano (misma política
  que las notas). Si necesitas renombrar en producción, provee una
  migración SQL que actualice `exam_id`.
- Si añades multiusuario (login), añade columna `user_id uuid` con FK a
  `auth.users`, sustituye las políticas abiertas por
  `using (auth.uid() = user_id)` y actualiza `exam_id` a `(user_id, exam_id)`
  como clave compuesta. La API `guardarProgreso` se mantiene igual.
- Si cambias la forma de guardar `origen_idxs` (por ejemplo para soportar
  modo por bloques con subconjuntos), asegúrate de que
  `continuarExamen()` sabe reconstruir `questions` correctamente a partir
  del CSV actual.

---

## 7. Rollback

Borrar la tabla entera:

```sql
drop table if exists public.progreso cascade;
```

Y vuelve a ejecutar el bloque del paso 1 si quieres recrearla.
