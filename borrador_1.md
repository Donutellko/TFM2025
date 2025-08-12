# Resumen

TO DO: traducir del inglés

<!-- <s>Este trabajo de fin de máster presenta una propuesta innovadora para mejorar el
contexto que reciben los Modelos de Lenguaje de Gran Tamaño (LLMs) durante la
generación y regeneración automática de pruebas unitarias. La solución se basa en la
integración de técnicas de Recuperación Aumentada por Generación (RAG) para
enriquecer el contexto del modelo, utilizando embeddings generados con el modelo
microsoft/graphcodebert-base [1], un índice vectorial construido con FAISS [2]
y la incorporación de información relevante extraída del archivo pom.xml. Además,
se utiliza la API Mistral con Codestral 25.01 [3] para generar pruebas unitarias de
alta calidad en proyectos de software. La metodología propuesta ha sido evaluada en
el proyecto JInstagram [4], considerando diversos porcentajes de contexto (100 %,
60 %, 40 % y 10 %) para analizar su impacto en la cobertura y calidad de las pruebas
generadas.
Los resultados indican que el uso de un contexto completo mejora significativa-
mente la cobertura de sentencias y ramas, superando a enfoques existentes como
ChatUnitTest [5] o incluso los casos de prueba creados por los mismos desarrollado-
res del proyecto. Este estudio contribuye a la reducción del esfuerzo manual en la
escritura de pruebas y abre nuevas vías para la integración de LLMs en el ciclo de
desarrollo de software.
Palabras clave: Pruebas Unitarias, LLMs, RAG, Contexto -->

<!-- ... 
El transporte p´ublico, especialmente los servicios de autobuses urbanos, desempe˜na un
papel crucial en la movilidad de las ciudades modernas. Sin embargo, desaf´ıos como la congesti´on
del tr´afico, las condiciones clim´aticas impredecibles y los incidentes operativos a menudo resultan en
retrasos que afectan negativamente la calidad del servicio y la experiencia del usuario. Este estudio
explora la aplicaci´on de t´ecnicas de aprendizaje autom´atico automatizado (AutoML) para predecir
los retrasos de autobuses, con el objetivo de mejorar la eficiencia y confiabilidad de los servicios de
autobuses urbanos.
En este trabajo, se han desarrollado y evaluado una serie de modelos predictivos utilizando AutoML
para automatizar la selecci´on y optimizaci´on de algoritmos e hiperpar´ametros. El enfoque se prob´o
utilizando datos del mundo real del sistema de autobuses p´ublicos de M´alaga, Espa˜na, cubriendo varios
meses de operaciones. Los resultados demostraron que AutoML supera significativamente a los m´eto-
dos tradicionales de aprendizaje autom´atico, como el Bosque Aleatorio (Random Forest), Perceptr´on
Multicapa (MLP) y M´aquinas de Vectores de Soporte (SVM), en t´erminos de precisi´on de predicci´on.
Los hallazgos clave resaltan los beneficios sustanciales del uso de AutoML, especialmente para usua-
rios sin conocimientos profundos en aprendizaje autom´atico. Sin embargo, el estudio tambi´en identific´o
el alto costo computacional de AutoML como un desaf´ıo significativo. Para abordar esto, se explo-
raron t´ecnicas de aprendizaje por transferencia para reducir las demandas computacionales, logrando
resultados prometedores.
Este estudio subraya el potencial de AutoML para transformar los sistemas de transporte p´ublico, pro-
porcionando predicciones precisas y fiables de los retrasos, mejorando en ´ultima instancia la satisfacci´on
del usuario y promoviendo la movilidad urbana sostenible.
</s> -->



# Abstract

Benchmarks are widely used for measuring and comparing performance of LLMs in different areas, including logical puzzles, factual accuracy, and coding tasks among others. 
The use of benchmarks allows developers to fine-tune a model, and a user to choose a model that better fits their needs or adjust its responses by changing a system prompt and parameters like temperature. 
This requires a lot of benchmark runs. Running a benchmark is costly, time- and energy-consuming. Also, it is often inefficient, as they may contain problems that are not relevant for the use case of a model being adjusted. 

So the goal of this work is to explore existing benchmarks, and design and develop an approach to benchmarking that is highly customizable and extendible, time-efficient, eco-friendly. As a result, a modular benchmark was developed, that allows to add and modify test cases using a user interface, and configure a benchmark run using task filters and toggles for enabling or disabling specific checks. Also, the output of the benchmark allows to examine the results in detail. 

<s>The existing benchmarks differ in contents and implementation. Most benchmarks are developed from scratch or forked from existing ones, and are not created to be easily adaptable or extendible for use cases.  </s>




<!-- <s>This master’s thesis presents an innovative approach to enhancing the context
provided to Large Language Models (LLMs) during the automatic generation and
regeneration of unit tests. The solution is based on integrating Retrieval Augmented
Generation (RAG) techniques to enrich the model’s context by using embeddings
generated with the model microsoft/graphcodebert-base [1], a vector index built
with FAISS [2], and incorporating relevant information extracted from the pom.xml
file. Additionally, the Mistral API with Codestral 25.01 [3] is employed to generate
high-quality unit tests for software projects. The proposed methodology has been
evaluated on the JInstagram project [4], using various context percentages (100 %,
60 %, 40 %, and 10 %) to analyze its impact on the coverage and quality of the
generated tests.
The results indicate that using complete context significantly improves statement
and branch coverage, outperforming existing approaches such as ChatUnitTest [5]
and even the tests created by the project’s own developers. This study contributes
to reducing the manual effort required for writing tests and opens up new avenues
for integrating LLMs into the software development lifecycle.
Keywords: Unit Testing, LLMs, RAG, Context</s> -->


# Introducción

Benchmarks are widely used for measuring and comparing performance of LLMs in different areas, including logical puzzles, factual accuracy, and coding tasks among others. 
The use of benchmarks allows developers to fine-tune a model, and a user to choose a model that better fits their needs or adjust its responses by changing a system prompt and parameters like temperature. 
This requires a lot of benchmark runs. Running a benchmark is costly, time- and energy-consuming. Also, it is often inefficient, as they may contain problems that are not relevant for the use case of a model being adjusted. 

So the goal of this work is to explore existing benchmarks, and design and develop an approach to benchmarking that is highly customizable and extendible, time-efficient, eco-friendly. As a result, a modular benchmark was developed, that allows to add and modify test cases using a user interface, and configure a benchmark run using task filters and toggles for enabling or disabling specific checks. Also, the output of the benchmark allows to examine the results in detail. 

The existing benchmarks differ in contents and implementation. Most benchmarks are developed from scratch or forked from existing ones, and are not created to be easily adaptable or extendible for use cases. 





Any developed benchmark eventually falls out of use due to its saturation: when models reach a level that is considered sufficient or human-like, and remaining errors can be attributed to problems in the benchmark itself, like ambiguity of the tasks . For example, most modern models can't achieve over 95% evaluations in GSM8K (a dataset of grade-school level math word problems), and with the remaining 5%, upon being analyzed, half of the errors are attributed to label noise and ambiguity. [cite vendrow2025largelanguagemodelbenchmarks]
<!-- So the difficulty of the tasks in benchmarks grows over time to reflect improvements of the models. -->

<!-- vendrow2025largelanguagemodelbenchmarks provides a table with analysis of common benchmarks -->



# Antecedentes y trabajos relacionados

# Descripción del problema

# Detalles de la propuesta

# Experimentación y resultados

# Conclusiones y trabajo futuro