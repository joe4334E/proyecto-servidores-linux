# CLASE 1 — BASE COMÚN · Hoja de práctica

## 8 fases — lo que vas a construir hoy

Al final de esta clase tu servidor será algo real: una **página publicada que muestra datos de una base de datos**, protegida con firewall, que vuelve sola después de un reinicio. Cada fase deja un artefacto; sin el artefacto no se pasa a la siguiente.

| Fase | Artefacto que queda |
|---|---|
| 1 — Llegar al servidor | sesión SSH desde otra máquina |
| 2 — El servidor está vivo | lista de servicios anotada |
| 3 — Cuando algo falla | `INCIDENTES.md` con su 1ª entrada |
| 4 — Quién está atendiendo | mapa de puertos |
| 5 — Primer servicio | **PÁGINA PUBLICADA** |
| 6 — La página decide | página dinámica PHP |
| 7 — Los datos viven aquí | **PÁGINA CON DATOS REALES** |
| 8 — Cerrar y sobrevivir | UFW activo + reinicio superado |

**Regla:** nadie trabaja como `root`. Usa tu usuario con `sudo`.
**Regla:** toda comprobación se hace **desde tu laptop, fuera de la VM** (SSH abierto desde la terminal de la laptop, navegador y `curl` desde ahí). La consola gráfica de VirtualBox es solo para emergencias.

La VM corre en VirtualBox/VMware **en tu laptop**; ahí vive todo el **LAMP**:

```
  TU LAPTOP                     TU VM (servidor)
┌──────────────┐  ssh/http:IP  ┌───────────────────────────────┐
│ terminal     │ ────────────► │ SSH:22 · Apache:80 ─► PHP     │
│ navegador    │ ◄─── HTML ──  │        ─► MariaDB (LAMP)      │
└──────────────┘               └───────────────────────────────┘
```

| Dato | Valor |
|---|---|
| IP de mi servidor | |
| Usuario | |
| Máquina desde la que me conecto | mi laptop |

---

## Fase 1 — Llegar al servidor

**Vas a construir:** una sesión de trabajo real desde tu laptop, con tu propio usuario — sin usar la consola de la VM.

Primero: anota la IP de tu VM (dentro de la VM: `ip a` → la dirección de la interfaz). Desde la **terminal de la laptop**:

```bash
ssh usuario@IP
whoami
hostnamectl
```

**Comprueba que:**

- [ ] Entré por SSH desde la laptop, con la consola de la VM cerrada
- [ ] `whoami` muestra mi usuario (no root)
- [ ] Anoté IP y hostname en la tabla de arriba
- [ ] Desde la laptop, `curl http://IP/` responde (o falla por servicio — aún no lo instalamos)

---

## Fase 2 — El servidor está vivo: servicios

**Vas a construir:** el mapa de qué está corriendo en tu servidor ahora mismo.

```bash
systemctl status ssh
systemctl list-units --type=service --state=running
systemctl is-enabled ssh
systemctl is-active ssh
```

**Comprueba que:**

- [ ] Puedo listar los servicios activos de mi servidor
- [ ] Sé decir si SSH vuelve solo tras un reinicio — y **cómo lo sé sin reiniciar**
- [ ] Anoté la diferencia: `start` = ______ , `enable` = ______

*(No pares SSH: te desconectarías.)*

---

## Fase 3 — Cuando algo falla: los logs

**Vas a construir:** tu primera entrada de bitácora. Este archivo te acompaña todo el proyecto.

```bash
journalctl -u ssh -n 10 --no-pager
journalctl --since today | tail -10
journalctl -p err..alert -n 5 --no-pager
systemctl --failed
```

Cadena que usarás siempre:

```
SERVICIO → ESTADO → LOG → ERROR → CAUSA → SOLUCIÓN
```

Crea tu bitácora con la entrada 1:

```
INCIDENTES.md

| # | Fecha | Síntoma | Comandos usados | Causa | Solución |
|---|-------|---------|-----------------|-------|----------|
| 1 |       |         |                 |       |          |
```

**Comprueba que:**

- [ ] Revisé los logs de mi propio servidor
- [ ] Busqué unidades fallidas (`systemctl --failed`) y errores
- [ ] `INCIDENTES.md` existe con su primera entrada

---

## Fase 4 — Quién está atendiendo: puertos

**Vas a construir:** el mapa de puertos de tu inventario.

```bash
ss -tulpn
ss -tulpn | grep :22
ss -tulpn | grep :80
```

**Comprueba que:**

- [ ] Identifiqué qué proceso atiende en el puerto 22
- [ ] Confirmé que el puerto 80 está libre (esperado: sin resultado)
- [ ] Anoté los puertos en el inventario

---

## Fase 5 — Instalar tu primer servicio (apt + Apache)

**Vas a construir:** una página visible para todo el salón + tu primer diagnóstico real.

```bash
sudo apt update
apt show apache2 | head -20
sudo apt install -y apache2
systemctl status apache2
systemctl is-enabled apache2
ss -tulpn | grep :80
```

**Comprueba que:**

- [ ] Apache activo (`status`) y escuchando en el 80
- [ ] Desde otra máquina: `http://IP/` muestra la página de Apache

### Tu primer diagnóstico

```bash
sudo systemctl stop apache2
journalctl -u apache2 -n 20 --no-pager
sudo systemctl start apache2
```

Vuelve a abrir la página desde otra máquina. Registra lo que pasó en `INCIDENTES.md` (entrada 2).

- [ ] Paré el servicio, vi el roto, miré el log, lo levanté, verifiqué

---

## Fase 6 — La página decide (PHP)

**Vas a construir:** una página que se ejecuta en el servidor.

```bash
sudo apt install -y php libapache2-mod-php php-mysql
php -v
sudo systemctl restart apache2
echo '<?php phpinfo(); ?>' | sudo tee /var/www/html/info.php
```

**Comprueba que:**

- [ ] Desde otra máquina, `http://IP/info.php` muestra la información de PHP

---

## Fase 7 — Los datos viven aquí (MariaDB)

**Vas a construir:** una página que muestra **datos reales** de una base de datos, con un usuario de aplicación — nunca root.

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

**Comprueba que:**

- [ ] Desde otra máquina, `http://IP/` dice **funciona desde MariaDB**
- [ ] La página usa el usuario `app`, no `root`

### Diagnóstico

```bash
sudo systemctl stop mariadb
journalctl -u mariadb -n 20 --no-pager
sudo systemctl start mariadb
```

- [ ] Detecté el problema, vi el log, lo reparé y la página volvió
- [ ] Registrado en `INCIDENTES.md`

---

## Fase 8 — Cerrar y sobrevivir (UFW + reinicio)

**Vas a construir:** un servidor cerrado al mundo excepto lo necesario, que sobrevive al apagón.

```bash
sudo ufw status
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw enable
sudo ufw status verbose
```

**Solo actívalo con la regla 22 confirmada o te quedas sin SSH.**

Desde otra máquina:

```bash
curl http://IP/     # debe responder
```

**Comprueba que:**

- [ ] UFW activo, 22 y 80 abiertos, verificado **desde fuera**
- [ ] `curl http://IP/` responde

**Reinicio de verificación:**

```bash
sudo reboot
# tras reconectar:
systemctl is-active ssh apache2 mariadb
sudo ufw status
curl http://IP/
```

- [ ] Tras el reinicio: SSH, Apache, MariaDB y la página vuelven **solos**

---

## Cierre — tu incremento está completo

1. **Inventario en el README** — completa la tabla: sistema, IP, hostname, SSH, servicios instalados/activos, puertos, usuarios, firewall.

2. **Commit:**

```bash
git add README.md INCIDENTES.md
git commit -m "incremento 1 - base comun"
```

3. **Roles del grupo** — entrega la tabla A/B/C/D con nombres.

### Comprobación final (te la puede hacer el docente)

1. ¿Qué diferencia hay entre `systemctl start` y `systemctl enable`?
2. ¿Dónde miras cuando un servicio no funciona?
3. Si mañana reinicias el servidor, ¿qué vuelve solo? ¿Cómo lo compruebas sin reiniciar?
4. ¿Por qué la página usa el usuario `app` y no `root`?
5. ¿Qué tendría que pasar para que tu página dejara de verse desde otra máquina y por dónde empezarías a mirar?

### Checklist de cierre de la clase

- [ ] SSH desde otra máquina con usuario no-root
- [ ] Servicios listados con `systemctl`
- [ ] Logs leídos con `journalctl`
- [ ] Puertos mapeados con `ss -tulpn`
- [ ] `apt` usado para instalar
- [ ] Apache + PHP publicando
- [ ] MariaDB con base y usuario `app`
- [ ] Página con datos reales visible desde fuera
- [ ] UFW activo y verificado fuera
- [ ] Reinicio superado
- [ ] `INCIDENTES.md` con al menos 2 entradas
- [ ] Inventario completo + commit
- [ ] Roles asignados
