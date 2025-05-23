# Depix

Depix es una prueba de concepto (PoC) de una técnica para recuperar texto plano de capturas de pantalla pixeladas.

Esta implementación funciona con imágenes pixeladas creadas con un filtro de caja lineal.
En [este artículo](https://www.spipm.nl/2030.html) explico la información de fondo sobre la pixelación e investigaciones similares.

## Ejemplo

![image](docs/img/Recovering_prototype_latest.png)

## Instalación

* Instalar las dependencias
* Ejecutar Depix:

```sh
python3 depix.py \
-p /path/to/your/input/image.png \
-s images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png \
-o /path/to/your/output.png
```

## Ejemplo de uso

* Depixelizar la imagen de ejemplo creada con el Bloc de notas y pixelada con Greenshot. Greenshot promedia los valores de 0 a 255 con codificación gamma, que es el modo predeterminado de Depix.

```sh
python3 depix.py \
-p images/testimages/testimage3_pixels.png \
-s images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png
```

Resultado: ![image](docs/img/example_output_multiword.png)

* Despixelizar la imagen de ejemplo creada con Sublime y pixelada con Gimp, donde el promedio se realiza en sRGB lineal. La opción backgroundcolor filtra el color de fondo del editor.

```sh
python3 depix.py \
-p images/testimages/sublime_screenshot_pixels_gimp.png \
-s images/searchimages/debruin_sublime_Linux_small.png \
--backgroundcolor 40,41,35 \
--averagetype linear
```

Resultado: ![image](docs/img/output_depixelizedExample_linear.png)

* (Opcional) Puedes comprobar si el detector de cajas encuentra tus píxeles con `tool_show_boxes.py`. Considera usar un lote más pequeño de píxeles si la imagen se ve distorsionada. Ejemplo de cuadros con buen aspecto:

```sh
python3 tool_show_boxes.py \
-p images/testimages/testimage3_pixels.png \
-s images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png
```

* (Opcional) Puedes crear una imagen pixelada usando `tool_gen_pixelated.py`.

```sh
python3 tool_gen_pixelated.py -i /path/to/image.png -o pixed_output.png
```

* Para obtener una explicación detallada, intenta ejecutar `$ python3 depix.py -h` y `tool_gen_pixelated.py`.

## Acerca de

### Crear una imagen de búsqueda

* Recorta los bloques pixelados de la captura de pantalla como un solo rectángulo. * Pegue una [secuencia De Bruijn](https://en.wikipedia.org/wiki/De_Bruijn_sequence) con los caracteres esperados en un editor con la misma configuración de fuente que la imagen de entrada (mismo tamaño de texto, fuente similar, mismos colores).
* Haga una captura de pantalla de la secuencia.
* Mueva esa captura de pantalla a una carpeta como `images/searchimages/`.
* Ejecute Depix con el indicador `-s` configurado en la ubicación de esta captura de pantalla.

### Creación de una imagen pixelada

* Recorte los bloques pixelados con exactitud. Consulte `testimages` para ver ejemplos.
* Intenta detectar bloques, pero no lo hace de maravilla. Experimente con el script `tool_show_boxes.py` y diferentes recortes si los bloques no se detectan correctamente.

### Algoritmo

El algoritmo aprovecha que el filtro de caja lineal procesa cada bloque por separado. Para cada bloque, pixela todos los bloques de la imagen de búsqueda para buscar coincidencias directas.

Para algunas imágenes pixeladas, Depix logra encontrar resultados de coincidencia única. Los asume como correctos. Las coincidencias de los bloques circundantes con coincidencias múltiples se comparan para determinar si están geométricamente a la misma distancia que en la imagen pixelada. Las coincidencias también se consideran correctas. Este proceso se repite un par de veces.

Una vez que los bloques correctos ya no tienen coincidencias geométricas, se generan todos los bloques correctos directamente. Para los bloques con coincidencias múltiples, se genera el promedio de todas las coincidencias.

### Limitaciones conocidas

* El algoritmo realiza la coincidencia mediante límites de bloque enteros. Como resultado, asume que, para todos los caracteres renderizados (tanto en la secuencia de De Brujin como en la imagen pixelada), el posicionamiento del texto se realiza a nivel de píxel. Sin embargo, algunos rasterizadores de texto modernos posicionan el texto con precisiones de subpíxel (http://agg.sourceforge.net/antigrain.com/research/font_rasterization/).
* Es necesario conocer las especificaciones de la fuente y, en algunos casos, la configuración de pantalla con la que se tomó la captura de pantalla. Sin embargo, si hay suficiente texto simple en la imagen original, es posible que puedas...

Usar la imagen original como imagen de búsqueda.
* Este enfoque no funciona si se realiza una compresión de imagen adicional, ya que altera los colores de un bloque.

### Desarrollo futuro

* Implementar más funciones de filtro

Crear más filtros de promedio que funcionen como algunos editores populares.

* Crear una nueva herramienta que utilice HMM

Aun así, se anima a cualquier persona interesada en este tipo de despixelización a implementar su propia versión basada en HMM y compartirla.
