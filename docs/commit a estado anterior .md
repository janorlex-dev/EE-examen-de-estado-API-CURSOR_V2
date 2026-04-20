Hecho
Tu rama actual (master) ha quedado exactamente en el commit
74b79f8110069d9dc78ca155dc9d33e7096d4491
(mensaje: git config _para configurar el credentil.helper).

Se ha usado:

git reset --hard 74b79f8110069d9dc78ca155dc9d33e7096d4491
Importante: todo lo que había después de ese commit en master y los cambios sin commitear en el working tree se han descartado respecto a ese punto.

Si también quieres que GitHub (origin/master) vuelva a ese commit
Ahí los commits “nuevos” seguirán existiendo en el remoto hasta que fuerces:

git push --force-with-lease origin master
Solo hazlo si estás seguro de que nadie más depende de los commits posteriores; --force-with-lease es más seguro que --force puro.

Si en el futuro solo quieres mirar ese commit sin mover master
git switch --detach 74b79f8110069d9dc78ca155dc9d33e7096d4491
(o git checkout 74b79f8 con el hash corto).