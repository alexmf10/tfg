> **AVISO — ESTATUS DE ESTE DOCUMENTO (D35)**
> Síntesis generada por IA a partir de una investigación en profundidad. Funciona como **ÍNDICE DE FUENTES**, no como fuente citable.
> **Ninguna afirmación de este documento entra en la memoria del TFG sin verificación personal en la fuente primaria (D10).**
> Uso correcto: localizar la fuente, ir a la sección indicada, leerla y citarla desde el original. Uso incorrecto: citar este documento o dar por buenos sus valores sin comprobarlos.
> Las cifras y recetas que aquí aparecen son *orientativas*: los valores que gobiernan el experimento son los de las tablas de `auditoria/`, obtenidas con evidencia localizable.

# Protocolo de comparación justa para L2P, DualPrompt y CODA-Prompt en aprendizaje incremental por clases (CIL) sobre Mammoth

## Resumen

- La literatura metodológica ofrece **prácticas recurrentes** según las cuales "comparación justa" en CIL significa **fijar el entorno** (mismo backbone y checkpoint preentrenado nombrado, mismos datasets/particiones, mismo orden de clases y semillas, misma augmentación, mismo protocolo de evaluación sin task-id y mismo presupuesto de memoria) y **respetar la receta original** de cada método, sin re-optimizar unos y otros no.
- La literatura budget-aware recomienda medir el coste a **presupuesto igualado en FLOPs y/o iteraciones (pasos forward-backward)**, siguiendo Prabhu et al. (CVPR 2023), además de parámetros entrenables, VRAM pico y tiempo de reloj; para métodos prompt hay que cuantificar el **doble forward** (query + inferencia) de L2P/DualPrompt y el overhead de composición atencional de CODA-Prompt. *(Recomendación de la literatura; la operacionalización de este TFG es la de D15/D25 — épocas comunes como palanca, coste medido como eje X. Véase §7.)*
- El marco de "tres cubos" (entorno compartido / receta por método / presupuesto como eje) es **coherente con la literatura**, con una salvedad crítica: la longitud/insertado de prompt y sobre todo el checkpoint preentrenado son simultáneamente "receta" y "fuente documentada de comparaciones injustas", por lo que exigen máxima transparencia y análisis de sensibilidad.

## 1. Definición operativa de comparación justa según la literatura

**Escenario (van de Ven, Tuytelaars & Tolias, "Three types of incremental learning", *Nature Machine Intelligence* 4(12):1185–1197, 2022; y van de Ven & Tolias 2018).** Distinguen task-, domain- y class-incremental según si en test se da la task-id, no se da, o debe inferirse. CIL es el caso en que la identidad de tarea NO se conoce ni se infiere explícitamente: el modelo debe discriminar entre todas las clases vistas con una única cabeza. Es el escenario del TFG. La motivación declarada del artículo es que comparar rendimientos resulta difícil por la falta de un marco común — exactamente lo que justifica el ejercicio del TFG.

**Qué fijar (Masana et al., "Class-incremental learning: survey and performance evaluation on image classification", *IEEE TPAMI* 45(5):5513–5533, 2023; preprint arXiv:2010.15277; framework FACIL, github.com/mmasana/FACIL).** Es la referencia canónica sobre evaluación de CIL en clasificación de imágenes. En la sección "Code framework" declara que, para hacer una comparación justa entre métodos, implementaron un framework común donde los datasets se dividen en las mismas particiones, los datos se encolan en el mismo orden al inicio de cada tarea, y todas las llamadas relacionadas con aleatoriedad se sincronizan con la misma semilla, de modo que las condiciones iniciales sean idénticas para todos los métodos. Añade que los métodos se implementaron a partir de las implementaciones originales de los autores cuando estaban disponibles, adaptándolas a los distintos setups experimentales — precedente directo de "partir del código oficial y adaptarlo al protocolo común". El survey documenta además sesgos conocidos: el *task-recency bias* (sesgo hacia clases recientes) y el fuerte efecto del orden de clases, que obliga a promediar sobre varios órdenes. Incorpora también un "Continual Hyperparameter Framework" para seleccionar el trade-off estabilidad-plasticidad sin acceso a datos de tareas pasadas. *(Verificación pendiente: el número exacto de sección varía entre la preprint de arXiv y la versión TPAMI.)*

**Baselines fuertes y críticas metodológicas.**
- **GDumb (Prabhu, Torr & Dokania, ECCV 2020)**: un "greedy sampler + dumb learner" que solo entrena desde cero sobre un buffer balanceado supera a muchos métodos de CL, cuestionando el progreso aparente del campo. Lección de protocolo: sin un baseline trivial fuerte, las mejoras aparentes pueden ser ilusorias.
- **Hsu, Liu, Ramasamy & Kira, "Re-evaluating Continual Learning Scenarios: A Categorization and Case for Strong Baselines" (NeurIPS Workshop 2018)**: introduce la categorización en escenarios y aboga por baselines fuertes y por especificar el escenario exacto, porque los números no son comparables entre escenarios distintos.
- **Para CIL con modelos preentrenados (PTM-CIL), dos baselines-suelo recomendados:** Janson et al., "A Simple Baseline that Questions the Use of Pretrained-Models in Continual Learning" (arXiv:2210.04428, NeurIPS-W 2022), que muestra que un simple Nearest Mean Classifier sobre features de un ViT congelado alcanza **83.70% en 10-Split-CIFAR-100**, superando a la mayoría de métodos del estado del arte inicializados con el mismo transformer preentrenado; y Zhou et al., "Revisiting Class-Incremental Learning with Pre-Trained Models" (SimpleCIL/ADAM; arXiv:2303.07338), donde SimpleCIL (prototipos sin entrenamiento) ya bate al estado del arte. Ambos advierten además de que los benchmarks basados en ImageNet están contaminados por solapamiento con el preentrenamiento del ViT.

**Marcos de referencia.** Mammoth (Buzzega et al., "Dark Experience for General Continual Learning: a Strong, Simple Baseline", NeurIPS 2020, pp. 15920–15930; Boschini et al., "Class-Incremental Continual Learning into the eXtended DER-verse", *IEEE TPAMI* 45(5):5497–5512, 2023) declara en su README un compromiso explícito con la reproducibilidad y mantiene un fichero `REPRODUCIBILITY.md` con los modelos verificados, advirtiendo que el proceso de rellenarlo está en curso y que la ausencia de un modelo de ese fichero no implica que no reproduzca. PILOT (Sun et al., *Science China Information Sciences*, 2025) y PyCIL (Zhou et al., *SCIS* 66(9):197101, 2023) son los toolboxes análogos para PTM-CIL y sirven como segunda fuente de recetas.

## 2. Qué se fija, qué se respeta y cómo se seleccionan los hiperparámetros

**Entorno compartido (fijar):** backbone y checkpoint exacto; datasets y particiones (p. ej. Split CIFAR-100 = 10 tareas × 10 clases; Split ImageNet-R = 10 tareas × 20 clases); orden de clases y semillas; augmentación; resize a 224×224 y normalización; protocolo de evaluación (CIL sin task-id, cabeza única); presupuesto de memoria (0 para los tres, son rehearsal-free); hardware y forma de medir el coste.

**Receta por método (respetar):** la práctica mayoritaria es "recetas originales con procedencia documentada". El apéndice de implementación de HiDe-Prompt (Wang et al., NeurIPS 2023; arXiv:2310.07234) es explícito al declarar que los hiperparámetros de cada baseline se fijan según la combinación óptima reportada en sus artículos. CODA-Prompt hace lo mismo, aunque reimplementa L2P y DualPrompt en su propio codebase. Es decir: **no** se unifican longitud de prompt, pool, learning rate, etc. entre métodos, porque forman parte del diseño de cada uno.

**Selección de hiperparámetros.** La práctica general en la literatura es: (a) usar la receta original del artículo cuando existe para ese dataset; (b) si hay que tunear (dataset nuevo), tunear sobre validación o primeras tareas, nunca sobre el test final; (c) documentar la procedencia de cada valor.

> **(b) es una ALTERNATIVA NO ADOPTADA por este TFG.** Es práctica general de la literatura, pero no es el protocolo aquí: el TFG ha descartado la reoptimización y, cuando falta un valor, aplica la cascada de la sección 5 del PLAN_MAESTRO hasta `DEFAULT_MAMMOTH`, etiquetando la procedencia. Se recoge para poder citarla como opción existente y justificar por qué no se sigue, no como regla a aplicar.

Ejemplo documentado de (b) en la literatura: DualPrompt posiciona sus prompts mediante búsqueda heurística en el conjunto de validación de Split ImageNet-R, encontrando que insertar el E-prompt (longitud 20) en las capas 3–5 y el G-Prompt (longitud 5) en las capas 1–2 (i_g=1, j_g=2) da la mejor accuracy media — la capa 2 sola fue el óptimo de la fase single-layer de la búsqueda, justificándolo en parte por usar menos parámetros adicionales (apéndice F del artículo). La literatura **no** exige re-tunear todos los métodos con el mismo presupuesto de búsqueda; re-tunear unos sí y otros no es una fuente reconocida de injusticia.

## 3. Métricas de precisión estándar en CIL

Definiendo a_{k,j} = accuracy en la tarea j tras entrenar hasta la tarea k, sobre N tareas:

- **Final Average Accuracy (A_N):** accuracy media sobre todas las clases/tareas al final. Es la métrica principal; el material suplementario de CODA-Prompt subraya que A_N es la métrica más importante porque engloba plasticidad y olvido, mientras que la medida de olvido aporta contexto adicional condicionado a la plasticidad del modelo.
- **Average Incremental Accuracy (AIA):** media de las accuracies medias tras cada tarea, AIA = (1/N)·Σ_k A_k. Captura la trayectoria, no solo el punto final (Douillard et al.; Hou et al.).
- **Forgetting Measure (FM) / Backward Transfer (BWT):** FM (Chaudhry et al. 2018) = caída media de rendimiento por tarea; BWT (Lopez-Paz & Ranzato 2017) mide la influencia del aprendizaje nuevo sobre tareas previas (negativo = olvido). Advertencia estándar: un olvido bajo con accuracy baja no es bueno — un modelo que aprende poco olvida poco.
- **Operacionalización CIL (sin task-id):** evaluación con una única cabeza sobre todas las clases vistas, sin máscara de task-id en test.

**Semillas y varianza.** L2P y DualPrompt reportan media ± desviación estándar. CODA-Prompt declara resultados sobre 5 ejecuciones en sus tablas principales (CIFAR-100 / DomainNet / ImageNet-R) y media sobre 3 en las ablaciones de componentes y longitud de prompt. 3 es un mínimo habitual de facto; 5 es frecuente en tablas principales. Debe reportarse media y desviación estándar, con distinto orden de clases.

## 4. Coste computacional: métricas y presupuestos igualados

**Prabhu et al., "Computationally Budgeted Continual Learning: What Does Matter?" (CVPR 2023).** Definen el presupuesto de cómputo en términos de FLOPs totales normalizados al número de pasos forward-backward, es decir, las iteraciones de entrenamiento para un batch dado. Igualan el presupuesto fijando el número de iteraciones por time-step y con ello restringen implícitamente las muestras de memoria utilizadas. Conclusión central: bajo restricción de cómputo, los enfoques tradicionales de CL no logran superar a un baseline mínimo que muestrea uniformemente de memoria.

**Ghunaim et al., "Real-Time Evaluation in Online Continual Learning: A New Hope" (CVPR 2023).** Proponen evaluar CL en tiempo real, donde el flujo de datos no espera a que el modelo termine de entrenar; miden los métodos respecto a su coste computacional y muestran que un baseline simple los supera. Refuerza que el coste de entrenamiento debe ser un eje de primera clase.

**Otros trabajos budget-aware.** "Continual Learning on a Diet" (Zhang et al., arXiv:2404.12766) fija la definición de presupuesto computacional como FLOPs totales normalizados en número de pasadas forward-backward, esto es, las iteraciones de entrenamiento para un tamaño de batch dado. El trabajo de ICLR 2025 "Budgeted Online Continual Learning by Adaptive Layer Freezing" argumenta usar FLOPs de entrenamiento por muestra en lugar del número de épocas, porque algunos métodos requieren mucho más cómputo por época que otros, y contabiliza además la memoria en bytes de logits y modelos, no solo la del buffer.

**Métricas de coste defendibles (síntesis):** parámetros entrenables (comparación limpia entre los tres; nótese que la variante pequeña de CODA se dimensiona explícitamente para igualar los de DualPrompt); FLOPs de inferencia por imagen y FLOPs de entrenamiento (estimación analítica independiente del hardware, siempre que se declare la metodología de conteo); tiempo de reloj de entrenamiento e inferencia (dependiente de hardware, reportar con cautela); VRAM pico; energía (opcional, poco reportado en CIL).

**Overhead específico de métodos prompt.** L2P y DualPrompt calculan una query que requiere un forward extra del ViT congelado antes de seleccionar el prompt y hacer el forward final: esto **aproximadamente duplica el coste de inferencia**. CODA-Prompt compone los prompts como suma ponderada por atención de sus componentes, añadiendo overhead de composición pero manteniendo un único esquema key-query optimizado end-to-end. Ambos costes deben cuantificarse en FLOPs, no solo describirse.

## 5. Recetas originales (orientativas — los valores vinculantes son los de `auditoria/`) — valores mezclan paper y repo; para CODA λ_ortho=0.1 es paper, la receta actual usa 0.0 (CASO 2)

Backbone común: ViT-B/16; optimizador Adam; imágenes 224×224; batch 128. *(Los coeficientes β de Adam no se fijan aquí como hecho común: la auditoría registra una ambigüedad en CODA. Su valor vinculante, con su evidencia, vive en las tablas de `auditoria/`.)*

| Método | Backbone / checkpoint | Epochs/tarea | LR | Batch | Prompt: pool / longitud / selección | Capas de inserción | Fuente |
|---|---|---|---|---|---|---|---|
| **L2P** | ViT-B/16 ImageNet-21k | 5 | 0.03 constante | 128 | pool M=10, longitud Lp=5, top-N=5 (CIFAR-100); M=30 en la reimplementación usada para comparaciones | prompt prepend a los embeddings / capa 1 (prompt-tuning) | Paper §5.2 / Ap. C (arXiv:2112.08654); repo oficial `google-research/l2p` (JAX/Flax). `JH-LEE-KR/l2p-pytorch` (timm 0.6.7) es **reimplementación PyTorch auxiliar**, no fuente primaria |
| **DualPrompt** | ViT-B/16 ImageNet-21k | 5 (CIFAR-100, 5-datasets); 50 (ImageNet-R) | 0.005 constante | 128 | G-prompt Lg=5; E-prompt Le=20; pool E indexado por tarea; λ_match=1 | G en capas 1–2; E en capas 3–5 (prefix-tuning) | Paper Ap. B/F (arXiv:2204.04799); repo oficial `google-research/l2p` (mismo repo que L2P, JAX/Flax). `JH-LEE-KR/dualprompt-pytorch` es **reimplementación PyTorch auxiliar**, no fuente primaria |
| **CODA-Prompt** | ViT-B/16 (checkpoint ambiguo: el texto menciona ImageNet-1K en un punto; la familia se asocia a Sup-21K y Mammoth usa `vit_base_patch16_224.augreg_in21k_ft_in1k`, 21k afinado en 1k) | 20 (CIFAR-100, DomainNet); 50 (ImageNet-R) | 0.001 con cosine decay | 128 | componentes M=100, longitud Lp=8; λ_ortogonalidad=0.1 | capas 1–5 (prefix-tuning) | Paper §5 / Ap. A (arXiv:2211.13218); repo GT-RIPL/CODA-Prompt |

**Configuración de reproducción empleada por HiDe-Prompt** (apéndice C, arXiv:2310.07234): L2P con M=30; DualPrompt Lg=5 en capas 1–2 y Le=20 en capas 3–5; CODA M=100, Lp=8, capas 1–5; ViT-B/16, Adam, batch 128; LR 0.001 con cosine para CODA frente a 0.005 para los demás.

**Esto NO corrobora las recetas originales.** Son las configuraciones con las que HiDe-Prompt reprodujo los baselines, no las de los artículos: usa valores propios de reproducción (p. ej. L2P con pool 30, que el propio apéndice atribuye a DualPrompt y no a L2P) y unifica el régimen de entrenamiento entre métodos. No debe usarse para validar los valores originales de la tabla anterior; para eso, la fuente es el artículo de cada método y las tablas de `auditoria/`.

## 6. Discrepancias conocidas de reproducción/implementación

1. **JAX oficial vs PyTorch.** El código oficial de L2P/DualPrompt (google-research/l2p) está en JAX/Flax, testeado en TPU; los autores advierten que estaban trabajando en verificar el entorno GPU. La comunidad usa la reimplementación PyTorch de JH-LEE-KR, enlazada desde el propio repo oficial. Las diferencias de framework pueden mover la accuracy.

2. **L2P original (prompt-tuning) vs "L2P++"/"Deep L2P++" (prefix-tuning).** CODA-Prompt reimplementa L2P con prefix-tuning y declara que su implementación rinde significativamente mejor que lo reportado originalmente porque usa la misma forma de prompting que DualPrompt, en aras de una comparación justa; y que su "Deep L2P++", que inserta en las mismas 5 capas que DualPrompt, rinde de forma similar a DualPrompt. Cifras de su Tabla 1 (10-task ImageNet-R, media±std sobre 5 ejecuciones): **Deep L2P++ 73.93±0.37 · DualPrompt 73.05±0.50 · CODA-P 76.51±0.38**. Consecuencia práctica: "L2P" no designa una sola cosa; hay que verificar qué variante implementa el framework que se use.

3. **Checkpoint preentrenado (fuente nº 1 de injusticia).** Coexisten el ViT-B/16 puro de ImageNet-21k (p. ej. `augreg_in21k` de timm, denominado "Sup-21K" en la literatura) y el 21k afinado después en ImageNet-1K. HiDe-Prompt y trabajos derivados estandarizan sobre Sup-21K precisamente porque los resultados originales de L2P/DualPrompt y de CODA-Prompt no partían del mismo checkpoint, y muestran que el checkpoint cambia sustancialmente los resultados. La confusión de nomenclatura está documentada en el issue #232 de facebookresearch/deit (los checkpoints etiquetados como IN21k eran en realidad IN21k preentrenado + IN1k afinado) y en el #181 del mismo repo. El TFG debe fijar y nombrar un único checkpoint para los tres métodos.

4. **Máscara de logits / manejo de la cabeza.** En CIL sin task-id, cómo se enmascaran (o no) los logits de clases no vistas durante el entrenamiento afecta al resultado; L2P/DualPrompt entrenan con máscara sobre las clases de la tarea actual. Este detalle difiere entre implementaciones y debe verificarse en el código del framework.

5. **Versiones de librería en Mammoth.** Mammoth fija `timm==0.9.8` para l2p/dualprompt/coda-prompt (en requirements.txt/pyproject, no en el README); los repos oficiales de L2P/DualPrompt son JAX y no usan timm, y CODA-Prompt fija `timm==0.4.12`; PyTorch ≥2.1. Distinta versión de timm implica potencialmente distinta construcción del ViT y del checkpoint por defecto. Mammoth usa el mecanismo `model_config` y dispone de overlays `best` para estos métodos; los YAML auditados no contienen un bloque `default` equivalente a una receta original. El README no documenta línea por línea sus diferencias frente a los repos oficiales: hay que auditar el código.

6. **CODA-Prompt: backbone reportado.** Ambigüedad entre el texto del artículo y la práctica de la familia/Mammoth; debe resolverse inspeccionando el código y fijando el mismo checkpoint para todos.

## 7. Checklist del entorno compartido

1. Backbone exacto + checkpoint nombrado (identificador del fichero de pesos / nombre timm), idéntico para los tres.
2. Datasets y particiones exactas, con resize y normalización idénticos.
3. Orden de clases fijado por semilla; ≥3 (idealmente 5) órdenes/semillas; reportar media ± desviación.
4. Augmentación idéntica y documentada.
5. Protocolo de evaluación CIL sin task-id, cabeza única, sin máscara en test.
6. Buffer/memoria = 0 para los tres (rehearsal-free); declararlo explícitamente.
7. Hardware y metodología de medición del coste (qué FLOPs, cómo se cuenta el doble forward, cómo se mide VRAM y tiempo de reloj).

**Regla de hiperparámetros por método (en la literatura):** recetas originales con procedencia documentada; si un valor no existe para el dataset, tunear sobre validación o primeras tareas y documentarlo; nunca sobre test. **La segunda mitad de esa regla es una alternativa NO adoptada por este TFG**, que resuelve los huecos con la cascada del PLAN_MAESTRO §5 hasta `DEFAULT_MAMMOTH` en lugar de reoptimizar.

**Presupuestos igualados.** La literatura recomienda comparar por iteraciones/FLOPs. Este TFG usa épocas comunes como palanca experimental y representa los resultados frente al coste medido, conforme a D15 y D25.

Detalle de la recomendación de la literatura: definir el presupuesto en iteraciones de entrenamiento (pasos forward-backward) y/o FLOPs, no en épocas, porque el coste por época puede diferir entre métodos. Barrer el presupuesto y trazar accuracy vs coste; reportar la frontera de Pareto en lugar de un único ganador. Las iteraciones/FLOPs son independientes de hardware; el tiempo de reloj es intuitivo pero depende de la máquina y de la implementación.

**Semillas/varianza:** 3 es un mínimo habitual de facto; 5 es frecuente en tablas principales. Distinto orden de clases; media y desviación estándar.

**Baselines-suelo recomendados:** NMC/SimpleCIL sobre el ViT congelado, para contextualizar la frontera (GDumb, Janson et al., Zhou et al.).

## 8. Advertencias

- La ambigüedad del checkpoint de CODA-Prompt no está resuelta en la literatura: debe resolverse por inspección del código y unificarse para los tres métodos.
- Los frameworks (Mammoth, PyCIL, Avalanche) no garantizan equivalencia numérica con los artículos originales; el estado de reproducción debe verificarse, no asumirse.
- Los benchmarks basados en ImageNet pueden solaparse con el preentrenamiento del ViT: interpretar la accuracy absoluta con cautela.
- El tiempo de reloj no es transferible entre máquinas; priorizar FLOPs/iteraciones para comparabilidad.
- No existe un número universal de semillas: 3–5 es la norma de facto, no un estándar formal.
- Algunas fuentes secundarias discrepan en la longitud de prompt reportada para L2P (5 en el artículo original para CIFAR-100 frente a 20 en la reimplementación usada para comparaciones): usar siempre el valor de la fuente primaria correspondiente al dataset y variante concretos.
