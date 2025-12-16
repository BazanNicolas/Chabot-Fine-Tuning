# Chatbot personalizado mediante Fine-Tuning de modelos de lenguaje

## Objetivo

Utilizar **chats de Whatsapp** para entrenar un modelo de lenguaje mediante **Fine-Tuning**, con el objetivo de poder crear un **chatbot** que, de cierta forma, emule la personalidad del autor de los chats.

## Implementación

### 1. Preparación de datos

#### 1.1 Limpieza de datos

El primer paso es la **limpieza de datos**. <br>
Para ello tuvimos en cuenta la estructura que mantienen los chats al ser exportados, para así poder parsearlos adecuadamente. También en esta instancia se eliminaron mensajes irrelevantes como por ejemplo aquellos que contenían imágenes.

#### 1.2 Agrupación de mensajes

Luego el siguiente paso fue la **agrupación de mensajes**. Al comienzo los resultados demostraban que algo estaba en falta, fue cuando nos dimos cuenta que los modelos necesitaban contexto. <br>
El código agrupa de dos formas:

Aquellos **mensajes seguidos de un mismo autor** se convierten en un único mensaje que contiene todos los mensajes enviados dentro de un **límite horario** separado por un tag interno. <br>
Cuando uno textea, generalmente, envía múltiples mensajes que son congruentes entre sí, esta decisión soluciona un problema contextual que ayudará al modelo a generar respuestas donde pueda simular aquella natural forma de escribir, enviando un único mensaje que utiliza el separados como si estos fueran
múltiples mensajes enviados en una corta franja de tiempo.

Luego encontramos necesario darle algo de **contexto sobra la conversación** que se estaba manteniendo entre el autor y el otro usuario. Para ello agrupamos bloques de conversaciones según un límite horario y
de cantidad, luego se entrenaron los modelos con estos grupos de mensajes.

### 2. Elección de modelos

Para llevar a cabo el proyecto, se investigaron **distintos modelos de lenguaje** que según sus especificaciones parecían adecuarse a nuestros requisitos: entrenado para español, orientado a conversación y que no sea un modelo demasiado _grande_.

### 3. Configuración y Entrenamiento

Esta etapa fue de prueba y error, donde también tuvimos en cuenta la recomendación particular según el modelo. <br>
Inicialmente estuvimos limitados por el hardware y el progreso no era tan significativo, ya que tuvimos que elegir modelos pequeños y reducir la cantidad de datos y épocas. Una vez eso no representó una barrera, comprobamos una gran mejora.<br>

### 4. Comparación y pruebas

En las distintas notebooks, **se compararon el modelo base y el modelo fine-tuned**, en algunas utilizando promps sacadas de los mismos chats y otras con promps predefinidas. En los 3 modelos hubieron cambios significativos entre su base y fine-tune, pero no todos se acercaron al objetivo planteado. Este análisis se puede ver a profundidad en la sección _Conclusiones_. <br>

Para comparar la mejora entre el modelo base y el fine-tuned, nos basamos en la métrica de **Perplexity**.<br>
En criollo esta métrica evalúa entre cuántas _opciones_ está debatiendo el modelo para poder predecir la siguiente palabra. Entonces a menor Perplexity tendrá menor cantidad de opciones, por lo que estas serán más certeras.

## Comparación de modelos

En este repositorio se podran encontrar tres distintas implementaciones del chatbot, cuya mayor diferencia recae en los distintos modelos de lenguaje elegidos. <br>
En la siguiente tabla se compararan los tres modelos elegidos: [**GPT-2 spanish**](https://huggingface.co/DeepESP/gpt2-spanish), [**SmolLM3-3B**](https://huggingface.co/HuggingFaceTB/SmolLM3-3B), [**Phi3**](https://huggingface.co/microsoft/Phi-3-mini-4k-instruct). <br>

TODO

## ¿Cómo utilizar las notebooks?

Lo más interesante de este proyecto será comprobar si el modelo de lenguaje consigue emular nuestra personalidad virtual, la intención de este repositorio público es que toda persona pueda interactuar con su propio chatbot personalizado. <br>

Con tal de lograrlo, deberá seguir los siguientes pasos. <br>
Además dentro de las notebooks hemos dejado comentarios y guías para que sepa qué está ejecutando en cada celda. <br>

### 1. Conseguir los chats de WhatsApp

Las conversaciones que mantenemos por WhatsApp no nos pertenecen únicamente a nosotros, por lo que antes de proseguir, es importante que consiga el consentimiento de toda persona cuyo chat desea utilizar para entrenar el modelo. <br>

Ahora, con el consentimiento dado: <br>

- Dentro del chat con la persona deberá seleccionar:
  los tres puntitos-> Más -> Exportar chat
- Esto generará un archivo .txt.
- Repita el proceso para conseguir una cantidad adecuada de ejemplos.
- Por último junte todos los chats, uno tras otro, en un archivo .txt, por ejemplo _chats.txt_.

Como guía, el archivo _chats.txt_ que utilizamos para el entrenamiento contaba con un total de 153700 líneas, lo que generaba TODO conversaciones y luego del filtrado quedaron TODO conversaciones útiles.

### 2. Asegurar un entorno de ejecución

Entrenar modelos de lenguaje no es tarea trivial para nuestro hardware, necesitaremos de una GPU que resista el entrenamiento y un entorno que nos permita utilizarla ininterrumpidamente por un tiempo que puede variar entre minutos u horas, dependiendo la cantidad de conversaciones de los datos o las especificaciones de la GPU. Por ejemplo nosotros empezamos utilizando [Google Colab](https://colab.google/), pero para la cantidad de chats y en modelos más grandes que GPT-2, las 15GB de la GPU Nvidia T4 del plan gratuito no fueron suficientes. Si usted desea obtener resultados similares, recomendamos ampliamente clonar este repositorio en un entorno que soporte esta clase de exigencias.

### 3. Carga de datos

En las notebooks encontrará una celda interactiva, que le permitirá cargar su archivo txt y especificar el nombre del autor, es escencial que este coincida tal cual aparece en el archivo de chats. <br>

En las siguientes celdas se filtrarán los datos y acomodarán según el modelo, generandose así un archivo que será el utilizado para el entrenamiento. <br>

En este punto recomendamos reiniciar el kernel, tendrá que ejecutar menos líneas y podrá continuar al siguiente paso. Esto para reducir la memoria de la GPU y prepararla para la siguiente etapa. <br>

### 4. Entrenamiento

Dependiendo del modelo que haya elegido, su hardware disponible y la cantidad de conversaciones que se hayan conservado de sus chats; esto tomará mayor o menor tiempo, pero a comparación con el resto, esta es la etapa que tomará más tiempo. <br>

Una vez terminado el entrenamiento, el modelo será guardado. Si cuenta con un entorno de ejecución como Google Colab que elimina los archivos una vez desconectado del Kernel, no olvide de descargar el modelo entrenado.

### 5. Comprobación, Chatbot y Métricas

Las siguientes celdas cuentan con una **comprobación** para aquellos que quieran ver ejemplos de respuesta entre el modelo base y el fine-tuneado. <br>
Seguido por el **chatbot** interactivo, en el cuál podrá enviarle el mensaje (prompt) que desee y obtendrá la respuesta del modelo fine-tuned. <br>
Por último, si desea visualizarlo, se encuentran **evaluaciones** donde se compara la Perplexity entre el modelo base y el fine-tuned. <br>

## Conclusión

TODO

## Tecnologías utilizadas

- Python 3.13.11
- JupyterHub
- Tarjeta gráfica A30
- [Centro de Computación de Alto Desempeño (CCAD)](https://supercomputo.unc.edu.ar/ccad/)

## Agradecimientos

Este trabajo utilizó recursos computacionales de [**UNC Supercómputo (CCAD) de la Universidad Nacional de Córdoba**](https://supercomputo.unc.edu.ar), que forman parte del Sistema Nacional de Computación de Alto Desempeño (SNCAD) de la República **Argentina**. <br>

## Créditos

Proyecto realizado para la materia Minería de Texto, de la Facultad de Astronomía, Matemática y Física (FAMAF) de la Universidad Nacional de Córdoba (UNC), bajo la supervisión de la profesora Laura Alonso Alemany. <br>

Integrantes:

- [Nicolás Bazan](https://github.com/BazanNicolas)
- [Melina Rocío Morales](https://github.com/Paaprikaa)

## Referencias

1. IBM. (s.f.). _What is fine-tuning?_. Recuperado de https://www.ibm.com/es-es/topics/fine-tuning
2. Hugging Face. (s.f.). _Training and fine-tuning_. En _Hugging Face Documentation_. Recuperado de https://huggingface.co/docs/transformers/es/training
3. Comet November 21, 2024. _Perplexity for LLM Evaluation_. Recuperado de https://www.comet.com/site/blog/perplexity-for-llm-evaluation/
4. UNC Supercómputo Wiki (s.f.). Recuperado de https://wiki.ccad.unc.edu.ar/
