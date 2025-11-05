# Fusión de Perspectivas: Registro y Medición Métrica en una Escena Real

**Curso:** Visión por Computador (3009228)  
**Semestre:** 2025-02  
**Facultad de Minas — Universidad Nacional de Colombia**  
**Departamento de Ciencias de la Computación y de la Decisión**  
**Autor:** [Tu nombre aquí]  

---

## 🟩 Introducción

El registro de imágenes es una tarea fundamental en visión por computador, cuyo propósito es alinear diferentes vistas de una misma escena para obtener una representación coherente y continua. Este proceso es esencial en aplicaciones como la fotogrametría, la reconstrucción 3D, el mapeo y la medición de objetos en el mundo real.

En este trabajo se implementa un pipeline de **registro y fusión de imágenes** empleando descriptores locales (SIFT, ORB y AKAZE), estimación de **homografías mediante RANSAC**, y técnicas de blending para lograr un panorama unificado. Posteriormente, a partir de la escena fusionada, se realiza una **calibración métrica** usando objetos de referencia para estimar distancias reales dentro del entorno.

El proyecto combina validación con imágenes sintéticas —para evaluar la robustez geométrica del método— y aplicación práctica en un conjunto de fotografías de un comedor real, donde se cuantifican errores, se analizan resultados y se discuten las limitaciones del método.

---

## 🧠 Marco Teórico

### Registro de Imágenes
El registro consiste en encontrar una transformación geométrica que alinee dos imágenes capturadas desde diferentes puntos de vista. En este caso, se asume que las vistas pertenecen a un mismo plano, por lo que la transformación se modela mediante una **homografía 3×3**.

### Detección y Descripción de Características
Los detectores y descriptores locales permiten identificar puntos de interés invariantes ante cambios de escala, rotación e iluminación:
- **SIFT (Lowe, 2004)**: robusto a rotaciones y escalas.
- **ORB (Rublee et al., 2011)**: eficiente y rápido, basado en BRIEF binario.
- **AKAZE (Alcantarilla et al., 2013)**: balance entre robustez y velocidad mediante detección en el espacio no lineal.

### Emparejamiento y Filtrado
Los descriptores se comparan usando métricas de distancia (L2 o Hamming) y se filtran con el **ratio test** (Lowe, 2004). Para reducir falsos emparejamientos se aplica **validación cruzada** y posteriormente **RANSAC** (Fischler & Bolles, 1981) para estimar la homografía rechazando outliers.

### Fusión y Blending
Una vez estimada la transformación, las imágenes se combinan mediante interpolación y blending (por ejemplo, *feather blending* o pirámides Laplacianas) para generar un mosaico continuo.

### Calibración Métrica
Al identificar objetos de dimensiones conocidas dentro de la imagen fusionada, se establece una relación **píxeles ↔ centímetros**, permitiendo estimar medidas reales a partir del modelo 2D registrado.

---

## 🧭 Metodología

El desarrollo siguió un flujo modular documentado en tres notebooks principales:

1. `01_exploratory_analysis.ipynb` – Análisis exploratorio y comparación de detectores.  
2. `02_synthetic_validation.ipynb` – Validación geométrica con imágenes sintéticas.  
3. `03_main_pipeline.ipynb` – Registro, fusión, calibración y medición en la escena real.

### Pipeline de procesamiento

1. **Extracción de características** con SIFT, ORB y AKAZE.  
2. **Emparejamiento de descriptores** mediante BFMatcher o FLANN con *ratio test*.  
3. **Estimación de homografía** con RANSAC.  
4. **Fusión y blending** de las imágenes alineadas.  
5. **Calibración métrica** con referencias físicas (cuadro y mesa).  
6. **Medición y análisis de incertidumbre.**

### Decisiones técnicas
- **SIFT** se usó como detector principal por su robustez.  
- Umbral de *ratio test* = 0.75.  
- RANSAC: 2000 iteraciones, umbral 5 px.  
- Blending: método *feather* para suavizar transiciones.  
- Escala calibrada: 9.32 píxeles/cm.  
- Referencias: cuadro (117 cm) y mesa (161.1 cm).  

### Diagrama de flujo

```mermaid
flowchart LR
    A[Imágenes de entrada] --> B[Detección de características]
    B --> C[Emparejamiento robusto (RANSAC)]
    C --> D[Fusión de imágenes]
    D --> E[Panorama final]
    E --> F[Calibración métrica]
    F --> G[Mediciones y análisis]
```

---

## 🧪 Resultados

### Imágenes de entrada
![Imágenes originales](01_original_images.jpeg)

### Comparación de detectores
![Comparación de detectores](02_detector_comparison.jpeg)

### Distribución de keypoints
![Distribución de keypoints](04_keypoint_distribution.jpeg)

### Emparejamientos válidos
![Matches 1-2](03_matches_img1_to_img2.jpeg)
![Matches 2-3](03_matches_img2_to_img3.jpeg)

### Validación con imágenes sintéticas
![Dataset sintético](05_synthetic_dataset.png)
![Errores de validación](06_validation_errors.png)
![Errores por esquina](07_corner_errors.png)

### Fusión de perspectivas
![Panorama final](08_panorama_final.jpeg)

### Calibración y medición
![Reporte de mediciones](09_measurement_report.png)

---

## 🔍 Discusión

- **Desempeño de detectores:** SIFT ofrece la mejor relación entre densidad y estabilidad; ORB destaca por eficiencia pero reduce correspondencias válidas.  
- **Precisión geométrica:** El modelo homográfico reconstruye correctamente el plano de la escena, con errores esperables por paralaje y variación de punto de vista.  
- **Blending:** El método *feather* logró una transición suave entre imágenes, sin artefactos notables.  
- **Medición:** La calibración manual permitió obtener escalas coherentes, aunque la incertidumbre indica sensibilidad a la selección de puntos.  
- **Limitaciones:** Las principales fuentes de error son la falta de control de iluminación, pequeñas deformaciones por perspectiva no coplanar y la ausencia de puntos de control automáticos.

---

## ✅ Conclusiones

1. El pipeline de registro basado en **SIFT + RANSAC + homografía** permitió fusionar exitosamente las tres imágenes del comedor en un panorama coherente.  
2. La **validación sintética** confirmó la precisión del modelo geométrico, con errores promedio aceptables.  
3. La **calibración métrica** demostró la viabilidad de estimar dimensiones reales a partir de una imagen registrada.  
4. Se identificaron oportunidades de mejora mediante la incorporación de **bundle adjustment** y métodos de calibración automática.  
5. El ejercicio permitió comprender integralmente las etapas del registro, desde la detección hasta la medición métrica aplicada.

---

## 📚 Referencias

- Lowe, D. G. (2004). *Distinctive Image Features from Scale-Invariant Keypoints*. International Journal of Computer Vision, 60(2), 91–110.  
- Hartley, R., & Zisserman, A. (2003). *Multiple View Geometry in Computer Vision* (2nd ed.). Cambridge University Press.  
- Fischler, M. A., & Bolles, R. C. (1981). *Random Sample Consensus: A Paradigm for Model Fitting with Applications to Image Analysis and Automated Cartography*. Communications of the ACM, 24(6), 381–395.  
- Rublee, E., Rabaud, V., Konolige, K., & Bradski, G. (2011). *ORB: An Efficient Alternative to SIFT or SURF*. IEEE International Conference on Computer Vision (ICCV), 2564–2571.  
- Alcantarilla, P. F., Nuevo, J., & Bartoli, A. (2013). *Fast Explicit Diffusion for Accelerated Features in Nonlinear Scale Spaces*. British Machine Vision Conference (BMVC).  

---

## 👥 Contribución Individual

| Integrante | Rol | Actividad principal |
|-------------|-----|---------------------|
| [Nombre A] | Desarrollo | Implementación del pipeline y scripts de validación |
| [Nombre B] | Análisis | Evaluación de detectores y redacción de resultados |
| [Nombre C] | Visualización | Diseño de figuras y reporte de mediciones |

---

**Repositorio del proyecto:** [https://github.com/TU_USUARIO/fusion-perspectivas](#)  
**Publicación:** [https://TU_USUARIO.github.io/fusion-perspectivas](#)
