 Flujo esquemático para el día a día.

Flujo normal de cambios (95 % de las veces)
┌───────────────────────────────────────────────────────────────┐
│  1. MODIFICAS CÓDIGO en Cursor                                │
│     (index.html, /examenes/*.csv, etc.)                       │
└───────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌───────────────────────────────────────────────────────────────┐
│  2. PRUEBA LOCAL                                              │
│     Doble clic en index.html → abre en navegador              │
│     Haz el flujo completo: examen → responder → nota          │
│     F12 → Console → que no haya errores rojos                 │
└───────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌───────────────────────────────────────────────────────────────┐
│  3. COMMIT + PUSH (PowerShell en Cursor)                      │
│     git status         ← ver qué cambió                       │
│     git add .          ← añade todo lo modificado             │
│     git commit -m "breve descripción de qué cambiaste"        │
│     git push           ← sube a GitHub (rama master)          │
└───────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌───────────────────────────────────────────────────────────────┐
│  4. VERCEL DESPLIEGA SOLO (~30-60 s)                          │
│     No tienes que hacer nada.                                 │
│     Panel de Vercel → Deployments → verás la fila nueva       │
│     con tu commit, estado "Building" → "Ready" (verde)        │
└───────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌───────────────────────────────────────────────────────────────┐
│  5. VERIFICA EN LA URL PÚBLICA                                │
│     https://ee-examen-de-estado-api-cursor-v2.vercel.app/     │
│     Ctrl + F5 para saltar caché                               │
│     Opcional: F12 → Console → typeof initNotas → "function"   │
└───────────────────────────────────────────────────────────────┘
Cuándo tocar Supabase (poco frecuente)
Solo cuando cambie el esquema de datos, no cuando toques código:

Añades un campo nuevo a la tabla notas (ej. user_id cuando metas login).
Creas una tabla nueva (ej. progreso, estadisticas).
Cambias reglas RLS.
Pasos:

1. Supabase Dashboard → SQL Editor
2. Ejecuta el SQL que toque (CREATE TABLE, ALTER TABLE, etc.)
3. Actualiza SUPABASE_SETUP.md con el nuevo SQL (para documentar)
4. Si cambió la lógica de acceso, ajusta el código en index.html
5. Commit + push normal (pasos 3-5 del flujo de arriba)
No toques Supabase para subir código — Supabase es solo base de datos, no sirve tu app. La app entera vive en Vercel.

Cuándo volver a redeploy manual en Vercel
Casi nunca. Solo en estos casos raros:

Un deploy se queda en estado Error y quieres reintentar.
Has cambiado una variable de entorno en Vercel (Settings → Environment Variables) y necesitas que la coja un build nuevo.
Sospechas caché rancia y quieres un build totalmente limpio.
En esos casos: Vercel → Deployments → fila más reciente → ··· → Redeploy → Environment: Production (¡siempre!) → desmarca build cache → Redeploy.

Checklist mental antes de git push

¿Probado en local?

¿Sin errores en F12 Console local?

¿Credenciales sensibles en
.secretos/
y no en el código? (la
anon key
de Supabase SÍ va en
index.html
, pero la
service_role
NUNCA).

¿Mensaje de commit claro en una frase?
Errores típicos y cómo reconocerlos
Síntoma tras push	Causa probable	Cómo arreglar
Vercel no crea deploy nuevo	Integración GitHub↔Vercel rota	Settings → Git → Reconnect
Vercel crea Preview pero no Production	Production Branch mal configurada	Settings → Git → Production Branch = master
Deploy Ready pero la URL sigue vieja	Caché del navegador	Ctrl + F5 o modo incógnito
Supabase unauthorized en consola	Caducó/cambió la anon key	Supabase → API → copiar anon key → pegar en index.html
Notas no aparecen pero sin errores	IDs no coinciden	Comparar id_pregunta de Supabase con keyQ()
Resumen en 3 líneas
Cambias código → git add/commit/push → Vercel despliega a Production automáticamente.
Cambias esquema de datos → SQL en Supabase + documentar en SUPABASE_SETUP.md.
Verifica abriendo la URL pública con Ctrl+F5 y F12 Console sin errores rojos.
Eso es todo. Con el Production Branch ya configurado como master, no tienes que volver a tocar la pestaña "Deployments" de Vercel salvo emergencias.