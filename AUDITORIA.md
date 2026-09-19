# Auditoría y optimización de rendimiento — AstroBitácora

**Estudiante:** René Francisco Pacheco Araniva  
**URL publicada:** https://astro-performance-lab-steel.vercel.app/  
**Fecha de medición inicial:** 19 de septiembre de 2026 (hora no registrada en la evidencia inicial)  
**Fecha de medición final:** 19 de septiembre de 2026, 1:24 p. m. (GMT-6)

## 1. Medición inicial (antes)

| Indicador | Móvil | Escritorio | Observación |
| --- | ---: | ---: | --- |
| Desempeño | 83 | 99 | En móvil había más oportunidades de mejora. |
| LCP | 3.2 s | 0.6 s | El LCP móvil estaba por encima del objetivo de 2.5 s. |
| CLS | 0.0 | 0.0 | La página era visualmente estable. |
| INP | 202 ms | — | Aceptable, pero mejorable en móvil. |
| TBT | 320 ms | 70 ms | Existía trabajo bloqueante adicional en móvil. |
| Imagen hero original | 8.18 MB | 8.18 MB | JPEG de 3840 × 2160 px. |

### Hallazgos principales

1. **Imágenes demasiado pesadas.** La imagen hero original pesaba aproximadamente 8.18 MB y afectaba el LCP móvil.
2. **Galería cargada de inmediato.** Las imágenes que estaban fuera del primer viewport se descargaban al abrir la página.
3. **Script bloqueante.** `script.js` se cargaba sin `defer`, por lo que podía interrumpir el análisis inicial del documento.

## 2. Hipótesis

1. Si se entregan versiones más pequeñas y comprimidas del hero, el LCP y los bytes transferidos disminuirán porque no se descargará el JPEG original de 8.18 MB.
2. Si la galería usa `loading="lazy"`, la carga inicial será más rápida porque sus imágenes quedan fuera de la vista inicial.
3. Si el script usa `defer`, el navegador podrá analizar primero el HTML y disminuirá el trabajo bloqueante al inicio.

## 3. Cambios aplicados

| Cambio | Motivo | Archivos | Resultado esperado |
| --- | --- | --- | --- |
| `defer` en `script.js` | Evitar que el script bloquee el análisis del HTML. | `index.html` | Menor bloqueo del hilo principal. |
| `preconnect` a Cloudinary | Iniciar antes la conexión con el origen de las imágenes. | `index.html` | Menor latencia al solicitar imágenes. |
| Hero con `srcset` de 640, 960 y 1280 px | Descargar una versión acorde a la pantalla y no el original de 3840 px. | `index.html` | Menos bytes y mejor LCP. |
| `f_auto,q_auto:eco` en el hero | Solicitar formato compatible y compresión eficiente. | `index.html` | Reducción del peso de la imagen. |
| `fetchpriority="high"` en el hero | Priorizar el recurso visual principal. | `index.html` | Hero disponible antes para el LCP. |
| Hero mantenido sin `loading="lazy"` | El hero es visible al cargar y no debe retrasarse. | `index.html` | Evitar empeorar el LCP. |
| `loading="lazy"` en la galería | Posponer recursos fuera del primer viewport. | `index.html` | Menos descarga inicial. |
| `width`, `height` y `aspect-ratio` | Reservar espacio mientras carga cada imagen. | `index.html`, `styles.css` | Mantener CLS bajo. |
| Menor altura del hero | Mostrar el contenido principal con menos píxeles. | `styles.css` | Menor necesidad de imagen y mejor uso de pantalla. |

## 4. Medición final (después)

Lighthouse se ejecutó en una carga inicial, con Moto G Power emulado y limitación de red 4G lenta para móvil. En escritorio se usó la limitación personalizada indicada por Lighthouse.

### Móvil

| Indicador | Antes | Después | Diferencia |
| --- | ---: | ---: | ---: |
| Desempeño | 83 | 100 | +17 puntos |
| LCP | 3.2 s | 1.7 s | -1.5 s |
| CLS | 0.0 | 0.0 | Sin cambio |
| INP | 202 ms | No mostrado por Lighthouse | — |
| TBT | 320 ms | 0 ms | -320 ms |
| FCP | 2.4 s | 0.8 s | -1.6 s |
| Speed Index | No registrado | 2.2 s | — |
| Bytes transferidos | Hero original: 8.18 MB | No mostrado por Lighthouse | — |

### Escritorio

| Indicador | Antes | Después | Diferencia |
| --- | ---: | ---: | ---: |
| Desempeño | 99 | 100 | +1 punto |
| FCP | 0.5 s | 0.2 s | -0.3 s |
| LCP | 0.6 s | 0.4 s | -0.2 s |
| TBT | 70 ms | 0 ms | -70 ms |
| Speed Index | 0.7 s | 0.3 s | -0.4 s |
| CLS | 0.0 | 0.0 | Sin cambio |

## 5. Conclusión

El cambio con mayor impacto fue la optimización del hero: el recurso JPEG original, servido sin transformaciones, se sustituyó por variantes responsivas comprimidas. Como referencia, la variante de 640 px pesa aproximadamente 22 KB, la de 960 px aproximadamente 35 KB y la de 1280 px aproximadamente 46 KB en la comprobación realizada. En móvil, el LCP bajó de 3.2 s a 1.7 s y el puntaje subió de 83 a 100.

La carga diferida se mantuvo únicamente para la galería, ya que usarla en el hero retrasaría el elemento principal visible y podría empeorar el LCP. El CLS debe mantenerse estable gracias al espacio reservado mediante dimensiones y `aspect-ratio`.

El cambio con menor impacto sobre la velocidad fue añadir dimensiones y `aspect-ratio` a las tarjetas: el CLS ya era 0, pero este cambio protege la estabilidad visual ante futuras imágenes. No apareció un problema nuevo en las mediciones finales. Conservaría todos los cambios porque reducen la transferencia inicial, preservan el diseño y mantienen la estabilidad visual. Una segunda iteración podría registrar los bytes transferidos desde la pestaña Network de las herramientas de desarrollo y revisar si los tamaños de `srcset` elegidos son los más adecuados para los dispositivos que visitan el sitio.

## 6. Respuestas finales

### ¿Qué significa LCP y cuál fue el elemento LCP antes y después?

LCP significa *Largest Contentful Paint*. Mide cuánto tarda en mostrarse el elemento con contenido más grande visible al inicio. En esta página el elemento LCP fue la imagen principal del hero; antes tardó 3.2 s en móvil y después 1.7 s.

### ¿Cómo puede una imagen sin dimensiones explícitas contribuir a CLS?

Si el navegador no conoce el espacio que ocupará una imagen antes de descargarla, puede recolocar el contenido cuando la imagen termina de cargar. Definir `width`, `height` o una relación de aspecto permite reservar ese espacio y evita el salto visual.

### ¿Por qué `loading="lazy"` sirve en la galería pero no en el hero?

La galería está debajo del primer viewport, por lo que diferir sus imágenes reduce los datos iniciales. El hero es visible al abrir la página y probablemente es el LCP; diferirlo retrasaría su descarga y empeoraría la percepción de carga.

### ¿Cuál es la diferencia entre `defer` y `async`? ¿Cuál se eligió?

`async` descarga el script en paralelo y lo ejecuta tan pronto termina, sin garantizar el orden respecto de otros scripts. `defer` también lo descarga en paralelo, pero espera a que el HTML termine de analizarse y conserva el orden. Se eligió `defer` para `script.js` porque el script necesita consultar elementos del DOM sin bloquear el análisis inicial del documento.

### ¿Qué formato se eligió y qué comparación de peso se obtuvo?

Se usó `f_auto`, que permite a Cloudinary entregar el formato más eficiente compatible con el navegador, junto con `q_auto:eco` para comprimir el hero. El JPEG original era de 3840 × 2160 px y pesaba aproximadamente 8.18 MB; las variantes comprobadas de 640, 960 y 1280 px pesaron aproximadamente 22 KB, 35 KB y 46 KB, respectivamente.

### Si el puntaje sube pero el LCP sigue por encima del objetivo, ¿terminó la optimización?

No. Un puntaje alto es útil, pero el LCP mide una parte importante de la experiencia real. Si se mantiene por encima de 2.5 s, se debe seguir investigando el recurso LCP, la red, el servidor y el trabajo de renderizado. En esta auditoría el LCP móvil final fue 1.7 s, por lo que sí quedó dentro del objetivo.
