# PREGUNTAS — Desafíos de investigación

> **Cómo funciona (para el docente):** son misiones, no ejercicios de copiar. Cada pregunta pide un estado **concreto de esa máquina** — las respuestas del salón son todas distintas. No se entrega ningún comando: solo la meta. La *pista* se destapa solo si se traban, y apunta al **lugar o concepto**, nunca al comando.
>
> **Evidencia obligatoria:** toda respuesta se apoya en la salida real de SU servidor (pantalla, log o archivo). Sin evidencia, no hay respuesta.
>
> **Si pegan un comando de internet:** no lo rechaces — pregúnteles *"ahora hazlo con el servicio X"* o *"y eso, ¿qué escribió en tu servidor?"*. Si no lo pueden variar, no lo entendieron.
>
> Los bloques 1–5 acompañan la Clase 1 (investigan mientras avanzan); el bloque 6 es el puente al proyecto.

---

## Bloque 1 — El territorio: `/etc` y el sistema de archivos

**1.** Recorre `/etc` y elige **8 archivos** que sean texto. Abre cada uno y explica con tus palabras, una línea por archivo, qué regula. (No vale "es de configuración": di *qué* configura.)

- *Evidencia:* tus 8 líneas + al menos 3 de los archivos abiertos.
- *Pista:* hay uno de ellos que decide cómo se llama esta máquina.

**2.** Tu servidor tiene un hostname. Encuentra **dónde está escrito de verdad**, cámbialo por otro nombre y demuestra en qué archivo quedó la escritura.

- *Evidencia:* contenido del archivo antes/después + `hostname` actual.
- *Pista:* hay dos lugares donde puede estar; el que manda no siempre es el primero que miras.

**3.** ¿Por qué tu contraseña **no** está en `/etc/passwd` aunque seas administrador del equipo? Localiza el archivo que sí la guarda e intenta leerlo: primero sin `sudo`, luego con `sudo`. Explica la diferencia.

- *Evidencia:* los dos intentos (el primero debe fallar).
- *Pista:* el nombre del archivo de la sombra.

**4.** En `/var` hay tres carpetas que un administrador abre a diario. Identifícalas, di qué guarda cada una y muestra un archivo real de cada una **de tu servidor**.

- *Evidencia:* `ls` de cada carpeta con contenido real.
- *Pista:* una guarda logs, otra programas instalados... no, revisa: solo dos de esas tres las usarás esta semana.

**5.** `/proc` no es un disco: abre `/proc/cpuinfo` y `/proc/uptime` y di qué están mostrando. ¿Se pueden modificar esos archivos? Compruébalo y explica por qué.

- *Evidencia:* las dos lecturas + tu intento de escritura.
- *Pista:* el nombre dice "proceso": ahí vive el estado del sistema **en este momento**.

---

## Bloque 2 — Usuarios, grupos y permisos

**6.** Crea el usuario `prueba01` y el grupo `equipo7`. Mete al usuario al grupo y demuestra con la salida de identidad que entró. Después borra ambos.

- *Evidencia:* `id` del usuario dentro del grupo y confirmación del borrado.
- *Pista:* crear; agregar a un grupo (**ojo:** sin la opción correcta lo *reescribes*, no lo agregas).

**7.** Escribe en una frase cada campo de este permiso: `drwxr-x---`. Después, crea un directorio con esos permisos exactos y muestra con `ls -l` que quedó así.

- *Evidencia:* tu explicación campo por campo + el `ls -l` del directorio nuevo.
- *Pista:* orden fijo: dueño, grupo, otros — cada uno con tres posiciones.

**8.** Deja a tu compañero **sin acceso** a una carpeta tuya. Que él entre por SSH e intente abrirla: su intento debe **fallar**. Después revierte.

- *Evidencia:* la salida del compañero (error real, no simulada).
- *Pista:* depende de quién es dueño y de los últimos tres dígitos.

**9.** Crea el archivo `/tmp/dueno_demo`, ponle de dueño a otro usuario y explica: ¿qué puede y qué no puede ahora ese dueño? ¿Y qué pasa si el archivo es de él pero está dentro de un directorio que no puede pisar?

- *Evidencia:* cambio de dueño + salida de `namei -l` de la ruta completa.
- *Pista:* los permisos se evalúan **nivel por nivel** de la ruta.

**10.** Busca en tu servidor **tres archivos con permisos distintos entre sí** (uno legible por todos, uno restringido, uno ejecutable). ¿Por qué tiene ese permiso y no otro? Justifica cada uno según su función.

- *Evidencia:* los tres `ls -l` + tu justificación.
- *Pista:* compara `/etc`, `/var/log` y algún binario en `/usr/bin`.

**11.** Un compañero dice que `chmod 777` resuelve todo. Pídele que lo aplique a una carpeta de prueba; después demuestren **con dos cuentas** qué se puede hacer ahora que antes no se podía. Concluyan si era "solución".

- *Evidencia:* la prueba con las dos cuentas.
- *Pista:* la pregunta no es "¿funciona?", es "¿a quién deja entrar de más?".

---

## Bloque 3 — Servicios y logs

**12.** Nombra **tres servicios activos en tu servidor que tú no instalaste** y di, de cada uno, para qué sirve (con tus palabras, investigando qué proceso es).

- *Evidencia:* lista de servicios activos + tu descripción de los tres.
- *Pista:* mira qué está corriendo, no qué está instalado.

**13.** De todos tus servicios activos, marca cuáles **vuelven solos tras un reinicio** y cuáles no. ¿Cómo lo sabes **sin reiniciar**?

- *Evidencia:* el comando/estado que te da esa respuesta, con salida real.
- *Pista:* hay una diferencia entre "está corriendo" y "arranca con el sistema". (Fase 2 de la clase.)

**14.** Rompe algo de verdad: para un servicio que no sea SSH. Encuentra en el log **el momento exacto con hora** en que se cayó y escríbelo. Después levántalo y verifica.

- *Evidencia:* la línea del log con fecha y hora + verificación de que volvió.
- *Pista:* el diario del sistema guarda todo por servicio.

**15.** `systemctl --failed`: si está vacío, provócalo (deja algo caído) y muéstralo con al menos una unidad fallida. Di qué te dice esa lista y cómo la vacías.

- *Evidencia:* lista llena → diagnosis → lista vacía.
- *Pista:* no basta con que el servicio esté "detenido": tiene que estar en estado de **fallo**.

**16.** Busca en los logs el error o advertencia **más viejo** que haya en tu servidor y di a qué servicio pertenece. ¿Le afecta hoy algo por eso?

- *Evidencia:* el mensaje con su fecha + tu diagnóstico.
- *Pista:* el diario no empieza hoy; hay filtros por nivel y por fecha.

---

## Bloque 4 — Red

**17.** ¿Cuál es la IP de tu VM? Descúbrelo tú (sin buscar "cómo ver la IP" — dedúcelo) y demuestra que es correcta desde **dos lugares distintos**: dentro de la VM y desde tu laptop.

- *Evidencia:* las dos salidas.
- *Pista:* la interfaz tiene un nombre; la dirección aparece con su máscara.

**18.** Lista todo lo que escucha tu servidor. Explica **dos puertos concretos** (qué proceso y qué protocolo). ¿El 80 está abierto antes de instalar Apache?

- *Evidencia:* la lista completa + tu explicación de dos líneas.
- *Pista:* "escuchar" es de TCP/UDP; el proceso dueño aparece al final de la línea.

**19.** Demuestra desde tu laptop (fuera de la VM) que un puerto **cerrado** no responde. ¿Por qué el mismo puerto sí responde desde dentro de la VM?

- *Evidencia:* los tres intentos: dentro-abierto, dentro-cerrado, fuera-cerrado.
- *Pista:* hay dos filtros distintos: el que atiende y el que deja pasar.

**20.** Si alguien hiciera `ufw allow 3306/tcp` hoy, ¿qué se expondría y quién podría conectarse? Responde **antes de hacerlo**, y después comprueba el estado actual del firewall para mostrar que no está así.

- *Evidencia:* justificación + `ufw status` real.
- *Pista:* ¿qué servicio usa el 3306 y con qué usuario debe hablar la app?

---

## Bloque 5 — Paquetes e instalación

**21.** Instala el programa `tree` **sin copiar el comando**: averigua cómo se instala paquetes en tu distro y hazlo tú. Demuéstralo funcionando sobre una carpeta.

- *Evidencia:* `tree` corriendo + la versión instalada.
- *Pista:* el gestor de paquetes de Debian/Mint; actualiza la lista antes.

**22.** El paquete que acabas de instalar dejó archivos en varias carpetas. Muestra **la lista exacta** y señala dónde quedó su binario.

- *Evidencia:* listado de archivos del paquete + ruta del ejecutable.
- *Pista:* hay un comando que pregunta "¿qué dejó este paquete instalado?".

**23.** Desinstala `tree` **con su configuración** y demuestra que no quedó nada (ni el binario ni archivos de config). Si no quedó nada, ¿por qué existía un modo de desinstalar "con todo"?

- *Evidencia:* búsquedas posteriores vacías.
- *Pista:* purge vs el borrado normal; investiga la diferencia.

**24.** Un script tuyo falla con `comando: command not found`. ¿Cómo sabes, **mirando el error**, en qué distro estás y qué nombre tendrías que usar? Averigua qué paquete provee el comando que te falta.

- *Evidencia:* el error original + el paquete dueño del comando.
- *Pista:* el error ya te dice qué shell lo lanza; el proveedor de un binario se puede preguntar.

**25.** Instala algo que **no viene en tu sistema y que tú escojas** (algo útil para un admin). Justifica qué es, por qué lo elegiste y muéstralo en uso.

- *Evidencia:* el programa corriendo + una línea de "para qué me sirve".
- *Pista:* pensá en algo que te ayude a mirar procesos, puertos o logs mejor.

---

## Bloque 6 — Misión LAMP: instalar sin procedimiento

> **Objetivo general:** convertir tu VM limpia en un servidor LAMP funcional. Aquí nadie te da pasos: investigas, instalas, escribes y documentas. Ver el mapa del `PROGRAMA.md` §0.

**26.** Consigue que `http://TU_IP/hola.php` muestre: **"Hola, soy TU_USUARIO desde TU_HOSTNAME"** (que salga escrito por PHP, no un HTML estático).

- *Evidencia:* la página abierta **desde tu laptop**.
- *Pista:* son tres piezas que trabajan juntas: el que recibe peticiones, el que ejecuta PHP y el servicio que debe estar activo. Si la página se *descarga* en vez de verse, falta una pieza.

**27.** Crea la base de datos de la empresa con una tabla de clientes (inventa 5 filas reales), crea un usuario de aplicación **que no sea root** y conecta `hola.php` a esa tabla: la página debe mostrar un cliente real desde MariaDB.

- *Evidencia:* página con el dato + `SHOW GRANTS` de tu usuario de app.
- *Pista:* dos niveles de usuarios distintos (sistema vs BD); la app habla en local.

**28.** Escribe **tu** `backup.sh` que respalde esa base de datos con **la fecha en el nombre del archivo**. Ejecútalo, restaura el respaldo y comprueba que los 5 clientes siguen ahí.

- *Evidencia:* el archivo de respaldo con fecha, tu script, y la restauración verificada.
- *Pista:* un script es un archivo de texto con permiso de ejecución; la fecha se pide al sistema al momento de correr.

**29.** Haz que tu `backup.sh` se ejecute **solo, cada 5 minutos, sin que nadie lo toque**. Después demuestra que corrió (el archivo de respaldo apareció solo).

- *Evidencia:* respaldos nuevos aparecidos + la configuración que lo programa.
- *Pista:* el sistema tiene un programador de tareas; su configuración es una línea con cinco campos.

**30.** Prueba final: para Apache y para MariaDB, demuestra desde **tu laptop** que funcionan, y luego demuestra que **vuelven solos** después de un reinicio. Escribe qué probaste y con qué.

- *Evidencia:* las cuatro pruebas (2 servicios × fuera + tras reinicio).
- *Pista:* lo que vuelve al reiniciar es lo que estaba *habilitado*, no solo *activado*.

---

## Rúbrica rápida de respuesta

| Criterio | Sí | No |
|---|---|---|
| Evidencia real de SU servidor | salida copiada de su máquina | respuesta teórica o de internet |
| Entendimiento | pueden variar el comando y explicarlo | repiten la secuencia sin entender |
| Verificación desde fuera | probaron desde la laptop | "funciona" visto desde la VM |
| Registro | queda en `INCIDENTES.md` si hubo fallo | no dejaron rastro |
