# Infraestructura del servidor `galatzo`

## Alcance y fuentes

Este fichero registra el entorno y las incidencias operativas del Piloto-0. No fija la receta experimental, no rellena valores finales y no autoriza cambios de código. Fuentes primarias recibidas:

- **Piloto-0 v1 / OOM:** `C:\Users\alex\Downloads\faseA_servidor_piloto0_oom_20260803_154958.tar.gz`; sidecar homónimo; SHA-256 esperado/calculado `6e9fc8d88595f8219f5a60f16274c281e3c4ab077e909d86c304eec88e3e0d25`; 19 entradas regulares o directorios, sin rutas absolutas ni componentes `..`.
- **Piloto-0 v2:** `C:\Users\alex\Downloads\piloto0_v2_20260804_004240.tar.gz`; sidecar homónimo; SHA-256 esperado/calculado `c25a5d8431dd5b863ddfd62271ed829f4922fbfae7c7283f174442e6fb3b83f3`; 52 entradas, 37 ficheros, sin rutas absolutas, componentes `..` ni tipos especiales.

El paquete v1 contiene una copia byte a byte del paquete previo `auditoria_dumps_e1_e20.tar.gz`, SHA-256 `7254a9033d174398b8e9ef2a35ba1951e3ece32fa0e0a48a51bb49d358fa19c8`. Ambos paquetes se validaron y extrajeron fuera de TFG; los artefactos se conservan en los tar adjuntos y no se copiaron al repositorio.

## Dependencias del overlay privado

La ejecución usó `PYTHONPATH=/home/amf380/.local/mammoth-pydeps` (`logs/piloto0/environment_before_openclip.txt:26-29` dentro del paquete).

| estado | evidencia observada |
|---|---|
| antes | `ftfy` y `open_clip` no estaban disponibles; `environment_before_openclip.txt:55-58`. |
| después | `open-clip-torch==2.32.0` y `ftfy==6.3.1` se importaron desde el overlay; `environment_after_openclip.txt:5-6`. |
| comprobación | `pip check` devolvió `No broken requirements found`; `environment_after_openclip.txt:13`. Esto comprueba el entorno visible, no la equivalencia de los manifiestos. |

Existe una discrepancia declarativa en Mammoth@`cda7f236`: `requirements.txt:9` incluye `ftfy` sin versión y no contiene `open-clip-torch`, mientras `pyproject.toml:27,33` exige `ftfy>=6.3.1` y `open-clip-torch>=2.32.0`. Además, `requirements.txt:3` no fija torchvision, `pyproject.toml:42` pide `torchvision>=0.21.0` y el servidor usó `0.17.1` (`environment_after_openclip.txt:8`). No se modificó ninguno de esos ficheros.

## Piloto-0 v1: OOM de L2P

Ejecución documentada sobre `galatzo`, rama `tfg-auditoria`, Mammoth `cda7f23681f7bffacee460d99e990bc803bccf04`, Python 3.10.12, Torch 2.2.1+cu121 y RTX 3060 (`logs/piloto0/diagnostico_oom_20260803_154949.txt:2-15`).

- El lanzador resolvió `model_config=best`, batch 128, LR nominal 0.03 y `n_epochs=1`; `driver.log:24-35,83-108`.
- L2P falló en la tarea 1 durante el forward; `diagnostico_oom_20260803_154949.txt:207-263`.
- PyTorch intentó reservar 334 MiB. Registró 11,62 GiB de capacidad, 135,88 MiB libres, 10,79 GiB usados por el proceso, 10,46 GiB asignados por PyTorch y 201,16 MiB reservados sin asignar; línea 263 del mismo fichero.
- El proceso devolvió código 1 y no produjo `logs.pyd`; `diagnostico_oom_20260803_154949.txt:117-123`. El fallo llegó a los 17,21 s; líneas 138-162.
- Había dos kernels ajenos con 304 y 332 MiB; líneas 31-45. El paquete no demuestra si liberarlos o usar `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True` habría hecho caber batch 128.
- El lanzador v1 encadenaba L2P y DualPrompt y salía ante el primer fallo (`run_piloto0.sh:43-49,60-61`); DualPrompt no se ejecutó. CODA-Prompt no figuraba en ese lanzador.

El paquete no contiene una corrida a batch 64. Que batch 64 cabe es un hecho previo registrado en `PLAN_MAESTRO.md §10 (hecho duro)`; no se presenta aquí como si lo demostrara el tar. El cierre de batch 128 combina ese hecho previo, el OOM v1 reproducido y la decisión de uniformidad de D31 (`PLAN_MAESTRO.md §13 (D31)`). La premisa «CODA-Prompt es el método de mayor consumo» pertenece a esa decisión de diseño; el paquete v1 no contiene una medición CODA.

## CODA-Prompt a una época y volcado doble

Los JSON del paquete no entrenan: terminan tras `main.check_args` y marcan las notas runtime como `NOT_APPLIED_BY_THIS_SCRIPT` (`auditoria_dumps/cifar100_n_epochs_{1,20}.json:1042-1049,956`). Su evidencia es estática:

- para L2P y DualPrompt solo cambia `n_epochs: 1→20`;
- para CODA-Prompt cambia también `custom_scheduler.K: 1→20`;
- E=1 registra `K=n_epochs=1`, restricción `K>1`, `runtime_valid=false` y la consecuencia prevista antes de la primera época; `cifar100_n_epochs_1.json:927-951`;
- E=20 registra `K=20` y `runtime_valid=true`; `cifar100_n_epochs_20.json:927-951`.

La ruta ejecutable está en el código leído: `CodaPrompt.begin_task` crea `CosineSchedule(..., K=n_epochs)` (`models/coda_prompt.py:65-73`) y el constructor exige `K>1` (`utils/schedulers.py:38-46`). Por tanto, el bloqueo de una corrida real E=1 está confirmado por la combinación de volcado y código, no por un traceback CODA incluido en este paquete.

## Higiene prevista para el Piloto-0 v2 (registro anterior a la ejecución)

- Los lanzadores v2 son independientes y usan micro-batch real 64; se documentan en `auditoria/piloto0.md`.
- Antes de medir tiempos debe comprobarse que no quedan procesos de cómputo ajenos en la GPU. La limpieza requiere coordinación con sus propietarios; no se incluyen órdenes de terminación.
- El reintento de batch 128 con GPU limpia y `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True` queda conservado solo como diagnóstico histórico en `auditoria/piloto0.md`: D31/D34 lo retiran como test decisorio y batch 128 permanece cerrado.

## Piloto-0 v2: ejecución y recursos observados

La ejecución final sobre `galatzo`, Mammoth `cda7f23681f7bffacee460d99e990bc803bccf04`, fue secuencial. `driver_all.log:2-4,7,254,256,503,505,745,747-748` registra código 0 por método y global; `resumen_final.txt:1-8` lo corrobora. Cada corrida llegó a su marcador de tarea 10 y generó una línea Class-IL y otra Task-IL bajo `data/results_piloto0_v2/`; localizadores completos en `auditoria/piloto0.md`, «Cierre real».

| método | configuración diagnóstica | Class-IL / Task-IL final | wall-clock | RSS CPU máximo | pico GPU registrado |
|---|---|---:|---:|---:|---:|
| L2P | `best`, b64, E1 | 81,67% / 96,83% | 30:35,89 | 2.660.756 kB | 6.099,35 MiB |
| DualPrompt | `best`, b64, E1 | 83,26% / 96,96% | 28:20,38 | 2.628.496 kB | 6.398,29 MiB |
| CODA-Prompt | `best`, b64, E2 | 83,42% / 97,63% | 54:26,92 | 2.547.640 kB | 8.705,77 MiB |

Los tiempos, RSS y códigos proceden de cada `logs/piloto0_v2/<método>...time.txt:5,10,23`; el RSS es memoria residente de CPU, no VRAM. El driver cubre 22:40:30–00:33:54, **DERIVADO 1:53:24**. CODA ejecutó dos épocas por tarea y los otros métodos una; debido al coste fijo de evaluación, el tiempo CODA no se divide entre dos como estimación automática por época.

### Límites de la medición de GPU

El preflight de las 22:17:24 —anterior a la tentativa interrumpida— registró una RTX 3060 de 12.288 MiB con 715 MiB usados y dos kernels Jupyter ajenos de 304 y 332 MiB (`gpu_preflight.txt:1,12-14,22-29`). La evidencia inmediata es que los tres `*.gpu_before.txt:1-2` vuelven a mostrar esos kernels justo antes de cada método válido: la GPU no estaba limpia. Las corridas completas demuestran que batch real64 cupo aun en ese estado, pero no son mediciones sobre una tarjeta libre.

En los tres logs falla la consulta `pynvml` (L2P `:116`, DualPrompt `:121`, CODA `:104`). Mammoth usa entonces `torch.cuda.max_memory_allocated` (`utils/conf.py:53-91`) y lo divide entre 1024² (`utils/stats.py:31-36`). Los «picos GPU» de la tabla son picos *allocated* del proceso PyTorch; excluyen memoria reservada y procesos ajenos y no equivalen al pico total de VRAM. Como solo se tomó `nvidia-smi` antes de cada corrida, ese pico total queda `NO_DISPONIBLE` en el paquete.

### Incidencias de ejecución y resumen

- `metricas_resumen.tsv:2-4` trunca los minutos del campo `elapsed`: muestra `35.89`, `20.38` y `26.92`. Los tiempos autoritativos son los `*.time.txt:5` de la tabla. El resto de campos comprobados del TSV coincide con logs/resultados.
- Antes del driver válido hubo una tentativa L2P aislada en `interrumpidos/l2p_20260803_223543/`: recibió `SIGINT`, guardó un checkpoint tras la tarea 5 y se detuvo al entrar en la 7 (`*.run.log:190-211`); su fichero de tiempo está vacío. `interrumpidos/l2p_20260803_224019/` está vacío. No se mezclan con los resultados finales.
- `driver_all.log` y `driver_nohup.log` son el mismo contenido, SHA-256 `ca611376fe48219d1e6b0730f3595eda40cd126fb6bc07b8ad2143403ae68859`, y no cuentan como evidencias independientes.
- `/usr/bin/time` registra 162.248 cambios de contexto involuntarios para L2P, frente a 10.619 para DualPrompt y 16.356 para CODA (`*.time.txt:15`). La causa no es determinable con el paquete; los wall-clock se consideran observaciones del piloto, no un benchmark limpio.
- Los lanzadores usan `MAMMOTH_TEST=1`: en este SHA solo evita cargar `.env` (`main.py:58-62`) y desactiva W&B (`utils/training.py:48-49`). Las diez tareas y resultados finales demuestran que no acortó el recorrido.

Conclusión limitada: batch real64 completó las diez tareas de los tres métodos bajo `model_config=best` diagnóstico; CODA fue el mayor consumidor según el pico *allocated* comparable dentro de esta ejecución. D31 mantiene batch128 cerrado. Estos datos no fijan batch64 como valor final, no validan `best` como receta y no resuelven D33 ni CODA a E=1.
