# Gelt · Conversor de Imágenes Promo

Herramienta interna para preparar automáticamente las imágenes de producto que se usan en las promociones de la app Gelt (banners, catálogo de cashback, etc.).

## Qué resuelve

Hoy cada imagen de promo (fotos de producto que llegan de marcas/proveedores en formatos y tamaños dispares) se recorta y adapta a mano al formato que exige la app. Esta herramienta automatiza ese proceso: se sube la imagen original y se descarga ya lista para producción.

## Qué hace

1. **Elimina el fondo** de la imagen automáticamente (IA de segmentación, corre en el propio navegador).
2. **Redimensiona y centra** el producto a **300 × 350 px**, el formato estándar de la app.
3. **Aplica nitidez** (unsharp mask) para compensar la pérdida de detalle del redimensionado.
4. **Exporta a WebP** con máxima calidad, listo para subir.
5. Soporta **conversión en lote** (varias imágenes a la vez, con estado individual por archivo).

## Formatos de entrada aceptados

JPG · PNG · TIFF · BMP · GIF · WebP · AVIF · HEIC

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
- Canvas API nativa del navegador para el resize progresivo y el sharpening

## Ventajas frente al proceso manual

| | Manual | Con la herramienta |
|---|---|---|
| Tiempo por imagen | Varios minutos (Photoshop/similar) | Segundos |
| Consistencia de formato | Depende de quién la edite | Siempre 300×350 WebP |
| Requiere software de diseño | Sí | No, solo el navegador |
| Privacidad | — | Sin subida a servidor, todo local |

## Estado y siguientes pasos

Herramienta funcional y en uso interno. Posibles mejoras futuras (no priorizadas):
- Despliegue en una URL fija accesible sin arrancar servidor local
- Ajuste de tamaño de salida configurable si en el futuro se necesitan otros formatos además de 300×350
