# Prompts de la Fase A — Auditoría de configuraciones
## v3 — workspace real `Documentos/TFG/` (mammothV2 + repos existentes)

## Instrucciones de uso (para ti, no para Codex)

0. **Único preliminar (opcional, recomendado):** un repositorio privado vacío `tfg` en tu GitHub para versionar el workspace. Escribe su URL en el hueco `<https://github.com/alexmf10/tfg>` del PROMPT 0, o escribe NO y el repo TFG quedará solo local. Ya no hace falta fork de mammoth: `mammothV2` existe en tu GitHub (alexmf10/mammothV2).
1. **TODOS los chats de Codex de la Fase A se abren con `C:\Users\alex\Documents\TFG\` como carpeta de trabajo** — también los tres de método. Cada auditoría cruza a la vez el paper (`papers/`), el repo oficial, `mammothV2/` y `auditoria/`; un chat abierto dentro de una subcarpeta no puede hacerla. Los tres chats paralelos existen por aislamiento de contexto, no por carpetas.
2. **Orden:** PROMPT 0 en un chat propio → verifica y completa el workspace (procedencia de mammothV2, clones que falten, retirada de `dualprompt/`, papers) y crea plantilla, script de volcado y comandos del piloto. Cuando termine: **tres chats en paralelo** con los PROMPTS 1, 2 y 3. Al acabar los tres: PROMPT 4 (consolidación), en el chat de la Tarea 0 si sigue fino o en uno nuevo.
3. **En paralelo, en cuanto exista `auditoria/piloto0.md`:** en el servidor, clona https://github.com/alexmf10/mammothV2 (si no está), `git checkout tfg-auditoria`, y lanza los comandos del piloto. No espera a la auditoría.
4. **Regla de tránsito:** lo que se ejecuta viaja por la rama de mammothV2 (commit + push de Codex → pull en el servidor); lo que se lee vive en TFG. Por eso `dump_config.py` está en `mammothV2/scripts_tfg/` y las tablas en `auditoria/`.
5. Tras la consolidación: ejecuta en el servidor `mammothV2/scripts_tfg/dump_config.py` **dos veces** (E=1 y E=20) según las instrucciones de `auditoria/fuentes.md`, guarda las dos salidas y pásaselas al chat de consolidación para la reconciliación. Después, tu **muestreo de 8–10 celdas** al azar contra las fuentes. Con eso hecho, tráeme `cola_revision.md` + los informes de embudo + los tiempos del Piloto-0 y hacemos la reunión de Fase B.
6. Estos prompts **no contienen valores esperados a propósito**: todos los valores deben salir de las fuentes. Si algún chat de Codex desvaría, se mata y se abre otro: los ficheros de `auditoria/` son la memoria, no el chat.
7. El repo antiguo (`Documents/Mammuth`, github.com/alexmf10/mammoth) queda en **cuarentena**: fuera del workspace y jamás fuente — ni para Codex ni para ti (D14 del plan).

---

## PROMPT 0 — TAREA 0: verificar y completar el workspace, fijar fuentes, plantilla y scripts

```
# TAREA 0 — VERIFICAR Y COMPLETAR EL WORKSPACE, Y PREPARAR LA AUDITORÍA

## Contexto
TFG: comparación controlada de L2P, DualPrompt y CODA-Prompt en class-incremental learning sobre el framework Mammoth. El directorio de trabajo actual es Documentos/TFG y está PARCIALMENTE montado por el usuario: existe al menos mammothV2/ (copia limpia del framework, repo propio en https://github.com/alexmf10/mammothV2) y pueden existir carpetas l2p/, dualprompt/ y codaprompt/ (vacías o con contenido). Vas a inventariar lo que hay, verificarlo, completar lo que falte y preparar la base de una auditoría de configuraciones (paper vs repo oficial vs Mammoth). Esta tarea es de verificación, LECTURA y creación de ficheros base; no es programación sobre el framework.

## Reglas de oro (aplican a toda la auditoría)
- PROHIBIDO inventar, rellenar huecos con conocimiento general o inferir sin marcarlo.
- NO destruyas ni modifiques nada existente, salvo lo explícitamente indicado abajo.
- Tras el montaje, el código de mammothV2 y de los repos oficiales es de SOLO LECTURA: prohibido modificar código, configs o dependencias. Únicas excepciones: los entregables listados al final.
- Si un comando de red falla (clone, fetch, descarga), NO lo simules: registra en auditoria/fuentes.md el comando exacto para que el usuario lo ejecute a mano, y continúa con el resto.
- Toda afirmación con evidencia localizable. Las ambigüedades se registran, no se resuelven en silencio.
- Entregables como FICHEROS (commit al terminar), no como texto de chat.

## Paso 0 — Verificar y completar el workspace
1. Verifica que el directorio actual es .../Documentos/TFG. Si no lo es, DETENTE y dilo.
2. INVENTARIO INICIAL: lista las carpetas existentes y, para cada una que sea repo git: remoto origin, SHA de HEAD, rama actual y si el working tree está limpio. Regístralo todo (irá a fuentes.md).
3. Inicializa TFG como repo git (git init si no lo es) y crea un .gitignore que excluya: mammothV2/, l2p/, dualprompt/, codaprompt/, repos_aux/, results*/, data/, datasets/, *.pth, __pycache__/, .DS_Store, Thumbs.db.
4. mammothV2/ (verificación de procedencia):
   - Confirma que su origin es https://github.com/alexmf10/mammothV2 y que el working tree está limpio. Si no está limpio, DETENTE en este punto y repórtalo.
   - Añade un remoto upstream = https://github.com/aimagelab/mammoth, haz fetch, y comprueba si el HEAD actual corresponde a un commit existente en upstream. Si sí: registra ese SHA como BASE OFICIAL VERIFICADA. Si el historial no conecta con upstream (p. ej. copia de ficheros sin historia git): regístralo, identifica el commit de upstream más parecido y verifica limpieza con un diff contra él; si la equivalencia no es determinable, márcalo BLOQUEANTE para el usuario y continúa con el resto de pasos.
   - Crea la rama de trabajo tfg-auditoria desde HEAD y registra su SHA.
5. l2p/: si ya es un clon de https://github.com/google-research/l2p, registra su SHA; si está vacía o no existe, clónalo ahí; si contiene otra cosa, regístralo y márcalo BLOQUEANTE sin tocar nada. Igual para codaprompt/ con https://github.com/GT-RIPL/CODA-Prompt. En ambos repos oficiales: busca tags/releases y, si existe uno cercano a la fecha de publicación del paper correspondiente, haz checkout de esa VERSIÓN DE REFERENCIA; si no existe, quédate en la rama principal y registra SHA y fecha del último commit. NOTA: google-research/l2p contiene L2P Y DualPrompt — es el repo oficial de ambos. El estado "archivado" de un repo no se ve desde el clon: márcalo como pendiente de verificación por el usuario en github.com.
6. dualprompt/: NO es fuente oficial (DualPrompt vive en l2p/). Si está vacía, elimínala y anótalo. Si contiene algo, NO lo borres: registra su contenido en fuentes.md y déjala explícitamente FUERA de la auditoría.
7. Opcional (solo evidencia auxiliar futura): puedes clonar JH-LEE-KR/l2p-pytorch y JH-LEE-KR/dualprompt-pytorch en repos_aux/. NO son fuente oficial.
8. Crea las carpetas auditoria/ y papers/ (con papers/l2p/, papers/dualprompt/, papers/coda-prompt/) si no existen.
9. Descarga cada paper a su carpeta: el PDF (https://arxiv.org/pdf/ID) y, sobre todo, la FUENTE LaTeX (https://arxiv.org/e-print/ID; extráela): la fuente permite localizar y citar secciones con exactitud. IDs: L2P 2112.08654 · DualPrompt 2204.04799 · CODA-Prompt 2211.13218. Registra la versión de arXiv descargada si es determinable; si no, márcala como pendiente de comprobación por el usuario en la página abs.
10. Si <https://github.com/alexmf10/tfg> no es NO: añade ese remoto al repo TFG (el push lo decide el usuario).

## Paso 1 — Registro de fuentes (auditoria/fuentes.md)
1. Inventario inicial del workspace (paso 0.2) y acciones realizadas sobre él.
2. mammothV2: SHA de HEAD, SHA de la rama tfg-auditoria, y resultado de la verificación de procedencia (BASE OFICIAL VERIFICADA con su SHA, o el estado alternativo). Versiones del entorno relevantes (python, torch, timm) según requirements/lockfiles del repo; si hay un entorno instalado accesible, también las versiones reales.
3. Para l2p/ y codaprompt/: SHA leído, versión de referencia elegida y su justificación (tag cercano a publicación, o principal con fecha), y el pendiente de "¿archivado?".
4. Reimplementaciones auxiliares (si se clonaron en repos_aux/): SHA y recordatorio de que NUNCA rellenan la columna "repo oficial".
5. Papers: ID de arXiv, versión (o pendiente), y ruta local de PDF y fuente LaTeX.
6. Árbol final del workspace (con el comando de árbol disponible en el sistema, 2 niveles).

## Paso 2 — Plantilla de la auditoría (auditoria/plantilla.md)
Crea el fichero que los tres prompts de método seguirán como autoridad. Debe contener, literal y sin ambigüedad:

A) COLUMNAS DE LA TABLA (12, en este orden): parámetro | ámbito (dataset: CIFAR-100 / ImageNet-R / ambos) | cubo (A / B / EJE) | paper (valor + evidencia) | repo oficial (valor + evidencia) | Mammoth (valor + evidencia + cadena de resolución) | caso (nombre) | grupo | valor propuesto | valor final | flags | observaciones.
- La evidencia NO es columna aparte: vive dentro de la celda de cada fuente, para que nunca haya duda de a qué fuente pertenece cada prueba.
- valor propuesto: lo rellena la cascada (solo cubo B). valor final: SIEMPRE VACÍO en la Fase A, en todos los cubos — lo rellena la revisión humana en las Fases B–D (cubo B: idéntico al propuesto salvo decisión registrada; cubo A: se decide en Fase D; EJE: no aplica, lo define la escalera de presupuestos). Codex NUNCA escribe en valor final.
- observaciones: nota corta. Si no cabe en una línea (cadena de evidencia larga, razonamiento del caso, historia de un grupo), se escribe un código OBS-nn en la celda y el contenido completo va a una sección "Notas" al final del mismo fichero, una entrada por código. Toda fila cuyo valor final acabe difiriendo del propuesto documentará su flujo de decisión en su nota OBS.

B) CUBOS:
- CUBO A (entorno compartido; una fila, sin método; SOLO evidencia de qué usa cada fuente, valor propuesto vacío): backbone y checkpoint EXACTO (identificador del fichero de pesos / nombre timm), datasets y particiones, orden de clases y semillas (cómo lo fija cada fuente), augmentación y preprocesado, tamaño de batch, protocolo de evaluación (task-id, cabeza, métricas), buffer/memoria.
- EJE (solo evidencia, valor propuesto vacío): n_epochs / iteraciones y todo lo ligado a la duración del entrenamiento (schedulers, warmup, early stopping, cálculos derivados de la duración).
- CUBO B (receta del método; fila por método): todo lo demás. Se aplica la cascada.
- Cubo dudoso: asignar provisionalmente y marcar CUBO_DUDOSO.

C) CASCADA (solo cubo B, por fila; siempre con nombre):
- CASO 1 COINCIDEN: paper y repo oficial especifican y coinciden → ese valor.
- CASO 2 EXPLICADO: especifican y discrepan CON explicación de los autores (tag de la época, changelog, issue, errata) → decide la explicación, citada.
- CASO 3 CONFLICTO→PAPER: especifican y discrepan SIN explicación → valor del paper por convención declarada. Marcar REVISAR.
- CASO 4 FUENTE_ÚNICA: solo una fuente especifica → esa fuente. Etiquetar SOLO_PAPER o SOLO_REPO.
- CASO 5 SIN_FUENTE: ninguna especifica → default efectivo de Mammoth. Etiquetar DEFAULT_MAMMOTH.

D) GRUPOS ACOPLADOS (columna "grupo"): parámetros que los autores ajustaron juntos o que dependen entre sí. Nombres tipo G-OPT (learning rate, scheduler, batch, duración), G-PROMPT (longitud, pool, top-k, capas de inserción), más los que identifiques con justificación. Regla: si una fila de un grupo cae en CONFLICTO→PAPER, todos los miembros del grupo también en conflicto se resuelven desde el paper EN BLOQUE. Si el valor propuesto de un grupo acaba mezclando fuentes (unos miembros del paper, otros del código), TODAS sus filas reciben la marca GRUPO_MIXTO y el grupo entra en la cola de revisión — no lo resuelvas tú. Los acoplamientos internos de Mammoth (un ajuste que cambia el comportamiento de otro) también se anotan.

E) FLAG S (sensibilidad; transversal, compatible con cualquier caso): proponerla cuando (a) haya indicios de alto impacto, (b) los valores en conflicto estén muy alejados, o (c) el valor elegido NO sea configurable en Mammoth. SIN TOPE de propuestas: registra TODAS las que cumplan los criterios, cada una con 1–2 líneas de justificación, evidencia y una severidad ALTA/MEDIA/BAJA, y preséntalas ordenadas por severidad. No infles la lista con dudas genéricas: cada S debe apoyarse en evidencia concreta. Ejecutar experimentos de sensibilidad NO es parte de la auditoría.

F) CRITERIO DE FILA: un parámetro tiene fila si y solo si (1) aparece en el setup experimental o apéndices del paper, O es configuración del experimento en el repo oficial, O es configurable en Mammoth y aplica a estos tres modelos, sus datasets o el bucle de entrenamiento/evaluación; Y ADEMÁS (2) puede afectar plausiblemente a resultados o a coste. Lo que solo afecta a infraestructura (logging, rutas, num_workers, device, guardado de checkpoints) va a una LISTA AGRUPADA al final del fichero, sin fila ni evidencia línea a línea. En caso de duda: fila, marcada LOW. No hay número objetivo de filas.

G) INFORME DE EMBUDO (obligatorio al final de cada tabla): cuántos parámetros se examinaron, cuántos obtuvieron fila, cuántos fueron a la lista agrupada, con los motivos de exclusión agrupados por categoría.

H) FORMATO DE EVIDENCIA (obligatorio por celda): paper → sección/tabla/apéndice (localizada en la fuente LaTeX de papers/; página del PDF si es posible); repo oficial → SHA + fichero + línea(s); Mammoth → SHA + fichero + línea(s) + cadena de resolución completa (default global → config del modelo → config del dataset → CLI), indicando qué nivel fija el valor efectivo. Etiquetas de ausencia: NO_ENCONTRADO (buscado y no hallado) / NO_APLICA (el concepto no existe para ese método o fuente) / NO_DISPONIBLE (fuente ausente) / NO_CONFIGURABLE (existe en Mammoth pero hardcodeado). Valores derivados: DERIVADO + fórmula. Comportamientos no determinables leyendo: NO_DETERMINABLE_ESTATICO + propuesta de script mínimo de comprobación.

I) DISCREPANCIAS DE COMPORTAMIENTO: diferencias de conducta, no de valor (tipo de prompting prefix vs prompt-tuning, cálculo de la query, enmascarado de logits en entrenamiento, manejo de la cabeza entre tareas, inicializaciones, trucos). Van al fichero comportamiento_{método}.md: descripción breve, evidencia en ambos lados, ¿configurable en Mammoth?, propuesta de flag S si procede. Solo documentar, nunca corregir.

## Paso 3 — Script de volcado (mammothV2/scripts_tfg/dump_config.py)
Crea la carpeta mammothV2/scripts_tfg/ DENTRO del repo mammothV2 (rama tfg-auditoria) y ahí el script: para cada uno de los tres métodos (l2p, dualprompt, coda-prompt) sobre CIFAR-100, construye la configuración efectiva de arranque exactamente como lo haría Mammoth (mismos parsers y mecanismos de configs) y la imprime completa, SIN entrenar nada. Debe aceptar el número de épocas como argumento, porque se ejecutará DOS VECES (n_epochs=1 y n_epochs=20) para comparar el diff y ver qué se deriva de la duración. Commitéalo en la rama tfg-auditoria (única escritura permitida dentro de mammothV2) y haz push a origin (alexmf10/mammothV2) si hay credenciales; si el push falla, regístralo en fuentes.md. Documenta en auditoria/fuentes.md los comandos exactos de las dos ejecuciones TAL COMO SE LANZARÁN EN EL SERVIDOR (Linux, desde la raíz del clon de mammothV2) y cómo guardar las dos salidas. No toques nada más del repo.

## Paso 4 — Comandos del Piloto-0 (auditoria/piloto0.md)
Escribe los comandos exactos de la CLI existente de Mammoth (sin modificar código) para ejecutar, por método, una corrida completa de CIFAR-100 con 1 época por tarea, las 10 tareas, hasta generar el fichero de métricas finales, con una semilla fija, e indicando dónde quedan los resultados y los logs de tiempo. Contexto de ejecución: el SERVIDOR del usuario (Linux), en un clon de https://github.com/alexmf10/mammothV2 con la rama tfg-auditoria; escribe los comandos desde la raíz de mammothV2. Añade, marcados como opcionales, los comandos equivalentes para Split ImageNet-R si el dataset está soportado en el repo. Si algo no puede determinarse leyendo (p. ej. si el dataset se descarga solo), indícalo.

## Entregables (commit)
En el repo TFG: .gitignore · auditoria/fuentes.md · auditoria/plantilla.md · auditoria/piloto0.md
En mammothV2 (rama tfg-auditoria): scripts_tfg/dump_config.py (commit + push si es posible)
```

---

## PROMPT 1 — AUDITORÍA L2P

```
# AUDITORÍA DE CONFIGURACIÓN — L2P (paper vs repo oficial vs Mammoth)

## Antes de nada
Lee auditoria/fuentes.md y auditoria/plantilla.md. Son la AUTORIDAD de esta tarea: columnas, cubos, cascada, grupos, flag S, criterio de fila, formato de evidencia y etiquetas. Todo lo que hagas debe cumplirlos. Usa las versiones/SHAs registrados en fuentes.md.

## Rutas de fuentes en este workspace
- Paper: papers/l2p/ (usa la FUENTE LaTeX para localizar secciones con exactitud; el PDF es apoyo).
- Repo oficial: l2p/ en su versión de referencia registrada en fuentes.md.
- Mammoth: mammothV2/ en la rama tfg-auditoria (la implementación de L2P: modelo, backbone y configs implicados).

## Recordatorio crítico (resumen; ante conflicto manda plantilla.md)
- Solo lectura. Prohibido inventar o inferir sin marcar. Evidencia localizable en cada celda.
- Cascada solo en cubo B: COINCIDEN / EXPLICADO / CONFLICTO→PAPER / FUENTE_ÚNICA / SIN_FUENTE.
- Filas de cubo A y EJE: solo evidencia, valor propuesto vacío. La columna valor final queda SIEMPRE vacía en todos los cubos: la rellena la revisión humana, no tú.
- Criterio de fila + lista agrupada de infraestructura + en duda → fila LOW + informe de embudo al final.
- Flags S: propón TODAS las que cumplan los criterios de plantilla.md, con severidad (ALTA/MEDIA/BAJA), justificación y evidencia; sin tope — el tope existe solo en la ejecución posterior, no en el registro.
- Este prompt no contiene valores esperados a propósito: todos los valores salen de las fuentes.

## Tarea
1. INVENTARIO: enumera la UNIÓN de parámetros y configuraciones que aparezcan en cualquiera de las tres fuentes para L2P: hiperparámetros de entrenamiento, hiperparámetros del método (prompts, pool, selección, pérdidas), preprocesado/augmentación, evaluación. Ámbito principal: CIFAR-100. Si una fuente reporta también ImageNet-R en la misma lectura, registra esas filas con su ámbito; no hagas búsqueda extra por ImageNet-R.
2. TABLA auditoria/valores_l2p.md según plantilla, con cascada aplicada en cubo B.
3. COMPORTAMIENTO auditoria/comportamiento_l2p.md según plantilla (sección I).
4. COLA auditoria/cola_l2p.md: solo lo que requiere decisión humana — filas CONFLICTO→PAPER, GRUPO_MIXTO, propuestas S, CUBO_DUDOSO, NO_CONFIGURABLE, hallazgos de comportamiento relevantes. Una línea por ítem con referencia a su fila.

## Filas y comprobaciones OBLIGATORIAS para L2P (además de lo que salga del inventario)
- Tipo de prompting que implementa el "l2p" de Mammoth: ¿inserción de prompts en la entrada (prompt-tuning, como el paper original) o prefix-tuning en varias capas (estilo "L2P++" de la literatura posterior)? Evidencia en código. Va a comportamiento_l2p.md y es candidata natural a flag S.
- Pool de prompts: tamaño, longitud del prompt, top-k de selección, mecanismo de query/matching y su pérdida (incluido cómo se calcula la query y si requiere un forward extra del backbone).
- Optimizador y learning rate: valores y si el LR es constante o con schedule; batch.
- Enmascarado de logits de clases no vistas durante el entrenamiento; manejo de la cabeza clasificadora entre tareas.
- n_epochs: ¿es parámetro limpio de CLI/config en Mammoth para este método? ¿A qué se propaga (schedulers, warmup, criterios de parada, cualquier cálculo derivado de la duración)? ¿Algo más codifica duración? Fila(s) EJE.
- Checkpoint del backbone: identificador EXACTO del fichero de pesos / nombre timm que usa cada fuente. Fila cubo A, solo evidencia.
- Batch y acumulación de gradiente: ¿existe soporte/opción de acumulación en Mammoth aplicable a este método? Fila cubo A, solo evidencia.
- Augmentación/preprocesado y protocolo de evaluación: qué hace cada fuente. Filas cubo A, solo evidencia.
- Cómo fija Mammoth el orden de clases y la semilla. Fila cubo A, solo evidencia.

## Entregables (commit en el repo TFG)
auditoria/valores_l2p.md · auditoria/comportamiento_l2p.md · auditoria/cola_l2p.md (con informe de embudo al final de la tabla)
```

---

## PROMPT 2 — AUDITORÍA DUALPROMPT

```
# AUDITORÍA DE CONFIGURACIÓN — DUALPROMPT (paper vs repo oficial vs Mammoth)

## Antes de nada
Lee auditoria/fuentes.md y auditoria/plantilla.md. Son la AUTORIDAD de esta tarea: columnas, cubos, cascada, grupos, flag S, criterio de fila, formato de evidencia y etiquetas. Usa las versiones/SHAs registrados en fuentes.md.

## Rutas de fuentes en este workspace
- Paper: papers/dualprompt/ (usa la FUENTE LaTeX para localizar secciones; el PDF es apoyo).
- Repo oficial: l2p/ — ATENCIÓN: es el MISMO repo que el de L2P (google-research/l2p contiene DualPrompt); usa su versión de referencia de fuentes.md. La carpeta dualprompt/ del workspace, si existe, NO es fuente: ignórala.
- Mammoth: mammothV2/ en la rama tfg-auditoria (la implementación de DualPrompt).

## Recordatorio crítico (resumen; ante conflicto manda plantilla.md)
- Solo lectura. Prohibido inventar o inferir sin marcar. Evidencia localizable en cada celda.
- Cascada solo en cubo B: COINCIDEN / EXPLICADO / CONFLICTO→PAPER / FUENTE_ÚNICA / SIN_FUENTE.
- Filas de cubo A y EJE: solo evidencia, valor propuesto vacío. La columna valor final queda SIEMPRE vacía en todos los cubos: la rellena la revisión humana, no tú.
- Criterio de fila + lista agrupada de infraestructura + en duda → fila LOW + informe de embudo al final.
- Flags S: propón TODAS las que cumplan los criterios de plantilla.md, con severidad (ALTA/MEDIA/BAJA), justificación y evidencia; sin tope — el tope existe solo en la ejecución posterior, no en el registro.
- Este prompt no contiene valores esperados a propósito: todos los valores salen de las fuentes.

## Tarea
1. INVENTARIO: unión de parámetros de las tres fuentes para DualPrompt (entrenamiento, método, preprocesado, evaluación). Ámbito principal: CIFAR-100; las filas de ImageNet-R se registran si aparecen en la misma lectura, sin búsqueda extra.
2. TABLA auditoria/valores_dualprompt.md según plantilla, con cascada en cubo B.
3. COMPORTAMIENTO auditoria/comportamiento_dualprompt.md.
4. COLA auditoria/cola_dualprompt.md (mismo criterio que la plantilla, incluyendo GRUPO_MIXTO).

## Filas y comprobaciones OBLIGATORIAS para DualPrompt
- G-Prompt: longitud y capas de inserción. E-Prompt: longitud, capas de inserción, organización del pool (¿indexado por tarea?) y cómo se selecciona en test (sin task-id). Pérdida de matching y su peso.
- Tipo de inserción implementado en Mammoth (¿prefix-tuning como el paper?): evidencia en código; a comportamiento si difiere de alguna fuente.
- Cálculo de la query (¿forward extra del backbone?); enmascarado de logits; manejo de la cabeza entre tareas.
- Optimizador y learning rate (¿constante o schedule?); batch.
- n_epochs: ¿parámetro limpio de CLI/config para este método en Mammoth? ¿A qué se propaga? ¿Algo más codifica duración? Fila(s) EJE.
- Checkpoint del backbone: identificador EXACTO por fuente. Cubo A, solo evidencia.
- Batch y soporte de acumulación de gradiente en Mammoth. Cubo A, solo evidencia.
- Augmentación/preprocesado, protocolo de evaluación, orden de clases y semilla en Mammoth. Cubo A, solo evidencia.

## Entregables (commit en el repo TFG)
auditoria/valores_dualprompt.md · auditoria/comportamiento_dualprompt.md · auditoria/cola_dualprompt.md (con informe de embudo)
```

---

## PROMPT 3 — AUDITORÍA CODA-PROMPT

```
# AUDITORÍA DE CONFIGURACIÓN — CODA-PROMPT (paper vs repo oficial vs Mammoth)

## Antes de nada
Lee auditoria/fuentes.md y auditoria/plantilla.md. Son la AUTORIDAD de esta tarea: columnas, cubos, cascada, grupos, flag S, criterio de fila, formato de evidencia y etiquetas. Usa las versiones/SHAs registrados en fuentes.md.

## Rutas de fuentes en este workspace
- Paper: papers/coda-prompt/ (usa la FUENTE LaTeX para localizar secciones; el PDF es apoyo).
- Repo oficial: codaprompt/ en su versión de referencia de fuentes.md.
- Mammoth: mammothV2/ en la rama tfg-auditoria (la implementación de CODA-Prompt).

## Recordatorio crítico (resumen; ante conflicto manda plantilla.md)
- Solo lectura. Prohibido inventar o inferir sin marcar. Evidencia localizable en cada celda.
- Cascada solo en cubo B: COINCIDEN / EXPLICADO / CONFLICTO→PAPER / FUENTE_ÚNICA / SIN_FUENTE.
- Filas de cubo A y EJE: solo evidencia, valor propuesto vacío. La columna valor final queda SIEMPRE vacía en todos los cubos: la rellena la revisión humana, no tú.
- Criterio de fila + lista agrupada de infraestructura + en duda → fila LOW + informe de embudo al final.
- Flags S: propón TODAS las que cumplan los criterios de plantilla.md, con severidad (ALTA/MEDIA/BAJA), justificación y evidencia; sin tope — el tope existe solo en la ejecución posterior, no en el registro.
- Este prompt no contiene valores esperados a propósito: todos los valores salen de las fuentes.

## Tarea
1. INVENTARIO: unión de parámetros de las tres fuentes para CODA-Prompt (entrenamiento, método, preprocesado, evaluación). Ámbito principal: CIFAR-100; las filas de ImageNet-R se registran si aparecen en la misma lectura, sin búsqueda extra.
2. TABLA auditoria/valores_coda.md según plantilla, con cascada en cubo B.
3. COMPORTAMIENTO auditoria/comportamiento_coda.md.
4. COLA auditoria/cola_coda.md (mismo criterio que la plantilla, incluyendo GRUPO_MIXTO).

## Filas y comprobaciones OBLIGATORIAS para CODA-Prompt
- SCHEDULER (máxima prioridad; miembro de G-OPT y candidato natural a flag S): tipo de schedule del learning rate, su duración total T, y — crítico — ¿T se deriva automáticamente de n_epochs o está fijado aparte? ¿El schedule es por-tarea (se reinicia en cada tarea) o global? ¿Hay warmup y de qué depende? Evidencia en las tres fuentes.
- Componentes de prompt: número, longitud, capas de inserción; mecanismo de composición atencional (query, claves, atención) y si requiere forward extra; penalización de ortogonalidad y su peso; expansión/particionado de componentes por tarea si existe.
- Tipo de inserción implementado en Mammoth (¿prefix-tuning?): evidencia; a comportamiento si difiere.
- CHECKPOINT del backbone: hay ambigüedad conocida en la literatura sobre qué checkpoint usó CODA-Prompt. NO la resuelvas: registra con exactitud qué identificador de pesos indica el paper (cita textual de la frase), cuál usa el repo oficial (fichero:línea) y cuál usa Mammoth. Cubo A, solo evidencia.
- Enmascarado de logits; manejo de la cabeza entre tareas; optimizador y LR; batch.
- n_epochs: ¿parámetro limpio de CLI/config en Mammoth para este método? ¿A qué se propaga (incluido el T del scheduler)? ¿Algo más codifica duración? Fila(s) EJE.
- Batch y soporte de acumulación de gradiente en Mammoth. Cubo A, solo evidencia.
- Augmentación/preprocesado, protocolo de evaluación, orden de clases y semilla en Mammoth. Cubo A, solo evidencia.

## Entregables (commit en el repo TFG)
auditoria/valores_coda.md · auditoria/comportamiento_coda.md · auditoria/cola_coda.md (con informe de embudo)
```

---

## PROMPT 4 — CONSOLIDACIÓN

```
# CONSOLIDACIÓN DE LA AUDITORÍA — tabla de entorno, fusión y veredictos

## Antes de nada
Lee auditoria/plantilla.md, auditoria/fuentes.md y los nueve ficheros de los tres métodos (valores_*, comportamiento_*, cola_*). Solo lectura del código; tus entregables son ficheros en auditoria/.

## Tarea
1. TABLA DE ENTORNO (auditoria/entorno.md): construye la tabla de cubo A + EJE juntando la evidencia común de los tres métodos. Una fila por parámetro (sin método), con la evidencia de qué usa cada fuente de cada método, y una columna de síntesis que señale explícitamente DÓNDE LOS TRES ORIGINALES NO COINCIDEN ENTRE SÍ (checkpoint, batch, augmentación, protocolo...). Valor propuesto y valor final: vacíos en todas (se deciden en Fase D). Presta atención especial al checkpoint: refleja las tres evidencias tal cual, sin resolver.
2. VEREDICTO DE LA DEPENDENCIA DURA (sección propia en entorno.md): por método, responde con evidencia: ¿n_epochs es un parámetro limpio de CLI/config en Mammoth?, ¿a qué se propaga?, ¿hay algo que codifique duración fuera de él? Concluye con un veredicto explícito: VIABLE / VIABLE_CON_CONDICIONES (cuáles) / PROBLEMA (cuál) para usar n_epochs como palanca del eje experimental en los tres métodos.
3. FUSIÓN: auditoria/comportamiento.md (une los tres, deduplica, conserva evidencias) y auditoria/cola_revision.md (une las tres colas; agrupa por tipo: CONFLICTO→PAPER con su grupo acoplado; GRUPOS_MIXTOS en sección propia — todo grupo cuyo valor propuesto mezcle fuentes, listando sus miembros y la fuente de cada uno; propuestas S ordenadas por severidad; CUBO_DUDOSO; NO_CONFIGURABLE; comportamiento relevante; una línea por ítem con referencia).
4. CONSISTENCIA: verifica que las tres tablas siguen la plantilla (12 columnas, nombres de caso, etiquetas). Si hay desviaciones de formato, corrígelas en los ficheros indicando el cambio; si hay contradicciones de contenido entre ficheros, NO las resuelvas: regístralas en cola_revision.md.
5. EMBUDO GLOBAL (al final de cola_revision.md): suma de los embudos de los tres métodos + lista agrupada global de defaults de infraestructura.

## Entregables (commit en el repo TFG)
auditoria/entorno.md · auditoria/comportamiento.md · auditoria/cola_revision.md (actualiza los valores_*.md solo si hubo correcciones de formato)
```
