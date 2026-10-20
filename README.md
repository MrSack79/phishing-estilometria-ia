# Detección estilométrica de correos de phishing generados por IA

Evaluación del aprendizaje automático estilométrico en la detección de phishing generado por IA en los comunicados y trámites institucionales de las universidades públicas peruanas, 2026.

## Resumen

Se compara un conjunto de 27 rasgos estilométricos (diversidad léxica, variación de la longitud de las oraciones, marcadores formales y persuasivos) frente a una línea base léxica (TF-IDF + regresión logística) para distinguir correos institucionales legítimos de correos de phishing generados por tres modelos de lenguaje. Se evalúan el rendimiento global, la generalización a un generador y a escenarios no vistos, la robustez ante paráfrasis y correos breves, y la importancia de los rasgos con SHAP y Gain.

## Estructura del repositorio

```
.
├── data/
│   ├── raw/
│   │   ├── legitimos.csv
│   │   ├── phishing_ia.csv
│   │   └── parafrasis_ia.csv
│   └── processed/
├── notebooks/
│   └── experimento.ipynb
├── results/
├── requirements.txt
└── README.md
```

## Datos

| Archivo | Contenido |
|---|---|
| `legitimos.csv` | Correos institucionales reales (`@uni.pe`): comunicados y anuncios recibidos por el autor. Columnas: `text`, `group` (remitente anonimizado). |
| `phishing_ia.csv` | Correos de phishing generados con IA (261 antes del emparejamiento). Columnas: `text`, `generator`, `scenario`, `tono`, `largo`. |
| `parafrasis_ia.csv` | 150 paráfrasis del phishing, hechas por un modelo distinto al que lo generó. Columnas: `parent_text`, `text`, `generator_origen`, `paraphraser`. |

Origen y tratamiento:

- **Legítimos.** Provienen del buzón institucional del autor (exportación Takeout, `.mbox`). Se anonimizaron con expresiones regulares (correos, teléfonos, nombres, fechas y códigos), se limitó el número de correos por remitente y se recortaron a 200 palabras. El corpus final tiene 159 correos de 39 remitentes (150 tras el emparejamiento).
- **Phishing sintético.** Generado con Ollama en Google Colab (GPU T4) con `qwen2.5:7b`, `mistral:7b` y `gemma2:9b`. Se usaron 10 escenarios, 5 tonos y varias longitudes, con un prompt que declara el propósito de construir un conjunto de datos de ciberseguridad. Los modelos `llama3.1` (rechazó la tarea) y `deepseek-r1:7b` (calidad insuficiente en español) se descartaron. Los textos se incluyen tal como se generaron; el cuaderno no vuelve a generarlos.
- **Preprocesamiento aplicado a ambas clases** (función `normalizar`): se eliminan el encabezado «Asunto:», emojis, marcas de formato Markdown, URLs y correos, y se sustituye el nombre de la universidad por `[UNIVERSIDAD]`, para que el clasificador no aprenda atajos superficiales.
- **Emparejamiento por longitud.** Se equilibraron las dos clases por rangos de longitud (0-40, 40-70, 70-100, 100-130, 130-170 y 170+ palabras). Resultado: 150 correos legítimos y 150 de phishing.

Los correos legítimos son reales pero están anonimizados. No contienen direcciones, teléfonos, DNI, códigos de alumno ni URLs. Se usan solo con fines académicos.

## Método

- **Rasgos.** 27 rasgos estilométricos extraídos con spaCy (`es_core_news_sm`): diversidad léxica (TTR, K de Yule, entropía), longitud y variación de oraciones, proporciones gramaticales, puntuación, marcadores formales (por ejemplo, saludo formal), indicadores de urgencia y llamadas a la acción. Se excluyen los rasgos de tamaño (`n_palabras`, `n_oraciones`, `n_saltos_linea`).
- **Modelos.** Regresión logística, Random Forest, LightGBM y XGBoost. Líneas base: TF-IDF + regresión logística y palabras funcionales.
- **Validación.** `StratifiedGroupKFold` de 5 pliegues agrupado por remitente, semilla 42. Intervalos de confianza de Clopper-Pearson (recall y FPR) y bootstrap con 1 000 réplicas (F1 y AUC).
- **Análisis.** Generador fuera del entrenamiento (ΔF1 con y sin el generador, sobre el mismo conjunto de prueba), escenarios no vistos (leave-scenario-out), correos breves (menos de 60 palabras), paráfrasis, sensibilidad sin el remitente más frecuente, curvas ROC y TPR a FPR fijo, y ablación del rasgo `saludo_formal`.
- **Explicabilidad.** SHAP (TreeExplainer sobre LightGBM) y Gain, ambos normalizados. Nivel Bajo si es menor que 1/p, Medio entre 1/p y 3/p y Alto si es mayor que 3/p, con p = 27.

## Cómo reproducir

1. Abre `notebooks/experimento.ipynb` en Google Colab (File → Open notebook → GitHub).
2. Ejecuta todas las celdas en orden (Runtime → Run all). No requiere GPU.

La primera celda clona el repositorio e instala las dependencias:

```python
!git clone https://github.com/MrSack79/phishing-estilometria-ia.git /content/phishing-estilometria-ia
!pip install -q -r /content/phishing-estilometria-ia/requirements.txt
!python -m spacy download es_core_news_sm
```

Fuera de Colab:

```bash
git clone https://github.com/MrSack79/phishing-estilometria-ia.git
cd phishing-estilometria-ia
pip install -r requirements.txt
python -m spacy download es_core_news_sm
```

Versiones usadas: Python de Colab, numpy 2.1.3, pandas 2.2.3, scipy 1.16.3, scikit-learn 1.6.1, matplotlib 3.10.0, tqdm 4.67.3, spacy 3.8.16, lightgbm 4.6.0, xgboost 3.4.1, shap 0.54.1 y `es_core_news_sm` 3.8.0. La semilla es 42. Con otras versiones los resultados pueden variar ligeramente.

## Resultados (`results/`)

| Archivo | Contenido |
|---|---|
| `tabla_resultados.csv` | Métricas globales de todos los modelos |
| `delta_f1.csv` | Caída de F1 al excluir un generador del entrenamiento |
| `shap_importancia.csv`, `shap_summary.png`, `shap_barras.png` | Importancia de rasgos con SHAP |
| `importancia_relativa.csv` | SHAP y Gain relativos con su nivel (Bajo, Medio, Alto) |
| `roc.png` | Curvas ROC |
| `ablacion_saludo_formal.csv` | Comparación con y sin `saludo_formal` |

Hallazgos principales:

- Existe señal estilométrica: AUC entre 0.95 y 0.97 según el modelo.
- Los rasgos más importantes son `saludo_formal`, `burstiness_oracion` y `long_media_oracion`, que concentran cerca del 60 % de la importancia SHAP.
- Sin `saludo_formal` el AUC se mantiene entre 0.945 y 0.959, pero el FPR aumenta en tres de los cuatro modelos.
- Las metas planteadas inicialmente (F1 > 0.95 y FPR < 1.5 %) no se alcanzan, y la línea base léxica rinde más que los rasgos estilométricos.

## Limitaciones

- Corpus pequeño (150 correos por clase). Con 150 negativos, aun con cero falsos positivos el límite superior del IC de Clopper-Pearson del FPR es cercano al 2.4 %, por lo que la meta de FPR < 1.5 % no es verificable.
- Los correos legítimos provienen de un solo buzón y de 39 remitentes; un remitente concentra gran parte de los falsos positivos y podría redactar con apoyo de IA.
- El phishing es sintético y proviene de tres generadores de código abierto; no incluye phishing real ni modelos comerciales.
- Parte del rendimiento se debe a convenciones de apertura de los generadores (por ejemplo, el saludo formal).
- Los escenarios se comparten entre generadores, lo que favorece a la línea base léxica.

