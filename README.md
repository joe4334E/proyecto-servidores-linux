# Servidor empresarial con Linux

**Proyecto integrador — Sistemas Operativos · Ingeniería de Sistemas**
**Modalidad:** 5 grupos de 3–4 estudiantes · 3 clases de laboratorio + 1 sesión de defensa · el mismo problema para todos
**Entorno:** un servidor Linux limpio por grupo (VM en la laptop de los estudiantes)
**Entrega:** repositorio Git con documentación en formato `.md`

> **Anuncio para pegar:**
> **Proyecto integrador — Servidor empresarial.** Cada grupo recibirá un servidor Linux **limpio** y deberá convertirlo en la infraestructura de una empresa: publicar su aplicación web con base de datos, organizar los documentos de 4 departamentos con una política de acceso correcta, administrarlo por SSH, protegerlo con firewall, registrar actividad en logs, respaldar y restaurar información, y documentarlo para que otro administrador pueda hacerse cargo. Además, cada estudiante entregará sus propios scripts de bash. No les daremos el procedimiento: les damos el problema. Investigan, prueban y documentan. Entrega: repositorio Git con README completo.

---

## Objetivo

Tomar un servidor Linux nuevo y configurarlo con los ajustes esenciales que todo servidor de producción debería tener: acceso remoto seguro (SSH), firewall, usuarios y permisos, servicios administrados, registro de actividad y respaldos — y usarlo para alojar la aplicación web y los documentos de una empresa. Al finalizar dispondrás de un servidor reforzado, documentado y listo para operar, y sabrás diagnosticar y reparar cuando algo se rompa.

La aplicación y los datos serán proporcionados por el docente. **No deben desarrollarla.** El trabajo está en convertir el servidor Linux en la plataforma que la sostiene.

---

## Requisitos

1. **SSH y acceso remoto.** Usuario administrador propio, entrada desde la laptop por SSH (nunca `root` directamente) y toda verificación hecha **desde otra máquina**, no desde el servidor.
2. **Firewall (UFW).** Solo los puertos justificados abiertos (22 y 80), comprobados desde fuera del servidor; saber añadir y quitar reglas.
3. **Servicios y logs.** `systemctl` y `journalctl` usados con casos reales; cada servicio del proyecto `enable` para que vuelva tras un reinicio.
4. **Web.** Apache + PHP con Virtual Host publicando la aplicación de la empresa, accesible desde otra máquina, con sus logs revisados.
5. **Datos.** MariaDB con la aplicación conectada a su usuario propio (`app_user` opera; el perfil `consulta` no puede modificar nada); respaldo y restauración de la base de datos demostrados.
6. **Archivos.** Estructura `/empresa` con los 4 departamentos — recibe los documentos desordenados de una empresa real, entregados por el docente — con usuarios y grupos por departamento y una política de acceso verificada con cuentas reales (un acceso permitido y uno denegado).
7. **Scripts (entrega individual).** Cada estudiante escribe sus propios scripts en bash: `organizar.sh`, que toma los documentos desordenados y los clasifica/copia por extensión, y al menos un segundo script a elección (respaldo, reporte de logs, inventario del servidor). El grupo además entrega `backup.sh` y `restore.sh`.
8. **Respaldo, restauración y reinicio.** Respaldos de BD y archivos con fecha; tras `reboot`, SSH, firewall, web, BD y archivos vuelven solos, sin intervención manual.
9. **Documentación.** Su `README.md` (descripción, arquitectura, usuarios, servicios, puertos, firewall, logs, respaldos, problemas y recuperación) e `INCIDENTES.md` con al menos 2 entradas. Test: *si mañana desaparecen, ¿otro administrador se hace cargo con ese README?*
10. **Diagnóstico.** En la sesión 4, diagnostican y reparan un fallo inyectado en el servidor de otro grupo siguiendo la secuencia síntoma → estado → log → causa → reparación → verificación.

**Fuera de alcance** (no se evalúa): despliegue con Git, Docker/contenedores, dominio, HTTPS y Samba/compartición de archivos en red.

---

## Reglas del proyecto

1. El docente **no** configurará el servidor por el grupo y **no** entregará el paso a paso. Investigan, prueban y documentan.
2. Cuando aparezca un problema, el grupo deberá investigarlo y demostrar **cómo encontró la causa**.
3. No se permite que solo uno o dos integrantes trabajen: cada estudiante deberá poder explicar una parte concreta de la infraestructura y su script individual.
4. `chmod 777` (o cualquier permisivo total) como solución universal se considera **incorrecto**.
5. Recibirán la aplicación y la base de datos; el servidor llega **limpio**.
6. Toda comprobación se hace **desde otra máquina**, no desde el propio servidor.

---

## Roles del equipo

| Rol | Responsable de |
|---|---|
| **A — Red y acceso** | SSH, UFW, puertos, verificación desde otra máquina |
| **B — Web** | Virtual Host, Apache/PHP, aplicación publicada, logs del web server |
| **C — Datos** | perfiles de MariaDB, conexión de la aplicación, respaldo y restauración de la BD |
| **D — Archivos y documentación** | `/empresa`, usuarios/grupos, política de permisos, `backup.sh`/`restore.sh`, README e `INCIDENTES.md` |

En grupos de 3 integrantes, el rol D se comparte, pero la documentación igualmente se entrega. Los scripts individuales (`organizar.sh` + uno a elección) son responsabilidad de **cada estudiante**, sin importar su rol.

---

## Entrega — repositorio Git

```
repo-grupo-N/
├── README.md          su documentación completa (se verá abajo)
├── INCIDENTES.md      bitácora de fallos diagnosticados
└── scripts/
    ├── backup.sh          respaldo de BD y archivos (grupo)
    ├── restore.sh         restauración (grupo)
    ├── organizar.sh       clasifica documentos por extensión (individual)
    └── ...                otros scripts individuales (reporte, respaldo, inventario)
```

**El README de su repositorio debe incluir:** descripción, arquitectura, instalación, configuración, usuarios, servicios, puertos, firewall, logs, backup, restore, scripts, problemas encontrados, soluciones y procedimiento de recuperación.

`INCIDENTES.md` usa esta tabla:

| # | Fecha | Síntoma | Comandos usados para diagnosticar | Causa | Solución |
|---|---|---|---|---|---|

Además, al final de la clase 3: IP, usuario y contraseña de administración, y una cuenta de prueba para el docente.

---

## Cronograma y checklists

| Sesión | Resultado esperado |
|---|---|
| Clase 1 | **Base común:** SSH, systemctl/journalctl, UFW, Apache + PHP + MariaDB publicando, inventario inicial, roles asignados |
| Clase 2 | **Núcleo:** aplicación, datos y política de `/empresa` funcionando, logs, primeros respaldos, `organizar.sh` en marcha |
| Clase 3 | **Demostrar y congelar:** pruebas del docente, restauración, reinicio completo, scripts individuales, README completo, entrega |
| Sesión 4 | **Intercambio + defensa:** diagnostican un fallo ajeno y defienden su trabajo individualmente |

### Cierre de clase 1

- [ ] Todos los integrantes entran por SSH con su propia cuenta
- [ ] `systemctl` y `journalctl` usados con un caso real
- [ ] Apache + PHP publicando desde otra máquina
- [ ] MariaDB con base `baseapp` y usuario `app` (nunca root)
- [ ] UFW activo: 22 y 80, verificado desde fuera
- [ ] Inventario inicial + primer commit
- [ ] Roles (A/B/C/D) asignados con nombres
- [ ] `INCIDENTES.md` con al menos 1 entrada

### Cierre de clase 2

- [ ] Aplicación real accesible desde otra máquina con login y datos
- [ ] Política de `/empresa` verificada: acceso permitido y denegado con cuentas reales
- [ ] `consulta` bloqueado por MariaDB; `app_user` operando
- [ ] Puertos abiertos verificados fuera; los demás cerrados
- [ ] Al menos un error provocado y encontrado en los logs
- [ ] Respaldos de BD y de archivos ejecutados con fecha
- [ ] `organizar.sh` propio ejecutado sobre los documentos de la empresa
- [ ] Su README avanzado (inventario, mapa, puertos, usuarios, firewall, logs, respaldos)

### Cierre de clase 3

- [ ] Prueba de restauración completada y registrada
- [ ] Pruebas del docente aprobadas (web, archivos, BD, seguridad)
- [ ] Servidor reiniciado y todo volvió solo
- [ ] Scripts individuales en el repo y ejecutados (`organizar.sh` + 1 a elección)
- [ ] README completo
- [ ] `INCIDENTES.md` con al menos 2 entradas
- [ ] Último commit y credenciales entregadas
- [ ] Snapshot tomado por el docente (antes del intercambio)

### Sesión 4

- [ ] Fallo ajeno diagnosticado, reparado y verificado desde otra máquina
- [ ] Entrada del caso en `INCIDENTES.md`
- [ ] Cada integrante defendió su módulo
