# Piano Virtual con la cámara (versión web) 🎹

Página web para tocar un piano con los dedos frente a la cámara, al estilo Kinect, y aprender **Jingle Bells**.
Usa **MediaPipe Hand Landmarker** en el navegador para detectar los 21 puntos de cada mano.

## Un dedo = una tecla
Cada dedo tiene su propia tecla (posición de **Do** del piano). La nota suena al **doblar ese dedo** hacia abajo, como si presionaras una tecla real.

| Mano izquierda | Meñique | Anular | Medio | Índice | Pulgar |
|---|---|---|---|---|---|
| Nota | Do3 | Re3 | Mi3 | Fa3 | Sol3 |

| Mano derecha | Pulgar | Índice | Medio | Anular | Meñique |
|---|---|---|---|---|---|
| Nota | Do4 | Re4 | Mi4 | Fa4 | Sol4 |

Jingle Bells se puede tocar entera con **la mano derecha**: en la canción, Mi es el medio, Sol el meñique, Do el pulgar, Re el índice y Fa el anular.
En modo guiado también vale tocarla con la mano izquierda (la misma nota, una octava más abajo).

## Cómo abrirla
- **Opción 1:** doble clic en `iniciar_servidor.bat` (necesita Python). Abre `http://localhost:8000` en el navegador.
- **Opción 2:** abre `index.html` directamente con **Chrome o Edge**.

Luego pulsa **Activar cámara** y acepta el permiso. Necesita internet la primera vez, para cargar MediaPipe y el modelo de manos.

## Controles
- **Modo guiado:** la tecla que sigue brilla en verde, el dedo correspondiente tiene un anillo verde en la cámara y arriba aparece la letra de la canción.
- **Modo libre:** tocas lo que quieras.
- **Demostración:** el piano toca Jingle Bells solo.
- **⇄ Manos (intercambiar):** úsalo si la app confunde tu mano izquierda con la derecha.
- **☝ 1 tecla a la vez** (activado por defecto): mientras un dedo tiene una tecla presionada, ningún otro dedo de ninguna mano suena. Si dos dedos se doblan al mismo tiempo, solo suena el más doblado. Para tocar la siguiente nota hay que estirar primero el dedo anterior. Si lo desactivas, puedes tocar acordes.
- **Calibrar:** pulsa este botón al empezar, con las manos relajadas y los dedos estirados.
- **Sensibilidad:** si suenan dedos de más, bájala; si cuesta que suene, súbela.
- También se puede tocar con el mouse o con el teclado: `A S D F G` (mano izquierda) y `H J K L Ñ` (mano derecha).

## Rendimiento (máximos FPS)
Arriba a la izquierda se ve en vivo: **FPS de la cámara**, **FPS de la detección**, milisegundos por fotograma, la resolución y si el modelo corre en **GPU** o en CPU.

Qué hace la app para sacar el máximo de fotogramas:
- Le pide a la cámara **la tasa de fotogramas más alta que soporte** (lee sus capacidades y aplica el máximo; 60 fps si la cámara puede).
- Por defecto usa **640×360**: a menor resolución, las cámaras dan más FPS y MediaPipe procesa más rápido. Desde el selector puedes cambiar a 960×540 o 1280×720.
- Analiza **cada fotograma nuevo apenas llega** (`requestVideoFrameCallback`), sin esperar al refresco de la pantalla.
- El navegador muestra el video directamente; el canvas solo dibuja las manos (no copia la imagen en cada fotograma).
- El modelo corre en la **GPU** (WebGL); si no se puede, pasa a CPU.
- Audio en modo de baja latencia y actualizaciones mínimas de la página.

Consejos: con **buena luz** la cámara no baja los FPS (con poca luz, Windows alarga la exposición y baja a ~15 fps). Usa Chrome o Edge con la aceleración por hardware activada. La mayoría de las cámaras de portátil llegan a 30 fps como máximo; las externas pueden dar 60.

## Cómo detecta la pulsación de cada dedo
1. **Puntos 3D reales:** usa los `worldLandmarks` de MediaPipe (en metros y centrados en la mano). Por eso la medida no cambia si giras la mano, la acercas o la alejas.
2. **Flexión en grados:** para cada dedo suma los ángulos de sus 3 articulaciones (0° = dedo recto). En el pulgar se suman sus 2 articulaciones y qué tanto se acerca la yema al centro de la palma.
3. **Filtro One Euro:** quita el temblor natural de la mano sin retrasar los golpes rápidos.
4. **Calibración:** al aparecer la mano se toma su postura relajada como base. Con el botón **🖐 Calibrar** se mide 1,5 s y se usa la mediana. Después, la base se ajusta sola, pero solo mientras el dedo está quieto.
5. **Golpe:** suena cuando la flexión supera el umbral (40° por defecto; la *Sensibilidad* lo mueve entre 65° y 20°). Si el dedo baja muy rápido (más de 350°/s), suena desde el 60 % del umbral, así responde antes.
6. **Histéresis:** para volver a tocar, el dedo tiene que estirarse por debajo del 45 % del umbral. Así no suenan notas dobles.
7. **Dedos arrastrados:** al doblar el meñique, el anular también se mueve un poco. Un dedo solo suena si se dobla al menos 1,3 veces más que sus vecinos. Si hay duda, espera como máximo 45 ms y elige el más doblado. Los vecinos quedan bloqueados hasta que se estiren.

Pruebas con una mano 3D simulada (temblor de ±2°, dedos vecinos arrastrados al 55 %):
- 3 s de reposo con temblor: 0 notas falsas.
- Giros de la mano de hasta 25°: 0 notas falsas.
- 26 notas de Jingle Bells: 26/26 correctas.
- Golpes rápidos de 3 fotogramas: todos detectados.

El CSV ahora incluye la columna `flexion_grados`: cuántos grados se dobló el dedo en cada golpe.

## Parte de Ciencia de datos
Con el botón **⬇ CSV** se baja un archivo con: tiempo, modo, mano, dedo, nota tocada, nota esperada y si fue correcta.
Guarda los CSV en la carpeta `registros/` y ejecuta:
```bash
pip install pandas matplotlib
python analizar_sesiones.py
```
Así ves la precisión por sesión, las notas donde más te equivocas, los dedos más usados y una gráfica (`analisis_sesiones.png`).

## Archivos
- `index.html`: la aplicación web (todo en un solo archivo).
- `iniciar_servidor.bat`: la abre en un servidor local.
- `analizar_sesiones.py`: análisis de los CSV de las sesiones.
- `piano_virtual.py`: versión anterior en Python/OpenCV (opcional).
