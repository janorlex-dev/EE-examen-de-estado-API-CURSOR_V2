# Configuración Supabase — Notas por pregunta

Esta guía te lleva de cero a sincronización móvil ↔ PC en ~5 minutos.

---

## 1. Crear el proyecto en Supabase

1. Entra en [https://supabase.com](https://supabase.com) y haz **Sign in** (puedes usar GitHub).
2. Pulsa **New project**.
3. Rellena:
   - **Name**: `norlexia-test-abogacia` (o el que quieras).
   - **Database Password**: genera una segura y guárdala en un sitio seguro (no se usa en el HTML, solo para administrar).
   - **Region**: `West EU (Ireland)` o la más cercana a ti.
   - **Pricing plan**: `Free`.
4. Pulsa **Create new project** y espera ~1 min a que se aprovisione.

---

## 2. Crear la tabla `notas`

1. En el panel izquierdo abre **SQL Editor** → **New query**.
2. Pega este SQL completo y pulsa **Run** (▶):

```sql
-- Tabla principal
create table if not exists public.notas (
  id_pregunta text primary key,
  nota        text not null default '',
  updated_at  timestamptz not null default now()
);

-- Índice por fecha (para futuras estadísticas)
create index if not exists notas_updated_at_idx on public.notas (updated_at desc);

-- Activar Row Level Security
alter table public.notas enable row level security;

-- Políticas abiertas (MVP compartido sin login)
-- ⚠ Esto permite a cualquiera con la anon key leer/escribir tus notas.
-- Para uso personal en tus dispositivos es aceptable. Cuando añadas login,
-- sustituye estas políticas por filtros por user_id.
drop policy if exists "open_select" on public.notas;
drop policy if exists "open_insert" on public.notas;
drop policy if exists "open_update" on public.notas;
drop policy if exists "open_delete" on public.notas;

create policy "open_select" on public.notas for select using (true);
create policy "open_insert" on public.notas for insert with check (true);
create policy "open_update" on public.notas for update using (true) with check (true);
create policy "open_delete" on public.notas for delete using (true);
```

Deberías ver `Success. No rows returned`.

3. Verifica que la tabla existe: panel izquierdo → **Table Editor** → debería aparecer `notas` con 3 columnas.

---

## 3. Copiar credenciales al `index.html`

1. En el panel izquierdo abre **Project Settings** (icono engranaje) → **API**.
2. Copia:
   - **Project URL** → algo como `https://xxxxxxxxxxxxxx.supabase.co`
   - **Project API keys → `anon` `public`** → un token largo `eyJ...`
3. Abre `index.html` y busca este bloque (línea ~1148 aprox):

```js
// ══ NOTAS (Supabase + localStorage fallback) ══
const SUPABASE_URL = "";   // ej: "https://xxxxxxxx.supabase.co"
const SUPABASE_KEY = "";   // anon/public key
```

4. Pega los valores entre las comillas:

```js
const SUPABASE_URL = "https://xxxxxxxxxxxxxx.supabase.co";
const SUPABASE_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...";
```

5. Guarda el archivo.

---

## 4. Probar

1. Abre `index.html` en el navegador.
2. Inicia un examen. Dentro de cada pregunta verás un cuadro **📝 Nota personal** con un identificador tipo `materias_comunes_2025.2__q01`.
3. Escribe una nota y haz click fuera del cuadro (`blur`). Debe aparecer **Sincronizada ✓** en la esquina derecha del cuadro.
4. Abre la app en otro dispositivo (móvil) o en otra pestaña → ve a la misma pregunta → la nota aparece cargada.
5. Al **responder** la pregunta, encima del cuadro de nota aparece un banner dorado **📝 Nota: ...** con tu texto.

---

## 5. Inspeccionar las notas en Supabase

- Panel izquierdo → **Table Editor** → `notas` → ves todas las filas con `id_pregunta`, `nota`, `updated_at`.
- También puedes hacer queries: **SQL Editor** →

```sql
select id_pregunta, length(nota) as len, updated_at
from public.notas
order by updated_at desc;
```

---

## 6. ¿Y si no quiero Supabase aún?

Deja `SUPABASE_URL` y `SUPABASE_KEY` vacíos. El sistema funciona igual pero las notas se guardan **solo en este navegador** (`localStorage`). Verás el indicador **Solo local** en cada nota.

Cuando rellenes las credenciales, las notas locales **se mantienen** y se suben automáticamente la próxima vez que las edites (o puedes forzar un sync escribiendo y desescribiendo).

---

## 7. Estados que verás en cada nota

| Estado | Color | Significado |
|---|---|---|
| `Sincroniza con Supabase` | verde | conectado y al día |
| `Solo local` | dorado | sin Supabase configurado o sin red |
| `Guardando…` | gris | enviando al servidor |
| `Sincronizada ✓` | verde | guardada en la nube |
| `Eliminada` | verde | nota borrada |
| `Error sync · guardada local` | rojo | fallo de red, se guardó local y se reintentará al volver a editar |

---

## 8. Punto de retorno (rollback)

Si algo va mal, restaura el estado previo con:

```bash
git reset --hard v-pre-notas
```

(La etiqueta `v-pre-notas` se creó antes de implementar esta funcionalidad.)

---

## 9. Próximos pasos opcionales (no necesarios para MVP)

- **Login por email/Google**: Auth → Providers → activar. Luego añadir `user_id uuid references auth.users` a la tabla y cambiar las políticas a `auth.uid() = user_id`.
- **Backup**: botón "⬇ Exportar notas JSON" en el dashboard para descargar copia local.
- **Búsqueda de notas**: pantalla `Mis notas` con buscador por texto.
- **Compartir notas**: tabla `notas_compartidas` con `slug` aleatorio.
