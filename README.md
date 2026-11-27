# Visión por Computadora

**Alumno:** Gonzalo Alvarez Campos

Evaluación Integradora de la materia Visión por Computadora.

La Parte 1 diseña e implementa un pipeline de visión por computadora que detecta **vehículos blancos** en un video real de tránsito (cámara frontal en autopista, 1280×720, 25 FPS, 50,4 s) y registra cada aparición con frame y timestamp. El pipeline recorta la región de interés y la divide en tres recortes solapados. Detecta vehículos con un modelo preentrenado (MobileNet-SSD con el módulo DNN de OpenCV) y decide el color segmentando la carrocería en HSV, con suavizado gaussiano y apertura morfológica. Después convierte las detecciones en eventos con seguimiento por IoU, una persistencia de 3 frames y una ventana de 4 s entre eventos. Se compararon dos pipelines de color (RGB vs. HSV) y tres umbrales de confianza (0,25 / 0,50 / 0,75), y se eligió 0,75.

Resultado: se registró **1 aparición de vehículo blanco** (frames 132 a 1185, de 00:05.28 a 00:47.40) y 4 eventos en total. La revisión manual dio los 4 eventos como correctos, sin falsos positivos de color. El auto blanco tiene una cobertura de 88,7 % de los frames en que es visible.

## Estructura del repositorio

| Archivo | Descripción |
|---|---|
| `parte1_colab.ipynb` | Notebook de la Parte 1 (Colab). Incluye inspección del video, preprocesamiento justificado, detección y segmentación por color, reglas temporales, registro de eventos en CSV, evidencia visual y validación manual. |
| `Parte 1 - Informe.pdf` | Informe de la Parte 1, con las cifras del notebook y citas por página al material de la materia. |
| `README.md` | Este archivo. |

## Cómo ejecutar

El notebook está pensado para **Google Colab**. También corre en Jupyter local con Python 3.

1. Abrir el notebook en Colab.
2. Ejecutar todas las celdas (*Entorno de ejecución → Ejecutar todas*). La corrida completa tarda unos 2–3 minutos.

La primera celda prepara el entorno:

- Verifica que OpenCV sea 4.x, porque OpenCV 5 ya no incluye `cv2.dnn.readNetFromCaffe`. Si hace falta, instala `opencv-python-headless<5` y pide reiniciar la sesión.
- Descarga el video y el modelo desde sus repositorios públicos: `project_video.mp4` de [udacity/CarND-Vehicle-Detection](https://github.com/udacity/CarND-Vehicle-Detection) y MobileNet-SSD de [chuanqi305/MobileNet-SSD](https://github.com/chuanqi305/MobileNet-SSD), ambos con licencia MIT. Los guarda en `data/` y `modelos/`, que no están incluidas en el repo.
- Si no hay acceso a internet, hay que subir `data/video_transito.mp4`, `modelos/deploy.prototxt` y `modelos/mobilenet_iter_73000.caffemodel` al panel de archivos de Colab. En local, se dejan en esas mismas carpetas junto al notebook.

Las salidas se guardan en `resultados/`:

- `eventos_vehiculos_blancos.csv`, `eventos_todos.csv` y `detecciones.csv`
- `parametros.json` e `indicadores_validacion.csv`
- `video_anotado.mp4` y las imágenes de `evidencias/`

Dependencias: `opencv-python` (4.x), `numpy`, `pandas`, `matplotlib`. Todas vienen instaladas en Colab; `ffmpeg` es opcional y solo se usa para recodificar el video anotado.

## Uso de herramientas de IA

Este trabajo se desarrolló con la ayuda de Claude (Anthropic), un asistente de IA. El autor resolvió el examen, eligió el caso de aplicación (video de tránsito y vehículos blancos) y revisó y aprobó cada paso. El asistente colaboró en la implementación del notebook y en la ejecución y ajuste de parámetros (umbral de confianza y ventana temporal). También colaboró en la revisión manual de los eventos y en la redacción del informe, bajo la supervisión del autor, tomando como referencia el material de la materia.
