# PruebaTecnicaEtraining
### REALIZADO POR JULIANA ALEJANDRA ARENAS LOBO
## Modelo Predictivo de Ventas - Prueba Técnica

# Objetivo
Este proyecto corresponde a una prueba técnica de análisis de datos e inteligencia artificial orientada al desarrollo de un modelo predictivo para la cadena FastFoods de Bogotá, para estimar el volumen de ventas en función de variables como la precipitación en mm, sacados de sensores y procesados según la hora más cercana al evento de compra, y ubicación más cercana a la tienda de compra.

El objetivo es desarrollar un modelo que prediga las ventas diarias por tienda, integrando información geográfica, características de la tienda y ambiental.
Se evalua también la influencia real de las precipitaciones en la dinámica de ventas, comparando un modelo simple contra uno más completo en cuanto a variables dependientes.
Se recomienda encarecidamente revisar el .ipynb para mirar con más detalle qué se hizo, cómo se hizo y por qué se hizo, así como la visualización e interpretación de resultados.

# Qué se hizo?
1. Extracción de datos
a. Se conectaron dos fuentes:

* Base de datos MySQL: para obtener ventas, productos, tiendas y regiones.
* MongoDB Atlas: para ubicar sensores y eventos de precipitación.

b. Se integraron ambas fuentes creando un dataset final: ventas_enriquecidas.csv. Dentro del .ipynb se pueden ver los detalles de procesamiento, limpieza y transformación.

2. Transformación y enriquecimiento

a. Se creó una columna ubicacion_tienda como tupla (lat, lon).

b. Se asignó el sensor más cercano a cada tienda usando geolocalización.

c. Se interpoló la precipitación más cercana en el tiempo por cada venta.

d. Se creó una columna temporada y variables como día, mes, año, hora, horario.

3. Agrupación geográfica (DBSCAN)

a. Se usó DBSCAN para agrupar tiendas en regiones similares basadas en su ubicación geográfica, sin requerir un número fijo de clústeres. Se obtiene region_cluster y se añadió como variable predictora clave.

4. Modelado
   
a. Se entrenaron dos modelos con CatBoostRegressor:

* Modelo 1 (completo): empleados, valor_precipitacion_promedio, region_cluster.
* Modelo 2 (solo lluvia): únicamente valor_precipitacion_promedio.

# Archivos generados
| Archivo                        | Descripción                                                                 |
|-------------------------------|------------------------------------------------------------------------------|
| `Modelo_Predictivo_Prueba_Tecnica.ipynb` | Notebook principal con todo el proceso técnico, análisis y visualizaciones.         |
| `ventas_enriquecidas.csv`     | Dataset final con todas las variables transformadas y enriquecidas.         |
| `modelo_catboost.pkl`         | Modelo 1 entrenado final          |

# Resultados

Dentro del .ipynb estarán más análisis y gráficos explicativos, en esta sección se resaltarán los principales que llevaron al modelo final.

Análisis de correlación de variables

![image](https://github.com/user-attachments/assets/1a239634-b2c8-49ed-b046-b730b3af4f29)

* Empleados y Escala tienen fuerte correlación positiva con ventas:
* empleados vs ventas_total_dia = 0.64
* escala vs ventas_total_dia = 0.65
* Precipitación promedio vs ventas = 0.12 es correlación débil
* Variables geográficas (latitud, longitud) tienen correlaciones variadas (posible relación con ubicación de zonas de alta o baja venta).
* No hay evidencia clara de que la temporada por sí sola tenga fuerte influencia.
* tipo_venta no está correlacionado directamente con ventas_total_dia, probablemente porque ambas están representadas en subconjuntos separados.

![image](https://github.com/user-attachments/assets/c1320409-8f34-4b37-bbea-372b3a18b0a0)

* Se puede evidenciar la correlación en el gráfico de barras de Volumen de ventas por Escala de tiendas, pues entre la tienda sea más grande, más clientes atraerá, más ventas tendrá.
  
![image](https://github.com/user-attachments/assets/c7491064-7b6f-4609-81e2-451b9f735222)

* Se evidencia que, entre más número de empleados tenga la tienda, más ventas producirá, aunque se ve que en algunas cantidades de mediana escala de empleados se evidencian más ventas que las de gran escala.
* La precipitación tiene muy baja correlación con las ventas, tanto globales como por canal.
* Las variables internas de la tienda como empleados y escala tienen una correlación mucho más significativa.
* Se toma con base en todo el análisis anterior la decisión de construir dos modelos predictivos: Uno solo con precipitación (para evaluar su efecto aislado), uno con precipitación + empleados (para mejorar el poder predictivo del modelo).

Modelo 1 , el completo (empleados, lluvia, ubicación):

* R2 score: 0.76
* MSE: 78.26
* Los gráficos mostraron una fuerte alineación entre ventas reales y predichas.
* Hay errores distribuidos de forma simétrica, centrados en cero.
* Hay algunos valores atípicos (outliers) que se alejan de 20 o -30, pero son pocos.
![image](https://github.com/user-attachments/assets/ecca7f1f-aa5f-493c-b780-0c14ba4d47e4)

![image](https://github.com/user-attachments/assets/21d95555-6fa7-4acc-aec1-544f3dca5725)

Modelo 2, solo con lluvia:
* R2 score: 0.08
* MSE: 295.06
* Predicciones pobres, baja sensibilidad a la variabilidad real.
* Los residuos fueron más dispersos y sesgados, se confirma que la precipitación por sí sola no explica las ventas.
![image](https://github.com/user-attachments/assets/c3d12d70-2a92-4c14-a6ac-a0019b6ac4df)

![image](https://github.com/user-attachments/assets/582aefc9-f095-43d9-98cb-5e6b5ef5c86d)

# Por qué R2 score y MSE (Mean Squared Error) como métricas?

* Porque R2 score explica la proporción de la variabilidad en la variable dependiente (ventas_totales_dia) que es explicada por las variables independientes (que puede ser solo lluvia, o las manejadas en el modelo 1) y MSE indica el tamaño de los errores en términos absolutos algo útil para tener una idea de cuánto se desvían las predicciones del modelo de los valores reales.

# Por qué DBSCAN para el 1mer modelo?

Porque se buscaba incorporar la ubicación geográfica de las tiendas como una variable predictora. Dado que la ubicación está representada por coordenadas (latitud, longitud), era necesario transformar esta información en una forma numéricamente útil para el modelo, DBSCAN es un algoritmo de clustering espacial que agrupa puntos que están cerca unos de otros según una distancia euclidiana y una densidad mínima de puntos, y a diferencia de KMEANS, ignora outliers (ruido geográfico) e identifica automáticamente áreas densas de tiendas, así como el número de clusters sin indicarle.

# Por qué se eligió CatBoost y no otros modelos como XGBoost, LightGBM o Random Forest?

Porque en comparación con XGBoost y Random Forest, CatBoost fue más estable en cuanto a tiempo de entrenamiento y resultados, pues XGBoost había logrado en pruebas un R2 score de 73% y un MSE de 84, y Random Forest había obtenido el peor, 64% de R2 score y 115 de MSE.

# Diagrama ETL

![image](https://github.com/user-attachments/assets/05ce01d2-f1eb-407b-b99e-dbdafb1a219b)

# Conclusiones
* La variable de precipitación tiene muy bajo poder predictivo por sí sola.
* Las ventas están mucho más correlacionadas con variables internas como: Número de empleados. Tamaño de la tienda. Ubicación geográfica, agrupada con DBSCAN.
* Es posible incorporar múltiples fuentes de datos heterogéneos (bases SQL, sensores, eventos) en un solo pipeline para generar un dataframe con el que entrenar un modelo predictivo.
