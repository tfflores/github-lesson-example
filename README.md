# Blog Materiales de Construcción — repositorio de demostración

Sitio estático (HTML + CSS puros, sin frameworks ni build) usado como ejemplo en
una capacitación interna sobre **GitHub como servicio**. Todo el contenido es
ficticio.

## Qué se demuestra aquí

| Tema                  | Dónde verlo                                            |
| --------------------- | ------------------------------------------------------ |
| Repositorio público   | Este repo                                              |
| Ramas                 | `post/tipos-de-vigas`, `post/aceros-estructurales`     |
| Ciclo de Pull Request | PRs #1 (mergeado) y #2 (abierto)                       |
| Actions — lint        | `.github/workflows/lint.yml` (corre en `pull_request`) |
| Actions — deploy      | `.github/workflows/deploy-pages.yml` (corre en `main`) |
| GitHub Pages          | Publicado desde Actions                                |
| CODEOWNERS            | `.github/CODEOWNERS`                                   |
| Plantilla de PR       | `.github/pull_request_template.md`                     |
| Environments          | `github-pages`, usado por el workflow de deploy        |
| Ruleset               | Protección de `main`: PR obligatorio + check `Lint`    |
| Releases              | Tag `v1.0.0`                                           |

## Estructura

```
index.html                        portada con el listado de posts
posts/                            un archivo HTML por publicación
assets/styles.css                 estilos del sitio
.prettierrc                       configuración del formateador
CLAUDE.md                         instrucciones para Claude Code (modo tutor)
.github/CODEOWNERS                dueños de código
.github/pull_request_template.md  plantilla de PR
.github/workflows/                lint y deploy
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

### 9. Mergea

El botón de merge sigue bloqueado hasta que `Lint` esté verde — así lo exige el
ruleset de `main`.

```bash
gh pr merge --squash --delete-branch
```

### 10. Verifica el despliegue

El merge dispara el workflow **Deploy to GitHub Pages**, que publica el sitio en
el environment `github-pages`.

```bash
gh run watch
```

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
gh pr merge --squash --delete-branch
```
