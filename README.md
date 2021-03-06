<p align="center"><img src="https://raw.githubusercontent.com/ericbiewener/vscode-vandelay-js/master/logo.png" width="128" height="112" align="center" /></p>
<h1 align="center">Vandelay JS</h1>

<p align="center">
  <strong>Official JavaScript plugin for the <a href="https://github.com/ericbiewener/vscode-vandelay">Vandelay VS Code extension</a></strong>.
  <br />
  An official <a href="https://github.com/ericbiewener/vscode-vandelay-py">Python</a> plugin also exists.
</p>

<br />
<p align="center">
<img src="https://raw.githubusercontent.com/ericbiewener/vscode-vandelay-js/master/artwork/animation.gif" width="757" height="426" align="center" />
</p>
<br />

## Table of Contents
- [Overview](#overview)
- [Flow & Typescript Support](#flow--typescript-support)
- [Commands](#commands)
- [Importing from node_modules and Node.js modules](#importing-from-node_modules-and-nodejs-modules)
- [How to Use](#how-to-use)
- [Configuration](#configuration-vandelay-jsjs)
- [Multi-Root Workspace](#multi-root-workspace)
- [Example Configuration File](#example-configuration-file)
- [Settings](#settings)

## Overview
<a href="https://www.youtube.com/watch?v=W4AN8Eb2LL0&t=2m10s" target="_blank"><img src="https://raw.githubusercontent.com/ericbiewener/vscode-vandelay/master/artwork/video.jpg" alt="He's an importer exporter" width="240" align="right" /></a>
Importing code is annoying and the current VS Code tooling around it isn't good enough.
This plugin keeps track of all available imports and allows you to quickly import them following
whatever style guide your project requires for how import statements get written (see
[Configuration](#configuration-vandelay-jsjs)). Multi-root workspaces are supported ([documentation](#multi-root-workspace)).

## Flow & Typescript Support
Flow types are supported, but Typescript currently is not.

## Commands
The following commands are available from the Command Palette. Feel free to set your own keyboard shortcuts.

### Cache Project Exports
Caches all project exports in all languages that have a Vandelay configuration file (see 
[How to Use](#how-to-use)). Vandelay will automatically run this command the first time it
initializes for a given project, and the plugin will watch for file changes (including branch
switching, file deletion, etc) in order to update its cache of available imports. But you may need
to manually run this command if files are changed while VS Code is not running.

### Import
Select an import from your project.

### Import active word
A shortcut to automatically import the word under the carat. If more than one import matching the
active word are found, you'll be asked to choose.

### Import type
Select an available import, but filtered to show just Flow types.

## Importing from node_modules and Node.js modules
Rather than try to actually parse and track all the possible imports in your project's
`node_modules` folder, Vandelay JS simply tracks the ones you use. This means you'll need to
write the import statement yourself the very first time you use a node_modules import, but the
plugin will remember it after that and make it available for automatic importing. The same thing
goes for importing Node.js modules.

## How to Use
Vandelay relies on JavaScript configuration files, not simply JSON. As the below configuration
options demonstrate, this allows the plugin to be fully customized to your project's needs.

## Configuration (vandelay-js.js)
You must create a file named `vandelay-js.js` at the root of your project. If using a multi-root
workspace, see [those instructions](#multi-root-workspace).

#### *Any time you make changes to this file, you must reload the window.*

Along with providing configuration options, the presence of this file tells the plugin that it
should track your project's JavaScript imports. The lack of a `vandelay-js.js` file in a given
project will simply cause the plugin not to run.

The configuration file must be written in JavaScript and export an object (`module.exports = { ...
}` syntax) containing the desired configuration options. This file may be as simple as something like:

```js
const path = require('path')
module.exports = {
  includePaths: [path.join(__dirname, 'src')]
}
```

See [this sample configuration file](#example-configuration-file) for a more complex example.

### `includePaths: Array<string>`
An array of filepaths that Vandelay should watch for exports. This is the only required configuration option.

### `excludePatterns: Array<string | RegExp>`
An array of glob patterns and regular expressions that match filepaths which should be excluded from caching.
This plugin automatically excludes `node_modules`.

### `importGroups: Array<string>`
Vandelay will automatically sort import statements so that node modules come before your project's
custom imports, and it will alphabetize them by path. This configuration option allows you to select
specific imports that should always be sorted first.

### `maxImportLineLength: number`
Defaults to 100. Used to determine when to write multiline import statements.

### `useES5: boolean`
Defaults to `false`. If your project uses ES5 module syntax (i.e. `require`) you should set this to
true. Only `module.exports = { foo, bar }` and `module.exports = defaultExport` syntax is supported.
  
### `padCurlyBraces: boolean`
Defaults to `true`. Whether import statements should include spaces between curly braces and import
names.

### `useSingleQuotes: boolean`
Defaults to `true`. Whether import statements should be writting with single or double quotes.

### `useSemicolons: boolean`
Defaults to `true`. Whether import statements should be writting with semicolons.

### `multilineImportStyle: 'multiple' | 'single'`
Defaults to `multiple`. Whether to allow multiple imports on a line when the import needs to span
multiple lines because it has gone over the allowed line length (LINK TO CORE README LINE LENGTH)

**multiple**
```js
import { var1, var2,
  var3 } from '...'
```

**single**
```js
import {
  var1,
  var2,
  var3
} from '...'
```

### `trailingComma: boolean`
Defaults to `true`. Whether multiline statements should include trailing commas. Only relevant when
`multilineImportStyle` is `single`.

### `processDefaultName: filepath => ?string`
Default exports will be tracked using the file name (i.e. a default export in `myFile.js` will be
named `myFile`). Implement this setting to modify this behavior on a file-by-file basis. By
returning a falsey value, the default filename-based naming will still be used.

* `filepath`: is the absolute path to the file on your computer.

```js
processDefaultName: filepath => filepath === "/Users/eric/my-project/src/foo/bar.js" ? "greatName" : null
```

### `processImportPath: (importPath: string, absImportPath: string, activeFilepath: string, projectRoot: string) => ?string`
When inserting a new import, this setting allows you to modify the import path that gets written to
the file. Useful if you have your build tool configured in a way that allows it to process
non-relative paths (for example, all your imports are relative to the project root). Returning a
falsey value will all the standard relative path to be used.

* `importPath`: relative import path that will be written if you don't return a value
* `absImportPath`: absolute path of the import file
* `activeFilepath`: absolute path to the active file open in your editor
* `projectRoot`: absolute path to the root of your project

```js
processImportPath: (importPath, absImportPath, activeFilepath, projectRoot) => (
  absImportPath.startsWith("/Users/eric/my-project/absoluteImportDirectory")
    ? absImportPath.slice(projectRoot.length + 1)
)
```

### `nonModulePaths: Array<string>`
if you have configured your build tool to allow imports relative to the project root for certain
paths (thus causing them not to begin with `./` or `../`), specify the roots of these paths here.
These should not be absolute paths, e.g. use `src1` not `Users/eric/my-project/src1`.

*This is done only to prevent them from being considered node_module imports when caching or
determining import order*. You must use `processImportPath` to have the desired path actually get
written to the file instead of the relative path.

### `shouldIncludeImport: (absImportPath: string, activeFilepath: string) => boolean`
May be used to exclude certain imports from the list of options.

* `absImportPath`: absolute path of the import file
* `activeFilepath`: absolute path to the active file open in your editor

```js
shouldIncludeImport: (absImportPath, activeFilepath) => (
  absImportPath.includes('__mocks__') && !activeFilepath.endsWith('.test.js')
)
```

### `preferTypeOutside: boolean`
Defaults to `false`. If using Flow, settings this to `true` will cause import statements for types
to put the type on the outside of the braces (`import type { type1, type2 } ...`) *so long as only
types are being imported from the given import path*. This can help mitigate circular dependency
issues under some circumstances. Regardless of this setting, if a value import exists for a given
path then the syntax `import { myVal, type1, type2 } ...` will be used.

## Multi-Root Workspace
You must add a `.vandelay` directory to your workspace that contains a file named `vandelay-js.js`.
Along with the above configuration options, you must also provide a `projectRoot` string that
specifies the absolute path to the directory that should be considered the overall root of your
project. This will be used for determining relative paths (these paths may always be adjusted via
the `processImportPath` configuration option described above.

## Example Configuration File
```js
const path = require('path')

const src1 = path.join(__dirname, 'src1')
const src2 = path.join(__dirname, 'src2')
const src3 = path.join(__dirname, 'src3')

module.exports = {
  includePaths: [src1, src2, src3],
  excludePatterns: [
    // No need to include `node_modules`. Vandelay will exclude that automatically.
    "**/*.test.*",
    /.*\/flow-typed\/.*/,
    /.*\/config\/.*/,
  ],
  importGroups: ['react', 'react-dom', 'redux', 'react-redux'],
  padCurlyBraces: false,
  useSingleQuotes: false,
  useSemicolons: false,
  multilineImportStyle 'single',
  trailingComma: false,
  preferTypeOutside: true,
  maxImportLineLength: 120,
  /**
   * Webpack configured to use allow imports relative to the project root for anything in `src1`.
   *    - `nonModulePaths` config tells Vandelay that imports beginning with these paths should not be
   *      considered node_modules.
   *    - `processImportPath` config tells Vandelay to write paths for `src1` relative to the project root.
   */
  nonModulePaths: ['src1'],
  processImportPath: (importPath, absImportPath, activeFilepath, projectRoot) => {
    if (absImportPath.startsWith(src1)) return absImportPath.slice(projectRoot.length + 1)
  },
  processDefaultName: (filepath) => {
    if (filepath === path.join(src1, 'services/api.js')) return 'apiService'
    if (filepath === path.join(src2, 'services/cold.js')) return 'coldService'
    if (filepath === 'react') return '* as React';
  },
  shouldIncludeImport: (absImportPath, activeFilepath) => {
    return (
      !(activeFilepath.startsWith(src2) && absImportPath.startsWith(src3)) && // src2 can't improt from src3
      !(activeFilepath.startsWith(src3) && absImportPath.startsWith(src1)) && // src3 can't import from src2
      // src1 can only import from src1
      !(activeFilepath.startsWith(src1) && (absImportPath.startsWith(src2) || absImportPath.startsWith(src3)))
    )
  },
  importGroups: ['react', 'react-dom', 'redux', 'react-redux'],
}
```

## Settings
Vandelay has one setting that may be specified in your VS Code user settings:

### `autoImportSingleResult: boolean`
Defaults to `true`. When the `Import active word` command is used, the import will be automatically
written to the file if only a single result is found.
