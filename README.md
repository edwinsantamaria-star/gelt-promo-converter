# Gelt · Conversor de Imágenes Promo

Herramienta interna para preparar automáticamente las imágenes de producto que se usan en las promociones de la app Gelt (banners, catálogo de cashback, etc.).

## Qué resuelve

Hoy cada imagen de promo (fotos de producto que llegan de marcas/proveedores en formatos y tamaños dispares) se recorta y adapta a mano al formato que exige la app. Esta herramienta automatiza ese proceso: se sube la imagen original y se descarga ya lista para producción.

## Qué hace

1. **Elimina el fondo** de la imagen automáticamente (IA de segmentación, corre en el propio navegador).
2. **Redimensiona y centra** el producto a **600 × 700 px** (2x del formato estándar de la app, 300 × 350, para que se vea nítido en pantallas de alta densidad).
3. **Aplica nitidez** (unsharp mask) para compensar la pérdida de detalle del redimensionado.
4. **Exporta a WebP** con máxima calidad, listo para subir.
5. Soporta **conversión en lote** (varias imágenes a la vez, con estado individual por archivo).
6. **Nombra el archivo de salida** con el prefijo `ImgPromo_` seguido del nombre original.

## Control de calidad automático del recorte

Packagings blancos o crema sobre fondo de estudio blanco (p.ej. botes de cacao) son el caso difícil: la IA y el relleno de fondo pueden confundir zonas del producto con fondo y dejarlas transparentes ("mordiscos" en la tapa o los bordes). Para que no dependa de nadie, la herramienta lo detecta y lo corrige en cada conversión, sin pasos manuales:

1. Calcula la máscara con la estrategia estándar y mide su **muesca máxima**: la profundidad del mayor hueco entre la silueta recortada y su casco convexo, relativa al tamaño del producto (un recorte sano queda en torno al 2-3%; un mordisco como el de la tapa crema da cerca del 40%).
2. Si la muesca supera el 1,5%, prueba estrategias alternativas más estrictas con el fondo (y, con fondo blanco, una que ignora la IA).
3. Se queda con la estrategia más limpia, y solo cambia de la estándar si la mejora es clara (>30%), así los casos que ya funcionaban no cambian.
4. Descarta además islas sueltas (restos de degradado del fondo).

Si se aplica una estrategia distinta de la estándar, se indica en el estado ("Recorte corregido automáticamente…"). El detalle de cada intento queda en la consola del navegador (`[recorte] estrategia …`).

### Imágenes que ya vienen con fondo transparente
Si el borde de la imagen ya es transparente (PNG/WebP/AVIF recortados por el proveedor), la herramienta **no pasa la IA ni el relleno de fondo**: usa el alfa original. Solo limpia los píxeles "fantasma" (alfa muy bajo, 1-15, restos de sombra/halo del recorte original) que, al volverse opacos, aparecían como siluetas blancas alrededor del producto y agrandaban el recuadro de recorte (el producto salía más pequeño). Umbral: alfa < 32 se borra; se descartan además islas sueltas y se recorta al producto.

### Imágenes de prueba
(Carpeta `tests/` solo en el equipo local de desarrollo; no se publica en GitHub por contener imágenes de marcas.)

- `tests/originales/` y `tests/resultados_ok/`: tres vistas del bote Valor Cacao Plus (fondo blanco opaco, packaging crema), que reproducían el recorte que se comía la tapa.
- `tests/originales_cetaphil/`: cuatro packshots Cetaphil WebP 2000×2000 con fondo transparente y píxeles fantasma, que reproducían las siluetas blancas.

Tras tocar el algoritmo de recorte, convertir todas y comprobar sobre fondo oscuro que el producto sale entero y sin siluetas.

## Formatos de entrada aceptados

JPG · PNG · TIFF · BMP · GIF · WebP · AVIF · HEIC · PDF (se toma la primera página)

## Cómo se usa

1. Abrir la herramienta (ver "Cómo se ejecuta" abajo).
2. Arrastrar una o varias imágenes al recuadro (o seleccionarlas manualmente).
3. Pulsar **Convertir**. La primera vez tarda ~30s porque descarga el modelo de IA; las siguientes conversiones son casi instantáneas.
4. Revisar el preview (Original vs. Resultado 300×350 WebP).
5. Pulsar **Descargar** para obtener el/los archivo(s) `.webp` final(es).

## Cómo se ejecuta

Es una aplicación **100% frontend** (un único `index.html`, sin backend ni servidor de aplicación). Todo el procesamiento —eliminación de fondo, redimensionado, exportación— ocurre en el propio navegador del usuario; ninguna imagen se sube a ningún servidor.

- Ruta: `tools/promo-converter/index.html`
- Servidor de desarrollo local: `npx serve tools/promo-converter` (puerto 5555)
- No requiere instalación, base de datos ni credenciales.

## Stack técnico

- HTML/CSS/JS vanilla (sin frameworks ni build step)
- [`@imgly/background-removal`](https://github.com/imgly/background-removal-js) — eliminación de fondo por IA, cargada vía CDN bajo demanda
- [`UTIF`](https://github.com/photopea/UTIF.js) — soporte de archivos TIFF, cargado bajo demanda
- [`PDF.js`](https://mozilla.github.io/pdf.js/) — renderiza la primera página de un PDF como imagen, cargado bajo demanda
- Canvas API nativa del navegador para el resize progresivo y el sharpening

## Ventajas frente al proceso manual

| | Manual | Con la herramienta |
|---|---|---|
| Tiempo por imagen | Varios minutos (Photoshop/similar) | Segundos |
| Consistencia de formato | Depende de quién la edite | Siempre 300×350 WebP |
| Requiere software de diseño | Sí | No, solo el navegador |
| Privacidad | — | Sin subida a servidor, todo local |

## Uso para el equipo

URL pública (sin instalar nada): https://edwinsantamaria-star.github.io/gelt-promo-converter/

No hace falta revisar ni retocar nada a mano: la herramienta detecta el tipo de imagen y aplica sola la estrategia adecuada (ver "Control de calidad automático" e "Imágenes que ya vienen con fondo transparente"). Basta con subir la imagen y descargar el resultado.

## Historial de mejoras

| Fecha | Problema detectado | Mejora (automática, sin pasos manuales) |
|---|---|---|
| 2026-10-19 | Packagings blancos/crema sobre fondo blanco (bote Valor Cacao Plus): la tapa y los bordes quedaban transparentes ("mordiscos") | Fondo estimado con degradado, exige blanco casi puro, descarta islas y control de calidad de la silueta con estrategias alternativas |
| 2026-10-20 | Packshots que ya vienen con fondo transparente (Cetaphil WebP): siluetas fantasma blancas y producto más pequeño | Si el borde ya es transparente se usa el alfa original, se borra el alfa residual y se recorta al producto, sin pasar por la IA |

## Estado y siguientes pasos

Herramienta funcional y en uso interno, desplegada en GitHub Pages (repo `gelt-promo-converter`, rama `master`). Posibles mejoras futuras (no priorizadas):
- Ajuste de tamaño de salida configurable si en el futuro se necesitan otros formatos además de 300×350

Al corregir un nuevo caso: añadir la imagen a `tests/`, anotarlo en el historial de arriba y comprobar que las imágenes de prueba anteriores siguen saliendo bien.
