# Detección estilométrica de phishing generado por IA en comunicados institucionales de universidades públicas peruanas (2026)

Repositorio del experimento del artículo *Evaluación del aprendizaje automático estilométrico en la detección de phishing generado por IA en los comunicados y trámites institucionales de las universidades públicas peruanas, 2026*.

[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1EhwRcujARI0a4I0Ik8ZZoOZZGjZD04z8)

Copia del notebook alojada en este repositorio: [Abrir en Colab desde GitHub](https://colab.research.google.com/github/MrSack79/phishing-estilometria-ia/blob/main/notebooks/experimento.ipynb).

## Resumen

Se evalúa si un enfoque estilométrico detecta phishing redactado con modelos de lenguaje de gran escala (LLM) frente a comunicados institucionales reales. El corpus tiene 150 correos legítimos (@uni.pe, 39 remitentes), 150 correos de phishing generados con Qwen2.5 7B, Mistral 7B y Gemma2 9B, y 150 paráfrasis producidas con otro modelo. De cada texto se extraen 27 rasgos léxico-sintácticos y estilístico-persuasivos con spaCy. Se comparan regresión logística, Random Forest, LightGBM y XGBoost contra una línea base TF-IDF con regresión logística, con validación cruzada de cinco pliegues agrupada por remitente. La explicabilidad se analiza con SHAP y ganancia de información (Gain).

## Resultados principales

Validación cruzada agrupada por remitente, umbral 0.5, N = 300.

| Modelo | Recall | F1 macro | AUC-ROC | FPR |
|---|---|---|---|---|
| Regresión logística | 0.960 | 0.910 | 0.965 | 0.140 |
| Random Forest | 0.960 | 0.876 | 0.971 | 0.207 |
| LightGBM | 0.953 | 0.852 | 0.965 | 0.247 |
| XGBoost | 0.953 | 0.876 | 0.970 | 0.200 |
| TF-IDF + regresión logística (línea base) | 1.000 | 0.957 | 0.999 | 0.087 |

Los intervalos de confianza al 95 % y el resto de análisis están en `results/` y en el artículo.

## Estructura

```
data/raw/
  legitimos.csv
  phishing_ia.csv
  parafrasis_ia.csv
notebooks/
  phishing-estilometria-ia
results/
requirements.txt
```

- `data/raw/`: datos crudos del corpus.
- `notebooks/phishing-estilometria-ia`: experimento completo, desde la construcción del corpus hasta la explicabilidad.
- `results/`: tablas de métricas, brechas ΔF1, importancia de rasgos, ablación y figuras.
- `requirements.txt`: dependencias con versiones fijas.

## Reproducción

1. Abrir el notebook en Google Colab con el botón de arriba (o la copia de `notebooks/phishing-estilometria-ia` desde GitHub).
2. Ejecutar las celdas en orden. La primera celda clona este repositorio, instala `requirements.txt` y descarga el modelo `es_core_news_sm` de spaCy.
3. Los resultados se guardan en `results/` dentro de la sesión de Colab. La última celda los comprime en un archivo `.zip` para descargarlos.

El repositorio es público, por lo que no hace falta iniciar sesión en GitHub ni vincular cuentas en Colab: la cuenta de Google que se use en Colab puede ser distinta de la de GitHub.

Para ejecutar en local:

```bash
git clone https://github.com/MrSack79/phishing-estilometria-ia.git
cd phishing-estilometria-ia
pip install -r requirements.txt
python -m spacy download es_core_news_sm
jupyter notebook notebooks/phishing-estilometria-ia
```

La semilla aleatoria es 42 en todo el experimento.

## Datos y ética

- Los correos legítimos son comunicados institucionales recibidos por el autor. Se anonimizaron con expresiones regulares (nombres, correos, teléfonos, enlaces y similares) y se limitaron a 200 palabras.
- El phishing es sintético y se generó con fines exclusivamente de investigación en ciberseguridad defensiva. No contiene campañas reales ni datos de víctimas.
- Los datos se publican solo con fines académicos y de reproducibilidad. No deben usarse para reidentificar a personas ni para fines distintos de la investigación.

## Limitaciones

- Corpus pequeño, de una sola institución y con muestreo por conveniencia.
- Phishing sintético de tres LLM abiertos de tamaño mediano; no incluye phishing redactado por humanos.
- Umbral fijo en 0.5 sin calibración; la tasa de falsos positivos de todos los modelos supera el 5 %.
- La explicabilidad se calcula sobre LightGBM y el rasgo `saludo_formal` concentra gran parte de la señal.

## Licencia

El código se distribuye bajo licencia MIT (ver `LICENSE`). Los datos se rigen por la nota de la sección *Datos y ética*.

## Cómo citar

Rios Gonzales, J. O. A. (2026). *Detección estilométrica de phishing generado por IA en comunicados institucionales de universidades públicas peruanas* [Repositorio de código y datos]. GitHub. https://github.com/MrSack79/phishing-estilometria-ia
