# Pre-análisis
--------------

## Introducción
En este trabajo vamos a recrear los métodos utilizados del paper [Predicting intelligence from brain gray matter volume](https://link.springer.com/article/10.1007/s00429-020-02113-7) para analizar si existe alguna relación entre la inteligencia y la morfología del cerebro e intentar predecir la inteligencia a partir de datos anatómicos del cerebro. En el paper, la forma de cuantificar esta inteligencia está medido por un test de IQ. Buscamos replicar los resultados obtenidos utilizando otro método de medición de la inteligencia a partir de los parámetros Memory, Fluid y Cristalization obtenidos por el [IST](https://www.google.com/url?q=https://www.biorxiv.org/lookup/google-scholar?link_type%3Dgooglescholar%26gs_type%3Darticle%26q_txt%3DVorst%252C%2BH.%2B(2010).%2BIntelligentie%2BStructuur%2BTest%2B(IST).%2BAmsterdam%253A%2BHogrefe.%2BDutch%2Badaptation%2Bfrom%253A%2BLiepmann%252C%2BD.%252C%2BBeauducel%252C%2BA.%252C%2BBrocke%252C%2BB.%2B%2526%2BAmthauer%2BR.%2B(2001).%2BIntelligentz%2BStructur%2BTest.%2BG%25C3%25B6ttingen%253A%2BHogrefe&sa=D&source=docs&ust=1715614056546540&usg=AOvVaw21xhwT2YUAl0VP_jJ0Ogfs). Nos resulta interesante analizar si estas nuevas métricas nos permiten conseguir los mismos resultados o incluso mejorarlos. 
La idea es generar modelos para cada métrica Memory, Cristallization Intelligence y Fluid Intelligence, evaluarlos y luego hacer la suma de los mismos y evaluar el resultado con el valor de inteligencia verdadero.


## Métodos

### Dataset
Utilizaremos el dataset de [AOMIC ID1000](https://openneuro.org/datasets/ds003097/versions/1.2.1) para conseguir los datos anatómicos, así como también los valores del IST para los output. En este dataset se tomaron muestras de distintas modalidades de MRI de 928 participantes e incluye información de cada uno de los cuales a nosotros nos interesa ver la inteligencia, y en particular tres parámetros sobre Memory, Fluid y Cristalization, siendo la inteligencia la suma de estos tres parámetros. La Fluid Intelligence se refiere a los procesos mentales que un individuo realiza al enfrentarse a problemas novedosos sin conocimiento previo. La Cristalization Intelligence se refiere a la amplitud de los conocimientos previamente adquiridos.

### Features
Para entrenar estaremos usando VBM - GM (Voxel-Based Morphology - Gray Matter), utilizando las parcelaciones usadas en el paper. La que se utiliza principalmente es Schaefer de 400 parcelaciones y 7 networks con la media para la agregación. Luego se probará con el atlas Shen de 2019 con 368 parcelaciones y la media para la agregación. Entrenaremos los modelos con cada conjunto de features por separado y compararemos la performance de cada uno. Nos guardaremos un 10% de los datos para validación para usar luego de entrenar los modelos. Los features de entrenamiento serán estandarizados usando z-score. 

### Modelos
Los modelos que usaremos para las predicciones son:
- **SVM (Support Vector Machine)** con kernel lineal, sigmoide y rbf. Para la lineal, utilizamos un rango de valores de 1e-6 a 1 con distribución logarítmica uniforme y un epsilon de 0.01 a 3 con distribución uniforme, los cuales sacamos del paper original. Para el rbf y sigmoide utilizamos los mismos rangos de valores pero también agregamos un gamma con valores 1e-5, 1e-4, 1e-3, 1e-2, 'scale' y 'auto'. Se utilizará una estrategia de optimización Bayesiana para conseguir la mejor configuración de estos valores y evaluaremos 50 configuraciones distintas al igual que en el paper.
- **RF (Random Forest)** con cantidad de estimadores 25, 50 y 100, y profundidad máxima de 5, 10 y 20.  
- **XG-Boost** (Extreme Gradient Boosting), con cantidad de estimadores 25, 50 y 100, y profundidad máxima de 3, 5 y 10. 

Los ensambles se calcularán usando Grid Search. Compararemos la performance de estos 3 modelos entrenando un modelo **Dummy** a la par, para ver si realmente el mejor modelo consigue un avance significativo a un modelo que solo elige la media. 


### Evaluación
Para evaluar los modelos utilizaremos continuous stratified repeated cross-validation, con 3 bins, 3 splits y 10 repeticiones, midiendo la métrica de Negative Mean Absolute Error (NMAE). De esta manera, la estratificación para valores continuos nos permitirá tener folds más balanceados con una mayor cantidad de features por fold, ya que solo se partirá en 3 por repetición. Las repeticiones ayudarán a reducir la varianza en las estimaciones. 

