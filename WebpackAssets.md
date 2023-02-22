# Asset Management Webpack

## Manage assets like images, stylesheets with webpack

Before webpack, _`grunt` or `gulp`_ were used to process these assets and JS modules, and move them from `src` to `dist`\
Now webpack dynamicaly generates modules, producing a __dependency graph__, so all modules _explicitly state their __dependencies___\
This way, _unused modules_ are _not bundled_ when generating output

With webpack, we can also include other types of files, not just js\
That is why _webpacks' advantages also apply to everything needed to build a website_

To bundle other types of `resources`, we use `loaders`,
which can be chained, i.e. each loader will transform resources being processed and pass it's output to next loader in chain, at the end of the chain, js is expected by webpack\
`Regex` are used to determine which _files to __look up and serve__ to loaders_\
ex. look up all css files and serve to style-loader and css-loader

## Load CSS

1. To enable `css` file imports in Js modules, _install and add_ the `style-loader` and `css-loader` to module configuration
and serve all css files to them

    ```bash
    npm install --save-dev style-loader css-loader
    ```

    ```diff
    module.exports = {
            ...

    +  module: {
    +       rules: [
    +           {
    +               test: /\.css$/i,
    +               use: ['style-loader', 'css-loader'],
    +           },
    +       ],
    +  },
    };
    ```

    > Above configuration looks up all css files and serves them to each loader in chain\
    > The order, 1. style loader and 2. css loader must be preserved to avoid errors

2. Now import './*.css' into a Js module that __depends__ on this file

    When this Js module is run(eg. in html), a `style` tag with stringified css version of the imported file is added to `head` tag in html

3. run `npx webpack --watch` or add `webpack --watch` to npm scripts

4. Open `index.html` in browser and inspect to see if css was added to head tag in `style`

> NOTE: in production, [minimize the css](https://webpack.js.org/plugins/mini-css-extract-plugin/#minimizing-for-production) and improve load times\
[Loaders for all flavors of css](https://webpack.js.org/plugins/mini-css-extract-plugin/#minimizing-for-production) are available such as Sass, postcss etc

## Load images

1. Use built in `Asset modules` to load images\
Modify the module config as below

    ```diff
    +      {
    +        test: /\.(png|svg|jpg|jpeg|gif)$/i,
    +        type: 'asset/resource',
    +      },
    ```

2. Now we can import images, these will be __processed and added to output dir__(final output)

    If we import it as `import myImage from './image.jpeg'`,
    `myImage` will then hold the _path/url of the processed image_ in the output dir

    any `url('./image.jpeg')` or `<img src='./image.jpeg'>` found in css(if using a css loader) or html will realize this is a local file and will swap the url with the final images' url

## Load Fonts

1. Just like css files are processed through loaders, _any asset file processed through __Asset module__ is included in dist output_

2. To add _fonts_, add following code to `module` config

    ```diff
    +      {
    +        test: /\.(woff|woff2|eot|ttf|otf)$/i,
    +        type: 'asset/resource',
    +      },
    ```

3. Add font files to `src/`

4. Now use them in css inside `@font-face` declaration

    The `url('xyzfont.woff')` code inside the declaration will be _resolved by webpack just like images_,\
    i.e. webpack replaces the _local path inside url_, and swaps for the _path of processed file_(in dist/)

    ```css
    @font-face {
        font-family: 'MyFont';
        src: url('./my-font.woff2') format('woff2'),
            url('./my-font.woff') format('woff');
        font-weight: 600;
        font-style: normal;
    }

    .hello {
        color: red;
        /* use declared font family in style rules */
        font-family: 'MyFont';
        background: url('./icon.png');
    }
    ```

## Loading data

Json is supported by _default_.
Just add a json file in `src/` and `import JsonData from './myFile.json'`

1. CSV, TSV, XML files can be imported using `csv-loader` and `xml-loader` npm packages

    ```bash
    npm install --save-dev csv-loader xml-loader
    ```

2. Change module config

    ```diff
    +      {
    +        test: /\.(csv|tsv)$/i,
    +        use: ['csv-loader'],
    +      },
    +      {
    +        test: /\.xml$/i,
    +        use: ['xml-loader'],
    +      },
    ```

3. Add files to `src/` and `import` them
    `import data from './xyz.xml';`

4. Now `data` variable declared above will contain _parsed JSON data_, which can be used in file

### To parse ALL: yaml, toml, json5 etc as JSON modules, use `custom loaders` instead of specific webpack loaders

1. Install custom loaders,

    ```bash
    npm install toml yamljs json5 --save-dev
    ```

2. Configure them in webpack config

    ```diff
    + const toml = require('toml');
    + const yaml = require('yamljs');
    + const json5 = require('json5');


    module.exports = {
        ...
        module: {
            rules: {
                ...
    +           {
    +               test: /\.toml$/i,
    +               type: 'json',
    +               parser: {
    +                   parse: toml.parse,
    +               },
    +           },
    +           {
    +               test: /\.yaml$/i,
    +               type: 'json',
    +               parser: {
    +                   parse: yaml.parse,
    +               },
    +           },
    +           {
    +               test: /\.json5$/i,
    +               type: 'json',
    +               parser: {
    +                   parse: json5.parse,
    +               },
    +           },
            }
        }
    }
    ```

3. Add files to `src/`

    ```bash
    touch abc.toml
    touch xyz.yaml
    ```

4. `import` and use in code

    ```js
    import tomlData from './abc.toml'
    import yamlData from './xyz.yaml'
    ```

---

## Loading assets this way allows you to group modules and assets in a more intuitive way

Instead of grouping everything inside global /assets dir, we can group assets along with the modules that depend on them\
This way, closely coupled code lives together.

Ex

```diff
- /assets
+ /components
+ |__ /hero-section
+ | |__ /index.jsx
+ |   |__ /index.css
+ | |__ /icon.svg
+ | |__ /product-image.png
```

Now if this component is to be reused in another codebase/project,
just copy the components with all contents,\
install and configure loaders for whatever assets exist in the dir,
import and use the component, assets anywhere in different project

This is for when assets are unique to different components..
when assets are shared between multiple components, it is possible to store these in a base dir, then use `aliases` for easier imports

---
