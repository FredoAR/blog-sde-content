---
layout: post
title:  "Firebase Auth en Angular: cómo una doble versión rompía Google Login"
date:  2026-05-21 23:43:52 -0600
categories: angular firebase auth google-login

---

Estaba validando el flujo de inicio de sesión con Google en Angular + AngularFire. A nivel de código, todo parecía correcto, pero en runtime seguían apareciendo errores como estos:

- `Component auth has not been registered yet`
- `Firebase: No Firebase App '[DEFAULT]' has been created - call initializeApp() first`

La causa real no estaba en una sola línea de configuración, sino en una combinación de conflicto de dependencias + contexto de ejecución.

---

## Contexto

Objetivo del flujo:

1. Inicializar Firebase App.
2. Inicializar Firebase Auth.
3. Ejecutar Google Login con popup.
4. Enviar el token al backend para autenticar contra la API.

La inyección de dependencias en Angular se veía bien, pero el registro interno de componentes de Firebase quedaba inconsistente en runtime.

---

## Punto clave de arquitectura: Browser vs SSR

Este punto fue importante para estabilizar la implementación: usar configuración separada para browser y server.

- `src/main.ts` usa `app.config.browser.ts`.
- `src/main.server.ts` usa `app.config.server.ts`.
- Firebase se inicializa solo en browser (`app.config.browser.ts`), no en SSR.

Eso evita errores por APIs exclusivas del navegador (popup OAuth, `indexedDB`, estado de sesión local) cuando el render corre en servidor.

---

## El problema real: doble versión interna de Firebase

Después de revisar stack traces y el árbol de dependencias, encontré mezcla de versiones internas de `@firebase/app`.

Aunque `firebase` estaba en 10.x, el árbol tenía resolución dual de `@firebase/app` en distintos niveles. Eso provoca que una parte del SDK registre componentes en un contenedor interno, y otra parte (como Auth) los intente resolver desde otro.

Resultado: errores de inicialización e inyección que parecen aleatorios.

---

## Señales que llevaron al diagnóstico

1. Error de registro de `auth` aun con providers declarados.
2. Error alternando entre `auth not registered` y `app/no-app`.
3. Comportamiento inconsistente según cómo se abría el navegador.
4. Confirmación con `npm ls @firebase/app` mostrando resolución conflictiva.

---

## Corrección aplicada

La corrección fue forzar una única versión compatible de `@firebase/app` para todo el proyecto y reinstalar dependencias.

```json
{
  "dependencies": {
    "@firebase/app": "0.10.13"
  }
}
```

Luego:

```bash
npm install
npm ls @firebase/app
```

Además, dejé la inicialización de Auth ligada explícitamente a la app inyectada para evitar dependencia implícita del orden:

```typescript
provideAuth((injector) => getAuth(injector.get(FirebaseApp)));
```

---

## Resultado

Después de deduplicar dependencias y mantener la inicialización explícita:

1. Firebase App y Auth iniciaron correctamente.
2. El login con Google funcionó.
3. El backend pudo recibir y validar el token sin errores de inicialización.

---

## Observación sobre Run and Debug

Detecté una diferencia importante:

1. Desde Run and Debug (Chrome lanzado por VS Code), el flujo no siempre funcionaba igual.
2. Desde navegador abierto manualmente en `http://localhost:4200`, el login sí funcionaba.

Esto sugiere diferencias de perfil/sesión del navegador (cookies, popups, extensiones, políticas locales) que impactan el flujo OAuth.

Para pruebas de Firebase Auth, me funcionó mejor este enfoque:

1. Validar el flujo OAuth en navegador normal como prueba de referencia.
2. Usar debugger para revisar lógica de app, no como único validador del login externo.

---

## Aprendizajes

1. Un error de DI en Firebase puede ser realmente un problema de versiones transitorias.
2. En incidentes de Auth, `npm ls` es parte obligatoria del diagnóstico.
3. Separar `app.config.browser` y `app.config.server` evita problemas en SSR.
4. El contexto de ejecución (debugger vs navegador normal) puede alterar flujos OAuth.

---

## Referencias

- [Firebase Auth Web - Google Sign-In](https://firebase.google.com/docs/auth/web/google-signin)
- [Firebase JS SDK](https://github.com/firebase/firebase-js-sdk)
- [AngularFire](https://github.com/angular/angularfire)
