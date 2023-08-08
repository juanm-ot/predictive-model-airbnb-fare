Airbnb es una compañia que ofrece una plataforma en línea para alquileres de casas y apartamentos a corto plazo. Permite alquilar tu casa por una semana mientras estás fuera, o alquilar tu habitación vacía. Uno de los retos que los anfitriones de Airbnb se enfrentan es determinar el precio de alquiler nocturno; aunque Airbnb proporciona a los anfitriones una guía general, no hay métodos de fácil acceso para determinar el mejor precio para alquilar un espacio. 

El objetivo de este proyecto es explorar un conjunto de datos relacionado con propiedades de Airbnb y desarrollar un modelo que permita estimar la tarifa nocturna ideal para estas propiedades basados en los datos historicos. La tarifa nocturna es un factor crítico para los anfitriones de Airbnb, ya que influye directamente en la competitividad de la propiedad y en el atractivo para los posibles huéspedes.


## Estructura del repositorio

```linux
.
├── README.md                            # Leeme
├── 01_pipeline_process_and_eda.ipynb    # pre-procesamiento y analisis exploratorio
├── 02_notebook_models_iteration.ipynb   # notebook de experimentación
├── 03_m_gradient_boost.ipynb            # modelos
├── 03_m_linear_regression.ipynb
├── 03_m_random_forest.ipynb
│ 
├── listingss                            # raw data
└── processed_data_listings              # output pre-procesamiento data

```

## Flujo de datos
Se recibe el archivo ```listings.csv``` con los anuncios de Airbnb en Reino Unido en 2022, este set de datos esta alojado en Kaggle. Se realiza evaluación de calidad de datos y un procesamiento inicial para disponibilizar la data en un análisis exploratorio de datos. El output de este proceso es ```processed_data_listings.csv```

Durante el proceso de **evaluación de calidad** se verifica la ausencia de registros duplicados y se eliminan columnas irrelevantes, como "license" y "neighbourhood". También, se identifica que aproximadamente el 25% de los datos para "reviews_per_month" y "last_review" estaban vacíos, y al investigar, se determino que estos valores nulos indican que las propiedades no han recibido reseñas. Por lo tanto, se llenan estos valores con ceros.

Se verifica el tipo de datos de cada columna para convertirla y finalmente se crea un diccionario de datos para mejorar la comprensión de las columnas con nombres no claros.

## EDA
En el **análisis exploratorio de datos** realizado, se han identificado diversas relaciones y patrones importantes:

* Se encontró que el tipo de habitación más común es "Entire home/apt", mientras que los barrios más populares se encuentran en el centro de Londres, siendo Westminster el que tiene la mayor oferta, representando el 11.9% del total. Sin embargo, se observó que la mayoría de los datos no siguen una distribución normal y presentan relaciones no lineales, lo que sugiere la necesidad de usar el método de correlación de Spearman.

* En cuanto a la correlación entre variables, se encontró una fuerte correlación (0.8) entre las columnas "reviews_per_month" y "number_of_reviews", así como con "number_of_reviews_ltm". Debido a estas relaciones, se decidió eliminar las dos primeras junto con "last_review". Además, se identificó que el precio está relacionado tanto con el tipo de habitación como con el barrio, mostrando que los barrios más céntricos tienen precios más elevados.

* Con respecto a la distribución del precio, se notó que el 75% de los valores son menores a 180, mientras que el último cuartil presenta valores desproporcionadamente altos, lo que podría afectar el rendimiento del modelo. Por esta razón, se propone seleccionar los listings con precio menor a 500 y aquellos donde el precio sea diferente a 0.

Estos hallazgos servirán como base para la construcción de un modelo adecuado que estime la tarifa nocturna ideal para las propiedades de Airbnb en esta ciudad. Visite el archivo ```01_pipeline_process_and_eda.ipynb`` para los detalles.


## Feature Engineering
La variable a predecir es **price**. Se realizó un análisis de la correlación de esta variable y no esta correlacionada ni positiva ni negativamente con las demas variables.
[![correlation-onehot-coding.png](https://i.postimg.cc/gcMk63nq/correlation-onehot-coding.png)](https://postimg.cc/G9y1W8PH)

Las variables explicativas seleccionadas son: 
[![var-Explain.png](https://i.postimg.cc/hvZfc5Jh/var-Explain.png)](https://postimg.cc/vx9QvXtd)

Gracias al EDA, teóricamente, se pudo identificar la relación del **price** con el *type_room* y *neighbourhood*. Estas variables categóricas se codificaron usando la técnica de one-hot enconding. Tambien se experimentó con LabelEncoder, con una combinación de LabelEncoder para neighbourhood y One-hot encoding para type_room; pero ambas afectaron negativamente el performance del modelo. En iteraciones iniciales se incluyó como variable el "number_of_reviews_ltm". Sin embargo, cuando se eliminó en el entrenamiento mejoró el performance del modelo. 

Finalmente se propone una transformación logarítmica para reducir la escala de valores y para manejar datos con sesgo o asimetría; y se escala la data con el método StandardScaler antes del entrenamiento.
[![trasnformacion-log.png](https://i.postimg.cc/fb3LfKrh/trasnformacion-log.png)](https://postimg.cc/VS8mYjw7)


## Modelos de prueba 

Como el objetivo es predecir un valor númerico, se utilizan modelos de regresion. Iniciando por el mas simple y complementando la toma de desición basado en la literatura y experiencia

* Regresión Lineal: Es uno de los modelos más sencillos. En este caso, puede ser útil porque se esta buscando una relación lineal entre las características y el price de una propiedad de Airbnb.
* Gradient Boosting: A diferencia de la regresión lineal, el gradient boosting puede capturar relaciones no lineales entre las características y el objetivo, también tiene la capacidad de manejar características categóricas y datos con valores atípicos. 
* Random Forest: Al igual que el gradient boosting, Random Forest puede manejar características no lineales y categóricas, así como datos con ruido o valores atípicos. Al crear múltiples decision trees y promediar sus predicciones, reduce el riesgo de sobreajuste y produce resultados más estables y precisos.

Nota: Además se probaron Decision Trees y XGBoost.El primero no tuvo buen performance y el XGboost tuvo métricas similares a los seleccionados,por tanto, debido al scope del proyecto se delimitó a la prueba de tres modelos

## Metricas de evaluación
Se utilizaron las siguientes métricas para evaluar el modelo:

* **MAE (Mean Absolute Error):** Esta métrica mide el promedio de la magnitud de los errores en las predicciones del modelo. Es útil porque nos da una idea de cuánto se desvían, en promedio, las predicciones del valor real. Un valor de MAE más bajo indica que el modelo tiene un menor error promedio en sus predicciones.

* **R^2 (Coeficiente de Determinación):** Esta métrica proporciona una medida de qué tan bien se ajustan las predicciones del modelo a los datos reales. Representa la proporción de la varianza en la variable objetivo que es predecible a partir de las variables explicativas. Un valor de R^2 más cercano a 1 indica que el modelo explica una gran parte de la variabilidad en los datos.

* **RMSE (Root Mean Squared Error):** Es una métrica que penaliza más los errores grandes en las predicciones del modelo. El RMSE es la raíz cuadrada del error cuadrático medio, que es la media de los errores al cuadrado entre las predicciones y los valores reales. Un valor de RMSE más bajo indica que el modelo tiene un menor error en sus predicciones.

Estas métricas se utilizan en conjunto para evaluar el rendimiento de un modelo de regresión. Un buen modelo debería tener un MAE, RMSE y R^2 lo más bajo posible, lo que indica que sus predicciones están cerca de los valores reales y que explica una gran parte de la variabilidad en los datos. Al comparar los resultados de los modelos desarrollados, se puede identificar cuál de ellos tiene un mejor desempeño. 

[![descarga.png](https://i.postimg.cc/SKQrfqjH/descarga.png)](https://postimg.cc/0z3D9TYC)

Se propone el calculo de los coeficientes de importancia para cada feature. Estos indican qué tan importante es cada variable en la toma de decisiones del modelo, cuanto mayor sea la importancia de una característica, más influencia tiene en la predicción del modelo y contribuye significativamente a su precisión.

[![feature-importance.png](https://i.postimg.cc/KYRsJkbT/feature-importance.png)](https://postimg.cc/wtYVBB7x)

Es interesante que los features propuestos en el EDA son los mas importantes para los modelos. Lideran el ranking room_type(hotel aumenta el price), la localización geográfica y la ubicación cercana a los barrios centrales. 

## Conclusiones y proximos pasos

El Random Forest es el modelo con buenas métricas de evaluación. Presenta un buen desempeño en r2:0.66, pero hay margen para mejorar el ajuste y reducir el error promedio (MAE y RMSE). Es posible que se puedan explorar otras técnicas de modelado, ajustar hiperparámetros, realizar cross validation o incluir más características para mejorar la precisión del modelo y lograr un mejor ajuste a los datos.

Se reviso el estado del arte de la solución a un problema similar y se identificó que los modelos de predicción de tarifas, con cantidad de variables reducida, tienen un performance en los rangos obtenidos en este proyecto; para mejorar la predicción significativamente han introducido mas variables(cerca de 56) y más registros. 

## Referencias e investigación

* A. Lektorov, E. Abdelfattah and S. Joshi, "Airbnb Rental Price Prediction Using Machine Learning Models," 2023 IEEE 13th Annual Computing and Communication Workshop and Conference (CCWC), Las Vegas, NV, USA, 2023, pp. 0339-0344, doi: 10.1109/CCWC57344.2023.10099266. 
* J. Dhillon et al., "Analysis of Airbnb Prices using Machine Learning Techniques," 2021 IEEE 11th Annual Computing and Communication Workshop and Conference (CCWC), NV, USA, 2021, pp. 0297-0303, doi: 10.1109/CCWC51732.2021.9376144.
* A. Garlapati, K. Garlapati, N. Malisetty, D. R. Krishna and G. Narayana, "Price Listing Predictions and Forthcoming Analysis of Airbnb," 2021 12th International Conference on Computing Communication and Networking Technologies (ICCCNT), Kharagpur, India, 2021, pp. 1-7, doi: 10.1109/ICCCNT51525.2021.9579773.












