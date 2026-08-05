# Blog Materiales de Construcción — repositorio de demostración

Sitio estático (HTML + CSS puros, sin frameworks ni build) usado como ejemplo en
una capacitación interna sobre **GitHub como servicio**. Todo el contenido es
ficticio.

## Qué se demuestra aquí

| Tema                  | Dónde verlo                                             |
| --------------------- | ------------------------------------------------------- |
| Repositorio público   | Este repo                                               |
| Ramas                 | `post/tipos-de-vigas`, `post/aceros-estructurales`      |
| Ciclo de Pull Request | PRs #1 (mergeado) y #2 (abierto)                        |
| Actions — lint        | `.github/workflows/lint.yml` (corre en `pull_request`)  |
| Actions — deploy      | `.github/workflows/deploy-pages.yml` (corre en `main`)  |
| GitHub Pages          | Publicado desde Actions                                 |
| CODEOWNERS            | `.github/CODEOWNERS`                                    |
| Plantilla de PR       | `.github/pull_request_template.md`                      |
| Environments          | `github-pages`, usado por el workflow de deploy         |
| Ruleset               | Protección de `main`: PR obligatorio + check `Lint`     |
| Releases              | Tag `v1.0.0`                                            |
| Secrets               | `SITE_DEPLOY_TOKEN`, exigido por el workflow de deploy  |
| Dependabot            | `.github/dependabot.yml` + alertas sobre `package.json` |

## Estructura

```
index.html                        portada con el listado de posts
posts/                            un archivo HTML por publicación
assets/styles.css                 estilos del sitio
.prettierrc                       configuración del formateador
CLAUDE.md                         instrucciones para Claude Code (modo tutor)
package.json                      scripts y dependencias de desarrollo
.github/CODEOWNERS                dueños de código
.github/dependabot.yml            configuración de Dependabot
.github/pull_request_template.md  plantilla de PR
.github/workflows/                lint y deploy
```

## Secrets

El workflow de deploy exige el secret **`SITE_DEPLOY_TOKEN`**. Es puramente
demostrativo: el workflow verifica que exista y **no hace nada con su valor**,
pero si falta, el job falla en el primer step y no se publica nada.

Para configurarlo:

```bash
gh secret set SITE_DEPLOY_TOKEN --body "valor-de-demostracion"
```

O desde la interfaz: **Settings → Secrets and variables → Actions → New
repository secret**.

Para demostrar la falla en vivo, bórralo y vuelve a lanzar el deploy:

```bash
gh secret delete SITE_DEPLOY_TOKEN
gh workflow run deploy-pages.yml
gh run watch
```

Nota didáctica: el step imprime el largo del secret y luego intenta imprimir su
valor, para mostrar que GitHub lo enmascara automáticamente en los logs como
`***`. Aun así, **nunca** hagas esto con un secret real: existen formas de
filtrar valores enmascarados, y los logs de un repo público son visibles para
cualquiera.

## Dependabot

`.github/dependabot.yml` configura dos ecosistemas: las dependencias npm de
`package.json` y las versiones de las actions usadas en los workflows.

⚠️ **Las `devDependencies` de `package.json` están desactualizadas a
propósito.** Son versiones antiguas con advisories públicos ya corregidos hace
años, puestas ahí para que Dependabot levante alertas de seguridad y abra Pull
Requests durante la capacitación:

| Paquete      | Versión fijada | Problema conocido                    |
| ------------ | -------------- | ------------------------------------ |
| `lodash`     | 4.17.15        | Prototype pollution (CVE-2020-8203)  |
| `minimist`   | 1.2.0          | Prototype pollution (CVE-2020-7598)  |
| `handlebars` | 4.0.11         | Varias, incluida ejecución de código |

**Por qué esto no representa un riesgo real acá:** este repo no tiene proceso de
build ni código JavaScript propio. Ninguno de esos paquetes se instala en CI —
los workflows usan `npx --yes prettier@3`, que descarga Prettier al vuelo—, y
ninguno se importa desde el sitio. GitHub Pages sirve HTML y CSS estáticos, sin
runtime de Node. Las tres alertas son sobre código que nunca se descarga ni se
ejecuta: existen solo como texto en `package.json`.

Dónde verlas: **Security → Dependabot alerts**, y los PRs que abra en la pestaña
**Pull requests**.

Cuando termine la capacitación, para dejar el repo limpio:

```bash
npx --yes npm-check-updates -u   # o edita package.json a mano
```

## Desarrollo local

No hay dependencias ni pasos de build. Abre `index.html` en el navegador, o
levanta un servidor estático:

```bash
python3 -m http.server 8000
```

## Formato

El CI valida el formato con Prettier. Antes de commitear:

```bash
npx --yes prettier@3 --write .
```

## Cómo agregar un post nuevo

Este es el recorrido completo, y es el mismo ejercicio que se hace en la
capacitación. Cada paso toca una pieza distinta de GitHub.

> **Durante la capacitación: haz los pasos 1 al 8. Detente ahí.**
>
> Los pasos **9 (mergear)** y **10 (verificar el despliegue)** los ejecuta en
> vivo quien dicta la charla, sobre un PR de ejemplo y proyectando la pantalla.
> No es una restricción técnica —con permiso de escritura sí podrías mergear—
> sino una de coordinación: `main` es una sola, y si varias personas mergean a
> la vez los despliegues de Pages se encolan y se cancelan entre ellos por el
> `concurrency: pages` del workflow. Nadie llega a ver el resultado.
>
> Tu PR queda abierto y en verde al terminar el paso 8: eso **es** el ejercicio
> completo. Los dos últimos pasos se muestran, no se practican.

### 1. Crea una rama desde `main`

`main` está protegida por un ruleset: no acepta pushes directos. Todo cambio
entra por Pull Request.

```bash
git checkout main
git pull
git checkout -b post/mi-articulo
```

La convención de nombre es `post/` + el slug del artículo en kebab-case.

### 2. Crea el archivo del post

Cada post es un HTML independiente en `posts/`. La forma más rápida y segura es
copiar uno existente y reemplazar el contenido:

```bash
cp posts/tipos-de-vigas.html posts/mi-articulo.html
```

Al editarlo, ajusta:

- `<title>` — título del post, seguido de `— Materiales & Estructuras`.
- `<meta name="description">` — una frase que resuma el artículo.
- El `<h1>` y el `<span class="post-meta">` con la fecha de publicación.
- El cuerpo dentro de `<article>`.

Detalles que importan:

- La ruta al CSS es `../assets/styles.css` (el post vive un nivel más abajo que
  la portada).
- Usa `&amp;` en lugar de `&` suelto, y `&larr;` para la flecha de "Volver".
- Para el bloque destacado de cada sección, usa la clase ya existente:

  ```html
  <p class="beam-use">
    <strong>¿Cuándo se usa?</strong>
    Texto del caso de uso.
  </p>
  ```

- No agregues CSS nuevo si puedes reutilizar las clases de `assets/styles.css`
  (`post-item`, `post-meta`, `beam-use`, `back-link`, `wrap`).

### 3. Enlázalo desde la portada

Un post que no está en `index.html` no se descubre. Agrega el `<li>` **al
principio** de la lista `.post-list`, para que el más reciente quede arriba:

```html
<li class="post-item">
  <h2>
    <a href="posts/mi-articulo.html">Título del post</a>
  </h2>
  <p>Resumen de una o dos frases.</p>
  <span class="post-meta">5 de agosto de 2026</span>
</li>
```

### 4. Formatea antes de commitear

El check **Lint** del CI corre `prettier --check` y falla si el formato no
coincide. Adelántate:

```bash
npx --yes prettier@3 --write .
npx --yes prettier@3 --check "**/*.{html,css,js,md}"
```

Prettier va a reindentar y a cortar líneas según el `printWidth: 80` de
`.prettierrc`. Es esperado: commitea el resultado formateado.

### 5. Revisa en el navegador

```bash
python3 -m http.server 8000
```

Abre <http://localhost:8000> y confirma que el post aparece en el listado, que
el enlace funciona y que los estilos cargan.

### 6. Commitea y sube la rama

Los mensajes de commit siguen
[Conventional Commits](https://www.conventionalcommits.org/):

```bash
git add .
git commit -m "feat: agrega post sobre mi tema"
git push -u origin post/mi-articulo
```

### 7. Abre el Pull Request

```bash
gh pr create --base main
```

Se carga sola la plantilla de `.github/pull_request_template.md`: completa **Qué
cambia**, **Por qué** y **Cómo probarlo**. Por CODEOWNERS, el dueño del repo
queda asignado como reviewer automáticamente.

### 8. Espera que el check `Lint` quede en verde

```bash
gh pr checks --watch
```

Si queda en rojo, casi siempre es formato: corre
`npx --yes prettier@3 --write .`, commitea y pushea de nuevo. El check se vuelve
a ejecutar solo con cada push.

**Si estás en la capacitación, este es tu último paso.** Deja el PR abierto y en
verde, y avisa. Los dos que siguen se muestran en pantalla.

---

### 9. Mergea — 🖥️ demostración en vivo

> Lo ejecuta quien dicta la capacitación. Míralo, no lo corras.

El botón de merge sigue bloqueado hasta que `Lint` esté verde — así lo exige el
ruleset de `main`.

```bash
gh pr merge --squash --delete-branch
```

Qué observar mientras pasa:

- El botón de merge estaba gris y se habilita solo cuando el check pasa a verde.
- `--squash` convierte todos los commits de la rama en uno solo sobre `main`,
  así que el historial queda con una línea por PR en lugar de con los intentos
  intermedios.
- `--delete-branch` borra la rama remota: ya cumplió su función.

### 10. Verifica el despliegue — 🖥️ demostración en vivo

> También en pantalla. Cualquiera puede _ver_ los runs porque el repo es
> público, pero conviene seguirlo una sola vez y juntos.

El merge dispara el workflow **Deploy to GitHub Pages**, que publica el sitio en
el environment `github-pages`.

```bash
gh run watch
```

Qué observar:

- Nadie lanzó el deploy a mano: lo disparó el push a `main` que produjo el
  merge.
- El primer step verifica el secret `SITE_DEPLOY_TOKEN` y falla el job entero si
  no existe.
- El job corre en el environment `github-pages` y publica la URL del sitio.

Cuando termine, el post está en línea en
<https://tfflores.github.io/github-lesson-example/>. El despliegue toma unos
segundos; si no ves el cambio, recarga forzando caché.

### Resumen en una pasada

```bash
git checkout main && git pull
git checkout -b post/mi-articulo
cp posts/tipos-de-vigas.html posts/mi-articulo.html
# editar posts/mi-articulo.html e index.html
npx --yes prettier@3 --write .
git add . && git commit -m "feat: agrega post sobre mi tema"
git push -u origin post/mi-articulo
gh pr create --base main
gh pr checks --watch
# ---- fin del ejercicio; lo de abajo es demostración en vivo ----
gh pr merge --squash --delete-branch
```
