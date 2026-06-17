---
layout: post
title: "El problema de centralizar mis archivos de GitHub Copilot (y cómo lo estoy resolviendo)"
date: 2026-06-16 00:00:00 -0600
categories: ia
tags: [copilot, backend]
excerpt: "Intenté centralizar mis instructions, agents y prompts para reutilizarlos entre proyectos. Terminé con archivos genéricos que no servían bien a ningún caso y créditos de Copilot que se agotaban rápido."
---

Estaba agregando una propiedad nueva a una entidad y su configuración en Entity Framework. Dos archivos, pocas líneas. El resultado: un gasto de tokens que no me hizo sentido con el tamaño del cambio.

Ahí fue donde me detuve a evaluar qué estaba pasando.

No es que Copilot no me sirviera — ese mes avancé bien y sentí que lo aproveché. Pero me quedé sin créditos a mitad de mes, y la desproporción entre una tarea pequeña y el costo que tuvo me hizo revisar toda mi estrategia.

El problema no era Copilot. Era cómo yo había configurado el contexto que cargaba en cada request.

---

Cuando empecé a adoptar GitHub Copilot más allá del autocompletado básico — usando agentes, instrucciones prompt files y skills — lo primero que quise fue no repetir. Un archivo en `~/.copilot/`, disponible en todos mis proyectos, sonaba como la solución correcta.

No lo era. O al menos, no completamente.

---

## El problema que quería resolver

Trabajo como dev backend .NET en varios proyectos: algunos personales bajo mi marca [bytersoft](https://bytersoft.com). Todos siguen una arquitectura similar — Clean Architecture, DDD, Entity Framework Core, .NET. Tenía sentido definir mis agentes y instrucciones una vez y no duplicarlos en cada repo.

La idea era buena. La ejecución tuvo problemas.

---

## Lo que salió mal

### Los archivos crecieron sin control

Mis agentes terminaron siendo documentos muy largos que intentaban cubrir demasiado. Por ejemplo, mi agente `backend-architect` tenía workflows para validar ubicación de lógica, detectar violaciones de dependencias, modelar aggregates, implementar Domain Events, refactorizar hacia DDD, y una sección completa de patrones de referencia con ejemplos de código.

Todo correcto y útil. Pero no todo al mismo tiempo.

Cuando Copilot cargaba ese agente, metía cientos de tokens de contexto que en la mayoría de las interacciones no eran relevantes. Un request simple como "¿dónde va esta validación?" no necesita el diagrama de CQRS ni los ejemplos de Unit of Work.

Lo mismo con `documentator`: templates para XML docs, guías técnicas, README, ADRs, diagramas Mermaid en cuatro formatos distintos. Un agente que intenta ser todo termina siendo costoso en contexto aunque la tarea sea pequeña.

### Paths absolutos mezclados con contenido reutilizable

El error más obvio: hardcodear rutas específicas dentro de archivos que supuestamente serían genéricos.

Ejemplos reales que tenía:

```markdown
**Ubicación**: `/docs/{FEATURE}_GUIDE.md`
**Ubicación**: `src/Alb.Project/docs/`
**Ubicación**: `docs/architecture/decisions/ADR-{número}-{slug}.md`
```

El problema con `src/Alb.Project/` es que eso es el namespace de un proyecto específico. Si copio el agente a otro proyecto, esa ruta está mal desde el día uno.

La corrección no es solo hacerla relativa. Es preguntarse si ese dato pertenece al agente o a la configuración del proyecto.

### Centralizar en `~/.copilot/` sin versionar

`~/.copilot/` no es un repositorio. No tiene historial, no tiene backup fácil, no se puede revisar qué cambió entre versiones. Estaba evolucionando mis archivos sin ningún rastro de las decisiones que tomé.

---

## El diagnóstico real

El problema de fondo no era la centralización — era que estaba escribiendo los archivos *pensando en un caso concreto* sin una estructura mental clara de qué va en cada tipo de archivo.

Cada tipo tiene un propósito distinto:

| Tipo | Propósito |
|------|-----------|
| **Instructions** | Estándares que siempre aplican (idioma, formato, convenciones) |
| **Agents** | Rol + metodología de trabajo |
| **Prompt files** | Contexto situacional para un task específico |
| **Skills** | Capacidad de ejecución especializada |

Cuando mezclas propósitos — por ejemplo, meter templates de código dentro de un agente — el archivo pierde cohesión y el contexto que Copilot carga sube innecesariamente.

---

## La solución: un repo como fuente de verdad

En lugar de mantener archivos globales en `~/.copilot/`, voy a centralizar todo en un repositorio dedicado — algo como `ai-config` — con esta estructura:

```markdown
ai-config/
├
├── instructions/
│   ├── base.md           # Español, Conventional Commits, tono
│   └── dotnet-backend.md # Convenciones de mi stack
├── agents/
│   └── documentador.md
├── prompts/
│   └── ticket-analysis.prompt.md
└── skills/
│       └── backend-skill.md
```

Cuando inicio un proyecto, copio de `ia-config/` lo que aplica y lo ajusto. La repetición entre proyectos es **intencional**: cada proyecto tiene exactamente el contexto que necesita, sin cargar lo que no le corresponde.

---

## La convención que me faltaba

Para evitar que los archivos vuelvan a crecer sin estructura, adopté una plantilla mínima por tipo:

**Instructions**

```markdown
# [Nombre]
## Propósito
Una línea.
## Reglas
Lista concisa. Sin ejemplos de código, sin paths.
```

**Agents**
```markdown
# [Nombre del agente]
## Rol
Qué es, con qué mentalidad opera.
## Comportamientos
Qué hace en cada situación.
## Output esperado
Formato de respuesta.
## Lo que NO hace
```

**Prompt files**
```markdown
# [Task]
## Contexto
Qué ya existe, qué problema se resuelve.
## Input esperado
## Output esperado
## Restricciones
```

**Skills**
```markdown
# [Skill name]
## Cuándo activar
## Conocimiento necesario
## Pasos de ejecución
## Criterios de calidad
```

### La regla para los paths

Si un path tiene que estar, que describa la convención del proyecto, no una ruta de máquina:

```markdown
## Convención de estructura
- API: src/Api/
- Dominio: src/Domain/
- Tests: tests/
```

Así el archivo documenta *tu arquitectura estándar*. Cualquier proyecto que la siga puede usarlo sin editar.

---

## Lo que aprendí

Centralizar tiene sentido, pero el mecanismo importa. `~/.copilot/` es conveniente pero frágil. Un repo versionado es más fricción inicial y más control real.

La repetición entre proyectos no es duplicación descuidada — es una inicialización deliberada. Cada proyecto toma lo que necesita de la fuente de verdad y lo adapta. Eso es mejor que un contexto global que aplica a todo aunque no sea relevante.

Y el problema de fondo: si escribes un archivo de agente pensando en un proyecto específico en lugar de pensar en el patrón, terminas con paths hardcodeados, secciones que no aplican, y contexto innecesario que tiene un costo real en tus créditos de Copilot.

---

## Referencias

- [GitHub Copilot — Customizing Copilot](https://docs.github.com/en/copilot/customizing-copilot) — Documentación oficial sobre instructions, agents y prompt files
- [About customizing GitHub Copilot in your organization](https://docs.github.com/en/copilot/customizing-copilot/about-customizing-github-copilot-in-your-organization) — Contexto sobre la jerarquía de configuración
