# Piloto-0 v2: CIFAR-100 con lanzadores independientes

## Alcance, decisiones aplicadas y límites

Los comandos están escritos para Linux, desde la raíz de un clon de `https://github.com/alexmf10/mammothV2`, rama `tfg-auditoria`, commit `cda7f23681f7bffacee460d99e990bc803bccf04`. El dataset es `seq-cifar100-224`: 10 tareas, 10 clases por tarea y 100 clases (`datasets/seq_cifar100_224.py:39-43`).

| método | presupuesto del piloto v2 | batch | motivo |
|---|---:|---:|---|
| L2P | E=1 por tarea | 64 | CLI limpia; lanzador independiente. |
| DualPrompt | E=1 por tarea | 64 | CLI limpia; lanzador independiente. |
| CODA-Prompt | E=2 por tarea | 64 | E=1 aborta en `begin_task` porque `CosineSchedule` exige `K=n_epochs>1`. |

E=2 para CODA es una excepción operativa del Piloto-0 v2 decidida en D32: sirve para comprobar el bucle completo y medir su coste total observado. Cualquier derivación de coste por época-tarea deberá separar el coste fijo de evaluación; no se obtiene dividiendo ciegamente el tiempo total entre dos. **No modifica el eje experimental**, cuyo peldaño mínimo sigue pendiente en D33. Batch 64 aplica D31/D32 al piloto; la columna `valor final` de la auditoría sigue vacía.

Los tres lanzadores usan `model_config=best` únicamente como baseline diagnóstico heredado del Piloto-0. La matriz final no debe reutilizar `best/default` en bloque: se construirá desde `configuracion_final` después de la revisión humana (`PLAN_MAESTRO.md §13 (D32) / §8 Fase D`).

## Cierre real del Piloto-0 v2 — 2026-08-04

### Integridad y procedencia de la evidencia

Paquete recibido: `C:\Users\alex\Downloads\piloto0_v2_20260804_004240.tar.gz`; sidecar homónimo `.sha256`. El SHA-256 esperado y el calculado localmente coinciden:

```text
c25a5d8431dd5b863ddfd62271ed829f4922fbfae7c7283f174442e6fb3b83f3
```

Antes de extraerlo fuera de TFG se revisaron sus 52 entradas: 37 son ficheros, no hay rutas absolutas, componentes `..` ni tipos especiales. No se copió el paquete ni su contenido al repositorio. La ejecución declara `galatzo` y el commit exacto `cda7f23681f7bffacee460d99e990bc803bccf04`; `resumen_final.txt:1-3` fue generado a `2026-08-04T00:39:26+02:00`, mientras el fin real del driver consta a las 00:33:54 en `driver_all.log:2-4,747`.

### Resultado comprobado

| método | configuración realmente ejecutada | tareas | salida | resultado final Class-IL / Task-IL | líneas comprobadas |
|---|---|---:|---:|---:|---|
| L2P | `best`, batch 64, E=1, seed 0 | 10/10 | 0 | 81,67% / 96,83% | +1 Class-IL verificada; 1 Task-IL presente |
| DualPrompt | `best`, batch 64, E=1, seed 0 | 10/10 | 0 | 83,26% / 96,96% | +1 Class-IL verificada; 1 Task-IL presente |
| CODA-Prompt | `best`, batch 64, E=2, seed 0 | 10/10 | 0 | 83,42% / 97,63% | +1 Class-IL verificada; 1 Task-IL presente |

- Las diez tareas aparecen en `l2p_seq-cifar100-224_seed0_b64_e1.run.log:138,148,158,168,178,188,198,208,218,228`, `dualprompt_seq-cifar100-224_seed0_b64_e1.run.log:138,148,158,168,178,188,198,208,218,228` y `coda-prompt_seq-cifar100-224_seed0_b64_e2.run.log:121,132,143,154,165,176,187,198,209,220`. Las líneas finales de medias y vectores por tarea están, respectivamente, en `:230-233`, `:230-233` y `:223-226`.
- El driver secuencial registra inicio/fin con código 0 para L2P (`driver_all.log:7,254`), DualPrompt (`:256,503`) y CODA-Prompt (`:505,745`), y termina globalmente con 0 (`:747-748`). Los siete `*.exit_code.txt` —salida propia y salida del driver por método, más la global— contienen `0`; `resumen_final.txt:5-8` los consolida.
- Cada uno de los seis `data/results_piloto0_v2/{class-il,task-il}/seq-cifar100-224/<modelo>/logs.pyd` contiene exactamente una línea terminada en LF; `<modelo>` es `l2p`, `dualprompt` o `coda_prompt` —con guion bajo—. Los lanzadores cuentan las líneas anteriores y solo devuelven 0 si aumenta el Class-IL (`run_l2p_b64_e1.sh:23-26,50-59`; `run_dualprompt_b64_e1.sh:22-25,49-58`; `run_coda_b64_e2.sh:22-25,49-58`); por tanto se añadió exactamente una línea Class-IL por método. La línea Task-IL no forma parte de esa aserción, pero también está presente y se comprobó directamente.
- Las líneas finales registran `batch_size=64`, `stop_after=10`, `seed=0` y el commit esperado. Registran además `n_epochs=1` para L2P/DualPrompt y `n_epochs=2` para CODA. No se interpreta ninguna accuracy como resultado comparable del TFG: son pruebas diagnósticas con presupuestos distintos y configuración `best`.

**Veredicto: PILOTO-0 v2 COMPLETADO** para las tres configuraciones diagnósticas anteriores, conforme al criterio de código 0 + diez tareas + nueva línea final de métricas. Esto materializa D32; no decide D33, no demuestra CODA a E=1 y no rellena ningún `valor final`.

### Tiempo y memoria observados

| método | tiempo real `/usr/bin/time -v` | RSS CPU máximo | máximo GPU que imprime Mammoth |
|---|---:|---:|---:|
| L2P | 30:35,89 | 2.660.756 kB | 6.099,35 MiB |
| DualPrompt | 28:20,38 | 2.628.496 kB | 6.398,29 MiB |
| CODA-Prompt | 54:26,92 | 2.547.640 kB | 8.705,77 MiB |

Fuentes de tiempo, RSS y salida: `l2p_seq-cifar100-224_seed0_b64_e1.time.txt:5,10,23`, `dualprompt_seq-cifar100-224_seed0_b64_e1.time.txt:5,10,23` y `coda-prompt_seq-cifar100-224_seed0_b64_e2.time.txt:5,10,23`. Fuentes de memoria GPU: los dos primeros `*.run.log:241-244` y el de CODA `:234-237`. El RSS de `/usr/bin/time` es memoria residente de CPU, no VRAM. El driver va de 22:40:30 a 00:33:54: **DERIVADO 1:53:24** (`driver_all.log:2,747`), coherente con la suma de los tres tiempos, 1:53:23,19; la diferencia residual ≈0,8 s es compatible con la resolución de un segundo de esos timestamps y la sobrecarga del driver.

La GPU no estaba limpia: el preflight de las 22:17:24 —anterior a la tentativa interrumpida— muestra una RTX 3060 de 12.288 MiB con 715 MiB usados y dos kernels Jupyter ajenos de 304 y 332 MiB (`gpu_preflight.txt:1,12-14,22-29`); la evidencia inmediata es que los tres `*.gpu_before.txt:1-2` vuelven a mostrar ambos kernels justo antes de cada método válido. Además, `pynvml` falló en las tres corridas (`run.log` de L2P `:116`, DualPrompt `:121`, CODA `:104`), por lo que Mammoth cayó a `torch.cuda.max_memory_allocated` (`utils/conf.py:53-91`) y dividió bytes entre 1024² (`utils/stats.py:31-36`). Los máximos de la tabla son, por tanto, picos acumulados de memoria *allocated* del proceso PyTorch —no VRAM total de la tarjeta, memoria reservada ni consumo de procesos ajenos—. Como la fuente es un máximo acumulado, los campos impresos «Average» y «Final» tampoco son un promedio temporal ni una lectura instantánea. No hubo muestreo de `nvidia-smi` durante la corrida: el pico total de VRAM queda `NO_DISPONIBLE` en el paquete.

El fichero `metricas_resumen.tsv:2-4` conserva correctamente batch, épocas, LR, accuracy y máximo GPU, pero su columna `elapsed` contiene solo el componente de segundos (`35.89`, `20.38`, `26.92`). Para duración, los `*.time.txt:5` anteriores son la fuente autoritativa. Los cambios de contexto involuntarios de `/usr/bin/time` fueron 162.248 para L2P, 10.619 para DualPrompt y 16.356 para CODA (`*.time.txt:15`); la causa del valor atípico de L2P no es determinable con el paquete, de modo que estos tiempos se conservan como observaciones de infraestructura, no como benchmark limpio.

### Incidencias aisladas y efecto sobre cuestiones abiertas

- Antes del driver válido hubo una tentativa L2P aislada en `logs/piloto0_v2/interrumpidos/l2p_20260803_223543/`: recibió `SIGINT`, guardó el checkpoint tras la tarea 5 y se detuvo al entrar en la 7 (`l2p_seq-cifar100-224_seed0_b64_e1.run.log:190-211`); su `.time.txt` está vacío. El directorio `l2p_20260803_224019/` está vacío. Ninguno cuenta como resultado; la corrida final reinició en tarea 1 y completó las diez.
- `driver_all.log` y `driver_nohup.log` son copias byte a byte, ambas SHA-256 `ca611376fe48219d1e6b0730f3595eda40cd126fb6bc07b8ad2143403ae68859`; no se cuentan como dos ejecuciones.
- Los lanzadores fijan `MAMMOTH_TEST=1`. En este SHA ese entorno hace que Mammoth ignore `.env` (`main.py:58-62`) y desactive W&B (`utils/training.py:48-49`); no reduce las tareas, como prueban los diez marcadores y `stop_after=10`.
- Los tres logs finales contienen avisos no bloqueantes de QuickGELU, importaciones opcionales y `pynvml`, pero ninguna aparición de `Traceback` ni `OutOfMemoryError`; los códigos 0 y resultados anteriores son la evidencia de terminación.
- Con batch 64, la configuración impresa conserva LR nominal 0,03 para L2P (`l2p_seq-cifar100-224_seed0_b64_e1.run.log:88`) y DualPrompt (`dualprompt_seq-cifar100-224_seed0_b64_e1.run.log:93`), mientras la línea final de cada `logs.pyd` registra 0,0075; CODA registra 0,001. El escalado observado coincide con `0,03 × 64 / 256 = 0,0075` (`models/l2p.py:66`; `models/dualprompt.py:71`). Esto amplía la evidencia de PD-04 sobre `best`; no valida el overlay como receta final.
- Batch real 64 completó una secuencia íntegra con los tres métodos, incluso con los procesos ajenos descritos. Es evidencia operativa nueva para PD-01 y respalda la premisa de que CODA tuvo el mayor pico *allocated* observado, pero no convierte batch 64 en `valor final`.
- CODA completó el bucle a E=2. Es evidencia operativa nueva para PD-02/HB-09/FD-04, no una prueba de E=1 ni una decisión sobre `{2,5,20}` frente a parche.

## Resultado del Piloto-0 v1: evidencia de infraestructura

Paquete: `C:\Users\alex\Downloads\faseA_servidor_piloto0_oom_20260803_154958.tar.gz`, SHA-256 `6e9fc8d88595f8219f5a60f16274c281e3c4ab077e909d86c304eec88e3e0d25`.

- L2P resolvió `best`, batch 128, LR nominal 0,03 y E=1, alcanzó la tarea 1 y abortó con OOM; `logs/piloto0/driver.log:24-35,83-108,141-198` y `diagnostico_oom_20260803_154949.txt:207-263` dentro del paquete.
- Código de salida 1, 17,21 s hasta el fallo y ningún `logs.pyd`; `diagnostico_oom_20260803_154949.txt:117-123,138-162`.
- El lanzador v1 salía en el primer error; DualPrompt no llegó a ejecutarse y CODA-Prompt no estaba incluido; `run_piloto0.sh:43-49,60-61`.
- Había dos kernels ajenos consumiendo 304 y 332 MiB. Esto limita la atribución fina del margen de VRAM, pero no convierte el intento en un piloto completado; `diagnostico_oom_20260803_154949.txt:31-45,263`.

Por D31, batch 128 permanece cerrado. El v1 se conserva como evidencia de infraestructura y no como Piloto-0 satisfactorio. El paquete no contiene una corrida a batch 64; el hecho previo de que 64 cabe está registrado en `PLAN_MAESTRO.md §10 (hecho duro)`.

## Precondiciones comunes

Cada bloque posterior es autocontenido y puede lanzarse por separado. Antes de medir tiempos, el usuario debe coordinar la liberación de procesos de cómputo ajenos; no se incluyen órdenes para terminarlos. Cada lanzador guarda el estado GPU previo para poder invalidar una medición contaminada.

El overlay privado ya comprobado debe seguir accesible:

```bash
export PYTHONPATH="$HOME/.local/mammoth-pydeps${PYTHONPATH:+:$PYTHONPATH}"
/opt/environment/bin/python -c 'import open_clip, ftfy; print(open_clip.__version__, ftfy.__version__)'
```

Resultado esperado del entorno auditado: `open-clip-torch==2.32.0` y `ftfy==6.3.1`; evidencia en `infra/servidor.md`.

## Lanzador independiente: L2P, E=1

```bash
#!/usr/bin/env bash
set -u
set -o pipefail

cd "$HOME/mammothV2"
EXPECTED_COMMIT="cda7f23681f7bffacee460d99e990bc803bccf04"
ACTUAL_COMMIT="$(git rev-parse HEAD)"
test "$ACTUAL_COMMIT" = "$EXPECTED_COMMIT" || {
  echo "ERROR: HEAD=$ACTUAL_COMMIT; esperado=$EXPECTED_COMMIT" >&2
  exit 2
}

PYTHON_BIN="/opt/environment/bin/python"
export PYTHONPATH="$HOME/.local/mammoth-pydeps${PYTHONPATH:+:$PYTHONPATH}"
export MAMMOTH_TEST=1
export PYTHONUNBUFFERED=1
unset WANDB_ENTITY WANDB_PROJECT

mkdir -p logs/piloto0_v2
prefix="logs/piloto0_v2/l2p_seq-cifar100-224_seed0_b64_e1"
result_file="data/results_piloto0_v2/class-il/seq-cifar100-224/l2p/logs.pyd"
before_lines=0
test ! -f "$result_file" || before_lines="$(wc -l < "$result_file")"
nvidia-smi --query-compute-apps=pid,process_name,used_memory \
  --format=csv,noheader > "${prefix}.gpu_before.txt"

/usr/bin/time -v -o "${prefix}.time.txt" \
  "$PYTHON_BIN" -u main.py \
    --model l2p \
    --dataset seq-cifar100-224 \
    --model_config best \
    --batch_size 64 \
    --fitting_mode epochs \
    --n_epochs 1 \
    --stop_after 10 \
    --seed 0 \
    --permute_classes 0 \
    --base_path ./data/ \
    --results_path results_piloto0_v2/ \
    --notes piloto0_v2_cifar100_l2p_seed0_b64_e1 \
    --non_verbose 1 \
    --code_optimization 0 \
    --distributed no \
    2>&1 | tee "${prefix}.run.log"

status=${PIPESTATUS[0]}
printf '%s\n' "$status" > "${prefix}.exit_code.txt"
test "$status" -eq 0 || exit "$status"
test -s "$result_file" || exit 81
after_lines="$(wc -l < "$result_file")"
test "$after_lines" -gt "$before_lines" || exit 82
tail -n 1 "$result_file"
```

## Lanzador independiente: DualPrompt, E=1

```bash
#!/usr/bin/env bash
set -u
set -o pipefail

cd "$HOME/mammothV2"
EXPECTED_COMMIT="cda7f23681f7bffacee460d99e990bc803bccf04"
ACTUAL_COMMIT="$(git rev-parse HEAD)"
test "$ACTUAL_COMMIT" = "$EXPECTED_COMMIT" || {
  echo "ERROR: HEAD=$ACTUAL_COMMIT; esperado=$EXPECTED_COMMIT" >&2
  exit 2
}

PYTHON_BIN="/opt/environment/bin/python"
export PYTHONPATH="$HOME/.local/mammoth-pydeps${PYTHONPATH:+:$PYTHONPATH}"
export MAMMOTH_TEST=1
export PYTHONUNBUFFERED=1
unset WANDB_ENTITY WANDB_PROJECT

mkdir -p logs/piloto0_v2
prefix="logs/piloto0_v2/dualprompt_seq-cifar100-224_seed0_b64_e1"
result_file="data/results_piloto0_v2/class-il/seq-cifar100-224/dualprompt/logs.pyd"
before_lines=0
test ! -f "$result_file" || before_lines="$(wc -l < "$result_file")"
nvidia-smi --query-compute-apps=pid,process_name,used_memory \
  --format=csv,noheader > "${prefix}.gpu_before.txt"

/usr/bin/time -v -o "${prefix}.time.txt" \
  "$PYTHON_BIN" -u main.py \
    --model dualprompt \
    --dataset seq-cifar100-224 \
    --model_config best \
    --batch_size 64 \
    --fitting_mode epochs \
    --n_epochs 1 \
    --stop_after 10 \
    --seed 0 \
    --permute_classes 0 \
    --base_path ./data/ \
    --results_path results_piloto0_v2/ \
    --notes piloto0_v2_cifar100_dualprompt_seed0_b64_e1 \
    --non_verbose 1 \
    --code_optimization 0 \
    --distributed no \
    2>&1 | tee "${prefix}.run.log"

status=${PIPESTATUS[0]}
printf '%s\n' "$status" > "${prefix}.exit_code.txt"
test "$status" -eq 0 || exit "$status"
test -s "$result_file" || exit 81
after_lines="$(wc -l < "$result_file")"
test "$after_lines" -gt "$before_lines" || exit 82
tail -n 1 "$result_file"
```

## Lanzador independiente: CODA-Prompt, E=2

```bash
#!/usr/bin/env bash
set -u
set -o pipefail

cd "$HOME/mammothV2"
EXPECTED_COMMIT="cda7f23681f7bffacee460d99e990bc803bccf04"
ACTUAL_COMMIT="$(git rev-parse HEAD)"
test "$ACTUAL_COMMIT" = "$EXPECTED_COMMIT" || {
  echo "ERROR: HEAD=$ACTUAL_COMMIT; esperado=$EXPECTED_COMMIT" >&2
  exit 2
}

PYTHON_BIN="/opt/environment/bin/python"
export PYTHONPATH="$HOME/.local/mammoth-pydeps${PYTHONPATH:+:$PYTHONPATH}"
export MAMMOTH_TEST=1
export PYTHONUNBUFFERED=1
unset WANDB_ENTITY WANDB_PROJECT

mkdir -p logs/piloto0_v2
prefix="logs/piloto0_v2/coda-prompt_seq-cifar100-224_seed0_b64_e2"
result_file="data/results_piloto0_v2/class-il/seq-cifar100-224/coda_prompt/logs.pyd"
before_lines=0
test ! -f "$result_file" || before_lines="$(wc -l < "$result_file")"
nvidia-smi --query-compute-apps=pid,process_name,used_memory \
  --format=csv,noheader > "${prefix}.gpu_before.txt"

/usr/bin/time -v -o "${prefix}.time.txt" \
  "$PYTHON_BIN" -u main.py \
    --model coda-prompt \
    --dataset seq-cifar100-224 \
    --model_config best \
    --batch_size 64 \
    --fitting_mode epochs \
    --n_epochs 2 \
    --stop_after 10 \
    --seed 0 \
    --permute_classes 0 \
    --base_path ./data/ \
    --results_path results_piloto0_v2/ \
    --notes piloto0_v2_cifar100_coda-prompt_seed0_b64_e2 \
    --non_verbose 1 \
    --code_optimization 0 \
    --distributed no \
    2>&1 | tee "${prefix}.run.log"

status=${PIPESTATUS[0]}
printf '%s\n' "$status" > "${prefix}.exit_code.txt"
test "$status" -eq 0 || exit "$status"
test -s "$result_file" || exit 81
after_lines="$(wc -l < "$result_file")"
test "$after_lines" -gt "$before_lines" || exit 82
tail -n 1 "$result_file"
```

### Motivo exacto de E=2 para CODA

1. `CodaPrompt.begin_task` construye `CosineSchedule(self.opt, K=self.args.n_epochs)` (`models/coda_prompt.py:65-73`).
2. `CosineSchedule.__init__` impone `assert K > 1` (`utils/schedulers.py:38-46`).
3. `meta_begin_task` se ejecuta antes del entrenamiento de cada tarea (`utils/training.py:197-216,232`).
4. El dump E=1 registra `K=1/runtime_valid=false`, pero termina antes de construir el modelo (`auditoria/reconciliacion_fase_b.md:73-82`).

E=2 es el mínimo ejecutable sin cambiar código. No resuelve la decisión entre una escalera `{2,5,20}` y un parche del scheduler; esa cuestión permanece en `auditoria/cola_revision.md`.

## Reintento diagnóstico L2P/batch128: registrado y retirado

Tras el OOM v1 se propuso repetir L2P con GPU limpia y el ajuste sugerido por PyTorch. D31 rev. y la regla D34 retiraron ese reintento como test de batch: 128 ya está cerrado y la limpieza GPU pasa a ser higiene para tiempos. Para conservar trazabilidad, este era el comando mínimo propuesto; queda **ARCHIVADO — NO EJECUTAR para reabrir D31**:

```bash
# ARCHIVADO — NO forma parte del Piloto-0 v2.
cd "$HOME/mammothV2"
export PYTHONPATH="$HOME/.local/mammoth-pydeps${PYTHONPATH:+:$PYTHONPATH}"
export MAMMOTH_TEST=1
export PYTHONUNBUFFERED=1
unset WANDB_ENTITY WANDB_PROJECT

# La salida debía mostrar cero procesos de cómputo ajenos antes de lanzar.
nvidia-smi --query-compute-apps=pid,process_name,used_memory --format=csv,noheader

env PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True \
  /usr/bin/time -v -o logs/piloto0/l2p_b128_e1_gpu_limpia.time.txt \
  /opt/environment/bin/python -u main.py \
    --model l2p \
    --dataset seq-cifar100-224 \
    --model_config best \
    --batch_size 128 \
    --fitting_mode epochs \
    --n_epochs 1 \
    --stop_after 10 \
    --seed 0 \
    --permute_classes 0 \
    --base_path ./data/ \
    --results_path results_piloto0_diagnostico/ \
    --notes diagnostico_retirado_l2p_b128_e1 \
    --non_verbose 1 \
    --code_optimization 0 \
    --distributed no
```

No hay resultado de este reintento en el paquete y no se le atribuye ninguno.

## Resultados, logs y criterio de éxito

Con `--base_path ./data/` y `--results_path results_piloto0_v2/`, los ficheros class-IL esperados son:

- L2P: `data/results_piloto0_v2/class-il/seq-cifar100-224/l2p/logs.pyd`;
- DualPrompt: `data/results_piloto0_v2/class-il/seq-cifar100-224/dualprompt/logs.pyd`;
- CODA-Prompt: `data/results_piloto0_v2/class-il/seq-cifar100-224/coda_prompt/logs.pyd`.

Mammoth también escribe la evaluación task-IL bajo `data/results_piloto0_v2/task-il/seq-cifar100-224/<modelo>/logs.pyd` (`utils/loggers.py:247-264`). `logs.pyd` es append-only; por eso cada lanzador exige que aumente su número de líneas, no solo que el fichero exista. Los logs de consola, tiempo, código de salida y GPU previa quedan bajo `logs/piloto0_v2/`.

Una corrida cuenta como Piloto-0 completado solo con código 0, las diez tareas y una nueva línea final de métricas. Un arranque, un dump o un fallo con log no cumplen ese criterio.

## Descarga de datos

CIFAR-100 usa torchvision con `download=True` (`datasets/seq_cifar100_224.py:80-83`); si falta o no supera su comprobación de integridad, intenta descargarlo bajo `./data/CIFAR100`.

## Opcional: Split ImageNet-R

`seq-imagenet-r` existe y declara 10 tareas de 20 clases (`datasets/seq_imagenet_r.py:112-120`), pero `models/config/l2p.yaml`, `models/config/dualprompt.yaml` y `models/config/coda_prompt.yaml` no tienen una entrada `seq-imagenet-r`. Por ello `--model_config best` falla para los tres y los defaults parciales de clase no forman una receta comparable.

Los comandos completos equivalentes siguen `NO_DETERMINABLE_ESTATICO`: escribirlos exigiría elegir LR, transforms y otros valores aún pendientes. Una vez aprobada una receta humana, debe volcarse primero la configuración efectiva y solo después materializar el piloto opcional. No se promueven los valores de smoke tests a receta experimental.
