# Manifiesto de archivos — yoDibujo

Lista completa de lo que debe existir en el proyecto, con el tamaño de
cada archivo. Sirve para detectar copias incompletas o archivos viejos
que quedaron de una entrega anterior.

## Cómo verificar tu copia

Desde la raíz del proyecto:

```bash
find lib test tool assets -type f | sort
```

Compará esa salida con la lista de abajo. Si te sobra algún archivo que
NO está acá, borralo: es de una entrega anterior y puede romper la
compilación (por ejemplo, si importa algo que ya no existe).

## Archivos eliminados en entregas recientes

Si estos existen en tu copia, **borralos**:

- `lib/ui/paneles/panel_tablas.dart` — reemplazado por
  `lib/ui/paneles/paneles_tablas.dart` (tres paneles separados, sin
  pestañas).

## Carpetas de plataforma

`android/`, `ios/`, `web/`, etc. **no** vienen en el zip: las genera
`flutter create` y viven solo en tu máquina. Si las borraste, se
recuperan con:

```bash
flutter create --platforms=android,ios,web .
```

Después hay que volver a aplicar los permisos de ubicación
(ver `UBICACION_PLATAFORMAS.md`).

## Listado completo

**143 archivos en total.**

| Archivo | Bytes |
|---|---|
| `lib/main.dart` | 2276 |
| `lib/editor/capa.dart` | 3077 |
| `lib/editor/derivador_grupos.dart` | 2434 |
| `lib/editor/editor_controller.dart` | 3530 |
| `lib/editor/grupo_conductores.dart` | 6935 |
| `lib/editor/line_style.dart` | 3045 |
| `lib/editor/poste_derivaciones.dart` | 5087 |
| `lib/editor/secondary_offset.dart` | 5462 |
| `lib/ui/editor/dialogo_conductores.dart` | 13491 |
| `lib/ui/editor/editor_map.dart` | 33246 |
| `lib/ui/editor/panel_poste.dart` | 8390 |
| `lib/ui/editor/simbolo_anclado.dart` | 3229 |
| `lib/ui/editor/troceador_guiones.dart` | 3824 |
| `lib/ui/simbolos/simbolo_miniatura.dart` | 2293 |
| `lib/ui/simbolos/simbolo_painter.dart` | 6199 |
| `lib/ui/cajetin/editor_cajetin.dart` | 19955 |
| `lib/ui/pantallas/arranque_editor.dart` | 4927 |
| `lib/ui/pantallas/pantalla_editor.dart` | 24662 |
| `lib/ui/pantallas/pantalla_impresion.dart` | 23587 |
| `lib/ui/pantallas/vista_plano_previa.dart` | 4052 |
| `lib/ui/paneles/barra_herramienta.dart` | 11437 |
| `lib/ui/paneles/capa_paneles.dart` | 5054 |
| `lib/ui/paneles/panel_capas.dart` | 9681 |
| `lib/ui/paneles/panel_configuracion.dart` | 12659 |
| `lib/ui/paneles/panel_datos_bloqueos.dart` | 8489 |
| `lib/ui/paneles/panel_estructuras.dart` | 16072 |
| `lib/ui/paneles/panel_flotante.dart` | 9553 |
| `lib/ui/paneles/panel_lineas_poste.dart` | 8325 |
| `lib/ui/paneles/paneles_tablas.dart` | 15444 |
| `lib/services/catalog/categorias_catalog.dart` | 2996 |
| `lib/services/catalog/categorias_repository.dart` | 1931 |
| `lib/services/catalog/categorias_sources.dart` | 2386 |
| `lib/services/projects/config_repository.dart` | 1538 |
| `lib/services/projects/nuevo_proyecto_service.dart` | 1945 |
| `lib/services/projects/project_repository.dart` | 5113 |
| `lib/services/auth/auth_service.dart` | 1549 |
| `lib/services/auth/fake_auth_service.dart` | 2483 |
| `lib/services/auth/firebase_auth_service.dart` | 1971 |
| `lib/services/export/export_service.dart` | 3026 |
| `lib/services/export/vp_block_builder.dart` | 6633 |
| `lib/services/kmz/kmz_importer.dart` | 3859 |
| `lib/services/storage/ydib_file.dart` | 1533 |
| `lib/services/ubicacion/geolocator_ubicacion_service.dart` | 3501 |
| `lib/services/ubicacion/ubicacion_service.dart` | 4764 |
| `lib/services/cloud/ydib_file_service.dart` | 2315 |
| `lib/tablas/corte_cuadro.dart` | 3165 |
| `lib/tablas/cuadro_simbologia.dart` | 5074 |
| `lib/tablas/recalculo_tablas.dart` | 6504 |
| `lib/tablas/tablas_vivas.dart` | 1968 |
| `lib/simbolos/anclaje_poste.dart` | 5680 |
| `lib/simbolos/estilo_global_simbolos.dart` | 2118 |
| `lib/simbolos/estilo_simbolo.dart` | 2633 |
| `lib/simbolos/instancia_simbolo.dart` | 4065 |
| `lib/simbolos/resolvedor_estilo.dart` | 1699 |
| `lib/simbolos/simbolo_enee.dart` | 19547 |
| `lib/simbolos/simbolos_catalog.dart` | 5796 |
| `lib/simbolos/sistema_color.dart` | 4772 |
| `lib/simbolos/svg_importer.dart` | 4696 |
| `lib/simbolos/validador_colocacion.dart` | 2014 |
| `lib/export/conversor_estructuras.dart` | 3640 |
| `lib/export/listado_estructuras.dart` | 8953 |
| `lib/geo/geo.dart` | 2143 |
| `lib/geo/utm_converter.dart` | 3809 |
| `lib/cajetin/cajetin.dart` | 10749 |
| `lib/cajetin/cajetin_ops.dart` | 4985 |
| `lib/cajetin/plantilla_cajetin.dart` | 2610 |
| `lib/cajetin/resolvedor_cajetin.dart` | 2142 |
| `lib/models/config_usuario.dart` | 1828 |
| `lib/models/datos_proyecto.dart` | 2777 |
| `lib/models/estructura_montada.dart` | 2681 |
| `lib/models/estructura_seleccionada_model.dart` | 1703 |
| `lib/models/line_model.dart` | 20602 |
| `lib/models/placeholders.dart` | 2920 |
| `lib/models/poste.dart` | 5899 |
| `lib/models/proyecto.dart` | 12779 |
| `lib/models/vep_block.dart` | 7883 |
| `lib/paneles/configuracion_dibujo.dart` | 5944 |
| `lib/paneles/estado_panel.dart` | 4120 |
| `lib/paneles/gestor_paneles.dart` | 3278 |
| `lib/paneles/panel_layout_store.dart` | 2463 |
| `lib/paneles/pestanas.dart` | 2567 |
| `lib/controller/editor_state.dart` | 9193 |
| `lib/controller/historial_comandos.dart` | 1999 |
| `lib/controller/proyecto_controller.dart` | 12730 |
| `lib/controller/commands/capa_commands.dart` | 3531 |
| `lib/controller/commands/command.dart` | 1309 |
| `lib/controller/commands/instancia_commands.dart` | 3113 |
| `lib/controller/commands/layout_commands.dart` | 5644 |
| `lib/controller/commands/linea_commands.dart` | 5910 |
| `lib/controller/commands/poste_commands.dart` | 5003 |
| `lib/core/estado_elemento.dart` | 2394 |
| `lib/core/tipos_base.dart` | 2893 |
| `lib/impresion/documento_impresion.dart` | 3877 |
| `lib/impresion/layout_impresion.dart` | 6495 |
| `lib/impresion/layouts_ops.dart` | 2497 |
| `lib/impresion/pdf_renderer.dart` | 8754 |
| `test/bloque1_test.dart` | 14407 |
| `test/bloque2_luminarias_test.dart` | 4786 |
| `test/bloque2_test.dart` | 16347 |
| `test/bloque3_estado_por_clave_test.dart` | 5445 |
| `test/bloque3_test.dart` | 12238 |
| `test/bloque4a_config_rendimiento_test.dart` | 6542 |
| `test/bloque4a_test.dart` | 8349 |
| `test/bloque4b_conteo_test.dart` | 6415 |
| `test/bloque4b_test.dart` | 6929 |
| `test/ubicacion_test.dart` | 3529 |
| `tool/check_tipos.py` | 19877 |
| `tool/verify_anclaje_lineas.py` | 7237 |
| `tool/verify_arrastre_conductores.py` | 7997 |
| `tool/verify_arrastre_mapa.py` | 6490 |
| `tool/verify_bloque1.py` | 12290 |
| `tool/verify_bloque1_correcciones.py` | 4164 |
| `tool/verify_bloque2.py` | 13612 |
| `tool/verify_bloque2_luminarias.py` | 6820 |
| `tool/verify_bloque3.py` | 11367 |
| `tool/verify_bloque3_estado_por_clave.py` | 5283 |
| `tool/verify_bloque4a_config_rendimiento.py` | 9825 |
| `tool/verify_bloque4a_integracion.py` | 4885 |
| `tool/verify_bloque4a_m1.py` | 10750 |
| `tool/verify_bloque4a_m2.py` | 7324 |
| `tool/verify_bloque4a_m3.py` | 5594 |
| `tool/verify_bloque4a_m4.py` | 4809 |
| `tool/verify_bloque4b_m1.py` | 8026 |
| `tool/verify_bloque4b_m2.py` | 6626 |
| `tool/verify_bloque4b_m3.py` | 10513 |
| `tool/verify_bloque4b_m4.py` | 4724 |
| `tool/verify_bloque4b_m5.py` | 5236 |
| `tool/verify_dibujo_simbolos.py` | 9786 |
| `tool/verify_editor_conductores.py` | 6583 |
| `tool/verify_etiquetas_distancias.py` | 7816 |
| `tool/verify_herramientas_bloqueos.py` | 6624 |
| `tool/verify_instancias.py` | 8306 |
| `tool/verify_lineas_dibujo.py` | 6626 |
| `tool/verify_seleccion_utm.py` | 8646 |
| `tool/verify_simbologia_estructuras.py` | 8539 |
| `tool/verify_trazado_secundario.py` | 6419 |
| `tool/verify_ubicacion.py` | 6030 |
| `assets/catalog/categorias.json` | 5143 |
| `assets/simbolos/simbolos_enee.json` | 120272 |
| `pubspec.yaml` | 1116 |
| `analysis_options.yaml` | 197 |
| `README_BLOQUE1.md` | 27152 |
| `UBICACION_PLATAFORMAS.md` | 3453 |
