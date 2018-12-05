# Vue Server Side Rendering

Vue Server Side Rendering or SSR for short renders all the code in the server rather than on the client, increasing the load times for your website. It saves all the code on the server. When the user clicks on your website instead of having to download all of the HTML, CSS and JavaScript the HTML and CSS will already be saved on the server, so the user does not have a blank screen while waiting for the it all to load. After the page loads it then adds the JavaScript required to add proper functionality to your website. This increases the speed your website will load and the responsiveness of the site. This is especially useful for large web applications using a large amount of JavaScript and other programming languages


## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Setting Up Vue SSR

Setting up Vue SSR involves a lot of steps
1.	Install the needed dependencies
2.	Webpack configuration
3.	NPM build scripts
4.	Folder structure
5.	App configuration
6.	Setting up Vue Router
7.	Client entry point
8.	Server entry point
9.	Server configuration

### Installing the needed dependencies

In the command terminal in visual studio code, you need to install the vue-cli and initialize the webpack-simple and vue-ssr

```
npm install -g vue-cli
vue init webpack-simple vue-ssr
cd vue-ssr
npm install i webpack@3.11.0
npm install
```
We also need to add the Vue library for SSR and SPA, we also need to have a NodeJS server running and a webpack merge to merge the webpack configuration
```
npm install vue-server-renderer vue-router express webpack-merge --save
```

## Webpack configuration
In the webpack.config.js file, change the entry to ```entry-client.js```
Create a new file called ```webpack.server.config.js``` and add the server webpack configuration
```
var path = require('path')
var webpack = require('webpack')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.config')
var webpackConfig = merge(baseWebpackConfig, {
  target: 'node',
  entry: {
    app: './src/entry-server.js'
  },
  devtool: false,
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'server.bundle.js',
    libraryTarget: 'commonjs2'
  },
  externals: Object.keys(require('./package.json').dependencies),
  plugins: [
    new webpack.DefinePlugin({
      'process.env': 'production'
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      }
    })
  ]
})
module.exports = webpackConfig
```

### Building the scripts
Inside the ```package.json``` need to then build the entries for both the client and server and then start the server

```
"scripts": {
  "start": "npm run build && npm run start-server",
  "build": "npm run build-client && npm run build-server",
  "build-client": "cross-env NODE_ENV=production webpack --progress --hide-modules",
  "build-server": "cross-env NODE_ENV=production webpack --config webpack.server.config.js --progress --hide-modules",
  "start-server": "node server.js"
}
```

### Index.html

Right now, your index.html file should look like this

```
<!doctype html>
<html lang="en">
<head>
  <!-- use triple mustache for non-HTML-escaped interpolation -->
  {{{ meta }}}
  <!-- use double mustache for HTML-escaped interpolation -->
  <title>{{ title }}</title>
</head>
<body>
    <!--vue-ssr-outlet-->
  <script src="dist/build.js"></script>
</body>
</html>
```

## App.vue

This is the root component of the web app
  1. It configures the a menu with Vue Render links
  2. It sets the container for the route components to render
  3. It sets an element with the id ```app``` that mounts the client side part of the application

It should look something like this
```
<template>
  <div id="app">
    Hello World!
    <p>
      <router-link to="/">Go To Home</router-link>
      <router-link to="/about">Go To About</router-link>
    </p>
    <router-view></router-view>
  </div>
</template>
<script>
  export default {
  };
</script>
```
  
## Router File Configuration
Our application code starts on the server so we need to provide a new instance of the router for each server request. Your router file should look like this:
```
// router.js
import Vue from 'vue';
import Router from 'vue-router';
import Home from '../components/Home.vue';
import About from '../components/About.vue';

Vue.use(Router);

export function createRouter () {
  return new Router({
    mode: 'history',
    routes: [
      { path: '/', component: Home },
      { path: '/about', component: About }
    ]
  });
}
```

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags). 

## Authors

* **Billie Thompson** - *Initial work* - [PurpleBooth](https://github.com/PurpleBooth)

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* Hat tip to anyone whose code was used
* Inspiration
* etc
