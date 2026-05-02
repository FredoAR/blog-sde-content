---
layout: post
title:  "POC — ¿Cómo conectar un dominio de Neubox a Cloudflare Pages?"
date:  2026-04-13 23:34:52 -0600
categories: cloudflare neubox poc github

---

**Dominio:** bytersoft.com  
**Registrador:** Neubox  
**Hosting:** Cloudflare Pages  
**Fecha:** Abril 2026

---

## Contexto

Este documento describe el procedimiento completo para conectar un dominio registrado en **Neubox** a **Cloudflare**, y apuntarlo a un sitio estático desplegado en **Cloudflare Pages**. 
Incluye los errores encontrados durante el proceso y cómo resolverlos.

---

## Prerrequisitos

- Cuenta en [Cloudflare](https://cloudflare.com) creada
- Dominio ya registrado en Neubox
- Proyecto desplegado en Cloudflare Pages
- Acceso al panel de administración de Neubox

---

## Paso 1 — Agregar el dominio en Cloudflare

1. Inicia sesión en el [dashboard de Cloudflare](https://dash.cloudflare.com)
2. Haz clic en **"Agregar un sitio"**
3. Escribe tu dominio (ej. `bytersoft.com`) y selecciona el plan **Free**
4. Cloudflare escanea los registros DNS existentes automáticamente
5. Confirma los registros detectados y continúa

Cloudflare te asignará dos nameservers propios. En este caso:

- `xxxxx.ns.cloudflare.com`
- `yyyyy.ns.cloudflare.com`

> ⚠️ Guarda estos dos valores, los necesitarás en el siguiente paso.

---

## Paso 2 — Corregir los nameservers en Neubox

Este fue el punto crítico del proceso. Después de 24 horas sin propagación, el dashboard de Cloudflare mostraba el siguiente error:

Cloudflare — Nameservers no válidos

![Cloudflare — Nameservers no válidos](/assets/images/posts/poc-cloudflare-dominio-neubox/image.png)

Al revisar la pantalla de información general del dominio, Cloudflare indicaba que seguía esperando la propagación:

Cloudflare — Esperando propagación de nameservers

![Cloudflare — Esperando propagación de nameservers](/assets/images/posts/poc-cloudflare-dominio-neubox/image-1.png)

La causa: en Neubox los nameservers estaban configurados de forma incompleta, solo se había agregado uno de los dos de Cloudflare y los de Neubox seguían activos.

Cloudflare mostraba exactamente qué debía quedar y qué debía eliminarse:

Cloudflare — Instrucciones de nameservers

![Cloudflare — Instrucciones de nameservers](/assets/images/posts/poc-cloudflare-dominio-neubox/image-2.png)

### Procedimiento en Neubox

1. Inicia sesión en tu cuenta de [Neubox](https://neubox.com)
2. Ve a **Mis dominios → Administrar → DNS / Nameservers**
3. **Elimina** todos los nameservers existentes y cualquier entrada parcial de Cloudflare que hubiera quedado:
   - `xxxx.neubox.net`
   - `yyyy.neubox.net`
   - `zzzz.neubox.net`
4. **Agrega únicamente** los dos nameservers asignados por Cloudflare:
   - `xxxx.ns.cloudflare.com`
   - `zzzz.ns.cloudflare.com`
5. Guarda los cambios

> ⚠️ Es importante que queden **exactamente esos dos nameservers y ninguno más**. Una configuración mixta (Neubox + Cloudflare) impide la propagación.

### Verificar propagación

Regresa a Cloudflare y presiona **"Compruebe los servidores de nombres ahora"**.

También puedes verificar desde consola:

```bash
nslookup -type=NS bytersoft.com
```

Cuando los nameservers respondan como `xxxx.ns.cloudflare.com` y `yyyy.ns.cloudflare.com`, la propagación está completa.

Una vez propagado, el dashboard de Cloudflare mostrará:

Cloudflare — Dominio activo y protegido

![Cloudflare — Dominio activo y protegido](/assets/images/posts/poc-cloudflare-dominio-neubox/image-3.png)

---

## Paso 3 — Resolver el error 522 (Connection Timeout)

Con el dominio ya activo en Cloudflare, al navegar a `bytersoft.com` apareció el siguiente error:

Error 522 — Connection timed out

![Error 522 — Connection timed out](/assets/images/posts/poc-cloudflare-dominio-neubox/image-4.png)

**Error 522** significa que Cloudflare llegó bien al dominio, pero no pudo conectarse al servidor de origen.

### Causa

El registro DNS en Cloudflare tenía un CNAME manual apuntando a `bytersoft-project.pages.dev`, pero un CNAME en el **apex del dominio** (`bytersoft.com` sin subdominio) no funciona correctamente de forma manual. Cloudflare requiere que los dominios root para Pages se configuren directamente desde el panel de Pages para aplicar CNAME Flattening automáticamente.

### Solución — Configurar el dominio desde Cloudflare Pages

1. En el sidebar de Cloudflare, ve a **Workers & Pages**
2. Abre tu proyecto (ej. `bytersoft-project`)
3. Ve a la pestaña **Dominios personalizados**
4. Haz clic en **"Configurar un dominio personalizado"**
5. Agrega:
   - `bytersoft.com`
   - `www.bytersoft.com`
6. Cloudflare configura automáticamente los registros DNS correctos

Cloudflare Pages — Dominios personalizados configurándose

![Cloudflare Pages — Dominios personalizados configurándose](/assets/images/posts/poc-cloudflare-dominio-neubox/image-5.png)

> El mensaje **"Domain information not found"** y el banner azul son normales en este momento — el proceso puede tardar entre 15 minutos y 48 horas, aunque en la práctica suele resolverse en menos de una hora.

---

## Paso 4 — Verificar resultado final

Una vez completada la configuración, navegar a `https://bytersoft.com` debe cargar el sitio correctamente con SSL activo (candado en el navegador).

Puedes confirmar el estado desde consola:

```bash
# Verificar que los nameservers apuntan a Cloudflare
nslookup -type=NS bytersoft.com

```

---

## Resumen de errores y soluciones

| Error | Causa | Solución |
|-------|-------|----------|
| "Servidores de nombres no válidos" | Nameservers de Neubox mezclados con uno de Cloudflare | Eliminar todos los nameservers en Neubox y dejar solo los dos de Cloudflare |
| Error 522 — Connection timed out | CNAME manual en el apex del dominio no funciona | Configurar el dominio desde la pestaña "Dominios personalizados" en Cloudflare Pages |

---

## Notas adicionales

- **DNSSEC:** Si está activado en Neubox, desactívalo antes de cambiar los nameservers. Puede interferir con la propagación. Se puede reactivar más tarde desde Cloudflare.
- **Plan Free de Cloudflare** es suficiente para este setup — incluye SSL universal, CDN, y protección básica.
- **Tiempo total del proceso:** Variable. La propagación de nameservers puede tardar hasta 24 horas dependiendo del TTL configurado en Neubox. Una vez correctos, Pages suele activarse en menos de una hora.
