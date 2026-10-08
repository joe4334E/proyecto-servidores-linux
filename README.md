# PROYECTO INTEGRADOR

## Administración de Servidores Linux

**Asignatura:** Sistemas Operativos
**Carrera:** Ingeniería de Sistemas
**Duración:** 3 clases de laboratorio + 1 sesión de defensa
**Modalidad:** 5 grupos · 3–4 estudiantes por grupo
**Entorno:** un servidor Linux limpio por grupo, red del aula
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

La aplicación será proporcionada por el docente. **No deben desarrollarla.** El trabajo está en convertir el servidor Linux en la plataforma que la sostiene.

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

---

## 5. Lo que se evalúa

El grupo deberá demostrar que puede:

- administrar usuarios y grupos;
- aplicar permisos correctamente;
- instalar y administrar servicios;
- configurar SSH;
- controlar puertos mediante firewall;
- identificar servicios activos;
- consultar logs;
- realizar respaldos;
- recuperar información;
- diagnosticar fallos;
- documentar la infraestructura;
- acceder remotamente;
- **explicar por qué realizó cada configuración**.

---

## 6. Requisitos comunes

### 6.1 Inventario del servidor

| Elemento | Configuración |
|---|---|
| Sistema operativo | |
| IP | |
| Hostname | |
| SSH (puerto, usuario, autenticación) | |
| Servicios instalados | |
| Servicios activos | |
| Puertos abiertos | |
| Usuarios | |
| Grupos | |
| Firewall | |

### 6.2 Mapa de infraestructura

No se acepta un dibujo decorativo. Deben explicar qué ocurre **técnicamente** en cada etapa:

```
CLIENTE
   │
   ▼
RED
   │
   ▼
FIREWALL
   │
   ▼
SERVICIO
   │
   ▼
DATOS
```

### 6.3 Tabla de puertos

| Puerto | Servicio | Motivo |
|---|---|---|
| 22 | SSH | administración remota |
| 80 | HTTP | aplicación web |
| 139/445 | Samba | archivos departamentales (restringido a la red del aula) |

Justificar **por qué cada puerto está abierto** y por qué los demás están cerrados.

### 6.4 Usuarios y permisos

Deben poder responder, para cada recurso:

```
¿QUIÉN?
   ↓
¿ACCEDE A QUÉ?
   ↓
¿CON QUÉ PERMISOS?
   ↓
¿POR QUÉ?
```

### 6.5 Firewall

- Comprobar qué puertos están abiertos y cuáles no.
- Justificar cada regla.
- **Verificarlo desde otra máquina**, no desde el propio servidor.

### 6.6 Logs

No basta con *"el servicio falló"*. Deben mostrar la cadena completa:

```
SERVICIO → ESTADO → LOG → ERROR → CAUSA → SOLUCIÓN
```

### 6.7 Backup

Demostrar en vivo el ciclo completo, aplicado a **la base de datos y a los archivos**:

```
ORIGINAL → BACKUP → PROBLEMA → RESTORE → VERIFICACIÓN
```

### 6.8 Reinicio completo

**Reiniciar el servidor** y comprobar que todo vuelve solo: SSH, firewall, Apache, MariaDB, Samba, datos. Si algo no vuelve, es su problema que diagnosticar.

### 6.9 Documentación

Un `README.md` que otra persona pueda utilizar:

> "Si mañana ustedes desaparecen de la empresa, ¿otro administrador puede hacerse cargo del servidor?"

Secciones obligatorias:

1. Descripción
2. Arquitectura
3. Requisitos
4. Instalación
5. Configuración
6. Usuarios
7. Servicios
8. Puertos
9. Firewall
10. Logs
11. Backup
12. Restore
13. Problemas encontrados
14. Soluciones
15. Procedimiento de recuperación

---

## 7. Roles del equipo

| Rol | Responsable de |
|---|---|
| **A — Red y acceso** | SSH, UFW, puertos, verificación desde otra máquina |
| **B — Web** | Virtual Host, Apache/PHP, aplicación publicada, logs del web server |
| **C — Datos** | perfiles de MariaDB, conexión de la aplicación, respaldo y restauración de la BD |
| **D — Archivos y documentación** | `/empresa`, usuarios/grupos, Samba, política de permisos, respaldo de archivos, README e `INCIDENTES.md` |

En grupos de 3 integrantes, el rol D se comparte, pero la documentación igualmente se entrega.

---

## 8. Entrega — repositorio Git

```
repo-grupo-N/
├── README.md          manual completo (las 14 secciones)
├── INCIDENTES.md      bitácora de fallos diagnosticados
└── scripts/
    ├── backup.sh      (respaldos BD y/o archivos)
    └── restore.sh
```

Además, al final de la clase 3: IP, usuario y contraseña de administración, y una cuenta de prueba para el docente.

`INCIDENTES.md` usa esta tabla:

| # | Fecha | Síntoma | Comandos usados para diagnosticar | Causa | Solución |
|---|---|---|---|---|---|

---

## 9. Cronograma

| Sesión | Nombre | Resultado esperado |
|---|---|---|
| Clase 1 | **Base común** | SSH, systemctl/journalctl, UFW, Apache + PHP + MariaDB publicando, inventario inicial, roles asignados |
| Clase 2 | **Núcleo** | módulos A, B y C funcionando: app publicada, perfiles de BD, archivos con política, logs, primeros respaldos |
| Clase 3 | **Demostrar y congelar** | pruebas del docente, restauración, reinicio completo, README completo, entrega |
| Sesión 4 | **Intercambio + defensa** | diagnostican un fallo ajeno y defienden su trabajo individualmente |

3 clases de laboratorio. La clase 1 es una **base común**: los 5 grupos construyen lo mismo y desde ahí arranca el proyecto — **idéntico para todos**. Cada clase cierra con un **hito verificable**: el docente comprueba el resultado y firma. No se avanza con el servidor a medias.

### Clase 1 — Base común (todos, guiada)

**Resultado:** las 5 VMs publicando una página PHP conectada a MariaDB, con SSH, UFW y servicios activos; inventario inicial, roles asignados y `INCIDENTES.md` con su primera entrada.

**Materiales:** guía del docente en `BASE-DOCENTE.md`, hoja del estudiante en `BASE-PRACTICA.md` (8 fases).

Es la única clase con procedimiento entregado: el docente enseña y demuestra; los estudiantes practican.

#### Cierre de clase 1 — checklist

- [ ] Todos los integrantes entran por SSH con su propia cuenta.
- [ ] `systemctl` y `journalctl` usados con un caso real.
- [ ] Apache + PHP publicando desde otra máquina.
- [ ] MariaDB con base `baseapp` y usuario `app` (nunca root).
- [ ] UFW activo: 22 y 80, verificado desde fuera.
- [ ] Inventario inicial + primer commit.
- [ ] Roles (A/B/C/D) asignados con nombres.
- [ ] `INCIDENTES.md` con al menos 1 entrada.

### Clase 2 — Núcleo (módulos A, B, C)

**Resultado:** los tres módulos del proyecto funcionando: aplicación web real publicada, perfiles de base de datos, archivos departamentales con política de acceso, logs visibles y primeros respaldos. Aquí rige la regla del proyecto: **no se entrega procedimiento**.

#### Secuencia recomendada (por dependencias)

1. **Módulo C — Archivos** (arranca primero: no depende de la web)
   - usuarios y grupos de los 4 departamentos;
   - estructura `/empresa`;
   - Samba compartiendo;
   - política de acceso aplicada (sin `chmod 777`) y verificada con una cuenta sí / una cuenta no;
   - logs de acceso localizados.

2. **Módulo A — Web** (sobre la base de la Clase 1)
   - Virtual Host con `ServerName`;
   - aplicación real del docente publicada;
   - permisos de directorio justificados;
   - logs de Apache/PHP localizados con un error provocado y encontrado.

3. **Módulo B — Datos** (la BD ya existe: se endurece)
   - perfiles `app_user` (la aplicación) y `consulta` (solo lectura) con privilegios distintos;
   - la aplicación conecta con `app_user`, nunca con root;
   - intento de modificación con `consulta` → **bloqueado y demostrado**;
   - primer respaldo de la BD (`mysqldump` o equivalente) con fecha.

4. **Transversal de la clase 2**
   - puertos de cada módulo abiertos en UFW **y verificados desde otra máquina**;
   - `INCIDENTES.md` actualizado;
   - README: secciones 6.1 a 6.9 avanzadas con su estado real.

#### Cierre de clase 2 — checklist

- [ ] Aplicación real accesible desde otra máquina con login y datos.
- [ ] Política de archivos verificada: acceso permitido y denegado con cuentas reales.
- [ ] `consulta` bloqueado por MariaDB; `app_user` operando.
- [ ] Puertos abiertos verificados fuera; los demás cerrados.
- [ ] Al menos un error provocado y encontrado en los logs.
- [ ] Respaldos de BD y de archivos ejecutados con fecha.
- [ ] README avanzado (inventario, mapa, puertos, usuarios, firewall, logs, respaldos).

### Clase 3 — Demostrar y congelar

**Resultado:** el servidor pasa las pruebas del docente, sobrevive un reinicio completo, está documentado y queda congelado con un snapshot.

#### Tareas

1. **Terminar la documentación** — README completo con las 14 secciones + `INCIDENTES.md`.
2. **Prueba de restauración interna** — antes del docente: romper algo ellos mismos (datos de BD o archivos), restaurar desde el respaldo y verificar. Registrar en `INCIDENTES.md`.
3. **Pruebas del docente** — web, archivos, BD, recuperación, seguridad (sección 10).
4. **Reinicio completo** — después: SSH, firewall, Apache, MariaDB, Samba y datos vuelven solos. Lo que no vuelva, lo diagnostican y anotan.
5. **Entrega en Git** — último commit. IP, usuario, contraseña y cuenta de prueba.
6. **Preparar la defensa** — cada integrante prepara su módulo (3 minutos).

#### Cierre de clase 3 — checklist

- [ ] Prueba de restauración completada y registrada.
- [ ] Pruebas del docente aprobadas (web, archivos, BD, seguridad).
- [ ] Servidor reiniciado y todo volvió solo.
- [ ] README completo con las 14 secciones.
- [ ] `INCIDENTES.md` con al menos 2 entradas.
- [ ] Último commit y credenciales entregadas.
- [ ] Snapshot tomado por el docente (antes del intercambio).

### Sesión 4 — Intercambio + defensa

No hay tareas de construcción. Es evaluación.

1. El docente toma snapshot de los 5 servidores.
2. Inyecta **un fallo** por servidor (Parte II de `DOCENTE.md`) — puede ser el **mismo** para los 5, así el diagnóstico es comparable.
3. Cada grupo recibe IP + credenciales + `README.md` del servidor ajeno.
4. Diagnostican y reparan. Registran en `INCIDENTES.md`: síntoma → comandos → causa → solución → prevención.
5. El docente verifica que el servicio afectado quedó funcionando desde otra máquina.
6. **Defensa** — guion completo en la Parte IV de `DOCENTE.md`. Cada estudiante explica **su módulo**.

---

## 10. Pruebas del docente

Durante la clase 3, el docente ejecutará:

1. **Web** — abrir la aplicación desde otra máquina, iniciar sesión, consultar, modificar; los datos deben **persistir tras reiniciar**.
2. **Archivos** — con cuentas reales: acceso permitido a su departamento, acceso **denegado** al de otro departamento.
3. **Base de datos** — `consulta` intenta modificar: **bloqueado por MariaDB**; `app_user` opera lo que le corresponde.
4. **Recuperación** — el docente eliminará o alterará información (datos de la BD o archivos). El grupo debe:
   **identificar → localizar el respaldo → restaurar → verificar**.
5. **Seguridad** — puertos y recursos que no le corresponden: denegados y justificados.
6. **Reinicio** — apagar y encender: todo vuelve solo.

---

## 11. Sesión 4 — intercambio y fallo controlado

Nadie administra su propio servidor:

| El grupo… | administra el servidor… |
|---|---|
| 1 | del grupo 3 |
| 2 | del grupo 5 |
| 3 | del grupo 4 |
| 4 | del grupo 1 |
| 5 | del grupo 2 |

Cada grupo recibe: IP, usuario, contraseña y el `README.md` del grupo dueño. **Y nada más.**

Previamente el docente inyecta **un fallo** (catálogo en la Parte II de `DOCENTE.md` — puede ser el mismo fallo para los 5 grupos, así se comparan directamente).

El grupo debe diagnosticar, reparar, verificar y registrarlo en `INCIDENTES.md`.

---

## 12. Defensa final

15–20 minutos por grupo. Estructura en la Parte IV de `DOCENTE.md`:

| Etapa | Tiempo |
|---|---|
| Problema | 2 min |
| Arquitectura | 3 min |
| Implementación | 3 min |
| Demo en vivo | 5 min |
| Fallo en vivo | 5 min |
| Preguntas individuales | 2 min |

**Regla individual:** si un estudiante dice *"el grupo hizo eso"*:

> "Perfecto. ¿Y tú qué configuraste?"

---

## 13. Evaluación

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
| README con las 14 secciones obligatorias, completo y actualizado | 4 |
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

Ver Parte IV de `DOCENTE.md`.

| Ítem | Pts |
|---|---|
| Cada integrante explica la etapa que le tocó | 2 |
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

---

## 14. Al terminar

El resultado esperado es que cada grupo haya pasado el ciclo real:

```
INSTALAR → CONFIGURAR → ASEGURAR → PUBLICAR → MONITOREAR
   → ROMPER → DIAGNOSTICAR → RECUPERAR → DOCUMENTAR
   → ADMINISTRAR INFRAESTRUCTURA AJENA
```
