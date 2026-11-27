# Visión por Computadora

**Alumno:** Gonzalo Alvarez Campos

Evaluación Integradora de la materia Visión por Computadora.

La Parte 1 diseña e implementa un pipeline de visión por computadora que detecta **vehículos blancos** en un video real de tránsito (cámara frontal en autopista, 1280×720, 25 FPS, 50,4 s) y registra cada aparición con frame y timestamp. El pipeline recorta la región de interés y la divide en tres recortes solapados. Detecta vehículos con un modelo preentrenado (MobileNet-SSD con el módulo DNN de OpenCV) y decide el color segmentando la carrocería en HSV, con suavizado gaussiano y apertura morfológica. Después convierte las detecciones en eventos con seguimiento por IoU, una persistencia de 3 frames y una ventana de 4 s entre eventos. Se compararon dos pipelines de color (RGB vs. HSV) y tres umbrales de confianza (0,25 / 0,50 / 0,75), y se eligió 0,75.

Resultado: se registró **1 aparición de vehículo blanco** (frames 132 a 1185, de 00:05.28 a 00:47.40) y 4 eventos en total. La revisión manual dio los 4 eventos como correctos, sin falsos positivos de color. El auto blanco tiene una cobertura de 88,7 % de los frames en que es visible.

## Video de presentación

[Ver la presentación en Loom](https://www.loom.com/share/2b96e4e3814b4282adc066db50711815)

## Parte 2: entrenamiento, integración y mejora

La Parte 2 está al final del mismo notebook, a partir de la sección *Parte 2 – Entrenamiento, integración y mejora*. Entrena un **clasificador vehículo / no vehículo** con *transfer learning* (MobileNetV2, Keras) y lo agrega al pipeline de la Parte 1 como segundo filtro de las cajas del detector.

- **Dataset:** imágenes de vehículos y no vehículos de Udacity (GTI y KITTI). Se deja afuera la carpeta `Extras`, recortada del mismo video, para evitar *data leakage*. El split es 70/15/15 por bloques de cada secuencia, no al azar. Además se prueba con las 1.685 cajas que el detector encuentra en el video, etiquetadas a mano.
- **Dos configuraciones:** A, red base congelada, y B, *fine-tuning* de las últimas 30 capas. Las dos dan 99,9 % en el test del dataset. En el video conservan más del 99 % de los autos, pero eliminan solo alrededor del 40 % de las cajas falsas, por cambio de dominio. Se elige B, que elimina apenas más falsas.
- **Pipeline final:** video → detector (confianza ≥ 0,5) → clasificador (p ≥ 0,5) → color HSV → seguimiento y eventos → CSV y video anotado. Los umbrales se eligieron con la primera mitad del video. Conserva el 96,6 % de las cajas de autos, contra el 93,2 % de la Parte 1, y registra los mismos 4 eventos, todos correctos.
- **Mejora prioritaria:** agregar al entrenamiento imágenes "no vehículo" de este tipo de ruta (arbustos, carteles) y volver a medir.

## Estructura del repositorio

| Archivo o carpeta | Descripción |
|---|---|
| `evaluacion_integradora_vc.ipynb` | Notebook de Colab con las dos partes, ya ejecutado. |
| `data/` | Video de entrada (`project_video.mp4` de udacity/CarND-Vehicle-Detection, licencia MIT). La Parte 2 descarga además acá el dataset de Udacity, de unos 125 MB. |
| `modelos/` | Detector MobileNet-SSD en formato Caffe (chuanqi305/MobileNet-SSD, licencia MIT). |
| `resultados_parte1/` | Salidas de la Parte 1: eventos y detecciones en CSV, parámetros, indicadores de validación, video anotado y evidencias. |
| `resultados_parte2/` | Salidas de la Parte 2: comparación A/B, eventos del pipeline final, detecciones con la probabilidad del clasificador, split del dataset, matrices de confusión, errores y video anotado. Los modelos entrenados no se suben porque el notebook los regenera en unos minutos. |

## Cómo ejecutar

1. Abrir `evaluacion_integradora_vc.ipynb` en Google Colab (botón al principio del notebook). Para la Parte 2 conviene usar GPU (*Entorno de ejecución → Cambiar tipo de entorno → T4*).
2. Subir las carpetas `data/` y `modelos/` al panel de archivos de Colab, o clonar el repo en la sesión con `!git clone https://github.com/Alvacampos/VC---vision_por_computadora` y entrar con `%cd VC---vision_por_computadora`.
3. Ejecutar todas las celdas (*Entorno de ejecución → Ejecutar todas*). La Parte 1 tarda unos 3 minutos y la Parte 2 unos 5 más con GPU. La Parte 2 usa variables de la Parte 1, así que hay que ejecutar las dos en orden.

La primera celda instala OpenCV 4.10, porque OpenCV 5 ya no incluye `cv2.dnn.readNetFromCaffe`. El resto de las dependencias (`numpy`, `pandas`, `matplotlib`, `tensorflow`, `scikit-learn`) ya vienen en Colab. `ffmpeg` solo se usa para recodificar los videos anotados.

## Uso de herramientas de IA

Este trabajo se desarrolló con la ayuda de Claude (Anthropic), un asistente de IA. El autor resolvió el examen, eligió el caso de aplicación (video de tránsito y vehículos blancos) y revisó y aprobó cada paso. El asistente colaboró en la implementación del notebook (en la Parte 2, también en el entrenamiento y la integración) y en la ejecución y ajuste de parámetros (umbral de confianza y ventana temporal). También colaboró en la revisión manual de los eventos y en la redacción del informe, bajo la supervisión del autor, tomando como referencia el material de la materia.
