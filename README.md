# Vue Server Side Rendering

Vue Server Side Rendering or SSR for short renders all the code in the server rather than on the client, increasing the load times for your website. It saves all the code on the server. When the user clicks on your website instead of having to download all of the HTML, CSS and JavaScript the HTML and CSS will already be saved on the server, so the user does not have a blank screen while waiting for the it all to load. After the page loads it then adds the JavaScript required to add proper functionality to your website. This increases the speed your website will load and the responsiveness of the site. This is especially useful for large web applications using a large amount of JavaScript and other programming languages


## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.

### Setting Up Vue SSR

Setting up Vue SSR involves a lot of steps
1.	Install the needed dependencies
2.	Webpack configuration
3.	NPM build scripts
4.	App configuration
5.	Setting up Vue Router
6.	Client entry point
7.	Server entry point
8.	Server configuration

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

## Main Vue file configuration
We also need to provide a new app instance that is responsible for starting the router and root app. It should look like this:
```
// main.js
import Vue from 'vue'
import App from './App.vue'
import { createRouter } from './router/router.js'

// export a factory function for creating fresh app, router and store
// instances
export function createApp() {
  // create router instance
  const router = createRouter();

  const app = new Vue({
    router,
    // the root instance simply renders the App component.
    render: h => h(App)
  });

  return { app, router };
}
```

## Client Entry Point
We import the dependencies we need
We create the app from the ```main.js``` file and mount it to a node with the id of ```app```

```
//client-entry.js
import { createApp } from './main.js';

const { app } = createApp()

// this assumes App.vue template root element has `id="app"`
app.$mount('#app')
```

## Server Entry Point
This is the webpack server entry build, and it contains what we need to target when we configure the server
```
//server-entry.js
import { createApp } from './main.js';

export default context => {
  // since there could potentially be asynchronous route hooks or components,
  // we will be returning a Promise so that the server can wait until
  // everything is ready before rendering.
  return new Promise((resolve, reject) => {
    const { app, router } = createApp();

    // set server-side router's location
    router.push(context.url);
      
    // wait until router has resolved possible async components and hooks
    router.onReady(() => {
      const matchedComponents = router.getMatchedComponents();
      // no matched routes, reject with 404
      if (!matchedComponents.length) {
        return reject({ code: 404 });
      }
  
      // the Promise should resolve to the app instance so it can be rendered
      resolve(app);
    }, reject);
  });
}
```

## Configuring and starting the server
Finally we configure and start up the express server
```
//server.js
const express = require('express');
const server = express();
const fs = require('fs');
const path = require('path');
//obtain bundle
const bundle =  require('./dist/server.bundle.js');
//get renderer from vue server renderer
const renderer = require('vue-server-renderer').createRenderer({
  //set template
  template: fs.readFileSync('./index.html', 'utf-8')
});

server.use('/dist', express.static(path.join(__dirname, './dist')));

//start server
server.get('*', (req, res) => { 
    
  bundle.default({ url: req.url }).then((app) => {    
    //context to use as data source
    //in the template for interpolation
    const context = {
      title: 'Vue JS - Server Render',
      meta: `
        <meta description="vuejs server side render">
      `
    };

    renderer.renderToString(app, context, function (err, html) {   
      if (err) {
        if (err.code === 404) {
          res.status(404).end('Page not found')
        } else {
          res.status(500).end('Internal Server Error')
        }
      } else {
        res.end(html)
      }
    });        
  }, (err) => {
    console.log(err);
  });  
});  

server.listen(8080);
```

## Acknowledgments

* Based off the tutorial found here https://vuejsdevelopers.com/2017/12/11/vue-ssr-router/
