# tenth-man

Un plugin de deliberación con varios modelos (una skill y los agentes que ocupan los puestos del consejo) para [Codex](https://developers.openai.com/codex) y [Claude Code](https://claude.com/claude-code). Úsalo para una decisión de diseño, la revisión de una política u otra decisión que no deba depender de la respuesta de un solo modelo. Varios agentes con modelos distintos votan y un puesto fijo siempre argumenta en contra, así que es menos probable que el consejo llegue a un acuerdo compartiendo el mismo punto ciego.

[English](README.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (Brasil)](README.pt-BR.md) | [Deutsch](README.de.md) | [Français](README.fr.md)

Los miembros con voto evalúan cada propuesta. Un **décimo hombre** (tenth man) aparte debe argumentar en contra de cada propuesta, sin importar su propia opinión, e indicar qué evidencia refutaría cada objeción. La sesión principal preside tres rondas, resuelve las preocupaciones graves y redacta un acta que muestra qué se acordó, por qué y qué queda pendiente.

La skill siempre presenta primero su plan del consejo y espera tu aprobación. Ejecutar un consejo nunca aplica sus conclusiones a tu proyecto de forma automática.

## Cómo funciona

1. El presidente propone los puntos, los modelos, las reglas, el costo y la carpeta de salida.
2. Ronda 1: los miembros votan; el décimo hombre cuestiona cada punto.
3. Ronda 2: los miembros reconsideran las preocupaciones graves y responden a las objeciones más importantes.
4. Ronda 3: los miembros votan el borrador del acta; el décimo hombre contrasta el resultado con la evidencia.
5. El presidente escribe `.tenth-man/<date>-<slug>/minutes.md`.

Los votos son **a favor**, **a favor con condiciones**, **en contra** y **preocupación grave**. El acuerdo requiere una mayoría de los miembros con voto y que no quede ninguna preocupación grave. El décimo hombre no tiene voto.

## Instalación en Codex

### Como plugin

```sh
codex plugin marketplace add Shintaro-My/tenth-man
codex plugin add tenth-man@tenth-man
```

Para una copia local del repositorio, añade su ruta en su lugar:

```sh
codex plugin marketplace add /path/to/tenth-man
codex plugin add tenth-man@tenth-man
```

Invoca la skill del plugin escribiendo `$tenth-man:council` seguido de la pregunta, o elígela en `/skills`:

```text
$tenth-man:council ¿Deberíamos mover el estado de sesión de Redis a cookies firmadas?
```

### Como skill independiente

Copia `skills/council` a un directorio de skills de Codex:

```sh
cp -r skills/council ~/.agents/skills/
```

Si la instalas de esta forma, invócala como `$council`.

Para un solo repositorio, cópiala en `<repo>/.agents/skills/`. Este repositorio también contiene en `.codex/agents/` perfiles de Codex sin rol fijo para los puestos; Codex los detecta automáticamente en una copia local del repositorio. Para usar los perfiles desde cualquier repositorio:

```sh
cp .codex/agents/seat-*.toml ~/.codex/agents/
```

La skill sigue funcionando sin esa copia: puede iniciar subagentes con las mismas asignaciones explícitas de modelo y mantener las conversaciones de los subagentes entre rondas. El empaquetado como plugin no instala automáticamente los archivos TOML de agentes personalizados.

## Instalación en Claude Code

### Como plugin

```text
/plugin marketplace add Shintaro-My/tenth-man
/plugin install tenth-man@tenth-man
```

Para probar una copia local del repositorio:

```sh
claude --plugin-dir /path/to/tenth-man
```

Invócala con:

```text
/tenth-man:council ¿Deberíamos mover el estado de sesión de Redis a cookies firmadas?
```

### Manualmente

```sh
cp -r skills/council ~/.claude/skills/
cp agents/seat-*.md ~/.claude/agents/
```

Con la instalación manual, el comando es `/council`; los nombres de los agentes no llevan el prefijo `tenth-man:`.

## Presets de modelos

El presidente propone `light` para una pregunta pequeña y `standard` en los demás casos. Al aprobar el plan, puedes elegir un preset o cambiar cualquier puesto.

El décimo hombre usa el mismo modelo que el miembro con mayor capacidad de razonamiento. El orden se basa en resultados medidos, no en los números de versión; las columnas “Décimo hombre” de las tablas siguientes reflejan esa clasificación en octubre de 2026. Cuando no se conoce la posición de un modelo nuevo, el presidente te pregunta. Para usar otro modelo, indícalo al dar tu aprobación.

### Codex

| Preset | Miembros con voto | Décimo hombre | Ejecuciones por ronda |
|---|---|---|---|
| `light` | GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra | GPT-6 Astra | 4 |
| `standard` | GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra, GPT-6 Sol | GPT-6 Astra | 5 |
| `no-astra` | GPT-6.1 Sol, GPT-6.1 Terra, GPT-6 Sol | GPT-6.1 Sol | 4 |

Estos valores predeterminados solo usan los modelos que ofrecen el host de Codex y la cuenta actuales. El presidente informa de los modelos no disponibles y nunca sustituye uno sin avisar. Si el límite de concurrencia del host es menor que el número de puestos del consejo, los agentes se ejecutan en tandas con las mismas instrucciones sin cambios; no se hace ningún recuento hasta que terminan todos.

Los agentes personalizados de Codex correspondientes son `seat_astra`, `seat_sol`, `seat_terra`, `seat_previous_sol` y el opcional de menor costo `seat_luna`. Sus definiciones TOML fijan el modelo y el esfuerzo de razonamiento, y mantienen el trabajo del consejo en modo de solo lectura.

### Claude Code

| Preset | Miembros con voto | Décimo hombre | Ejecuciones por ronda |
|---|---|---|---|
| `light` | Fable, Opus, Sonnet | Fable | 4 |
| `standard` | Fable, Opus, Sonnet, Opus 5, Opus 4.8 | Fable | 6 |
| `no-fable` | Opus, Sonnet, Opus 5 | Opus | 4 |

Todos los presets mantienen un puesto de Opus aunque Fable esté mejor clasificado, porque Opus a veces encuentra problemas que Fable pasa por alto.

El plugin de Claude incluye en `agents/` cinco agentes sin rol fijo para ocupar los puestos. Las instrucciones de cada ejecución le asignan el rol de miembro o de décimo hombre.

### Añadir un modelo de Claude

Se puede seleccionar un modelo de Claude más antiguo mediante el campo `model:` de una definición de agente. Copia uno de los archivos de agentes incluidos, cambia `name`, `description` y `model`, y guárdalo en `~/.claude/agents/` para todos los proyectos o en `.claude/agents/` para un solo proyecto:

```markdown
---
name: seat-opus-4-7
description: A seat in a tenth-man council, on Opus 4.7.
model: claude-opus-4-7
effort: high
tools: Read, Grep, Glob, Bash, Write
---
```

Mantén el mismo contenido sin rol fijo que los archivos de agentes incluidos. Así, el presidente puede asignar `seat-opus-4-7` como miembro o como décimo hombre.

### Usar tus propios agentes de Claude

Cualquier definición de agente de Claude existente puede ocupar un puesto. El presidente empieza su prompt indicando que, en esta tarea, el agente ocupará un puesto en el consejo en lugar de hacer su trabajo habitual, y luego le asigna el rol y las instrucciones comunes. Esto es útil para agentes de revisión fijados a un modelo más antiguo.

### Cuando se retira un modelo de Claude fijado

Si un modelo fijado deja de estar disponible, el agente de ese puesto no logra iniciarse. El presidente lo retira y te avisa; nunca lo sustituye por otro modelo sin avisar. Elimina o actualiza el archivo de ese agente para que coincida con los modelos disponibles en tu cuenta.

## Costo y requisitos

Cada puesto supone una ejecución completa de un subagente por ronda. En tres rondas, el preset `standard` de Codex usa 15 ejecuciones y el preset `standard` de Claude Code usa 18.

- Codex: una versión actual con los subagentes habilitados y acceso a los modelos seleccionados.
- Claude Code: las herramientas Agent y SendMessage, las notificaciones de finalización de agentes en segundo plano y acceso a los modelos seleccionados.

El chat de ChatGPT no es por ahora un destino compatible, porque no puede ejecutar subagentes con un modelo distinto para cada puesto.

## ¿Por qué “décimo hombre”?

La skill se inspira en “el décimo hombre”, una regla israelí ficticia de la película *World War Z* de 2013. Cuando nueve personas llegan a la misma conclusión a partir de la misma información, la décima persona asume que están equivocadas e investiga lo contrario. La regla es ficticia, pero existen prácticas similares:

- Después de que la inteligencia militar israelí interpretara mal las señales de alerta antes de la guerra de Yom Kipur de 1973, creó una función para cuestionar sus propias evaluaciones, conocida comúnmente como *ipcha mistabra*, que en arameo significa “lo contrario es probable”.
- El Talmud (Sanhedrin 17a) recoge que, si todos los jueces votan por condenar en un caso de pena capital, el acusado queda libre; un resultado unánime sugiere que nadie argumentó debidamente a favor de la defensa.
- En las causas de canonización católicas, el “abogado del diablo” argumentaba en contra del candidato y examinaba los puntos débiles de los relatos sobre su vida y de los milagros que se le atribuían. El cargo se formalizó en 1587 y se redujo mucho en 1983.

## Licencia

MIT. Consulta [LICENSE](LICENSE).
