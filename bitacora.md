# Bitácora del TFG

Registro append-only. Cada entrada contiene fecha, resumen, ruta de evidencia y estado conforme a `PLAN_MAESTRO.md`.

## 2026-08-03 · Dependencias del servidor y overlay privado

En `galatzo` faltaban `open-clip-torch` y `ftfy`; después quedaron disponibles como `2.32.0` y `6.3.1` desde `/home/amf380/.local/mammoth-pydeps`, con `pip check` correcto. `requirements.txt` omite `open-clip-torch` y deja `ftfy` sin pin, mientras `pyproject.toml` exige mínimos para ambos; la discrepancia se documenta sin modificar Mammoth.

Evidencia: `C:\Users\alex\Downloads\faseA_servidor_piloto0_oom_20260803_154958.tar.gz` → `logs/piloto0/environment_before_openclip.txt:26-29,55-58,310-345` y `environment_after_openclip.txt:5-13`; desarrollo en `infra/servidor.md`.

Estado: **RESUELTO**.

## 2026-08-03 · Incidencia OOM de L2P con `model_config=best`, batch 128

El Piloto-0 v1 sobre Mammoth@`cda7f236` alcanzó la tarea 1 y abortó durante el forward de L2P con `torch.cuda.OutOfMemoryError`; código 1 y sin fichero final de métricas. El tar registra la CLI, el estado GPU, el traceback y los recursos; no contiene una corrida a batch 64.

Evidencia: paquete anterior → `logs/piloto0/diagnostico_oom_20260803_154949.txt:117-123,138-162,207-263`, `driver.log:83-108` y `l2p_seq-cifar100-224_seed0_e1.time.txt:1-24`; síntesis en `infra/servidor.md`.

Estado: **RESUELTO**.

## 2026-08-03 · Bloqueo de CODA-Prompt a E=1

Mammoth acepta `--n_epochs 1` en el parser, pero `begin_task` crea `CosineSchedule(K=n_epochs)` y el constructor exige `K>1`; la corrida abortaría antes de la primera época. El dump aporta la derivación estática, no un traceback CODA; el Piloto-0 v2 usa E=2 solo para infraestructura y coste, sin cambiar el eje pendiente.

Evidencia: `auditoria/reconciliacion_fase_b.md:73-82`; Mammoth `models/coda_prompt.py:65-73`, `utils/schedulers.py:38-46`; procedimiento en `auditoria/piloto0.md`.

Estado: **DECIDIDO→D32**.

## 2026-08-03 · Hallazgo del volcado E=1/E=20

Tras excluir metadatos volátiles, L2P y DualPrompt solo cambian `n_epochs`; CODA-Prompt cambia además `custom_scheduler.K`, con `K=n_epochs`. El objeto se crea de nuevo en cada `begin_task`; para K válido, el scheduler custom sigue sin avanzar en el bucle auditado.

Evidencia: `auditoria/reconciliacion_fase_b.md:27-35,60-71`; JSON internos `auditoria_dumps/cifar100_n_epochs_{1,20}.json:927-951`; código `models/coda_prompt.py:65-73` y `utils/training.py:239-263`.

Estado: **DECIDIDO→D33**.

## 2026-08-03 · Transforms no idénticos entre métodos

En CIFAR-100, L2P/DualPrompt usan en Mammoth la config de datos `l2p`, mientras CODA-Prompt fuerza `coda_prompt`; difieren tanto la augmentación como el preprocesado de evaluación. La auditoría conserva las tres tuberías tal cual y no elige una común.

Evidencia: filas `A-C10`, `A-C12` y `A-C14` de `auditoria/entorno.md:27-31`; valores runtime e integridad en `auditoria/reconciliacion_fase_b.md:49-58`.

Estado: **PENDIENTE→reunión**.

## 2026-08-03 · Cierre definitivo de batch 128

Batch 128 queda descartado por D31 rev.: ya constaba que 128 produce OOM y 64 cabe; el Piloto-0 v1 reprodujo el OOM en Mammoth@`cda7f236`; y D31 exige un batch común y toma CODA-Prompt como método de mayor consumo. El tar actual solo prueba el segundo punto: el hecho de batch 64 y la premisa de uniformidad proceden del registro decisorio del Plan Maestro.

Evidencia: `PLAN_MAESTRO.md:161-166,220`; paquete → `diagnostico_oom_20260803_154949.txt:263`; filas `A-C15` y `A-C17` de `auditoria/entorno.md:32-34`; detalle en `infra/servidor.md`.

Estado: **DECIDIDO→D31**.

## 2026-08-03 · Consultas dirigidas Q1–Q3

Q1 queda cubierta: Mammoth no ofrece acumulación real común por CLI/config; L2P y DualPrompt actualizan cada minibatch y la opción de CODA para N>1 borra gradientes intermedios. Q2 ya estaba capturada en dos filas DualPrompt: 0.03 es nominal/parseado, 0.015 es el LR del optimizador con batch 128, y ambas filas son `CASO 3 CONFLICTO→PAPER` con propuesta 0.005. Q3 confirma que `best` son overlays de Mammoth y que los tres YAML carecen de bloque `default`; no se equiparan globalmente a las recetas originales.

Evidencia: `auditoria/entorno.md:34`; `auditoria/valores_dualprompt.md:91-92`; `auditoria/reconciliacion_fase_b.md:41-45`; Mammoth `models/utils/__init__.py:47-82`, `main.py:169-186`; agenda en `auditoria/cola_revision.md`.

Estado: **PENDIENTE→reunión**.

## Parte de novedades para el chat maestro — 2026-08-03

- Paquete post-piloto verificado: SHA-256 `6e9fc8d…e3e0d25`; inventario, overlay y OOM quedan en `infra/servidor.md` y `auditoria/fuentes.md`.
- Piloto-0 v1: L2P/batch128 OOM; DualPrompt no llegó a ejecutarse y CODA no estaba en el lanzador. Es evidencia de infraestructura, no un piloto completado. El v2 queda documentado con tres lanzadores independientes, batch64, E=1 para L2P/DualPrompt y E=2 para CODA en `auditoria/piloto0.md`.
- Q1: no hay acumulación real común configurable; la consecuencia del árbol D31 permanece para decisión humana. Q2: DualPrompt ya registra el conflicto; `0.03` es nominal y el optimizador recibe `0.015`, con propuesta paper `0.005`. Q3: `best/default` son mecanismos de configuración Mammoth; no demuestran equivalencia global con las recetas originales.
- La cola dirigida contiene cuatro asuntos con filas: batch/OOM, peldaño mínimo, preprocesado común y alineación `model_config`/cascada (`auditoria/cola_revision.md`). No se modificó ningún `valor final`, el eje ni el código.

## 2026-08-04 · Piloto-0 v2 completado

En `galatzo`, Mammoth@`cda7f236`, L2P/b64/E1, DualPrompt/b64/E1 y CODA-Prompt/b64/E2 completaron las diez tareas con código 0 y añadieron una línea Class-IL por método; el paquete contiene además una línea Task-IL por método. El driver global terminó con 0; esto satisface el criterio operativo de D32 exclusivamente para las configuraciones diagnósticas `model_config=best` ejecutadas.

Evidencia: `C:\Users\alex\Downloads\piloto0_v2_20260804_004240.tar.gz`, SHA-256 `c25a5d8431dd5b863ddfd62271ed829f4922fbfae7c7283f174442e6fb3b83f3` → `logs/piloto0_v2/driver_all.log:2-4,7,254,256,503,505,745,747-748`, `resumen_final.txt:1-8`, los tres `*.run.log` y los seis `data/results_piloto0_v2/**/logs.pyd`; cierre detallado en `auditoria/piloto0.md` e infraestructura en `infra/servidor.md`.

Estado: **RESUELTO**. D33, batch como `valor final` y la alineación de `best` permanecen pendientes; no se decidió ninguno.

## 2026-08-04 · Incidencias de medida del Piloto-0 v2

La GPU conservaba dos kernels ajenos de 304 y 332 MiB y, al fallar `pynvml`, los máximos que imprime Mammoth son picos `torch.cuda.max_memory_allocated` del proceso, no VRAM total. Además, `metricas_resumen.tsv` truncó los minutos de `elapsed`; las duraciones autoritativas son 30:35,89, 28:20,38 y 54:26,92 en los `*.time.txt`. Una tentativa L2P previa acabó por `SIGINT` y quedó aislada fuera del driver válido.

Evidencia: paquete anterior → `logs/piloto0_v2/gpu_preflight.txt:12-14,22-29`, los tres `*.gpu_before.txt:1-2`, `metricas_resumen.tsv:1-4`, cada `*.time.txt:5,10,15,23` e `interrumpidos/l2p_20260803_223543/*.run.log:190-211`; interpretación y límites en `auditoria/piloto0.md` e `infra/servidor.md`.

Estado: **RESUELTO** mediante selección explícita de las fuentes autoritativas y conservación de las limitaciones; no se usa esta incidencia para abrir una decisión metodológica nueva.

## Parte de novedades para el chat maestro — 2026-08-04

- Integridad: sidecar y cálculo local coinciden en SHA-256 `c25a5d84…b83f3`; 52 entradas/37 ficheros seguros, revisados fuera de TFG. Inventario en `auditoria/fuentes.md`.
- Piloto-0 v2 **completado**: L2P b64/E1 (Class-IL 81,67%; Task-IL 96,83%; 30:35,89), DualPrompt b64/E1 (83,26%; 96,96%; 28:20,38) y CODA b64/E2 (83,42%; 97,63%; 54:26,92). Los tres recorrieron diez tareas, salieron con 0 y escribieron Class-IL/Task-IL; detalle y localizadores en `auditoria/piloto0.md`.
- Memoria reportada por Mammoth: 6.099,35 / 6.398,29 / 8.705,77 MiB para L2P/Dual/CODA, pero es pico *allocated* del proceso por fallback de `pynvml`, no VRAM total. La GPU no estaba limpia y el TSV perdió los minutos; usar los `*.time.txt`. Límites en `infra/servidor.md`.
- Evidencia para la reunión: batch real 64 es operativamente viable en las tres recetas diagnósticas y CODA E=2 completa el bucle. Siguen abiertos PD-01, PD-02/D33 y PD-04; el éxito no demuestra CODA E=1 ni convierte `best` o batch64 en configuración final (`auditoria/cola_revision.md`).
- No se modificó código, el eje, D33, las tablas de valores ni ninguna columna `valor final`. La tentativa L2P interrumpida se registró como incidencia aislada y no se mezcla con los resultados válidos.

## 2026-08-04 · Reapertura declarada D33/D17 conforme a D34(b)

El dispatcher comprueba `model.scheduler` en `utils/training.py:241`, mientras CODA solo define `custom_scheduler` en `models/coda_prompt.py:73`; no existe ninguna llamada `.step()` sobre el CosineSchedule de CODA. El LR efectivo es constante y contradice el supuesto anterior de D17.

Evidencia: `mammothV2/utils/training.py:241`, `mammothV2/models/coda_prompt.py:73` y búsqueda estática de `custom_scheduler.step` en `mammothV2/` sin resultados.

Estado: **DECIDIDO→D39**.

## 2026-08-04 · Verificación estática Codex (retroactiva): scheduler CODA, alias de modelo y mínimos de dependencias

LR constante confirmado. `scripts/reproduce.json:49,52` usa `coda_prompt` y el parser acepta ambas grafías mediante la normalización guion bajo→guion; `torch>=2.1.0` y `torchvision>=0.21.0` aparecen solo en `pyproject.toml:41-42`. No existe lockfile.

Evidencia: `mammothV2/scripts/reproduce.json:49,52`; `mammothV2/utils/__init__.py:121-122`; `mammothV2/utils/args.py:241-242`; `mammothV2/pyproject.toml:41-42`; inventario recursivo de ficheros de lock.

Estado: **RESUELTO**.

## 2026-08-04 · Verificación Codex (retroactiva): commit documental, inventario y anclas de papers

El commit `a0adbdfaca0b7ae06c452a44196faa0ff3fab576` quedó verificado. El inventario contiene 16 ficheros en `auditoria/` más `infra/servidor.md`; `instrucciones_proyecto.md` no existe. Las anclas transcritas de L2P, DualPrompt y CODA coinciden con las tablas, y los SHA-256 de los tres tarballs fuente coinciden con `fuentes.md`.

Evidencia: `git show -s a0adbdf`; inventario de `auditoria/`; `auditoria/fuentes.md:166-170`; tarballs `papers/{l2p,dualprompt,coda-prompt}/*_source.tar.gz`.

Estado: **RESUELTO**.

## 2026-08-04 · Fe de erratas de citas al plan

Las referencias `PLAN_MAESTRO.md:161-166,220` de la entrada del 3-ago, `:163` y `:221` corresponden a v2.2/v2.3. En v2.5 corresponden a §10 (hecho duro), §13 D31/D32/D25/D26 y §8-D.

Convención en vigor: el plan se cita por §/Dxx, nunca por línea.

Estado: **RESUELTO**.

## 2026-08-04 · Corrección del parte 04: PD-03 (preprocesado común) sigue abierto

PD-03 permanece abierto junto a PD-01, PD-02 y PD-04; los cuatro asuntos siguen destinados a la reunión de Fase B.

Evidencia: `auditoria/cola_revision.md`, bloque de pendientes dirigidos PD-01…PD-04.

Estado: **PENDIENTE→reunión**.

## 2026-08-04 · Discrepancia declarativa torchvision

`pyproject.toml:42` exige `torchvision>=0.21.0`; `galatzo` usó `0.17.1+cu121`; no existe lockfile. La discrepancia ya está registrada en `infra/servidor.md` y no requiere acción de código.

Evidencia: `mammothV2/pyproject.toml:42`; `infra/servidor.md`; registro de entorno de `galatzo`.

Estado: **RESUELTO**.

## 2026-08-04 · Estado archived de los repos oficiales

`GT-RIPL/CODA-Prompt` tiene `archived=False` y `aimagelab/mammoth` tiene `archived=False` (API GitHub, 4-ago, verificación del maestro). `google-research/l2p` sí está archivado, verificado por el usuario en github.com el 4-ago.

Evidencia: verificación del maestro y del usuario en GitHub, 2026-08-04; consolidación en `auditoria/fuentes.md`.

Estado: **RESUELTO**.

## 2026-08-04 · Dosieres: adjudicación y congelación

Se eligieron como canónicas las copias auditadas por el maestro, por mayor contenido. El protocolo amplía o matiza Resumen y §§1–7; metodología cambia §§4.2, 6 y 6.1. Se añadió la nota de cabecera de §5. Las cuatro cadenas origen de errata no existen literalmente en la versión canónica (`G-prompt (longitud 5)…`; `timm==0.9.8 … (indicado en su README)`; `timm==0.6.7…`; ausencia de `checkpoint 21k`), por lo que quedaron **NO-APLICADO-PREMISA-ERRÓNEA** sin sustituir variantes.

SHA-256 finales: `referencias/dossier_protocolo_cil.md` → `3af69dbd3bcc764f4f93cea094ef83213a9ac43b392bd55fd7ba3e6c93b8f345` (21.826 bytes); `referencias/dossier_metodologia_conflictos.md` → `9b1edf2ebfedcc30af181e74a3988e696f0424ce10d74e76485a07b83c5aae47` (28.711 bytes).

Estado: **RESUELTO**.

## Parte de novedades para el chat maestro — sanación documental 2026-08-04

- `PLAN_MAESTRO.md` pasa a v2.5: D33/D17 se reabren conforme a D34(b), D39 resuelve el scheduler y la escalera {1,5,20} queda firme condicionada a su validación técnica.
- Se sanaron el inventario de fuentes, las anclas estables al plan, la tabla de entorno y las tablas/colas por método; PD-01…PD-04 permanecen abiertos para la reunión, con PD-02 informado por D39.
- El árbol de `auditoria/`, `infra/`, `referencias/`, `papers/` y `prompts_fase_A.md` refleja el inventario real; L2P consta archivado y CODA-Prompt/Mammoth no archivados a 4-ago.
- Los dosieres canónicos son las copias auditadas de mayor contenido. La nota de §5 se aplicó; cuatro sustituciones se rechazaron por premisa literal errónea. Hashes finales registrados en la entrada anterior.
- Próximas acciones: ejecutar y validar D39, realizar el muestreo D10 y celebrar la reunión de Fase B con PD-01…PD-04.

## 2026-08-04 · Limpieza de restos históricos D33/peldaño en el plan

Se aplicaron las seis sustituciones elevadas en el informe de aplicación: §9, D20, D33, D38 y §14(b) ya remiten a la validación de D39 y a la escalera resuelta. D30 incorpora además el reparto permanente de capacidades de maestro, Codex y usuario.

Evidencia: `PLAN_MAESTRO.md` §9, §13 (D20, D30, D33 y D38) y §14(b).

Estado: **RESUELTO**.

## 2026-08-04 · Erratas de dossier_protocolo corregidas en commit de compleción

El primer intento no aplicó por desajuste de literales; con los literales exactos se corrigieron las cuatro erratas. SHA-256 definitivo de `referencias/dossier_protocolo_cil.md`: `92d3db585ac9c438482e796c9a1523d3ba77333a3fc06c218af4d120bd97cd97`. La congelación D35 es efectiva desde este commit. `dossier_metodologia_conflictos.md` queda sin cambios (`9b1edf2ebfedcc30af181e74a3988e696f0424ce10d74e76485a07b83c5aae47`).

Estado: **RESUELTO**.

## 2026-08-04 · D39 implementado

D39 quedó implementado en `mammothV2` commit `5f102e5fd34d60636b19c11fab99ab44781a7d64`: parche confinado a CODA-Prompt, traza de LR y validador E=5/E=1. La validación está **PENDIENTE de ejecución en servidor por el usuario (Modo E)**; Codex no ejecutó `main.py` ni entrenamiento.

Evidencia: `mammothV2/models/coda_prompt.py:73-93`, `mammothV2/scripts_tfg/valida_d39.sh:155-213`, `mammothV2/scripts_tfg/valida_d39_trace.py:59-96,142-205` y `auditoria/fuentes.md` §3.1/§7.

Estado: **PENDIENTE→ejecución del usuario en servidor**.

## Parte de novedades para el chat maestro — compleción de sanación + implementación D39 — 2026-08-04

- `PLAN_MAESTRO.md` queda limpio de los seis restos históricos pedidos: peldaño {1,5,20} resuelto, condicionado solo a validar D39; D30 fija que el runtime corresponde exclusivamente al usuario.
- El dosier de protocolo incorpora las cuatro correcciones exactas y queda congelado con SHA-256 `92d3db58…bd97cd97`; el dosier metodológico no cambia (`9b1edf2e…`).
- `mammothV2/tfg-auditoria@5f102e5` conecta el CosineSchedule por tarea solo en CODA: E=1 conserva LR base; E>1 hace step al inicio salvo época 0 y reproduce los puntos oficiales 0…K−2.
- `scripts_tfg/valida_d39.sh` lanza E=5 y E=1 (CODA-Prompt, `best`, batch 64); el verificador imprime `D39_TRACE`/`D39_E1` antes de interrumpir cada proceso. No se ejecutó runtime local.
- Pendiente único de este bloque: el usuario ejecuta el validador en galatzo (Modo E). Fuentes y localizadores: `auditoria/fuentes.md` §3.1 y §7.

## 2026-08-04 · Incidencias de arranque de la validación D39 en galatzo

El primer lanzamiento no encontró el ejecutable `python`. Con `/opt/environment/bin/python`, el siguiente no encontró `timm.layers`; el usuario restauró el entorno del Piloto-0 mediante `PYTHONPATH=$HOME/.local/mammoth-pydeps…` y verificó `timm 0.9.8`. El tercer lanzamiento llegó a construir CODA-Prompt, pero terminó antes de `begin_epoch`: `--disable_log 1` impidió crear `logger` en `utils/training.py:153-154` y el mismo fichero lo usó en `:159`, produciendo `UnboundLocalError`.

Ninguno de esos intentos emitió `D39_LR`; sus `FAIL observado=[NA…]` son fallos de arranque y no validan ni refutan el scheduler ni activan el fallback D15. Se retiró únicamente `--disable_log 1` del lanzador en `mammothV2@4353bc18897e71362a481ff24e59f025069c1817`; el parche de modelo D39 no cambió.

Evidencia: logs de servidor `d39_validation_20260804T190248Z` y `d39_validation_20260804T190357Z`; `mammothV2/utils/training.py:153-159`; `mammothV2/scripts_tfg/valida_d39.sh:155-212`; `auditoria/fuentes.md` §3.1/§7.

Estado: **PENDIENTE→nuevo lanzamiento por el usuario en servidor**.

## Parte de novedades para el chat maestro — corrección de arranque del validador D39 — 2026-08-04

- Los dos `FAIL` con trazas `NA` fueron anteriores a `begin_epoch`: dependencias incompletas primero y `logger` no inicializado después; no son un fallo técnico de D39.
- El entorno correcto queda confirmado con `/opt/environment/bin/python`, `PYTHONPATH=$HOME/.local/mammoth-pydeps…` y `timm 0.9.8`.
- `mammothV2/tfg-auditoria@4353bc1` elimina solo `--disable_log 1`; quedan pendientes el nuevo lanzamiento E=5/E=1 y sus veredictos `D39_TRACE`/`D39_E1`.

## 2026-08-04 · Validación técnica D39 completada en galatzo

El usuario ejecutó `mammothV2@4353bc18897e71362a481ff24e59f025069c1817` con CODA-Prompt, `seq-cifar100-224`, `model_config=best` y batch 64. Para E=5, las cinco observaciones coincidieron exactamente con la secuencia esperada: `[0.001, 0.001, 0.00092537519929086212, 0.00071263851892520535, 0.00039354082365465138]`; resultado `D39_TRACE: PASS`. Para E=1, arrancó sin aserción y conservó LR base `0.001`; resultado `D39_E1: PASS`. Veredicto global: `D39_OVERALL_VERDICT verdict=PASS`.

El lanzador interrumpió cada proceso únicamente después del veredicto y ambos devolvieron código 0. La condición técnica de D15/D39 queda satisfecha: se mantiene la escalera firme `{1,5,20}` y no se activa el fallback `{2,5,20}`.

Evidencia: servidor `scripts_tfg/d39_validation_20260804T203937Z/` y `~/d39_validacion.log`; consolidación en `auditoria/fuentes.md` §7.

Estado: **RESUELTO**.

## Parte de novedades para el chat maestro — validación D39 superada — 2026-08-04

- D39 queda técnicamente validado en galatzo sobre `mammothV2@4353bc1`: E=5 reproduce exactamente los cinco LR esperados y E=1 arranca con LR base constante; ambos casos y el veredicto global son `PASS`.
- Los fallos anteriores quedaron confirmados como incidencias de arranque sin trazas y no afectan al resultado; el parche de modelo permaneció inalterado.
- Se satisface la condición de D15/D39: escalera `{1,5,20}` firme; fallback `{2,5,20}` no activado. Queda pendiente que el maestro emita, si procede, el literal autorizado para limpiar del plan las menciones históricas a «pendiente de validación».

## 2026-08-04 · Limpieza de las menciones condicionales de D39 en el plan tras la validación

Se aplicaron los literales autorizados por el maestro en §9, D15, D20, D38, §14(b) y §15. El plan registra D39 validado, la escalera `{1,5,20}` firme y el fallback `{2,5,20}` no activado; la acción viva de ejecutar/validar D39 fue retirada y la lista quedó renumerada.

Evidencia: `PLAN_MAESTRO.md` §9, §13 (D15, D20 y D38), §14(b) y §15.

Estado: **RESUELTO**.

## Parte de novedades para el chat maestro — limpieza final de D39 — 2026-08-04

- Las siete ediciones autorizadas quedaron aplicadas en `PLAN_MAESTRO.md`: desaparecen las condiciones pendientes de D39 y §15 incorpora el cierre validado.
- La escalera `{1,5,20}` queda firme; el fallback `{2,5,20}` consta como no activado. La acción viva de validar D39 fue eliminada y la lista se renumeró.
