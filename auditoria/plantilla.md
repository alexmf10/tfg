# Plantilla normativa de la auditoría

Este fichero es la autoridad de formato y decisión para las auditorías de L2P, DualPrompt y CODA-Prompt. Se aplica durante la Fase A y en las revisiones humanas posteriores.

## A) Columnas de la tabla

Las tablas tendrán exactamente 12 columnas, en este orden:

| parámetro | ámbito (dataset: CIFAR-100 / ImageNet-R / ambos) | cubo (A / B / EJE) | paper (valor + evidencia) | repo oficial (valor + evidencia) | Mammoth (valor + evidencia + cadena de resolución) | caso (nombre) | grupo | valor propuesto | valor final | flags | observaciones |
|---|---|---|---|---|---|---|---|---|---|---|---|
|  |  |  |  |  |  |  |  |  |  |  |  |

- La evidencia NO es una columna aparte: vive dentro de la celda de cada fuente, para que nunca haya duda de a qué fuente pertenece cada prueba.
- `valor propuesto`: lo rellena la cascada, solo para el cubo B.
- `valor final`: queda SIEMPRE VACÍO en la Fase A, en todos los cubos. Lo rellena la revisión humana en las Fases B–D. En cubo B será idéntico al propuesto salvo decisión registrada; en cubo A se decide en la Fase D; en EJE no aplica, porque lo define la escalera de presupuestos. Codex NUNCA escribe en `valor final`.
- `observaciones`: contiene una nota corta. Si no cabe en una línea (cadena de evidencia larga, razonamiento del caso o historia de un grupo), la celda contiene un código `OBS-nn` y el contenido completo va en la sección `Notas` al final del mismo fichero, con una entrada por código.
- Toda fila cuyo `valor final` acabe difiriendo de `valor propuesto` documentará su flujo de decisión en su nota `OBS`.

## B) Cubos

### CUBO A

Entorno compartido; una fila, sin método; SOLO evidencia de qué usa cada fuente y `valor propuesto` vacío:

- backbone y checkpoint EXACTO (identificador del fichero de pesos / nombre timm);
- datasets y particiones;
- orden de clases y semillas (cómo lo fija cada fuente);
- augmentación y preprocesado;
- tamaño de batch;
- protocolo de evaluación (task-id, cabeza, métricas);
- buffer/memoria.

### EJE

Solo evidencia y `valor propuesto` vacío:

- `n_epochs` / iteraciones;
- todo lo ligado a la duración del entrenamiento: schedulers, warmup, early stopping y cálculos derivados de la duración.

### CUBO B

Receta del método; fila por método. Contiene todo lo demás y se le aplica la cascada.

### Cubo dudoso

Asignar provisionalmente un cubo y marcar `CUBO_DUDOSO`.

## C) Cascada de decisión

Se aplica solo al cubo B, fila por fila y siempre con el nombre completo del caso:

1. `CASO 1 COINCIDEN`: paper y repo oficial especifican y coinciden → ese valor.
2. `CASO 2 EXPLICADO`: paper y repo oficial especifican y discrepan CON explicación de los autores (tag de la época, changelog, issue o errata) → decide la explicación, citada.
3. `CASO 3 CONFLICTO→PAPER`: paper y repo oficial especifican y discrepan SIN explicación → valor del paper por convención declarada. Marcar `REVISAR`.
4. `CASO 4 FUENTE_ÚNICA`: solo una fuente especifica → esa fuente. Etiquetar `SOLO_PAPER` o `SOLO_REPO`.
5. `CASO 5 SIN_FUENTE`: ninguna especifica → default efectivo de Mammoth. Etiquetar `DEFAULT_MAMMOTH`.

## D) Grupos acoplados

La columna `grupo` identifica parámetros que los autores ajustaron juntos o que dependen entre sí. Se usarán nombres tipo:

- `G-OPT`: learning rate, scheduler, batch y duración;
- `G-PROMPT`: longitud, pool, top-k y capas de inserción;
- otros grupos solo cuando se identifiquen con justificación localizable.

Reglas obligatorias:

- Si una fila de un grupo cae en `CASO 3 CONFLICTO→PAPER`, todos los miembros del grupo que también estén en conflicto se resuelven desde el paper EN BLOQUE.
- Si el `valor propuesto` de un grupo acaba mezclando fuentes (unos miembros del paper y otros del código), TODAS sus filas reciben la marca `GRUPO_MIXTO` y el grupo entra en la cola de revisión. Codex no resuelve esa mezcla.
- Los acoplamientos internos de Mammoth, donde un ajuste cambia el comportamiento de otro, también se anotan.

## E) Flag S (sensibilidad)

El flag `S` es transversal y compatible con cualquier caso. Se propone cuando se cumple al menos uno de estos criterios:

1. hay indicios de alto impacto;
2. los valores en conflicto están muy alejados;
3. el valor elegido NO es configurable en Mammoth.

No hay tope de propuestas: se registran TODAS las que cumplan los criterios. Cada propuesta incluye 1-2 líneas de justificación, evidencia y severidad `ALTA`, `MEDIA` o `BAJA`. Se presentan ordenadas por severidad. No se infla la lista con dudas genéricas: cada `S` debe apoyarse en evidencia concreta. Ejecutar experimentos de sensibilidad NO forma parte de la auditoría.

Formato de la cola de sensibilidad:

| severidad | parámetro / grupo | método y ámbito | justificación (1-2 líneas) | evidencia |
|---|---|---|---|---|
|  |  |  |  |  |

## F) Criterio de fila

Un parámetro tiene fila si y solo si cumple las dos condiciones siguientes:

1. aparece en el setup experimental o en los apéndices del paper, O es configuración del experimento en el repo oficial, O es configurable en Mammoth y aplica a estos tres modelos, sus datasets o el bucle de entrenamiento/evaluación; Y ADEMÁS
2. puede afectar plausiblemente a resultados o a coste.

Lo que solo afecta a infraestructura (`logging`, rutas, `num_workers`, `device`, guardado de checkpoints) va a una LISTA AGRUPADA al final del fichero, sin fila ni evidencia línea a línea. En caso de duda: crear fila y marcar `LOW`. No hay número objetivo de filas.

### Lista agrupada de infraestructura

- Pendiente de completar durante la auditoría.

## G) Informe de embudo

Al final de cada tabla se incluye obligatoriamente:

- número de parámetros examinados;
- número de parámetros que obtuvieron fila;
- número de parámetros enviados a la lista agrupada;
- motivos de exclusión, agrupados por categoría.

Formato:

| etapa | cantidad | desglose / motivo |
|---|---:|---|
| parámetros examinados |  |  |
| parámetros con fila |  |  |
| lista agrupada |  |  |
| exclusiones |  |  |

## H) Formato de evidencia

El formato es obligatorio dentro de cada celda de fuente:

- Paper: sección / tabla / apéndice localizada en la fuente LaTeX de `papers/`; página del PDF si es posible.
- Repo oficial: SHA + fichero + línea(s).
- Mammoth: SHA + fichero + línea(s) + cadena de resolución completa (`default global → config del modelo → config del dataset → CLI`), indicando qué nivel fija el valor efectivo.

Etiquetas de ausencia, con significado estricto:

- `NO_ENCONTRADO`: se buscó y no se halló.
- `NO_APLICA`: el concepto no existe para ese método o fuente.
- `NO_DISPONIBLE`: la fuente está ausente.
- `NO_CONFIGURABLE`: existe en Mammoth pero está hardcodeado.

Valores y comportamientos especiales:

- Valores derivados: `DERIVADO` + fórmula.
- Comportamientos no determinables leyendo: `NO_DETERMINABLE_ESTATICO` + propuesta de script mínimo de comprobación.

No se sustituye una etiqueta por una inferencia silenciosa.

## I) Discrepancias de comportamiento

Las diferencias de conducta, no de valor, se documentan en `comportamiento_{método}.md`. Incluyen, entre otras: tipo de prompting (`prefix` frente a `prompt-tuning`), cálculo de la query, enmascarado de logits en entrenamiento, manejo de la cabeza entre tareas, inicializaciones y trucos.

Cada entrada contiene:

1. descripción breve;
2. evidencia en ambos lados;
3. si es configurable en Mammoth;
4. propuesta de flag `S`, si procede.

Estas discrepancias solo se documentan; nunca se corrigen durante la auditoría.

## Notas

Las entradas se numeran `OBS-01`, `OBS-02`, etc. Cada código aparece una sola vez y debe poder rastrearse hasta su fila.

- Pendiente de completar durante la auditoría.
