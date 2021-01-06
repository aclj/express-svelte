Express Svelte
==================

A straightforward [Svelte](https://github.com/sveltejs/svelte) view engine for [Express](https://github.com/expressjs/express).

[![npm version][npm-badge]][npm]
[![dependency status][dep-badge]][dep-status]

## Install
Install via npm.
```bash
npm install express-svelte --save
```

## Goals & Design

I created this project because of the lack of a simple way to create, compile, render and hydrate partially or complete views for simple non-SPA web apps.

It is a view engine rather than an app framework (like [Sapper](https://github.com/sveltejs/sapper) or [Next.js](https://github.com/vercel/next.js)).

It [bypasses the express built-in view engine](https://strongloop.com/strongblog/bypassing-express-view-rendering-for-speed-and-modularity) and instead it sets a `res.svelte` function. The reason behind this is to avoid be tightly coupled with express (can be easily extended for use with [Polka](https://github.com/lukeed/polka)) and some limitations like the `app.locals` and `res.locals` merge logic. 

Inspired on this [Medium post](https://medium.com/@luke_schmuke/how-we-achieved-the-best-web-performance-with-partial-hydration-20fab9c808d5) that explains the reasons and goals behind partial hydration.

If hydration (partial or complete) is desired, the simple [rollup-plugin-express-svelte](https://github.com/maxiruani/rollup-plugin-express-svelte) plugin will take care of bundling the views. Full example of working app at [express-svelte-example](https://github.com/maxiruani/express-svelte-example).

## Features
- Static content without hydration.
- **Partial** and **complete** hydration.
- Compile views asynchronously (no need of `svelte/register`) with sane defaults and customizable configuration.
    * Support for preprocess plugins.
    * Replace values (like `process.env.NODE_ENV` and `process.browser`).
    * Dedupe dependencies.
    * Sourcemap support.
    * Cache.
- Global values set at top-level component via [Svelte Context API](https://svelte.dev/docs#getContext):
    * Global **props** based on `app.locals`, `res.locals`, `req.locals` and `opts.globalProps`.
    * Global **store** based on `app.locals`, `res.locals`, `req.locals` and `opts.globalStore`.
    * Global **assets** (server side only). You can access the compiled script or style urls from the views directory.
- Customizable root [lodash template](https://lodash.com/docs/4.17.15#template) ([very fast](https://ghcdn.rawgit.org/eta-dev/eta/master/browser-tests/benchmark.html) and similar to EJS syntax). 

## Basic setup and usage

> Full setup example available at [express-svelte-example](https://github.com/maxiruani/express-svelte-example).

```javascript
const expressSvelte = require('express-svelte');
// ...

// Configure
app.use(expressSvelte({ /* config */ }));

// Render view
app.get((req, res, next) => {
    res.svelte('View', {
        props: { /* ... */ },
        globalProps: { /* ... */ },
        globalStore: { /* ... */ }
    });
});
```

## Config options

#### `viewsDirname`
**Type:** `String | String[]`.

Defaults to `process.cwd() + "/views"`.

A directory or an array of directories for the app's views (svelte files).

#### `bundlesDirname`
**Type:** `String | String[]`.

Defaults to `process.cwd() + "/public/dist"`.

A directory or an array of directories for the app's views compiled bundles (js and css files) generated by the client rollup config.

#### `bundlesPattern`
**Type:** `String`.

Defaults to `"[name]-[hash][extname]"`.

Bundles rollup output format. `[name]` will be used for easy reference at `globalAssets`.

#### `bundlesHost`
**Type:** `String`.

Defaults to `""`.

An optional host to prefix JS or CSS bundles. Eg, a CDN host or different subdomain.

#### `defaultExtension`
**Type:** `String`.

Defaults to `".svelte""`.

Engine default extension resolution. The lookup has the same behavior as Express built-in `res.render` function.
If you do `res.svelte('View')` it will look for `"View.svelte"`.

#### `templateFilename`
**Type:** `String`.

Defaults to absolute path to package's default root lodash template.

If the default template does not fit for you, you can set a customized root template.

#### `env`
**Type:** `Boolean`.

Defaults to `process.env.NODE_ENV` or `"development"` if not set.

Used to determine `dev` and `cache` values if not specified.

It replaces `process.env.NODE_ENV` with [@rollup/plugin-replace plugin](https://github.com/rollup/plugins/tree/master/packages/replace#readme) by default.

#### `dev`
**Type:** `Boolean`.

Default is inferred with `env`. If `env` is `"development"` or `"testing"` is set to `true`.

It sets `dev`, `preserveComments` and `preserveWhitespace` properties at [rollup-plugin-svelte](https://github.com/sveltejs/rollup-plugin-svelte) plugin and enables source map support.

#### `cache`
**Type:** `Boolean`.

Default is inferred with `env`. If `env` is `"development"` or `"testing"` is set to `false`.

#### `hydratable`
**Type:** `Boolean`.

Default is `false`.

Hydratable value to be used at [rollup-plugin-svelte](https://github.com/sveltejs/rollup-plugin-svelte) plugin.

If you are going to make `"complete"` or `"partial"` hydration from [rollup-plugin-express-svelte](https://github.com/maxiruani/rollup-plugin-express-svelte) it needs to be configured to build both bundles. You can check out the [express-svelte-example](https://github.com/maxiruani/express-svelte-example) this must be enabled.

If disabled, props, globals and scripts won't be exposed in the HTML.

#### `legacy`
**Type:** `Boolean`.

Default is `true`.

A combination of modern and legacy  builds will be used, the modern with `type="module"` attribute and the legacy with the `nomodule` attribute.
 
Browsers that understand `type="module"` should ignore scripts with a `nomodule` attribute. This means you can serve a module tree to module-supporting browsers while providing a fall-back to other browsers.
 
You can read the full article about this [here](https://jakearchibald.com/2017/es-modules-in-browsers/).

If enabled, when using [rollup-plugin-express-svelte](https://github.com/maxiruani/rollup-plugin-express-svelte) it needs to be configured to build both bundles. You can check out the [express-svelte-example](https://github.com/maxiruani/express-svelte-example).

#### `replace`
**Type:** `Object`.

Default is `{}`.

Object with key-value pairs to be replaced with [@rollup/plugin-replace plugin](https://github.com/rollup/plugins/tree/master/packages/replace#readme) plugin.

Like `process.env.NODE_ENV` replacement, a default replacement for `process.env.browser` is made.

#### `preprocess`
**Type:** `Array`.

Default is `[]`.

Preprocess array to be used at [rollup-plugin-svelte](https://github.com/sveltejs/rollup-plugin-svelte) plugin.

#### `dedupe`
**Type:** `String[]`.

Default is `[]`.

Dependencies array to dedupe array to be used at [@rollup/plugin-node-resolve](https://github.com/rollup/plugins/tree/master/packages/node-resolve/#readme) plugin.

Every dependency you add will extend the following dedupe array:
```javascript
 [
    'svelte',
    'svelte/animate',
    'svelte/easing',
    'svelte/internal',
    'svelte/motion',
    'svelte/store',
    'svelte/transition'
]
```

## Render options

#### `cache`
**Type:** `Boolean`.

Overrides option set at the main config.

#### `hydratable`
**Type:** `Boolean`.

Overrides option set at the main config.

#### `templateFilename`
**Type:** `String`.

Overrides option set at the main config or package's default.

#### `props`
**Type:** `Object`.

Props passed to View svelte component.

#### `globalProps`
**Type:** `Object`.

Props passed global context accessible via `getContext('global.props')`.

#### `globalStore`
**Type:** `Object`.

Props passed global store accessible via `getContext('global.store')`.

## Global props and store logic and behavior

If you need to make data available globally to all components in the view you can set your values at:

For `globalProps` accessed via `getContext('global.props')`:
- `app.locals` (Lowest priority at merge)
- `req.locals`
- `res.locals`
- `renderOpts.globalProps` (Highest priority at merge)

For `globalStore` accessed via `getContext('global.store')`:
- `app.locals` (Lowest priority at merge)
- `req.locals`
- `res.locals`
- `renderOpts.globalStore` (Highest priority at merge)

To prevent sensitive data to be accidentally shipped to the browser, by default none of the keys in `app.locals`, `req.locals` or `res.locals` are serialized. If you want the data to be serialized and ship to the frontend you need to specify it in `$globalProps` or `$globalStore` inside of one the `locals` object.

Example:
```javascript
res.locals.device = 'mobile';
res.locals.username = '@user';
res.locals.email = 'example@example.com'; // Not exposed

res.locals.$globalProps = { device: true }; 
res.locals.$globalStore = { username: true };

res.render('View', {
    globalProps: {
        title: 'View title!'
    },
    globalStore: {
        posts: []
    }
});
```

Merge output:
- `globalProps` -> `{ device: 'mobile', title: 'View title!' }`
- `globalStore` -> `{ username: '@user', posts: [] }`

## Template

The variables it receives are:
 - `head`: Output from `<svelte:head>` component.
 - `style`: CSS code.
 - `globalProps`: Serialized global props.
 - `globalStore`: Serialized global store.
 - `props`: View props.
 - `script`: Script url.
 - `scriptLegacy`: Script url of legacy build.
 - `scriptModern`: Script url of modern build.
 - `hydratable`: Boolean indicating if the view is hydratable.
 - `legacy`: Boolean indicating if legacy support is enabled.
 - `html`: HTML code.
 
 Also [@nuxt/devalue](https://github.com/nuxt/devalue) function is provided to serialize props.
 This is made at template level to avoid serializing props that are not necessary according to hydration config.
 
 