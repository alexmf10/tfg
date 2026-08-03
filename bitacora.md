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
