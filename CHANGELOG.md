# Changelog

All notable changes to this project will be documented in this file. See [commit-and-tag-version](https://github.com/absolute-version/commit-and-tag-version) for commit guidelines.

## 1.7.3 (2026-11-14)

### Features

* añade la carpeta .mats al .gitignore ([63826e0](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/63826e08b2c82284349deb59bb7a869fdba9b21c))
* **bot:** añade adaptadores de Twitch, Discord y persistencia ([af55e65](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/af55e65a28a14d2e1831947ffa69aaaa68adb0ae))
* **bot:** añade el dominio y los casos de uso ([c70341e](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/c70341e71057ba0330fcb0be565483befde70db6))
* **bot:** enruta por ID de broadcaster con un índice de suscriptores ([cd01578](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/cd0157876e57157754c4e952895327157b35e52b))
* **bot:** reintenta el arranque de Twitch y lo hace testeable ([3b4fb98](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/3b4fb9849121f3932dc3cd7a234230f208187d77))
* **discord:** muestra la versión en los avisos ([11043a7](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/11043a75a36cb5802ef450d75ae72e1a4a11381b))
* **discord:** permite mostrar los espectadores ([a90a0e0](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/a90a0e05ff86491a1cc59e4708b8d7da71b4abbd))
* **discord:** pinta gris la barra del embed al cerrar el directo ([e0d9e51](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/e0d9e51d6ca0f48e1b23891d18e2c646ecb03c10)), references [#4F545C](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/issues/4F545C) [#9146FF](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/issues/9146FF)
* **logging:** escribe también en LOG_FILE además de la consola ([30d6e8e](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/30d6e8ea612b0906ff465c88900e00fc24e6ab95))

### Bug Fixes

* **bot:** entrega alertas por suscriptor y migra el estado a v2 ([ef998de](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/ef998de76a5d762970f8fcedced5ec3480737b6b))
* **build:** limpia dist antes de compilar para no dejar restos ([4825982](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/482598274b873d154d36811ad0d068a32f952694))
* **deploy:** instala la unidad desde un fichero temporal ([635f6ed](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/635f6edff0d94d33c12fc60d9bbb5748cd344ffd))
* **deploy:** omite enable cuando la unidad ya esta habilitada ([e652eca](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/e652ecafe3835e8da6c84c7ef723f071beb7319d))
* **discord:** adjunta la miniatura para que no desaparezca al cerrar el aviso ([6c89fca](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/6c89fca4f3d96f870d839083c32683459f1a9b91))
* **discord:** congela la miniatura en los avisos vivos para que no cambie ni se pierda ([c8004ae](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/c8004ae4f4309d3a6f4a4278a95d4739c38daf60))
* **discord:** deja de re-subir la miniatura tras publicar para que el cierre no la rompa ([63fdc44](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/63fdc44a386f14e4458ae899e53c7f724cc2aa5a))
* **discord:** el cierre re-referencia la miniatura como attachment:// para que no salga suelta ([0aa81d1](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/0aa81d1ff84d4c082a6607812e67944930a86be3))
* **discord:** elimina la hora relativa del aviso de directo ([01450ad](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/01450ad08bfce7cbf3f533b5a46fbb9de21841bf))
* **discord:** espera a la miniatura real de Twitch antes de publicar el aviso ([3537087](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/3537087450055d600df88b41eab265b2d2af0eb7))
* **discord:** firma el pie de los avisos con el nombre del bot ([77359a6](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/77359a6ff0b5ae02e9733e47882cd7790bf53fd4))
* **discord:** refresca la miniatura cada 5 min en vivo y deja el aviso cerrado intacto ([50a5d0c](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/50a5d0cff2f34cdf9dba0026eb2132513a9b3532))
* **discord:** valida los canales de alerta al arrancar ([a8c7660](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/a8c76605d7c7be7a76c7102820f2698a6d241e48))
* **eventsub:** reconecta con keepalive, revocación y backoff ([5dd4476](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/5dd4476175808afe851616bbfc0b46c8f2c3326d))
* **twitch:** evita la miniatura cacheada de un directo anterior ([2cf8088](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/2cf80881db8a062de7ca4997bf5268585a366960))
* **twitch:** reintenta fallos transitorios en la API Helix ([8919572](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/8919572ec1e053af1190565b52b1cce6223a0b59))
* **twitch:** reintenta Helix al llegar stream.online para no perder la miniatura ([bb5ec16](https://github.com/x0jonatanfp/discord-twitch-stream-alerts/commit/bb5ec16cbc040d37c3a322d26e9deb89980b4ee1))
