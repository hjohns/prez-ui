# Prez UI
Prez UI is the front end of [Prez](https://github.com/RDFLib/prez) - a linked data API.

Prez UI is a [Vue.js](https://vuejs.org/) single page application (SPA) that uses [N3.js](https://github.com/rdfjs/N3.js) to process RDF data from the Prez API.

See the Prez UI demo website - https://rdflib.dev/prez-ui/

## Environment Variables
Configuring an instance of Prez UI is done by supplying environment variables.

[Vite](https://vitejs.dev) automatically imports environment variables from `.env*` files with a preference order (see [Vite Env Variables and Modes](https://vitejs.dev/guide/env-and-mode.html#env-files)), where `.env.[mode].local` will be preferenced over `.env.[mode]` files for example, where `[mode]` would be `development`, `production`, etc. For convenience for local development, it is recommended to create a `.env.development.local` file to set your local environment variables.

See [`.env`](./.env) for the default values for the available environment variables. Note that environment variables must be prefixed with `VITE_`.

## Theming
Prez UI instances can be themed by providing header & footer HTML files, as well as custom CSS styling. After building your instance of Prez UI (**before which your [environment variables](#environment-variables) must be set**):

```bash
npm install
npm run build
```

copy your files into the `dist/theme/` directory, overriding existing files when necessary. See the [`public/theme/`](public/theme) folder for the required files:

- `favicon.ico`
- `header.html`
- `footer.html`
- `theme.css`

These files should be left blank if not being used.

The [`public/style.css`](public/style.css) CSS file contains CSS variables that can be overridden in your `theme.css` file for easy theming.

## Docker
Prez UI can be deployed as a Docker container, which contains an NGINX server serving static files. To theme your Docker container, create a volume at `/app/theme` called `theme/` containing the theming files, e.g.:

```bash
docker run -v <YOUR_THEME_FOLDER>:/app/theme rdflib/prez-ui
```

### SSL
To enable SSL for a Docker container deployment, supply your own `nginx.conf` file containing the commented-out lines for SSL in the supplied [`nginx.conf`](./nginx.conf) file in a Docker volume (yet to be implemented).

## Development
[Visual Studio Code](https://code.visualstudio.com/) is the recommended IDE for development on Prez UI because of its intellisense with TypeScript. These extensions are highly recomended:

- [Vue Language Features (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.volar)
- [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin)
    - Make sure to enable [takeover mode](https://vuejs.org/guide/typescript/overview.html#volar-takeover-mode)

Nice-to-haves:

- [Vue VSCode Snippets](https://marketplace.visualstudio.com/items?itemName=sdras.vue-vscode-snippets)