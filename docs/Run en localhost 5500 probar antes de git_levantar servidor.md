## run en local para ver lo modificado antes de git .add, commit y push

Abrir terminal en Cursor: Terminal > New Terminal
Ve a tu carpeta del proyecto (si no estás ya ahí):
cd "D:\♦PROGRAMAS_APIS_APPS_IA\EE test examen abogacia\EE examen de estado API CURSOR "
Levanta servidor con Python (lo más simple):

python -m http.server 5500

  Si python no funciona, prueba:  

py -m http.server 5500

Abre en navegador:

**************  http://localhost:5500  ************** parar el servidor:
 en esa terminal pulsa Ctrl + C.

## opción con recarga automática (live reload) para no refrescar manualmente cada cambio.

en vez de hacer F5 en el navegador cada vez que guardas, el navegador se actualiza solo al detectar cambios en index.html, .css, .js.

La forma más fácil:

En terminal del proyecto:
*********      npx live-server     **********

Se abrirá una URL local (normalmente http://127.0.0.1:8080).
Editas + Ctrl+S y la página se recarga automáticamente.
Si quieres fijar puerto:

npx live-server --port=5500
Para detenerlo: Ctrl + C.


