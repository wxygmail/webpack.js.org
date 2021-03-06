---
title: postcss-loader
source: https://raw.githubusercontent.com/webpack-contrib/postcss-loader/master/README.md
edit: https://github.com/webpack-contrib/postcss-loader/edit/master/README.md
repo: https://github.com/webpack-contrib/postcss-loader
---

  
  <p>Loader for <a href="https://webpack.js.org/">webpack</a> to process CSS with <a href="https://postcss.org/">PostCSS</a></p>
</div>

## Install {#install}

```bash
npm i -D postcss-loader
```

## Usage {#usage}

### `Configuration` {#configuration}

**`postcss.config.js`**

```js
module.exports = {
  parser: 'sugarss',
  plugins: {
    'postcss-import': {},
    'postcss-preset-env': {},
    cssnano: {},
  },
};
```

You can read more about common PostCSS Config [here](https://github.com/michael-ciniawsky/postcss-load-config).

### `Config Cascade` {#config-cascade}

You can use different `postcss.config.js` files in different directories.
Config lookup starts from `path.dirname(file)` and walks the file tree upwards until a config file is found.

```
|– components
| |– component
| | |– index.js
| | |– index.png
| | |– style.css (1)
| | |– postcss.config.js (1)
| |– component
| | |– index.js
| | |– image.png
| | |– style.css (2)
|
|– postcss.config.js (1 && 2 (recommended))
|– webpack.config.js
|
|– package.json
```

After setting up your `postcss.config.js`, add `postcss-loader` to your `webpack.config.js`.
You can use it standalone or in conjunction with `css-loader` (recommended).
Use it **before** `css-loader` and `style-loader`, but **after** other preprocessor loaders like e.g `sass|less|stylus-loader`, if you use any (since [webpack loaders evaluate right to left/bottom to top](/concepts/loaders/#configuration)).

**`webpack.config.js`**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'postcss-loader'],
      },
    ],
  },
};
```

> ⚠️ When `postcss-loader` is used standalone (without `css-loader`) don't use `@import` in your CSS, since this can lead to quite bloated bundles

**`webpack.config.js` (recommended)**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          { loader: 'css-loader', options: { importLoaders: 1 } },
          'postcss-loader',
        ],
      },
    ],
  },
};
```

## Options {#options}

|                Name                 |            Type             |                Default                | Description                                     |
| :---------------------------------: | :-------------------------: | :-----------------------------------: | :---------------------------------------------- |
|           [`exec`](#exec)           |         `{Boolean}`         |              `undefined`              | Enable PostCSS Parser support in `CSS-in-JS`    |
|         [`config`](#config)         | `{String\|Object\|Boolean}` |              `undefined`              | Set `postcss.config.js` config path && `ctx`    |
| [`postcssOptions`](#postcssoptions) |         `{Object}`          | `defaults values for Postcss.process` | Set Postcss.process options and postcss plugins |
|      [`sourceMap`](#sourcemap)      |         `{Boolean}`         |          `compiler.devtool`           | Enables/Disables generation of source maps      |

### `Exec` {#exec}

Type: `Boolean`
Default: `undefined`

If you use JS styles without the [`postcss-js`][postcss-js] parser, add the `exec` option.

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.style.js$/,
        use: [
          'style-loader',
          { loader: 'css-loader', options: { importLoaders: 1 } },
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                parser: 'sugarss',
              },
              exec: true,
            },
          },
        ],
      },
    ],
  },
};
```

### `Config` {#config}

Type: `Boolean|String|Object`
Default: `undefined`

Options specified in the config file are combined with options passed to the loader.
Loader options overwrite options from config.

#### Boolean {#boolean}

Enables/Disables autoloading config.

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: 'postcss-loader',
        options: {
          config: false,
        },
      },
    ],
  },
};
```

#### String {#string}

Allows to specify the absolute path to the config file.

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: 'postcss-loader',
        options: {
          config: path.resolve(__dirname, 'custom.config.js'),
        },
      },
    ],
  },
};
```

#### Object {#object}

|           Name            |    Type    |   Default   | Description              |
| :-----------------------: | :--------: | :---------: | :----------------------- |
|      [`path`](#path)      | `{String}` | `undefined` | PostCSS Config Directory |
| [`context`](#context-ctx) | `{Object}` | `undefined` | PostCSS Config Context   |

##### `Path` {#path}

Type: `String`
Default: `undefined`

You can manually specify the path to search for your config (`postcss.config.js`) with the `config.path` option. This is needed if you store your config in a separate e.g `./config || ./.config` folder.

> ⚠️ Otherwise it is **unnecessary** to set this option and is **not** recommended

> ⚠️ Note that you **can't** use a **filename** other than the [supported config formats] (e.g `.postcssrc.js`, `postcss.config.js`), this option only allows you to manually specify the **directory** where config lookup should **start** from

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: 'postcss-loader',
        options: {
          config: {
            path: 'path/to/.config/', // ✅
            path: 'path/to/.config/css.config.js', // ❌
          },
        },
      },
    ],
  },
};
```

[supported config formats]: https://github.com/michael-ciniawsky/postcss-load-config#usage

##### `Context (ctx)` {#context-ctx}

Type: `Object`
Default: `undefined`

|   Name    |    Type    |        Default        | Description                      |
| :-------: | :--------: | :-------------------: | :------------------------------- |
|   `env`   | `{String}` |    `'development'`    | `process.env.NODE_ENV`           |
|  `file`   | `{Object}` | `loader.resourcePath` | `extname`, `dirname`, `basename` |
| `options` | `{Object}` |         `{}`          | Options                          |

`postcss-loader` exposes context `ctx` to the config file, making your `postcss.config.js` dynamic, so can use it to do some real magic ✨

**`postcss.config.js`**

```js
module.exports = ({ file, options, env }) => ({
  parser: file.extname === '.sss' ? 'sugarss' : false,
  plugins: [
    // Plugins with options and without
    ['postcss-import', { root: file.dirname }],
    'postcss-preset-env',
  ],
});
```

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: 'postcss-loader',
        options: {
          config: {
            ctx: {
              'postcss-preset-env': { ...options },
              cssnano: { ...options },
            },
          },
        },
      },
    ],
  },
};
```

### `postcssOptions` {#postcss-options}

|             Name              |                     Type                      |   Default   | Description                    |
| :---------------------------: | :-------------------------------------------: | :---------: | :----------------------------- |
|     [`plugins`](#plugins)     | `{Function\|Object\|Array<Function\|Object>}` |    `[]`     | Set PostCSS Plugins            |
|      [`parser`](#parser)      |         `{String\|Object\|Function}`          | `undefined` | Set custom PostCSS Parser      |
|      [`syntax`](#syntax)      |              `{String\|Object}`               | `undefined` | Set custom PostCSS Syntax      |
| [`stringifier`](#stringifier) |         `{String\|Object\|Function}`          | `undefined` | Set custom PostCSS Stringifier |

#### `Plugins` {#plugins}

Type: `Function|Object|Array<String|Function\|Object|Array>`
Default: `[]`

It is recommended to specify plugins in the format `Array<String\|Array>` or `Function` that returns the same array as shown below.
`Object` format (`{pluginName: pluginOptions}`) is deprecated and will be removed in the next major release.

**`webpack.config.js`**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: 'postcss-loader',
        options: {
          plugins: [
            'postcss-import',
            'postcss-nested',
            ['postcss-short', { prefix: 'x' }],
          ],
        },
      },
    ],
  },
};
```

**`webpack.config.js`**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: 'postcss-loader',
        options: {
          plugins: (loader) => [
            ['postcss-import', { root: loader.resourcePath }],
            'postcss-nested',
            'cssnano',
          ],
        },
      },
    ],
  },
};
```

**`webpack.config.js`**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: 'postcss-loader',
        options: {
          postcssOptions: {
            plugins: (loader) => [
              require('postcss-import')({ root: loader.resourcePath }),
              require('postcss-preset-env')(),
              require('cssnano')(),
            ],
          },
        },
      },
    ],
  },
};
```

> ⚠️ The method below for specifying plugins is deprecated.

**`webpack.config.js`**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: 'postcss-loader',
        options: {
          plugins: {
            'postcss-import': {},
            'postcss-nested': {},
            'postcss-short': { prefix: 'x' },
          },
        },
      },
    ],
  },
};
```

It is possible to disable the plugin specified in the config.

**`postcss.config.js`**

```js
module.exports = {
  plugins: {
    'postcss-short': { prefix: 'x' },
    'postcss-import': {},
    'postcss-nested': {},
  },
};
```

**`webpack.config.js`**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: 'postcss-loader',
        options: {
          postcssOptions: {
            plugins: {
              'postcss-import': {},
              'postcss-nested': {},
              // Turn off the plugin
              'postcss-short': false,
            },
          },
        },
      },
    ],
  },
};
```

#### `Parser` {#parser}

Type: `String|Object|Function`
Default: `undefined`

##### `String` {#string}

The passed `string` is converted to the form `require('string')`.

**`webpack.config.js`**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.sss$/i,
        loader: 'postcss-loader',
        options: {
          postcssOptions: {
            // Will be converted to `require('sugarss')`
            parser: 'sugarss',
          },
        },
      },
    ],
  },
};
```

##### `Object` {#object}

**`webpack.config.js`**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.sss$/i,
        loader: 'postcss-loader',
        options: {
          postcssOptions: {
            parser: require('sugarss'),
          },
        },
      },
    ],
  },
};
```

##### `Function` {#function}

**`webpack.config.js`**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.sss$/i,
        loader: 'postcss-loader',
        options: {
          postcssOptions: {
            parser: require('sugarss').parse,
          },
        },
      },
    ],
  },
};
```

#### `Syntax` {#syntax}

Type: `String|Object`
Default: `undefined`

##### `String` {#string}

The passed `string` is converted to the form `require('string')`.

**`webpack.config.js`**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: 'postcss-loader',
        options: {
          postcssOptions: {
            // Will be converted to `require('sugarss')`
            syntax: 'sugarss',
          },
        },
      },
    ],
  },
};
```

##### `Object` {#object}

**`webpack.config.js`**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: 'postcss-loader',
        options: {
          postcssOptions: {
            stringifier: require('sugarss'),
          },
        },
      },
    ],
  },
};
```

#### `Stringifier` {#stringifier}

Type: `String|Object|Function`
Default: `undefined`

##### `String` {#string}

The passed `string` is converted to the form `require('string')`.

**`webpack.config.js`**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: 'postcss-loader',
        options: {
          postcssOptions: {
            // Will be converted to `require('sugarss')`
            stringifier: 'sugarss',
          },
        },
      },
    ],
  },
};
```

##### `Object` {#object}

**`webpack.config.js`**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: 'postcss-loader',
        options: {
          postcssOptions: {
            stringifier: require('sugarss'),
          },
        },
      },
    ],
  },
};
```

##### `Function` {#function}

**`webpack.config.js`**

```js
const Midas = require('midas');
const midas = new Midas();

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: 'postcss-loader',
        options: {
          postcssOptions: {
            stringifier: midas.stringifier,
          },
        },
      },
    ],
  },
};
```

### `SourceMap` {#sourcemap}

Type: `Boolean`
Default: depends on the `compiler.devtool` value

By default generation of source maps depends on the [`devtool`](/configuration/devtool/) option. All values enable source map generation except `eval` and `false` value.

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          { loader: 'style-loader' },
          { loader: 'css-loader', options: { sourceMap: true } },
          { loader: 'postcss-loader', options: { sourceMap: true } },
          { loader: 'sass-loader', options: { sourceMap: true } },
        ],
      },
    ],
  },
};
```

## Examples {#examples}

### `Stylelint` {#stylelint}

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: ['postcss-import', 'stylelint'],
              },
            },
          },
        ],
      },
    ],
  },
};
```

### `Autoprefixing` {#autoprefixing}

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [['autoprefixer', { ...options }]],
              },
            },
          },
        ],
      },
    ],
  },
};
```

> :warning: [`postcss-preset-env`](https://github.com/csstools/postcss-preset-env) includes [`autoprefixer`](https://github.com/postcss/autoprefixer), so adding it separately is not necessary if you already use the preset.

### `CSS Modules` {#css-modules}

This loader [cannot be used] with [CSS Modules] out of the box due
to the way `css-loader` processes file imports. To make them work properly,
either add the css-loader’s [`importLoaders`] option.

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: { modules: true, importLoaders: 1 },
          },
          'postcss-loader',
        ],
      },
    ],
  },
};
```

or use [postcss-modules] instead of `css-loader`.

[`importloaders`]: https://github.com/webpack-contrib/css-loader#importloaders
[cannot be used]: https://github.com/webpack/css-loader/issues/137
[css modules]: https://github.com/webpack/css-loader#css-modules
[postcss-modules]: https://github.com/css-modules/postcss-modules

### `CSS-in-JS` {#css-in-js}

If you want to process styles written in JavaScript, use the [postcss-js] parser.

[postcss-js]: https://github.com/postcss/postcss-js

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.style.js$/,
        use: [
          'style-loader',
          { loader: 'css-loader', options: { importLoaders: 2 } },
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                parser: 'postcss-js',
              },
            },
          },
          'babel-loader',
        ],
      },
    ],
  },
};
```

As result you will be able to write styles in the following way

```js
import colors from './styles/colors';

export default {
  '.menu': {
    color: colors.main,
    height: 25,
    '&_link': {
      color: 'white',
    },
  },
};
```

> :warning: If you are using Babel you need to do the following in order for the setup to work

> 1. Add [babel-plugin-add-module-exports] to your configuration
> 2. You need to have only one **default** export per style module

[babel-plugin-add-module-exports]: https://github.com/59naga/babel-plugin-add-module-exports

### [Extract CSS][extractplugin] {#extract-cssextractplugin}

[extractplugin]: https://github.com/webpack-contrib/mini-css-extract-plugin

**`webpack.config.js`**

```js
const devMode = process.env.NODE_ENV !== 'production';

const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          devMode ? 'style-loader' : MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader',
        ],
      },
    ],
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: devMode ? '[name].css' : '[name].[hash].css',
    }),
  ],
};
```

### `Emit assets` {#emit-assets}

To write a asset from the postcss plugin to the webpack's output file system, need to add a message in `result.messages`.
The message should contain the following fields:

- `type` = `asset` - Message type (require, should be equal `asset`)
- `file` - file name (require)
- `content` - file content (require)
- `sourceMap` - sourceMap
- `info` - asset info

**`webpack.config.js`**

```js
const customPlugin = () => (css, result) => {
  result.messages.push({
    type: 'asset',
    file: 'sprite.svg',
    content: '<svg>...</svg>',
  });
};

const postcssPlugin = postcss.plugin('postcss-assets', customPlugin);

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [postcssPlugin()],
              },
            },
          },
        ],
      },
    ],
  },
};
```

### `Add dependencies` {#add-dependencies}

There are two way to add dependencies:

1. (Recommended). Postcss plugin should emit message in `result.messages`.

The message should contain the following fields:

- `type` = `dependency` - Message type (require, should be equal `dependency`)
- `file` - absolute file path (require)

**`webpack.config.js`**

```js
const path = require('path');

const customPlugin = () => (css, result) => {
  result.messages.push({
    type: 'dependency',
    file: path.resolve(__dirname, 'path', 'to', 'file'),
  });
};

const postcssPlugin = postcss.plugin('postcss-assets', customPlugin);

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [postcssPlugin()],
              },
            },
          },
        ],
      },
    ],
  },
};
```

2. Pass `loaderContext` in plugin.

**`webpack.config.js`**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              config: 'path/to/postcss.config.js',
            },
          },
        ],
      },
    ],
  },
};
```

**`postcss.config.js`**

```js
module.exports = (loaderContext) => ({
  postcssOptions: {
    plugins: [require('path/to/customPlugin')(loaderContext)],
  },
});
```

**`customPlugin.js`**

```js
const path = require('path');

const customPlugin = (loaderContext) => (css, result) => {
  loaderContext.webpack.addDependency(
    path.resolve(__dirname, 'path', 'to', 'file')
  );
};

module.exports = postcss.plugin('postcss-assets', customPlugin);
```

## Maintainers {#maintainers}

<table>
  <tbody>
    <tr>
      <td align="center">
        <a href="https://github.com/michael-ciniawsky">
          <img width="150" height="150" src="https://github.com/michael-ciniawsky.png?v=3&s=150">
          </br>
          Michael Ciniawsky
        </a>
      </td>
      <td align="center">
        <a href="https://github.com/evilebottnawi">
          <img width="150" height="150" src="https://github.com/evilebottnawi.png?v=3&s=150">
          </br>
          Alexander Krasnoyarov
        </a>
      </td>
    </tr>
  <tbody>
</table>

[npm]: https://img.shields.io/npm/v/postcss-loader.svg
[npm-url]: https://npmjs.com/package/postcss-loader
[node]: https://img.shields.io/node/v/postcss-loader.svg
[node-url]: https://nodejs.org/
[deps]: https://david-dm.org/postcss/postcss-loader.svg
[deps-url]: https://david-dm.org/postcss/postcss-loader
[tests]: https://img.shields.io/travis/postcss/postcss-loader.svg
[tests-url]: https://travis-ci.org/postcss/postcss-loader
[cover]: https://coveralls.io/repos/github/postcss/postcss-loader/badge.svg
[cover-url]: https://coveralls.io/github/postcss/postcss-loader
[chat]: https://badges.gitter.im/postcss/postcss.svg
[chat-url]: https://gitter.im/postcss/postcss
