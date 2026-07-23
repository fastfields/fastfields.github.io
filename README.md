# fastfields.github.io

Source for the **fastfields** landing page at <https://fastfields.github.io/>.

A [zensical](https://github.com/mkdocs/zensical) site whose home page introduces
the project and links out to each package's own API docs (published at
`fastfields.github.io/<repo>/`) and to the [wheel index](https://fastfields.github.io/whl/).

## Build locally

```sh
pip install -r docs/requirements.txt
zensical build --clean      # output in site/
python -m http.server -d site
```

## Deploy

`.github/workflows/docs.yaml` builds and deploys to GitHub Pages on every push
to `main`. One-time: set **Settings → Pages → Source** to **GitHub Actions**.
