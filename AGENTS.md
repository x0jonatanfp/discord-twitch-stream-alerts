# AGENTS.md · announcements

Guía para agentes de código que trabajan en este repositorio. Qué es, cómo se instala y cómo se usa está en el `README.md`.

## Qué es

Bot de Discord en TypeScript que avisa cuando los canales de Twitch configurados empiezan a emitir. Cada servidor de Discord tiene su propio canal de avisos y su lista de canales de Twitch. Un canal de Twitch compartido se vigila una sola vez y se anuncia en todos los servidores suscritos.

## Estructura

- `assets/`
- `config/`
- `deploy/`
- `src/`
- `tests/`
- `.env.example`
- `.gitignore`
- `.versionrc.json`
- `CHANGELOG.md`
- `Makefile`
- `README.md`
- `package.json`
- `pnpm-lock.yaml`
- `pnpm-workspace.yaml`
- `tsconfig.build.json`
- `tsconfig.json`
- `vitest.config.ts`

## Comandos

Scripts de `package.json` (raíz):

| Comando | Ejecuta |
| --- | --- |
| `pnpm dev` | `tsx watch src/index.ts` |
| `pnpm auth:twitch` | `tsx src/infrastructure/twitch/oauth-setup.ts` |
| `pnpm clean` | `rm -rf dist` |
| `pnpm build` | `pnpm clean && tsc -p tsconfig.build.json` |
| `pnpm start` | `node dist/index.js` |
| `pnpm typecheck` | `tsc --noEmit` |
| `pnpm test` | `vitest run` |
| `pnpm test:watch` | `vitest` |
| `pnpm test:coverage` | `vitest run --coverage` |

Objetivos del `Makefile`:

- `make build`
- `make check`
- `make typecheck`
- `make test`
- `make install`
- `make deploy`
- `make restart`
- `make status`
- `make logs`
- `make check-clean`
- `make release`

## Convenciones

- Gestor de paquetes: **pnpm** (según el lockfile).
- Tests en `tests/`.
- El despliegue versionado (unidad systemd y similares) vive en `deploy/`. Los datos de esta máquina (rutas, puertos, perfiles de secretos) no van aquí: están fuera del repo.
- Configuración por entorno: copia `.env.example` a `.env` (ignorado por git). Nunca se versionan valores reales.
- Versión y `CHANGELOG.md` los genera la release (skill `my-commits`); no se editan a mano.

## Mantenimiento

- Este fichero y el `README.md` se actualizan en el mismo cambio que los deja obsoletos.
- Aquí solo va lo que puede ser público: nada de rutas locales, puertos de la máquina, IPs, perfiles de secretos ni datos personales.
