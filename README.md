# vue-webpack-simple

基于webpack搭建的vue简易demo

  > 以下内容的搭建是基于webpack 3.x ,webpack 4.x的部分内容可能有所更改。

  #### 安装webpack

``` 
npm install webpack -g
```

  #### 项目初始化

  新建一个文件夹`vue-webpack-simple`

  新建`package.json`

  ```
  npm init -y 
  ```
  
  安装`vue`、`webpack`、`webpack-dev-server`

  ```
  npm install vue --save
  npm install webpack webpack-dev-server --save-dev
  ```

  根目录下新建`index.html`

  ```
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
    </head>
    <body>
        
    </body>
    </html>
  ```

  根目录下新建`webpack.config.js`

  ```
    var path = require('path');
    var webpack = require('webpack');

    module.exports = {};
  ```

  新建`src`文件夹，在`src`文件夹中新建`main.js`

  目录结构参考如下：

  ![image](https://image-static.segmentfault.com/172/240/1722404829-5a55f92773d82_articlex)

  #### js模块化

  `src`目录下新建文件`util.js`

  ```
    module.exports = function say() {
      console.log('hello world!');
    }
  ```

  修改`main.js`

  ```
    var say = require('./util');
    say();
  ```

  修改`webpack.config.js`

  ```
    var path = require('path');
    var webpack = require('webpack');

    module.exports = {
      entry: './src/main.js',//项目的入口文件，webpack会从main.js开始，把所有依赖的js都加载打包。
      output: {
        path: path.resolve(__dirname, './dist'), //项目的打包文件路径
        publicPath: '/dsit/',//通过devServer的访问路径
        filename: 'build.js',
      },
      devServer: {
        historyApiFallback:true,
        overlay:true
      }
    }
  ```

  修改`package.json`

  ```
    "scripts": {
      "dev" : "webpack-dev-server --open --hot",
      "build" : "webpack --progress --hide-modules"
    }
  ```

  > 注意：webpack-dev-server会自动一个静态资源web服务器，--hot参数表示启动热更新

  修改`index.html`文件，引入打包后的js文件

  ```
    <!DOCTYPE html>
    <html lang="en">

    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
    </head>

    <body>
        <script src="/dist/build.js"></script>
    </body>

    </html>
  ```

  执行`运行`命令

  ```
    npm run dev
  ```

  如无意外，可以看到浏览器会自动打开一个页面，在控制台中会输出`hello world！`，尝试随意修改`util.js`中的内容，会发现浏览器会自动刷新。

  执行`打包`命令

  ```
    npm run build
  ```
  可以看到自动生成了一个`dist`文件夹，里面就又打包好的`build.js`

  ![image](https://image-static.segmentfault.com/656/204/656204522-5a55f92726b24_articlex)

  > webpack 默认不支持转码es6，但`import` `export`这两个语法却单独支持，所以我们可以改写前面的模板化写法

  `util.js`

  ```
    export default function say() {
      console.log('hello world!');
    }
  ```

  `main.js`

  ```
    import say from './util';
    say();
  ```

  #### 引入Vue

  下面尝试引入Vue进行开发（暂不考虑单文件组件）

  `main.js`
  
  ```
    import Vue from 'vue';

    var app = new Vue({
      el: "#app",
      data: {
        message:"hello Vue!"
      }
    });
  ```

  `index.html`

  ```
    <!DOCTYPE html>
    <html lang="en">

    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
    </head>

    <body>
        <div id="app">
            {{ message }}
        </div>
        <script src="/dist/build.js"></script>
        
    </body>

    </html>
  ```

  `webpack.config.js`

  ```
    var path = require('path');
    var webpack = require('webpack');

    module.exports = {
        entry: './src/main.js',
        output: {
            path: path.resolve(__dirname, './dist'),
            publicPath: '/dist/',
            filename: 'build.js'
        },
        devServer: {
            historyApiFallback: true,
            overlay: true
        },
        resolve: {
            alias: {
                'vue$': 'vue/dist/vue.esm.js'
            }
        }
    };
  ```

  > 重新运行`npm run dev`，可以看到页面正常显示`hello Vue`

  #### 引入css和scss

  webpack默认只支持js的模块化，如果需要把其他的文件也当成模块引入，则需要相对应的`loader`解析器

  ```
    npm install node-sass css-loader vue-style-loader sass-loader --save-dev
  ```

  `webpack.config.js`

  ```
    var path = require('path');
    var webpack = require('webpack');

    module.exports = {
        entry: './src/main.js',
        output: {
            path: path.resolve(__dirname, './dist'),
            publicPath: '/dist/',
            filename: 'build.js'
        },
        devServer: {
            historyApiFallback: true,
            overlay: true
        },
        resolve: {
            alias: {
                'vue$': 'vue/dist/vue.esm.js'
            }
        },
        module: {
            rules: [
                {
                    test: /\.css$/,
                    use: [
                        'vue-style-loader',
                        'css-loader'
                    ],
                }
            ]
        }
    };
  ```

  解释：

  ```
    {
        test: /\.css$/,
        use: [
            'vue-style-loader',
            'css-loader'
        ],
    }
  ```

  > 这段代码意思是：匹配后缀名为`css`的文件，然后用`css-loader`,`vue-style-loader`去解析

  >> 解析器的执行顺序是从下往上，从右往左（先`css-loader`再`vue-style-loader`）

  > 请注意，因为以上案例是基于`vue`开发，所以使用`vue-style-loader`，其他情况使用`style-loader`

  `css-loader`使得我们可以用模块化的方式引入`css`，`vue-style-loader`会将引入的`css`插入到`html`文件的`style`标签里面

  引入`scss`也是同样的配置写法：

  ```
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    'vue-style-loader',
                    'css-loader'
                ],
            },
            {
                test: /\.scss$/,
                use: [
                    'vue-style-loader',
                    'css-loader',
                    'sass-loader'
                ],
            },
            {
                test: /\.sass$/,
                use: [
                    'vue-style-loader',
                    'css-loader',
                    'sass-loader?indentedSyntax'
                ],
            }]
    }
  ```

  try

  在`src`文件夹下新建`style`文件夹，`style`文件夹内新建`common.scss`

  ```
    body {
        background: #ccc;
    }
  ```

  `main.js`

  ```
   import './style/common.scss';
  ```

  如无意外，可以看到css的样式已经生效了

  #### 使用babel转码

  目前主流浏览器大部分都已经支持ES6的语法，但还有部分浏览器或者部分版本的浏览器不支持，所以需要通过`babel`将ES6转码成为ES5的语法

  ```
    npm install babel-core babel-loader babel-preset-env babel-preset-stage-3 --save-dev
  ```

  配置：

  在项目的根目录下新增一个`.babelrc`文件

  ```
    {
      "presets": [
        ["env", { "modules": false }],
        "stage-3"
      ]
    }
  ```

  在`webpack.config.js`中添加一个`loader`

  ```
    {
        test: /\.js$/,
        loader: 'babel-loader',
        exclude: /node_modules/
    }
  ```

  > exclude 表示忽略`node-modules`文件夹下的文件，不需要转码

  try

  `util.js`

  ```
    export default function getData() {
      return new Promise((resolve,reject) =>{
        resolve("ok");
      })
    }
  ```

  `main.js`

  ```
    import getData from "./util";
    import Vue from "vue";

    import "./style/common.scss";

    var app = new Vue({
      el: "#app",
      data: {
        message: "hello Vue!"
      },
      methods:{
        async fetchData(){
          const data = await getData();
          this.message = data;
        }
      },
      created() {
        this.fetchData();
      }
    })
  ```

  正常情况下，控制台会报错误`regeneratorRuntime is not defined`，因为我们上面安装`babel`的时候并没有安装`babel-polyfill`

  ```
    npm install babel-polyfill --save-dev
  ```

  修改`webpack.config.js`文件的入口

  ```
    entry:["babel-polyfill", "./src/main.js"],
  ```

  再执行`npm run dev`，发现可以正常运行了

  > 原因：Babel默认只转换新的JavaScript句法（syntax），而不转换新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如Object.assign）都不会转码。

  >> 举例来说，ES6在Array对象上新增了Array.from方法。Babel就不会转码这个方法。如果想让这个方法运行，必须使用babel-polyfill，为当前环境提供一个垫片。

  >>> Babel默认不转码的API非常多，详细清单可以查看babel-plugin-transform-runtime模块的[definitions.js](https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/definitions.js)文件。

  #### 引入图片资源

  把图片也当成模块引入

  ```
    npm install file-loader --save-dev
  ```

  `webpack.config.js`也添加对应的配置

  ```
    {
        test: /\.(png|jpg|gif|svg)$/,
        loader: 'file-loader',
        options: {
            name: '[name].[ext]?[hash]'
        }
    }
  ```

  在`src`目录下新建一个`img`文件夹，存放一张照片`logo.png`

  修改`main.js`

  ```
    import getData from './util';
    import Vue from 'vue';

    import './style/common.scss';


    Vue.component('my-component', {
      template: '<img :src="url" />',
      data() {
        return {
          url: require('./img/logo.png')
        }
      }
    })

    var app = new Vue({
      el: '#app',
      data: {
        message: 'Hello Vue !'
      },
      methods: {
        async fetchData() {
          const data = await getData();
          this.message = data;
        }
      },
      created() {
        this.fetchData()
      }
    });

  ```

  修改`index.html`

  ```
    <!DOCTYPE html>
    <html lang="en">

    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
    </head>

    <body>
        <div id="app">
            {{message}}
            <my-component/>
        </div>
        <script src="/dist/build.js"></script>
        
    </body>

    </html>

  ```

  可以看到，图片能够被正确地加载了。

  #### 单文件组件

  在前面的例子里面，我们使用`Vue.component`来定义组件，在实际项目中，更推荐使用单文件组件

  ```
    npm install vue-loader vue-template-compiler --save-dev
  ```

  > 注意：这里使用的`vue-loader`是13.x.x的版本，15.x.x以上的版本，必须带有VueLoaderPlugin 

  在`webpack.config.js`中添加对应的`loader`

  ```
    {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {
            loaders: {
                'scss': [
                    'vue-style-loader',
                    'css-loader',
                    'sass-loader'
                ],
                'sass': [
                    'vue-style-loader',
                    'css-loader',
                    'sass-loader?indentedSyntax'
                ]
            }
        }
    }
  ```

  在`src`目录下新建文件`App.vue`

  ```
    <template>
      <div id="app">
        <h1>{{ msg }}</h1>
        <img src="./img/logo.png">
        <input type="text" v-model="msg">
      </div>
    </template>

    <script>

    import getData from './util';

    export default {
      name: 'app',
      data () {
        return {
          msg: 'Welcome to Your Vue.js'
        }
      },
      created() {
        this.fetchData();
      },
      methods: {
        async fetchData() {
          const data = await getData();
          this.msg = data;
        }
      }
    }
    </script>

    <style lang="scss">
    #app {
      font-family: "Avenir", Helvetica, Arial, sans-serif;

      h1 {
        color: green;
      }
    }
    </style>
  
  ```

  `main.js`

  ```
    import Vue from 'vue';
    import App from './App.vue';

    import './style/common.scss';

    new Vue({
      el: '#app',
      template: '<App/>',
      components: { App }
    })

  ```

  `index.html`

  ```
    <!DOCTYPE html>
    <html lang="en">

    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
    </head>

    <body>
        <div id="app"></div>
        <script src="/dist/build.js"></script>
    </body>

    </html>
  ```

  执行命令`npm run dev`，可以看到单文件被正确加载了。

  #### source-map

  在开发阶段，调试也是非常重要的一个环节。
  
  `App.vue`

  ```
    created() {
        this.fetchData();
        console.log("23333");
    }
  ```

  打开控制台，可以看到

  ![image](https://image-static.segmentfault.com/327/597/327597702-5a55f926ee703_articlex)

  点击这个console.log的输出地址

  ![image](https://image-static.segmentfault.com/138/311/1383111606-5a55f9270291e_articlex)

  可以看到进入的是打包后的`build.js`文件，这个时候，文件已经被压缩，难以判断该代码是在哪个组件上面写的，调试的难度较大

  为了便于调试，我们修改`webpack.config.js`文件

  ```
    module.exports = {
      entry: ["babel-polyfill","./src/main.js"],
      //省略其他...
      devtool:"#eval-source-map"
    }
  ```

  重新`npm run dev`

  可以看到

  ![image](https://image-static.segmentfault.com/296/582/2965820798-5a55f926f3b0a_articlex)

  > 这个时候看到的是当前组件的代码，而不是打包后的代码了。

  #### 打包发布

  执行命令`npm run build`

  ![image](https://image-static.segmentfault.com/410/274/4102746376-5a55f926db050_articlex)

  这个时候，看到打包后的`build.js`文件有587KB，对于当前项目的代码量来讲，这个体积有点过大了，我们尝试下优化

  ```
    npm install cross-env --save-dev
  ```

  > cross-env能跨平台地设置及使用环境变量

  >> 大多数情况下，在windows平台下使用类似于: NODE_ENV=production的命令行指令会卡住，windows平台与POSIX在使用命令行时有许多区别（例如在POSIX，使用$ENV_VAR,在windows，使用%ENV_VAR%。。。）cross-env让这一切变得简单，不同平台使用唯一指令，无需担心跨平台问题




  修改`package.json`

  ```
    "scripts": {
        "dev": "cross-env NODE_ENV=development webpack-dev-server --open --hot",
        "build": "cross-env NODE_ENV=production webpack --progress --hide-modules"
    }
  ```

  在这里我们设置了环境变量，通过开发环境或者生产环境来判断是否需要压缩代码

  修改`webpack.config.js`，判断当处于生产环境的时候，压缩js代码

  ```
    var path = require('path');
    var webpack = require('webpack');

    module.exports = {
        // 省略...
    }

    if (process.env.NODE_ENV === 'production') {
        module.exports.devtool = '#source-map';
        module.exports.plugins = (module.exports.plugins || []).concat([
          new webpack.DefinePlugin({
            'process.env': {
              NODE_ENV: '"production"'
            }
          }),
          new webpack.optimize.UglifyJsPlugin(),
        ])
    }
  
  ```

  再尝试打包`npm run build`

  ![image](https://image-static.segmentfault.com/174/634/1746349625-5a55f926d74e5_articlex)

  可以看到，`build.js`的文件大小从584KB >> 188KB ，压缩的效果还是比较明显的。
