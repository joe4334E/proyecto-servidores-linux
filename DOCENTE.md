# DOCENTE — Manual del docente

> Documento del docente. No se entrega a los estudiantes.

**Partes:** I Operación · II Catálogo de fallos · III Intercambio · IV Defensa.

---

# Parte I — Operación


---

## 1. Preparación previa (antes de la clase 1)

Cada grupo recibe **una VM Linux limpia**. Limpia significa:

- sistema actualizado;
- **SSH del servidor instalado y funcionando** (es acceso al laboratorio, no es el contenido del proyecto);
- IP fija en la red del aula y hostname propio (ej. `srv-g1` … `srv-g5`);
- un usuario administrador creado por usted (credenciales que entregarán al grupo);
- **ningún servicio preinstalado** (Apache y MariaDB se instalan en la clase 1; Samba y el resto, en clase 2: todo eso lo hacen ellos);
- snapshot de imagen base con la VM en ese estado.

### Comandos mínimos de preparación (por VM)

```bash
# identidad
sudo hostnamectl set-hostname srv-gN
ip a                          # anotar la IP

# acceso
sudo apt update && sudo apt install -y openssh-server
sudo systemctl enable --now ssh

# usuario administrador del grupo (ellos harán los suyos)
sudo adduser adminN
sudo usermod -aG sudo adminN

# snapshot limpio
```

> **Regla:** usted prepara **acceso y red**. Todo lo demás es trabajo del grupo.

---

## 2. Acceso de los estudiantes

### Opción principal — red del aula

Las VMs con IP fija; los estudiantes entran desde las máquinas del salón:

```bash
ssh adminN@192.168.X.Y10
```

Entregales: IP, usuario, contraseña. Nada más.

### Respaldo opcional — Tailscale

Si la red del aula falla o un grupo necesita trabajar fuera de clase: instalar Tailscale en las VMs y en las máquinas de los estudiantes, y usar la IP de Tailnet en lugar de la de aula. Es solo un seguro; el proyecto se evalúa igual. Si no quieres configurarlo, con la intranet basta.

**Importante:** no bloqueen el puerto 22 sin probarlo desde otra máquina. Si un grupo se queda sin SSH en la clase 1, es una buena primera lección — pero que la resuelvan ellos.

---

## 3. Rol del docente durante las 3 clases

**Clase 1 (base común):** usted enseña y demuestra todo (`BASE-DOCENTE.md`); los estudiantes repiten en su VM con `BASE-PRACTICA.md`.

**Clases 2 y 3 (proyecto):** usted **no configura**. Usted:

- verifica hitos (checklist de la sección 9 del `README.md`);
- pregunta, no resuelve: *"¿qué dice el log?"*, *"¿desde dónde estás probando?"*, *"¿por qué ese puerto?"*;
- rechaza soluciones de bricolaje: `chmod 777`, `ufw allow all`, copiar la config de internet sin entenderla;
- anota quién hizo qué (lo necesitarás para la defensa).

### Preguntas estándar ante un grupo atascado

1. ¿Qué cambió respecto a antes?
2. ¿Qué dice exactamente el mensaje de error?
3. ¿En qué log buscaste?
4. ¿Funciona desde el propio servidor? ¿Y desde otra máquina?
5. ¿Qué dice la documentación oficial de ese servicio?

Si tras esas 5 siguen bloqueados, señala el **área** (red, servicio, permiso, base de datos) — nunca el comando.

---

## 4. Antes de cada clase

| Clase | Preparación suya |
|---|---|
| 1 (base común) | VMs encendidas, IP/credenciales listas para entregar, snapshot base verificado, `BASE-DOCENTE.md` a la mano; probar antes en una VM que `sudo apt update` llega a los repositorios |
| 2 | VMs encendidas; revisar en 2 minutos que la clase 1 quedó cerrada (los hitos atrasados se arreglan en clase 2, no en casa sin control) |
| 3 | VMs encendidas; preparar tu plan de pruebas para los 5 servidores (ver sección 5); lugar para anotar resultados |
| Sesión 4 | snapshot de los 5 servidores + fallos inyectados del catálogo + sobres con credenciales |

---

## 5. Clase 3 — pruebas del docente

Recorre los 5 servidores con esta hoja:

| Prueba | Qué haces | Aprueba si… |
|---|---|---|
| Funcional | usas el servicio como usuario real desde otra máquina | funciona sin tocar nada en el servidor |
| Permisos | intentas acceder a algo que no te corresponde | te deniega y el grupo explica por qué |
| Firewall | verificas `ufw status` y pruebas puertos desde fuera | solo los justificados están abiertos |
| Restauración | alteras o borras datos de la BD o de `/empresa` | detectan, restauran y verifican |
| Reinicio | `sudo reboot` y esperas | SSH, firewall, servicio y datos vuelven solos |
| Documentación | comparas README con el servidor real | coinciden; el README sirve para operar |

Anota el resultado por grupo. Lo usarás en la rúbrica.

---

## 6. Sesión 4 — inyección y verificación

1. **Snapshot** de los 5 servidores.
2. **Inyecta** un fallo por servidor con los comandos de la **Parte II** de este documento (uno a la vez; puedes dar el **mismo** fallo a los 5 para comparar diagnósticos, o rotar del catálogo).
3. **Entrega** credenciales + README del dueño. Ni pistas ni catálogo.
4. Observa con la secuencia: ¿estado → log → causa, o puro ensayo y error?
5. **Verifica** contra el catálogo: encontraron el fallo correcto, repararon y comprobaron desde otra máquina.
6. **Restaura** los servidores (snapshot o corrección) antes de irte.
7. Recoge los `INCIDENTES.md` de los grupos receptores.

---

## 7. Errores comunes y hasta dónde intervenir

| Error común | Hasta dónde llegas |
|---|---|
| Se bloquean a sí mismos con UFW | pregunta: *"¿desde dónde estás probando?"* — que lo resuelvan |
| `chmod -R 777` | lo rechazas: *"¿quién queda sin restricción con eso?"* |
| Instalan pero no configuran | *"el servicio está instalado y corriendo, ¿por qué no responde?"* |
| No leen logs | *"muéstrame el log donde aparece ese error"* |
| Respaldo manual pero no restaurado | no apruebas: deben restaurar |
| Servicio sin `enable` | lo descubren en el reinicio de la clase 3 — no los avises antes |
| README copiado o genérico | devuelves la entrega; la comparas contra el servidor real |
| Uno solo hace todo | le preguntas a los otros: *"¿y tú qué configuraste?"* — desde la clase 1 |

---

## 8. Control rápido del estado del servidor (para usted)

```bash
# servicios activos
systemctl list-units --type=service --state=running

# puertos escuchando
ss -tulpn

# firewall
sudo ufw status verbose

# últimas líneas de un log
journalctl -u <servicio> -n 50 --no-pager

# quién está conectado
who
```

Para comprobar el intercambio (recursos compartidos de `/empresa`):

```bash
smbclient -L localhost -N
```

---

## 9. Entregables que debes recibir

Al cierre de la clase 3, por grupo:

- [ ] repositorio Git con `README.md` (14 secciones), `INCIDENTES.md` y `scripts/`;
- [ ] IP + usuario + contraseña del servidor;
- [ ] cuenta de prueba si el proyecto la requiere;
- [ ] resultados de tus pruebas anotados (hoja de la sección 5).

Al cierre de la sesión 4:

- [ ] `INCIDENTES.md` con la entrada del fallo ajeno;
- [ ] defensa individual realizada;
- [ ] servidores restaurados (snapshot).

---

# Parte II — Catálogo de fallos controlados

Catálogo único: el proyecto es idéntico para los 5 grupos, así que **puedes dar el mismo fallo a todos** (diagnóstico directamente comparable) o variar por servidor rotando de esta lista.

## Reglas de inyección

1. **Snapshot antes** de cada servidor.
2. **Una sola cosa rota a la vez.**
3. **No dar pistas:** el mensaje es siempre *"el servidor X tiene un problema. Investíguenlo."*
4. **Anotar** servidor → fallo → solución esperada.
5. **Restaurar** al terminar (snapshot o corrección manual).

## Tipos de fallo

| Tipo | Qué pone a prueba |
|---|---|
| 1 — Servicio detenido/desactivado | `systemctl status`, `journalctl`, `enable` |
| 2 — Configuración alterada | leer configs, logs, comparar con el README |
| 3 — Permiso o firewall cambiado | `namei -l`, `ufw status`, pruebas desde otra máquina |
| 4 — Datos degradados | detectar → respaldo → restaurar → verificar |

---

## Catálogo

| # | Tipo | Inyección | Síntoma | Diagnóstico esperado |
|---|---|---|---|---|
| 1 | Servicio | `systemctl stop apache2 && systemctl disable apache2` | la aplicación web no responde desde otra máquina; no vuelve tras reinicio | `systemctl status` → inactive; `journalctl -u apache2`; `is-enabled` → disabled |
| 2 | Servicio | `systemctl stop smbd && systemctl disable smbd` | desaparecen los recursos compartidos de `/empresa` | `systemctl status smbd`; journal; `is-enabled` |
| 3 | Servicio | `systemctl stop mariadb && systemctl disable mariadb` | la aplicación web muestra error de conexión a BD | `systemctl status mariadb`; journal; mensaje de la app |
| 4 | Firewall | `ufw deny 80/tcp` | falla desde fuera, **funciona desde el propio servidor** | `ufw status verbose`; probar desde otra máquina (el error "solo desde fuera" es la pista) |
| 5 | Firewall | `ufw deny 445/tcp` | desde otra máquina no conectan a Samba; en el servidor sí | `ufw status verbose`; prueba cruzada entre máquinas |
| 6 | Permisos | `chmod 700 /empresa/ventas` (ajustar a la ruta real) | el departamento de ventas perdió el acceso a su propio directorio | `namei -l /empresa/ventas`, `getfacl` → comparar con la política del README |
| 7 | Permisos | `chown root:root /empresa/direccion` (si la política dependía de grupo) | usuarios de dirección sin acceso | revisar propietario/grupo vs tabla de política del README |
| 8 | Config | `sed -i 's/^\[ventas\]/\[ventas_tmp\]/' /etc/samba/smb.conf && systemctl reload smbd` | el share de ventas ya no aparece | `smbclient -L localhost` / revisar `smb.conf` vs documentación |
| 9 | Config | `sed -i 's/^ServerName.*/ServerName equivocado.local/' /etc/apache2/sites-available/empresa.conf && apachectl graceful` | llega la página por defecto de Apache o error de vhost | `apachectl -S`; comparar con el `ServerName` documentado |
| 10 | Privilegios | `mysql -uroot -p -e "REVOKE UPDATE ON <bd>.* FROM 'app_user'@'localhost'; FLUSH PRIVILEGES;"` | la aplicación falla al editar | mensaje de MariaDB (denied) → `SHOW GRANTS FOR 'app_user'@'localhost'` |
| 11 | Datos (BD) | `mysql -uroot -p -e "DELETE FROM <tabla_principal> WHERE 1=1;"` o `DROP TABLE` | aplicación con datos vacíos o error de tabla inexistente | detectar → localizar dump con fecha → restaurar → verificar conteos |
| 12 | Datos (archivos) | mover el contenido de `/empresa/publico` a `/tmp/copia-seguridad` | documentos "desaparecidos" | buscar el respaldo de archivos → restaurar → verificar |

---

## Verificación posterior

Tras cada fallo, el grupo debe entregar en `INCIDENTES.md`:

| Campo | Qué se busca |
|---|---|
| Síntoma | lo que observaron, no la causa |
| Comandos usados | evidencian el método de diagnóstico |
| Causa | correcta y específica |
| Solución | aplicada y verificada |
| Prevención | cómo evitan que vuelva a pasar |

**Diagnóstico correcto con reparación a medias = aprobado parcial.** La reparación debe quedar verificada con una prueba desde otra máquina.

---

# Parte III — Protocolo de intercambio

## Rotación

| El grupo… | administra el servidor… | Recibe |
|---|---|---|
| 1 | del grupo 3 | servidor del grupo 3 + su README + credenciales |
| 2 | del grupo 5 | servidor del grupo 5 + su README + credenciales |
| 3 | del grupo 4 | servidor del grupo 4 + su README + credenciales |
| 4 | del grupo 1 | servidor del grupo 1 + su README + credenciales |
| 5 | del grupo 2 | servidor del grupo 2 + su README + credenciales |

---

## Secuencia

### Antes de la sesión

1. Snapshot de los 5 servidores.
2. Inyectar **un fallo por servidor** (catálogo en la Parte II de este documento).
3. Anotar en tu control: servidor → fallo inyectado → solución esperada.
4. Preparar el sobre con credenciales de cada servidor (IP, usuario, contraseña).

### Durante la sesión

**Paso 1 — Entrega (5 min)**

Cada grupo recibe:

- IP;
- usuario y contraseña;
- el `README.md` del grupo dueño.

Nada más. Ni el catálogo de fallos, ni el historial de cambios, ni al dueño.

**Paso 2 — Diagnóstico (tiempo asignado por grupo)**

El grupo trabaja con esta secuencia obligatoria:

```
SÍNTOMA
   ↓
¿QUÉ SERVICIO ESTÁ AFECTADO?  (systemctl / ss -tulpn)
   ↓
¿ESTÁ CORRIENDO?  (estado)
   ↓
¿QUÉ DICE EL LOG?  (journalctl -u <servicio>)
   ↓
¿DÓNDE ESTÁ LA CAUSA?  (config / permiso / firewall / datos)
   ↓
CORREGIR
   ↓
VERIFICAR DESDE OTRA MÁQUINA
   ↓
REGISTRAR EN INCIDENTES.MD
```

**Paso 3 — Reporte (2 min por grupo)**

El grupo entrega verbalmente, con evidencia en pantalla:

1. qué estaba roto;
2. cómo lo descubrieron (comandos);
3. cuál era la causa;
4. qué cambiaron;
5. cómo lo verificaron;
6. qué harían para prevenirlo.

**Paso 4 — Verificación del docente**

El docente comprueba, con su control, que:

- el fallo inyectado fue el que encontraron;
- la reparación quedó aplicada (no solo explicada);
- el servicio funciona desde otra máquina;
- el `INCIDENTES.md` del grupo receptor tiene la entrada completa.

---

## Reglas

1. **No comunicarse con el grupo dueño.** Es la prueba: la documentación debe bastar.
2. **Un solo fallo a la vez.** Si encuentran algo que "también está raro", lo anotan y siguen con el principal.
3. **No romper nada más.** Si no están seguros de un cambio, lo prueban y lo revierten.
4. **Todo queda registrado** en `INCIDENTES.md`.
5. Si la reparación requiere datos del dueño (claves de BD, rutas), está en el README. Si no está, eso es una observación a la documentación del grupo dueño — anótelo, también evalúa.

---

## Cierre de la sesión

1. Snapshot o restauración de los 5 servidores al estado original.
2. Recoger `INCIDENTES.md` actualizado de cada grupo receptor.
3. Anotar las observaciones de documentación que aparecieron (faltó info, info mala).

---

# Parte IV — Guion de defensa


**Duración:** 15–20 minutos por grupo.
**Formato:** exposición + demo en vivo + fallo en vivo + preguntas individuales.

---

## Estructura

| Etapa | Tiempo | Quién | Contenido |
|---|---|---|---|
| 1. Problema | 2 min | un integrante | ¿qué problema empresarial resolvieron? |
| 2. Arquitectura | 3 min | un integrante | cómo funciona la infraestructura, etapa por etapa |
| 3. Implementación | 3 min | un integrante | qué servicios configuraron y **por qué** |
| 4. Demo | 5 min | el grupo | servidor funcionando en vivo |
| 5. Fallo | 5 min | el grupo | diagnostican un problema que entrega el docente |
| 6. Preguntas | 2 min | cada uno | preguntas individuales |

Las etapas 1–3 las hace **un distinto por etapa**. No puede hablar siempre el mismo.

---

## Qué se espera en cada etapa

### 1. Problema (2 min)

No es resumen del enunciado. Es: qué tenían antes la empresa y qué tienen ahora.

Punto a favor si mencionan: qué habría pasado si no hacían nada.

### 2. Arquitectura (3 min)

El mapa de la sección 5.2 del README, pero **explicado en voz alta**, diciendo qué ocurre técnicamente en cada etapa:

> "El cliente envía el paquete al puerto X; UFW lo acepta porque…; el servicio lo recibe…; los datos viven en…"

Si dicen solo "pasa por el firewall y llega al servicio", es insuficiente.

### 3. Implementación (3 min)

Cada servicio con su **porqué**:

- "Montamos X porque…"
- "Este permiso es Y porque el usuario Z necesita…"
- "Abrimos el puerto N porque…"
- "El respaldo se hace a las … porque…"

### 4. Demo (5 min)

En vivo, desde una máquina que no sea el servidor:

1. la aplicación: login, consulta y modificación con datos reales;
2. la política de archivos: un acceso **permitido** y uno **denegado** con cuentas reales;
3. la base de datos: `app_user` opera, `consulta` intenta modificar y queda bloqueado;
4. un log con evidencia;
5. el respaldo con su fecha;
6. si aplica: resultado del reinicio de la clase 3.

### 5. Fallo en vivo (5 min)

El docente entrega un problema nuevo (del catálogo o improvisado con el mismo patrón). El grupo trabaja con la secuencia:

```
SÍNTOMA → ESTADO → LOG → CAUSA → SOLUCIÓN → VERIFICACIÓN
```

Se observa:

- ¿usa herramientas de diagnóstico o "prueba y error" a ciegas?;
- ¿sabe qué log revisar?;
- ¿repara y verifica desde otra máquina?;
- ¿explica qué pasó?

### 6. Preguntas individuales (2 min)

Cada integrante responde al menos una pregunta de su rol.

---

## Preguntas individuales por módulo

### Rol A — Red y acceso

- ¿Qué puertos tiene abiertos su servidor y por qué exactamente esos?
- ¿Cómo comprobaron que el firewall funciona desde **fuera** del servidor?
- Si el docente no puede conectarse por SSH, ¿por dónde empiezan?
- ¿Qué diferencia hay entre escuchar en un puerto y tenerlo abierto en UFW?

### Rol B — Web

- ¿Qué servicios corren para que la aplicación responda y cómo lo comprueban?
- ¿Dónde están los logs de Apache/PHP y qué buscaron ahí?
- ¿Por qué el sitio está en un Virtual Host y no en la configuración por defecto?
- ¿Cómo saben que la aplicación arranca sola tras un reinicio?

### Rol C — Datos

- ¿Qué perfiles tiene MariaDB y qué puede hacer cada uno?
- ¿Por qué la aplicación no se conecta con root?
- ¿Dónde está el respaldo de la BD, con qué fecha y cómo lo restauraron?
- ¿Qué verifican después de una restauración?

### Rol D — Archivos y documentación

- ¿Quién puede acceder a qué directorio y por qué? Justifique la política.
- ¿Por qué no usaron `chmod 777`?
- ¿Cómo comparten los archivos: qué servicio, qué puertos, restringido a quién?
- Si mañana desaparecen, ¿un administrador nuevo se hace cargo con su README? ¿Qué incidentes registraron?

---

## Regla de la defensa

Si un estudiante dice:

> "El grupo hizo eso."

Se le responde:

> **"Perfecto. ¿Y tú qué configuraste?"**

Cada estudiante debe poder explicar al menos una parte concreta de la infraestructura y demostrarla en el servidor.

Si un estudiante no puede explicar su parte, la calificación **individual** de defensa es cero, sin importar cómo le fue al grupo en el resto.

---

## Preguntas de cierre (iguales para los 5 grupos)

| # | Pregunta |
|---|---|
| 1 | Si mañana reinicio completamente el servidor, ¿qué necesitan comprobar para saber que la web, la base de datos y los archivos vuelven a funcionar? |
| 2 | Si aplican `chmod 777` a `/empresa`, ¿qué problemas concretos se producen en esta empresa y cómo lo comprobarían? |
| 3 | Si la aplicación necesita INSERT y UPDATE pero el perfil `consulta` no debe modificar nada, ¿cómo se lo impiden y cómo comprueban que está bloqueado? |
| 4 | ¿Por qué no basta con que un servicio esté `active`? ¿Qué les falta para que vuelva tras un reinicio? |
| 5 | Si la página carga pero los archivos de `/empresa` no se ven desde otra máquina, ¿por dónde empiezan a mirar? |

---

## Criterio rápido de calificación de la defensa (10 pts)

| Ítem | Pts |
|---|---|
| Resuelve la etapa que le tocó, con propiedad | 2 |
| Explica el **porqué** de sus configuraciones, no solo el qué | 2 |
| Demo en vivo completa y correcta | 2 |
| Diagnóstico del fallo: método ordenado y reparación verificada | 2 |
| Respuesta individual: cada integrante domina su parte | 2 |
| **Total** | **10** |

Cada ítem de "individual" se reparte entre los integrantes: quien no demuestra su parte, no saca su porción.
