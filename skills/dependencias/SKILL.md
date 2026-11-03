---
name: dependencias
description: Usar cuando el usuario quiera procesar los PRs de Dependabot y las alertas de seguridad de sus repos Rails (cashflow, coshinotes, enlacito, supermenu, postlane-backend). Recorre cada repo, mezcla parches y versiones menores vigilando el despliegue, valida a fondo las versiones mayores y deja registro. Dispara con "dependencias", "dependabot", "actualiza dependencias", "alertas de seguridad".
---

# Dependencias

Recorrido semanal por los PRs de Dependabot y las alertas de seguridad. Esta skill está pensada para correr de forma mayormente autónoma: decide y mezcla tú lo que las reglas permiten, y deja para el final solo las decisiones que de verdad le tocan al usuario.

## Repos

Todos en `~/projects/devaspros-projects/`:

- `cashflow`
- `coshinotes`
- `enlacito`
- `supermenu` (tiene frontend con Vite, React e Inertia)
- `postlane-backend`

Si el usuario pide solo uno o algunos, procesa solo esos.

Datos comunes que ya conoces de estos repos:

- Rails 8, Ruby en `.ruby-version`, pruebas con RSpec.
- `.github/workflows/workflow.yml` corre las pruebas en cada PR y en cada push a `main`. Si pasan en `main`, llama a `deploy.yml`, que despliega por SSH. O sea: **mezclar a `main` es desplegar a producción**.
- Los merges se hacen con merge commit (`gh pr merge --merge`), no squash.
- Dependabot está configurado solo para `bundler`, con `open-pull-requests-limit: 2`. Por eso puede haber alertas de seguridad sin PR.

Antes de asumir cualquiera de estos datos, confírmalo leyendo los archivos del repo. Si algo no coincide, el repo manda.

## Sin visibilidad en producción

No tienes acceso a producción: ni al servidor, ni a sus logs, ni a la base de datos, ni a herramientas de monitoreo. Lo único que ves es lo que está en GitHub (PRs, alertas, runs de Actions y sus logs) y lo que corres en local.

Si un paso necesita información de producción (ej: confirmar que la app responde después de un despliegue, revisar un error en los logs del servidor, ver qué versión está corriendo), no lo adivines ni lo des por bueno:

1. Pausa ese paso y anota en el registro qué necesitas ver y por qué.
2. Agrégalo a las decisiones pendientes como "verificación en producción", con el comando o la ruta exacta que el usuario debería revisar.
3. Sigue con el siguiente paso o el siguiente repo. Si lo pendiente bloquea más merges en ese repo (ej: no sabes si el último despliegue dejó la app sana), no mezcles nada más en ese repo.

Si el usuario está presente y la respuesta es rápida, puedes preguntarle en el momento con `AskUserQuestion` en vez de dejarlo para el final.

## Límite de merges

Máximo **5 merges por repo en cada ejecución**. Cuentan todos: parches, menores, mayores y los PRs que abras tú para alertas sin PR. El commit del registro no cuenta.

Como el orden de proceso pone primero las alertas por severidad, el límite se gasta en lo más importante. Al llegar a 5, no mezcles más en ese repo: registra lo que quedó como `pendiente, límite alcanzado` y sigue con el siguiente repo. Lo que quede se procesa en la ejecución de la semana siguiente.

Lo que no implica merge (recolectar, clasificar, validar en local una mayor para dejarla como decisión pendiente) sí puedes hacerlo aunque ya hayas llegado al límite, si el tiempo lo permite.

## Reglas de decisión

Compara la versión actual con la nueva (`x.y.z`):

| Cambio | Ejemplo | Acción |
|---|---|---|
| Parche (`z`) | 2.9.0 a 2.9.6 | Mezclar ya, vigilar el despliegue |
| Menor (`y`) | 6.4.0 a 6.5.1 | Mezclar ya, vigilar el despliegue |
| Mayor (`x`) | 5.0.4 a 6.0.0 | Validación completa en local antes de mezclar |

Notas:

- En gemas `0.y.z` el cambio de `y` se trata como **mayor** (en semver, 0.x no promete compatibilidad). Solo el cambio de `z` es parche.
- Si el PR agrupa varias gemas, manda la más riesgosa del grupo.
- Si el PR no pasa CI, no se mezcla aunque sea parche. Trátalo como mayor: investiga en local.
- Una alerta de seguridad no cambia el tipo de cambio, pero sí la prioridad: se procesa primero.

### Lo que nunca se sube de versión mayor

- Ruby
- Rails y sus componentes (`rails`, `railties`, `actionpack`, `activerecord`, `activesupport`, `actionview`, `actioncable`, `actionmailer`, `actionmailbox`, `actiontext`, `activejob`, `activemodel`, `activestorage`)
- `devise`
- `inertia_rails`, `@inertiajs/*`

Parches y menores de estas sí se mezclan con las reglas normales (ej: `rails` 8.0.5.1 a 8.0.5.2, o `devise` 5.0.3 a 5.0.4).

Si llega un PR de versión mayor de alguna de estas: no lo mezcles, no lo cierres, no comentes en él. Regístralo como "omitido, protegida" y menciónalo en el resumen final. Si hay una alerta de seguridad cuyo único arreglo es una versión mayor de una protegida, eso es una decisión del usuario: agrégalo a las decisiones pendientes con severidad y advisory.

## Flujo

### 0. Preparación (una sola vez)

1. `gh auth status`. Si no hay sesión, para y pídele al usuario que corra `! gh auth login`.
2. Fecha del recorrido: `date +%F`. La usas para el nombre del registro.

### 1. Por cada repo

Procesa los repos uno por uno, nunca en paralelo. Dentro de un repo, los PRs también van uno por uno: cada merge cambia `Gemfile.lock` y Dependabot tendrá que rebasar los demás.

#### 1.1 Estado local

```bash
cd ~/projects/devaspros-projects/<repo>
git status --porcelain
git branch --show-current
```

Si hay cambios sin commitear (fuera de `docs/security-updates/`) o la rama no es `main`, **no toques ese repo**. Regístralo como "omitido, árbol sucio" y sigue con el siguiente. No hagas stash ni checkout por encima del trabajo del usuario.

Si está limpio: `git pull --ff-only`.

#### 1.2 Recolectar

```bash
gh pr list --author "app/dependabot" --state open \
  --json number,title,headRefName,mergeable,body,url

gh api "repos/{owner}/{repo}/dependabot/alerts?state=open&per_page=100" \
  --jq '.[] | {number, package: .dependency.package.name, ecosystem: .dependency.package.ecosystem, severity: .security_advisory.severity, summary: .security_advisory.summary, ghsa: .security_advisory.ghsa_id, vulnerable: .security_vulnerability.vulnerable_version_range, patched: .security_vulnerability.first_patched_version.identifier}'
```

`{owner}/{repo}` sale de `gh repo view --json nameWithOwner -q .nameWithOwner`.

Cruza alertas con PRs por nombre de paquete. Para cada PR obtén gema, versión origen y destino del título (`Bump <gema> from <a> to <b>`) y clasifícalo según las reglas.

Orden de proceso:

1. PRs que resuelven alertas, por severidad (critical, high, medium, low)
2. Parches
3. Menores
4. Mayores

Si no hay PRs ni alertas, registra "sin novedades" y pasa al siguiente repo.

#### 1.3 Parche o menor

1. Revisa CI del PR: `gh pr checks <n> --watch --fail-fast`.
   - Si falla, pasa el PR al flujo de mayor (1.4) para investigar.
2. Revisa `mergeable`. Si hay conflicto, comenta `@dependabot rebase` en el PR, sigue con otro y vuelve a él al final del repo. Si sigue en conflicto después de eso, regístralo como pendiente.
3. Mezcla: `gh pr merge <n> --merge`.
4. Vigila el despliegue (ver 1.6). **No mezcles el siguiente PR hasta que el despliegue termine bien.**

#### 1.4 Mayor

1. Lee el cuerpo del PR (release notes, changelog, commits) y, si hace falta, el CHANGELOG o la guía de migración de la gema. Busca cambios incompatibles.
2. Busca dónde usa el proyecto la gema: `grep -rn` en `app/`, `config/`, `lib/`, `spec/`, initializers.
3. Baja la rama: `gh pr checkout <n>`.
4. Instala: `bundle install`. En `supermenu`, si el cambio es de npm, también `npm ci`.
5. Valida:
   - `bin/rails db:prepare RAILS_ENV=test`
   - `bundle exec rspec`
   - `bundle exec rubocop` si el repo lo usa (compara contra `main` para no culpar a la gema de ofensas viejas)
   - `bin/brakeman` si existe
   - En `supermenu`: `NODE_ENV=production bundle exec vite build`
   - Arranca el servidor (`bin/rails server -p <puerto libre>`), pide `/up` y una o dos rutas principales con `curl`, revisa `log/development.log` por errores y deprecaciones nuevas relacionadas con la gema. **Tumba el servidor al terminar.**
6. Decide:
   - **Todo pasa y no hay cambios de código necesarios**: vuelve a `main`, mezcla con `gh pr merge <n> --merge` y vigila el despliegue (1.6).
   - **Hace falta un ajuste pequeño** (uno o dos archivos, ej: renombrar una opción de configuración o una llamada deprecada, con la guía de migración diciendo exactamente qué hacer): haz el ajuste en la rama del PR, corre de nuevo las validaciones, haz commit y push a esa rama, espera CI verde, mezcla y vigila. Anota el ajuste en el registro.
   - **Hay muchos cambios, cambios de comportamiento que no puedes verificar con las pruebas, o dudas**: no mezcles. Agrégalo a las decisiones pendientes con un resumen de qué cambia, qué rompe, qué archivos tocaría y tu recomendación.
7. Siempre al terminar: `git checkout main` y `bundle install` para dejar el repo como estaba.

#### 1.5 Alertas sin PR

Pasa cuando Dependabot llegó al límite de PRs abiertos o no pudo generar el PR.

- Si el arreglo es parche o menor y la gema no está bloqueada por otra: crea una rama `deps/<gema>-<version>`, corre `bundle update --conservative <gema>`, verifica que `Gemfile.lock` quedó en una versión parcheada, corre `bundle exec rspec`, abre PR con `gh pr create` mencionando la alerta (GHSA), espera CI, mezcla y vigila igual que 1.3.
- Si `bundle update --conservative` no alcanza la versión parcheada (por restricciones del `Gemfile` u otras gemas), o el arreglo es mayor: agrégalo a decisiones pendientes con la explicación.

#### 1.6 Vigilar el despliegue

Después de cada merge:

```bash
sha=$(gh pr view <n> --json mergeCommit -q .mergeCommit.oid)
gh run list --commit "$sha" --json databaseId,name,status,conclusion
gh run watch <databaseId> --exit-status
```

El run puede tardar unos segundos en aparecer; reintenta hasta que aparezca. El run de `workflow.yml` incluye pruebas y el job `Deploy`; el despliegue está bien solo si **ambos** terminan en `success`.

Si falla (pruebas en `main` o despliegue):

1. **Detén el repo**. No mezcles nada más en él.
2. Saca el log del paso que falló: `gh run view <id> --log-failed`. Es lo único que puedes ver; el estado real del servidor no lo ves (ver "Sin visibilidad en producción"). Si el log no basta para saber si la app quedó caída o a medio desplegar, dilo así y pídeselo al usuario.
3. Regístralo en el registro con el error.
4. No reviertas por tu cuenta. Agrega a decisiones pendientes, marcado como urgente, con el diagnóstico y la propuesta (normalmente `git revert` del merge commit). Sigue con el siguiente repo.

#### 1.7 Registro

Mantén un archivo por recorrido en cada repo: `docs/security-updates/<YYYY-MM-DD>.md`. Créalo al empezar el repo y ve agregando filas a medida que procesas cada PR o alerta, no al final. Si el recorrido se interrumpe, el registro debe reflejar lo que ya se hizo.

Formato:

```markdown
# Actualizaciones de dependencias, 2026-11-03

| PR | Paquete | De | A | Tipo | Alerta | Acción | CI y despliegue |
|---|---|---|---|---|---|---|---|
| #78 | devise | 5.0.3 | 5.0.4 | parche | | mezclado | ok (run 41650823522) |
| #80 | rack | 3.1.8 | 3.1.12 | parche | GHSA-xxxx (high) | mezclado | ok (run ...) |
| #81 | pagy | 8.6.3 | 9.0.0 | mayor | | validado en local y mezclado | ok (run ...) |
| #82 | rails | 8.0.5.1 | 9.0.0 | mayor | | omitido, protegida | |

## Notas

- #81 pagy: se renombró `Pagy::DEFAULT[:items]` a `[:limit]` en `config/initializers/pagy.rb`, según la guía de migración de la 9.0.

## Pendientes para el usuario

- ...
```

Valores para "Acción": `mezclado`, `validado en local y mezclado`, `mezclado con ajustes`, `omitido, protegida`, `omitido, árbol sucio`, `pendiente de decisión`, `pendiente, conflicto`, `pendiente, CI falla`, `pendiente, límite alcanzado`, `pendiente, verificación en producción`, `sin novedades`.

Al terminar el repo, deja el registro commiteado en `main` así:

```bash
git checkout main
git pull --ff-only
git add docs/security-updates/<fecha>.md
git commit -m "Registro de actualizaciones de dependencias <fecha> [skip ci]"
git push
```

El `[skip ci]` evita un despliegue solo por el registro. Si el push a `main` es rechazado (protección de rama u otro motivo), deja el archivo sin commitear y dilo en el resumen.

### 2. Resumen final

Al terminar todos los repos, muestra al usuario:

1. Una tabla por repo con lo mezclado, lo omitido y lo pendiente (puede ser la misma del registro, resumida).
2. **Decisiones pendientes**, cada una con: repo, PR o alerta, qué cambia, riesgo, archivos afectados y tu recomendación. Las urgentes (despliegues fallidos) van primero.
3. Repos que se saltaron y por qué.

Si hay decisiones pendientes, hazlas con `AskUserQuestion` (una pregunta por decisión, máximo cuatro por llamada) y ejecuta lo que el usuario elija, actualizando el registro.

## Reglas generales

- Nunca hagas push directo a `main` salvo el commit del registro descrito arriba.
- Nunca uses `--admin` ni saltes checks para mezclar.
- Nunca cierres PRs ni ignores versiones en Dependabot (`@dependabot ignore`) sin que el usuario lo pida.
- Un merge a la vez por repo. Espera el despliegue antes del siguiente.
- Máximo 5 merges por repo en cada ejecución.
- Nada de producción se da por supuesto: lo que no puedas ver, lo pides o lo dejas pendiente.
- Si levantas servidores, tumbados al terminar para no dejar puertos ocupados.
- Deja cada repo en `main`, limpio y con `bundle install` al día.

## Textos

Procura no usar em-dash cuando escribas textos. Usa separación normal con comas, punto o punto y coma.

Procura no usar el middle-dot para separar elementos de una lista.
