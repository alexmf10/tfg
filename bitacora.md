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
