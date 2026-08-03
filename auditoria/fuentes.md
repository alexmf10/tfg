# Registro de fuentes y montaje del workspace

Fecha de comprobación: 2026-08-02. Zona horaria del workspace: Europe/Madrid.

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
- Único commit añadido: `cda7f23681f7bffacee460d99e990bc803bccf04` (`Add TFG effective config dump`).
- HEAD final de `tfg-auditoria`: `cda7f23681f7bffacee460d99e990bc803bccf04`.
- Push: `git push -u origin tfg-auditoria` terminó correctamente; `origin/tfg-auditoria` apunta al mismo SHA.
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
- Estado «¿archivado?»: **PENDIENTE DE VERIFICACIÓN POR EL USUARIO EN github.com**, como exige la tarea; no se deduce del clon.

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
- Estado «¿archivado?»: **PENDIENTE DE VERIFICACIÓN POR EL USUARIO EN github.com**.

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

## 7. Bloqueos y pendientes declarados

1. **CODA-Prompt / Piloto-0 con una época:** `CodaPrompt.begin_task` deriva `CosineSchedule.K=n_epochs`, pero `CosineSchedule` exige `K>1`. No existe un comando CLI que complete exactamente una época por tarea sin modificar código. Evidencia y comando diagnóstico en `auditoria/piloto0.md`.
2. **ImageNet-R opcional:** el dataset existe, pero los tres YAML de modelo carecen de preset `best` para `seq-imagenet-r`; una receta completa requeriría elegir manualmente valores todavía no auditados. Marcado `NO_DETERMINABLE_ESTATICO` en `auditoria/piloto0.md`.
3. **Estado archivado:** pendiente de comprobación humana en github.com para los repos oficiales.
4. **Historia del repo TFG:** la descripción de §2 conserva el estado inicial. En fases posteriores se enlazó y actualizó `origin/master`; los entregables de auditoría se publican por petición expresa del usuario.

## 8. Árbol final del workspace (2 niveles)

Comando PowerShell utilizado; enumera la raíz y un nivel interior, y omite únicamente el contenido interno de cada `.git`:

```powershell
Get-ChildItem -Force | Sort-Object Name | ForEach-Object {
    $_.Name
    if ($_.PSIsContainer -and $_.Name -ne '.git') {
        Get-ChildItem -LiteralPath $_.FullName -Force |
            Sort-Object Name |
            ForEach-Object { "  $($_.Name)" }
    }
}
```

```text
TFG/
├── .git/
├── auditoria/
│   ├── fuentes.md
│   ├── piloto0.md
│   └── plantilla.md
├── codaprompt/
│   ├── .git/
│   ├── configs/
│   ├── dataloaders/
│   ├── experiments/
│   ├── learners/
│   ├── models/
│   ├── utils/
│   ├── .gitignore
│   ├── data
│   ├── install-requirements.sh
│   ├── LICENSE
│   ├── main-idea_coda-p.png
│   ├── method_coda-p.png
│   ├── README.md
│   ├── requirements.txt
│   ├── run.py
│   ├── run-all.sh
│   └── trainer.py
├── l2p/
│   ├── .git/
│   ├── augment/
│   ├── configs/
│   ├── helper/
│   ├── libml/
│   ├── models/
│   ├── CONTRIBUTING.md
│   ├── dualprompt_illustration.png
│   ├── l2p_illustration.png
│   ├── LICENSE
│   ├── main.py
│   ├── README.md
│   ├── requirements.txt
│   └── train_continual.py
├── mammoth/                         (vacío)
├── mammothV2/
│   ├── .git/
│   ├── .github/
│   ├── backbone/
│   ├── checkpoints/                    (vacío)
│   ├── datasets/
│   ├── docs/
│   ├── examples/
│   ├── hub/
│   ├── models/
│   ├── scripts/
│   ├── scripts_tfg/
│   ├── tests/
│   ├── utils/
│   ├── .gitignore
│   ├── __init__.py
│   ├── AGENTS.md
│   ├── gem_license
│   ├── LICENSE
│   ├── main.py
│   ├── NOTICE.md
│   ├── pyproject.toml
│   ├── py.typed
│   ├── README.md
│   ├── REPRODUCIBILITY.md
│   ├── requirements-optional.txt
│   └── requirements.txt
├── papers/
│   ├── coda-prompt/
│   ├── dualprompt/
│   └── l2p/
├── .gitignore
├── PLAN_MAESTRO.md
└── prompts_fase_A.md
```

Ausencias intencionadas: `dualprompt/` (estaba vacío y se eliminó) y `repos_aux/` (no se usó la opción de clonar reimplementaciones).
