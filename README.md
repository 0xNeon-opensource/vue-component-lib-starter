[![Netlify Status](https://api.netlify.com/api/v1/badges/b1b84831-789e-4629-a9e3-55a36e136653/deploy-status)](https://app.netlify.com/sites/sharp-babbage-154f0a/deploys)

# Vue Component Library Starter

> Create your own [Vue 3](https://v3.vuejs.org/) component library with TypeScript, [Vite](https://vitejs.dev) and [VuePress 2](https://v2.vuepress.vuejs.org).

Sooner or later, you will find that creating a component library is much better than having all components inside your app project. A component library force to you remove app specific logic from your components, making it easier to test and reuse them in other apps.

Once the components are in a libraray, documentation becomes critical. This starter project includes a documentation app powered by VuePress. It not only documents the usage of the component, but also provides a testing bed during the development of components. See the generated documentation app [here](https://sharp-babbage-154f0a.netlify.com/).

## Setup

```bash
# install dependencies
npm install

# start the doc app with hot reload, great for testing components
npm run docs:dev

# build the library, available under dist
npm run build

# build the doc app, available under docs/.vuepress/dist
npm run docs:build
```

You may use [Netlify](https://www.netlify.com/) to auto build and deloy the doc app like this project does. To preview the docs build locally, serve the folder `docs/.vuepress/dist` using [http-server](https://www.npmjs.com/package/http-server). Double-clicking the `index.html` won't work.

## Develop and test locally

The best way to develop and test your component is by creating demos in `docs` folder, as shown by the example components.

If you want to test the library in your Vue3 app:

- In the root folder of this library, run `npm link`. This will create a symbolic link to the library.
- In the root folder of your client app, run `npm link my-lib`. This will add the symbolic link to the `node_modules` folder in your client app.

If you made changes to the library, you will need to rebuild the library. Your Vue3 app will hot reload when the library is built.

## How it works

### Components

The library is a [Vue plugin](https://v3.vuejs.org/guide/plugins.html). The `install` function in [index.ts](src/index.ts) registers all components under [components](src/components) to Vue globably.

The components are also exported by [index.ts](src/index.ts) so that the client app can import them individually and register them locally, instead of using the library as a plugin. This may be a better option if the client app only use a small set of components in your library.

### Utilities and constants

The library includes example utilities and constants. They are also exported in [index.ts](src/index.ts). The client app may use them as below:

```js
<script lang="ts">
import { MyConstants, MyUtil } from 'my-lib'

export default {
  data () {
    return {
      magicNum: MyConstants.MAGIC_NUM
    }
  },
  methods: {
    add (a:number, b:number) {
      return MyUtil.add(a, b)
    }
  }
}
</script>
```

### Styling

Individual compopnent may have styles defined in its `.vue` file. They will be processed, combined and minified into `dist/style.css`, which is included in the `exports` list in [package.json](package.json).

If you have library level styles shared by all components in the library, you may add them to `assets/main.scss`. This file is imported in [index.ts](src/index.ts), and the build also includes the processed styles into `dist/style.css`. To avoid conflicting with other global styles, consider pre-fixing the class names or wrapping them into a namespace class.

The client app shall import `my-lib/style.css`, usually in the entry file:

```js
import 'my-lib/dist/style.css';
```

### Third-party dependencies

Third-party libraries you library is using may bloat the size of your library, if you simply add them to the `dependencies` in [package.json](package.json).

The following are some strategies to reduce the size of your library:

#### Externalization

If you expect the client app of your library may also need the same dependency, you may externalize the dependency. For example, in [vite.config.ts](vite.config.ts), you may have

```js
module.exports = defineConfig({
    rollupOptions: {
      external: ['moment'] // exclude 'moment' from your library
    }
  }
})
```

The dependency to be externalized may be declared as peer dependency in your library.

#### Cherry picking

If you don't expect the client app of your library also needing the same dependency, you may embed cherry-picked functions. For example, to embed the `fill` function of popular library [lodash](https://lodash.com), import the `fill` function like the following:

```js
import fill from 'lodash/fill';
```

Even with tree-shaking, the codes being brought into your library may still be large, as the function may have its own dependencies.

Note that `import { fill } from 'lodash'` or `import _ from 'lodash'` will not work and will embed the whole `lodash` library.

Finally, if your client app also use `lodash` and you don't want `lodash` to be in both the client app and your libraries, even after cherry-picking, you may consider cherry-picking in component library and re-export them as utils for client to consume, so that the client does not need to depend on `lodash`, therefore avoiding duplication.

### Type generation

The type definitions of the components, utilities and constants are generated by the `build` script in [package.json](package.json), via command line option `vue-tsc --declaration --emitDeclarationOnly`. The generated `.d.ts` files are output to `types` folder, as specified by `compilerOptions.declarationDir` in [tsconfig.json](tsconfig.json).

In [package.json](package.json), `"types": "./types/index.d.ts"` locates the generated types for consumption by the client.

The folder `types` is also included in `files` in [package.json](package.json), so that it will be included in npm publish.

### Misc

In [tsconfig.json](tsconfig.js), `compilerOptions.isolatedModules` is set to `true` as recommended by Vite (since esbuild is used). However, enableing this option leads to [https://github.com/vitejs/vite/issues/5814](https://github.com/vitejs/vite/issues/5814). The workaround is to also enable `compilerOptions.skipLibCheck`.
