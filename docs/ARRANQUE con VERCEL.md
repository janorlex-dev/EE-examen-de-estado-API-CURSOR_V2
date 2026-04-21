
# Plan de arranque recomendado (mínimo esfuerzo, MVP)
Confirmar que el repo está al día en GitHub Ahora mismo tienes cambios sin commitear (index.html modificado, .gitignore, SUPABASE_SETUP.md, etc.). Primero commit + push para que Vercel tenga la versión buena.

Conectar Vercel a GitHub

Entras en https://vercel.com con la cuenta janorlex-dev (o la que uses) → Add New… → Project.
Importas el repo EE-examen-de-estado-API-CURSOR_V2.
Framework preset: Other (es HTML plano).
Build command: (vacío).
Output directory: ./ (la raíz, donde está index.html).
Deploy.
Obtener la URL pública Vercel te dará algo como:

https://ee-examen-de-estado-api-cursor-v2.vercel.app
Esa es la URL con la que arrancas. Funciona en PC y móvil sin instalar nada, cumpliendo tu regla de MVP.

Supabase Las claves (SUPABASE_URL y SUPABASE_KEY tipo anon/publishable) ya van dentro del index.html — está bien, la seguridad la da RLS, como ya indica tu regla del proyecto. No hay que configurar nada extra en Vercel para que funcione.

Importante: en Supabase → Project Settings → API → URL Configuration / CORS, añade tu dominio de Vercel a los orígenes permitidos cuando te pida.
Nunca subas la service_role key al frontend (tu regla ya lo dice).
Dominio propio (opcional, más tarde) Cuando quieras test.norlexia.com o similar, en Vercel → Settings → Domains y apuntas tu DNS. Esto no es necesario para arrancar.

Por qué Vercel y no Railway aquí
Tu app de momento es 100% estática (un index.html + Supabase como backend). Railway tiene sentido cuando haya un servidor Node/Next con lógica propia. Para el MVP actual:

Vercel → gratis, HTTPS automático, deploy en cada push, previews por rama. ✅
Railway → sobra, pensado para servicios con proceso activo. Lo reservamos para cuando metas backend propio (auth, corrección avanzada, estadísticas server-side).
Check rápido antes de desplegar

index.html
abre bien en local con doble clic y guarda notas en Supabase.

.gitignore
protege
.secretos/
,
.env*
y PDFs/DOCX.

Commit + push de los cambios pendientes a
master
.

Vercel conectado al repo, primer deploy OK.

Probar la URL de Vercel desde el móvil (responsive + Supabase sync)

Paso 1 — Forzar la sincronización YA (arreglo inmediato)
Entra en https://vercel.com → tu proyecto EE-examen-de-estado-API-CURSOR-V2 → pestaña Deployments.

Opción A (rápida): pulsa el botón ··· del último deploy y elige Redeploy → desmarca Use existing Build Cache → Redeploy.

Opción B (si no aparece el commit nuevo): pestaña Settings → Git. Comprueba:

Connected Git Repository = janorlex-dev/EE-examen-de-estado-API-CURSOR_V2 ← tiene que ser exactamente este.
Production Branch = master ← si pone main, cámbialo a master y guarda.
Después haz un commit vacío para disparar el deploy:

De hecho, aún tienes un archivo staged sin commitear (docs/ARRANQUE con VERCEL.md). En cuanto hagas commit + push de ese, Vercel debería desplegar solo (si está bien conectado).

Cuando quieras, lánzalo tú (no lo hago sin permiso):

git commit -m "docs: plan arranque Vercel"
git push
Paso 2 — Auto-deploy en cada git push (lo que pides)
Buenas noticias: Vercel ya hace auto-deploy por diseño cuando el proyecto está conectado a GitHub vía su integración oficial. No hay que configurar CI/CD ni GitHub Actions. El flujo queda así para siempre:

git add .
git commit -m "..."
git push        ← Vercel detecta el push y despliega automáticamente
Para garantizar que esa integración está viva:

Vercel → tu proyecto → Settings → Git → debe verse Connected to janorlex-dev/EE-examen-de-estado-API-CURSOR_V2 con un check verde. Si ves “Disconnected” o “Reconnect”, pulsa Reconnect y autoriza la GitHub App de Vercel.
GitHub → https://github.com/settings/installations → busca Vercel → Configure → en Repository access asegúrate de que EE-examen-de-estado-API-CURSOR_V2 está en la lista de repos permitidos. Si usas Only select repositories y no está marcado, Vercel no recibe los webhooks.
Vercel → Settings → Git → Ignored Build Step: déjalo vacío. Si ahí hay un script tipo exit 0, está cancelando los deploys.
Vercel → Settings → Git → Production Branch = master (clave: tu rama es master, no main).
Con esos 4 puntos OK, cada git push a master dispara un deploy de producción en ~30 s, y cada push a otra rama genera un Preview URL sin tocar producción.

Paso 3 — Comprobación final
Después de reconectar y del siguiente push, mira en Vercel → Deployments. Debe aparecer una fila nueva con:

Commit: el hash que acabas de pushear (ej. 88453ad o el siguiente).
Branch: master.
Status: Ready (verde).
Si sigue sin aparecer el commit después de reconectar, dime qué ves exactamente en Settings → Git (repo conectado y rama) y lo afinamos. Como atajo de emergencia siempre puedes instalar la CLI y desplegar a mano con npx vercel --prod, pero no hace falta si la integración de GitHub funciona.