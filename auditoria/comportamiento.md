# Auditoría consolidada de comportamiento

## Alcance

Fusión deduplicada de `comportamiento_l2p.md`, `comportamiento_dualprompt.md` y `comportamiento_coda.md`. Cada hallazgo conserva evidencia de paper, repo oficial y Mammoth cuando existe. Las diferencias se documentan; no se corrigen ni se convierten en valores finales.

Claves: `R-L2P/DP@dd8836e6`, `R-CODA@6417d4f`, `M@e75a491` y rama auditada Mammoth `cda7f236`.

## CB-01 · Backbone y checkpoint internos; el argumento nominal se descarta

- **Conducta común.** L2P, DualPrompt y CODA reciben un backbone del framework, lo descartan y construyen ViT internos. Por ello `--backbone` no cambia la red efectiva y una optimización aplicada antes al objeto externo puede no alcanzar el modelo usado.
- **L2P.** Paper: ViT-B/16 preentrenado sin fichero exacto (`L2P_arxiv.tex:442-443`). Repo: `ViT-B_16.npz` (`README.md:50-52,59-64`). Mammoth: `del backbone`, alias `vit_base_patch16_224_in21k_fn_in1k_old`; la rama manual comprueba y abre nombres incompatibles (`models/l2p.py:52-67`; `models/l2p_utils/l2p_model.py:39-46`).
- **DualPrompt.** Paper: ViT-B/16 congelado (`dual_prompt_camera_ready.tex:1317`). Repo: mismo NPZ (`README.md:50-52,69-74`). Mammoth: descarte y dos ViT internos (`models/dualprompt.py:61-74`; `models/dualprompt_utils/model.py:12-46`). El override textual de `freeze` puede convertir elementos en listas de caracteres y abortar (`models/dualprompt.py:61`; `model.py:48-54`).
- **CODA-Prompt.** Paper: ViT-B/16 preentrenado en ImageNet-1K (`sections/5_experiments.tex:8`). Repo: llamada timm genérica (`models/zoo.py:363-371`). Mammoth: identificador fijo `vit_base_patch16_224.augreg_in21k_ft_in1k` (`models/coda_prompt.py:35-48`; `models/coda_prompt_utils/model.py:159-169`). `code_optimization=3` compila el backbone externo antes de descartarlo (`main.py:482-528`).
- **Configurable en Mammoth.** Arquitectura/checkpoint efectivos: **no**. Seleccionar la ruta manual L2P es nominalmente posible, pero está defectuosa en este SHA.
- **Sensibilidad.** ALTA: checkpoint y representación congelada condicionan todo el método.

## CB-02 · Familia de prompting, capas y semántica de longitud

- **L2P.** Las tres fuentes hacen prompt-tuning de entrada una sola vez, no prefix-tuning multicapa: paper `L2P_arxiv.tex:253-263,296-301,344-351`; repo `configs/cifar100_l2p.py:91-94`, `models/vit.py:224-253`; Mammoth `models/l2p_utils/prompt.py:119-121`, `vit_prompt.py:76-101`. Punto/familia no configurables.
- **DualPrompt.** Las tres fuentes implementan Pre-T: Q queda intacta y se anteponen prefijos a K/V (`dual_prompt_camera_ready.tex:395,405-409`; R `models/prefix_attention.py:125-136`; M `models/dualprompt_utils/attention.py:27-50`). La discrepancia es semántica: el paper define una longitud total dividida entre K/V, mientras ambos códigos crean `length` posiciones completas en cada rama; `L_g=5` deja una mitad no entera sin convención publicada.
- **CODA-Prompt.** Coincide en prefix-tuning K/V: paper `sections/3_prelim.tex:10-29`; R `models/vit.py:63-75`; M `models/coda_prompt_utils/vit.py:41-64`. Pool100, longitud8 y capas1–5 coinciden, pero las capas/estrategia son hardcodeadas en Mammoth.
- **Configurable.** Longitudes y listas DualPrompt son CLI, no su interpretación K/V. L2P no expone familia/capas. CODA expone pool/longitud, no capas ni tipo de inserción.
- **Sensibilidad.** ALTA para L2P/DualPrompt y para la geometría CODA; cambia capacidad y coste.

## CB-03 · Query con un segundo ViT, forward adicional y universo de candidatos

- **Conducta común.** Los tres métodos calculan una query CLS con un ViT sin prompts y ejecutan después otro forward con prompts. Esto duplica el backbone residente y añade un forward completo, aunque la query se calcule sin gradientes.
- **L2P.** Paper `L2P_arxiv.tex:309-323,344-351`; R `train_continual.py:249-264,375-383,969-984`; M `models/l2p_utils/l2p_model.py:15-37,126-134`.
- **DualPrompt.** Paper `dual_prompt_camera_ready.tex:318-322,1274-1304`; R `train_continual.py:249-258,375-383,969-984`; M `models/dualprompt_utils/model.py:12-65`. En test, R/M comparan contra los diez slots, incluidos futuros; el paper no fija ese universo (`models/prompt.py:177-223`; M `vision_transformer.py:118-133`).
- **CODA-Prompt.** La figura muestra la query y el encoder promptado, pero no cuenta forwards (`figures/method_fig.tex:5`); R/M ejecutan dos (`models/zoo.py:389-405`; M `models/coda_prompt_utils/model.py:180-197`).
- **Configurable.** Modo textual de query en L2P/DualPrompt: parcial. Eliminar el segundo ViT/forward o limitar candidatos futuros DualPrompt: **no**.
- **Sensibilidad.** ALTA para selección/coste; MEDIA en CODA donde el mecanismo coincide pero el coste es fijo.

## CB-04 · Posiciones, inicializaciones y transferencia de prompts/keys

- **L2P.** Repo suma posiciones antes de anteponer prompts, que no reciben `posembed_input`; Mammoth amplía `pos_embed` y suma después de concatenarlos (`R models/vit.py:198-245`; M `models/l2p_utils/vit_prompt.py:47-99`). Repo solo fija inicializador uniforme y cabeza con kernel cero; Mammoth usa `U[-1,1]` para prompt/key y trunc-normal para cabeza (`models/l2p_utils/prompt.py:22-37`; `backbone/vit.py:463-468`).
- **DualPrompt.** Repo/Mammoth copian el E-Prompt anterior al nuevo slot, no su key; el paper no lo documenta (`R train_continual.py:550-584`; M `models/dualprompt.py:80-99`). Mammoth usa `U[-1,1]` y acopla prompt/key a `prompt_key_init`; el repo solo deja un uniforme Flax sin escala local (`R models/prompt.py:192-199`; M `models/dualprompt_utils/prompt.py:25-67`).
- **CODA-Prompt.** La inicialización ortogonal se trata separadamente en CB-16; la cabeza/pool se preasignan, tratado en CB-17.
- **Configurable.** Posiciones/escala/cabeza L2P: no. Copia/no-copia y escala DualPrompt: no; tipo de inicializador, parcialmente y acoplado.
- **Sensibilidad.** MEDIA.

## CB-05 · Selección por muestra frente a voto batchwise y dominio del voto

- **L2P.** El paper selecciona por instancia (`L2P_arxiv.tex:122,313-323`). Repo y preset Mammoth `best` votan por los k IDs más frecuentes (`R configs/cifar100_l2p.py:109`, `models/prompt.py:210-216`; M `models/config/l2p.yaml:1-2`, `models/l2p_utils/prompt.py:72-85`). Repo vota por shard local de16; Mammoth por batch128 con `distributed=no`, o por shard en DataParallel (`R train_continual.py:664-716`; M `main.py:531-542`).
- **DualPrompt.** El paper selecciona por muestra en test (`dual_prompt_camera_ready.tex:1296-1304`). CIFAR oficial vota sobre grupos locales de24 y Mammoth sobre batches de hasta128; ImageNet-R oficial desactiva el voto (`R configs/{cifar100,imr}_dualprompt.py:113`, `models/prompt.py:207-216`; M `models/dualprompt_utils/prompt.py:101-115`). En train, la máscara de task-id sobrescribe el resultado.
- **CODA-Prompt.** `NO_APLICA`: compone con cosenos crudos sobre componentes visibles, no elige top-k.
- **Configurable.** `batchwise_prompt=0` sí; dominio global frente a shard no tiene selector separado.
- **Sensibilidad.** ALTA: la salida depende de compañeros, tamaño, remanente y sharding.

## CB-06 · Máscara de train, persistencia de cabeza/pool y reset del optimizador

- **L2P.** Cabeza/pool/keys persisten; CE compite solo en la tarea actual; Adam se recrea por tarea. Paper no fija máscara ni momentos; R `train_continual.py:271-286,536-539,986-1003`; M `models/l2p.py:71-94`.
- **DualPrompt.** Cabeza total persistente; R la inicializa a cero y M trunc-normal. Ambos reinician Adam y copian solo E al slot nuevo (`R models/vit.py:472-478`, `train_continual.py:536-584`; M `models/dualprompt.py:77-100`, `vision_transformer.py:97-108`).
- **CODA-Prompt.** Cabeza100 y pool completo se preasignan; offsets/particiones amplían la vista lógica. Repo/Mammoth reinician Adam por tarea; `optim_mom=0.9` del preset Mammoth no llega al constructor (`R learners/default.py:81-84,238-261`; M `models/coda_prompt.py:52-73`).
- **Configurable.** Persistencia/máscara/reset: **no**. Inicialización de cabeza: no.
- **Sensibilidad.** ALTA para máscara/universo; MEDIA para init/reset.

## CB-07 · Universo de logits y métricas de evaluación

- **L2P/DualPrompt.** Los repos evalúan la cabeza completa100/200, incluidas clases futuras; Mammoth recorta a clases vistas y luego calcula una task-IL oracle adicional (`R train_continual.py:338-417`; M `models/l2p.py:101-102`, `models/dualprompt.py:130-133`, `utils/evaluate.py:35-109`). El paper solo exige class-IL sin task-id.
- **CODA-Prompt.** Paper/R/M evalúan entre clases vistas, pero R no integra forgetting y su `pt-local` excluye una clase por límite estricto (`learners/default.py:181-196`; `trainer.py:223-247`). Mammoth calcula forgetting/BWT/FWT solo con `enable_other_metrics=1` (`utils/args.py:382-383`; `utils/training.py:353-363`).
- **Cadencia.** Repos L2P/DualPrompt evalúan intra-tarea; Mammoth, al final por defecto. CODA evalúa por tarea en R/M.
- **Configurable.** Métricas extra/cadencia: sí. Alcance de logits L2P/DualPrompt: **no**.
- **Sensibilidad.** ALTA para espacio del argmax; MEDIA para métricas/cadencia.

## CB-08 · Learning rate visible frente al LR aplicado

- **L2P.** Paper publica0,03 con batch128. Repo/Mammoth escalan `lr×batch/256`; el Adam recibe0,015 (`R train_continual.py:593-595,637-648`; M `models/l2p.py:66`).
- **DualPrompt.** Paper publica0,005. Repo escala el valor del config por batch global; Mammoth `best` muestra0,03 y aplica0,015 con batch128 (`R train_continual.py:593-648`; M `models/dualprompt.py:64-75`).
- **CODA-Prompt.** LR nominal/aplicado0,001 coincide; la discrepancia es el decay, tratado en CB-09.
- **Configurable.** LR y batch sí; desactivar el escalado L2P/DualPrompt, no. Igualar un LR efectivo exige precompensar la CLI, decisión no ejecutada.
- **Sensibilidad.** ALTA.

## CB-09 · `n_epochs`, scheduler y modo temporal

- **L2P/DualPrompt.** `n_epochs` es un entero limpio y el YAML resuelve5; en modo epochs controla el corte y updates. Cosine genérico usaría `T_max=n_epochs`, inactivo en la receta constante. La Fase B confirma que E=1→20 no cambia ningún otro argumento estable ni nota runtime de estos dos métodos. `fitting_mode=time` aparece en choices sin presupuesto ni condición de salida (`utils/args.py:290-307`; `utils/training.py:232-299`; `utils/schedulers.py:19-35`; `reconciliacion_fase_b.md`).
- **CODA-Prompt.** Paper/repo usan coseno por tarea. Mammoth crea `CosineSchedule(K=n_epochs)` pero el dispatcher busca `model.scheduler`, no `custom_scheduler`, y recibe `None`; LR queda constante (`models/coda_prompt.py:42,65-73`; `utils/training.py:239-263`; `utils/schedulers.py:13-54`). La Fase B registra E=1 como `K=1/runtime_valid=false` y E=20 como `K=20/runtime_valid=true`. El dump E=1 sale con código 0 porque termina antes de construir el modelo; la corrida real aborta al ejecutar el assert K>1 en `begin_task`. `time` tampoco termina (`reconciliacion_fase_b.md`).
- **Configurable.** `n_epochs`, iters y early stopping sí. El cableado CODA, K independiente y criterio temporal: **no**.
- **Sensibilidad.** ALTA; es el bloqueo duro del eje 1→20.

## CB-10 · Early stopping puede seleccionar sobre test

- **Conducta común Mammoth.** La ayuda pide validación, pero no se impone. Con `fitting_mode=early_stopping` y `validation=None`, el evaluador usado para paciencia/mejor estado opera sobre test-loaders (`utils/args.py:283-303`; `main.py:438-440`; `utils/training.py:183-195,267-298`).
- **Fuentes originales.** L2P, DualPrompt y CODA usan épocas fijas; early stopping `NO_ENCONTRADO` en paper/repos.
- **CODA.** Además, los modos alternativos no adaptan el `custom_scheduler`/K.
- **Configurable.** Sí; proporcionar `--validation` evita selección directa sobre test, pero la combinación insegura no se rechaza.
- **Sensibilidad.** ALTA para L2P/DualPrompt; MEDIA como ruta no canónica CODA.

## CB-11 · Acumulación, lote final y número de updates

- **L2P/DualPrompt.** Paper no especifica acumulación; repos y Mammoth hacen un update por minibatch (`R train_continual.py:302-323`; M `models/l2p.py:91-94`, `models/dualprompt.py:123-126`). Repos derivan steps por floor; Mammoth conserva el lote parcial.
- **CODA-Prompt.** Repo hace update por batch y `drop_last=True` (`learners/prompt.py:25-44`; `trainer.py:184-198`). Mammoth conserva el remanente. Para `virtual_bs_iterations=N>1`, hace `zero_grad` cada batch, divide la loss y solo ejecuta algunos steps: descarta gradientes, no acumula ni hace flush (`models/coda_prompt.py:31-32,75-90`; `models/utils/continual_model.py:336-343,445-446`).
- **Efecto CIFAR.** Con5000/128: repos que truncan usan39 lotes; Mammoth40. En CODA a20 épocas son7800 frente a8000 updates totales.
- **Configurable.** `drop_last` sí. Acumulación real: **no**; la opción CODA no implementa esa semántica.
- **Sensibilidad.** ALTA para `virtual_bs_iterations`; MEDIA para cola de batch.

## CB-12 · Tuberías de imagen y precedencia de config

- **L2P/DualPrompt CIFAR.** Paper solo fija224/[0,1]. Repo añade resize256, JPEG, RRC escala0,05–1 y flip. Mammoth YAML l2p omite JPEG y delega defaults a torchvision; la Fase B los resuelve en `galatzo` como escala 0,08–1, ratio 0,75–1,3333, bilineal/antialias en train y bicúbica/antialias en test (`R libml/input_pipeline.py:239-321,426-500`; `libml/preprocess.py:74-104`; M YAML l2p `:5-26`; `reconciliacion_fase_b.md`).
- **CODA CIFAR.** Repo/Mammoth usan RRC224+flip en train y Resize224 en test. En Mammoth solo `model_config=best` fuerza `dataset_config=coda_prompt` y normalización identidad; la Fase B resuelve train/test como bilineales y antialias en `galatzo`, con escala 0,08–1 y ratio 0,75–1,3333 en train. El YAML default usaría stats ImageNet (`models/config/coda_prompt.yaml:1-8`; YAML coda/default; `main.py:169-186,308-351`; `reconciliacion_fase_b.md`).
- **ImageNet-R.** DualPrompt/CODA repos preservan aspecto con `Resize(256)` en test; Mammoth fuerza `(256,256)` bicúbico antes de crop224 (`R libml/input_pipeline.py:431-500`; CODA `dataloaders/utils.py:42-62`; M `datasets/seq_imagenet_r.py:112-132`). No existe preset `best` para ninguno de los tres métodos.
- **Configurable.** Requiere seleccionar otro dataset config/YAML; no es un flag de método. CODA `best` fuerza el YAML.
- **Sensibilidad.** ALTA/MEDIA.

## CB-13 · Orden de clases, semillas y rutas custom defectuosas

- **L2P/DualPrompt.** Papers no publican orden concreto. Repos usan orden natural y seed JAX42; `rand_seed>=0` activa una permutación NumPy sin sembrarla. Mammoth usa orden natural y seed `None`; `permute_classes+seed` funciona.
- **CODA-Prompt.** Paper agrega cinco shuffles y afirma impacto. El script actual usa `REPEAT=1`, seed0 y no entra en `seed>0`, por lo que ejecuta orden natural (`experiments/cifar-100.sh:9-14`; `run.py:119-133`; `trainer.py:58-76`). Mammoth tampoco fija seed/permuta.
- **Rutas Mammoth defectuosas.** `custom_task_order` intenta `range(tuple)`; `custom_class_order` sola queda inerte y con permutación indexa una lista Python con array NumPy (`datasets/utils/continual_dataset.py:212-232,309-329,456-459`).
- **Configurable.** Seed/permutación simple sí; listas custom no son fiables en este SHA.
- **Sensibilidad.** ALTA para orden; MEDIA para seed/trials.

## CB-14 · Reducción y signo de pérdidas auxiliares

- **L2P.** Paper usa `CE+λΣdistancia`; códigos maximizan similitud y suman sobre k, dividiendo solo por batch. Cambiar top-k cambia también la escala (`L2P_arxiv.tex:344-352`; R `models/prompt.py:221-250`; M `models/l2p_utils/prompt.py:103-110`). λ es0,5 paper frente a1,0 repo y0,5 Mammoth best.
- **DualPrompt.** El paper llama a γ «similitud» pero selecciona `argmin` y minimiza `CE+λγ`; R/M hacen top-k máximo y `CE−λ·sim` (`dual_prompt_camera_ready.tex:315-322,458-464`; R `models/prompt.py:177-250`; M `models/dualprompt_utils/prompt.py:75-145`). No se corrige la ambigüedad.
- **CODA.** Su regularización ortogonal se separa en CB-16.
- **Configurable.** Coeficiente/activación, parcialmente; signo/métrica/reducción, no.
- **Sensibilidad.** ALTA DualPrompt; MEDIA/BAJA L2P según forma/λ.

## CB-15 · Interfaces CLI/config con conducta silenciosa o defectuosa

- **L2P.** `type=bool` dificulta desactivar flags; `pull_constraint` sin conversor deja `"False"` truthy; prompt mask activa no recibe task-id; `global_pool` y `predefined_key` se parsean pero no se consumen; `freeze` puede convertirse en caracteres (`models/l2p.py:28-44,98-99`; `models/l2p_utils/l2p_model.py:23-37,116-134`).
- **DualPrompt.** YAML escribe `use_fix_permute`, pero el argumento funcional es `use_permute_fix`; sin el permute, el reshape mezcla ejes K/V y batch (`models/config/dualprompt.yaml:6`; `models/dualprompt.py:29-30`; `models/dualprompt_utils/prompt.py:117-126`). Capas G/E solapadas priorizan G; `global_pool` es inerte; G no-prefix desactiva G; `same_key_value=1` intenta reemplazar un `Parameter` por un Tensor (`vision_transformer.py:53-73,135-154`; `prompt.py:29-38`).
- **CODA-Prompt.** `optim_mom` del preset no se consume en Adam/SGD custom (`models/config/coda_prompt.yaml:6-8`; `models/coda_prompt.py:24-26,52-63`). Los dos pesos ortogonales se multiplican, tratado en CB-16.
- **Configurable.** Los nombres aparecen en CLI, pero las semánticas anteriores no son corregibles con otro valor, salvo activar `use_permute_fix` con el nombre correcto.
- **Sensibilidad.** ALTA para permute; MEDIA/BAJA para interfaces fuera del preset.

## CB-16 · CODA: inicialización y penalización ortogonal

- **Cambio explicado.** Paper inicializa ortogonalmente al principio (`sections/4_method.tex:47-54`). Repo actual y Mammoth ejecutan Gram–Schmidt solo sobre la partición nueva al comenzar cada tarea; el README lo explica (`R README.md:16-17`, `models/zoo.py:21-39,51-131`; M `models/coda_prompt_utils/model.py:19-69`, helper `__init__.py:27-71`).
- **Fórmula/pesos.** Paper escribe norma-2 y λ=0,1; repo usa MSE medio y λ=0 con explicación ligada a GS. Mammoth replica el MSE, pero multiplica `ortho_mu` dentro y `mu` fuera: coeficiente efectivo `mu×ortho_mu`, default0 (`sections/4_method.tex:49-66`; `sections/7_appendix.tex:9`; R `models/zoo.py:178-198`; M `models/coda_prompt.py:27-30,78-83`, `model.py:116-137`).
- **Configurable.** Escalares sí; forma, doble multiplicación e inicialización: no.
- **Sensibilidad.** ALTA: el paper reporta 75,45→70,66 al retirar ortogonalidad en ImageNet-R 10-task.

## CB-17 · Preasignación física y parámetros no usados

- **DualPrompt.** Repo asigna G-prefix para12 bloques aunque solo consume los dos configurados; Mammoth dimensiona por `len(g_prompt_layer_idx)=2` (`R models/vit.py:318-341,204-223`; M `vision_transformer.py:43-45,57-80`). La inserción funcional coincide, el estado físico no.
- **CODA-Prompt.** Paper describe expansión conceptual de componentes/cabeza. Repo/Mammoth preasignan pool completo y100 salidas; usan `floor(M/N)`, requieren conocer N y dejan cualquier resto sin usar (`sections/4_method.tex:44-49,57-66`; R `models/zoo.py:13-39,94-161,353-375`; M `models/coda_prompt_utils/model.py:9-37,57-99,171-174`).
- **Configurable.** Tamaño de pool, sí; estrategia física/floor/cabeza total, no.
- **Sensibilidad.** MEDIA; BAJA para el resto si M es divisible por N.

## CB-18 · Niveles de optimización numérica no equivalentes a su ayuda

- **Mammoth.** La receta usa nivel0. Nivel1 cambia precisión matmul; nivel2 comprueba BF16 pero no se localiza cast/autocast; nivel3 fija matmul y compila el backbone externo que L2P/DualPrompt/CODA descartan (`utils/args.py:387-393`; `main.py:482-528`; constructores citados en CB-01).
- **Fuentes originales.** Controles equivalentes `NO_ENCONTRADO`; precisión física JAX/Torch queda `NO_DETERMINABLE_ESTATICO` sin script de dtypes/backend.
- **Configurable.** El entero0–3 sí; las semánticas anunciadas no se completan con otra opción.
- **Sensibilidad.** LOW para el preset; registrar hardware/dtypes si se abandona nivel0.

## CB-19 · L2P: logging y reanudación

- **Logging.** `disable_log=1` deja `logger` sin asignar y la siguiente llamada lo usa, por lo que falla antes del bucle (`utils/training.py:153-159`). El campo permanece infraestructura, pero el defecto entra en revisión.
- **Reanudación.** El repo L2P con `save_last_ckpt_only=True` solo construye/restaura automáticamente el checkpoint al entrar en la última tarea (`configs/cifar100_l2p.py:34-38`; `train_continual.py:509-510,650-657`). Mammoth `loadcheck` hereda args y carga pesos/buffer/resultados, pero no consume optimizer/scheduler ni deduce cursor; `start_from` es independiente (`main.py:75-101,240-318`; `utils/checkpoints.py:118-247`; `utils/training.py:165-180`).
- **Configurable.** Sí, pero no es reanudación completa. Deben registrarse ruta, args heredados y tarea inicial.
- **Sensibilidad.** ALTA para estado inicial; el bug de logging no recibe S por la regla de infraestructura.

## Matriz de cobertura de los ficheros originales

| entrada consolidada | L2P | DualPrompt | CODA-Prompt |
|---|---|---|---|
| CB-01 | BHV-08 | DP-BHV-10 | C-05 |
| CB-02 | BHV-01 | DP-BHV-01 | comprobación obligatoria «Tipo de inserción» |
| CB-03 | BHV-03 | DP-BHV-02 y comprobación «Query» | comprobación «Query y composición» |
| CB-04 | BHV-02, BHV-07 | DP-BHV-08 | C-03 se desarrolla en CB-16 |
| CB-05 | BHV-04, BHV-17 | DP-BHV-03 | NO_APLICA |
| CB-06 | BHV-05 | DP-BHV-06 | C-10, C-11 |
| CB-07 | BHV-06 | DP-BHV-05 | C-12 |
| CB-08 | BHV-11 | DP-BHV-09 | C-01 se desarrolla en CB-09 |
| CB-09 | BHV-12 | DP-BHV-11 | C-01, C-13 |
| CB-10 | BHV-16 | DP-BHV-15 | controles genéricos de C-13 |
| CB-11 | BHV-13 | comprobación «Batch y acumulación» | C-02, C-09 |
| CB-12 | BHV-14 | comprobación «Preprocesado» | C-06, C-14 |
| CB-13 | BHV-10 | DP-BHV-16 | C-07, C-08 |
| CB-14 | BHV-15 | DP-BHV-04 | C-04 se desarrolla en CB-16 |
| CB-15 | BHV-09 | DP-BHV-07, DP-BHV-12 | C-11 |
| CB-16 | — | — | C-03, C-04 |
| CB-17 | — | DP-BHV-14 | C-10 |
| CB-18 | A19/OBS; sin BHV propio | DP-BHV-13 | parte de C-05 |
| CB-19 | BHV-18, BHV-19 | — | — |

La matriz cubre los 19 hallazgos L2P, los 16 DualPrompt y los 14 CODA-Prompt, además de las comprobaciones obligatorias sin discrepancia demostrada. Las ausencias originales (replay, warmup, mixup/cutmix, etc.) permanecen en las tablas de valores y no se duplican como diferencias de conducta.
