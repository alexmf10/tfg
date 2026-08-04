# Registro de fuentes y montaje del workspace

Fecha de última actualización: 2026-08-04. Zona horaria del workspace: Europe/Madrid.

## 1. Verificación de ruta e inventario inicial

Ruta comprobada con `$PWD.Path`:

```text
C:\Users\alex\Documents\TFG
```

El directorio era el solicitado (`.../Documentos/TFG`) y todavía no era un repositorio Git. Inventario tomado antes de inicializar, clonar, borrar o crear entregables:

| entrada inicial | tipo / contenido | estado Git inicial |
|---|---|---|
| `mammothV2/` | directorio con contenido | Repo Git. `origin=https://github.com/alexmf10/mammothV2.git`; `upstream=https://github.com/aimagelab/mammoth.git`; HEAD `e75a491c69fd729edeb01431afb753d9157d9a81`; rama `master`; working tree limpio. |
| `l2p/` | directorio vacío, 0 entradas incluidas ocultas | No era repo Git. |
| `dualprompt/` | directorio vacío, 0 entradas incluidas ocultas | No era repo Git. |
| `codaprompt/` | directorio vacío, 0 entradas incluidas ocultas | No era repo Git. |
| `mammoth/` | directorio vacío, 0 entradas incluidas ocultas | No era repo Git; se dejó intacto y está fuera de las fuentes de la auditoría. |
| `PLAN_MAESTRO.md` | fichero preexistente, 40.607 bytes | Sin seguimiento porque TFG aún no era repo. No se modificó. |
| `prompts_fase_A.md` | fichero preexistente, 29.993 bytes | Sin seguimiento porque TFG aún no era repo. No se modificó. |

Comandos de inventario Git empleados por repo: `git remote -v`, `git rev-parse HEAD`, `git branch --show-current` y `git status --porcelain=v1 --untracked-files=all`.

## 2. Acciones de montaje

1. Se inicializó el repo raíz con `git init`.
2. Se comprobó que `https://github.com/alexmf10/tfg.git` existe mediante `git ls-remote ... HEAD` (HEAD remoto `6d6259ef763e693df7ec18fbd39642a3b8920464`) y se añadió como `origin`. No se hizo `fetch`, merge ni push: el usuario decide el push y el repo local comenzó con historia nueva, todavía no enlazada a la historia remota.
3. Se creó `.gitignore`. Los patrones se anclaron a la raíz para no ocultar, por ejemplo, `papers/l2p/`: `/mammothV2/`, `/l2p/`, `/dualprompt/`, `/codaprompt/`, `/repos_aux/`, `/results*/`, `/data/`, `/datasets/`, `*.pth`, `__pycache__/`, `.DS_Store` y `Thumbs.db`.
4. Se verificó `mammothV2`, se actualizó `upstream`, se creó `tfg-auditoria` desde el HEAD verificado y se añadió únicamente `scripts_tfg/dump_config.py`.
5. Desde las carpetas inicialmente vacías se ejecutaron con éxito:

   ```text
   (cwd: TFG/l2p)        git clone https://github.com/google-research/l2p .
   (cwd: TFG/codaprompt) git clone https://github.com/GT-RIPL/CODA-Prompt .
   ```

6. Se eliminó `dualprompt/` solo después de volver a comprobar que seguía vacío. DualPrompt se audita desde `l2p/`, su fuente oficial.
7. No se clonaron reimplementaciones auxiliares en `repos_aux/`; por tanto no hay fuentes auxiliares que puedan confundirse con la columna «repo oficial».
8. Se crearon `auditoria/`, `papers/l2p/`, `papers/dualprompt/` y `papers/coda-prompt/`.
9. Se descargaron los tres PDF y los tres e-print; los originales comprimidos se conservaron y se extrajeron en `source_v2/` después de validar que el tar no contenía rutas absolutas ni componentes `..`.

Todas las operaciones de red requeridas (`fetch`, clones, descargas arXiv y push de `mammothV2`) terminaron correctamente. No hay comando de red fallido que el usuario deba repetir manualmente.

## 3. MammothV2: procedencia, rama y entorno

### 3.1 Procedencia

- `origin` configurado: `https://github.com/alexmf10/mammothV2.git` (el sufijo `.git` es la forma Git equivalente a la URL indicada).
- `upstream` configurado: `https://github.com/aimagelab/mammoth.git`.
- Comando de actualización ejecutado con éxito: `git fetch upstream --tags --prune`.
- HEAD inicial: `e75a491c69fd729edeb01431afb753d9157d9a81`, fecha de commit `2026-05-20T22:13:32+02:00`.
- `git branch -r --contains e75a491...` devolvió `upstream/master` y `upstream/HEAD`; además `upstream/master` apuntaba exactamente a ese SHA.
- Resultado: **BASE OFICIAL VERIFICADA: `e75a491c69fd729edeb01431afb753d9157d9a81`**.
- Rama `tfg-auditoria` creada desde ese SHA; SHA del punto de creación: `e75a491c69fd729edeb01431afb753d9157d9a81`.
- Commits añadidos: `cda7f23681f7bffacee460d99e990bc803bccf04` (`Add TFG effective config dump`), `5f102e5fd34d60636b19c11fab99ab44781a7d64` (`Implement CODA scheduler D39`) y `4353bc18897e71362a481ff24e59f025069c1817` (`Fix D39 validator logging startup`).
- HEAD final de `tfg-auditoria`: `4353bc18897e71362a481ff24e59f025069c1817`.
- Contenido propio de la rama: `scripts_tfg/dump_config.py`; parche D39 confinado a `models/coda_prompt.py:73-93` (K≥1, scheduler por tarea, hook de época y traza); lanzador `scripts_tfg/valida_d39.sh:155-212` (E=5 y E=1) y verificador `scripts_tfg/valida_d39_trace.py:59-96,142-205` (secuencia esperada y veredictos). No se modificó `utils/schedulers.py` ni código de L2P/DualPrompt. El commit `4353bc1` retira del lanzador `--disable_log 1`, incompatible con `utils/training.py:153-159` porque deja `logger` sin inicializar.
- Push: `git push origin tfg-auditoria` terminó correctamente; `origin/tfg-auditoria` apunta al mismo SHA.
- Estado final: working tree limpio.

### 3.2 Versiones declaradas

| componente | restricción localizable | evidencia en `mammothV2` |
|---|---|---|
| Python | `>=3.10` | SHA base `e75a491...`, `pyproject.toml:7`. |
| torch | `>=2.1.0` | SHA base `e75a491...`, `requirements.txt:1`; también `pyproject.toml:41`. |
| torchvision | sin pin en requirements; `>=0.21.0` en metadatos actuales | SHA base `e75a491...`, `requirements.txt:3` y `pyproject.toml:42`. |
| timm | `==0.9.8` | SHA base `e75a491...`, `requirements.txt:6` y `pyproject.toml:40`. |
| open-clip-torch | `>=2.32.0`; ausente de `requirements.txt` | SHA base `e75a491...`, `pyproject.toml:33`. |

No se localizó lockfile. Por tanto torch y torchvision no quedan fijados a una versión exacta por estos ficheros.

### 3.3 Entorno instalado accesible en esta máquina

Intérprete: `C:\Users\alex\miniconda3\python.exe`.

| componente | versión observada |
|---|---|
| Python | `3.11.5` (Anaconda) |
| torch | `2.13.0+cpu` |
| torchvision | `0.28.0+cpu` |
| timm | `0.9.8` |
| numpy | `1.26.3` |
| kornia | `0.8.3` |
| Pillow | `10.2.0` |
| PyYAML | `6.0.1` |
| open-clip-torch | `3.3.0` |
| requests | `2.31.0` |
| GitPython | no instalado |

Estas son versiones del entorno local accesible, no una afirmación sobre el entorno del servidor.

### 3.4 Entorno del servidor `galatzo` usado en la Fase B

El archivo externo `auditoria_dumps_e1_e20.tar.gz`, recibido el 2026-08-03 y conservado fuera del repo, tiene SHA-256 `7254a9033d174398b8e9ef2a35ba1951e3ece32fa0e0a48a51bb49d358fa19c8`. Sus registros sitúan la ejecución en `galatzo`, sobre la rama `tfg-auditoria` y el commit `cda7f23681f7bffacee460d99e990bc803bccf04`.

Antes de instalar las dos dependencias que faltaban en el overlay privado, `logs/piloto0/environment_before_openclip.txt` (SHA-256 `fe58357f761eda22393ee9cb3481617aa641a2053725203d7b1e7b4b77f2b6c0`) registró:

| componente | versión efectiva observada en `galatzo` |
|---|---|
| sistema | Ubuntu 22.04.5 LTS; kernel 6.8.0-124-generic |
| Python | `3.10.12`; ejecutable `/opt/environment/bin/python` |
| torch | `2.2.1+cu121` |
| torchvision | `0.17.1+cu121` |
| timm | `0.9.8` (overlay privado) |
| kornia | `0.7.1` (overlay privado) |
| numpy | `1.26.4` |
| Pillow | `10.2.0` |
| PyYAML | `6.0.1` |
| CUDA / cuDNN / GPU | CUDA de Torch 12.1; cuDNN 8902; NVIDIA GeForce RTX 3060 |
| `open-clip-torch` / `ftfy` | `NO_DISPONIBLE` antes de la instalación |

El registro posterior `logs/piloto0/environment_after_openclip.txt` (SHA-256 `85005cded823431421a3b844d19819f66c9922e5fdc8cb782b2bdf6ceb7524ac`) confirma `open-clip-torch==2.32.0` y `ftfy==6.3.1` en `/home/amf380/.local/mammoth-pydeps`, manteniendo torch 2.2.1, torchvision 0.17.1, timm 0.9.8, kornia 0.7.1 y numpy 1.26.4. `pip check` terminó sin dependencias rotas antes y después. Los volcados E=1/E=20 se generaron después de este registro posterior.

Estas versiones son evidencia del servidor para esta ejecución, no un nuevo pin del proyecto. En particular, permiten resolver la representación efectiva de los transforms que el YAML deja a defaults de torchvision 0.17.1; no autorizan generalizarla a otro entorno con dependencias distintas.

Discrepancia declarativa torchvision: `pyproject.toml:42` exige `>=0.21.0`; `galatzo` usó `0.17.1+cu121`; sin lockfile; ya registrada en `infra/servidor.md`; sin acción de código.

### 3.5 Efecto lateral observado y retirado

La carga de los registries durante la prueba del parser importa todos los datasets (`datasets/__init__.py:173-176`). En este SHA, `datasets/seq_8vision.py:47-48` crea un OpenCLIP ViT-B/16 en ámbito de clase; por ello un parseo sin entrenamiento puede descargar pesos. Durante la validación se generaron bajo rutas ignoradas un cache `checkpoints/ViT-B-16/` de 598.517.020 bytes y 29 directorios `__pycache__/`. Como eran artefactos creados por esta comprobación y no entregables, se retiraron después de verificar rutas, contenido y fechas. La comprobación final dio 0 `__pycache__`, ausencia de `checkpoints/ViT-B-16/` y working tree limpio.

## 4. Repos oficiales de los métodos

### 4.1 L2P y DualPrompt

- Repo oficial único: `https://github.com/google-research/l2p`.
- Origin local: `https://github.com/google-research/l2p`.
- Rama: `main`.
- SHA: `dd8836e6e372df29f03d83bf3dc3a806114e9d8e`.
- Último commit: `2022-12-08T00:06:01-08:00`, asunto `Update README.md`.
- Estado: limpio y sin divergencia respecto de `origin/main` al comprobarlo.
- `git tag --list` y `git ls-remote --tags https://github.com/google-research/l2p` no devolvieron tags.
- La API `https://api.github.com/repos/google-research/l2p/releases?per_page=100` devolvió 0 releases.
- El repo identifica ambos métodos en `README.md:5-6`; las instrucciones/configs de ambos aparecen en `README.md:55-74`.
- DualPrompt fue incorporado explícitamente en `3e102b00b834c92fd0d3f71001a0fa2378302d94` (`2022-07-20T04:21:49-07:00`, `codebase update, including more benchmarks and DualPrompt`).
- Versión de referencia elegida para L2P y DualPrompt: **`main` @ `dd8836e6e372df29f03d83bf3dc3a806114e9d8e`**, por la regla «sin tag/release → rama principal».
- Esta elección no demuestra equivalencia exacta entre ese commit y cada versión del paper; solo aplica el fallback prescrito.
- Estado «¿archivado?»: **SÍ, ESTÁ ARCHIVADO Y VERIFICADO POR MÍ (USUARIO) EN github.com, 4-ago**.

### 4.2 CODA-Prompt

- Repo oficial: `https://github.com/GT-RIPL/CODA-Prompt`.
- Origin local: `https://github.com/GT-RIPL/CODA-Prompt`.
- Rama: `main`.
- SHA: `6417d4f68754be68b697c7ca2323ee61e791e1a3`.
- Último commit: `2023-09-25T17:43:08Z`, asunto `domainnet, FT, and forgetting`.
- Estado: limpio y sin divergencia respecto de `origin/main` al comprobarlo.
- `git tag --list` y `git ls-remote --tags https://github.com/GT-RIPL/CODA-Prompt` no devolvieron tags.
- La API `https://api.github.com/repos/GT-RIPL/CODA-Prompt/releases?per_page=100` devolvió 0 releases.
- Versión de referencia elegida: **`main` @ `6417d4f68754be68b697c7ca2323ee61e791e1a3`**, por la regla «sin tag/release → rama principal».
- La elección no demuestra equivalencia exacta con el paper; aplica el fallback prescrito.
- Ambigüedad registrada: `README.md:36` indica `experiments/cifar100.sh`, pero el fichero existente es `experiments/cifar-100.sh`. No se corrigió ni se resolvió en silencio.
- Estado «¿archivado?»: **NO ARCHIVADO (4-ago)**.

### 4.3 Reimplementaciones auxiliares

No se clonó `JH-LEE-KR/l2p-pytorch` ni `JH-LEE-KR/dualprompt-pytorch`. Si se incorporan después, serán evidencia auxiliar y **NUNCA** rellenarán la columna «repo oficial».

## 5. Papers y fuentes LaTeX

La versión se determinó con el `<entry><id>` y el enlace PDF versionado guardados en cada `arxiv_api_*.xml`. Los endpoints sin sufijo de versión devolvieron la última versión, `v2`, en los tres casos.

| método | arXiv y versión | fechas de la entrada | PDF local | fuente original y extracción | validación / SHA-256 |
|---|---|---|---|---|---|
| L2P | `2112.08654v2` | publicado `2021-12-16T06:17:07Z`; actualizado `2022-03-21T19:26:32Z` | `papers/l2p/arxiv_2112.08654.pdf` | `papers/l2p/arxiv_2112.08654_source.tar.gz`; `papers/l2p/source_v2/` (principal: `L2P_arxiv.tex`) | 13 páginas. PDF `c32a3012be38e3ab1d6dcc2b38ed350c2dcf40d746899ff3e93b1b681fc08d87`; fuente `3a1744097a755cf840874e2f2527ef8c0cc06606893d52e0dd9b9ed5b3702f5c`. |
| DualPrompt | `2204.04799v2` | publicado `2022-04-10T23:36:55Z`; actualizado `2022-08-05T11:26:06Z` | `papers/dualprompt/arxiv_2204.04799.pdf` | `papers/dualprompt/arxiv_2204.04799_source.tar.gz`; `papers/dualprompt/source_v2/` (principal: `dual_prompt_camera_ready.tex`) | 24 páginas. PDF `dae3d69fc8a191754bbac2c02c633faf2ce7f42ba6ab706845d6947d488b8fe9`; fuente `b15dea20e51708b0255dd9eaeea589c4567870f57143e118f7c1805c4d305b90`. |
| CODA-Prompt | `2211.13218v2` | publicado `2022-11-23T18:57:11Z`; actualizado `2023-03-30T17:58:59Z` | `papers/coda-prompt/arxiv_2211.13218.pdf` | `papers/coda-prompt/arxiv_2211.13218_source.tar.gz`; `papers/coda-prompt/source_v2/` (principal: `main.tex`) | 15 páginas. PDF `b9c5a8640bd8f39b1b90df0a99433ae89965e0c874e12e5d654a432b360f8c95`; fuente `a19b5102b6c2e6f16b7d32e3100503fb108c23d5b4b3726071186c2a30700ca3`. |

Metadatos arXiv locales: `papers/l2p/arxiv_api_2112.08654.xml`, `papers/dualprompt/arxiv_api_2204.04799.xml` y `papers/coda-prompt/arxiv_api_2211.13218.xml`.

Validación PDF: `pdfinfo` abrió los tres documentos; Poppler generó 13, 24 y 15 PNG respectivamente y se inspeccionaron visualmente las portadas. Poppler emitió avisos no fatales sobre las fuentes de display `Symbol`/`ArialUnicode`; el proceso devolvió código 0 y todas las páginas produjeron PNG no vacíos. Los 52 PNG temporales se retiraron tras la comprobación.

Los papers y fuentes quedan disponibles localmente y visibles como no seguidos. No se añadieron al commit del repo TFG porque la lista explícita de entregables a commitear contiene `.gitignore` y los tres ficheros de `auditoria/`, no los binarios ni árboles fuente de `papers/`.

## 6. Volcado de configuración de Mammoth

Fichero: `mammothV2/scripts_tfg/dump_config.py`, commit y rama indicados en §3.1.

El script usa `main.parse_args`, `datasets.get_dataset`, `main.extend_args` y `main.check_args`; no construye backbone/modelo, no abre loaders y no entrena. Emite:

- Namespace completo inmediatamente después del parser;
- Namespace completo después de la extensión dependiente del dataset;
- vista estable sin metadatos `conf_*` para comparar;
- atributos efectivos que el YAML aplica a la clase del dataset;
- mutaciones runtime localizadas pero no ejecutadas (escalado del LR en L2P/DualPrompt y scheduler dependiente de épocas en CODA-Prompt).

Comandos exactos para Linux, desde la raíz de `mammothV2`, guardando stdout JSON y stderr por separado:

```bash
mkdir -p auditoria_dumps

env -u WANDB_ENTITY -u WANDB_PROJECT \
  python scripts_tfg/dump_config.py --n_epochs 1 \
  > auditoria_dumps/cifar100_n_epochs_1.json \
  2> auditoria_dumps/cifar100_n_epochs_1.stderr.log

env -u WANDB_ENTITY -u WANDB_PROJECT \
  python scripts_tfg/dump_config.py --n_epochs 20 \
  > auditoria_dumps/cifar100_n_epochs_20.json \
  2> auditoria_dumps/cifar100_n_epochs_20.stderr.log
```

Comparación centrada en la vista estable y las derivaciones runtime (requiere `jq`):

```bash
diff -u \
  <(jq -S '.methods[] | {requested_model, stable_effective_args_for_diff, post_parser_runtime_notes}' auditoria_dumps/cifar100_n_epochs_1.json) \
  <(jq -S '.methods[] | {requested_model, stable_effective_args_for_diff, post_parser_runtime_notes}' auditoria_dumps/cifar100_n_epochs_20.json)
```

Prueba local realizada: ambos JSON fueron válidos y contuvieron `l2p`, `dualprompt` y `coda-prompt`; en `stable_effective_args_for_diff` solo cambió `n_epochs` (`1` frente a `20`). La derivación de CODA-Prompt marcó `custom_scheduler.K=1` como inválida y `K=20` como válida. Los JSON de prueba se retiraron.

### 6.1 Réplica de Fase B en `galatzo`

Los dos volcados aportados por el usuario son JSON válidos, declaran `schema_version=1` y el commit esperado. Integridad:

| artefacto dentro del tar | bytes | SHA-256 |
|---|---:|---|
| `auditoria_dumps/cifar100_n_epochs_1.json` | 33.809 | `0156a129b0ce25f2fa8e2f20587904196b14d984cf574d2f0bbed8951f199955` |
| `auditoria_dumps/cifar100_n_epochs_20.json` | 33.764 | `258277b6aa90f79a95172693889eccae4208e07e4cd1fd0ec85652ca130013a9` |
| `auditoria_dumps/cifar100_n_epochs_1.stderr.log` | 284 | `9d7bec0c8276ac90a8a722a3ab59bd8aadc87605a6a72059b89149a87a1c6079` |
| `auditoria_dumps/cifar100_n_epochs_20.stderr.log` | 284 | `9d7bec0c8276ac90a8a722a3ab59bd8aadc87605a6a72059b89149a87a1c6079` |

El diff recursivo exhaustivo encontró 34 hojas distintas. Fuera de `conf_jobnum` y `conf_timestamp`, que son metadatos volátiles, solo cambia `n_epochs` en el scope, comando, parser y configuración efectiva de cada método. CODA-Prompt añade las tres consecuencias derivadas esperadas: `custom_scheduler.K: 1→20`, `runtime_valid: false→true` y `consequence_if_invalid: "AssertionError at begin_task before the first training epoch"→null`. Normalizando exclusivamente esos campos, ambos documentos tienen el mismo SHA-256 canónico: `bd1e8e55d15bce26d1feac5d71fa0183acc7c69468761128c78ad8a646c93fba`.

La frontera del script es esencial: el último paso ejecutado fue `main.check_args(args, dataset)`; no se construyeron backbone/modelo/loaders ni se entrenó. Por eso el código de salida 0 del volcado E=1 confirma que el parser acepta el valor, pero no contradice el bloqueo posterior de CODA en `begin_task`. Las notas de LR y K están marcadas `NOT_APPLIED_BY_THIS_SCRIPT` y son derivaciones estáticas localizadas.

Advertencia reproducible: aunque no entrena, el import eager del parser puede poblar `checkpoints/ViT-B-16/` con unos 599 MB de OpenCLIP. No es una descarga de CIFAR-100 ni una construcción intencionada del método seleccionado; es el efecto lateral descrito en §3.5. Los dos stderr son idénticos y contienen solo el aviso de OpenCLIP de que esos pesos se entrenaron con QuickGELU pero la configuración importada no lo activa. Por la frontera del script, el aviso se registra como efecto lateral del import y no se atribuye a L2P, DualPrompt o CODA-Prompt.

### 6.2 Paquete post-piloto y OOM del Piloto-0 v1

El 2026-08-03 se recibió `C:\Users\alex\Downloads\faseA_servidor_piloto0_oom_20260803_154958.tar.gz`. Su sidecar y el cálculo local coinciden en SHA-256 `6e9fc8d88595f8219f5a60f16274c281e3c4ab077e909d86c304eec88e3e0d25`. Antes de extraerlo fuera del repo se comprobaron sus 19 entradas: no hay rutas absolutas ni componentes `..`. El tar contiene la copia exacta del paquete anterior `auditoria_dumps_e1_e20.tar.gz`, SHA-256 `7254a9033d174398b8e9ef2a35ba1951e3ece32fa0e0a48a51bb49d358fa19c8`.

| artefacto dentro del tar | SHA-256 | uso como evidencia |
|---|---|---|
| `logs/piloto0/diagnostico_oom_20260803_154949.txt` | `b6818073dd03ed61bc76f1d069bfc6ddba42aeaf3c7ba74d349559fee64fbccf` | commit/host/entorno, CLI, estado GPU, código de salida, recursos y traceback OOM. |
| `logs/piloto0/l2p_seq-cifar100-224_seed0_e1.run.log` | `0954299a32d5b70425271ad14d7aab5a92df1c156faffc3fa6ebae0713e18ec7` | configuración efectiva visible y fallo de L2P en tarea 1. |
| `logs/piloto0/l2p_seq-cifar100-224_seed0_e1.time.txt` | `a43f360fa489b40263bf62b52432cd49d8e6ca7781bb7c4ed850eb8ab828e2b7` | comando, 17,21 s hasta el fallo y código 1. |
| `logs/piloto0/run_piloto0.sh` | `5c92f72b12e86323a5fa5cec9cc30fc9bf10107c21fcbde5c815048d5affc21d` | demuestra el encadenamiento v1 y la salida al primer fallo. |

El intento resolvió `model_config=best`, batch 128, LR nominal 0,03 y E=1. L2P falló al intentar reservar 334 MiB: el log registra 11,62 GiB de capacidad, 135,88 MiB libres y 10,79 GiB usados por el proceso. Había además dos kernels ajenos con 304 y 332 MiB. El paquete no permite determinar si liberar esos procesos o activar `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True` habría evitado este OOM concreto.

El lanzador v1 solo contenía L2P y DualPrompt y salía ante el fallo del primero; por tanto DualPrompt no se ejecutó y CODA-Prompt no formó parte del intento. No se generó `logs.pyd`. El paquete tampoco contiene una corrida a batch 64: el hecho previo de que 64 cabe está registrado en `PLAN_MAESTRO.md §10 (hecho duro)`, no demostrado por este tar. La incidencia y el procedimiento posterior se desarrollan en `infra/servidor.md` y `auditoria/piloto0.md`.

La discrepancia de dependencias queda precisada con la misma evidencia del servidor: `requirements.txt:9` declara `ftfy` sin versión y omite `open-clip-torch`, mientras `pyproject.toml:27,33` exige `ftfy>=6.3.1` y `open-clip-torch>=2.32.0`. Antes faltaban ambos imports (`environment_before_openclip.txt:55-58`) y después el overlay aportó exactamente esas versiones (`environment_after_openclip.txt:5-6`). No se modificaron los manifiestos.

### 6.3 Significado de `model_config=best/default`

La interfaz acepta `default`/`base` o `best`, y usa `default` si se omite (`utils/args.py:228-231`). `load_model_config` lee `models/config/<modelo>.yaml`; `default` devuelve el bloque literal `default:` o `{}` si falta, mientras `best` mezcla ese bloque con la entrada cuyo nombre es el dataset (`models/utils/__init__.py:47-82`). Si el overlay contiene `dataset_config`, se carga ese YAML; en otro caso se usa el solicitado por CLI o `default.yaml` (`main.py:169-186`; `datasets/utils/__init__.py:125-151`). La CLI explícita se aplica después (`main.py:308-351`).

Los tres YAML auditados carecen de bloque `default:` y solo tienen una entrada `seq-cifar100-224`:

| método | `best` en Mammoth | `default` efectivo sin `--dataset_config` explícito |
|---|---|---|
| L2P | `models/config/l2p.yaml:1-6`: `batchwise_prompt=1`, `dataset_config=l2p`, LR0,03, pull0,5 y modelo L2P; carga `datasets/configs/seq-cifar100-224/l2p.yaml:1-28` con batch128/E5. | Overlay de modelo `{}` + defaults de `models/l2p.py:26-49` + `datasets/configs/seq-cifar100-224/default.yaml:1-25`. No aporta LR y `--lr` sigue siendo obligatorio (`utils/args.py:263-266`): no es una receta completa por sí solo. |
| DualPrompt | `models/config/dualprompt.yaml:1-6`: `dataset_config=l2p`, batch128, LR0,03, modelo y clave inerte `use_fix_permute=0`; carga el mismo YAML `l2p`. El flag funcional se llama `use_permute_fix` (`models/dualprompt.py:30`). | Overlay `{}` + defaults de `models/dualprompt.py:27-61` + YAML de dataset `default`. Tampoco aporta LR; requiere `--lr` manual. |
| CODA-Prompt | `models/config/coda_prompt.yaml:1-8`: `dataset_config=coda_prompt`, batch128, LR0,001, Adam, `optim_mom=0.9` y E20; carga `datasets/configs/seq-cifar100-224/coda_prompt.yaml:1-23`. | Overlay `{}` + defaults de `models/coda_prompt.py:26-32` —sí incluyen LR0,001/Adam/momento0,9— + YAML de dataset `default`. Usa las estadísticas/transforms ImageNet de `default.yaml`, no la identidad de la receta CODA; fila `CODA-A09`. |

Los comandos de reproducción Mammoth invocan `best` para CODA, DualPrompt y L2P (`scripts/reproduce.json:49-52,79-82,170-173`). La documentación solo afirma que esos hiperparámetros pueden proceder de los autores **o** de búsquedas de Mammoth, sin trazar el origen de cada campo (`docs/getting_started/reproducibility.rst:8-12`); esa autoría parámetro a parámetro queda `NO_DETERMINABLE`.

Por tanto, `best` es un overlay mantenido por Mammoth y no se equipara globalmente a la receta original. La auditoría demuestra, entre otros ejemplos, voto batchwise y LR aplicado0,015 en L2P (`valores_l2p.md`, `L2P-B15/B29`), LR nominal0,03/efectivo0,015 frente a0,005 en DualPrompt (`valores_dualprompt.md:91-92`) y scheduler CODA construido pero no avanzado (`comportamiento_coda.md:7-12`). Algunos escalares coinciden; la procedencia o equivalencia total no se infiere.

### 6.4 Paquete de cierre del Piloto-0 v2

El 2026-08-04 se recibió `C:\Users\alex\Downloads\piloto0_v2_20260804_004240.tar.gz` y su sidecar. El SHA-256 declarado y el calculado localmente coinciden: `c25a5d8431dd5b863ddfd62271ed829f4922fbfae7c7283f174442e6fb3b83f3`. Antes de extraer fuera del repositorio se comprobaron 52 entradas —37 ficheros— sin rutas absolutas, componentes `..` ni tipos especiales. El contenido no se copió a TFG.

| artefacto dentro del tar | bytes | SHA-256 | uso como evidencia |
|---|---:|---|---|
| `logs/piloto0_v2/driver_all.log` | 168.059 | `ca611376fe48219d1e6b0730f3595eda40cd126fb6bc07b8ad2143403ae68859` | host/commit, secuencia de los tres métodos, códigos 0 y final global. `driver_nohup.log` es idéntico y no constituye una segunda ejecución. |
| `logs/piloto0_v2/l2p_seq-cifar100-224_seed0_b64_e1.run.log` | 14.038 | `aaa174b0a444ffbe36d3ec6374aac1bee829f31fbcbde71d5256f76d17877eec` | diez tareas, resultados finales, avisos y memoria L2P. |
| `logs/piloto0_v2/dualprompt_seq-cifar100-224_seed0_b64_e1.run.log` | 13.710 | `6f80a7cedbfb9be52d76e2357c05a53a18ed6ba7919669fea743d2097307471e` | diez tareas, resultados finales, avisos y memoria DualPrompt. |
| `logs/piloto0_v2/coda-prompt_seq-cifar100-224_seed0_b64_e2.run.log` | 12.787 | `fea7856c5602a2ae49c4bccd40893e3f35facf703613228ba99f9efea04cc000` | diez tareas, resultados finales, avisos y memoria CODA-Prompt. |
| `logs/piloto0_v2/l2p_seq-cifar100-224_seed0_b64_e1.time.txt` | 1.088 | `28d3dba1402243a023e32ae0745c9622333b2a31b2644af373fdefeb43e09223` | comando, wall-clock 30:35,89, RSS CPU y salida 0. |
| `logs/piloto0_v2/dualprompt_seq-cifar100-224_seed0_b64_e1.time.txt` | 1.101 | `db46af2a7630fd61e41b8efe049ff946434b91a85e7245cc32ff597019f75ea7` | comando, wall-clock 28:20,38, RSS CPU y salida 0. |
| `logs/piloto0_v2/coda-prompt_seq-cifar100-224_seed0_b64_e2.time.txt` | 1.103 | `9f0ef740dcab5c484d4dbbfb3967a91f351f44c882934b6f40cafcde13b21340` | comando, wall-clock 54:26,92, RSS CPU y salida 0. |
| `logs/piloto0_v2/gpu_preflight.txt` | 2.555 | `5835e55f0210f7268e31dd0c691bb146a67eb962247bee89fa5fcc267edfaafa` | GPU disponible y procesos ajenos antes de ejecutar. |
| `logs/piloto0_v2/metricas_resumen.tsv` | 192 | `05816ed46327283c9157a521b904f9c70d048ca9d7621a0485b2d78190c05cae` | resumen auxiliar; `elapsed` está truncado y no se usa como tiempo autoritativo. |
| `logs/piloto0_v2/resumen_final.txt` | 129 | `e228a74fb2ee05040046fa77fd4abd469b47718d4dfc6d6dfba498a0150b735f` | commit y cuatro códigos de salida 0. |

Los seis resultados tienen una línea cada uno. SHA-256 Class-IL: L2P `4705ba9fa8588d5e0a3f018e23e4df52bb0a26585687e0174243af3d760b8f67`, DualPrompt `a95e5b95dc704e5aba7c6dc6ec6530827e260357f06b8e3887b370e823c82b49`, CODA-Prompt `31fc65ef30a79315dce916774f495f4cba39579fdd2b9198cbd0cbed63d23090`. SHA-256 Task-IL: L2P `416aab5805474329e47bd981d04bbf3949ada6b3c3b0c7b351a850b61520e112`, DualPrompt `9446aa19caa39567d2565473e0f7789d3a2cef8db8fec301227e38333c410fe7`, CODA-Prompt `844a0dd89b5d9aabfe2e5e0fb153b2981967048848e8f27e604156d830aae031`.

El cierre operativo, los localizadores de tarea/resultado y las limitaciones de interpretación se desarrollan en `auditoria/piloto0.md`; el estado de GPU, los tiempos y las incidencias están en `infra/servidor.md`. La evidencia confirma el Piloto-0 v2 solo para `best` diagnóstico, batch64, E1/E1/E2; no modifica D33, el eje ni valores finales.

## 7. Bloqueos y pendientes declarados

1. **CODA-Prompt / validación D39 — RESUELTA (4-ago):** el usuario ejecutó en galatzo `mammothV2@4353bc18897e71362a481ff24e59f025069c1817`. E=5 produjo `D39_TRACE: PASS`, con esperado=observado=`[0.001, 0.001, 0.00092537519929086212, 0.00071263851892520535, 0.00039354082365465138]`; E=1 produjo `D39_E1: PASS` con LR base `0.001`; veredicto global `PASS`. Evidencia de servidor: `scripts_tfg/d39_validation_20260804T203937Z/`. La condición técnica de D15/D39 queda satisfecha y no se activa el fallback `{2,5,20}`.
2. **ImageNet-R opcional:** el dataset existe, pero los tres YAML de modelo carecen de preset `best` para `seq-imagenet-r`; una receta completa requeriría elegir manualmente valores todavía no auditados. Marcado `NO_DETERMINABLE_ESTATICO` en `auditoria/piloto0.md`.
3. **Estado archivado (4-ago):** `google-research/l2p`: **SÍ, ESTÁ ARCHIVADO Y VERIFICADO POR MÍ (USUARIO) EN github.com**; `GT-RIPL/CODA-Prompt`: **NO ARCHIVADO**; `aimagelab/mammoth`: **NO ARCHIVADO**.
4. **Historia del repo TFG:** la descripción de §2 conserva el estado inicial. En fases posteriores se enlazó y actualizó `origin/master`; los entregables de auditoría se publican por petición expresa del usuario.

## 8. Árbol final del workspace (inventario recursivo solicitado)

Inventario real de `auditoria/`, `infra/`, `referencias/`, `papers/` y `prompts_fase_A.md`, obtenido con `Get-ChildItem -Force -Recurse`:

```text
TFG/
├── auditoria/
│   ├── cola_coda.md
│   ├── cola_dualprompt.md
│   ├── cola_l2p.md
│   ├── cola_revision.md
│   ├── comportamiento.md
│   ├── comportamiento_coda.md
│   ├── comportamiento_dualprompt.md
│   ├── comportamiento_l2p.md
│   ├── entorno.md
│   ├── fuentes.md
│   ├── piloto0.md
│   ├── plantilla.md
│   ├── reconciliacion_fase_b.md
│   ├── valores_coda.md
│   ├── valores_dualprompt.md
│   └── valores_l2p.md
├── infra/
│   └── servidor.md
├── referencias/
│   ├── dossier_metodologia_conflictos.md
│   └── dossier_protocolo_cil.md
├── papers/
│   ├── coda-prompt/
│   │   ├── arxiv_2211.13218.pdf
│   │   ├── arxiv_2211.13218_source.tar.gz
│   │   ├── arxiv_api_2211.13218.xml
│   │   └── source_v2/
│   │       ├── cvpr.sty
│   │       ├── figures/
│   │       │   ├── capacity_sweep.tex
│   │       │   ├── main_fig.tex
│   │       │   ├── method_fig.tex
│   │       │   └── source/
│   │       │       ├── approach-new.pdf
│   │       │       ├── cvpr-len-sweep.png
│   │       │       ├── cvpr-pool-sweep.png
│   │       │       └── key_idea.pdf
│   │       ├── ieee_fullname.bst
│   │       ├── main.bbl
│   │       ├── main.tex
│   │       ├── sections/
│   │       │   ├── 0_abstract.tex
│   │       │   ├── 1_intro.tex
│   │       │   ├── 2_related.tex
│   │       │   ├── 3_prelim.tex
│   │       │   ├── 4_method.tex
│   │       │   ├── 5_experiments.tex
│   │       │   ├── 6_conclusions.tex
│   │       │   └── 7_appendix.tex
│   │       ├── tables/
│   │       │   ├── ablations.tex
│   │       │   ├── cifar100_domainnet.tex
│   │       │   ├── domainshift.tex
│   │       │   └── imagenet-r.tex
│   │       └── tables_sup/
│   │           ├── cifar100.tex
│   │           └── imagenet-r_5.tex
│   ├── dualprompt/
│   │   ├── arxiv_2204.04799.pdf
│   │   ├── arxiv_2204.04799_source.tar.gz
│   │   ├── arxiv_api_2204.04799.xml
│   │   └── source_v2/
│   │       ├── axessibility.lua
│   │       ├── axessibility.sty
│   │       ├── comment.sty
│   │       ├── dual_prompt_camera_ready.tex
│   │       ├── figures/
│   │       │   ├── dual_prompt.pdf
│   │       │   ├── DualPrompts.pdf
│   │       │   ├── DualPrompts_new.pdf
│   │       │   ├── DualPrompts_new2.pdf
│   │       │   ├── DualPrompts_new3.pdf
│   │       │   ├── imagenet_r.pdf
│   │       │   ├── intro_compare.pdf
│   │       │   ├── method.pdf
│   │       │   ├── overview.pdf
│   │       │   ├── prompt_combiner.pdf
│   │       │   ├── prompt_visualization.pdf
│   │       │   ├── val_prompt_depth.pdf
│   │       │   └── val_prompt_length.pdf
│   │       ├── llncs.cls
│   │       ├── math_commands.tex
│   │       ├── refs.bib
│   │       ├── ruler.sty
│   │       └── splncs04.bst
│   └── l2p/
│       ├── arxiv_2112.08654.pdf
│       ├── arxiv_2112.08654_source.tar.gz
│       ├── arxiv_api_2112.08654.xml
│       └── source_v2/
│           ├── cvpr.sty
│           ├── figures/
│           │   ├── cifar_5datasets_hists.pdf
│           │   ├── cifar100_ablation_N_L.pdf
│           │   ├── five_datasets_ablation_N_L.pdf
│           │   ├── method.pdf
│           │   ├── overview.pdf
│           │   ├── overview3.pdf
│           │   └── promp_pool_size.pdf
│           ├── ieee_fullname.bst
│           ├── L2P_arxiv.tex
│           ├── math_commands.tex
│           └── README.md
└── prompts_fase_A.md
```
