# PROYECTO INTEGRADOR

## Administración de Servidores Linux

**Asignatura:** Sistemas Operativos · **Carrera:** Ingeniería de Sistemas
**Duración:** 3 clases de laboratorio + 1 sesión de defensa
**Modalidad:** 5 grupos · 3–4 estudiantes por grupo
**Entorno:** un servidor Linux limpio por grupo (VM en su laptop), red del aula
**Entrega:** repositorio Git con documentación en formato `.md`

> **Anuncio para pegar:**
> **Proyecto integrador — Servidor empresarial.** Cada grupo recibirá un servidor Linux **limpio** y deberá convertirlo en la infraestructura de una empresa: publicar su aplicación web con base de datos, centralizar los archivos de 4 departamentos con una política de acceso coherente, administrarlo por SSH, protegerlo con firewall, registrar actividad en logs, respaldar y restaurar información, y documentarlo para que otro administrador pueda hacerse cargo. No les daremos el procedimiento: les damos el problema. Investigan, prueban y documentan. Los 5 grupos reciben **el mismo problema** — se compararán entre ustedes. Entrega: repositorio Git con README completo.

---

## 1. Situación problemática

Una pequeña empresa trabaja así hoy:

- tiene una aplicación web (clientes, productos, ventas) desarrollada por un equipo externo, pero **no infraestructura propia** para alojarla;
- guarda los documentos de administración, ventas, desarrollo y dirección en **computadoras individuales**: hay duplicados, pérdida de información y gente viendo documentos que no le corresponden;
- no tiene respaldos ni forma de saber quién accedió a qué;
- no tiene una forma remota y segura de administrar sus equipos.

La empresa solicita **un servidor Linux** que resuelva los tres frentes a la vez.

---

## 2. Objetivo

Implementar un servidor Linux capaz de:

1. **publicar la aplicación web** de la empresa, conectada a MariaDB;
2. **centralizar los archivos** de los departamentos con una política de acceso correcta;
3. ser **administrable remotamente**, protegido, registrado, respaldado y documentado.

La aplicación y los datos serán proporcionados por el docente. **No deben desarrollarla.** El trabajo está en convertir el servidor Linux en la plataforma que la sostiene.

---

## 3. Alcance del proyecto

| Módulo | Contenido |
|---|---|
| **Base común** (Clase 1) | SSH, systemctl, logs, puertos, UFW, Apache + PHP + MariaDB publicando una página |
| **A — Web** | Virtual Host con `ServerName`, aplicación real publicada, permisos de directorio, logs de Apache/PHP |
| **B — Datos** | MariaDB con perfiles de privilegio distintos (`app_user` y `consulta`), aplicación conectada con su usuario propio, respaldo y restauración de la base de datos |
| **C — Archivos** | estructura `/empresa` con 4 departamentos, usuarios y grupos, Samba, política de acceso verificable, respaldo de archivos |

**Fuera de alcance** (no se evalúa): despliegue con Git, Docker/contenedores, dominio y HTTPS.

---

## 4. Condición general

1. El docente **no** configurará el servidor por el grupo.
2. El docente **no** entregará el paso a paso. Investigan, prueban y documentan.
3. Cuando aparezca un problema, el grupo deberá investigarlo y demostrar **cómo encontró la causa**.
4. No se permite que solo uno o dos integrantes trabajen. Cada estudiante deberá poder explicar una parte concreta de la infraestructura.
5. `chmod 777` como solución universal se considera **incorrecto**.
6. Recibirán la aplicación y la base de datos; el servidor llega **limpio**.
7. Toda comprobación se hace **desde otra máquina**, no desde el propio servidor.

---

## 5. Roles del equipo

| Rol | Responsable de |
|---|---|
| **A — Red y acceso** | SSH, UFW, puertos, verificación desde otra máquina |
| **B — Web** | Virtual Host, Apache/PHP, aplicación publicada, logs del web server |
| **C — Datos** | perfiles de MariaDB, conexión de la aplicación, respaldo y restauración de la BD |
| **D — Archivos y documentación** | `/empresa`, usuarios/grupos, Samba, política de permisos, respaldo de archivos, README e `INCIDENTES.md` |

En grupos de 3 integrantes, el rol D se comparte, pero la documentación igualmente se entrega.

---

## 6. Entrega — repositorio Git

```
repo-grupo-N/
├── README.md          su documentación completa (se verá abajo)
├── INCIDENTES.md      bitácora de fallos diagnosticados
└── scripts/
    ├── backup.sh      (respaldos BD y/o archivos)
    └── restore.sh
```

**El README de su repositorio debe incluir:** descripción, arquitectura, instalación, configuración, usuarios, servicios, puertos, firewall, logs, backup, restore, problemas encontrados, soluciones y procedimiento de recuperación. Test: *si mañana desaparecen, ¿otro administrador se hace cargo con ese README?*

`INCIDENTES.md` usa esta tabla:

| # | Fecha | Síntoma | Comandos usados para diagnosticar | Causa | Solución |
|---|---|---|---|---|---|

Además, al final de la clase 3: IP, usuario y contraseña de administración, y una cuenta de prueba para el docente.

---

## 7. Cronograma y checklists

| Sesión | Resultado esperado |
|---|---|
| Clase 1 | **Base común:** SSH, systemctl/journalctl, UFW, Apache + PHP + MariaDB publicando, inventario inicial, roles asignados |
| Clase 2 | **Núcleo:** módulos A, B y C funcionando, logs, primeros respaldos |
| Clase 3 | **Demostrar y congelar:** pruebas del docente, restauración, reinicio completo, README completo, entrega |
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
- [ ] Política de archivos verificada: acceso permitido y denegado con cuentas reales
- [ ] `consulta` bloqueado por MariaDB; `app_user` operando
- [ ] Puertos abiertos verificados fuera; los demás cerrados
- [ ] Al menos un error provocado y encontrado en los logs
- [ ] Respaldos de BD y de archivos ejecutados con fecha
- [ ] Su README avanzado (inventario, mapa, puertos, usuarios, firewall, logs, respaldos)

### Cierre de clase 3

- [ ] Prueba de restauración completada y registrada
- [ ] Pruebas del docente aprobadas (web, archivos, BD, seguridad)
- [ ] Servidor reiniciado y todo volvió solo
- [ ] README completo
- [ ] `INCIDENTES.md` con al menos 2 entradas
- [ ] Último commit y credenciales entregadas
- [ ] Snapshot tomado por el docente (antes del intercambio)

### Sesión 4

- [ ] Fallo ajeno diagnosticado, reparado y verificado desde otra máquina
- [ ] Entrada del caso en `INCIDENTES.md`
- [ ] Cada integrante defendió su módulo

---

## 8. Evaluación

| Criterio | Puntos |
|---|---|
| Infraestructura funcional (web + datos + archivos) | 40 |
| Seguridad: SSH, firewall, usuarios y permisos | 15 |
| Respaldo, restauración y reinicio | 15 |
| Documentación y bitácora | 10 |
| Diagnóstico de fallo ajeno (intercambio) | 10 |
| Defensa oral individual | 10 |
| **Total** | **100** |

### 1. Infraestructura funcional — 40 pts

| Ítem | Pts |
|---|---|
| **Módulo A — Web:** aplicación real publicada con Virtual Host, accesible desde otra máquina, con login y operación sobre datos | 12 |
| **Módulo B — Datos:** perfiles `app_user` y `consulta` con privilegios distintos; la aplicación conecta con su usuario propio; intento de modificación con `consulta` bloqueado en vivo | 10 |
| **Módulo C — Archivos:** `/empresa` con departamentos, Samba publicando y política de acceso verificada (permitido y denegado con cuentas reales) | 10 |
| Servicios activos y configurados para arrancar con el sistema (`enable`) | 4 |
| Trabajo distribuido: se evidencia que todos participaron en la implementación | 4 |

### 2. Seguridad — 15 pts

| Ítem | Pts |
|---|---|
| SSH funcionando con usuario administrador; `root` no usado directamente | 3 |
| UFW activo: solo puertos justificados abiertos, verificado **desde otra máquina** | 5 |
| Usuarios y grupos creados según la necesidad del proyecto | 3 |
| Política de permisos coherente y justificada; `chmod 777` = 0 en este ítem | 4 |

### 3. Respaldo, restauración y reinicio — 15 pts

| Ítem | Pts |
|---|---|
| Respaldo de la BD y de los archivos ejecutados, con fecha | 4 |
| Restauración demostrada en vivo (datos de BD o archivos) con verificación | 6 |
| Reinicio completo: SSH, firewall, Apache, MariaDB y Samba vuelven sin intervención manual | 5 |

### 4. Documentación — 10 pts

| Ítem | Pts |
|---|---|
| README completo (las secciones de la Entrega) y actualizado | 4 |
| Inventario y mapa de infraestructura explicados técnicamente | 3 |
| `INCIDENTES.md` con al menos 2 entradas completas (síntoma → comandos → causa → solución) | 3 |

### 5. Diagnóstico de fallo ajeno — 10 pts

| Ítem | Pts |
|---|---|
| Identificaron el fallo inyectado | 3 |
| Método de diagnóstico ordenado (estado → log → causa), no a ciegas | 3 |
| Reparación aplicada **y verificada** desde otra máquina | 2 |
| Registraron el caso en `INCIDENTES.md` con prevención propuesta | 2 |

### 6. Defensa oral individual — 10 pts

| Ítem | Pts |
|---|---|
| Cada integrante explica su módulo | 2 |
| Cada integrante explica el **porqué** de sus configuraciones | 2 |
| Demo en vivo completa | 2 |
| Diagnóstico del fallo en vivo | 2 |
| Respuesta individual: domina su parte concreta | 2 |

**Nota:** el ítem 6 se califica **por estudiante**. Un integrante que no demuestra su parte no obtiene sus 2 puntos de respuesta individual, sin importar el resultado grupal.

### Criterios de anulación

Se anula el puntaje del ítem correspondiente cuando:

- `chmod 777` (o equivalente permisivo total) se usa como solución de permisos;
- la verificación de firewall o acceso se hace solo desde el propio servidor;
- el respaldo existe pero no se demostró la restauración;
- la documentación no refleja el servidor real (copiada de otro grupo o genérica);
- en la defensa, un estudiante responde "lo hizo el grupo" sin poder explicar su parte.

### Escala sugerida

| Puntos | Desempeño |
|---|---|
| 90–100 | Excelente: podría administrar este servidor en producción |
| 75–89 | Sólido: funciona, con deudas menores de documentación o detalle |
| 60–74 | Aceptado: funciona pero con huecos (permisos, respaldo o reinicio flojos) |
| < 60 | Insuficiente: el servidor no es mantenible por otro |
