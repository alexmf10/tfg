# Cola consolidada de revisión humana

## Estado formal de las tres tablas

La validación programática de `valores_l2p.md`, `valores_dualprompt.md` y `valores_coda.md` da **PASS** para la estructura normativa:

| método | filas | A | B | EJE | 12 columnas | `valor final` escrito / vacío | A/EJE sin caso/propuesta | casos B válidos |
|---|---:|---:|---:|---:|---|---|---|---|
| L2P | 92 | 24 | 46 | 22 | sí | 92 / 0 | 46/46 | C1=19; C2=0; C3=7; C4=18; C5=2 |
| DualPrompt | 83 | 26 | 47 | 10 | sí | 83 / 0 | 36/36 | C1=22; C2=0; C3=7; C4=17; C5=1 |
| CODA-Prompt | 56 | 20 | 24 | 12 | sí | 56 / 0 | 32/32 | C1=12; C2=2; C3=2; C4=7; C5=1 |

La escritura final conserva las 12 columnas y los casos de Fase A: las **231/231** celdas de `valor final` tienen cierre explícito. El censo de 65 filas quedó completado el 5-ago (L2P 27, DualPrompt 20, CODA-Prompt 18) con recuento C1-DECISIÓN 4, A-COMÚN 24, A-FASE-D 3, A-PROTOCOLO 12, EJE-DERIVADO 0, EJE-NO-USADO 19 y B-MARCO 3. Quedan **0 filas `REVISAR-MAESTRO`**. Los `CASO 3` conservan su marca histórica `REVISAR` como trazabilidad, no como estado abierto; los `CASO 4`, `SOLO_PAPER` o `SOLO_REPO`; los `CASO 5`, `DEFAULT_MAMMOTH`. Las asimetrías X-01…X-09 siguen cerradas por la sanación 4-ago / D34 rev.

## Registro de cierre de Fase B — 2026-08-05

Este ledger es el estado vigente y sustituye cualquier palabra «abierto» o «pendiente de reunión» que describiera la entrada a la reunión. No borra el motivo ni la evidencia histórica. `CERRADO` significa cierre contra D40–D42 y el triaje completado; no queda ninguna fila fuera de la configuración por `REVISAR-MAESTRO`.

### Conflictos y grupos

| entradas | cierre |
|---|---|
| CP-01, CP-02, CP-03, CP-05 | **CERRADO→D41-B4**; en CP-02 el dominio del voto queda `NO_APLICA` al fijarse selección por instancia. |
| CP-04, CP-07 | **CERRADO→D40**. |
| CP-06, CP-08 | **CERRADO→C1.2/C1.1 (decisión del usuario, 5-ago)**: matching Dual en forma `CE−λ·sim`/máximo y betas CODA `(0,9, 0,999)`. |
| CP-09 | **CERRADO→C1.3/D41-B4**; la fórmula queda inaplicada con `lambda_ortho=0` y `mu=0`. |
| GM-01…GM-11 | **CERRADO→D41-B6**; GM-03/06/09 añaden D31/D15/D39/D40 y GM-11 añade D41-B4. |

### Sensibilidades consolidadas

| entradas | cierre |
|---|---|
| SA-01, SA-03, SA-05, SA-07, SA-17, SA-18, SA-28 | **CERRADO→D41-B2**. |
| SA-02, SA-19, SA-23, SA-26 | **CERRADO→D41-B5**; SA-02 añade D41-B3 para la rama manual. |
| SA-11 | **CERRADO→D28/D41-B2**; reanudación/loadcheck pasa como guarda de Fase D. |
| SA-04, SA-06, SA-13, SA-14, SA-16, SA-24, SA-29 | **CERRADO→D41-B4/D41-B6/C1**; las inicializaciones efectivas de Mammoth se cierran como B-MARCO y Pre-T/matching por C1.2/C1.4. |
| SA-08 | **CERRADO→D40**. |
| SA-09, SA-10 | **CERRADO→D41-B3**. |
| SA-15 | **CERRADO→D41-B4/D41-B2**. |
| SA-12, SA-22 | **CERRADO→D31/D15/D39/D40/D41-B1**, según método. |
| SA-20 | **CERRADO→D15/D39/D41-B3**. |
| SA-21 | **CERRADO→D19/D41-B3**. |
| SA-25 | **CERRADO→D31/D41-a**. |
| SA-27 | **CERRADO→D19**. |
| SM-01, SM-02, SM-07, SM-09…SM-14, SM-17, SM-20, SM-21, SM-23, SM-24, SM-27 | **CERRADO→D41-B2/D41-B4/D41-B6**, según procedencia ya escrita en cada fila. |
| SM-03 | **CERRADO→D19/D41-B3**. |
| SM-04, SM-16, SM-26 | **CERRADO→D41-B5/§9**. |
| SM-05, SM-15 | **CERRADO→D31/D41-a**. |
| SM-06, SM-22 | **CERRADO→D25/D26/D28/D41-B1**. |
| SM-08, SM-28 | **CERRADO→D41-B3**; SM-28 añade D39. |
| SM-18 | **CERRADO→D41-B2**. |
| SM-19 | **CERRADO→D19**. |
| SM-25 | **CERRADO→D41-B4**. |
| SB-01 | **CERRADO→D41-B4**. |
| SB-02, SB-04, SB-05 | **CERRADO→D41-B2/D41-B4**. |
| SB-03 | **CERRADO→D41-B2/D41-B4**. |

### CD, NC, HB y FD

| entradas | cierre |
|---|---|
| CD-01 | **CERRADO→D34 rev.** |
| CD-02 | **CERRADO→D34 rev./D41-B2**. |
| CD-03 | **CERRADO→D31/D41-a**. |
| CD-04 | **CERRADO→D28/D41-B2**; la reanudación con LR mutado pasa como guarda de Fase D. |
| NC-01…NC-03 | **CERRADO→D41-B2/B4/B5** según arquitectura, receta y checkpoint. |
| NC-04 | **CERRADO→D31/D41-a**. |
| NC-05…NC-08 | **CERRADO→D41-B2/B4/B6/D25/D28** según la procedencia ya registrada. |
| NC-09 | **CERRADO→D39/D41-B2**. |
| NC-10 | **CERRADO→D41-B3**. |
| NC-11 | **CERRADO→D41-B3** para overrides L2P; **D41-B2** para conductas fijas Dual/CODA. |
| NC-12 | **CERRADO→D19/D41-B3**. |
| NC-13 | **CERRADO→D41-B5/§9**. |
| HB-01…HB-07 | **CERRADO→D41-B2/B4/B5/B6/D25/D28**, según su referencia consolidada. |
| HB-08 | **CERRADO→D40**. |
| HB-09 | **CERRADO→D39/D41-b**. |
| HB-10 | **CERRADO→D41-B3**. |
| HB-11 | **CERRADO→D31/D41-a/D41-B2**. |
| HB-12 | **CERRADO→D41-c/D41-d/D41-B5**. |
| HB-13 | **CERRADO→D19/D41-B3**. |
| HB-14 | **CERRADO→D41-B4/B6**. |
| HB-15 | **CERRADO→D41-B3**. |
| HB-16, HB-17 | **CERRADO→D41-B2/B4/B6**. |
| HB-18 | **CERRADO→D41-B2/D25**. |
| HB-19 | **CERRADO→D41-B3** para `disable_log=1`; reanudación **D28/D41-B2**. |
| FD-01…FD-03 | **CERRADO→D41-B5/§9**; ImageNet-R fuera por defecto. |
| FD-04 | **CERRADO→D39/D41-b**. |

### Experimentos decididos y encolados

**FC-01 — A/B CODA scheduler truncado `T=20` frente a recalculado `T=E`: DECIDIDO Y ENCOLADO por el usuario (5-ago), dos corridas cortas y 1 semilla; script por Modo E y ejecución del usuario en galatzo durante su hueco de Fase C/D, nunca antes (D17/D39/D41-j/D42-a).** La pasada de acoplamientos no produjo otro A/B empírico inequívoco.

**FC-02 — A/B del learning rate en L2P y DualPrompt — receta D40 (aplicado = publicado; nominales 0,12/0,02) frente al aplicado por defecto de Mammoth (nominales 0,03/0,005 → aplicados 0,0075/0,00125): DECIDIDO Y ENCOLADO por el usuario (5-ago), cuatro corridas cortas (E=5, batch 64) y 1 semilla; script por Modo E y ejecución del usuario en galatzo durante su hueco de Fase C/D, nunca antes (D40/D41-k/D42-a).**

## Contradicciones y asimetrías detectadas — resueltas

| id | referencia | contradicción / asimetría y cierre |
|---|---|---|
| X-01 | `L2P-B24`; `cola_l2p.md` S-19 | S-19 citaba B24 sin figurar en sus flags. Cierre: B24 incorpora `S-19 MEDIA`. **RESUELTO — sanación 4-ago / D34 rev.** |
| X-02 | `L2P-B19`, `L2P-B34`; `cola_l2p.md` NC-07/NC-09 | Cierre: NC-07 explicita que B19 es `PARCIALMENTE_CONFIGURABLE`; NC-09 registra B34 como ausencia estructural (`NO_APLICA`). **RESUELTO — sanación 4-ago / D34 rev.** |
| X-03 | L2P `BHV-06`; `HB-06` | Cierre: HB-06 incorpora B24/B25 y declara que el cruce con HB-05 es intencionado (origen BHV-05/BHV-06). **RESUELTO — sanación 4-ago / D34 rev.** |
| X-04 | `CODA-B19`; `comportamiento_coda.md` C-10; `cola_coda.md` S-08/S-19 | Cierre: B19 separa `S-08 ALTA` para el grupo completo, `S:MEDIA` para preasignación/floor y `S-19 BAJA` para el resto no usado. **RESUELTO — sanación 4-ago / D34 rev.** |
| X-05 | CODA C-08; A05; S-06/Q-09 | Cierre: S-06 enlaza C-08 y su S MEDIA condicional para las rutas custom defectuosas. **RESUELTO — sanación 4-ago / D34 rev.** |
| X-06 | CODA E04/E05/E08/E10, B02, B18/B19, B23, A07/A08 | Cierre: las filas incorporan `PARCIALMENTE_CONFIGURABLE`/`NO_CONFIGURABLE`, según corresponda, con alcance de faceta; A17 hace explícito el alcance de `code_optimization`. **RESUELTO — sanación 4-ago / D34 rev.** |
| X-07 | CODA E09/B24 | Cierre: E09 y B24 incorporan patrón y ámbito de las búsquedas negativas en repo y Mammoth. **RESUELTO — sanación 4-ago / D34 rev.** |
| X-08 | Dual A «Backbone»; B semántica Pre-T, mask train, init, LR efectivo, pooling y persistencia | Cierre: cada faceta incorpora etiqueta literal y alcance sin declarar no configurable el parámetro entero cuando existen controles parciales. **RESUELTO — sanación 4-ago / D34 rev.** |
| X-09 | `L2P-A19`, Dual «Precisión», `CODA-A17` | Cierre: `code_optimization` queda en cubo A + `LOW` en los tres métodos; se retira `CUBO_DUDOSO` de L2P. **RESUELTO — sanación 4-ago / D34 rev.** |

## `CONFLICTO→PAPER` y grupo acoplado

Cada línea agrupa los conflictos que deben revisarse en bloque; no altera el valor propuesto de Fase A.

| id | método / grupo | miembros `CASO 3` | cuestión que llegó pendiente y evidencia |
|---|---|---|---|
| CP-01 | L2P / G-PROMPT | `L2P-B06`, `B07`, `B08`, `B38` | Longitud5 vs10, top-k5 vs4 y derivados25 tokens/46.080 params frente a40/84.480; paper `L2P_arxiv.tex:565-567`, repo `configs/cifar100_l2p.py:99-101`. |
| CP-02 | L2P / G-QUERY | `L2P-B15` | Lookup por instancia del paper frente a voto batchwise de repo/Mammoth; paper `:122,313-323`, R `models/prompt.py:210-216`. |
| CP-03 | L2P / G-LOSS | `L2P-B20` | λ=0,5 paper frente a1,0 repo; `L2P_arxiv.tex:569-570`, R `configs/cifar100_l2p.py:121-122`. |
| CP-04 | L2P / G-OPT | `L2P-B29` | LR aplicado0,03 del paper frente a0,015 de ambos códigos con batch128; R `train_continual.py:637-648`, M `models/l2p.py:66`. |
| CP-05 | DualPrompt / G-PROMPT | E-length CIFAR; semántica Pre-T; batchwise CIFAR | Paper propone E=20, longitud total K/V y selección por muestra; repo/Mammoth usan5, L por rama y voto; filas B correspondientes, OBS-08/09/11. |
| CP-06 | DualPrompt / G-MATCH | métrica/signo del matching | Literal paper: similitud+argmin+`CE+λγ`; códigos: max coseno+`CE−λsim`; `dual_prompt_camera_ready.tex:315-322,458-464`. |
| CP-07 | DualPrompt / G-OPT | LR nominal CIFAR; LR efectivo CIFAR e ImageNet-R | Paper0,005 sin escalado; códigos escalan por batch, topología oficial indeterminada y Mammoth0,015 en CIFAR; OBS-14. |
| CP-08 | CODA-Prompt / G-OPT | `CODA-B02` | El paper repite β1=0,9 y β1=0,999; no forma un par ejecutable. Repo/Mammoth usan0,9/0,999; `sections/7_appendix.tex:5`, OBS-08. |
| CP-09 | CODA-Prompt / G-ORTHO | `CODA-B21` | Norma-2 del paper frente a MSE medio de repo/Mammoth, sin explicación autoral; `sections/4_method.tex:49-54`, R `models/zoo.py:178-198`. |

## Grupos mixtos

Las fuentes se indican solo para las filas B con propuesta; A/EJE permanecen sin propuesta por norma.

| id | grupo | miembros y fuente de `valor propuesto` | estado |
|---|---|---|---|
| GM-01 | L2P / G-PROMPT | **Coinciden:** B01,B02,B04,B05. **Paper:** B06–B08,B38. **Repo:** B03,B09,B39. | **CERRADO→D41-B6**; geometría e inicialización ratificadas como mezcla operativa. |
| GM-02 | L2P / G-QUERY | **Coinciden:** B10,B12–B14,B16–B17. **Paper:** B15. **Repo:** B11,B41,B44. | **CERRADO→D41-B6/B4**; selección por instancia vuelve inerte el dominio de voto. |
| GM-03 | L2P / G-OPT | A10–A12/E01–E16/E21–E22: sin propuesta. En B: **coinciden:** B26,B28; **paper:** B27,B29; **repo:** B30–B32; **Mammoth:** B45. | **CERRADO→D41-B1/B6/D31/D15/D39/D40**. |
| GM-04 | L2P / G-HEAD | **Coinciden:** B21–B24,B42. **Repo:** B25,B33–B37,B43. **Mammoth:** B40. | **CERRADO→D41-B2/B4/B6**. |
| GM-05 | L2P / G-LOSS | **Coinciden:** B18,B19. **Paper:** B20. **Repo:** B46. | **CERRADO→D41-B4/B6**; λ y reducción permanecen acopladas. |
| GM-06 | DualPrompt / G-OPT | 14 A/EJE: sin propuesta. En B: **coinciden:** optimizer y LR nominal IMR; **paper:** LR nominal CIFAR, LR efectivo CIFAR/IMR, betas; **repo:** WD, clipping, reset. | **CERRADO→D41-B1/B6/D31/D15/D39/D40**. |
| GM-07 | DualPrompt / G-PROMPT | 2 A batch: sin propuesta. En B: **13 coincidentes** (composición, capas/tipo/pool y estados compatibles); **paper:** E-length CIFAR, semántica Pre-T, batchwise CIFAR; **repo:** candidatos de test, init prompt, copia E; **Mammoth:** permute-fix false. | **CERRADO→D41-B4/B6/C1.4**; Pre-T usa longitud 20 por rama K/V. |
| GM-08 | DualPrompt / G-MATCH | **8 coincidentes:** top-k, mask train, query/forward, key aprendible, activación y λ; **paper:** signo/métrica; **repo:** candidatos de test, init key, no copia key. | **CERRADO→D41-B4/B6/C1.2**; forma final `CE−λ·sim`, selección por máximo. |
| GM-09 | CODA / G-OPT | A10/E01–E12: sin propuesta. En B: **coinciden:** B01,B03; **paper:** B02; **repo:** B04,B06. | **CERRADO→D41-B1/B6/D31/D15/D39/D40/C1.1**; betas `(0,9, 0,999)`. |
| GM-10 | CODA / G-PROMPT | **Coinciden:** B09–B14,B16,B18. **Repo:** B15,B17,B19. **Paper:** B24. | **CERRADO→D41-B4/B6**. |
| GM-11 | CODA / G-ORTHO | **Explicación autoral/repo:** B20,B22. **Paper:** B21. **Mammoth:** B23. | **CERRADO→D41-B4/B6/C1.3**; fórmula inaplicada con pesos cero. |

Desglose nominal de los tres grupos DualPrompt, cuyas filas no tienen identificador alfanumérico en `valores_dualprompt.md`:

- **G-OPT.** Sin propuesta: batch CIFAR, batch ImageNet-R, acumulación, lote incompleto, precisión y las nueve filas EJE salvo cadencia G-EVAL. Coinciden: `Optimizador`, `Learning rate nominal/configurado — ImageNet-R`. Paper: `Learning rate nominal/configurado — CIFAR`, los dos `Learning rate efectivo` y `Betas de Adam`. Repo: `Weight decay`, `Gradient clipping`, `Reinicio del optimizador entre tareas`.
- **G-PROMPT.** Sin propuesta: las dos filas A de batch. Coinciden: `Composición G/E`, long/capas G, longitud E ImageNet-R, capas E, tipo de inserción, K/V separados, pool, top-k, selección train, batchwise ImageNet-R, componentes entrenables y conocimiento de T. Paper: longitud E CIFAR, semántica Pre-T y batchwise CIFAR. Repo: candidatos en test, inicialización G/E y copia E. Mammoth: fix de permutación false.
- **G-MATCH.** Coinciden: top-k, selección train, query, forward extra, task keys, activación de matching, λ y pérdida CE+matching. Paper: métrica/signo. Repo: candidatos en test, inicialización de task keys y no copia de key.

DualPrompt G-HEAD no se marca mixto: sus propuestas son solo-repo o coincidentes; la diferencia repo–Mammoth del universo de logits está en A/comportamiento y no mezcla `valor propuesto`.

## Propuestas S — ALTA

| id consolidado | método / S original | objeto y justificación | referencia |
|---|---|---|---|
| SA-01 | L2P S-01 | Backbone/CLS/congelación hardcodeados definen representación. | `L2P-A01`, `B22`, `B23`; `cola_l2p.md` S-01. |
| SA-02 | L2P S-02 | Checkpoint no identificado por paper y rama manual defectuosa. | `L2P-A02`; L2P S-02. |
| SA-03 | L2P S-03 | Familia y punto de prompting no configurables. | `L2P-B01`, `B02`; L2P S-03. |
| SA-04 | L2P S-04 | M/Lp/N cambian tokens y parámetros en gran magnitud. | `L2P-B05`–`B08`, `B38`; L2P S-04. |
| SA-05 | L2P S-06 | Query/matching exige segundo ViT y coseno fijo. | `L2P-B12`–`B14`; L2P S-06. |
| SA-06 | L2P S-07 | Instancewise vs voto y dominio16/128 alteran predicción. | `L2P-B15`, `B44`; L2P S-07. |
| SA-07 | L2P S-08 | Máscara/cabeza/logits100 vs vistos cambian CE/accuracy. | `L2P-A15`, `B24`, `B25`; L2P S-08. |
| SA-08 | L2P S-14 | LR aplicado difiere por factor2. | `L2P-B28`, `B29`; L2P S-14. |
| SA-09 | L2P S-20 | `fitting_mode=time` no termina. | `L2P-E21`; L2P S-20. |
| SA-10 | L2P S-21 | Early stopping sin validación puede seleccionar sobre test. | `L2P-E12`; L2P S-21. |
| SA-11 | L2P S-22 | `loadcheck` no restaura estado completo/cursor. | `L2P-A24`; L2P S-22. |
| SA-12 | Dual S-01 | G-OPT: LR, batch, updates, scheduler y `drop_last` están acoplados. | Dual S-01; filas A/EJE/B G-OPT. |
| SA-13 | Dual S-02 | G-PROMPT: E-length y semántica K/V discrepan. | Dual S-02; DP-BHV-01. |
| SA-14 | Dual S-03 | Selección por muestra vs voto local24/Mammoth128. `distributed=dp` cambia el dominio del voto (por shard); cubierto por esta S. En CODA, voto batchwise `NO_APLICA`. | Dual S-03; DP-BHV-03. |
| SA-15 | Dual S-04 | Reshape K/V–batch defectuoso y typo del YAML. | Fila permute fix; DP-BHV-07. |
| SA-16 | Dual S-05 | Signo/métrica del matching ambiguos y no configurables. | Fila matching; DP-BHV-04. |
| SA-17 | Dual S-06 | Segundo ViT y candidatos futuros no limitables. | Filas query/candidatos; DP-BHV-02. |
| SA-18 | Dual S-07 | Cabeza/máscara y universo de logits difieren. | Fila A protocolo y B head; DP-BHV-05. |
| SA-19 | Dual S-08 | Checkpoint/preprocesado/congelación determinan features. | Filas A checkpoint/transforms; DP S-08. |
| SA-20 | Dual S-09 | Duración/validación/warmup/cadencia; time no termina. | EJE; DP-BHV-11/15. |
| SA-21 | Dual S-10 | Orden/seed no publicados y rutas custom defectuosas. | Filas A order/seed; DP-BHV-16. |
| SA-22 | CODA S-01 | LR/scheduler/n_epochs/K/scope: K=1 aborta y LR Mammoth queda constante. | CODA E01–E08; C-01. |
| SA-23 | CODA S-02 | Arquitectura/checkpoint hardcodeados y artefacto no demostrablemente común. | CODA A01/A02; C-05. |
| SA-24 | CODA S-03 | G-ORTHO mezcla GS, fórmula y doble peso con ablation grande. | CODA B20–B23; C-03/C-04. |
| SA-25 | CODA S-04 | `virtual_bs_iterations>1` descarta gradientes, no acumula. | CODA A11; C-02. |
| SA-26 | CODA S-05 | `model_config=best` cambia normalización/config de datos. | CODA A09; C-06. |
| SA-27 | CODA S-06 | Orden de clases afecta resultados y receta usa natural. | CODA A05; C-07. |
| SA-28 | CODA S-07 | Máscara de logits calificada crucial y hardcodeada. | CODA B08; comprobación de comportamiento. |
| SA-29 | CODA S-08 | Pool/longitud/capas/partición determinan capacidad. | CODA B09–B19; S-08. |

## Propuestas S — MEDIA

| id consolidado | método / S original | objeto y justificación | referencia |
|---|---|---|---|
| SM-01 | L2P S-05 | Posiciones de prompts difieren y están hardcodeadas. | `L2P-B03`; BHV-02. |
| SM-02 | L2P S-09 | Escala/init de prompts/keys/cabeza no equivalente. | `L2P-B09`, `B11`, `B36`. |
| SM-03 | L2P S-10 | Orden/seed/cache de ruido pueden reutilizar estado. | `L2P-A05`, `A06`, `A23`. |
| SM-04 | L2P S-11 | JPEG/defaults de crop difieren. | `L2P-A07`, `A08`. |
| SM-05 | L2P S-12 | Sin acumulación, acoplada a batch/LR. | `L2P-A12`. |
| SM-06 | L2P S-13 | Forgetting/runs/cadencia no coinciden por defecto. | `L2P-A16`, `E17`. |
| SM-07 | L2P S-15 | Betas/epsilon/reset de Adam no cerrados. | `L2P-B27`, `B32`, `B45`. |
| SM-08 | L2P S-17 | CLI L2P contiene flags inertes/rotos. | `L2P-B04`, `B10`, `B16`, `B19`, `B23`, `B40`, `B41`. |
| SM-09 | L2P S-18 | Normalización/temperatura/dropout/head fijos. | `L2P-B33`–`B37`, `B42`, `B43`. |
| SM-10 | L2P S-19 | Persistencia y reducción pull fijas; top-k escala loss. | `L2P-B17`–`B19`, `B24`, `B46`; X-01 resuelto por sanación 4-ago / D34 rev. |
| SM-11 | L2P S-23 | Warmup repo0 y no conectado en Mammoth. | `L2P-E11`. |
| SM-12 | Dual S-11 | Init/copia de E y keys no documentada/equivalente. | Filas init/copia; DP-BHV-08. |
| SM-13 | Dual S-12 | Reset Adam e init de cabeza distintos. | Filas reset/head; DP-BHV-06. |
| SM-14 | Dual S-13 | Betas solo paper y no configurables. | Fila betas. |
| SM-15 | Dual S-14 | Ausencia de acumulación. | Fila A acumulación. |
| SM-16 | Dual S-15 | Receta ImageNet-R `best` ausente. | Filas IMR; OBS-02. |
| SM-17 | Dual S-16 | Backbone nominal construido y descartado. | Fila A backbone; DP-BHV-10. |
| SM-18 | Dual S-17 | Override `freeze` mal tipado. | Fila componentes; DP-BHV-10. |
| SM-19 | CODA S-09 | Seed y cinco trials no reproducidos por receta. | CODA A06; C-07. |
| SM-20 | CODA S-10 | `drop_last` da39 vs40 lotes. | CODA A10; C-09. |
| SM-21 | CODA S-11 | Cabeza física preasignada vs expansión conceptual. | CODA A12; C-10. |
| SM-22 | CODA S-12 | `A_N/F_N` no equivalentes por defecto. | CODA A14; C-12. |
| SM-23 | CODA S-13 | Betas erróneas/`optim_mom` inerte. | CODA B02; C-11. |
| SM-24 | CODA S-14 | Query/cosenos crudos/doble forward fijos. | CODA B14–B17. |
| SM-25 | CODA S-15 | Sweep/transferencia de HP solo paper. | CODA B24. |
| SM-26 | CODA S-16 | Augment/geometría IMR difieren. | CODA A07–A09; C-14. |
| SM-27 | CODA S-17 | Selección prompt+cabeza y freeze no configurables. | CODA B05. |
| SM-28 | CODA S-18 | `fitting_mode=time` no termina y no adapta K. | CODA E10; C-13. |

## Propuestas S — BAJA

| id consolidado | método / S original | objeto y justificación | referencia |
|---|---|---|---|
| SB-01 | L2P S-16 | λ pull0,5 vs1,0; paper reporta poca sensibilidad. | `L2P-B20`. |
| SB-02 | Dual S-18 | CE/norm/temperatura/dropout coinciden efectivamente, pero no son configurables. | Filas loss/head. |
| SB-03 | Dual S-19 | Pooling/capas solapadas/no-prefix/KV compartido; rutas fuera del preset. | DP-BHV-12. |
| SB-04 | CODA S-19 | Resto `floor(M/N)` al cambiar pool/tareas; X-04 resuelto con los tres alcances explícitos en B19. | CODA B19; sanación 4-ago / D34 rev. |
| SB-05 | CODA S-20 | Tipo CE solo repo y no configurable en Mammoth. | CODA B07. |

## `CUBO_DUDOSO`

| id | referencia | cuestión abierta |
|---|---|---|
| CD-01 | `L2P-A19` | **RESUELTO:** cubo A, `LOW`; referencia D34 rev./test §3. |
| CD-02 | Dual fila A «Precisión»; `CODA-A17`; X-09 | Decidir conjuntamente el mismo concepto: los otros dos métodos no llevan la marca, aunque la clasificación es idéntica. |
| CD-03 | `CODA-A11` | Confirmar si `virtual_bs_iterations`, específico del método, permanece A por alterar batch efectivo o debe ir a B. No se cambia sin revisión. |
| CD-04 | `L2P-A24`; entorno A-C28 | Reanudación obtuvo fila solo en L2P por evidencia demostrada; decidir si debe auditarse simétricamente para Dual/CODA o quedar como infraestructura allí. |

## Valores y conductas `NO_CONFIGURABLE`

| id | métodos / referencia | cierre solicitado al entrar en reunión |
|---|---|---|
| NC-01 | L2P/DP/CODA backbones y checkpoints; CB-01 | Aceptar los ViT internos/artefactos o autorizar cambio de código. |
| NC-02 | L2P/DP/CODA doble forward/query; CB-03 | Aceptar coste y matching/candidatos fijos. |
| NC-03 | L2P B01–B03; Dual Pre-T; CODA capas/type; CB-02 | Aceptar familia, semántica de longitud y capas hardcodeadas donde aplique. |
| NC-04 | L2P A12, Dual acumulación, CODA A11; CB-11 | Aceptar un update/batch y prohibir `virtual_bs_iterations>1`, o autorizar soporte real. |
| NC-05 | L2P/DP universo de logits y heads; CODA head física; CB-06/07/17 | Fijar protocolo de cabeza/máscara/evaluación. |
| NC-06 | L2P/DP init/copia/reset; CODA reset/GS; CB-04/06/16 | Aceptar estados iniciales y reinicios o parametrizarlos. |
| NC-07 | L2P pull, Dual matching, CODA ortogonalidad; CB-14/16 | Fijar signo, reducción, fórmula y doble peso. |
| NC-08 | L2P/Dual head transforms y pooling; X-02/X-08 | Alcance exacto documentado por X-02/X-08; aceptar las conductas fijas o parametrizarlas sin etiquetar como `NO_CONFIGURABLE` ramas `NO_APLICA` o controles indirectos. |
| NC-09 | L2P/Dual warmup; CODA warmup/cableado scheduler; E-C06–E-C10 | Aceptar ausencia/defecto o autorizar conexión del scheduler. |
| NC-10 | Los tres `fitting_mode=time`; E-C17 | Prohibir el modo o implementar presupuesto/condición de salida. |
| NC-11 | L2P flags inertes; Dual permute/pooling/KV; CODA `optim_mom`; CB-15 | Prohibir opciones engañosas o corregirlas antes de sweeps. |
| NC-12 | Mammoth orden custom; CB-13 | Usar solo permutación simple con seed o reparar las listas custom. |
| NC-13 | ImageNet-R transforms y ausencia de presets; A-C04/A-C11/A-C13 | No presentar defaults de clase como receta hasta una decisión humana. |

## Comportamiento relevante

| id | referencia consolidada | revisión solicitada al entrar en reunión |
|---|---|---|
| HB-01 | `comportamiento.md` CB-01 | Ratificar backbone/checkpoint interno de cada método. |
| HB-02 | CB-02 | Ratificar input prompting L2P y semántica Pre-T/CODA. |
| HB-03 | CB-03 | Aceptar segundo ViT/forward y candidatos futuros DualPrompt. |
| HB-04 | CB-04 | Resolver posiciones, escalas y transferencia de slots. |
| HB-05 | CB-05 | Fijar selección instancewise/batchwise y dominio del voto. |
| HB-06 | CB-06 | Cerrar máscara, persistencia y reset del optimizador. |
| HB-07 | CB-07 | Fijar universo de logits, métricas y cadencia. |
| HB-08 | CB-08 | Elegir LR publicado o precompensar el escalado del código. |
| HB-09 | CB-09 | Resolver bloqueo CODA K=1 y scheduler inerte. El Piloto-0 v2 completa CODA a E=2, pero no prueba E=1 ni que el scheduler avance; `piloto0.md`, «Cierre real». |
| HB-10 | CB-10 | Prohibir early stopping sin validación. |
| HB-11 | CB-11 | Fijar `drop_last` y prohibir falsa acumulación. |
| HB-12 | CB-12 | Elegir tubería exacta y exigir config `best` CODA si procede. |
| HB-13 | CB-13 | Definir lista de seeds/órdenes y evitar rutas custom. |
| HB-14 | CB-14 | Resolver signo/reducción de pérdidas auxiliares. |
| HB-15 | CB-15 | Prohibir/corregir interfaces CLI inertes o defectuosas. |
| HB-16 | CB-16 | Cerrar en bloque la receta ortogonal CODA. |
| HB-17 | CB-17 | Decidir preasignación física, capacidad y restos. |
| HB-18 | CB-18 | Mantener nivel0 o medir dtypes/backend antes de cambiarlo. |
| HB-19 | CB-19 | Prohibir `disable_log=1` en este SHA y fijar política de reanudación. |

## Fuentes incompletas / ejecución no determinable

| id | referencia | estado |
|---|---|---|
| FD-01 | L2P A04/A11/E02 | No existe receta L2P ImageNet-R; excluir o autorizar una receta manual. |
| FD-02 | Dual filas IMR / S-15 | Dataset/defaults parciales, pero falta preset `best` y LR completo. |
| FD-03 | CODA A03/E01 / OBS-09 | Ruta default parcial, sin preset `best` de reproducción. |
| FD-04 | Entorno, veredicto CODA | `n_epochs=1` no es ejecutable en una corrida: `CosineSchedule` exige K>1. Fase B confirma parser E=1, derivación estática `K=1/runtime_valid=false` y frontera pre-modelo del dump; `reconciliacion_fase_b.md`. El Piloto-0 v2 aporta ejecución completa solo para E=2 (`piloto0.md`, «Cierre real»); no cierra el bloqueo E=1. |

## Agenda dirigida post-piloto — reunión de Fase B

Estas entradas consolidan las consultas del 2026-08-03. No cambian el eje, los valores propuestos ni ningún `valor final`.

| id | tipo y filas de referencia | evidencia / cuestión que llega a reunión | estado |
|---|---|---|---|
| PD-01 | **Batch/OOM y árbol D31.** Entorno `A-C15`, `A-C17`; `L2P-A10/A12`; DualPrompt «Batch de train/eval CIFAR» y «Acumulación de gradiente»; `CODA-A10/A11`. | Batch128 está cerrado por D31: hecho previo 128 OOM/64 cabe (`PLAN_MAESTRO.md`, §10), OOM v1 reproducido en Mammoth@cda7f236 (`infra/servidor.md`) y uniformidad común. El v2 demuestra ahora una secuencia completa a batch real64 con los tres bajo `best` diagnóstico; códigos0, diez tareas y `logs.pyd` en `piloto0.md`, «Cierre real». CODA tuvo el mayor pico *allocated* observado (8.705,77 MiB), con la limitación de que no es VRAM total. Q1 determina que no hay acumulación real común por CLI/config (`entorno.md:34`, `comportamiento.md:90-96`). | **DECIDIDO→D31/D40/D41-a**. |
| PD-02 | **Peldaño mínimo del eje.** Entorno `E-C01`, `E-C08`; `CODA-E01/E02/E05`; FD-04/HB-09. | D39 validado permite E=1 y conserva la semántica oficial; escalera común `{1,5,20}`. | **DECIDIDO→D15/D39/D41-b**. |
| PD-03 | **Preprocesado común.** Entorno `A-C10`, `A-C12`, `A-C14`; L2P `A07/A08`; filas DualPrompt «Augmentación train CIFAR»/«Preprocesado de evaluación»; CODA `A07-A09`. | Principio común ratificado; checkpoint y tubería concreta se deciden juntos en Fase D con evidencia. | **DECIDIDO→D2/D41-c/D41-B5**. |
| PD-04 | **`model_config=best/default` frente a cascada.** `L2P-B15/B28/B29`; DualPrompt «Learning rate nominal/configurado — CIFAR», «Learning rate efectivo — CIFAR», longitud E y selección batchwise; `CODA-A09/E04-E08`; CP-01/02/04/05/07 y GM-01/03/06/07/09. | La matriz usará `configuracion_final.md` explícita y nunca overlays `best/default` en bloque; D40 precompensa el LR nominal. | **DECIDIDO→D40/D41-d**. |

## Embudo global

### Suma de los tres embudos

| etapa | L2P | DualPrompt | CODA-Prompt | total global |
|---|---:|---:|---:|---:|
| parámetros examinados | 131 | 126 | 90 | **347** |
| parámetros con fila | 92 | 83 | 56 | **231** |
| cubo A | 24 | 26 | 20 | **70** |
| cubo B | 46 | 47 | 24 | **117** |
| EJE | 22 | 10 | 12 | **44** |
| lista agrupada de infraestructura | 24 | 24 | 19 | **67** |
| exclusiones | 15 | 19 | 15 | **49** |

Comprobación: `231 + 67 + 49 = 347`. Los 67 controles de infraestructura conservan multiplicidad por método; la lista normalizada siguiente deduplica aliases, por lo que no debe sumarse de nuevo al embudo.

### Motivos globales de exclusión

- Selectores/placeholders sin valor experimental independiente: model/dataset/config, loaders nominales y aliases ya expandidos.
- Campos declarados sin consumidor o deprecados: opciones offline/recreate/mask/predefined-key y otros no-ops identificados en cada tabla.
- Ramas inactivas o no aplicables: controles SGD bajo Adam, BatchNorm ausente, gaussian mode inerte y variantes ajenas.
- Variantes fuera del objeto: replay/L2P-R, FT/FT++, L2P++/DualPrompt cruzados y smoke tests ImageNet-R.

### Lista agrupada global de infraestructura

- **Datos, rutas y caches:** `base_path`/`dataroot`, `workdir`, `results_path`/`log_dir`, `checkpoint_path`, `cache_path_noisy_labels`, descarga y rutas README, nombre/ruta de config.
- **Dispositivo y paralelismo:** `device`/`gpuid`, `distributed`/DataParallel, `num_workers`/`workers`. Los efectos de topología sobre batch/LR/voto ya tienen filas y S; el selector operativo permanece aquí.
- **Logging y metadatos:** `non_verbose`, `disable_log`, `notes`, `exp_id`/`trial`, W&B name/entity/project, UUID, timestamp, host, git metadata y cadencias de log.
- **Checkpointing y artefactos:** `savecheck`, `loadcheck` cuando no obtuvo fila, `save_checkpoint_mode`, `ckpt_name`, `save_after_interrupt`, `checkpoint_every_steps`, `save_last_ckpt_only`, `max_to_keep`, `overwrite`, guardado de prompts/histogramas.
- **Selección nominal de software:** `learner_type/name`, `model_type/name` y componentes nominales ya resueltos a valores efectivos en otras filas.
- **Diagnóstico/portabilidad:** scripts de descarga, rutas de documentación y controles que solo describen el entorno sin cambiar por sí mismos la receta matemática.

## Recuento de la cola consolidada

- Conflictos `CASO 3`: 16 filas, consolidadas en 9 entradas de grupo (L2P7 + Dual7 + CODA2).
- Grupos mixtos cerrados administrativamente por D41-B6: 11 (L2P5 + Dual3 + CODA3).
- Propuestas S: 62 (29 ALTA + 28 MEDIA + 5 BAJA), sin tope artificial.
- Cubos dudosos: 4 entradas de revisión, una marcada originalmente y tres surgidas de la comparación transversal.
- Hallazgos de comportamiento consolidados: 19, con matriz de cobertura completa en `comportamiento.md`.
