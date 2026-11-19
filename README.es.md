# Alertas de directos de Twitch para Discord

![Alertas de directos de Twitch para Discord](assets/banner.svg)

[![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Node.js](https://img.shields.io/badge/Node.js-22%2B-5FA04E?logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![Twitch EventSub](https://img.shields.io/badge/Twitch-EventSub-9146FF?logo=twitch&logoColor=white)](https://dev.twitch.tv/docs/eventsub/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Un bot de Discord en TypeScript que publica un aviso cuando los canales de Twitch configurados empiezan a emitir. Cada servidor de Discord tiene su propio canal de avisos y su propia lista de canales de Twitch. Los canales de Twitch compartidos se vigilan una sola vez y se anuncian en todos los servidores que se suscriben a ellos.

[English](README.md)

## Características

- Avisos casi en tiempo real a partir del evento `stream.online` de Twitch EventSub, con sondeo de Helix como red de seguridad.
- Un solo bot para muchos servidores de Discord: cada servidor tiene su propio canal de avisos y su propia lista de canales de Twitch.
- Los canales de Twitch compartidos se vigilan una sola vez y se anuncian en todos los servidores que se suscriben a ellos.
- La barra del embed del aviso es morada (color de Twitch) mientras el directo está en curso y pasa a gris cuando termina (lo detecta el sondeo de reconciliación).
- El seguimiento de entrega por suscriptor reintenta los envíos fallidos sin duplicar los que ya se completaron.
- EventSub se reconecta con backoff, los errores transitorios de Helix se reintentan y el arranque está protegido.
- Permisos mínimos en Discord y sin intents de contenido de mensajes, de miembros ni de presencia.

## Cómo funciona

![Arquitectura en ejecución](assets/architecture.svg)

Twitch EventSub aporta los eventos `stream.online` casi en tiempo real; el sondeo de Helix reconcilia los eventos perdidos tras el arranque o una reconexión. El bot resuelve cada canal de Twitch una sola vez, registra la entrega por suscriptor en `data/live-state.json` y publica el embed en cada canal de servidor suscrito.

## Ejemplo de aviso

![Ejemplo de aviso de directo de Twitch](assets/alert-example.svg)

*Ejemplo ilustrativo con datos ficticios. El embed real muestra la vista previa del directo y la categoría. El número de espectadores está oculto por defecto; define `SHOW_VIEWER_COUNT=true` en `.env` para activarlo.*

## Requisitos

- Node.js 22+
- pnpm 11+
- Una aplicación/token de bot de Discord
- Una aplicación de desarrollador de Twitch (Client ID y Client Secret)

## Configuración de Discord

1. Crea una aplicación y un bot de Discord en el [Portal de desarrolladores](https://discord.com/developers/applications).
2. Invítalo a cada servidor con el ámbito `bot` y los permisos `View Channels`, `Send Messages`, `Embed Links` y `Read Message History`.
3. No hace falta el permiso de Administrador, el intent de contenido de mensajes ni el intent de miembros o de presencia.
4. Copia el token del bot en el `.env` local como `DISCORD_TOKEN`.

## Configuración de Twitch (sin necesidad de la CLI de Twitch)

1. Registra una aplicación de Twitch en la [consola de desarrolladores de Twitch](https://dev.twitch.tv/console/apps).
2. Añade esta URL de redirección OAuth exacta: `http://localhost:3000`.
3. Copia su Client ID y su Client Secret en el `.env` local como `TWITCH_CLIENT_ID` y `TWITCH_CLIENT_SECRET`.
4. Instala las dependencias y ejecuta `pnpm auth:twitch` desde el directorio del proyecto.
5. Abre la URL de autorización de Twitch que se muestra y aprueba la aplicación. El callback local valida el estado de OAuth y escribe en `.env` los tokens de acceso y de refresco del usuario sin mostrarlos.
6. El WebSocket de Twitch EventSub necesita un token de acceso de usuario. `stream.online` no requiere ámbitos adicionales. El bot renueva los tokens de acceso caducados y persiste la rotación de tokens.

El servidor de redirección solo se enlaza a loopback, en el puerto 3000, durante la configuración de OAuth, y se apaga tras la autorización o al agotarse el tiempo. No hace falta ninguna URL de callback pública.

## Configuración local

```bash
cp .env.example .env
chmod 600 .env
cp config/guilds.example.json config/guilds.json
```

Rellena `.env` en local. Edita `config/guilds.json` para cada servidor de Discord:

```json
{
  "guilds": [
    {
      "guildId": "YOUR_DISCORD_SERVER_ID",
      "alertChannelId": "YOUR_TEXT_CHANNEL_ID",
      "twitchChannels": ["streamer_one", "streamer_two"]
    }
  ]
}
```

Activa el Modo de desarrollador en Discord y copia los IDs del servidor y del canal de texto. Los nombres de Twitch son los logins de los canales, no URLs. El mismo canal de Twitch puede aparecer en distintas entradas de servidor; ese directo genera un aviso en cada canal de Discord listado.

Si un login de Twitch configurado no se puede resolver (la cuenta se renombró o se borró), el bot registra un aviso y sigue vigilando el resto de canales. Actualiza `config/guilds.json` y reinicia para corregirlo.

La presencia del bot coincide con la de `server-status`: `BOT_ACTIVITY_TYPE=Custom`, `BOT_ACTIVITY_NAME=x0void.com` y `BOT_ACTIVITY_STATE=🔗 x0void.com`. Este estado personalizado de texto libre no es un enlace clicable. `BOT_ACTIVITY_URL` solo lo usa una actividad `Streaming` y, en ese caso, debe ser una URL HTTPS de Twitch o YouTube. La presencia se vuelve a aplicar tras los eventos ready/resume de los shards de Discord y en cada ciclo de reconciliación de Twitch.

Los números de espectadores están ocultos en los embeds de aviso por defecto. Define `SHOW_VIEWER_COUNT=true` en `.env` para activarlos; `true` y `false` no distinguen mayúsculas y cualquier otro valor se rechaza durante el arranque.

## Ejecución

```bash
pnpm install
pnpm auth:twitch
pnpm dev
```

Para compilar y ejecutar el JavaScript de producción:

```bash
pnpm build
pnpm start
```

## Estructura del proyecto

```text
src/
  application/    reconciliación e indexación de suscriptores
  domain/         entidades y puertos
  infrastructure/ adaptadores de Discord, Twitch, persistencia y configuración
  index.ts        composition root y arranque
tests/            suite de Vitest con fakes
```

## Despliegue con systemd

La unidad está en `deploy/announcements.service` y es una plantilla: `make install` sustituye `@DEPLOY_USER@`, `@DEPLOY_GROUP@`, `@DEPLOY_HOME@`, `@DEST@` y `@NODE_BIN@` antes de instalarla. Por defecto usa el usuario que ejecuta `make`, `/srv/services/announcements` y el `node` del `PATH`; se pueden cambiar por variables:

```bash
make install DEPLOY_USER=announcements DEPLOY_GROUP=announcements DEST=/srv/announcements
```

La instalación del servicio y la actualización del código son pasos separados:

```bash
make install  # crea el destino, instala la unidad y la config; no la habilita ni arranca
make deploy   # build, dependencias de producción y código; habilita si hace falta y reinicia la unidad
```

Antes del primer `make deploy`, prepara `/srv/services/announcements/.env` con permisos `600` y `config/guilds.json` para producción. El deploy no copia `.env` y conserva la configuración de guild existente; `make install` crea esa configuración a partir de la copia local solo si aún no existe. La unidad arranca con el lanzador de secretos (`~/.secrets/bin/run srv:announcements`), que carga el `.env` del destino; la aplicación también lo lee desde su directorio de trabajo. Con `LOG_FILE=/var/log/announcements.log` el bot escribe además en ese fichero, que crean `deploy/announcements.tmpfiles.conf` y rota `deploy/announcements.logrotate.conf` (los instala `make install`).

`TWITCH_RECONCILE_INTERVAL_SECONDS` vale 60 por defecto (válido de 15 a 3600). EventSub aporta las notificaciones normales de inicio casi en tiempo real; el sondeo de Helix reconcilia tras la reconexión o el arranque y recupera los eventos perdidos mientras estuvo desconectado. El estado se guarda por defecto en `data/live-state.json`.

## Desarrollo y tests

```bash
pnpm typecheck
pnpm test
pnpm build
```

Los tests usan fakes y no deben enviar mensajes reales a Discord ni crear suscripciones en Twitch.

## Gestión de secretos

- No subas nunca `.env`, `config/guilds.json`, tokens de OAuth, tokens de Discord ni el Client Secret de Twitch.
- `.env.example` y `config/guilds.example.json` son solo plantillas seguras.
- Cada persona que aloje una copia debe crear sus propias aplicaciones de Discord y de Twitch y aportar sus propias credenciales.
- Si un token o un client secret se comparte o se sube por accidente, revócalo o rótalo de inmediato.

## Licencia

MIT. Consulta [LICENSE](LICENSE).
