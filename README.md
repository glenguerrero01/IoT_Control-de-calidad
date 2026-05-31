# IoT_Control-de-calidad --- Control de calidad con IA + Gemelo digital en Ubidots
Respositorio código actividad IoT control de calidad

Proyecto de la asignatura **IoT y gemelos digitales**. Se entrenó un modelo de visión por computadora en **Teachable Machine** para clasificar botellas como **OK (con tapa)** o **Defectuoso (sin tapa)**, y lo conecté a un **gemelo digital en Ubidots** mediante un notebook de Python que clasifica en tiempo real desde la webcam y envía el resultado a un dashboard con genmelo digital en Ubidots.
## Video funcionamiento
https://youtu.be/1_sOn2W7N7k
---

## Objetivo

Demostrar cómo la IA puede apoyar un control de calidad visual sin hardware complejo, e
integrar esa clasificación en una plataforma IoT que registre y visualice los datos en
tiempo real (trazabilidad, estadísticas y alertas).

---

## Producto elegido

Una **botella**. El estado "con tapa" se considera **Producto OK** y "sin tapa" se
considera **Producto Defectuoso**.

---

##  Teachable Machine (pasos 1–6 de la guía)

1. **Entrenamiento:** entré a Teachable Machine de Google y creé un proyecto de imagen.
2. **Clases:** definí dos clases: *Producto OK* (botella con tapa) y *Producto Defectuoso*
   (botella sin tapa).
3. **Captura:** con la webcam tomé más de 30 fotos por clase, moviendo y girando la botella
   para que el modelo viera distintos ángulos y posiciones.
4. **Enriquecimiento del dataset:** repetí capturas con fondo neutro y variando la
   iluminación para que el modelo fuera más robusto y no aprendiera el fondo en vez del objeto.
5. **Entrenamiento del modelo:** pulsé *Train Model* y dejé la pestaña abierta mientras
   aprendía los patrones visuales.
6. **Prueba (Preview):** puse la botella frente a la cámara y comprobé que la IA acertaba
   con un porcentaje de confianza alto, distinguiendo correctamente "con tapa" de "sin tapa".
   Donde el modelo dudaba, añadí más fotos y reentrené.

---

## Paso 7 (opcional): integración con el gemelo digital

Exporté el modelo y lo conecté a Ubidots con un notebook de Python:

- **Export Model → Tensorflow → Keras → Download.** Esto genera `keras_model.h5` y
  `labels.txt`.
- El notebook `control_calidad_ubidots.ipynb` carga el modelo, captura la webcam con OpenCV,
  clasifica cada frame y envía el estado a Ubidots cada pocos segundos.

---

## Estructura del proyecto

```
.
├── control_calidad_ubidots.ipynb   # Notebook principal (clasificación + envío a Ubidots)
├── keras_model.h5                  # Modelo exportado de Teachable Machine
├── labels.txt                      # Etiquetas de las clases
└── README.md
```

---

## Requisitos

- **Python 3.10 o 3.11** (importante, ver Solución de problemas).
- Webcam.
- Cuenta de Ubidots con un token de dispositivo.

Instalación de dependencias:

```bash
pip install "tensorflow==2.12.1" opencv-python pillow numpy requests
```

---

## Cómo ejecutarlo

1. Coloca `keras_model.h5` y `labels.txt` en una carpeta con ruta **sin acentos ni eñes ni
   espacios** (por ejemplo `C:\tm_modelo`).
2. En el notebook, ajusta la configuración:
   - `TOKEN` y `DEVICE_LABEL` de tu cuenta de Ubidots.
   - `MODELO_PATH` y `LABELS_PATH` apuntando a tus archivos.
   - `INDICE_CLASE_OK` según el orden de tu `labels.txt` (qué índice es la botella con tapa).
3. Ejecuta las celdas en orden.
4. Se abre una ventana con la webcam; muestra la botella y observa la clasificación.
   **Pulsa `q`** sobre la ventana para detener.

---

## Datos que se envían a Ubidots

| Variable | Significado |
|---|---|
| `estado` | 1 = OK (con tapa), 0 = Defectuoso (sin tapa) |
| `confianza` | % de confianza de la predicción |
| `porcentaje_defectuosos` | % acumulado de defectuosos detectados |

En el dashboard se pueden montar un indicador para `estado`, una gráfica de línea para
`porcentaje_defectuosos`, un gauge para `confianza`, y un evento de alerta cuando el
porcentaje de defectuosos supere un umbral.

---

## Solución de problemas

- **`OSError: No file or directory found`** → la ruta del modelo es incorrecta. Verifica con
  `os.getcwd()` y `os.listdir()` dónde está parado el notebook, y corrige `MODELO_PATH`.
- **`UnicodeDecodeError` al cargar el modelo** → TensorFlow en Windows no soporta rutas con
  acentos, eñes o caracteres especiales. Mueve los archivos a una ruta solo ASCII, o sin tildes
  (ej. `C:\tm_modelo`).
- **Error de deserialización al cargar el `.h5`** → tienes Keras 3 (TensorFlow ≥ 2.16). El
  `.h5` de Teachable Machine es formato Keras 2. Usa `tensorflow==2.12.1`, o instala
  `tf-keras` y añade `os.environ["TF_USE_LEGACY_KERAS"] = "1"` antes de importar keras.
- **La cámara no abre** → ejecuta el notebook en tu máquina local; `cv2.imshow` no funciona
  en entornos remotos como Colab.
- **Clasifica al revés** → cambia el valor de `INDICE_CLASE_OK` (0 ↔ 1).
- **Error 401 de Ubidots** → token incorrecto. **403** → el token no tiene permisos de escritura.

---

## Tecnologías

Teachable Machine · TensorFlow/Keras 2.12 · OpenCV · Python · Ubidots (API REST v1.6).
