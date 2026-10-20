# phishing-estilometria-ia
Detección estilométrica de phishing generado por IA (universidades públicas peruanas, 2026)
# Detección estilométrica de phishing generado por IA en comunicados institucionales de universidades públicas peruanas, 2026


[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/MrSack79/phishing-estilometria-ia/blob/main/notebooks/experimento.ipynb)

## Cómo reproducir
1. Abre el cuaderno con el botón de Colab (no requiere GPU).
2. Ejecuta las celdas en orden. El cuaderno clona este repositorio, normaliza los datos, extrae los rasgos, entrena los modelos y genera tablas y gráficos.

## Datos (data/raw)
- legitimos.csv: comunicados y avisos institucionales reales, anonimizados (columnas: text, group; group es un hash del remitente).
- phishing_ia.csv: correos de phishing simulados, generados con qwen2.5:7b, mistral:7b y gemma2:9b mediante Ollama (columnas: text, generator, scenario, tono, largo).
- parafrasis_ia.csv: paráfrasis de los correos de phishing, cada una hecha por un modelo distinto del generador original.

Los modelos de lenguaje no son deterministas: los CSV publicados son los datos fijos del experimento. Todo lo posterior (normalización, emparejamiento por longitud, rasgos, modelos y evaluación) es reproducible con la semilla 42.

## Decisiones metodológicas
- Los correos legítimos se recortaron a 200 palabras y se limitó a 40 o 50 correos por remitente.
- Se normalizaron ambas clases para eliminar artefactos de formato (etiqueta "Asunto:", emojis, URL y correos).
- Se emparejó la longitud entre clases (rangos de palabras) y los modelos no usan variables de tamaño.

## Limitaciones
- El corpus compara comunicados legítimos reales con phishing generado por IA; no incluye phishing escrito por humanos.
