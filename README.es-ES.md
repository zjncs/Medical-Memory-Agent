

# Uniq-Cluster Memory (UCM)

> Un sistema de gestión de memoria temporal consciente de conflictos para agentes de IA médicos

Uniq-Cluster Memory (UCM) es un sistema orientado a la investigación diseñado para la **construcción de memoria temporal consciente de conflictos** a partir de diálogos médicos largos y multironda. 

En las consultas médicas del mundo real, el estado de los pacientes (p. ej., medicamentos, síntomas, resultados de pruebas) se actualiza, refina o contradice con frecuencia con el tiempo. Las extensiones puras de la ventana de contexto o los enfoques estándar de Generación Aumentada por Recuperación (RAG) tienen dificultades para mantener una vista consistente y autorizada del estado del paciente. UCM aborda este problema extrayendo hechos médicos estructurados, agrupándolos en paquetes de información causal, contextualizando expresiones de tiempo relativas y detectando explícitamente conflictos de valores mediante un seguimiento bitemporal. El sistema genera en última instancia un conjunto de memorias únicas y canónicas, completas con su procedencia y historiales de conflictos.

---

## 🌟 Contribuciones Algorítmicas Clave

1. **Grafo de Conflictos Bitemporales**: 
A diferencia de las actualizaciones de memoria tradicionales de "el ganador se lo lleva todo", cada memoria en UCM lleva cuatro marcas de tiempo distintas (`t_event`, `t_ingest`, `t_valid_start`, `t_valid_end`). Introducimos un mecanismo de resolución de conflictos ponderado por confianza y multi-candidato que preserva el contexto histórico y resuelve incoherencias temporales.
   
2. **Agrupación de Información Desconfundida Causalmente**: 
Utilizamos un modelo causal estructural con ajuste backdoor para mitigar co-referencias espurias de eventos. Esto maneja eficazmente confundidores comunes en textos médicos, como superposición léxica, proximidad temporal y sesgo de identidad del hablante.
   
3. **Restricciones Formales del Dominio Médico**: 
UCM integra restricciones de lógica temporal basadas en reglas (p. ej., lógica de inicio/parada de medicamentos, orden diagnóstico diagnóstico de diagnóstico-prueba, monotonía de la dosis). Estas restricciones ajustan las puntuaciones de confianza de los candidatos de manera determinista, mejorando la precisión sin incurrir en costos adicionales de inferencia de LLM.

---

## 📊 Resultados Experimentales

UCM demuestra un rendimiento de vanguardia en el mantenimiento de memorias únicas y consistentes en comparación con líneas base sólidas. 

**Resultados principales en Med-LongMem v0.1 (n=20, dificultad alta)**

![Main Results](assets/ucm_main_results.png)

| System | Unique-F1 (Strict) | Unique-F1 (Relaxed) | Conflict-F1 |
|:---|:---:|:---:|:---:|
| **UCM (Ours)** | **0.8508** | **0.8585** | **0.9762** |
| Long-Context LLM | 0.3848 | 0.5273 | 0.8867 |
| Graphiti (Simulated) | 0.0627 | 0.5622 | 0.3450 |
| No Memory | 0.0000 | 0.5938 | 0.1292 |

**Estudio de Ablación**

![Ablation Study](assets/ucm_ablation.png)

| Variant | Unique-F1 (Strict) | Performance Drop (Δ) | Conflict-F1 |
|:---|:---:|:---:|:---:|
| **Full UCM Pipeline** | **0.8508** | — | **0.9762** |
| w/o Time Grounding | 0.1233 | -85.5% | 0.2583 |
| w/o M2 Clustering | 0.5680 | -33.2% | 0.0000 |
| w/o Causal Deconfounding | 0.7700 | -9.5% | 0.8533 |
| w/o Formal Constraints | 0.7900 | -7.1% | 0.9233 |
| w/o M4 Compression | 0.8349 | -1.9% | 0.9662 |

---

## ⚙️ Pipeline de Arquitectura del Sistema

![UCM System Architecture](assets/ucm_pipeline.png)

UCM opera a través de un pipeline de 5 etapas:

```text
Raw Dialogue 
  ↓
[M1] Event Extraction (LLM-based sliding window)
  ↓
[M2] Attribute Clustering (Semantic grouping)
  ↓
[M2.5] Information Bundle Construction & Causal Deconfounding
  ↓
[M3] Uniqueness Management (Time Grounding + Bi-Temporal Conflict Resolution + Formal Constraints)
  ↓
[M4] Memory Compression (Lightweight redundancy removal)
  ↓
[M5] Hybrid Retrieval (Structured + Semantic + Recency)
```
*Punto de entrada principal:* `src/uniq_cluster_memory/pipeline.py`

---

## 📂 Estructura del Repositorio

```text
uniq_cluster_memory/
├── baselines/              # Baseline implementations (Long-Context LLM, Graphiti, Hybrid RAG)
├── benchmarks/             # Dataset loaders (Med-LongMem, MedDialog, etc.)
├── evaluation/             # Evaluation metrics (Uniqueness, Conflict, Extraction, LLM-as-Judge)
├── experiments/            # Experiment execution scripts (Eval, Ablation)
├── scripts/                # Utility scripts (Dataset generation, Validation tools)
├── src/uniq_cluster_memory/ # Core source code (M1 to M5 modules, schema, pipeline)
└── tests/                  # Unit and regression test suite (88 tests)
```

---

## 🚀 Inicio Rápido

### 1. Instalación

Crea un entorno virtual e instala las dependencias:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 2. Variables de Entorno

Establece tu clave de API de LLM preferida. UCM es compatible con DashScope (Qwen) y puntos de conexión compatibles con OpenAI:

```bash
export DASHSCOPE_API_KEY="your-api-key-here"
# Alternativamente: export OPENAI_API_KEY="your-api-key-here"
```

### 3. Preparación de Conjuntos de Datos

UCM utiliza la suite de benchmarks `Med-LongMem`. Para generar el benchmark sintético completo (v1.0):

```bash
PYTHONPATH=. .venv/bin/python scripts/generate_med_longmem.py \
  --n_samples 200 \
  --difficulty mix \
  --seed 100 \
  --output_dir data/raw/med_longmem_v1
```

---

## 💻 Ejecución de Experimentos

### Evaluación del Pipeline Completo
Ejecuta la evaluación estándar del método UCM:

```bash
PYTHONPATH=. .venv/bin/python experiments/eval_our_method.py \
  --data_path data/raw/med_longmem \
  --output_path results/main_results/our_method_eval.json
```

### Estudios de Ablación
Ejecuta todas las variantes de abulación (`full`, `w/o_time`, `w/o_conflict`, `w/o_m4`, `w/o_m2`, `w/o_bitemporal`, `w/o_formal_constraints`, `w/o_causal_deconfound`):

```bash
PYTHONPATH=. .venv/bin/python experiments/run_ablation.py \
  --ablation all \
  --max_samples 20
```

### Comparaciones con Líneas Base
Evalúa las líneas base LLM de Contexto Largo y Graphiti:

```bash
# Long-Context LLM Baseline
PYTHONPATH=. .venv/bin/python baselines/long_context_llm.py \
  --data_path data/raw/med_longmem

# Graphiti (Simulated) Baseline
PYTHONPATH=. .venv/bin/python baselines/graphiti_baseline.py \
  --data_path data/raw/med_longmem
```

### Pruebas Automatizadas
Ejecuta la suite de pruebas completa para garantizar la integridad del sistema:

```bash
.venv/bin/python -m pytest -q
```
