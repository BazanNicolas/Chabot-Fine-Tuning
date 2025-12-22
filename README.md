# Chatbot personalizado mediante Fine-Tuning de modelos de lenguaje

## Objetivo

Utilizar **chats de Whatsapp** para entrenar un modelo de lenguaje mediante **Fine-Tuning** de LLMs, con el objetivo de poder crear un **chatbot** que, de cierta forma, emule la personalidad del autor de los chats.

## Implementación

### 1. Preparación de datos

#### 1.1 Limpieza de datos

El primer paso es la **limpieza de datos**. <br>
Para ello tuvimos en cuenta la estructura que mantienen los chats al ser exportados, para así poder parsearlos adecuadamente. También en esta instancia se eliminaron mensajes irrelevantes como por ejemplo aquellos que contenían imágenes. <br>

Para los modelos _Phi-3_ y _GPT2-spanish_ se filtró además ciertas conversaciones por similitud, es decir si la conversación parecía "incoherente" se la descartaba. En cuanto a _GPT2_ el descarte era necesario, ya que al ser un modelo con menor capacidad aquel _ruido_ podría confudirlo más que beneficiarlo. Luego se optó por contrastar _Phi-3_ y _SmolLM3-3B_, aunque ninguno de estos modelos necesita el filtrado por similitud, se quiso comprobar si al no filtrar aquellas conversaciones caóticas y ruidosas, se puede llegar a potenciar un diálogo con mayor naturalidad. <br>

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

Al momento de fine-tunear los modelos, tanto para _Phi-3_ y _SmolLM3-3B_ se utilizó **SFT** (Supervised Fine-Tuning), para poder especificarle al modelo la respuesta esperada (mensaje del autor) dada cierta entrada (mensaje de otro usuario). <br>
Mientras que para _GPT2-spanish_ se aplicó fine-tuning de tipo **CLM** (Casual Language Modeling), esta decisión se debe a que _GPT2_ fue creado específicamente para ser entrenado bajo CLM.

El entrenamiento de los modelos _Phi-3_ y _SmolLM3-3B_ se optimizó mediante **QLoRA** (Quantized Low-Rank Adaptation), una técnica que combina **cuantización** y **adaptación de bajo rango** para reducir significativamente el uso de memoria GPU durante el fine-tuning. <br>
En este enfoque, el modelo base se carga en formato cuantizado de **4 bits**, lo que disminuye su tamaño en memoria sin modificar sus pesos originales. Sobre este modelo congelado se agregan capas **LoRA**, que consisten en matrices de bajo rango entrenables encargadas de ajustar el comportamiento del modelo al caso de uso específico.

Como describe Cloudflare en [_¿Qué es la adaptación de bajo rango (LoRA)?_](https://www.cloudflare.com/es-es/learning/ai/what-is-lora/):  
"LoRA congela las ponderaciones y los parámetros del modelo tal como están. Luego, sobre este modelo original, agrega una adición ligera llamada matriz de rango bajo, que luego se aplica a nuevas entradas para obtener resultados específicos para el contexto. La matriz de rango bajo se ajusta a las ponderaciones del modelo original para que los resultados coincidan con el caso de uso deseado". <br>

Esta combinación permite realizar fine-tuning supervisado de modelos grandes de forma eficiente, manteniendo un buen rendimiento y reduciendo considerablemente los requisitos computacionales.  
En el caso de _GPT2-spanish_, no se consideró necesario aplicar QLoRA debido a que se trata de un modelo relativamente pequeño, cuyo entrenamiento puede realizarse directamente sin restricciones significativas de memoria.



### 4. Comparación y pruebas

En las distintas notebooks, **se compararon el modelo base y el modelo fine-tuned**, en algunas utilizando promps sacadas de los mismos chats y otras con promps predefinidas. En los 3 modelos hubieron cambios significativos entre su versión base y fine-tuned, pero no todos se acercaron al objetivo planteado. <br>
Este análisis se puede ver a profundidad en la sección _Conclusiones_. <br>

## Comparación de modelos

En este repositorio se podran encontrar tres distintas implementaciones del chatbot, cuya mayor diferencia recae en los distintos modelos de lenguaje elegidos. <br>
En la siguiente tabla se comparan datos técnicos y elecciones de diseño que se tomaron de los tres modelos elegidos: [**GPT2-spanish**](https://huggingface.co/DeepESP/gpt2-spanish), [**SmolLM3-3B**](https://huggingface.co/HuggingFaceTB/SmolLM3-3B), [**Phi-3**](https://huggingface.co/microsoft/Phi-3-mini-4k-instruct).

| Característica                  |                  SmolLM3-3B                  |                             Phi-3                             |                   GPT2-spanish                    |
| ------------------------------- | :------------------------------------------: | :-----------------------------------------------------------: | :-----------------------------------------------: |
| Cantidad de parámetros          |                      3B                      |                             3.8B                              |                       124M                        |
| Fine-Tuning aplicado            |         SFT (Supervised Fine-Tuning)         |                 SFT (Supervised Fine-Tuning)                  |          CLM (Causal Language Modeling)           |
| QLoRA                           |                      ✅                      |                              ✅                               |                        ❌                         |
| Filtrado de datos por similitud |                      ❌                      |                              ✅                               |                        ✅                         |
| Tags                            |     <\|im_start\|>role ... <\|im_end\|>      | <\|system\|>...<\|end\|><\|user\|>...<\|end\|><\|assistant\|> | <\|talk\|><\|ax1\|>...<\|ax2\|>...<\|endoftext\|> |
| Interfaz de Datos               | **Chat Templates** {role: ..., content: ...} |                         Manual String                         |                   Manual String                   |

Dados estos aspectos técnicos, utilizamos el siguiente dataset para poder comparar tanto los modelos con su versión base, como los modelos entre si. <br>

- Conversaciones agrupadas totales: 19766
- Palabras únicas: 27356
- Diversidad léxica (porcentaje): 37.28967%
- Cantidad de palabras por mensaje (Percentil 95): 75.0
- Mensajes idénticos 177
- Top palabras utilizadas:
  1. no (muletilla) - bueno (no muletila)
  2. me (muletilla) - como (no muletila)
  3. la (muletilla) - eso (no muletila)

### Modelo base vs modelo fine-tuned: ejemplos

Los siguientes ejemplos buscan comparar individualmente cada modelo base contra su propio modelo fine-tuneado, basandonos en la respuesta obtenida según una prompt predefinida. <br>

#### GPT2-spanish

> Pregunta: Nos vemos más tarde?

| Base                                                                                                                                                                                                                                                                                        | Fine-Tuned                                                                                                                                                                                                                 |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| antomm. —Sí, sí —respondió el capitán con aire inocente—. No tenemos tiempo para discutir sobre qué quiere usted decir "matar". ¿No está claro que no lo están haciendo de forma profesional o personal como una especie tan importante en las negociaciones entre la CIA y su gobierno pro | bueno, entonces la veré cuando salgo del trabajo y te digo que voy a necesitar para el fin de semana q viene. pero no se si me conviene en eso.. ya veremos!!!!! ahora vuelvo! tipo 17:20 o 18:30 por donde llego??ooo xdx |

#### Phi-3

> Pregunta: Nos vemos más tarde?

| Base                                                                                                                                                                                                                                                                                                                        | Fine-Tuned |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| La pregunta hace referencia a Melina, una persona o entidad ficticia en este contexto. Para abordar esta cuestión desde su perspectiva hipotética sobre la programática podría estructurarse así: "En relación con mi experiencia personal como asistente virtual basado en inteligencia artificial (IA), me resulta fascin |
| jsjsjshhhd <\|msg_sep\|> oooh si xd [OTRO] mmm me di cuenta q era una pajarita con la mamá ahí le gusta a mí, y cree yo es solo por tener 10 años!! no sé dónde estamos. te busco en esta parte para verla mejor???                                                                                                         |

#### SmolLM3-3B

> Pregunta: Nos vemos más tarde?

| Base                                                                                                                                               | Fine-Tuned                                                              |
| -------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| ¡Claro! Háblame más tarde y cuéntame qué te ha parecido mi respuesta hasta ahora. Estoy aquí para ayudarte siempre que lo necesites. ¡Hasta luego! | nop <\|msg_sep\|> me voy a dormir xq tengo q levantarme temprano mañana |

### GPT2-spanish vs Phi-3 vs SmolLM3-3B: ejemplos

| Prompt/Pregunta                | Respuesta: GPT2                                                                                                                                                                                                                                                   | Respuesta: Phi-3                                                                                                                                                                                                                                             | Respuesta: SmolLM3-3B                                                                                                                                                                                        |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Hola, cómo estás?              | bienn. al menos lo hice por las dudas de q me iba a molestar un poco con el celu.. y no tuve la más ligera duda en su favor... pero creo que si me atendieron hoy mañana va siendo muy largo: es una relación algo incómoda para vos porque está mal hacer        | hmm no se <\|msg_sep\|> me parece a lo q te dije [OTRO] okaw.. pero eso tengo la intención en terminar y ver si qué haces esa parte del código con los datos .y.js ... voy por allí... yo también veía cuando vagaba desde casa hasta ahí para ir al trabajo | bien                                                                                                                                                                                                         |
| Eu sale algo el finde?         | pero no c q onda la idea, te va a ir bien xdxdd3 y es más fácil para mí hija que si lo haga de vuelta al frente del grupo. como yo siento mucho todo esto desde hace tiempo :01579000 me parece raro eso ya veo!! o                                               | yo ya xq estoy en dónde <\|msg_sep\|> me fui del colegio a las 8 y luego volvi [OTRO] u.u, con los amigos hice una apunantada q no sé cómo se dice la otra forma :p es como mia tío le digo ojala queres hacer cosas                                         | si <\|msg_sep\|> mi mamá dice q si le digo a mis abuelos no les molesta... puedo ir al centro <\|msg_sep\|> pero necesito q me lleven a casa después <\|msg_sep\|> entonces voy a preguntarles cuando llegue |
| Qué opinás de la programación? | que cosa?? como vas a tener q ver con los colectivos o algo así me dice mi mamá xq no puedo seguir estudiando pero estoy en un curso porque tengo clases y para el cole ya hace unas semanas era muy tarde, luego tuve clase por lo menos hasta las 12:30. asi si | uwu <\|msg_sep\|> no sé si lo pienso q estés con él o ella se puede hacer en algún momento, pero creo estar perdida... :c [OTRO] mmmm hablami hoy pq me gustaría quedar para ver y decirle porqué es tan lento a veces xd 30 minutos                         | mmh no sé <\|msg_sep\|> me gusta hasta cierto punto <\|msg_sep\|> pero es difícil para mí <\|msg_sep\|> y el tiempo q necesito... <\|msg_sep\|> y yo quiero ser buena en algo más rápido                     |
| Me aburrooo, contame algo      | xd :c sarae tmb re bien q lo pasaste en el curso? o la universidad universitaria ? que te paso a vos como si yo fuera tu chica de sexto año y no me va nada del cole. así se nota cuando estás con alguien asi puede ser uggg                                     | ooh si bueno <\|msg_sep\|> y cómo te va? [OTRO] :c la otra mañana me da hambre xd no sé por q tengo los palillitos en las manitas a mí le pasa lo mismo pero se reaplica como con quines echados sin saberlo... yo voy mas fuerte eso                        | y te digo q no me gusta hablar mucho < \| msg_sep \| > yo estoy haciendo lo mismo y luego me arrepiento xd                                                                                                   |
| Nos vemos más tarde?           | bueno, entonces la veré cuando salgo del trabajo y te digo que voy a necesitar para el fin de semana q viene. pero no se si me conviene en eso.. ya veremos!!!!! ahora vuelvo! tipo 17:20 o 18:30 por donde llego??ooo xdx                                        | jsjsjshhhd <\|msg_sep\|> oooh si xd [OTRO] mmm me di cuenta q era una pajarita con la mamá ahí le gusta a mí, y cree yo es solo por tener 10 años!! no sé dónde estamos. te busco en esta parte para verla mejor???                                          | nop <\|msg_sep\|> me voy a dormir xq tengo q levantarme temprano mañana                                                                                                                                      |

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

Entrenar modelos de lenguaje no es tarea trivial para nuestro hardware, necesitaremos de una GPU que resista el entrenamiento y un entorno que nos permita utilizarla ininterrumpidamente por un tiempo que puede variar entre minutos u horas, dependiendo la cantidad de conversaciones de los datos o las especificaciones de la GPU. Por ejemplo nosotros empezamos utilizando [Google Colab](https://colab.google/), pero para la cantidad de chats y en modelos más grandes que GPT2-spanish, las 15GB de la GPU Nvidia T4 del plan gratuito no fueron suficientes. Si usted desea obtener resultados similares, recomendamos ampliamente clonar este repositorio en un entorno que soporte esta clase de exigencias.

### 3. Carga de datos

En las notebooks encontrará una celda interactiva, que le permitirá cargar su archivo txt y especificar el nombre del autor, es escencial que este coincida tal cual aparece en el archivo de chats. <br>

En las siguientes celdas se filtrarán los datos y acomodarán según el modelo, generandose así un archivo que será el utilizado para el entrenamiento. <br>

La celda _Análisis estadístico y calidad del dataset por etapas_ le permitirá evaluar si el dataset provisto es adecuado para el entrenamiento, se exhiben los datos en dos etapas del dataset: inicial (la que usted ingresa sin modificaciones) y filtrada (se realizó la limpieza necesaria). <br>

En este punto recomendamos reiniciar el kernel, tendrá que ejecutar menos líneas y podrá continuar al siguiente paso. Esto para reducir la memoria de la GPU y prepararla para la siguiente etapa. <br>

### 4. Entrenamiento

Dependiendo del modelo que haya elegido, su hardware disponible y la cantidad de conversaciones que se hayan conservado de sus chats; esto tomará mayor o menor tiempo, pero a comparación con el resto, esta es la etapa que tomará más tiempo. <br>

Una vez terminado el entrenamiento, el modelo será guardado. Si cuenta con un entorno de ejecución como Google Colab que elimina los archivos una vez desconectado del Kernel, no olvide de descargar el modelo entrenado.

### 5. Comprobación, Chatbot y Métricas

Las siguientes celdas cuentan con una **comprobación** para aquellos que quieran ver ejemplos de respuesta entre el modelo base y el fine-tuneado. <br>
Seguido por el **chatbot** interactivo, en el cuál podrá enviarle el mensaje (prompt) que desee y obtendrá la respuesta del modelo fine-tuned. <br>
Por último, si desea visualizarlo, se encuentran **evaluaciones** donde se compara la Perplexity entre el modelo base y el fine-tuned. <br>

## Conclusión

Cuando se comparan los distintos modelos entre sí en su versión fine-tuned, las respuestas de **SmolLM3-3B** destacan frente al resto por su coherencia y capacidad de capturar la escencia del autor. <br>
La victoria de SmolLM3-3B no es una casualidad, las etapas de prueba y error nos indicaron el camino hacia nuestro objetivo. En esta sección exploraremos las distintas extrategias aplicadas dada ciertas hipótesis. <br>

En la sección _Comparación de modelos_ los ejemplos de respuesta del modelo base versus su versión fine-tuned demostraron que, aunque no sea siempre una respuesta coherente, los modelos consiguieron recuperar manerismos de la forma de escritura del autor. Se rescatan palabras o expresiones conmunmente utilizados que contrastan con el modo original del modelo. Por ejemplo _Phi3_ aclara constantemente que es una IA, para luego dejar de nombrar esto en cada respuesta, o _SmolLM3-3B_ es formal, utliza los símbolos "¿" y "¡" que no tienden a ser respetados en una conversación virtual coloquial. El cambio radical de cada modelo respecto a su versión inicial, demuestra el éxito del fine-tune aplicado. <br>

Pero como fue nombrado anteriormente, SmolLM3-3B destacó frente al resto. Uno de los mayores factores se debió a la orientación del modelo en sí. <br>
**GPT2** [6] es un modelo de lenguaje base, fue creado para "completar texto" y por lo mismo se lo ajustó mediante CLM (Causal Language Modeling), además es un modelo con apenas 124 millones de parámetros, pequeño en comparación a los otros dos, cercanos a los 3 billones. Lo elegimos en un inicio por la limitación de hardware que teníamos en aquel momento, tuvimos que aplicar filtrado por similitud para evitar ruido en los datos de entrenamiento. <br>
El siguiente modelo que pusimos a prueba fue **Phi-3**, un modelo orientado a instrucciones de Microsoft. Esta segunda vuelta elegimos un modelo con mayor cantidad de parámetros y más robusto. A diferencia de GPT2 que fue únicamente entrenado con artículos de Wikipedia y libros, Phi-3 fue entrenado, además de artículos y libros, con "datos supervisados en formato de chat de alta calidad, que cubren diversos temas para reflejar las preferencias humanas en diferentes aspectos, tales como el seguimiento de instrucciones, la veracidad, la honestidad y la utilidad" [7]. Esta diversificación en su entrenamiento contrasta desde el modelo base de Phi-3 respecto a GPT2. <br>
Por último pusimos a prueba a **SmolLM3-3B**, un modelo creado para preservar la eficiencia, es decir que pueda funcionar en GPUs locales, sin compremeter su asertividad [8]. Desde su modelo base notamos una mejora significativa: el modelo comprende que debe entablar una conversación y responde coherentemente. Decidimos omitir el filtrado por similitud, esperando enriquecer la conversación y confiando en que no confundiría al modelo. <br>
Una de las mayores mejoras es su interfaz de datos de entrada: {role: ..., content: ...}, este formato nos permitió ingresar conversaciones de la forma:

```
{role: "user", content: "hola, cómo estas"}
{role: "assistant", content: "bieen, pero algo cansada, vos?"}
{role: "user", content: "bien también, uh cansada porq?"}
```

Cuando en los modelos anteriores (Phi-3 y GPT2) habíamos intentado replicar lo mismo (dar el contexto de una conversación), tuvimos que crear tags internos:

```
[OTRO] hola, cómo estas
[YO] bieen, pero algo cansada, vos?
[OTRO] bien también, uh cansada porq?
```

Los anteriores modelos no pudieron asimilar correctamente el uso de estos tags, en los ejemplos anteriores aparece entre la respuesta "[OTRO]" cuando este tag representa que el autor no ha enviado ese mensaje. <br>
En contraste, SmolLM3-3B ha podido incorporar los tags internos _<\|msgsep\|_> correctamente, estos representan cuando el autor del mensaje ha presionado _enter_ pero continúa enviando mensajes relacionados entre sí.

Por último decidimos medir la **Perplexity** de los modelos, tanto su versión base como la fine-tuned. <br>
En criollo esta métrica evalúa entre cuántas _opciones_ está debatiendo el modelo para poder predecir la siguiente palabra. Entonces a menor Perplexity tendrá menor cantidad de opciones, por lo que estas serán más certeras. <br>
Dado el dataset especificado en _Comparación de modelos_, se obtuvieron los siguientes resultados:

| GPT2-spanish                                             |                       Phi-3                       |                          SmolLM3-3B                          |
| -------------------------------------------------------- | :-----------------------------------------------: | :----------------------------------------------------------: |
| ![gpt2 perplexity](./images/gpt2-spanish_perplexity.png) | ![phi3 perplexity](./images/phi-3_perplexity.png) | ![smollm3 3b perplexity](./images/smollm3-3b_perplexity.png) |

| Model        | Base | Fine-Tuned |
| ------------ | :--: | :--------: |
| GPT2-spanish | 1903 |   749.2    |
| Phi-3        | 37.3 |    6.1     |
| SmolLM3-3B   | 29.7 |     3      |

A partir de estos resultados, se observa una caída fuerte de Perplexity en los tres modelos luego del fine-tuning, lo que indica que el entrenamiento efectivamente alineó las predicciones del modelo con el estilo y distribución del dataset. En particular, **SmolLM3-3B** y **Phi-3** alcanzan valores bajos (3 y 6.1), lo que sugiere que el modelo “duda menos” al elegir la próxima palabra dentro de este dominio conversacional como habíamos explicado recién, generando respuestas más consistentes con los chats.  
En cambio, aunque **GPT2-spanish** también mejora (de 1903 a 749.2), sus valores siguen siendo muy altos en comparación, lo cual es coherente con su menor capacidad y con el hecho de que es un modelo base orientado a completar texto.

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

- [Nicolás Marcelo Bazán](https://github.com/BazanNicolas)
- [Melina Rocío Morales](https://github.com/Paaprikaa)

## Referencias

1. IBM. (s.f.). _What is fine-tuning?_. Recuperado de https://www.ibm.com/es-es/topics/fine-tuning
2. Hugging Face. (s.f.). _Training and fine-tuning_. En _Hugging Face Documentation_. Recuperado de https://huggingface.co/docs/transformers/es/training
3. Comet November 21, 2024. _Perplexity for LLM Evaluation_. Recuperado de https://www.comet.com/site/blog/perplexity-for-llm-evaluation/
4. UNC Supercómputo Wiki (s.f.). Recuperado de https://wiki.ccad.unc.edu.ar/
5. CLOUDFLARE (s.f.). _¿Qué es la adaptación de bajo rango (LoRA)?_ Recuperado de https://www.cloudflare.com/es-es/learning/aiwhat-is-lora/
6. Hugging Face. (s.f.). _DeepESP/gpt2-spanish_. Recuperado de https://huggingface.co/DeepESP/gpt2-spanish
7. Hugging Face. (s.f.). _microsoft/Phi-3-mini-4k-instruct_. Recuperado de https://huggingface.co/microsoft/Phi-3-mini-4k-instruct
8. Hugging Face. (s.f.). _HuggingFaceTB/SmolLM3-3B_. Recuperado de https://huggingface.co/HuggingFaceTB/SmolLM3-3B
