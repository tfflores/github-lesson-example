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

## Flujo de trabajo

1. Crea una rama desde `main` (ej. `post/mi-articulo`).
2. Haz commits y abre un Pull Request usando la plantilla.
3. Espera que el check **Lint** quede en verde.
4. Mergea con squash; el workflow de deploy publica el sitio en Pages.
