# ![Logotipo](wsPlayerlogo.svg) wsPlayer

wsPlayer es un reproductor de video web que se centra en el protocolo WebSocket-fmp4. En comparación con RTMP, HLS y HTTP-FLV, el protocolo HTTP/WebSocket-fmp4 tiene las ventajas de un retraso de reproducción corto y una buena compatibilidad con HTML5;

Vea la comparación de varios protocolos de medios de transmisión:


| Nombre del protocolo | Protocolo de transmisión de red | Retraso | Tipo de codificación | Soporte HTML5 |
| :----------: | :----------: | :---: | :-------------- -- ----: | :--------------------------: |
| RTSP | TCP/UDP/Multicast | 0~3s | H264/H265 | No compatible (excepto RTSP sobre HTTP) |
| RTMP | TCP | 0~3s | H264/H265(CodecID =12) | No compatible |
| HLS | Conexión corta HTTP | 1~10s | H264/H265 | soporte de etiquetas de vídeo |
| HTTP-FLV | Conexión HTTP larga | 0~3s | H264/H265(CodecID =12) | flv → fmp4 → etiqueta de video |
| HTTP-fmp4 | Conexión larga HTTP | 0~3s | H264/H265 | Soporte nativo de etiquetas de vídeo |
| WebSocket-FLV | WebSocket | 0~3s | H264/H265(CodecID =12) | flv → fmp4 → etiqueta de vídeo |
| WebSocket-fmp4 | WebSocket | 0~3s | H264/H265 | Usar MSE, reproducción de etiquetas de vídeo |

Nota: El navegador limita el número de conexiones largas HTTP simultáneas en una sola página a 6, lo que significa que HTTP-FLV y HTTP-fmp4 solo pueden reproducir 6 pantallas de vídeo al mismo tiempo; pero el navegador no tiene límite en el número de Conexiones WebSocket;



## Dependencias del proyecto

Debes usar [mp4box.js](https://github.com/gpac/mp4box.js) para analizar los códecs en fmp4 moov;



## Inicio rápido

Se recomienda utilizar [ZLMediaKit](https://github.com/ZLMediaKit/ZLMediaKit) como servidor de transmisión backend del protocolo WebSocket-fmp4.

1. Implementar el servidor de transmisión backend

```shell
docker pull panjjo/zlmediakit
docker run -id -p 8080:80 -p 554:554 panjjo/zlmediakit
```

2. Utilice el comando ffmpeg para agregar una transmisión push RTSP a ZLMediaKit
```shell
fmpeg -re -stream_loop -1 -i test.mp4 -an -vcodec copy -f rtsp -rtsp_transport tcp rtsp://100.100.154.29/live/test
```

3. De acuerdo con las [reglas de URL de reproducción] de ZLMediaKit (https://github.com/zlmediakit/ZLMediaKit/wiki/%E6%92%AD%E6%94%BEurl%E8%A7%84%E5%88%99) Se aprende que el formato de URL del protocolo WebSocket-fmp4 es:
```cáscara
ws://100.100.154.29:8080/live/test.live.mp4
```

4. Luego llama al código del jugador:

```html
<html>
<head>
</head>
<body>
    <script type="text/javascript" src="mp4box.all.min.js"></script>
    <script type="text/javascript" src="wsPlayer.js"></script>
    <video muted autoplay id="video"></video>
    <script>
        document.addEventListener('DOMContentLoaded', function () {
            var player = new wsPlayer("video", "ws://100.100.154.29:8083/live/test.live.mp4");
            player.open();
        });
    </script>
</body>
</html>
```

## Principio del jugador
Agregue el fragmento de segmento fmp4 `appendBuffer` recibido por WebSocket a `MediaSource`. En este momento, `video.buffered` registrará el período de tiempo de video actual de `appendBuffer` y luego establecerá el tiempo de inicio de `video.buffered`. a `video.currentTime`, entonces el navegador comenzará a reproducir desde el video almacenado en caché en `video.buffered`

## Plan de proyecto
* v1.0 implementa el uso de la etiqueta de video para llamar a MSE para reproducir la transmisión en vivo de WebSocket-fmp4 (H.264) y encapsula el reproductor como un componente npm estándar;
* v2.0 implementa el uso de WebAssembly FFmpeg para decodificar H.265 y luego usa la etiqueta de lienzo WebGL para representar YUV, logrando así la reproducción de la transmisión en vivo de WebSocket-fmp4 (H.265) y el cambio dinámico entre H.264 y H.265. .mecanismo;
* v3.0 implementa devolución de llamada para información SEI de transmisión de video

## Información del contacto
- Grupo de comunicación QQ: Nombre del grupo: wsPlayer Número de grupo: 710185138
