# Auditoría de comportamiento — DualPrompt

## Fuentes y criterio

Se compara el paper `2204.04799v2`, el repo oficial `google-research/l2p` en `dd8836e6e372df29f03d83bf3dc3a806114e9d8e` (`R@dd8836e6`) y Mammoth base `e75a491c69fd729edeb01431afb753d9157d9a81` (`M@e75a491`) en la rama `tfg-auditoria` @ `cda7f23681f7bffacee460d99e990bc803bccf04`. El commit adicional de la rama solo añade `scripts_tfg/dump_config.py`. Este fichero documenta conducta y riesgos; no modifica código ni rellena decisiones humanas.

## DP-BHV-01 · Prefix-tuning real, con distinta semántica de longitud

1. **Descripción.** Mammoth sí implementa el Pre-T del paper: separa un prompt en prefijos K/V y los concatena a `k` y `v`, nunca a `q`. La discrepancia está en el conteo: el paper define una longitud total dividida entre K y V; repo y Mammoth crean `length` posiciones completas para cada una de las dos ramas. El `L_g=5` del propio paper deja además una mitad no entera sin convención publicada.
2. **Evidencia.** Paper §4.2, `papers/dualprompt/source_v2/dual_prompt_camera_ready.tex:395,405-409`, PDF p. 8; longitudes/capas en Ap. B, `:1315`. Repo: construcción de pares y atención prefix en `l2p/models/vit.py:329-341,377-399`; `l2p/models/prefix_attention.py:125-136` (`R@dd8836e6`). Mammoth: selección de `PreT_Attention` en `mammothV2/models/dualprompt_utils/vision_transformer.py:20-34`; shapes K/V en `:67-80,136-150`; concatenación efectiva en `mammothV2/models/dualprompt_utils/attention.py:27-50` (`M@e75a491`).
3. **Configurable en Mammoth.** Longitudes, listas de capas y booleanos prefix son CLI. La interpretación de `length` como longitud por cada rama K/V es `NO_CONFIGURABLE`.
4. **Flag S.** **S-02 ALTA**. Cambia capacidad y coste, y la convención literal del paper no puede reproducirse fielmente para longitud impar sin una decisión humana.

## DP-BHV-02 · E-Prompt por tarea en train; pool completo en test

1. **Descripción.** Paper, repo y Mammoth usan el task-id durante train para actualizar el E-Prompt de la tarea corriente, pero no lo usan en inferencia. En ambos códigos, test compara la query con los diez slots, incluidos los reservados para tareas futuras; el paper no define ese universo de candidatos.
2. **Evidencia.** Paper §4.1 y Alg. 1–2, `dual_prompt_camera_ready.tex:309-322,1274-1304`, PDF pp. 7, 19. Repo: máscara de train y ausencia en eval en `l2p/train_continual.py:237-258,375-383`; selección en `l2p/models/prompt.py:177-223`. Mammoth: máscara solo con `train=True` en `mammothV2/models/dualprompt_utils/vision_transformer.py:110-132`; `forward(...task_id=-1, train=False)` en `mammothV2/models/dualprompt.py:130-133`; retrieval en `mammothV2/models/dualprompt_utils/prompt.py:75-117`.
3. **Configurable en Mammoth.** Pool size y top-k sí; restringir test a slots ya entrenados no tiene flag y es `NO_CONFIGURABLE`.
4. **Flag S.** **S-06 ALTA**. Los slots futuros se inicializan aleatoriamente y entran en la decisión; el paper no permite validar que este sea el candidato pretendido.

## DP-BHV-03 · Voto por minibatch en CIFAR

1. **Descripción.** El paper selecciona E-Prompt por muestra en inferencia. La receta oficial CIFAR y Mammoth sustituyen en test el top-1 individual por el ID más frecuente del minibatch y lo asignan a todas las muestras. Durante train también calculan la moda, pero la máscara de task-id sobrescribe después el índice y la deja sin efecto en la receta. ImageNet-R oficial desactiva el voto; Mammoth no posee un preset DualPrompt para ImageNet-R y conserva `True` si no se pasa un override manual.
2. **Evidencia.** Paper Alg. 2, `dual_prompt_camera_ready.tex:1296-1304`. Repo: CIFAR/IMR en `l2p/configs/cifar100_dualprompt.py:103-115` y `l2p/configs/imr_dualprompt.py:103-115`; voto en `l2p/models/prompt.py:207-216`; batch `[dispositivos locales,24]` y `pmap` en `l2p/libml/input_pipeline.py:539-553,582-593` y `l2p/train_continual.py:462-470`, por lo que cada voto ve 24 muestras. Mammoth: parser con advertencia en `mammothV2/models/dualprompt.py:45-51`; moda sobre minibatches de hasta 128 en `mammothV2/models/dualprompt_utils/prompt.py:101-115`, con último lote parcial habilitado en `mammothV2/datasets/utils/continual_dataset.py:530-534`.
3. **Configurable en Mammoth.** Sí, `--batchwise_prompt=0`; la receta CIFAR efectiva lo deja en 1.
4. **Flag S.** **S-03 ALTA**. La salida pasa a depender de composición, tamaño y orden de los batches de evaluación, y contradice la unidad de selección descrita en el paper.

## DP-BHV-04 · Signo del matching ambiguo en el paper

1. **Descripción.** El paper llama a `γ` “cosine similarity”, selecciona por `argmin` y minimiza `CE+λγ`. Repo y Mammoth toman top-k de similitud máxima y minimizan `CE−λ·sim`. Esto sería compatible si `γ` fuese distancia coseno, pero no con la etiqueta literal “similarity”; la auditoría no corrige la ambigüedad.
2. **Evidencia.** Paper §4.1/§4.3, `dual_prompt_camera_ready.tex:315-322,458-464`, PDF pp. 7–8. Repo: normalización/top-k y `reduce_sim` en `l2p/models/prompt.py:177-250`; loss en `l2p/train_continual.py:285-295`. Mammoth: similitud/top-k en `mammothV2/models/dualprompt_utils/prompt.py:75-145`; resta en `mammothV2/models/dualprompt.py:114-126`.
3. **Configurable en Mammoth.** Coeficiente y activación sí; métrica, reducción y signo no.
4. **Flag S.** **S-05 ALTA**. El signo afecta directamente al objetivo y no puede seleccionarse por CLI; requiere decisión humana antes de cualquier reproducción.

## DP-BHV-05 · Universo de logits distinto en evaluación

1. **Descripción.** En train, repo y Mammoth calculan CE solo sobre las clases de la tarea actual. En test, el repo deja competir las 100/200 salidas, incluidas clases futuras; Mammoth recorta a clases vistas. Los dos mantienen una sola cabeza persistente y no entregan task-id al selector. El paper no especifica expansión, máscara o universo de logits con precisión suficiente para resolver la diferencia.
2. **Evidencia.** Paper §3.1/§4.3, `dual_prompt_camera_ready.tex:249-255,458-464`; Alg. 1 `:1274-1286`. Repo: máscara train `l2p/train_continual.py:271-284`; eval completa `:338-417`; cabeza `l2p/models/vit.py:441-478`. Mammoth: máscara/recorte train y test `mammothV2/models/dualprompt.py:77-79,111-133`; evaluación class/task-IL `mammothV2/utils/evaluate.py:35-109`.
3. **Configurable en Mammoth.** `NO_CONFIGURABLE` para el recorte class-IL; la máscara task-IL de métricas es posterior y no cambia esta conducta principal.
4. **Flag S.** **S-07 ALTA**. Cambia el espacio del `argmax` y por tanto la accuracy, especialmente en etapas tempranas.

## DP-BHV-06 · Cabeza y estado del optimizador entre tareas

1. **Descripción.** Repo y Mammoth mantienen la misma cabeza de tamaño total durante toda la secuencia y reinician Adam al comienzo de cada tarea. Difieren en su estado inicial: el repo usa pesos/bias cero; Mammoth trunc-normal con std 0.02 y bias cero. El paper solo indica una cabeza recién inicializada y no documenta el reset de Adam.
2. **Evidencia.** Paper §4.3 y Alg. 1, `dual_prompt_camera_ready.tex:458-464,1274-1286`. Repo: inicialización `l2p/models/vit.py:472-478`; sustitución del checkpoint `l2p/libml/utils_vit.py:141-151`; reset `l2p/train_continual.py:536-539`. Mammoth: cabeza/init `mammothV2/models/dualprompt_utils/vision_transformer.py:97-108`; `mammothV2/backbone/vit.py:463-468,506-512`; reset `mammothV2/models/dualprompt.py:77-100`.
3. **Configurable en Mammoth.** Tipo de pooling de cabeza parcialmente sí; inicialización, dimensión total, persistencia y reset del optimizador son `NO_CONFIGURABLE`.
4. **Flag S.** **S-12 MEDIA**. Estado inicial y momentos de Adam pueden cambiar la trayectoria, aunque no hay impacto cuantificado en las fuentes.

## DP-BHV-07 · Reshape K/V–batch defectuoso por defecto

1. **Descripción.** Después de indexar E-Prompts, Mammoth obtiene ejes capas×K/V×batch×top-k×longitud×heads×dim. El default hace `reshape` directamente a capas×batch×K/V, mezclando el orden de K/V y batch. La rama `use_permute_fix` permuta antes de reshape y conserva la semántica. El YAML escribe `use_fix_permute`, nombre que el parser no consume.
2. **Evidencia.** Paper: `NO_ENCONTRADO`, es un detalle de implementación. Repo JAX conserva explícitamente ejes K/V y batch en `l2p/models/prompt.py:225-232`; `l2p/models/prefix_attention.py:125-136`. Mammoth: `mammothV2/models/dualprompt_utils/prompt.py:117-126`; parser `mammothV2/models/dualprompt.py:29-30`; typo `mammothV2/models/config/dualprompt.yaml:6`.
3. **Configurable en Mammoth.** Sí mediante el nombre correcto `--use_permute_fix=1`; el preset no lo activa. La corrección interna no es automática.
4. **Flag S.** **S-04 ALTA**. Afecta directamente qué prefijos K/V recibe cada muestra y el camino canónico usa la rama defectuosa.

## DP-BHV-08 · Inicialización y transferencia de slots

1. **Descripción.** Al empezar una tarea posterior, repo y Mammoth copian el E-Prompt previo al slot nuevo, pero no la key; el paper no documenta esta transferencia. Mammoth inicializa prompts y keys en `U[-1,1]`, mientras el repo solo fija un inicializador Flax uniforme sin escala local. Además, `prompt_key_init` controla ambos tensores en Mammoth y `initializer` es inerte.
2. **Evidencia.** Paper §4.1/Alg. 1, `dual_prompt_camera_ready.tex:309-315,1274-1286`. Repo: config `l2p/configs/cifar100_dualprompt.py:100-115`; copia `l2p/train_continual.py:550-584`; inicialización `l2p/models/prompt.py:192-199`; `l2p/models/vit.py:329-341`. Mammoth: copia `mammothV2/models/dualprompt.py:80-99`; wiring `mammothV2/models/dualprompt_utils/model.py:21-34`; init `mammothV2/models/dualprompt_utils/prompt.py:25-67`.
3. **Configurable en Mammoth.** Tipo uniforme/cero de prompts/keys sí mediante un control acoplado; escala, copia del prompt y no-copia de key son `NO_CONFIGURABLE`.
4. **Flag S.** **S-11 MEDIA**. Cambia el punto de partida de cada tarea y no hay equivalencia demostrada entre escalas de inicialización.

## DP-BHV-09 · LR visible distinto del LR que recibe Adam

1. **Descripción.** El paper declara LR constante 0.005. El repo escala linealmente el LR del config por batch global. Mammoth muta `args.lr` antes de construir Adam con la misma fórmula: en CIFAR, 0.03 visible y batch128 producen 0.015 efectivo. No hay scheduler posterior por defecto.
2. **Evidencia.** Paper Ap. B, `dual_prompt_camera_ready.tex:1315`, PDF p. 19. Repo: configs `l2p/configs/cifar100_dualprompt.py:38-44`, `l2p/configs/imr_dualprompt.py:38-44`; cálculo `l2p/train_continual.py:593-648`. Mammoth: mutación `mammothV2/models/dualprompt.py:64-75`; construcción Adam `mammothV2/models/utils/continual_model.py:262-301`; scheduler default `mammothV2/utils/schedulers.py:13-35`.
3. **Configurable en Mammoth.** El LR nominal y batch sí; desactivar el escalado automático no. Para obtener un LR efectivo dado hay que precompensar el valor CLI.
4. **Flag S.** **S-01 ALTA**. Hay una diferencia 3× frente al paper en la receta Mammoth efectiva (0.015 frente a 0.005) y el valor expuesto no es el valor optimizado.

## DP-BHV-10 · Backbone nominal descartado y congelación CLI frágil

1. **Descripción.** Mammoth construye el backbone solicitado por el framework, DualPrompt lo elimina y crea dos ViT internos con un checkpoint hardcodeado. El override `--backbone` no cambia el modelo efectivo. La lista `--freeze` sí se consulta, pero usa `type=list`: un override textual se convierte en listas de caracteres y `str.startswith(tuple(args.freeze))` recibe elementos que no son strings, por lo que esa ruta puede abortar con `TypeError`; el default Python no sufre la conversión.
2. **Evidencia.** Paper: ViT-B/16 congelado, Ap. B `dual_prompt_camera_ready.tex:1317`. Repo: `ViT-B_16` y freeze en `l2p/configs/cifar100_dualprompt.py:24,118-120`; carga `l2p/train_continual.py:948-984`. Mammoth: descarte/checkpoint `mammothV2/models/dualprompt.py:61-74`; parser freeze `:61`; filtro `mammothV2/models/dualprompt_utils/model.py:48-54`; los dos ViT `:12-46`.
3. **Configurable en Mammoth.** El backbone efectivo/checkpoint son `NO_CONFIGURABLE`. El default freeze funciona; personalizarlo por CLI no ofrece una ruta funcional fiable en este SHA.
4. **Flag S.** **S-16 MEDIA** por el backbone construido y descartado (coste/expectativas) y **S-17 MEDIA** por la interfaz de congelación. La identidad de checkpoint/preprocesado permanece además en **S-08 ALTA**.

## DP-BHV-11 · `n_epochs` limpio, pero existe un modo de duración sin salida

1. **Descripción.** `n_epochs` sí es un parámetro limpio: CIFAR resuelve a 5 y el loop lo consume. `fitting_mode=iters` y early stopping tienen condiciones de salida separadas. `fitting_mode=time` está aceptado por el parser pero el `while True` no contiene rama que termine ese modo.
2. **Evidencia.** Paper/Repo: 5 épocas CIFAR y 50 ImageNet-R, `dual_prompt_camera_ready.tex:1315`; `l2p/configs/cifar100_dualprompt.py:45`; `l2p/configs/imr_dualprompt.py:45`. Mammoth: parser `mammothV2/utils/args.py:290-307`; clase/preset `mammothV2/datasets/seq_cifar100_224.py:113-119`; `mammothV2/datasets/configs/seq-cifar100-224/l2p.yaml:27-28`; loop y condiciones `mammothV2/utils/training.py:232-303`.
3. **Configurable en Mammoth.** Épocas, iteraciones y early stopping sí. El modo `time` no dispone de duración/timeout consumido y es funcionalmente inseguro sin interrupción externa.
4. **Flag S.** **S-09 ALTA** para duración/updates de la receta; la rama `time` se añade al mismo flag porque puede impedir terminación.

## DP-BHV-12 · Interfaces de prompting/pooling con conducta silenciosa

1. **Descripción.** Si una capa aparece a la vez en listas G y E, el `if/elif` de Mammoth aplica G y omite E silenciosamente. `global_pool` se parsea pero no se propaga al ViT; el valor efectivo sigue siendo token. La ruta no-prefix para G tampoco implementa prompt-tuning: desactiva G. Además, `same_key_value=1` crea un `Parameter` y luego intenta sustituirlo por el Tensor devuelto por `repeat`; `DERIVADO` por lectura estática de la semántica de `nn.Module`, la construcción abortaría para G o E. Ninguna rama afecta al preset canónico, cuyas capas no se solapan, cuyos prompts son prefix y cuyo K/V está separado; no se ejecutó la alternativa.
2. **Evidencia.** Paper y repo usan listas disjuntas G 1–2/E 3–5; `dual_prompt_camera_ready.tex:1315`; `l2p/configs/cifar100_dualprompt.py:89-105`. Mammoth: prioridad `if/elif` en `mammothV2/models/dualprompt_utils/vision_transformer.py:135-154`; condición G no-prefix `:53-64`; rama K/V compartido G `:66-73` y E `mammothV2/models/dualprompt_utils/prompt.py:29-38`; parser/wiring en `mammothV2/models/dualprompt.py:56,58-61`; `mammothV2/models/dualprompt_utils/model.py:21-45`.
3. **Configurable en Mammoth.** Las listas, booleanos, `same_key_value` y `global_pool` aparecen en CLI, pero las cuatro semánticas descritas no ofrecen la conducta anunciada sin cambiar código.
4. **Flag S.** **S-19 BAJA**. Es un riesgo de configuración fuera del preset, no evidencia de que la receta canónica cambie resultados.

## DP-BHV-13 · Niveles de optimización no equivalentes a su ayuda

1. **Descripción.** La receta usa nivel 0. En niveles alternativos, 1 selecciona precisión matmul `high`; 2 comprueba soporte BF16 pero solo selecciona precisión matmul `medium`, sin cast/autocast localizado; 3 también fija globalmente matmul `medium`, efecto que sí alcanza los ViT internos, pero compila únicamente el backbone genérico antes de que DualPrompt lo descarte. Por lectura estática no se afirma la precisión física final de cada kernel.
2. **Evidencia.** Paper y repo: control equivalente `NO_ENCONTRADO`. Mammoth: ayuda `mammothV2/utils/args.py:387-393`; implementación y orden de construcción `mammothV2/main.py:482-528`; descarte `mammothV2/models/dualprompt.py:64-74`; búsqueda repo-wide sin `autocast` en la ruta de entrenamiento DualPrompt.
3. **Configurable en Mammoth.** El entero 0–3 sí; las semánticas anunciadas no se pueden completar mediante otro flag.
4. **Flag S.** No se propone uno separado: el preset usa 0 y no hay evidencia fuente de impacto para esta ruta no canónica. Se mantiene `LOW` por criterio de fila.

## DP-BHV-14 · Tensores G-Prompt no usados en el repo oficial

1. **Descripción.** El repo oficial asigna G-Prefix para los 12 bloques del ViT, aunque el encoder solo consume los dos correspondientes a las capas configuradas 1–2; diez bloques quedan sin uso. Mammoth dimensiona el tensor por `len(g_prompt_layer_idx)=2`. La inserción funcional coincide, pero no el número físico de parámetros/estado guardado.
2. **Evidencia.** Paper define G solo para el intervalo elegido en `dual_prompt_camera_ready.tex:371`; capas 1–2 en `:1315`. Repo: asignación con `num_layers` en `l2p/models/vit.py:318-341` y consumo condicionado/counter en `:204-223` (`R@dd8836e6`). Mammoth: `num_g_prompt=len(...)` y shape en `mammothV2/models/dualprompt_utils/vision_transformer.py:43-45,57-80` (`M@e75a491`).
3. **Configurable en Mammoth.** La lista de capas y, con ella, el número de tensores sí. No existe un control separado para preasignar los diez bloques inactivos del repo.
4. **Flag S.** Cubierto por **S-02 ALTA**: forma parte de la geometría/coste acoplados de G-Prompt y evita crear una sensibilidad duplicada.

## DP-BHV-15 · Early stopping puede seleccionar sobre test

1. **Descripción.** Mammoth anuncia que early stopping requiere validación, pero no lo comprueba. Si se activa `fitting_mode=early_stopping` sin `--validation`, el loader de evaluación sigue siendo el test y su loss/accuracy decide tanto la parada como el checkpoint restaurado. Con `--validation`, la selección usa el split de validación durante las tareas y Mammoth reconstruye el test real para una evaluación final tras toda la secuencia. El preset canónico por épocas no entra en esta ruta.
2. **Evidencia.** Paper y repo: `NO_ENCONTRADO`, usan épocas fijas; paper `dual_prompt_camera_ready.tex:1274-1286,1313-1317`; repo `l2p/configs/{cifar100,imr}_dualprompt.py:45`, `l2p/train_continual.py:593-607,786-857`. Mammoth: ayuda `mammothV2/utils/args.py:283-303`; sustitución del loader por validación `mammothV2/datasets/utils/continual_dataset.py:457-471`; selección/restauración y evaluación final en test real `mammothV2/utils/training.py:267-298,339-351`.
3. **Configurable en Mammoth.** Modo, métrica, patience y split de validación sí; la interfaz no exige combinar early stopping con `--validation`.
4. **Flag S.** Cubierto por **S-09 ALTA** por riesgo de leakage y cambio de duración al abandonar la receta por épocas.

## DP-BHV-16 · Controles custom de orden defectuosos

1. **Descripción.** El camino canónico natural y la combinación `permute_classes+seed` funcionan. `custom_task_order` pasa la tupla devuelta por `get_offsets` a `range` y aborta. `custom_class_order` guarda una lista pero no activa `permute_classes`, por lo que sola queda inerte; si se combina con permutación, el loader intenta indexar esa lista Python con un array NumPy y también aborta.
2. **Evidencia.** Paper: orden concreto `NO_ENCONTRADO`, §5.1 `dual_prompt_camera_ready.tex:487-495`. Repo: orden natural `l2p/configs/cifar100_dualprompt.py:66-80`; `l2p/libml/input_pipeline.py:615-654`. Mammoth: construcción custom `mammothV2/datasets/utils/continual_dataset.py:212-232`; offsets tupla `:309-329`; aplicación condicionada `:456-459`.
3. **Configurable en Mammoth.** La permutación por seed sí; las dos interfaces custom no ofrecen por separado la conducta anunciada en este SHA.
4. **Flag S.** Cubierto por **S-10 ALTA**, porque la revisión puede necesitar un orden exacto y no debe confiar en esas dos rutas.

## Comprobaciones obligatorias sin discrepancia demostrada

- **Query y forward extra.** Las tres fuentes usan `q(x)=f(x)[CLS]` del ViT sin prompts. Repo y Mammoth ejecutan un forward completo adicional con un segundo ViT congelado en train y test; Mammoth lo hace bajo `no_grad` en `mammothV2/models/dualprompt_utils/model.py:49-65`. Es arquitectura fija y coste relevante (**S-06 ALTA**), no una divergencia demostrada.
- **Capas de inserción.** G se inserta en capas semánticas 1–2 (índices 0–1) y E en 3–5 (2–4) en las tres fuentes. La longitud G=5 también coincide. La longitud E coincide en ImageNet-R (20), pero no en CIFAR (paper 20, repo/Mammoth 5); el historial del repo no aporta explicación autoral.
- **Optimizador y schedule.** Las tres fuentes usan Adam y LR constante dentro de cada tarea; no hay warmup efectivo. Betas 0.9/0.999 solo aparecen expresamente en el paper; repo/Mammoth delegan defaults (**S-13 MEDIA**). Adam se reinicia por tarea en ambos códigos.
- **Batch y acumulación.** Paper fija batch global128; repo fija 24 por dispositivo y Mammoth 128. No hay acumulación temporal en repo ni Mammoth: cada batch realiza exactamente `zero_grad → backward → clip → step` en `mammothV2/models/dualprompt.py:123-126`; no existe flag de acumulación (**S-14 MEDIA**).
- **E-Prompt y cabeza entre tareas.** Pool/key y cabeza persisten. Solo se copia el E-Prompt previo al slot nuevo; las keys no se copian. La cabeza no se expande físicamente ni se reinicia.
- **ImageNet-R Mammoth.** La clase de dataset existe, pero no hay entrada `best` DualPrompt; una ejecución completa exige fijar manualmente al menos LR/receta. El preset completo queda `NO_ENCONTRADO`, no como valor dinámico descubrible, y se marca **S-15 MEDIA**.
- **Orden y semilla.** Repo CIFAR usa orden natural y seed JAX 42; Mammoth usa orden natural y seed no fijada. Ambos son configurables, pero la receta no es reproducible sin decidir y registrar el seed (**S-10 ALTA**).
- **Preprocesado.** Paper solo fija resize224/[0,1]; repo añade resize256, JPEG, RRC con escala mínima 0.05 y flip. Mammoth usa el pipeline `l2p` sin JPEG; la Fase B lo resuelve en `galatzo` con escala mínima 0,08, ratio 0,75–1,3333, bilineal/antialias en train y bicúbica/antialias en test (`methods[1].dataset_effective`, hashes en `reconciliacion_fase_b.md`). Con backbone congelado, la diferencia permanece en **S-08 ALTA**.

## Ausencias comprobadas

No se hallaron replay, acumulación de gradiente, label smoothing, mixup/cutmix ni warmup efectivo en la receta DualPrompt auditada. No se convierten ausencias en defaults inferidos: sus etiquetas y evidencias están en `valores_dualprompt.md`.
