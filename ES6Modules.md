# ES6 Modules

Previously, to use `javascript` in a website, we _downloaded and linked_ js files manually using __script tags__ inside `html` files.
> js file can be your custom written js
or it can be written by someone else, available to donload as a library

This enabled linking _js files present in project dir_, using _relative paths_

To include and use js code written by someone else, i.e. `3rd party libraries`\
we need to _download the __library's js file__ in our project dir_, and _include it_ with `script` tags
> The disadvantage to this is _keeping track of updates_ to 3rd party packages we use, and needing to __download them manually everytime__

## Package manager

The _disadvantage_ to above approach is _overcome by using package managers_ like `npm`, `yarn`, `pnpm`.\
Package managers __automate__ _the downloads and upgrades_ of js packages from a __central repository__

> __`npm` was initially designed to use with `node` on the _backend_, not on _frontend_,[^npm]__\
and `yarn` came later as _alternative to npm_\
These package managers can be interracted with _using the cli_

### Getting started with npm

1. Install `node.js`, `npm` comes along with it
2. `cd` into your project _root dir_
3. run `npm init`
    > This will generate a `package.json`, a _configuration file_ for npm. Information related to the project is stored here\
    Optionally run with the `-y` flag to skip all the questions, defaults are good enough
4. run `npm install <package name>`
    > see the package documentation before running.

    This will _download_ all package code into `node_modules` folder in your project root dir\
    and automatically _add the package name, along with version_ to the `dependencies` section of `package.json`
    > Node modules folder can become very large quickly and is added to `.gitignore` file.
     When sharing a project with other people, just share `package.json`. Now someone else can install these packages by running `npm install`
5. Now that we have the _package files downloaded_ inside the _local `node_modules`_, we can _link to that files_ in the script tags
    > We still need to look for downloaded files and __link them manually__ in the _html_

## Modularization in js

_Organizing code in multiple files_ was different in js, compared to other languages that have `import`-`export` syntax for loading files\
This is because `js` was designed to _run in browser_ with __no filesystem access for security reasons__\

The way _code sharing_ worked was by __manually__ _loading each file separately_ and using _global variables_ so that _other files can access_ them\
As in above example, the js files are downloaded into node_modules, then loaded in html, this leads to creation of a global variable named after the package\
This global variable is available to any other file if needed

Later `CommonJS` came out, and _specified JS ecosystem outside browsers_\
The _CommonJS module specification_ allowed the use of `import-export` statements instead of the _global variables solution_\
`node.js` is well known _implementation of CommonJS modules_ which is designed to __run `js` on server__ and can __access server's filesystem__\
So instead of _loading scripts in html_, using __node.js modules__ we can _load scripts in `js` files_ directly

```js
var moment = require('moment')
```

> Node.js __knows each module's path__, so we can write `moment` _instead of specifying full path_ in `require(...)`

This is _good for use on the server_, i.e. backend,\
but if we _use this in browser_, we get __`require` not defined__\
Because __browser can't access filesystem__, these files _need to be loaded dyanamically_, using synchronous[^sync] or asynchronous[^async] methods

__This problem of filesystem access is solved by `module bubndler`__

## Module Bundlers

These are tools that introduce a `build step` which _can access filesystem_, thus solving above problem.\
The `build step` _generates a final output_ that is _browser compatible_ by __replacing `require` statements[^bs-i] with actual code[^bs-v] in these files__\
This _single output file is then loaded in html_

> So now, we use `require` statements on frontend, use a bundler to introduce build step, and get browser compatible js\
This automates the manual loading of scripts in html and and using global variables for code sharing

`Browserify` came out first, later `webpack` became popular due to it's use in `react`

## Webpack

1. Install the `webpack` and `webpack-cli` npm package as a dev dependency

    ```bash
    $ npm install webpack webpack-cli --save-dev
    Installs webpack as dev dependency and the cli tool to interact with it
    ```

2. Use the installed binary to bundle your js file

    ```bash
    $ ./node_modules/.bin/webpack index.js --mode=development
    ```

    > This will look for require statements in index.js, replace them with appropriate code, and by default, generate final output in `dist/main.js`
    The mode=development flag tells webpack to generate a dev friendly, readable output, instead of a minified output which can be generated for the production environment
3. Load this output in html using script tag

    ```html
    ...
    <script src="dist/main.js"></script>
    ...
    ```

Now, everytime the index.js file changes, _we need to re run the webpack command_
To _avoid specifying the same options again and again_, we can add a `webpack.config.js` to our project root\
Then just run the webpack binary without any command line options

For the above command, config could look like this

```js
// webpack.config.js
module.exports = {
  mode: 'development',
  entry: './index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

> Note: due to addition of a build step, we can now add other powerful features that can be added to the workflow. See below features

---

### Transpiling to add new language features

Adding new features take time as browsers are slow in this aspect
The solution to this is to add new features, in `js` or in another language like `sass`, `typescript` etc and transpile[^transpile] these to browser compatible languages\
ex. sass -> css, typescript -> javascript

`babel` is a transpiler that takes latest features[^ES2015] in js and transpiles them to browser compatible versions of js.

We can use babel with our webpack build step

1. Install babel npm package

    ```bash
    $ npm install @babel/core @babel/preset-env babel-loader --save-dev
    ```

    installs 3 separate packages as dev dependencies — `@babel/core` (the main part of babel), `@babel/preset-env` (preset defining which new JavaScript features to transpile) and `babel-loader` (package to enable babel to work with webpack)

2. Configure webpack to use babel-loader

    ```js
    // webpack.config.js
    module.exports = {
        mode: 'development',
        entry: './index.js',
        output: {
            filename: 'main.js',
            publicPath: 'dist'
        },
        module: {
            rules: [
                {
                    test: /\.js$/,
                    exclude: /node_modules/,
                    use: {
                        loader: 'babel-loader',
                        options: {
                            presets: ['@babel/preset-env']
                        }
                    }
                }
            ]
        }
    };
    ```

    > This tells webpack to look for js files except those in node_modules, perform transpilation using `babel-loader` with options defined in `@babel/preset/env`

3. Use features released after ES2015 in your project files, re run the webpack binary, see the changes in effect
4. Cross check the generated ouput file to see if transpilation worked

## Task runner

_Task runners_ are tools that _automate parts of build process_
Common `tasks` in frontend development include __minifying code, optimizing images, running tests etc__

Popular task runners like `grunt`, `gulp` use _pluggins that wrap other tools_\
But these days, `npm scripts` are used which _work with other tools directly_

We can modify `package.json` and add `npm scripts` to make working with webpack easier

```js
{
    scripts: {
        ...
        "build": "webpack --progress --mode=production",
        // this runs webpack, minifies output for production, and shows the progress %

        "watch": "webpack --progress --watch"
        // this will re run webpack everytime files js change
    }
}
```

> as node.js knows path of each module, we can run webpack without specifying full path inside `scripts`

Run these scripts with `npm run watch` and `npm run build`

We can also add webpack live server to serve the app contents

- `$ npm install webpack-dev-server --save-dev`
- add a `serve` script in `package.json`

    ```diff
    {
        scripts: {
            ...
    +       "serve": "webpack-dev-server --open"
        }
    }
    ```

- start the server with `npm run serve`

We can write npm scripts for running other tasks as well, such as converting Sass to CSS, compressing images, running tests —\
_anything that has a command line tool_

---

[^sync]: can slow execution speed
[^async]: can have timing issues
[^bs-i]: invalid js syntax in case of browser
[^bs-v]: valid js syntax in case of browser
[^npm]: see module bundlers section to see how they enabled use of require statements(which was node.js specification), thus making npm preferred for frontend too
[^transpile]: convert code from one language to another similar language
[^ES2015]: next gen features that came after es2015
