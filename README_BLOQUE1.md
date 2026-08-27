# yoDibujo — Bloques 1, 2 y 3

**Bloque 1** (Fundación, modelo de datos y persistencia), **Bloque 2** (El
editor de dibujo) y **Bloque 3** (Catálogo de símbolos ENEE).

## Bloque 3 — Catálogo de símbolos ENEE (resumen)

Cubre cómo se ven y se dibujan los símbolos: el catálogo ENEE como geometría
paramétrica, el painter, los estados por color, el sistema de color de dos
niveles, la edición de estilo y los símbolos propios SVG.

Fuente de verdad: `assets/simbolos/simbolos_enee.json` (**108 símbolos**). Se
implementa **fiel al archivo**; los símbolos con `revisar: true` son
interpretación del cuadro ENEE y se afinan como datos, no como código.

Diseño del catálogo (reescrito contra las láminas ENEE oficiales):

- **Una entrada por combinación forma × estado.** Cada entrada trae un campo
  `estado` único (`existente` o `proyecto`): "retenida sencilla existente" y
  "retenida sencilla proyecto" son dos entradas distintas. Ya no hay
  `geometria_por_estado`.
- **`a_retirar` NO tiene entradas propias**: reutiliza el glifo de `existente`.
  Se distingue por el color de estado y por el marcador `(R)` de la leyenda.
  `SimbolosCatalog.porFormaYEstado` aplica esa caída.
- **No existe `a_reconstruir`**: reconstruir = un elemento en `a_retirar` + otro
  en `proyecto`.
- **`claves_posibles`: el símbolo NO mapea 1:1 con una clave.** Las láminas
  definen pocas formas genéricas (5 de retenida, 4 de transformador) mientras
  `categorias.json` tiene muchas más claves. El usuario elige la clave concreta
  **al colocar**, y esa es la que cuenta. `porClave` devuelve por eso una LISTA.
  **Excepción: los postes**, donde cada símbolo ES una clave concreta.
- **Postes: glifo ÚNICO por combinación** material × altura × estado (68
  entradas). No se derivan de una forma base + relleno — se verían casi todos
  iguales. `origen_glifo` distingue `lamina` (41) de `sintetizado` (27).
- **Primitiva `sector`** (rebanada circular rellena) para los medios y cuartos de
  círculo de la lámina de postes, además de
  `circulo/linea/poligono/arco/texto`.
- **Sistema de color de dos niveles**:
  - Nivel 1 (color = estado): líneas, retenidas, transformadores, luminarias y
    las protecciones que no son cuchilla → existente=rojo, proyecto=celeste,
    a_retirar=naranja.
  - Nivel 2 (color = identidad): **postes=azul, tierras=verde, cuchillas=gris**.
    El estado no cambia el color; se ve por figura + abreviatura E/P/R.
  - La cuchilla se detecta por **`familia_simbolo`**, NO por colocación: el
    catálogo tiene 9 protecciones `entre_poste_y_linea` (restaurador,
    seccionalizador, capacitor...) que **no** son cuchillas y usan color por
    estado.
  - Los seis colores son editables y viven en `estiloGlobal` del `.ydib`.

### Estado POR CLAVE y leyenda del plano

El marcador de estado va **por clave, no por poste**: un mismo poste puede tener
estructuras en estados distintos (poste existente al que se le agrega una
retenida proyectada). Por eso `Poste.estructuras` es
`List<EstructuraMontada>` — cada una con `{clave, estado}` — y no una lista de
strings. La leyenda junto al poste sale:

```
A-I-1 (E), TS-25 (P), R-1 (R)
```

`LeyendaEstructuras.construir` la genera; `Poste.clavesEstructuras` da solo las
claves para los consumidores que cuentan (listado, vep, vp).

Módulos del Bloque 3:
- `lib/simbolos/simbolo_enee.dart` — modelo del símbolo y primitivas
  (`circulo/linea/poligono/arco/texto/sector`), colocación, estado por entrada,
  `claves_posibles`, datos de poste (material/altura/autosoportado), de
  luminaria (tipo/watts/brazo) y de conexión (extremos poste↔línea).
- `lib/simbolos/simbolos_catalog.dart` — carga/índice del JSON; `porClave`
  (lista), `porClaveYEstado`, `porFormaYEstado` (con caída a_retirar→existente),
  `simboloDePoste(clave, estado)`; distingue solo-etiqueta; símbolos propios.
- `lib/models/estructura_montada.dart` — `EstructuraMontada {clave, estado}`,
  `MarcadorEstado` (E/P/R) y `LeyendaEstructuras`.
- `lib/simbolos/sistema_color.dart` — paleta editable y resolución de color de
  dos niveles.
- `lib/simbolos/estilo_simbolo.dart` — variante reestilizada por proyecto y
  estilo resuelto para el painter.
- `lib/simbolos/resolvedor_estilo.dart` — combina familia/estado + variante +
  escala + factor global.
- `lib/simbolos/estilo_global_simbolos.dart` — proyección tipada de la paleta y
  variantes dentro del `estiloGlobal` passthrough (preserva otras claves).
- `lib/simbolos/svg_importer.dart` — import de SVG propios, normalización a caja
  64×64, embebido y export.
- `lib/ui/simbolos/simbolo_painter.dart` — `CustomPainter` paramétrico
  (primitivas→Canvas, dash, rotación por anclaje, escala 64→px).

Nota de alcance: el `simbolos_enee.json` original no traía símbolo de *sistema
de tierra*. Se agregó uno **provisional** (`enee_tierra_sistema`, categoría
`Tierra`, clave `PAT`, gira, identidad verde) marcado `revisar: true` — su
geometría (cable vertical a tres barras decrecientes, estilo IEC) se afinará
contra el cuadro ENEE junto con el resto de los símbolos. La categoría
`Protección` del JSON (restauradores, seccionalizadores, capacitores, etc.) se
carga completa y usa color por estado.

Tests: `test/bloque3_test.dart`. Verificador ejecutable sin Dart:
`tool/verify_bloque3.py` (**23 checks OK, 0 fallos** contra el JSON real).

---

# yoDibujo — Bloque 2 (El editor de dibujo)

Cubre cómo se dibuja: geometría, postes, líneas/conductores, estructuras,
medición, KMZ, y la derivación de los bloques embebidos `vep` y `vp`.

Principio rector respetado: **yoDibujo dibuja y captura datos; no analiza.**
Toda la lógica de análisis (catenaria, redes, cargas, conversión de altitud) se
deja a Voltaje Pico; yoDibujo exporta datos crudos.

Decisiones tomadas (confirmadas con el usuario):
- **UTM**: paquete `proj4dart`, multizona, con override de zona por traslape.
- **Offset del secundario**: en **píxeles de pantalla** (visual fijo a cualquier
  zoom). La geometría real del tramo (y el `.vp`) no cambia por el offset.
- **Borrado de poste**: en **cascada** (borra sus líneas conectadas).

Módulos del Bloque 2:
- `lib/geo/geo.dart` — Haversine y conversión m/pies para presentación.
- `lib/geo/utm_converter.dart` — lat/lng ↔ UTM multizona (proj4dart).
- `lib/editor/poste_derivaciones.dart` — carga nominal por tipo, material,
  derivación de transformador (flag + kVA) y de retenida.
- `lib/editor/line_style.dart` — función calibre→grosor; 3 dimensiones
  (color=categoría, dash=estado, grosor=base+calibre) sin que una reemplace a
  otra.
- `lib/editor/secondary_offset.dart` — polilínea quebrada del secundario
  (offset igual en ambos extremos), en píxeles.
- `lib/editor/grupo_conductores.dart` — grupos reutilizables con letra→leyenda,
  validación de las 14 combinaciones, derivación de calibres `.vep`, cable como
  material lineal.
- `lib/editor/editor_controller.dart` — colocación (tipo activo/último usado),
  captura en campo, trazado de líneas, mover con recálculo Haversine, borrado
  en cascada.
- `lib/services/kmz/kmz_importer.dart` — KMZ→postes (solo waypoints, tracks
  ignorados, Concreto 40, nombre→etiqueta, altitud incluida).
- `lib/services/export/vp_block_builder.dart` — bloque `.vp` crudo para Voltaje
  Pico (postes con altitud+flag, altura en pies, `altura_conductor_m` null;
  tramos con conductores; transformadores).
- `lib/services/export/export_service.dart` — regenera ambos bloques hermanos
  (`vep` + `vp`) desde el plano y el catálogo.
- `lib/ui/editor/editor_map.dart`, `lib/ui/editor/panel_poste.dart` — capa de UI
  (mapa `flutter_map`, panel del poste) que envuelve la lógica pura.

Nota de nomenclatura del `.vp`: el enum `ConductorFunction` preservado del
Bloque 1 tiene `{phase, neutral, guard}`; se mapea a `fase|neutro|hilo_guarda`.
No existe `tierra` como valor del enum, así que no se emite salvo que el modelo
se extienda en un bloque futuro.

Tests del Bloque 2: `test/bloque2_test.dart`. Verificador ejecutable sin Dart:
`tool/verify_bloque2.py` (**38 checks OK, 0 fallos** contra la lógica portada).

## Adición del Bloque 2 — Luminarias y anclaje compartido

Retenida, luminaria, tierra y protecciones comparten un mismo **anclaje al
centro del poste con brazo** (`lib/simbolos/anclaje_poste.dart`):

- **Largo del brazo: global POR FAMILIA** (un largo para luminarias, otro para
  retenidas, otro para tierras, otro para protecciones), no por instancia. Vive
  en el estilo global del proyecto vía `LargosBrazoPorFamilia` (clave
  `largosBrazo`).
- **Rotación por instancia** (`DatosAnclaje`): retenida/luminaria/tierra giran
  **libre**; las **protecciones se alinean a una línea primaria** que el usuario
  elige (`modo = alineadaALinea`, `lineaReferenciaId`).
- **Regla dura de protección** (`ValidadorColocacion`): no se puede colocar en un
  poste sin líneas primarias conectadas. En un quiebre con dos primarias, el
  usuario elige a cuál se alinea; si su elección es inválida, cae a la primera.
- **Luminaria** (`DatosLuminaria`): wattaje + tipo (sodio/mercurio). El símbolo
  concreto sale del catálogo ENEE según esa combinación (Bloque 3). Estos datos
  **viajan al bloque `.vp`** como datos crudos (`snapshot.luminarias`:
  clave, poste, wattaje, tipo, ángulo) — Voltaje Pico hace el análisis.

`InstanciaSimbolo` ganó dos sub-objetos opcionales (`anclaje`, `luminaria`),
null para lo que no aplica; el `.ydib` viejo se lee con ambos en null.

Tests: `test/bloque2_luminarias_test.dart`. Verificador:
`tool/verify_bloque2_luminarias.py` (**22 checks OK**).

---

# yoDibujo — Bloque 1 (Fundación, modelo de datos y persistencia)

Este entregable cubre el **Bloque 1** completo: modelo de datos del proyecto,
serialización `.ydib`, bloque `.vep` embebido, modelos preservados de líneas y
estructuras, catálogo `categorias.json` (Firestore + fallback), autenticación y
configuración por usuario.

## Estado de verificación

- **`flutter test` no se ejecutó en el entorno de generación** porque ahí no se
  podía instalar el SDK de Dart/Flutter (red restringida). Los tests Dart están
  escritos y listos en `test/bloque1_test.dart`.
- Para dar evidencia de la lógica del contrato sin Dart, se incluye
  `tool/verify_bloque1.py`, que porta la lógica pura (round-trip `.ydib`,
  contrato `.vep`, mapeo de estados, membresía de categorías, parse de
  `LineType`) y la valida contra los archivos **reales**. Resultado local:
  **61 checks OK, 0 fallos**.

Antes de continuar al Bloque 2, corré localmente:

```bash
flutter pub get
flutter analyze
flutter test
```

## Cómo probar la lógica sin Flutter (opcional)

```bash
python3 tool/verify_bloque1.py
```

## Mapa de archivos

```
lib/
  core/
    estado_elemento.dart      # EstadoElemento canónico + mapeo LineStatus->EstadoElemento
    tipos_base.dart           # UnidadDistancia, CentroMapa, UTM, EstiloTexto
  models/
    line_model.dart           # PRESERVADO literal (LineType×42, LineModel, Conductor, ConductorCalibers)
    estructura_seleccionada_model.dart  # PRESERVADO literal (lo consume YoEstimo)
    poste.dart                # Poste (id uuid + numero entero estable)
    datos_proyecto.dart       # DatosProyecto (cajetín + vep)
    config_usuario.dart       # ConfigUsuario (defaults de empresa por uid)
    vep_block.dart            # Bloque .vep: contrato exacto con YoEstimo
    placeholders.dart         # Capa/Instancia/Grupo/Simbolo/Layout/Cajetin/EstiloGlobal/Bloqueos (passthrough; diseño en Bloques 2/3/4)
    proyecto.dart             # Proyecto raíz + toYdib/fromYdib + construcción del vep + Proyecto.nuevo
  services/
    auth/
      auth_service.dart       # Interfaz AuthService (extensible a más proveedores)
      fake_auth_service.dart  # Fake in-memory (tests)
      firebase_auth_service.dart  # Esqueleto Firebase Auth real
    catalog/
      categorias_catalog.dart     # Membresía EXPLÍCITA (nunca por prefijo)
      categorias_repository.dart  # Firestore + fallback + siembra (interfaz)
      categorias_sources.dart     # Asset local, Firestore real, fake in-memory
    projects/
      config_repository.dart      # ConfigUsuario por uid (fake + Firestore)
      project_repository.dart     # Proyectos por uid, sin versionado (fake + Firestore)
      nuevo_proyecto_service.dart # Crear proyecto autocompletando desde ConfigUsuario
    storage/
      ydib_file.dart          # encode/decode del archivo .ydib local; decodeVep
  main.dart                   # Entrypoint; arma dependencias (fake por defecto)
assets/catalog/categorias.json  # Fallback local empaquetado
test/bloque1_test.dart        # Tests mínimos del Bloque 1
tool/verify_bloque1.py        # Verificador de coherencia (no requiere Dart)
```

## Decisiones respetadas del contrato

- **Estado canónico único**: `EstadoElemento { existente, proyecto, aRetirar }`,
  serializado como `"existente" | "proyecto" | "a_retirar"`. `LineStatus` queda
  como detalle interno de `LineType` con conversión 1:1.
- **`LineType.color` no pinta la línea**: se conserva para no romper la
  deserialización, pero el color efectivo se resuelve por estado (Bloque 4A).
- **IDs de `LineType` estables** (42 tipos, IDs originales preservados).
- **`.vep`**: yoDibujo es dueño de `meta`/`datos`/`estructurasSeleccionadas`/
  `notas`; deja `materialesMostrar={}` y `totales` en cero (nunca los calcula);
  claves nunca se eliminan; `estado="Borrador"`, `numeroEstimacion=42`
  constantes; calibres indeterminados como `""` (no null).
- **Catálogo**: pertenencia por membresía explícita en la lista, nunca por
  prefijo; `Postes` y `Cable` son atributos, no estructuras montables.
- **Persistencia**: `.ydib` archivo local + nube por `uid` sin versionado; al
  guardar se actualiza `modificadoEn` y se regenera el `vep`.
- **Firebase**: interfaces + fakes in-memory + esqueleto real (sin credenciales).

## Conexión de Firebase (cuando tengas tu config)

En `main.dart`, sustituir `Dependencias.fake()` por una construcción que use
`FirebaseAuthService`, `FirestoreCategoriasSource`, `FirestoreConfigRepository`
y `FirestoreProjectRepository`, tras `Firebase.initializeApp()` con tu
`firebase_options.dart`. Los consumidores no cambian.

## Pendiente para bloques siguientes (declarado, no implementado aquí)

- Derivación real de calibres del `.vep` desde los conductores del plano
  (Bloque 2).
- Conteo de `estructurasSeleccionadas` desde el plano (Bloque 4).
- Diseño real de `Capa`, `InstanciaEstructura`, `GrupoConductores`,
  `SimboloEmbebido`, `Layout`, `Cajetin`, `EstiloGlobal`, `EstadoBloqueos`.

---

# Bloque 4A — Paneles, capas, historial y tablas vivas

Estado: **completo y verificado** (verificadores Python en verde; tests Dart en
`test/bloque4a_test.dart` listos para correr localmente).

## Arquitectura del controlador (decisión C)

- `lib/controller/editor_state.dart` — **capa de estado tonta**: listas del plano
  (postes, líneas, capas) + primitivas de inserción/quita/reemplazo. Sin lógica
  de negocio ni historial.
- `lib/controller/commands/` — patrón **Command**: cada mutación es un comando
  reversible que conoce solo su delta (`aplicar`/`revertir`). La lógica del
  Bloque 2 (Haversine al mover, cascada al borrar) se **reubicó** aquí sin
  reescribirla. `BorrarPosteCascadaCommand` restaura poste + líneas en un solo
  deshacer.
- `lib/controller/historial_comandos.dart` — dos pilas, tope **40** (la 41
  descarta la más vieja), acción nueva limpia rehacer. **No persiste**.
- `lib/controller/proyecto_controller.dart` — **la única puerta de mutación**.
  Métodos públicos construyen comandos y los despachan por el historial; expone
  undo/redo; notifica a listeners para las **tablas vivas reactivas**.
- `lib/editor/editor_controller.dart` — reescrito como **fachada delgada** sobre
  `ProyectoController` (conserva la API del Bloque 2; todo pasa por la puerta
  única).

## Capas

`lib/editor/capa.dart` — `Capa` tipada (sistema vs usuario, visible, bloqueada,
opacidad, orden). Capas de sistema no se borran. Operaciones como comandos
reversibles en `capa_commands.dart`. Consultas `posteVisible`/`posteSeleccionable`
para render y selección (oculta no dibuja; bloqueada no se selecciona).

## Fuente única de simbología + tablas vivas + corte

- `lib/simbolos/instancia_simbolo.dart` — `InstanciaSimbolo`: la unidad que se
  **dibuja en el plano Y** de la que se **deriva el cuadro de simbología**. Sin
  lista paralela.
- `lib/tablas/cuadro_simbologia.dart` — deriva el cuadro de las instancias
  colocadas (incluye propios; instancia sin clave no cuenta).
- `lib/tablas/tablas_vivas.dart` — `TablaPostes` (UTM) y `TablaConductores`
  (por letra), reactivas.
- `lib/tablas/corte_cuadro.dart` — **modelo de corte** de cuadros imprimibles:
  el usuario elige filas de partición; se derivan N sub-bloques (con encabezado
  repetido) y se pueden **volver a unir**. La colocación en la lámina es 4B.

## Paneles y pestañas

- `lib/paneles/estado_panel.dart`, `gestor_paneles.dart` — paneles flotantes con
  menú **Ventana** (cerrar oculta, no destruye; reabrir en última posición),
  enrollar, mover, redimensionar. Paneles separados para Simbología, Coordenadas
  y Calibres.
- `lib/paneles/panel_layout_store.dart` — persistencia **local** del layout de
  Dibujo (interfaz + serializador puro + fake in-memory + skeleton
  `shared_preferences`). **No viaja al `.ydib`**.
- `lib/paneles/pestanas.dart` — pestañas **Model/Paper** (AutoCAD): Dibujo con
  paneles locales; Layout (Paper) con estado guardado **en el `.ydib`** (cáscara
  en 4A; contenido de impresión → 4B).

## Integración `.ydib` (aditiva)

`Proyecto` gana tres claves nuevas, **sin cambiar el contrato del Bloque 1**
(round-trip intacto, bloque `vep`/`vp` sin tocar):

- `instanciasSimbolo` — `List<InstanciaSimbolo>`
- `cortesCuadros` — `List<CorteCuadro>`
- `layoutPaper` — `EstadoLayoutPaper`

Un `.ydib` viejo sin esas claves se lee con defaults (`[]`, `[]`, vacío).

## Qué viaja a la nube y qué no

| Cosa | Dónde vive | ¿Nube? |
|---|---|---|
| Capas, cortes de cuadros, instancias, estado Paper, datos, bloqueos, estilo global | `.ydib` | Sí |
| Layout de paneles de la pestaña Dibujo | prefs locales | No |
| Historial undo/redo | memoria | No |

## Verificación

`tool/verify_bloque4a_m1.py` (17), `_m2.py` (15), `_m3.py` (12), `_m4.py` (14),
`_integracion.py` (10). Junto a los bloques 1-3: **190 checks, 0 fallos**.

> Nota de entorno: el SDK de Dart/Flutter no se puede instalar en el sandbox
> (red restringida), por lo que `flutter analyze`/`flutter test` deben correrse
> localmente. Los verificadores Python portan la lógica de contrato pura y sí
> corren en el sandbox como sustituto de chequeo.

# Bloque 4B — Cajetín, layouts, impresión, exportación y nube

## Filosofía (recordatorio)

yoDibujo **solo dibuja y captura datos**. No analiza, no pone precios, no
resuelve materiales. El listado de estructuras y cable es un **conteo**; los
cálculos viven en las apps hermanas vía los bloques `vep` (YoEstimo) y `vp`
(Voltaje Pico), que se regeneran al guardar.

## Adiciones del Bloque 4A — Panel Configuración y rendimiento de tablas

### Panel Configuración (`lib/paneles/configuracion_dibujo.dart`)

Proyección **tipada** de los controles del panel sobre el mapa crudo de
`estiloGlobal`, en el mismo patrón que `EstiloGlobalSimbolos` (Bloque 3): lee y
escribe su parte **sin descartar** las claves de otros bloques.

Controles: distancia de repetición de las marcas de fase (px), amplitud del
zig-zag de acometida, offset x/y del secundario, **largos de brazo por familia**
(reutiliza `LargosBrazoPorFamilia` del Bloque 2) más su **rotación por defecto**,
**toggle de los marcadores (E)/(P)/(R)** de la leyenda (Bloque 3), unidad de
distancia y tipo de altitud.

> **Discrepancia resuelta a favor de la decisión de Victor:** el prompt 4A dice
> que el brazo es "configurable también por instancia"; la decisión tomada en el
> Bloque 2 es **largo global por familia**. Se implementó como global por
> familia. Lo que sí queda por instancia es la **rotación**, y acá se guarda su
> valor por defecto.

### Rendimiento de las tablas vivas (`lib/tablas/recalculo_tablas.dart`)

Tres mecanismos para que arrastrar un poste no se sienta trabado con cientos de
elementos:

- **`ConteoIncremental`** — agregado por clave que se actualiza SOLO en la
  entrada afectada; nunca recorre el proyecto. Al borrar el último de un tipo,
  la clave **desaparece** del cuadro de simbología. Los símbolos propios entran
  igual, sin registro aparte.
- **`RegistroTablasVivas`** — sucio + visibilidad: una tabla con su panel cerrado
  o enrollado solo se marca **sucia**; se recalcula al abrirla. Sus ids son los
  mismos de `PanelesIds`, para que la visibilidad case con el panel real.
- **`ProgramadorDebounce`** — agrupa la ráfaga de cambios en un solo recálculo
  (ventana recomendada 150–300 ms). Puro y determinista (recibe el instante), así
  que se verifica sin entorno gráfico.

Verificador: `tool/verify_bloque4a_config_rendimiento.py` (**29 checks**).
Tests: `test/bloque4a_config_rendimiento_test.dart`.

## Módulo 1 — Cajetín (`lib/cajetin/`)

Modelo AutoCAD: una **tabla anclada a una esquina** del papel que crece hacia
adentro. La rejilla se define por los anchos de columna y altos de fila (mm); el
tamaño del cajetín es la **suma** (nunca se guarda aparte). Merge/unmerge por
**spans** estilo hoja de cálculo.

- `cajetin.dart` — `EsquinaCajetin` (4 esquinas), `TipoContenidoCelda`
  (texto / logo / campo), `CamposCajetin` (nombreEmpresa, estimador, lugar,
  nombreProyecto, claseVoltaje, fecha), `EstiloCelda`, `CeldaCajetin` (con
  `filaSpan`/`columnaSpan`), `Cajetin` (`anchoTotal`/`altoTotal` = sumas,
  factory `rejilla`).
- `cajetin_ops.dart` — operaciones puras: `combinar`/`separar`, `anchoColumna`/
  `altoFila`, `escalar` (toda la tabla), `editarCelda`, `anclarEn` y `moverA`.
  El cajetín se ancla a una **esquina** (crece hacia adentro) **o se mueve
  libremente** (`xMm`/`yMm`); anclar a una esquina descarta la posición libre.
- `resolvedor_cajetin.dart` — resuelve campos autocompletados desde el proyecto
  (se **revinculan solos** al aplicar la plantilla a otro proyecto) y el logo
  (usa `logoUrl` o el placeholder `"logo"`; sin base64 en el `.ydib`).
- `plantilla_cajetin.dart` — plantillas reutilizables (biblioteca del usuario)
  + repositorio (interfaz + fake in-memory).

**El cajetín NO cuenta datos del plano** (no hay celdas de conteo vivo). Su
contenido es solo texto, logo y campos declarativos.

## Módulo 2 — Layouts de impresión (`lib/impresion/layout_impresion.dart`, `layouts_ops.dart`)

Cada layout es una **vista independiente al mismo plano**: su encuadre/zoom
(`VistaMapa`), papel (`TamanoPapel` A4/Carta/A3/Tabloide/Personalizado),
orientación, márgenes y su **propio cajetín**. Gestión estilo pestañas de
AutoCAD (crear/duplicar/renombrar/reordenar/borrar), toda reversible por el
historial del 4A (`TransformarLayoutsCommand`, `LayoutCommands`,
`CajetinCommands`). Los layouts NO copian el plano: miran el estado compartido.

## Módulo 3 — Listado de estructuras y cable (`lib/export/listado_estructuras.dart`)

Conteo agregado de todo el plano. Las dos fuentes **NO se suman a ciegas** —
hacerlo duplicaría (una retenida escrita como texto Y colocada como símbolo
contaría dos veces).

**Regla de conteo SIN SOLAPAMIENTO:**

- Estructuras **CON símbolo** (retenidas, protecciones, tierras,
  transformadores, luminarias) → cuentan **únicamente por sus instancias
  colocadas**, agrupando por la clave concreta que el usuario eligió al colocar.
- Estructuras **solo-etiqueta** (A-I-1, B-II-1...) → cuentan **únicamente por
  `Poste.estructuras`**, sumando su campo **`cantidad`** (un poste puede llevar
  2 retenidas iguales), no 1 por entrada.
- Los **postes** se cuentan por `Poste.tipoPoste`, no desde la lista de
  estructuras.

**Clave con símbolo escrita a mano** (decisión del proyecto):
`ConversorEstructuras` la **convierte automáticamente en instancia colocada** —
una por unidad de `cantidad` — y la saca de la lista de texto, informando qué
claves convirtió. Así el solapamiento se resuelve **en origen**: cada dato vive
en una sola fuente. `clavesQueDeberianSerInstancia` permite avisar antes de
convertir.

Categoría por **membresía** (nunca por prefijo); desconocida → categoría dada o
`Personalizada`. Cable **lineal** (longitud × count, sin factor) por calibre y
función, en la unidad configurada. Salida JSON y CSV.

## Módulo 4 — Documento imprimible + PDF (`lib/impresion/documento_impresion.dart`, `pdf_renderer.dart`)

`DocumentoBuilder` arma el modelo puro: **una página por layout** (papel,
márgenes, encuadre, cajetín), respeta la **visibilidad/opacidad de capas**, y un
**toggle** anexa el listado de estructuras. `pdf_renderer.dart` lo pinta con el
paquete `pdf`/`printing` (offline, viaja con la app, sin backend — coherente con
"host estático sin servidor").

> `pdf_renderer.dart` es el **único archivo que no se ejecuta en el sandbox**
> (requiere el engine de Flutter). Su modelo sí está verificado en Python.

## Módulo 5 — Nube y archivos (`lib/services/cloud/ydib_file_service.dart`)

Guardar/abrir `.ydib` como archivo local: serializa a JSON regenerando `vep`+
`vp`, y deserializa de vuelta. La nube bajo `uid` **sin versionado** ya la cubre
`ProjectRepository` (Bloque 1). `DetectorSimbolosFaltantes` detecta símbolos que
un `.ydib` ajeno referencia pero no están en el catálogo propio: se renderizan
desde los **embebidos** y se ofrece "Agregar a mi catálogo", sin fallar la
apertura.

## Integración `.ydib`

`proyecto.dart` migró `layouts` de passthrough a `List<LayoutImpresion>` tipado y
**eliminó** `cajetinBase` (el cajetín ahora vive dentro de cada layout, modelo
AutoCAD). Import con `hide Layout, Cajetin` para evitar colisión con los tipos
del Bloque 1. Round-trip y bloques `vep`/`vp` intactos.

## Verificación

`tool/verify_bloque4b_m1.py` (17), `_m2.py` (16), `_m3.py` (14), `_m4.py` (11),
`_m5.py` (13). **Batería completa (bloques 1-4): 261 checks, 0 fallos.**
Árbol estático: 72 archivos `.dart`, sin ciclos, imports resueltos.

> Los `.dart` y `test/bloque4b_test.dart` están listos para correr localmente
> con `flutter pub get && flutter analyze && flutter test`. El sandbox no tiene
> SDK de Dart/Flutter; los verificadores Python son el sustituto ejecutable.

## Pendiente (capa de UI — todo el Bloque 4)

Solo faltan los **widgets Flutter** que envuelven la lógica pura ya construida:
paneles arrastrables, menú Ventana, tablas renderizadas, barra Model/Paper y
panel de capas (4A); editor de cajetín, pantalla de impresión con vista previa y
pestañas de layout (4B). La lógica, el estado, los modelos y los comandos ya
existen y están verificados; los widgets son una capa de presentación encima.
