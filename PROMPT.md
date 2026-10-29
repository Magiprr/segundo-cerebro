Quiero que me armes un "segundo cerebro" para este proyecto: una base con todo lo que se ha hecho aquí, con fecha y resumen por pieza, que consultes antes de responderme. Trabaja en la carpeta del proyecto donde estás abierto y crea todo dentro de una subcarpeta `_cerebro/`.

## Qué tiene que resolver

- Olvidas entre sesiones, y Claude Code borra las sesiones a los 30 días. Lo que se decidió en una conversación tiene que quedar respaldado y encontrable.
- Lo viejo se mezcla con lo nuevo. Cada pieza lleva fecha y lo reciente pesa más; si un dato cambió, gana el último.
- Meter todo en la conversación sale caro y empeora las respuestas. Buscas primero y lees solo lo que sirve.

## Cómo armarlo (4 partes)

**1. Conectores.** Un script `_cerebro/cerebro.py`, solo con la librería estándar de Python. `actualizar` trae solo lo nuevo desde la última vez (compara fecha de modificación y tamaño) y le pone fecha a cada pieza. Todos solo leen; no escriben fuera de `_cerebro/`.

- `archivos`: los .md, .txt, .docx, .pptx, .xlsx, .csv, .pdf y .html de la carpeta. Si un mismo documento está en dos formatos (.md y .docx), lee uno solo. Si un .md tiene varias secciones con fecha en el título, pártelo por fecha. Deja una lista `EXCLUIR` para código, carpetas de build y lo que yo diga.
- `sesiones`: mis conversaciones con Claude Code en este proyecto (los .jsonl de `~/.claude/projects/<ruta del proyecto con guiones>/`). Pártelas por día, y en trozos si son muy largas. Guarda una copia comprimida (.gz) en `_cerebro/respaldo_sesiones/`.
- `memoria`: los archivos de la carpeta `memory/` de ese mismo proyecto, si existe.
- `entrada`: una carpeta `_cerebro/entrada/<fuente>/` donde tú dejas un .md por correo o reunión que traigas con tus conectores (Gmail, Granola u otros), con nombre `AAAA-MM-DD - asunto.md`. El script los indexa.
- `linkedin` (opcional): el .zip más reciente de la exportación de datos de LinkedIn en `~/Downloads`; trae mensajes e invitaciones con nota desde la fecha que yo diga.

La configuración de las fuentes va en `_cerebro/fuentes.json`: desde qué fecha, la consulta y la regla de qué entra para cada conector, y una lista `sesiones_fuera` para sesiones que no deben entrar.

**2. Resúmenes con un modelo barato.** Por cada pieza, un subagente Haiku (`model: haiku`) escribe `_cerebro/resumenes/<id>.json` con:
- `resumen`: 2 a 4 frases. Qué es, qué se decidió o encontró, y en qué quedó, con la fecha.
- `preguntas`: 3 a 6 preguntas con que yo lo buscaría después, en mis palabras.
- `atomos`: 5 a 25 hechos sueltos que se entienden solos. Decisiones con su porqué ("Decisión (fecha): X, porque Y"), lo descartado y por qué, cifras con su fuente, personas con su rol y lo que aportaron, citas cortas y literales, pendientes ("Abierto: …"), y en lo técnico, comandos que funcionaron y errores con su arreglo.
- `temas`: 3 a 10 palabras clave.

Las instrucciones para Haiku van en `_cerebro/prompt_resumen.md`: devolver solo el JSON, no inventar nada que no esté en la pieza, y en correos o mensajes dejar fuera lo personal y quedarse con lo que toca el proyecto. `pendientes` lista las piezas sin resumen. Si hay `ANTHROPIC_API_KEY` o el CLI `claude` con sesión iniciada, `resumir` las hace solo; si no, las hacen subagentes dentro de la sesión.

**3. Búsqueda simple.** BM25 sobre texto plano, sin base vectorial. Normaliza tildes y quita palabras vacías. El título y el resumen pesan más que el texto bruto (por ejemplo 1,0 contra 0,4), para que una sesión larga que nombra mucho algo no tape a la pieza que trata de eso. Ajuste por fecha: una pieza nueva vale hasta el doble que una vieja, con vida media de 3 semanas (multiplicador entre 0,6 y 1,2). Lo que esté en una carpeta `_archivo/` pesa la mitad, y lo de `_cerebro/descartados.txt` no sale. Cada resultado muestra fecha, título, fuente, id, ruta al texto completo, el resumen y los átomos que calzan con la búsqueda.

**4. Salud.** `estado` dice cuándo corrió cada conector, cuántas piezas trajo y si falló. `buscar` avisa si lleva más de 24 horas sin actualizarse o si un conector falló.

Comandos:
```
python3 _cerebro/cerebro.py actualizar
python3 _cerebro/cerebro.py buscar <palabras>
python3 _cerebro/cerebro.py ver <id>
python3 _cerebro/cerebro.py pendientes
python3 _cerebro/cerebro.py resumir
python3 _cerebro/cerebro.py estado
```

Carpetas: `crudo/` (el texto de cada pieza), `resumenes/`, `respaldo_sesiones/`, `entrada/`, más `indice.json` y `estado.json`, que arma el script.

## Para que se use solo

Agrega una sección al `CLAUDE.md` del proyecto (créalo si no existe) que diga:

- Al empezar una sesión, correr `actualizar`. Si hay piezas sin resumen, resumirlas con un subagente Haiku que siga `prompt_resumen.md`, dejando fuera la sesión en curso.
- Si hay conectores de correo o reuniones, traer lo nuevo desde la fecha del último archivo en `entrada/<fuente>/`, solo con herramientas de lectura (buscar y leer; nunca crear borradores, reenviar, etiquetar ni borrar), siguiendo la regla de `fuentes.json`. Después correr `actualizar` otra vez.
- Antes de responder algo sobre el proyecto (una decisión, un dato, qué dijo alguien), buscar primero con palabras que usaría el documento, probar sinónimos si no aparece, abrir el texto completo de lo que sirva, y decir de qué fecha es lo que se cita. Si dos piezas chocan, manda la más reciente.

Escribe también `_cerebro/LEEME.md` con qué es, cómo funciona y cómo mantenerlo (podar una vez al mes con `descartados.txt`).

## Antes de construir, pregúntame

- Qué carpetas o archivos dejar fuera.
- Qué fuentes quiero aparte de archivos, sesiones y memoria (correo, reuniones, LinkedIn), y desde qué fecha.
- Para cada fuente de correo o mensajes, qué entra y qué no (por ejemplo, solo lo del proyecto y nada personal). Arma la consulta con mis respuestas y muéstramela.
- Si subo `cleanupPeriodDays` en `~/.claude/settings.json` para que Claude Code no borre las sesiones a los 30 días (por ejemplo a 3650). Dime cuánto espacio ocupan hoy.

No leas chats personales (WhatsApp, iMessage, Telegram u otros) ni sus bases locales, aunque estén en el computador, salvo que yo te lo pida de forma explícita. Si un conector necesita que yo inicie sesión o haga clic en "Conectar", avísame y sigue con el resto.

## Cuando termines

Corre `actualizar`, resume todas las piezas con subagentes Haiku en paralelo, y prueba `buscar` con tres preguntas reales sobre el proyecto. Muéstrame los resultados, cuántas piezas quedaron por fuente y qué conectores quedaron pendientes de mi parte.
