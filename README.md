# segundo-cerebro

Un prompt para que Claude Code arme un "segundo cerebro" de tu proyecto: una
base con todo lo que se ha hecho (archivos, sesiones con Claude, memoria y,
si quieres, correo, reuniones y LinkedIn), con fecha y resumen por pieza, que
Claude consulta antes de responderte.

- Respalda las sesiones de Claude Code, que se borran a los 30 días.
- Le pone fecha a todo y lo reciente pesa más, para que lo viejo no se mezcle con lo nuevo.
- Busca primero y lee solo lo que sirve, en vez de meter todo en la conversación.

Funciona con un script de Python sin dependencias, resúmenes hechos por Haiku y
búsqueda BM25 sobre texto plano.

## Usar

1. Abre Claude Code en la carpeta de tu proyecto.
2. Pega el contenido de [PROMPT.md](PROMPT.md) (también está en Word: `PROMPT.docx`).
3. Responde las preguntas que te hace antes de construir: qué dejar fuera, qué
   fuentes sumar y qué correos o mensajes entran.

Los conectores solo leen. Claude no lee chats personales salvo que se lo pidas.
