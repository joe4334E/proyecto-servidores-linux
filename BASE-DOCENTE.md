# CLASE 1 — BASE COMÚN · Guía del docente

## 8 fases — el proyecto empieza hoy

> Los 5 grupos construyen lo mismo, en su propia VM. Esta es la **única clase con procedimiento entregado**: usted demuestra, el estudiante repite con `BASE-PRACTICA.md`.
>
> **Regla de progresión:** cada fase cierra con un **hito verificable**. Sin hito, no se pasa a la siguiente.
>
> Template de cada fase: **Disparador → Meta → Explicación → Demo → Práctica → Hito → Conexión con el proyecto.**

---

## Marco de apertura

Antes de la Fase 1, deje claro el hilo de la clase:

- La empresa necesita publicar su página, guardar sus datos y centralizar sus documentos. Ese es el punto de partida de **su proyecto**.
- Hoy construyen, todos igual, el **incremento 1**: un servidor que cualquiera puede ver desde otra máquina, con una página que guarda datos reales, protegido por firewall y preparado para sobrevivir un apagón.
- Al final de la clase, esa máquina es la base sobre la que cada grupo levantará los tres módulos del proyecto en la Clase 2.
- Reglas: nadie trabaja como `root` (usa `sudo`); hoy sí hay comandos entregados; siempre se comprueba **desde otra máquina**.

### Tabla de progresión

| Fase | Artefacto que queda | Se usa después en… |
|---|---|---|
| 1 — Llegar al servidor | sesión SSH no-root + credenciales anotadas | acceso de todo el proyecto |
| 2 — El servidor está vivo | lista de servicios activos | gestionar el servicio central de cada proyecto |
| 3 — Cuando algo falla | `INCIDENTES.md` con su 1ª entrada | bitácora obligatoria en Clase 3 e intercambio |
| 4 — Quién está atendiendo | mapa de puertos | tabla de puertos del README |
| 5 — Primer servicio | **PÁGINA PUBLICADA** | módulo web del proyecto; `apt` y servicios |
| 6 — La página decide | página dinámica PHP | la aplicación real del docente |
| 7 — Los datos viven aquí | **PÁGINA CON DATOS REALES** | perfiles de la BD (`app_user` / `consulta`) |
| 8 — Cerrar y sobrevivir | UFW activo + reinicio superado | firewall y `enable` de cada proyecto |

---

## Fase 1 — Llegar al servidor

**Disparador**

> "Su empresa tiene un servidor en un rack, ustedes nunca lo han visto y tienen que empezar a trabajar hoy. ¿Cómo entran?"

**Meta observable:** sesión SSH abierta **desde otra máquina**, con usuario no-root, y los datos de identidad anotados.

**Explicación (breve)**

- Cliente-servidor: ustedes son el cliente; la VM es el servidor.
- SSH: sesión remota cifrada; es la puerta de administración del servidor.
- Por qué no `root`: si te equivocas como root no hay vuelta atrás; además nadie puede decir quién hizo qué. `sudo` ejecuta lo mismo pero **deja registro** y exige intención.

**Demo**

```bash
ssh usuario@IP_DE_LA_VM    # desde otra máquina, no desde la VM
whoami
hostnamectl
who
```

**Práctica:** bloque 1 de `BASE-PRACTICA.md`.

**Hito**

- [ ] Entré por SSH desde una máquina distinta a la VM
- [ ] `whoami` muestra mi usuario, no root
- [ ] IP y hostname anotados

**Conexión con el proyecto:** este es el mismo acceso que usarán las 3 clases; en la Clase 3 entregarán IP + usuario + contraseña al docente exactamente así.

---

## Fase 2 — El servidor está vivo: servicios

**Disparador**

> "Este servidor lleva días encendido. ¿Qué está haciendo justo ahora?"

**Meta observable:** lista de servicios activos anotada + poder explicar `start` vs `enable`.

**Explicación**

- **Daemon:** proceso de segundo plano que espera peticiones (SSH espera conexiones; Apache esperará peticiones HTTP). No tiene pantalla ni usuario interactivo.
- **systemd:** gestor de servicios y arranque; es el que trae todo al aire al encender la máquina.
- **La pregunta del día:**

```
systemctl start   →  lo arranca AHORA
systemctl enable  →  lo trae SOLO cuando se reinicia
```

`start` no sobrevive al reinicio. `enable` sí. La respuesta completa llega en la Fase 8.

**Demo**

```bash
systemctl status ssh
systemctl list-units --type=service --state=running
systemctl is-enabled ssh
systemctl is-active ssh
```

*(No paren ni arranquen SSH: los desconectarían. El start/stop real lo hacen en la Fase 5.)*

**Práctica:** bloque 2 de la hoja.

**Hito**

- [ ] Lista de servicios activos anotada
- [ ] Sé explicar la diferencia entre `start` y `enable`

**Conexión con el proyecto:** el servicio central del proyecto (Apache, MariaDB, Samba) se maneja con estos mismos comandos.

---

## Fase 3 — Cuando algo falla: los logs

**Disparador**

> "El servicio dice 'activo'… pero ¿funciona? El estado dice si corre; el log dice qué hace realmente."

**Meta observable:** `INCIDENTES.md` creado con su primera entrada.

**Explicación**

- Cadana que usarán todo el curso:

```
SERVICIO → ESTADO → LOG → ERROR → CAUSA → SOLUCIÓN
```

- `journalctl` consulta el diario de systemd: cada servicio escribe ahí.
- Niveles: `err` y peor = lo que hay que mirar primero.

**Demo**

```bash
journalctl -u ssh -n 20 --no-pager
journalctl --since today | tail -20
journalctl -p err..alert -n 10 --no-pager
systemctl --failed
```

**Práctica:** bloque 3. Revisan sus logs, buscan unidades fallidas o errores, y crean `INCIDENTES.md` con la entrada 1 (aunque el resultado sea "revisé los logs, sin errores").

**Hito**

- [ ] `INCIDENTES.md` existe con una entrada completa (síntoma → comandos → causa/situación → solución)
- [ ] Pueden leer un log de su servidor sin ayuda

**Conexión con el proyecto:** `INCIDENTES.md` es obligatorio en la Clase 3 y en el intercambio de servidores. Es la hoja de vida del servidor.

---

## Fase 4 — Quién está atendiendo: puertos

**Disparador**

> "Dentro de un rato vamos a publicar una página. Antes: ¿alguien está usando ya el puerto 80?"

**Meta observable:** mapa de puertos en el inventario.

**Explicación**

- Puerto = número de puerta de entrada a un servicio.
- **Escuchar** ≠ funcionar: un servicio atiende en un puerto; que además responda depende de firewall y configuración.
- `ss -tulpn`: qué proceso atiende en cada puerto.

**Demo**

```bash
ss -tulpn
ss -tulpn | grep :22
ss -tulpn | grep :80     # hoy: vacío → el 80 está libre para Apache
```

**Práctica:** bloque 4.

**Hito**

- [ ] Identificaron qué proceso atiende en el 22
- [ ] Confirmaron que el 80 está libre
- [ ] Puertos anotados en el inventario

**Conexión con el proyecto:** la tabla de puertos del README justificará cada puerto abierto y por qué los demás están cerrados.

---

## Fase 5 — Instalar su primer servicio (apt + Apache)

**Disparador**

> "Hoy instalan su primer servicio como administradores. Cuando terminen, cualquiera del salón verá su página desde su propia máquina."

**Meta observable:** **la página de Apache se ve desde otra máquina** + primer diagnóstico real completado.

**Explicación**

- **Paquete** ≠ servicio: el paquete contiene archivos; el servicio es lo que corre después.
- `apt update` solo actualiza la **lista** de paquetes disponibles; `apt install` descarga e instala.
- El instalador casi siempre arranca el servicio — pero eso no significa que esté **habilitado**. Miren siempre `is-enabled` (el error que pagarán en la Fase 8).

**Demo**

```bash
sudo apt update
apt show apache2 | head -20
sudo apt install -y apache2
systemctl status apache2
systemctl is-enabled apache2
ss -tulpn | grep :80
```

Desde otra máquina: `http://IP/` → página de Apache.

**Diagnóstico (segunda vuelta de la espiral)**

```bash
sudo systemctl stop apache2     # la página se cae
journalctl -u apache2 -n 20 --no-pager
sudo systemctl start apache2
# verificar desde otra máquina
```

Registrar en `INCIDENTES.md`.

**Práctica:** bloque 5.

**Hito**

- [ ] Página de Apache visible **desde otra máquina**
- [ ] Pasaron por parar → ver roto → mirar log → arrancar → verificar

**Conexión con el proyecto:** la página de prueba crecerá hasta ser la aplicación real (módulo A); todo servicio del proyecto se instala y arranca igual.

---

## Fase 6 — La página decide (PHP)

**Disparador**

> "La empresa no quiere un cartel colgado; quiere que la página reaccione."

**Meta observable:** página dinámica ejecutándose en el servidor.

**Explicación**

- PHP se ejecuta **en el servidor**; el navegador solo recibe el resultado.
- Va como **módulo dentro de Apache** — por eso hay que reiniciar Apache al instalarlo.

**Demo**

```bash
sudo apt install -y php libapache2-mod-php php-mysql
php -v
sudo systemctl restart apache2
echo '<?php phpinfo(); ?>' | sudo tee /var/www/html/info.php
```

Desde otra máquina: `http://IP/info.php`.

**Práctica:** bloque 6.

**Hito**

- [ ] `info.php` se ve desde otra máquina

**Conexión con el proyecto:** de aquí sale la aplicación real que publicarán en el módulo A.

---

## Fase 7 — Los datos viven aquí (MariaDB)

**Disparador**

> "¿Dónde viven los datos de una empresa? ¿Y quién tiene derecho a tocarlos?"

**Meta observable:** **la página muestra datos reales leídos de MariaDB**, usando un usuario de aplicación — nunca root.

**Explicación**

- El paquete `mariadb-server` instala el daemon, que corre como cualquier otro servicio.
- Hoy MariaDB está **solo para esta máquina** (se usa por conexión local). Que escuche para la red es trabajo de la Clase 2 del proyecto — no lo abran ahora.
- En Debian/Mint, el `root` de MariaDB usa autenticación del sistema: se entra con `sudo mysql`.
- **Hito de hábito:** la página se conecta con el usuario `app`, no con `root`. Empezar hoy con root es una deuda que después cobra (y en la Clase 2 será la base de los perfiles del módulo de datos).

**Demo**

```bash
sudo apt install -y mariadb-server
systemctl status mariadb
systemctl is-enabled mariadb
sudo mysql
```

```sql
CREATE DATABASE baseapp;
CREATE USER 'app'@'localhost' IDENTIFIED BY 'claveapp';
GRANT ALL PRIVILEGES ON baseapp.* TO 'app'@'localhost';
FLUSH PRIVILEGES;
CREATE TABLE baseapp.saludo (id INT AUTO_INCREMENT PRIMARY KEY, mensaje VARCHAR(100));
INSERT INTO baseapp.saludo (mensaje) VALUES ('funciona desde MariaDB');
EXIT;
```

```bash
sudo tee /var/www/html/index.php <<'EOF'
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=baseapp', 'app', 'claveapp');
echo $pdo->query('SELECT mensaje FROM saludo')->fetchColumn();
EOF
```

Desde otra máquina: `http://IP/` → **funciona desde MariaDB**.

**Diagnóstico (tercera vuelta de la espiral)**

```bash
sudo systemctl stop mariadb       # la página se cae
journalctl -u mariadb -n 20 --no-pager
sudo systemctl start mariadb      # verificar
```

Registrar en `INCIDENTES.md`.

**Práctica:** bloque 7.

**Hito**

- [ ] `http://IP/` muestra **funciona desde MariaDB** desde otra máquina
- [ ] La conexión usa el usuario `app`, no root
- [ ] `INCIDENTES.md` tiene el diagnóstico del ejercicio

**Conexión con el proyecto:** de aquí parte el módulo B — la base de datos real tendrá perfiles con privilegios distintos (`app_user` para la aplicación, `consulta` solo lectura) y la aplicación seguirá conectando **nunca con root**. Los mismos criterios de usuario y permisos se aplicarán después en el módulo de archivos.

---

## Fase 8 — Cerrar y sobrevivir (UFW + reinicio)

**Disparador**

> "Ya publican. Ahora: que solo entre quien debe… y que esto sobreviva un apagón."

**Meta observable:** UFW activo con 22 y 80 verificado **desde fuera** + servicios que vuelven solos tras el reinicio.

**Explicación**

- UFW: política por defecto **denegar**; se abre solo lo necesario.
- **Advertencia:** solo `ufw enable` con la regla 22 confirmada; si no, se quedan sin SSH.
- Cierre de la pregunta de la Fase 2: lo que vuelve al reiniciar es lo que estaba **enabled**.

**Demo**

```bash
sudo ufw status
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw enable
sudo ufw status verbose
```

Desde otra máquina:

```bash
curl http://IP/      # debe responder
ss -tulpn            # en la VM: comprobar qué quedó expuesto
```

**Reinicio de verificación** (si no alcanza el tiempo, queda como prueba de la Clase 3):

```bash
sudo reboot
# tras reconectar:
systemctl is-active ssh apache2 mariadb
sudo ufw status
curl http://IP/
```

**Práctica:** bloque 8.

**Hito**

- [ ] UFW activo; desde otra máquina responde el 80 y los demás puertos prueban cerrados
- [ ] Tras el reinicio: SSH, Apache, MariaDB y la página vuelven **solos**

**Conexión con el proyecto:** en la Clase 2 el proyecto abre además sus puertos (139/445 para Samba, 443 solo si se hace el extra; 3306 **no** se abre: la aplicación conecta en local). El `enable` de hoy es lo que garantiza la prueba de reinicio de la Clase 3.

---

## Cierre del incremento

1. **Inventario completo** — tabla de la sección 5.1 del README: sistema, IP, hostname, SSH, servicios instalados y activos, puertos, usuarios, firewall.
2. **Commit:**

```bash
git add README.md INCIDENTES.md
git commit -m "incremento 1 - base comun"
```

3. **Roles** — cada grupo entrega la tabla A/B/C/D con nombres. Se usa desde la Clase 2.
4. **Puente a la Clase 2:**

> "Esto es idéntico en las 5 máquinas. Su proyecto empieza mañana: sobre esta misma base construyen los tres módulos — la web, la base de datos y los archivos — sin entregarnos procedimiento."

| Módulo | De lo de hoy, la Clase 2 parte de… |
|---|---|
| A — Web | Apache + PHP + MariaDB ya corriendo → Virtual Host, aplicación real, logs, permisos |
| B — Datos | usuario de aplicación ya conectando → perfiles con privilegios distintos, respaldo de la BD |
| C — Archivos | reglas de usuario y permisos de hoy → estructura `/empresa`, Samba, política y respaldos |

---

## Errores típicos del salón

| Fase | Lo que pasa | Qué decir (no dar el comando) |
|---|---|---|
| 1 | trabajan dentro de la VM como si fuera el cliente | *"¿desde qué máquina están? ¿La prueba es válida?"* |
| 2 | confunden `start` con `enable` | *"¿y si lo reinicio?"* |
| 3 | dicen "el servicio falló" sin mirar nada | *"¿qué dice exactamente el log?"* |
| 5 | `apt: command not found` | en Mint usan `sudo apt ...` |
| 5 | Apache no arranca: puerto ocupado | *"¿qué está escuchando en el 80?"* |
| 6 | la página PHP se descarga en vez de ejecutarse | *"¿instalaron el módulo de Apache o solo PHP?"* |
| 7 | `mysql: access denied` | *"¿cómo se entra en esta distro?"* (sudo) |
| 8 | se bloquean con UFW sin regla 22 | reconexión por consola; lección: leer antes de `enable` |
| — | se olvidan del `sudo` | que lean el error: `permission denied` ya lo dice |

---

## Reglas de la clase

- **Sí** se entregan comandos: es la clase de enseñanza. La investigación empieza en la Clase 2.
- **No** se resuelve el proyecto en la clase 1: la conexión con la Clase 2 es la tabla de la Fase 7 y el puente del cierre.
- Cada fase cierra con su **hito**: si el salón no lo pasa, no avancen.
- El que termine antes ayuda al de al lado **sin tomarle el teclado**.
- Toda falla vivida hoy se registra en `INCIDENTES.md`: es su primera práctica de diagnóstico.
