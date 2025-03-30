# Template for Vue3 website with automatic deployments to GitHub Pages

## GitHub Pages branch

The 'master' brach of this repository is set to automatically compile a production build and deploy to GitHub Pages. This is configurable [here](https://github.com/Morganhtrotter/vue-gh-pages-template/blob/master/.github/workflows/main.yml#L7).

## Template Setup

1. Follow [this](https://docs.github.com/en/repositories/creating-and-managing-repositories/duplicating-a-repository) guide to duplicate this template into a NEW_REPOSITORY.
2. In your NEW_REPOSITORY in browser, under /settings/pages, change source to GitHub Actions.
3. Change the [base_url](https://github.com/Morganhtrotter/vue-gh-pages-template/blob/master/vite.config.js#L13) in your vite.config.js file to match the name of your NEW_REPOSITORY.
4. Verify your changes made it to production at the GitHub pages url. NOTE: you may need to manually run the 'Deploy static content to Pages' workflow if it didnt run automatically.

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```

### Lint with [ESLint](https://eslint.org/)

```sh
npm run lint
```
