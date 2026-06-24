# SZoluciones — Tornillos de TypeScript
### El ADN técnico extraído del monorepo Movete

> **Qué es esto.** Este es el manual de "tornillos": las decisiones de ingeniería que un programador Senior tomó, de forma deliberada y repetida, a lo largo del monorepo **Movete** (una plataforma SaaS de gestión de gimnasios/centros deportivos). Cada tornillo es una regla reutilizable —no una curiosidad del proyecto— pensada para que cualquier agente de IA (o dev nuevo) la aplique en un proyecto SZoluciones **completamente distinto**.

> **Cómo se generó.** Se auditó (solo lectura) todo el repo: backend **Fastify + TypeScript** (`packages/api`), web admin **Next.js 14 + Tailwind** (`apps/web-admin`), app móvil **Expo / React Native** (`apps/mobile`), base **PostgreSQL + Prisma**, contrato compartido (`packages/shared`), y el build con **pnpm workspaces + Turborepo**. Cada tornillo está respaldado por evidencia real (archivo + líneas + fragmento verbatim) verificada contra el código por un segundo agente que actuó de fact-checker.

> **Cómo leer "Aplica a".** `Universal` = sirve en cualquier stack/lenguaje. `Solo TypeScript` = es un idiom de TS/JS. `Solo este proyecto` = es específico de Movete (úsalo como referencia, no lo copies a ciegas).

**60 tornillos · 13 áreas.** Empezá por el índice; si vas a tocar seguridad multi-tenant, leé primero el *Mapa de dependencias* al final.

---

## Índice de tornillos

| # | Nombre coloquial | Área | Aplica a |
|---|------------------|------|----------|
| 1 | El package interno se referencia con workspace:* y se importa con alias @scope/* | ÁREA 1 — El plano de la casa | Solo TS |
| 2 | apps/ son deployables, packages/ son librerías, y se nota en el build | ÁREA 1 — El plano de la casa | Universal |
| 3 | tsconfig raíz único + override por proyecto | ÁREA 1 — El plano de la casa | Solo TS |
| 4 | Turbo orquesta respetando el grafo (^build) y cachea solo lo determinístico | ÁREA 1 — El plano de la casa | Solo TS |
| 5 | JWT flaco para identidad, DB gorda para permisos | ÁREA 2 — La puerta y el portero | Universal |
| 6 | Cadena onRequest: primero autenticar, después autorizar (y de paso cobrar) | ÁREA 2 — La puerta y el portero | Solo proyecto |
| 7 | Access token en memoria, refresh token persistido y rotado | ÁREA 2 — La puerta y el portero | Solo TS |
| 8 | Catálogo de permisos como array de strings, fuente única compartida | ÁREA 2 — La puerta y el portero | Universal |
| 9 | El org_id sale SIEMPRE del JWT, nunca del cliente | ÁREA 2b — Mismo dato, distinta respuesta | Universal |
| 10 | Cada query carga su propio WHERE organization_id (filtro app-layer omnipresente) | ÁREA 2b — Mismo dato, distinta respuesta | Universal |
| 11 | Fail-closed: sin contexto de tenant, cero datos | ÁREA 2b — Mismo dato, distinta respuesta | Universal |
| 12 | RLS de Postgres con GUC como red de seguridad de DB (defensa en profundidad) | ÁREA 2b — Mismo dato, distinta respuesta | Solo proyecto |
| 13 | La transacción como guardián de la regla de negocio | ÁREA 3 — El viaje de un dato | Universal |
| 14 | organization_id en cada query, sin excepción | ÁREA 3 — El viaje de un dato | Universal |
| 15 | Validar con Zod safeParse antes de tocar la DB | ÁREA 3 — El viaje de un dato | Solo TS |
| 16 | El cliente tolera el envelope, no lo asume | ÁREA 3 — El viaje de un dato | Solo TS |
| 17 | Invalidación dirigida de React Query con actualización optimista | ÁREA 3 — El viaje de un dato | Solo TS |
| 18 | La plantilla de tabla multi-tenant (el molde que se copia y pega) | ÁREA 4 — El archivo del negocio | Universal |
| 19 | Índices deliberados sobre FK y deleted_at (no dejar que Postgres adivine) | ÁREA 4 — El archivo del negocio | Universal |
| 20 | onDelete con intención: Cascade para hijos, SetNull para histórico | ÁREA 4 — El archivo del negocio | Universal |
| 21 | RLS como última línea fail-closed (defense in depth en la base) | ÁREA 4 — El archivo del negocio | Solo proyecto |
| 22 | Conexiones como singleton con shutdown ordenado, health check y escape hatch a pg crudo | ÁREA 4 — El archivo del negocio | Solo TS |
| 23 | Errores de negocio se atajan, errores del sistema explotan | ÁREA 5 — Cuando algo se rompe | Universal |
| 24 | Envelope de error { message } en español, uniforme (con una deuda viva en auth) | ÁREA 5 — Cuando algo se rompe | Universal |
| 25 | Validar en la frontera con Zod safeParse y traducir SIEMPRE igual | ÁREA 5 — Cuando algo se rompe | Solo TS |
| 26 | Webhook de pago a prueba de duplicados (idempotencia + firma + verdad del proveedor) | ÁREA 5 — Cuando algo se rompe | Universal |
| 27 | Refresh silencioso de token en 401 con deduplicación (mismo patrón en mobile y web) | ÁREA 5 — Cuando algo se rompe | Solo TS |
| 28 | Factory de rutas: setupXRoutes(app) | ÁREA 6 — La letra del Senior | Universal |
| 29 | Fail-closed multi-tenant: nunca una query sin org_id + soft delete | ÁREA 6 — La letra del Senior | Universal |
| 30 | Validación con Zod safeParse y 400 estructurado antes del SQL | ÁREA 6 — La letra del Senior | Solo TS |
| 31 | Auto-migración ensureXTable/ensureXColumn en el arranque | ÁREA 6 — La letra del Senior | Solo proyecto |
| 32 | Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission | ÁREA 6 — La letra del Senior | Universal |
| 33 | El .env.example como contrato narrado, no como lista muerta | ÁREA 7 — Las llaves del local | Universal |
| 34 | Secrets multiempresa cifrados con AES-256-GCM, nunca en texto plano | ÁREA 7 — Las llaves del local | Universal |
| 35 | Fallbacks de env en cascada: DB -> env -> default local | ÁREA 7 — Las llaves del local | Solo TS |
| 36 | Config de cliente como build-arg, config de servidor como runtime env | ÁREA 7 — Las llaves del local | Solo proyecto |
| 37 | Feature flags con default apagado para lo caro o riesgoso | ÁREA 7 — Las llaves del local | Universal |
| 38 | Test de integración con DB real, sin mocks (inject + seed) | ÁREA 8 — Cuándo algo está realmente terminado | Universal |
| 39 | Probar el caso negativo y el contrato, no solo el happy path | ÁREA 8 — Cuándo algo está realmente terminado | Universal |
| 40 | RLS como red de seguridad final, testeada como atacante | ÁREA 8 — Cuándo algo está realmente terminado | Universal |
| 41 | QA por smoke tests contra entorno real, gateando el release | ÁREA 8 — Cuándo algo está realmente terminado | Universal |
| 42 | Un solo paquete, tres puertas (barrel @movete/shared) | ÁREA 9 — El contrato entre pantallas | Solo TS |
| 43 | Zod como fuente única: el validador ES el tipo | ÁREA 9 — El contrato entre pantallas | Solo TS |
| 44 | Permisos como datos: el array const que tipa todo | ÁREA 9 — El contrato entre pantallas | Solo TS |
| 45 | El espejo Prisma↔TS: enums de DB = union types de shared | ÁREA 9 — El contrato entre pantallas | Solo TS |
| 46 | Contrato de respuesta y de identidad uniforme (ApiResponse / JwtPayload / RequestContext) | ÁREA 9 — El contrato entre pantallas | Universal |
| 47 | setupXRoutes: el módulo es una función auto-instalable | ÁREA 10 — Cómo crece esto | Solo proyecto |
| 48 | Guards gemelos read/manage al tope del módulo | ÁREA 10 — Cómo crece esto | Solo proyecto |
| 49 | Endpoint CRUD canónico: safeParse + tenant + soft-delete + mapX | ÁREA 10 — Cómo crece esto | Solo proyecto |
| 50 | Schema Zod compartido: Create + Update.partial() como fuente de verdad | ÁREA 10 — Cómo crece esto | Solo TS |
| 51 | Registrar una vez en index.ts; gatear en Sidebar y hook en mobile | ÁREA 10 — Cómo crece esto | Solo proyecto |
| 52 | Credenciales cifradas en reposo con clave derivada y cadena de fallback | Integraciones externas y dinero | Universal |
| 53 | Webhook idempotente que no confia en el cliente y re-verifica server-to-server | Integraciones externas y dinero | Universal |
| 54 | Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero | Integraciones externas y dinero | Universal |
| 55 | Billing de la plataforma como gate de permisos con 402 | Integraciones externas y dinero | Universal |
| 56 | Fallback por entorno con bandera de origen explicita | Integraciones externas y dinero | Solo TS |
| 57 | Paginación con clamp defensivo y envelope { data, total, page, totalPages } | ÁREA TRANSVERSAL — Patrones no cubiertos | Universal |
| 58 | Constructor incremental de WHERE + params para filtros opcionales | ÁREA TRANSVERSAL — Patrones no cubiertos | Solo TS |
| 59 | CSV listo para Excel: BOM, CRLF, escapeo RFC y filename con fecha | ÁREA TRANSVERSAL — Patrones no cubiertos | Universal |
| 60 | Jobs en background: gate por header-secret, run-log en DB y dedupe idempotente por índice único parcial | ÁREA TRANSVERSAL — Patrones no cubiertos | Universal |

---

## Los tornillos

### ÁREA 1 — El plano de la casa (estructura del monorepo)

#### TORNILLO #1 — El package interno se referencia con workspace:* y se importa con alias @scope/*

**En una línea:** Las dependencias entre paquetes del monorepo nunca son rutas relativas ni versiones de npm: son workspace:* + un alias namespaced.

**Lo que hace el Senior:** Cada vez que un paquete necesita otro paquete del repo lo declara como dependencia con el protocolo de pnpm 'workspace:*' (el asterisco = 'usá la versión que esté en el workspace, no busques en el registry'). Esto aparece de forma idéntica en 4 paquetes que consumen @movete/shared: packages/api, packages/ui, apps/web-admin y apps/mobile (los dos apps además consumen @movete/ui). El nombre del paquete siempre lleva el namespace @movete/ (ej @movete/shared, @movete/api) y se importa por ese alias, no por '../../packages/shared'. Los alias están mapeados en el tsconfig raíz (paths) apuntando al src/ del paquete, así el editor resuelve sin compilar. Resultado: podés mover carpetas o publicar paquetes sin romper imports, y pnpm linkea todo automáticamente.

**Ejemplo real del repo:**

`packages/api/package.json:26-26`

```json
"@movete/shared": "workspace:*",
```

`packages/ui/package.json:15-15`

```json
"@movete/shared": "workspace:*",
```

`apps/web-admin/package.json:15-16`

```json
"@movete/shared": "workspace:*",
    "@movete/ui": "workspace:*",
```

`apps/mobile/package.json:25-26`

```json
"@movete/shared": "workspace:*",
    "@movete/ui": "workspace:*",
```

`tsconfig.json:27-32`

```json
"paths": {
      "@movete/shared": ["./packages/shared/src"],
      "@movete/api": ["./packages/api/src"],
      "@movete/ui": ["./packages/ui/src"],
      "@movete/config": ["./packages/config/src"]
    }
```

**La regla para el próximo proyecto:** En cualquier monorepo, declará las dependencias internas con el protocolo del package manager (workspace:* en pnpm/yarn, * en npm workspaces) y dales un namespace @scope/ común. Importá siempre por ese alias (@scope/foo), nunca por ruta relativa que cruce paquetes (../../packages/foo). Replicá esos alias en tsconfig paths apuntando al src/ para que el IDE resuelva sin build previo.

**Señal de que lo estás usando bien:** Buscás 'workspace:*' o '../../packages/' en los imports de todo el repo: el primero aparece en cada dependencia interna y el segundo no aparece nunca. Renombrar la carpeta de un paquete no rompe ningún import.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #3 — tsconfig raíz único + override por proyecto

---

#### TORNILLO #2 — apps/ son deployables, packages/ son librerías, y se nota en el build

**En una línea:** La separación apps/ vs packages/ no es cosmética: los packages emiten .d.ts y dist, los apps no emiten nada (los empaqueta su bundler).

**Lo que hace el Senior:** Dividió el repo en dos carpetas con roles distintos declarados en pnpm-workspace.yaml: packages/* (shared, ui, config: librerías reutilizables) y apps/* (api, web-admin, mobile: cosas que se despliegan). La diferencia se ve en los package.json y tsconfig: los packages compartidos tienen 'main', 'types', 'files:[dist]' y un build 'tsc' que EMITE (en shared: noEmit:false, declaration:true, declarationMap:true) para que otros consuman tipos; los apps en cambio no exponen 'types' ni 'files' y su tsconfig deja noEmit:true porque quien empaqueta es Next (next build), esbuild (api, vía scripts/build.mjs después de un tsc --noEmit de chequeo) o EAS/Expo (mobile). O sea: una librería 'produce un dist consumible', una app 'produce un deploy'. La frontera es deliberada y consistente.

**Ejemplo real del repo:**

`pnpm-workspace.yaml:1-3`

```yaml
packages:
  - 'packages/*'
  - 'apps/*'
```

`packages/shared/package.json:6-10`

```json
"main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "files": ["dist"],
  "scripts": {
    "build": "tsc",
```

`packages/shared/tsconfig.json:6-8`

```json
"noEmit": false,
    "declaration": true,
    "declarationMap": true,
```

`apps/web-admin/package.json:6-7`

```json
"dev": "next dev -p 3000",
    "build": "next build",
```

**La regla para el próximo proyecto:** Separá el monorepo en apps/ (deployables) y packages/ (librerías) desde el día uno y declaralo en el archivo de workspaces. A los packages dales main/types/files:[dist] y un build que emita declaraciones (.d.ts); a las apps dejalas con noEmit:true y que las empaquete su bundler (Next/Vite/esbuild/Expo). Si un paquete no tiene un consumidor claro o un app exporta tipos, algo está mal ubicado.

**Señal de que lo estás usando bien:** Cada carpeta en packages/ tiene 'types' y emite dist/; ninguna carpeta en apps/ exporta 'types'. Un app nunca aparece como dependencia de otro paquete; solo los packages aparecen como dependencia.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #1 — El package interno se referencia con workspace:* y se importa con alias @scope/*

---

#### TORNILLO #3 — tsconfig raíz único + override por proyecto

**En una línea:** Hay un solo tsconfig.json en la raíz con todas las reglas estrictas; cada package compartido extiende y solo cambia lo que necesita.

**Lo que hace el Senior:** Definió las reglas de TypeScript una vez en el tsconfig raíz: strict:true, noUnusedLocals, noUnusedParameters, noFallthroughCasesInSwitch, moduleResolution:bundler, isolatedModules, target ES2020 y los paths @movete/*. Después packages/api y packages/shared hacen 'extends: ../../tsconfig.json' y solo sobreescriben lo puntual: prenden noEmit:false + outDir/rootDir para poder compilar, y AFLOJAN deliberadamente noUnusedLocals/noUnusedParameters a false (la regla estricta vive arriba pero el código emitible no se traba por una variable sin usar). El api además pone allowImportingTsExtensions:false porque va a emitir JS real. Así la config estricta es DRY (sin repetir) y cada package solo expresa su diferencia, no toda la config.

**Ejemplo real del repo:**

`tsconfig.json:21-24`

```json
"strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
```

`packages/api/tsconfig.json:2-12`

```json
"extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "../../",
    "module": "ESNext",
    "target": "ES2020",
    "noEmit": false,
    "allowImportingTsExtensions": false,
    "noUnusedLocals": false,
    "noUnusedParameters": false
```

`packages/shared/tsconfig.json:2-8`

```json
"extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src",
    "noEmit": false,
    "declaration": true,
```

**La regla para el próximo proyecto:** Poné las reglas duras de TS (strict, módulos, target, paths) en un tsconfig raíz único y hacé que cada package las herede con 'extends'. En el sub-tsconfig escribí SOLO lo que difiere (outDir, noEmit, declaration). No copies la config completa en cada paquete: si tenés que tocar strict en 5 lugares, perdiste el DRY.

**Señal de que lo estás usando bien:** Cada tsconfig de package compartido tiene 'extends' y pocas líneas de compilerOptions (solo las diferencias). Cambiar una regla global (ej strict) se hace en un solo archivo y aplica a todos. Nota: apps/web-admin y apps/mobile NO heredan del raíz (web-admin tiene config completa sin extends; mobile hace 'extends: expo/tsconfig.base') porque sus toolchains —Next y Expo— traen su propia base (plugins:[next] / la base de Expo); eso es la excepción consciente, no un descuido.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

---

#### TORNILLO #4 — Turbo orquesta respetando el grafo (^build) y cachea solo lo determinístico

**En una línea:** El pipeline de Turborepo construye en orden de dependencias y marca explícitamente qué tareas NO se cachean.

**Lo que hace el Senior:** En turbo.json definió el pipeline donde 'build' declara dependsOn:['^build'] (el ^ significa 'antes de buildear este paquete, buildeá los paquetes de los que depende'), así shared se compila antes que api/web/mobile sin que nadie lo coordine a mano. Y distinguió a conciencia qué cachear: build declara outputs (dist/**, .next/**) y lint declara outputs (.eslintcache) para que Turbo cachee el resultado; dev/test:watch llevan persistent:true (procesos largos que no terminan); y test/type-check/clean/lint:fix llevan cache:false porque su resultado no es reproducible o no querés que un cache viejo mienta. Arriba, los scripts del package.json raíz son todos 'turbo run X' (dev con --parallel), y los scripts por paquete son homogéneos (dev/build/lint/lint:fix/type-check/clean), que es justo lo que Turbo necesita para mapear tareas uniformemente.

**Ejemplo real del repo:**

`turbo.json:3-10`

```json
"build": {
      "outputs": ["dist/**", ".next/**"],
      "dependsOn": ["^build"]
    },
    "test": {
      "outputs": ["coverage/**"],
      "cache": false
    },
```

`turbo.json:25-31`

```json
"type-check": {
      "outputs": [".tsc"],
      "cache": false
    },
    "clean": {
      "cache": false
    }
```

`package.json:12-14`

```json
"dev": "turbo run dev --parallel",
    "build": "turbo run build",
    "test": "turbo run test",
```

`packages/api/package.json:17-20`

```json
"lint": "eslint src --ext .ts,.tsx",
    "lint:fix": "eslint src --ext .ts,.tsx --fix",
    "type-check": "tsc --noEmit",
    "clean": "rm -rf dist"
```

**La regla para el próximo proyecto:** Usá un task runner con conciencia de grafo (Turborepo/Nx) y declará dependsOn:['^build'] en las tareas que dependen de otros paquetes. Nombrá los scripts de forma idéntica en cada paquete (build/lint/type-check/clean) para que el runner los mapee. Marcá cache:false en lo no determinístico (tests, type-check, dev, clean) y declará outputs solo en lo que produce artefactos cacheables. El package.json raíz solo orquesta (turbo run X), no contiene lógica de build.

**Señal de que lo estás usando bien:** Correr 'build' desde la raíz compila las librerías antes que las apps sin que lo configures por proyecto. Cambiar solo un app no re-buildea las librerías intactas (cache hit). Los scripts raíz de dev/build/test/lint/type-check son todos 'turbo run ...' (aunque el raíz tenga además scripts qa:* y ops:* que llaman node directamente para tooling fuera del grafo Turbo).

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #2 — apps/ son deployables, packages/ son librerías, y se nota en el build; #1 — El package interno se referencia con workspace:* y se importa con alias @scope/*

---

### ÁREA 2 — La puerta y el portero (autenticación y autorización)

#### TORNILLO #5 — JWT flaco para identidad, DB gorda para permisos

**En una línea:** El token dice quién sos; la base de datos, en cada request, dice qué podés hacer.

**Lo que hace el Senior:** El JWT incluye sub, org_id, roles, permissions y email (se arma idéntico en register, login, platformLogin, refreshAccessToken y switchOrganization — 6 lugares en auth.ts). Pero el hook requirePermission IGNORA el array de permisos del token y vuelve a consultar la DB (organization_users JOIN roles JOIN users con is_active=true y deleted_at IS NULL) para traer los permisos vivos. Recién ahí decide. Esto evita el problema clásico de 'le saqué un permiso a alguien pero su token viejo sigue funcionando 15 minutos': como se relee de la DB, el cambio de permisos pega instantáneo sin esperar a que expire el access token. Al final reescribe request.user con roles/permissions frescos para que el handler los use.

**Ejemplo real del repo:**

`packages/api/src/auth/permissions.ts:49-92`

```ts
export function requirePermission(required: string): FastifyHook {
  return async (request, reply) => {
    const user = request.user as any;
    if (!user?.sub || !user?.org_id) {
      return reply.code(401).send({ message: 'No autenticado' });
    }

    const result = await getPgPool().query(
      `SELECT r.name, COALESCE(r.permissions, ARRAY[]::TEXT[]) AS permissions
       FROM organization_users ou
       JOIN roles r ON r.id = ou.role_id
       JOIN users u ON u.id = ou.user_id
       WHERE ou.user_id = $1
         AND ou.organization_id = $2
         AND u.is_active = true
         AND u.deleted_at IS NULL
       LIMIT 1`,
```

`packages/api/src/services/auth.ts:289-295`

```ts
const jwtPayload: JwtPayload = {
        sub: user.id,
        org_id: organizationId,
        roles: [orgUser.role_name],
        permissions: orgUser.permissions ?? [],
        email: user.email,
      };
```

`ROLES_PERMISSIONS.md:8-8`

```md
El sistema ya no depende solo del nombre del rol en el JWT. Las rutas sensibles validan permisos efectivos desde la base de datos en cada request, usando `organization_users -> roles.permissions`.
```

**La regla para el próximo proyecto:** Mete en el JWT solo identidad estable (user id, tenant/org id, email) y como mucho un snapshot de permisos para mostrar UI. Para AUTORIZAR cada endpoint sensible, releé los permisos vivos desde la DB en un hook/middleware, filtrando por usuario activo y no borrado. Nunca dejes que un permiso revocado siga valiendo hasta que expire el token.

**Señal de que lo estás usando bien:** Si le sacás un permiso a un usuario en la DB y su próximo request (con el mismo access token vigente) ya recibe 403, lo aplicaste bien. Si tenés que esperar a que expire el token para que el cambio pegue, lo hiciste mal.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #8 — Catálogo de permisos como array de strings, fuente única compartida; #6 — Cadena onRequest: primero autenticar, después autorizar (y de paso cobrar)

---

#### TORNILLO #6 — Cadena onRequest: primero autenticar, después autorizar (y de paso cobrar)

**En una línea:** Cada ruta protegida compone dos hooks en orden fijo: identidad y luego permiso, con el gate de billing escondido adentro del segundo.

**Lo que hace el Senior:** Todas las rutas privadas usan onRequest: [fastify.authenticate, requirePermission('algo.read')]. authenticate verifica el JWT y planta request.user; requirePermission valida el permiso contra la DB. Dentro de requirePermission, ANTES de dejar pasar, llama a canUseOrganizationPermission y si el billing no está al día devuelve 402 (Payment Required) con billingStatus, no 403. Así el 'portero' y el 'cajero' viven en el mismo hook: tenés el permiso pero si la suscripción venció, no entrás igual. En las rutas con varios endpoints, el Senior hostea los hooks en constantes reutilizables (readAuth/manageAuth) para no repetir el array. El patrón aparece 35 veces en 18 archivos de rutas, así que es la convención de facto.

**Ejemplo real del repo:**

`packages/api/src/auth/permissions.ts:75-85`

```ts
if (!hasPermission(roleName, permissions, required)) {
      return reply.code(403).send({ message: `Permiso requerido: ${required}` });
    }

    const billingAccess = await canUseOrganizationPermission(user.org_id, roleName, required);
    if (!billingAccess.allowed) {
      return reply.code(402).send({
        message: 'La suscripción de Movete requiere atención',
        billingStatus: billingAccess.status,
      });
    }
```

`packages/api/src/routes/payments.ts:20-21`

```ts
const readAuth = { onRequest: [app.authenticate, requirePermission('payments.read')] } as any;
  const manageAuth = { onRequest: [app.authenticate, requirePermission('payments.manage')] } as any;
```

`packages/api/src/routes/auth.ts:1138-1140`

```ts
fastify.get(
    '/api/organizations/me',
    { onRequest: [fastify.authenticate, requirePermission('settings.read')] },
```

**La regla para el próximo proyecto:** Separá autenticación de autorización en dos hooks/middlewares distintos y aplicá los dos en orden (auth primero, permiso después) por ruta. Reutilizá constantes tipo readAuth/manageAuth en lugar de repetir el array en cada endpoint. Y elegí el status code correcto: 401 sin sesión, 403 sin permiso, 402 cuando el bloqueo es comercial (suscripción/billing), porque el front necesita distinguir 'no podés' de 'pagá'.

**Señal de que lo estás usando bien:** Una ruta nueva se protege agregando un solo item al array onRequest, sin tocar lógica de permisos. Y un usuario con permiso pero suscripción vencida recibe 402 (no 403 ni 200).

**Aplica a:** [ ] Universal  [ ] Solo TypeScript  [x] Solo este proyecto

**Depende de:** #5 — JWT flaco para identidad, DB gorda para permisos; #8 — Catálogo de permisos como array de strings, fuente única compartida

---

#### TORNILLO #7 — Access token en memoria, refresh token persistido y rotado

**En una línea:** El token de acceso vive solo en RAM; el de refresh se guarda y se cambia por uno nuevo cada vez que se usa.

**Lo que hace el Senior:** En el store de Zustand (igual en web y mobile) el partialize EXCLUYE el accessToken de la persistencia: solo sobreviven al reload refreshToken + user + organization. El access token (900s = 15 min, constante ACCESS_TOKEN_EXPIRES) queda en memoria; el refresh (2592000s = 30 días, REFRESH_TOKEN_EXPIRES) se persiste. Cuando llega un 401, el cliente intercambia el refresh por un par nuevo de tokens. Del lado server, refreshAccessToken hace ROTACIÓN: marca el refresh viejo como revocado (is_revoked=true, revoked_at=NOW()) y emite uno nuevo, así un refresh token solo se usa una vez. El cliente además deduplica refreshes concurrentes (una sola promesa en vuelo) para no disparar N rotaciones en paralelo. En web, SessionRestorer hace este intercambio silencioso al montar porque tras un hard reload el access token ya no está.

**Ejemplo real del repo:**

`apps/web-admin/store/auth.ts:83-89`

```ts
// accessToken is intentionally excluded — stored in memory only.
      // refreshToken + user + organization survive page reloads.
      partialize: (state) => ({
        refreshToken: state.refreshToken,
        user: state.user,
        organization: state.organization,
      }),
```

`apps/mobile/src/store/auth.ts:66-72`

```ts
// accessToken is kept in memory only (omit from persistence)
      partialize: (state) => ({
        refreshToken: state.refreshToken,
        user: state.user,
        organization: state.organization,
      }),
```

`packages/api/src/services/auth.ts:476-491`

```ts
// Rotate: revoke old token
    await pool.query(
      `UPDATE refresh_tokens SET is_revoked = true, revoked_at = NOW() WHERE id = $1`,
      [token.id],
    );

    // Issue new tokens
    const jwtPayload: JwtPayload = {
      sub: token.user_id,
      org_id: orgUser.organizationId,
      roles: [orgUser.role_name],
      permissions: orgUser.permissions ?? [],
      email: token.email,
    };
    const accessToken = this.jwtSign(jwtPayload);
    const refreshToken = await this.createRefreshTokenPg(token.user_id, orgUser.organizationId);
```

`apps/mobile/src/lib/api.ts:44-57`

```ts
// Deduplicate concurrent refresh calls
    if (!refreshing) {
      refreshing = axios
        .post(`${API_URL}/auth/refresh`, { refreshToken })
        .then((r) => {
          setAuth(r.data);
          return r.data.accessToken as string;
        })
        .catch((err) => {
          clearAuth();
          return Promise.reject(err);
        })
        .finally(() => { refreshing = null; });
    }
```

**La regla para el próximo proyecto:** Guardá solo el refresh token (en storage persistente) y mantené el access token en memoria. Hacé rotación de refresh tokens: cada uso revoca el viejo y emite uno nuevo. En el cliente, interceptá el 401, refrescá una sola vez (deduplicando llamadas concurrentes con una promesa compartida) y reintentá el request original; si el refresh falla, limpiá la sesión y mandá a login.

**Señal de que lo estás usando bien:** Si recargás la app y la sesión se restaura sola sin volver a pedir password (vía SessionRestorer), y un refresh token usado dos veces falla la segunda, está bien hecho. Si ves el access token en localStorage/AsyncStorage, está mal.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #5 — JWT flaco para identidad, DB gorda para permisos

---

#### TORNILLO #8 — Catálogo de permisos como array de strings, fuente única compartida

**En una línea:** Los permisos son strings tipo 'members.manage' definidos una sola vez en el paquete shared, con '*' como comodín y ADMIN siempre normalizado.

**Lo que hace el Senior:** Definió ALL_PERMISSIONS y DEFAULT_ROLE_PERMISSIONS en packages/shared/src/permissions.ts (recurso.acción: members.read, payments.manage...) y los exporta para que API y web los consuman sin duplicar la verdad. La regla de chequeo es simple y se repite casi idéntica en server y cliente: ADMIN pasa siempre, '*' pasa siempre, o el permiso exacto tiene que estar en el array (el front además trata aparte los permisos 'platform.' con su comodín 'platform.*'). ADMIN se normaliza SIEMPRE a ['*'] al sembrar roles (el UPDATE de ADMIN pisa siempre los permisos), mientras los demás roles solo reciben defaults si están vacíos (WHERE ... array_length(permissions,1)=0, para no pisar configs custom). El front además mapea rutas a permisos (ROUTE_PERMISSIONS) y replica hasPermission para gatear el menú/redirigir — defensa en profundidad, pero el server sigue siendo la autoridad.

**Ejemplo real del repo:**

`packages/shared/src/permissions.ts:24-55`

```ts
export type Permission = typeof ALL_PERMISSIONS[number]['key'] | '*';
...
export const DEFAULT_ROLE_PERMISSIONS: Record<string, Permission[]> = {
  ADMIN: ['*'],
  RECEPTIONIST: [
    'members.read',
    'members.manage',
    ...
  ],
  INSTRUCTOR: [ ... ],
  CUSTOMER: [
    'classes.read',
    'plans.read',
  ],
};
```

`packages/api/src/auth/permissions.ts:7-9`

```ts
function hasPermission(roleName: string, permissions: string[], required: string) {
  return roleName === 'ADMIN' || permissions.includes('*') || permissions.includes(required);
}
```

`apps/web-admin/lib/permissions.ts:24-30`

```ts
export function hasPermission(user: PermissionUser | null | undefined, permission: Permission) {
  const permissions = user?.permissions ?? [];
  if (permission.startsWith('platform.')) {
    return permissions.includes('platform.*') || permissions.includes(permission);
  }
  return user?.role === 'ADMIN' || permissions.includes('*') || permissions.includes(permission);
}
```

`packages/api/src/auth/permissions.ts:31-44`

```ts
if (roleName === 'ADMIN') {
      await pool.query(
        `UPDATE roles
         SET permissions = $1::TEXT[], is_system = true, updated_at = NOW()
         WHERE name = 'ADMIN'`,
        [permissions],
      );
    } else {
      await pool.query(
        `UPDATE roles
         SET permissions = $1::TEXT[], is_system = true, updated_at = NOW()
         WHERE name = $2::VARCHAR AND COALESCE(array_length(permissions, 1), 0) = 0`,
        [permissions, roleName],
      );
    }
```

**La regla para el próximo proyecto:** Modelá permisos como strings recurso.acción (members.read) en UN solo módulo compartido entre back y front, con un comodín '*' y un rol super-admin que siempre se normaliza a ['*']. La función de chequeo (admin OR comodín OR match exacto) debe ser la misma en ambos lados. Al sembrar roles por defecto, pisá siempre al admin pero respetá los permisos ya configurados de los demás roles (solo completá si están vacíos, con un WHERE sobre array_length).

**Señal de que lo estás usando bien:** Agregar un permiso nuevo es agregar un string al catálogo del paquete shared y back y front lo reconocen sin cambios de tipos en otro lado. El rol admin nunca queda sin acceso aunque alguien edite roles en la UI.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #5 — JWT flaco para identidad, DB gorda para permisos; #6 — Cadena onRequest: primero autenticar, después autorizar (y de paso cobrar)

---

### ÁREA 2b — Mismo dato, distinta respuesta (multi-tenancy y Row Level Security)

#### TORNILLO #9 — El org_id sale SIEMPRE del JWT, nunca del cliente

**En una línea:** El identificador de tenant que filtra los datos se toma del token firmado (request.user.org_id), nunca de un parámetro o body que el cliente pueda manipular.

**Lo que hace el Senior:** En cada handler hace `const user = request.user as any` y usa `user.org_id` como primer parámetro del WHERE. El tenancyMiddleware valida que el token traiga org_id y, si falta, corta con 401 antes de tocar la base (fail-closed, código INVALID_TOKEN). El cliente puede mandar memberId, classId, etc., pero el tenant viene del token verificado por @fastify/jwt, así que un atacante no puede pedir datos de otra organización cambiando un campo del request. Los :id de params solo se usan DENTRO del scope del org (siempre acompañados de `AND organization_id = $org` / `AND ou.organization_id = $org`).

**Ejemplo real del repo:**

`packages/api/src/middleware/tenancy.ts:25-34`

```ts
// CRITICAL: Verify org_id exists in token
    if (!payload.org_id) {
      return reply.status(401).send({
        success: false,
        error: {
          code: 'INVALID_TOKEN',
          message: 'Token missing organization context',
```

`packages/api/src/routes/reservations.ts:49-51`

```ts
      WHERE r.organization_id = $1`;

    const params: any[] = [user.org_id];
```

`packages/api/src/routes/members.ts:268-280`

```ts
    const { id } = request.params as { id: string };
...
       WHERE u.id = $1 AND ou.organization_id = $2 AND u.deleted_at IS NULL`,
      [id, user.org_id],
```

**La regla para el próximo proyecto:** El identificador de tenant (org/cuenta/workspace) SIEMPRE se extrae del token de sesión ya verificado, nunca de query params, body ni headers controlables por el cliente. Si el token no trae tenant, rechazá la request con 401 antes de tocar la base. Los IDs que vienen del cliente (params de ruta, body) se usan solo como filtro adicional, siempre combinados con `AND tenant_id = <del token>`.

**Señal de que lo estás usando bien:** No existe ninguna query de negocio donde el tenant_id provenga de request.params, request.body o request.query. Buscás 'org_id' en un route y todas las ocurrencias apuntan a user.org_id / request.user. Cambiar un :id en la URL por uno de otro tenant devuelve 404, no datos ajenos.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #11 — Fail-closed: sin contexto de tenant, cero datos; #10 — Cada query carga su propio WHERE organization_id (filtro app-layer omnipresente)

---

#### TORNILLO #10 — Cada query carga su propio WHERE organization_id (filtro app-layer omnipresente)

**En una línea:** Toda consulta a una tabla de negocio incluye organization_id en el WHERE (y en el INSERT/UPDATE/DELETE), sin excepción, como disciplina manual repetida.

**Lo que hace el Senior:** Como los routes corren sobre un Pool de pg crudo (getPgPool, no Prisma) y por lo tanto la RLS de Postgres no se activa en esas conexiones, el Senior compensa filtrando a mano en cada SELECT/INSERT/UPDATE/DELETE. El patrón se repite por todos los routes: los WHERE llevan organization_id, los INSERT incluyen la columna organization_id con el valor del token, y hasta los DELETE de notas exigen `AND organization_id = $org`. El tenant también se propaga explícitamente a la capa de servicios: restoreReservationAccess recibe organizationId como primer parámetro y lo usa en su WHERE, y consumeReservationAccess lo recibe dentro de su input (input.organizationId).

**Ejemplo real del repo:**

`packages/api/src/routes/members.ts:589-592`

```ts
      `DELETE FROM member_notes
       WHERE id = $1::UUID AND user_id = $2::UUID AND organization_id = $3::UUID
       RETURNING id`,
      [noteId, id, user.org_id],
```

`packages/api/src/routes/reservations.ts:305-307`

```ts
        `INSERT INTO reservations (id, organization_id, user_id, class_id, reservation_date, status, notes)
         VALUES ($1, $2, $3, $4, $5, 'CONFIRMED', $6)`,
        [id, user.org_id, user.sub, b.classId, b.reservationDate, b.notes ?? null],
```

`packages/api/src/routes/reservations.ts:581-583`

```ts
         WHERE id = $2 AND organization_id = $3
         RETURNING id`,
        [status, id, user.org_id],
```

`packages/api/src/services/membership-access.ts:178-193`

```ts
export async function restoreReservationAccess(
  organizationId: string,
...
       WHERE r.id = $1::UUID AND r.organization_id = $2::UUID
       LIMIT 1`,
      [reservationId, organizationId],
```

**La regla para el próximo proyecto:** En un sistema multi-tenant, TODA query a una tabla con datos de tenant lleva `WHERE tenant_id = $x` y todo INSERT/UPDATE setea/verifica esa columna, sin atajos. No confíes en que un :id sea único globalmente: combinalo siempre con el tenant. Propagá el tenant_id también a las funciones de servicio (recibilo como parámetro explícito o dentro del input), no lo dejes implícito. Si tu ORM no garantiza el filtro automáticamente, el filtro manual no es opcional: es la única barrera real.

**Señal de que lo estás usando bien:** grep de tenant_id en cualquier archivo de routes devuelve una ocurrencia por cada operación de DB (selects, inserts, updates, deletes). No hay un solo SELECT/UPDATE/DELETE sobre tabla de negocio que omita el filtro de tenant. Un test que intenta tocar un id de otro tenant devuelve 0 filas / 404.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #9 — El org_id sale SIEMPRE del JWT, nunca del cliente

---

#### TORNILLO #11 — Fail-closed: sin contexto de tenant, cero datos

**En una línea:** Si falta el contexto de organización, el sistema devuelve vacío o rechaza, nunca 'todo' — el default seguro es negar.

**Lo que hace el Senior:** El principio está escrito explícitamente en el middleware ('Security principle: Fail closed') y se materializa en dos niveles. En app-layer: si el JWT no trae org_id, 401 inmediato antes de cualquier query. En DB-layer (RLS de Postgres, donde el contexto SÍ se setea): las policies usan current_setting('app.organization_id') y, sin ese GUC, las comparaciones no matchean y la query devuelve 0 filas en vez de filtrarse mal. El comentario del test suite lo documenta como nota para devs: 'If you forget to set context, queries return empty (fail-closed)'. (Ojo: la nota describe el diseño deseado; en runtime los routes de datos no setean ese GUC — ver tornillo de RLS.)

**Ejemplo real del repo:**

`packages/api/src/middleware/tenancy.ts:13`

```ts
 * Security principle: Fail closed (reject if org_id missing or mismatched)
```

`packages/api/src/db/rls.test.ts:171-180`

```ts
    it('should fail-closed when organization_id is missing', async () => {
      // Don't set RLS context
      
      const result = await prisma.$queryRaw`
        SELECT COUNT(*) as count FROM organizations;
      `;
      
      // Should return 0 or throw due to missing context
      expect((result as any)[0].count).toBe(0);
```

`packages/api/src/db/rls.test.ts:274-275`

```ts
 * 3. If you forget to set context, queries return empty (fail-closed)
 * 4. Never disable RLS or trust app-layer filtering alone
```

**La regla para el próximo proyecto:** Diseñá el aislamiento de forma que la AUSENCIA de contexto de tenant produzca cero resultados o un rechazo explícito, jamás acceso total. En RLS de Postgres esto sale gratis: una policy con `tenant_id = current_setting('app.tenant_id')::uuid` no matchea nada si el GUC no está seteado. En app-layer, validá la presencia del tenant ANTES de la query y cortá con 401. La pregunta de diseño es siempre: 'si esto falla a medias, ¿se filtran datos o se niegan?' — la respuesta correcta es 'se niegan'.

**Señal de que lo estás usando bien:** Un test que NO setea el contexto de tenant y corre una query devuelve 0 filas (no todas). Quitar el org_id del token produce 401, no una respuesta con datos. No hay ningún code path donde 'tenant indefinido' se traduzca en 'sin filtro'.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #9 — El org_id sale SIEMPRE del JWT, nunca del cliente; #12 — RLS de Postgres con GUC como red de seguridad de DB (defensa en profundidad)

---

#### TORNILLO #12 — RLS de Postgres con GUC como red de seguridad de DB (defensa en profundidad)

**En una línea:** Postgres aplica policies por fila usando variables de sesión (set_config('app.organization_id')) para que, aun bypaseando la app, la base rechace datos de otro tenant — pero hoy está desconectado del camino real.

**Lo que hace el Senior:** Construyó un wrapper withRLSContext que setea dos GUC (variables de sesión de Postgres) — app.user_id y app.organization_id — vía set_config antes de la query y las limpia en el finally. Las policies SQL comparan organization_id contra current_setting('app.organization_id')::uuid en SELECT/INSERT(WITH CHECK)/UPDATE/DELETE. La idea era que esta fuera la 'Layer 4': aunque fallen JWT, middleware y filtros app-layer, Postgres no deja leer ni escribir cruzado. BRECHA REAL verificada: setRLSContext/withRLSContext SOLO se invocan en services/auth.ts y en rls.test.ts; ningún route de datos los usa. Los routes corren sobre getPgPool() (Pool de pg crudo, adoptado por un bug de Prisma en Windows) donde el GUC nunca se setea — incluso auth.ts tiene un comentario 'skip Prisma-based setRLSContext — use pg directly'. O sea: hoy la RLS NO protege esos endpoints y toda la carga recae en el filtro app-layer. Nota: el test 'should NOT see other organizations data without RLS context' es débil/ambiguo (solo verifica que el resultado sea un array, no que vuelvan o no las filas), así que NO demuestra la protección.

**Ejemplo real del repo:**

`packages/api/src/db/rls-context.ts:12-19`

```ts
export async function setRLSContext(
  prisma: PrismaClient,
  userId: string,
  organizationId: string
): Promise<void> {
  await prisma.$executeRaw`SELECT set_config('app.user_id', ${userId}, FALSE);`;
  await prisma.$executeRaw`SELECT set_config('app.organization_id', ${organizationId}, FALSE);`;
}
```

`packages/api/src/db/rls-context.ts:46-57`

```ts
  try {
    // Set context for RLS policies
    await setRLSContext(prisma, userId, organizationId);
    
    // Execute query (now protected by RLS)
    const result = await queryFn(prisma);
    
    return result;
  } finally {
    // Clean up
    await clearRLSContext(prisma);
  }
```

`docs/archive/DATABASE_RLS.md:49-60`

```md
CREATE POLICY org_isolation_policy ON reservations
  FOR SELECT
  USING (
    organization_id = current_setting('app.organization_id')::uuid
  );
...
CREATE POLICY org_insert_policy ON reservations
  FOR INSERT
  WITH CHECK (
    organization_id = current_setting('app.organization_id')::uuid
```

`packages/api/src/db/prisma.ts:5-11`

```ts
// --- Raw pg Pool (used instead of Prisma due to Windows env bug) ---
let pgPool: Pool | null = null;

export function getPgPool(): Pool {
  if (!pgPool) {
    pgPool = new Pool({
      connectionString: process.env.DATABASE_URL || ...
```

`packages/api/src/services/auth.ts:288`

```ts
      // Generate tokens (skip Prisma-based setRLSContext — use pg directly)
```

**La regla para el próximo proyecto:** Tratá la RLS de Postgres como la ÚLTIMA línea de defensa, no la primera: policies por fila con `tenant_id = current_setting('app.tenant_id')::uuid` y un wrapper que setee el GUC con set_config(...) antes de cada query y lo limpie en un finally. PERO: la RLS solo protege la conexión donde el GUC está seteado. Si usás un pool crudo o un cliente distinto del que setea el contexto, o conexiones de un pool que se reciclan entre requests, la RLS NO te cubre — verificá con un test de integración que un query SIN contexto sobre un endpoint REAL devuelve vacío (y que ese test asserte filas==0, no solo que el resultado sea un array). No declares 'protegido por RLS' algo que en runtime nunca setea el contexto: es seguridad teórica.

**Señal de que lo estás usando bien:** El wrapper que setea el GUC se usa en TODA conexión que toca datos de tenant (no solo en auth y tests). Un test de integración contra un endpoint real, con el contexto deshabilitado, asserta 0 filas. Si grep de setRLSContext/withRLSContext solo aparece en services/auth y *.test.ts pero los routes usan getPgPool(), es señal de ALERTA: la RLS está desconectada del camino real.

**Aplica a:** [ ] Universal  [ ] Solo TypeScript  [x] Solo este proyecto

**Depende de:** #11 — Fail-closed: sin contexto de tenant, cero datos; #10 — Cada query carga su propio WHERE organization_id (filtro app-layer omnipresente)

---

### ÁREA 3 — El viaje de un dato (de la pantalla a la base y de vuelta)

#### TORNILLO #13 — La transacción como guardián de la regla de negocio

**En una línea:** Toda escritura que toca varias tablas o consume un recurso limitado corre dentro de BEGIN/COMMIT y hace ROLLBACK ante cualquier fallo, sea de DB o de validación de negocio.

**Lo que hace el Senior:** En POST /api/reservations/me abre un cliente dedicado del pool, hace BEGIN, y dentro de la transacción: re-chequea si ya existe reserva, inserta/reactiva la reserva, consume el crédito de membresía (consumeReservationAccess) y borra de la waitlist. Lo clave: si consumeReservationAccess devuelve {ok:false} (ej: no quedan créditos, actividad no incluida), hace ROLLBACK manual ANTES de responder con el statusCode que vino del servicio. O sea, un fallo de regla de negocio dispara el mismo rollback que un error de DB. Además consumeReservationAccess usa SELECT ... FOR UPDATE OF mm para lockear la fila de membresía y evitar que dos reservas concurrentes gasten el mismo crédito. El catch captura el código 23505 (unique violation) de Postgres y lo traduce a un 409 humano.

**Ejemplo real del repo:**

`packages/api/src/routes/reservations.ts:255-286`

```ts
await client.query('BEGIN');
...
          const access = await consumeReservationAccess(client, {
...
          if (!access.ok) {
            await client.query('ROLLBACK');
            return reply.code(access.statusCode).send({ message: access.message });
          }
```

`packages/api/src/routes/reservations.ts:330-338`

```ts
} catch (err: any) {
      await client.query('ROLLBACK');
      if (err.code === '23505') {
        return reply.code(409).send({ message: 'Ya tenés una reserva para esta clase en esa fecha' });
      }
      throw err;
    } finally {
      client.release();
    }
```

`packages/api/src/services/membership-access.ts:87-89`

```ts
       mm.end_date ASC,
       mm.created_at ASC
     FOR UPDATE OF mm`,
```

**La regla para el próximo proyecto:** Cuando una operación de escritura toque más de una tabla o consuma un recurso finito (créditos, stock, cupo), envolvela en una transacción explícita (BEGIN/COMMIT). Diseñá los servicios de negocio para que devuelvan un resultado tipo {ok:false, statusCode, message} en vez de tirar excepción, y que el handler haga ROLLBACK explícito ante {ok:false} igual que ante un error de DB. Lockeá la fila del recurso escaso con SELECT ... FOR UPDATE. Capturá los códigos de error nativos de la DB (ej 23505 unique violation) y traducilos a respuestas HTTP entendibles.

**Señal de que lo estás usando bien:** Si simulás dos reservas concurrentes sobre la última clase de un pack, solo una consume el crédito y la otra recibe el statusCode de negocio (ej 402); y si la validación de negocio falla a mitad de camino, no queda NINGUNA fila insertada ni crédito gastado.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #29 — Fail-closed multi-tenant: nunca una query sin org_id + soft delete; #15 — Validar con Zod safeParse antes de tocar la DB

---

#### TORNILLO #14 — organization_id en cada query, sin excepción

**En una línea:** Cada consulta SQL filtra por organization_id (y por user_id en endpoints /me) sacándolo del JWT, nunca del body del request.

**Lo que hace el Senior:** El handler lee user.org_id y user.sub directamente del token decodificado (request.user), nunca confía en un orgId que venga del cliente. Cada SELECT/UPDATE/DELETE incluye WHERE organization_id = $1 como primer filtro, y los endpoints self-service agregan AND user_id = $2. El servicio membership-access repite el mismo patrón en todas sus subqueries. El filtro está internalizado: aparece cientos de veces a lo largo de todos los archivos de rutas (un grep de organization_id/org_id da más de 400 ocurrencias en 23 archivos). El JwtPayload define org_id como campo de primera clase con el comentario 'CRITICAL: multi-tenancy'.

**Ejemplo real del repo:**

`packages/api/src/routes/reservations.ts:138-139`

```ts
let where = `WHERE r.organization_id = $1 AND r.user_id = $2`;
    const params: any[] = [user.org_id, user.sub];
```

`packages/api/src/routes/reservations.ts:353-356`

```ts
       WHERE r.id = $1::UUID
         AND r.organization_id = $2::UUID
         AND r.user_id = $3::UUID`,
      [id, user.org_id, user.sub],
```

`packages/shared/src/types.ts:124-132`

```ts
export interface JwtPayload {
  sub: string; // user_id
  org_id: string; // organization_id (CRITICAL: multi-tenancy)
```

**La regla para el próximo proyecto:** En un SaaS multi-tenant, el identificador de tenant (y el de usuario en endpoints self-service) SIEMPRE sale del token autenticado, jamás del body o query params del cliente. Hacé que el primer WHERE de toda query sea el filtro de tenant. Tratá un endpoint sin ese filtro como un bug de seguridad, no como un detalle de performance.

**Señal de que lo estás usando bien:** No existe ninguna query de lectura o escritura de datos de negocio sin WHERE organization_id; un usuario del tenant A nunca puede leer ni modificar filas del tenant B aunque mande el id correcto.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #13 — La transacción como guardián de la regla de negocio

---

#### TORNILLO #15 — Validar con Zod safeParse antes de tocar la DB

**En una línea:** El esquema Zod corre con safeParse al inicio del handler y, si falla, devuelve 400 con errores por campo ANTES de cualquier acceso a datos.

**Lo que hace el Senior:** Define los schemas en packages/shared (reutilizables por web y mobile) y/o inline en la propia ruta (ej CreateReservationSelfSchema está definido inline en reservations.ts). En el handler hace CreateReservationSelfSchema.safeParse(request.body); si !parsed.success responde 400 con parsed.error.flatten().fieldErrors y corta. Recién entonces usa parsed.data (tipado). Los schemas codifican reglas de dominio reales: reservationDate con z.string().date(), notes con max(500), y refinements cruzados como isEndTimeAfterStartTime o que un CLASS_PACK requiera cantidad de clases (validatePlan/superRefine). El tipo se deriva del schema con z.infer, así nunca se desincronizan validación y tipo.

**Ejemplo real del repo:**

`packages/api/src/routes/reservations.ts:201-205`

```ts
const parsed = CreateReservationSelfSchema.safeParse(request.body);
    if (!parsed.success) {
      return reply.code(400).send({ message: 'Datos inválidos', errors: parsed.error.flatten().fieldErrors });
    }
    const b = parsed.data;
```

`packages/shared/src/validations.ts:160-167`

```ts
export const CreateReservationAdminSchema = z.object({
  userId: z.string().uuid(),
  classId: z.string().uuid(),
  reservationDate: z.string().date(),
  notes: z.string().max(500).optional(),
});

export type CreateReservationAdminInput = z.infer<typeof CreateReservationAdminSchema>;
```

`packages/shared/src/validations.ts:281-300`

```ts
function validatePlan(data: z.infer<typeof PlanBaseSchema>, ctx: z.RefinementCtx) {
  const mode = data.mode ?? (data.maxClasses ? 'CLASS_PACK' : 'FREE_PACK');
...
      message: 'El Pack de clases requiere cantidad de clases',
```

**La regla para el próximo proyecto:** Definí los esquemas de validación en un paquete compartido (o inline en la ruta si son específicos) y derivá los tipos TS con z.infer (una sola fuente de verdad). En cada handler validá la entrada con safeParse y respondé 400 con errores por campo ANTES de tocar la base. Meté las reglas de dominio (rangos, formatos, dependencias entre campos) en refinements/superRefine del schema, no desperdigadas en el handler.

**Señal de que lo estás usando bien:** Un body malformado nunca llega a ejecutar una query; el cliente recibe 400 con un objeto de errores por campo, y el tipo de parsed.data coincide exactamente con lo que validó el schema sin casts manuales.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #13 — La transacción como guardián de la regla de negocio

---

#### TORNILLO #16 — El cliente tolera el envelope, no lo asume

**En una línea:** El frontend acepta tanto una lista cruda como un objeto { data: [...] } gracias a un helper unwrapList, desacoplándose de la forma exacta de la respuesta.

**Lo que hace el Senior:** En useApi.ts el tipo ListResponse<T> = T[] | { data: T[] } y la función unwrapList devuelve Array.isArray(payload) ? payload : payload.data. Cada query de lista (reservas, clases, plans, activities) pasa por unwrapList, así el endpoint puede devolver { data, total, page, ... } (como /reservations/me) o un array pelado sin romper el cliente. El mismo principio aparece en el update optimista de useCancelReserva, que vuelve a chequear Array.isArray del cache para escribir correctamente en cualquiera de las dos formas.

**Ejemplo real del repo:**

`apps/mobile/src/hooks/useApi.ts:242-246`

```ts
type ListResponse<T> = T[] | { data: T[] };

function unwrapList<T>(payload: ListResponse<T>): T[] {
  return Array.isArray(payload) ? payload : payload.data;
}
```

`apps/mobile/src/hooks/useApi.ts:320-323`

```ts
queryFn: async () => {
      const { data } = await api.get<ListResponse<Reservation>>('/api/reservations/me?limit=50');
      return unwrapList(data);
    },
```

`apps/mobile/src/hooks/useApi.ts:346-355`

```ts
qc.setQueryData<ListResponse<Reservation>>(['reservas'], (current) => {
        if (!current) return current;
        if (Array.isArray(current)) {
          return current.map((item) => item.id === id ? { ...item, status: 'CANCELLED' } : item);
        }
```

**La regla para el próximo proyecto:** En el cliente, nunca asumas una única forma de respuesta para las listas: definí un tipo unión (array | {data:array}) y un helper de unwrap que normalice antes de devolver del queryFn. Así el frontend sobrevive si el backend cambia entre envelope y array crudo, y los componentes siempre reciben un array limpio.

**Señal de que lo estás usando bien:** Si un endpoint cambia de devolver [...] a { data:[...] } (o viceversa), ningún componente de UI se rompe ni hay que tocar el render; solo el helper de unwrap conoce la forma del envelope.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #17 — Invalidación dirigida de React Query con actualización optimista

---

#### TORNILLO #17 — Invalidación dirigida de React Query con actualización optimista

**En una línea:** Cada mutación, en onSuccess, invalida solo las queryKeys afectadas y para acciones reversibles parchea el cache al instante antes de refetchear.

**Lo que hace el Senior:** Las queries usan queryKeys estables y semánticas (['reservas'], ['clases'], ['waitlist']). Cada mutación declara en onSuccess exactamente qué keys quedaron obsoletas: crear reserva invalida reservas + clases + waitlist (porque cambia cupo y te saca de la espera). Para cancelar (acción que el usuario quiere ver reflejada YA) primero hace setQueryData optimista marcando status CANCELLED en el cache local y DESPUÉS invalida reservas + clases para reconciliar con el server. El cache se persiste en AsyncStorage con staleTime de 2 min y networkMode offlineFirst, así la app abre con datos aunque no haya red. Al cambiar de organización (logout/switch), en cambio, hace qc.clear() total para no mezclar tenants.

**Ejemplo real del repo:**

`apps/mobile/src/hooks/useApi.ts:327-337`

```ts
export function useCreateReserva() {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (payload: { classId: string; reservationDate: string }) =>
      api.post('/api/reservations/me', payload),
    onSuccess: () => {
      qc.invalidateQueries({ queryKey: ['reservas'] });
      qc.invalidateQueries({ queryKey: ['clases'] });
      qc.invalidateQueries({ queryKey: ['waitlist'] });
    },
```

`apps/mobile/src/hooks/useApi.ts:356-358`

```ts
      qc.invalidateQueries({ queryKey: ['reservas'] });
      qc.invalidateQueries({ queryKey: ['clases'] });
    },
```

`apps/mobile/src/lib/queryClient.ts:7-12`

```ts
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 2,   // 2 minutes
      gcTime: 1000 * 60 * 60 * 24,
      retry: 2,
      networkMode: 'offlineFirst',
```

`apps/mobile/src/hooks/useApi.ts:459-474`

```ts
      qc.clear();
...
      qc.clear();
```

**La regla para el próximo proyecto:** Usá queryKeys estables y semánticas, una por recurso. En el onSuccess de cada mutación invalidá EXPLÍCITAMENTE todas las keys que esa acción dejó obsoletas (incluí los recursos indirectos, ej: reservar afecta también el cupo de la clase). Para acciones reversibles que el usuario debe ver al instante (cancelar, marcar leído), hacé un setQueryData optimista y después invalidá para reconciliar. Al cambiar de tenant/usuario, limpiá el cache entero con clear() en vez de invalidar key por key.

**Señal de que lo estás usando bien:** Tras reservar, la lista de reservas, el cupo de la clase y la waitlist se actualizan solas sin recargar la pantalla; al cancelar, la tarjeta pasa a 'Cancelada' instantáneamente; y al cambiar de organización no aparece ni un dato del tenant anterior.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #16 — El cliente tolera el envelope, no lo asume; #27 — Refresh silencioso de token en 401 con deduplicación (mismo patrón en mobile y web)

---

### ÁREA 4 — El archivo del negocio (diseño de base de datos)

#### TORNILLO #18 — La plantilla de tabla multi-tenant (el molde que se copia y pega)

**En una línea:** Cada tabla de negocio nace con el mismo esqueleto de columnas y metadatos, no se improvisa modelo por modelo.

**Lo que hace el Senior:** Definió un patrón fijo que repite en TODAS las tablas de negocio: id String @id @default(cuid()) como PK opaca (no autoincrement), organization_id String como discriminador de tenant (inquilino), y la tripleta de auditoría created_at (@default(now())), updated_at (@updatedAt) y deleted_at DateTime? para soft-delete (borrado lógico: no se borra la fila, se marca con fecha). Las columnas van en snake_case y los campos string acotados llevan tipo nativo de Postgres explícito (@db.VarChar(100), @db.VarChar(255)). El dinero NUNCA es Float: usa Decimal @db.Decimal(10, 2). Esto se ve idéntico en Organization, User, Facility, Class, Reservation y Payment.

**Ejemplo real del repo:**

`packages/api/prisma/schema.prisma:176-205`

```prisma
model Facility {
  id                      String    @id @default(cuid())
  organization_id         String
  
  name                    String    @db.VarChar(255)
```

`packages/api/prisma/schema.prisma:285-291`

```prisma
model Payment {
  id                      String    @id @default(cuid())
  organization_id         String
  user_id                 String?   // NULL si es pago de empresa
  
  // Payment details
  amount                  Decimal   @db.Decimal(10, 2)
```

`packages/api/prisma/schema.prisma:89-91`

```prisma
  created_at              DateTime  @default(now())
  updated_at              DateTime  @updatedAt
  deleted_at              DateTime?
```

**La regla para el próximo proyecto:** Definí un molde de tabla estándar y aplicalo sin excepción a toda entidad de negocio: PK opaca (cuid/uuid, nunca int autoincremental en SaaS multi-tenant), columna de tenant (organization_id) en cada tabla que contenga datos de cliente, y siempre created_at + updated_at + deleted_at para auditoría y soft-delete. Para dinero usá Decimal con precisión y escala explícitas, jamás Float. Antes de agregar una tabla, copiá el molde primero y después agregá los campos específicos.

**Señal de que lo estás usando bien:** Podés abrir cualquier modelo nuevo y encontrar las mismas 4-5 columnas base; ningún campo de dinero es Float; ninguna tabla de negocio carece de organization_id ni de la tripleta de timestamps.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #19 — Índices deliberados sobre FK y deleted_at (no dejar que Postgres adivine); #20 — onDelete con intención: Cascade para hijos, SetNull para histórico

---

#### TORNILLO #19 — Índices deliberados sobre FK y deleted_at (no dejar que Postgres adivine)

**En una línea:** Indexa explícitamente cada clave foránea, el discriminador de tenant, las columnas de filtro frecuente y deleted_at, y usa uniques compuestos para reglas de negocio.

**Lo que hace el Senior:** En vez de confiar en índices automáticos, declara @@index a mano para cada FK (organization_id, user_id, class_id, facility_id), para columnas que se filtran seguido (status, reservation_date, expired_at) y para deleted_at en las tablas con soft-delete (porque las queries soft-delete filtran WHERE deleted_at IS NULL). Además codifica reglas de negocio como @@unique compuestos: un usuario no puede reservar dos veces la misma clase el mismo día (@@unique([user_id, class_id, reservation_date])), un rol es único por org (@@unique([organization_id, name])), un usuario pertenece una sola vez a una org (@@unique([organization_id, user_id])).

**Ejemplo real del repo:**

`packages/api/prisma/schema.prisma:272-277`

```prisma
  @@unique([user_id, class_id, reservation_date])
  @@index([organization_id])
  @@index([user_id])
  @@index([class_id])
  @@index([status])
  @@index([reservation_date])
```

`packages/api/prisma/schema.prisma:56-58`

```prisma
  @@index([slug])
  @@index([subscription_status])
  @@index([deleted_at])
```

`packages/api/prisma/schema.prisma:371-374`

```prisma
  @@unique([organization_id, email])
  @@index([organization_id])
  @@index([email])
  @@index([expired_at])
```

**La regla para el próximo proyecto:** Indexá a mano: toda FK, la columna de tenant, toda columna que aparezca recurrentemente en WHERE/ORDER BY, y deleted_at si usás soft-delete. Convertí cada invariante de negocio del tipo 'no puede haber dos X iguales' en un @@unique compuesto en la base, no solo en la lógica de la app. No dependas de que el motor cree los índices por vos.

**Señal de que lo estás usando bien:** No existe una FK sin su @@index correspondiente; las reglas de unicidad del dominio están reflejadas como uniques compuestos en el schema; las queries con WHERE deleted_at IS NULL no hacen seq scan.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #18 — La plantilla de tabla multi-tenant (el molde que se copia y pega)

---

#### TORNILLO #20 — onDelete con intención: Cascade para hijos, SetNull para histórico

**En una línea:** La política de borrado en cascada se elige según si el dato es accesorio (cae con el padre) o evidencia que debe sobrevivir (se desliga, no se borra).

**Lo que hace el Senior:** No pone onDelete al voleo. Usa Cascade en las relaciones de pertenencia donde el hijo no tiene sentido sin el padre (OrganizationUser, Facility, Class, Reservation, RefreshToken cuelgan de su Organization/User y se borran con él). Pero en datos que son evidencia o registro financiero/legal (Payment, AuditLog) usa SetNull sobre user_id: si se borra el usuario, el pago y el log de auditoría sobreviven con user_id en NULL, preservando el histórico. Es coherente con que esas columnas sean nullable (user_id String?).

**Ejemplo real del repo:**

`packages/api/prisma/schema.prisma:307-308`

```prisma
  organization            Organization @relation(fields: [organization_id], references: [id], onDelete: Cascade)
  user                    User? @relation(fields: [user_id], references: [id], onDelete: SetNull)
```

`packages/api/prisma/schema.prisma:339-340`

```prisma
  organization            Organization @relation(fields: [organization_id], references: [id], onDelete: Cascade)
  user                    User? @relation(fields: [user_id], references: [id], onDelete: SetNull)
```

`packages/api/prisma/schema.prisma:116-118`

```prisma
  organization            Organization @relation(fields: [organization_id], references: [id], onDelete: Cascade)
  user                    User @relation(fields: [user_id], references: [id], onDelete: Cascade)
  role                    Role @relation(fields: [role_id], references: [id])
```

**La regla para el próximo proyecto:** Para cada relación decidí explícitamente el onDelete según la naturaleza del dato: Cascade cuando el hijo carece de sentido sin el padre (membresías, tokens, configuraciones), SetNull cuando el dato es evidencia que debe persistir aunque el actor desaparezca (pagos, logs de auditoría, registros legales). Cuando elijas SetNull, la columna FK debe ser nullable. Nunca dejes el onDelete por defecto sin pensarlo.

**Señal de que lo estás usando bien:** Borrar un usuario no destruye sus pagos ni los logs de auditoría (quedan con FK en NULL); borrar una organización limpia todos sus datos operativos sin dejar huérfanos.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #18 — La plantilla de tabla multi-tenant (el molde que se copia y pega)

---

#### TORNILLO #21 — RLS como última línea fail-closed (defense in depth en la base)

**En una línea:** Aunque la app ya filtra por tenant, PostgreSQL fuerza el aislamiento con Row-Level Security que devuelve vacío si falta el contexto.

**Lo que hace el Senior:** Asume que las capas de la app (JWT, middleware, query) pueden fallar o ser bypaseadas, así que pone una cuarta capa en la base: activa ROW LEVEL SECURITY en todas las tablas y crea políticas que comparan organization_id contra current_setting('app.organization_id'). El contexto se inyecta por request con current_setting antes de cada query. La gracia es que es fail-closed: si nadie setea el contexto, current_user_id() captura la excepción (EXCEPTION WHEN OTHERS) y devuelve NULL, con lo cual las políticas no matchean nada y la query devuelve cero filas en vez de filtrar de más. Los audit_logs además se diseñan inmutables: solo tienen políticas de SELECT e INSERT, con un comentario explícito 'Audit logs are immutable (no update/delete)'. El documento de setup lo enmarca como 'Defense in Depth' con 4 capas.

**Ejemplo real del repo:**

`packages/api/prisma/migrations/002_add_rls_policies.sql:9-15`

```
CREATE OR REPLACE FUNCTION current_user_id() RETURNS UUID AS $$
BEGIN
  RETURN current_setting('app.user_id')::UUID;
EXCEPTION WHEN OTHERS THEN
  RETURN NULL::UUID;
END;
$$ LANGUAGE plpgsql STABLE;
```

`packages/api/prisma/migrations/002_add_rls_policies.sql:208-213`

```
CREATE POLICY reservations_insert_policy ON reservations
  FOR INSERT
  WITH CHECK (
    organization_id = current_setting('app.organization_id')::UUID
    AND user_id = current_user_id()
  );
```

`packages/api/prisma/migrations/002_add_rls_policies.sql:282-288`

```
CREATE POLICY audit_logs_insert_policy ON audit_logs
  FOR INSERT
  WITH CHECK (
    organization_id = current_setting('app.organization_id')::UUID
  );

-- Audit logs are immutable (no update/delete)
```

`docs/archive/PRISMA_SETUP.md:36-64`

```md
### Principio: Defense in Depth

**Capa 1: JWT Token**
...
**Capa 4: Row-Level Security (RLS)**
- PostgreSQL enforces si alguien bypasea layers 1-3
```

**La regla para el próximo proyecto:** En SaaS multi-tenant no confíes solo en el WHERE organization_id de la app: activá RLS en la base y escribí políticas que comparen el tenant contra una variable de sesión inyectada por request (current_setting/set_config). Hacelo fail-closed: la función que lee el contexto debe capturar la excepción y devolver NULL, de modo que sin contexto la consulta devuelva vacío, nunca todo. Tratá las tablas de auditoría como inmutables (solo políticas de INSERT/SELECT, sin UPDATE/DELETE).

**Señal de que lo estás usando bien:** Una query lanzada sin setear app.organization_id devuelve 0 filas en lugar de filtrarse datos de otros tenants; no existen políticas de UPDATE/DELETE sobre audit_logs.

**Aplica a:** [ ] Universal  [ ] Solo TypeScript  [x] Solo este proyecto

**Depende de:** #18 — La plantilla de tabla multi-tenant (el molde que se copia y pega)

---

#### TORNILLO #22 — Conexiones como singleton con shutdown ordenado, health check y escape hatch a pg crudo

**En una línea:** Una sola instancia de cliente DB por proceso (lazy), con cierre ordenado, chequeo de salud y un Pool de pg crudo de respaldo cuando Prisma falla.

**Lo que hace el Senior:** Centraliza el acceso a DB en db/prisma.ts con dos singletons lazy: getPrisma() (crea PrismaClient una sola vez, reutiliza la misma instancia) y getPgPool() (un Pool de pg crudo). Mantiene el Pool crudo a propósito porque Prisma tiene un bug leyendo env vars en Windows, y para ese caso provee seed-pg.ts que siembra con SQL en vez de Prisma. Expone checkDatabaseHealth() que hace SELECT 1 (ping barato), y disconnectDatabase() que cierra ambas conexiones en paralelo (Promise.all) para un graceful shutdown. El arranque va por bootstrap.ts, que garantiza que DATABASE_URL esté en process.env ANTES de importar el cliente (import dinámico de ./index.js al final), evitando el bug de orden de carga.

**Ejemplo real del repo:**

`packages/api/src/db/prisma.ts:5-15`

```ts
// --- Raw pg Pool (used instead of Prisma due to Windows env bug) ---
let pgPool: Pool | null = null;

export function getPgPool(): Pool {
  if (!pgPool) {
    pgPool = new Pool({
      connectionString: process.env.DATABASE_URL || 'postgresql://movete:movete123@localhost:5434/movete',
    });
  }
  return pgPool;
}
```

`packages/api/src/db/prisma.ts:85-98`

```ts
export async function disconnectDatabase(): Promise<void> {
  await Promise.all([
    disconnectPrisma(),
    disconnectPgPool(),
  ]);
}

/**
 * Health check for database connection
 */
export async function checkDatabaseHealth(): Promise<boolean> {
  try {
    const client = getPrisma();
    await client.$queryRaw`SELECT 1`;
```

`packages/api/src/seeds/seed-pg.ts:8-15`

```ts
// Use pg pool directly to avoid Prisma Windows environment variable bug
const pool = new Pool({
  host: 'localhost',
  port: 5434,
  database: 'movete',
  user: 'movete',
  password: 'movete123',
});
```

`packages/api/src/bootstrap.ts:1-20`

```ts
import 'dotenv/config';
// Force read DATABASE_URL into process.env
if (!process.env.DATABASE_URL) {
  process.env.DATABASE_URL = 'postgresql://movete:movete_dev_password@localhost:5432/movete';
}
...
// NOW import and run the app
import('./index.js').catch(err => {
```

**La regla para el próximo proyecto:** Encapsulá el acceso a DB en un módulo único con un cliente singleton lazy (no instancies el cliente en cada archivo). Exponé siempre tres funciones: getClient(), checkHealth() (un SELECT 1 para el endpoint /health y readiness probes) y disconnect() que cierre todas las conexiones para graceful shutdown. Si una herramienta (ORM) tiene un bug conocido en cierto entorno, dejá documentado en comentario el por qué del workaround y mantené un camino alternativo con el driver crudo. Cargá las variables de entorno críticas ANTES de importar el cliente (usá un bootstrap con import dinámico).

**Señal de que lo estás usando bien:** No hay más de una instancia del cliente DB viva por proceso; existe un checkDatabaseHealth() que hace SELECT 1; al cerrar el proceso se liberan todas las conexiones; cada workaround tiene un comentario que explica la razón.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #18 — La plantilla de tabla multi-tenant (el molde que se copia y pega)

---

### ÁREA 5 — Cuando algo se rompe (errores, logging, observabilidad)

#### TORNILLO #23 — Errores de negocio se atajan, errores del sistema explotan

**En una línea:** Distinguir el error esperado del usuario (HTTP semántico legible) del error inesperado del sistema (throw → 500 logueado por el handler global).

**Lo que hace el Senior:** En cada route, los errores de dominio conocidos (credenciales inválidas, no encontrado, conflicto) se traducen a un status HTTP explícito con un mensaje para el usuario. Pero cuando algo inesperado revienta dentro de una transacción de DB, hace ROLLBACK y `throw err` para que el error suba al error handler por defecto de Fastify, que es donde pino loguea el stack real y se devuelve un 500. Es decir: el catch local nunca traga un error que no entiende; solo mapea los que sí conoce y re-lanza el resto. En auth.ts el `fastify.log.error(error)` aparece recién en el último escalón del catch, justo antes del 500, nunca para los 401/403/409/400 de validación, así pino no se llena de ruido por errores que son culpa del usuario.

**Ejemplo real del repo:**

`packages/api/src/routes/payments.ts:821-826`

```ts
} catch (err) {
      await client.query('ROLLBACK');
      throw err;
    } finally {
      client.release();
    }
```

`packages/api/src/routes/auth.ts:611-646`

```ts
if (message === 'INVALID_CREDENTIALS') {
          return reply.status(401).send({ error: 'Invalid email or password' });
        }
        ...
        fastify.log.error(error);
        return reply.status(500).send({ error: 'Internal server error' });
```

`packages/api/src/routes/payments.ts:962-967`

```ts
} catch (err) {
      await client.query('ROLLBACK');
      throw err;
    } finally {
      client.release();
    }
```

**La regla para el próximo proyecto:** En cada handler, hacé try/catch SOLO para mapear errores de dominio que conocés a su HTTP correspondiente (401/403/404/409/422). Para todo lo demás, re-lanzá el error (después de limpiar recursos como ROLLBACK/release) y dejá que el error handler global del framework lo loguee con stack y devuelva un 500 genérico. No loguees el stack en errores que son culpa del usuario.

**Señal de que lo estás usando bien:** Tus logs de nivel error solo contienen fallas reales del sistema (DB caída, bug), no validaciones fallidas ni 'password incorrecto'. Buscás 'throw err' en tus catches de transacciones y aparece; buscás 'reply.status(500)' y casi solo aparece en el catch raíz, no por route.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** “Logger pino centralizado con handler global”; “Transacciones con ROLLBACK en el catch”

---

#### TORNILLO #24 — Envelope de error { message } en español, uniforme (con una deuda viva en auth)

**En una línea:** Toda respuesta de error que ve el usuario es `reply.code(XXX).send({ message: 'texto en español' })`, repetido idénticamente en casi todas las rutas.

**Lo que hace el Senior:** El contrato de error de cara al cliente es siempre `{ message }` con texto en español listo para mostrar: 182 ocurrencias de `.send({ message:` repartidas en 15 archivos de rutas. El cliente web sabe leer exactamente ese campo (con fallback a `error` por compatibilidad). La excepción es auth.ts, el archivo más viejo, que quedó usando `{ error: '...' }` en inglés: las 22 ocurrencias de `.send({ error:` están TODAS concentradas ahí. Es la deuda técnica que delata cuál fue el primer archivo escrito antes de fijar la convención (de hecho, dentro de auth.ts conviven ambos shapes: ver el 409 que manda `error` y `message` juntos). Y el tipo `ApiResponse<T>` con `error: { code, message }` está DEFINIDO en shared/types.ts pero NO se usa en runtime en ninguna route: el envelope teórico y el real divergieron.

**Ejemplo real del repo:**

`packages/api/src/routes/classes.ts:163-163`

```ts
if (result.rows.length === 0) return reply.code(404).send({ message: 'Clase no encontrada' });
```

`packages/api/src/routes/auth.ts:611-612`

```ts
if (message === 'INVALID_CREDENTIALS') {
          return reply.status(401).send({ error: 'Invalid email or password' });
```

`packages/shared/src/types.ts:90-102`

```ts
export interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: Record<string, unknown>;
  };
```

`apps/web-admin/lib/api.ts:136-143`

```ts
const payload = data as { message?: unknown; error?: unknown };
    const rawMessage = payload.message ?? payload.error;
    ...
    throw new ApiError(message, res.status, data);
```

**La regla para el próximo proyecto:** Elegí UN solo shape de error de cara al cliente (ej. `{ message }`) y un solo idioma, y respetalo en el 100% de las rutas desde el primer día. Si definís un tipo de envelope en el paquete compartido, USALO en runtime o no lo definas: un tipo que nadie aplica es una mentira que confunde. El cliente debe parsear ese único campo, con a lo sumo un fallback de compatibilidad temporal.

**Señal de que lo estás usando bien:** Un grep de `.send({ error:` da resultados concentrados en un solo archivo legacy (o cero) y `.send({ message:` aparece consistente en todos los demás. El parser de errores del cliente lee un solo campo con un único fallback (`payload.message ?? payload.error`).

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #30 — Validación con Zod safeParse y 400 estructurado antes del SQL; “Parser de error en el cliente HTTP”

---

#### TORNILLO #25 — Validar en la frontera con Zod safeParse y traducir SIEMPRE igual

**En una línea:** Cada endpoint que recibe body hace `Schema.safeParse(request.body)` y, si falla, devuelve 400 con `{ message: 'Datos inválidos', errors: parsed.error.flatten().fieldErrors }`.

**Lo que hace el Senior:** Antes de tocar la DB, valida el input con un schema de Zod usando `safeParse` (no `parse`, así no tira excepción: el error de validación es un caso de negocio, no del sistema). Cuando `!parsed.success`, devuelve 400 con un mensaje fijo en español más el detalle por campo via `flatten().fieldErrors`, que el front puede mapear a cada input. El literal 'Datos inválidos' aparece 37 veces en 14 archivos: es copy-paste deliberado de la misma plantilla. Los schemas viven en @movete/shared (CreatePaymentSchema, CreateClassAdminSchema, JoinOrganizationWithCredentialsSchema), compartidos entre back y front.

**Ejemplo real del repo:**

`packages/api/src/routes/payments.ts:974-977`

```ts
const parsed = CreatePaymentSchema.safeParse(request.body);
    if (!parsed.success) {
      return reply.code(400).send({ message: 'Datos inválidos', errors: parsed.error.flatten().fieldErrors });
    }
```

`packages/api/src/routes/classes.ts:346-349`

```ts
const parsed = CreateClassAdminSchema.safeParse(request.body);
    if (!parsed.success) {
      return reply.code(400).send({ message: 'Datos inválidos', errors: parsed.error.flatten().fieldErrors });
    }
```

`packages/api/src/routes/auth.ts:490-493`

```ts
const parsed = JoinOrganizationWithCredentialsSchema.safeParse(request.body);
      if (!parsed.success) {
        return reply.status(400).send({ message: 'Datos inválidos', errors: parsed.error.flatten().fieldErrors });
      }
```

**La regla para el próximo proyecto:** Validá todo input en la frontera del handler con un schema (Zod o equivalente) usando la variante que NO lanza excepción (safeParse). En el fallo devolvé un 400 con un mensaje genérico fijo y el desglose por campo (fieldErrors) para que el cliente lo pinte en cada input. Definí los schemas en el paquete compartido para reusarlos en el front.

**Señal de que lo estás usando bien:** Ningún handler accede a `request.body` sin antes pasarlo por un schema. Un input inválido nunca llega a la DB ni genera un 500: siempre devuelve 400 con el mismo shape (`{ message: 'Datos inválidos', errors: ... }`) y el front puede resaltar el campo culpable.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #24 — Envelope de error { message } en español, uniforme (con una deuda viva en auth); “Schemas compartidos en packages/shared”

---

#### TORNILLO #26 — Webhook de pago a prueba de duplicados (idempotencia + firma + verdad del proveedor)

**En una línea:** El webhook de Mercado Pago descarta reentregas con una tabla de eventos UNIQUE + ON CONFLICT DO NOTHING, valida firma HMAC timing-safe y nunca confía en el payload: re-consulta el pago al proveedor.

**Lo que hace el Senior:** El handler del webhook arma una `event_key` determinística, inserta en `payment_webhook_events` con un UNIQUE (provider, event_key) y `ON CONFLICT (provider, event_key) DO NOTHING RETURNING id`: si no devuelve filas, ya se procesó y responde `{ received: true, duplicate: true }` sin re-ejecutar nada (idempotencia ante reintentos de MP). Valida la firma con HMAC SHA256 usando `crypto.timingSafeEqual` sobre buffers hex (resistente a timing attacks y a inputs malformados, que devuelven false en el catch), pero si la firma falla solo loguea un warn con `request.log.warn` y cae al plan B (validación por lookup) en vez de cortar. Nunca confía en el status que llega en el body: re-consulta el pago real con `fetchMercadoPagoPayment` antes de activar. Toda la activación va dentro de una transacción y, si revienta, ROLLBACK + throw (escala al 500 global).

**Ejemplo real del repo:**

`packages/api/src/routes/payments.ts:884-894`

```ts
const eventInsert = await pool.query(
      `INSERT INTO payment_webhook_events (provider, event_key, provider_payment_id, payload)
       VALUES ('MERCADO_PAGO', $1, $2, $3::JSONB)
       ON CONFLICT (provider, event_key) DO NOTHING
       RETURNING id`,
      ...
    if (!eventInsert.rows.length) return { received: true, duplicate: true };
```

`packages/api/src/routes/payments.ts:214-222`

```ts
function timingSafeEqualHex(a: string, b: string) {
    try {
      const left = Buffer.from(a, 'hex');
      const right = Buffer.from(b, 'hex');
      return left.length === right.length && crypto.timingSafeEqual(left, right);
    } catch {
      return false;
    }
  }
```

`packages/api/src/routes/payments.ts:876-881`

```ts
if (!signatureValid && settings.webhookSecret) {
      request.log.warn(
        { organizationId, dataId, eventType },
        'Mercado Pago webhook signature invalid; falling back to payment lookup validation',
      );
    }
```

`packages/api/src/routes/payments.ts:59-71`

```ts
CREATE TABLE IF NOT EXISTS payment_webhook_events (
        ...
        event_key TEXT NOT NULL,
        ...
        UNIQUE (provider, event_key)
      )
```

**La regla para el próximo proyecto:** Todo webhook de pago debe ser idempotente: persistí cada evento con una clave única y un INSERT ... ON CONFLICT DO NOTHING; si no insertaste fila, ya lo procesaste, respondé 200 y no hagas nada. Validá la firma con comparación timing-safe (y que un input malformado devuelva false, no que explote). Nunca confíes en el status del body: re-consultá el estado real al proveedor antes de mover plata o activar accesos. Envolvé la activación en una transacción.

**Señal de que lo estás usando bien:** Reenviar el mismo webhook dos veces no crea dos membresías ni cobra dos veces: la segunda vez devuelve duplicate:true. Un payload falsificado con status 'approved' no activa nada porque vos consultás al proveedor real con fetchMercadoPagoPayment.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #23 — Errores de negocio se atajan, errores del sistema explotan; “Logger pino centralizado”

---

#### TORNILLO #27 — Refresh silencioso de token en 401 con deduplicación (mismo patrón en mobile y web)

**En una línea:** El cliente HTTP intercepta el 401, refresca el access token UNA sola vez aunque haya N requests en paralelo, reintenta el original y, si el refresh falla, limpia sesión y manda al login.

**Lo que hace el Senior:** Tanto el interceptor de axios en mobile como el wrapper de fetch en web-admin implementan el mismo plan: ante un 401, si hay refresh token, disparan `/auth/refresh`, pero guardan la promesa en una variable módulo-level (`refreshing` en mobile / `refreshInFlight` en web) para que múltiples requests que reciben 401 al mismo tiempo compartan UNA sola llamada de refresh en vez de spamear N; ambas la limpian en `.finally(() => ... = null)`. Usan un flag (`original._retry` en mobile, `_isRetry` en web) para no entrar en loop infinito. Si el refresh falla, hacen `clearAuth()` y, en web, redirigen a /login con `throw new Error('Sesión expirada. Por favor, ingresá nuevamente.')`. El error de red/timeout (vía AbortController → AbortError) se traduce a un mensaje humano en español, no se muestra el error crudo.

**Ejemplo real del repo:**

`apps/mobile/src/lib/api.ts:44-57`

```ts
// Deduplicate concurrent refresh calls
    if (!refreshing) {
      refreshing = axios
        .post(`${API_URL}/auth/refresh`, { refreshToken })
        .then((r) => {
          setAuth(r.data);
          return r.data.accessToken as string;
        })
        .catch((err) => {
          clearAuth();
          return Promise.reject(err);
        })
        .finally(() => { refreshing = null; });
    }
```

`apps/web-admin/lib/api.ts:100-130`

```ts
if (res.status === 401 && !_isRetry) {
    ...
        if (!refreshInFlight) {
          refreshInFlight = request<AuthResponse>('/auth/refresh', ...).finally(() => { refreshInFlight = null; });
        }
        const refreshed = await refreshInFlight;
    ...
      } catch {
        clearAuth();
        if (typeof window !== 'undefined') window.location.replace('/login');
        throw new Error('Sesión expirada. Por favor, ingresá nuevamente.');
```

`apps/web-admin/lib/api.ts:90-94`

```ts
} catch (err) {
    if (err instanceof DOMException && err.name === 'AbortError') {
      throw new Error('La API tardó demasiado en responder. Verificá que el servidor esté activo.');
    }
    throw new Error(`No se pudo conectar con la API en ${API_URL}. Verificá que el backend esté levantado.`);
```

`apps/mobile/src/lib/feedback.ts:22-24`

```ts
export function showError(message: string) {
  showMessage('Error', message);
}
```

**La regla para el próximo proyecto:** Centralizá el manejo de auth en el cliente en UN interceptor: ante 401, refrescá el token reutilizando una única promesa compartida módulo-level (deduplicación) que limpiás en finally, marcá el request reintentado con un flag para evitar loops, y si el refresh falla limpiá sesión y redirigí al login con un mensaje legible. Traducí errores de red/timeout (AbortError) a texto humano, nunca muestres el error crudo al usuario.

**Señal de que lo estás usando bien:** Si 10 requests reciben 401 a la vez, en la red ves UNA sola llamada a /auth/refresh, no 10. El usuario nunca ve un toast con 'AxiosError: 401'; ve 'Sesión expirada' o un mensaje de negocio. No hay loops de refresh infinitos.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #24 — Envelope de error { message } en español, uniforme (con una deuda viva en auth); #7 — Access token en memoria, refresh token persistido y rotado

---

### ÁREA 6 — La letra del Senior (estilo personal y anti-patrones)

#### TORNILLO #28 — Factory de rutas: setupXRoutes(app)

**En una línea:** Cada módulo de rutas es una función que recibe la app como dependencia y registra sus endpoints, en vez de tocar un singleton global.

**Lo que hace el Senior:** Define `export async function setupXRoutes(app: FastifyInstance)` en TODOS los archivos de routes. La función es async porque adentro hace await de la auto-migración y registra los handlers contra el `app` que le inyectan. La app principal después llama a cada setup pasándole la instancia. Cuando un módulo necesita más colaboradores (como auth) los suma a la firma: `setupAuthRoutes(fastify, authService)`. Es inyección de dependencias por parámetro, no imports de un objeto compartido. Confirmado en 19 archivos de routes: 18 reciben `app: FastifyInstance` y solo auth.ts recibe `fastify, authService`.

**Ejemplo real del repo:**

`packages/api/src/routes/activities.ts:19-23`

```ts
export async function setupActivitiesRoutes(app: FastifyInstance) {
  await ensureActivitiesTables();

  const readAuth = { onRequest: [app.authenticate, requirePermission('classes.read')] } as any;
```

`packages/api/src/routes/facilities.ts:16-20`

```ts
export async function setupFacilitiesRoutes(app: FastifyInstance) {
  const readAuth = { onRequest: [app.authenticate, requirePermission('facilities.read')] } as any;
  const manageAuth = { onRequest: [app.authenticate, requirePermission('facilities.manage')] } as any;
  const pool = getPgPool();
  await ensureFacilityColumns(pool);
```

`packages/api/src/routes/auth.ts:265`

```ts
export async function setupAuthRoutes(fastify: FastifyInstance, authService: AuthService) {
```

`packages/api/src/routes/wods.ts:66-67`

```ts
export async function setupWodsRoutes(app: FastifyInstance) {
  await ensureWodsTable();
```

**La regla para el próximo proyecto:** No registres rutas tocando un objeto global importado. Exportá una función `setupXRoutes(app, ...deps)` por dominio que reciba la instancia del servidor (y cualquier servicio que necesite) como parámetro, y registrá los handlers ahí adentro. El bootstrap central llama a cada setup. Si un módulo necesita un servicio extra, sumalo a la firma en vez de importarlo desde un singleton.

**Señal de que lo estás usando bien:** Podés instanciar el módulo de rutas en un test pasándole un `app` falso sin levantar todo el servidor; ningún archivo de routes importa una instancia global del framework.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #32 — Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission; #31 — Auto-migración ensureXTable/ensureXColumn en el arranque

---

#### TORNILLO #29 — Fail-closed multi-tenant: nunca una query sin org_id + soft delete

**En una línea:** Toda consulta a la DB filtra por organization_id del usuario y por deleted_at IS NULL, sin excepción.

**Lo que hace el Senior:** Saca el tenant del token (`request.user.org_id`) y lo mete como parámetro de CADA query (SELECT, UPDATE, DELETE). Nunca confía en un orgId que venga del body o del path. Suma siempre `deleted_at IS NULL` para hacer borrado lógico (soft delete) en vez de DELETE físico, y los DELETE en realidad son UPDATE que setean `deleted_at = NOW()`. El UPDATE/DELETE lleva el `organization_id` en el WHERE además del id, así un id válido de otra organización devuelve 404 en vez de filtrar datos. Es fail-closed: si te olvidás el filtro, no devolvés de más, directamente la query no matchea. (Nota: audit-logs filtra por org pero es read-only y no usa deleted_at, ya que es la tabla de logs.)

**Ejemplo real del repo:**

`packages/api/src/routes/activities.ts:29-35`

```ts
let where = `organization_id = $1::UUID AND deleted_at IS NULL`;
    if (includeInactive !== 'true') where += ` AND is_active = true`;

    const result = await getPgPool().query(
      `SELECT * FROM activities WHERE ${where} ORDER BY name`,
      [user.org_id],
```

`packages/api/src/routes/wods.ts:219-222`

```ts
`UPDATE wods SET deleted_at = NOW(), updated_at = NOW()
       WHERE id = $1::UUID AND organization_id = $2::UUID AND deleted_at IS NULL
       RETURNING id`,
      [id, user.org_id],
```

`packages/api/src/routes/facilities.ts:176-181`

```ts
const result = await pool.query(
      `UPDATE facilities SET deleted_at = NOW(), updated_at = NOW()
       WHERE id = $1 AND organization_id = $2 AND deleted_at IS NULL RETURNING id`,
      [id, user.org_id],
    );
    if (!result.rows.length) return reply.code(404).send({ message: 'Sede no encontrada' });
```

`packages/api/src/routes/audit.ts:20-21`

```ts
let where = `WHERE al.organization_id = $1`;
    const params: any[] = [user.org_id];
```

**La regla para el próximo proyecto:** En una app multi-tenant: derivá el tenant SIEMPRE del token autenticado, nunca del input del cliente, y metelo en el WHERE de toda query incluyendo UPDATE/DELETE. Usá soft delete (columna deleted_at) y filtrá `deleted_at IS NULL` en cada lectura de entidades de negocio. Que un recurso de otro tenant devuelva 404, no 403 ni los datos. Si vas a confiar en RLS de la base, igual mantené el filtro explícito como segunda barrera.

**Señal de que lo estás usando bien:** Buscás `organization_id` con grep y aparece en absolutamente todas las queries del proyecto; pedir un id que existe pero de otra org da 404. No hay un solo SELECT/UPDATE/DELETE de entidades sin el filtro de tenant.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #28 — Factory de rutas: setupXRoutes(app); #32 — Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission

---

#### TORNILLO #30 — Validación con Zod safeParse y 400 estructurado antes del SQL

**En una línea:** Toda mutación parsea el body con un schema Zod usando safeParse y corta con un 400 de errores por campo si falla, antes de tocar la base.

**Lo que hace el Senior:** Al arranque de cada POST/PATCH/PUT corre `Schema.safeParse(request.body)`; si `!parsed.success` devuelve `reply.code(400)` con `{ message: 'Datos inválidos', errors: parsed.error.flatten().fieldErrors }`. Recién después usa `parsed.data` (a veces renombrado a `b`) para armar el INSERT/UPDATE. Los schemas viven en el paquete shared (`@movete/shared`) para compartir tipos con el front (activities, facilities), o se definen inline con `z.object({...})` y `.partial()` para el update (wods). En los UPDATE usa `COALESCE($n, columna)` para que un campo omitido no pise el valor existente: parcialidad real sin lógica de armado dinámico de SQL.

**Ejemplo real del repo:**

`packages/api/src/routes/activities.ts:41-44`

```ts
const parsed = CreateActivitySchema.safeParse(request.body);
    if (!parsed.success) {
      return reply.code(400).send({ message: 'Datos inválidos', errors: parsed.error.flatten().fieldErrors });
    }
```

`packages/api/src/routes/wods.ts:6-17`

```ts
const WodInputSchema = z.object({
  date: z.string().regex(/^\d{4}-\d{2}-\d{2}$/),
  title: z.string().min(1).max(255),
  description: z.string().min(1),
  activityId: z.string().uuid().nullable().optional(),
  facilityId: z.string().uuid().nullable().optional(),
  coachNotes: z.string().nullable().optional(),
  level: z.string().max(50).nullable().optional(),
  isPublished: z.boolean().optional(),
});

const WodUpdateSchema = WodInputSchema.partial();
```

`packages/api/src/routes/wods.ts:185-189`

```ts
`UPDATE wods SET
         activity_id = COALESCE($3::UUID, activity_id),
         facility_id = COALESCE($4::UUID, facility_id),
         wod_date = COALESCE($5::DATE, wod_date),
         title = COALESCE($6, title),
```

`packages/api/src/routes/facilities.ts:123-127`

```ts
const parsed = UpdateFacilitySchema.safeParse(request.body);
    if (!parsed.success) {
      return reply.code(400).send({ message: 'Datos inválidos', errors: parsed.error.flatten().fieldErrors });
    }
    const b = parsed.data;
```

**La regla para el próximo proyecto:** Validá TODO input de mutación con un schema (Zod) usando safeParse, no parse, para controlar vos el error: ante fallo devolvé 400 con un payload estructurado `{ message, errors: fieldErrors }` y nunca sigas al SQL. Definí los schemas en un paquete compartido para reusar los tipos en el front. Para updates parciales usá `.partial()` + `COALESCE($n, columna)` en el SQL en vez de construir queries dinámicas frágiles.

**Señal de que lo estás usando bien:** Ningún handler de POST/PATCH/PUT accede a `request.body` crudo: siempre pasa por `Schema.safeParse` primero, y mandar un body inválido devuelve 400 con errores por campo, no un 500.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #29 — Fail-closed multi-tenant: nunca una query sin org_id + soft delete

---

#### TORNILLO #31 — Auto-migración ensureXTable/ensureXColumn en el arranque

**En una línea:** Cada módulo se asegura su propio esquema con DDL idempotente (CREATE TABLE/INDEX IF NOT EXISTS, ALTER ADD COLUMN IF NOT EXISTS) al registrarse.

**Lo que hace el Senior:** Antes de exponer endpoints, el setup llama a una función `ensureXTables()` o `ensureXColumn()` que corre DDL idempotente: `CREATE TABLE IF NOT EXISTS`, `CREATE INDEX IF NOT EXISTS`, `ALTER TABLE ... ADD COLUMN IF NOT EXISTS`. Incluso hace backfill de datos viejos dentro de la misma función (`backfillClassActivities` al final de ensureActivitiesTables). El módulo es dueño de su esquema y se auto-repara en cada boot sin depender de un paso de migración manual; las funciones tienen prefijo `ensure` y son seguras de correr N veces. Algunas viven en services/ (activities, business-hours) y otras inline en el archivo de routes (facilities, wods).

**Ejemplo real del repo:**

`packages/api/src/services/activities.ts:5-22`

```ts
export async function ensureActivitiesTables() {
  const pool = getPgPool();
  await pool.query(`
    CREATE TABLE IF NOT EXISTS activities (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      organization_id UUID NOT NULL,
      name VARCHAR(255) NOT NULL,
      description TEXT,
      individual_price DECIMAL(10,2),
      is_active BOOLEAN NOT NULL DEFAULT true,
      created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
      updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
      deleted_at TIMESTAMPTZ,
      UNIQUE (organization_id, name)
    )
  `);
  await pool.query(`CREATE INDEX IF NOT EXISTS idx_activities_org ON activities(organization_id)`);
```

`packages/api/src/services/business-hours.ts:26-31`

```ts
export async function ensureFacilityBusinessHoursColumn() {
  await getPgPool().query(`
    ALTER TABLE facilities
      ADD COLUMN IF NOT EXISTS business_hours JSONB
  `);
}
```

`packages/api/src/routes/facilities.ts:8-14`

```ts
async function ensureFacilityColumns(pool: ReturnType<typeof getPgPool>) {
  await pool.query(`
    ALTER TABLE facilities
    ADD COLUMN IF NOT EXISTS is_active BOOLEAN NOT NULL DEFAULT TRUE
  `);
  await ensureFacilityBusinessHoursColumn();
}
```

`packages/api/src/routes/wods.ts:19-22`

```ts
async function ensureWodsTable() {
  const pool = getPgPool();
  await pool.query(`
    CREATE TABLE IF NOT EXISTS wods (
```

**La regla para el próximo proyecto:** Si vas a auto-gestionar esquema desde el código (sin migraciones formales), encapsulá el DDL de cada módulo en una función `ensureXTable/ensureXColumn` 100% idempotente (IF NOT EXISTS en tablas, índices y columnas) y llamala al inicializar el módulo. Nombrá con prefijo `ensure` para señalar 'seguro de correr siempre'. Ojo: esto sirve para prototipos y features incrementales, pero para producción seria conviene migrar a migraciones versionadas; aun así la regla del nombre + idempotencia es oro.

**Señal de que lo estás usando bien:** Podés borrar la tabla y levantar el server y se recrea sola; correr el arranque dos veces no tira error de 'ya existe'. Cada función que toca DDL empieza con `ensure`.

**Aplica a:** [ ] Universal  [ ] Solo TypeScript  [x] Solo este proyecto

**Depende de:** #28 — Factory de rutas: setupXRoutes(app)

---

#### TORNILLO #32 — Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission

**En una línea:** El permiso se chequea como hook de Fastify en la definición de la ruta, y el mismo modelo de permisos filtra qué ve el usuario en el front.

**Lo que hace el Senior:** En el back arma objetos de guard reutilizables (`readAuth`, `manageAuth`) que combinan `app.authenticate` con `requirePermission('recurso.accion')` y los pasa como `onRequest` a cada endpoint. La autorización es declarativa y queda al lado de la ruta, no escondida dentro del handler. Usa una convención de strings `recurso.accion` (`classes.read`, `facilities.manage`, `audit.read`). El mismo concepto se refleja en el front: el Sidebar filtra los items de navegación con `hasPermission(user, required)` contra un mapa `ROUTE_PERMISSIONS` (ambos importados de `@/lib/permissions`), así el usuario ni ve lo que no puede tocar. Es defense-in-depth: el front oculta, el back impone.

**Ejemplo real del repo:**

`packages/api/src/routes/facilities.ts:17-18`

```ts
const readAuth = { onRequest: [app.authenticate, requirePermission('facilities.read')] } as any;
  const manageAuth = { onRequest: [app.authenticate, requirePermission('facilities.manage')] } as any;
```

`packages/api/src/routes/audit.ts:6-10`

```ts
const auth = { onRequest: [app.authenticate, requirePermission('audit.read')] } as any;
  const pool = getPgPool();

  // GET /api/audit-logs
  app.get('/api/audit-logs', auth, async (request) => {
```

`apps/web-admin/components/layout/Sidebar.tsx:35-38`

```ts
const visibleNavItems = navItems.filter((item) => {
    const required = ROUTE_PERMISSIONS[item.href];
    return !required || hasPermission(user, required);
  });
```

**La regla para el próximo proyecto:** Modelá permisos como strings `recurso.accion` y chequealos de forma DECLARATIVA en la capa de ruteo (hook/middleware) atado a la definición del endpoint, no con if's dentro del handler. Armá objetos de guard reutilizables (readAuth/manageAuth) y reusalos. En el front, derivá el menú y los botones del MISMO modelo de permisos (un mapa ruta->permiso + un helper hasPermission) para ocultar lo no autorizado, pero tratá esto solo como UX: la verdad la impone el back.

**Señal de que lo estás usando bien:** Cada endpoint declara su permiso al lado de la URL y se lee de un vistazo; el front nunca muestra una sección que el back rechazaría. Los permisos siguen la convención `recurso.accion` en ambos lados.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #28 — Factory de rutas: setupXRoutes(app); #29 — Fail-closed multi-tenant: nunca una query sin org_id + soft delete

---

### ÁREA 7 — Las llaves del local (variables de entorno, config por ambiente, secrets)

#### TORNILLO #33 — El .env.example como contrato narrado, no como lista muerta

**En una línea:** Documentar cada variable con comentarios que explican qué hace, en qué ambiente aplica y cuál es el camino recomendado vs el fallback legacy.

**Lo que hace el Senior:** Mantiene un .env.example agrupado por dominio (DATABASE, JWT & AUTH, MERCADO PAGO, FEATURE FLAGS, etc.) donde cada bloque tiene comentarios que no solo nombran la variable sino que explican la decisión arquitectónica detrás: por qué WhatsApp está deshabilitado (es caro para notificaciones), que las credenciales de MP se configuran por gimnasio desde el Web Admin y que PAYMENT_SETTINGS_SECRET cifra esas credenciales mientras que las MERCADO_PAGO_* del .env son solo fallback legacy/local. El README de Cloud Run complementa esto con la lista de 'Variables minimas' por servicio (API y Web por separado) y un mapa de 'Flujos que dependen de variables', así un dev nuevo sabe qué se rompe si falta una. Los placeholders son valores de dev seguros y obvios (JWT_SECRET="dev_secret_change_in_production_...", minioadmin, change_me_to_a_long_stable_secret) que gritan 'cambiame en prod'.

**Ejemplo real del repo:**

`.env.example:29-39`

```bash
# ============= MERCADO PAGO =============
# Multiempresa: cada gimnasio configura Access Token/Public Key desde Web Admin.
# Este secreto cifra credenciales guardadas en organization_payment_settings.
PAYMENT_SETTINGS_SECRET="change_me_to_a_long_stable_secret"
# Fallback legacy/local only. No usar como configuración principal en producción multiempresa.
MERCADO_PAGO_ACCESS_TOKEN=""
```

`.env.example:43-46`

```bash
# ============= WHATSAPP =============
# WhatsApp Business API is intentionally disabled by default because it is costly
# for notification use cases. Web Admin uses manual wa.me links from Socios.
WHATSAPP_BUSINESS_ENABLED=false
```

`deploy/cloud-run/README.md:44-51`

```md
## Variables minimas Cloud Run

API:
- `DATABASE_URL`
- `JWT_SECRET`
- `API_CORS_ORIGIN`
- `API_PORT=8080`
- `NODE_ENV=production`
```

**La regla para el próximo proyecto:** Tratá el .env.example como documentación de primera clase: agrupá por dominio con headers (# ===== DOMINIO =====), y en cada variable sensible agregá un comentario de 1-2 líneas que diga (a) qué hace, (b) en qué ambiente se usa, y (c) si es la opción recomendada o un fallback. Usá placeholders de dev que sean obviamente inseguros (ej: 'change_me', 'dev_secret_change_in_production_...') para que nadie los confunda con valores de prod. Mantené en paralelo una lista de 'variables mínimas por servicio' en el doc de deploy.

**Señal de que lo estás usando bien:** Un dev nuevo puede copiar .env.example a .env, levantar todo en local sin pedirle secretos a nadie, y al leer los comentarios entiende qué variables son obligatorias en producción y por qué.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #35 — Fallbacks de env en cascada: DB -> env -> default local; #34 — Secrets multiempresa cifrados con AES-256-GCM, nunca en texto plano

---

#### TORNILLO #34 — Secrets multiempresa cifrados con AES-256-GCM, nunca en texto plano

**En una línea:** Las credenciales sensibles de cada organización se guardan cifradas en la DB con una clave maestra derivada de env, no en columnas de texto plano.

**Lo que hace el Senior:** Para el SaaS multiempresa, cada gimnasio carga sus propias credenciales de Mercado Pago. En vez de guardarlas tal cual, deriva una clave de cifrado por SHA-256 de PAYMENT_SETTINGS_SECRET (con cascada de fallbacks: MERCADO_PAGO_CREDENTIALS_SECRET -> JWT_SECRET -> default local 'movete-local-payment-settings-secret') y cifra cada secreto con AES-256-GCM, guardando IV (12 bytes) + auth tag + ciphertext en un formato versionado 'v1:iv:tag:cipher' (todo en base64). Las columnas en DB se llaman access_token_encrypted / webhook_secret_encrypted, dejando claro que están cifradas. El decryptSecret detecta el prefijo 'v1:' para soportar migración: si el valor no empieza con 'v1:', lo devuelve tal cual (compat con datos legacy). El public_key se guarda en claro a propósito porque no es secreto.

**Ejemplo real del repo:**

`packages/api/src/services/payment-settings.ts:16-31`

```ts
function encryptionKey() {
  const source = process.env.PAYMENT_SETTINGS_SECRET
    ?? process.env.MERCADO_PAGO_CREDENTIALS_SECRET
    ?? process.env.JWT_SECRET
    ?? 'movete-local-payment-settings-secret';
  return crypto.createHash('sha256').update(source).digest();
}

export function encryptSecret(value?: string | null) {
  if (!value) return null;
  const iv = crypto.randomBytes(12);
  const cipher = crypto.createCipheriv('aes-256-gcm', encryptionKey(), iv);
  const encrypted = Buffer.concat([cipher.update(value, 'utf8'), cipher.final()]);
  const tag = cipher.getAuthTag();
  return `v1:${iv.toString('base64')}:${tag.toString('base64')}:${encrypted.toString('base64')}`;
}
```

`packages/api/src/services/payment-settings.ts:33-36`

```ts
export function decryptSecret(value?: string | null) {
  if (!value) return null;
  if (!value.startsWith('v1:')) return value;
  const [, ivRaw, tagRaw, encryptedRaw] = value.split(':');
```

`packages/api/src/services/payment-settings.ts:53-55`

```ts
public_key TEXT,
      access_token_encrypted TEXT,
      webhook_secret_encrypted TEXT,
```

**La regla para el próximo proyecto:** Cuando guardes credenciales de terceros por tenant/organización, nunca las pongas en texto plano en la DB. Derivá una clave con SHA-256 de un secreto de env dedicado (ej: PAYMENT_SETTINGS_SECRET) y cifrá cada valor con AES-256-GCM (que da confidencialidad + integridad via auth tag). Guardá el resultado en formato versionado 'v1:iv:tag:cipher' para poder rotar el esquema después, y que la rutina de descifrado pase derecho los valores sin prefijo para soportar datos legacy. Nombrá las columnas con sufijo _encrypted, y dejá en claro solo lo que no es secreto (ej: public_key).

**Señal de que lo estás usando bien:** Si abrís la tabla en la DB, las columnas de tokens muestran strings 'v1:...' ilegibles en vez de los tokens reales, y rotar el secreto maestro invalida (no expone) los datos viejos.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #35 — Fallbacks de env en cascada: DB -> env -> default local; #33 — El .env.example como contrato narrado, no como lista muerta

---

#### TORNILLO #35 — Fallbacks de env en cascada: DB -> env -> default local

**En una línea:** Cada lectura de config intenta primero la fuente correcta, después alias alternativos, y termina en un default de dev, para que local funcione sin setup pero prod degrade limpio.

**Lo que hace el Senior:** En todo el backend usa el patrón `process.env.X ?? process.env.X_ALIAS ?? 'default-local'` de forma consistente. La config de pagos primero busca el setting por organización en la DB, y solo si no existe la fila cae al fallback de env (marcándolo con lastValidationStatus 'env_fallback' para que sea auditable). Las URLs públicas aceptan varios nombres de variable como alias (API_PUBLIC_URL ?? PUBLIC_API_URL; WEB_ADMIN_PUBLIC_URL ?? WEB_PUBLIC_URL ?? NEXT_PUBLIC_WEB_ADMIN_URL) para tolerar configuraciones distintas entre ambientes. El launcher start.ts setea DATABASE_URL/JWT_SECRET/NODE_ENV con defaults de dev ANTES de importar la app (y por ende antes de que Prisma cargue), garantizando que arranque en local aunque falte el .env. El mismo patrón se repite en platform-billing.ts y auth.ts.

**Ejemplo real del repo:**

`packages/api/src/start.ts:4-6`

```ts
process.env.DATABASE_URL = process.env.DATABASE_URL || 'postgresql://movete:movete_dev_password@localhost:5432/movete';
process.env.JWT_SECRET = process.env.JWT_SECRET || 'dev-secret';
process.env.NODE_ENV = process.env.NODE_ENV || 'development';
```

`packages/api/src/services/payment-settings.ts:98-112`

```ts
if (!row) {
    const fallbackToken = process.env.MERCADO_PAGO_ACCESS_TOKEN ?? null;
    return fallbackToken
      ? {
          organizationId,
          enabled: true,
          publicKey: process.env.MERCADO_PAGO_PUBLIC_KEY ?? null,
          accessToken: fallbackToken,
          webhookSecret: process.env.MERCADO_PAGO_WEBHOOK_SECRET ?? null,
          lastValidationStatus: 'env_fallback',
```

`packages/api/src/services/platform-billing.ts:40-46`

```ts
function publicWebAdminUrl() {
  return (process.env.WEB_ADMIN_PUBLIC_URL ?? process.env.WEB_PUBLIC_URL ?? process.env.NEXT_PUBLIC_WEB_ADMIN_URL ?? 'http://localhost:3000').replace(/\/$/, '');
}

function publicApiUrl() {
  return (process.env.API_PUBLIC_URL ?? process.env.PUBLIC_API_URL ?? 'http://localhost:3001').replace(/\/$/, '');
}
```

`packages/api/src/routes/auth.ts:119-130`

```ts
function publicApiBaseUrl(request: FastifyRequest) {
  const configured = process.env.API_PUBLIC_URL ?? process.env.PUBLIC_API_URL;
  ...
function publicWebAdminBaseUrl(request: FastifyRequest) {
  const configured = process.env.WEB_ADMIN_PUBLIC_URL ?? process.env.WEB_PUBLIC_URL ?? process.env.ADMIN_PUBLIC_URL;
```

**La regla para el próximo proyecto:** Para cada pieza de config implementá una cascada explícita: fuente principal (DB/setting por tenant) -> env var canónica -> alias tolerantes -> default de desarrollo. Usá ?? (nullish coalescing), no ||, cuando un string vacío deba ser un valor válido (ojo: start.ts usa || a propósito para tratar '' como ausente). Cuando caigas al fallback de env, marcá el origen del dato (ej: lastValidationStatus: 'env_fallback') para que sea auditable en prod. Seteá los defaults de dev en un launcher ANTES de importar el ORM, para que la app levante sin ningún .env; en prod, esos defaults deben ser claramente inseguros para forzar a configurarlos.

**Señal de que lo estás usando bien:** Clonás el repo, corrés el comando de arranque sin tocar variables, y la app levanta contra los servicios locales. En prod, las respuestas/datos indican explícitamente cuándo un valor vino de un fallback de env (status 'env_fallback') en vez de la config real por organización.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #33 — El .env.example como contrato narrado, no como lista muerta; #34 — Secrets multiempresa cifrados con AES-256-GCM, nunca en texto plano

---

#### TORNILLO #36 — Config de cliente como build-arg, config de servidor como runtime env

**En una línea:** Las variables NEXT_PUBLIC_/EXPO_PUBLIC_ se inyectan en build (Docker build-args), mientras que los secretos del servidor se inyectan en runtime (env de Cloud Run).

**Lo que hace el Senior:** Distingue claramente dos clases de config. La que se expone al cliente (prefijo NEXT_PUBLIC_ en web, bloque 'extra' en mobile) se hornea en tiempo de build porque Next la inlinea en el bundle: por eso el cloudbuild-web-admin.yaml pasa NEXT_PUBLIC_API_URL y NEXT_PUBLIC_DEFAULT_ORGANIZATION_ID como --build-arg, y el README las documenta también como build-args de docker. Los secretos del servidor (DATABASE_URL, JWT_SECRET, PAYMENT_SETTINGS_SECRET) van como env vars de runtime de Cloud Run, nunca al build. El output 'standalone' de Next se activa solo vía flag NEXT_OUTPUT_STANDALONE=1 para no afectar el dev local. El mobile fija la URL de prod directamente en app.json -> extra.apiUrl, separada de la EXPO_PUBLIC_API_URL de dev del .env.example.

**Ejemplo real del repo:**

`deploy/cloud-run/cloudbuild-web-admin.yaml:12-15`

```yaml
- --build-arg
      - NEXT_PUBLIC_API_URL=${_NEXT_PUBLIC_API_URL}
      - --build-arg
      - NEXT_PUBLIC_DEFAULT_ORGANIZATION_ID=${_NEXT_PUBLIC_DEFAULT_ORGANIZATION_ID}
```

`apps/web-admin/next.config.js:2-7`

```js
const nextConfig = {
  ...(process.env.NEXT_OUTPUT_STANDALONE === '1' ? { output: 'standalone' } : {}),
  experimental: {
    serverComponentsExternalPackages: [],
  },
};
```

`apps/mobile/app.json:37-40`

```json
"extra": {
      "apiUrl": "https://api.movete.szoluciones.com",
      "defaultOrganizationId": "3c0f34b8-98f1-4cc2-9797-85a4b7d5955a"
    },
```

`deploy/cloud-run/README.md:61-65`

```md
Web:
- `NEXT_PUBLIC_API_URL`
- `NEXT_PUBLIC_DEFAULT_ORGANIZATION_ID`
- `PORT=8080`
- `NODE_ENV=production`
```

**La regla para el próximo proyecto:** Separá tajantemente dos clases de config. (1) Config pública del cliente (prefijos NEXT_PUBLIC_/EXPO_PUBLIC_): inyectala en TIEMPO DE BUILD como docker --build-arg, porque el bundler la inlinea y no se puede cambiar en runtime sin rebuild. Asumí que TODO lo prefijado es público y nunca metas un secreto ahí. (2) Secretos del servidor (DB, JWT, claves de cifrado): inyectalos en TIEMPO DE RUNTIME como env del orquestador (Cloud Run), nunca como build-args (quedarían en capas de la imagen). Activá optimizaciones de build pesadas (Next output 'standalone') detrás de un flag (NEXT_OUTPUT_STANDALONE=1) para no romper el dev local.

**Señal de que lo estás usando bien:** Inspeccionás el bundle del cliente y solo encontrás variables con prefijo público (URLs, IDs de organización no sensibles); ningún secreto del servidor aparece en la imagen ni en el JS del cliente. Cambiar un secreto del servidor no requiere rebuildear la imagen, pero cambiar la API URL del cliente sí.

**Aplica a:** [ ] Universal  [ ] Solo TypeScript  [x] Solo este proyecto

**Depende de:** #33 — El .env.example como contrato narrado, no como lista muerta

---

#### TORNILLO #37 — Feature flags con default apagado para lo caro o riesgoso

**En una línea:** Funcionalidad costosa o en desarrollo se gobierna con flags ENABLE_*/FEATURE_* que vienen apagados por defecto y se chequean explícitamente contra el string 'true'.

**Lo que hace el Senior:** Declara una batería de flags en el .env.example (ENABLE_NOTIFICATION_JOBS, FEATURE_WHITE_LABEL, FEATURE_QR_CONTROL, FEATURE_LIVE_STREAMING, FEATURE_AFIP_INTEGRATION) todos en false, además de WHATSAPP_BUSINESS_ENABLED=false con comentario explicando que se apaga por costo. En el código el chequeo es siempre estricto contra el string 'true' (=== 'true', no truthy genérico), evitando que un valor accidental encienda la feature. El flag de notification jobs no solo prende el setInterval (con piso de 15 min y .unref() para no bloquear el cierre) sino que el preflight de deploy advierte si queda prendido con MIN_INSTANCES=0, combinación que no ejecuta intervalos en Cloud Run económico, atando la feature flag a la realidad de la infra.

**Ejemplo real del repo:**

`.env.example:72-78`

```bash
# ============= FEATURE FLAGS =============
ENABLE_NOTIFICATION_JOBS=false
NOTIFICATION_JOBS_INTERVAL_MINUTES=60
FEATURE_WHITE_LABEL=false
FEATURE_QR_CONTROL=false
FEATURE_LIVE_STREAMING=false
FEATURE_AFIP_INTEGRATION=false
```

`packages/api/src/index.ts:102-108`

```ts
if (process.env.ENABLE_NOTIFICATION_JOBS === 'true') {
  const intervalMinutes = Math.max(15, parseInt(process.env.NOTIFICATION_JOBS_INTERVAL_MINUTES ?? '60', 10));
  notificationJobTimer = setInterval(() => {
    runNotificationJobs(null).catch((error) => app.log.error({ error }, 'Notification jobs failed'));
  }, intervalMinutes * 60_000);
  notificationJobTimer.unref();
```

`scripts/deploy/preflight.mjs:122-123`

```
if (process.env.ENABLE_NOTIFICATION_JOBS === 'true' && process.env.MIN_INSTANCES === '0') {
    warnings.push('ENABLE_NOTIFICATION_JOBS=true con MIN_INSTANCES=0 puede no ejecutar intervalos; usar Cloud Scheduler para jobs.');
```

**La regla para el próximo proyecto:** Gobierná toda funcionalidad cara, opcional o en desarrollo con un feature flag de env (ENABLE_* para procesos, FEATURE_* para capacidades de producto). Default SIEMPRE en false/off, así nada riesgoso se enciende por accidente al provisionar un ambiente nuevo. Chequeá el flag de forma estricta (=== 'true'), nunca como truthy. Documentá los flags en su propia sección del .env.example. Si una feature requiere una condición de infra (ej: una instancia siempre activa), validá esa combinación en el script de preflight del deploy.

**Señal de que lo estás usando bien:** Un ambiente recién provisionado sin configurar flags arranca en el modo más barato y seguro (jobs apagados, features beta ocultas), y prender una feature requiere un acto deliberado de setear la variable en 'true'.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #33 — El .env.example como contrato narrado, no como lista muerta; #35 — Fallbacks de env en cascada: DB -> env -> default local

---

### ÁREA 8 — Cuándo algo está realmente terminado (tests y QA)

#### TORNILLO #38 — Test de integración con DB real, sin mocks (inject + seed)

**En una línea:** Probar las rutas levantando el framework HTTP de verdad y pegando contra una base con datos de seed conocidos, en vez de mockear el ORM.

**Lo que hace el Senior:** Cada archivo .test.ts arma una instancia real de Fastify con los mismos plugins de producción (cors, jwt, decorate authenticate) y registra el setup real de las rutas (setupMembersRoutes, setupReservationsRoutes, setupClassesRoutes, etc). En lugar de un servidor de red usa app.inject() (peticiones HTTP en memoria). No mockea Prisma ni la base: vitest.config.ts apunta DATABASE_URL a una Postgres local fija (127.0.0.1:5434) con NODE_ENV=test, y los tests asumen que el seed ya está aplicado (constantes ORG_ID y USER_IDs fijos del seed). Para datos que necesitan existir, los buscan en vivo (getSeedClassId hace un SELECT real vía getPgPool). Como tocan una DB compartida, fuerzan fileParallelism:false para que las suites no compitan, y cierran la conexión en un afterAll global (vitest.setup.ts llama disconnectDatabase).

**Ejemplo real del repo:**

`packages/api/src/routes/members.test.ts:16-31`

```ts
const app = Fastify({ logger: false });

  await app.register(fastifyCors, { origin: '*' });
  await app.register(fastifyJwt, { secret: JWT_SECRET });

  app.decorate('authenticate', async function (request: any, reply: any) {
    ...
  });

  await setupMembersRoutes(app);
```

`packages/api/vitest.config.ts:15-22`

```ts
fileParallelism: false,
    setupFiles: ['./src/test/vitest.setup.ts'],
    root: path.resolve(__dirname),
    env: {
      DATABASE_URL: 'postgresql://movete:movete123@127.0.0.1:5434/movete',
      JWT_SECRET: 'dev-secret',
      NODE_ENV: 'test',
    },
```

`packages/api/src/routes/reservations.test.ts:30-35`

```ts
async function getSeedClassId(): Promise<string> {
  const result = await getPgPool().query(
    `SELECT id FROM classes WHERE organization_id = $1 AND deleted_at IS NULL LIMIT 1`,
    [ORG_ID],
  );
```

`packages/api/src/test/vitest.setup.ts:1-6`

```ts
import { afterAll } from 'vitest';
import { disconnectDatabase } from '../db/prisma';

afterAll(async () => {
  await disconnectDatabase();
});
```

**La regla para el próximo proyecto:** Para endpoints, escribí tests de integración que levanten el framework HTTP real con los mismos plugins/middleware que producción y usen inyección en memoria (app.inject en Fastify, supertest en Express). Pegá contra una base de datos real de test con seed determinista, no contra mocks del ORM. Si las suites comparten esa base, desactivá el paralelismo entre archivos (fileParallelism:false) y cerrá conexiones en un teardown global. Para datos que deben existir, buscalos en vivo con un SELECT en vez de hardcodearlos, y generá datos mutables con un sufijo único (timestamp) para evitar colisiones entre corridas.

**Señal de que lo estás usando bien:** Tus tests fallan cuando rompés una query SQL real, una constraint, una validación de esquema o un cambio de status code, no solo cuando cambia un mock. Podés correr la suite dos veces seguidas sin limpiar la DB y sigue verde.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #39 — Probar el caso negativo y el contrato, no solo el happy path; #41 — QA por smoke tests contra entorno real, gateando el release

---

#### TORNILLO #39 — Probar el caso negativo y el contrato, no solo el happy path

**En una línea:** Para cada endpoint, testear explícitamente los rechazos de validación (400), la falta de auth (401), conflictos (409) y la forma exacta del cuerpo de la respuesta.

**Lo que hace el Senior:** Las suites no se conforman con 'devuelve 200'. Por cada ruta hay un bloque de casos negativos: 400 por campo faltante (amount), monto negativo, método de pago no permitido (BITCOIN), y falta de identificador de socio; y 409 por duplicado. Además verifica el contrato de la respuesta (toHaveProperty sobre 'errors', 'id', 'totals', arrays de memberships/payments). El test de export es el más explícito sobre 'forma correcta': usa it.each para las tres exportaciones (members, payments, reservations), exige content-type text/csv, que aparezca el header de columnas esperado, y que el CSV NO contenga una fuga de error serializado ('"statusCode"'). Cuando un resultado es ambiguo por estado de datos (clase llena), acepta el set válido de outcomes (expect([201,409]).toContain(...)) en vez de asumir uno.

**Ejemplo real del repo:**

`packages/api/src/routes/payments.test.ts:143-178`

```ts
it('rejects missing amount', async () => {
    ...
    expect(res.statusCode).toBe(400);
    expect(res.json()).toHaveProperty('errors');
  });

  it('rejects negative amount', async () => {
    ... body: JSON.stringify({ amount: -100, userId: CUSTOMER_USER_ID }),
    ...
    expect(res.statusCode).toBe(400);
  });

  it('rejects invalid payment method', async () => {
    ... paymentMethod: 'BITCOIN' ...
    expect(res.statusCode).toBe(400);
```

`packages/api/src/routes/export.test.ts:45-60`

```ts
it.each([
    ['/api/export/members', 'Nombre,Apellido,Email'],
    ['/api/export/payments', 'Fecha,Socio,Email'],
    ['/api/export/reservations', 'Fecha,Socio,Email'],
  ])('returns csv for %s', async (url, expectedHeader) => {
    ...
    expect(res.headers['content-type']).toContain('text/csv');
    expect(res.body).toContain(expectedHeader);
    expect(res.body).not.toContain('"statusCode"');
```

`packages/api/src/routes/members.test.ts:171-184`

```ts
it('returns 409 if member already in org', async () => {
    ...
      body: JSON.stringify({
        email: 'admin@fitlife.com',
        ...
      }),
    });

    expect(res.statusCode).toBe(409);
```

`packages/api/src/routes/reservations.test.ts:201-205`

```ts
// 201 created or 409 if class is full — both are valid scenarios
    expect([201, 409]).toContain(res.statusCode);
    if (res.statusCode === 201) {
      expect(res.json()).toHaveProperty('id');
    }
```

**La regla para el próximo proyecto:** Por cada endpoint cubrí, además del happy path: 401 sin credenciales, 400 por cada regla de validación relevante (campo faltante, tipo/formato inválido, valor fuera de enum, número negativo), y 409 en operaciones con unicidad. Afirmá la forma exacta del cuerpo de respuesta (propiedades esperadas, errors) y, en respuestas no-JSON como CSV, verificá tanto el content-type y el header esperado como la AUSENCIA de marcadores de error filtrado (ej. '"statusCode"'). Si los datos hacen ambiguo el resultado, aceptá el conjunto de status válidos con expect([...]).toContain en vez de elegir uno.

**Señal de que lo estás usando bien:** Mandar un payload basura a cualquier endpoint nunca termina en 500 ni en un 200 silencioso: siempre cae en un 400/401/409 cubierto por un test, y la respuesta tiene exactamente las claves esperadas.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #38 — Test de integración con DB real, sin mocks (inject + seed)

---

#### TORNILLO #40 — RLS como red de seguridad final, testeada como atacante

**En una línea:** Verificar el aislamiento multi-tenant a nivel de base de datos (Row-Level Security) probando activamente los accesos cruzados que deben fallar.

**Lo que hace el Senior:** Hay una suite dedicada (rls.test.ts) que crea dos organizaciones y dos usuarios y luego se pone en el rol del atacante: setea el contexto RLS de org1 e intenta INSERTAR (y leer/actualizar/borrar) datos de org2, esperando rejects.toThrow() en escrituras por la policy WITH CHECK. Prueba el comportamiento fail-closed: sin contexto RLS seteado, un COUNT sobre organizations devuelve 0 en vez de filtrar en la app. El archivo documenta la filosofía de defensa en profundidad de 4 capas (JWT, middleware, filtro app, RLS) y deja reglas duras para devs: nunca deshabilitar RLS ni confiar solo en el filtrado de la capa de aplicación, nunca commitear código que bypassee RLS, siempre testear queries con el org_id equivocado. Es el mismo principio que valida el hardening-check en infra: que DATABASE_URL/JWT_SECRET vengan de Secret Manager y que Cloud SQL no tenga authorized networks abiertas.

**Ejemplo real del repo:**

`packages/api/src/db/rls.test.ts:152-164`

```ts
it('should prevent INSERT into other org facilities', async () => {
      await setRLSContext(prisma, user1Id, org1Id);
      ...
      const query = async () => {
        await prisma.$executeRaw`
          INSERT INTO facilities (organization_id, name, address, city, state)
          VALUES (${org2Id}, 'Hacked Facility', 'Evil', 'Evil', 'Evil');
        `;
      };
      // Should fail due to RLS WITH CHECK policy
      expect(query).rejects.toThrow();
```

`packages/api/src/db/rls.test.ts:171-179`

```ts
it('should fail-closed when organization_id is missing', async () => {
      // Don't set RLS context
      const result = await prisma.$queryRaw`
        SELECT COUNT(*) as count FROM organizations;
      `;
      // Should return 0 or throw due to missing context
      expect((result as any)[0].count).toBe(0);
```

`packages/api/src/db/rls.test.ts:280-286`

```ts
SECURITY PRINCIPLE: Defense in Depth
 * - JWT validation (Layer 1)
 * - Middleware checking (Layer 2)
 * - App-layer filtering (Layer 3)
 * - RLS policies (Layer 4) ← We're testing this
 * 
 * Even if layers 1-3 fail, RLS prevents data leaks.
```

`scripts/ops/hardening-check.mjs:77-87`

```
assert(apiEnv.some((env) => env.name === 'DATABASE_URL' && env.valueFrom?.secretKeyRef), 'API DATABASE_URL no viene de Secret Manager');
  assert(apiEnv.some((env) => env.name === 'JWT_SECRET' && env.valueFrom?.secretKeyRef), 'API JWT_SECRET no viene de Secret Manager');
  ...
  assert(authorizedNetworks.length === 0, 'Cloud SQL tiene authorized networks abiertas');
```

**La regla para el próximo proyecto:** En cualquier sistema multi-tenant, no confíes solo en el WHERE org_id de la capa de aplicación: ponelo también a nivel de base (Row-Level Security en Postgres) y escribí una suite dedicada que actúe como atacante: parada en el tenant A, intentá insertar/actualizar/borrar datos del tenant B y exigí excepción (rejects.toThrow). Probá también el caso fail-closed: sin contexto seteado, las queries deben devolver 0 filas, no todo. Complementalo con un check de infra que verifique que los secretos vienen de un Secret Manager y que la base no tiene redes abiertas.

**Señal de que lo estás usando bien:** Si comentás el WHERE org_id en una query de aplicación, la suite RLS sigue en verde porque la base bloquea la fuga igual; y si olvidás setear el contexto, las queries devuelven vacío en vez de exponer datos de otros tenants.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #38 — Test de integración con DB real, sin mocks (inject + seed); #41 — QA por smoke tests contra entorno real, gateando el release

---

#### TORNILLO #41 — QA por smoke tests contra entorno real, gateando el release

**En una línea:** Antes de liberar, correr scripts que pegan contra el entorno real validando matriz de permisos por rol, contratos de endpoints, push real y hardening de infra; cualquier fallo aborta con exit 1.

**Lo que hace el Senior:** Además de los unit/integration tests, hay un set de scripts .mjs ejecutables (scripts/qa, scripts/deploy, scripts/ops) que loguean con usuarios reales y validan comportamiento end-to-end. profiles.mjs define una routeMatrix declarativa: para cada endpoint, el status esperado por rol (ej. payments list: ADMIN 200, INSTRUCTOR 403, RECEPTIONIST 200, CUSTOMER 403) y acumula todos los fallos antes de salir con exit 1. mobile-api.mjs valida el contrato de cada endpoint móvil con una función validate por ruta. push.mjs hace un envío push REAL y exige deliveryStatus === 'PUSH_SENT'. with-release-env.mjs inyecta los dominios finales y orquesta los modos (profiles/web/mobile/web:visual/push); en package.json, qa:release = profiles && web && mobile && web:visual. preflight.mjs chequea variables de entorno con validadores (JWT_SECRET >=32 chars y != 'dev-secret', DATABASE_URL postgresql:// sin localhost), migraciones SQL, type-check de api y web-admin, y /health antes de deployar. hardening-check.mjs verifica infra en GCP. El RELEASE_CHECKLIST.md ata todo: declara `pnpm qa:release` + `ops:hardening` como QA obligatorio antes de release contra los dominios finales.

**Ejemplo real del repo:**

`scripts/qa/profiles.mjs:21-21`

```
{ name: 'payments list', method: 'GET', path: '/api/payments?limit=1', expect: { ADMIN: 200, INSTRUCTOR: 403, RECEPTIONIST: 200, CUSTOMER: 403 } },
```

`scripts/qa/profiles.mjs:117-121`

```
if (failures.length) {
    console.error('\nFallos de QA perfiles:');
    for (const failure of failures) console.error(`- ${failure}`);
    process.exit(1);
  }
```

`scripts/qa/push.mjs:101-105`

```
assert(sent.status === 201, `push send failed (${sent.status}): ${JSON.stringify(sent.body)}`);
  assert(
    sent.body.deliveryStatus === 'PUSH_SENT',
    `push deliveryStatus esperado PUSH_SENT, recibido ${sent.body.deliveryStatus}: ${JSON.stringify(sent.body)}`,
```

`scripts/deploy/preflight.mjs:29-34`

```
name: 'JWT_SECRET',
    scope: 'api',
    validate: (value) => value.length >= 32 && value !== 'dev-secret',
    hint: 'Usar un secreto productivo de al menos 32 caracteres.',
```

`package.json:26-26`

```json
"qa:release": "pnpm qa:release:profiles && pnpm qa:release:web && pnpm qa:release:mobile && pnpm qa:release:web:visual",
```

`RELEASE_CHECKLIST.md:39-46`

````md
## QA Obligatorio Antes De Release

Ejecutar contra los dominios finales:

```powershell
corepack pnpm qa:release

corepack pnpm ops:hardening
````

**La regla para el próximo proyecto:** Aparte de los tests automáticos, mantené un set de smoke tests ejecutables (scripts simples que peguen al entorno real con credenciales reales) y hacelos requisito de release. Modelá los permisos como una matriz declarativa endpoint x rol con el status esperado, acumulá TODOS los fallos antes de salir con process.exit(1), y validá contratos reales (incluyendo un envío push/email/pago real verificando su estado de entrega, ej. deliveryStatus === 'PUSH_SENT'). Sumá un preflight que valide variables de entorno con reglas concretas (secretos largos != valor de dev, URLs sin 'localhost'), migraciones, type-check y /health antes de deployar. Componé todo en un único comando (qa:release) y documentá en un RELEASE_CHECKLIST qué comandos son obligatorios.

**Señal de que lo estás usando bien:** Existe un único comando (ej. pnpm qa:release) que un agente o dev puede correr contra el entorno real y que, si algo de permisos, contrato, push o infra está roto, sale con exit 1 listando exactamente qué falló; ningún release se firma sin esa corrida en verde anotada en el checklist.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #39 — Probar el caso negativo y el contrato, no solo el happy path; #40 — RLS como red de seguridad final, testeada como atacante

---

### ÁREA 9 — El contrato entre pantallas (código compartido entre backend, web y mobile)

#### TORNILLO #42 — Un solo paquete, tres puertas (barrel @movete/shared)

**En una línea:** Centraliza tipos, validaciones y constantes en un paquete workspace y los re-exporta desde un único index para que backend/web/mobile importen todo desde @movete/shared.

**Lo que hace el Senior:** Creó packages/shared como paquete npm propio (@movete/shared) con un index.ts que es puro barrel (re-exporta types, validations y permissions). Lo declaró como dependencia workspace:* en api, web-admin, mobile y ui, y lo cableó tanto en tsconfig paths (resolución en dev, en el tsconfig.json raíz) como en vitest.config del API (alias a ../shared/src/index.ts para tests). Así cualquier consumidor escribe import { X } from '@movete/shared' sin rutas relativas frágiles entre paquetes.

**Ejemplo real del repo:**

`packages/shared/src/index.ts:1-8`

```ts
// Types
export * from './types';

// Validations
export * from './validations';

// Permissions
export * from './permissions';
```

`packages/shared/package.json:2-7`

```json
"name": "@movete/shared",
  "version": "1.0.0",
  "description": "Shared types, utilities, and constants for Movete",
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
```

`apps/web-admin/package.json:15`

```json
"@movete/shared": "workspace:*",
```

`apps/mobile/package.json:25`

```json
"@movete/shared": "workspace:*",
```

`packages/api/vitest.config.ts:6-8`

```ts
alias: {
      '@movete/shared': path.resolve(__dirname, '../shared/src/index.ts'),
    },
```

`tsconfig.json:28`

```json
"@movete/shared": ["./packages/shared/src"],
```

**La regla para el próximo proyecto:** En un monorepo, poné el contrato compartido (tipos, validaciones, constantes) en UN paquete con nombre de scope (@org/shared) y un index.ts que sea solo re-exports. Declaralo como workspace:* en cada consumidor y agregá el alias tanto en tsconfig paths (raíz) como en la config del test runner, para que todos importen con la misma ruta de paquete y nunca con '../../..'.

**Señal de que lo estás usando bien:** Ningún archivo de app importa lógica compartida con rutas relativas que cruzan paquetes; todo entra por '@org/shared' y los tests resuelven el alias sin romper.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #43 — Zod como fuente única: el validador ES el tipo; #44 — Permisos como datos: el array const que tipa todo

---

#### TORNILLO #43 — Zod como fuente única: el validador ES el tipo

**En una línea:** Cada entrada de la API se define una sola vez como schema Zod en shared y el tipo TypeScript se deriva con z.infer, así runtime y compile-time nunca se desincronizan.

**Lo que hace el Senior:** En validations.ts define ~25 schemas (CreateClass, CreatePlan, Login, etc.) y al lado de cada uno exporta el tipo con 'export type XInput = z.infer<typeof XSchema>'. Comparte sub-piezas (TIME_REGEX, PHONE_REGEX, PASSWORD_REGEX, helper isEndTimeAfterStartTime, BusinessHourSchema, ClassAdminBaseSchema, PlanBaseSchema) y compone schemas con .partial()/.extend()/.omit()/.superRefine() en vez de reescribirlos. Las rutas y servicios del API importan el schema desde @movete/shared y lo usan para validar; el mismo tipo inferido viaja como contrato. Las reglas de negocio (hora fin > hora inicio, CLASS_PACK requiere créditos) viven dentro del schema vía refine/superRefine, no sueltas en los handlers.

**Ejemplo real del repo:**

`packages/shared/src/validations.ts:77-93`

```ts
export const CreateClassSchema = z.object({
  name: z.string().min(1).max(255),
  ...
}).refine(
  (data) => isEndTimeAfterStartTime(data.start_time, data.end_time),
  {
    message: 'End time must be after start time',
    path: ['end_time'],
  }
);

export type CreateClassInput = z.infer<typeof CreateClassSchema>;
```

`packages/shared/src/validations.ts:281-302`

```ts
function validatePlan(data: z.infer<typeof PlanBaseSchema>, ctx: z.RefinementCtx) {
  const mode = data.mode ?? (data.maxClasses ? 'CLASS_PACK' : 'FREE_PACK');
  ...
  if (mode === 'CLASS_PACK' && !creditsTotal) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      path: ['creditsTotal'],
      message: 'El Pack de clases requiere cantidad de clases',
    });
  }
  ...
}

export const CreatePlanSchema = PlanBaseSchema.superRefine(validatePlan);

export type CreatePlanInput = z.infer<typeof CreatePlanSchema>;
```

`packages/api/src/routes/classes.ts:4`

```ts
import { CreateClassAdminSchema, CreateScheduleBlockSchema, UpdateClassAdminSchema } from '@movete/shared';
```

`packages/api/src/services/auth.ts:5-12`

```ts
import {
  RegisterSchema,
  LoginSchema,
  RefreshTokenSchema,
  type RegisterInput,
  type LoginInput,
  type RefreshTokenInput,
} from '@movete/shared';
```

**La regla para el próximo proyecto:** Definí cada payload de entrada UNA vez como schema Zod en el paquete compartido y derivá el tipo con 'export type XInput = z.infer<typeof XSchema>'. Nunca declares por separado una interface y un validador para lo mismo. Meté las reglas de negocio cross-field dentro del schema (refine/superRefine) y reutilizá regex/helpers/sub-schemas componiendo con partial/extend/omit en vez de copiar.

**Señal de que lo estás usando bien:** No existe ninguna interface escrita a mano que duplique un schema Zod; el handler valida con el schema importado y el tipo del body sale solo de z.infer. Cambiar una regla en shared rompe la compilación en quien la use mal.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #42 — Un solo paquete, tres puertas (barrel @movete/shared)

---

#### TORNILLO #44 — Permisos como datos: el array const que tipa todo

**En una línea:** Un único array const ALL_PERMISSIONS es la fuente de verdad de la que se derivan el union type Permission, los roles por defecto y los labels de UI, evitando strings mágicos repetidos.

**Lo que hace el Senior:** En permissions.ts declara ALL_PERMISSIONS as const (cada permiso con key/label/group). De ahí deriva el tipo 'Permission = typeof ALL_PERMISSIONS[number]['key'] | "*"' (union estricto, no string), la lista ALL_PERMISSION_KEYS (.map), el record DEFAULT_ROLE_PERMISSIONS:Record<string,Permission[]> con los roles ADMIN/RECEPTIONIST/INSTRUCTOR/CUSTOMER, y PERMISSION_GROUP_LABELS (Set sobre los group) para agrupar en la UI. El backend usa exactamente ese DEFAULT_ROLE_PERMISSIONS para sembrar los roles en la DB (INSERT/UPDATE de la tabla roles), así el seed y el front hablan del mismo catálogo.

**Ejemplo real del repo:**

`packages/shared/src/permissions.ts:22-28`

```ts
] as const;

export type Permission = typeof ALL_PERMISSIONS[number]['key'] | '*';

export const ALL_PERMISSION_KEYS = ALL_PERMISSIONS.map((permission) => permission.key);

export const DEFAULT_ROLE_PERMISSIONS: Record<string, Permission[]> = {
```

`packages/api/src/auth/permissions.ts:20-29`

```ts
for (const [roleName, permissions] of Object.entries(DEFAULT_ROLE_PERMISSIONS)) {
    await pool.query(
      `INSERT INTO roles (id, organization_id, name, description, permissions, is_system, created_at, updated_at)
       SELECT gen_random_uuid(), o.id, $1::VARCHAR, $2::TEXT, $3::TEXT[], true, NOW(), NOW()
       ...`,
      [roleName, `${roleName} role`, permissions],
    );
```

`packages/api/src/routes/platform.ts:5`

```ts
import { DEFAULT_ROLE_PERMISSIONS } from '@movete/shared';
```

**La regla para el próximo proyecto:** Modelá los catálogos cerrados (permisos, roles, estados) como un array/objeto const TS y derivá de ahí TODO: el union type ('as const' + indexed access), las listas, los defaults y los labels de UI. Que el seed de la base y la UI consuman la MISMA constante. Nunca repitas el string 'members.read' a mano en el código.

**Señal de que lo estás usando bien:** Agregar un permiso nuevo es editar UN array; el compilador exige que aparezca con el literal correcto en los roles por defecto y la UI lo lista sola. No hay literales de permiso sueltos fuera del catálogo.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #42 — Un solo paquete, tres puertas (barrel @movete/shared)

---

#### TORNILLO #45 — El espejo Prisma↔TS: enums de DB = union types de shared

**En una línea:** Cada enum de Prisma tiene su gemelo exacto como union type de string en shared/types.ts, dándole el vocabulario de la DB a web y mobile sin que importen el cliente de Prisma.

**Lo que hace el Senior:** Por cada enum del schema.prisma (PlanType, SubscriptionStatus, ReservationStatus, PaymentStatus, PaymentMethod, InvitationStatus) hay un union type idéntico en types.ts con los mismos miembros y en el mismo orden. Esto deja que el front (que no puede ni debe importar @prisma/client) tipee estados con los mismos literales que la DB. Esos union types se vuelven a usar en las entidades compartidas (ej. Reservation.status: ReservationStatus) y se cruzan con los enums Zod de validations.ts (UpdateMembershipStatusSchema usa exactamente ACTIVE/CANCELLED/PAUSED/EXPIRED).

**Ejemplo real del repo:**

`packages/shared/src/types.ts:3-5`

```ts
export type ReservationStatus = 'PENDING' | 'CONFIRMED' | 'CHECKED_IN' | 'COMPLETED' | 'CANCELLED' | 'NO_SHOW';
export type PaymentStatus = 'PENDING' | 'PROCESSING' | 'COMPLETED' | 'FAILED' | 'REFUNDED';
export type PaymentMethod = 'CREDIT_CARD' | 'DEBIT_CARD' | 'MERCADO_PAGO' | 'BANK_TRANSFER' | 'CASH';
```

`packages/api/prisma/schema.prisma:396-419`

```prisma
enum ReservationStatus {
  PENDING
  CONFIRMED
  CHECKED_IN
  COMPLETED
  CANCELLED
  NO_SHOW
}
...
enum PaymentMethod {
  CREDIT_CARD
  DEBIT_CARD
  MERCADO_PAGO
  BANK_TRANSFER
  CASH
}
```

`packages/shared/src/types.ts:42-50`

```ts
export interface Reservation {
  id: string;
  organization_id: string;
  user_id: string;
  class_id: string;
  status: ReservationStatus;
  reservation_date: Date;
  check_in_time: Date | null;
}
```

**La regla para el próximo proyecto:** Para cada enum de Prisma que el front necesite, declará un union type de string literales idéntico en el paquete compartido y usalo en las entidades/DTOs compartidos. Así web y mobile hablan el vocabulario exacto de la DB sin acoplarse a @prisma/client. Mantené el mismo orden y los mismos miembros que el enum, y reutilizá esos literales en los z.enum de las validaciones.

**Señal de que lo estás usando bien:** El front escribe status === 'CHECKED_IN' con autocompletado y sin importar nada de Prisma; si alguien renombra un miembro del enum en la DB, el union type compartido queda como el lugar obvio (y único) a actualizar. Ojo: no hay verificación automática de que ambos lados queden sincronizados, es disciplina manual.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #42 — Un solo paquete, tres puertas (barrel @movete/shared); #43 — Zod como fuente única: el validador ES el tipo

---

#### TORNILLO #46 — Contrato de respuesta y de identidad uniforme (ApiResponse / JwtPayload / RequestContext)

**En una línea:** Define una sola forma de respuesta (ApiResponse<T>), un solo shape de token (JwtPayload) y un solo contexto de request (RequestContext) en shared, y el backend los respeta literalmente.

**Lo que hace el Senior:** En types.ts define el sobre genérico ApiResponse<T> (success/data/error{code,message,details}/meta) y PaginatedResponse<T>, más JwtPayload (con sub/org_id marcados como críticos para multi-tenancy) y RequestContext. El middleware de tenancy importa esos tipos desde @movete/shared, castea request.user a JwtPayload, arma el RequestContext y, cuando rechaza, responde con la forma exacta de ApiResponse (success:false, error:{code,message}). Es decir, el contrato no es solo declarativo: el backend serializa los errores 401 con esa estructura.

**Ejemplo real del repo:**

`packages/shared/src/types.ts:90-102`

```ts
export interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: Record<string, unknown>;
  };
  meta?: {
    timestamp: string;
    request_id: string;
  };
}
```

`packages/shared/src/types.ts:124-144`

```ts
export interface JwtPayload {
  sub: string; // user_id
  org_id: string; // organization_id (CRITICAL: multi-tenancy)
  email: string;
  ...
}

...

export interface RequestContext {
  userId: string;
  organizationId: string;
  userEmail: string;
  isAdmin: boolean;
  permissions: string[];
}
```

`packages/api/src/middleware/tenancy.ts:37-43`

```ts
(request as any).context = {
      userId: payload.sub,
      organizationId: payload.org_id,
      userEmail: payload.email,
      isAdmin: false, // Populate from DB if needed
      permissions: [], // Populate from DB if needed
    } as RequestContext;
```

**La regla para el próximo proyecto:** Definí en shared un sobre de respuesta genérico ApiResponse<T> con campo de error estructurado {code,message}, más el shape del JWT y un RequestContext, y hacé que TODOS los endpoints (incluidos los caminos de error/401/403) serialicen con esa forma. El front tipea sus fetch contra ApiResponse<T> y maneja error.code en vez de adivinar formatos por endpoint.

**Señal de que lo estás usando bien:** Toda respuesta del backend (éxito o error) cabe en ApiResponse<T>; el front puede escribir un solo handler de errores leyendo res.error.code, y sub/org_id del token tienen un único lugar de definición.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #42 — Un solo paquete, tres puertas (barrel @movete/shared); #44 — Permisos como datos: el array const que tipa todo

---

### ÁREA 10 — Cómo crece esto (patrón de expansión para un módulo nuevo)

#### TORNILLO #47 — setupXRoutes: el módulo es una función auto-instalable

**En una línea:** Cada dominio se expone como una sola función async setupXRoutes(app) que se auto-registra y arranca creando sus propias tablas, sin migraciones externas.

**Lo que hace el Senior:** Define export async function setupXRoutes(app: FastifyInstance) como punto de entrada único del módulo. Lo primero que hace dentro es await ensureXTables() (o ensureXColumns), que corre CREATE TABLE IF NOT EXISTS / ALTER TABLE ADD COLUMN IF NOT EXISTS idempotentes: el schema vive en el código del módulo, no en una carpeta de migraciones aparte. Luego declara los guards de permiso y registra cada endpoint con app.get/post/patch/delete. Ojo con el orden: en activities.ts y wods.ts el ensure es la primera línea; en facilities.ts los guards se declaran antes y el ensure va justo después de obtener el pool, pero la idea es la misma: el módulo se autoinstala. Agregar un módulo = crear un archivo routes/x.ts con esta forma y nada más.

**Ejemplo real del repo:**

`packages/api/src/routes/activities.ts:19-23`

```ts
export async function setupActivitiesRoutes(app: FastifyInstance) {
  await ensureActivitiesTables();

  const readAuth = { onRequest: [app.authenticate, requirePermission('classes.read')] } as any;
  const manageAuth = { onRequest: [app.authenticate, requirePermission('classes.manage')] } as any;
```

`packages/api/src/routes/wods.ts:66-70`

```ts
export async function setupWodsRoutes(app: FastifyInstance) {
  await ensureWodsTable();

  const readAuth = { onRequest: [app.authenticate, requirePermission('classes.read')] } as any;
  const manageAuth = { onRequest: [app.authenticate, requirePermission('classes.manage')] } as any;
```

`packages/api/src/routes/facilities.ts:16-20`

```ts
export async function setupFacilitiesRoutes(app: FastifyInstance) {
  const readAuth = { onRequest: [app.authenticate, requirePermission('facilities.read')] } as any;
  const manageAuth = { onRequest: [app.authenticate, requirePermission('facilities.manage')] } as any;
  const pool = getPgPool();
  await ensureFacilityColumns(pool);
```

`packages/api/src/services/activities.ts:5-8`

```ts
export async function ensureActivitiesTables() {
  const pool = getPgPool();
  await pool.query(`
    CREATE TABLE IF NOT EXISTS activities (
```

**La regla para el próximo proyecto:** Para cada dominio nuevo creá un único archivo routes/<dominio>.ts que exporte una función async setup<Dominio>Routes(app). Adentro, lo PRIMERO (o casi, como en facilities) es llamar a ensure<Dominio>Tables() que ejecute CREATE TABLE IF NOT EXISTS / ALTER TABLE ... ADD COLUMN IF NOT EXISTS de forma idempotente. Definí los guards de permiso justo alrededor de eso y recién ahí registrá los endpoints. No agregues lógica de un dominio fuera de su setupXRoutes.

**Señal de que lo estás usando bien:** Podés arrancar el server desde una DB vacía y el módulo crea sus tablas solo; el archivo del módulo es autocontenido y se entiende leyéndolo de arriba a abajo sin saltar a otros archivos para entender su schema.

**Aplica a:** [ ] Universal  [ ] Solo TypeScript  [x] Solo este proyecto

**Depende de:** #48 — Guards gemelos read/manage al tope del módulo; #51 — Registrar una vez en index.ts; gatear en Sidebar y hook en mobile

---

#### TORNILLO #48 — Guards gemelos read/manage al tope del módulo

**En una línea:** Cada módulo declara exactamente dos objetos de auth (readAuth y manageAuth) que combinan authenticate + requirePermission y se reusan en todos los endpoints.

**Lo que hace el Senior:** Apenas entra a setupXRoutes, crea const readAuth = { onRequest: [app.authenticate, requirePermission('x.read')] } y const manageAuth = { onRequest: [app.authenticate, requirePermission('x.manage')] }. Los GET usan readAuth; los POST/PATCH/PUT/DELETE usan manageAuth. El permiso sigue la convención de string '<recurso>.<accion>' (facilities.read, facilities.manage). Detalle real: activities.ts y wods.ts reusan los permisos 'classes.read'/'classes.manage' aunque el recurso sea otro, o sea que la matriz de permisos es consistente pero no siempre 1:1 con el nombre del recurso. Esto fija una matriz de permisos predecible en todo el backend y evita repetir el array de hooks en cada ruta.

**Ejemplo real del repo:**

`packages/api/src/routes/facilities.ts:17-18`

```ts
  const readAuth = { onRequest: [app.authenticate, requirePermission('facilities.read')] } as any;
  const manageAuth = { onRequest: [app.authenticate, requirePermission('facilities.manage')] } as any;
```

`packages/api/src/routes/wods.ts:69-70`

```ts
  const readAuth = { onRequest: [app.authenticate, requirePermission('classes.read')] } as any;
  const manageAuth = { onRequest: [app.authenticate, requirePermission('classes.manage')] } as any;
```

`packages/api/src/routes/activities.ts:25-39`

```ts
  app.get('/api/activities', readAuth, async (request) => {
...
  app.post('/api/activities', manageAuth, async (request, reply) => {
```

**La regla para el próximo proyecto:** Al abrir setupXRoutes, declará dos guards reutilizables: readAuth (authenticate + permiso '<recurso>.read') y manageAuth (authenticate + permiso '<recurso>.manage'). Aplicá readAuth a todos los GET y manageAuth a todo lo que muta. Nunca inline-es el array de hooks en cada endpoint ni inventes verbos de permiso fuera del par read/manage salvo necesidad real. Si un módulo nuevo es 'satélite' de otro (como wods/activities respecto de clases), está OK reusar el permiso del módulo padre.

**Señal de que lo estás usando bien:** Todos los GET del módulo comparten el mismo readAuth y todas las mutaciones el mismo manageAuth; el nombre del permiso es predecible (recurso.accion) y matchea con lo que el frontend chequea en ROUTE_PERMISSIONS.

**Aplica a:** [ ] Universal  [ ] Solo TypeScript  [x] Solo este proyecto

**Depende de:** #47 — setupXRoutes: el módulo es una función auto-instalable; #51 — Registrar una vez en index.ts; gatear en Sidebar y hook en mobile

---

#### TORNILLO #49 — Endpoint CRUD canónico: safeParse + tenant + soft-delete + mapX

**En una línea:** Cada handler valida con safeParse (400 con fieldErrors), filtra siempre por organization_id y deleted_at IS NULL, hace soft-delete y devuelve filas pasadas por un mapper snake_case→camelCase.

**Lo que hace el Senior:** El cuerpo de cada endpoint sigue 4 pasos fijos: (1) validar con XSchema.safeParse(request.body) y, si falla, reply.code(400).send({ message: 'Datos inválidos', errors: parsed.error.flatten().fieldErrors }); (2) toda query SQL lleva organization_id = $N (aislamiento multi-tenant, RLS a mano) y deleted_at IS NULL; (3) borrar es UPDATE ... SET deleted_at = NOW() (soft delete), no DELETE físico; (4) las filas crudas pasan por una función mapX(row) que renombra snake_case a camelCase y castea tipos (parseFloat de decimales, fechas a ISO). El UPDATE usa COALESCE($n, col) para updates parciales. Matiz real: en activities.ts y wods.ts se usa una función mapX explícita; en facilities.ts el mapeo se hace con alias SQL (postal_code AS "postalCode") en vez de un mapper en JS, pero el efecto camelCase es el mismo.

**Ejemplo real del repo:**

`packages/api/src/routes/activities.ts:41-44`

```ts
    const parsed = CreateActivitySchema.safeParse(request.body);
    if (!parsed.success) {
      return reply.code(400).send({ message: 'Datos inválidos', errors: parsed.error.flatten().fieldErrors });
    }
```

`packages/api/src/routes/activities.ts:8-17`

```ts
function mapActivity(row: any) {
  return {
    id: row.id,
    name: row.name,
    description: row.description,
    individualPrice: row.individual_price != null ? parseFloat(row.individual_price) : null,
    isActive: row.is_active,
    createdAt: row.created_at,
  };
}
```

`packages/api/src/routes/wods.ts:218-224`

```ts
    const result = await pool.query(
      `UPDATE wods SET deleted_at = NOW(), updated_at = NOW()
       WHERE id = $1::UUID AND organization_id = $2::UUID AND deleted_at IS NULL
       RETURNING id`,
      [id, user.org_id],
    );
    if (!result.rows.length) return reply.code(404).send({ message: 'WOD no encontrado' });
```

`packages/api/src/routes/facilities.ts:176-180`

```ts
    const result = await pool.query(
      `UPDATE facilities SET deleted_at = NOW(), updated_at = NOW()
       WHERE id = $1 AND organization_id = $2 AND deleted_at IS NULL RETURNING id`,
      [id, user.org_id],
    );
```

**La regla para el próximo proyecto:** En cada handler que muta: 1) validá el body con Schema.safeParse y devolvé 400 con error.flatten().fieldErrors si falla; 2) incluí SIEMPRE organization_id = $N AND deleted_at IS NULL en el WHERE de toda query (multi-tenant); 3) implementá DELETE como UPDATE SET deleted_at = NOW(); 4) nunca devuelvas la fila cruda de la DB: o pasala por un mapX(row) que traduzca snake_case a camelCase y castee decimales/fechas, o usá alias en el SELECT (col AS "camelCase"). Usá COALESCE($n, col) para updates parciales.

**Señal de que lo estás usando bien:** Ningún query toca datos de otra organización (todos llevan organization_id), no existe DELETE físico, el JSON de respuesta es 100% camelCase y un body inválido siempre devuelve 400 con el shape { message, errors }.

**Aplica a:** [ ] Universal  [ ] Solo TypeScript  [x] Solo este proyecto

**Depende de:** #50 — Schema Zod compartido: Create + Update.partial() como fuente de verdad; #47 — setupXRoutes: el módulo es una función auto-instalable

---

#### TORNILLO #50 — Schema Zod compartido: Create + Update.partial() como fuente de verdad

**En una línea:** Las validaciones viven en packages/shared como CreateXSchema, se derivan los updates con .partial() y el tipo TS con z.infer, reusándolos en backend y frontend.

**Lo que hace el Senior:** Define el schema una vez en packages/shared/src/validations.ts: CreateXSchema = z.object({...}) con mensajes de error en español, y UpdateXSchema = CreateXSchema.partial() para que el PATCH/PUT acepte campos opcionales sin reescribir el shape. Exporta también el tipo con export type CreateXInput = z.infer<typeof CreateXSchema>. El backend importa estos schemas desde @movete/shared. Detalle real: en facilities, UpdateFacilitySchema usa CreateFacilitySchema.partial().extend({...}) (agrega algún campo extra al partial). Excepción pragmática: cuando un módulo es muy chico o nuevo (wods), define el schema inline con z dentro del propio routes file, pero igual usa la dupla Input + Input.partial().

**Ejemplo real del repo:**

`packages/shared/src/validations.ts:253-263`

```ts
export const CreateActivitySchema = z.object({
  name: z.string().min(1, 'El nombre es requerido').max(255),
...
});

export type CreateActivityInput = z.infer<typeof CreateActivitySchema>;

export const UpdateActivitySchema = CreateActivitySchema.partial();
```

`packages/api/src/routes/facilities.ts:4`

```ts
import { CreateFacilitySchema, UpdateFacilitySchema } from '@movete/shared';
```

`packages/shared/src/validations.ts:360-378`

```ts
export const CreateFacilitySchema = z.object({
...
export type CreateFacilityInput = z.infer<typeof CreateFacilitySchema>;

export const UpdateFacilitySchema = CreateFacilitySchema.partial().extend({
```

`packages/api/src/routes/wods.ts:6-17`

```ts
const WodInputSchema = z.object({
  date: z.string().regex(/^\d{4}-\d{2}-\d{2}$/),
  title: z.string().min(1).max(255),
...
});

const WodUpdateSchema = WodInputSchema.partial();
```

**La regla para el próximo proyecto:** Definí la validación de cada recurso una sola vez con Zod: CreateXSchema = z.object({...}) y derivá el update con CreateXSchema.partial() (o .partial().extend({...}) si necesitás un campo más) en vez de reescribirlo. Exportá el tipo con z.infer. Ponelo en el paquete shared para reusarlo en API y clientes; solo dejalo inline en el routes file si el módulo es trivial. Los mensajes de error van en el idioma del usuario, dentro del schema.

**Señal de que lo estás usando bien:** El POST y el PATCH/PUT del mismo recurso comparten el mismo objeto de campos (uno es .partial() del otro), no hay tipos duplicados a mano, y el mismo schema valida en backend y tipa en frontend.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #49 — Endpoint CRUD canónico: safeParse + tenant + soft-delete + mapX

---

#### TORNILLO #51 — Registrar una vez en index.ts; gatear en Sidebar y hook en mobile

**En una línea:** El módulo se enchufa con un solo await setupXRoutes(app) en index.ts, una entrada en navItems del Sidebar (gateada por ROUTE_PERMISSIONS) y un hook React Query con queryKey por dominio en useApi.ts.

**Lo que hace el Senior:** El último paso del backend es agregar el import y un await setupXRoutes(app) en la lista secuencial de index.ts (todos los módulos se registran ahí, en orden, dentro de la sección // Setup routes). En web-admin el módulo se hace visible agregando un objeto { href, label, icon } a navItems del Sidebar; la visibilidad la decide ROUTE_PERMISSIONS (mapa ruta→permiso) filtrando con hasPermission (visibleNavItems = navItems.filter(...)), así el nav respeta los mismos permisos que el backend. En mobile, cada recurso expone un hook (useActivities, useTodayWods) que envuelve useQuery con un queryKey nombrado por dominio y unwrapList para tolerar respuestas array o { data }; las mutaciones invalidan esas mismas queryKeys.

**Ejemplo real del repo:**

`packages/api/src/index.ts:82-96`

```ts
  await setupActivitiesRoutes(app);
  await setupClassesRoutes(app);
...
  await setupFacilitiesRoutes(app);
...
  await setupWodsRoutes(app);
```

`apps/web-admin/components/layout/Sidebar.tsx:35-38`

```ts
  const visibleNavItems = navItems.filter((item) => {
    const required = ROUTE_PERMISSIONS[item.href];
    return !required || hasPermission(user, required);
  });
```

`apps/web-admin/lib/permissions.ts:8-22`

```ts
export const ROUTE_PERMISSIONS: Record<string, Permission> = {
  '/dashboard': 'stats.read',
  '/clases': 'classes.read',
...
  '/sedes': 'facilities.read',
...
};
```

`apps/mobile/src/hooks/useApi.ts:301-309`

```ts
export function useActivities() {
  const enabled = useHasAccessToken();
  return useQuery({
    queryKey: ['activities'],
    enabled,
    queryFn: async () => {
      const { data } = await api.get<ListResponse<Activity>>('/api/activities');
      return unwrapList(data);
```

**La regla para el próximo proyecto:** Para terminar de enchufar un módulo: 1) backend: agregá su import y un await setup<Dominio>Routes(app) a la lista secuencial de registro en el entrypoint del server (sección // Setup routes de index.ts); 2) admin web: agregá { href, label, icon } al array navItems y una entrada ruta→permiso en el mapa ROUTE_PERMISSIONS para que el nav se filtre por permisos; 3) mobile/cliente: creá un hook use<Recurso> que envuelva useQuery con un queryKey por dominio y unwrapList para normalizar la respuesta, y en las mutaciones invalidá esas mismas queryKeys. Mantené los nombres de permiso idénticos entre backend y frontend.

**Señal de que lo estás usando bien:** Un módulo recién agregado aparece en el nav solo para usuarios con el permiso correcto, sus datos se cachean bajo un queryKey claro y las mutaciones refrescan justo las listas afectadas; el server lo registra en un único lugar predecible.

**Aplica a:** [ ] Universal  [ ] Solo TypeScript  [x] Solo este proyecto

**Depende de:** #47 — setupXRoutes: el módulo es una función auto-instalable; #48 — Guards gemelos read/manage al tope del módulo

---

### Integraciones externas y dinero (Mercado Pago multi-tenant, billing de plataforma)

#### TORNILLO #52 — Credenciales cifradas en reposo con clave derivada y cadena de fallback

**En una línea:** Los secretos de terceros (Access Token, webhook secret) nunca se guardan en texto plano: se cifran con AES-256-GCM y se versionan con prefijo para poder rotar el formato.

**Lo que hace el Senior:** Centraliza encryptSecret/decryptSecret en payment-settings.ts. La clave se deriva con SHA-256 de PAYMENT_SETTINGS_SECRET (con fallback a MERCADO_PAGO_CREDENTIALS_SECRET, luego JWT_SECRET, y por ultimo una constante local 'movete-local-payment-settings-secret' solo para dev). Usa AES-256-GCM (cifrado autenticado: detecta manipulacion via authTag), IV aleatorio de 12 bytes por valor, y serializa todo como 'v1:iv:tag:ciphertext' en base64. El prefijo 'v1:' permite versionar el esquema y, en decrypt, si el valor no empieza con 'v1:' lo devuelve tal cual (compatibilidad con datos viejos sin cifrar). Reusa exactamente estas mismas funciones desde platform-billing.ts (las importa) para los secretos de la plataforma, en vez de reimplementar el cifrado.

**Ejemplo real del repo:**

`packages/api/src/services/payment-settings.ts:16-31`

```ts
const source = process.env.PAYMENT_SETTINGS_SECRET
    ?? process.env.MERCADO_PAGO_CREDENTIALS_SECRET
    ?? process.env.JWT_SECRET
    ?? 'movete-local-payment-settings-secret';
  return crypto.createHash('sha256').update(source).digest();
...
  const cipher = crypto.createCipheriv('aes-256-gcm', encryptionKey(), iv);
...
  return `v1:${iv.toString('base64')}:${tag.toString('base64')}:${encrypted.toString('base64')}`;
```

`packages/api/src/services/payment-settings.ts:33-39`

```ts
export function decryptSecret(value?: string | null) {
  if (!value) return null;
  if (!value.startsWith('v1:')) return value;
...
  decipher.setAuthTag(Buffer.from(tagRaw, 'base64'));
```

`packages/api/src/services/platform-billing.ts:3-3`

```ts
import { decryptSecret, encryptSecret } from './payment-settings';
```

**La regla para el próximo proyecto:** Nunca guardes secretos de terceros (API keys, access tokens, webhook secrets) en texto plano en la DB. Centraliza encrypt/decrypt en UN modulo, usa cifrado autenticado (AES-256-GCM, no AES-CBC), IV aleatorio por valor, deriva la clave de un secreto de entorno dedicado (no reuses el de JWT salvo como fallback explicito), y prefija el ciphertext con una version de esquema ('v1:') para poder rotar el algoritmo sin romper datos existentes. En decrypt, trata los valores sin prefijo como legacy/plaintext.

**Señal de que lo estás usando bien:** Si abris la tabla en la DB ves cadenas tipo 'v1:...:...:...' y nunca el token real; si cambias PAYMENT_SETTINGS_SECRET, los valores viejos dejan de descifrarse (senal de que realmente dependian de la clave); y agregar un nuevo proveedor de secretos solo requiere importar las dos funciones.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #56 — Fallback por entorno con bandera de origen explicita; #54 — Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero

---

#### TORNILLO #53 — Webhook idempotente que no confia en el cliente y re-verifica server-to-server

**En una línea:** Cada webhook de pago se desduplica con una fila UNIQUE, valida firma HMAC con comparacion de tiempo constante, y confirma el pago re-consultando la API del proveedor en lugar de creer lo que llega en el body o el redirect.

**Lo que hace el Senior:** Construye un eventKey deterministico (tipo:dataId:requestId) e inserta en payment_webhook_events con ON CONFLICT (provider, event_key) DO NOTHING; si el INSERT no devuelve fila, responde {duplicate:true} y corta (idempotencia ante reintentos de Mercado Pago, respaldado por UNIQUE(provider,event_key) en la tabla). Valida la firma x-signature reconstruyendo el manifest 'id:..;request-id:..;ts:..;' y comparando el HMAC-SHA256 con timingSafeEqualHex, que usa crypto.timingSafeEqual sobre buffers hex (evita timing attacks). NO activa el abono con el dato del webhook: re-consulta GET /v1/payments/{id} con el Access Token de esa organizacion (verificacion server-to-server) y recien ahi mapea el estado. Toda la activacion corre dentro de una transaccion (BEGIN) con SELECT ... FOR UPDATE sobre la solicitud de compra. La doc lo deja explicito: 'El redirect de Mercado Pago a la app no confirma el pago'. Nota: si la firma es invalida pero hay webhookSecret, no rechaza duro, solo loguea un warning y sigue con la validacion via lookup del pago.

**Ejemplo real del repo:**

`packages/api/src/routes/payments.ts:884-894`

```ts
INSERT INTO payment_webhook_events (provider, event_key, provider_payment_id, payload)
       VALUES ('MERCADO_PAGO', $1, $2, $3::JSONB)
       ON CONFLICT (provider, event_key) DO NOTHING
       RETURNING id
...
    if (!eventInsert.rows.length) return { received: true, duplicate: true };
```

`packages/api/src/routes/payments.ts:236-238`

```ts
const manifest = `id:${String(dataId).toLowerCase()};request-id:${requestId};ts:${signature.ts};`;
    const expected = crypto.createHmac('sha256', secret).update(manifest).digest('hex');
    return timingSafeEqualHex(expected, signature.v1);
```

`packages/api/src/routes/payments.ts:214-221`

```ts
function timingSafeEqualHex(a: string, b: string) {
    try {
      const left = Buffer.from(a, 'hex');
      const right = Buffer.from(b, 'hex');
      return left.length === right.length && crypto.timingSafeEqual(left, right);
```

`packages/api/src/routes/payments.ts:896-918`

```ts
const mpPayment = await fetchMercadoPagoPayment(organizationId, dataId);
    const externalReference = mpPayment.external_reference;
    const providerStatus = mpPayment.status;
    const mappedStatus = paymentStatusFromMercadoPago(providerStatus);
...
         WHERE pr.organization_id = $2
           AND (pr.external_reference = $1 OR pr.id::TEXT = $1)
         FOR UPDATE
```

`docs/mercado-pago.md:40-40`

```md
El redirect de Mercado Pago a la app no confirma el pago. La activación solo ocurre por webhook y verificación server-to-server.
```

**La regla para el próximo proyecto:** Para cualquier webhook de pagos: (1) desduplica con una tabla cuyo UNIQUE sea (provider, event_key) y corta si ya existia; (2) valida la firma con HMAC y crypto.timingSafeEqual sobre buffers, nunca con '==='; (3) NUNCA marques un pago como pagado con los datos que vienen en el body o el query del redirect: re-consulta el recurso en la API del proveedor con tu propia credencial y deriva el estado de ahi; (4) haz la mutacion de estado dentro de una transaccion con FOR UPDATE sobre la fila objetivo. Responde 200 a duplicados y a no-matcheados para que el proveedor deje de reintentar.

**Señal de que lo estás usando bien:** Reenviar el mismo webhook dos veces no duplica abonos ni pagos; un body falsificado no activa nada porque la verdad viene del GET al proveedor; y un atacante no puede distinguir por tiempo si la firma estuvo cerca de ser valida.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #52 — Credenciales cifradas en reposo con clave derivada y cadena de fallback; #54 — Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero

---

#### TORNILLO #54 — Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero

**En una línea:** Cada operacion de pago resuelve y filtra por organization_id de punta a punta, y cada gimnasio usa sus propias credenciales de Mercado Pago, de modo que el dinero liquida en la cuenta correcta y nada se mezcla entre tenants.

**Lo que hace el Senior:** El webhook exige el org_id (viene en el query string de la notification_url como org_id y como fallback en metadata.movete_organization_id) y rechaza con 400 si falta; con ese org_id carga las credenciales especificas de esa organizacion via getMercadoPagoSettings (404 si no esta configurada). La preferencia de pago se crea con el Access Token de la org activa (Mercado Pago liquida en la cuenta del gimnasio, Movete no cobra en su nombre). En el circuito de plataforma, el external_reference codifica el tenant como 'movete:{organizationId}' y al sincronizar lo desempaqueta con replace(/^movete:/,''). La query de matcheo de solicitudes incluye 'WHERE pr.organization_id = $2' antes de aplicar el pago.

**Ejemplo real del repo:**

`packages/api/src/routes/payments.ts:859-873`

```ts
const organizationId = String(query?.org_id ?? body?.metadata?.movete_organization_id ?? '');
...
    if (!organizationId) {
      return reply.code(400).send({ message: 'Missing Mercado Pago organization id' });
    }
...
    const settings = await getMercadoPagoSettings(organizationId);
    if (!settings?.enabled || !settings.accessToken) {
```

`packages/api/src/routes/payments.ts:914-919`

```ts
FROM membership_purchase_requests pr
         WHERE pr.organization_id = $2
           AND (pr.external_reference = $1 OR pr.id::TEXT = $1)
         FOR UPDATE
```

`packages/api/src/services/platform-billing.ts:583-635`

```ts
const externalReference = `movete:${input.organizationId}`;
...
  const organizationId = String(payload.external_reference ?? '').replace(/^movete:/, '');
```

`docs/mercado-pago.md:3-3`

```md
Movete no cobra en nombre de todos los gimnasios: la API crea la preferencia usando el Access Token de la organización activa y Mercado Pago liquida el dinero en esa cuenta.
```

**La regla para el próximo proyecto:** En un SaaS multi-tenant que integra pagos, haz que el tenant viaje en todo el circuito: incluye el tenant_id en la notification_url del webhook (no solo en metadata, que puede no venir), codificalo tambien en el external_reference con un prefijo namespaced ('app:{id}') para reconciliar, carga las credenciales del proveedor por-tenant (cada cliente cobra en su cuenta), y agrega 'AND tenant_id = $X' a TODA query que toque dinero antes de mutar. Rechaza con 400/404 si el tenant no se puede resolver, en vez de aplicar el pago a ciegas.

**Señal de que lo estás usando bien:** Un webhook sin org_id se rechaza con 400; un pago de un gimnasio jamas activa un abono en otro; y el dinero aparece en la cuenta Mercado Pago del gimnasio correcto, no en una cuenta central.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #52 — Credenciales cifradas en reposo con clave derivada y cadena de fallback; #53 — Webhook idempotente que no confia en el cliente y re-verifica server-to-server

---

#### TORNILLO #55 — Billing de la plataforma como gate de permisos con 402

**En una línea:** El estado de suscripcion del propio SaaS se hace cumplir en el mismo middleware de permisos: si el gimnasio no pago, se degrada a solo-lectura y se responde HTTP 402 en las acciones de gestion.

**Lo que hace el Senior:** Modela el estado de suscripcion como un enum acotado (trialing/active/past_due/restricted/suspended/cancelled) respaldado por un CHECK en la DB. refreshBillingStatus hace lazy-expiration: en cada chequeo, si el trial o la gracia vencieron, transiciona el estado a 'restricted' en la misma query UPDATE (no necesita un cron para degradar). canUseOrganizationPermission decide acceso con sets explicitos: FULL_ACCESS_STATUSES (trialing/active/past_due) pasan todo; 'restricted' solo deja permisos que estan en READ_ONLY_PERMISSIONS; suspended/cancelled solo dejan los de SUSPENDED_PERMISSIONS, que es {'settings.read'}. Los CUSTOMER (alumnos) nunca quedan bloqueados por el billing B2B (return temprano). Esto se enchufa dentro de requirePermission: tras validar el permiso RBAC normal (403 si falta), consulta el billing y, si no esta permitido, responde 402 (Payment Required) con el billingStatus, distinguiendo 'no tenes permiso' (403) de 'tu suscripcion no esta al dia' (402).

**Ejemplo real del repo:**

`packages/api/src/services/platform-billing.ts:528-535`

```ts
export async function canUseOrganizationPermission(organizationId: string, roleName: string, requiredPermission: string) {
  if (roleName === 'CUSTOMER') return { allowed: true };
  const subscription = await refreshBillingStatus(organizationId);
  const status = subscription?.status ?? 'trialing';
  if (FULL_ACCESS_STATUSES.has(status)) return { allowed: true };
  if (status === 'restricted' && READ_ONLY_PERMISSIONS.has(requiredPermission)) return { allowed: true, status };
```

`packages/api/src/services/platform-billing.ts:511-516`

```ts
SET status = CASE
           WHEN status = 'trialing' AND trial_ends_at IS NOT NULL AND trial_ends_at < NOW() THEN 'restricted'
           WHEN status = 'past_due' AND grace_ends_at IS NOT NULL AND grace_ends_at < NOW() THEN 'restricted'
           ELSE status
         END,
```

`packages/api/src/services/platform-billing.ts:25-38`

```ts
export type PlatformBillingStatus = 'trialing' | 'active' | 'past_due' | 'restricted' | 'suspended' | 'cancelled';

const FULL_ACCESS_STATUSES = new Set<PlatformBillingStatus>(['trialing', 'active', 'past_due']);
const READ_ONLY_PERMISSIONS = new Set([
...
const SUSPENDED_PERMISSIONS = new Set(['settings.read']);
```

`packages/api/src/services/platform-billing.ts:119-121`

```ts
CONSTRAINT organization_subscriptions_status_check CHECK (
        status IN ('trialing', 'active', 'past_due', 'restricted', 'suspended', 'cancelled')
      )
```

`packages/api/src/auth/permissions.ts:79-85`

```ts
const billingAccess = await canUseOrganizationPermission(user.org_id, roleName, required);
    if (!billingAccess.allowed) {
      return reply.code(402).send({
        message: 'La suscripción de Movete requiere atención',
        billingStatus: billingAccess.status,
      });
```

**La regla para el próximo proyecto:** Para gatear features por estado de suscripcion del propio SaaS: modela el estado como un enum cerrado con CHECK en la DB; calcula la expiracion de trial/gracia de forma lazy dentro del mismo UPDATE que lee el estado (asi un trial vencido se degrada solo, sin depender de un cron); define los permisos permitidos por estado con Sets explicitos (read-only para 'restricted', minimo para 'suspended'/'cancelled'); enchufa el chequeo en el MISMO middleware que ya valida permisos RBAC; y responde 402 Payment Required (no 403) para que el front pueda distinguir 'falta de permiso' de 'falta de pago' y mostrar un CTA de upgrade. Excluye explicitamente a los usuarios finales (clientes del cliente) del bloqueo B2B.

**Señal de que lo estás usando bien:** Un gimnasio con trial vencido puede seguir viendo sus datos pero no editar, y la API le devuelve 402 con el billingStatus en vez de 403; los alumnos del gimnasio nunca se ven afectados; y agregar un estado nuevo te obliga a tocar el enum, el CHECK y los Sets, sin logica de fechas dispersa.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #54 — Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero

---

#### TORNILLO #56 — Fallback por entorno con bandera de origen explicita

**En una línea:** Si una organizacion no configuro sus credenciales en la DB, el sistema cae a variables de entorno pero marca el 'source' del valor para que el resto del codigo (y el operador) sepa de donde salio.

**Lo que hace el Senior:** getMercadoPagoSettings, si no hay fila en la DB, intenta MERCADO_PAGO_ACCESS_TOKEN del entorno y devuelve un objeto con lastValidationStatus:'env_fallback' (no null silencioso). getPlatformPaymentSettings hace lo mismo a nivel plataforma pero mas explicito: devuelve source:'database'|'env'|'none' y banderas hasAccessToken/configured, combinando DB y env (la DB tiene prioridad, el env rellena lo que falta). La doc deja claro que el fallback por env es solo compatibilidad y 'no se recomienda usarlos en produccion multiempresa'. Asi el mismo codigo corre en dev (con un token global en .env) y en prod multi-tenant (cada org con su token en la DB) sin ramas distintas.

**Ejemplo real del repo:**

`packages/api/src/services/payment-settings.ts:98-110`

```ts
const fallbackToken = process.env.MERCADO_PAGO_ACCESS_TOKEN ?? null;
    return fallbackToken
      ? {
          organizationId,
          enabled: true,
...
          lastValidationStatus: 'env_fallback',
```

`packages/api/src/services/platform-billing.ts:347-355`

```ts
configured: Boolean(accessToken || envAccessToken),
    accessToken: accessToken ?? envAccessToken,
    webhookSecret,
    hasAccessToken: Boolean(accessToken || envAccessToken),
    hasWebhookSecret: Boolean(webhookSecret),
    source: accessToken ? 'database' : (envAccessToken ? 'env' : 'none'),
```

`docs/mercado-pago.md:100-100`

```md
`MERCADO_PAGO_ACCESS_TOKEN` y `MERCADO_PAGO_WEBHOOK_SECRET` siguen funcionando como fallback si una organización no tiene configuración propia. No se recomienda usarlos en producción multiempresa.
```

**La regla para el próximo proyecto:** Cuando una config pueda venir de la DB (por-tenant) o del entorno (global), implementa un fallback en cascada DB -> env -> none, pero NUNCA lo devuelvas como un valor opaco: incluye en el resultado un campo 'source'/'origin' que diga de donde salio y banderas booleanas (configured, hasX). Eso hace el comportamiento auditable, permite que la UI muestre 'usando credencial del entorno (no recomendado)', y deja que el mismo codigo corra en dev y prod sin if-else por ambiente. Documenta que el fallback por env es solo compatibilidad/dev.

**Señal de que lo estás usando bien:** Podes preguntarle al sistema 'de donde salio esta credencial' y te responde 'database'/'env'/'none'; en dev funciona con un solo token en .env; y en prod un operador detecta de inmediato una org que quedo usando el fallback global por error.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #52 — Credenciales cifradas en reposo con clave derivada y cadena de fallback

---

### ÁREA TRANSVERSAL — Patrones no cubiertos

_Encontré cuatro patrones transversales recurrentes que no estaban en la lista: (1) una receta de paginación idéntica copiada en al menos tres rutas, con clamp defensivo de límites y un envelope fijo { data, total, page, totalPages }; (2) un constructor incremental de WHERE/params dinámico para filtros y búsqueda que aparece en 5+ rutas; (3) una convención de exportación CSV con BOM para Excel, CRLF, escapeo RFC-4180 y filename con fecha, repetida 3 veces y testeada por contrato; y (4) jobs en background gateados por header-secret, con tabla de run-log y deduplicación idempotente vía índice único parcial. Todo respaldado por código real en packages/api/src/routes y services._

#### TORNILLO #57 — Paginación con clamp defensivo y envelope { data, total, page, totalPages }

**En una línea:** Toda lista paginada lee page/limit del query, los acota con Math.min/Math.max para que el cliente no rompa la DB, y devuelve siempre el mismo sobre con total y totalPages.

**Lo que hace el Senior:** En cada endpoint de listado repite exactamente tres líneas: pageNum = Math.max(1, parseInt(page ?? default)), limitNum = Math.min(100, Math.max(1, parseInt(limit ?? default))) y offset = (pageNum - 1) * limitNum. El limit nunca puede pasar de 100 ni bajar de 1, así un cliente malicioso o un bug no piden 'limit=999999'. Después corre DOS queries: un COUNT(*) con el mismo WHERE para el total, y la query de datos con LIMIT/OFFSET parametrizados por posición ($N-1 y $N). El response es siempre el mismo objeto: { data, total, page, totalPages: Math.ceil(total / limitNum) }. El contrato está pineado por test (members.test.ts espera page, limit y totalPages en el body).

**Ejemplo real del repo:**

`packages/api/src/routes/audit.ts:16-18`

```ts
const pageNum = Math.max(1, parseInt(page ?? '1', 10));
    const limitNum = Math.min(100, Math.max(1, parseInt(limit ?? '30', 10)));
    const offset = (pageNum - 1) * limitNum;
```

`packages/api/src/routes/facilities.ts:29-31`

```ts
const pageNum = Math.max(1, parseInt(page ?? '1', 10));
    const limitNum = Math.min(100, Math.max(1, parseInt(limit ?? '20', 10)));
    const offset = (pageNum - 1) * limitNum;
```

`packages/api/src/routes/members.ts:163-168`

```ts
return {
      data: result.rows,
      total,
      page: pageNum,
      limit: limitNum,
      totalPages: Math.ceil(total / limitNum),
```

`packages/api/src/routes/members.test.ts:68-70`

```ts
expect(body).toHaveProperty('page', 1);
    expect(body).toHaveProperty('limit', 20);
    expect(body).toHaveProperty('totalPages');
```

**La regla para el próximo proyecto:** Para cualquier endpoint que devuelva una lista, parseá page/limit del query y acotalos SIEMPRE: limitNum = Math.min(TOPE, Math.max(1, parseInt(limit ?? DEFAULT))). Corré un COUNT(*) con el mismo WHERE que la query de datos, y devolvé un envelope uniforme { data, total, page, totalPages }. Nunca confíes en el limit del cliente sin techo.

**Señal de que lo estás usando bien:** Pedir ?limit=999999 devuelve como máximo el tope (ej 100) y ?page=0 o negativo cae en page 1; el body siempre trae total y totalPages aunque la lista venga vacía; un test de integración puede afirmar la forma del envelope sin conocer los datos.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #14 — organization_id en cada query, sin excepción; #16 — El cliente tolera el envelope, no lo asume

---

#### TORNILLO #58 — Constructor incremental de WHERE + params para filtros opcionales

**En una línea:** Los filtros opcionales (search, role, action, isActive) se arman empujando al array params y referenciando $${params.length}, de modo que el placeholder se numera solo y nunca hay SQL por concatenación de valores.

**Lo que hace el Senior:** Arranca con un WHERE base que ya incluye organization_id como $1 y un array params = [user.org_id]. Por cada filtro opcional que vino en el query hace params.push(valor) y concatena al WHERE usando el índice calculado: AND col = $${params.length}. Para búsqueda hace params.push(`%${search}%`) y reusa el mismo $${params.length} en varios ILIKE. Para paginar arma dataParams = [...params, limitNum, offset] y cierra con LIMIT $${dataParams.length - 1} OFFSET $${dataParams.length}. El valor del usuario JAMÁS entra interpolado en el string SQL; solo entra el número de posición, que es seguro. Esto evita inyección y mantiene el orden de binds sincronizado aunque agregues o saques filtros.

**Ejemplo real del repo:**

`packages/api/src/routes/members.ts:129-137`

```ts
if (search) {
      params.push(`%${search}%`);
      baseWhere += ` AND (u.first_name ILIKE $${params.length} OR u.last_name ILIKE $${params.length} OR u.email ILIKE $${params.length})`;
    }

    if (role) {
      params.push(role);
      baseWhere += ` AND r.name = $${params.length}`;
    }
```

`packages/api/src/routes/audit.ts:23-24`

```ts
if (entityType) { params.push(entityType); where += ` AND al.entity_type = $${params.length}`; }
    if (action)     { params.push(action);     where += ` AND al.action = $${params.length}`; }
```

`packages/api/src/routes/facilities.ts:36-39`

```ts
if (search) {
      params.push(`%${search}%`);
      where += ` AND (name ILIKE $${params.length} OR city ILIKE $${params.length} OR address ILIKE $${params.length})`;
    }
```

**La regla para el próximo proyecto:** Para construir queries con filtros opcionales y SQL crudo parametrizado, mantené un array params y agregá cada condición con params.push(valor) referenciando $${params.length}. Nunca interpoles el valor en el string; solo el número de placeholder. Para LIMIT/OFFSET, derivá los índices de [...params, limit, offset].length. Si usás un ORM, esto se traduce en construir el objeto where condicionalmente, nunca string SQL a mano con valores.

**Señal de que lo estás usando bien:** Agregar o quitar un filtro no descoordina la numeración de los binds (todo sigue funcionando porque los $N se calculan, no se hardcodean); ningún valor de usuario aparece literal en el SQL; pasar comillas o ' OR 1=1 en search no rompe ni inyecta, solo no matchea.

**Aplica a:** [ ] Universal  [x] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #14 — organization_id en cada query, sin excepción; #57 — Paginación con clamp defensivo y envelope { data, total, page, totalPages }

---

#### TORNILLO #59 — CSV listo para Excel: BOM, CRLF, escapeo RFC y filename con fecha

**En una línea:** Las exportaciones serializan a CSV con un helper que escapa según RFC-4180, unen con CRLF, prependen un BOM UTF-8 y mandan Content-Disposition con nombre fechado, para que Excel abra acentos y comas sin romperse.

**Lo que hace el Senior:** Tiene un único toCSV(rows) que toma las keys de la primera fila como headers, y por cada celda: si es null devuelve vacío, si tiene coma/comilla/salto de línea la envuelve en comillas dobles y duplica las comillas internas (str.replace(/\"/g, '\"\"')). Une las filas con '\r\n' (CRLF, no \n) porque Excel en Windows lo espera. Al responder setea Content-Type 'text/csv; charset=utf-8', un Content-Disposition attachment con filename que incluye la fecha ISO recortada (`socios-${new Date().toISOString().slice(0,10)}.csv`), y CRÍTICO: antepone '﻿' + csv (el BOM) para que Excel detecte UTF-8 y muestre los acentos. Las cabeceras de columna van en español directamente desde el alias SQL (AS "Nombre"). El patrón se repite idéntico en members, payments y reservations, y hay un test que verifica el header y que NO se filtre un statusCode de error en el body.

**Ejemplo real del repo:**

`packages/api/src/routes/export.ts:5-22`

```ts
function toCSV(rows: Record<string, unknown>[]): string {
  if (!rows.length) return '';
  const headers = Object.keys(rows[0]);
  const lines = [
    headers.join(','),
    ...rows.map((row) =>
      headers
        .map((h) => {
          const val = row[h];
          if (val == null) return '';
          const str = String(val).replace(/"/g, '""');
          return str.includes(',') || str.includes('\n') || str.includes('"') ? `"${str}"` : str;
        })
```

`packages/api/src/routes/export.ts:50-52`

```ts
reply.header('Content-Type', 'text/csv; charset=utf-8');
    reply.header('Content-Disposition', `attachment; filename="socios-${new Date().toISOString().slice(0, 10)}.csv"`);
    return reply.send('﻿' + csv); // BOM for Excel
```

`packages/api/src/routes/export.test.ts:45-60`

```ts
it.each([
    ['/api/export/members', 'Nombre,Apellido,Email'],
    ['/api/export/payments', 'Fecha,Socio,Email'],
    ['/api/export/reservations', 'Fecha,Socio,Email'],
  ])('returns csv for %s', async (url, expectedHeader) => {
    ...
    expect(res.headers['content-type']).toContain('text/csv');
    expect(res.body).toContain(expectedHeader);
    expect(res.body).not.toContain('"statusCode"');
```

**La regla para el próximo proyecto:** Cuando exportes CSV pensado para abrirse en Excel: escapá cada celda con reglas RFC-4180 (comillas dobles + duplicar comillas internas si hay coma/comilla/newline), uní filas con '\r\n', prependé el BOM '﻿' al string final, y respondé con Content-Type text/csv; charset=utf-8 y Content-Disposition attachment con filename que lleve la fecha. Generá las cabeceras humanas directamente como alias de la query. Cubrílo con un test de contrato que afirme el header de columnas.

**Señal de que lo estás usando bien:** Abrís el archivo en Excel en Windows y los acentos (á, é, ñ) se ven bien, las columnas no se desarman cuando una celda tiene comas, y el archivo descargado ya viene con nombre fechado tipo socios-2026-06-24.csv.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #14 — organization_id en cada query, sin excepción; #32 — Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission

---

#### TORNILLO #60 — Jobs en background: gate por header-secret, run-log en DB y dedupe idempotente por índice único parcial

**En una línea:** Los jobs programados (recordatorios, vencimientos, pagos vencidos, billing) se disparan por un endpoint sin auth de usuario pero protegido por un secret en header, registran cada corrida en una tabla de run-log, y evitan duplicados con una dedupeKey persistida más un índice único parcial que hace el INSERT idempotente.

**Lo que hace el Senior:** Expone /api/jobs/notifications/run-scheduled SIN el guard normal de usuario (porque lo llama un cron externo, no una persona): compara process.env.NOTIFICATION_JOBS_SECRET contra el header x-notification-jobs-secret y si no coincide responde 401. Además mantiene la variante /run con permiso de usuario para disparo manual. Cada suite de jobs abre un registro en notification_job_runs con status RUNNING, y al final lo marca COMPLETED o FAILED con un details JSONB de resumen (created/skipped), envolviendo todo en try/catch para que un fallo quede asentado en la DB. La idempotencia es la joya: cada notificación lleva una dedupeKey determinística (`job:${jobName}:${dateKey}:${entityId}`) que incluye la fecha, y el INSERT usa ON CONFLICT (organization_id, delivery_details) WHERE delivery_details LIKE 'job:%' DO NOTHING, respaldado por un índice único parcial. Así, si el cron corre dos veces el mismo día, la segunda vez todo cae en 'skipped' en vez de spamear al socio. El contador created vs skipped sale de mirar si rowCount === 1.

**Ejemplo real del repo:**

`packages/api/src/routes/jobs.ts:13-20`

```ts
app.post('/api/jobs/notifications/run-scheduled', async (request, reply) => {
    const expected = process.env.NOTIFICATION_JOBS_SECRET;
    const provided = request.headers['x-notification-jobs-secret'];
    if (!expected || provided !== expected) {
      return reply.code(401).send({ message: 'Unauthorized' });
    }
    return runNotificationJobs(null);
  });
```

`packages/api/src/services/notification-jobs.ts:34-38`

```ts
await pool.query(`
    CREATE UNIQUE INDEX IF NOT EXISTS idx_notifications_job_dedupe
      ON notifications(organization_id, delivery_details)
      WHERE delivery_details LIKE 'job:%'
  `);
```

`packages/api/src/services/notification-jobs.ts:56-61`

```ts
ON CONFLICT (organization_id, delivery_details) WHERE delivery_details LIKE 'job:%'
     DO NOTHING
     RETURNING id`,
    [uuidv4(), input.orgId, input.title, input.message, input.targetUserId, input.dedupeKey, input.createdBy ?? null],
  );
  return result.rowCount === 1;
```

`packages/api/src/services/notification-jobs.ts:303-308`

```ts
const runId = uuidv4();
  await pool.query(
    `INSERT INTO notification_job_runs (id, job_name, status, started_at)
     VALUES ($1, 'notification-suite', 'RUNNING', NOW())`,
    [runId],
  );
```

**La regla para el próximo proyecto:** Para jobs disparados por cron externo: protegé el endpoint con un secret comparado contra un header (no con la auth de usuario) y respondé 401 si no coincide o si el env no está seteado. Registrá cada corrida en una tabla de run-log con status RUNNING -> COMPLETED/FAILED y un resumen JSONB dentro de try/catch. Hacé cada efecto idempotente: armá una dedupeKey determinística que incluya la fecha del día y un INSERT ... ON CONFLICT DO NOTHING respaldado por un índice único (parcial si solo aplica a filas de job). Distinguí 'creado' de 'omitido' mirando rowCount.

**Señal de que lo estás usando bien:** Correr el job dos veces el mismo día no genera notificaciones duplicadas (el segundo run reporta todo como skipped); llamar el endpoint sin el header secreto da 401; y siempre podés mirar la tabla de run-log para ver cuándo corrió, si falló y cuántos registros tocó.

**Aplica a:** [x] Universal  [ ] Solo TypeScript  [ ] Solo este proyecto

**Depende de:** #53 — Webhook idempotente que no confia en el cliente y re-verifica server-to-server; #9 — El org_id sale SIEMPRE del JWT, nunca del cliente; #37 — Feature flags con default apagado para lo caro o riesgoso

---

## Mapa de dependencias

Qué tornillos necesitan a otros para funcionar bien. Si vas a aplicar uno, mirá de qué se apoya: rara vez sirven sueltos.

- **#1 — El package interno se referencia con workspace:* y se importa con alias @scope/*** se apoya en **#3** (tsconfig raíz único + override por proyecto).
- **#2 — apps/ son deployables, packages/ son librerías, y se nota en el build** se apoya en **#1** (El package interno se referencia con workspace:* y se importa con alias @scope/*).
- **#4 — Turbo orquesta respetando el grafo (^build) y cachea solo lo determinístico** se apoya en **#2** (apps/ son deployables, packages/ son librerías, y se nota en el build), **#1** (El package interno se referencia con workspace:* y se importa con alias @scope/*).
- **#5 — JWT flaco para identidad, DB gorda para permisos** se apoya en **#8** (Catálogo de permisos como array de strings, fuente única compartida), **#6** (Cadena onRequest: primero autenticar, después autorizar (y de paso cobrar)).
- **#6 — Cadena onRequest: primero autenticar, después autorizar (y de paso cobrar)** se apoya en **#5** (JWT flaco para identidad, DB gorda para permisos), **#8** (Catálogo de permisos como array de strings, fuente única compartida).
- **#7 — Access token en memoria, refresh token persistido y rotado** se apoya en **#5** (JWT flaco para identidad, DB gorda para permisos).
- **#8 — Catálogo de permisos como array de strings, fuente única compartida** se apoya en **#5** (JWT flaco para identidad, DB gorda para permisos), **#6** (Cadena onRequest: primero autenticar, después autorizar (y de paso cobrar)).
- **#9 — El org_id sale SIEMPRE del JWT, nunca del cliente** se apoya en **#11** (Fail-closed: sin contexto de tenant, cero datos), **#10** (Cada query carga su propio WHERE organization_id (filtro app-layer omnipresente)).
- **#10 — Cada query carga su propio WHERE organization_id (filtro app-layer omnipresente)** se apoya en **#9** (El org_id sale SIEMPRE del JWT, nunca del cliente).
- **#11 — Fail-closed: sin contexto de tenant, cero datos** se apoya en **#9** (El org_id sale SIEMPRE del JWT, nunca del cliente), **#12** (RLS de Postgres con GUC como red de seguridad de DB (defensa en profundidad)).
- **#12 — RLS de Postgres con GUC como red de seguridad de DB (defensa en profundidad)** se apoya en **#11** (Fail-closed: sin contexto de tenant, cero datos), **#10** (Cada query carga su propio WHERE organization_id (filtro app-layer omnipresente)).
- **#13 — La transacción como guardián de la regla de negocio** se apoya en **#29** (Fail-closed multi-tenant: nunca una query sin org_id + soft delete), **#15** (Validar con Zod safeParse antes de tocar la DB).
- **#14 — organization_id en cada query, sin excepción** se apoya en **#13** (La transacción como guardián de la regla de negocio).
- **#15 — Validar con Zod safeParse antes de tocar la DB** se apoya en **#13** (La transacción como guardián de la regla de negocio).
- **#16 — El cliente tolera el envelope, no lo asume** se apoya en **#17** (Invalidación dirigida de React Query con actualización optimista).
- **#17 — Invalidación dirigida de React Query con actualización optimista** se apoya en **#16** (El cliente tolera el envelope, no lo asume), **#27** (Refresh silencioso de token en 401 con deduplicación (mismo patrón en mobile y web)).
- **#18 — La plantilla de tabla multi-tenant (el molde que se copia y pega)** se apoya en **#19** (Índices deliberados sobre FK y deleted_at (no dejar que Postgres adivine)), **#20** (onDelete con intención: Cascade para hijos, SetNull para histórico).
- **#19 — Índices deliberados sobre FK y deleted_at (no dejar que Postgres adivine)** se apoya en **#18** (La plantilla de tabla multi-tenant (el molde que se copia y pega)).
- **#20 — onDelete con intención: Cascade para hijos, SetNull para histórico** se apoya en **#18** (La plantilla de tabla multi-tenant (el molde que se copia y pega)).
- **#21 — RLS como última línea fail-closed (defense in depth en la base)** se apoya en **#18** (La plantilla de tabla multi-tenant (el molde que se copia y pega)).
- **#22 — Conexiones como singleton con shutdown ordenado, health check y escape hatch a pg crudo** se apoya en **#18** (La plantilla de tabla multi-tenant (el molde que se copia y pega)).
- **#23 — Errores de negocio se atajan, errores del sistema explotan** se apoya en _Logger pino centralizado con handler global_, _Transacciones con ROLLBACK en el catch_.
- **#24 — Envelope de error { message } en español, uniforme (con una deuda viva en auth)** se apoya en **#30** (Validación con Zod safeParse y 400 estructurado antes del SQL), _Parser de error en el cliente HTTP_.
- **#25 — Validar en la frontera con Zod safeParse y traducir SIEMPRE igual** se apoya en **#24** (Envelope de error { message } en español, uniforme (con una deuda viva en auth)), _Schemas compartidos en packages/shared_.
- **#26 — Webhook de pago a prueba de duplicados (idempotencia + firma + verdad del proveedor)** se apoya en **#23** (Errores de negocio se atajan, errores del sistema explotan), _Logger pino centralizado_.
- **#27 — Refresh silencioso de token en 401 con deduplicación (mismo patrón en mobile y web)** se apoya en **#24** (Envelope de error { message } en español, uniforme (con una deuda viva en auth)), **#7** (Access token en memoria, refresh token persistido y rotado).
- **#28 — Factory de rutas: setupXRoutes(app)** se apoya en **#32** (Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission), **#31** (Auto-migración ensureXTable/ensureXColumn en el arranque).
- **#29 — Fail-closed multi-tenant: nunca una query sin org_id + soft delete** se apoya en **#28** (Factory de rutas: setupXRoutes(app)), **#32** (Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission).
- **#30 — Validación con Zod safeParse y 400 estructurado antes del SQL** se apoya en **#29** (Fail-closed multi-tenant: nunca una query sin org_id + soft delete).
- **#31 — Auto-migración ensureXTable/ensureXColumn en el arranque** se apoya en **#28** (Factory de rutas: setupXRoutes(app)).
- **#32 — Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission** se apoya en **#28** (Factory de rutas: setupXRoutes(app)), **#29** (Fail-closed multi-tenant: nunca una query sin org_id + soft delete).
- **#33 — El .env.example como contrato narrado, no como lista muerta** se apoya en **#35** (Fallbacks de env en cascada: DB -> env -> default local), **#34** (Secrets multiempresa cifrados con AES-256-GCM, nunca en texto plano).
- **#34 — Secrets multiempresa cifrados con AES-256-GCM, nunca en texto plano** se apoya en **#35** (Fallbacks de env en cascada: DB -> env -> default local), **#33** (El .env.example como contrato narrado, no como lista muerta).
- **#35 — Fallbacks de env en cascada: DB -> env -> default local** se apoya en **#33** (El .env.example como contrato narrado, no como lista muerta), **#34** (Secrets multiempresa cifrados con AES-256-GCM, nunca en texto plano).
- **#36 — Config de cliente como build-arg, config de servidor como runtime env** se apoya en **#33** (El .env.example como contrato narrado, no como lista muerta).
- **#37 — Feature flags con default apagado para lo caro o riesgoso** se apoya en **#33** (El .env.example como contrato narrado, no como lista muerta), **#35** (Fallbacks de env en cascada: DB -> env -> default local).
- **#38 — Test de integración con DB real, sin mocks (inject + seed)** se apoya en **#39** (Probar el caso negativo y el contrato, no solo el happy path), **#41** (QA por smoke tests contra entorno real, gateando el release).
- **#39 — Probar el caso negativo y el contrato, no solo el happy path** se apoya en **#38** (Test de integración con DB real, sin mocks (inject + seed)).
- **#40 — RLS como red de seguridad final, testeada como atacante** se apoya en **#38** (Test de integración con DB real, sin mocks (inject + seed)), **#41** (QA por smoke tests contra entorno real, gateando el release).
- **#41 — QA por smoke tests contra entorno real, gateando el release** se apoya en **#39** (Probar el caso negativo y el contrato, no solo el happy path), **#40** (RLS como red de seguridad final, testeada como atacante).
- **#42 — Un solo paquete, tres puertas (barrel @movete/shared)** se apoya en **#43** (Zod como fuente única: el validador ES el tipo), **#44** (Permisos como datos: el array const que tipa todo).
- **#43 — Zod como fuente única: el validador ES el tipo** se apoya en **#42** (Un solo paquete, tres puertas (barrel @movete/shared)).
- **#44 — Permisos como datos: el array const que tipa todo** se apoya en **#42** (Un solo paquete, tres puertas (barrel @movete/shared)).
- **#45 — El espejo Prisma↔TS: enums de DB = union types de shared** se apoya en **#42** (Un solo paquete, tres puertas (barrel @movete/shared)), **#43** (Zod como fuente única: el validador ES el tipo).
- **#46 — Contrato de respuesta y de identidad uniforme (ApiResponse / JwtPayload / RequestContext)** se apoya en **#42** (Un solo paquete, tres puertas (barrel @movete/shared)), **#44** (Permisos como datos: el array const que tipa todo).
- **#47 — setupXRoutes: el módulo es una función auto-instalable** se apoya en **#48** (Guards gemelos read/manage al tope del módulo), **#51** (Registrar una vez en index.ts; gatear en Sidebar y hook en mobile).
- **#48 — Guards gemelos read/manage al tope del módulo** se apoya en **#47** (setupXRoutes: el módulo es una función auto-instalable), **#51** (Registrar una vez en index.ts; gatear en Sidebar y hook en mobile).
- **#49 — Endpoint CRUD canónico: safeParse + tenant + soft-delete + mapX** se apoya en **#50** (Schema Zod compartido: Create + Update.partial() como fuente de verdad), **#47** (setupXRoutes: el módulo es una función auto-instalable).
- **#50 — Schema Zod compartido: Create + Update.partial() como fuente de verdad** se apoya en **#49** (Endpoint CRUD canónico: safeParse + tenant + soft-delete + mapX).
- **#51 — Registrar una vez en index.ts; gatear en Sidebar y hook en mobile** se apoya en **#47** (setupXRoutes: el módulo es una función auto-instalable), **#48** (Guards gemelos read/manage al tope del módulo).
- **#52 — Credenciales cifradas en reposo con clave derivada y cadena de fallback** se apoya en **#56** (Fallback por entorno con bandera de origen explicita), **#54** (Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero).
- **#53 — Webhook idempotente que no confia en el cliente y re-verifica server-to-server** se apoya en **#52** (Credenciales cifradas en reposo con clave derivada y cadena de fallback), **#54** (Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero).
- **#54 — Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero** se apoya en **#52** (Credenciales cifradas en reposo con clave derivada y cadena de fallback), **#53** (Webhook idempotente que no confia en el cliente y re-verifica server-to-server).
- **#55 — Billing de la plataforma como gate de permisos con 402** se apoya en **#54** (Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero).
- **#56 — Fallback por entorno con bandera de origen explicita** se apoya en **#52** (Credenciales cifradas en reposo con clave derivada y cadena de fallback).
- **#57 — Paginación con clamp defensivo y envelope { data, total, page, totalPages }** se apoya en **#14** (organization_id en cada query, sin excepción), **#16** (El cliente tolera el envelope, no lo asume).
- **#58 — Constructor incremental de WHERE + params para filtros opcionales** se apoya en **#14** (organization_id en cada query, sin excepción), **#57** (Paginación con clamp defensivo y envelope { data, total, page, totalPages }).
- **#59 — CSV listo para Excel: BOM, CRLF, escapeo RFC y filename con fecha** se apoya en **#14** (organization_id en cada query, sin excepción), **#32** (Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission).
- **#60 — Jobs en background: gate por header-secret, run-log en DB y dedupe idempotente por índice único parcial** se apoya en **#53** (Webhook idempotente que no confia en el cliente y re-verifica server-to-server), **#9** (El org_id sale SIEMPRE del JWT, nunca del cliente), **#37** (Feature flags con default apagado para lo caro o riesgoso).

### La cadena de seguridad multi-tenant (la más importante)

Esta es LA cadena que no podés romper. Si falla un eslabón, se te filtran datos entre clientes. Leéla de arriba (lo que el cliente manda) hacia abajo (la última barrera en la base):

```
     [ El cliente manda un request con un :id cualquiera ]
                          |
                          v
  #9 (El org_id sale SIEMPRE del JWT, nunca del cliente)  El tenant sale del JWT firmado, NUNCA del cliente
       (si el token no trae org_id -> 401: fail-closed)
                          |
                          v
  #11 (Fail-closed: sin contexto de tenant, cero datos)  Sin contexto de tenant, cero datos
       (el default seguro es negar, no devolver todo)
                          |
                          v
  #10 (Cada query carga su propio WHERE organization_id (filtro app-layer omnipresente))  Cada query lleva WHERE organization_id = $tokenOrg
       (+ los :id de la URL solo resuelven dentro del scope del org)
                          |
                          v
  #12 (RLS de Postgres con GUC como red de seguridad de DB (defensa en profundidad))  Row Level Security como red de seguridad en la base
       (capa final: aunque te olvides el WHERE, la DB te corta)
                          |
                          v
  #40 (RLS como red de seguridad final, testeada como atacante)  Un test que actúa como atacante intenta cruzar tenants y exige el fallo
```

> Regla mental: el `tenant_id` **se origina en el token y viaja en el `WHERE` de toda query**. Las demás capas (fail-closed, RLS, test atacante) existen para cuando un humano se olvida de la primera.

---

## Cómo usar este manual

Sos un agente de IA (o un dev nuevo) y te toca arrancar un proyecto SZoluciones distinto. **Antes de escribir una sola línea de código**, hacé esto:

1. **Leé el índice completo y filtrá por la columna "Aplica a".** Los `Universal` son tu piso obligatorio: aplican casi siempre. Los `Solo TS` sumalos si el stack es TypeScript. Los `Solo proyecto` son específicos de Movete (RLS desconectada del pool, auto-migración con `ensure`, build-args de Cloud Run): leelos como referencia, no los copies a ciegas.

2. **¿El proyecto es multi-tenant? Entonces la cadena de seguridad (#9 → #11 → #10 → #12 → #40) no es negociable.** Implementala COMPLETA y EN ORDEN antes de cualquier feature: primero el tenant sale del token, después fail-closed, después el `WHERE organization_id` en cada query, y al final la red de seguridad (RLS) y el test que actúa como atacante. Si te salteás el primer eslabón, todo lo demás es decoración.

3. **Montá el esqueleto antes que las features**, en este orden: (a) estructura del monorepo y tsconfig heredado; (b) el contrato compartido en un paquete `shared` (tipos + validaciones Zod + constantes); (c) la base con tu plantilla de tabla; (d) auth + permisos; (e) la config y los secrets. Tornillos clave: #1, #2, #3, #42, #43, #18, #5, #8, #34, #33.

4. **Para cada módulo de negocio nuevo, seguí el patrón de expansión** (Área 10) al pie de la letra: tipos+validación en `shared` → service con la regla → route con `setup*Routes` + guard de permiso + filtro de tenant → registrar en un solo lugar → página web → pantalla+hook mobile. Tornillo ancla: **#47 (setupXRoutes: el módulo es una función auto-instalable)**.

5. **Antes de declarar "terminado", pasá por tests y QA** (Área 8). No alcanza el happy path: probá el caso negativo (401/400/409), el aislamiento multi-tenant como atacante, y corré los smoke tests contra un entorno real con la matriz de permisos por rol. Si algo sale con `exit != 0`, no está terminado. Tornillo ancla: **#38 (Test de integración con DB real, sin mocks (inject + seed))**.

### Ejemplo trabajado: un sistema de fidelización de clientes

Imaginá que el proyecto nuevo es un **programa de puntos** (cada empresa con sus propios clientes y sus reglas de canje). Los tornillos que casi seguro vas a necesitar:

- **Multi-tenant (los puntos de una empresa no tocan a otra):** #9 (El org_id sale SIEMPRE del JWT, nunca del cliente) + #10 (Cada query carga su propio WHERE organization_id (filtro app-layer omnipresente)).
- **Roles (quién carga reglas de canje vs quién consulta su saldo):** #5 (JWT flaco para identidad, DB gorda para permisos) + #8 (Catálogo de permisos como array de strings, fuente única compartida).
- **Contrato compartido (qué es un punto, un canje, una recompensa: definido una sola vez):** #42 (Un solo paquete, tres puertas (barrel @movete/shared)) + #43 (Zod como fuente única: el validador ES el tipo).
- **Consistencia bajo concurrencia (canjear consume un saldo finito: dos canjes no gastan el mismo punto):** #13 (La transacción como guardián de la regla de negocio) + #15 (Validar con Zod safeParse antes de tocar la DB).
- **Secrets multiempresa (si cada empresa integra su proveedor de notificaciones/pagos):** #34 (Secrets multiempresa cifrados con AES-256-GCM, nunca en texto plano) + #35 (Fallbacks de env en cascada: DB -> env -> default local).
- **Arranque y flags (levantar sin pedir secretos, apagar por defecto lo caro):** #33 (El .env.example como contrato narrado, no como lista muerta) + #37 (Feature flags con default apagado para lo caro o riesgoso).

Fijate el patrón: **no inventás arquitectura nueva**, ensamblás tornillos ya probados. Un programa de puntos es, estructuralmente, las reservas de Movete con otro nombre.

### Regla de oro

> El `tenant_id` sale del token firmado y viaja en el `WHERE` de toda query; el contrato vive una sola vez en el paquete `shared`; y nada se da por "terminado" hasta que un test que actúa como atacante intenta cruzar tenants y **falla**. Si dudás entre filtrar de más o devolver de más, **negá**: una fuga de datos no se perdona, un 404 de más sí.
