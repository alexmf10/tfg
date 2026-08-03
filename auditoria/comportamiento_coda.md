# Auditoría de comportamiento — CODA-Prompt

## Fuentes y criterio

Se compara el paper `2211.13218v2`, el repo oficial `6417d4f68754be68b697c7ca2323ee61e791e1a3` y Mammoth base `e75a491c69fd729edeb01431afb753d9157d9a81` en la rama `tfg-auditoria`. Este fichero documenta conducta; no corrige código ni resuelve valores de cubo A/EJE.

## C-01 · Scheduler declarado frente a scheduler efectivo

1. **Descripción.** El paper prescribe LR con decay coseno, y el repo oficial ejecuta un coseno reiniciado por tarea. Mammoth construye por tarea un `CosineSchedule(K=n_epochs)`, pero el dispatcher nunca lo selecciona: el LR efectivo queda constante. La validación `K>1` sí se ejecuta, por lo que `n_epochs=1` aborta aunque el scheduler sea inerte.
2. **Evidencia.** Paper: Ap. A, `papers/coda-prompt/source_v2/sections/7_appendix.tex:5,7`, PDF p. 12; no publica T, fórmula, step, scope ni warmup. Repo: `codaprompt/configs/cifar-100_prompt.yaml:4-6`; `codaprompt/learners/default.py:81-97`; `codaprompt/learners/prompt.py:72-76`; `codaprompt/utils/schedulers.py:47-57`. Mammoth: `mammothV2/models/coda_prompt.py:42,65-73`; `mammothV2/utils/training.py:239-263`; `mammothV2/utils/schedulers.py:13-54` en `M@e75a491`.
3. **Configurable en Mammoth.** `n_epochs` sí; K se deriva automáticamente. T separado, fórmula, warmup y el cableado del scheduler son `NO_CONFIGURABLE`. `--lr_scheduler` no es alternativa: cualquier valor falla el assert de `models/coda_prompt.py:42`.
4. **Flag S.** **ALTA**. Los autores atribuyen parte de la mejora al LR decreciente y Mammoth cambia la conducta a LR constante. También acopla la validez de arranque a `n_epochs>1`.

Detalle de alcance:

- Paper: 20 épocas/tarea en CIFAR-100 y 50 en ImageNet-R; T/scope no publicados.
- Repo: K es el mismo `schedule[-1]` que cuenta épocas; se recrean optimizer+scheduler por tarea. Hace step al comienzo de épocas `epoch>0`, por lo que las épocas 0 y 1 usan LR base y la última época usa el punto K−2.
- Mammoth: K se deriva de `n_epochs` y el objeto se recrea por tarea, pero el bucle recibe `None`. No hay warmup conectado en ninguna de las dos implementaciones.

## C-02 · `virtual_bs_iterations` no implementa acumulación

1. **Descripción.** Paper y repo oficial no ofrecen acumulación. Mammoth expone `--virtual_bs_iterations`, pero para N>1 borra el gradiente en cada minibatch, divide la pérdida por N y solo actualiza en ciertos índices; descarta los demás gradientes y no hace flush.
2. **Evidencia.** Paper: `NO_ENCONTRADO` tras búsqueda en los 18 `.tex`. Repo: actualización por minibatch en `codaprompt/learners/prompt.py:25-44`. Mammoth: parser y update en `mammothV2/models/coda_prompt.py:31-32,75-90`; reset de contador por época en `mammothV2/models/utils/continual_model.py:336-343,445-446` (`M@e75a491`).
3. **Configurable en Mammoth.** El entero N sí; la semántica defectuosa es `NO_CONFIGURABLE`. Solo el default N=1 equivale al entrenamiento ordinario.
4. **Flag S.** **ALTA**. Usar la opción como acumulación cambia el número y la escala de updates de forma distinta a lo anunciado.

## C-03 · Inicialización ortogonal cambiada después del paper

1. **Descripción.** El paper describe inicialización ortogonal al comienzo. El repo auditado y Mammoth preasignan el pool, pero reinicializan mediante Gram–Schmidt únicamente la partición nueva al empezar cada tarea. Es un cambio autoral explícitamente explicado, no un conflicto silencioso.
2. **Evidencia.** Paper: Sec. 4.3, `papers/coda-prompt/source_v2/sections/4_method.tex:47-54`, PDF p. 5. Repo: explicación `codaprompt/README.md:16-17` y código `codaprompt/models/zoo.py:21-39,51-71,75-131` (`R@6417d4f`). Mammoth: comentarios y código `mammothV2/models/coda_prompt_utils/model.py:19-37,49-69`; Gram–Schmidt en `mammothV2/models/coda_prompt_utils/__init__.py:27-71` (`M@e75a491`).
3. **Configurable en Mammoth.** `NO_CONFIGURABLE`.
4. **Flag S.** **ALTA**. Está acoplado a la eliminación de la penalización λ y la ortogonalidad muestra una ablation grande en el paper.

## C-04 · Penalización ortogonal: fórmula y doble peso Mammoth

1. **Descripción.** El paper escribe una norma-2 y λ=0.1. El repo usa media del error cuadrático y λ=0.0 en la receta actual. Mammoth replica esa fórmula pero añade dos multiplicadores consecutivos: `ortho_mu` dentro del prompt y `mu` fuera; el coeficiente efectivo es `mu×ortho_mu`, por defecto 0×0.
2. **Evidencia.** Paper: `papers/coda-prompt/source_v2/sections/4_method.tex:49-66` y `sections/7_appendix.tex:9`, PDF pp. 5-6, 12; ablation `tables/ablations.tex:11-15`, PDF p. 8. Repo: `codaprompt/models/zoo.py:178-198`; `codaprompt/experiments/cifar-100.sh:23-30`; explicación `README.md:16-17`. Mammoth: `mammothV2/models/coda_prompt.py:27-30,78-83`; `mammothV2/models/coda_prompt_utils/model.py:116-137`.
3. **Configurable en Mammoth.** Ambos escalares son CLI, pero la doble multiplicación y la forma de la penalización son `NO_CONFIGURABLE`. Cambiar solo uno no activa la regularización si el otro sigue en cero.
4. **Flag S.** **ALTA**. Ablacionar ortogonalidad reduce `A_N` de 75.45 a 70.66 en ImageNet-R 10-task; además, el grupo mezcla paper, repo y Mammoth.

## C-05 · Checkpoint hardcodeado y `--backbone` descartado

1. **Descripción.** Mammoth construye primero el backbone del dataset, CODA lo descarta y crea un ViT interno con checkpoint hardcodeado. No se resuelve la ambigüedad entre el texto del paper, la llamada genérica del repo y el identificador Mammoth. Por la misma razón, `code_optimization=3` compila el backbone externo antes de que CODA lo descarte, no el ViT interno efectivo.
2. **Evidencia.** Paper, cita literal: `papers/coda-prompt/source_v2/sections/5_experiments.tex:8`, PDF p. 6. Repo: `codaprompt/models/zoo.py:363-371`; entorno inestable en `requirements.txt:1` e `install-requirements.sh:5`. Mammoth: `mammothV2/models/coda_prompt.py:35-48`; `mammothV2/models/coda_prompt_utils/model.py:159-169`; identificador `vit_base_patch16_224.augreg_in21k_ft_in1k`.
3. **Configurable en Mammoth.** `NO_CONFIGURABLE`; `--backbone` no cambia el backbone interno efectivo. `code_optimization` sí es CLI, pero su objetivo de compilación queda fijado por el orden de construcción (`main.py:482-528`).
4. **Flag S.** **ALTA** por la ambigüedad conocida y el impacto del preentrenamiento.

## C-06 · `model_config best` cambia el preprocesado

1. **Descripción.** La receta Mammoth reproducible depende de pedir `--model_config best`. Ese preset fuerza `dataset_config: coda_prompt` y normalización identidad; el `model_config default` no tiene receta CODA y el YAML default del dataset usa estadísticas ImageNet. La CLI `--dataset_config` no prevalece frente al selector incluido por el modelo.
2. **Evidencia.** `mammothV2/models/config/coda_prompt.yaml:1-8`; `datasets/configs/seq-cifar100-224/coda_prompt.yaml:5-23`; `datasets/configs/seq-cifar100-224/default.yaml:5-24`; precedencia en `main.py:169-186,308-351`; carga en `models/utils/__init__.py:60-82` (`M@e75a491`). Paper: [0,1] en `sections/7_appendix.tex:5`. Repo: identidad en `codaprompt/dataloaders/utils.py:33-62`.
3. **Configurable en Mammoth.** Selección `model_config` sí; el override de `dataset_config` por el modelo forma parte de la resolución. Interpolación del YAML CODA queda al default de torchvision y no está fijada por CLI.
4. **Flag S.** **ALTA**. Ejecutar sin `best` cambia datos efectivos y no reproduce la receta registrada.

## C-07 · Orden de clases, semillas y número de trials

1. **Descripción.** El paper agrega cinco shuffles y advierte que el orden cambia el resultado. El script oficial auditado usa una sola repetición con seed 0, caso en que su condición evita el shuffle. La receta Mammoth canónica usa seed no fijada y orden nativo.
2. **Evidencia.** Paper: `papers/coda-prompt/source_v2/sections/5_experiments.tex:14`; `sections/7_appendix.tex:25`; tablas CIFAR/ImageNet-R, PDF pp. 6-7, 12, 14. Repo: `codaprompt/experiments/cifar-100.sh:9-14`; `run.py:119-133`; `trainer.py:58-76`. Mammoth: `mammothV2/utils/args.py:274-281,359-362`; `datasets/utils/continual_dataset.py:212-232`; comando `scripts/reproduce.json:49-52`.
3. **Configurable en Mammoth.** Seed y `permute_classes` sí; el preset no los fija. Las cinco ejecuciones deben orquestarse externamente.
4. **Flag S.** **ALTA** para orden y **MEDIA** para seed/trials. El propio paper aporta evidencia de impacto.

## C-08 · Controles de orden explícito defectuosos en Mammoth

1. **Descripción.** El camino estándar `permute_classes+seed` funciona, pero `custom_task_order` construye `range(tuple)` y `custom_class_order` no activa la permutación; combinado con ella intenta indexar una lista con un array NumPy.
2. **Evidencia.** Mammoth `datasets/utils/continual_dataset.py:212-232,456-459,541-555` (`M@e75a491`). Paper y repo no exponen listas explícitas equivalentes; el repo genera el orden mediante seed.
3. **Configurable en Mammoth.** Los argumentos existen, pero los caminos descritos no ofrecen un control funcional fiable en este SHA. La receta canónica no entra en ellos.
4. **Flag S.** **MEDIA** si la revisión decide reproducir órdenes exactos del paper; sin esa necesidad, el riesgo queda fuera del camino canónico.

## C-09 · Tratamiento del último lote

1. **Descripción.** El repo descarta el último lote incompleto; Mammoth no. Con 5000 imágenes por tarea y batch 128 son 39 frente a 40 lotes por época, y 7800 frente a 8000 pasos en CIFAR completo a 20 épocas.
2. **Evidencia.** Paper: `drop_last NO_ENCONTRADO`; batch 128 en `sections/7_appendix.tex:5`. Repo: `codaprompt/trainer.py:184-198`. Mammoth: `mammothV2/utils/args.py:320-321`; `datasets/utils/continual_dataset.py:530-534`; tareas/batch/épocas en el YAML CODA.
3. **Configurable en Mammoth.** Sí, `--drop_last`; default 0.
4. **Flag S.** **MEDIA**. Cambia coste y conjunto de muestras usado en cada época.

## C-10 · Expansión conceptual frente a preasignación física

1. **Descripción.** El paper describe expansión de M/N componentes por tarea y una cabeza expansiva. Repo y Mammoth preasignan desde el inicio el pool completo y las 100 salidas, y solo controlan slices/gradientes. Necesitan conocer N para `floor(M/N)`; cualquier resto queda sin usar.
2. **Evidencia.** Paper: `sections/4_method.tex:44-49,57-66`; “single, expanding classification head” en `sections/1_intro.tex:15` y protocolo en `sections/3_prelim.tex:5`. Repo: `codaprompt/models/zoo.py:13-39,94-161,353-375`; `trainer.py:72-78,124-127`. Mammoth: `mammothV2/models/coda_prompt_utils/model.py:9-37,57-69,82-99,171-174`.
3. **Configurable en Mammoth.** Pool y longitud sí; capas, estrategia física, floor y cabeza fija son `NO_CONFIGURABLE`.
4. **Flag S.** **MEDIA** por acoplamiento entre número de tareas, capacidad útil, inicialización y coste.

## C-11 · Reset del optimizador y `optim_mom` no consumido

1. **Descripción.** Repo y Mammoth reinician Adam por tarea. En Mammoth, `optim_mom=0.9` aparece en el preset pero el constructor Adam no lo pasa; incluso al elegir SGD, no pasa momentum ni Nesterov.
2. **Evidencia.** Paper: estado entre tareas `NO_ENCONTRADO`; betas con errata en `sections/7_appendix.tex:5`. Repo: `codaprompt/learners/default.py:81-84,238-261`; `learners/prompt.py:47-76`. Mammoth: `mammothV2/models/config/coda_prompt.yaml:6-8`; `models/coda_prompt.py:24-26,52-73`; wrapper `models/utils/continual_model.py:449-475`.
3. **Configurable en Mammoth.** Tipo, LR, WD y `optim_mom` son CLI; reset y parámetros realmente pasados son hardcodeados. `optim_mom` no modifica este optimizador.
4. **Flag S.** **MEDIA** para reset; **BAJA** para `optim_mom` engañoso.

## C-12 · Métricas auxiliares no equivalentes

1. **Descripción.** El paper reporta `A_N` y `F_N`. El repo principal no calcula forgetting y su salida `pt-local` excluye la última clase de cada tarea por un límite estricto. Mammoth calcula Class-IL y Task-IL correctamente, pero forgetting/BWT/FWT solo al activar `enable_other_metrics`.
2. **Evidencia.** Paper: `sections/5_experiments.tex:18-19`; `sections/7_appendix.tex:21-23`. Repo: global/local en `codaprompt/learners/default.py:160-203`; bug `:181-196`; agregación `trainer.py:223-247`; utilitario aislado `utils/calc_forgetting.py:1-24`. Mammoth: `mammothV2/utils/evaluate.py:11-109`; `utils/args.py:382-383`; `utils/training.py:353-363`.
3. **Configurable en Mammoth.** Métricas adicionales sí mediante flag; Class-IL/Task-IL base no. El bug `pt-local` pertenece al repo oficial, no a Mammoth.
4. **Flag S.** **MEDIA** si se comparan métricas del paper; la accuracy Class-IL principal no depende del bug local del repo.

## C-13 · `fitting_mode=time` no tiene condición de terminación

1. **Descripción.** Mammoth acepta `fitting_mode=time`, pero no define presupuesto temporal ni una rama de salida para ese modo. El bucle `while True` solo termina por épocas, iteraciones o early stopping; con `time` continúa hasta interrupción externa. La receta canónica usa `epochs` y no entra en esta ruta.
2. **Evidencia.** Paper: modos alternativos `NO_ENCONTRADO` en Ap. A, `papers/coda-prompt/source_v2/sections/7_appendix.tex:5,7`. Repo: duración por `schedule[-1]`, sin modo time, `codaprompt/learners/default.py:92-110`. Mammoth: parser `mammothV2/utils/args.py:290-307`; bucle y únicas condiciones de salida `mammothV2/utils/training.py:257-299` (`M@e75a491`).
3. **Configurable en Mammoth.** El selector sí; el presupuesto/criterio de salida para `time` es `NO_CONFIGURABLE` porque no existe. Además CODA sigue accediendo a `n_epochs` para construir K.
4. **Flag S.** **MEDIA**. Es una ruta no canónica, pero seleccionarla puede producir entrenamiento no terminante y coste sin cota.

## C-14 · ImageNet-R redimensiona distinto en repo y Mammoth

1. **Descripción.** En train, el repo usa RRC224 con interpolación default y Mammoth fija bicúbica. En test, `Resize(256)` del repo preserva la relación de aspecto al llevar el lado corto a 256; Mammoth fuerza `Resize((256,256), BICUBIC)` antes de CenterCrop224. Es una diferencia incidental de preprocesado, no una receta Mammoth `best`.
2. **Evidencia.** Paper: solo resize 224×224 y rango [0,1], Ap. A `papers/coda-prompt/source_v2/sections/7_appendix.tex:5`; interpolación/crop `NO_ENCONTRADO`. Repo: `codaprompt/dataloaders/utils.py:42-62` (`R@6417d4f`). Mammoth: `mammothV2/datasets/seq_imagenet_r.py:112-132` (`M@e75a491`).
3. **Configurable en Mammoth.** Estos transforms de `SequentialImagenetR` son `NO_CONFIGURABLE` por la CLI de CODA en este SHA. No existe preset `best` para ImageNet-R; la ruta `default` sí resuelve los defaults de clase.
4. **Flag S.** **MEDIA**. Cambia geometría e interpolación de la entrada; debe decidirse antes de comparar ImageNet-R, aunque no se buscó una receta adicional.

## Comprobaciones obligatorias sin discrepancia demostrada

- **Tipo de inserción:** las tres fuentes describen/implementan prefix-tuning sobre K/V. Paper `sections/3_prelim.tex:10-29`; repo `models/vit.py:63-75`; Mammoth `models/coda_prompt_utils/vit.py:41-64`. No se crea entrada de discrepancia.
- **Query y composición:** paper, repo y Mammoth coinciden en CLS del ViT, atención elemento a elemento, coseno y suma ponderada. El número exacto de forwards solo es explícito en código; se conserva `NO_ENCONTRADO` para el paper.
- **Enmascarado de logits:** las tres fuentes coinciden en excluir logits pasados durante entrenamiento y evaluar Class-IL entre clases vistas. Se propone `S ALTA` por la evidencia de impacto del paper, no por conflicto.
- **Task-id de inferencia:** el protocolo paper no entrega task-id; CODA en repo y Mammoth no usa el argumento `task_id` para elegir componentes. La métrica Task-IL de Mammoth es una máscara oracle externa, no la conducta principal del método.
- **Backbone actualizado:** repo y Mammoth lo excluyen del optimizador; el paper dice que queda sin modificar. Que el grafo del segundo forward atraviese el backbone aumenta coste, pero no demuestra actualización de pesos.

## Ausencias de comportamiento comprobadas

No se hallaron en la receta CODA del paper/repo: warmup, early stopping, AMP, gradient clipping, label smoothing, mixup o cutmix. Mammoth ofrece controles genéricos para algunos modos de duración, pero no forman parte de su preset CODA y no se presentan como conducta del paper.
