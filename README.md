# CampusFest — Gestión de eventos universitarios

Aplicación de escritorio en **Java 21 + JavaFX 21** para consultar cupos disponibles, gestionar reservas temporales de entradas, editar eventos y registrar ingresos mediante código QR, desarrollada bajo **Arquitectura Limpia** y persistencia embebida en **H2**.

---

## Integrantes del equipo

| Integrante | Usuario GitHub | Módulo y aportes principales |
| --- | --- | --- |
| Santiago Niño | `@Dslegacy07` | Capa `domain` (`EstadoReserva`, `Asistencia`, `Reserva`, `Evento` — `PR #2`, `PR #3`) y revisiones de interfaz (`PR #1`, `PR #9`) |
| Carlos Villamil | `@Carlos-Villamil` | Capa `application` (`RepositorioEventos`, `Reloj`, `ConsultarCupos`, `ReservarEntrada`, `EditarEvento`, `RegistrarIngreso` — `PR #4`, `PR #5`) y revisiones de calidad (`PR #2`, `PR #3`, `PR #6`, `PR #8`) |
| Andres Ramos | `@Andres-Ramos89` | Capa `infrastructure` (`RelojDelSistema`, `RepositorioEventosEnMemoria`, `db/schema.sql`, `RepositorioEventosH2` — `PR #6`, `PR #7`, `PR #8`) y revisiones (`PR #5`, `PR #10`) |
| Juan Sebastian Papa Guerrero | `@Papitas-Chipzhp` | Vistas FXML/CSS (`PR #1`), controladores JavaFX (`PR #9`), ensamblaje (`App`, `Launcher` — `PR #10`), documentación (`PR #11`) y revisiones (`PR #4`, `PR #7`) |

---

## Arquitectura y tecnologías

- **Lenguaje y compilación**: Java 21 (`maven-compiler-plugin:3.13.0`, `<release>21</release>`)
- **Interfaz gráfica**: JavaFX 21.0.5 (`javafx-controls`, `javafx-fxml`, `javafx-maven-plugin:0.0.8`)
- **Persistencia**: H2 Database 2.2.224 (`jdbc:h2:file:./datos/campusfest`, esquema `EVENTO`, `RESERVA`, `ASISTENCIA` en `src/main/resources/db/schema.sql`)
- **Pruebas automatizadas**: JUnit Jupiter 5.10.2 (`maven-surefire-plugin:3.2.5`) — **90 pruebas** en 10 clases de prueba (+ 2 dobles de prueba en `src/test/`)

### Organización por anillos (`co.edu.unillanos.campusfest`)

| Anillo | Paquete | Clases y responsabilidad |
| --- | --- | --- |
| **Entidades** | `domain` | `Evento`, `Reserva`, `Asistencia`, `EstadoReserva`. Reglas de negocio (plazo de 10 minutos, cupos disponibles, aforo mínimo, transiciones de estado e ingreso único). |
| **Puertos** | `application.port` | Interfaces `RepositorioEventos` (`buscar`, `guardar`) y `Reloj` (`ahora`). |
| **Casos de uso** | `application.usecase` | `ConsultarCupos` (CU1), `ReservarEntrada` (CU2), `EditarEvento` (CU3) y `RegistrarIngreso` (CU4). Solo coordinan puertos y entidades. |
| **Adaptadores** | `infrastructure.{persistence,reloj,ui}` | `RepositorioEventosEnMemoria`, `RepositorioEventosH2`, `RelojDelSistema` (única clase que llama a `LocalDateTime.now()`), y los 4 controladores JavaFX (`Pantalla*Controller`). |
| **Ensamblaje y vistas** | Paquete raíz y `resources/.../vistas/` | `App`, `Launcher`, hoja `estilos.css` (solo propiedades `-fx-`) y las 4 vistas `.fxml` sin `fx:controller`. |

---

## Requisitos, ejecución y guía de uso

### Requisitos previos

- **Java**: JDK 21 o superior.
- **Apache Maven**: 3.9 o superior.

### Comandos

```bash
# Ejecutar la suite completa de 90 pruebas automatizadas
mvn clean test

# Abrir la aplicación JavaFX con persistencia local H2 (./datos/campusfest)
mvn javafx:run
```

- La clase principal configurada en `javafx-maven-plugin` es `co.edu.unillanos.campusfest.App` (también se incluye `co.edu.unillanos.campusfest.Launcher` para ejecución directa desde el IDE).
- Al iniciar `App`, si la base local aún no los contiene, se precargan dos eventos de ejemplo:
  - `ev-1`: **Concierto de bienvenida** (`Auditorio Barcelona`, aforo `100`, bloqueados `10`).
  - `ev-2`: **Feria de emprendimiento** (`Plazoleta central`, aforo `50`, bloqueados `5`).

### Casos de uso en la interfaz

1. **Consultar cupos (`CU1`)**: consulta cupos disponibles actuales de cualquier evento por ID (`ev-1`, `ev-2`, etc.).
2. **Reservar entrada (`CU2`)**: genera reserva pendiente con código único y hora de vencimiento (+10 min), permite copiar el código al portapapeles (`Copiar código QR`) y confirmar la reserva dentro del plazo vigente.
3. **Editar evento (`CU3`)**: precarga `ev-1` al abrir la pestaña; permite consultar otro evento por ID y actualizar nombre, fecha (`AAAA-MM-DDTHH:MM`), lugar y aforo máximo respetando `bloqueados + ocupados por reservas`.
4. **Ingreso QR (`CU4`)**: valida el código QR de una reserva confirmada perteneciente al evento y registra la asistencia única con la hora del sistema.

---

## Declaración de uso de IA (Aspecto 8.1)

| Aspecto | Detalle del equipo |
| --- | --- |
| **Herramientas utilizadas** | Asistentes basados en modelos de lenguaje (**Gemini / Claude**) empleados como apoyo de redacción, estructuración de clases, vistas JavaFX y consultas SQL durante el desarrollo por ramas. |
| **Para qué y en qué clases apoyó la IA** | Configuración inicial de `pom.xml`; borradores de `EstadoReserva`, `Asistencia`, `Reserva` y `Evento` (`domain`); puertos `RepositorioEventos`/`Reloj` y casos de uso `ConsultarCupos`, `ReservarEntrada`, `EditarEvento`, `RegistrarIngreso` (`application`); esquema `db/schema.sql` y adaptadores `RelojDelSistema`, `RepositorioEventosEnMemoria` y `RepositorioEventosH2` con `MERGE INTO` (`infrastructure`); vistas `estilos.css`, `pantalla-*.fxml`, controladores `PantallaCuposController`, `PantallaReservaController`, `PantallaEventoController`, `PantallaIngresoController` y ensamblaje en `App` y `Launcher`. |
| **Qué verificó manualmente el equipo** | Cumplimiento estricto de `CONTRATO.md` y los principios GRASP/Arquitectura Limpia en cada Pull Request (`PR #1` a `PR #10`), revisión cruzada entre integrantes antes de fusionar a `main`, integridad intacta de las 12 clases originales en `src/test/`, ejecución manual de las 4 pantallas con persistencia real en H2 (`mvn javafx:run`) y pase completo de `mvn clean test` (**90/90**). |

---

## Propuestas de IA que violaban el diseño y cómo se corrigieron (Aspecto 8.2)

Durante las revisiones cruzadas en Pull Requests se detectaron y corrigieron los siguientes casos antes de aprobar su integración a `main`:

### Caso 1 (`PR #5` — `RegistrarIngreso.java`, rama `feature/casos-uso-reservas-evento-ingreso`)
- **Qué propuso inicialmente la IA** *(commit `a15f28e`)*: en el caso de uso `RegistrarIngreso`, la primera versión importaba `EstadoReserva` y verificaba `if (reserva.getEstado() != EstadoReserva.CONFIRMADA) { throw new IllegalStateException(...); }` antes de llamar a `reserva.registrarIngreso(reloj.ahora())`.
- **Qué regla violaba**: **Experto en información**, encapsulamiento del dominio (consultar `reserva.getEstado()` desde fuera de `Reserva`) y **Regla única** (duplicaba una validación que ya estaba implementada dentro de `Reserva.registrarIngreso`).
- **Cómo se detectó y corrigió**: en la revisión del `PR #5` (`@Andres-Ramos89` marcó *Request changes*) se solicitó eliminar el `import EstadoReserva` y la condición repetida, subiendo el commit de corrección `1ccb825` que delega íntegramente en `reserva.registrarIngreso(reloj.ahora())`.

### Caso 2 (`PR #8` — `RepositorioEventosH2.java`, rama `feature/h2-guardar-transaccional`)
- **Qué propuso inicialmente la IA** *(commit `23085c3`)*: en los manejadores `catch (SQLException e)` de `RepositorioEventosH2`, el adaptador concatenaba `+ e.getMessage()` al mensaje de la excepción `IllegalStateException` propagada hacia los casos de uso y la interfaz gráfica.
- **Qué regla violaba**: aislamiento de infraestructura y protección de errores de persistencia (exponer detalles técnicos de JDBC/SQL hacia capas superiores y pantallas, incumpliendo el Aspecto 6.4).
- **Cómo se detectó y corrigió**: durante la revisión del `PR #8` (`@Carlos-Villamil` marcó *Request changes*) se señaló la filtración del detalle SQL y se corrigió en los commits `ce5830c` y `a6f533f`, conservando `e` únicamente como causa interna y mostrando mensajes claros en español.

### Caso 3 (`PR #9` — `PantallaReservaController.java`, rama `feature/controladores-ui`)
- **Qué propuso inicialmente la IA** *(commit `f0989ba`)*: al construir el mensaje de éxito tras solicitar una reserva, el controlador calculaba la hora de vencimiento localmente con `LocalDateTime.now().plusMinutes(10)` en lugar de consultar el objeto retornado por el caso de uso.
- **Qué regla violaba**: **Regla única** (duplicaba el plazo de 10 minutos que pertenece a `Reserva`), **Indirección del reloj** (invocaba `LocalDateTime.now()` fuera de `RelojDelSistema`) y aislamiento de controladores JavaFX sin reglas de negocio.
- **Cómo se detectó y corrigió**: al ejecutar `mvn clean test "-Dtest=PantallasTest"`, fallaron `losControladoresNoTienenReglasDelNegocio` y `losControladoresNoConocenLaPersistenciaNiElReloj`, además de la revisión del `PR #9` (`@Dslegacy07` marcó *Request changes*). Se subió el commit de corrección `a25a9d3` leyendo directamente `reserva.getVenceEn().format(FORMATO_HORA)`.

---

## Control de calidad final

- `mvn clean test`: **90/90 pruebas en verde** sin modificar ni desactivar ninguna prueba de `src/test/`.
- Cambio limpio de almacenamiento en `PR #10` (`feature/app-ensamblaje-y-h2`): pasando de `RepositorioEventosEnMemoria` (commit `326fb1d`) a `RepositorioEventosH2` (commit `517de68`) modificando **únicamente `App.java`** (Aspecto 6.5).
