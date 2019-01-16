# Why Webpack 4 / Babel 7 ?

React components are mostly written in Javascript ES6+. Older browsers cannot understand this new syntax. So, for older browsers to recognize this code, we need some kind of transformation, called **transpiling**.

Webpack doesn't know how to transform ES6+ into ES5, but it has a concept called **loaders** (think of them as transformers). A webpack loader takes something as input and produces an output.

`babel-loader` is the webpack loader responsible for transpiling your javascript code into your list of supported browsers. Obviously, this used Babel. And Babel must be configured with some presets:

1. `babel-preset-env` for transpiling ES6+ down to ES5

> Webpack 4 does not need a configuration file by default. There is no need to define neither the entry point, nor the output file. Webpack 4 takes `./src/index.js` as the default entry point. And it will spit out the bundle in `./dist/main.js`

## Install Webpack

`npm i webpack webpack-cli --save-dev`

Next, add the webpack command inside `package.json`
```javascript
"scripts": {
  "build": "webpack --mode production"
}
```

## Install Babel

`npm i @babel/core babel-loader @babel/preset-env --save-dev`

Create a new file named `.babelrc` inside the project root folder

```javascript
{
  "presets": ["@babel/preset-env"]
}
```

## Initial Webpack configuration file

Create a file `webpack.config.js` and add the following:

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      }
    ]
  }
};
```

What this does is: For every file with a `.js` or `.jsx` extension, Webpack runs the code through `babel-loader` and transpiles it down to ES5.

## Setting the Webpack development/production modes

**Production** mode enables all sorts of optimizations out of the box. Including minification, scope hoisting, tree-shaking and more.

**Development** mode on the other hand is optimized for speed and provides an un-minified bundle.

In webpack 4 you can get by without a single line of configuration! Just define the `--modeflag` and you get everything for free!

### Development

This step makes sure you don't have to type `npm run build` everytime you change a file. 

Once configured, webpack will launch your application inside a browser and everytime your save a file, it will automatically refresh the browser's window.

Install the package with:

`npm i webpack-dev-server --save-dev`

Add the start script inside `package.json`

```javascript
"scripts": {
  "start": "webpack-dev-server --open --mode development",
}
```

~~Now you can just run `npm start`. This produces a bundle (un-minified) inside `./dist/main.js`~~


### Production

This step minifies all your javascript, bundles them up, has tree-shaking, scope hoisting, defines sourcemaps, you can define **UglifyJSPlugin**, and so on.

```javascript
"scripts": {
    "build": "webpack --mode production"
}
```

Now `npm run build` adds a processed bundle inside `./dist/main.js`

### Overriding the defaults entry/output point

Just configure them in `package.json`, like so:

```javascript
"scripts": {
  "dev": "webpack --mode development ./foo/src/js/index.js --output ./foo/main.js",
  "build": "webpack --mode production ./foo/src/js/index.js --output ./foo/main.js"
}
```

## Hooking up our javascript into the HTML

For React to work we need to include a `<script>` tag inside an HTML page. 

Webpack needs two additional packages for processing HTML: `html-webpack-plugin` and `html-loader`.

Create a file `./src/index.html`

Install the dependencies with:

`npm i html-webpack-plugin html-loader --save-dev`

Then update the `webpack.config.js` file:

```javascript
const HtmlWebPackPlugin = require("html-webpack-plugin");

const htmlPlugin = new HtmlWebPackPlugin({
  template: "./src/index.html",
  filename: "./index.html"
});

module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        test: /\.html$/,
        use: [
          {
            loader: "html-loader"
          }
        ]
      }
    ]
  },
  plugins: [
      htmlPlugin
  ]
};
```

There is no need to set a `<script>` tag manually inside your `index.html` file. In our config file, we tell Webpack to do exactly that. 

## Extracting CSS

Install the dependencies:

`npm i style-loader css-loader --save-dev`

Next, create a css file for testing:

```css
/* */
/* CREATE THIS FILE IN ./src/main.css */
/* */
body {
    line-height: 2;
}
```

Configure `webpack.config.js` :

```javascript
const HtmlWebPackPlugin = require("html-webpack-plugin");

const htmlPlugin = new HtmlWebPackPlugin({
    template: "./src/index.html",
    filename: "./index.html"
});

module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        test: /\.html$/,
        use: [
          {
            loader: "html-loader",
            options: { minimize: true }
          }
        ]
      },
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"]
      }
    ]
  },
  plugins: [ htmlPlugin, ]
};
```

Finally, import the CSS in the entry point:

```javascript
//
// PATH OF THIS FILE: ./src/index.js
//
import style from "./main.css";
```

Running `npm start` or `npm run build` will output the resulting CSS in the `./dist` folder.

