# Auditoría de comportamiento — L2P

## Alcance y fuentes

Este fichero aplica la sección I de `auditoria/plantilla.md`: documenta conductas observables sin corregirlas ni convertirlas en valores finales. Las referencias `L2P-Axx`, `L2P-Bxx` y `L2P-Exx` apuntan a `auditoria/valores_l2p.md`.

- Paper: arXiv `2112.08654v2`, `papers/l2p/source_v2/L2P_arxiv.tex`.
- Repo oficial: `l2p/` @ `dd8836e6e372df29f03d83bf3dc3a806114e9d8e`.
- Mammoth: `mammothV2/` @ `cda7f23681f7bffacee460d99e990bc803bccf04` sobre base oficial `e75a491c69fd729edeb01431afb753d9157d9a81`.

## BHV-01 — El `l2p` de Mammoth es prompt-tuning de entrada, no prefix-tuning multicapa

- **Descripción.** Los tres lados anteponen tokens de prompt una sola vez antes de la pila Transformer. El nombre `l2p` de Mammoth no implementa el esquema posterior a veces denominado “L2P++”, que inyecta prefijos K/V en varias capas.
- **Evidencia paper/repo.** El paper define `x_p=[P_{s_1};…;P_{s_N};x_e]` y después aplica `f_r` (§3.2–3.4, `L2P_arxiv.tex:253-263,296-301,344-351`). El repo fija `e_prompt_layer_idx=[0]` y `use_prefix_tune_for_e_prompt=False`, y concatena antes de `encoderblock_0` (R:dd8836e6, `configs/cifar100_l2p.py:91-94`; `models/vit.py:224-253`).
- **Evidencia Mammoth.** `Prompt.forward` concatena prompts y patch embeddings; después se añade CLS y la secuencia atraviesa todos los bloques (M:cda7f236, `models/l2p_utils/prompt.py:119-121`; `models/l2p_utils/vit_prompt.py:76-101,200-207`).
- **Configurable en Mammoth.** **No.** No existe selector input/prefix ni lista de capas para L2P.
- **Flag S.** `S-03 ALTA`, por definir la familia algorítmica y estar hardcodeado. Filas `L2P-B01` y `L2P-B02`.

## BHV-02 — Mammoth añade posiciones aprendibles a los prompts; el repo oficial no

- **Descripción.** El punto funcional de inserción coincide, pero el orden de concatenación y suma posicional cambia la representación de los tokens de prompt.
- **Evidencia paper/repo.** El paper no especifica este detalle (`L2P_arxiv.tex:259-263,296-301`). En el repo, `AddPositionEmbs` se ejecuta antes de `prepend_prompt`; los prompts añadidos en la capa 0 no reciben `posembed_input` (R:dd8836e6, `models/vit.py:198-209,224-245`).
- **Evidencia Mammoth.** Amplía `pos_embed` en `Lp×top_k`, concatena prompts antes de patches y suma el embedding posicional a la secuencia ya extendida (M:cda7f236, `models/l2p_utils/vit_prompt.py:47-50,70-74,76-99`).
- **Configurable en Mammoth.** **No.**
- **Flag S.** `S-05 MEDIA`. Fila `L2P-B03`.

## BHV-03 — La query requiere un segundo ViT y un forward adicional

- **Descripción.** La selección no consulta los patch embeddings del mismo forward: obtiene el CLS de un ViT original congelado y luego ejecuta otro ViT con prompts. Esto duplica el backbone residente y añade coste de inferencia/entrenamiento, aunque el primer forward no guarde gradientes.
- **Evidencia paper/repo.** El paper define `q(x)=f(x)[CLS]` y después procesa `x_p` con el resto del modelo (§3.3–3.4, `L2P_arxiv.tex:309-323,344-351`; algoritmo 1, `1152-1157`). El repo construye `original_vit_model` y llama primero a su `apply` en train y eval (R:dd8836e6, `train_continual.py:249-264,375-383,969-984`).
- **Evidencia Mammoth.** `L2PModel` contiene `original_model` y `model`; ejecuta el primero bajo `torch.no_grad()` y pasa su `pre_logits` como `cls_features` al segundo (M:cda7f236, `models/l2p_utils/l2p_model.py:15-37,126-134`).
- **Configurable en Mammoth.** Solo puede cambiarse el modo textual de `embedding_key`; no se puede eliminar/sustituir el ViT de query por configuración.
- **Flag S.** `S-06 ALTA`. Filas `L2P-B12`–`L2P-B14`.

## BHV-04 — El preset Mammoth selecciona prompts por mayoría del minibatch

- **Descripción.** El paper hace lookup por instancia. El repo y Mammoth `best` reemplazan esos índices por los k IDs más frecuentes del batch y asignan el mismo conjunto a todas las muestras. La salida pasa a depender de los compañeros de minibatch.
- **Evidencia paper/repo.** El paper dice “instance-wise” y define top-N por muestra (`L2P_arxiv.tex:122,168,227-228,313-323`). El repo fija `batchwise_prompt=True` e implementa conteo/mayoría (R:dd8836e6, `configs/cifar100_l2p.py:109`; `models/prompt.py:210-216`).
- **Evidencia Mammoth.** El parser default es 0, pero `models/config/l2p.yaml:1-2` lo sobrescribe a 1; `Prompt.forward` hace `unique`/conteo/top-k y expande los ganadores al batch (M:cda7f236, `models/l2p.py:35-36,57-58`; `models/l2p_utils/prompt.py:72-85`). El mismo forward se usa en train y test, aunque la ayuda solo advierta sobre test.
- **Configurable en Mammoth.** **Sí**, mediante `--batchwise_prompt 0`; la receta registrada lo activa.
- **Flag S.** `S-07 ALTA`. Fila `L2P-B15`.

## BHV-05 — Máscara de entrenamiento, persistencia y reinicio de estado

- **Descripción.** La cabeza, el pool y las claves persisten entre tareas. La CE se restringe a las clases de la tarea actual y el estado de Adam se reinicia al comenzar cada tarea. El paper no especifica ni la máscara de logits ni el ciclo de vida de los momentos de Adam.
- **Evidencia paper/repo.** El paper presenta un único `gφ`, pool y claves a lo largo del bucle, pero no describe máscara/reinicio (`L2P_arxiv.tex:344-352,1145-1165`). El repo fija a `−∞` todas las clases no actuales, conserva parámetros y recrea optimizador cuando `reinit_optimizer=True` (R:dd8836e6, `configs/cifar100_l2p.py:34,69`; `train_continual.py:271-286,536-539,986-1003`).
- **Evidencia Mammoth.** En train enmascara clases pasadas, recorta futuras y aplica CE; `begin_task` recrea el optimizador, mientras `self.net` persiste (M:cda7f236, `models/l2p.py:71-94`; `models/utils/continual_model.py:449-475`).
- **Configurable en Mammoth.** Máscara y reinicio: **no**. Persistencia de cabeza/pool: **no**. Coeficiente/término pull: parcialmente configurables.
- **Flag S.** `S-08 ALTA`, `S-15 MEDIA` y `S-19 MEDIA`. Filas `L2P-B17`–`L2P-B19`, `L2P-B24`, `L2P-B25`, `L2P-B27`, `L2P-B32`, `L2P-B45` y `L2P-B46`.

## BHV-06 — El espacio de logits de evaluación difiere

- **Descripción.** El repo oficial hace class-IL sobre las 100 salidas, incluso las clases aún no vistas. Mammoth recorta a clases vistas antes del argmax y además reporta una vista task-IL enmascarada. La cabeza única no se reinicia en ninguno.
- **Evidencia paper/repo.** El paper exige inferencia sin task-id, pero no localiza el alcance exacto de logits (`L2P_arxiv.tex:238-248,366-375`). El repo fija `eval_task_inc=False` y evalúa la cabeza de 100 sin máscara de tarea (R:dd8836e6, `configs/cifar100_l2p.py:69-71`; `train_continual.py:338-417`). Evalúa durante cada época (`configs/cifar100_l2p.py:35,46-49`; `train_continual.py:608-614,784-817`).
- **Evidencia Mammoth.** `L2P.forward` devuelve `[:, :n_seen_classes]`; `evaluate` calcula class-IL y una segunda accuracy con máscara de tarea. Por defecto evalúa al final de cada tarea y no calcula Forgetting/BWT/FWT (M:cda7f236, `models/l2p.py:101-102`; `utils/evaluate.py:35-115`; `utils/args.py:377-384`; `utils/training.py:298-307,353-361`). `eval_future=False`; activarlo no amplía este protocolo porque L2P hereda de `ContinualModel` y `main` exige `FutureModel` (`models/l2p.py:19-22`; `utils/args.py:273`; `main.py:528-529`).
- **Configurable en Mammoth.** Métricas extra y cadencia: **sí**. Alcance “100 logits” de L2P y evaluación futura: **no** para esta clase.
- **Flag S.** `S-08 ALTA` y `S-13 MEDIA`. Filas `L2P-A15`, `L2P-A16`, `L2P-A21`, `L2P-B24`, `L2P-B25`, `L2P-E17`.

## BHV-07 — Las inicializaciones no son equivalentes

- **Descripción.** El repo solo dice uniforme para prompts/claves y pone a cero el kernel de la cabeza. Mammoth usa `U[-1,1]` para prompts/claves, acopla ambas inicializaciones a un único argumento y usa normal truncada para el peso de la cabeza.
- **Evidencia paper/repo.** El paper no fija distribuciones (`L2P_arxiv.tex:1145-1165`). Repo: `initializer=uniform`, `prompt_key_init=uniform` y kernel de cabeza cero; `bias_init` no se fija en la receta (R:dd8836e6, `configs/cifar100_l2p.py:102,110`; `models/prompt.py:193-199`; `models/vit.py:472-476`).
- **Evidencia Mammoth.** Pool y claves usan `uniform_(-1,1)`; `args.prompt_key_init` se pasa como `prompt_init` y `prompt_key_init`; la cabeza recibe inicialización timm trunc-normal `std=.02`, bias cero (M:cda7f236, `models/l2p_utils/prompt.py:22-37`; `models/l2p_utils/l2p_model.py:23-37`; `models/l2p_utils/vit_prompt.py:62-66`; `backbone/vit.py:318-324,463-468`).
- **Configurable en Mammoth.** Prompts/claves: solo conjuntamente y con interfaz parcial; escala **no**. Cabeza: **no**.
- **Flag S.** `S-09 MEDIA`. Filas `L2P-B09`, `L2P-B11` y `L2P-B36`.

## BHV-08 — Backbone y checkpoint están fijados internamente

- **Descripción.** Mammoth descarta el backbone resuelto por el framework y construye dos ViT-B/16 internos con token CLS. La opción de checkpoint manual contiene una discrepancia de nombre de fichero.
- **Evidencia paper/repo.** Paper: ViT-B/16 preentrenado, sin fichero exacto (`L2P_arxiv.tex:442-443`). Repo: `ViT-B_16.npz` desde `.../imagenet21k/ViT-B_16.npz` (R:dd8836e6, `README.md:50-52,59-64`; `configs/cifar100_l2p.py:24,60-61`).
- **Evidencia Mammoth.** `del backbone` y alias `vit_base_patch16_224_in21k_fn_in1k_old` (M:cda7f236, `models/l2p.py:52-67`; `models/l2p_utils/vit_prompt.py:200-207`). El class token se crea/conserva en `backbone/vit.py:230-249,274-279`; sus roles cambian con `embedding_key`/`head_type`, no su presencia. La rama manual comprueba `data/imagenet21k_ViT-B_16.npz`, descarga `data/ViT-B_16.npz` y abre el primero (`models/l2p_utils/l2p_model.py:39-46`).
- **Configurable en Mammoth.** Arquitectura efectiva/presencia de CLS: **no**. Elección ruta timm/manual: sí, pero la ruta manual queda defectuosa en este SHA. Ampliar entrenamiento mediante `--freeze` tampoco supera el filtro prompt/head de `get_parameters` (`models/l2p.py:98-99`).
- **Flag S.** `S-01 ALTA` y `S-02 ALTA`. Filas `L2P-A01`, `L2P-A02`, `L2P-B22`, `L2P-B23`.

## BHV-09 — Varias opciones CLI no representan una configuración efectiva

- **Descripción.** La superficie CLI contiene booleans difíciles de desactivar, un boolean sin conversor y argumentos aceptados pero ignorados o rotos.
- **Evidencia paper/repo.** El paper define el método, no una CLI (§3.2–3.4, `L2P_arxiv.tex:253-352`; búsqueda fuente-wide de `command line|CLI|argparse` sin control de receta). El repo fija `use_prompt_mask=False`, `predefined_key_path=""` y no usa G-Prompt (R:dd8836e6, `configs/cifar100_l2p.py:88-112`); no se localiza consumidor de `predefined_key_path`.
- **Evidencia Mammoth.** M:cda7f236, `prompt_pool`, `prompt_key` y `use_prompt_mask` usan `type=bool` (`models/l2p.py:28-34`); activar mask no funciona porque el wrapper siempre llama con `task_id=-1` (`models/l2p_utils/l2p_model.py:126-134`; `models/l2p_utils/vit_prompt.py:76-88`). `pull_constraint` no declara `type`: `--pull_constraint False` queda como string no vacío y sigue siendo truthy (`models/l2p.py:39`; consumidor `87-89`). `global_pool` se parsea pero no se pasa; como `head_type=gap` solo tiene rama con `global_pool=avg`, esa opción queda inutilizable (`models/l2p.py:42-43`; `models/l2p_utils/l2p_model.py:23-37`; `models/l2p_utils/vit_prompt.py:106-119`). `predefined_key` solo aparece en su declaración (`models/l2p.py:38`). `freeze` combina `nargs='*'` con `type=list`: un override convierte cada texto en lista de caracteres y puede romper `startswith(tuple(args.freeze))`; aun con el default válido, `get_parameters` restringe a prompt/head (`models/l2p.py:44,98-99`; `models/l2p_utils/l2p_model.py:116-125`).
- **Configurable en Mammoth.** Nominalmente sí; efectivamente no o no de forma fiable para `pull_constraint=False`, `global_pool`/`head_type=gap`, `predefined_key`, mask activa y override de `freeze`. Los bool `type=bool` tampoco ofrecen una interfaz textual fiable.
- **Flag S.** `S-17 MEDIA`. Filas `L2P-B04`, `L2P-B10`, `L2P-B16`, `L2P-B19`, `L2P-B23`, `L2P-B40`, `L2P-B41`.

## BHV-10 — Orden de clases configurable con rutas defectuosas; semilla omitida

- **Descripción.** La receta efectiva usa orden natural y no fija semilla en Mammoth. Los mecanismos alternativos de orden presentan defectos en ambos códigos, aunque no se activan por defecto; la cache de ruido puede añadir dependencia de estado si se activa ruido sin seed.
- **Evidencia paper/repo.** El paper solo fija 10×10 y tres runs, sin semillas (`L2P_arxiv.tex:570,1120-1121`). El repo fija orden natural (`rand_seed=-1`) y `seed=42`; si `rand_seed>=0`, el valor solo activa `np.random.permutation` y nunca se pasa a NumPy ni existe `np.random.seed` repo-wide (R:dd8836e6, `configs/cifar100_l2p.py:53-54,65-68,79`; `libml/input_pipeline.py:628-640`).
- **Evidencia Mammoth.** Default `permute_classes=0`, órdenes custom ausentes y `seed=None` (M:cda7f236, `utils/args.py:274-281,359-362`; `scripts/reproduce.json:169-174`). `custom_task_order` pasa una tupla a `range`; `custom_class_order` no activa la condición que aplica el orden (M:cda7f236, `datasets/utils/continual_dataset.py:212-232,456-459,496-518`). Si se fija semilla, Mammoth siembra Python/NumPy/Torch/CUDA/loaders (`main.py:363-366`; `utils/conf.py:169-223`). Con ruido activo y seed omitida, la cache usa la clave literal `disabled` y puede reutilizar etiquetas anteriores (`utils/args.py:340-344`; `datasets/utils/label_noise.py:46-87`).
- **Configurable en Mammoth.** Semilla y permutación simple: **sí**. Órdenes custom: expuestos pero defectuosos en este SHA.
- **Flag S.** `S-10 MEDIA`. Filas `L2P-A05`, `L2P-A06` y `L2P-A23`.

## BHV-11 — El LR que llega a Adam es la mitad del valor publicado

- **Descripción.** Paper, repo config y Mammoth `best` muestran 0,03, pero ambas implementaciones de código escalan por batch/256. Con batch total 128 el optimizador recibe 0,015.
- **Evidencia paper/repo.** Paper: Adam, batch 128 y LR constante 0,03 (`L2P_arxiv.tex:562-565`). Repo: config 0,03 y fórmula `learning_rate×global_batch/256` (R:dd8836e6, `configs/cifar100_l2p.py:25,38,42`; `README.md:66`; `train_continual.py:593-595,637-648`).
- **Evidencia Mammoth.** YAML batch 128/LR 0,03; `L2P.__init__` muta `args.lr *= batch_size/256` antes de construir Adam (M:cda7f236, `models/config/l2p.yaml:3-4`; `datasets/configs/seq-cifar100-224/l2p.yaml:27`; `models/l2p.py:66`).
- **Configurable en Mammoth.** El LR nominal y batch sí; el escalado no. Para 0,03 aplicado con batch 128 habría que pedir 0,06 nominal, decisión que esta auditoría no ejecuta.
- **Flag S.** `S-14 ALTA`. Filas `L2P-B28` y `L2P-B29`.

## BHV-12 — `n_epochs` es limpio, pero no es el único codificador de duración

- **Descripción.** La receta Mammoth resuelve `n_epochs=5` limpiamente y permite override. En modo epochs controla parada/recuento y, si se activa cosine, `T_max`. También existen `n_iters`, early stopping, rango de tareas, debug e inference-only. Incluso en modos no basados en épocas, el bloque de train exige `n_epochs>0`. La opción `fitting_mode=time` carece de temporizador/condición de salida.
- **Evidencia paper/repo.** Paper y repo fijan cinco épocas (`L2P_arxiv.tex:562-565`; R:dd8836e6, `configs/cifar100_l2p.py:45`). El repo deriva steps con división entera e ignora el campo declarado `num_train_steps_per_task` al forzar `num_train_steps=-1` (R:dd8836e6, `train_continual.py:593-607`).
- **Evidencia Mammoth.** `n_epochs` es `int`, el YAML L2P lo fija a 5 y la CLI prevalece; el bloque exige `n_epochs>0`, y la rama epochs usa `epoch>=n_epochs` y `n_epochs×len(loader)` (M:cda7f236, `utils/args.py:283-307`; `datasets/configs/seq-cifar100-224/l2p.yaml:28`; `main.py:426-428`; `utils/training.py:232,248-298`). Cosine usa `T_max=n_epochs` (`utils/schedulers.py:19-35`). `time` aparece en choices, pero no en ninguna rama de parada (`utils/args.py:290-295`; `utils/training.py:257-298`).
- **Configurable en Mammoth.** `n_epochs`, iters, early stopping, rango y debug: sí. Warmup L2P: no. `time`: expuesto pero no funcional.
- **Flag S.** `S-20 ALTA` para la opción `time`; el `n_epochs=5` coincidente no requiere S propio. Filas `L2P-E01`–`L2P-E22`.

## BHV-13 — No hay acumulación temporal de gradiente

- **Descripción.** Un minibatch produce inmediatamente un update. El `pmean` del repo agrega entre dispositivos, no entre pasos; Mammoth no ofrece opción de acumulación.
- **Evidencia paper/repo.** El paper no menciona acumulación en su setup (Paper §5.2, `L2P_arxiv.tex:562-570`; búsqueda fuente-wide `gradient accum|accumulat.*gradient` sin coincidencias). El repo ejecuta un `value_and_grad`, `pmean` y `apply_gradient` por batch (R:dd8836e6, `train_continual.py:302-323,778-779`).
- **Evidencia Mammoth.** Cada `observe` hace `zero_grad`, `backward`, clip y `step`; no aparecen flags `grad_accum`/`gradient_accumulation` en el repo (M:cda7f236, `models/l2p.py:91-94`).
- **Configurable en Mammoth.** **No.** El comportamiento efectivo es acumulación 1, pero el paper no proporciona ese valor literal.
- **Flag S.** `S-12 MEDIA`, por su acoplamiento con batch y LR. Fila `L2P-A12`.

## BHV-14 — Las tuberías de imagen no son equivalentes

- **Descripción.** Coinciden en 224 y rango [0,1], pero el paper no localiza augmentación; el repo y Mammoth aplican tuberías distintas en detalles que pueden modificar resultados.
- **Evidencia paper/repo.** Paper: resize 224 y [0,1] (`L2P_arxiv.tex:562-563`). Repo train: resize 256, recodificación JPEG, random-resized-crop 224 con área 0,05–1, flip; eval: resize 256/center crop 224 (R:dd8836e6, `configs/cifar100_l2p.py:56-58,77-78`; `libml/input_pipeline.py:426-500`; `libml/preprocess.py:74-104,181-197`).
- **Evidencia Mammoth.** Train YAML: Resize256, `RandomResizedCrop(224)`, flip, tensor, Normalize(0,1); eval: Resize256 con `interpolation=3`, center crop, tensor, Normalize(0,1) (M:cda7f236, `datasets/configs/seq-cifar100-224/l2p.yaml:5-26`). La escala/ratio del crop quedan en defaults de torchvision del entorno.
- **Configurable en Mammoth.** No mediante argumentos L2P; requiere otro dataset config/YAML.
- **Flag S.** `S-11 MEDIA`. Filas `L2P-A07` y `L2P-A08`.

## BHV-15 — El código del pull suma sobre k y divide solo por batch

- **Descripción.** Los códigos expresan la distancia del paper como maximización de similitud. Su reducción suma sobre batch, k y las 768 dimensiones, y divide únicamente por batch, no por k; por ello cambiar top-k también cambia la escala del término auxiliar. El paper no fija esa normalización de minibatch.
- **Evidencia paper/repo.** El paper usa pérdida por muestra `CE + λΣγ(q,k)` con distancia coseno y habla de acumular el batch, sin divisor exacto (`L2P_arxiv.tex:344-352,1152-1161`). El repo calcula producto de vectores normalizados, suma y divide por batch, y resta el resultado a CE (R:dd8836e6, `models/prompt.py:221-250`; `train_continual.py:285-295`).
- **Evidencia Mammoth.** Replica la suma/división por batch y `loss -= coeff×reduce_sim.mean()` (M:cda7f236, `models/l2p_utils/prompt.py:103-110`; `models/l2p.py:87-89`).
- **Configurable en Mammoth.** Coeficiente y activación parcial: sí; métrica/reducción: **no**.
- **Flag S.** `S-19 MEDIA`; además `S-16 BAJA` cubre el conflicto 0,5 frente a 1,0. Filas `L2P-B19`, `L2P-B20` y `L2P-B46`.

## BHV-16 — Early stopping puede seleccionar sobre test si falta validación

- **Descripción.** La ayuda exige un split de validación, pero no hay comprobación que obligue a proporcionarlo. Al activar early stopping con `validation=None`, el evaluador usado para decidir paciencia/mejor checkpoint es el mismo dataset con sus test-loaders.
- **Evidencia paper/repo.** Paper y repo oficial usan cinco épocas fijas; no describen early stopping (`L2P_arxiv.tex:562-565`; R:dd8836e6, `configs/cifar100_l2p.py:45`; `train_continual.py:593-607`).
- **Evidencia Mammoth.** La ayuda declara el requisito (M:cda7f236, `utils/args.py:283-303`), pero `extend_args` solo comprueba `validation_mode` si ya existe validación (`main.py:438-440`). Sin `eval_future`, `eval_dataset=dataset`; la rama early llama a `eval_dataset.evaluate` para decidir la parada (`utils/training.py:183-195,272-296`).
- **Configurable en Mammoth.** Sí: proporcionar `--validation` evita usar directamente el test split. La combinación insegura no se rechaza automáticamente.
- **Flag S.** `S-21 ALTA`, por posible fuga de test al criterio de selección. Fila `L2P-E12`.

## BHV-17 — “Batchwise” usa un dominio distinto en repo y Mammoth

- **Descripción.** El mismo flag no implica el mismo conjunto de votantes. El repo hace pmap y cada réplica elige mayoría dentro de 16 ejemplos; Mammoth por defecto ejecuta un dispositivo y vota sobre los 128. Con DataParallel, Mammoth vuelve a votar por shard y el resultado depende del número de GPU.
- **Evidencia paper/repo.** El paper no usa voto batchwise (`L2P_arxiv.tex:122,313-323`). El repo combina `per_device_batch_size=16`, batchwise activo y `jax.pmap`; `models/prompt.py` recibe la dimensión local de batch (R:dd8836e6, `configs/cifar100_l2p.py:25,109`; `libml/input_pipeline.py:540-553`; `train_continual.py:664-716`; `models/prompt.py:210-216`).
- **Evidencia Mammoth.** Default `distributed=no`; la receta usa batch 128 y `Prompt.forward` cuenta sobre `x_embed.shape[0]`. `distributed=dp` envuelve `model.net` en DataParallel y DDP se rechaza (M:cda7f236, `utils/args.py:393`; `main.py:531-542`; `datasets/configs/seq-cifar100-224/l2p.yaml:27`; `models/l2p_utils/prompt.py:72-85`).
- **Configurable en Mammoth.** Sí, indirectamente mediante distribución/número de GPU y batch. No existe selector separado de “voto global” frente a “voto por shard”.
- **Flag S.** `S-07 ALTA`, compartido con la decisión instancewise/batchwise. Filas `L2P-B15` y `L2P-B44`.

## BHV-18 — `disable_log=1` falla antes de entrenar

- **Descripción.** El flag de infraestructura que promete desactivar logging deja `logger` sin asignar, pero la siguiente llamada lo usa incondicionalmente; la ejecución termina antes del bucle de tareas.
- **Evidencia paper/repo.** El paper no define controles de logging en el setup (§5.2, `L2P_arxiv.tex:523-570`; búsqueda fuente-wide de `wandb|tensorboard|disable_log` sin coincidencias). El repo oficial expone cadencias/guardado, pero no un `disable_log` equivalente en la receta (R:dd8836e6, `configs/cifar100_l2p.py:46-50,123-128`; búsqueda repo-wide de `disable_log` sin coincidencias).
- **Evidencia Mammoth.** `logger` solo se crea bajo `if not args.disable_log`, e inmediatamente se pasa a `track_system_stats(logger, ...)` fuera de esa rama (M:cda7f236, `utils/args.py:380`; `utils/training.py:153-159`).
- **Configurable en Mammoth.** Nominalmente sí; el valor no-default es defectuoso en este SHA.
- **Flag S.** No se crea uno: `disable_log` permanece en la lista agrupada por la regla explícita de infraestructura. El defecto sí entra en cola como `HB-18`, referencia `INF-M08`.

## BHV-19 — La semántica de carga/reanudación no coincide

- **Descripción.** Con la receta oficial solo se restaura automáticamente el checkpoint de la última tarea. Mammoth parte de cero por defecto; `loadcheck` puede heredar argumentos y cargar pesos/resultados, pero no constituye una reanudación completa porque no restaura optimizer/scheduler ni deduce la tarea siguiente.
- **Evidencia paper/repo.** El paper no registra reanudación como parte del protocolo (`L2P_arxiv.tex:562-570`; búsqueda de `resume|training checkpoint` sin coincidencias). El repo fija `save_last_ckpt_only=True`: el guard solo crea `MultihostCheckpoint` y llama `restore_or_initialize` cuando `task_id==9`; ese state sí contiene optimizador/step/parámetros (R:dd8836e6, `configs/cifar100_l2p.py:34-38`; `train_continual.py:509-510,650-657,860-866,928-929`).
- **Evidencia Mammoth.** `loadcheck=None`; con ruta, el parser carga args como defaults y el loader restaura modelo/buffer/resultados. Aunque el saver escribe optimizer/scheduler, `mammoth_load_checkpoint` termina sin consumirlos; el rango de tareas se decide aparte mediante `start_from` (M:cda7f236, `main.py:75-101,240-274,304-318,356-357`; `utils/checkpoints.py:118-211,215-247`; `utils/training.py:165-180`).
- **Configurable en Mammoth.** Sí, mediante `--loadcheck` y, separadamente, `--start_from`; la auditoría debe registrar ruta, argumentos heredados y tarea inicial.
- **Flag S.** `S-22 ALTA`, por el impacto directo sobre estado inicial, receta, coste y resultados. Fila `L2P-A24`.
