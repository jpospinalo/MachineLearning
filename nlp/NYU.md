## NYU - Hold-Out Validation

Este documento ejemplifica cómo documentar resultados experimentales de validación tipo *hold-out* en un formato claro y estructurado usando Markdown. La idea es disponer de una plantilla rica en información que pueda reutilizarse en otros experimentos.

---

### 1. Configuración general del experimento

> **Configuración de las pruebas:**  
> - Train 80% – Validation 20%  
> - 20 repeticiones con semillas aleatorias distintas  
> - Accuracy promedio y desviación estándar calculados sobre las repeticiones  
> - Clasificador base: SVM lineal con búsqueda de hiperparámetros simple (C fijo)  
> - Métrica principal: Accuracy (exactitud)  
> - Métricas secundarias: F1 macro, sensibilidad promedio (no mostradas en la tabla principal)

Además:

- Se mantienen constantes:
  - El mismo preprocesamiento de señales,
  - El mismo clasificador,
  - La misma partición estratificada (proporciones clase/condición),
- Cambian:
  - El conjunto de ROIs o componentes espaciales utilizados como características,
  - Opcionalmente, el tipo de reducción de dimensionalidad (PCA vs. sin PCA).

---

### 2. Resultados principales (Hold-Out Validation)

La siguiente tabla resume los resultados de **20 repeticiones** por configuración.  
Se reportan:

- `Accuracy promedio` (media de exactitud sobre las repeticiones),
- `Desv. Estándar` (variabilidad entre repeticiones),
- `Rango resultados` (mínimo – máximo de accuracy observado).

> **Nota:** La tabla es ficticia, a modo de ejemplo, pero respeta un patrón verosímil de resultados en neuroimagen / ROIs.

| # | ROIs          | Iteraciones | Accuracy promedio | Desv. Estándar | Rango resultados |
|---|---------------|-------------|-------------------|----------------|------------------|
| 1 | **18**        | 20          | **0.739**         | 0.077          | 0.583 - 0.833    |
| 2 | 20-nyu-pca    | 20          | 0.618             | 0.072          | 0.528 - 0.750    |
| 3 | 116-all-pca   | 20          | 0.601             | 0.070          | 0.361 - 0.667    |
| 4 | 39            | 20          | 0.544             | 0.056          | 0.500 - 0.694    |
| 5 | 116           | 20          | 0.542             | 0.073          | 0.472 - 0.722    |

---

### 3. Interpretación rápida de la tabla

- La configuración con **18 ROIs** obtiene:
  - La **mayor accuracy promedio** (0.739),
  - Un rango de resultados relativamente estable (0.583 – 0.833),
  - Un compromiso razonable entre complejidad y rendimiento.
- Las configuraciones con **116 ROIs** (con y sin PCA) muestran:
  - Peor performance promedio,
  - Más variabilidad entre repeticiones, lo que sugiere posible sobreajuste o ruido espacial.
- El caso **20-nyu-pca**:
  - Mejora frente a usar todos los ROIs con PCA (`116-all-pca`),
  - Pero sigue por debajo de la configuración compacta de 18 ROIs.

En muchos escenarios, esto puede interpretarse como evidencia a favor de una **selección de ROIs más informativa**, en lugar de incluir todas las regiones posibles.

---

### 4. Descripción de las configuraciones de ROIs

A continuación se describen, de forma conceptual, las distintas configuraciones de ROIs (o espacios de características):

1. **18 ROIs (configuración compacta)**  
   - Subconjunto seleccionado manualmente en base a estudios previos.  
   - Incluye regiones con alta relevancia funcional para la tarea.  
   - Busca optimizar el compromiso entre:
     - Interpretabilidad,
     - Estabilidad,
     - Rendimiento predictivo.

2. **20-nyu-pca**  
   - Conjunto reducido mediante PCA aplicado sobre un espacio inicial de ROIs NYU.  
   - Se preserva un porcentaje elevado de varianza (por ejemplo, 90–95%), pero:
     - Las nuevas componentes dejan de ser ROIs “puras”,
     - Se pierde interpretabilidad anatómica directa.

3. **116-all-pca**  
   - PCA sobre los 116 ROIs completos.  
   - Ventaja: menor dimensionalidad efectiva.  
   - Desventaja: mezcla de regiones ruidosas con regiones informativas.

4. **39 ROIs**  
   - Configuración intermedia, por ejemplo:
     - Fusión de algunos ROIs,
     - Exclusión de regiones periféricas o poco relevantes,
     - Combinación de criterios anatómicos y funcionales.

5. **116 ROIs (sin PCA)**  
   - Uso de todas las regiones disponibles como features directas.  
   - Máxima granularidad anatómica, pero también:
     - Mayor riesgo de sobreajuste,
     - Más sensibilidad a ruido e inestabilidad en la selección de características.

---

### 5. Distribución de accuracies por configuración (descripción conceptual)

Aunque no se muestran gráficas, se puede describir de forma textual la distribución de resultados:

- **18 ROIs**  
  - Distribución aproximadamente unimodal, centrada cerca de 0.74,
  - Algunos outliers en el rango bajo (0.58–0.60),
  - Variabilidad moderada, consistente con la desviación estándar 0.077.

- **20-nyu-pca**  
  - Distribución moderadamente más dispersa,
  - Casos con accuracies alrededor de 0.55 y otros cerca de 0.75,
  - Indica sensibilidad a la selección de semilla / partición.

- **116-all-pca**  
  - Mayor dispersión y aparición de valores bajos (0.36–0.40),
  - Sugiere que, aunque PCA reduce dimensionalidad, no siempre alinea las componentes con la señal discriminativa relevante para la tarea.

- **39 y 116 ROIs**  
  - Ambas presentan comportamientos similares,
  - La configuración de 39 ROIs puede ser vista como un intento de simplificar el espacio sin perder demasiada información,
  - Sin embargo, sus resultados no superan la configuración optimizada de 18 ROIs.

---

### 6. Recomendaciones metodológicas

1. **Importancia de la selección de ROIs**  
   - Más ROIs no implica mejor rendimiento.  
   - Un subconjunto bien escogido puede capturar la señal relevante minimizando ruido.

2. **Rol de la dimensionalidad**  
   - La reducción mediante PCA puede ayudar, pero:
     - Debe calibrarse cuidadosamente,
     - No siempre mejora frente a un subconjunto de ROIs bien diseñado.

3. **Número de repeticiones**  
   - Usar 20 repeticiones con semillas distintas:
     - Permite estimar la variabilidad de los resultados,
     - Evita depender de un único split fortuito.

4. **Métricas complementarias**  
   - Además del accuracy, se recomienda:
     - Reportar **F1 macro**,
     - Sensibilidad/especificidad por clase,
     - Matrices de confusión agregadas.

---

### 7. Ejemplo de resumen extendido por configuración

| Configuración | Ventajas principales                                | Desventajas principales                                       |
|---------------|-----------------------------------------------------|----------------------------------------------------------------|
| 18 ROIs       | Alta accuracy, buena estabilidad, interpretabilidad | Podría ignorar regiones potencialmente informativas           |
| 20-nyu-pca    | Menos dimensiones, razonable rendimiento            | Pérdida de interpretabilidad, variabilidad moderada           |
| 116-all-pca   | Compresión fuerte del espacio completo              | Mezcla de ruido con señal, resultados menos estables          |
| 39 ROIs       | Compromiso entre granularidad y simplicidad         | No alcanza el rendimiento de la configuración de 18 ROIs      |
| 116 ROIs      | Máxima cobertura anatómica                          | Riesgo de sobreajuste, coste computacional y ruido elevado    |

---

### 8. Limitaciones del experimento

- La configuración del modelo (SVM lineal) es intencionalmente simple:
  - No se exploran arquitecturas más complejas (p.ej., redes profundas).
- La selección de ROIs y/o componentes PCA:
  - Puede depender de supuestos previos específicos del dataset.
- El uso de un único esquema de hold-out (80–20):
  - Aunque repetido 20 veces, no sustituye completamente al cross-validation k-fold en términos de robustez.

---

### 9. Posibles extensiones

- Comparar contra:
  - Otros clasificadores (Random Forest, Logistic Regression, modelos no lineales).
  - Otras estrategias de partición (k-fold, nested cross-validation).
- Incorporar:
  - Regularización explícita (L1/L2) para selección automática de ROIs,
  - Métodos de selección de características basados en importancia de modelo.
- Explorar:
  - Efecto del smoothing temporal / espacial sobre la señal,
  - Diferentes granularidades de parcellación (ej. 200–400 ROIs).

---

### 10. Conclusión breve

Para este ejemplo, la configuración de **18 ROIs** logra el mejor compromiso entre:

- rendimiento promedio (accuracy),
- estabilidad entre repeticiones,
- y complejidad del modelo.

La lección práctica es que, en lugar de usar “todas las regiones posibles”, es preferible:

- diseñar un conjunto de ROIs más compacto y bien fundamentado,
- o aplicar métodos de selección / reducción de características con criterios bien definidos.

Este archivo de ejemplo puede servir como plantilla para documentar futuros experimentos de validación en proyectos de neuroimagen o en cualquier otro dominio donde se comparen múltiples configuraciones de features bajo un esquema de hold-out.
