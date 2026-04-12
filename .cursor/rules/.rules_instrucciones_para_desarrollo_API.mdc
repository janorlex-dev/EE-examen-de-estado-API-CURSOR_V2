---
description: Examen de Estado — diseño del producto y normas para el agente
alwaysApply: true
---

## Reglas de diseño y producto (desarrollo)

DESARROLLO WEB APP “TEST EXAMEN ABOGACIA, EXAMEN DE ESTADO”
CONTEXTO Y ROL 
Actúa como un desarrollador senior full-stack especializado en aplicaciones web educativas escalables, con experiencia en sistemas de repetición espaciada tipo Anki, arquitectura modular y despliegue en Vercel, Railway y Github . 
Tu objetivo es ayudarme a construir una aplicación robusta, simple de momento, pero que sea esclable en el futuro, flexible y ampliable en el tiempo.

FUNCIONALIDADES CLAVE:
1.hacer un MVP o una app simple o API  para hacer exámenes de test 
1.1. para correr (RUN) en cualqueir pc o movil (responsive) sin  registro , libre, lo mas facil posible 
2. Estructura mínima posible  (ver PLAN MVP abajo  (puedes recomendar otras)
•	1 archivo HTML 
•	1 archivo JS 
•	(opcional) 1 archivo CSS
•	scroll para desplazar
•	boton Zoom — control fijo en esquina inferior derecha. (mas usado en el movil)

3. prioriza siempre soluciones simples.
4. que sea escalable. Modular que me permita mas adelante añadir opciones sin tener que rediseñar de nuevo todo .(tengo GITHUB , RAILWAY para hacerla en remoto mas adelante).  

TABLERO/ dashboard
 Materias Comunes centrada y grande arriba, especialidades en grid 2×2 debajo, Cada materia tiene su color propio: verde (comunes), ámbar (laboral), rojo (penal), azul (civil), violeta (admin).Materias Comunes arriba centrado grande, especialidades en grid 2×2
5. tablero dashboard= panel de control= tablero con cuadros con materias=bloques y dentro los examenes  por  años y convocatoria , primera y segunda .
6. Badges/píldoras por convocatorias dentro de cada card (estilo imagen)
Expandir convocatorias si hay más de las que caben (+7 más)
Colores distintos por materia (como en las imágenes — verde, rojo, naranja, teal...)
— las años aparecen como píldoras clicables dentro de cada card. 
7. Opcion de “desplagar mas” Si hay más de las que caben, aparece "+N más" expandible (si es necesario se comparte la imagen de muestra VER Maiz Peña)
8. tiene que tener un temporizador que vaya contando , que no pare hasta el final 
El tiempo límite es configurable antes de empezar cada test
9. cuando se abre el examen muestra las preguntas de una en una , (si es necesario se comparte la imagen de muestra)
10. al posicionar el cursor en la pregunta se ensombrece la elegida, al hacer click se marca. si es correcta se pone en verde, si no en rojo  ademas de marcar en rojo la correcta (si es necesario se comparte la imagen de muestra) 
11.  abajo opciones de : “Anterior”  vuelve, “seguir”pasa a  la siguiente pregunta , “repetir” limpia y da opcion a repetir y “siguiente” pasa a la siguiente 
12.  contador durante la realizacion del examen con los aciertos en verde y errores en rojo con las materias.
13. boton salir en cualqueir momento cierra la aplicación. Pide confirmación si estás en medio de un tes
Botón volver — aparece en el header durante el examen y en la barra inferior de cada pantalla. Pide confirmación si estás en medio de un test.
14. boton de volver al dashboard principalde paleta de examenes en cualqueir momento 
15. añadir en el pie lo que esta en esta imagen c de copyrigh 2026 horlexia y aviso legal , privacidad y cookies
16. los pdfs por años y convocatoria de examen convertidos a csv (VER CONVERSION) se integran en el HTML 
17. preguntas ERROR que se han contestado mal se indexan o se guardan en base temporal para posteriormente volver a responder las erroneas antes de finalizar el test.
18 .finalizado el test se muestra un marcador con el tiempo total , la puntuacion y las que se han repetido o contestado mal , para contarlas como errores. 
19. una vez finalizado dar opcion con unos botones de repetir o volver al panel principal para volver a seleccionar algun test haciendo click en la casilla del test de la materia y año seleccionado
20. Antes habria que preparar los examenes por años de convocatoria los tengo en pdf con las respuestas marcadas y creo que los tengo que pasar a  csv o texto plano . Puedo cargarlos en el IDE, o en el directorio donde alojemos la aplicación, para cuando seleccione desde el panel de control o dashboard me empiece el test. 
22. hazme las preguntas que necesites para no empezar hasta tener todo claro , por si me he olvidado de algo , prefiero tardar en empezr, que tener que estar perdiendo el tienmpo modificando despues 
22.Puedes sugerir o preguntar si hay mejores opciones
23. Te comparto unas capturas de pantalla modo de muestra para dar una idea mas precisa de lo que quiero 
24. Usar agentes de AI para estructurar preguntas
25.ESCALABILIDAD FUTURA
26.Estadísticas avanzadas
27. Sistema de progreso por materia




ARQUITECTURA (CRÍTICA):
·	Diseño modular, escalable y desacoplado
·	Separación por capas: UI / lógica / datos
·	Evitar hardcoding (todo configurable)
·	Preparado para añadir nuevas funcionalidades sin refactorización global

  
El formato CSV 
pregunta,a,b,c,d,correcta, materia (materias= civil-mercantil . laboral , penal y administrativo )
"¿Texto de la pregunta?","Opción A","Opción B","Opción C","Opción D","b","Derecho Laboral"

CRITERIOS DE CALIDAD
·	Código limpio, modular y mantenible
·	Componentes reutilizables
·	Separación clara entre UI, lógica y datos
·	No hardcodear valores críticos
·	Documentar decisiones importantes
·	Evitar complejidad innecesaria

CÓMO DEBE SER LA RESPUESTA DEL AGENTE DE LA IA EN EL DESAROLLO
·	Paso a paso
·	Código listo para copiar/pegar
·	Indicar en qué archivo va cada parte
·	Explicaciones técnicas breves
·	Priorizar MVP funcional pero escalable
·	No dar teoría innecesaria

VERIFICACIÓN
Antes de responder:
·	Validar compatibilidad con programas (ejemplo : Next.js y Vercel)
·	Confirmar que el sistema es escalable
·	Asegurar que se pueden añadir nuevos CSV O PDFs sin romper estructura
·	Verificar que la repetición espaciada es configurable
·	Evitar dependencias innecesarias

INSTRUCCIÓN DE TRABAJO
Siempre prioriza:
1.	Simplicidad inicial (MVP funcional)
2.	Escalabilidad futura
3.	Código reutilizable
4.	Arquitectura limpia
Si falta información:
·	Haz solo las preguntas estrictamente necesarias
·	Si no, toma decisiones razonables por defecto

Botón para subir CSV desde el navegador. (EN EL FUTURO CONVERTIRA LOS QUE SE SUBAN TAMBIEN)
•	subir pdf  o word con las preguntas y respuestas marcadas con [OK] o con una PLANTILLA DE RESPUESTAS para no tener que responderlas ni buscarlas, convierten PDFs a CSV una vez 
•	Puedes tener todos los exámenes como archivos .csv en una carpeta y cargar el que quieras
•	Funciona en cualquier PC o móvil sin instalar nada
•	El sistema debe permitir añadir nuevos PDFs sin modificar la app
•	Soporte para usuarios
Pantalla 1 — Dashboard
•	Título de la app + fecha
•	Tarjetas Cards por convocatoria/materia/ año (las cargo como CSV ej. "Laboral 2025.2", "Civil 2024")
•	Cada card muestra: nº preguntas + tiempo límite 
•	Botón "Cargar examen" (sube un CSV desde tu PC o móvil)
•	Al seleccionar: elige modo → Examen completo o Preguntas sueltas (N aleatorias)
•	3 modos: Examen completo · Por bloques · Preguntas sueltas (aleatorio, N preguntas)
•	Botón cargar CSV (PARA proximas modificaciones del codigo para convertir de pdf o word  y añadir nuevos exámenes sin tocar código
•	Botones: Salir  (de la aplicación  con confirmacion) 

Pantalla 2 — Examen (pregunta a pregunta)
•	Cabecera fija: bloque, pregunta X de N, temporizador cuenta atrás ⏱ countdown (configurable, ej. 3h), contadores OK / ERR en verde/rojo
•	Barra de progreso
•	Pregunta en card central
•	Cuerpo: enunciado + 4 opciones en tarjetas
•	Hover → sombra/resalte. Click → verde (correcta) / rojo (elegida errónea) + verde (correcta revelada)
•	Botones inferiores: ← Anterior · Repetir · Siguiente →
•	Repetir limpia la respuesta y permite volver a contestar (cuenta como error adicional)
•	4 opciones: hover = sombra azul, click = verde (correcta) / rojo (elegida) + verde (correcta)
Botones inferiores: Anterior · Repetir · Siguiente  Repetir test · Repasar falladas · Volver al panel . Salir  (de la aplicación  con confirmacion)
Pantalla 3 — Resultados
•	Tiempo total empleado 
•	Puntuación · Aciertos / Errores / Repetidas / total)
•	Lista de falladas (con respuesta correcta)
•	Botones: Repetir test · Repasar falladas · Volver al panel · Salir (de la aplicación  con confirmacion)
________________________________________
Lógica de falladas
•	Se guardan acumulan en memoria durante el test
•	Repaso de falladas — al finalizar te pregunta si quieres repasar antes de ver resultados. También se listan desde la pantalla de resultados y en el marcador final
•	Al finalizar: opción de repasar falladas antes de ver resultados (sesión de repaso inline)
•	Al finalizar: opción de repasar falladas ahora (misma sesión) o guardar para después (descarga JSON o sessionStorage)
Resultados — correctas, errores, sin responder, nota en %, tiempo empleado, lista de falladas con respuesta correcta.

________________________________________

Formato CSV
pregunta,a,b,c,d,correcta, materia (materias= civil-mercantil . laboral , penal y administrativo )
"¿Texto de la pregunta?","Opción A","Opción B","Opción C","Opción D","b","Derecho Laboral"

EXAMENES pdf  csv
Fase 1 — Construcción del banco
•	AYUDAD DE IA  pasar los PDFs uno a uno
•	La IA devuelve cada uno como CSV independiente
•	yo  los guardo en la carpeta del proyecto  “EE test abogacia /examen de estado API CURSOR” en la carpeta /examenes/
Fase 2 — Integración final
•	Integrar en el codigo HTML los csv en el index.html.
•	Resultado: un solo archivo que abre sin necesidad de cargar nada

La idea sería incrustar los CSV directamente en el index.html como datos JavaScript. Al abrir el archivo los exámenes ya están ahí, sin cargar nada.

La estructura sería algo así:
javascript
const EXAMENES_INTEGRADOS = [
  {
    name: "Materias Comunes 2025.2",
    año: "2025",
    
    materia: "Materias Comunes",
    preguntas: [ /* todas las preguntas aquí */ ]
  },
  {
    name: "Laboral 2024",
    año: "2024.1", 
    materia: "Derecho Laboral",
    preguntas: [ /* ... */ ]
  },
  // etc.
]
El botón "Cargar CSV" seguiría disponible para añadir exámenes nuevos en el futuro.

________________________________________

Parámetro	Decisión
Archivos	index.html + carpeta /examenes/ con CSV
Carga	Automático (carpeta) + botón manual
Temporizador	Cuenta atrás, configurable antes de cada test
Modos	Examen completo · Por bloques · Preguntas sueltas
Falladas	Repasar inline al final + opción sesión separada
Estilo	Verde #0d4a45 + Dorado #c9a04a — marca NOrlexia
Logo	En cabecera (con el PNG que me pases)

📋 STACK TECNOLÓGICO .MVP
Estructura de archivos
/test-abogacia/
│── index.html       ← todo en uno (HTML + CSS + JS)
│── /examenes/
│     ├── laboral_2025.csv
│     ├── civil_2024.csv
│     └── ...
Un solo archivo HTML autocontenido. Los CSVs en una subcarpeta. Pantalla 1,etc — Dashboard
•	Título + fecha
•	Cards por convocatoria/materia (ej. "Laboral 2025.2", "Civil 2024")
•	Cada card muestra: nº preguntas + tiempo límite
•	3 modos: Examen completo · Por bloques · Preguntas sueltas (aleatorio, N preguntas)
•	Botón cargar CSV propio (para añadir nuevos exámenes sin tocar código) Pantalla 2 — Pregunta
•	mi logo de NORLEXia incrustado o directamente en el HTML como imagen. — funciona, pero si tienes versión PNG transparente queda mejor sobre la cabecera.
paleta de colores 
   Verde oscuro (fondo/sidebar): #0d4a45
   Dorado (acento/títulos): #c9a04a
   Blanco/crema (texto): #f5f0e8
                Fondo claro (contenido): #f0f2f0

•	Paleta NOrlexia — verde #0d4a45 + dorado #c9a04a en toda la interfaz
•	Footer — © 2026 NOrlexia LEGALTECH con Aviso Legal · Privacidad · Cookies, mismo estilo que tu web corporativa.
•	Cabecera fija: bloque · "Pregunta X de N" · ⏱ countdown · contador OK/ERR
•	Barra de progreso
•	Pregunta en card central
•	4 opciones: hover = sombra azul, click = verde (correcta) / rojo (elegida) + verde (correcta)
•	Botones inferiores: Anterior · Repetir · Siguiente Lógica de falladas
•	Se acumulan en memoria durante el test
•	Al finalizar: opción de repasar falladas ahora (misma sesión) o guardar para después (descarga JSON o sessionStorage) Pantalla 3 — Resultados
•	Tiempo total · Puntuación · Aciertos / Errores / Repetidas
•	Botones: Repetir test · Repasar falladas · Volver al panel · Salir Formato CSV pregunta,a,b,c,d,correcta,bloque "¿Cuándo...?","Opción a","Opción b","Opción c","Opción d","a","Laboral" Lo que NO incluyo (MVP)
•	Backend / base de datos
•	Login / registro
•	Estadísticas históricas entre sesiones (eso sería v2) ¿Confirmas el plan y arrancamos? Si quieres cambiar algo antes de que empiece, es el momento.
Formato CSV
pregunta,a,b,c,d,correcta,bloque
"¿Cuándo...?","Opción a","Opción b","Opción c","Opción d","a","Laboral"

---
description: Proyecto Test Examen Abogacía / Examen de Estado — MVP, UI, CSV, arquitectura y estilo de respuesta del agente
alwaysApply: true
---

## Reglas para el agente de IA


Estas son instrucciones de como deben de responder los agentes de IA en los IDE cuando se esta dessplegando el codigo del proyecto
CRITERIOS DE CALIDAD
·	Código limpio, modular y mantenible
·	Componentes reutilizables
·	Separación clara entre UI, lógica y datos
·	No hardcodear valores críticos
·	Documentar decisiones importantes
·	Evitar complejidad innecesaria

CÓMO DEBE SER LA RESPUESTA DEL AGENTE DE LA IA EN EL DESAROLLO
·	Paso a paso
·	Código listo para copiar/pegar
·	Indicar en qué archivo va cada parte
·	Explicaciones técnicas breves
·	Priorizar MVP funcional pero escalable
·	No dar teoría innecesaria

VERIFICACIÓN
Antes de responder:
·	Validar compatibilidad con programas (ejemplo : Next.js y Vercel)
·	Confirmar que el sistema es escalable
·	Asegurar que se pueden añadir nuevos CSV O PDFs sin romper estructura
·	Verificar que la repetición espaciada es configurable
·	Evitar dependencias innecesarias

INSTRUCCIÓN DE TRABAJO
Siempre prioriza:
1.	Simplicidad inicial (MVP funcional)
2.	Escalabilidad futura
3.	Código reutilizable
4.	Arquitectura limpia
Si falta información:
·	Haz solo las preguntas estrictamente necesarias
·	Si no, toma decisiones razonables por defecto



