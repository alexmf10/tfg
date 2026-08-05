# Cola de revisión — CODA-Prompt

Claves de evidencia: paper `2211.13218v2`; `R@6417d4f` = repo oficial `6417d4f68754be68b697c7ca2323ee61e791e1a3`; `M@e75a491` = implementación Mammoth base `e75a491c69fd729edeb01431afb753d9157d9a81` (la rama auditada está en `cda7f23681f7bffacee460d99e990bc803bccf04`).

## Cierre de Fase B — 2026-08-05

El ledger siguiente es el estado vigente; las tablas posteriores conservan la redacción de entrada a reunión como evidencia histórica. Las betas y la fórmula ortogonal permanecen `REVISAR-MAESTRO` aunque su entrada de cola quede administrativamente cerrada.

| entradas | cierre |
|---|---|
| S-01 | **CERRADO→D41-B1/B6/D31/D15/D39/D40**. |
| S-02 | **CERRADO→D41-B2/B5**. |
| S-03 | **CERRADO→D41-B4/B6**; fórmula inerte con pesos cero. |
| S-04 | **CERRADO→D31/D41-a**. |
| S-05 | **CERRADO→D41-d/D41-B5**. |
| S-06 | **CERRADO→D19/D41-B3**. |
| S-07 | **CERRADO→D41-B2**. |
| S-08 | **CERRADO→D41-B2/B4/B6**. |
| S-09 | **CERRADO→D19**. |
| S-10, S-11 | **CERRADO→D41-B2**. |
| S-12 | **CERRADO→D41-B1/D25/D28**. |
| S-13 | **CERRADO→D41-B4/B2**; el literal de betas queda `REVISAR-MAESTRO`. |
| S-14 | **CERRADO→D41-B2/D25**. |
| S-15 | **CERRADO→D41-B4**. |
| S-16 | **CERRADO→D41-B5**. |
| S-17 | **CERRADO→D41-B2**. |
| S-18 | **CERRADO→D41-B3/D39**. |
| S-19, S-20 | **CERRADO→D41-B2/B4**. |
| Q-01 | **CERRADO→D41-B1/B6/D31/D15/D39/D40**. |
| Q-02, Q-03 | **CERRADO→D41-B4/B6**. |
| Q-04 | **CERRADO→D39**. |
| Q-05 | **CERRADO→D31/D41-a**. |
| Q-06 | **CERRADO→D41-B5**. |
| Q-07 | **CERRADO→D41-B4**, sin convertir la errata del paper en betas ejecutables. |
| Q-08 | **CERRADO→D41-B4**; diferencia de fórmula inerte con pesos cero. |
| Q-09 | **CERRADO→D19**. |
| Q-10 | **CERRADO→D41-B5/§9**. |
| Q-11 | **CERRADO→D41-B3**. |
| Q-12 | **CERRADO→D41-B5**. |
| Q-13 | **CERRADO→D41-B2/B3**, según control. |
| G-OPT | **CERRADO→D41-B1/B6/D31/D15/D39/D40**. |
| G-PROMPT, G-ORTHO | **CERRADO→D41-B4/B6**. |

## Cola de sensibilidad S

Las entradas están consolidadas por parámetro o grupo acoplado; no hay tope. Una entrada agrupada cubre todas las filas marcadas con esa S en `valores_coda.md`.

| severidad | parámetro / grupo | método y ámbito | justificación (1-2 líneas) | evidencia |
|---|---|---|---|---|
| ALTA | S-01 · G-OPT: LR, scheduler, `n_epochs`, T/K, scope y reset | CODA-Prompt · ambos | El paper atribuye mejora al LR reducido y decreciente. Mammoth deriva K de `n_epochs`, K=1 aborta y, aun así, el custom scheduler no llega al bucle: LR efectivo constante. | Paper `sections/7_appendix.tex:5,7`, PDF p. 12; repo `configs/cifar-100_prompt.yaml:4-10`, `learners/default.py:81-97`, `utils/schedulers.py:47-57`; Mammoth `models/coda_prompt.py:42,65-73`, `utils/training.py:239-263`, `utils/schedulers.py:13-54`. |
| ALTA | S-02 · arquitectura y checkpoint del backbone | CODA-Prompt · ambos | Mammoth hardcodea tanto ViT-B/16 como sus pesos internos; `--backbone` se descarta. Las fuentes no identifican un mismo artefacto de pesos de manera demostrable. | Paper `sections/3_prelim.tex:15,29`, cita de checkpoint en `sections/5_experiments.tex:8`; repo `models/zoo.py:353-371`, `requirements.txt:1`, `install-requirements.sh:5`; Mammoth `models/coda_prompt.py:35-48`, `models/coda_prompt_utils/model.py:152-177`, `requirements.txt:6`. |
| ALTA | S-03 · G-ORTHO: inicialización, fórmula y pesos | CODA-Prompt · ambos | El paper usa λ=0.1 y reporta una caída de 4.79 puntos al ablar ortogonalidad; repo cambia a GS online y λ=0 con explicación; Mammoth añade el producto `mu×ortho_mu`. | Paper `sections/4_method.tex:47-66`, `sections/7_appendix.tex:9`, `tables/ablations.tex:11-15`; repo `README.md:16-17`, `models/zoo.py:21-39,51-71,178-198`; Mammoth `models/coda_prompt.py:27-30,78-83`, `models/coda_prompt_utils/model.py:19-69,116-137`. |
| ALTA | S-04 · acumulación / `virtual_bs_iterations` | CODA-Prompt · ambos | Para N>1 Mammoth borra los gradientes en cada minibatch, divide la pérdida y omite updates; la opción del método no implementa el virtual batch que su nombre sugiere. | Mammoth `models/coda_prompt.py:31-32,75-90`; `models/utils/continual_model.py:336-343,445-446`. Repo sin acumulación: `learners/prompt.py:25-44`; paper `NO_ENCONTRADO` en Ap. A `sections/7_appendix.tex:5`. |
| ALTA | S-05 · selección `model_config` y normalización | CODA-Prompt · CIFAR-100 | `best` fuerza el YAML CODA con rango [0,1]; sin él se usa otra configuración con stats ImageNet. La ruta de config cambia el input efectivo. | Mammoth `models/config/coda_prompt.yaml:1-8`; `datasets/configs/seq-cifar100-224/coda_prompt.yaml:5-23`; `datasets/configs/seq-cifar100-224/default.yaml:5-24`; `main.py:175-186`. |
| ALTA | S-06 · orden de clases | CODA-Prompt · ambos | El paper declara que distintos shuffles cambian la dificultad y los resultados; repo actual con repeat 1/seed 0 y Mammoth canónico usan orden nativo. Las rutas custom de orden son defectuosas; C-08 las clasifica S MEDIA de forma condicional. | Paper `sections/5_experiments.tex:14`, `sections/7_appendix.tex:25`; repo `experiments/cifar-100.sh:9-14`, `trainer.py:58-76`; Mammoth `utils/args.py:274-281,359-362`, `datasets/utils/continual_dataset.py:212-232`; `comportamiento_coda.md` C-08. |
| ALTA | S-07 · enmascarado de logits | CODA-Prompt · ambos | El paper lo califica de “crucial”; desactivarlo cambia CE de tarea actual por una cabeza sesgada hacia clases nuevas. En Mammoth está hardcodeado. | Paper `sections/7_appendix.tex:11`; repo `learners/prompt.py:25-37`; Mammoth `models/coda_prompt.py:75-83`. |
| ALTA | S-08 · G-PROMPT: componentes, longitud, capas y partición | CODA-Prompt · ambos | El paper muestra retornos fuertes al escalar componentes y fija 100/8/capas 1-5. Capas, prefix y partición floor están hardcodeados en Mammoth. | Paper `sections/5_experiments.tex:11-12`, `sections/7_appendix.tex:9`, `sections/4_method.tex:44-49`; repo `models/zoo.py:41-49,94-176`; Mammoth `models/coda_prompt_utils/model.py:39-47,82-114`. |
| MEDIA | S-09 · seed y cinco trials | CODA-Prompt · ambos | El paper agrega cinco trials con seeds consistentes; repo ejecuta uno y Mammoth no fija seed en la receta canónica. Afecta inicialización, shuffle y augment. | Paper `tables/cifar100_domainnet.tex:2`, `tables/imagenet-r.tex:2`, `sections/7_appendix.tex:25`; repo `run.py:119-133`; Mammoth `scripts/reproduce.json:49-52`, `utils/conf.py:169-223`. |
| MEDIA | S-10 · batch tail / `drop_last` | CODA-Prompt · ambos | Repo descarta el remanente y Mammoth no: con CIFAR son 39 frente a 40 lotes por época y 7800 frente a 8000 pasos totales. | Repo `trainer.py:184-198`; Mammoth `utils/args.py:320-321`, `datasets/utils/continual_dataset.py:530-534`; batch/épocas/tareas en los YAML citados. |
| MEDIA | S-11 · cabeza física y offsets | CODA-Prompt · ambos | Paper describe cabeza expansiva; ambas implementaciones preasignan todas las salidas y amplían solo la vista. La estrategia y la inicialización futura no son configurables. | Paper `sections/3_prelim.tex:5`, `sections/4_method.tex:62-66`; repo `models/zoo.py:353-375`, `learners/default.py:286-291`; Mammoth `models/coda_prompt_utils/model.py:171-174`, `models/coda_prompt.py:65-95`. |
| MEDIA | S-12 · métricas y agregación | CODA-Prompt · ambos | Paper reporta `A_N` y `F_N`; repo no integra forgetting y su métrica local excluye una clase; Mammoth no activa forgetting por defecto. | Paper `sections/5_experiments.tex:18-19`, `sections/7_appendix.tex:21-23`; repo `learners/default.py:181-196`, `trainer.py:223-247`; Mammoth `utils/args.py:382-383`, `utils/training.py:353-363`. |
| MEDIA | S-13 · betas Adam / `optim_mom` | CODA-Prompt · ambos | El paper repite β1 y no da β2; el repo usa (0.9,0.999). Mammoth configura `optim_mom=0.9` pero su Adam custom no lo consume. | Paper `sections/7_appendix.tex:5`; repo `configs/cifar-100_prompt.yaml:10`, `learners/default.py:238-255`; Mammoth `models/config/coda_prompt.yaml:6-7`, `models/coda_prompt.py:52-63`. |
| MEDIA | S-14 · query, atención y forward extra | CODA-Prompt · ambos | Repo y Mammoth ejecutan dos forwards completos y usan cosenos crudos sin softmax/top-k; la conducta y el coste no son configurables. El paper solo deja indicio del doble uso del encoder. | Paper `figures/method_fig.tex:5`, `sections/4_method.tex:23-42`; repo `models/zoo.py:163-171,389-405`; Mammoth `models/coda_prompt_utils/model.py:101-109,180-197`. |
| MEDIA | S-15 · sweep y transferencia de HP | CODA-Prompt · ambos | Los rangos solo están en el paper y los HP CIFAR se transfieren desde ImageNet-R; repo/Mammoth solo conservan una receta final. Repetir tuning cambia coste y selección. | Paper `sections/7_appendix.tex:7,9`, `sections/5_experiments.tex:32`; repo/Mammoth `NO_ENCONTRADO` para script de sweep CODA equivalente. |
| MEDIA | S-16 · augmentación y preprocesado | CODA-Prompt · ambos | Paper no fija augment/interpolación. Los pipelines son parcialmente hardcodeados; en ImageNet-R repo preserva aspecto al redimensionar y Mammoth fuerza 256×256 bicúbico. | Paper `sections/7_appendix.tex:5`; repo `dataloaders/utils.py:28-65`; Mammoth YAML CODA y `datasets/seq_imagenet_r.py:112-132`. |
| MEDIA | S-17 · parámetros entrenables / congelación | CODA-Prompt · ambos | La selección prompt+cabeza y exclusión del backbone no es configurable en Mammoth; el segundo forward todavía atraviesa el backbone y afecta coste. | Paper `sections/3_prelim.tex:29`; repo `learners/prompt.py:47-70`, `models/zoo.py:389-405`; Mammoth `models/coda_prompt.py:52-63`, `models/coda_prompt_utils/model.py:180-195`. |
| MEDIA | S-18 · fitting alternativo / modo `time` | CODA-Prompt · ambos | Mammoth acepta `time`, pero no hay presupuesto ni condición de salida; el bucle queda no terminante. Los modos alternativos tampoco adaptan K del scheduler CODA. | Mammoth `utils/args.py:290-307`; `models/coda_prompt.py:73`; `utils/training.py:257-299`; paper Ap. A `sections/7_appendix.tex:5,7` y repo `learners/default.py:92-110` sin modo temporal. |
| BAJA | S-19 · pool no divisible por tareas | CODA-Prompt · ambos | Repo y Mammoth usan `floor(M/N)` sin validar; al cambiar pool o número de tareas, el resto queda reservado pero nunca se usa. | Repo `models/zoo.py:94-103,141-161`; Mammoth `models/coda_prompt_utils/model.py:17,57-59,82-85`. |
| BAJA | S-20 · pérdida de clasificación | CODA-Prompt · ambos | El paper no concreta el tipo; repo selecciona CE y Mammoth la toma del dataset sin opción CLI para CODA. Se registra por el criterio de valor no configurable, con severidad baja. | Paper `sections/4_method.tex:19,59-66`; repo `learners/default.py:42-43,145-158`; Mammoth `datasets/seq_cifar100_224.py:99-101`, `models/coda_prompt.py:75-92`. |

## Cola de revisión por cascada y grupos

| prioridad | elemento | disparador | motivo de entrada a la reunión | evidencia mínima |
|---|---|---|---|---|
| ALTA | Q-01 · G-OPT | `GRUPO_MIXTO` | Betas se remiten literalmente al paper ambiguo; weight decay y reset proceden solo del repo; batch y duración son A/EJE sin propuesta. No se resuelve la mezcla. | Filas A10, E01-E12 y B01-B04/B06 de `valores_coda.md`; OBS-05. |
| ALTA | Q-02 · G-ORTHO | `GRUPO_MIXTO` + CASO 3 | GS online y λ=0 vienen de la explicación autoral; la fórmula queda en paper; `mu` solo existe en Mammoth. | Filas B20-B23; paper `sections/4_method.tex:47-66`; repo `README.md:16-17`; Mammoth doble peso. |
| ALTA | Q-03 · G-PROMPT | `GRUPO_MIXTO` | Sweep solo paper; doble forward, coseno sin softmax y preasignación física solo repo; los valores comunes no eliminan esa mezcla. | Filas B09-B19/B24; OBS-06. |
| ALTA | Q-04 · scheduler Mammoth | diferencia de comportamiento | El objeto custom existe y valida K, pero el bucle no lo usa. Revisar antes de cualquier ejecución de sensibilidad o comparación. | `mammothV2/models/coda_prompt.py:65-73`; `utils/training.py:239-263`; `utils/schedulers.py:13-54`. |
| ALTA | Q-05 · acumulación Mammoth | conducta incompatible con el nombre | N>1 no acumula; no usar como equivalencia de batch sin decisión humana/corrección posterior. | `mammothV2/models/coda_prompt.py:75-90`; `models/utils/continual_model.py:336-343,445-446`. |
| ALTA | Q-06 · checkpoint | cubo A / ambigüedad explícita | Paper, repo y Mammoth no identifican el mismo artefacto de manera demostrable. La Fase A no elige. | Fila A02; OBS-01. |
| MEDIA | Q-07 · betas Adam | CASO 3 CONFLICTO→PAPER | El literal del paper no forma un par ejecutable; queda `REVISAR` y no se corrige la segunda β1 a β2. | Fila B02; OBS-08. |
| MEDIA | Q-08 · fórmula ortogonal | CASO 3 CONFLICTO→PAPER | Norma-2 del paper frente a MSE medio del código, sin explicación autoral localizada. | Fila B21. |
| MEDIA | Q-09 · orden/seeds/trials | cubo A + S | Elegir un protocolo de cinco órdenes/seeds corresponde a revisión humana; no se rellena propuesta. | Filas A05-A06; OBS-02. |
| BAJA | Q-10 · ImageNet-R Mammoth | lectura incidental sin preset `best` | La ruta `default` es resoluble, pero no existe preset `best` ni receta de reproducción equivalente publicada; decidir su autoridad corresponde a revisión humana. | Filas A03/E01; OBS-09. |
| MEDIA | Q-11 · `fitting_mode=time` | diferencia de comportamiento | El parser lo acepta, pero el bucle carece de límite temporal y no termina por ese modo. No usarlo como presupuesto antes de revisión. | Fila E10; `mammothV2/utils/args.py:290-307`; `utils/training.py:257-299`. |
| MEDIA | Q-12 · preprocesado ImageNet-R | diferencia incidental | Repo preserva aspecto en Resize256 y Mammoth fuerza 256×256 bicúbico; no se elige uno en Fase A. | Filas A07-A09; `codaprompt/dataloaders/utils.py:42-62`; `mammothV2/datasets/seq_imagenet_r.py:112-132`. |
| BAJA | Q-13 · controles `LOW` fuera de la receta canónica | criterio de fila en duda | Evaluación futura/frecuencia, precisión, fracción etiquetada, ruido, joint, inference-only y límites de tareas pueden alterar resultados o coste; sus defaults quedan documentados, no elegidos. | Filas A13, A15, A17-A20 y E11-E12 de `valores_coda.md`. |

## Estado de `GRUPO_MIXTO`

- **G-OPT — CERRADO→D41-B1/B6/D31/D15/D39/D40.**
- **G-PROMPT — CERRADO→D41-B4/B6.** La coincidencia de 100/8/capas 1-5 se combina con las decisiones solo-paper/solo-repo ya registradas.
- **G-ORTHO — CERRADO→D41-B4/B6.** El valor 0 hace inerte la diferencia de fórmula; B21 permanece `REVISAR-MAESTRO` para no confundir fórmula ejecutada y literal del paper.

## Lista agrupada de infraestructura

Se conserva la misma agrupación normativa de `valores_coda.md`: 19 controles de dispositivo/distribución, rutas/artefactos, operación del proceso, selección nominal y diagnóstico/portabilidad. Ninguno entra en la cola S solo por ser infraestructura. `code_optimization` no se agrupa: permanece como fila A17 con `LOW` porque puede cambiar coste y resultados numéricos.

## Informe de embudo

| etapa | cantidad | desglose / motivo |
|---|---:|---|
| parámetros examinados | 90 | 56 con fila + 19 agrupados como infraestructura + 15 excluidos, misma unidad conceptual que `valores_coda.md`. |
| parámetros con fila | 56 | 20 cubo A + 12 EJE + 24 cubo B. |
| lista agrupada | 19 | Controles sin efecto directo propio sobre la receta matemática. |
| exclusiones | 15 | 6 no-op/incompatibles/sin efecto, 5 ausentes sin valor y 4 variantes ajenas. |

### Embudo específico de sensibilidad y revisión

| etapa | cantidad | desglose / motivo |
|---|---:|---|
| filas evaluadas para S | 56 | Se aplicaron los tres criterios de sensibilidad a todas las filas, también A y EJE. |
| filas con marca S | 41 | Se consolidan cuando pertenecen al mismo parámetro o grupo acoplado. |
| entradas S consolidadas | 20 | 8 ALTA, 10 MEDIA y 2 BAJA; sin tope artificial. |
| grupos mixtos en cola | 3 | G-OPT, G-PROMPT y G-ORTHO. |
| revisiones adicionales | 10 | Scheduler, acumulación, checkpoint, dos CASO 3, orden/seeds, ImageNet-R, modo time, preprocesado incidental y controles LOW; algunas se solapan con S. |
| filas sin S | 15 | Coincidencias o ausencias sin indicio concreto de impacto y sin valor no configurable que justifique sensibilidad. |
