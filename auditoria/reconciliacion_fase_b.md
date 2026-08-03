# Reconciliación de la Fase B: volcados E=1 y E=20

Fecha de reconciliación: 2026-08-03. Esta fase incorpora evidencia, no decide el EJE ni rellena ningún `valor final`.

## 1. Alcance y procedencia

Se auditó el archivo externo `auditoria_dumps_e1_e20.tar.gz` (SHA-256 `7254a9033d174398b8e9ef2a35ba1951e3ece32fa0e0a48a51bb49d358fa19c8`). Los dos JSON declaran:

- host `galatzo`;
- Mammoth `cda7f23681f7bffacee460d99e990bc803bccf04`, rama aportada `tfg-auditoria`;
- dataset `seq-cifar100-224`;
- modelos `l2p`, `dualprompt` y `coda-prompt`;
- `model_config=best` y `seed=0`;
- E=1 o E=20 según el artefacto.

| artefacto dentro del tar | bytes | SHA-256 |
|---|---:|---|
| `auditoria_dumps/cifar100_n_epochs_1.json` | 33.809 | `0156a129b0ce25f2fa8e2f20587904196b14d984cf574d2f0bbed8951f199955` |
| `auditoria_dumps/cifar100_n_epochs_20.json` | 33.764 | `258277b6aa90f79a95172693889eccae4208e07e4cd1fd0ec85652ca130013a9` |
| `auditoria_dumps/cifar100_n_epochs_1.stderr.log` | 284 | `9d7bec0c8276ac90a8a722a3ab59bd8aadc87605a6a72059b89149a87a1c6079` |
| `auditoria_dumps/cifar100_n_epochs_20.stderr.log` | 284 | `9d7bec0c8276ac90a8a722a3ab59bd8aadc87605a6a72059b89149a87a1c6079` |
| `logs/piloto0/environment_before_openclip.txt` | 9.540 | `fe58357f761eda22393ee9cb3481617aa641a2053725203d7b1e7b4b77f2b6c0` |
| `logs/piloto0/environment_after_openclip.txt` | 2.685 | `85005cded823431421a3b844d19819f66c9922e5fdc8cb782b2bdf6ceb7524ac` |

Ambos JSON son válidos, usan `schema_version=1` y contienen los tres métodos en el mismo orden. Los stderr son byte a byte idénticos: solo recogen un `UserWarning` de OpenCLIP sobre QuickGELU causado por el import eager. No se atribuye ese aviso a ninguno de los tres métodos.

## 2. Comparación estructural E=1 frente a E=20

El diff recursivo exhaustivo encontró 34 hojas distintas:

- una en el scope: `scope.n_epochs: 1 → 20`;
- diez por método, comunes a los tres: el valor textual del comando, `n_epochs` en el parser, namespace efectivo y vista estable, más `conf_jobnum` y `conf_timestamp`;
- tres adicionales en CODA-Prompt: `custom_scheduler.K: 1 → 20`, `runtime_valid: false → true` y la consecuencia inválida pasa de `AssertionError at begin_task before the first training epoch` a `null`.

`conf_jobnum` y `conf_timestamp` son metadatos volátiles, no configuración experimental. Fuera de ellos, `n_epochs` es el único argumento estable que cambia. No cambia ningún campo de `dataset_effective`, ningún hiperparámetro de prompt, optimizador, LR, batch, seed, protocolo o transform. Tras normalizar solo los campos enumerados, los dos documentos producen el mismo JSON canónico, SHA-256 `bd1e8e55d15bce26d1feac5d71fa0183acc7c69468761128c78ad8a646c93fba`.

## 3. Verificación de los valores Mammoth

La verificación se limita a lo que alcanza el script. Su último paso ejecutado es `main.check_args(args, dataset)`; no construye backbone, modelo ni loaders y no entrena. Por ello valida la resolución parser/config/dataset, pero las notas de LR y K son derivaciones estáticas marcadas `NOT_APPLIED_BY_THIS_SCRIPT`.

| método | valores confirmados por la vista estable | nota posterior al parser | resultado frente a las tablas |
|---|---|---|---|
| L2P | `dataset_config=l2p`, batch 128, Adam, LR parseado `0.03`, pool 10, longitud 5, top-k 5, batchwise activo, `drop_last=false`, scheduler genérico `None` | LR runtime localizado `0.03×128/256=0.015`, no ejecutado por el dump | Sin discrepancia numérica en `valores_l2p.md`; se completa solo la transform efectiva del servidor. |
| DualPrompt | `dataset_config=l2p`, batch 128, Adam, LR parseado `0.03`, G en capas `[0,1]`, E en `[2,3,4]`, longitudes 5, pool 10, top-k 1, batchwise activo, `use_permute_fix=false` y clave YAML inerte `use_fix_permute=0` | LR runtime localizado `0.03×128/256=0.015`, no ejecutado por el dump | Sin discrepancia numérica en `valores_dualprompt.md`; se completa solo la transform efectiva del servidor. |
| CODA-Prompt | `dataset_config=coda_prompt`, batch 128, Adam, LR `0.001`, `optim_mom=0.9`, pool 100, longitud 8, `mu=0`, `ortho_mu=0`, `virtual_bs_iterations=1`, `drop_last=false`, scheduler genérico `None` | `K=n_epochs`; K=1 inválido y K=20 válido, no ejecutados por el dump | Sin discrepancia numérica en `valores_coda.md`; se precisa la transform efectiva del servidor. |

El `seed=0` observado pertenece al comando diagnóstico. No corrige las filas que describen el default de receta sin semilla (`None`). Del mismo modo, el override E=1 no reemplaza los presupuestos originales de cinco o veinte épocas registrados en las tablas.

### Transform efectiva cerrada para este entorno

Los dos volcados contienen exactamente el mismo `dataset_effective`. Con torchvision 0.17.1+cu121 en `galatzo`:

| método | train CIFAR-100 | evaluación CIFAR-100 |
|---|---|---|
| L2P / DualPrompt | `Resize(256, bilinear, antialias=True) → RandomResizedCrop(224, scale=(0.08,1.0), ratio=(0.75,1.3333), bilinear, antialias=True) → RandomHorizontalFlip(p=0.5) → ToTensor → Normalize(0,1)` | `Resize(256, bicubic, antialias=True) → CenterCrop(224) → ToTensor → Normalize(0,1)` |
| CODA-Prompt | `RandomResizedCrop(224, scale=(0.08,1.0), ratio=(0.75,1.3333), bilinear, antialias=True) → RandomHorizontalFlip(p=0.5) → ToTensor → Normalize(0,1)` | `Resize(224, bilinear, antialias=True) → ToTensor → Normalize(0,1)` |

Esta evidencia corrige la indeterminación anterior de Mammoth para el servidor auditado. El YAML sigue delegando parte de esos valores a torchvision; por tanto no se convierte en un pin portable a otros entornos.

## 4. Valores y comportamientos afectados por `n_epochs`

La lista exhaustiva dentro del alcance de los volcados y de las rutas de código ya auditadas es:

1. **Namespace y trazabilidad, los tres métodos.** Cambian el argumento CLI, `scope.n_epochs`, `parser_args_complete.n_epochs`, `effective_args_after_dataset_extension_complete.n_epochs` y `stable_effective_args_for_diff.n_epochs`.
2. **Bucle canónico por épocas, los tres métodos.** Con `fitting_mode=epochs`, fija `n_epochs×len(train_loader)` como total de progreso/updates previsto y termina cuando `epoch >= n_epochs` (`utils/training.py:248-269`). El número concreto de batches no se obtiene de estos dumps porque no construyen loaders.
3. **Gate de entrenamiento, los tres métodos.** La entrada al bloque exige `n_epochs>0`; tanto 1 como 20 lo satisfacen (`utils/training.py:232`).
4. **Evaluación intra-tarea condicional, los tres métodos.** Si se configurase `eval_epochs`, `n_epochs` limita la condición `epoch < n_epochs`; en los volcados `eval_epochs=None`, por lo que no se activa (`utils/training.py:298-301`).
5. **Cosine genérico condicional, L2P y DualPrompt.** Si se abandonase la receta y se activase `lr_scheduler=cosine`, su `T_max` sería `n_epochs`; en ambos volcados el scheduler es `None`, de modo que E=1/E=20 no cambia el LR (`utils/schedulers.py:19-35`).
6. **Scheduler custom y validez, CODA-Prompt.** `begin_task` deriva obligatoriamente `custom_scheduler.K=n_epochs`. E=1 produce K=1 e incumple K>1; E=20 produce K=20 y supera la validación (`models/coda_prompt.py:65-73`; `utils/schedulers.py:38-46`). El dispatcher no entrega ese objeto al bucle, por lo que para K válido el LR efectivo sigue constante; cambiar E no repara el cableado (`utils/training.py:239-263`).

No se detectó dependencia adicional de E en transforms, batch, LR parseado, LR runtime localizado de L2P/DualPrompt, optimizador, prompts, dataset, orden, seed, `drop_last`, modos de ajuste ni controles de evaluación. Los modos `iters`, `early_stopping`, `time`, los límites de tareas, debug e `inference_only` siguen siendo controles de duración separados; no se modifican en esta fase.

## 5. Bloqueo de CODA-Prompt con E=1

**Confirmado como bloqueo de la corrida de entrenamiento, no como fallo del script de volcado.** La distinción explica por qué ambos volcados terminaron con código 0:

1. el parser acepta `--n_epochs 1` y el dump termina después de `main.check_args`;
2. el propio JSON E=1 registra `K=1`, `runtime_valid=false` y la consecuencia `AssertionError at begin_task before the first training epoch`;
3. en una corrida real, el bucle llama a `meta_begin_task` antes de entrar al bloque de epochs, y el wrapper llama a `CodaPrompt.begin_task` (`utils/training.py:197-216,232-263`; `models/utils/continual_model.py:449-475`);
4. `begin_task` construye `CosineSchedule(self.opt, K=self.args.n_epochs)` y su constructor ejecuta `assert K > 1, "K must be greater than 1"` (`models/coda_prompt.py:65-73`; `utils/schedulers.py:38-46`).

Veredicto de Fase B: el bloqueo documentado en `entorno.md`, `piloto0.md`, `comportamiento.md` y `cola_revision.md` queda confirmado. No se cambia el EJE, no se adopta E=2, no se simula una época con `iters` y no se modifica Mammoth. Cualquiera de esas alternativas sigue requiriendo una decisión humana posterior.

## 6. Correcciones aplicadas

Se corrigió exclusivamente la indeterminación demostrada de transforms CIFAR de Mammoth y se añadió trazabilidad de la evidencia de Fase B en:

- `auditoria/fuentes.md`;
- `auditoria/valores_l2p.md`;
- `auditoria/valores_dualprompt.md`;
- `auditoria/valores_coda.md`;
- `auditoria/entorno.md`;
- `auditoria/comportamiento_l2p.md`;
- `auditoria/comportamiento_dualprompt.md`;
- `auditoria/comportamiento_coda.md`;
- `auditoria/comportamiento.md`;
- `auditoria/piloto0.md` y `auditoria/cola_revision.md` para dejar inequívoca la frontera del bloqueo CODA.

No se ha escrito ningún `valor final`, no se ha cambiado ningún `valor propuesto`, caso, grupo, flag S o decisión pendiente.
