# TP-Analitica-Descriptiva

## Estructura y adquisición de los datos

El proyecto utiliza información proveniente de la plataforma **Inside Airbnb**, específicamente de la sección correspondiente a la ciudad de **Buenos Aires**.

En el repositorio, dentro de la carpeta `/data/`, se incluyen **archivos de muestra** extraídos del paquete de datos publicado el **29 de enero de 2025**.  
Estos archivos son una versión reducida del dataset original, seleccionada para facilitar la manipulación y el análisis en entornos colaborativos (por ejemplo, Google Colab o GitHub).  
La muestra conserva la misma estructura de columnas y tipos de datos que el dataset completo, por lo que es plenamente representativa para el desarrollo del análisis exploratorio inicial.

El dataset contiene información detallada de **publicaciones activas en Airbnb**, incluyendo tanto características del alojamiento como del anfitrión.

Para quienes deseen replicar o ampliar el análisis con la totalidad de los datos, el dataset completo se encuentra disponible públicamente en:  
👉 https://insideairbnb.com/get-the-data/  

Allí pueden descargarse los archivos `.csv.gz` correspondientes, que incluyen todas las observaciones y variables utilizadas.

En caso de trabajar con el dataset completo, se recomienda ubicarlo en la misma carpeta `/data/` y actualizar las rutas en el notebook de análisis (`TP_Descriptiva.ipynb`).

---

## Limpieza de datos

Se realizó una estandarización inicial de tipos y valores:

- Conversión de campos monetarios (`price`) a **float**, eliminando símbolos “$” y comas.  
- Conversión de porcentajes (`host_response_rate`, `host_acceptance_rate`) a **enteros** (0–100).  
- Campos booleanos (`host_is_superhost`, `instant_bookable`, `host_identity_verified`) mapeados a **True/False**.  
- Fechas (`host_since`) transformadas al tipo **datetime**.  
- Reemplazo de valores imposibles o incoherentes por `NaN`:  
  - `price == 0`  
  - `maximum_nights == 0`  
  - porcentajes o fechas vacías  

Estas transformaciones aseguran la consistencia del dataset antes de evaluar nulos, duplicados y outliers.

---

## Tratamiento de 0s y nulos

- **Reemplazo de valores inválidos:**  
  Los `0` en `price` y `maximum_nights` fueron tratados como faltantes, ya que representan datos inexistentes y no valores válidos del fenómeno.  

- **Detección de nulos:**  
  Se generó una tabla de porcentaje de faltantes por columna para evaluar su relevancia.  

- **Eliminación de columnas:**  
  Aquellas con más del **20 % de valores nulos** fueron descartadas por su bajo aporte analítico y alto costo de imputación.  

- **Imputación de faltantes:**  
  - Variables numéricas imputadas por **mediana**, para evitar sesgos causados por outliers.  
  - Variables categóricas imputadas por **moda**, o moda dentro del mismo barrio.  

- **Duplicados:**  
  Se eliminaron registros duplicados según el identificador `id`.

---

## Eliminación de columnas
Se comienza el analisis unicamente con estas columnas de listings: 'id', 'name', 'description', 'neighborhood_overview', 'host_id', 'host_since', 'host_location', 'host_about', 'host_response_time', 'host_response_rate', 'host_acceptance_rate', 'host_is_superhost', 'host_identity_verified','neighbourhood_cleansed', 'property_type', 'room_type', 'accommodates', 'bathrooms', 'bathrooms_text', 'bedrooms', 'beds', 'amenities', 'price', 'minimum_nights', 'maximum_nights', 'availability_90', 'number_of_reviews_ltm','review_scores_rating','review_scores_accuracy','review_scores_cleanliness','review_scores_checkin','review_scores_communication','review_scores_location','review_scores_value','instant_bookable'.

Las demas se pueden considerar eliminadas.
De calendar se borra 'adjusted_price' .
Las columnas con >20% de datos faltantes no se imputaron y pueden considerarse eliminadas.


---

## Detección y análisis de outliers

- Se utilizó el método **IQR (rango intercuartílico)** para detectar valores extremos en variables numéricas como `price`, `bedrooms` y `accommodates`.  
- No se eliminaron outliers salvo los imposibles, dado que reflejan casos reales (propiedades de lujo, mayor capacidad, ubicaciones premium).  
- Se agregaron columnas auxiliares (`_outlier = True/False`) para rastrear observaciones extremas.  
- En el **análisis multivariado**, los outliers se analizaron según variables explicativas: `room_type`, `accommodates` y `neighbourhood_cleansed`.

---

## Contexto de negocio, hipótesis y objetivos

El análisis busca comprender los **factores que influyen en el precio y la calificación** de los alojamientos en Buenos Aires.  
Se pretende identificar relaciones entre características del alojamiento, ubicación y desempeño del anfitrión.

### Hipótesis planteadas

| Nº | Hipótesis | Variables involucradas |
|----|------------|------------------------|
| 1 | Los precios en barrios turísticos (Palermo, Recoleta, San Telmo) son sistemáticamente más altos que en barrios periféricos. | `price`, `neighbourhood_cleansed` |
| 2 | La presencia de amenities clave (wifi, aire acondicionado, cocina equipada) incrementa tanto el precio como la calificación promedio. | `amenities`, `price`, `review_scores_rating` |
| 3 | A menor tiempo de respuesta del host, mayor probabilidad de recibir una review positiva. | `host_response_time`, `review_scores_rating` |
| 4 | Existe relación entre el score de limpieza y el de ubicación. | `review_scores_cleanliness`, `review_scores_location` |

---

## Visualizaciones y conclusiones preliminares

**Gráficos realizados para contrastar hipótesis:**
1. **Distribución de precios (log-scale):**  
   Muestra asimetría hacia la derecha y outliers explicables por capacidad o ubicación.  
2. **Precio vs tipo de habitación:**  
   Boxplot: los *Entire home/apt* tienen medianas de precio mucho mayores que las *Private room*.  
3. **Precio vs capacidad (`accommodates`):**  
   Relación positiva clara: mayor capacidad → mayor precio.  
4. **Precio por barrio:**  
   Barplot (top-10 barrios): Palermo, Recoleta y Puerto Madero concentran los valores más altos.  
5. **Calificación vs amenities:**  
   Los alojamientos con wifi o aire acondicionado tienen mejores calificaciones promedio.  
6. **Tiempo de respuesta del host vs review positiva:**  
   Hosts con respuesta rápida obtienen mejores puntuaciones.  
7. **Cleanliness vs Location:**  
   Correlación positiva moderada entre ambos indicadores.

**Conclusiones preliminares:**
- Los **barrios turísticos** presentan precios significativamente más altos (hipótesis 1 confirmada).  
- Las **amenities clave** influyen positivamente en el precio y la valoración (hipótesis 2 parcialmente confirmada).  
- Los **superhosts** y anfitriones con **rápida respuesta** reciben mejores reviews.  
- Se confirma una **correlación moderada entre limpieza y ubicación** (hipótesis 4).  
- Los **outliers** representan casos reales de alta demanda o lujo y se mantuvieron en el análisis.

---

## Estructura del repositorio

