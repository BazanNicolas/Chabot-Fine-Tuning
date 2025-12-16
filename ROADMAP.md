# Roadmap: Chatbot personalizado mediante Fine-Tuning de modelos de lenguaje

## Resumen

Este proyecto tiene como objetivo desarrollar un chatbot personalizado que imite el estilo conversacional de una persona a través de técnicas de inteligencia artificial. La solución se basará en el fine-tuning de un modelo de lenguaje preentrenado para generar respuestas automáticas adaptadas a un estilo conversacional específico. Para ello, se utilizarán datos reales proporcionados por los integrantes del proyecto, ajustando el comportamiento del sistema para replicar el tono, estilo y patrones comunicacionales deseados. El proyecto incluirá también la limpieza y preparación de los datos, el entrenamiento del modelo, y la evaluación de su rendimiento. El sistema resultante será capaz de ofrecer respuestas coherentes y personalizadas en entornos que requieran interacción automática.

## Hipótesis a evaluar

1. **Fine-tuning mejora el rendimiento:** Se espera que el modelo elegido, que ha pasado por el proceso de fine-tuning para ajustarse a un conjunto de datos personalizado, genere respuestas más relevantes, naturales y similares al estilo conversacional del usuario que un modelo preentrenado sin ajustes.
2. **Evaluación de técnicas de fine-tuning:** Existen diversas formas de realizar fine-tuning (p. ej., diferentes modelos base, variaciones en los hiperparámetros y subconjuntos del dataset). Se explorarán estas variantes para identificar la más efectiva.
3. **Adaptación regional del lenguaje:** Realizar un fine-tuning para ajustar el modelo al español de Argentina y luego a los datos conversacionales del usuario será beneficioso en la mejora del rendimiento del chatbot.

## Objetivos preliminares

1. **Recopilación y Preprocesamiento de Datos:** Obtener datos conversacionales de la plataforma _WhatsApp_ y consentimiento de las personas que provean su información personal y estructurarlos adecuadamente para su uso en el proceso de entrenamiento del modelo. Este paso incluye limpieza de elementos y mensajes innecesarios, la preparación de un dataset adecuado aplicando técnicas de normalización para estandarizar el texto, tokenización y segmentación de diálogos.
2. **Delimitación del alcance del chatbot:** Definir las interacciones que el chatbot puede realizar y aquellas que no debe ejecutar. Se debe asegurar que el chatbot no genere respuestas que comprometan la privacidad o proporcionen información confidencial del entrenamiento.
3. **Elección del modelo de lenguaje:** Determinar qué modelo se utilizará como base para realizar el fine-tuning, evaluando factores como el tamaño del modelo, su capacidad de generar texto coherente en español, y su compatibilidad con los datos disponibles.
4. **Fine-Tuning del Modelo:** Aplicar el fine-tuning al modelo preentrenado utilizando los datos conversacionales proporcionados. Esto incluye ajustes para adaptar el estilo conversacional, así como el lenguaje específico de la región.
5. **Evaluación del Desempeño y Optimización:** Medir la calidad de las respuestas generadas utilizando métricas de entropía para determinar el nivel de coherencia, naturalidad y similitud con el estilo conversacional original. Realizar ajustes en el modelo en función de los resultados obtenidos en las evaluaciones. Esto incluye optimizar el proceso de fine-tuning y ajustar los hiperparámetros para mejorar la precisión y relevancia de las respuestas generadas.

## Técnicas relevantes a aplicar

1. **Fine-Tuning:** Esta técnica permite adaptar un modelo generativo preentrenado a un conjunto de datos específico, lo que reduce los costos de computación, la huella de carbono y permite utilizar modelos de última generación sin tener que entrenar uno desde cero.
2. **Procesamiento de Lenguaje Natural (NLP):** Las técnicas de NLP serán fundamentales para limpiar y estructurar los datos de conversación, lo que incluye la eliminación de elementos no textuales, la normalización de datos, y el análisis semántico para permitir que el modelo genere respuestas contextualmente adecuadas.

3. **Evaluación con Métricas de Calidad Conversacional:** En la evaluación del chatbot se utilizarán métricas basadas en **entropía** para medir la diversidad léxica, asegurando que el modelo no sea repetitivo ni incoherente. Además, la **perplejidad** complementará este análisis al medir qué tan bien el modelo predice secuencias de palabras coherentes. Junto con estas métricas automáticas, se realizarán **evaluaciones humanas** para juzgar la coherencia y naturalidad de las respuestas desde la perspectiva del usuario.

## Planificación

1.  **Recolección y Preprocesamiento de Datos (Semana 1-2):**

    - Recopilar datos conversacionales.
    - Limitación de datos sensibles para proteger la privacidad.
    - Limpieza y formateo de los datos (eliminación de emojis y archivos multimedia, nombres, duplicados, y mensajes irrelevantes).
    - Aplicación de técnicas de normalización, tokenización y segmentación de diálogos para preparar los datos para el entrenamiento del modelo.

2.  **Fine-Tuning del Modelo (Semana 3-4):**

    - Selección de un modelo preentrenado.
    - Entrenamiento del modelo utilizando los datos preprocesados, ajustando los hiperparámetros y explorando diferentes técnicas.
    - Monitorización del proceso de entrenamiento.

3.  **Evaluación y Optimización (Semana 5-6):**

    - Evaluar la calidad de las respuestas generadas utilizando métricas basadas en entropía (entropía de palabras, entropía condicional) y perplejidad para medir la diversidad y coherencia.
    - Realizar evaluaciones humanas para analizar la coherencia, naturalidad y similitud con el estilo conversacional original del usuario.
    - Ajustar los hiperparámetros y realizar iteraciones de mejora según los resultados obtenidos en las evaluaciones.

4.  **Documentación y presentación (Semana 7-8):**

    - Redacción del informe final con un análisis de los resultados, incluyendo la efectividad de las técnicas empleadas, el rendimiento del chatbot y los desafíos encontrados.
    - Propuestas de mejoras futuras basadas en los resultados del proyecto y las evaluaciones de usuarios.

## Referencias

1. IBM. (s.f.). _What is fine-tuning?_. Recuperado de [https://www.ibm.com/es-es/topics/fine-tuning](https://www.ibm.com/es-es/topics/fine-tuning)
2. Hugging Face. (s.f.). _Training and fine-tuning_. En _Hugging Face Documentation_. Recuperado de [https://huggingface.co/docs/transformers/es/training](https://huggingface.co/docs/transformers/es/training)
