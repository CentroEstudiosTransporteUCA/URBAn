# TraTrac — Especificación de UI del cliente Tauri

Editor de configuración para generar un `run.toml` válido para `tratrac` (el paso de
percepción de TraTrac) y lanzarlo.

La fuente de verdad del esquema es el docstring de módulo de `application/config.py`
(`RunConfig.resolve`) en el repositorio **TraTrac** (`src/tratrac/application/config.py`) y la
plantilla `tratrac.example.toml`. Esta UI es un **editor fiel** de ese esquema: el paquete **no
tiene defaults ocultos** — toda clave es obligatoria — así que la UI debe (a) precargar valores
sensatos y (b) reflejar exactamente las validaciones del resolver.

---

## Alcance: solo el config de `tratrac`, no el pipeline completo

Esta pantalla cubre **únicamente** el `RunConfig` de `tratrac` — el paso de percepción
(detección + tracking, escribe el record Parquet). No cubre:

- **`tratrac-preprocess`** — resuelve la escala GSD y el ego-motion, y escribe el archivo de
  transforms compartido que `tratrac` necesita (`input.transforms_in` abajo). Es un
  **prerequisito obligatorio** de todo run, incluso con cámara estática — ver "El campo
  `transforms_in`" más abajo. Conducir este paso desde la UI (elegir método de calibración,
  cargar `background_zones.json` para ego-motion) es una pantalla propia, aún no diseñada.
- **`tratrac-postprocess`** — filtrado, proyección de mundo, y suavizado; produce el `.trj`. Ver
  "Pos-proceso" al final.

## Principios de diseño (leer antes de implementar)

1. **La UI es la proveedora de defaults que el paquete deliberadamente no tiene.**
   Precargar cada campo con los valores de `tratrac.example.toml`. El operador edita
   pocos campos, no todos. El archivo escrito sigue siendo completo y explícito, así que la
   garantía de reproducibilidad se mantiene.

2. **No reimplementar la validación en JS/Rust — derivará de `resolve()`.**
   Las restricciones inline son solo pistas de UX. La autoridad debe ser
   `tratrac --config <archivo> --check --json` (ya implementado en TraTrac — ver el docstring de
   módulo de `src/tratrac/cli.py` en ese repositorio) que corre `RunConfig.resolve` y emite los
   problemas agregados de `ConfigError` como JSON; Tauri lo invoca por shell. Así
   `application/config.py` sigue siendo la única fuente de verdad.

3. **`--force` es estado de la acción de ejecución, NO un campo del formulario.**
   La política de sobrescritura nunca afecta a las trayectorias, por eso no es clave de
   config. Modelar como checkbox en el botón "Ejecutar" y pasarlo como flag CLI; nunca
   serializarlo al TOML.

4. **Habilitación condicional = guardas de coherencia del resolver.**
   Deshabilitar/colapsar los controles dependientes en vez de permitir escribir una
   contradicción que el run rechazaría.

---

## El campo `transforms_in` (prerequisito, no un formulario propio)

`input.transforms_in` (abajo) espera la ruta a un archivo JSON-Lines ya producido por
`tratrac-preprocess estimate` (y, opcionalmente, `project`) — **no** un valor que este editor
resuelve. Hasta que exista una pantalla dedicada para conducir `tratrac-preprocess`, este campo
es un simple **selector de archivo** apuntando a un `.jsonl` generado por fuera de la UI (por
línea de comandos). Esto es intencional, no un campo a medio implementar: `tratrac` en sí mismo
nunca resuelve escala, ego-motion, ni homografía — siempre lee este archivo. Ya no existen
secciones `[calibration]`/`[ego_motion]` en el config de `tratrac` (ver "Campos por sección" —
fueron removidas, no renombradas, cuando ese cálculo se movió a `tratrac-preprocess`).

## Campos por sección

### `[input]`
| Clave | Widget | Restricción |
| --- | --- | --- |
| `video` | selector de archivo (mp4…) | debe existir en disco (la CLI lo verifica) |
| `process_fps` | número + toggle "cada frame" | `>= 0`; `0.0` = procesar cada frame |
| `transforms_in` | selector de archivo (`.jsonl`) | debe existir en disco; ver sección anterior |

### `[detector]`
| Clave | Widget | Restricción |
| --- | --- | --- |
| `name` | dropdown | enum: `yolov8_visdrone` (default actual) \| `rt_detr` (inactivo) \| `yolo_obb` (aún no es el default) |
| `checkpoint` | texto (precargar según `name`) | id de repo HF |
| `conf` | slider | `[0, 1]` |
| `filename` | texto | obligatorio para los tres detectores (aunque `rt_detr`/`yolo_obb` lo ignoren) |

### `[runtime]`
| Clave | Widget | Restricción |
| --- | --- | --- |
| `device` | segmented + spinner de índice | `cpu` \| `mps` \| `cuda[:N]` |

### `[tracker]`
| Clave | Widget | Restricción |
| --- | --- | --- |
| `det_thresh` | slider | `[0, 1]` (típicamente por debajo de `detector.conf`) |

### `[export]`
| Clave | Widget | Restricción |
| --- | --- | --- |
| `out` | selector de guardado (`.parquet`) | no vacío; salida primaria del run — el record Parquet, **no** un `.trj` |

### `[window]`
| Clave | Widget | Restricción |
| --- | --- | --- |
| `start` | input de timecode | `SS` / `MM:SS` / `HH:MM:SS`; "" = inicio del clip |
| `end` | input de timecode | "" = fin del clip; `end > start` |

### `[run]`
| Clave | Widget | Restricción |
| --- | --- | --- |
| `timing_csv` | path opcional, "" = off | CSV de timings por frame |

`force` **no** es una clave de config en ninguna sección — ver Principio 3.

---

## Acción de ejecución (fuera del TOML)

- **Botón "Ejecutar"** → escribe el TOML y lanza `tratrac --config <archivo>`.
- **Checkbox "Sobrescribir salidas"** → añade `--force` al comando (no va al TOML).
- Mostrar stdout/stderr del proceso (la CLI ya reporta progreso a stderr).

## Pos-proceso (siguiente pantalla, opcional)

El run es solo percepción: produce el record Parquet, no un `.trj`. Un `.trj` se obtiene
con `tratrac-postprocess RECORD --transforms TRANSFORMS.jsonl --out run.trj [...]` (el mismo
archivo de transforms que `input.transforms_in` arriba). Si la UI lo cubre, sus inputs serían
otra pantalla (filtros de exclusión, proyección de mundo, parámetros de suavizado) — fuera del
alcance de este editor de config.
