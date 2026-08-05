# Cola de revisión humana — L2P

Esta cola contiene únicamente decisiones humanas derivadas de `auditoria/valores_l2p.md` y hallazgos de `auditoria/comportamiento_l2p.md`. No rellena `valor final` ni ejecuta cambios/experimentos. `R:dd8836e6` identifica el repo oficial @ `dd8836e6e372df29f03d83bf3dc3a806114e9d8e`; `M:cda7f236`, Mammoth @ `cda7f23681f7bffacee460d99e990bc803bccf04`.

## Cierre de Fase B — 2026-08-05

El ledger siguiente es el estado vigente; las tablas posteriores conservan la redacción de entrada a reunión solo como evidencia histórica.

| entradas | cierre |
|---|---|
| C-01…C-05, C-07 | **CERRADO→D41-B4**. |
| C-06 | **CERRADO→D40**: nominal 0,12 → aplicado 0,03 con batch 64. |
| GM-01…GM-05 | **CERRADO→D41-B6**; GM-03 añade D41-B1/D31/D15/D39/D40. |
| S-01 | **CERRADO→D41-B2**. |
| S-02 | **CERRADO→D41-B5/D41-B3**. |
| S-03, S-05 | **CERRADO→D41-B2**. |
| S-04 | **CERRADO→D41-B4/B6**. |
| S-06 | **CERRADO→D41-B2/D25**. |
| S-07 | **CERRADO→D41-B4**; voto batchwise y dominio quedan `NO_APLICA`. |
| S-08 | **CERRADO→D41-B2/D25/D28**. |
| S-09 | **CERRADO→D41-B2/B4/B6**; inicialización de cabeza queda `REVISAR-MAESTRO`. |
| S-10 | **CERRADO→D19/D41-B3**. |
| S-11 | **CERRADO→D41-B5**. |
| S-12 | **CERRADO→D31/D41-a**. |
| S-13 | **CERRADO→D19/D25/D28**. |
| S-14 | **CERRADO→D40**. |
| S-15 | **CERRADO→D41-B2/B4/B6**; epsilon queda `REVISAR-MAESTRO`. |
| S-16 | **CERRADO→D41-B4**. |
| S-17 | **CERRADO→D41-B3**. |
| S-18, S-19 | **CERRADO→D41-B2/B4/B6**. |
| S-20, S-21 | **CERRADO→D41-B3**. |
| S-22 | **CERRADO→D28/D41-B2**; LR–reanudación pasa como guarda de Fase D. |
| S-23 | **CERRADO→D41-B2**. |
| CD-01 | **CERRADO→D34 rev.** |
| FD-01 | **CERRADO→D41-B5/§9**. |
| NC-01…NC-11 | **CERRADO**, respectivamente, por `D41-B2; D41-B5/B3; D31/D41-a; D41-B2; D41-B2; D41-B2/D25; D41-B2/B4; D41-B2/B4/B6; D41-B2/B4; D41-B3; D41-B2`. |
| HB-01…HB-19 | **CERRADO**, respectivamente, por `D41-B2; D41-B2; D41-B2/D25; D41-B4; D41-B2/B4/B6; D41-B1/B2/D25/D28; D41-B2/B4; D41-B2/B5/B3; D41-B3; D19/D41-B3; D40; D15/D39/D41-B3; D31/D41-a; D41-B5; D41-B4; D41-B3; D41-B4; D41-B3; D28/D41-B2`. |

## Conflictos resueltos provisionalmente por la cascada

| ítem | referencia de fila | cuestión que llegó pendiente | evidencia mínima |
|---|---|---|---|
| C-01 | `L2P-B06` | Confirmar longitud propuesta 5 frente a 10 del repo. | Paper `L2P_arxiv.tex:565-567`; R:dd8836e6 `configs/cifar100_l2p.py:100`. |
| C-02 | `L2P-B07` | Confirmar top-k propuesto 5 frente a 4 del repo. | Paper `L2P_arxiv.tex:565-567`; R:dd8836e6 `configs/cifar100_l2p.py:101`. |
| C-03 | `L2P-B08` | Confirmar 25 tokens efectivos frente a 40; depende conjuntamente de C-01/C-02. | Derivación de `L2P-B06` y `L2P-B07`; código R:dd8836e6 `models/prompt.py:234-240`. |
| C-04 | `L2P-B15` | Elegir lookup por instancia (propuesto) o mayoría de batch (repo/Mammoth best). | Paper `L2P_arxiv.tex:122,313-323`; R:dd8836e6 `models/prompt.py:210-216`; M:cda7f236 `models/config/l2p.yaml:1-2`. |
| C-05 | `L2P-B20` | Confirmar λ=0,5 frente a 1,0 del repo. | Paper `L2P_arxiv.tex:569-570`; R:dd8836e6 `configs/cifar100_l2p.py:121-122`. |
| C-06 | `L2P-B29` | Decidir si se reproduce 0,03 publicado o 0,015 realmente aplicado por el código con batch 128. | Paper `L2P_arxiv.tex:562-565`; R:dd8836e6 `train_continual.py:637-648`; M:cda7f236 `models/l2p.py:66`. |
| C-07 | `L2P-B38` | Confirmar 46.080 parámetros pool+key frente a 84.480 del repo. | Derivación de C-01; paper `L2P_arxiv.tex:289-301`; R:dd8836e6 `configs/cifar100_l2p.py:99-103`. |

## Grupos mixtos

| ítem | grupo y filas | cuestión que llegó pendiente | motivo |
|---|---|---|---|
| GM-01 | `G-PROMPT`: `L2P-B01`–`L2P-B09`, `L2P-B38`, `L2P-B39` | Revisar el bloque completo antes de fijar longitud/top-k/inicialización. | La cascada toma longitud/top-k del paper, inicialización del repo y otros miembros coincidentes. |
| GM-02 | `G-QUERY`: `L2P-B10`–`L2P-B17`, `L2P-B41`, `L2P-B44` | Revisar conjuntamente keys, selección batchwise, dominio de voto y clave predefinida. | La selección por instancia viene del paper; inicialización/clave externa/dominio local provienen del código oficial. |
| GM-03 | `G-OPT`: `L2P-A10`–`L2P-A12`, `L2P-B26`–`L2P-B32`, `L2P-B45`, `L2P-E01`–`L2P-E16`, `L2P-E21`, `L2P-E22` | Resolver como receta conjunta batch–LR–Adam–duración. | Betas y LR aplicado propuestos desde paper; decay, clip y reinicio desde repo; epsilon/default y modos provienen de Mammoth. |
| GM-04 | `G-HEAD`: `L2P-B21`–`L2P-B25`, `L2P-B33`–`L2P-B37`, `L2P-B40`, `L2P-B42`, `L2P-B43` | Revisar conjuntamente lectura, CLS, máscara y transformaciones de cabeza. | La mayoría de estados se apoya en paper/repo, pero `global_pool=token` de B40 procede solo del default efectivo Mammoth. |
| GM-05 | `G-LOSS`: `L2P-B18`–`L2P-B20`, `L2P-B46` | Fijar conjuntamente CE, forma/reducción del pull y λ. | La cascada toma λ=0,5 del paper y la reducción exacta `/batch` sin `/k` solo del repo oficial. |

## Propuestas de sensibilidad S

| severidad | parámetro / grupo | método y ámbito | justificación (1-2 líneas) | evidencia |
|---|---|---|---|---|
| ALTA | `S-01` · backbone/CLS/congelación | L2P · ambos | El ViT efectivo y su CLS definen la representación; Mammoth ignora el backbone de CLI y el filtro del optimizador impide ampliar el entrenamiento mediante `--freeze`. | `L2P-A01`, `L2P-B22`, `L2P-B23`; M:cda7f236 `backbone/vit.py:187-207,243-284`; `models/l2p.py:52-67,98-99`. |
| ALTA | `S-02` · checkpoint | L2P · ambos | El paper no identifica fichero y Mammoth fija alias/URL; la rama manual tiene nombres de ruta incompatibles. | `L2P-A02`; R:dd8836e6 `README.md:50-52`; M:cda7f236 `models/l2p_utils/l2p_model.py:39-46`. |
| ALTA | `S-03` · familia/punto de prompting | L2P · ambos | Cambiar input prompt-tuning por prefix multicapa cambiaría el método; Mammoth lo fija sin selector. | `L2P-B01`, `L2P-B02`; paper `L2P_arxiv.tex:253-263`; M:cda7f236 `models/l2p_utils/vit_prompt.py:76-101`. |
| ALTA | `S-04` · `G-PROMPT` M/Lp/N | L2P · CIFAR-100 | Longitud/top-k discrepan y cambian 25→40 tokens y 46.080→84.480 parámetros; el paper identifica M, Lp y N como hiperparámetros clave. | `L2P-B05`–`L2P-B08`, `L2P-B38`; paper `L2P_arxiv.tex:585-595,663-668`. |
| ALTA | `S-06` · query/matching | L2P · ambos | El CLS de un segundo ViT y matching coseno determinan selección y coste; arquitectura/métrica no son configurables. | `L2P-B12`–`L2P-B14`; M:cda7f236 `models/l2p_utils/l2p_model.py:15-37,126-134`. |
| ALTA | `S-07` · batchwise selection y dominio de voto | L2P · CIFAR-100 | Contradice el lookup por instancia y depende del minibatch; además repo vota por shard de 16 y Mammoth default sobre 128. | `L2P-B15`, `L2P-B44`; R:dd8836e6 `train_continual.py:664-716`; M:cda7f236 `main.py:531-542`; `models/l2p_utils/prompt.py:72-85`. |
| ALTA | `S-08` · máscara/cabeza/espacio de evaluación | L2P · CIFAR-100 | La CE usa solo clases actuales, la cabeza persiste y Mammoth evalúa clases vistas frente a 100 logits oficiales; el alcance no es configurable. | `L2P-A15`, `L2P-B24`, `L2P-B25`; R:dd8836e6 `train_continual.py:271-286,338-417`; M:cda7f236 `models/l2p.py:79-102`. |
| ALTA | `S-14` · LR aplicado | L2P · CIFAR-100 | Conflicto de factor 2: 0,03 publicado frente a 0,015 aplicado por repo/Mammoth con batch 128. | `L2P-B28`, `L2P-B29`; paper `L2P_arxiv.tex:562-565`; M:cda7f236 `models/l2p.py:66`. |
| ALTA | `S-20` · `fitting_mode=time` | L2P · ambos | La CLI acepta el modo pero el bucle no implementa presupuesto ni salida, por lo que puede prolongarse sin límite. | `L2P-E21`; M:cda7f236 `utils/args.py:290-295`; `utils/training.py:257-298`. |
| ALTA | `S-21` · early stopping sin validación | L2P · ambos | Si se activa sin split de validación, Mammoth usa test-loaders para paciencia/mejor estado; la ayuda lo desaconseja pero el código no lo impide. | `L2P-E12`; M:cda7f236 `main.py:438-440`; `utils/training.py:183-195,272-296`. |
| ALTA | `S-22` · carga/reanudación de estado | L2P · ambos | El repo solo auto-restaura la última tarea con esta receta; Mammoth `loadcheck` hereda argumentos y carga pesos/resultados, pero no optimizer/scheduler/cursor. Ambas rutas pueden cambiar silenciosamente el punto inicial. | `L2P-A24`; R:dd8836e6 `configs/cifar100_l2p.py:34-38`; `train_continual.py:650-657`; M:cda7f236 `utils/checkpoints.py:118-211,215-247`. |
| MEDIA | `S-05` · posiciones de prompts | L2P · ambos | Repo no añade posiciones a los prompts y Mammoth sí; el comportamiento está hardcodeado. | `L2P-B03`; R:dd8836e6 `models/vit.py:198-245`; M:cda7f236 `models/l2p_utils/vit_prompt.py:47-99`. |
| MEDIA | `S-09` · inicializaciones | L2P · ambos | Escala de prompt/key no demostrada equivalente y kernel de cabeza cero oficial frente a trunc-normal Mammoth; varios valores no son configurables. | `L2P-B09`, `L2P-B11`, `L2P-B36`; M:cda7f236 `models/l2p_utils/prompt.py:22-37`; `backbone/vit.py:463-468`. |
| MEDIA | `S-10` · orden/semilla/cache de ruido | L2P · CIFAR-100 | Mammoth omite seed, sus rutas custom de orden presentan defectos y la cache de ruido sin seed puede reutilizar estado; el `rand_seed` oficial tampoco siembra NumPy. | `L2P-A05`, `L2P-A06`, `L2P-A23`; R:dd8836e6 `libml/input_pipeline.py:632-640`; M:cda7f236 `datasets/utils/continual_dataset.py:212-232,456-459`; `datasets/utils/label_noise.py:46-87`. |
| MEDIA | `S-11` · preprocesado/augmentación | L2P · CIFAR-100 | Repo y Mammoth difieren en recodificación JPEG y defaults del crop; el paper no desambigua. | `L2P-A07`, `L2P-A08`; R:dd8836e6 `libml/input_pipeline.py:426-500`; M:cda7f236 `datasets/configs/seq-cifar100-224/l2p.yaml:5-26`. |
| MEDIA | `S-12` · acumulación de gradiente | L2P · ambos | No hay soporte; un update por batch queda acoplado a batch y al escalado de LR. | `L2P-A12`; R:dd8836e6 `train_continual.py:302-323`; M:cda7f236 `models/l2p.py:91-94`. |
| MEDIA | `S-13` · métricas/cadencia | L2P · CIFAR-100 | Forgetting y tres runs son parte del paper, pero Mammoth no los activa/fija por defecto y evalúa con otra cadencia. | `L2P-A16`, `L2P-E17`; paper `L2P_arxiv.tex:533-534,570`; M:cda7f236 `utils/args.py:377-384`. |
| MEDIA | `S-15` · estado/configuración Adam | L2P · ambos | Betas/epsilon no son configurables y los momentos se reinician por tarea; epsilon exacto además depende de una versión Torch no fijada. | `L2P-B27`, `L2P-B32`, `L2P-B45`; M:cda7f236 `models/l2p.py:71-77`; `models/utils/continual_model.py:285-297`; `requirements.txt:1`. |
| MEDIA | `S-17` · fidelidad de la CLI L2P | L2P · ambos | Booleans `type=bool`, `pull_constraint` sin conversor, mask rota, `freeze` mal tipado y `global_pool`/`predefined_key` ignorados pueden producir ejecuciones distintas a lo solicitado. | `L2P-B04`, `L2P-B10`, `L2P-B16`, `L2P-B19`, `L2P-B23`, `L2P-B40`, `L2P-B41`; M:cda7f236 `models/l2p.py:28-44`; `models/l2p_utils/l2p_model.py:23-37,116-134`. |
| MEDIA | `S-18` · transformaciones antes/de la cabeza | L2P · ambos | Normalización, weight norm, temperatura, dropout, reweight y pre-logits están ausentes o fijos; son cambios directos del clasificador efectivo. | `L2P-B33`–`L2P-B35`, `L2P-B37`, `L2P-B42`, `L2P-B43`; R:dd8836e6 `models/vit.py:461-477`; M:cda7f236 `models/l2p_utils/vit_prompt.py:62-66,106-125`. |
| MEDIA | `S-19` · persistencia y reducción de pérdida | L2P · ambos | Pool/cabeza, CE/forma del pull y reducción exacta están fijados; top-k escala también la magnitud del pull al no dividir por k. | `L2P-B17`–`L2P-B19`, `L2P-B24`, `L2P-B46`; M:cda7f236 `models/l2p_utils/prompt.py:103-110`; `models/l2p.py:71-89`. |
| MEDIA | `S-23` · warmup | L2P · ambos | El repo fija cero y Mammoth no expone ni conecta warmup para L2P; llegó a revisión por poder cambiar la fase inicial de optimización. | `L2P-E11`; R:dd8836e6 `configs/cifar100_l2p.py:43`; M:cda7f236 `utils/args.py:309-330`; `utils/schedulers.py:13-35,57-80`. |
| BAJA | `S-16` · λ del pull | L2P · ambos | Hay conflicto 0,5 frente a 1,0, pero el propio paper reporta poca sensibilidad en un rango amplio. | `L2P-B20`; paper `L2P_arxiv.tex:569-570`; R:dd8836e6 `configs/cifar100_l2p.py:121-122`. |

## Cubo dudoso y fuente incompleta

| ítem | referencia de fila | cuestión que llegó pendiente |
|---|---|---|
| CD-01 | `L2P-A19` | Confirmar si precisión/`code_optimization` queda en cubo A o debe tratarse como control de receta B; actualmente `LOW`, `CUBO_DUDOSO`. |
| FD-01 | `L2P-A04`, `L2P-A11`, `L2P-E02` | Decidir si se excluye ImageNet-R de la ejecución L2P o se autoriza construir manualmente una receta; no existe preset comparable y batch/duración permanecen `NO_DETERMINABLE_ESTATICO`. |

## Valores/comportamientos no configurables en Mammoth

| ítem | referencia de fila | cuestión que llegó pendiente |
|---|---|---|
| NC-01 | `L2P-A01`, `L2P-B22`, `L2P-B23` | Aceptar ViT-B/16, presencia de CLS y filtro prompt/head hardcodeados o autorizar cambio de código. |
| NC-02 | `L2P-A02` | Elegir ruta timm funcional o reparar/descartar la rama manual del checkpoint. |
| NC-03 | `L2P-A12` | Aceptar un update por batch o autorizar soporte de acumulación. |
| NC-04 | `L2P-A15`, `L2P-B24`, `L2P-B25` | Aceptar máscara/cabeza/alcance de logits Mammoth o autorizar parametrización. |
| NC-05 | `L2P-B01`–`L2P-B03` | Aceptar inserción única y posiciones Mammoth o autorizar variante arquitectónica. |
| NC-06 | `L2P-B13`, `L2P-B14` | Aceptar segundo ViT y coseno hardcodeados o autorizar arquitectura/matching alternativos; `embedding_key=cls` de B12 sí tiene override. |
| NC-07 | `L2P-B17`–`L2P-B19`, `L2P-B24`, `L2P-B46` | Aceptar persistencia, CE/forma del pull y reducción fija de loss o autorizar opciones. `L2P-B19` es `PARCIALMENTE_CONFIGURABLE`. |
| NC-08 | `L2P-B27`, `L2P-B32`, `L2P-B45` | Fijar la versión/defaults de Adam y aceptar reinicio por tarea o exponerlos. |
| NC-09 | `L2P-B33`–`L2P-B37`, `L2P-B42`, `L2P-B43` | Aceptar normalización/weight norm/temperatura/init/dropout/reweight/pre-logits ausentes o fijos, o parametrizar la cabeza. `L2P-B34` es ausencia estructural (`NO_APLICA`), no «no configurable». |
| NC-10 | `L2P-B40`, `L2P-B41` | Retirar/ignorar argumentos sin efecto o conectarlos al modelo. |
| NC-11 | `L2P-E11` | Aceptar warmup ausente o autorizar conexión de un scheduler con warmup. |

## Hallazgos de comportamiento que requieren cierre

| ítem | referencia | cuestión que llegó pendiente |
|---|---|---|
| HB-01 | `BHV-01`; `L2P-B01`, `L2P-B02` | Ratificar que el objeto de estudio es L2P original de input prompting, no L2P++ prefix multicapa. |
| HB-02 | `BHV-02`; `L2P-B03` | Elegir tratamiento posicional repo oficial o Mammoth. |
| HB-03 | `BHV-03`; `L2P-B12`–`L2P-B14` | Aceptar coste de dos ViT/forwards y matching fijo. |
| HB-04 | `BHV-04`; `L2P-B15` | Resolver lookup instancewise frente a batchwise. |
| HB-05 | `BHV-05`; `L2P-B17`–`L2P-B19`, `L2P-B24`, `L2P-B25`, `L2P-B27`, `L2P-B32`, `L2P-B45`, `L2P-B46` | Cerrar conjuntamente máscara de train, persistencia, reducción y estado/defaults de Adam. |
| HB-06 | `BHV-06`; `L2P-A15`, `L2P-A16`, `L2P-A21`, `L2P-B24`, `L2P-B25`, `L2P-E17` | Fijar alcance de logits, tareas cubiertas, métricas y cadencia comparables. `L2P-B24`/`L2P-B25` cruzados intencionadamente con HB-05 (origen BHV-05/BHV-06). |
| HB-07 | `BHV-07`; `L2P-B09`, `L2P-B11`, `L2P-B36` | Resolver escala/init de prompts, claves y cabeza. |
| HB-08 | `BHV-08`; `L2P-A01`, `L2P-A02`, `L2P-B22`, `L2P-B23` | Ratificar backbone/CLS/checkpoint y tratamiento de la rama manual defectuosa. |
| HB-09 | `BHV-09`; `L2P-B04`, `L2P-B10`, `L2P-B16`, `L2P-B19`, `L2P-B23`, `L2P-B40`, `L2P-B41` | Decidir si los flags engañosos/rotos se prohíben o se corrigen antes de ejecutar. |
| HB-10 | `BHV-10`; `L2P-A05`, `L2P-A06`, `L2P-A23` | Fijar seed/orden/cache y evitar las rutas custom defectuosas. |
| HB-11 | `BHV-11`; `L2P-B28`, `L2P-B29` | Elegir LR publicado o LR efectivo de código. |
| HB-12 | `BHV-12`; `L2P-E01`–`L2P-E22` | Fijar el único presupuesto autorizado y prohibir/solucionar `fitting_mode=time`. |
| HB-13 | `BHV-13`; `L2P-A12` | Ratificar ausencia de acumulación en el presupuesto compartido. |
| HB-14 | `BHV-14`; `L2P-A07`, `L2P-A08` | Fijar una tubería exacta de train/eval en revisión humana. |
| HB-15 | `BHV-15`; `L2P-B19`, `L2P-B20`, `L2P-B46` | Ratificar forma/reducción del pull y su acoplamiento top-k–λ. |
| HB-16 | `BHV-16`; `L2P-E12` | Prohibir early stopping sin validación o exigir una salvaguarda antes de usarlo. |
| HB-17 | `BHV-17`; `L2P-B15`, `L2P-B44` | Fijar si el voto batchwise, si se conserva, se define sobre batch global o shard local. |
| HB-18 | `BHV-18`; lista agrupada `INF-M08 disable_log` | Prohibir `disable_log=1` en este SHA o corregir la referencia a `logger`; no obtiene fila por la regla de infraestructura. |
| HB-19 | `BHV-19`; `L2P-A24` | Fijar política de directorios/carga; si se usa Mammoth, registrar argumentos heredados, `start_from` y la ausencia de restauración optimizer/scheduler. |
