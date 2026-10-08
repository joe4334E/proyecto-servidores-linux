# PREGUNTAS — Desafíos básicos de investigación

> **Uso (docente):** preguntas cortas para investigar en clase. Cada una se responde **haciendo en SU servidor** y mostrando la salida real. No se entregan comandos: solo la meta. Si pegan algo de internet, pidan que lo repitan con su servicio/archivo.

---

## Bloque 1 — El sistema de archivos

**1.** Entra a `/etc`, abre **5 archivos** de texto y explica en una línea cada uno. Identifica cuál dice cómo se llama este equipo, cámbialo y muéstralo escrito.

- *Evidencia:* los 5 abiertos + el archivo del hostname antes/después.

**2.** Intenta leer tu contraseña en `/etc/passwd` y luego en el archivo de la sombra: uno debe fallar sin `sudo`. Además, con un ejemplo real de **tu** equipo, di qué se guarda en `/var/log`, `/usr/bin` y `/proc`.

- *Evidencia:* los dos intentos + un `ls` de cada carpeta.

---

## Bloque 2 — Usuarios y permisos

**3.** Crea el usuario `prueba01` y el grupo `equipo7`, mételo al grupo, demuéstralo con `id` y luego borra ambos.

- *Evidencia:* `id` dentro del grupo + confirmación del borrado.

**4.** Explica en una frase por campo `drwxr-x---`. Después deja a tu compañero **sin acceso** a una carpeta tuya: que él entre por SSH e intente abrirla y **falle**. Revierte después.

- *Evidencia:* tu explicación + el error real del compañero.

**5.** Aplica `chmod 777` a una carpeta de prueba y demuestra **con dos cuentas** qué puede hacer ahora un tercero que antes no podía. ¿Sigue pareciendo solución?

- *Evidencia:* la prueba con las dos cuentas.

---

## Bloque 3 — Servicios y logs

**6.** Nombra **3 servicios activos que tú no instalaste** y di cuáles vuelven solos tras un reiniciar y cuáles no — sin reiniciar.

- *Evidencia:* la lista de servicios + el estado que te da esa respuesta.

**7.** Para un servicio que no sea SSH, encuentra en el log **la hora exacta** en que se cayó, levántalo y verifica. Y muestra `systemctl --failed` con una unidad fallida (provócala).

- *Evidencia:* la línea del log con hora + la lista de fallidos.

---

## Bloque 4 — Red y puertos

**8.** ¿Cuál es la IP de tu VM? Descúbrelo y compruébalo **desde dos lugares** (dentro de la VM y desde tu laptop). Luego lista qué escucha tu servidor y explica **2 puertos concretos**.

- *Evidencia:* la IP en ambos lugares + la lista de puertos con tu explicación.

**9.** Demuestra desde tu laptop que un puerto **cerrado** no responde (y desde la VM sí sabes que el servicio atiende). ¿Qué se expondría si alguien abriera el 3306 al mundo?

- *Evidencia:* intento fallido desde la laptop + tu justificación.

---

## Bloque 5 — Instalar algo

**10.** Instala `tree` **sin copiar el comando**. Después muestra qué archivos dejó el paquete, dónde quedó su binario, y desinstálalo **con su configuración** demostrando que no quedó nada.

- *Evidencia:* `tree` corriendo → listado de archivos → búsquedas vacías tras el purge.

**11.** Instala un programa útil que **tú elijas** para administrar. Y esto: un script/comando tuyo falla con `command not found` — mirando el error, di en qué distro estás y qué paquete provee el comando que falta.

- *Evidencia:* el programa en uso + el error original + el paquete dueño.

---

## Bloque 6 — Tu primer servicio web

**12.** Consigue que `http://TU_IP/` muestre **una página tuya** (con tu nombre) **desde la laptop**. Si la página se *descarga* en vez de verse, averigua qué le falta al servidor.

- *Evidencia:* la página abierta desde la laptop + qué instalaste/configuraste.

---

## Cómo responder

| Criterio | Sí | No |
|---|---|---|
| Evidencia | salida real de SU servidor | respuesta teórica o pegada |
| Entendimiento | pueden repetirlo explicándolo | copiaron sin entender |
| Desde fuera | probaron desde la laptop | "funciona" visto desde la VM |
