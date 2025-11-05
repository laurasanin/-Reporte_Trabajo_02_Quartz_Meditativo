

---

## 🧩 Análisis Detallado de Resultados

### 1. Calidad de las imágenes de entrada
Las tres fotos del comedor muestran un solapamiento adecuado, con variaciones notables en ángulo y punto de vista. 
La tercera imagen incluye elementos no coplanares (plantas, columnas), lo que limita la validez de una homografía global. 
Estas condiciones permiten una fusión coherente en la zona central (mesa, cuadro, ventanales) y errores esperables en periferias.

### 2. Desempeño de detectores
**SIFT** (≈61.9k keypoints) destaca por su cobertura uniforme y repetitividad; 
**AKAZE** (≈18.3k) logra buen equilibrio entre densidad y estabilidad; 
**ORB** (≈2k) es veloz pero sensible a iluminación y contraste. 
SIFT es el más adecuado para escenas estáticas con variaciones de rotación/escala, mientras que ORB podría usarse en aplicaciones en tiempo real.
Se recomienda:
- En ORB, aumentar `nfeatures` y activar `WTA_K=4` y `HARRIS_SCORE`.
- En SIFT, ajustar `contrastThreshold` (0.03–0.06) para filtrar keypoints poco definidos.

### 3. Matching y robustez geométrica
Los emparejamientos 1↔2 y 2↔3 alcanzan ~60 % de *inliers* tras RANSAC, mostrando coherencia geométrica.
Los *outliers* se concentran en objetos fuera del plano (plantas, sombras).
Se recomienda afinar:
- `ratio test` = 0.7–0.8 según textura repetitiva.
- Validación cruzada (*cross-check*).
- Umbral RANSAC entre 3–6 px para balancear precisión y estabilidad.

### 4. Validación sintética
El conjunto sintético (rotaciones ±15–20°, traslaciones ±50 px, escalas 0.95–1.05) verificó la precisión del pipeline.  
Resultados:
- RMSE ≈ 93 px (≈ 10 cm con escala 9.32 px/cm).
- *Inliers* ≈ 61.5 %.
- Error máximo por esquina: 180–236 px (~20–25 cm), concentrado en bordes derechos.
El error crece en zonas con baja densidad de puntos, consistente con la teoría de soporte geométrico parcial.

### 5. Fusión y blending
El mosaico resultante presenta continuidad central (mesa/cuadro/ventanas) y mínima distorsión lateral.
El método de *feather blending* suaviza las transiciones, pero persisten leves diferencias fotométricas.  
Posibles mejoras:
1. **Exposure compensation** para igualar luminancia entre imágenes.  
2. **Graph-cut seam finding** para optimizar ubicación de costuras.  
3. **Multiband blending** con pirámides Laplacianas en bordes de alto contraste.  
4. **Proyección cilíndrica** si se desea un panorama tipo 360°.

### 6. Calibración y medición
Referencias: cuadro (117 cm) y mesa (161.1 cm). Escala promedio: **9.32 px/cm**.  
Mediciones: 5 elementos con incertidumbre media **20.08 %**.
Fuentes de error:
- Selección manual de puntos.  
- No coplanaridad local.  
- Variación de iluminación.
Para reducir incertidumbre a 5–10 %:
- Incorporar más referencias métricas.  
- Repetir mediciones ≥ 5 veces y promediar.  
- Localizar bordes subpíxel con detección Canny + ajuste lineal.  
- Rectificar el plano mediante homografía antes de medir.

### 7. Diagnóstico resumido
| Tipo de error | Causa principal | Efecto | Mitigación |
|----------------|-----------------|---------|-------------|
| Geométrico | Rotaciones altas, objetos fuera de plano | Desalineación periférica | Bundle adjustment, modelos no lineales |
| Textural | Rejillas, patrones repetidos | Matches ambiguos | Ratio test bajo, validación cruzada |
| Fotométrico | Cambios de exposición | Costuras visibles | Exposure compensation, equalización |
| Humano | Selección manual | Alta dispersión | Mediciones repetidas, snap a bordes |

### 8. Interpretación física
Con la escala de 9.32 px/cm, un error de 93 px equivale a 10 cm; 
la incertidumbre del 20 % implica ±2 cm para medidas pequeñas (~10 cm) y ±20 cm para medidas grandes (~1 m).
Esto confirma que el modelo homográfico es válido para planos interiores, 
pero no adecuado para reconstrucción 3D o zonas con paralaje elevado.

### 9. Propuestas de mejora
1. Integrar *GraphCutSeamFinder* y *ExposureCompensator* del módulo **cv2.detail**.  
2. Añadir calibración radial del lente.  
3. Incorporar control automático de exposición en la captura.  
4. Ampliar validación sintética con ruido gaussiano y desenfoque para probar robustez.  
5. Implementar bundle adjustment en Python o COLMAP para análisis 3D futuro.

---
