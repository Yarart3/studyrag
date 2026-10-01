# Decision log

## 2026-10-01 — Python 3.12 en lugar de 3.13
**Contexto:** el PC ya tenía Python 3.13 instalado.
**Opciones:** 3.13 (la más reciente) / 3.12.
**Decisión:** 3.12, instalada junto a la 3.13 y elegida con `py -3.12`. Las librerías de IA (ChromaDB y sus dependencias) suelen tardar en dar soporte a las versiones nuevas de Python.
**Consecuencias:** `requires-python = ">=3.12"`, así que el proyecto también se podrá instalar con versiones posteriores cuando sean compatibles.

## 2026-10-01 — LLM: qwen2.5:3b en Windows, qwen2.5:7b en el futuro Mac
**Contexto:** GTX 1650 con 4 GB de VRAM. La 7b (4,7 GB) no cabe entera en la GPU.
**Opciones:** qwen2.5:7b / qwen2.5:3b.
**Medición** (`ollama run --verbose`, mismo prompt):

| Modelo | Prompt eval | Eval | Processor |
|---|---|---|---|
| qwen2.5:7b | 1,48 tok/s | 3,9 tok/s | 55 %/45 % CPU/GPU |
| qwen2.5:3b | 127,6 tok/s | 49 tok/s | 100 % GPU |

**Decisión:** qwen2.5:3b. En RAG lo que más pesa es la lectura del prompt (instrucciones + chunks); con la 7b, un prompt de unos 1.000 tokens tardaría minutos.
**Consecuencias:** algo menos de calidad (en la prueba confundió mutex con semáforo). El modelo irá en `.env` para cambiarlo sin tocar código. A revisar con la evaluación de la Fase 7.

## 2026-10-01 — Ventana de contexto: se mantiene en 4.096 tokens
**Contexto:** Ollama usa por defecto `num_ctx = 4096` y recorta el prompt sin avisar si lo supera.
**Estimación:** `ask` con k = 5 y chunks de 500 caracteres ≈ 1.400 tokens.
**Decisión:** mantener 4.096 de momento (subirlo consume VRAM). `num_ctx` será configurable en `.env`, y se registrará un aviso si `prompt_eval_count >= num_ctx`.
**Consecuencias:** `summarize` no cabrá con documentos largos y necesitará un enfoque map-reduce (Fase 6).