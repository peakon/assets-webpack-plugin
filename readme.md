# assets-webpack-plugin
[![Build Status](https://travis-ci.org/kossnocorp/assets-webpack-plugin.svg?branch=master)](https://travis-ci.org/kossnocorp/assets-webpack-plugin) [![Build status](https://ci.appveyor.com/api/projects/status/qmvi3h6lk0xu8833?svg=true)](https://ci.appveyor.com/project/kossnocorp/assets-webpack-plugin)

Webpack plugin that emits a json file with assets paths.

## Table of Contents

- [Why Is This Useful?](#why-is-this-useful)

- [Install](#install)

- [Configuration](#configuration)

- [Test](#test)

- [Change Log](./CHANGELOG.md)

- [Contributing Guidelines](./CONTRIBUTING.md)

## Why Is This Useful?

When working with Webpack you might want to generate your bundles with a generated hash in them (for cache busting).

This plug-in outputs a json file with the paths of the generated assets so you can find them from somewhere else.

### Example output:

The output is a JSON object in the form:

```json
{
    "bundle_name": {
        "asset_kind": "/public/path/to/asset"
    }
}
```

Where:

  * `"bundle_name"` is the name of the bundle (the key of the entry object in your webpack config, or "main" if your entry is an array).
  * `"asset_kind"` is the camel-cased file extension of the asset

For example, given the following webpack config:

```js
{
    entry: {
        one: ['src/one.js'],
        two: ['src/two.js']
    },
    output: {
        path: path.join(__dirname, "public", "js"),
        publicPath: "/js/",
        filename: '[name]_[hash].bundle.js'
    }
}
```

The plugin will output the following json file:

```json
{
    "one": {
        "js": "/js/one_2bb80372ebe8047a68d4.bundle.js"
    },
    "two": {
        "js": "/js/two_2bb80372ebe8047a68d4.bundle.js"
    }
}
```

## Install

```sh
npm install assets-webpack-plugin --save-dev
```

## Configuration

In your webpack config include the plug-in. And add it to your config:

```js
var path = require('path')
var AssetsPlugin = require('assets-webpack-plugin')
var assetsPluginInstance = new AssetsPlugin()

module.exports = {
    // ...
    output: {
        path: path.join(__dirname, "public", "js"),
        filename: "[name]-bundle-[hash].js",
        publicPath: "/js/"
    },
    // ....
    plugins: [assetsPluginInstance]
}
```

### Options

You can pass the following options:

#### `filename`

Optinal. `webpack-assets.json` by default.

Name for the created json file.

```js
new AssetsPlugin({filename: 'assets.json'})
```

#### `fullPath`

Optinal. `true` by default.

If `false` the output will not include the full path
of the generated file.

```js
new AssetsPlugin({fullPath: false})
```

e.g.

`/public/path/bundle.js` vs `bundle.js vs`

#### `includeManifest`

Optional. `false` by default.

Inserts the manifest javascript as a `text` property in your assets.
Accepts the name of your manifest chunk. A manifest is the last CommonChunk that
only contains the webpack bootstrap code. This is useful for production
use when you want to inline the manifest in your HTML skeleton for long-term caching.
See [issue #1315](https://github.com/webpack/webpack/issues/1315)
or [a blog post](https://medium.com/@matt.krick/a-production-ready-realtime-saas-with-webpack-7b11ba2fa5b0#.p1vvfr3bm)
to learn more.

```js
new AssetsPlugin({includeManifest: 'manifest'})
// assets.json:
// {entries: {manifest: {js: `hashed_manifest.js`, text: 'function(modules)...'}}}
//
// Your html template:
// <script>
// {assets.entries.manifest.text}
// </script>
```

#### `path`

Optional. Defaults to the current directory.

Path where to save the created JSON file.

```js
new AssetsPlugin({path: path.join(__dirname, 'app', 'views')})
```

#### `prettyPrint`

Optional. `false` by default.

Whether to format the JSON output for readability.

```js
new AssetsPlugin({prettyPrint: true})
```

#### `processOutput`

Optional. Defaults is JSON stringify function.

Formats the assets output.

```js
new AssetsPlugin({
  processOutput: function (assets) {
    return 'window.staticMap = ' + JSON.stringify(assets)
  }
})
```

#### `update`

Optinal. `false` by default.

When set to `true`, the output JSON file will be updated instead of overwritten.

```js
new AssetsPlugin({update: true})
```

#### `metadata`

Inject metadata into the output file. All values will be injected into the key "metadata".

```js
new AssetsPlugin({metadata: {version: 123}})
// Manifest will now contain:
// {
//   metadata: {version: 123}
// }
```

### Using in multi-compiler mode

If you use webpack multi-compiler mode and want your assets written to a single file,
you **must** use the same instance of the plugin in the different configurations.

For example:

```js
var webpack = require('webpack')
var AssetsPlugin = require('assets-webpack-plugin')
var assetsPluginInstance = new AssetsPlugin()

webpack([
  {
    entry: {one: 'src/one.js'},
    output: {path: 'build', filename: 'one-bundle.js'},
    plugins: [assetsPluginInstance]
  },
  {
    entry: {two:'src/two.js'},
    output: {path: 'build', filename: 'two-bundle.js'},
    plugins: [assetsPluginInstance]
  }
])
```


### Using this with Rails

You can use this with Rails to find the bundled Webpack assets via Sprockets.
In `ApplicationController` you might have:

```ruby
def script_for(bundle)
  path = Rails.root.join('app', 'views', 'webpack-assets.json') # This is the file generated by the plug-in
  file = File.read(path)
  json = JSON.parse(file)
  json[bundle]['js']
end
```

Then in the actions:

```ruby
def show
  @script = script_for('clients') # this will retrieve the bundle named 'clients'
end
```

And finally in the views:

```erb
<div id="app">
  <script src="<%= @script %>"></script>
</div>
```

## Test

```sh
npm test
```
