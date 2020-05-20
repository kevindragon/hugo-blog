---
title: react初始配置
date: 2018-02-13 17:11:29
tags: [react, webpack]
---

下面webpack babel react 2018年最新最小化配置模板

<!-- more -->

package.js
```
{
    "name": "{{NAME}}",
    "version": "1.0.0",
    "description": "{{DESCRIPTION}}",
    "main": "index.js",
    "scripts": {
        "start": "webpack-dev-server",
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    "repository": {
        "type": "git",
        "url": ""
    },
    "author": "",
    "license": "LexisNexis",
    "devDependencies": {
        "babel-core": "^6.26.0",
        "babel-loader": "^7.1.2",
        "babel-preset-env": "^1.6.1",
        "babel-preset-react": "^6.24.1",
        "html-webpack-plugin": "^2.30.1",
        "path": "^0.12.7",
        "webpack": "^3.11.0",
        "webpack-dev-server": "^2.11.1"
    },
    "dependencies": {
        "react": "^16.2.0",
        "react-dom": "^16.2.0"
    }
}

```

webpack.config.js
```
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const HtmlWebpackPluginConfig = new HtmlWebpackPlugin({
  template: './client/index.html',
  filename: 'index.html',
  inject: 'body'
});

module.exports = {
    entry: './client/index.jsx',
    output: {
        path: path.resolve('dist'),
        filename: 'app.js'
    },
    module: {
        loaders: [
            { test: /\.js$/, loader: 'babel-loader', exclude: /node_modules/ },
            { test: /\.jsx$/, loader: 'babel-loader', exclude: /node_modules/ }
        ]
    },
    plugins: [HtmlWebpackPluginConfig]
};
```

.babelrc
```
{"presets": ["env", "react"]}
```

client/index.html
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>React App Setup</title>
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
```

client/index.jsx
```
import React from 'react';
import {render} from 'react-dom';
import MyComponent from './MyComponent.jsx';

class App extends React.Component {
    render () {
        return (
	    <div>
              <h1>Hello React.js</h1>
	      <MyComponent/>
	    </div>
        );
    }
}

render(<App/>, document.getElementById('app'));
```

client/MyComponent.jsx
```
import React from 'react';

export default class MyComponent extends React.Component {
    render () {
        return (
            <div>This is my component</div>
        );
    }
}
```

然后执行`npm install`安装必要的依赖

最后执行`npm start`
