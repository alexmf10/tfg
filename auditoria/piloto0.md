# Piloto-0: CIFAR-100, 10 tareas y 1 época por tarea

## Alcance y estado

Los comandos están escritos para Linux, desde la raíz de un clon de `https://github.com/alexmf10/mammothV2` situado en la rama `tfg-auditoria`. El dataset correcto para estos métodos es `seq-cifar100-224`: declara 10 tareas, 10 clases por tarea y 100 clases (`datasets/seq_cifar100_224.py:39-43`). `seq-cifar100` no es equivalente porque usa el backbone por defecto de esa otra clase de dataset.

| método | estado del Piloto-0 solicitado | motivo |
|---|---|---|
| L2P | EJECUTABLE | La CLI acepta la receta `best` y `n_epochs=1`. |
| DualPrompt | EJECUTABLE | La CLI acepta la receta `best` y `n_epochs=1`. |
| CODA-Prompt | BLOQUEANTE / NO_EJECUTABLE | Su scheduler exige `n_epochs > 1`; no existe flag para desactivarlo. |

No se presenta una corrida fallida de CODA-Prompt como si fuera completa.

## Preparación común en el servidor

```bash
git switch tfg-auditoria
git rev-parse HEAD

set -o pipefail
mkdir -p logs/piloto0
unset WANDB_ENTITY WANDB_PROJECT
export MAMMOTH_TEST=1
export PYTHONUNBUFFERED=1
```

`MAMMOTH_TEST=1` evita cargar un `.env` local (`main.py:50-64`). Al dejar sin definir `WANDB_ENTITY` y `WANDB_PROJECT`, Mammoth desactiva W&B durante `extend_args` (`main.py:447-457`). No se fija `num_workers`: Mammoth conserva su resolución dependiente del entorno; es infraestructura y debe anotarse al comparar tiempos.

## L2P

```bash
/usr/bin/time -v -o logs/piloto0/l2p_seq-cifar100-224_seed0_e1.time.txt \
python main.py \
  --model l2p \
  --dataset seq-cifar100-224 \
  --model_config best \
  --fitting_mode epochs \
  --n_epochs 1 \
  --stop_after 10 \
  --seed 0 \
  --permute_classes 0 \
  --base_path ./data/ \
  --results_path results_piloto0/ \
  --notes piloto0_cifar100_l2p_seed0_e1 \
  --non_verbose 1 \
  --code_optimization 0 \
  --distributed no \
  2>&1 | tee logs/piloto0/l2p_seq-cifar100-224_seed0_e1.run.log

test -s data/results_piloto0/class-il/seq-cifar100-224/l2p/logs.pyd
tail -n 1 data/results_piloto0/class-il/seq-cifar100-224/l2p/logs.pyd
```

## DualPrompt

```bash
/usr/bin/time -v -o logs/piloto0/dualprompt_seq-cifar100-224_seed0_e1.time.txt \
python main.py \
  --model dualprompt \
  --dataset seq-cifar100-224 \
  --model_config best \
  --fitting_mode epochs \
  --n_epochs 1 \
  --stop_after 10 \
  --seed 0 \
  --permute_classes 0 \
  --base_path ./data/ \
  --results_path results_piloto0/ \
  --notes piloto0_cifar100_dualprompt_seed0_e1 \
  --non_verbose 1 \
  --code_optimization 0 \
  --distributed no \
  2>&1 | tee logs/piloto0/dualprompt_seq-cifar100-224_seed0_e1.run.log

test -s data/results_piloto0/class-il/seq-cifar100-224/dualprompt/logs.pyd
tail -n 1 data/results_piloto0/class-il/seq-cifar100-224/dualprompt/logs.pyd
```

## CODA-Prompt: bloqueo de la corrida solicitada

El comando análogo sería el siguiente, pero **no completa** la primera tarea y no genera el fichero final de métricas:

```bash
# NO EJECUTAR esperando una corrida completa: reproduce el bloqueo documentado.
/usr/bin/time -v -o logs/piloto0/coda-prompt_seq-cifar100-224_seed0_e1.time.txt \
python main.py \
  --model coda-prompt \
  --dataset seq-cifar100-224 \
  --model_config best \
  --fitting_mode epochs \
  --n_epochs 1 \
  --stop_after 10 \
  --seed 0 \
  --permute_classes 0 \
  --base_path ./data/ \
  --results_path results_piloto0/ \
  --notes piloto0_cifar100_coda-prompt_seed0_e1 \
  --non_verbose 1 \
  --code_optimization 0 \
  --distributed no \
  2>&1 | tee logs/piloto0/coda-prompt_seq-cifar100-224_seed0_e1.run.log
```

Cadena de evidencia del bloqueo:

1. `CodaPrompt.begin_task` construye siempre `CosineSchedule(self.opt, K=self.args.n_epochs)` (`models/coda_prompt.py:65-73`).
2. `CosineSchedule.__init__` impone `assert K > 1` (`utils/schedulers.py:38-45`).
3. `meta_begin_task` se ejecuta antes del bloque de entrenamiento de la tarea (`utils/training.py:197-216,232`).
4. El test del propio repo usa dos épocas para CODA-Prompt (`tests/test_codaprompt.py:16-19`).
5. CODA-Prompt exige que `lr_scheduler` sea `None`, por lo que la CLI tampoco permite sustituir el scheduler (`models/coda_prompt.py:42`).

Resultado: no existe en este SHA un comando de la CLI, en modo `epochs` y sin modificar código, que complete exactamente una época por tarea para CODA-Prompt. Cambiar a dos épocas o a `fitting_mode=iters` alteraría el EJE y no es el Piloto-0 pedido; requiere una decisión humana registrada.

### Confirmación de Fase B en `galatzo`

Los volcados E=1 y E=20 del commit `cda7f23681f7bffacee460d99e990bc803bccf04` confirman que la CLI resuelve limpiamente `n_epochs` y que CODA deriva `custom_scheduler.K` del mismo valor. E=1 registra `K=1`, `runtime_valid=false` y `AssertionError at begin_task before the first training epoch`; E=20 registra `K=20`, `runtime_valid=true` y consecuencia nula. Evidencia e integridad: `auditoria/reconciliacion_fase_b.md`.

Ambos scripts de volcado terminan con código 0 porque su frontera declarada es `main.check_args`: no construyen el modelo y no llaman a `begin_task`. Ese éxito no es una corrida CODA E=1 ni contradice el bloqueo anterior. La confirmación combina el valor efectivo del parser, la derivación registrada por el dump y la ruta ejecutable `meta_begin_task → CodaPrompt.begin_task → CosineSchedule.__init__`; no se afirma que el tar contenga un traceback de entrenamiento, porque no lo contiene.

## Resultados y tiempos

Con `--base_path ./data/` y `--results_path results_piloto0/`, Mammoth compone las rutas (`utils/args.py:363-368`, `utils/loggers.py:236-245`) y añade una línea de texto con `str(dict)` a:

- L2P class-IL: `data/results_piloto0/class-il/seq-cifar100-224/l2p/logs.pyd`.
- DualPrompt class-IL: `data/results_piloto0/class-il/seq-cifar100-224/dualprompt/logs.pyd`.
- CODA-Prompt, solo si se resolviera el bloqueo: `data/results_piloto0/class-il/seq-cifar100-224/coda_prompt/logs.pyd`.
- La evaluación con máscara de tarea se duplica bajo `data/results_piloto0/task-il/seq-cifar100-224/<modelo>/logs.pyd` (`utils/loggers.py:247-264`).

El fichero `logs.pyd` se escribe al terminar la corrida (`utils/training.py:365-368`) y es append-only: una repetición con la misma ruta añade otra línea. Los logs completos de consola quedan en `logs/piloto0/*.run.log`; `/usr/bin/time -v` guarda duración y uso de recursos en `logs/piloto0/*.time.txt`. Mammoth no persiste por sí solo un resumen equivalente del tiempo total.

## Descarga de datos

- CIFAR-100: el loader de 224 píxeles llama a torchvision con `download=True` (`datasets/seq_cifar100_224.py:80-83`); si falta o no supera la comprobación de integridad, torchvision intenta descargarlo bajo `./data/CIFAR100`.
- Split ImageNet-R: `datasets/seq_imagenet_r.py:43-69,134-141` intenta descargar y extraer el tar si no existe `./data/imagenet-r/`. Requiere `requests`, el ejecutable `tar` y acceso a `https://people.eecs.berkeley.edu/~hendrycks/imagenet-r.tar`. Si la carpeta existe pero está vacía o incompleta, la comprobación estática muestra que el código no fuerza una nueva descarga.

## Opcional: Split ImageNet-R

`seq-imagenet-r` existe y declara 10 tareas de 20 clases (`datasets/seq_imagenet_r.py:112-120`). Hay smoke tests para L2P y CODA-Prompt (`tests/test_l2p.py:8-28`, `tests/test_codaprompt.py:8-31`), pero no para DualPrompt.

Los YAML `models/config/l2p.yaml`, `models/config/dualprompt.yaml` y `models/config/coda_prompt.yaml` no contienen una entrada `seq-imagenet-r`. En consecuencia, `--model_config best` falla y la CLI exige elegir manualmente, como mínimo, un learning rate. Los valores `lr=1e-4` y `batch_size=2` de los tests son solo de smoke/debug y no constituyen una receta experimental. Además, CODA-Prompt conserva el bloqueo `n_epochs=1`.

Por estas evidencias, los comandos completos equivalentes de ImageNet-R quedan marcados `NO_DETERMINABLE_ESTATICO`: escribirlos ahora exigiría inventar o trasladar silenciosamente valores no resueltos por Mammoth. Propuesta mínima de comprobación posterior: una vez que la auditoría humana fije la receta ImageNet-R por método, ejecutar primero `scripts_tfg/dump_config.py` adaptado a ese dataset o un script homólogo de parseo, y solo entonces materializar los comandos del piloto opcional.
