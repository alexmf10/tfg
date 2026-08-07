# Análisis estático integral pre-Acta — dependencias ocultas y re-auditoría de las 231 filas

Fecha: 2026-08-07. Ejecutor: Claude Code (suplencia D30). Bloque solo-lectura sobre código y tablas; este documento es la única escritura nueva en `auditoria/`.

**Base contrastada (FASE 0).** TFG `master` @ `392aab9` («Resuelve los elevados de la auditoría de traspaso (visto bueno 6-ago)»), árbol limpio. mammothV2 `tfg-auditoria` @ `4353bc1` («Fix D39 validator logging startup»), árbol limpio. Salvo indicación, las referencias `fichero:línea` de Mammoth son de mammothV2 @ `4353bc1` (los ficheros citados coinciden con la base auditada `e75a491` salvo `models/coda_prompt.py`, que incluye el parche D39). Repos pineados: `l2p/` @ `dd8836e6`, `codaprompt/` @ `6417d4f6`. Papers: LaTeX congelado en `papers/` (SHA-256 en `fuentes.md` §5).

**Regla de lectura.** Este bloque DESCRIBE, no decide. Cada hallazgo lleva una clase: `CONTRADICE-VALOR` (una celda no dice lo que dice su fuente), `CONTRADICE-CUBO` (la fila está en el cubo equivocado según el test de §3), `DEPENDENCIA-NO-REGISTRADA` (dependencia real del código ausente del registro operativo), `TRAMPA-CONFIG` (valor silencioso del framework que difiere del valor pre-final si no se pasa flag), `NO-CONFIGURABLE` (el valor pre-final no es fijable por CLI/config), `CONFIRMADO` (la fila/el supuesto resiste el contraste).

---

## 1. FASE 1 — Dependencias ocultas en el código (inventario con file:línea)

### 1.1 Operaciones dependientes del batch

| # | Operación | Evidencia | Afecta a |
|---|---|---|---|
| 1 | Voto batchwise L2P: sustituye el top-k por instancia por los k IDs más frecuentes del minibatch; **activo en train y test** cuando `batchwise_prompt=1` | `models/l2p_utils/prompt.py:74-85` | L2P (preset `best` lo activa: `models/config/l2p.yaml:2`) |
| 2 | Voto batchwise DualPrompt: **en train queda anulado** por la máscara de task-id (la máscara se aplica *después* del voto); **en test sí actúa** (train=False ⇒ `prompt_mask=None`) | voto `models/dualprompt_utils/prompt.py:101-112`; máscara posterior `:114-115`; gating por train `models/dualprompt_utils/vision_transformer.py:119-127`; default 1 `models/dualprompt.py:50` | DualPrompt (dependencia del batch en inferencia) |
| 3 | Pérdida pull/matching: `reduce_sim = torch.sum(sim)/B` — suma sobre batch, k y dimensión dividida por el batch real (el último lote de 8 con b64 pesa igual por muestra) | `models/l2p_utils/prompt.py:108`; `models/dualprompt_utils/prompt.py:143` | L2P, DualPrompt |
| 4 | CE con reducción media por batch (default de `F.cross_entropy` del dataset) | `datasets/seq_cifar100_224.py` (get_loss); consumo `models/l2p.py:87`, `models/dualprompt.py:117`, `models/coda_prompt.py:102` | los tres |
| 5 | Escalado runtime del LR por batch: `args.lr = lr·batch_size/256` | `models/l2p.py:66`; `models/dualprompt.py:71`; CODA **no** escala | L2P, DualPrompt |
| 6 | CODA `virtual_bs_iterations`: default 1 correcto; con N>1 la semántica es defectuosa (`zero_grad` al inicio de cada `observe` descarta los gradientes de lotes intermedios; no acumula) | `models/coda_prompt.py:31-32,95-110` (zero_grad en `:97`) | CODA (coincide con OBS-03 de `valores_coda.md`) |
| 7 | Normalizaciones: **solo LayerNorm** en los tres caminos; **cero BatchNorm** ⇒ ningún estadístico de lote en la normalización | `backbone/vit.py:249,288,307,310`; `models/coda_prompt_utils/vit.py:70-77,118`; L2P/Dual heredan `backbone/vit.py` | los tres |
| 8 | `drop_last=0` (default): con b64 y 5.000 muestras/tarea, 79 pasos/época con último lote de 8; interactúa con (1)-(3) | `utils/args.py:320-321`; `datasets/utils/continual_dataset.py:530-534` | los tres |
| 9 | CODA: selección de prompts **por instancia** (atención coseno por muestra), sin ramas batchwise; `ortho_penalty` opera solo sobre parámetros | `models/coda_prompt_utils/model.py:101-114,136-137` | CODA (limpio) |

### 1.2 Mutaciones y recálculos de hiperparámetros en runtime

| # | Mutación | Evidencia |
|---|---|---|
| 1 | `args.lr` mutado en `__init__` al LR aplicado y **persistido en args** (checkpoints/volcados) — guarda D40 vigente | `models/l2p.py:66`; `models/dualprompt.py:71` |
| 2 | Scheduler CODA por tarea (D39): `K = args.n_epochs`; `n_epochs==1 ⇒ scheduler None` (LR base constante); `last_epoch` restaurado a −1; `step()` al comenzar cada época salvo la primera | `models/coda_prompt.py:73-90`; `utils/schedulers.py:38-54` (assert `K>1` en `:44`, evitado por la rama `n_epochs==1` del caller) |
| 3 | Optimizador recreado POR TAREA en los tres (Adam pierde momentos entre tareas, uniforme): wrapper + hook de cada método | `models/utils/continual_model.py:466-470`; `models/l2p.py:74-77`; `models/dualprompt.py:99`; `models/coda_prompt.py:50,71` |
| 4 | CODA `get_optimizer` propio: Adam recibe **solo** `lr` y `weight_decay` ⇒ `optim_mom=0.9` del YAML `best` es **inerte** con Adam (igual en el genérico) | `models/coda_prompt.py:52-63`; `models/utils/continual_model.py:295-297`; `models/config/coda_prompt.yaml:6` |
| 5 | DualPrompt `begin_task`: copia de prompt slot t−1→t dependiente de `top_k`/`size`; la línea 97 reasigna params al opt viejo y el 99 lo recrea (inocuo pero confuso) | `models/dualprompt.py:80-99` |
| 6 | `args.class_order` se escribe en runtime al permutar | `datasets/utils/continual_dataset.py:229-232` |
| 7 | `n_iterations = n_epochs×len(train_loader)` (progreso), scheduler genérico solo si `--lr_scheduler` (CODA lo prohíbe con assert) | `utils/training.py:250`; `utils/schedulers.py:19-35`; `models/coda_prompt.py:42` |
| 8 | Guard del wrapper: `meta_begin_task` comprueba el atributo `scheduler` pero consume `custom_scheduler` (`models/utils/continual_model.py:466-472`); como CODA no define `scheduler`, el opt se recarga siempre y el aviso de la rama else es inalcanzable. Con D39 la conexión real va por `begin_epoch` (`models/coda_prompt.py:88-90`), no por el dispatcher (`utils/training.py:241`). Sin efecto funcional; cableado asimétrico documentado. |

### 1.3 Cadena de precedencia efectiva y mapa de valores silenciosos

**Cadena real** (verificada en `main.py:139-188,304-318,351`): default de `add_argument` < defaults de la clase de dataset < **YAML de dataset** < `set_defaults` del parser del modelo < **YAML de modelo** (merge literal `{**dataset_config, **base_config, **model_config}` en `main.py:186`; `update_cli_defaults` en `:309`) < **CLI** (gana en `parse_args`, `:351`). Matiz crítico: el YAML de dataset que se carga lo elige el **YAML de modelo** si trae clave `dataset_config` (`main.py:177-180`); sin `--model_config best`, esa clave no existe y cae a `datasets/configs/seq-cifar100-224/default.yaml`.

**Mapa de valores que se aplican EN SILENCIO si no se pasa el flag, y difieren del valor pre-final de las tablas** (clase `batchwise_prompt`):

| Método | Parámetro | Silencioso (sin flag) | Pre-final | Flag obligatorio |
|---|---|---|---|---|
| L2P | batchwise_prompt | `1` (best `models/config/l2p.yaml:2`) | 0 (por instancia) | `--batchwise_prompt 0` |
| L2P | lr nominal | `0.03` (best `l2p.yaml:4`) → aplicado 0,0075 con b64 | nominal 0,12 → aplicado 0,03 (D40) | `--lr 0.12` |
| L2P/Dual/CODA | batch_size | `128` (dataset YAML; Dual además `set_defaults` `models/dualprompt.py:27`; CODA best `coda_prompt.yaml:3`) | 64 (D31/D41-a) | `--batch_size 64` |
| L2P/Dual | n_epochs | `5` (`datasets/configs/seq-cifar100-224/l2p.yaml:28`) | peldaño {1,5,20} | `--n_epochs E` |
| CODA | n_epochs | `20` (best `coda_prompt.yaml:8` + dataset `coda_prompt.yaml:23`) | peldaño {1,5,20} | `--n_epochs E` (arrastra K) |
| Dual | length (L_e) | `5` (parser `models/dualprompt.py:46`; best NO lo fija) | 20 (D41-B4) | `--length 20` |
| Dual | batchwise_prompt | `1` (parser `models/dualprompt.py:50`) | 0 (por muestra) | `--batchwise_prompt 0` |
| Dual | lr nominal | `0.03` (best `dualprompt.yaml:3`) → aplicado 0,0075 con b64 | nominal 0,02 → aplicado 0,005 (D40) | `--lr 0.02` |
| los tres | seed | `None` (aleatorio; `utils/args.py:359-360`) | 5 semillas (D19) | `--seed s` |
| los tres | orden de clases | natural (`permute_classes=0`, `utils/args.py:361-362`) | orden por semilla (D19) | `--permute_classes 1` |
| los tres | olvido (F_N) | no se calcula (`enable_other_metrics=0`, `utils/args.py:382-383`) | protocolo §11 incluye olvido | `--enable_other_metrics 1` (o agregación offline) |
| los tres | **dataset_config** | `default.yaml` (normalización ImageNet 0.485/0.229, otros transforms, n_epochs 20) si no hay `--model_config best` | tubería [0,1] de l2p.yaml / coda_prompt.yaml | `--dataset_config l2p` (L2P/Dual) · `--dataset_config coda_prompt` (CODA) |
| L2P (solo con `model_config=default`) | pull_constraint_coeff | `0.1` (parser `models/l2p.py:40`) | 0,5 | `--pull_constraint_coeff 0.5` |

Coinciden en silencio con el pre-final (sin riesgo): optimizador adam (`set_defaults`), wd 0, clip_grad 1, top-k/pool/longitudes L2P (5/10/5), G/E de Dual (5, [0,1], [2,3,4], prefix, size 10, top_k 1, λ=1), CODA lr 1e-3 / pool 100 / len 8 / mu 0 / ortho_mu 0, drop_last 0, fitting_mode epochs, lr_scheduler None, code_optimization 0.

### 1.4 Propagación de la semilla

- `--seed` (si se pasa): `main.py:363-366` → `utils/conf.py:169-182` siembra `random`, `np.random`, `torch.manual_seed`, `torch.cuda.manual_seed_all`. Controla la **inicialización** de prompts/claves/cabeza (creaciones posteriores al parseo: `models/l2p_utils/prompt.py:27-37`; `models/dualprompt_utils/prompt.py:36-62`; `models/coda_prompt_utils/model.py:140-149` + `gram_schmidt`).
- **Orden de datos (shuffle)**: generator del DataLoader sembrado con la semilla + `worker_init_fn` por worker — solo si `--seed` (`utils/conf.py:185-192,214-221`).
- **Permutación de clases**: SOLO con `--permute_classes 1`; re-siembra `np.random` con `args.seed` justo antes de permutar (`datasets/utils/continual_dataset.py:228-232`). `--seed` a secas NO cambia el orden de clases.
- No se fuerza `torch.use_deterministic_algorithms` ⇒ reproducibilidad aproximada en GPU (el repo oficial de CODA sí activa `cudnn.deterministic`; registrado en valores_coda A06).

### 1.5 Enmascarado de evaluación a clases vistas

- L2P: `forward` recorta a vistas — `models/l2p.py:101-102` (`[:, :self.n_seen_classes]`).
- DualPrompt: `models/dualprompt.py:130-133` (`[:, :self.offset_2]`).
- CODA: `models/coda_prompt.py:114-115` (`[:, :self.offset_2]`).
- Evaluador: `utils/evaluate.py:54` (`n_classes = dataset.get_offsets()[1]`, tarea actual vía `c_task`, `datasets/utils/continual_dataset.py:309-324`) y `:88` (argmax sobre `outputs[:, :n_classes]`). Task-IL como máscara post-hoc `utils/evaluate.py:11-25`.
- Entrenamiento: pasadas a −∞ + recorte a vistas — `models/l2p.py:85-87`; `models/dualprompt.py:115-117`; `models/coda_prompt.py:100-102`.

---

## 2. FASE 2 — Re-auditoría de las 231 filas

### 2.1 Pasada mecánica (231/231)

Universo: `valores_l2p.md` 92 filas (24 A + 46 B + 22 EJE), `valores_dualprompt.md` 83 (26 A + 47 B + 10 EJE), `valores_coda.md` 56 (20 A + 24 B + 12 EJE). Casos verificados contra el recuento: CASO 1 = 19/22/12; CASO 2 = 0/0/2; **CASO 3 = 7/7/2 (16 total)**; CASO 4 = 18/17/7; **CASO 5 = 2/1/1 (4 total)**.

- **(a) Test de cubo (§3):** 231/231 conformes. Ninguna fila de cubo B es entorno no reclamado ni viceversa; las filas límite (code_optimization en A, dominio del voto batchwise B44 en B, betas de Adam en B) justifican su cubo en la propia celda. `CONTRADICE-CUBO: 0`.
- **(b) Coherencia caso↔evidencia:** 231/231 coherentes con sus columnas de evidencia. Las 6 filas cuyo valor final difiere del propuesto (L2P B28/B29/B36/B45; Dual semántica Pre-T, signo del matching, init de cabeza; CODA B02/B21) documentan el flujo vía OBS/decisión C1.x/[B-MARCO], conforme a plantilla §A. `CONTRADICE-VALOR: 0`.
- **(c) Configurabilidad del valor pre-final:** flags exactos consolidados en §1.3 (obligatorios) y en las tablas por fila; **74 filas portan `NO_CONFIGURABLE`** (L2P 28, Dual 25, CODA 21) y el contraste de código de FASE 1 no encontró ninguna marcada en falso. Los booleanos con override roto (`type=bool`: prompt_pool, prompt_key, use_prompt_mask, pull_constraint de L2P) tienen default correcto y sus overrides están PROHIBIDOS por D41-B3; sin cambio.

Filas `ELEVADA-AL-ACTA`: **0** (todas resueltas en tiempo).

### 2.2 Pasada profunda (fuente primaria delante)

Verificado en el LaTeX congelado y en los repos pineados; extracto mínimo + localizador. Las 16 CONFLICTO→PAPER, las 4 SIN_FUENTE y las filas S/grupo señaladas por FASE 1: **todas CONFIRMADAS** (la celda dice exactamente lo que dice la fuente).

**L2P** — `papers/l2p/source_v2/L2P_arxiv.tex:561-570`: «a batch size of 128, and a constant learning rate of 0.03», «train every task for 5 epochs», «M= 10, N= 5, L_p= 5», «46,080», «λ=0.5», «averaged over 3 runs». Repo `l2p/configs/cifar100_l2p.py` @dd8836e6: `per_device_batch_size=16` (:25), `learning_rate=0.03` (:38), `num_epochs=5` (:45), `pool_size=10/length=10/top_k=4` (:99-101), `batchwise_prompt=True` (:109), `pull_constraint_coeff=1.0` (:122), clip 1.0 (:41), wd 0 (:44), constant/warmup 0 (:42-43). ⇒ B06 (5↔10), B07 (5↔4), B08 (25↔40), B15 (instancia↔batchwise), B20 (0,5↔1,0), B29 (0,03↔escalado), B38 (46.080↔84.480): CONFIRMADAS.

**DualPrompt** — `papers/dualprompt/source_v2/dual_prompt_camera_ready.tex:1315`: «constant learning rate of 0.005», «batch size of 128», «5 epochs», «λ = 1», «i_g=1, j_g=2, i_e=3, j_e=5», «L_g=5 and L_e=20». Split Pre-T `:395,405-409`: «splits p into p_K, p_V ∈ R^{L_p/2×D}» — el conflicto texto↔código de la semántica de longitud es real y está bien citado. Grid `:1458-1468`: «{5,10,20,40}×{5,10,20,40}… L_g=5, L_e=20 is the optimal choice» (sostiene D41-i). Repo `l2p/configs/cifar100_dualprompt.py`: `per_device_batch_size=24` (:25), `learning_rate=0.03` (:38), `num_epochs=5` (:45), `length=5` (:105), `top_k=1` (:105), `batchwise_prompt=True` (:113), capas [0,1]/[2,3,4] (:92,97). ⇒ las 7 CONFLICTO→PAPER de Dual: CONFIRMADAS.

**CODA** — `papers/coda-prompt/source_v2/sections/7_appendix.tex:5`: «with β1 = 0.9 and β1 = 0.999» — **errata verificada** (β1 dos veces, β2 ausente), «batch size of 128», «normalize them to [0,1]», «CIFAR-100 … for 20 epochs»; `:7`: «learning rate of 1e-3 … cosine-decaying learning rate»; `:9`: «prompt length of 8 and 100 prompt components … [4,40] … [5,500] … λ = 0.1»; `:11`: máscara −∞ «crucial». Repo `codaprompt/` @6417d4f: `configs/cifar-100_prompt.yaml` schedule [20]/cosine/batch 128/lr 0.001/momentum 0.9/wd 0; `experiments/cifar-100.sh:28` `--prompt_param 100 8 0.0`; fórmula `utils/schedulers.py:53-54` `base_lr·cos(99π·e/(200(K−1)))` — **idéntica** a `mammothV2/utils/schedulers.py:48-51`; semántica de step `learners/default.py:95` (`if epoch > 0: self.scheduler.step()`) y restauración `last_epoch=-1` (`utils/schedulers.py:19-20`) — coincide con el parche D39 (`models/coda_prompt.py:79-86`). ⇒ B02, B21 y todo el bloque E04-E08: CONFIRMADAS.

**Las 4 SIN_FUENTE**: L2P B40 `global_pool` — verificado que `L2PModel` no lo pasa al constructor (`models/l2p_utils/l2p_model.py:23-37`); L2P B45 eps de Adam — Adam recibe solo lr/wd (`models/utils/continual_model.py:295-297`); Dual fix de permutación — clave YAML muerta `use_fix_permute: 0` (`models/config/dualprompt.yaml:6`) frente al flag funcional `use_permute_fix` (`models/dualprompt.py:30`; consumo `models/dualprompt_utils/prompt.py:121-122`); CODA B23 `mu` — `loss_ce + mu·loss_prompt` (`models/coda_prompt.py:103`). Todas CONFIRMADAS.

---

## 3. Clasificación de hallazgos y recuentos

### 3.1 Hallazgos por clase

**CONTRADICE-VALOR — 0.** Ninguna celda contradice su fuente primaria en las verificaciones de §2.2.

**CONTRADICE-CUBO — 0.**

**TRAMPA-CONFIG — 8** (valor silencioso ≠ pre-final; todos con flag de corrección en §1.3):
1. **[ALTA · los tres · NUEVO en su consecuencia operativa]** `dataset_config` silencioso: D41-d prohíbe lanzar con `model_config best` en bloque, pero sin `best` el YAML de dataset cae a `default.yaml` (normalización ImageNet, transforms distintos, n_epochs 20) — `main.py:177-186`. La `configuracion_final` DEBE llevar `--dataset_config l2p` / `--dataset_config coda_prompt` explícito; los transforms/normalización **no son fijables por flags individuales**, solo por selección de YAML. El mecanismo estaba documentado (fuentes.md §6.3; CODA-A09); la obligación operativa post-D41-d no estaba consignada.
2. [ALTA · L2P] `batchwise_prompt=1` en best ≠ final 0 (`models/config/l2p.yaml:2`).
3. [ALTA · Dual] `batchwise_prompt=1` en parser ≠ final 0; además activo solo en test (§1.1-2).
4. [ALTA · Dual] `length=5` en parser ≠ final 20; best no lo fija (`models/dualprompt.py:46`).
5. [ALTA · L2P/Dual] LR nominal best 0,03 ≠ nominales D40 0,12/0,02; mutación runtime ×batch/256 persistida en args (guarda D40 vigente).
6. [MEDIA · los tres] batch 128 silencioso ≠ 64; doble efecto en Dual/L2P vía reescalado de LR.
7. [MEDIA · los tres] `n_epochs` silencioso (5/5/20) ≠ peldaño de la escalera.
8. [MEDIA · los tres · NUEVO en su consecuencia operativa] `--seed` NO permuta clases; «orden por semilla» (D19) exige `--permute_classes 1` (`datasets/utils/continual_dataset.py:228-232`). Sin él, las 5 semillas compartirían el orden natural y D19 quedaría incumplida en silencio.

**DEPENDENCIA-NO-REGISTRADA — 3:**
1. [MEDIA · los tres] El olvido (F_N) del protocolo §11 no se calcula sin `--enable_other_metrics 1` (`utils/args.py:382-383`; `utils/training.py:353-363`). Las tablas lo señalan (S-13/A14-A16); falta consignarlo como flag de la configuración final o asignarlo al script de agregación (los accuracies por tarea sí se registran siempre y permiten calcularlo offline).
2. [BAJA · CODA] `optim_mom=0.9` del YAML best es inerte con Adam (§1.2-4); registrado en B02, se re-confirma; solo mordería si se cambiara el optimizador.
3. [BAJA · CODA] Cableado del scheduler: guard `scheduler`/`custom_scheduler` asimétrico e inalcanzable (§1.2-8); con D39 la conexión va por `begin_epoch`; sin efecto funcional.

**NO-CONFIGURABLE — 74 filas** (L2P 28, Dual 25, CODA 21) portan la marca en las tablas; contraste de código sin falsos positivos detectados. Ninguna de ellas requiere flag: su valor pre-final coincide con la conducta hardcodeada (esa es precisamente la condición B-MARCO/D41-B2 bajo la que se cerraron).

**CONFIRMADO — el resto del censo:** 231/231 filas pasan test de cubo y coherencia; 16/16 CONFLICTO→PAPER, 4/4 SIN_FUENTE y las filas S/grupo re-verificadas contra fuente primaria sin desviación. Supuestos estructurales confirmados: solo LayerNorm (sin estadísticos de batch), CODA sin ramas batchwise, fórmula del coseno Mammoth ≡ oficial, semántica de step D39 ≡ oficial, propagación limpia de `n_epochs` (única derivada: K de CODA), enmascarado a vistas idéntico en los tres.

**Notas menores (sin clase, cosméticas):** OBS-04 de `valores_coda.md` describe el estado pre-D39 sin remitir a D39 (E08 sí remite); `models/dualprompt.py:97` asigna un generator a `param_groups` justo antes de recrear el optimizador (inocuo).

### 3.2 Recuentos

| Clase | Total | L2P | DualPrompt | CODA | Transversales |
|---|---:|---:|---:|---:|---:|
| CONTRADICE-VALOR | 0 | 0 | 0 | 0 | — |
| CONTRADICE-CUBO | 0 | 0 | 0 | 0 | — |
| TRAMPA-CONFIG | 8 | 3 (¤2,5,6) | 4 (¤3,4,5,6) | 2 (¤6,7) | 4 (¤1,6,7,8) |
| DEPENDENCIA-NO-REGISTRADA | 3 | — | — | 2 | 1 |
| NO-CONFIGURABLE (filas) | 74 | 28 | 25 | 21 | — |
| CONFIRMADO (filas del censo) | 231/231 | 92/92 | 83/83 | 56/56 | — |
| ELEVADA-AL-ACTA | 0 | 0 | 0 | 0 | — |

(¤n remite a la numeración de §3.1; las trampas 1, 6, 7 y 8 afectan a los tres métodos y se cuentan una vez en Transversales y en cada método al que obligan a flag.)

### 3.3 Lo que el Acta de configuración debe recoger (descripción, no decisión)

1. La `configuracion_final` de cada método necesita, además de los valores de las tablas, los flags de corrección de §1.3 — en particular `--dataset_config` explícito (trampa ¤1), `--permute_classes 1` junto a `--seed` (trampa ¤8) y la decisión sobre `--enable_other_metrics` u olvido offline (dependencia 1).
2. El piloto de receta final (D41-h) con volcado contrastado valor a valor detectaría cualquiera de estas trampas si se colara: el mecanismo de salida de Fase D ya cubre el riesgo residual.
3. Ningún valor final del censo requiere reapertura: 0 contradicciones, 0 elevaciones.
