# Configuración Supabase — Enlaces (Códigos y leyes)

Esta guía añade la tabla `enlaces` al mismo proyecto Supabase que ya creaste
para las notas (ver `SUPABASE_SETUP.md`). Reutiliza la misma `SUPABASE_URL`
y `SUPABASE_KEY` que ya están en `index.html`, no hay que añadir credenciales
nuevas.

---

## 1. Crear la tabla `enlaces`

1. En el panel izquierdo de Supabase abre **SQL Editor** → **New query**.
2. Pega este SQL completo y pulsa **Run** (▶):

```sql
-- Tabla de enlaces (códigos y leyes, links útiles)
create table if not exists public.enlaces (
  nombre      text primary key,
  url         text not null,
  updated_at  timestamptz not null default now()
);

-- Índice por fecha (para ordenar por recientes si hiciera falta)
create index if not exists enlaces_updated_at_idx on public.enlaces (updated_at desc);

-- Activar Row Level Security
alter table public.enlaces enable row level security;

-- Políticas abiertas (MVP compartido sin login)
-- ⚠ Cualquiera con la anon/publishable key puede leer/escribir.
-- Para uso personal en tus dispositivos es aceptable. Cuando añadas login,
-- sustituye estas políticas por filtros por user_id.
drop policy if exists "enlaces_open_select" on public.enlaces;
drop policy if exists "enlaces_open_insert" on public.enlaces;
drop policy if exists "enlaces_open_update" on public.enlaces;
drop policy if exists "enlaces_open_delete" on public.enlaces;

create policy "enlaces_open_select" on public.enlaces for select using (true);
create policy "enlaces_open_insert" on public.enlaces for insert with check (true);
create policy "enlaces_open_update" on public.enlaces for update using (true) with check (true);
create policy "enlaces_open_delete" on public.enlaces for delete using (true);
```

Deberías ver `Success. No rows returned`.

3. Verifica que la tabla existe: panel izquierdo → **Table Editor** →
   `enlaces` con 3 columnas (`nombre`, `url`, `updated_at`).

---

## 2. Probar

1. Abre `index.html` en el navegador.
2. Pulsa **📖 Códigos y leyes** en el dashboard (o el mismo botón dentro de
   cualquier pregunta del examen).
3. Introduce un nombre (ej: `LEC`) y pega una URL (ej:
   `https://www.boe.es/eli/es/l/2000/01/07/1/con`).
4. Pulsa **＋ Añadir**. El indicador debe pasar a **Sincronizado ✓**.
5. Abre la app en otro dispositivo (móvil, otro navegador) → pulsa
   **📖 Códigos y leyes** → el enlace debe aparecer cargado.
6. Clic en el enlace → abre en **pestaña nueva** sin salir de la app.

---

## 3. Comportamiento clave

| Acción | Qué hace |
|---|---|
| URL sin `http(s)://` | Se antepone `https://` automáticamente al guardar |
| Nombre duplicado | Se bloquea con aviso, no sobrescribe (comparación sin tildes y sin distinguir mayúsculas) |
| Editar nombre | Borra la fila anterior y crea la nueva (renombrado real) |
| Borrar | Pide confirmación antes |
| Buscador | Filtra por nombre en tiempo real, sin tildes |
| Sin credenciales Supabase | Funciona igual, modo **Solo local** (se guarda en `localStorage`) |

---

## 4. Inspeccionar los enlaces en Supabase

Panel izquierdo → **Table Editor** → `enlaces`. O queries SQL:

```sql
select nombre, url, updated_at
from public.enlaces
order by nombre;
```

---

## 5. Estructura de datos

- **Fuente de verdad en sesión**: `enlacesCache` en memoria `{ nombre: url }`
- **Espejo offline**: `localStorage["norlexia_enlaces_v1"]`
- **Sincronización**: tabla `public.enlaces` en Supabase

Orden de escritura en `guardarEnlace`:
1. Validar nombre no vacío y no duplicado (bloquea si ya existe).
2. Normalizar URL (autofix `https://`).
3. Actualizar `enlacesCache` (síncrono).
4. Persistir `localStorage` (síncrono).
5. `upsert` (+ `delete` previo si es renombrado) a Supabase en background.
6. Reflejar estado en el indicador UI.

---

## 6. Rollback

Si quieres borrar toda la tabla y empezar de cero:

```sql
drop table if exists public.enlaces cascade;
```

Y vuelve a ejecutar el bloque `create table ...` del paso 1.

---

## 7. Próximos pasos opcionales

- **Categorías**: añadir columna `categoria text` (ej. "Civil", "Penal") y
  agrupar en la tabla del modal.
- **Links por pregunta**: tabla adicional `enlaces_pregunta(id_pregunta,
  id_enlace)` para asociar un enlace concreto a una pregunta concreta.
- **Orden manual**: columna `orden int` para drag-and-drop.
- **Exportar JSON**: botón "⬇ Exportar enlaces" en el modal para backup local.

Eso significa que el SQL para crear enlaces aún no se ha ejecutado (o se ejecutó pero dio error silencioso y te quedaste mirando la tabla de notas). Vamos a solucionarlo en 3 pasos.

Paso 1 — Comprueba si realmente no existe
En Supabase → SQL Editor → New query → pega esto y pulsa Run:

select table_name
from information_schema.tables ```sql select * from public.enlaces; ```
where table_schema = 'public'
order by table_name;
Debería devolverte una lista. Si solo ves notas → efectivamente falta crear enlaces.

```sql Paso 2 — Crea la tabla enlaces ahora´´´
En el mismo SQL Editor → New query → pega exactamente este bloque completo y pulsa Run. Si sale el aviso de RLS, elige Run and enable RLS:

create table if not exists public.enlaces (
  nombre      text primary key,
  url         text not null,
  updated_at  timestamptz not null default now()
);
create index if not exists enlaces_updated_at_idx on public.enlaces (updated_at desc);
alter table public.enlaces enable row level security;
drop policy if exists "enlaces_open_select" on public.enlaces;
drop policy if exists "enlaces_open_insert" on public.enlaces;
drop policy if exists "enlaces_open_update" on public.enlaces;
drop policy if exists "enlaces_open_delete" on public.enlaces;
create policy "enlaces_open_select" on public.enlaces for select using (true);
create policy "enlaces_open_insert" on public.enlaces for insert with check (true);
create policy "enlaces_open_update" on public.enlaces for update using (true) with check (true);
create policy "enlaces_open_delete" on public.enlaces for delete using (true);
Debe aparecer Success. No rows returned.

´´´Paso 3 — Verifica que todo quedó bien
En el mismo SQL Editor, pega esto y pulsa Run:

select
  c.relname as tabla,
  c.relrowsecurity as rls_activa,
  (select count(*) from pg_policies p
     where p.schemaname = 'public' and p.tablename = c.relname) as politicas
from pg_class c
where c.relname in ('notas', 'enlaces')
order by c.relname;
Debe devolver 2 filas:

tabla	rls_activa	politicas
enlaces	true	4
notas	true	4
Si enlaces no aparece después del paso 2
Revisa estos puntos típicos:

¿Pulsaste Cancel antes por la advertencia de RLS? Vuelve al paso 2 y esta vez pulsa Run and enable RLS.
¿Estás en el proyecto correcto? Arriba a la izquierda en Supabase — confirma que el proyecto es el mismo donde ves notas (mismo SUPABASE_URL que tienes en index.html: kyedoeauobqqfcpkihfs.supabase.co).
¿La pestaña Table Editor está cacheada? Pulsa F5 en el navegador o cambia a otra sección y vuelve a Table Editor. A veces no refresca sola tras un CREATE TABLE.
¿Hubo un error que no viste? Vuelve al SQL Editor, pestaña Results abajo → si hubo error aparece en rojo. Pégamelo y lo resolvemos.
Paso 4 — Probar desde la app
Abre index.html y pulsa F12 (consola del navegador).
Pulsa 📖 Códigos y leyes en el dashboard.
Añade un enlace de prueba (ej: LEC + https://www.boe.es/eli/es/l/2000/01/07/1/con).
El indicador debe pasar a Sincronizado ✓.
En la consola NO debe aparecer ningún [Enlaces] Error sync.
Vuelve a Supabase → Table Editor → enlaces → debe aparecer la fila.
Si tras el paso 2 sigue sin aparecer la tabla o te sale algún error, pégame aquí el mensaje exacto (de Supabase o de la consola del navegador) y lo depuramos.
