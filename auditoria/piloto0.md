# Piloto-0 v2: CIFAR-100 con lanzadores independientes

## Alcance, decisiones aplicadas y límites

Los comandos están escritos para Linux, desde la raíz de un clon de `https://github.com/alexmf10/mammothV2`, rama `tfg-auditoria`, commit `cda7f23681f7bffacee460d99e990bc803bccf04`. El dataset es `seq-cifar100-224`: 10 tareas, 10 clases por tarea y 100 clases (`datasets/seq_cifar100_224.py:39-43`).

| método | presupuesto del piloto v2 | batch | motivo |
|---|---:|---:|---|
| L2P | E=1 por tarea | 64 | CLI limpia; lanzador independiente. |
| DualPrompt | E=1 por tarea | 64 | CLI limpia; lanzador independiente. |
| CODA-Prompt | E=2 por tarea | 64 | E=1 aborta en `begin_task` porque `CosineSchedule` exige `K=n_epochs>1`. |

E=2 para CODA es una excepción operativa del Piloto-0 v2 decidida en D32: sirve para comprobar el bucle completo y derivar coste por época-tarea. **No modifica el eje experimental**, cuyo peldaño mínimo sigue pendiente en D33. Batch 64 aplica D31/D32 al piloto; la columna `valor final` de la auditoría sigue vacía.

Los tres lanzadores usan `model_config=best` únicamente como baseline diagnóstico heredado del Piloto-0. La matriz final no debe reutilizar `best/default` en bloque: se construirá desde `configuracion_final` después de la revisión humana (`PLAN_MAESTRO.md:221`).

## Resultado del Piloto-0 v1: evidencia de infraestructura

Paquete: `C:\Users\alex\Downloads\faseA_servidor_piloto0_oom_20260803_154958.tar.gz`, SHA-256 `6e9fc8d88595f8219f5a60f16274c281e3c4ab077e909d86c304eec88e3e0d25`.

- L2P resolvió `best`, batch 128, LR nominal 0,03 y E=1, alcanzó la tarea 1 y abortó con OOM; `logs/piloto0/driver.log:24-35,83-108,141-198` y `diagnostico_oom_20260803_154949.txt:207-263` dentro del paquete.
- Código de salida 1, 17,21 s hasta el fallo y ningún `logs.pyd`; `diagnostico_oom_20260803_154949.txt:117-123,138-162`.
- El lanzador v1 salía en el primer error; DualPrompt no llegó a ejecutarse y CODA-Prompt no estaba incluido; `run_piloto0.sh:43-49,60-61`.
- Había dos kernels ajenos consumiendo 304 y 332 MiB. Esto limita la atribución fina del margen de VRAM, pero no convierte el intento en un piloto completado; `diagnostico_oom_20260803_154949.txt:31-45,263`.

Por D31, batch 128 permanece cerrado. El v1 se conserva como evidencia de infraestructura y no como Piloto-0 satisfactorio. El paquete no contiene una corrida a batch 64; el hecho previo de que 64 cabe está registrado en `PLAN_MAESTRO.md:163`.

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
