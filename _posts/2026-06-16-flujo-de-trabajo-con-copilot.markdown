---
layout: post
title: "Cómo estructuré mi flujo de trabajo con GitHub Copilot como dev backend"
date: 2026-06-16 00:00:00 -0600
categories: ia
tags: [copilot, backend]
excerpt: "Durante un tiempo usé Copilot solo para pedir código y revisiones. Cuando empecé a pensar en él como un flujo con etapas distintas — Ask, Plan, Agent — y a calibrar el nivel de autonomía por tarea, cambió la forma en que trabajo."
---

Durante un tiempo usé GitHub Copilot de una forma bastante plana: preguntaba cosas, pedía código, pedía revisiones, generaba XML docs o tests cuando los necesitaba. Funcionaba, pero sentía que no estaba aprovechando la herramienta.

El cambio llegó cuando dejé de pensar en Copilot como un asistente al que le haces preguntas, y empecé a pensarlo como un flujo con etapas distintas — cada una con un propósito y una forma de calibrarlo.

---

## Los tres modos como etapas del desarrollo

GitHub Copilot tiene tres modos: **Ask**, **Plan** y **Agent**. En la práctica los uso como fases de trabajo, no como opciones intercambiables.

### Ask — exploración y análisis

Antes de escribir una sola línea de código, uso Ask para entender el problema. Preguntas sobre conceptos que no conozco bien, identificar participantes en un flujo, saber si lo que estoy enfrentando es un patrón conocido en la industria.

Un ejemplo concreto: cuando decidí implementar Identity + JWT con Google Auth a través de Firebase para un flujo OIDC, me topé con varios conceptos nuevos al mismo tiempo — authorization codes, id_tokens, el rol de Firebase como provider, cómo encajaba con mi propio servidor de identidad. Usé Ask intensamente para desmenusar el flujo, identificar cada componente y ubicar el patrón dentro de lo que ya conocía de implementaciones similares en mi trabajo.

Ask no produce código. Produce entendimiento. Ese es su valor en esta etapa.

### Plan — decisiones, fases y alcance

Una vez que entiendo el problema, cambio a Plan para tomar decisiones y armar un orden de ejecución. Qué implementar primero, dónde poner los límites del alcance, qué alternativas evaluar antes de comprometerse con una dirección.

En el mismo ejemplo del flujo OIDC, Plan me ayudó a decidir cómo separar las responsabilidades entre Identity, Firebase y mi propia capa de tokens de sesión, y a definir en qué fases atacar la implementación para no intentar todo al mismo tiempo.

Plan es donde se toman las decisiones de arquitectura antes de implementar. Si salto directo a Agent sin pasar por aquí, termino implementando más rápido pero en la dirección equivocada.

### Agent — implementación

Con el entendimiento de Ask y el plan de Plan, Agent implementa. Aquí es donde se escribe código real, se modifican archivos, se ejecutan pasos encadenados.

También es donde uso mis propios agentes definidos en archivos: `backend-architect` para validar decisiones de Clean Architecture y DDD, `backend-developer` para la implementación, `backend-code-reviewer` para revisión, `documentator` para XML docs y guías técnicas.

El flujo típico en una implementación no trivial sigue ese orden — architect primero si hay decisiones de diseño, developer para el código, reviewer al final.

---

## Los niveles de autonomía: low, middle, high

Dentro de cada modo, Copilot permite calibrar cuánta autonomía tiene para tomar decisiones por su cuenta. Esto cambió bastante cómo uso la herramienta.

Mi regla general:

**Middle** es mi default. La mayoría de las tareas caen aquí — tengo contexto suficiente para guiar y quiero que Copilot ejecute sin preguntarme en cada paso, pero tampoco sin inventar demasiado.

**High** cuando la tarea involucra múltiples archivos o varios pasos encadenados. Le doy el marco, él ejecuta con más libertad dentro de ese marco. También lo uso cuando no tengo mucho contexto del problema y prefiero que explore más antes de proponer.

**Low** para cambios pequeños y focalizados — pocas líneas sobre un solo archivo. No necesito que razone mucho, solo que ejecute con precisión.

La combinación modo + nivel es lo que realmente calibra el comportamiento. Ask en high es muy distinto a Agent en high.

---

## Lo que todavía estoy construyendo

Este flujo no está terminado. Hay dos áreas donde sigo explorando:

**Prompt files** — son el tipo de archivo que más me ha costado definir. Instructions, agentes y skills los tengo relativamente claros. Pero identificar qué situaciones merecen un prompt file dedicado, y cómo escribirlo para que aporte contexto sin repetir lo que ya está en otras partes, es algo en lo que sigo trabajando.

**MCP** — es el siguiente nivel que estoy explorando. Conectar Copilot con herramientas externas como Atlassian o Context7 desde el mismo flujo tiene mucho potencial, pero merece su propio post.

---

## Lo que cambió

Antes mi uso de Copilot era reactivo — lo llamaba cuando necesitaba algo puntual. Ahora hay una intención detrás de cada interacción: en qué etapa estoy, qué nivel tiene sentido para esta tarea, qué agente aplica.

No es un flujo rígido. A veces vuelvo a Ask en medio de una implementación porque algo no cuadra. Pero tener el modelo mental de las etapas evita el error más común que cometía: pedirle código a Copilot antes de tener claro qué quería construir.

---

## Referencias

- [GitHub Copilot Chat modes](https://docs.github.com/en/copilot/using-github-copilot/asking-github-copilot-questions-in-your-ide) — Documentación oficial sobre Ask, Plan y Agent
- [Customizing GitHub Copilot](https://docs.github.com/en/copilot/customizing-copilot) — Instructions, agents, prompt files y skills
