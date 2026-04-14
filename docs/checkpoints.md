### “puntos de control”: no dependes de encadenar 40 veces Ctrl+Z.

Antes de una tanda de cambios, guarda un checkpoint rápido:

git stash push -m "checkpoint antes de probar X"

Pruebas en navegador con live reload.

Si sale mal, vuelves al punto exacto en segundos:

git stash pop (recuperas ese estado)

Si solo quieres tirar lo último y volver a lo commiteado:

git restore index.html (un archivo)
git restore . (todo el proyecto, cuidado)


Flujo eficiente recomendado (sin perder tiempo):

git status (ver limpio/sucio)
git stash push -m "checkpoint"

probar cambios
si funciona, sigues; si no, git stash pop o git restore ...

cuando ya esté bien: git add/commit/push
Para ti, lo más cómodo suele ser: live-server + stash por tandas.
Así no dependes de encadenar 40 veces Ctrl+Z.


## VOLVER AL CHECKPOINT(N)

Si quieres volver a uno específico (por ejemplo “el 5”), se hace así:

Ver lista:
git stash list
Aplicar uno concreto y quitarlo de la lista:
git stash pop stash@{5}
Si prefieres probar sin borrarlo (más seguro), usa apply:

git stash apply stash@{5}
Y luego, si te convence, lo borras manualmente:

git stash drop stash@{5}

Clave importante:


## PONER MENSAJES AL STASH 
El número stash@{N} no es fijo para siempre; cambia cuando haces pop/drop.
Por eso conviene poner mensajes claros al crear stash (-m "checkpoint X").

Para crear un stash (que luego aparecerá como stash@{0} porque será el más reciente), usa:

git stash push -m "checkpoint 0 - antes de cambiar timer"
Eso guarda tus cambios actuales en un checkpoint.

Luego comprobar:

git stash list
Verás algo tipo:

stash@{0}: On master: checkpoint 0 - antes de cambiar timer

## EL STASH  LO HACE A TODO EL PROYECTO O AL FILE MODIFICADO, POR EJEMPLO EL INDEX ??

Por defecto, git stash push guarda todos los cambios tracked del proyecto (todos los archivos versionados que estén modificados), no solo index.html.

Si quieres stash solo de index.html, hazlo así:

git stash push -m "checkpoint solo index" -- index.html
Resumen rápido:

git stash push -m "..." → todo el proyecto (archivos trackeados modificados).
git stash push -m "..." -- index.html → solo ese archivo.
Si quieres incluir también archivos nuevos (untracked), añade -u:
git stash push -u -m "checkpoint con nuevos archivos"


## EJOR HACER EL STASH DE TODO CREO YA QUE PUEDE HABERSE AFECTADO ALGUNO RELACIONADO

Sí, en tu caso es mejor opción hacer stash de todo.
Para cambios de una app conectada (HTML/CSS/JS), suele ser más seguro guardar el estado completo.

Usa este comando estándar:

git stash push -u -m "checkpoint completo antes de pruebas"
-u incluye también archivos nuevos.
Así te aseguras de poder volver al estado exacto del proyecto.
Luego:

ver lista: git stash list
recuperar último checkpoint: git stash pop
recuperar uno concreto: git stash pop stash@{N}
Buena decisión para evitar perder dependencias entre archivos.
