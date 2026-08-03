# Infraestructura del servidor `galatzo`

## Alcance y fuente

Este fichero registra el entorno y las incidencias operativas del Piloto-0. No fija la receta experimental, no rellena valores finales y no autoriza cambios de código. La fuente primaria recibida es:

- paquete externo: `C:\Users\alex\Downloads\faseA_servidor_piloto0_oom_20260803_154958.tar.gz`;
- SHA-256 esperado y calculado: `6e9fc8d88595f8219f5a60f16274c281e3c4ab077e909d86c304eec88e3e0d25`;
- sidecar: `C:\Users\alex\Downloads\faseA_servidor_piloto0_oom_20260803_154958.tar.gz.sha256`;
- validación del contenido: 19 entradas, todas regulares o directorios, sin rutas absolutas ni componentes `..`.

El paquete contiene una copia byte a byte del paquete previo `auditoria_dumps_e1_e20.tar.gz`, SHA-256 `7254a9033d174398b8e9ef2a35ba1951e3ece32fa0e0a48a51bb49d358fa19c8`. Los artefactos se conservan dentro del tar adjunto; no se copiaron al repositorio.

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

El paquete no contiene una corrida a batch 64. Que batch 64 cabe es un hecho previo registrado en `PLAN_MAESTRO.md:163`; no se presenta aquí como si lo demostrara el tar. El cierre de batch 128 combina ese hecho previo, el OOM v1 reproducido y la decisión de uniformidad de D31 (`PLAN_MAESTRO.md:220`). La premisa «CODA-Prompt es el método de mayor consumo» pertenece a esa decisión de diseño; el paquete v1 no contiene una medición CODA.

## CODA-Prompt a una época y volcado doble

Los JSON del paquete no entrenan: terminan tras `main.check_args` y marcan las notas runtime como `NOT_APPLIED_BY_THIS_SCRIPT` (`auditoria_dumps/cifar100_n_epochs_{1,20}.json:1042-1049,956`). Su evidencia es estática:

- para L2P y DualPrompt solo cambia `n_epochs: 1→20`;
- para CODA-Prompt cambia también `custom_scheduler.K: 1→20`;
- E=1 registra `K=n_epochs=1`, restricción `K>1`, `runtime_valid=false` y la consecuencia prevista antes de la primera época; `cifar100_n_epochs_1.json:927-951`;
- E=20 registra `K=20` y `runtime_valid=true`; `cifar100_n_epochs_20.json:927-951`.

La ruta ejecutable está en el código leído: `CodaPrompt.begin_task` crea `CosineSchedule(..., K=n_epochs)` (`models/coda_prompt.py:65-73`) y el constructor exige `K>1` (`utils/schedulers.py:38-46`). Por tanto, el bloqueo de una corrida real E=1 está confirmado por la combinación de volcado y código, no por un traceback CODA incluido en este paquete.

## Higiene y alcance del siguiente piloto

- Los lanzadores v2 son independientes y usan micro-batch real 64; se documentan en `auditoria/piloto0.md`.
- Antes de medir tiempos debe comprobarse que no quedan procesos de cómputo ajenos en la GPU. La limpieza requiere coordinación con sus propietarios; no se incluyen órdenes de terminación.
- El reintento de batch 128 con GPU limpia y `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True` queda conservado solo como diagnóstico histórico en `auditoria/piloto0.md`: D31/D34 lo retiran como test decisorio y batch 128 permanece cerrado.
