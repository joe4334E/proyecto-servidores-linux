# PROGRAMA — De los permisos de usuario al LAMP

> Programa de estudio de la unidad. Orden sugerido de aprendizaje: cada tema usa el anterior.
> Al final de este programa puedes construir y administrar un servidor **LAMP** completo — que es exactamente lo que hace tu proyecto.

---

## 0. El destino: ¿qué es LAMP?

**LAMP** es el acrónimo del stack que vas a construir:

| Letra | Tecnología | Qué aporta |
|---|---|---|
| **L** | **L**inux | El sistema operativo: usuarios, permisos, servicios, red |
| **A** | **A**pache | El servidor web: recibe peticiones HTTP y sirve archivos/páginas |
| **M** | **M**ariaDB (MySQL) | El motor de base de datos: guarda los datos de la empresa |
| **P** | **P**HP | El lenguaje de la página: se ejecuta en el servidor y habla con la BD |

Así se ve una petición recorriendo el stack desde tu laptop:

```
  TU LAPTOP                              LA VM (servidor)
┌────────────┐   http://IP/    ┌──────────────────────────────────────────┐
│  navegador │ ──────────────► │ :80  Apache ──► lee index.php            │
│  (cliente) │                │                    │                     │
└────────────┘                │              PHP ejecuta el código       │
                              │                    │ PDO (usuario app)   │
                              │                    ▼                     │
                              │ :3306  MariaDB ──► tabla clientes        │
                              │          (SOLO en 127.0.0.1, nunca       │
                              │           expuesto al exterior)          │
                              └──────────────────────────────────────────┘
                                        ◄── vuelve HTML ──
```

**Lo que necesitas dominar antes de poder construir eso** es, en orden:

```
USUARIOS → PERMISOS → SERVICIOS → LOGS → PUERTOS/FIREWALL → APACHE → PHP → MARIADB → LAMP
 (quién)   (qué puede)  (qué corre) (qué pasó)  (quién entra)   (A)      (P)    (M)
```

Las siguientes secciones cubren cada eslabón.

---

## 1. Usuarios y grupos: quién es quién

### 1.1 El sistema identifica a todos por UID/GID

| Archivo | Qué guarda | Ejemplo de línea |
|---|---|---|
| `/etc/passwd` | usuario, UID, shell, home | `ana:x:1001:1001::/home/ana:/bin/bash` |
| `/etc/shadow` | contraseñas cifradas (solo root lee) | — |
| `/etc/group` | grupos y quiénes pertenecen | `ventas:x:1005:luis,carlos` |

```bash
whoami        # quién soy
id            # UID, GID y todos mis grupos
cat /etc/passwd | tail -5
```

### 1.2 Administrar usuarios

```bash
sudo adduser carlos          # crear usuario (pide datos y contraseña)
sudo usermod -aG ventas carlos   # AGREGAR a un grupo (-a = append, sin él lo reemplazas)
sudo passwd carlos           # cambiar contraseña
sudo deluser carlos          # eliminar
```

**Usuario `root` vs `sudo`:** `root` puede todo y no deja registro de quién fue. `sudo` ejecuta lo mismo **con tu identidad**, exige contraseña y queda en el log. Regla del curso: **nunca trabajes como root, siempre `sudo`**.

### 1.3 Para qué sirve en el proyecto

- Cada departamento de `/empresa` tendrá su **grupo** (`ventas`, `direccion`…) y sus usuarios;
- los servicios corren con su propio usuario (`www-data` para Apache);
- los roles A/B/C/D del equipo son la versión "humana" de lo mismo: cada quien responde de algo.

---

## 2. Permisos de archivos: qué puede hacer cada quien

### 2.1 Los tres actores

Todo archivo tiene **dueño**, **grupo** y **otros**:

```bash
ls -l /etc/passwd
# -rw-r--r-- 1 root root 2841 ... /etc/passwd
#  ^^^ ^^^ ^^^   ^^^^  ^^^^
#  u   g   o   dueño  grupo
```

### 2.2 Lectura, escritura, ejecución

| Letra | En archivo | En directorio |
|---|---|---|
| **r** (4) | ver el contenido | listar los nombres (`ls`) |
| **w** (2) | modificar | crear/borrar archivos dentro |
| **x** (1) | ejecutar (programa, script) | **entrar al directorio** (`cd`) |

Un directorio **sin `x`** no se puede pisar, aunque tenga `r`.

### 2.3 Cambiar permisos y dueños

```bash
chmod 640 archivo        # numérico: u=rw, g=r, o=
chmod u+x,go-rwx archivo # simbólico
chown carlos:ventas acta.docx   # cambiar dueño y grupo
chgrp ventas acta.docx          # solo grupo
umask                        # permisos por defecto de mis archivos nuevos
```

### 2.4 La trampa: `chmod 777`

`chmod -R 777 /empresa` = **todos pueden todo**: leer, modificar y borrar los documentos de cualquier departamento. En este curso se considera **incorrecto como solución**. Si lo usas como respuesta a un problema de acceso, el ítem correspondiente vale **0**.

Diagnóstico correcto de permisos:

```bash
namei -l /empresa/ventas/contrato.docx   # muestra cada nivel de la ruta
getfacl /empresa/ventas                  # si hay ACLs (permisos extra por usuario)
ls -laR /empresa | less
```

**Verifica siempre desde otra máquina o con la cuenta real**: `ls -l` dice lo que *debería* pasar; un login con la cuenta dice lo que **pasa**.

### 2.5 Para qué sirve en el proyecto

- La política de acceso de `/empresa` (módulo C) **es** esta sección aplicada;
- `777`, permisos mal heredados en directorios (`umask`) y dueños incorrectos son fallos típicos del intercambio.

---

## 3. Servicios: qué está corriendo

Un **daemon** (servicio) es un proceso de segundo plano que espera peticiones: SSH espera conexiones, Apache espera HTTP, MariaDB espera consultas SQL.

```bash
systemctl status apache2        # estado ahora
systemctl start apache2         # arrancar AHORA
systemctl stop apache2          # parar AHORA
systemctl enable apache2        # que arranque SOLO al prender la máquina
systemctl is-enabled apache2    # ¿está habilitado?
systemctl list-units --type=service --state=running
```

**La diferencia clave:** `start` no sobrevive a un reinicio; `enable` sí. Un servicio que funciona hoy pero no vuelve al reiniciar = **no estaba habilitado**.

### Logs: qué pasó realmente

```bash
journalctl -u apache2 -n 50 --no-pager    # diario de un servicio
journalctl -p err..alert -n 10 --no-pager # solo errores
systemctl --failed                        # servicios caídos
```

Cadena de diagnóstico que se usa en todo el curso:

```
SERVICIO → ESTADO → LOG → ERROR → CAUSA → SOLUCIÓN → VERIFICACIÓN
```

---

## 4. Red: puertos, firewall y SSH

### 4.1 Puerto: la puerta de cada servicio

| Puerto | Servicio |
|---|---|
| 22 | SSH (administración remota) |
| 80 | HTTP (Apache) |
| 443 | HTTPS |
| 3306 | MariaDB (**no se abre al exterior**: la app habla en local) |

```bash
ss -tulpn              # qué proceso escucha en qué puerto
ss -tulpn | grep :80
```

**Escuchar ≠ accesible:** un servicio puede atender en su puerto y aun así el firewall bloquearlo. Hace falta que las **dos** cosas permitan el tráfico.

### 4.2 Firewall (UFW)

```bash
sudo ufw status verbose
sudo ufw allow 22/tcp    # primero SIEMPRE el 22 (o te quedas sin SSH)
sudo ufw allow 80/tcp
sudo ufw enable
```

Política por defecto: **denegar todo** y abrir solo lo justificado. Toda regla se verifica **desde otra máquina** (`curl`, navegador, `ssh`).

### 4.3 SSH: la puerta de administración

```bash
ssh ana@192.168.1.50     # desde TU LAPTOP hacia la VM (no desde dentro de la VM)
```

SSH es una sesión remota cifrada. En el proyecto: red de la laboratorio con IP fija; en el mundo real: igual, más llaves y menos contraseñas.

---

## 5. A — Apache: el servidor web

```bash
sudo apt update && sudo apt install -y apache2
systemctl status apache2 && systemctl is-enabled apache2
```

| Concepto | Qué es |
|---|---|
| `DocumentRoot` | carpeta que se sirve: `/var/www/html` por defecto |
| **Virtual Host** | sitio con su propio nombre (`ServerName`) y su propia carpeta |
| Módulos | capacidades que se activan/desactivan (`a2enmod`, `a2ensite`) |
| Logs | `/var/log/apache2/access.log` y `error.log` |

```bash
# truco clásico: ver un error real
sudo tail -f /var/log/apache2/error.log
```

---

## 6. P — PHP: la página que piensa

PHP se ejecuta **en el servidor**; el navegador solo recibe el HTML resultante.

```bash
sudo apt install -y php libapache2-mod-php php-mysql
sudo systemctl restart apache2
php -v
```

- `libapache2-mod-php` = el módulo que le enseña a Apache a ejecutar `.php` (sin él, Apache **descarga** el archivo en vez de mostrarlo);
- `php-mysql` = el driver para hablar con MariaDB;
- conexión a la BD con **PDO**, usando el usuario de aplicación — jamás root:

```php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=empresa', 'app_user', '***');
```

---

## 7. M — MariaDB: los datos

```bash
sudo apt install -y mariadb-server
sudo mysql            # en Debian/Mint el root usa autenticación del sistema
```

### 7.1 Dos niveles de usuarios (no confundirlos)

| Nivel | Quién | Ejemplo |
|---|---|---|
| **Del sistema (Linux)** | quién entra al servidor y qué archivos toca | `carlos`, `www-data` |
| **De la base de datos** | quién consulta/modifica tablas | `app_user`, `consulta` |

Son independientes: tener usuario de Linux **no** da acceso a la BD, y viceversa.

### 7.2 Privilegios: el corazón de la seguridad de datos

```sql
CREATE DATABASE empresa;
CREATE USER 'app_user'@'localhost' IDENTIFIED BY '***';
CREATE USER 'consulta'@'localhost' IDENTIFIED BY '***';
GRANT SELECT, INSERT, UPDATE ON empresa.* TO 'app_user'@'localhost';
GRANT SELECT ON empresa.*        TO 'consulta'@'localhost';
FLUSH PRIVILEGES;
SHOW GRANTS FOR 'app_user'@'localhost';
```

- `app_user` = lo que usa la aplicación (nunca root);
- `consulta` = solo lectura → probar que `UPDATE` le sale **denegado**;
- `'localhost'` = la BD no necesita puerto abierto al exterior: la app corre en la misma máquina.

### 7.3 Respaldo y restauración

```bash
mysqldump -u app_user -p empresa > respaldo_$(date +%F).sql
mysql -u app_user -p empresa < respaldo_2026-10-08.sql   # restaurar
```

**Un respaldo que no se restauró y verificó no es un respaldo.**

---

## 8. El LAMP completo: cómo se ensambla

Instalación mínima del stack entero:

```bash
sudo apt update
sudo apt install -y apache2 php libapache2-mod-php php-mysql mariadb-server
sudo ufw allow 22/tcp && sudo ufw allow 80/tcp && sudo ufw enable
```

Recorrido de la petición, con los comandos para mirar cada eslabón:

```
navegador ──HTTP:80──► Apache ──► PHP ──PDO:3306──► MariaDB
     │                   │          │                 │
  probar:            probar:     probar:           probar:
  curl http://IP/     status      info.php          SHOW GRANTS;
                     tail error   php -v            mysqldump
                     log
```

Si algo falla, recorre el stack **en orden** hasta encontrar el eslabón roto:

1. ¿Llega el paquete? (`curl`, firewall con `ufw status`)
2. ¿Apache corre y escucha? (`systemctl status`, `ss -tulpn`)
3. ¿Ejecuta PHP? (`info.php`, `error.log`)
4. ¿PHP conecta a la BD? (credenciales de `app_user`, `status` de MariaDB)
5. ¿Los privilegios alcanzan? (`SHOW GRANTS`)

---

## 9. Buenas prácticas del administrador (resumen)

1. `sudo`, nunca sesión como root.
2. Nunca `chmod 777`; permisos mínimos necesarios.
3. La aplicación conecta a la BD con **su usuario**, no root.
4. Abre solo los puertos justificados y **verifica desde fuera**.
5. Todo servicio con `enable` — si no vuelve tras el reinicio, no estaba.
6. Lee los logs antes de opinar; registra todo incidente en `INCIDENTES.md`.
7. Respaldos con fecha **y restauración demostrada**.
8. Documenta: si otro no puede operarlo con tu README, no está terminado.

---

## 10. Scripts bash: tus primeras herramientas

Un script es un archivo de texto con comandos que se ejecutan en orden: lo que harías escribiendo en la terminal, guardado y repetible.

```bash
#!/bin/bash                  # quién lo interpreta (siempre la primera línea)
set -e                        # detenerse ante el primer error (buena práctica)

echo "Hola, $USER"            # salida en pantalla
FECHA=$(date +%F)             # guardamos un valor en una variable
cp archivo.sql "archivo-$FECHA.sql"

if [ -f /etc/passwd ]; then   # condición: ¿existe el archivo?
    echo "existe"
fi

for f in *.txt; do            # recorre cada archivo .txt
    echo "procesando $f"
done
```

### Crear y ejecutar

```bash
nano organizar.sh             # o el editor que prefieras
chmod +x organizar.sh         # permiso de ejecución (si falta: "Permission denied")
./organizar.sh                # ejecutar desde la carpeta
bash organizar.sh             # alternativa sin chmod +x
```

### Ejercicio del proyecto: `organizar.sh`

Clasifica los documentos desordenados de la empresa por extensión (cada estudiante el suyo):

```bash
#!/bin/bash
# TODO: tuyo — clasifica los documentos en carpetas por extensión
for ext in docx xlsx pdf txt; do
    mkdir -p "$DESTINO/$ext"
    # ¿cómo encuentras todos los archivos .$ext y los copias? investiga `find` y `cp`
done
```

| Concepto | Para qué |
|---|---|
| `$1`, `$2` … | argumentos: `./organizar.sh entrada salida` |
| `>` y `>>` | redirigir salida a archivo (`>` sobrescribe, `>>` agrega) |
| `read VAR` | pedir datos al usuario |
| `find`, `grep`, `wc` | las herramientas que todo script combina |

**Regla:** un script que no puedes explicar **línea por línea** no es tuyo — en la defensa se pregunta.

---

## 11. Mapa al proyecto

| Sección del programa | Dónde se vive en el proyecto |
|---|---|
| 1 — Usuarios y grupos | Clase 1 (roles, usuarios); módulo C (grupos por departamento) |
| 2 — Permisos | módulo C: política de `/empresa`; criterio `777` = 0 |
| 3 — Servicios y logs | Clase 1 fases 2/3; diagnóstico en todo el proyecto |
| 4 — Red y firewall | Clase 1 fase 8; puertos 22 y 80 del README |
| 5 — Apache | módulo A (Virtual Host, logs) |
| 6 — PHP | módulo A (la aplicación real del docente) |
| 7 — MariaDB | módulo B (`app_user` / `consulta`, respaldos) |
| 8 — LAMP completo | la clase 1 **es** este montaje; los módulos lo endurecen |
| 9 — Buenas prácticas | la entrega: seguridad, respaldos, documentación |
| 10 — Scripts bash | entrega individual: `organizar.sh` + 1 script a elección; `backup.sh`/`restore.sh` del grupo |

---

## Autoevaluación

Para practicar **haciendo** (evidencia de tu propio servidor): ver `PREGUNTAS.md` — 12 desafíos básicos, desde `/etc` hasta tu primer servicio web.

Antes de arrancar el proyecto, puedes responder:

1. ¿Qué diferencia hay entre `chmod` y `chown`? ¿Y entre `r` en un archivo y `r` en un directorio?
2. ¿Por qué `start` de un servicio no sobrevive al reinicio y `enable` sí?
3. Si la página no carga, ¿cuál es tu orden de diagnóstico? (5 pasos del §8)
4. ¿Por qué la app no debe conectarse con root a la BD, aunque "es más fácil"?
5. ¿Qué dos condiciones deben cumplirse a la vez para que un puerto sea accesible desde fuera?
6. ¿Qué significa que `consulta` tenga solo `SELECT` y cómo demuestras que `UPDATE` le falla?
7. Menciona 3 diferencias entre respaldar y restaurar, y por qué solo la segunda prueba que sirve el respaldo.
8. ¿Qué hace `chmod +x script.sh` y qué mensaje ves si lo ejecutas sin ese permiso?
