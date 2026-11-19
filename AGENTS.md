# AGENTS.md · announcements

Guía para agentes de código. Qué es y cómo se configura para personas está en `README.md`.

## Qué es

Bot de Discord (TypeScript, ESM, Node 22+) que publica un aviso cuando un canal de Twitch configurado empieza a emitir. Cada servidor tiene su canal de avisos y su lista de canales de Twitch. Un canal compartido por varios servidores se vigila una sola vez y se anuncia en todos los suscritos.

## Arquitectura

Hexagonal, con las capas de `domain/` → `application/` → `infrastructure/` y el composition root en `src/index.ts`.

- `src/domain/entities/stream.ts`: tipos del dominio (`LiveStream`, `StreamSubscriber`, `TwitchChannel`, etc.).
- `src/domain/ports/`: puertos (`twitch-streams`, `stream-notifier`, `live-state-store`, `stream-config`, `logger`).
- `src/application/build-subscriber-index.ts`: índice `broadcasterId → suscriptores` a partir de la configuración de guilds.
- `src/application/reconcile-live-streams.ts`: caso de uso central. Gestiona el estado de directos, el reparto por suscriptor y la entrega pendiente (reintenta solo lo que falló).
- `src/infrastructure/`: adaptadores. `twitch/` (Helix, EventSub por WebSocket, almacén de tokens, `oauth-setup.ts`), `discord/` (avisos y presencia), `persistence/` (JSON con el estado de directos), `config/` (entorno y `config/guilds.json`), `logging/`.
- `src/index.ts`: composition root. Carga la configuración, resuelve los canales de Twitch con reintentos, valida los canales de Discord, arranca EventSub y el intervalo de reconciliación, y gestiona el cierre por señal.

Flujo principal: EventSub `stream.online` → `handleOnlineEvent` → avisos a cada suscriptor. Al reconectar y en cada ciclo (`TWITCH_RECONCILE_INTERVAL_SECONDS`, por defecto 60) se hace un `reconcile` con Helix, que recupera lo perdido y marca como terminados los directos que ya no aparecen.

Dependencias: `domain/` y `application/` no importan de `infrastructure/`, ni de config ni del logger concreto. Esto se comprobó con grep; mantenerlo al añadir código.

## Comandos

Scripts de `package.json`:

| Comando | Qué hace |
| --- | --- |
| `pnpm install` | Instala dependencias (pnpm 11, fijado en `packageManager`). |
| `pnpm dev` | Arranca con `tsx watch src/index.ts`. |
| `pnpm auth:twitch` | Flujo OAuth local que escribe tokens en el `.env` local. No imprime tokens. |
| `pnpm build` | Limpia `dist/` y compila con `tsconfig.build.json`. |
| `pnpm start` | Ejecuta `dist/index.js`. |
| `pnpm typecheck` | `tsc --noEmit`. |
| `pnpm test` | `vitest run`. |
| `pnpm test:watch` / `pnpm test:coverage` | Vitest en modo watch / con cobertura. |

`Makefile` (despliegue en un servidor propio con systemd):

- `make check`: typecheck, test y build.
- `make install`: prepara el directorio de destino (`DEST`), instala la unidad, tmpfiles y logrotate, y crea `config/guilds.json` solo si no existe. **Usa `sudo`. No lo lances sin orden expresa.** No habilita ni arranca la unidad.
- `make deploy`: depende de `release`, así que ejecuta el versionado y `git push --follow-tags`. Después hace build, sincroniza `dist/` y los manifiestos, instala dependencias de producción, habilita y reinicia la unidad. **Usa `sudo` y publica en remoto. No lo lances sin orden expresa.**
- `make release`: exige árbol limpio (`check-clean`), ejecuta `commit-and-tag-version` y hace push con etiquetas.
- `make restart`, `make status`, `make logs`: operaciones sobre la unidad. `restart` usa `sudo`.

## Cómo probar un cambio

- Tests en `tests/*.test.ts`, con Vitest (`vitest.config.ts` incluye `tests/**/*.test.ts`, entorno `node`, `clearMocks` y `restoreMocks` activos).
- Los tests usan fakes. No deben enviar mensajes reales a Discord ni crear suscripciones en Twitch.
- Un solo fichero, derivado del script `test`: `pnpm vitest run tests/reconcile-live-streams.test.ts`. Filtrar por nombre: `pnpm vitest run -t "<texto>"`.
- Antes de dar un cambio por bueno: `pnpm typecheck && pnpm test && pnpm build`.
- Cada comportamiento nuevo o bug corregido lleva su test en el mismo cambio.

## Convenciones

- Gestor: **pnpm** (lockfile `pnpm-lock.yaml`, `pnpm-workspace.yaml`).
- Código y nombres en inglés; ficheros en kebab-case; imports relativos con extensión `.js` (compilación ESM con `tsc`).
- Versión y `CHANGELOG.md`: los genera `commit-and-tag-version` (`.versionrc.json`, prefijo de etiqueta `v`) durante `make release`. **No se editan a mano.** El commit de release es `chore(release): <etiqueta>`.
- Los commits siguen Conventional Commits en español (`feat`, `fix`, `docs`, `chore`...). Las subidas de versión dependen de estos tipos.
- Configuración por entorno: `.env` (no versionado) a partir de `.env.example`. Los `.env.example` y `config/guilds.example.json` son plantillas sin valores reales.

## Decisiones y trampas

- **Entrega por suscriptor.** El estado guarda `deliveredTo` y `alerts` por directo. Un reintento no vuelve a enviar a quien ya recibió el aviso. Si cambias el formato del estado, revisa `json-live-state-store.ts` y el caso de uso antes de asumir compatibilidad.
- **Un directo nuevo tiene `streamId` distinto.** Si el `streamId` cambia, el estado se reinicia para ese canal. Así se distingue un directo nuevo de uno ya anunciado.
- **Miniatura y avisos cerrados.** El historial de commits (`fix(discord)` y `revert`) muestra que la miniatura se adjunta como `attachment://` y que el aviso se congela al cerrarse. Antes de tocar `infrastructure/discord/stream-notifier.ts`, lee el historial de esos commits (`git log --oneline -- src/infrastructure/discord/stream-notifier.ts`).
- **Presencia.** Se reaplica tras los eventos ready/resume de Discord y en cada ciclo de reconciliación.
- **Viewer count oculto por defecto.** `SHOW_VIEWER_COUNT` solo admite `true` o `false` (sin distinguir mayúsculas); cualquier otro valor rechaza el arranque.
- **Intervalo de reconciliación acotado.** `TWITCH_RECONCILE_INTERVAL_SECONDS` debe estar entre 15 y 3600.
- **Canal de Twitch sin resolver.** Si un login no se resuelve, se avisa y se sigue con el resto. Se corrige en `config/guilds.json` y reiniciando.
- **Logs.** `LOG_FILE` añade un fichero además de la consola; vacío o ausente, solo consola.
- **Nunca se versiona** `.env`, `config/guilds.json`, `data/`, tokens ni secretos de Discord o Twitch. Si alguno se filtra, rotarlo.

## Mantenimiento

- Este fichero y el `README.md` se actualizan en el mismo cambio que los deja obsoletos.
- Aquí solo va lo que puede ser público: nada de rutas locales, puertos de la máquina, IPs, perfiles de secretos ni datos personales.
