Git no te da una URL para “ejecutar” la app en el navegador: solo guarda el código. Lo que puedes ver en GitHub son otras cosas:

1. URL del repositorio (para ver el proyecto en la web)
En GitHub abre tu repo y arriba verás la barra de direcciones, por ejemplo:

https://github.com/janorik/EE-examen-de-estado-API-CURSOR

El botón verde Code muestra la URL para clonar (https://github.com/janorik/....git), no es la de “abrir la app”.

2. Cómo abrir tu app (tu index.html)
En tu PC: abre el archivo index.html con el navegador (doble clic o arrastrar a Chrome/Edge). La URL será tipo file:///D:/.../index.html. Eso no sale de Git; es la ruta de tu carpeta.
En internet (compartir enlace): hace falta publicar el sitio, por ejemplo GitHub Pages (Ajustes del repo → Pages → elegir rama master y carpeta / (root)). Entonces GitHub te mostrará una URL tipo
https://janorik.github.io/EE-examen-de-estado-API-CURSOR/
(el texto exacto lo indica la propia pantalla de Pages cuando está activo).
Resumen
Qué quieres	Dónde lo ves
Repo en GitHub	Barra del navegador en la página del repo
URL para clonar (git clone)	Botón Code → HTTPS
URL pública para abrir la app	Solo si activas GitHub Pages (u otro hosting: Vercel, Netlify, etc.)
Si me dices si quieres usar solo archivo local o un enlace público, te indico los pasos concretos para tu caso.

janorik.github.io/EE-examen-de-estado-API-CURSOR
 es esta qeu ya la tenia pero no se de donde la he sacado

Esa dirección no la “inventa” Git a parte; es la URL estándar de GitHub Pages cuando publicas un sitio desde tu cuenta y ese repositorio.

De dónde sale exactamente
GitHub usa esta plantilla:

https://<usuario>.github.io/<nombre-del-repo>/

En tu caso:

Usuario: janorik → janorik.github.io
Repositorio: EE-examen-de-estado-API-CURSOR → el path después del dominio
Por eso es: https://janorik.github.io/EE-examen-de-estado-API-CURSOR/

Dónde lo “sacas” en GitHub (por si lo vuelves a buscar)
Entra al repo EE-examen-de-estado-API-CURSOR en GitHub.
Settings (del repositorio) → menú izquierdo Pages.
Ahí pone la URL del sitio cuando Pages está activo (rama, carpeta / o /docs, etc.).
También puede aparecer un aviso en la pestaña Actions si el despliegue lo hace un workflow, pero la URL visible y oficial suele estar en Settings → Pages.

Resumen
La sacaste (o GitHub te la mostró) al activar Pages en ese repo: no hace falta recordarla de memoria; siempre es usuario.github.io + nombre del repo, salvo que uses un dominio personalizado.