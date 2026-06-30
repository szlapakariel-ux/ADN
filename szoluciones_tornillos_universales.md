# SZoluciones — Tornillos Universales
### Los principios que aplican en cualquier stack (Python, Node, PHP, Go… lo que sea)

> Esta es la versión condensada de [`szoluciones_tornillos_typescript.md`](szoluciones_tornillos_typescript.md). Acá quedaron **solo los tornillos marcados `Universal`**: los que no dependen de TypeScript, Prisma, Fastify ni de ningún stack en particular. Son principios de ingeniería —tokens, queries, transacciones, errores, secrets, tests— que valen igual si mañana escribís en Django, Laravel, Spring o Express.

> **Los números se conservan del manual completo.** Si ves "#9" acá, es el mismo #9 de allá: así podés saltar al original para ver el ejemplo de código concreto cuando lo necesites.

**32 tornillos universales** (de 60 totales). Lo que NO está acá (Solo TypeScript / Solo este proyecto) vive en el manual completo.

---

## Índice

| # | Nombre coloquial | Área |
|---|------------------|------|
| 2 | apps/ son deployables, packages/ son librerías, y se nota en el build | ÁREA 1 — El plano de la casa |
| 5 | JWT flaco para identidad, DB gorda para permisos | ÁREA 2 — La puerta y el portero |
| 8 | Catálogo de permisos como array de strings, fuente única compartida | ÁREA 2 — La puerta y el portero |
| 9 | El org_id sale SIEMPRE del JWT, nunca del cliente | ÁREA 2b — Mismo dato, distinta respuesta |
| 10 | Cada query carga su propio WHERE organization_id (filtro app-layer omnipresente) | ÁREA 2b — Mismo dato, distinta respuesta |
| 11 | Fail-closed: sin contexto de tenant, cero datos | ÁREA 2b — Mismo dato, distinta respuesta |
| 13 | La transacción como guardián de la regla de negocio | ÁREA 3 — El viaje de un dato |
| 14 | organization_id en cada query, sin excepción | ÁREA 3 — El viaje de un dato |
| 18 | La plantilla de tabla multi-tenant (el molde que se copia y pega) | ÁREA 4 — El archivo del negocio |
| 19 | Índices deliberados sobre FK y deleted_at (no dejar que Postgres adivine) | ÁREA 4 — El archivo del negocio |
| 20 | onDelete con intención: Cascade para hijos, SetNull para histórico | ÁREA 4 — El archivo del negocio |
| 23 | Errores de negocio se atajan, errores del sistema explotan | ÁREA 5 — Cuando algo se rompe |
| 24 | Envelope de error { message } en español, uniforme (con una deuda viva en auth) | ÁREA 5 — Cuando algo se rompe |
| 26 | Webhook de pago a prueba de duplicados (idempotencia + firma + verdad del proveedor) | ÁREA 5 — Cuando algo se rompe |
| 28 | Factory de rutas: setupXRoutes(app) | ÁREA 6 — La letra del Senior |
| 29 | Fail-closed multi-tenant: nunca una query sin org_id + soft delete | ÁREA 6 — La letra del Senior |
| 32 | Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission | ÁREA 6 — La letra del Senior |
| 33 | El .env.example como contrato narrado, no como lista muerta | ÁREA 7 — Las llaves del local |
| 34 | Secrets multiempresa cifrados con AES-256-GCM, nunca en texto plano | ÁREA 7 — Las llaves del local |
| 37 | Feature flags con default apagado para lo caro o riesgoso | ÁREA 7 — Las llaves del local |
| 38 | Test de integración con DB real, sin mocks (inject + seed) | ÁREA 8 — Cuándo algo está realmente terminado |
| 39 | Probar el caso negativo y el contrato, no solo el happy path | ÁREA 8 — Cuándo algo está realmente terminado |
| 40 | RLS como red de seguridad final, testeada como atacante | ÁREA 8 — Cuándo algo está realmente terminado |
| 41 | QA por smoke tests contra entorno real, gateando el release | ÁREA 8 — Cuándo algo está realmente terminado |
| 46 | Contrato de respuesta y de identidad uniforme (ApiResponse / JwtPayload / RequestContext) | ÁREA 9 — El contrato entre pantallas |
| 52 | Credenciales cifradas en reposo con clave derivada y cadena de fallback | Integraciones externas y dinero |
| 53 | Webhook idempotente que no confia en el cliente y re-verifica server-to-server | Integraciones externas y dinero |
| 54 | Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero | Integraciones externas y dinero |
| 55 | Billing de la plataforma como gate de permisos con 402 | Integraciones externas y dinero |
| 57 | Paginación con clamp defensivo y envelope { data, total, page, totalPages } | ÁREA TRANSVERSAL — Patrones no cubiertos |
| 59 | CSV listo para Excel: BOM, CRLF, escapeo RFC y filename con fecha | ÁREA TRANSVERSAL — Patrones no cubiertos |
| 60 | Jobs en background: gate por header-secret, run-log en DB y dedupe idempotente por índice único parcial | ÁREA TRANSVERSAL — Patrones no cubiertos |

---

## Los tornillos universales

### ÁREA 1 — El plano de la casa (estructura del monorepo)

#### TORNILLO #2 — apps/ son deployables, packages/ son librerías, y se nota en el build

**En una línea:** La separación apps/ vs packages/ no es cosmética: los packages emiten .d.ts y dist, los apps no emiten nada (los empaqueta su bundler).

**La regla para el próximo proyecto:** Separá el monorepo en apps/ (deployables) y packages/ (librerías) desde el día uno y declaralo en el archivo de workspaces. A los packages dales main/types/files:[dist] y un build que emita declaraciones (.d.ts); a las apps dejalas con noEmit:true y que las empaquete su bundler (Next/Vite/esbuild/Expo). Si un paquete no tiene un consumidor claro o un app exporta tipos, algo está mal ubicado.

**Señal de que lo estás usando bien:** Cada carpeta en packages/ tiene 'types' y emite dist/; ninguna carpeta en apps/ exporta 'types'. Un app nunca aparece como dependencia de otro paquete; solo los packages aparecen como dependencia.

---

### ÁREA 2 — La puerta y el portero (autenticación y autorización)

#### TORNILLO #5 — JWT flaco para identidad, DB gorda para permisos

**En una línea:** El token dice quién sos; la base de datos, en cada request, dice qué podés hacer.

**La regla para el próximo proyecto:** Mete en el JWT solo identidad estable (user id, tenant/org id, email) y como mucho un snapshot de permisos para mostrar UI. Para AUTORIZAR cada endpoint sensible, releé los permisos vivos desde la DB en un hook/middleware, filtrando por usuario activo y no borrado. Nunca dejes que un permiso revocado siga valiendo hasta que expire el token.

**Señal de que lo estás usando bien:** Si le sacás un permiso a un usuario en la DB y su próximo request (con el mismo access token vigente) ya recibe 403, lo aplicaste bien. Si tenés que esperar a que expire el token para que el cambio pegue, lo hiciste mal.

---

#### TORNILLO #8 — Catálogo de permisos como array de strings, fuente única compartida

**En una línea:** Los permisos son strings tipo 'members.manage' definidos una sola vez en el paquete shared, con '*' como comodín y ADMIN siempre normalizado.

**La regla para el próximo proyecto:** Modelá permisos como strings recurso.acción (members.read) en UN solo módulo compartido entre back y front, con un comodín '*' y un rol super-admin que siempre se normaliza a ['*']. La función de chequeo (admin OR comodín OR match exacto) debe ser la misma en ambos lados. Al sembrar roles por defecto, pisá siempre al admin pero respetá los permisos ya configurados de los demás roles (solo completá si están vacíos, con un WHERE sobre array_length).

**Señal de que lo estás usando bien:** Agregar un permiso nuevo es agregar un string al catálogo del paquete shared y back y front lo reconocen sin cambios de tipos en otro lado. El rol admin nunca queda sin acceso aunque alguien edite roles en la UI.

---

### ÁREA 2b — Mismo dato, distinta respuesta (multi-tenancy y Row Level Security)

#### TORNILLO #9 — El org_id sale SIEMPRE del JWT, nunca del cliente

**En una línea:** El identificador de tenant que filtra los datos se toma del token firmado (request.user.org_id), nunca de un parámetro o body que el cliente pueda manipular.

**La regla para el próximo proyecto:** El identificador de tenant (org/cuenta/workspace) SIEMPRE se extrae del token de sesión ya verificado, nunca de query params, body ni headers controlables por el cliente. Si el token no trae tenant, rechazá la request con 401 antes de tocar la base. Los IDs que vienen del cliente (params de ruta, body) se usan solo como filtro adicional, siempre combinados con `AND tenant_id = <del token>`.

**Señal de que lo estás usando bien:** No existe ninguna query de negocio donde el tenant_id provenga de request.params, request.body o request.query. Buscás 'org_id' en un route y todas las ocurrencias apuntan a user.org_id / request.user. Cambiar un :id en la URL por uno de otro tenant devuelve 404, no datos ajenos.

---

#### TORNILLO #10 — Cada query carga su propio WHERE organization_id (filtro app-layer omnipresente)

**En una línea:** Toda consulta a una tabla de negocio incluye organization_id en el WHERE (y en el INSERT/UPDATE/DELETE), sin excepción, como disciplina manual repetida.

**La regla para el próximo proyecto:** En un sistema multi-tenant, TODA query a una tabla con datos de tenant lleva `WHERE tenant_id = $x` y todo INSERT/UPDATE setea/verifica esa columna, sin atajos. No confíes en que un :id sea único globalmente: combinalo siempre con el tenant. Propagá el tenant_id también a las funciones de servicio (recibilo como parámetro explícito o dentro del input), no lo dejes implícito. Si tu ORM no garantiza el filtro automáticamente, el filtro manual no es opcional: es la única barrera real.

**Señal de que lo estás usando bien:** grep de tenant_id en cualquier archivo de routes devuelve una ocurrencia por cada operación de DB (selects, inserts, updates, deletes). No hay un solo SELECT/UPDATE/DELETE sobre tabla de negocio que omita el filtro de tenant. Un test que intenta tocar un id de otro tenant devuelve 0 filas / 404.

---

#### TORNILLO #11 — Fail-closed: sin contexto de tenant, cero datos

**En una línea:** Si falta el contexto de organización, el sistema devuelve vacío o rechaza, nunca 'todo' — el default seguro es negar.

**La regla para el próximo proyecto:** Diseñá el aislamiento de forma que la AUSENCIA de contexto de tenant produzca cero resultados o un rechazo explícito, jamás acceso total. En RLS de Postgres esto sale gratis: una policy con `tenant_id = current_setting('app.tenant_id')::uuid` no matchea nada si el GUC no está seteado. En app-layer, validá la presencia del tenant ANTES de la query y cortá con 401. La pregunta de diseño es siempre: 'si esto falla a medias, ¿se filtran datos o se niegan?' — la respuesta correcta es 'se niegan'.

**Señal de que lo estás usando bien:** Un test que NO setea el contexto de tenant y corre una query devuelve 0 filas (no todas). Quitar el org_id del token produce 401, no una respuesta con datos. No hay ningún code path donde 'tenant indefinido' se traduzca en 'sin filtro'.

---

### ÁREA 3 — El viaje de un dato (de la pantalla a la base y de vuelta)

#### TORNILLO #13 — La transacción como guardián de la regla de negocio

**En una línea:** Toda escritura que toca varias tablas o consume un recurso limitado corre dentro de BEGIN/COMMIT y hace ROLLBACK ante cualquier fallo, sea de DB o de validación de negocio.

**La regla para el próximo proyecto:** Cuando una operación de escritura toque más de una tabla o consuma un recurso finito (créditos, stock, cupo), envolvela en una transacción explícita (BEGIN/COMMIT). Diseñá los servicios de negocio para que devuelvan un resultado tipo {ok:false, statusCode, message} en vez de tirar excepción, y que el handler haga ROLLBACK explícito ante {ok:false} igual que ante un error de DB. Lockeá la fila del recurso escaso con SELECT ... FOR UPDATE. Capturá los códigos de error nativos de la DB (ej 23505 unique violation) y traducilos a respuestas HTTP entendibles.

**Señal de que lo estás usando bien:** Si simulás dos reservas concurrentes sobre la última clase de un pack, solo una consume el crédito y la otra recibe el statusCode de negocio (ej 402); y si la validación de negocio falla a mitad de camino, no queda NINGUNA fila insertada ni crédito gastado.

---

#### TORNILLO #14 — organization_id en cada query, sin excepción

**En una línea:** Cada consulta SQL filtra por organization_id (y por user_id en endpoints /me) sacándolo del JWT, nunca del body del request.

**La regla para el próximo proyecto:** En un SaaS multi-tenant, el identificador de tenant (y el de usuario en endpoints self-service) SIEMPRE sale del token autenticado, jamás del body o query params del cliente. Hacé que el primer WHERE de toda query sea el filtro de tenant. Tratá un endpoint sin ese filtro como un bug de seguridad, no como un detalle de performance.

**Señal de que lo estás usando bien:** No existe ninguna query de lectura o escritura de datos de negocio sin WHERE organization_id; un usuario del tenant A nunca puede leer ni modificar filas del tenant B aunque mande el id correcto.

---

### ÁREA 4 — El archivo del negocio (diseño de base de datos)

#### TORNILLO #18 — La plantilla de tabla multi-tenant (el molde que se copia y pega)

**En una línea:** Cada tabla de negocio nace con el mismo esqueleto de columnas y metadatos, no se improvisa modelo por modelo.

**La regla para el próximo proyecto:** Definí un molde de tabla estándar y aplicalo sin excepción a toda entidad de negocio: PK opaca (cuid/uuid, nunca int autoincremental en SaaS multi-tenant), columna de tenant (organization_id) en cada tabla que contenga datos de cliente, y siempre created_at + updated_at + deleted_at para auditoría y soft-delete. Para dinero usá Decimal con precisión y escala explícitas, jamás Float. Antes de agregar una tabla, copiá el molde primero y después agregá los campos específicos.

**Señal de que lo estás usando bien:** Podés abrir cualquier modelo nuevo y encontrar las mismas 4-5 columnas base; ningún campo de dinero es Float; ninguna tabla de negocio carece de organization_id ni de la tripleta de timestamps.

---

#### TORNILLO #19 — Índices deliberados sobre FK y deleted_at (no dejar que Postgres adivine)

**En una línea:** Indexa explícitamente cada clave foránea, el discriminador de tenant, las columnas de filtro frecuente y deleted_at, y usa uniques compuestos para reglas de negocio.

**La regla para el próximo proyecto:** Indexá a mano: toda FK, la columna de tenant, toda columna que aparezca recurrentemente en WHERE/ORDER BY, y deleted_at si usás soft-delete. Convertí cada invariante de negocio del tipo 'no puede haber dos X iguales' en un @@unique compuesto en la base, no solo en la lógica de la app. No dependas de que el motor cree los índices por vos.

**Señal de que lo estás usando bien:** No existe una FK sin su @@index correspondiente; las reglas de unicidad del dominio están reflejadas como uniques compuestos en el schema; las queries con WHERE deleted_at IS NULL no hacen seq scan.

---

#### TORNILLO #20 — onDelete con intención: Cascade para hijos, SetNull para histórico

**En una línea:** La política de borrado en cascada se elige según si el dato es accesorio (cae con el padre) o evidencia que debe sobrevivir (se desliga, no se borra).

**La regla para el próximo proyecto:** Para cada relación decidí explícitamente el onDelete según la naturaleza del dato: Cascade cuando el hijo carece de sentido sin el padre (membresías, tokens, configuraciones), SetNull cuando el dato es evidencia que debe persistir aunque el actor desaparezca (pagos, logs de auditoría, registros legales). Cuando elijas SetNull, la columna FK debe ser nullable. Nunca dejes el onDelete por defecto sin pensarlo.

**Señal de que lo estás usando bien:** Borrar un usuario no destruye sus pagos ni los logs de auditoría (quedan con FK en NULL); borrar una organización limpia todos sus datos operativos sin dejar huérfanos.

---

### ÁREA 5 — Cuando algo se rompe (errores, logging, observabilidad)

#### TORNILLO #23 — Errores de negocio se atajan, errores del sistema explotan

**En una línea:** Distinguir el error esperado del usuario (HTTP semántico legible) del error inesperado del sistema (throw → 500 logueado por el handler global).

**La regla para el próximo proyecto:** En cada handler, hacé try/catch SOLO para mapear errores de dominio que conocés a su HTTP correspondiente (401/403/404/409/422). Para todo lo demás, re-lanzá el error (después de limpiar recursos como ROLLBACK/release) y dejá que el error handler global del framework lo loguee con stack y devuelva un 500 genérico. No loguees el stack en errores que son culpa del usuario.

**Señal de que lo estás usando bien:** Tus logs de nivel error solo contienen fallas reales del sistema (DB caída, bug), no validaciones fallidas ni 'password incorrecto'. Buscás 'throw err' en tus catches de transacciones y aparece; buscás 'reply.status(500)' y casi solo aparece en el catch raíz, no por route.

---

#### TORNILLO #24 — Envelope de error { message } en español, uniforme (con una deuda viva en auth)

**En una línea:** Toda respuesta de error que ve el usuario es `reply.code(XXX).send({ message: 'texto en español' })`, repetido idénticamente en casi todas las rutas.

**La regla para el próximo proyecto:** Elegí UN solo shape de error de cara al cliente (ej. `{ message }`) y un solo idioma, y respetalo en el 100% de las rutas desde el primer día. Si definís un tipo de envelope en el paquete compartido, USALO en runtime o no lo definas: un tipo que nadie aplica es una mentira que confunde. El cliente debe parsear ese único campo, con a lo sumo un fallback de compatibilidad temporal.

**Señal de que lo estás usando bien:** Un grep de `.send({ error:` da resultados concentrados en un solo archivo legacy (o cero) y `.send({ message:` aparece consistente en todos los demás. El parser de errores del cliente lee un solo campo con un único fallback (`payload.message ?? payload.error`).

---

#### TORNILLO #26 — Webhook de pago a prueba de duplicados (idempotencia + firma + verdad del proveedor)

**En una línea:** El webhook de Mercado Pago descarta reentregas con una tabla de eventos UNIQUE + ON CONFLICT DO NOTHING, valida firma HMAC timing-safe y nunca confía en el payload: re-consulta el pago al proveedor.

**La regla para el próximo proyecto:** Todo webhook de pago debe ser idempotente: persistí cada evento con una clave única y un INSERT ... ON CONFLICT DO NOTHING; si no insertaste fila, ya lo procesaste, respondé 200 y no hagas nada. Validá la firma con comparación timing-safe (y que un input malformado devuelva false, no que explote). Nunca confíes en el status del body: re-consultá el estado real al proveedor antes de mover plata o activar accesos. Envolvé la activación en una transacción.

**Señal de que lo estás usando bien:** Reenviar el mismo webhook dos veces no crea dos membresías ni cobra dos veces: la segunda vez devuelve duplicate:true. Un payload falsificado con status 'approved' no activa nada porque vos consultás al proveedor real con fetchMercadoPagoPayment.

---

### ÁREA 6 — La letra del Senior (estilo personal y anti-patrones)

#### TORNILLO #28 — Factory de rutas: setupXRoutes(app)

**En una línea:** Cada módulo de rutas es una función que recibe la app como dependencia y registra sus endpoints, en vez de tocar un singleton global.

**La regla para el próximo proyecto:** No registres rutas tocando un objeto global importado. Exportá una función `setupXRoutes(app, ...deps)` por dominio que reciba la instancia del servidor (y cualquier servicio que necesite) como parámetro, y registrá los handlers ahí adentro. El bootstrap central llama a cada setup. Si un módulo necesita un servicio extra, sumalo a la firma en vez de importarlo desde un singleton.

**Señal de que lo estás usando bien:** Podés instanciar el módulo de rutas en un test pasándole un `app` falso sin levantar todo el servidor; ningún archivo de routes importa una instancia global del framework.

---

#### TORNILLO #29 — Fail-closed multi-tenant: nunca una query sin org_id + soft delete

**En una línea:** Toda consulta a la DB filtra por organization_id del usuario y por deleted_at IS NULL, sin excepción.

**La regla para el próximo proyecto:** En una app multi-tenant: derivá el tenant SIEMPRE del token autenticado, nunca del input del cliente, y metelo en el WHERE de toda query incluyendo UPDATE/DELETE. Usá soft delete (columna deleted_at) y filtrá `deleted_at IS NULL` en cada lectura de entidades de negocio. Que un recurso de otro tenant devuelva 404, no 403 ni los datos. Si vas a confiar en RLS de la base, igual mantené el filtro explícito como segunda barrera.

**Señal de que lo estás usando bien:** Buscás `organization_id` con grep y aparece en absolutamente todas las queries del proyecto; pedir un id que existe pero de otra org da 404. No hay un solo SELECT/UPDATE/DELETE de entidades sin el filtro de tenant.

---

#### TORNILLO #32 — Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission

**En una línea:** El permiso se chequea como hook de Fastify en la definición de la ruta, y el mismo modelo de permisos filtra qué ve el usuario en el front.

**La regla para el próximo proyecto:** Modelá permisos como strings `recurso.accion` y chequealos de forma DECLARATIVA en la capa de ruteo (hook/middleware) atado a la definición del endpoint, no con if's dentro del handler. Armá objetos de guard reutilizables (readAuth/manageAuth) y reusalos. En el front, derivá el menú y los botones del MISMO modelo de permisos (un mapa ruta->permiso + un helper hasPermission) para ocultar lo no autorizado, pero tratá esto solo como UX: la verdad la impone el back.

**Señal de que lo estás usando bien:** Cada endpoint declara su permiso al lado de la URL y se lee de un vistazo; el front nunca muestra una sección que el back rechazaría. Los permisos siguen la convención `recurso.accion` en ambos lados.

---

### ÁREA 7 — Las llaves del local (variables de entorno, config por ambiente, secrets)

#### TORNILLO #33 — El .env.example como contrato narrado, no como lista muerta

**En una línea:** Documentar cada variable con comentarios que explican qué hace, en qué ambiente aplica y cuál es el camino recomendado vs el fallback legacy.

**La regla para el próximo proyecto:** Tratá el .env.example como documentación de primera clase: agrupá por dominio con headers (# ===== DOMINIO =====), y en cada variable sensible agregá un comentario de 1-2 líneas que diga (a) qué hace, (b) en qué ambiente se usa, y (c) si es la opción recomendada o un fallback. Usá placeholders de dev que sean obviamente inseguros (ej: 'change_me', 'dev_secret_change_in_production_...') para que nadie los confunda con valores de prod. Mantené en paralelo una lista de 'variables mínimas por servicio' en el doc de deploy.

**Señal de que lo estás usando bien:** Un dev nuevo puede copiar .env.example a .env, levantar todo en local sin pedirle secretos a nadie, y al leer los comentarios entiende qué variables son obligatorias en producción y por qué.

---

#### TORNILLO #34 — Secrets multiempresa cifrados con AES-256-GCM, nunca en texto plano

**En una línea:** Las credenciales sensibles de cada organización se guardan cifradas en la DB con una clave maestra derivada de env, no en columnas de texto plano.

**La regla para el próximo proyecto:** Cuando guardes credenciales de terceros por tenant/organización, nunca las pongas en texto plano en la DB. Derivá una clave con SHA-256 de un secreto de env dedicado (ej: PAYMENT_SETTINGS_SECRET) y cifrá cada valor con AES-256-GCM (que da confidencialidad + integridad via auth tag). Guardá el resultado en formato versionado 'v1:iv:tag:cipher' para poder rotar el esquema después, y que la rutina de descifrado pase derecho los valores sin prefijo para soportar datos legacy. Nombrá las columnas con sufijo _encrypted, y dejá en claro solo lo que no es secreto (ej: public_key).

**Señal de que lo estás usando bien:** Si abrís la tabla en la DB, las columnas de tokens muestran strings 'v1:...' ilegibles en vez de los tokens reales, y rotar el secreto maestro invalida (no expone) los datos viejos.

---

#### TORNILLO #37 — Feature flags con default apagado para lo caro o riesgoso

**En una línea:** Funcionalidad costosa o en desarrollo se gobierna con flags ENABLE_*/FEATURE_* que vienen apagados por defecto y se chequean explícitamente contra el string 'true'.

**La regla para el próximo proyecto:** Gobierná toda funcionalidad cara, opcional o en desarrollo con un feature flag de env (ENABLE_* para procesos, FEATURE_* para capacidades de producto). Default SIEMPRE en false/off, así nada riesgoso se enciende por accidente al provisionar un ambiente nuevo. Chequeá el flag de forma estricta (=== 'true'), nunca como truthy. Documentá los flags en su propia sección del .env.example. Si una feature requiere una condición de infra (ej: una instancia siempre activa), validá esa combinación en el script de preflight del deploy.

**Señal de que lo estás usando bien:** Un ambiente recién provisionado sin configurar flags arranca en el modo más barato y seguro (jobs apagados, features beta ocultas), y prender una feature requiere un acto deliberado de setear la variable en 'true'.

---

### ÁREA 8 — Cuándo algo está realmente terminado (tests y QA)

#### TORNILLO #38 — Test de integración con DB real, sin mocks (inject + seed)

**En una línea:** Probar las rutas levantando el framework HTTP de verdad y pegando contra una base con datos de seed conocidos, en vez de mockear el ORM.

**La regla para el próximo proyecto:** Para endpoints, escribí tests de integración que levanten el framework HTTP real con los mismos plugins/middleware que producción y usen inyección en memoria (app.inject en Fastify, supertest en Express). Pegá contra una base de datos real de test con seed determinista, no contra mocks del ORM. Si las suites comparten esa base, desactivá el paralelismo entre archivos (fileParallelism:false) y cerrá conexiones en un teardown global. Para datos que deben existir, buscalos en vivo con un SELECT en vez de hardcodearlos, y generá datos mutables con un sufijo único (timestamp) para evitar colisiones entre corridas.

**Señal de que lo estás usando bien:** Tus tests fallan cuando rompés una query SQL real, una constraint, una validación de esquema o un cambio de status code, no solo cuando cambia un mock. Podés correr la suite dos veces seguidas sin limpiar la DB y sigue verde.

---

#### TORNILLO #39 — Probar el caso negativo y el contrato, no solo el happy path

**En una línea:** Para cada endpoint, testear explícitamente los rechazos de validación (400), la falta de auth (401), conflictos (409) y la forma exacta del cuerpo de la respuesta.

**La regla para el próximo proyecto:** Por cada endpoint cubrí, además del happy path: 401 sin credenciales, 400 por cada regla de validación relevante (campo faltante, tipo/formato inválido, valor fuera de enum, número negativo), y 409 en operaciones con unicidad. Afirmá la forma exacta del cuerpo de respuesta (propiedades esperadas, errors) y, en respuestas no-JSON como CSV, verificá tanto el content-type y el header esperado como la AUSENCIA de marcadores de error filtrado (ej. '"statusCode"'). Si los datos hacen ambiguo el resultado, aceptá el conjunto de status válidos con expect([...]).toContain en vez de elegir uno.

**Señal de que lo estás usando bien:** Mandar un payload basura a cualquier endpoint nunca termina en 500 ni en un 200 silencioso: siempre cae en un 400/401/409 cubierto por un test, y la respuesta tiene exactamente las claves esperadas.

---

#### TORNILLO #40 — RLS como red de seguridad final, testeada como atacante

**En una línea:** Verificar el aislamiento multi-tenant a nivel de base de datos (Row-Level Security) probando activamente los accesos cruzados que deben fallar.

**La regla para el próximo proyecto:** En cualquier sistema multi-tenant, no confíes solo en el WHERE org_id de la capa de aplicación: ponelo también a nivel de base (Row-Level Security en Postgres) y escribí una suite dedicada que actúe como atacante: parada en el tenant A, intentá insertar/actualizar/borrar datos del tenant B y exigí excepción (rejects.toThrow). Probá también el caso fail-closed: sin contexto seteado, las queries deben devolver 0 filas, no todo. Complementalo con un check de infra que verifique que los secretos vienen de un Secret Manager y que la base no tiene redes abiertas.

**Señal de que lo estás usando bien:** Si comentás el WHERE org_id en una query de aplicación, la suite RLS sigue en verde porque la base bloquea la fuga igual; y si olvidás setear el contexto, las queries devuelven vacío en vez de exponer datos de otros tenants.

---

#### TORNILLO #41 — QA por smoke tests contra entorno real, gateando el release

**En una línea:** Antes de liberar, correr scripts que pegan contra el entorno real validando matriz de permisos por rol, contratos de endpoints, push real y hardening de infra; cualquier fallo aborta con exit 1.

**La regla para el próximo proyecto:** Aparte de los tests automáticos, mantené un set de smoke tests ejecutables (scripts simples que peguen al entorno real con credenciales reales) y hacelos requisito de release. Modelá los permisos como una matriz declarativa endpoint x rol con el status esperado, acumulá TODOS los fallos antes de salir con process.exit(1), y validá contratos reales (incluyendo un envío push/email/pago real verificando su estado de entrega, ej. deliveryStatus === 'PUSH_SENT'). Sumá un preflight que valide variables de entorno con reglas concretas (secretos largos != valor de dev, URLs sin 'localhost'), migraciones, type-check y /health antes de deployar. Componé todo en un único comando (qa:release) y documentá en un RELEASE_CHECKLIST qué comandos son obligatorios.

**Señal de que lo estás usando bien:** Existe un único comando (ej. pnpm qa:release) que un agente o dev puede correr contra el entorno real y que, si algo de permisos, contrato, push o infra está roto, sale con exit 1 listando exactamente qué falló; ningún release se firma sin esa corrida en verde anotada en el checklist.

---

### ÁREA 9 — El contrato entre pantallas (código compartido entre backend, web y mobile)

#### TORNILLO #46 — Contrato de respuesta y de identidad uniforme (ApiResponse / JwtPayload / RequestContext)

**En una línea:** Define una sola forma de respuesta (ApiResponse<T>), un solo shape de token (JwtPayload) y un solo contexto de request (RequestContext) en shared, y el backend los respeta literalmente.

**La regla para el próximo proyecto:** Definí en shared un sobre de respuesta genérico ApiResponse<T> con campo de error estructurado {code,message}, más el shape del JWT y un RequestContext, y hacé que TODOS los endpoints (incluidos los caminos de error/401/403) serialicen con esa forma. El front tipea sus fetch contra ApiResponse<T> y maneja error.code en vez de adivinar formatos por endpoint.

**Señal de que lo estás usando bien:** Toda respuesta del backend (éxito o error) cabe en ApiResponse<T>; el front puede escribir un solo handler de errores leyendo res.error.code, y sub/org_id del token tienen un único lugar de definición.

---

### Integraciones externas y dinero (Mercado Pago multi-tenant, billing de plataforma)

#### TORNILLO #52 — Credenciales cifradas en reposo con clave derivada y cadena de fallback

**En una línea:** Los secretos de terceros (Access Token, webhook secret) nunca se guardan en texto plano: se cifran con AES-256-GCM y se versionan con prefijo para poder rotar el formato.

**La regla para el próximo proyecto:** Nunca guardes secretos de terceros (API keys, access tokens, webhook secrets) en texto plano en la DB. Centraliza encrypt/decrypt en UN modulo, usa cifrado autenticado (AES-256-GCM, no AES-CBC), IV aleatorio por valor, deriva la clave de un secreto de entorno dedicado (no reuses el de JWT salvo como fallback explicito), y prefija el ciphertext con una version de esquema ('v1:') para poder rotar el algoritmo sin romper datos existentes. En decrypt, trata los valores sin prefijo como legacy/plaintext.

**Señal de que lo estás usando bien:** Si abris la tabla en la DB ves cadenas tipo 'v1:...:...:...' y nunca el token real; si cambias PAYMENT_SETTINGS_SECRET, los valores viejos dejan de descifrarse (senal de que realmente dependian de la clave); y agregar un nuevo proveedor de secretos solo requiere importar las dos funciones.

---

#### TORNILLO #53 — Webhook idempotente que no confia en el cliente y re-verifica server-to-server

**En una línea:** Cada webhook de pago se desduplica con una fila UNIQUE, valida firma HMAC con comparacion de tiempo constante, y confirma el pago re-consultando la API del proveedor en lugar de creer lo que llega en el body o el redirect.

**La regla para el próximo proyecto:** Para cualquier webhook de pagos: (1) desduplica con una tabla cuyo UNIQUE sea (provider, event_key) y corta si ya existia; (2) valida la firma con HMAC y crypto.timingSafeEqual sobre buffers, nunca con '==='; (3) NUNCA marques un pago como pagado con los datos que vienen en el body o el query del redirect: re-consulta el recurso en la API del proveedor con tu propia credencial y deriva el estado de ahi; (4) haz la mutacion de estado dentro de una transaccion con FOR UPDATE sobre la fila objetivo. Responde 200 a duplicados y a no-matcheados para que el proveedor deje de reintentar.

**Señal de que lo estás usando bien:** Reenviar el mismo webhook dos veces no duplica abonos ni pagos; un body falsificado no activa nada porque la verdad viene del GET al proveedor; y un atacante no puede distinguir por tiempo si la firma estuvo cerca de ser valida.

---

#### TORNILLO #54 — Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero

**En una línea:** Cada operacion de pago resuelve y filtra por organization_id de punta a punta, y cada inquilino usa sus propias credenciales de Mercado Pago, de modo que el dinero liquida en la cuenta correcta y nada se mezcla entre tenants.

**La regla para el próximo proyecto:** En un SaaS multi-tenant que integra pagos, haz que el tenant viaje en todo el circuito: incluye el tenant_id en la notification_url del webhook (no solo en metadata, que puede no venir), codificalo tambien en el external_reference con un prefijo namespaced ('app:{id}') para reconciliar, carga las credenciales del proveedor por-tenant (cada cliente cobra en su cuenta), y agrega 'AND tenant_id = $X' a TODA query que toque dinero antes de mutar. Rechaza con 400/404 si el tenant no se puede resolver, en vez de aplicar el pago a ciegas.

**Señal de que lo estás usando bien:** Un webhook sin org_id se rechaza con 400; un pago de un inquilino jamas activa un abono en otro; y el dinero aparece en la cuenta Mercado Pago del inquilino correcto, no en una cuenta central.

---

#### TORNILLO #55 — Billing de la plataforma como gate de permisos con 402

**En una línea:** El estado de suscripcion del propio SaaS se hace cumplir en el mismo middleware de permisos: si el inquilino no pago, se degrada a solo-lectura y se responde HTTP 402 en las acciones de gestion.

**La regla para el próximo proyecto:** Para gatear features por estado de suscripcion del propio SaaS: modela el estado como un enum cerrado con CHECK en la DB; calcula la expiracion de trial/gracia de forma lazy dentro del mismo UPDATE que lee el estado (asi un trial vencido se degrada solo, sin depender de un cron); define los permisos permitidos por estado con Sets explicitos (read-only para 'restricted', minimo para 'suspended'/'cancelled'); enchufa el chequeo en el MISMO middleware que ya valida permisos RBAC; y responde 402 Payment Required (no 403) para que el front pueda distinguir 'falta de permiso' de 'falta de pago' y mostrar un CTA de upgrade. Excluye explicitamente a los usuarios finales (clientes del cliente) del bloqueo B2B.

**Señal de que lo estás usando bien:** Un inquilino con trial vencido puede seguir viendo sus datos pero no editar, y la API le devuelve 402 con el billingStatus en vez de 403; los alumnos del inquilino nunca se ven afectados; y agregar un estado nuevo te obliga a tocar el enum, el CHECK y los Sets, sin logica de fechas dispersa.

---

### ÁREA TRANSVERSAL — Patrones no cubiertos

#### TORNILLO #57 — Paginación con clamp defensivo y envelope { data, total, page, totalPages }

**En una línea:** Toda lista paginada lee page/limit del query, los acota con Math.min/Math.max para que el cliente no rompa la DB, y devuelve siempre el mismo sobre con total y totalPages.

**La regla para el próximo proyecto:** Para cualquier endpoint que devuelva una lista, parseá page/limit del query y acotalos SIEMPRE: limitNum = Math.min(TOPE, Math.max(1, parseInt(limit ?? DEFAULT))). Corré un COUNT(*) con el mismo WHERE que la query de datos, y devolvé un envelope uniforme { data, total, page, totalPages }. Nunca confíes en el limit del cliente sin techo.

**Señal de que lo estás usando bien:** Pedir ?limit=999999 devuelve como máximo el tope (ej 100) y ?page=0 o negativo cae en page 1; el body siempre trae total y totalPages aunque la lista venga vacía; un test de integración puede afirmar la forma del envelope sin conocer los datos.

---

#### TORNILLO #59 — CSV listo para Excel: BOM, CRLF, escapeo RFC y filename con fecha

**En una línea:** Las exportaciones serializan a CSV con un helper que escapa según RFC-4180, unen con CRLF, prependen un BOM UTF-8 y mandan Content-Disposition con nombre fechado, para que Excel abra acentos y comas sin romperse.

**La regla para el próximo proyecto:** Cuando exportes CSV pensado para abrirse en Excel: escapá cada celda con reglas RFC-4180 (comillas dobles + duplicar comillas internas si hay coma/comilla/newline), uní filas con '\r\n', prependé el BOM '﻿' al string final, y respondé con Content-Type text/csv; charset=utf-8 y Content-Disposition attachment con filename que lleve la fecha. Generá las cabeceras humanas directamente como alias de la query. Cubrílo con un test de contrato que afirme el header de columnas.

**Señal de que lo estás usando bien:** Abrís el archivo en Excel en Windows y los acentos (á, é, ñ) se ven bien, las columnas no se desarman cuando una celda tiene comas, y el archivo descargado ya viene con nombre fechado tipo socios-2026-06-24.csv.

---

#### TORNILLO #60 — Jobs en background: gate por header-secret, run-log en DB y dedupe idempotente por índice único parcial

**En una línea:** Los jobs programados (recordatorios, vencimientos, pagos vencidos, billing) se disparan por un endpoint sin auth de usuario pero protegido por un secret en header, registran cada corrida en una tabla de run-log, y evitan duplicados con una dedupeKey persistida más un índice único parcial que hace el INSERT idempotente.

**La regla para el próximo proyecto:** Para jobs disparados por cron externo: protegé el endpoint con un secret comparado contra un header (no con la auth de usuario) y respondé 401 si no coincide o si el env no está seteado. Registrá cada corrida en una tabla de run-log con status RUNNING -> COMPLETED/FAILED y un resumen JSONB dentro de try/catch. Hacé cada efecto idempotente: armá una dedupeKey determinística que incluya la fecha del día y un INSERT ... ON CONFLICT DO NOTHING respaldado por un índice único (parcial si solo aplica a filas de job). Distinguí 'creado' de 'omitido' mirando rowCount.

**Señal de que lo estás usando bien:** Correr el job dos veces el mismo día no genera notificaciones duplicadas (el segundo run reporta todo como skipped); llamar el endpoint sin el header secreto da 401; y siempre podés mirar la tabla de run-log para ver cuándo corrió, si falló y cuántos registros tocó.

---

## Mapa de dependencias

Solo se muestran las dependencias **entre tornillos universales**. Si un universal se apoyaba en uno que era Solo-TS o Solo-proyecto, esa flecha se omite acá (buscala en el manual completo).

- **#5 — JWT flaco para identidad, DB gorda para permisos** se apoya en **#8** (Catálogo de permisos como array de strings, fuente única compartida).
- **#8 — Catálogo de permisos como array de strings, fuente única compartida** se apoya en **#5** (JWT flaco para identidad, DB gorda para permisos).
- **#9 — El org_id sale SIEMPRE del JWT, nunca del cliente** se apoya en **#11** (Fail-closed: sin contexto de tenant, cero datos), **#10** (Cada query carga su propio WHERE organization_id (filtro app-layer omnipresente)).
- **#10 — Cada query carga su propio WHERE organization_id (filtro app-layer omnipresente)** se apoya en **#9** (El org_id sale SIEMPRE del JWT, nunca del cliente).
- **#11 — Fail-closed: sin contexto de tenant, cero datos** se apoya en **#9** (El org_id sale SIEMPRE del JWT, nunca del cliente).
- **#13 — La transacción como guardián de la regla de negocio** se apoya en **#29** (Fail-closed multi-tenant: nunca una query sin org_id + soft delete).
- **#14 — organization_id en cada query, sin excepción** se apoya en **#13** (La transacción como guardián de la regla de negocio).
- **#18 — La plantilla de tabla multi-tenant (el molde que se copia y pega)** se apoya en **#19** (Índices deliberados sobre FK y deleted_at (no dejar que Postgres adivine)), **#20** (onDelete con intención: Cascade para hijos, SetNull para histórico).
- **#19 — Índices deliberados sobre FK y deleted_at (no dejar que Postgres adivine)** se apoya en **#18** (La plantilla de tabla multi-tenant (el molde que se copia y pega)).
- **#20 — onDelete con intención: Cascade para hijos, SetNull para histórico** se apoya en **#18** (La plantilla de tabla multi-tenant (el molde que se copia y pega)).
- **#26 — Webhook de pago a prueba de duplicados (idempotencia + firma + verdad del proveedor)** se apoya en **#23** (Errores de negocio se atajan, errores del sistema explotan).
- **#28 — Factory de rutas: setupXRoutes(app)** se apoya en **#32** (Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission).
- **#29 — Fail-closed multi-tenant: nunca una query sin org_id + soft delete** se apoya en **#28** (Factory de rutas: setupXRoutes(app)), **#32** (Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission).
- **#32 — Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission** se apoya en **#28** (Factory de rutas: setupXRoutes(app)), **#29** (Fail-closed multi-tenant: nunca una query sin org_id + soft delete).
- **#33 — El .env.example como contrato narrado, no como lista muerta** se apoya en **#34** (Secrets multiempresa cifrados con AES-256-GCM, nunca en texto plano).
- **#34 — Secrets multiempresa cifrados con AES-256-GCM, nunca en texto plano** se apoya en **#33** (El .env.example como contrato narrado, no como lista muerta).
- **#37 — Feature flags con default apagado para lo caro o riesgoso** se apoya en **#33** (El .env.example como contrato narrado, no como lista muerta).
- **#38 — Test de integración con DB real, sin mocks (inject + seed)** se apoya en **#39** (Probar el caso negativo y el contrato, no solo el happy path), **#41** (QA por smoke tests contra entorno real, gateando el release).
- **#39 — Probar el caso negativo y el contrato, no solo el happy path** se apoya en **#38** (Test de integración con DB real, sin mocks (inject + seed)).
- **#40 — RLS como red de seguridad final, testeada como atacante** se apoya en **#38** (Test de integración con DB real, sin mocks (inject + seed)), **#41** (QA por smoke tests contra entorno real, gateando el release).
- **#41 — QA por smoke tests contra entorno real, gateando el release** se apoya en **#39** (Probar el caso negativo y el contrato, no solo el happy path), **#40** (RLS como red de seguridad final, testeada como atacante).
- **#52 — Credenciales cifradas en reposo con clave derivada y cadena de fallback** se apoya en **#54** (Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero).
- **#53 — Webhook idempotente que no confia en el cliente y re-verifica server-to-server** se apoya en **#52** (Credenciales cifradas en reposo con clave derivada y cadena de fallback), **#54** (Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero).
- **#54 — Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero** se apoya en **#52** (Credenciales cifradas en reposo con clave derivada y cadena de fallback), **#53** (Webhook idempotente que no confia en el cliente y re-verifica server-to-server).
- **#55 — Billing de la plataforma como gate de permisos con 402** se apoya en **#54** (Aislamiento multi-tenant: la organizacion es ciudadano de primera clase en el dinero).
- **#57 — Paginación con clamp defensivo y envelope { data, total, page, totalPages }** se apoya en **#14** (organization_id en cada query, sin excepción).
- **#59 — CSV listo para Excel: BOM, CRLF, escapeo RFC y filename con fecha** se apoya en **#14** (organization_id en cada query, sin excepción), **#32** (Autorización declarativa: requirePermission en el guard, y el front filtra con hasPermission).
- **#60 — Jobs en background: gate por header-secret, run-log en DB y dedupe idempotente por índice único parcial** se apoya en **#53** (Webhook idempotente que no confia en el cliente y re-verifica server-to-server), **#9** (El org_id sale SIEMPRE del JWT, nunca del cliente), **#37** (Feature flags con default apagado para lo caro o riesgoso).

### La cadena de seguridad multi-tenant (sin atarse a Postgres)

La misma cadena del manual completo, pero **sin el eslabón de RLS** (esa era una red específica de Postgres y quedó como "Solo este proyecto"). Estos cuatro eslabones se implementan en cualquier base y cualquier lenguaje:

```
  #9  El tenant sale del token firmado, NUNCA del cliente
         |
         v
  #11  Fail-closed: si no hay tenant en el token -> rechazá, no devuelvas nada
         |
         v
  #10  Cada query lleva su filtro de tenant (WHERE tenant_id = <del token>)
         |
         v
  #40  Un test que actúa como atacante intenta cruzar tenants y exige el fallo
```

> Si tu base soporta filtrado a nivel de fila (RLS en Postgres, VPD en Oracle, políticas equivalentes), sumalo como quinta capa de defensa. Pero **nunca** lo uses como reemplazo del filtro en la query: es red, no piso.

---

## Cómo usar este manual en cualquier stack

No importa si el proyecto nuevo es Python, Node, PHP, Go, Ruby o Java. Estos tornillos son principios, no APIs. Antes de escribir una línea, hacé esto:

1. **Leélos como principios y traducí el "cómo" a tu stack.** Donde el original dice Zod, leé "validación en la frontera" (pydantic, Joi, class-validator, Laravel Requests, Bean Validation). Donde dice Prisma, leé "capa de acceso a datos" (SQLAlchemy, Eloquent, GORM, JPA). Donde dice Fastify, leé "tu framework HTTP". El tornillo te dice *qué* garantizar; vos elegís *con qué*.

2. **¿Es multi-tenant? La cadena #9 → #11 → #10 → #40 va primero, sí o sí.** El identificador de cliente/cuenta sale del token verificado, jamás de un parámetro que el usuario controle; si falta, cortás (fail-closed); cada consulta de datos lleva su filtro de tenant; y lo cerrás con un test que se hace pasar por atacante de otro tenant y DEBE fallar. Esto es idéntico en SQL, NoSQL o un microservicio: cambia la sintaxis, no el principio.

3. **Poné los cimientos antes que las features:** un molde de tabla/colección con id estable, marcas de tiempo, borrado lógico e índices pensados (#18, #19, #20); permisos como datos, no como `if` desparramados (#8); un `.env.example` que documenta cada variable y secrets cifrados en reposo (#33, #34); y un formato de respuesta/error uniforme desde el día uno (#24). Tornillos: #18, #19, #20, #8, #33, #34, #24.

4. **Para cada endpoint nuevo, repetí siempre el mismo molde:** autenticar → autorizar por permiso (#32) → validar la entrada → envolver la regla de negocio que toca varios registros en una transacción (#13) → responder con el envelope uniforme (#24). Si recibís webhooks o callbacks externos, hacelos idempotentes y no confíes en el payload del que llama (#26). Tornillos: #32, #13, #24, #26.

5. **Definí "terminado" igual en todos lados:** no alcanza el happy path. Probá el caso negativo y el contrato (#39), el aislamiento entre tenants como atacante (#40), y corré un smoke test contra un entorno real con la matriz de permisos por rol (#41). Si algo termina con `exit != 0`, no está terminado. Tornillos: #39, #40, #41.

### Regla de oro (vale en cualquier lenguaje)

> El identificador de tenant sale del token firmado y viaja en el filtro de toda consulta; la validación y el contrato viven en un solo lugar; y nada se da por "terminado" hasta que un test que actúa como atacante intenta cruzar tenants y **falla**. Si dudás entre filtrar de más o devolver de más, **negá**: una fuga de datos no se perdona, un 404 de más sí.
